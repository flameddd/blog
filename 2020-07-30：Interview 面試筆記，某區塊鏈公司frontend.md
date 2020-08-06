# 2020-07-30：面試筆記，某區塊鏈公司frontend
- 以下筆記**不分順序**，只是把記得的題目寫下來，不是被問的順序

第一關的筆記，大概分三個方向
- frontend 知識
- react 知識
- leetcode coding

## `macro task` and `micro task`
```js
// 哪一個 func 會 block UI process
function bar() {
  Promise.resolve().then(bar)
}

function foo() {
  setTimeout(foo, 0)
}
```

我沒答出來，我只說了 `event loop`, `tick` 的關鍵字
考的方向是
- 知不知道什麼是 `macro task` and `micro task`
- 兩種是什麼關係
- 哪個會 block UI render

之後在 review 前面的筆記來補充！！！！！！


micro task and macro task，ECMAScript 中
- macrotask 稱為 `task`
- microtask 稱為 `jobs`


```js
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve()
  .then(function() {
    console.log('promise1');
  }).then(function() {
    console.log('promise2');
  });

console.log('script end');

// 上面的 output 順序是
// script start
// script end
// promise1
// promise2
// setTimeout
```


macrotask (也被簡稱為task)
- setTimeout
- setInterval
- setImmediate
- requestAnimationFrame
- I/O
- UI rendering

`Event loop` 的主要工作就是一輪一輪地檢查 macrotask queue，並處理這些任務

microtask (job)
- process.nextTick
- Promise callback
- Object.observe
- MutationObserver


結果是
- 「Promise 的立即返回的異步任務」會優先於「setTimeout延時為 0 的任務」執行

原因是
- setTimeout 的任務會被推入 `macrotasks` queue
- Promise 的 `then` 方法的函數會被推入 `microtasks` queue
- 在每一次 Event loop 中
  - `macrotask` 只會提取一個執行
  - 而 `microtask` 會一直提取，直到 `microtasks` 隊列清空


也就是說如果
- 某個 microtask 任務**又**推入了一個任務進入 microtasks 隊列
- 那麽在主線程完成該任務之後，**仍然會繼續運行 microtasks 任務直到任務隊列耗盡**

而 Event loop 每次只會入棧一個 macrotask
- 主線程執行完該任務**後又會先檢查 microtasks queue**
  - 並完成裏面的**所有任務**後再執行 macrotask

以瀏覽器的角度來看
- 在每一個 task 結束之前，不會有任何瀏覽器的 rending 產生
- task 執行所花的時間過長，那麼瀏覽器就無法執行其他的 Task，所以過一段時間之後會提出「頁面沒有回應」的警告，建議你關閉這個分頁


## leetcode find pattern
找出連續重複出現三次的字串

```
input: 'aaabcdddddfcd'
res: ['a', 'd'] // a, d 都有連續出現三次

input: abababacdfe
res: ['ab'] // ab 連續出現三次，而 ab 已經有了，ba 在後面就不算
```

他讓我先口頭說說解法
- 先取「一個 letter」，往下找，有符合就 push 到 res array，沒符合就往下一個 letter 跑 loop
- 「一個 letter」跑完，就換取「二個 letter」
- 口頭說明時，我有多說一步一步怎麼執行
- 講完後，面試官讓我試著 coding 看看...

```js
// 我還沒寫完，面試官就說，時間關係，先 pass ...

let test = str => {
  if (!str.length) return [];

  const res = [];
  const temp = Array.from({ length: str.length }).map((_, index) => {
    const targetStr = str.slice(0, index + 1);
    const targetStrLength = targetStr.length;
    const targetStrIndexOf = str.indexOf(targetStr);

    console.log(targetStr)
    console.log(targetStrLength)
    console.log(targetStrIndexOf)
  })
}

test('aaabcdddddfcd')
```

晚點再重新看一夏之前練習的吧，總之就是以戰養戰  
沒有看到一樣的 leetcode 要自己練了

## react hooks
```
寫一個 localStorage 的 custom hooks
能夠這樣呼叫
const [value, setValue, remove] = useLocalStorage('my-key', 'foo')
```

