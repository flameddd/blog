## Triton: Golang ElasticSearch clean code
### Triton Ho 2019  May 3
#### https://www.facebook.com/groups/616369245163622/permalink/1613332635467273/ 

###  Golang  中常犯 的anti-pattern

很多 Golang 新手會大量濫用Go Routine，然後就把降低了那麼一點點的Response Time，包裝成自己很會multi-threading很會concurrency，最後引發Application在peak hours時OOM當機……

####  例子：
- server 數量變多，需要把 access log 集中到 ElasticSearch 管理
- 所以，所有 application server 在「每一個 request 結束前」把 Http Response Code 發到 ElasticSearch

為了方便說明，我們先當我們用最原始的 HTTP call 機制跟ElasticSearch溝通，而不是別人寫好的 asynchronous library
- （註：正常人應該用別人寫好的library，這樣子才會效能高同時source code好看的）

所以，新手寫出來的程式大約是長這樣子的：

```golang
func Somehandler() {
// your application logic

  go SendToElastic(responseCode)
}
```

新手會這麼寫的原因：
1. SendToElastic( )是需要等 ElasticSearch 收到再回答 OK 才完成的。
 - 以會額外用上了1 ms。
2. 對 client-side 來說，根本不用等 SendToElastic() 的結果
 - 即使有一點點的SendToElastic()是失敗了，對系統整體是沒什麼問題的。

所以，對新手來說使用 go routine
- 建立一個 pseudo-thread
- 讓 SendToElastic() 在背景的另一個 pseudo-thread 來慢慢工作。
- 然後，在某一天的peak hours，這台以 Golang 來寫的 application server 就 OOM 爆掉了

你要做 multithreading
- 就需要thread manager
- 另外每一個thread都需要一些記憶體空間來記錄運行狀況的。

如果你是在 Golang 中
- 胡 pseudo thread，那就單純是OOM爆掉

如果你是用 OS-level 的 threading / process
- 你的災情只會更慘重。

### 這是一個老手會寫出來的程式：
（註：這是方便解說的版本，現實這麼寫，請預期在code review時被人詛咒）

```go
var responseChannel chan int
func init() {
  responseChannel = make(chan int, 10000)

  for i := 0; i < runtimme.NumOfCPU(); i++ {
    go SendToElasticWorkerThread()
  }
}

func SendToElasticWorkerThread() {
  for code := range responseChannel {
    SendToElastic(code)
  }
}

func Somehandler() {
// your application logic

  if len(responseChannel) < 10000 {
    responseChannel <- responseCode
  }
}
```

（註：len(responseChannel)實務上不能這寫的，這是為了方便說明。

解釋：

1. SendToElasticWorkerThread 是一個不停等待 responseChannel 的 while 1 loop
  - 如果channel內有資料就拿出一個來丟到ElasticSearch
2. 現在
 - 在程式啟動時就建立了固定數量的 backend thread
 - 這樣 thread 的數量會是相對固定
 - 不會在 peak hours 時造成 thread manager 過勞
3. 現在每一份要發到ElasticSearch的工作只佔 4 byte
  - 而且是channel中最大的排隊工作上限是 10000
  - 這保障了 application 不會在 peak hours 時吃掉無限的 RAM 然後當掉。
4. `if len(responseChannel) < 10000`
  - 這一句是說如果 channel 滿掉就不再放進了
  - （註：再說一次：現實寫法不是這樣而是用 switch 的，這只是要方便說明）
  - 如果 ElasticSearch 不知有什麼原因跑太慢讓 channel 滿掉，就乾脆不發訊息


所謂的 robustness，就是盡可能讓「個別 system component」趴掉時，系統還能維持一定程度的服務而不是全面趴掉
再補充：HA 不單純是 system admin 的責任，讓 robustness 是需要 developer 在每一個小細節都小心的
5. 把這種 worker pool 再延伸出去，就是 Java Akka / Erlang他們談到的 actor model（自己google啦）
 - 在高 concurrency 和高 robustness 要求的系統很常用的手法。


現實上
- 在 application layer 自行建一個 channel global variable
- 然後還自行建一堆background threading

這是高度邪惡的  
所以，再走前一步 
- 如果沒有 elasticSearch 官方提供的 asynchronous library
- 最終你也要把剛才的 channel 啊 backend threads 啊包裝成一個 asynchronous library
- 然後所有 application layer 只是爽爽的 yourLibrary.AsyncSendToElastic(code) 就好


比較大的系統，我們
- 會把 source code 分成２層
  - application logic layer
  - core library layer

business logic layer（像是Somehandler()）：
- 專心做 business logic，但是禁止一切的 threading 啊 mutex locking 啊
- 讓 business logic 以最簡潔的方式來寫出來
- 方便除錯和未來業務改變而要改動 business logic

code layer（像是剛才提到的elastic Search library）：
- 禁止business logic
- 提供 API 讓 business logic layer 來使用，把重覆性的code包起來。
- 能為了效能，去做threading啊，local caching 啊 mutex locking 一堆很困難的事情


結語一句：

clean code是同時由好的架構還有coding能力組成的。架構是戰略方向，而coding能力是具體執行細節。
二者缺一你都只會得到一堆難以維護的垃圾。

最後一句，不管架構還有coding skill，這２者都需要時間去磨練出來的。
看書看文章可以讓你知道你前進方向。但是跟游泳相同，你不親自跳下去痛苦掙扎，看再多書你都只會沉下去。
