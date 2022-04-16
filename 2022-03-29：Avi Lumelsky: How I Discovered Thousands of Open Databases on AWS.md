# 2022-03-29：Avi Lumelsky: How I Discovered Thousands of Open Databases on AWS
## Avi Lumelsky Jan 23
### source: https://infosecwriteups.com/how-i-discovered-thousands-of-open-databases-on-aws-764729aa7f32


## Overview
掃描 託管服務的 [CIDR (無類別域間路由)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) blocks (IP ranges)，是很容易在 cloud 上找到 misconfigured 的資源的，因為它們是已知的並由它們發布。

作者發現上千個 `ElasticSearch` 和 `Kibana` 能被存取，上面都洩漏很多敏感資訊，如
- 有關客戶的敏感信息：電子郵件、地址、當前職業、工資、私人錢包地址、位置、銀行賬戶和其他敏感信息。
- Kubernetes cluster 寫的 Production log，這包含存 application 到 system 的 log、node、pod 都有

## Background
那些已發佈的 CIDR blocks 上，一定還有很多 assets，IT 常見的錯誤設定如下:
- 把 socket binding 到錯誤的 network interfaces
  - e.x 監聽來自 `0.0.0.0/*` 的連接，這樣它對所有的網路接口，就是可見的狀況，不只是內部的 IP 位置 (`172.x.x.x`)
- 對 cluster 的 security group 設定錯誤，允許 CIDR blocks 上所有的 `TCP` 和所有的 `UDP` (有時候可能是事後其他人改的，但沒意識到後果)
- 使用默認網絡或 subnet，派生 subnet 設置，並 silently 分配公共 IPv4 地址

## 假設
假設如果我們從 cloud operator (An Instance / VPS) 中掃描特定的 CIDR 塊，我們可以很容易地找到 misconfigured assets，，其中主要的原因是由於人為錯誤。  
關鍵是 scan smartly，利用已知的 network infrastructure (known CIDR-blocks for the software we like to scan) 來找到 live servers  

稍微找一下，就能找到每家 cloud provider 相關的 CIDR blocks
- AWS: https://docs.aws.amazon.com/general/latest/gr/aws-ip-ranges.html
  - https://ip-ranges.amazonaws.com/ip-ranges.json

假如我是 IT，需要允許「特定 cloud」的 service 的 incoming connections，例如 AWS 的 Elastic Container Service (ECS)。可以透將將 service 的 CIDR 加入到 Security Group rules 來允許這些 CIDR blocks 的 connections

```shall
wget -O- https://ip-ranges.amazonaws.com/ip-ranges.json | jq -r '[.prefixes[] | select(.service=="AMAZON").ip_prefix] - [.prefixes[] | select(.service=="ES").ip_prefix] | .[]'
```

上面的指令會過濾出 ES 的服務 IP
```
2022-03-29 13:42:28 (9.73 MB/s) - 已輸出到 stdout [1217311/1217311]

3.5.140.0/22
13.34.37.64/27
13.34.65.64/27
43.224.79.154/31
43.224.79.174/31
...
```

有些 CIDR blocks 只能從 cloud provider 內部訪問。 你只要在要掃描的 cloud provider 內啟動一個具有互聯網連接的 VPS / instance 就能拿到了  

## 需要哪些東西才能找到它們？
- 對網絡、IP stack 和 routing 以及 cloud infrastructure 有基本的了解
-  tool for Port Scanning (like MasScan, or NMap)  
    - https://github.com/robertdavidgraham/masscan
    - https://nmap.org/
- 要掃描的 CIDR 列表（託管服務——如 Kubernetes 或 ElasticSearch）以及最有可能在這些 IP 範圍內的 instances 上打開的 port
- 可視化我們收集的所有 data 的工具（如 ElasticSearch+Kibana）


## Port Scanning — Collecting the data about the assets
- 作者用 `MasScan` 掃描選定的 CIDR blocks
- MasScan is a TCP port scanner
- The input is the CIDR blocks (e.g 50.60.0.0/16 or 118.23.1.0/24) and the ports we would like to scan (9200, 5600, 80, 443, etc.).

用 Docker 起 ELK stack，用 MasScan 掃描。MasScan 是 streaming output 到 ElasticSearch，然後用 `LogStash` 和 `Kibana` 視覺化所有東西  

