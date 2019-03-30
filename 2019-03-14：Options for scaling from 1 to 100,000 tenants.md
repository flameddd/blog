## Options for scaling from 1 to 100,000 tenants
## https://www.citusdata.com/blog/2018/06/28/scaling-from-one-to-one-hundred-thousand-tenants/
## 2018/06/28

Sharding 這做法很成熟了，先聽聽別人的經驗是很好的起步（去 youtube 查有很影片）  
Sharding seems straightforward enough on the surface:  

- Re-architect your application to be sharding aware
- Handle edge cases for multi-node differences
- Expand your DevOps team to manage tens, hundreds, thousands of separate databases

一步步往下挖後，你會了解到，上面這些主題各自有又涵蓋不同的小區域  
這些 edge cases 從 `single node` 變成 `sharded setup` 架構時變得很繁重  

### consistency model
為了思考 consistency model，你可能要 rebuild logic 讓 App 直接 transactions，因為資料 span 在 shards

### failures
某個節點 offline 時該怎麼辦，這時候你允許其他 transactions 成功還是失敗？

### connections
要允許 App 直接 connect DB 嗎？還是要有 proxy/pooling layer？特定節點需要 hotspots 嗎？
 
逐步思考下來，你要 run 自己的 sharding 架構，好像變成 yearlong engineering project 了  
你的 best developers 離開了 App 戰線，轉來處理 DB 這些 infra，而不是改善一線的客戶需求、features。  
競爭力好像又趕不上，一堆事情要考慮...

### 轉到 NoSQL
有聽到人開始說，NoSQL 才能 scale，relational DB 沒法玩。NoSQL 為了跟 RDBS 競爭，做出很多讓步。  
如果你（的需求）可以玩 NoSQL，那可能有這些優勢

- Constraints and referential integrity
- Transactional guarantees
- Ability to easily analyze your data (SQL)

所以如果你一開始是 RDBS，現在要 scale 的話 NoSQL 能幫你  
1. don’t have the above requirements 
2. can invest in all the application changes needed to detangle your relational model.

### Adopt a solution that takes care of sharding for you, at the database layer
sharding 來 scales out 是最 flexibility 的。不需要 locked in。  
最大的問題就是是否要 re-architect App 來在 App layer 來做 shard？  
還是用 **Citus extension to Postgres** 來 scale out、用 database layer 處理。  
 
實務上，很難搞 re-architect App。Citus 比較現實，用 Citus 能  

- Stick with an open source foundation、避免 lock-in
- 不必在 in-App 裡面去 create(and maintain) 複雜的 sharding logic
- 用 Citus 的方案，減少很多工，比起 re-architect App 方案，整體減少 70% 工(早6個月上線)
- 每年減少 40萬鎂 的維護費


> 看完這段，有點小知識。Sharding 以前聽過，看來是很實用的方案。但也沒聽過有人在 App 層做。  
另外是 Citus 是常見的方案嗎？  

### sharding 是對數據進行橫向擴展的一種方式
數據量增加，加一台機器，來擴展其容納能力和處理能力。  
Sharding其實需要解決三個問題：
1. 數據的路由
2. 數據的分片
3. 分片的元數據信息保存。

#### 數據的路由
**數據路由是 DB 告訴應用程序**，你讓我查的數據目前在哪個分片上，這條路怎麽走過去。
#### 數據的分片
數據分片就是**實際數據的存放地點**，往往每個分片就是一台單獨的服務器（含存儲）。
#### 分片的元數據信息保存。
由於分片的數據實際是被切割放在不同的機器上，那麽**需要有個集中的地點存放數據分片的信息，即分片元數據的信息**。

應用問路由怎麽走，路由去查詢元數據得知需要的數據在哪個分片上，最終應用訪問到該分片上。  
最著名的sharding database就是mongoDB了。mongoDB的sharding功能的架構也是為了解決上面的三個問題。  


#### Shardcat (這可能是 Oracle or Mongo 的專有名詞)
是非常重要的一個模塊，上面有  
- 分片的元數據資訊
- duplicated table 的 master table資訊  

另外，當進行 cross shard query 的時候，他有 `coordinator database` 的作用。  
所以建議對這個部分搭建RAC+adg架構，避免shardcat的單點故障。  

#### shard node
單個shard node的失效，將導致整個表的不可用。  
所以也要對 shard node 建立高可用的副本，這裏可以用ADG或者OGG的技術。  

#### 機器
做sharding，又要在做HA，那麽就變成了堆機器，堆存儲的方式了。  
我們假設在一個10個shard node的環境，需要多少台機器？：
- 一個shardcate，做rac+adg，最少就是3台
- 10個shard node，如果都有adg，那麽最少就是20台。

那麽當前這個環境，就至少要23台機器了。

#### 挑戰
Sharding 架構的挑戰，**極其考驗對應用的熟悉程度**，需要配合應用進行合理的分區和分片。另外  
- sharding key必須建索引
- sharding的方式可以有一致性hash，讓數據均勻分布
- 也還是可以是range或者list分區，或者hash-range，hash-list的子分區。

**分片**和**分區**方式需要結合業務:  
- 有些場景需要相關數據都在一個分區，避免cross shard join
- 有些場景需要均勻分片，禁止集中分片，導致熱塊數據都在一個分片上

（如序列增長，做range分區，熱點數據將會都在一個分片上）。

#### 事實表和維度表
事實表和維度表，似乎可以很好的利用sharding功能：
- 維度表: 做duplicated table
- 事實表: 做sharded table