```js
// 我寫的內容，其中寫到一半，它有提醒我，data 可能不是 string
// 所以我才加入了 JSON.stringify
// 最後面試官說我解對了，說剩下很多都是一些 case 的判斷

import React from 'react';

const useLocalStorage = (key, initValue) => {
  // useLocalStorage
  // const [value, setValue, remove] = useLocalStorage('my-key', 'foo')
  // localStorage.setItem('myCat', 'Tom');
  // var cat = localStorage.getItem('myCat');
  // localStorage.removeItem('myCat');

  const [data, setter] = React.useState(() => {
    localStorage.setItem(key, JSON.stringify(initValue));
    return initValue;
  });
  const setValue = (value) => {
    localStorage.setItem(key, JSON.stringify(value));
    setter(value);
  }
  const remove = key => {
    localStorage.removeItem(key);
    setter(null);
  }
  return [data, setValue, remove]
}

export default useLocalStorage
```

再去 reiew 一下 知名的 hooks 套件怎麼實作吧
昨天有看一眼了，我的基本上方向是對的，只是有時候 value 會是 function，就不能用 JSON 處理
- https://github.com/streamich/react-use/blob/master/src/useLocalStorage.ts

## react 知識
- 問了我 react 更新優先順序（一開始他這樣問，我其實搞不懂，以為是要問 fiber)
- 用 Element 的 key 變化做為題目
  - 各自會驅動什麼 life cycle

key 不變、props 變，這時候會怎麼更新
- Node 不變，會 update value
- trigger shouldComponentUpdate, componentDidUpdate

key 變、props 變，這時候會怎麼更新
- key 變化了，就等於換一個新的 Node
- 舊key 會 unmount、然後 mount 新的 key 上去
  - ComponentWillUnMount, CommentDidMount, shouldComponentUpdate, componentDidUpdate ...


這題面試官說很滿意我的回答

## react 設計
- 有一個欄位是「國家」
- 第二個欄位是「金額」，金額需要依據國家來限制「上限、下限」
- 也就是說，第一個欄位切換國家時，第二個欄位要變動

這時候你怎麼設計
- 我回答，「金額 Component」會設計「上限、下限」props
- 把「金額 Component」放在至少跟「國家 Component」同一 level or 是它的 children
- 當國家變動了，就會有新的 value 傳入到「金額 Component」
- 我不會把「國家與對應金額上下限」的邏輯傳入「金額 Component」
- 「金額 Component」的「上限、下限」設計這樣才保持 Component 的中立性、不帶有主觀場景

我大概是這樣回答，面試官聽到我說中立性的時候，也立刻回應說對，這樣設計
（聽起來就是很認同我的答案～）

## 架構設計

今天有動態表單，是根據 User 設置 config，那怎麼讀取這個表單
- 我的回答是說 JSON 文件需要一直盡量描述欄位的內容 title: money, type: Number, max: 100, min: 1
- 他的回答是，要把一些邏輯判斷式寫在 Config 裡面，例如金額的計算。把這些程式碼從 config 讀出來執行

這樣動態的表單，會有很多 Component ，我們怎麼設計，可以讓沒用到的 code 別包進去
- (一開始我猶豫一下，面試官立刻補充，他想問 tree shaking 的問題)
- 我是先回答，各個欄位我會抽出來當作 Component，然後假如我們有三種 Form
- 那就會各自有三個 form 的 Component
- 沒用到的 Componet，那 webpack 自然會 tree shaking 掉
- 或者用 lazy load，需要時才 load 進來

(面試官聽到我對 webpack 理解，加上 lazy load 就認同我答案了)

後面繼續討論，如果動態表單，有很多複雜邏輯，我會怎麼設計
- 相互對話一下，我沒有回答出特別的答案
- 面試官分享他們的做法，是把邏輯存在 config 裡面
- 我 double check，所以你們是把程式碼存在 config 裡面？ Ans: YES

我知道這樣可以解，但我個人會極度避免這種 solution
- 因為你很難維護、你也沒辦法測試、版本控管
- 我就這樣跟面試官講，我個人的思維，這方法是避免的，因為上面的原因
- 他也說對，這是問題（所以他是認同我後面的說法的）


