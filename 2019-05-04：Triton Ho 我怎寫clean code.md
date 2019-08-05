## Triton Ho 我怎寫clean code
### Triton Ho 2019 05 03
#### https://www.facebook.com/groups/616369245163622/permalink/1613332635467273/
（話說最近待業中，終於算是比較有空寫一下技術文了）

之前有人問我怎寫clean code，我發現我想了很久都不知怎動筆
（謎之聲：你是一面打LOL一面想當然想不出來啊）
### 先把範圍收小一下，先寫一下在golang中常犯的anti-pattern
#### 廣域得罪人的前言：

- 很多Golang新手會大量濫用Go Routine
- 然後就把降低了那麼一點點的Response Time
- 包裝成自己很會multi-threading很會concurrency
- 最後引發Application在peak hours時OOM當機……

例子：
- 現在server的數量變多，需要把access log集中到ElasticSearch管理
- 所以，所有application server在每一個request結束前把Http Response Code發到ElasticSearch

為了方便說明，我們先當我們用最原始的HTTP call機制跟ElasticSearch溝通，而不是別人寫好的asynchronous library
- 註：正常人應該用別人寫好的library
- 這樣子才會效能高同時source code好看的

所以，新手寫出來的程式大約是長這樣子的：

```go
func Somehandler() {
  // your application logic

  go SendToElastic(responseCode)
}
```

##### 新手會這麼寫的原因：
1. SendToElastic() 是需要等 ElasticSearch 收到再回答OK才完成的。所以會額外用上了1 ms。
2. 對client-side來說，他根本不用等 SendToElastic() 的結果

而且，即使有一點點的SendToElastic()是失敗了，對系統整體是沒什麼問題的。

所以，對新手來說
- 使用 go routine，建立一個pseudo-thread
- 讓 SendToElastic() 在背景的另一個 pseudo-thread 來慢慢工作
- 然後，在某一天的 peak hours，這台以 golang 來寫的 application server 就OOM爆掉了

今天不是來戰語言底層implementatio，不管你所用的語言
- 是像 golang 建 pseudo-thread
- 還是真的建立 OS-level 的 threading / process。

你
- 要做 multithreading，就需要 thread manager
- 另外每一個 thread 都需要一些記憶體空間來記錄運行狀況的。

如果是
- 在 golang 中 pseudo thread，那就單純是OOM爆掉
- 如果是用 OS-level 的 threading / process 的災情只會更慘重

這是一個老手會寫出來的程式：  
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
#### 1.
- SendToElasticWorkerThread 是一個不停等待 responseChannel 的while 1 loop
- 如果 channel 內有資料就拿出一個來丟到 ElasticSearch 

#### ２.　
現在在
- 程式啟動時就建立了固定數量的 backend thread
- 這樣子thread的數量會是相對固定，不會在 peak hours 時造成 thread manager 過勞

#### 3.
- 現在每一份要發到 ElasticSearch 的工作只佔 4 byte
- 而且是 channel 中最大的排隊工作上限是 10000
- 這保障了application不會在 peak hours 時吃掉無限的 RAM 然後當掉。

#### 4.
if len(responseChannel) < 10000  
這一句是說如果 channel 滿掉就不再放進了（註：再說一次：現實寫法不是這樣而是用switch的，這只是要方便說明）  
如果 ElasticSearch 不知有什麼原因跑太慢讓 channel 滿掉，就乾脆不發訊息  
（所謂的robustness，就是盡可能讓個別system component趴掉時，系統還能維持一定程度的服務而不是全面趴掉）  
（再補充：HA不單純是system admin的責任，讓robustness是需要developer在每一個小細節都小心的）  

一般用 select 搭 default 就可以做 non-blocking send 了  
而且不會直接對照 channel buffer 住的資料數量所造成的 race condition 問題  

#### 5.
把這種worker pool再延伸出去，就是Java Akka / Erlang他們談到的actor model（自己google啦）  
在高concurrency和高robustness要求的系統很常用的手法。

### 現實上

在 application layer 自行建一個channel global variable  
然後還自行建一堆background threading，這是高度邪惡的  

所以，再走前一步
- 如果沒有elasticSearch官方提供的asynchronous library
- 最終你也要把剛才的 channel 啊 backend threads 啊包裝成一個asynchronous library。
- 然後所有application layer 只是爽爽的 yourLibrary.AsyncSendToElastic(code)就好

#### 比較大的系統
我們會把source code分成２層
- application logic layer
- core library layer

business logic layer（像是Somehandler()）：
- 專心做business logic
- 但是禁止一切的 threading 啊 mutex locking啊
- 讓business logic以最簡潔的方式來寫出來
- 方便除錯和未來業務改變而要改動business logic

code layer（像是剛才提到的elastic Search library）：
- 禁止business logic
- 提供API讓business logic layer來使用，把重覆性的code包起來。
- 能為了效能，去做threading啊，local caching啊mutex locking一堆很困難的事情。

### 結語一句：
clean code是同時由好的架構還有coding能力組成的  
架構是戰略方向，而coding能力是具體執行細節  
二者缺一你都只會得到一堆難以維護的垃圾  

最後一句，不管架構還有coding skill，這２者都需要時間去磨練出來的
看書看文章可以讓你知道你前進方向。但是跟**游泳相同，你不親自跳下去痛苦掙扎，看再多書你都只會沉下去**