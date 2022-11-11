# Ryan Carniato: Why Efficient Hydration in JavaScript Frameworks is so Challenging
## Ryan Carniato 2022.02.03
### https://dev.to/this-is-learning/why-efficient-hydration-in-javascript-frameworks-is-so-challenging-1ca3


Hydration 是
- JavaScript framework 中用於在 browser 中初始化頁面之前被 server render 後的過程的名稱

雖然 server 可以產生初始 HTML，但需要使用 event handler 來加強 output、初始化 application 的 state，使 application 在 browser 能夠被互動  

在大多數框架中，當第一次載入時，這個過程會帶來相當大的成本
- 根據 JavaScript 載入和 hydration 完成所需的時間，可以呈現一個「看起來」是交互式的頁面
- 但實際上並非如此。這對於 UX 來說可能很糟糕，尤其是在低端設備和較慢的網絡上


多年來，大家一直在圍繞這一點不斷改進技術。這邊專門看看 `hydration` 這個話題，最好地了解正在處理的問題  

### Debunking Server Rendering as the Silver Bullet

SSR (相比 CSR) 更好支援 SEO，又有更好的效能？
- 這是一個普遍的誤解。 簡單地 SSR 並不會突然解決所有問題
- 事實上，很可能增加了 JavaScript payload，並可能需要更長的時間才使 application 能夠互動

大多 framework 的 hydration 的 code 都比他們 typical client code 大，因為最終它需要同時做這兩件事 hydrating and client side rendering

現在
- 我們不需要立即發送大部分空白的 HTML page
- 因為它可能會在載入 data 時向 User 顯示一些反饋
- 該頁面也更大，因為它包含所有的 HTML 和 application 需要引導自身的 data

這並不全是壞事
- 一般來說，主要內容應該更快 visible，因為不必等 browser 在 JavaScript 開始工作之前進行額外的往返來載入 JavaScript
- 但是也延遲了 assets 載入，包括為 application hydrate 所需的 JavaScript

注意：
- 這高度取決於網絡和 data 延遲。 並且有許多技術可以解決這種載入性能時序問題，例如 streaming
- 但這不是一個明顯的勝利，我們有新的 tradeoffs 和考量


### The Fundamental Problem

`hydration` 有兩個非常不幸的特徵
1. 一個是在 server 上渲染，現在基本上需要在 browser 中 re-render 它來 hydrate 一切
2. 第二個是我們傾向於將所有內容通過網絡發送兩次，一次是 HTML，一次是 JavaScript

通常它以 3 種形式發送：
1. The Template: component code/static templates
    - template views 存在於  bundled JavaScript 和 rendered HTML
2. The Data: 用來 fill template 的 data
   - 通常出現在 rendered page 裡面的 `script` tag 和 final HTML 裡面的一部分
3. The Realized View: final HTML


透過 client rendering，我們只是發送 template，並 request data 來 render 它。  
沒有重複。但是，在可以顯示任何內容之前，我們要受 network loading JavaScript  bundle 的支配

從 SSR 拿到所有的 HTML，它讓我們不受 JavaScript 載入時間影響來顯示網站
- 但是要如何對抗來自 server render 的固有額外開銷呢？


### Static Routes (No Hydration)
- Examples: Remix, SvelteKit, SolidStart

我們已經看到在許多 JS SSR 框架中採用的一個想法是能夠只刪除某些頁面上的 `<script>` tag
- 這些頁面是 static 的，不需要 JS，沒有 JavaScript 意味著沒有額外的網絡流量
  - no data serialization, and no hydration. Win

當然，除非你需要 JS
- 可以在頁面上潛入一些普通的 JavaScript，也許這對某些事情來說很好，但這遠非理想
- 您基本上是在創建第二個應用層

不過，這是處理這個問題的不二法門
- 但實際上，一旦添加了動態的東西，並且想利用框架，你就會把所有東西都拉進去
- 這種方法是我們一直能夠用 SSR 做的事情，幾乎所有的解決方案都在那裡
  - 但它也不是特別靈活，但它並不能真正解決大多數事情

### Lazy-loading the JavaScript (Progressive Hydration)

Progressive (漸進式) 或稱 Lazy 的 Hydration
- 這種做法，不會立刻載入 JS
- 而是在 interaction 時才載入
  - 無論是 click、 hover 還是 scroll into view
- 這樣做額外好處是，如果我們從不與頁面的一部分進行交互，我們可能永遠不會發送該 JS

但有個問題
- 大多數 JS framework 都需要由 top-down 的 hydrate，React、Svelte 都這樣
- 因此，如果 application 包含一個公共 root folder（如 Single Page application 那樣），需要載入它
  - 除非 render tree 真的很淺，否則可能會發現，當 click btn 時，都需要載入和補充大量 code
- 將 cost 推遲到 User 做某事之前並沒有真正更好。這可能更糟，因為現在它保證你會讓他們等待
  - (But your site will have a better Lighthouse Score)

因此
- 這可能會使具有 wide and shallow trees 的 application 受益
  - 但這在現代 SPA 中並不常見
  - 我們圍繞 client side routing, Context Providers, and Boundary Components (Suspense, Error, or otherwise) 的模式讓我們 building things deep

這種方法本身也
- 無法免於 serializing 所有可能需要用的 data
- 我們不知道最終會載入什麼，所以它們都需要 available


### Extracting Data from the HTML