# 第二關
- 原本一、二關是一起，但當天第二位主管臨時有事，所以改天
- 主要是**針對我的履歷上寫的技術**來問，例如我寫到 A，他就問 A 相關的
- 感覺得出來，面試官也是有點實力的


## coding
要我 coding 出一個 function compose 
```
const arrFn = [fn1,fn2,fn3....]
func(...arrFn)('peter').  fn1('peter')-->return XXX.   fn2(XXX).---->. return 
```

我有寫出來，一開始有用了 if 判斷，但他有提醒一下，if 不需要，後來調整
```js
const func = (...fns) => str => fns.reduce((acc, cur) => {
   return cur(acc)  
}, str)
```

## redux connect
看到我有用 redux，要我說說看 connect 是在做什麼？  

後來覺得我講得太詳細了，連 redux 的目的的說了  

## PureComponent 是在做什麼的？
- 幫我們實作 `shouldComponentUpdate`
- 比較所有的 prevProp, prevState (淺比較 shallow compare)


## webpack
- 說說看什麼是 tree shaking
  - 延伸題如果今天有 `import 'a'` 和 `require('b')`，然後兩的都沒用到，那會被 shaking 掉嗎？
    - 我說我不知道答案，但我想猜猜看
    - 說出，因為 require 比較動態，對 bundler 來說 他比較危險，因為不知道會不會用到
    - 所以我猜 import 會 shaking 掉、require 會保留（他說我說對了）
- 怎麼縮短 build time
  - 我回答 multi thread、用 plugin 處理
  - 他說對，但另外說到 dll 處理
  - 我說我知道，我們的預設 startkit 也是這個
  - 但我知道 community 現在似乎慢慢不用了，主要開始改 `hot module replacement` 了


這邊我事後在多補充點 HMR 的知識
- cn: https://webpack.docschina.org/concepts/hot-module-replacement/
- en: https://webpack.js.org/concepts/hot-module-replacement/
- 越看越覺得 HMR 跟 dll 好像不是同一件事情，好像算不同的，但又好像能直接拿掉 dll，單純靠 HMR 就好... 

看到 next.js 的 Timer 說
- https://github.com/vercel/next.js/issues/8732
```
Timer (commented on 14 Sep 2019)
Hi! Thanks for the suggestion, but we'll be removing DLL plugin once we upgrade to webpack 5.

Because of that, we don't want to increase the API surface in the short-term. You can always inject a second version of this plugin yourself.
```

所以是靠 `Module Federation` 來 replace dll 摟！  
繼續搜尋看到，另外一個是 `react hot reload`，我可能是搞錯這個了！！  

> Dan Abramov: HMR is just a fancy way to poll the development server, inject script tags with the updated modules, and run a callback in your existing code

> Dan: React Hot Loader is a project that attempts to build on top of Webpack HMR and preserve DOM and React component state when components are saved. Normally it's hard to achieve with vanilla Webpack HMR because if you just replace a module, React will think it's a different component type and destroy DOM and local state. So React Hot Loader does many tricks attempting to prevent this, but it's also way more complex than underlying HMR API.

看完 Dan 之前寫的文章
- https://medium.com/@dan_abramov/hot-reloading-in-react-1140438583bf
- `react hot reload` 目的是 hot reload 時，保持住 state 的狀態
- `react hot reload` 已經被 `React Fast Refresh` 取代了
  - 囧，又一個，好吧，畢竟是 2017 左右的東西，這也代表 react 活躍
- `react hot reload` 概念是把 state 拉出去儲存，用類似 proxy 的方式監控 component
  - hot reload 之後，把 state 弄回去

What Is `Fast Refresh`?
> It's a reimplementation of "hot reloading" with full support from React. It's originally shipping for React Native but most of the implementation is platform-independent. The plan is to use it across the board — as a replacement for purely userland solutions (like react-hot-loader).

> Dan: Sneak peek of Fast Refresh, coming in React Native 0.61. It’s like the old "hot reloading" feature, but rebuilt from scratch for better reliability, error resilience, and function component support.

其中一的大優點是支援全平台、expo 上用，就 iOS, android 都能用！  

