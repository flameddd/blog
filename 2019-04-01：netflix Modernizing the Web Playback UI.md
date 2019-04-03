## netflix Modernizing the Web Playback UI
### https://medium.com/netflix-techblog/modernizing-the-web-playback-ui-1ad2f184a5a0
### Dec 12, 2018

Web UI team 打算改善 playback 的使用者體驗，更 modernize。  
Playback 主要的 canvases 有：

- Pre Play: A video to be shown to members before the main content (e.g. a season recap).
- Video Playback: The member is watching their selected content, and has access to controls to pause the video, change the volume, etc.
- Post Play: When the selected content ends, members are presented with the next episode in a series, or recommendations on what to watch next.

透過 AB testing 跟隨後的學習，我們上線了一版 modern playback UI 希望帶來更好的 experience。

### 如果一開始你失敗了...
2015 年夏天的時候，整個網站都已經轉 react 了，只有 `playback UI` 還是 vanilla JavaScript framework，而這塊只有少數幾位 RD 有經驗。  
轉為 react 的話，我們能
- reduce bottlenecks when working on fixes or features
- enable more developers to contribute to building a better experience because of the familiarity and ergonomics it provided
- could get rid of that complexity

AB test design:
- AB test 測試 visual design
- data analytics partners 做 AB test

所以這次的 experiment 治療會是 playback 全新的 visual design 並且是用 React 來實作。  
我們很興奮的開始了，但沒多久我們了解到，我們太興奮了。我們沒有花足夠的時間去 thanking。  

> 2016 年夏天，我們 launch test...，我們失敗了。

### Doing Too Much at Once 一次別搞太多事情
我們困惑 test 的結果。因為其他網站跟其他平台的 UI 也已經採用了，所以我們假設了新的 visual design 沒有問題。但繼續挖下去

### Isolating Changes 區隔開改變
起初的 test design 是致命缺陷的，我們同時改動了
- visual design 跟
- underlying UI architecture

到頭來，我們根本沒法搞清楚是哪邊使我們這次 test fail。

> 寶貴的一堂課，確保測試變數是獨立的。 （all your test variables are isolated）

### Rendering Performance Gap (React 效能問題)
改成 React 時，整題重構了整個 UI architecture，AB test 發現 startup to take longer (than 之前的做法)  
- drop more frames of video

前後比較：  
- 舊有做法，直接 binding video player events 來拿到 UI state，直接對到 DOM
- 新的做法有一個 root component receive all video player state，然後傳下去給所有 children components

Re-rendering for each video player state change from this root component contributed to the performance delta.

### Try, Try Again
#### Test Design
決定下次的 AB test 專注在 UI architecture 改變。先 React 開發原本的 visual design。

#### Need for Speed  
埋 performance.mark 來測量速度。像這樣：
```js
// Inside Player.jsx...
componentDidMount() {
    performance.mark(
        `playbackMilestone:start:${this.state.playbackView}`
    );
}

componentDidUpdate(prevProps, prevState) {
    if (prevState.playbackView !== this.state.playbackView) { 
        performance.mark(
            `playbackMilestone:end:${this.state.playbackView}`
        );
        performance.mark(
            `playbackMilestone:start:${this.state.playbackView}`
        );
    }
    if (this.state.playbackView === PlaybackViews.PLAYBACK) {
        // serialize and log playback performance data.
    }
}
```

（到時候用 `window.performance.getEntriesByName('name')` or `window.performance.getEntries()` 來查結果）  
這些 milestone data 讓我們看出來 React 的 initial loading view 比較慢。  

### (react) improve render times
#### 1. 採用 React 的 React Best
> shouldComponentUpdate 該用的時候就要用。

#### 2. Less HOCs
> 儘量減少 HOC，改用 utility functions 來取代或者把邏輯往 parent component 搬

#### 3. No Prop Spreads, and Collapsing Props
- `Spreading props` 就會把時間讓費在 iterating 整個 objects
- `Collapsing multiple props` 成為一個 object 的話，能夠減少 shouldComponentUpdate 的比較時間。

#### 4. Observability
把 page 拉出 custom framework’s playbook 之外。video player 加入一個觀察用的 state 用來判斷大部分何時該 re-rendered。  
這招減少 root component （架構造成的）render cycles （次數）


再次 AB test 看得出來 React UI 跟 custom framework 相比，有同樣的 drumroll please… members （這個不知道是不是 netflix 內部的指標）  
總之就是 happy :)。2017 年夏天， React UI 全面上線（推給所有 member）。


### Under the Hood: Simplifying Playback Logic 額外一點，簡化了 Playback 的 Logic
採用 React 讓 UI component layer 更有 accessible，讓 across multiple teams 更輕鬆 dev。  
我們有 multiple teams 同時建構不同類型的 playback logic，但我們想要有一樣的 player-related business logic。