![](https://res.cloudinary.com/practicaldev/image/fetch/s--c_VnPcix--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ebkdm43s1tgajb7zqufs.jpeg)  
(Examples: Prism Compiler)  

大家通常馬上想到另個方法是
- 也許可以從 rendered HTML 中對 state 進行逆向工程
- 可以從 HTML 中插入的值初始化 state，而不是送一個大的 JSON blob


從表面上看，這並不是一個可怕的想法。挑戰在於要查看的模型並不總是一對一的
- 如果已經導出 data，試圖回到原始 data 重新導出，在許多情況下是不可能的
- 例如，在 HTML 中顯示格式化的 timestamp，可能沒有對秒進行編碼，但如果另一個 UI 選項允許更改為可以編碼
- 這不僅適用於初始化的 state，還適用於來自 data library 和 API 的 data
  - 通常它並不像不將整個內容 serializing 到頁面中那麼簡單
  
大多數 hydration 作用會在 browser 自 top-down 的初始化時再次運行 application
- Isomorphic data fetching services will often try to refetch it again in the browser at this time if you don't send it and setup some sort of client side cache with the data.

### Islands (Partial Hydration)  

![](https://res.cloudinary.com/practicaldev/image/fetch/s--m_gYL1Xa--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fdf88s3z57vtnn52eabu.jpeg)  
(Examples: Marko, Astro)  

將網頁想像成不需要在 browser 中 re-rendered 或 hydrated 的大部分靜態 HTML
- 在其中有幾個 User 可以 interact 的地方，可以將其稱為 islands
- 這方法常被稱為 Partial Hydration
  - 因為只需要對這些 islands 進行 hydrate，並且可以跳過為頁面上的任何其他內容 sending JavaScript

對於以這種方式構建的 application
- 只需要 serialize the inputs or props 到 top level components
  - 因為我們不知道它們之上的任何內容都是有狀態的
  - 我們 100% 知道它永遠不會重新渲染。islands 是無法改變的
- 因此，我們可以通過從不發送我們不使用的 data 來解決很多雙重 data 問題
- 如果它不是 top-level input，則 browser 中不可能需要它。

 
但在哪裡設定界限呢？
- 在 component level 上做這件事是合理的，因為是人類可以理解的事情
- 但 islands 越細化，它們就越有效
  - 當 islands 下的任何東西都可以 re-render 時，你需要將該 ccode 發送到 browser

一種解決方案是開發足夠 smart 的 compiler 來確定 subcomponent 級別的狀態。因此
- 不僅從我們的 tree 中刪除了 static branches，甚至還有 nested 在有 stateful components 下的 branches
- 但是這樣的 compiler 需要專門的領域特定語言（DSL），以便可以跨模塊的方式對其進行分析。


更重要的是
- islands 意味著 server 在 navigation 時 render 每個頁面。這種多頁面 (MPA) 方法是 Web 的經典工作方式
- 但這意味著 navigation 時沒有 client side 的 transitions and loss of client state
- 實際上，Partial Hydration 是上述靜態路線的改進版本。只需為使用的功能付費

### Out of Order Hydration


如果 Partial Hydration 是 Static Routes 的更新版本
- 那麼 Out of Order Hydration (無序) 是對 Lazy Loading 的改進
- 如果不受典型的 top-down rendering frameworks 的限制，會怎樣做 hydrate
- 它可以讓頁面中間的那個按鈕在 component 層次結構中載入它上方頁面上的一堆 client routing and state managers

這有一個相當嚴格的限制
- 為此，component 必須具備最初運行所需的一切，而不依賴於其父級
- 但是 component 與 parent 有直接的關係，這通過他們的 input or props 來 present

一種解決方案是 dependency injection 以獲取相應 component 中的所有 input
- 父子之間的交流不是直接的
- 在 server render 上，所有 component 的 input 都可以 serialized（當然是刪除重複 data ）
- 但這也適用於被傳遞到 component 中的 children。它們需要提前完全 render

現有的 framework 不能以這種方式工作是有充分理由的
- Lazy evaluation 使 child 能夠控制插入 child 的方式和時間
- Almost every framework that eager evaluated children at one point now does it lazily.


這使得這種方法的發展感覺非常不同
- 因為我們習慣於需要精心策劃和限制的 parent-child interactions 規則
- 和 lazy-loading 一樣，這種方法並不能節省 data duplication
  - 因為雖然它可以相當精細地進行 hydrate，但它永遠不知道哪些 component 實際上需要發送到 browser

### Server Components

如果可以使用 Partial Hydration 但然後在 server 上 re-render static 部分怎麼辦？
- 如果這樣做，將擁有 Server Components
- 通過減少 component code 大小和刪除 duplicate data，將獲得與 Partial Hydration 相同的許多好處，但不會放棄在 navigation 時維持 maintaining client side state

挑戰在於
- 要在 server 上 re-render static 部分，需要專門的 data format 才能將現有的 HTML 與它進行比較
- 還需要為 initial render 維護正常的 server HTML rendering
  - 這意味著更複雜的構建步驟以及這些 server component 和那些 client component 之間的不同類型的 compilation and bundling


更重要的是
- 即使已經消除了 incremental overhead，也需要在 browser 中使用更大的 runtime 時才能完成這工作
- 因此，在訪問更大的 application 之前，該系統的複雜性可能無法抵消成本
- 但是當達到那個門檻時，感覺就像天空是極限
- 也許不是 maximizing initial page loads 的最佳方法，而是一種獨特的方式來保留 SPA 的好處，而無需將網站擴展到無限的 JS

### Conclusion

這是一個不斷發展的領域，因此新技術不斷湧現
- 最好的解決方案可能是不同技術的組合

如果我們採用一個可以自動產生 sub-component islands、可以無序混合併支持 server components 的 compiler 會怎樣？
- 我們會擁有世界上最好的，對吧？
- 或者，tradeoffs 可能過於極端，以至於不會與人們的心理模型相衝突。 也許解決方案的複雜性太極端了

有很多方法可以做到這一點
- 希望現在你對過去幾年為解決現代 JavaScript 最具挑戰性的問題之一所做的工作有更多的了解
