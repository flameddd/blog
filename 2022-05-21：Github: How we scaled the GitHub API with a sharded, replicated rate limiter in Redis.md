# Github: How we scaled the GitHub API with a sharded, replicated rate limiter in Redis
## Github, Robert Mosolgo April 5, 2021
### https://github.blog/2021-04-05-how-we-scaled-github-api-sharded-replicated-rate-limiter-redis/

Github 用 redis 重構了 API 的 rate limiter，後來發現一些問題並寫成這篇 blog

-------------------------

### 問題
Github 之前有個非常簡單的 rate limiter
- 每一個 request，確認一個 `key` 來為 current rate limit
- 在 `Memcached` 中、`key` 為  increment value、如果沒有 current value，就設 `1`
- 此外，如果還沒有，在 `Memcached` 中設一個 `“reset at”` 值，使用相關鍵
  - （如: `#{key}:reset_at`）
- 遞增時，如果之前 value 是 `reset at`，就忽略現有值並設置新的 `reset at`
- 在每個 request 開始時，如果 `key` 的值超過限制，並且 `reset at` 是在將來，則拒絕該 request
- （可能會有更多細部差別，但這是主要概念）

然而，這 rate limiter 有兩個問題：
- 由於它主要是 caching layer，因此 Github 將從單個共享的 Memcached 切換到每個 Data Center 一個 Memcached
  - 雖然這對 application caching 來說很好，但如果 client request 被路由到不同的 Data Center，會導致 rate limiter 表現得非常奇怪
- Memcached 的 `persistence` 也沒有用
  - Memcached 後端由 rate limiter 和其他 application caches 共享
  - 當滿的時候，它有時會驅逐 rate limiter data，即使它仍然處於 active 狀態
  - （結果，client 會在不應該的時候獲得 `fresh` rate limit windows。有時，只有一個 key 會被驅逐——他們會保持相同的 “used” value，但會獲得新的、未來的、`“reset at”` values）


### 建議的解決方案
Github 提出 rate limiter 新的設計
- 採用 `Redis`，因為它具有更合適的 `persistence`、simple `sharding` 和 `replication` setups
- application 內部的 Shard: application 將為每個 key 選擇從哪個 Redis cluster 讀取和寫入
- 為了減輕 Redis 受 CPU 限制的性質，每個 cluster 中放一個主要的副本 (primary)（用於寫入）和多個副本 (replicas)（用於讀取）
- 與其在 db 中寫 `“reset at”`，不如使用 Redis `expiration` 讓值在不再適用時消失
- 在 Lua 中實作 storage logic，以保證操作的原子性（這是對之前設計的改進）

