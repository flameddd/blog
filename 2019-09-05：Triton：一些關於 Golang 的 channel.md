## Triton：一些關於 Golang 的 channel

今天要舉的事例是：Golang 狂熱者的過分 channel 迷戀
- 新手邏輯：所以就凡事應該用 Channels，yeah～
  - 「Channels orchestrate」，那不就代表好棒棒沒問題囉。
  - 「Mutexes serialize」，那不就代表只能單線程運作，效能很差很不好囉。

認真地回應：Channels orchestrate; Mutexes serialize

首先：
- Channel 和 Mutex 的應用場合是完全不同的 -> 本身不具可比性的

### Channel 的應用場景：

Golang Channel / MPI library 也好
- 其中一個主要應用場景就是Thread pool Pattern
  - （別名：Manager-worker pattern）

最簡單例子：PHP FRM就是囉  
具體舉例囉：  

有一個很巨型的 input array
- 我們要把 array 內的每一個 item 都需要 gunzip 後放到對應的 output array 中

```go
// package variable
var taskChannel chan task

// 請用 goroutine 去預先打開 N 個worker thread
func workerThread() {

  for t := range taskChannel {
    i := t.index
    t.output[i] = gunzip(t.input[i])
    t.wg.Add(-1)
  }
}

// manager thread
func ArrayGunzip(input array, output array) {

  waitgroup := sync.NewWaitGroup(len(input))

  for i := 0; i < len(input); i++{
    t := task {
      index: i,
      input: input,
      output: output,
      wg: waitgroup,
    }

    taskChannel <- t
  }

  // 等待所有item工作完成
  waitgroup.Wait()
}
```

在 Golang 中，以 channel 來寫 Manager-worker pattern，最大優勢
- 語法簡單

在剛才例子
- 只需要建一個 channel variable
- 然後以讓一堆 worker thread 以 while(1) loop 不停地從 channel 接受工作
- 然後你在 manager thread 把工作丟進 channel，然後你就做完你要做的東西了。

重點是：
- workerThread 可以並行工作，並不是因為你用了 channel
- 事實上你用mutex也能寫出以上架構的（只是要３倍程式碼），而且效能更會快一點點

workerThread 可以並行工作的最大原因：
- 這個例子的工作是 embarrassing parallel 啊！！！
- 這個例子的工作是 embarrassing parallel 啊！！！
- 這個例子的工作是 embarrassing parallel 啊！！！

補充：
- 如果把 Manager-worker pattern 再進一步延伸，就是 actor model 的 design pattern 了

再延伸應用：
- Channel 有人會用作 task buffering，但這是很高階的手段了

### Mutexes serialize：

在 multi-thread 的環境
- 總有一些資料是被多個 thread 共用，critical zone 不可迴避的。

重點：
- 這時要限制只有一個 thread 能進 critical zone
- 最簡單手段當然就是用Mutex囉

例子
1. 在本機上的 localcache，一份資料當然只能同時只有一個人可以改動囉，這時間當然是用 mutex

2. 剛才 channel 的例子
- 一個 channel 可以有多個 thread 在接收工作
- Channel 的底層，也是用了 mutex 來保證一份工作只由一個 thread 收到的

重點是：
- 不是用了 mutexes 會讓你的工作一個一個來做（serialize）
- 而是當你應該保護 critical zone，讓同時只有一個thread進來時
  - 你應該用mutex而不是channel！

### 這個例子，跟good coding有什麼關係？

1. top coder應該像西廚，擁有各式各樣的刀具
- 在不同的場合使用最適用的刀具的
- 而不是像中廚，一把菜刀走天下

像剛才 channel 和 mutex
- 我跟大家示範了他們在自己最適合的場合怎讓source code變得精簡好看

2. 把第一點再延伸：約定俗成
- 在看別人的 source code 看到 Channel 時
  - 正常人會期待看到 Manager-worker pattern 之類的東西。

你用 channel 不用 mutex 用來保護 critical zone 可不可行？
- 當然行啊，只是你會坑了之後讀你的 code 的人，在大幅降低你的 code readibility 而已

如果你是做 web 相關的 backender，http status code 你有摸熟嗎？  
你能分清楚4XX系列怎使用嗎？  