實在找不到，看來是我錯了，`Hot Module Replacement` 跟 dll 的方案是不同事情
- HMR 是針對你改的 code
- dll 是針對你的 dependencies

那我來找找為什麼 `react-boilerplate` 會拔掉 dll 吧  
終於找到了，這我以前也找過一次，當時不了了之
- https://github.com/react-boilerplate/react-boilerplate/commit/f3a3b2113545d32b2fc05c89c1169be50214d163#diff-d39565f942a94f37050a306fa0c42157
- 用 `webpack.optimization` and `compression-webpack-plugin` 來取代 dll
  - 用 `uglifyjs-webpack-plugin` 進一步優化 (to enable caching and multi-process parallel running at the same time (this is handled in optimization as well))
- 再加上 gzip 傳輸
- PR: https://github.com/react-boilerplate/react-boilerplate/pull/2382

整個 bundle size 減了一半以上，所以能替換掉 dll 了  
- 所以跟 `HRM` 沒關係
- 但 webpack5 就能靠 Module Federation 取代了  

## docker (談到 CI/CD 時，所以這題其實在問 ci/cd for docker)
- 問我經驗，說我們用 docker 時，通常會 `npm install`，那怎麼縮短 build time

我回答，依據 `package.lock.json` or `yarn.lock.json` 來做 cache  
這樣不用每次都重新 download

## 說說看 ci/cd 的經驗，用來做什麼、怎麼用
- 我說跑 lint, test, deploy develmern branch
- 而 production 是由 system team 處理，我們這邊沒接手
  - （後來聊到他們的組織架構，他們也是類似的模式。production 由專門的 team deploy）

後來我接著順口提到，我們 develop test 目前是用 host 的方式
- 這方法我討厭，因為有時候搞不清楚 QA 到底是不是測對了
- 這方面也是由 system team 來處理，他們要找到能用的機器部署
- 但這樣也是因為同時有多個 feature 要 QA

所以後續公司開始採用 k8s 時，我們自己在摸索
- 希望能用 k8s 來 deploy，這樣就能有獨立的 url，不需要靠 host

（k8s 這段算是我自己順著聊下去的。有時候面試就是這樣，多順著聊聊你的想法 or 經驗，讓對方多了解你。後面他介紹他們內部時，也是有說到用 k8s，就對我說，就是你剛剛說的東西，我們也有在用）

## webpack 優化經驗
- 預設有 code splite, 後面有 code splite baseon router
- 還說了 bundler analysis

後來想想，應該要多聊聊 frontend 優化經驗的  
好像不需要 focus 在 webpack 上...

## 說說 h5 的經驗 （後來補充是指說說 mobile 經驗
我回答
- 我們產品 releaser 後一年，也有來有 mobile version

他問說，那說說說 RWD
- 我回答，因為版面差異比較大， layout 相對單純，所以我們重做 Component
- RWD 用得比較少

## 問我 SSR 的東西 (因為我履歷上寫 CSR，所以他大概知道我 SSR 不熟)
- 我說我自己玩玩，但沒有 production 經驗
- 直接說到，我最關注 next.sj 的方案，因為不只單純可以 ssr, 也能 static

他問一題情境，說 init data 透過 ssr 拿，那要怎麼給到 redux init store 裡面？
- 我答不出來（我只有說，拿到後，它也會是一個 props，但有沒有適合的 life cycle ...，後面答不出來）
- 後來跟我說  就是 next.sj 自己的的 life cycle -> `getInitialProps`

剛剛查官網
- https://nextjs.org/docs/api-reference/data-fetching/getInitialProps
```
Recommended: `getStaticProps` or `getServerSideProps`

If you're using Next.js 9.3 or newer, we recommend that you use `getStaticProps` or `getServerSideProps` instead of `getInitialProps`.

...
```

## 情境題
- 交易所網站上要 即時 update data，每秒可能 100 筆、用 websocket 傳
- 這時候我們不能去每筆都去 setState，你怎麼處理

我回答
- 說會用 `throttle` 每 3 秒去 set 一次  
- 另種方法考慮用 uncontroll Component 但這有缺點，後續要靠 native JS or 比較難控制

他說我一個 point 對了，不管怎樣，就是要收集 data 再 update  
