## GCP meetup
### google cloud next
- 去年 2.5 萬  
- 今年 3 萬人參加  
  - 成長很多  
- 強調 hybrid cloud, serverless  
- 可以透過 google GKE(google k8s engine) 的工具去部署 Azure, AWS  
  - anthos-overview 有文件教
- InfluxDB 是什麼？最近常聽到

- google could run => serverless base run container  
  - 有流量才收錢，down to zero
  - 可run在 k8s 上

### google 合作的 open source
- Neo4j
- InfluxData
還有好幾個

### storage 的分法
- 越便宜的方案，通常取出來的費用越貴

### cloud run
cloud run vs cloud run on GKE
這兩個不一樣  
- cloud run 偏向 cloud function
  - run HTTP services and App 快速 autoscale
  - 非部署在 VM 上，是 container  (run on Google infra)
  - 可以用各式開發語言（因為是 container)
- cloud run on GKE
  - run 在 GKE (google k8s)
  - 要先建立 GKE
  - 我已經有k8s, GKE 了，那我想用 serverless 的話，所以我用 cloud run on GKE
- GAE 是 VM，底層就不一樣

### Anthos
- 可以管理 AWS, Azure 第三方的東西  
- GPC 上可以管理自己的東西，跟 On-Prem 的東西
- 轉過來一定要是 container
- 即時遷移至 GKE
