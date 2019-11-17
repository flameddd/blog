## Microsoft Azure traning event

Migrage 的流程應該是 4 段並行反覆做
- rehost (life and shift)
- refactor
- rearchitect
- rebuild

如果只是搬上去
- VM 的 IO 跟 local 的 VM 絕對不一樣
  - Azure 傳統硬碟是 500 iops, SSD 更高

rehost 之前，一定要先用 rearchitect  

case
VM 的 iops 的預設只有 500  
客戶 IIS 上面跑 20個網頁，這樣上去 iops 就會讓你效能很低  

要把 App 轉 microservics
- 不會推薦把舊有 App 轉為 microservices
  - 很費工，工非常大

實務上最花時間的是
- reflow

Azure Container Instances
- 最適合的情境是`任務性質`、`臨時性`的 case
- 不適合每日、長時間的 taske
- 跑完，就要多一段程式把 instances 砍掉

Azure Container Registry
- 好處可以做到 Geo-replication

範例
- https://github.com/Azure-Samples/acr-build-helloworld-node

實在太不熟 Azure 了  會恍神  
聽到一個 `virtual kubelet` 是什麼？  
k8s 創始人寫的，已 open 到 CNCF 去  

下午講的太商務了
- 用這個可以幫你們公司省錢  之類的...  

聽不太下去...  
一方面也是自己沒有更專心  