像是：  
- interactive titles (where the user makes choices to participate in the story)
- movie 
- episode playback
- video previews in the browse experience
- unique content during Post Play

採用 `Redux` 來達到 **single-source and encapsulate** 複雜的 playback 商業邏輯。  
透過 `data normalization 的 Redux`，我們能夠提供 standardized、可預測的方式呈現複雜的商業邏輯，來同時提供給多個 team 平行做開發。  

### Separating Video Lifecycle From UI Lifecycle 解耦合 Video 跟 UI 的 Lifecycle
如果用 UI component tree 來控制 video 的邏輯，會導致速度變慢。  
UI component trees 有著自己的 lifecycle 標準，例如 React 的 componentDidMount, componentDidUpdate 等等  
當 `產生新的 video playback 邏輯` 是藏在 UI lifecycle method 底下，這樣深層的 component tree，User 就必須等到某一個 component 被 call 後，才會 intial playback、載入資料。  
當 UI 從 server render 起來，initial DOM 被傳送到 client。這個 DOM 並沒有包含 讀取完的 video 或任何可以開始 playback 的 buffered data。這種情況，client UI 需要自己整個從 top 開始 rebuild DOM、然後接著跑 lifecycle，最後才開始 loading video  

如果 video playback 的邏輯在 UI component tree 之外的話，就不需要等待這個 loading sequence。  
creation of a video 跟 rendering UI 平行進行，這樣也給了 video 更多時間。當 UI 讀好了，Uesr 就能盡快播放。

### Standardizing the Data Representation of a Video Playback 標準化 video playback 的 data 表現層
video playback 由一系列 dynamic events 組層，當 application 不同部分都關心 playback 的 state 時，可能會有些問題。舉例  
- 某部分可能負責 creating a video
- 另外某部分可能負責 video 的 configuring
- 另外某部分可能負責 video 的即時 playing, pausing, and seeking.

為了封裝 video playback 的 knowledge，建立了標準 data 結構，為每一個 video playback 建構單一、集中存放的資料結構。讓 business logic 跟 UI 可以同時存取。  

### Adding Support for Multiple Video Playbacks
統一來源後，很多東西就能一次支持多個 video playbacks 了  

- 音量、靜音控制
- 播放、暫停控制
- Playback 的先 autoplay
- 限制多少 players 允許同時存在

### An Implementation of Application State 實作 Application 的 state
為了提供 well-structured 的 UI-independent state，我們決定倚靠 Redux。  
但我們也知道，我們需要允許 multiple teams 來新增、移除他們的一些不屬於 all user cases 的邏輯。  
所以我們創建了 一層極度小的 (middleware?) 在 Redux core 的上層，讓我們可以打包一些相關的 domains logic，可以組合 redux。  
我們系統的 domain 是簡單的 static object，裡面包含了這些：

- State data structure.
- State reducer.
- Actions.
- Middleware.
- Custom API to query state.

application 可以選擇要不要用它們，當一個 domain 用來 create application，domain 個別的部分會自動 bound 它自己的 bound state。  
這樣強化了兩個地方：
1. a single-level standard Redux application where each part knows about the entire state, or an application where each domain is enforced to only manage its own piece of the application’s substate. 

2. The benefit of identifying areas of logic that can be encapsulated into a logical domain is that the domain can easily be added, removed, or ported to any other application without breaking anything else.

### Enabling Plug & Play Logic
藉由 domains 的這個概念，我們就可以同時多個 team 開發各自 custom logical domains 來 plug 到 player application。像是互動標題（interactive titles）的邏輯。  

我們不斷的對許多 features 做 AB test、一致的壓縮邏輯（到 redux 中)，讓我們在 AB test or feature flags 可以輕鬆 add and remove logic。  
加入了這個結構跟可預測性，讓 develop team 可以自由發展，可以新增更多 feature、更好進行測試， code quality 更好。  

### A New Coat of Paint with A Better Engine
新的架構設計被接續的 RD 採用之後，我們的 modernization journey 來到了 visual design 階段。  

如同前面的教訓，這次得 AB test 只專注在 update video playback experience UI。不對動到任何 canvases 或 architecture。  

利用 redux 跟擴展現行的 react components，讓我們更輕鬆把 design prototypes 轉為 production code 來做 AB test。

2018 夏天，推出測試，秋天得到結果。mereber 偏好 modern visual design 跟 新的 video player control，讓他們可以 seek back and forth, or pause the video by simply clicking on the screen.  

> 這次 final AB test 能輕鬆進行跟分析是因為前面的經驗學到的。
