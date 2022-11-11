# Jason Haddix: The Bug Hunter’s Methodology v4.02 Recon
## Jason Haddix 2020年8月8日
### https://www.youtube.com/watch?v=gIz_yn0Uvb8
#### https://github.com/jhaddix/tbhm

Recon 的 Methodology  

這個影片算是上半部，下半部是 application analysis
- 但 Jason 還沒有講過 talk
- 但最近有 slide 放出來
  - https://docs.google.com/presentation/d/1cMSRVlJJ5de6Pyv-09YgzOGS0OYrP6p7ggGl0f42wmw/edit#slide=id.p

Jason
- wordlist: https://gist.github.com/jhaddix/86a06c5dc309d08580a018c66354a056
- The Web Application Hacker's Handbook: https://gist.github.com/jhaddix/6b777fb004768b388fefadf9175982ab#miscellaneous-tests
------------------------------

做 Recon 時
- 要有工具 track and 記錄你的結果

作者用
- `Xmind`: mind map 讓他 visualize large scrope bug hunting targets
- 一些 note 工具，寫自己的 checklist

總之，你需要有工具來紀錄自己 recon 的進度  

作者對 Tesla 的 mind map
- 左邊 autonomous system numbers, Acquisitions, reverse WHOIS
- 右邊 CNAME

