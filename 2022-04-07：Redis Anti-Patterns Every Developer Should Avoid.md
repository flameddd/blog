# 2022-04-07：Redis Anti-Patterns Every Developer Should Avoid
## Ajeet Raina, Developer Growth Manager at Redis
### https://developer.redis.com/howtos/antipatterns/
---------------------
Redis 官方寫的 anti patterns

1. 在單個分片/Redis 實例上運行的大型數據庫#
由於大型數據庫在單個分片/Redis 實例上運行，故障轉移、備份和恢復都可能需要更長的時間。因此，始終建議將分片保持在推薦的大小。一般保守的經驗法則是 25Gb 或 25K Ops/Second。

如果您有超過 25 GB 的數據和大量操作，Redis Enterprise 建議進行分片。另一個方面是，如果你每秒有超過 25,000 次操作，那麼分片可以提高性能。由於每秒操作次數較少，它也可以處理高達 50GB 的數據。

### 在單個 shard/Redis instance 上跑大型 db
大型 db 跑在單一 shard/Redis instance 時
- 故障轉移、備份和恢復都可能需要更長的時間
- 推薦將 shards 保持在 25Gb 或者 25K Ops/Second


有超過 25 GB 的 data 和大量操作
- Redis Enterprise 建議進行 shard
- 另方面是，如果你每秒有超過 25,000 次操作，sharding 可以提高 perf
  - 由於每秒操作次數較少，它也可以處理 50GB 的 data
  
### 2. 每個 request 都建立 connection

每個 operation 都建立 connection
- 會有大量 cost 來建立和斷開 TCP connection

作為 developer，您有時可能會 create a connection, run a command, and close the connection
- 雖然技術上是可行的，但它非常糟，並且不必要地降低了「整個 Redis」的 perf
- 建議使用 connection pool (Jedis) 或具有響應式設計的 Redis client (Lettuce)。

使用 OSS Cluster API。developer 會在任何給定時間用 connection 到不同 node
- 使用 Redis Enterprise、 connection 實際上是一個代理，它負責處理複雜 cluster level connection

示例 #1 - redis-py #
讓我們看一下使用連接池來管理與 Redis 服務器的連接的 redis-py。默認情況下，您創建的每個 Redis 實例都會依次創建自己的連接池。您可以通過將已創建的連接池實例傳遞給 Redis 類的 connection_pool 參數來覆蓋此行為並使用現有連接池。您可以選擇這樣做以實現客戶端分片或對連接的管理方式進行細粒度控制。

Examples #1 - `redis-py`
- 可以通過將已建立的 connection pool instance 傳給 Redis connection pool 參數來覆蓋 default 行為（建立新的 connectino pool），此行為並使用現有 connection pool

```
>>> pool = redis.ConnectionPool(host='localhost', port=6379, db=0)
>>> r = redis.Redis(connection_pool=pool)
```

Example #2 - Lettuce
Lettuce
- Lettuce 提供通用 connection pool
- Lettuce connection 被設計為 thread-safe 的
  - 因此一個 connection 可以在多個 thread 間共享
- default 情況下 Lettuce 連接會自動重新 connection

```
 RedisClient client = RedisClient.create(RedisURI.create(host, port));

 GenericObjectPool<StatefulRedisConnection<String, String>> pool = ConnectionPoolSupport
               .createGenericObjectPool(() -> client.connect(), new GenericObjectPoolConfig());
 
 // executing work
 try (StatefulRedisConnection<String, String> connection = pool.borrowObject()) {

    RedisCommands<String, String> commands = connection.sync();
    commands.multi();
    commands.set("key", "value");
    commands.set("key2", "value2");
    commands.exec();
 }

 // terminating
 pool.close();
 client.shutdown();
```


### 3. 直接連 Redis instances
當有大量的 client 時，reconnect flood 發生時，很容易就把 single thread 的 Redis 弄死，然後 force a failover，所以建議用些工具減少 redis server 的 open connections 

