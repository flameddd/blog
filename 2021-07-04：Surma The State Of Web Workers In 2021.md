# The State Of Web Workers In 2021
## june 30, 2021 Surma
### https://www.smashingmagazine.com/2021/06/web-workers-2021/


## 不可預測的性能問題
根據 [RAIL 原則](https://web.dev/rail/)，responsive 意思代表在 user 的行為 100ms 內做出反應，而且要維持穩定的 60 fps。我們 developer 有 16.6ms 來產生每一 frame，這是我們的 frame budget

但事實上，16.6ms 指的是 browser 有 16.6ms 的時間來處理 render 一個 frame 所需的工作。上面提到的「我們 developer」其實只處理 browser 的一些工作而已  

這些工作包括
- Detecting which element the user may or may not have tapped;
- firing the corresponding events;
- running associated JavaScript event handlers;
- calculating styles;
- doing layout;
- painting layers;
- and compositing those layers into the final image the user sees on screen;

還有很多。

與此同時，我們的 performance gap 比以前更大了 ([widening performance gap](https://infrequently.org/2021/03/the-performance-inequality-gap/))
隨著更多設備出現，有的裝置已經達到 90Hz and 120Hz，讓 frame budget 緊迫到 11.1ms and 8.3ms。更麻煩的，也沒有辦法知道你的 code 是跑在哪種頻率上。


## JavaScript
JavaScript 被設計為跟 browser 的 main rendering loop 同步運行，這個架構導致 JS 有機會拖慢 browser 繼續 rendering loop。為了 JS 要能夠執行  longer-running tasks，因此有了 [callbacks 和 later promises 的 asynchronicity model](https://vimeo.com/254947206#t=2364s)。


## Web Workers
web worker 讓能 code 跑在別的 thread，不阻塞 main thread

## JavaScript’s Concurrency Models
JS 實際上隻元兩種非常不同的 concurrency models，這些通常稱為 Off-Main-Thread Architecture。兩種架構的 tradeoffs 各自不同。

## CONCURRENCY MODEL #1: ACTORS
(Surma) 個人偏好用這種方法，把 Worker 想像成 Actors，
- Each actor may or may not run on a separate thread
- fully owns the data it is operating on
- No other thread can access it
- making rendering synchronization mechanisms like mutexes unnecessary

Actors 只能相互發送消息並對他們收到的消息做出反應  


例如
- main thread 視為有 DOM 和所有 UI 的 actor
  -負責更新 UI 和 capturing 所有的 input event
- 另外有一個 actor 負責 app’s state

1. DOM actor 將 input event 轉換成語意 event 後，轉給 state actor  
2. state actor 接收到後，更新 state、可能使用 state machine 或甚至有其他的 actor
3. state update 之後，送一份 copy 回去給 DOM actor 來更新 UI

之前 Surma 在 Chrome Dev Summit 2018 介紹過這 case
- https://www.youtube.com/watch?v=Vg60lf92EkM


### 缺點！
- 每一則訊息，都需要用 **copy** 的
  - copy 要多久就依據 message 多大，和機器等級

Surma 的經驗
- postMessage 「[通常足夠快](https://surma.dev/things/is-postmessage-slow/)」

postMessage 可以送複雜的 messages，其底下的演算法 (`structured clone`) 可以處理 circular data structures，甚至 `Map`、`Set` 都可以  

但！沒法處理 `functions` 和 `classes`  

您可以通過 postMessage 發送的消息非常複雜。底層算法（稱為“結構化克隆”）可以處理循環數據結構和偶數Map和Set.但是，它無法處理函數或類，

因為 code can’t be shared across scopes in JavaScript  


此外，postMessage 是一種 fire-and-forget 的消息傳遞機制，沒有內建 request and response 機制，需要自己實做。這就是為什麼我寫 [Comlink](https://github.com/GoogleChromeLabs/comlink)  

Comlink
- 是 RPC protocol，讓你不用處理 postMessage  
- 沒有魔術，仍然用 postMessage 來 RPC protocol

如果 postMessage 成為你 App 的瓶頸
- 了解 [ArrayBuffers](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer) 可 transferred 會有幫助
- Transferring an ArrayBuffer 幾乎是即時
  - The sending JavaScript scope loses access to the data in the process
  - I used this trick when I was experimenting with running the physics simulations of a WebVR app off the main thread.
    - https://surma.dev/things/omt-for-three-xr/index.html


## CONCURRENCY MODEL #2: SHARED MEMORY
傳統的 threading 方法是透過 shared memory，但在 JS 世界中，沒辦法這樣用，幾乎所有的 API 都是設計成 no concurrent access  

改動架構幾乎等同 break 整個 Web，或者會有嚴重的 performance 問題  

反之，shared memory 的概念被侷限在一專有類別中 [SharedArrayBuffer（或簡稱 SAB）](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer)  


SAB
- 與 ArrayBuffer 一樣，是一個可用 Typed Arrays 或 DataViews 操作的 a linear chunk of memory
- 用 postMessage 送 SAB 的話，另一端不是收到 copy，而是 exact same memory chunk
  - 所以如果某一端改動，另一端也會一樣
- 為了能夠做 mutexes 或其他 concurrent data structures。[Atomics](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Atomics) 提供 atomic operations 或 thread-safe waiting mechanisms


這方法有多種缺點
- First and foremost，它只是一塊 memory、非常 low-level primitive
  -  giving you lots of flexibility and power at the cost of increased engineering efforts and maintenance. 
  - 無法像 JS 一樣，用存取 object or array 的方法，它只是一連串的  bytes.


## Adopting Web Workers


Worker 還沒有被大量採用，因此也沒很多關於 Worker 的經驗和架構。  
我沒有提倡一種特定的架構，或否定  
但我有成功的經驗逐步增加採用 worker  


主要技巧是嚴格將 UI 代碼與純計算部分分開  
- This will reduce the number of modules that make use of the main-thread-only API like the DOM and as a result can do their work in a worker.

此外
- 盡可能少地依賴 synchronicity
- 這樣就能更輕鬆把工作拉到 worker

套用 Worker 到現有專案比較難，需要好好分析哪些工作是依賴 DOM or 其他 main thread 工作。如果可能，就逐步重構拆解，再逐步採用上面的方法 (UI 代碼與純計算部分分開 )

關鍵部分，還是要使 off-main-thread architecture 的有效果出來  
不要假設 or 猜測用 worker 就會比較好，browser 有時運作太神祕了  

## Web Workers And Bundlers

常看到有些人把 worker 轉成 `Data URLs` or `Blob URLs`  
這兩個方法都有**大問題**
- `Data URLs` Safari 不能用
- `Blob URLs` 沒有 origin or path 的概念，代表 path resolution and fetches 都沒辦法用

Webpack
- Webpack v4 用 worker-loader
- Webpack v5 內建支援，能自動理解 Worker 構造函數，甚至可以在 main thread 和 Worker 之間共享 module 以避免雙重載入。

Rollup
- rollup-plugin-off-main-threa

Parcel
- v1 和 v2 支援
對於所有這些打包器，使用 ES 模塊開發應用程序是很常見的。然而，這本身帶來了另一個問題。

## Web Workers 和 ES module

所有 modern browser 都通過
```html
<script type="module" src="file.js">
```

支持 js module  

對於 worker，除了 Firefox 外，可以這樣用
```js
new Worker("./worker.js", {type: "module"})
```

safari 是最近才支援
- https://caniuse.com/mdn-api_worker_worker_ecmascript_modules

但，browser 不支持，就靠 bundler 處理就好了。從這個角度來說， bundler 可以當做  Module Workers 的 polyfill 

