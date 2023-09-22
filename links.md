 A link list for everything I think cool, useful and easy forget >o< .
Can't remember everything.

## A Comprehensive Guide to Font Loading Strategies
- https://www.zachleat.com/web/comprehensive-webfonts/
- load font strategy. very detail. 

## puppeteer-webperf
- https://github.com/addyosmani/puppeteer-webperf
- `puppeteer` web perf testing, lot of examples.

## The most accurate way to schedule a function in a web browser
- https://medium.com/teads-engineering/the-most-accurate-way-to-schedule-a-function-in-a-web-browser-eadcd164da12
- excellent study for `setTimeout`, `requestAnimationFrame`, `worker`
- tl;tr
    - The `setTimeout` function is okay, but overall, for a 250ms theoretical timeout, the real/effective timeout value ranges from 251ms to 1.66+s.
    - As of October 2020, the most accurate way of scheduling a function/callback is using the `setTimeout` function in a `Web Worker`, in a cross-origin iframe.
    - The `requestAnimationFrame` function is the **least accurate** and requires to be inside the viewport, otherwise the real timeout can go up to several seconds, or even minutes.

## monaco-editor
- https://github.com/Microsoft/monaco-editor
- A browser based code editor which powers VS Code

## bestofjs
- https://bestofjs.org/
- javascript library collection

## pure css background 
- https://www.magicpattern.design/tools/css-backgrounds
- beautiful css background examples

## markdown-here
- https://github.com/adam-p/markdown-here
- `markdown`, `extension`, Google Chrome, Firefox, and Thunderbird extension that lets you write email in Markdown and render it before sending.

## material web-components
- https://github.com/material-components/material-components-web-components

## tweakpane
- https://github.com/cocopon/tweakpane
- `control panel`, Compact GUI for fine-tuning parameters and monitoring value changes 
- demo: https://cocopon.github.io/tweakpane/
- react: https://github.com/pmndrs/use-tweaks

## prosemirror
- New York Time's open source `collaborative WYSIWYM editor`
- https://github.com/ProseMirror/prosemirror
  - https://prosemirror.net/docs/guide/
- ref
  - Oak: https://www.youtube.com/watch?v=LEoWPdQh27c
    - backend: firestore as sub/pub, firebase to store something else (simple backend logic, simple architecture)
    - `redux`, `redux-saga`
  - We Built Collaborative Editing for Our Newsroom’s CMS. Here’s How.
    - https://open.nytimes.com/we-built-collaborative-editing-for-our-newsrooms-cms-here-s-how-415618a3ec49
    - more Oak implement detail
  - Building a Text Editor for a Digital-First Newsroom
    - https://open.nytimes.com/building-a-text-editor-for-a-digital-first-newsroom-f1cb8367fc21
  - tiptap
    - https://tiptap.dev/markdown-shortcuts
    - a renderless rich-text editor for Vue.js

## gifshot
- https://github.com/yahoo/gifshot
- `js library`, create animated GIFs from media streams, videos, or images. 

## New York Time's engineer culture & hiring process
- Diversity, Inclusion and Culture: How to Build Great Teams
  - https://open.nytimes.com/diversity-inclusion-and-culture-steps-for-building-great-teams-ca157bd98c07
- Engineers at The New York Times
  - https://open.nytimes.com/how-we-hire-front-end-engineers-at-the-new-york-times-e1294ea8e3f8
- How We Designed Our Front-End Engineer Hiring Process
  - https://open.nytimes.com/how-we-designed-our-front-end-engineer-hiring-process-9b8f20cc31fb
- We’re Making Changes to Our Front-End Engineering Application Process
  - https://open.nytimes.com/were-making-changes-to-our-front-end-engineering-process-abfc9654693d


## Pancake 
- https://github.com/Rich-Harris/pancake
- demo: https://pancake-charts.surge.sh/
- intro: https://dev.to/richharris/a-new-technique-for-making-responsive-javascript-free-charts-gmp
- `chart library`
- charting library for `Svelte`. Pancake is designed with server-side rendering in mind, meaning you can create beautiful responsive charts that may not even need JavaScript to render. Here's how.

## bytemd
- https://github.com/bytedance/bytemd
- `Markdown editor` component built with Svelte. **bytedance** publish. looks high quality. support any framework 
- demo: https://bytedance.github.io/bytemd/

## How to provide your own in-app install experience
- https://web.dev/customize-install/
- `PWA`, javascript to detect PWA status. 
- ref
  - better promotion install PWA: https://web.dev/promote-install/#browser-promotion

## worker
- https://developers.cloudflare.com/workers/
- cloudflare's serverless solution. worker is relay on **V8**, not k8s or container base.
- `serverless`, `kv worker` (key/value store), `JAMstack`, `static site`
- ref
  - https://medium.com/@zackbloom/isolates-are-the-future-of-cloud-computing-cf7ab91c6142
  - https://www.youtube.com/watch?v=HK04UxENH10&list=ULSZUq9n1HTEc&index=417