[Redis Enterprise DMC proxy](https://docs.redis.com/latest/rs/administering/designing-production/networking/multiple-active-proxy/)
- 通過充當代理來減少與 cache server 的連接數

[Twemproxy](https://github.com/twitter/twemproxy)
-  Twtter 的 open source
- Redis 跟 Memcache proxy server
- 用來管理 Redis 和 Memcached cluster，減少 cache server connection 數量

這些，加上 protocol pipelining and sharding enables
- 這樣能夠 horizontally scale your distributed caching architecture.


### 4. 兩個以上的「二級」shard (Redis OSS)
Redis OSS 使用 shard-based quorum
- 建議使用至少 3 個 data 副本（一個 master shard，搭配 2 個 replica shards）來防止 `split-brain`
- 簡單來說，Redis OSS 用奇數個 shards（主 + 2 個副本）來解決仲裁問題 (quorum challenge)

`Redis Enterprise`
- 不需要奇數也能處理 quorum challenge
- 這樣更省成本
- In addition, the so-called ‘quorum-only node' can be used to bring a cluster up to an odd number of nodes if an additional, not necessary data node would be too expensive.

5. 執行單個操作#
連續執行多個操作會增加連接開銷。相反，使用Redis Pipelining。流水線是在管道中發送多條消息而不等待每個消息的回复的過程 - 並且（通常）在回復進入時稍後處理。

流水線完全是客戶端實現。它旨在解決高網絡延遲環境中的響應延遲問題。

因此，通過網絡發送命令和讀取響應所花費的時間越少越好。這可以通過緩衝有效地實現。在將命令發送到服務器之前，客戶端可能（也可能不會）在 TCP 堆棧中緩衝命令（如其他答案中所述）。一旦它們被發送到服務器，服務器就會執行它們並在服務器端緩衝它們。流水線的好處是大大提高了協議性能。通過流水線獲得的加速範圍從連接到 localhost 的 5 倍到較慢的 Internet 連接的至少 100 倍。

### 5. 執行單一 operation
連續執行多個 operations 會增加 connection 開銷。 [Redis Pipelining](https://redis.io/topics/pipelining)，pipeline 是一種發送多個訊息，但不等待每個訊息回覆的過程。另外，當回覆進來時，通常會稍後處理。

Pipelining 是 client side 實現的。目標是解決 network 延遲很大的時候，回應延遲的問題  


6. 緩存沒有 TTL 的鍵#
Redis 主要用作鍵值存儲。可以在這些鍵上設置超時值。也就是說，超時到期會自動刪除密鑰。此外，當我們使用刪除或覆蓋密鑰內容的命令時，它會清除超時。Redis TTL 命令用於獲取密鑰過期的剩餘時間，以秒為單位。TTL 返回具有超時的鍵的剩餘生存時間。這種自省功能允許 Redis 客戶端檢查給定密鑰將繼續成為數據集的一部分的秒數。密鑰將累積並最終被驅逐。因此，建議在所有緩存鍵上設置 TTL。

### 6. Caching keys 沒有 TTL
Redis 主要的功能是 key-value store，能針對這些 keys 設一個 timeout
- 當 timeout 時，自動刪除 key
- 另外，當我們用指令刪除或覆蓋某個 key 時，它也會清除那些 timeout
- Redis 取的 TTL 剩下多久時間，是用「秒」為單位
- Redis client 檢查給 TTL 剩多少，最終會把這些 key 刪除
- 因此，建議在「所有」 caching keys 設 TTL

7. 無休止的 Redis 複製循環#
當嘗試通過緩慢或飽和的鏈接複製非常大的活動數據庫時，由於持續更新，複製永遠不會完成。因此，建議調整從屬和客戶端緩衝區以允許較慢的複制。看看這個詳細的博客。

### 7. 無限 Redis Replication loop
當嘗試複製很大、正在活動的 db 時
- 由於 db 持續更新，所以複製永遠不會完成
- 建議調整 slave 和 client buffer 允許較慢的複製
詳情: https://redis.com/blog/the-endless-redis-replication-loop-what-why-and-how-to-solve-it/


8. 熱鍵#
Redis 可以輕鬆成為您應用運行 data 的核心，保存有價值且經常訪問的信息。

但是，如果您將訪問集中到不斷訪問的幾條 data 上，就會產生所謂的熱鍵問題。

在 Redis 集群中，鍵實際上決定了 data 在集群中的存儲位置。根據散列該鍵， data 存儲在一個單一的主要位置。

因此，當您一遍又一遍地訪問單個密鑰時，​​您實際上是在一遍又一遍地訪問單個節點/分片。讓我們換一種說法——如果你有一個由 99 個節點組成的集群，並且你有一個密鑰可以在一秒鐘內收到一百萬個請求，那麼所有這些數百萬個請求都將發送到一個節點，而不是分佈在其他 98 個節點上。

Redis 甚至提供了查找熱鍵所在位置的工具。使用帶有 –hotkeys 參數的 redis-cli 以及您需要連接的任何其他參數：

### 8. Hot Keys
Redis 很容易成為你的 application 的 data 的核心，儲存那些有價值、經常被 query 的 data  
但，如果集中訪問到不斷被訪問的幾條 data 是，就會產生所謂的 `hot-key` 問題  

在 Redis cluster 架構中，key 就決定了哪個 cluster 儲存了這筆 data  
- 根據 hashing key， data 儲在一個單一的主要位置

所以，當一次次訪問單一一個 key 時
- 實際上是一次次訪問同一個  node/shard 

例如，假設 cluster 有 99 node
- 某一筆資料一秒被訪問了 100 萬次
- 這 100 萬次 request 都到那一台 node 了，其他 98 台都沒用


Redis 指令查找 hot key 所在位置

```
$ redis-cli --hotkeys
```

在可能的情況下
- 最好在開發時就避免來造成這情況
- data 用多個 key，寫道不同的 shards 能避免問題
  - 換句話說，每個 client 都可以訪問特定的 key
  - 因此，建議 shard out hot keys using hashing algorithms
  - 可以設定 policy 為 `LFU`，另外 `redis-cli --hotkeys` 來確認
  


### 9. 使用 Keys 指令
Redis 中，`KEYS` 指令對所有 stored keys 執行 exhaustive match pattern
- 不建議使用
- 某些 instance 上有大量 keys 時，就很耗時間、也拖慢 instance perf
- 從 relation db 的角度來看，這相當於 select 一個沒有 where 的查詢

要小心使用這個操作
- 甚至要想辦法確認 app 沒有使用 KEYS
- 用 `SCAN` 指令，多呼叫幾次，把查詢拆散成幾個 call 來執行，避免一次佔用整台 server

掃 key 是非常慢的，是 O(N)、N 是 key 的數量，建議使用 RediSearch 根據 data 的內容返回資料  
而不是掃過所有 key

```
FT.SEARCH orders "@make: ford @model: explorer"
 2SQL: SELECT * FROM orders WHERE make=ford AND model=explorer"
```


Redis 通常用作應用程序的主存儲引擎。與使用 Redis 作為緩存不同，使用 Redis 作為主數據庫需要兩個額外的功能才能生效。任何主數據庫都應該真正具有高可用性。

如果緩存出現故障，則通常您的應用程序處於欠壓狀態。如果主數據庫出現故障，您的應用程序也會出現故障。同樣，如果緩存出現故障並且您將其重新啟動為空，那也沒什麼大不了的。但是，對於主數據庫來說，這是一筆巨大的交易。

Redis 可以輕鬆處理這些情況，但它們通常需要與作為緩存運行不同的配置。Redis 作為主數據庫很棒，但您必須通過打開正確的功能來支持它。

使用 Redis 開源，您需要設置 Redis Sentinel 以獲得高可用性。在 Redis Enterprise 中，它是一個核心功能，您只需要在創建數據庫時打開它。至於持久性，Redis Enterprise 和開源 Redis 都通過 AOF 或快照提供持久性，因此您的實例以您離開它們的方式啟動。

### 10. 執行 Ephemeral Redis 作為主要的 db
Redis 常常被用來當作 application 的主要 store。不像用來當作 cache 時，當 Redis 被用來當作 db 有兩個特性非常重要
- db 要是高度 available
  - cache 出事，頂多拖慢 app，但 db 出事，整個 app 死掉

只要調整設定，Redis 當作是 db 也沒問題  

使用 Redis open source 時，只要設定 `Redis Sentinel` 就有 high availability  
`Redis Enterprise` 中，這是核心功能，在 create db 時設定就好  

關於 durability，`Redis Enterprise` 和 `open source Redis` 都有 AOF 或 snapshotting instances 的功能，來 persistence


### 11. 把 JSON blobs 存在 string
有很多東西可能無法一致性的 組成/解開 JSON，這就會需要 application 那邊寫 logic 來確認 key 是否有 update，而這些 JSON 操作通常很耗時
- 建議使用 `HASH` 資料結構，和 `RedisJSON` module


### 12. 沒有考慮 query pattern 的情況下，把 table or JSON 轉為 HASH
唯一的查詢機制是 `SCAN`
- 它需要讀取 data structure 並將過濾限制為 `MATCH` 指令
- 建議將 table 或 JSON 存為 string。使用 `SET` 或 `SORTED SET` 將 index 分解為反向 index 並指向 sting 的 key

在一個 Redis instance 中使用 `SELECT` 和多個 db

Salvatore（Redis creator）提到
- 在一個 Redis instance 中使用 `SELECT` 和多個 db 是一種 anti pattern
- 建議為每個 db 需要使用專用的 Redis instance
- 在 microservice 中尤其如此，其中 application client 可能會互相干擾（db setting、維護、升級）

`RedisTimeSeries` module 提供了 compete to time series databases
- 但是，如果唯一的查詢是基於 ordering 的，那就是不必要的複雜性
- 因此，建議對每個值使用得分為 0 的 `SORTED SET`
  - 或者使用 timestamps 作為基於時間的簡單查詢的分數