![](https://i.imgur.com/BBnrnJd.png)  

開始 recon 後，漸漸變成這樣
- 這些都是 live domain, subdomin
- 每個 node (`https://www.tesla.com`) 有 check list，還有自己疑問的問題ㄜ
- 用顏色標記是非常非常重要的，目前在哪邊、檢查了沒之類的

![](https://i.imgur.com/LXbd3RQ.png)  


## Wide Recon
目標是盡可能地找到更多目標相關的 assets

方向
- Scope Domains
- Acquisitions
- ASN Enumeration
- Reverse WHOIS
- Subdomain Enumeration
- Port Analysis
- Other
  - Vulns: Subdomain Takeover, Buckets, Github leaks
  - Automation/Helper: Interlace, screenshotting, framsworks

你必須要弄清楚，所有對方擁有的 website  
- 找到越多、對 infra 更了解，你的機會就更大  

### Finding Seeds/Roots
第一步是先去解讀 program
- https://bugcrowd.com/tesla
  - (看來有多個平台可以多看看，像 hackerone )
  - https://hackerone.com/pornhub?type=team

從這邊看
- https://bugcrowd.com/tesla

你的第一組 seed domain 就是
- `*.tesla.com`
- `*.tesla.cn`
- `*.teslamotors.com`
- `*.tesla.services`
- `*.teslainsuranceservices.com`
- `*.solarcity.com`

這邊就是起點

### Acquisitions
第二步，找到 Acquisitions  
- www.crunchbase.com
  - 輸入關鍵字、找到公司頁面後，點選 `Acquisitions`  

可能可以查看看
- 是不是有些被收購的公司，其 infra 服務有沒有可能有漏洞，infra 可能還沒下線  
- 有沒有一些 Subdomain

這些 acquisitions 有機會提供更多 infra and site 的資訊，讓我們有機會攻擊  
例如，搜尋 `twitch`
- 最近幾年有併購 A 公司，從 crunchbase 裡面點選，會連到 A 公司的頁面
- 就可以考慮把 A 公司的 domain 也放到目標裡面

也記得要去 project page 確認，有沒有在目標範圍之內  

### Autonomous system numbers
找到幾個 seed domain 後，來找 autonomous system numbers
- 公司夠大的話，基本上都會申請 ASN
- ASN 會知道你的 IP range
- 用公司名稱查 ASN
- https://bgp.he.net/  <- 手動查詢

但這邊就沒有 cover cloud 的 IP  
大公司通常會找 cloud host service

ASNS cli tool
- metabigor: https://github.com/j3ssie/metabigor
- Asnlookup: https://github.com/yassineaboukir/Asnlookup (diff data source)

這些都是用 company name keyword 去搜尋  
結果出來時，要確認到底是不是在目標範圍之內  

目前為止，我們
- 有一些 seed domain
- ASNS 也拿到一些 IP

繼續，嘗試拿到更多 seed domain

`Amass`: 
- 用 `ASN` 來查 seed domain
- https://github.com/OWASP/Amass
- user guide: https://github.com/OWASP/Amass/blob/master/doc/user_guide.md
- e.x. `amass intel -asn 46489` 這樣有機會查更多 seed domain

![](https://i.imgur.com/8dePdse.png)


### Reverse WHOIS
- https://www.whoxy.com/
- 輸入 domain，會提供其他相關的公司 and domain and subdomain


DomLink, `WHOXY.com` cli tool
- https://github.com/vysecurity/DomLink
- DomLink is a tool that uses a `domain` name to discover organisation name and associated e-mail address to then find further `associated domains`.


### Ad/Analytics relationships
看哪些網站使用相同的 Ad/Analytices code  
- https://builtwith.com/

搜尋後，切到 `Relationship profile` tab 去  
這樣能看到哪些 domain 也使用相同的 Ad/Analytics Id  
藉此找出更多關聯  

cli
- https://raw.githubusercontent.com/m4ll0k/Bug-Bounty-Toolz/master/getrelationship.py
  - https://github.com/m4ll0k/BBTz/blob/master/getrelationship.py
 
### 也可以用 Google 來
- Copyright text
- Terms of service text
- Privacy policy text 

Google 搜尋: `"© 2022 Twitch Interactive, Inc." inurl:twitch`
- `inurl` 返回的網頁鏈接中包含第一個關鍵字，後面的關鍵字則出現在鏈接中或者網頁文檔中
  - 也可以搜尋 2021, 2020 等，也是有機會  

來找些相關的網站  

### shodan
- https://www.shodan.io/
  - 提供 hosts information from an infrastructure-based
  - 左邊列出 `Top Organizations`
  - 右邊，能看看 SSL certification 的 位置，說不定在 target 範圍  
- 也能找 seed domain

- 能找去關於


### Finding Subdomains

你找到越多 seed domain -> 就能找到越多 Subdomain -> 就能找到越多 site 能探索  

三招來找 Subdomain
- Linked and JS Discovery
- Subdomain Scraping
- Subdomain Bruteforce


Linked Discovery ( https://youtu.be/gIz_yn0Uvb8?t=1470 )
- Burp suite Pro 的功能
- 訪問 seed domain ，然後把裡面的所有 sub domain 連結找出來
  - 一層層一直往下找
- (Burp suite Pro 這功能也會順便找 seed domain)
- (這個似乎沒有其他工具，只有 Burp pro 的樣子 ...)

誤會了，還是有人寫 cli tool 來跑 linked discovery
- `GoSpider`
  - https://github.com/jaeles-project/gospider
- `hakrawler`
  - https://github.com/hakluke/hakrawler

Subdomain Enumeration
- 繼續 enum 更多 domain

SubDomainizer
1. Find Subdomain referenced in JS file
2. Find cloud service referenced in JS files
3. Use the [Shannon Entropy](https://hackmd.io/@sXG2cRDpRbONCsrtz8jfqg/ry-0k0PwH) find potentially sensitive items (API key) in JS file

SubDomainizer 
- https://github.com/nsonaniya2010/SubDomainizer
- demo: https://www.youtube.com/watch?v=whWL_1t1JbU

Subdomain Scraping
- 概念是，去網路上一些服務，直接輸入 domain, like `www.github.com`，然後 parse 結果看看能不能挖出什麼來

![](blob:https://imgur.com/49213adc-9a08-48fd-bfe5-6b6c1a8b9938)  

舉例，去 google
- `site:twitch.tv -www.twitch.tv -watch.twitch.tv -dev.twitch.tv`

這樣找出來我們還不知道的 Subdoamin

Amass 能做這點(這個我之前有玩過！難怪跑起來蠻久的)
- https://github.com/OWASP/Amass

Subfinder 也是做這個的（這個我也玩過）  

Jason 的流程，兩套都會跑，然後 cat sort 合併結果   
Jason 又另外介紹了幾個工具  
- 一個是專門 github scraping 的，他說跟上面的結果不一樣，所以他會用另一套另外跑
- 另一個跑起來比較快，也偶額會有不一樣的結果
- 另外有一招是，當你知道 ip 時，就去掃 ip range ，然後看 response 回應的 certification 是不是跟我們目標的一樣，是的話，也很可能是目標，但掃 ip range 是很花費時間的  
  - Jason 説這招有機會找到很多東西


第三招 Subdomain Bruteforce
- bruteforce 很吃 wordlist，不然效果不好
- Amass 有支援，要用 `-rf` 來用多個 DNS resolvers 來加速，不然很久
  - (想想，當 wordlist 有萬時 ...)

Amass 用 `emum` (工具)，來實現 bruteforcing
- `amass enum -brute -d twitch.tv -src`
  - 這樣就有內建的 wordlist
- `amass enum -brute -d twitch.tv -rf resolvers.txt -w bruteforce.list`
  - 指定自己的 wordlist

有自己的 wordlist 當然更好，甚至是針對目標的 wordlist
- TomNomNom great talk: https://www.youtube.com/watch?v=W4_QCSIujQ4


commonspeak2 這個是是用 bigquery 去產的，有產出類似 最常見的 subdomain data
- https://github.com/assetnote/commonspeak2
  - (Jason 有把 commonspeak1 加到他自己的 wordlist 中)  

Favicon analysis (`favfreak`)
- 把 favicon hash 過後來搜尋  

`FavFreak`:
- https://github.com/devanshbatham/FavFreak
- https://medium.com/@Asm0d3us/weaponizing-favicon-ico-for-bugbounties-osint-and-what-not-ace3c214e139

![](https://camo.githubusercontent.com/27db16257374afee3ee940ce6e5f35d3e84029eb070b3b8c56d416f55570c8e1/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f313230302f312a737176314b4c6f354242614c4b5347535577465566772e706e67)  


### Port Analysis
- `masscan`
  - https://danielmiessler.com/study/masscan/

```
masscan -p1-65535 -iL $ipFile --max-rate 1800 -oG $outPutFile.log
```

`masscan` 只掃 ip，所以要用類似 `dnmasscan` shell 轉 ip 
- https://github.com/rastating/dnmasscan/blob/master/dnmasscan

Jason 的流程是
1. `masscan` scan 所有找到的 domain and Subdomain
2. 把結果用 `Nmap` (`scan -oG`) 去產生 OG outputfile
3. 用 `Burtespray` 掃描，來找 service，像是 ssh, ftp, pop3 ...  
    - https://github.com/x90skysn3k/brutespray


### Github Dorking 
這段 Jaons 是手動的，他的 script
- https://gist.github.com/jhaddix/1fb7ab2409ab579178d2a79959909b33

另外他推薦 talk
- GitHub Recon and Sensitive Data Exposure
- https://www.youtube.com/watch?v=l0YsEk_59fQ


### Screbshotting 
- `Eyewitness`, `Aquatone`, `httpscreenshot`  

`Aquatone` 似乎最多人用，Jason 用 Eyewitness  
看了 screenshot 之後，來思考哪個我們要優先研究  

### Subdomain takeover
類似 Github, Heroku 這樣的服務，讓 User 自己 depoly 頁面在 Subdomain 上  
一旦這位 User 移除了帳號之類的，而又有其他人之前有連結到這邊  
這時候你去建立這個帳號，就等於接管了這個 Subdomain  

這邊有整理各服務，並且討論每個服務該怎麼測試找到這類目標
- https://github.com/EdOverflow/can-i-take-over-xyz

`nuclei`
- 丟 list 進去，能掃出可能 take over 的 Subdomain
- https://github.com/projectdiscovery/nuclei
- https://github.com/projectdiscovery/nuclei-templates
 

## Automation++

`interlace` 能幫助你整合這些工具的流程   
- https://github.com/codingo/Interlace
- 因為有些工具不好 thread、input output 的格式問題等等  

TomNomNom 所寫的所有工具
- Jason 直接這樣點名說他的 talk、工具都推薦  

## Framework
很多人會整合自己的工具  
Jason 把這些分成 C, B, A, S 等級，S 即代表整合度很高、有 GUI，有各種環境等等  
- https://youtu.be/gIz_yn0Uvb8?t=4035

但 Jason 自己用的大多是 C 等級的   
Jason 用這段
- https://github.com/venom26/recon/blob/master/ultimate_recon.sh
- 他說因為這用了 `Nuclei` 

C
```
https://gist.github.com/dwisiswant0/5f647e3d406b5e984e6d69d3538968cd
https://github.com/AdmiralGaust/bountyRecon
https://github.com/offhourscoding/recon
https://github.com/Sambal0x/Recon-tools
https://github.com/JoshuaMart/AutoRecon
https://github.com/yourbuddy25/Hunter
https://github.com/venom26/recon/blob/master/ultimate_recon.sh
```

(B ~ S 不一一列出了，剩下的去看影片吧)  

Recon flow
- https://pentester.land/cheatsheets/2019/03/25/compilation-of-recon-workflows.html
- 這邊有列出大家的 recon flow  

