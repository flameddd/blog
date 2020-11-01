# Sapper: Towards the ideal web app framework
## Rich Harris: Sun Dec 31 2017
### https://svelte.dev/blog/sapper-towards-the-ideal-web-app-framework

---------------------------------------------
**這篇是 2017 舊文**
- Sapper 已經確定不會 release 1.0，而是整併回 Svelte (可能會歸在 SvelteKit 中)
- 但這邊的 modern web framework 的特性還是可以看看，未來思考 framework 用  
---------------------------------------------


如果必須列出完美的 Node.js Web application framework 的特徵，那麼可能會有以下這些：


1. 應該要是 SSR、快速的 initial load、不用注意 SEO
2. app's codebase 應該要可以 universal (write once for **server** and **client**)
3. Client-side 應該是 hydrate the server-rendered HTML，在已存在的 element 上面 attaching event listeners，而不是重新　re-rendering element
4. Navigating 到其他頁面時，應該要非常快速
5. 支持 Offline 和其他 PAW characteristics
6. 第一頁只 load 最初所需的 JavaScript 和 CSS。這意味著 frameword 應該在路由級別執行自動 code-splitting，並支持 dynamic import（...）以實現更精細的手動控制
7. 不影響性能
8. 一流的開發體驗、具有 hot module reloading and all the trimmings (最後這點看不懂)
9. 最後生成的 codebase 應該易於使用和維護
10. 應該有可能了解和自定義系統的各個方面、框架中沒有 locked in Webpack 的 config，並且盡可能少的隱藏“管道”
11. 在一個小時內學習整個框架應該很容易，而不僅僅是針對有經驗的開發人員

[nextjs](https://nextjs.org/docs/getting-started) 很接近這個理想
- 我強烈建議讀一下 doc
- nextjs 一個絕妙的 idea：Application 的所有 page 都是`your-project/pages` 目錄中的文件，而每個文件都只是一個 React Component

查找負責特定 page 的 code 非常容易
- 因為可以直接查看 file system，而不是猜 Component name
- 還結合了 SSR (server-side rendering) 跟 code-splitting

## But it's not perfect
（沒有很懂這段，但這篇文章是 2017 年的，不保證這些評論現在有沒有落差。看看就好）

NextJS 用的方法可以稱為 **"route masking"**
- create nice URLs (e.g. `/blog/hello-world` instead of `/post?slug=hello-world`)
- 這破壞了「「App 結構」會相對應「目錄結構」」的保證，並迫使您維護兩種形式間轉換的 config
- 所有的 routes 都被假定為一種 universal 'pages'
  - 但是，通常只需要在 server 上呈現的 routes 是很常見的，例如301 redirect 或 API endpoints 來提供 data fetch，而 Next 對此沒有很好的解決方案
  - (現在有 API route 了，這應該解決了)
- 需要用 client-side router
  - 就是說，你沒法使用標準 `<a />` tag，而必須使用 framework-specific `<Link>` components
  - 這會讓像現在這種 markdown 的 content 就無法使用


所有的好處都是有代價的
- 最簡單的 Next App (hello world page)
  - 包含 **66kb** 的壓縮 JavaScript
  - 未壓縮的是 **204kb**
  

我們 (svelte) 可以做得更好！

## Sapper
Sapper is the answer to that question.
- Sapper is a **Next.js-style framework**
- Sapper aims to meet the eleven criteria at the top of this article while **dramatically reducing the amount of code that gets sent to the browser**.
- It's implemented as **Express-compatible middleware**, meaning it's easy to understand and customise.

The same 'hello world' app that took
- 204kb with React and Next weighs
- 7kb with Sapper.

(文章的後面就沒甚麼內容了)
