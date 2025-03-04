# 2025-01-05: 筆記 Vercel 的 How Google handles JavaScript throughout the indexing process.md

## Giacomo Zecchini, alice alexandra moore, Ryan Siddle, Malte Ubl, Jul 31, 2024

### https://vercel.com/blog/how-google-handles-javascript-throughout-the-indexing-process

---

Vercel 發表一篇對 Google 怎麼對 JavaScript 產出的網頁做 SEO indexing  
感覺就是很實務的研究，一直存著還沒看

---

一些舊有的觀念依然存在，developer 對 SEO best practice 感到不確定:

1. Google 無法 render JavaScript 的 content
2. Google 對 JavaScript page 有不同處理方式
3. Rendering queue and timing 會顯著影響 SEO
4. JavaScript heavy 的網站會比較慢才被使用者看到(發現)

為了解決這些觀念

- 我們與 [MERJ](https://merj.com/) 合作，進行 Google crawling 的實驗
- 分析了跨網站的超過 100,000 次 Googlebot fetches，以測試並驗證 SEO 能力

---

## 1. Google rendering 能力的演變

多年來，Google crawl and index 有了顯著的變化。了解演變對於理解 SEO 現狀非常重要

- 2009 年前: 有限的 JavaScript 支援
  - 早期，Google 主要 index static HTML
  - 由 JavaScript 產生的 content 在很大程度上對搜尋引擎是看不到的
  - 這導致為了 SEO 目的廣泛使用 static HTML
- 2009 - 2015 年: AJAX crawling 方案
  - Google 引入 AJAX crawling 方案
  - 允許網站提供動態產生內容的 HTML snapshots
  - 這是臨時解決方案，需要 developer 為頁面建立單獨的、可被爬的版本
- 2015-2018 年: 早期的 JavaScript rendering
  - Google 開始用 headless Chrome 來 render page
  - 這是個重要的進步
  - 然而，這個舊版 browser 在處理 modern JS 功能方面仍然有限制
- 2018 至今: Modern rendering 能力。Google 用最新版 Chrome render，跟上最新的技術。關鍵方面包括:
  1. **Universal rendering**: Google 現在 render 所有的 HTML，而不只是其中一部分
  2. **最新版 browser**: Googlebot 用最新穩定版本的 Chrome/Chromium，支援 modern JS
  3. **Stateless rendering**: 每個頁面 render 都在全新的 browser session 中進行，不保留之前 render 的 cookie 或 state
     - 通常，Google 不會點擊頁面上的項目，[如 tabbed content](https://merj.com/blog/12-experiments-for-tabbed-content-seo) 或 cookie banners
  4. **Cloaking(隱匿技術)**: Google 禁止為了操縱排名而向 User 和搜尋引擎顯示不同的內容
     - 避免根據 User-Agent 修改內容的 code
     - 相反，改良 application 的 stateless render 以適應 Google，並通過 stateful 的方法實現個性化
  5. **Asset caching**: cache 來加速 render
     - 對於共用資源的頁面和多次 render 相同頁面非常有用
     - Google 用內部算法來確定 cache 是否仍然新鮮，何時需要重新下載，而非用 Cache-Control header

現在 Google 的 index 過程像這樣  
![alt text](assets/img/How_Google_handles_JavaScript_throughout_the_indexing_process_01.avif)

---

## 方法

為了調查 myths

- 我們用 Vercel 和 MERJ 的 Web Rendering Monitor (WRM) 進行研究
- 研究 在 nextjs.org，並補充來自 monogram.io 和 basement.io 的資料
- 研究範圍從 2024/04/01 ~ 2024/04/30

---

## 資料收集

我們在網站上放了自定義[Edge Middleware](https://vercel.com/docs/functions/edge-middleware)來攔截和分析 search engine bots 的 request

Middleware 能:

1. 識別和追蹤來自不同 search engine 和 AI crawlers 的 request
2. 給 bot 的 HTML response 中 inject 一個 [lightweight JavaScript library](https://github.com/merj/wrm-research-vercel)

這 JavaScript 在 page render 完成時觸發，將資料送回 server，包括:

- 頁面 URL。
- Unique request identifier (用於將 page render 與常規 server 訪問 log 匹配)
- Render 完成的 timestamp(這是用 librart 在 server 上的 request 接收時間計算得出)

---

## 資料分析

通過比較 server access log 中的初始 request 與 middleware 發到外部 server 的資料，我們可以:

1. 確認哪些 page 被搜尋引擎成功 render
2. 計算從初始爬取到完成 render 之間的時間差異
3. 分析不同類型內容和 URL 的處理模式

---

## 資料範圍

主要聚焦於 Googlebot 的資料，這提供了最大且最可靠的資料集

- 分析包括超過 37,000 個 rendered HTML page
- 這些 page 與 server log match

在接下來的部分中，將深入探討每個 myth

---

## 迷思 1: Google 無法 render JavaScript 的 content

這個迷思導致許多 developer 避免用 JS framework 或採用複雜的 SEO workarounds

### 測試方法

為了測試 Google render JavaScript content 的能力，我們專注於三個方面:

1. **JS framework 兼容性**:
   - 我們分析 Googlebot 與 Next.js 的互動
   - 用 nextjs.org 的資料，該網站混合用 static prerendering, server-side rendering 和 client-side rendering
2. **動態內容索引**:
   - 我們檢查 nextjs.org 上通過 API async call 載入的內容頁面，使我們能夠確定 Googlebot 是否能處理並索引 initial HTML response 中不存在的內容
3. **用 RSC 傳輸的內容**:
   - 類似上述情況，nextjs.org 很大部分是用 Next.js App Router 和 RSC
   - 我們可以看到 Googlebot 如何處理和索引 content incrementally streamed 頁面的內容。
4. **渲染成功率**:
   - 我們比較了 server log 中 Googlebot request 的數量與接收到的成功 render 的 log 的數量
   - 這讓我們了解爬取的頁面中有多少百分比是完全 render 的

### 結論

1. 在分析的超過 100,000 次 Googlebot fetches 的 `nextjs.org` page 中，排除 status code error 和不可 index 的 page，**所有的 HTML 頁面都完全 render，包括具有複雜 JS 互動的頁面**
2. 所有透過 API asyc call 載入的內容都成功 index，證明 Googlebot 能夠處理動態載入的內容
3. Next.js 被 Googlebot 完全 render，證實與 modern JavaScript framework 的兼容性
4. RSC 的內容也完全 render，證實了[streaming 不會對 SEO 產生不利影響](https://vercel.com/guides/does-streaming-affect-seo)
5. Google 嘗試 render 爬的所有 HTML 頁面，不會因為 JavaScript-heavy 就只 render 部分內容

---

## 迷思 2: Google 對 JavaScript page 有不同處理方式

一個常見誤解是，Google 對 JavaScript heavy 頁面有單獨的處理過程或標準

- 我們的研究結合了[Google 的官方聲明](https://www.youtube.com/watch?v=XBT_DUzUbOI)，揭穿這個迷思

### 測試方法

我們採取了幾個有針對性的方法:

1. **CSS `@import` 測試**

   - 我們建立了沒有 JavaScript 的測試頁面，但包含一個 CSS 文件，該文件 `@import` 另個 CSS 文件
     - (僅在 render 第一個 CSS 文件後才會下載並出現在 server log 中)
     - 通過這行為與 JS 的 page 進行比較，可以驗證 Google 的 renderer 在 enable/disable JS 的情況下是否以不同方式處理 CSS

2. **status code 和 meta tag 處理**:

   - 我們開發了一個帶有 middleware 的 Next.js application 來測試 Google 對各種 HTTP status code 的處理
   - 分析重點是 Google 如何處理不同 code 的頁面(200, 304, 3xx, 4xx, 5xx)和帶有 `noindex` meta tag 的頁面
   - 這幫我們了解在這些情況下 JS heavy 頁面是否有不同的處理方式

3. **JavaScript 複雜性分析**:
   - 我們比較 Google 在 nextjs.org 上對不同 JS 複雜性頁面的 render 行為
   - 這包括最低限度 JS 的頁面、中度互動的頁面以及具有大量 client side render 的高度動態頁面
   - 我們還計算並比較從 initial crawl 到完成 render 之間的時間，查看更複雜的 JS 是否導致更長的 render queue 或處理時間

### 結論

1. `CSS @import` 測試證實 Google 能夠成功 render 有或沒有 JS 的 page
2. 無論 JS 內容如何，Google 都渲染所有 200 status code 的 HTML page
   - 304 會用原始 200 status code page 的內容進行 render
   - 具有其他 3xx, 4xx 和 5xx error 的 page 不會 render
3. initial HTML response 中包含 `noindex` meta tag 的頁面不會被 render，無論 JS 內容如何
   - **client side 去移除 noindex tag 對於 SEO 的行為也沒有影響**
4. Google 在 render 不同 JS 複雜性頁面上的成功率沒有顯著差異
   - 在 nextjs.org 的規模上，也沒有發現 JavaScript 複雜性與渲染延遲之間的相關性
   - 然而，對於更大規模的網站，更多複雜的 JS [可能會影響 crawl 效率](https://developers.google.com/search/docs/crawling-indexing/large-site-managing-crawl-budget?hl=zh-tw)。

---

## 迷思 3: Rendering queue and timing 會顯著影響 SEO

許多 SEO 技術認為，JavaScript heavy page 因為 rendering queue 而大幅拖慢 indexing

### 測試方法: 我們進行了以下調查

1. **渲染延遲**:

   - 檢查 Google initial crawl 頁面與 completion of rendering 之間的時間差
   - 用的是來自 nextjs.org 超過 37,000 個 match 的 server-beacon 對應資料

2. **URL 類型**:

   - 分析了帶和不帶 query strings 的 URL 的 rendering times，以及 nextjs.org 不同 section 的渲染時間(如 `/docs`, `/learn`, `/showcase`)

3. **頻率模式**:
   - 觀察 Google re-renders 頁面的頻率以及不同類型內容的 rendering frequency 是否有模式

### 結論

Rendering delay 的分佈為:

- 第 50 百分位(中位數): 10 秒
- 第 75 百分位 : 26 秒
- 第 90 百分位 : 約 3 小時
- 第 95 百分位 : 約 6 小時
- 第 99 百分位 : 約 18 小時

![alt text](assets/img/rendering-data-timeline-dark-mobile.avif)

令人驚訝的是

- 25 百分位的頁面在 initial crawl 後 4 秒內就渲染完成，這顛覆了 `long queue` 的概念

雖然有些頁面顯著的延遲(99 百分位時最多可達約 18 小時)

- 但這是例外而非常態

還觀察到與 Google render 帶有 queryString (`?param=xyz`)的 URL 的速度相關的有趣模式:

|                        |              |              |              |
| :--------------------- | :----------- | :----------- | :----------- |
| URL 類型               | 第 50 百分位 | 第 75 百分位 | 第 90 百分位 |
| 所有 URL               | 10 秒        | 26 秒        | 約 3 小時    |
| 無 Query String 的 URL | 10 秒        | 22 秒        | 約 2.5 小時  |
| 有 Query String 的 URL | 13 秒        | 31 分鐘      | 約 8.5 小時  |

資料顯示，Google 對待帶有不影響內容的 query strings 的 URL 有所不同

- 例如，在 nextjs.org 上，具有 `?ref=` 參數的頁面在較高百分位上經歷了更長的渲染延遲

此外

- 我們注意到更新頻繁的部分如 `/docs` 的中位渲染時間比靜態部分要短
- 例如，雖然 `/showcase` 頁面經常被鏈接，但它顯示出較長的渲染時間
- 這表示 Google 可能會減慢對變化不大的頁面的重新渲染速度

---

## 迷思 4: JavaScript heavy 的網站會比較慢才被使用者看到(發現)

這是在網路上非常常見的迷思

- JavaScript heavy 網站，特別是那些依賴 client side render 的網站，如 SPA，在 Google 的頁面，會比較慢才能被發現

### 測試方法: 我們進行了以下測試

1. **分析不同渲染場景下的 link discovery**:
   - 我們比較了 Google 在 nextjs.org 上的 `server-rendered`, `statically generated` 和 `client-side rendered` 頁面的 link discovery/crawl 速度
2. **測試 non-rendered JavaScript payloads**:
   - 我們在 nextjs.org 的 `/showcase` page 加了一個類似於 RSC payload 的 JSON object，其中指向新的、先前未發現頁面的連結。這能測試 Google 是否能發現 non-rendered JavaScript 資料中的連結
3. **比較發現時間**:
   - 監控 Google 發現和爬取以不同方式連結的新頁面的速度
     - `標準 HTML 連結`、`client side render 出來的連結`和 non-rendered JavaScript payloads 中的連結

### 結論

1. 無論 render 方法是哪一種，Google 都能成功發現並 crawl fully rendered 的 page 中的 link

2. Google 能發現頁面上的 non-rendered JavaScript payloads 中的 link，例如 React Server Components 或類似結構中的連結

3. 在 initial HTML 和 rendered HTML 中，Google 會識別看起來像 URL 的 string 來處理內容，用當前 host 和 port 作為相對 URL 的基礎

   - (Google 沒辦法在我們的 RSC payload 中發現 encoded URL，例如`https%3A%2F%2Fwebsite.com`，這表明解析 link 非常嚴格。)

4. Link 的來源和格式(例如在 `<a>` tag 中或在 JSON payload 中)並未影響 Google 的爬取優先級
   - 無論 URL 是在 initial crawl 中還是 post-rendering 發現的，爬取優先級都保持一致
5. 雖然 Google 成功在 CSR page 中發現 link
   - 但這些 page 首先需要被渲染
   - Server side render 或 partially pre-rendered 在 immediate link discovery 上有輕微的優勢
6. Google 區分 link discovery 和 link 價值評估

   - 連結對網站架構和爬取優先級的價值評估在 full-page rendering 之後才進行

7. 有更新 `sitemap.xml` 能有效減少甚至消除不同渲染模式間的發現時間差異

---

## 整體影響和建議

研究揭穿關於 Google 處理 JavaScript heavy 網站常見迷思。以下主要結論和可行的建議:

### 影響

1. **JavaScript 相容性**:
   - Google 能有效地渲染和索引 JavaScript content
   - 包括複雜的 SPA、動態載入的內容和 streamed content
2. **渲染一致性**:

   - 處理 JavaScript heavy 與 static HTML 頁面沒有本質上的區別
   - 所有頁面都會被渲染

3. **Rendering queue 的實際狀況**:

   - 儘管存在 rendering queue，其影響比以前想像的要小
   - 大多數頁面在幾分鐘內渲染完畢，而不是幾天或幾週

4. **Page discovery**:

   - JavaScript heavy，包括 SPA，在 page discovery 方面，都沒有劣勢

5. **內容時機**:
   - 將某些 element(如 `noindex` tag )加到 page 的時機非常關重要
   - Google 可能不會處理 client-side 的 change
6. **連結價值評估(Link value assessment)**:

   - Google 區分 link discovery 和 link value assessment
   - Link value assessment 在 full-page rendering 之後進行

7. **Rendering 優先級**:

   - Google 的渲染過程不是嚴格的 first-in-first-out
   - 「內容新鮮度」和「更新頻率」比 JavaScript 複雜度更影響 priority

8. **渲染效能和爬取預算**:
   - 儘管 Google 能渲染 JS heavy page，但相較於 static HTML，這過程需求更高
   - 對於你和 Google 都是如此
   - 對於大型網站( 10,000 個以上獨特且經常變化的頁面)，這可能[影響到網站的爬取預算](https://developers.google.com/search/docs/crawling-indexing/large-site-managing-crawl-budget?hl=zh-tw)
   - 改善 application 效能並減少不必要的 JS 可以幫助加速 render speed，提高 crawl 效率，並有可能讓更多的頁面被 crawl, render and indexed

---

## 建議

1. **接受 JavaScript**:

   - 自由運用 JavaScript framework 提升 User 和 developer 體驗
   - 但需優先考慮效能並遵守 [Google's best practices for lazy-loading](https://developers.google.com/search/docs/crawling-indexing/javascript/lazy-loading?hl=zh-tw)

2. **Error handling**:

   - 在 React 用 [error boundaries](ttps://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)以防止由於單個 component 錯誤而導致整個頁面渲染失敗

3. **關鍵 SEO element**:

   - 用 server side render 或 static 來產生關鍵的 SEO tag 和重要內容
   - 確保它們出現在 initial HTML response 中

4. **資源管理**:
   0 確保 [critical resources for rendering](https://merj.com/blog/managing-webpages-resources-for-efficient-crawling-and-rendering)(API、JavaScript file, CSS file) 不被 `robots.txt` 阻擋

5. **更新內容**:

   - 對於需要快速重新索引的內容，確保變更有反映在 server side render 的 HTML 中，而不僅僅是 client side JavaScript 中
   - 考慮用像 [Incremental Static Regeneration](https://vercel.com/docs/incremental-static-regeneration#differences-between-isr-and-cache-control-headers) 這樣的策略來平衡 content 新鮮度與 SEO 和效能

6. **內部連結和 URL 結構**:

   - 建立清晰、合乎邏輯的內部連結結構
   - 將重要的 navigational links 用真正的 HTML tag (`<a href="...">`)，而不是用 JavaScript 來 navigation
     - 這方法有助於 User navigation 和搜尋引擎 crawl 效率，同時可能減少渲染延遲

7. **Sitemaps**:

   - [用並定期更新 sitemaps](https://developers.google.com/search/docs/crawling-indexing/sitemaps/overview?hl=zh-tw)
   - 對大型網站或經常更新的網站，用 `<lastmod>` 來指導 Google 的爬取和索引過程
   - 記住只在發生重大內容更新時更新 `<lastmod>`

8. **監控**:
   - 用 Google Search Console 的 [URL Inspection Tool](https://support.google.com/webmasters/answer/9012289?hl=zh-tw)
   - 或 [Rich Results Tool](https://search.google.com/test/rich-results)來驗證 Googlebot 如何查看頁面
   - 監控 crawl 統計資料，以確保所選的渲染策略不會引起意外問題

---

## 根據上面的實驗測試，來總結一下新的知識

如我們所探討的，Google 在不同 [rendering strategies](https://vercel.com/blog/how-to-choose-the-best-rendering-strategy-for-your-app) 間有差異:

|                                                                             |                              |                                       |                             |                                   |
| :-------------------------------------------------------------------------- | :--------------------------- | :------------------------------------ | :-------------------------- | :-------------------------------- |
| 功能                                                                        | Static Site Generation (SSG) | Incremental Static Regeneration (ISR) | Server-Side Rendering (SSR) | Client-Side Rendering (CSR)       |
| Crawl 效率: Google 訪問、渲染和檢索網頁的速度和效果                         | 優秀                         | 優秀                                  | 非常好                      | 不佳                              |
| 發現: 尋找新 URL 進行爬取的過程(`*`)                                        | 優秀                         | 優秀                                  | 優秀                        | 平均                              |
| 渲染的完整性(錯誤、失敗等): Google 沒有錯誤的載入和處理網頁的準確性和完整性 | 穩健                         | 穩健                                  | 穩健                        | 可能失敗(`**`)                    |
| 渲染時間: Google 完全渲染和處理網頁所需的時間                               | 優秀                         | 優秀                                  | 優秀                        | 不佳                              |
| Link 結構評估: Google 如何評估 link 以理解網站架構和頁面的重要性            | 渲染後                       | 渲染後                                | 渲染後                      | 渲染後，若渲染失敗 link 可能 miss |
| 索引: Google 儲存和組織網站內容的過程                                       | 穩健                         | 穩健                                  | 穩健                        | 渲染失敗就可能不索引              |

`*`

- 有 update `sitemap.xml` 能大幅減少甚至消除不同渲染模式之間的發現時間差異

`**`

- 根據研究，Google 的渲染通常不會失敗
- 當渲染失敗時，通常是由於 `robots.txt` 中的受阻資源或特定的 edge case 導致的

這些細微的差異確實存在

- 但無論哪種渲染策略，Google 都會快速發現並索引網站
- 將重點放在創造對 User 有益的高效 Web application 上，而不是擔心 Google 的渲染過程需要特別適應

畢竟，**頁面速度仍然是排名因素**

- 因為 Google 頁面體驗排名系統會根據 Google 的 `Core Web Vitals` 評估網站效能

- 頁面速度與良好的 User 體驗相關
  - 每節省 100ms 的載入[與網站轉換率的 8% 增長相關](https://www.deloitte.com/content/dam/Deloitte/ie/Documents/Consulting/Milliseconds_Make_Millions_report.pdf)
  - 更少的 User 跳離開你的 web，意味著 Google 認為它更具相關性
  - 效能會累積；milliseconds 很重要