## blurhash
- https://github.com/woltapp/blurhash
- A very **compact** representation of a **placeholder for an image**.
  - typescript: https://github.com/woltapp/blurhash/tree/master/TypeScript
  - react component: https://woltapp.github.io/react-blurhash/
- other ref:
  - https://jmperezperez.com/medium-image-progressive-loading-placeholder/
  - https://engineering.fb.com/android/the-technology-behind-preview-photos/  fb engineering

## 11ty, Eleventy
-  https://www.11ty.dev/docs/
- Eleventy is a simpler static site generator.

## squoosh
- https://squoosh.app/
  - https://github.com/GoogleChromeLabs/squoosh
  - talk: https://www.youtube.com/watch?v=ipNW6lJHVEs
- `image` compress, resize, `PWA`, `offline`, `WebAssembly` 

## markdown-it
- https://www.npmjs.com/package/markdown-it
- `markdown` `npm` `nodejs`
- A nodejs lib turn content into `html`. Have lots of plugins

## Github markdown **API**
- https://developer.github.com/v3/markdown/
- `markdown` `github`
- github markdown API, turn your content into `markdown` or `GitHub Flavored Markdown (gfm)` format

## github-markdown-css
- https://github.com/sindresorhus/github-markdown-css
- `markdown` `github` `css`
- github's markdown style. If you want code syntax highlighted, use GitHub Flavored Markdown rendered from [GitHub's /markdown API](https://developer.github.com/v3/markdown/).


## micromark
- https://github.com/micromark/micromark
- `markdown`
- the **smallest** commonmark compliant **markdown parser**

## microsoft language
- ~~https://www.microsoft.com/zh-tw/language/Search?&searchTerm=iterator&langID=129&Source=true&productid=0~~
  - 好像改版了，上面連結不能用，好像換成這個 [https://msit.powerbi.com/view?r=eyJrIjoiODJmYjU4Y2YtM2M0ZC00YzYxLWE1YTktNzFjYmYxNTAxNjQ0IiwidCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsImMiOjV9](https://msit.powerbi.com/view?r=eyJrIjoiODJmYjU4Y2YtM2M0ZC00YzYxLWE1YTktNzFjYmYxNTAxNjQ0IiwidCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsImMiOjV9)
- technical term `translation` search engine
  - https://www.microsoft.com/zh-tw/language/  <- lots of international guideline

## one line layout (web.dev)
- https://web.dev/one-line-layouts/
- lots of `grid`, `flex` examples (most of them are responsive example) 

## JavaScript counters the hard way - HTTP 203
- https://www.youtube.com/watch?v=MCi6AZMkxcU&list=PLNYkxOF6rcIAKIQFsNbV0JDws_G_bnNo9&index=1

## font load STRATEGIES
- 網頁加載字型Web Font FOIT& FOUT與效能測試: https://medium.com/lucys-design-life/%E7%B6%B2%E9%A0%81%E5%8A%A0%E8%BC%89%E5%AD%97%E5%9E%8Bfoit-fout%E8%88%87%E6%95%88%E8%83%BD%E6%B8%AC%E8%A9%A6-cb0b03daad60
- Font Loading Performance 📉 6 Experiments with FOUT & FOIT: https://www.youtube.com/watch?v=vTf9HRTWKtM
- A COMPREHENSIVE GUIDE TO FONT LOADING STRATEGIES: https://www.zachleat.com/web/comprehensive-webfonts

## localhost use https (and test `__Host-` cookie)
- https://web.dev/when-to-use-local-https/
- https://web.dev/how-to-use-local-https/
  1. 修改 host file ，如 mysite.example 指回 127.0.0.1
  2. 用 mkcert  建 CA
  3. 用 CA 起 https server
  4. Chrome 要調整 #allow-insecure-localhost
  5. 用 mysite.example 來測試

## Dinero.js
- https://v2.dinerojs.com/docs
- Dinero.js is the most advanced JavaScript library for manipulating money, with a deep understanding of the model. It's a modern, reliable, lightweight, fully tested library that works both in the browser and on the server. It follows Martin Fowler's money pattern, it's immutable, modular, side-effects free, generic, and natively supports TypeScript.

## patterns.dev
- https://www.patterns.dev/posts/
- Addy Osmani 是作者之一
- 列舉和講解 JS pattern 跟一些 render 的模式和優化的方法，JS pattern 那邊講得很清楚易懂，有 code 參考  

## best practices create modern npm package
- https://snyk.io/blog/best-practices-create-modern-npm-package/
- snyk 教怎麼 release npm package
  - 涵蓋確認步驟、包含 CommonJS (CJS) and ECMAScript (ESM) module formats、typescript 怎麼設定、測試 and CI 設定等等
  - 應該很完整，有需要時再來看這篇

## Designing Front-End Components
- https://ponyfoo.com/articles/designing-front-end-components
- Nicolás Bevacqua, 幾個高品質 web UI component 的作者，分享他 build 這些工具的經驗
  - 另外還有幾篇感覺也不錯
  - https://ponyfoo.com/articles/baking-modularity-tag-editing
  - https://ponyfoo.com/articles/building-high-quality-front-end-modules

## bigfrontend.dev
- https://bigfrontend.dev/
- 一個蠻詳細的 frontend interview 線上練習，有很多 JS, css, framework 的不錯題目

## https://deck-24abcd.netlify.app/
- https://deck-24abcd.netlify.app/
- 用 css 做出各種閃卡效果，閃亮背景隨滑鼠變動、指紋效果也有感覺出來
- https://github.com/simeydotme/pokemon-cards-css

## html2svg
- https://github.com/fathyb/html2svg
  - https://fathy.fr/html2svg
- 用 chromium 把 html 轉為 svg。目前想到是，這樣可以丟進類似 figma 的工具，到時候可能可操作性會比較好，如果是純圖片的話，就沒辦法編輯之類的

## postgrest
- https://postgrest.org/en/stable/
- PostgreSQL 的工具，自動幫你建構 restful API 出來
- https://blog.frankel.ch/poor-man-api/ 別人的心得，包含怎麼 auth, rate limit monitor

## FluidFramework
- https://github.com/microsoft/FluidFramework
- 竟然沒記錄到這個，由 microsoft 推出的即時協作 framework
- 而且是 frontend base 的，協作邏輯寫在 frontend，backend 只負責轉送資訊

## Dashboard design patterns
- https://dashboarddesignpatterns.github.io/patterns.html
- 各種 Dashboard 的設計方向。有點意思，先存起來

## Rollback
- Rollback 機制。我大概不會接觸到，只是好奇而已。這邊看到好的資源，先留著
- https://gist.github.com/rcmagic/f8d76bca32b5609e85ab156db38387e9
  - Written by Zinac, the lead programmer of KI’s and GGST’s NetCode
- https://www.reddit.com/r/Fighters/comments/jfzym8/is_there_a_resource_to_learning_how_to_create/
- 另外，前陣子 youtube 關鍵字也能找到一些 coding 的影片

## Real-Time Voice Cloning
- EN: https://github.com/CorentinJ/Real-Time-Voice-Cloning
- https://github.com/babysor/MockingBird

## React dark mode
這兩篇都不錯
Josh 的 Gatsby dark mode 實作
- 最一開始那張圖表很清楚
- 有大部分原始碼，不過一些關鍵針對 Gatsby 的，套用在其他 framework 的話，就要改改
- https://www.joshwcomeau.com/react/dark-mode/

另一位網路上逛到的，`Matt Stobbs` 用 Remix 實作 dark mode
- https://www.mattstobbs.com/remix-dark-mode/

## whisper.cpp
- https://github.com/ggerganov/whisper.cpp
- automatic speech recognition, openAI, 好像非常精準

## MochiDiffusion
- Run Stable Diffusion on Mac natively
- https://github.com/godly-devotion/MochiDiffusion


## deoptigate
- https://github.com/thlorenz/deoptigate
- 看起來是很不錯的 Nodejs/V8 perf debugger, 感覺很適合做 micro perf 工具  

## best-dnd-map-makers
- https://www.dicebreaker.com/games/dungeons-and-dragons-5e/best-games/best-dnd-map-makers
- 介紹好幾個製作地圖的工具

## A11y
- `A11ycasts with Rob Dodson`
- https://www.youtube.com/playlist?list=PLNYkxOF6rcICWx0C9LVWWVqvHlYJyqw7g

## JSDoc for typescript
- https://dev.to/thepassle/using-typescript-without-compilation-3ko4

## bug bounty 回報範例
- https://www.microsoft.com/en-us/msrc/bounty-example-report-submission
- https://www.zerodayinitiative.com/blog/2017/9/5/getting-into-submitting-how-to-maximize-your-research

## total typescript
- https://www.totaltypescript.com/
- 高手的 typescript 收費課程，有好幾篇 blog 可以看看

## 開啟 mac 能夠 inspect 一些 mac 自家 app embbed 的 web 頁面
- https://blog.jim-nielsen.com/2022/inspecting-web-views-in-macos/

## 一些 JS, react 的線上練習
- https://bigfrontend.dev/

## 一個 repo 收集 JSDoc 來做 ts type check 的範例
- https://github.com/DavidWells/types-with-jsdocs#examples