幾個小時後，作者在 AWS 的 ElasticSearch service CIDR blocks (customers clusters) 查到上千個 open 的 port  
![](https://miro.medium.com/max/1400/1*c_wmJwIe7P-c7fN0fFG7JA.png)  

然後，從 Kibana 匯出成 CSV，用 [pandas](https://pandas.pydata.org/) 載入  

![](https://miro.medium.com/max/552/1*JmgBJHRtCA6aLXObyQ4rNA.png)  

對每一組 IP，作者送了一個 HTTP HEAD request 來拿到 HTTP response 和 asset 的 fingerprint  

![](https://miro.medium.com/max/778/1*cw9OUliReATo2rJC8eNr-w.png)  
![](https://miro.medium.com/max/874/1*F_1Whdkh0USVh2u7P54FIg.png)  
![](https://miro.medium.com/max/704/1*fTdby2gUlfDOHC7vFTO9Cw.png)  

然後把 HTTP response 的 web page title 印出來  
![](https://miro.medium.com/max/588/1*spRQfef0vxfyWkf297TA6w.png)  
![](https://miro.medium.com/max/1400/1*oDBq3BmgJzWyJpEYkVayVg.png)  

這樣就有機會看到這些 後台的 web tilte  
![](https://miro.medium.com/max/1400/1*NxOo3DS3ZzyWxGvxSwIqyw.png)  

接著，作者就針對特定的 address，利用 browser 找看看這些東西  
- `/_aliases`
- `/<index>/_search`

這個 ElasticSearch REST API 非常方便。 很容易拿到 fields metadata, documents count, and everything you wanted to know about the ES cluster.  

![](https://miro.medium.com/max/1400/1*ogTOoUE4GwuJyehg6kixag.png)  

透過這樣挖，作者找到
- Kubernetes 整個 production clusters log ...
- 能成功 SSH login 的 private key
- 有些公司把 Jira 的 tasks and issue 傳到 ElasticSearch 去，這裡面包含 客戶 data、code example、account name。有個 Kibana dashboard 包含有關公司研發的所有 data。
- 銀行轉賬以及全名和銀行賬戶詳細信息
- 找到一些已經被勒索軟體串改的資料，這些都代表這些服務已經被黑了

## 結論
- 在 port 掃描方面，為每個 service 發布 CIDR blocks 是一個邏輯問題。 我們需要它的原因有很多——但同時它也給雲提供商帶來了巨大的風險，讓他們的客戶可以輕鬆掃描。
- 錯誤配置一直在發生，不知不覺中造成了公司安全的許多漏洞。
- 我們在 config instance 時經常會使用 default 的 VPC subnet。 因此，許多 instance 會自動分配公共 IP 地址。
- 如果你 public open 你的 service，至少要使用授權和身份驗證。

## 其他
隨手看了一下上面 AWS 的 CIDR 資料裡面的 service 總類  
1. "AMAZON"
1. "CHIME_VOICECONNECTOR"
2. "ROUTE53_HEALTHCHECKS"
3. "S3"
4. "DYNAMODB"
5. "EC2"
6. "ROUTE53"
7. "CLOUDFRONT"
8. "GLOBALACCELERATOR"
9. "AMAZON_CONNECT"
10. "ROUTE53_HEALTHCHECKS_PUBLISHING"
11. "CHIME_MEETINGS"
12. "CLOUDFRONT_ORIGIN_FACING"
13. "CLOUD9"
14. "CODEBUILD"
15. "API_GATEWAY"
16. "EBS"
17. "EC2_INSTANCE_CONNECT"
18. "WORKSPACES_GATEWAYS"
19. "ROUTE53_RESOLVER"
20. "AMAZON_APPFLOW"
21. "KINESIS_VIDEO_STREAMS"

## 自己玩
Dockerize it. I had same issue and with docker works like a charm
https://hub.docker.com/r/ilyaglow/masscan/dockerfile

docker run ilyaglow/masscan -p80,443 --rate 1000 --banners IPBLOCK

待會應該
```
docker pull ilyaglow/masscan
```

這樣跑嗎？
```
docker run ilyaglow/masscan -p80,443 --banners --range 13.34.37.64/27
```

我怎麼看結果？  <-- 我下面這樣掃 youtube
```
docker run ilyaglow/masscan -p80,443 --banners --range 172.217.163.46
```

會有這樣的結果
```
Discovered open port 443/tcp on 172.217.163.46
Discovered open port 80/tcp on 172.217.163.46
```


後面用 zx 玩，拿到這個結果
```
Discovered open port 443/tcp on 3.5.141.179
Discovered open port 443/tcp on 3.5.143.161
Discovered open port 80/tcp on 3.5.141.179
Discovered open port 80/tcp on 3.5.143.161
```

用這個指令 request IP 然後拿 title
```
wget -O- https://3.5.141.179/ --no-check-certificate | grep -oE "<title>.*</title>"
```

curl 只能 domain address
```
curl 'https://www.patterns.dev' -so - | grep -oE "<title>.*</title>"
```

後續持續寫 zx 來學習  