Github 有考慮過用 Github 的 MySQL 來支持的 KV store ([GitHub::KV](https://github.com/github/github-ds#githubkv))
- 後來放棄這個方案，因為 MySQL 已經很忙了，不想向主結點增加流量
  - 避免對 MySQL 增加額外大量的 write 流量


使用 Redis 的另一個優點是它是一條行之有效的路徑。可以從兩個優秀的現有資源中獲得靈感：
- Redis 自己的 document，其中包括一些 rate limiter 模式
  - https://redis.io/commands/incr/#pattern-rate-limiter
- Stripe 的 tech blog "Scaling your API with Rate Limiters"
  - https://stripe.com/blog/rate-limiters
  - 其中包括 Ruby 和 Redis demo 實作




### release

導入時
- Github 將當前的 persistence logic 隔離到一個 MemcachedBackend class 中
- 並為 rate limiter 構建了​新的  RedisBackend class 
- 使用 feature flag 來控制對新後端的訪問。使能夠逐漸增加使用新後端的客戶百分比
  - 如果出現問題，可以快速切回舊的

release 很順利，完成後
- 我們刪除了 feature flag 和 MemcachedBackend clasee，並 RedisBackend 直接與 Throttler class 整合

但，開始收到 bug reports 了 ...

### bugs
很多 integrators 都有密切監控他們的 rate limit usage  
release 後的幾週內，Github 收到兩個很有意思的 bug report  

一些客戶觀察到他們的 `X-RateLimit-Reset` header value 搖擺不定
- 它可能會在一個 request 上顯示 `2020-01-01 10:00:00` 
- 但另一個 request `2020-01-01 10:00:01`（相差一秒）。

一些客戶的 request 因 over the limit 而被 reject
- 但 response header 顯示 `X-RateLimit-Remaining: 5000`
- 這不合理，如果他們面前有一個完整的 rate limit window，為什麼 request 會被 reject ?


### Fix 1：在 application code 中管理 “reset at”
使用 Redis 的內建 time-to-live (TTL) 來取代之前的 `“reset at”` 導致上面的搖晃問題  
1. Lua script 會 return client's TTL
2. 然後在 Ruby 中加上它 `Time.now.to_i` 以取 `X-RateLimit-Reset` header 的 timestamp

問題是，在取 TTL，Redis 和 Ruby 之間有時間流逝
- `Time.now.to_i` 取決於確切的時間
- 以及它落在 clock 的第二個邊界的位置，生成的 timestamp 可能會有所不同

例如，考慮以下 calls：

| Redis call begins | latency | TTL (Redis) | latency | Time.now returns | sum of TTL and Time.now |
| :------: | :------: | :------: | :------: | :------: | :------: |
|10:00:04.2|0.1|5|0.1|10:00:05.4|10:00:10.1|
|(then, a half-second later)||||||
|10:00:05.9|0.05|5|0.1|10:00:06.05|10:00:11.05|


在這種情況下，由於第二個邊界發生在 `call to TTL` 和 `Time.now` 之間
- 因此產生的 timestamp 比之前的 timestamp 比 `Time.now` 多一秒

我們本可以嘗試提高此 operation 的精度（例如，`Redis PTTL`）
- 但仍然會有一些擺動，即使擺動大大降低了

另一種可能性是只使用 Redis 來計算時間
- 而不混合 Ruby 和 Redis 建立它
- 本來可以用 Redis 的 `TIME` 處理
  - （舊的 Redis 版本不允許 `TIME` 在 Lua script 中用，但 Redis 5+ 可以）
- 但 Github 避免了這種設計，因為它更難測試

通過使用 Ruby 的時間作為事實的來源
- 可以在測試中進行 time-travel 使用 `Timecop`，assert 已正確處理過期密鑰，而無需實際等待Redis 對系統時鐘的 call 來取回真實的未來時間
- （仍然需要等待 Redis 來測試 EXPIRE-based database cleanup，但由於 `expires_at` 來自 Ruby-land，可以 inject 非常短的 expiration windows 來簡化測試）

相反，Github 決定
- 將 Ruby 的 `“reset at”` 時間保存在 db 中
- 這樣，可以確定它不會搖晃
  - （擺動是計算的結果，但是從 db 中讀取可以保證一個穩定的值）
- 我們沒有從 Redis 讀取 TTL，而是在數據庫中存儲了另一個值
  - （實際上使我們的存儲空間增加了一倍，但是還可以）

Github 仍然將 TTL 應用於 rate limit keys
- 但它們在 `“reset at”` 時間後設置為一秒鐘
- 這樣，可以使用 Redis 自己的語義來清理 “dead” rate limit windows


### Fix 2：在 replicas 中考慮過期
奇怪的是，許多客戶報告了包括 header 在內的 reject
- `X-RateLimit-Remaining: 5000` 這是怎麼回事！？

在 request 開始時
- 檢查 client 當前的 rate limit value
- 如果超過限制，準備 rejection response
- 在 delivering the response 之前，增加 current rate limit value，並使用 response 寫入`X-RateLimit-...` header。


上面的第 1 步 hit 了一個 Redis replicas，因為它是讀取操作
- 讀取操作返回有關 client’s previous window，並且 application 準備了 rejection response

然後，第 2 步 hit Redis primary
- 在該 db call 期間，Redis 將使先前的窗口 data 過期並返回 data 以獲得新的 rate limit
- 這是 Redis 的一個已知限制
  - replicas 不會使 data 過期，直到它們從其主節點接收到這樣做的指令，並且主節點不會使密鑰過期，直到它們被訪問（GitHub 問題）
  - （事實上，primary 會不時地隨機採樣密鑰，並酌情使它們過期）
    - (https://redis.io/commands/expire/#how-redis-expires-keys)

解決這個問題需要兩件事：
- 我們需要在 application 中管理該功能，而不是依賴 Redis 的 TTL 來過期舊的 rate limit windows
  - （application 應該準備好從 replicas 中讀取過時(stale)的 data，然後忽略它）

即使在解決了這個問題之後，也需要更好的設計：
- 在 rate limit request 的情況下，應該避免第二次調用 db
- client 的 window 可能會在兩次 call 之間過期，從而導致上述那種不一致的 response
- 此 fix 需要改進準備 response 的 Ruby，以便使用上面第 1 步的 response 來填充 `X-RateLimit-...` header


### final scripts
以下是 Github 最終用於實現此模式的 Lua scripts：

```lua
-- RATE_SCRIPT:
--   count a request for a client
--   and return the current state for the client
-- rename the inputs for clarity below
local rate_limit_key = KEYS[1]
local increment_amount = tonumber(ARGV[1])
local next_expires_at = tonumber(ARGV[2])
local current_time = tonumber(ARGV[3])
local expires_at_key = rate_limit_key .. ":exp"
local expires_at = tonumber(redis.call("get", expires_at_key))
if not expires_at or expires_at 
```

### 結論
Github 從這種新方法中學到了很多東西，但仍在考慮一個缺點
- 現在的作法在 request 完成之前不會增加 “current” rate limit value
- 這樣做是因為 Github 不向客戶收取 `304 Not Modified` response 費用
  - （這可能在客戶提供 `E-Tag` 時發生）
- 更好的方法可能會在 request 開始時增加該值，然後如果 response 是 304，則向客戶端退款
  - 這將防止在最終允許的 request 仍在處理時客戶端可能超出其限制的某些 edge case


---------------------

上面的 redis 官方文件補充不長，這邊也筆記一下   

### INCR key (command)
- Time complexity: `O(1)`
- ACL categories: `@write @string @fast`

把 key 的 value + 1，key 不存在的話，會先設為 `0` (然後再執行指令)  
- This operation is limited to 64 bit signed integers.

Examples 
```
redis:6379> SET mykey "10"
"OK"
redis:6379> INCR mykey
(integer) 11
redis:6379> GET mykey
"11"
redis:6379> 
```

### Pattern: Counter
`INCR` 最常見的用途就是 counter，每次簡單送個 INCR 指令就好了  
例如每位 User 觀看某個頁面，我們就送一次指令，這樣就能統計觀看次數了  


有幾個方法可以擴展這模式
- 可以在每次頁面查看時一起用 `INCR` 和 `EXPIRE`，使 counter 僅計算最近的 N 次頁面查看，間隔小於指定的秒數
- client 可能使用 `GETSET` 原子方式，來 get 現在這值 or reset value
- 使用其他原子 increment/decrement commands like `DECR` or `INCRBY`
  - 可根據 User 執行的操作處理變大或變小的值
  - 如，online game 中不同 user 的得分。



### Pattern: Rate limiter
rate limiter 也是一種 counter 的應用、常見的需求  
如避免 public API 被大量呼叫  

  
假設需求是將 API calls 的數量限制為每個 IP 每秒最多十個請求
- 這邊用 `INCR` 提供兩種模式

### Pattern: Rate limiter 1
最簡單的方法

```
FUNCTION LIMIT_API_CALL(ip)
ts = CURRENT_UNIX_TIME()
keyname = ip+":"+ts
MULTI
    INCR(keyname)
    EXPIRE(keyname,10)
EXEC
current = RESPONSE_OF_INCR_WITHIN_MULTI
IF current > 10 THEN
    ERROR "too many requests per second"
ELSE
    PERFORM_API_CALL()
END
```
 

「每個 IP、每秒鐘」都有一個 counter，但這 counter 總是遞增設置 10 秒的過期時間，這樣當與前秒不同時，Redis 會自動刪除它們  

注意 `MULTI` 和 `EXEC` 的使用，以確保在每次 API call 時都會增加並設置過期時間  

### Pattern: Rate limiter 2
上面那個有 `race condition` 的問題，所以這邊要提出第二種方法

counter
- 從當前秒執行的第一個請求開始，建立方式是它只能存活一秒
- 如果同一秒內有超過 10 個請求，則 counter 將達到大於 10 的值，否則它將過期並從 0 開始

```
FUNCTION LIMIT_API_CALL(ip):
current = GET(ip)
IF current != NULL AND current > 10 THEN
    ERROR "too many requests per second"
ELSE
    value = INCR(ip)
    IF value == 1 THEN
        EXPIRE(ip,1)
    END
    PERFORM_API_CALL()
END
```


**在上面的 code 存在 race condition**
- 如果由於某種原因 client 執行了 `INCR` 但沒有執行 `EXPIRE`，則 key will be leaked，直到再次看到相同的 IP 地址

這可將帶有 optional `EXPIRE` 的 `INCR` 轉為使用 `EVAL` 命令發送的 Lua script

```lua
local current
current = redis.call("incr",KEYS[1])
if current == 1 then
    redis.call("expire",KEYS[1],1)
end
```

有種不同的方法可以在不用 script 的情況下解決此問題
- 用 Redis lists 而不是 counter
- 該實作更複雜，但具有記住當前執行 API call 的 client 的 IP 的優點，這取決於對 app 是否有用

```
FUNCTION LIMIT_API_CALL(ip)
current = LLEN(ip)
IF current > 10 THEN
    ERROR "too many requests per second"
ELSE
    IF EXISTS(ip) == FALSE
        MULTI
            RPUSH(ip,ip)
            EXPIRE(ip,1)
        EXEC
    ELSE
        RPUSHX(ip,ip)
    END
    PERFORM_API_CALL()
END
```

`RPUSHX` 只有在 `key` 已存在時，才會推 push element  

這裡還是有 race，但不是大問題
- `EXISTS` 可能會 return `false`
- 但，在我們 `MULTI / EXEC` 裡面要去建立 `key` 時，`key` 可能被其他 client 建立了
- this race will just miss an API call under rare conditions, so the rate limiting will still work correctly.