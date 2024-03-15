# 2022-09-28：初讀 remix_run document
### https://remix.run/docs/en/v1
### https://remix.run/blog/remix-vs-next (Ryan Florence (Co-Founder). JANUARY 11TH, 2022)

本來對 remix_run 沒太大興趣，太多東西冒出來，沒精力碰了，直到
1. Isaacs (npm creator) 大力好評 remix_run
    - [2022-07-20：Isaacs: DONT sleep on remix_run.md](2022-07-20：Isaacs:&#32;DONT&#32;sleep&#32;on&#32;remix_run.md)
2. Ryan Dahl (Node.JS creator) 在 RemixConf 2022 講 `My Dream Stack` 時，提到他認為像 remix 這樣能部署在 edge 上的特性是未來的趨勢（影片中他應該是用 Deno 部署在 `Deno Deploy` 來 demo）
    - Ryan Dahl's "My Dream Stack": https://www.youtube.com/watch?v=3NR9Spj0DmQ

這讓我開始對 remix_run 感到興趣  
- Remix 自己這樣寫 `Remix is an edge native`
- 那我就疑惑，什麼是 `edge native` 呢？
- 一方面我一直有興趣實務上結合 `cloudflare workers` 這樣的服務，remix 也支援(什麼叫做支援呢？)，就讓我更感興趣了
  - (也支援 `Deno Deploy`)
  - (如果你不知道 `cloudflare workers` 跟 `Firebase Functions`、`AWS Lambda` 底層有什麼不一樣的話 -> https://developers.cloudflare.com/workers/learning/how-workers-works/ )

我有稍微看了一些 youtube 怎麼開發，感受到跟一般 react + backend (Node.JS, expressJS) 有蠻大的差異了  
remix 看起來已經內建很多 API 來取代些 backend 的 boilerplate，抽象化了非常多東西  

------------------------------------------------


## [Remix 的 Philosophy](https://remix.run/docs/en/v1/pages/philosophy#philosophy)

Remix 的理念可以歸納為四點
- 擁抱 server/client 端模型，包括將 source code 與 content/data 分離
- 與網絡的基礎一起工作，而不是對抗。瀏覽器、HTTP 和 HTML。它一直都很好，而且在過去的幾年裡變得非常好
- 通過模擬瀏覽器行為，使用 JavaScript 來增強 UX
- 不要對底層技術進行過度抽象

### Server/Client Model
你可以使你的服務器快速，但你不能控制 User 的網絡 
- 在今天的網絡基礎設施中，你不需要靜態文件來使你的服務器變快
- 這個網站 ([remix doc](https://remix.run/docs/en/v1)) 的第一個字節的時間是很難被打敗的
- 而且是完全 fresh 的
- 我們利用 edge 的分佈式系統，而不是靜態構建
- 我們可以做到修復一個錯字，網站在幾秒鐘內就能反映出來：無需重建，無需重新部署，甚至無需 HTTP caching

你無法使 User 的網絡變得快速
- 唯一能做的就是減少在網絡上發送的東西。減少 JavaScript、JSON、少CSS
- 當有一個可以移動邏輯的服務器，以及一個支持漸進式增強的框架時，這是最容易的

舉例: fetching a list of records
- 看看的個 [Github Gist API](https://api.github.com/gists) data，這 data 的 payload 為 `75kb`，通過網絡壓縮後為 `12kb`
- 如果瀏覽器中 fetct，你會讓 User 下載**所有**的內容。它可能看起來像這樣

```js
export default function Gists() {
  const gists = useSomeFetchWrapper(
    "https://api.github.com/gists"
  );
  if (!gists) {
    return <Skeleton />;
  }
  return (
    <ul>
      {gists.map((gist) => (
        <li key={gist.id}>
          <a href={gist.html_url}>
            {gist.description}, {gist.owner.login}
          </a>
          <ul>
            {Object.keys(gist.files).map((key) => (
              <li key={key}>{key}</li>
            ))}
          </ul>
        </li>
      ))}
    </ul>
  );
}
```

用 Remix 的話，可以先在 server 端過濾才送給 User  
- (其實這就跟一般的 frontend 自己多架一層 backedn 一樣，能更有彈性的調整)

```js
import { json } from "@remix-run/node"; // or cloudflare/deno

export async function loader() {
  const res = await fetch("https://api.github.com/gists");
  const gists = await res.json();
  return json(
    gists.map((gist) => {
      return {
        description: gist.description,
        url: gist.html_url,
        files: Object.keys(gist.files),
        owner: gist.owner.login,
      };
    })
  );
}

export default function Gists() {
  const gists = useLoaderData();
  return (
    <ul>
      {gists.map((gist) => (
        <li key={gist.id}>
          <a href={gist.url}>
            {gist.description}, {gist.owner}
          </a>
          <ul>
            {gist.files.map((key) => (
              <li key={key}>{key}</li>
            ))}
          </ul>
        </li>
      ))}
    </ul>
  );
}
```


這樣，payload 從 `12kB` 降為 `3.8kB`
- 減小 20 倍
- 也不需要 skeleton UI 了
- 這只是一個 server/client model 幫助改善 UX 的小例子

### Web Standards, HTTP, and HTML
Remix 完全接受了這些技術
- 結合 HTTP Caching、Remix 對 asset URL 的關注、動態服務器渲染以及 `<link rel=prefetch>` 等 HTML 功能
- 你擁有了所有的工具來使 App 變快。瀏覽器和 HTML 在這段時間，越變越好

我們試圖將 Remix API 保持在最低水平，而與 Web Standards 合作
- 例如，沒有發明自己的 `req/res` API，甚至沒有用 Node 的 API，而是讓 Remix 與 Web Fetch API 一起工作
- 這意味著當精通 Remix 時，你實際上只是精通了 `Request`、`Response`、`URLSearchParams` 和 `URL` 等 Web Standards
- 所有這些都已經在你的瀏覽器中，現在無論你部署到哪裡，它們都在你的服務器上

在做 data mutations 的時候
- 我們增強了 HTML form
- 當為下一個頁面預取 data 和 asset 時，我們使用 `<link rel="prefetch">`，讓瀏覽器處理所有復雜的緩存資源
- 如果瀏覽器有一個用於某種情況的API，Remix就使用它。

### Progressive Enhancement
Remix 同時具有 read and write API
- 自 90 年代以來，HTML `<form>` 一直是 data mutations 的主力軍
- Remix 接受並加強了該 API，這使得 Remix 的 data layer 能夠在頁面上有**沒有 JavaScript 的情況下發揮作用**

加入 JavaScript 後，Remix可以通過兩種方式加快頁面轉換時的 UX
1. 不下載和評估 JavaScript 和 CSS assets
2. 只針對 layout 變化的部分而 fetch data

另外，頁面上有 JavaScript 時，Remix 可以為開發者提供 API，使頁面轉換時的 UX 更好
1. 加上比瀏覽器預設的 spinning favicon 更好的客製 pending UI
2. 在操作 data（create, read, update, delete, etc）上加上更好的 UI

最後，由於 data mutation 是內建於 Remix 中的
- 它知道何時重新 fetch 可能被改變的 data，確保頁面的不同部分不會失去同步

重點不是讓 App 在沒有 JavaScript 的情況下工作
- 而是要保持更簡單的 client/server model
  - 能夠將 JavaScript 留在門外是個很好的 side-effect


### Don’t Over Abstract
Remix 的 API 都是用 fundamental Browser/HTTP/JavaScript
- 這些技術並沒有隱藏在背後
- Remix 的抽象化足以優化你的 App 性能，而不隱藏底層技術

> (我: 這點，可能要多看細部 doc，才能有更多了解)


> (我: 上面這段 `Remix Philosophy` 是官網為了簡介 remix 的內容，要繼續看下去才能更理解細部)

------------------------------------------

## 技術解釋: 什麼是Remix？
Remix有四個方面:
- 一個compiler
- 一個服務器端的 HTTP handler
- 一個 server framework
- 一個 browser framework

我們經常把 Remix 描述為 `React Router 的 compiler`
- 因為 Remix 的一切都利用了嵌套路由的優勢
- App 開發人員將文件加到 `app/routes/*`，然後 Remix 從那裡開始

### Compiler
Remix 的一切都從 `Compiler: remix build` 開始，用 `esbuild` 來 create 一些東西

1. `A server HTTP handler`:
    - 通常在 `server/build/index.js` 中（可配置）
    - 包括所有 routes and modules，以便能夠在 server 上 render 並 handle 任何 server side requests
2. `A browser build`:
    - 通常在 `public/build/*`
    - 這包括按照 route 自動做 code splitting
    - fingerprinted asset imports (like CSS and images)
    - 在 browser 中運行一個 App 所需的任何東西
3. `An asset manifest`
    - client 和 server 都使用這個清單來了解整個依賴關係圖
    - 這對於在最初的 server side render 中 preloading resource 以及為 client side prefetching 資源非常有用
    - 這就是 Remix 如何能夠消除當今 Web App 中常見的 render+fetch 瀑布的原因

有了這些構建工具，App 可以被部署到任何運行 JavaScript 的託管服務上



### HTTP Handler 和 Adapters
雖然 Remix 在 server 上運行，但它實際上並不是一個 server
- 它只是一個 handler，被賦予一個實際的 JavaScript server
- **它是建立在 Web Fetch API 而不是 Node.js 上的**
- 這使得 Remix 可以在任何 Node.js server 中運行
  - 如 `Vercel`、`Netlify`、`Architect` 等，以及 `Cloudflare Workers` 和 `Deno Deploy` 等非 Node.js 環境上

這是 `Remix` 在 `Express` 中運行時的樣子

```js
const express = require("express");
const remix = require("@remix-run/express");

const app = express();

app.all(
  "*",
  remix.createRequestHandler({ build: require("./build") })
);
```

`Express`（或 `Node.js`）是實際的 server
- Remix 只是該 server 上的一個 handler
- `@remix-run/express` 被稱為 adapter
- Remix handlers 是不受服務器影響的
- Adapters 通過將 server 的 request/response API 轉換為 Fetch API，然後將來自 Remix的 Fetch Response 轉換為server's response AP，使其適用於特定的 server

下面是 Adapters 的一些 pseudo code:
```js
export function createRequestHandler({ build }) {
  // 從 server build 中建立一個 Fetch API request handler
  let handleRequest = createRemixRequestHandler(build);

  // returns an express.js specific handler for the express server
  return async (req, res) => {
    // adapts the express.req to a Fetch API request
    let request = createRemixRequest(req);

    // calls the app handler and receives a Fetch API response
    let response = await handleRequest(request);

    // adapts the Fetch API response to the express.res
    sendRemixResponse(res, response);
  };
}
```

真正的 adapters 做的事情比這更多一些
- 但這是它的 gist
- 這不僅能讓你在任何地方部署 Remix，還能讓你在現有的 JavaScript server 中逐步採用它
- 因為可以在 Remix 之外設置 routes，在進入 Remix 之前，你的 server 會繼續處理這些 routes

此外，如果 Remix 沒有為你的 server 提供 adapters，你可以查看其中一個 adapter 的 source code 並建立自己的 adapter
- https://remix.run/docs/en/v1/other-api/adapter


### Server Framework
如果你熟悉 `Rails` 和 `Laravel` 等 server side MVC web frameworks
- Remix 就是 `View` 和 `Controller`，但把 `Model` 留給你
  - - Remix 的 Route modules 沒有在 `View` 和`Controller` 間進行分割，而是同時承擔了這兩個職責

大多 server side frameworks 都是 "關注模型"
- 一個控制器為一個單一的模型管理多個 URL

Remix 是以 UI 為中心的
- Route 可以處理整個 URL 或只是 URL 的一部分
- 當一個 route maps 到一個片段時，嵌套的 URL 片段成為 UI 中的嵌套佈局
  - 這樣，每個佈局（view）都可以成為自己的 controller，然後 Remix 會匯總 data 和 component 構建完整的 UI

Route modules 有**三個**主要 exports
- `loader`, `action`, and `default (component)`


```js
// Loaders 只會在 server 上執行
// 用在當 compoent GET request 時，來 provide data
export async function loader() {
  return json(await db.projects.findAll());
}

// Action 只會在 server 上執行
// 用來 handle "POST", "PUT", "PATCH", "DELETE" request
// 也能為 component provide data
export async function action({ request }) {
  const form = await request.formData();
  const errors = validate(form);
  if (errors) {
    return json({ errors });
  }
  await createProject({ title: form.get("title") });
  return json({ ok: true });
}

// default export 是 (UI) component
// 當 URL match 時，就會 render
// 這會同時在 server and client side 執行
export default function Projects() {
  const projects = useLoaderData();
  const actionData = useActionData();

  return (
    <div>UI</div>
  );
}
```


實際上，你可以把 Remix 僅僅作為一個 server side framework 來使用，不需要用任何 browser 的JavaScript
- route conventions 用 loader 來 loading data
- 用 Action 和 HTML form 做 mutations
- 以及在 URL render 的 component
- 可以提供很多 web app 的 core feature 集

這樣一來，Remix 的規模就縮小了，在 App 中，並不是每個頁面都需要在 browser 中使用大量的 JavaScript
- 在 Remix 中，你可以先以簡單的方式構建，然後在不改變基本模型的情況下進行擴展


如果不熟悉傳統的 backend web frameworks
- 可以把 Remix routes 看作是 React component
- 它們已經是自己的 API routes，已經知道如何在 server 上 load and submit data 給自己


### Browser Framework
一旦 Remix 向 browser 提供了 document
- 它就會用 browser 構建的 JavaScript modules 將頁面 `hydrates`，這就是 Remix `emulating the browser`
- 當 User 點擊一個連結時，Remix 不會為了整個 document 和所有 assets 而往返於 server
- 而是簡單地 fetch 下一個頁面的 data 並更新 UI。這比「請求完整的 document」效能更好
    1. Assets 不需要重新下載（或 pulled from cache）
    2. Assets 不需要被 browser 再次解析
    3. 獲取的 data 比整個文檔要小得多（有時是數量級的）

Remix 還對 client side navigation 進行了些 built in optimizations
- 它知道哪些 layout 會在兩個 URL 之間持續存在，所以它只 fetch 那些正在變化的 layout 的 data
- 一個完整的 document request 需要在 server 上獲取所有的 data，浪費了後端的資源，並降低了 App 的速度

這種方法也有 UX 方面的好處
- 比如不重置 sidebar 的 scroll position

Remix 還可以在 User 即將點擊一個 link 時 prefetch 頁面的所有資源
- `browser framework` 知道 `compiler` 的 `asset manifest`
- 它可以匹配鏈接的 URL，讀取清單，然後 prefetch 下個頁面的所有 data、JavaScript modules，甚至 CSS resources
- 這就是 Remix 在網絡緩慢的情況下也能感覺到快速的原因


上面的 route module 為例，這裡有幾個小的，但有用的 UX 改善的表格，你只能在 browser 中用 JavaScript 做
1. 當 form is being submitted 時，disable 該按鈕
2. 當 server side form validation fails 時，focus the input
3. Animate in the error messages

```js
export default function Projects() {
  const projects = useLoaderData();
  const actionData = useActionData();
  const { state } = useTransition();
  const busy = state === "submitting";
  const inputRef = React.useRef();

  React.useEffect(() => {
    if (actionData.errors) {
      inputRef.current.focus();
    }
  }, [actionData]);

  return (
    <div>
      {projects.map((project) => (
        <Link key={project.slug} to={project.slug}>
          {project.title}
        </Link>
      ))}

      <Form method="post">
        <input ref={inputRef} name="title" />
        <button type="submit" disabled={busy}>
          {busy ? "Creating..." : "Create New Project"}
        </button>
      </Form>

      {actionData?.errors ? (
        <FadeIn>
          <ErrorMessages errors={actionData.errors} />
        </FadeIn>
      ) : null}

      <Outlet />
    </div>
  );
}
```

這 sample 最有趣的是，它只是加法的。整個互動從根本上說仍然是相同的事情
- (上面這句話要看原始網站，才知道它 mark 哪幾行程式碼 https://remix.run/docs/en/v1/pages/technical-explanation#browser-framework ，這些 mark 就是它要表達的，這邊只加了點東西，沒有改動其他的)
- 因為 Remix 可以接觸到 backend 的 controller，所以它可以無縫地做到這一點
- 雖然沒有像 `Rails` 和 `Laravel` 這樣的  server side frameworks 那樣深入到 stack 中，但它確實深入到了瀏覽器中的 stack，使後端到前端的過渡變得無縫

For example
- 在後端重度 web framework 中構建一個普通的 HTML form 和 server side handler，就像在 Remix 中一樣容易做到
- 但是，一旦想跨越到一個有動畫驗證 data、焦點管理和 pending UI 的體驗，就需要對 code 進行根本性的改變
- 通常情況下，人們會建立一個 API 路由，然後引入大量的 client side JavaScript來連接這兩者
- Remix 只需在現有的 `server side view` 加一些代碼，而無需從根本上改變它的工作方式

Remix 藉用了一個古老的術語，稱其為 Remix 的漸進式增強
- 從一個普通的 HTML From 開始（Remix會縮小規模），當你有時間和雄心時，再擴大 UI 的規模

---------------------------------------------

## Performance
- (這部分目前還是草稿，未來應該會有更多內容)
- (https://remix.run/docs/en/v1/guides/performance)

Remix 
- 不是像 SSG(Static Site Generators) 那種規定一個精確的架構及其所有的限制
- Remix 旨在鼓勵利用 distributed computing 的性能特點
- 發給 User 最快的東西當然是離 User 近的 CDN 上的 static file 
  - 在這之前 server 幾乎只在世界的「某一個」地方運行，這使得其他地方的 response 速度很慢
  - 這也許是 SSG 如此受歡迎的原因之一，它允許 developer 將 data "cache" 到 HTML 中，然後分發到世界各地
    - 這當然也有很多 trade off:
      - build time, build complexity, 翻譯的重複網站, 不能用它來做認證的 UX、無法用它來做非常大的 dynamic data sources（ex `unpkg.com`）... etc

### The Edge
現今，在 edge 的 distributed computing 方面發生了很多令人興奮的事情
- 在 edge 做 computing 通常意味著在靠近 User 的 server 上執行程式，而不是只在同一個地方（如美國東海岸）
- 現在，我們看到越來越多這樣的應用，也看到 distributed databases 也在向 edge 移動
- 我們已經預見到這一切有一段時間了，這就是為什麼 Remix 是這樣設計的

隨著 distributed servers 和 databases 在 edge 運行
- 現在是能夠以跟 static file 差不多的速度來提供 dynamic content
- 你能夠讓你的 server 更快，但你對 User 的網路無能為力
  - 唯一要做的是將 code 從 browser bundles 轉到 server 上
  - 通過網絡發送更少的 bytes，並提供無與倫比的網絡性能。這就是 Remix 的設計目的


### This Website + `Fly.io`
- 現在這個 [Remix 的 document 網站](https://remix.run/docs/en/v1))的第一個 byte 的時間是很難被打敗的
- 對於世界上大多數人來說，它低於 `100ms`
- 我們可以修復 doc 中的一個錯別字，在一兩分鐘內，網站就可以在全球範圍內更新，無需重建，無需重新部署，也無需 HTTP caching

我們通過 distributed systems 實現了這一點
- 這 doc application 在世界各地的 `Fly` 上的幾個地區運行，所以它離「你」很近
- 每個 instance 都有自己的 `SQLite`
- 當 app 啟動時，它從 GitHub 上的  Remix repo fetch tarballs，將 markdown docs 處理成 HTML，然後 insert 到 SQLite 中

實際上與 `Gatsby` 在 `gatsby-node.js` 或 `Next.js` 的 `getStaticProps` 中可能做的事情非常相似
- 這個想法是把慢的部分（從 GitHub fetch file，處理 markdown）和 cache 它（SSG caches 到 HTML，這個網站 caches 到 server 上的 `SQLite`）

當 User 請求一個 page 時
- App 會查詢其本地 SQLite 庫並發送該頁面
- Server 在幾毫秒內就能完成這些請求
- 這架構最有趣的地方在於，不必為了新鮮度而犧牲速度
- 在 GitHub 上編輯 doc 時，GitHub Action 會用最近的 app instance 的 webhook，然後將該請求復製到世界各地的所有其他 instance 上
- 然後它們都從 GitHub 上拉出新的 tarball，並將其 DB 與 doc 同步，就像它們啟動時那樣
  - 世界各地的文檔在一兩分鐘內就會更新

但這只是我們想要探索的一種方法


### Cloudflare Workers
- [Remix Cloudflare Workers Demo](https://remix-cloudflare-demo.jacob-ebey.workers.dev/)

Cloudflare 一直在推動 edge computing 的發展
- Remix 的定位是充分利用優勢
- 你可以看到 demo 的 response time 與 serving static files 的 response time 相同
- 但它所展示的功能絕對不是靜態的

Cloudflare 不僅在 User 身邊運行 app
- 他們還有像 `KV` 和 `Durable Objects` 這種的 persistent storage systems，以實現 SSG 級別的速度
- 而沒有將 data 與 deploy 和 bespoke incremental builder backend 相耦合的束縛

還有其他類似的平台，我們已經有計劃很快支持它們

### 其他 Technologies
這裡有些其他技術可以幫助你的 server 加速
- `FaunaDB`: 一個 distributed database，在 User 附近運行
- `LRU Cache`: memory cache，當它變滿時自動清除更多空間
- `Redis`: server-side cache



(我: 到這邊為止是官網文件幾個比較描述與概念的部分，其他部分都是實作與 API 文件，這些我想放到更後面。相反，我想先看看官方 blog 寫的 `Remix vs Next.js` 的文章，所以下面先插入這篇文章的筆記)
------------------------------------------

## Remix vs Next.js: TL;DR (JANUARY 11TH, 2022)
- Serving static content 方面，`Remix` fast or faster than `Next.js`
- Serving dynamic content 方面，`Remix` faster than `Next.js`
- 網路慢的時候，`Remix` 能提供快速的 UX
- `Remix` 自動能 handles errors, interruptions, and race conditions，`Next.js` 沒這功能
- `Next.js` 鼓勵用 client side JavaScript 來 serving dynamic content，`Remix` 不鼓勵
- `Next.js` 需要 client side JavaScript 才能做到 data mutations，`Remix` 不需要
- `Next.js` 的 build times 是隨著你的資料量線性成長，`Remix` build times 是 decoupled from data
- 當你的 data scales 變大時，`Next.js` 就會需要調整架構和犧牲 performance
- Remix 團隊認為 `Remix` 的 abstractions 能達成更好的 application code


### Remix vs Next.js: 實驗背景
Remix 為了比較 Next.js，拿 Vercel 的一個 Next.js ECommerce Demo 專案來比較
- https://demo.vercel.store/

關於這個 ECommerce Demo
- Initial page load 對 ECommerce 有著決定性的重要性
- search page 有著 Dynamic data
- shopping cart 這邊有 Data mutations 
- 與多個供應商整合的能力，說明了框架如何幫助你抽象化

而 Remix 團隊用 Remix 重寫了兩個版本
1. `Minimal Port`:
    - 單純 copy/pasted/tweaked Next.js 的 code 到 Remix 裡面跑
    - 部署在 Vercel 上，跟 Next.js Demo 一樣
    - 這是一個很好的比較，除了 framework 之外，其他所有的東一樣
2. `Rewrite`:
    - 兩個 framework 其實沒有太多 API 重疊，為了鍛鍊 Remix 的設計，團隊把這個 case 用 Remix 的方式重新寫了一次
    - 甚至在 App 中建立 image optimization route，所以這是 100% Remix way
      - (強調這點，可能是因為 Next.js 有專門對 image 做優化，所以 Remix 也嘗試做優化)

### Remix vs Next.js: 自我描述
很多事物，你可以從建造它的人對它的描述中了解到很多東西。 (如果你在twitter上關注我，你會知道我一直在費力地迭代我們的！）。

`Next.js` 將自己描述為:
- 用於 production 的 React Framework
- Next.js 提供了最好的 DX 和你在 production 中需要的所有功能
  - hybrid static & server rendering, 支持 TypeScript, smart bundling, route pre-fetching, etc
  - No config needed

`Next.js` 是由 `Vercel` 構建的，Vercel 的 GitHub repo 指出:
 - Vercel 是一個用於靜態網站和前端框架的平台，建立在你的 headless content, commerce or db 的整合上

`Remix` 的描述為:
- Remix is an **edge native**, full stack JavaScript framework
- For building modern, fast, and resilient user experiences
- 它採用 web standards 統一了 client and server side ，所以可以少考慮代碼，多考慮產品

後面會對比這些描述

### Remix vs Next.js: 首頁， Visually Complete

> `Remix` 和 `Next.js` 一樣快嗎？
- 這是大家最長問的第一個問題
  - `Next.js` 經常用 `performance by default` 這句話，

下面來看看每個 App 頁面 `Visually Complete` 的速度
- WebPageTest 測試
  - WebPageTest 可以產生下面這些對比 gif
  - 每次比較，我們給每個framework 做五次，並從每個 framework 中取了最好的一次
- 每個比較的上方都有一個標題，link 到生成動畫的結果，可以點擊 `WebPageTest.com` 上的 `rerun test`  來驗證



| [Home Page, Virginia, Cable](https://www.webpagetest.org/video/compare.php?tests=220113_BiDc2H_cfaa25c420a8552959c39df6a7d24e08,220113_BiDc1H_6121ab1c9a5c969874e64344aecce3ec,220113_BiDc9N_b9e84f012470b1aeb51754fb010133d9)(這個測試是在 Virginia 用 cable modem 連到 internet 進行的) |
| :----: |
| ![](https://remix.run/blog-images/posts/remix-vs-next/wpt-virginia-homepage-cable.gif) |


這三個版本都非常快
- **甚至不值得比較誰更快**
- 這對 `Next.js` 也有點不公平，因為 Cookie banner 是 Visually Complete 一環，而 Remix 網站沒有這個

用慢動作看一下
![](https://remix.run/blog-images/posts/remix-vs-next/wpt-virginia-homepage-cable-slow.gif)


結果是: **都很快，看起來都差不多**  

### Remix vs Next.js: 為這麼這些 App 這麼快？
`Next.js` 方面:
- homepage 是透過 `getStaticProps` 做到 `Static Site Generation (SSG)`
- Build time 時，`Next.js` 從 `Shopify` 拉資料，將頁面 render 成 HTML file，然後放在 public directory 中
- 部署時，這些 static file 是分佈在 edge 上 (Vercel's CDN)，而不是從原本的 server，這單一 location 的地方
- Request 來的時候，CDN 單純地提供這些檔案。Data loading 和 rendering 已經提前完成
  - (所以 User 不需付出 download 和 render 的 cost)
- CDN 分佈全世界，靠近 User (這就是 edge 的意思)，所以對這些 static file 的 request 就不需要一路訪問到後面這一個單一的 server 源頭

`Remix port` 方面:
- **`Remix` 不支援 SSG**
- Remix 採用 HTTP 的 [stale-while-revalidate](https://web.dev/i18n/zh/stale-while-revalidate/) ([MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#stale-while-revalidate)) caching directive
  - (別跟 Vercel 的 swr (fetching) library 搞混)

結果是一樣的
- static document 在 edge 上（甚至可能跟 Vercel's 同一個 CDN）
- 差別在於，document 怎麼到達那邊

不同於 Next.js 的方案
- cache 會在你獲得流量時啟動
- 下一位 User 訪問時，document 就會從 cache 拿，並且驗證
- 這就像 SSG 一樣，當有流量時，User 不需要支付 download and rendering 的 cost  

`SWR` 是 `SSG` 很好的另過方案
- 另一件讓部署到 Vercel 的事情是，他們的 CDN 支持它

為什麼 `Remix port` 沒有跟 `Next.js` 一樣快
- 因為(**現在**) Remix 沒有 built-in image optimization 
- 我們只是把 images 指向 Next.js App
- Browser 必須 connect 兩個 domain，這使 image load delay 了0.3s（可在 network waterfall 驗證）
  - 如果 iamge 是自己 host 的，那就會和其他兩個一樣，在 0.7 秒左右


`Remix rewrite` 方面:
- 這版**不是透過 SSG or SWR 把 doucment cache 在 edge 上面！**
- 這版是把 data cache 在 `Redis` 上
- 這版也是跑在 edge 上，是在 [Fly.io](https://fly.io/docs/reference/regions/) 上
- 這版有 quick image optimization Resource Route，這是寫在 persistent volume，基本上是它自己的 CDN
- 這在幾年前可能很難建立，但在過去的幾年裡，server 的情況已經發生了很大的變化，而且只會越來越好


### Remix vs Next.js: Loading Dynamic Pages 方面
Remix 會被問的第二個問題會是:
> `Remix` 跟 `Next.js` 有什麼差別？

在功能設置上有很多不同
- 一個主要的架構上的不同，`Remix` 不依賴 `SSG` 來獲得 perf
- 實際上，在每個 App 中，都會遇到 SSG 無法支持的情況

我們在這裡比較的 feature，它是 search page
- 這邊的約束條件是 User 可以無止境的一直查詢，這我們就不能靜態地產生無限的頁面，SSG 是不存在的

| [Search Page, Cached, Virginia, Cable](https://www.webpagetest.org/video/compare.php?tests=220114_AiDcG6_3792ae086fc7d2decbddddc1cc521705,220114_BiDcWS_3b730524b3294cc4aabd961753821fe2,220114_AiDcD6_7aa4143672ff6295a8b88392f3e5ef42) |
| :----: |
| ![](https://remix.run/blog-images/posts/remix-vs-next/wpt-virginia-search-cable.gif) |


因為 SSG 不能 scale  dynamic pages
- `Next.js` 轉而從 User 的 browser 發起 client fetch data
- 看看網絡瀑布就知道為什麼它比 Remix 慢

| `Remix` | `Next.js` |
| :---: | :---: |
| ![](https://remix.run/blog-images/posts/remix-vs-next/wpt-search-remix-waterfall.png) | ![](https://remix.run/blog-images/posts/remix-vs-next/wpt-search-next-waterfall.png) |


在 Next.js App 開始 loda image 之前
- Remix App 已經完全完成
- 因為這裡不能使用 SSG，App 從 client side fetch data。在獲取 data 前，它不能 load image 圖片
  - 在 loaded, parsed, and evaluated JavaScript 之前，它也不能 fetch data
- 也許在網絡性能方面，最重要的事情是將網絡瀑布並行化。Remix 團隊 對此非常重視


從 client 做 fetch 代表:
- 透過網路傳遞更多 JavaScript
- 花更多時間進行 parse/eval
- `Next.js` 發送更多 JavaScript， `Next.js` vs `Remix`: `566 kB` vs. `371 kB` unpacked
  - Over the network it's 50 kB more compressed (172 kB vs. 120 kB).

為什麼 `Remix` 這邊還是跟 homepage 的頁面一樣快？:
- Remix 的兩個例子實際上都沒有在 request 中與 Shopify 的 API 對話
- 雖然 SSG 不能 cache search page，但 Remix 版本可以
  - 用 SWR 或 Redis
- 當你有個單一的、動態的方法來生成 page 時，你可以調整 cache 策略而不用改變 App 的 code
  - 其結果是在經常訪問的頁面上能有 SSG 等級的 perf
  - `/search` page 可能會被產生，還有左側 nav 上的分類和 `tshirt` 等常見查詢



### Remix vs Next.js: 當 Dynamic Page 的 Cache Miss 的時候呢？

| [Search Page, Empty Cache, Virginia, Cable](https://www.webpagetest.org/video/compare.php?tests=220114_AiDcMN_e671a25cc0dc7b5bb0986e397e00f044,220114_AiDc8W_e897cd8815342449977013c2d57a4daf,220114_AiDcQX_d4ce4649434538f369982fa828abae82) |
| :----: |
| ![](https://remix.run/blog-images/posts/remix-vs-next/wpt-virginia-search-miss-cable.gif) |

好吧，上面其實有點撒謊
- 上面的 `Remix rewrite` 版本是較慢 hit cache 的版本
- 怕讀者不相信，因為其實 cache miss 的版本才花了 0.6s
  - https://www.webpagetest.org/video/compare.php?tests=220114_BiDc1T_9ea8b52894cbd02675159ae963750ace-r:1-c:0

這不可能、不科學
- 是因為，其實 `Shopify` 的 API 相當快

由於 `Next.js` App 直接從 browser fetch Shopify API的
- 可以看一下測試的網絡圖，`request` 只花了 224ms
  - browser 與 API 建立 connection 的時間比發出 request 的時間還長！
    - (這可以用 `<link rel="preconnect" />` 來加速)

如果 User 的 browser 可以這麼快向 Shopify 發出 request
- Remix 的 server 當然也可以做得更快
- User connect 到 cloud 的速度總是比你的 server 去 connect 還要慢，最好保持在那裡 fetch data

所以
- 在用 Shopify API 時，cache 幾乎毫無意義
- cache 的 hit/miss rate 幾乎是沒有區別的

再來一個 cache miss 的 test，搭配 3G 網路速度

| [Search Page, Empty Cache, Hong Kong, 3G](https://www.webpagetest.org/video/compare.php?tests=220114_BiDcDX_a0db98ec1d32d3be3e263fab2628df47,220114_AiDcJF_f139b368029039e7de112963e2fb43b8) |
| :----: |
| ![](https://remix.run/blog-images/posts/remix-vs-next/wpt-hkg-search-3G.gif) |

`Next.js`:
- 在 load data 之前無法 load image
- 在 loda JavaScript 之前無法 load data
- 在 load document 之前無法 load JavaScript
- User 的網路是上面這條 chain 中每一步，都是乘數（更久）


`Remix` 中:
- 唯一的 chain 是等待 document 能夠 load image
- Remix 的設計 **always** 在 server 上 fetch
- 這避開了因為 User 的網路在其他任何地方都是一個乘數的問題
- Remix 可以在收到 request 時立刻開始從 Shopify fetch
  - 不需要等 browser 下載 document，然後再下載 JavaScript
  - 不管 User 的網路有多慢，Server 上的 Shopify API 的 ferch 不會改變，可能在 200ms 內

### Remix vs Next.js: 架構的分歧

當 `Next.js` 改從 client side 執行 data fetch 時
- UX 不是唯一受到打擊的地方
- 該 App 現在有兩套不同的抽象來與 Shopify 對話: 一套用於 SSG，另一套用於 browser

像這樣的架構分歧帶來了一些主要問題:
- 你必須在瀏覽器中進行 authenticate 嗎？
- API 是否支持 CORS？
- API SDK 是否能在 browser 中工作？
- 如何在 build 和 browser code 之間共享 code？
- 在 browser 中暴露 API token 可以嗎？
- 我們剛剛傳給每個 User 的 token 有什麼權限？
- 這個 function 能用 `process.env` 嗎？
- 這個 function 可以讀取 `window.location.origin` 嗎？
- 怎樣才能使一個 network request 在兩個地方都有效？
- 可以在某個地方 cache 這些 response 嗎？
- 我們是否應該做一個在兩個地方都能使用的 isomorphic cache object，並把它傳遞給不同的 data fetch function？

對於上面這些問題，`Remix` 只需要在 server 上抽像出 Shopify 的 API:
- 你必須在瀏覽器中進行 authenticate 嗎？ **(NO)**
- API 是否支持 CORS？**(無所謂)**
- API SDK 是否能在 browser 中工作？**(不需要)**
- 如何在 build 和 browser code 之間共享 code？**(不需要)**
- 在 browser 中暴露 API token 可以嗎？**(不需要)**
- 我們剛剛傳給每個 User 的 token 有什麼權限？**(並沒有傳給 User)**
- 這個 function 能用 `process.env` 嗎？**(YES)**
- 這個 function 可以讀取 `window.location.origin` 嗎？**(NO)**
- 怎樣才能使一個 network request 在兩個地方都有效？**(不管你怎麼想，都不在瀏覽器中)**
- 可以在某個地方 cache 這些 response 嗎？**(當然可以，HTTP、redis、lru-cache、persistent volume、sqlite...)**
- 我們是否應該做一個在兩個地方都能使用的 isomorphic cache object，並把它傳遞給不同的 data fetch function？**(不需要)**

這些問題回答得越簡單，你的抽象就越好，從而使工作的 code 更簡單  


如果 `Next.js` App 改用 `getServerSideProps` (server fetching way) 的話
- 差距可能會縮小
- 對這些問題有更簡單的答案
- 有趣的是，`Next.js` 的 document 經常把你從 server fetching 推到 SSG 或 client fetching
  - https://nextjs.org/docs/basic-features/data-fetching/get-server-side-props#when-should-i-use-getserversideprops



這裡的根本區別在於，`Next.js` 有四種模式來 fetch page 上的 data:
- `getInitialProps`:    called server and client side
- `getServerSideProps`: called server side
- `getStaticProps`:     called at build time
- `client fetching`:    called in the browser

`Remix` 只有一種模式，就是 `loader`
- 在一個地方運行的東西比在三個地方運行的四個東西更容易抽象化

### Remix vs Next.js: 架構分歧的 cost
我們試著量化這種架構分歧的成本
- 也許這個 App 最困難的開發任務是抽像出 commerce back end
- 這個 App 的設計方式是，你可以將任何東西插入其中: `Shopify`, `BigCommerce`, `Spree`, `Saleor` 等

`Next.js` App 中，Shopify 的集成就在[這個文件夾](https://github.com/vercel/commerce/tree/f3cdbe682b6153b6881d8a6597b44429424de269/framework/shopify)中  
- 在它上面運行 `cloc`，結果為
  - 101 text files.
  - 93 unique files.
  - 8 files ignored.

```
github.com/AlDanial/cloc v 1.92
---------------------------------------------------------------------
Language           files          blank        comment           code
---------------------------------------------------------------------
TypeScript            88            616           2256           5328
GraphQL                1           1610           5834           2258
Markdown               1             40              0             95
JSON                   2              0              0             39
JavaScript             1              1              0              7
---------------------------------------------------------------------
SUM:                  93           2267           8090           7727
---------------------------------------------------------------------
```

近 100 個 files 中的近 8000 行 code
- 對其他的集成進行了測試，情況也是如此
  - 都接近 100 個 files，並徘徊在 10,000 行 code 左右。幾乎所有的代碼都能在 broweer 上


[下面是 Remix 與 Shopify 的集成](https://github.com/jacob-ebey/remix-ecommerce/blob/0abc6a5ad8117961d49dce22764ad06a4508733c/app/models/ecommerce-providers/shopify.server.ts):
- 1 file
- 608 lines of code
- 沒有任何 code 進入 browser

這就是架構分歧的代價
- `Next.js` 的抽象必須預見並參與到構建和 browser 中
- `Remix` 的抽像只在 server 上

你可能想知道這兩個 Shopify providers 是否有相同的 feature sets
- 也許 Remix 是在欺騙
- 在很多的 code 中都有一些 authentication 和 wishlists 的 code，但 Shopify providers 沒有使用這兩種 code
  - （但確實要為它們 export modules）
- 使用這兩個網站，它們似乎有相同的  feature sets
- 不管怎麼說，如果 Remix 確實錯過了什麼，很難想像需要 7000 行 code 才能達到，而 App 中可見的功能只需要 1/10 的 code

即使 Next.js 將 search 轉移到 `getServerSideProps`
- 仍然需要幾乎所有的 code 來實現 data mutation 的功能

### Remix vs Next.js: 關於 Edge Native

Remix 經常提到 `deploying to the edge` 到底是什麼意思？
- 來看另一個 case: 香港、fast user network、cache miss 的情境

| [Search Page, Empty Cache, Hong Kong, Cable](https://www.webpagetest.org/video/compare.php?tests=220114_AiDc2R_5dfaa245cd27404654074fba9cd73248,220114_AiDcM4_9b913562cae343803f0c4efef48b51b8,220114_AiDcYP_534551c67b82cd664aae1c0813c384de) |
| :----: |
| ![](https://remix.run/blog-images/posts/remix-vs-next/wpt-hkg-search-miss-cable.gif) |


我們已經知道 `Next.js` 版本因為 network waterfall chains 的原因，造成速度較慢，這邊來談兩個 Remix App 的區別
- 兩個 Remix App 都是 server fetch，那為什麼 `Remix port` 慢 `Remix Rewrite` 這麼多？

`Remix Port` 跑在 `Vercel function`
- `Vercel functions` 不是在 edge 上執行，它們是在一個地區 run 的
  - default 是 `Washington DC`，離香港很遠
- 這代表 server 從 Shopify fetch data 之前，User 必須從香港到 `Washington DC`，當 server 完成後，還需要把 docuemnt 一路傳回來

而 `Remix Rewrite` 同時在 `Washington DC` 和`香港`都有運行  

看看 network waterfall

|`Remix Rewrite` @ Edge|`Remix Port` in US East|
| :---: | :---: |
| ![](https://remix.run/blog-images/posts/remix-vs-next/wpt-hkg-search-rewrite-waterfall.png) | ![](https://remix.run/blog-images/posts/remix-vs-next/wpt-hkg-search-port-waterfall.png) |
| <a href="https://remix.run/blog-images/posts/remix-vs-next/wpt-hkg-search-rewrite-waterfall.png" target="_blank">(圖片另開頁面)</a> | <a href="https://remix.run/blog-images/posts/remix-vs-next/wpt-hkg-search-rewrite-waterfall.png" target="_blank">(圖片另開頁面)</a> |


infrastructure 的差異體現在第一個藍條上
- `Remix Port` 要大得多，User 繞了半個地球在 Vercel function 行駛
- `Remix Rewrite` 更快到達 Shopify API 並 return


`Remix Rewrite` 在 [Fly.io](https://fly.io/docs/reference/regions/) 上
- `Fly.io` 可以在全球幾十個地區跑 `Node.js` server
- 不過，`Remix` 並不依賴於 `Node.js`
  - 它可以在任何 JavaScript 環境中跑
  - 事實上，它已經在 `Cloudflare Workers` 上，這意味著你在他們分佈在世界各地的 250 台 server 上運行 code。沒有比這更接近用戶的了
    - (我對 `Fly.io` 不熟，Remix 團隊這樣寫的話，我想是指 `Fly.io` 部分是基於 `Cloudflare Workers` 的)

這就是為什麼說 `Remix` 是 `edge native`
- `Next.js` 依賴於 `Node.js`，所以它的 `deploy the edge` 能力是有限的
- `Remix` 還在努力，現在只正式支持 `Node.js` 和 `Cloudflare`
  - 但團隊正在積極研究 `Deno`
  - community members 已經在 `Fastly` 上運行 `Remix`

當你用像 Remix 這樣的 `edge native` framework 時
- 你不再需要決定要優先哪些 User 獲得更快的 UX
- 你可以給每位 User 一個快速的 UX，無論在世界的什麼地方
- `edge` 就是 `Remix` 誕生的理由
  - 據我們了解，Vercel 團隊也在努力將你 App 署到 edge


### Remix vs Next.js: 關於 Client side 的 Transitions

Clientside Transitions
Both frameworks enable instant transitions with link prefetching, but Next.js only does this for pages created from SSG. The search page is out, again. (maybe next time, sport)

However, Remix can prefetch any page because there was no architectural divergence for data loading. Prefetching an unknowable, user-driven search page URL is not any different than prefetching a knowable product URL.

In fact, Remix prefetching isn't limited to just links, it can prefetch any page, at any time, for any reason! Check this out, prefetching the search page as the user types:


兩個 frameworks 都能通過 link prefetching 實現 instant transitions
- 但 `Next.js` 只有 SSG page 能，所以 search page 就不行
- 而 `Remix` 可以 prefetch 任何 page，因為 data fetch 的行為上，沒有架構分歧
  - prefetch 一個不可知的、user-driven search page URL 與 prefetch 一個可知的 product URL 沒有任何區別
  - Remix的 prefetch 並不局限於 link，它可在任何時間、以任何理由預取任何頁面


看看 User 輸入時 prefetch search page:  
| Search Input Prefetching, Fast 3G |
| :---: |
| <video width="100%" muted autoplay loading="lazy" src="https://github.com/flameddd/blog/assets/22259196/68727219-d9cf-4357-b95b-a4d16f3a1c07" ></video> |
(video source: https://remix.run/blog-images/posts/remix-vs-next/prefetch-search.mp4)


沒有 spinners, skeletons，就算網路慢，也有快速的 UX  
- 這非常容易實作

```js
import { Form, PrefetchPageLinks } from "@remix-run/react";

function Search() {
  let [query, setQuery] = useState("");
  return (
    <Form>
      <input type="text" name="q" onChange={(e) => setQuery(e.target.value)} />
      {query && <PrefetchPageLinks page={`/search?q=${query}`} />}
    </Form>
  );
}
```

由於 `Remix` 使用 HTML 的`<link rel="prefetch">`（而不像 `Next.js` 用 memory cache）
- 實際上是 browser 發出請求，而不是 Remix
- 仔細看上面影片，你可以看到當 User 中斷當前的 fetch 時，request 是如何被取消的
- Remix 在處理異步時不需要任何 code 就能實現

### Remix vs Next.js: 關於 Data Mutations
This is where Remix and Next.js start to look completely different. Half of your app code is related to data mutations. It's time your web framework respects that.

How mutations work in Next.js: Next.js doesn't do anything for you here. `<button onClick={itsAllUpToYou}>`. Typically you'll manage the form's state to know what to post, add an API route to post to, track loading and errors states yourself, revalidate data and propagate changes throughout the UI, and finally deal with errors, interruptions, and race conditions (but let's be honest, nobody actually deals with that stuff).

Data Mutations 就是 `Remix` 和 `Next.js` 開始看起來完全不同的地方  

`Next.js` 中的 mutation 是如何工作的:
- `Next.js` 沒有為你做任何事情
- `<button onClick={itsAllUpToYou}>`，你會管理表單的狀態以知道發布什麼
- 加一個 API route來 post，自己跟踪 loading 和 error 狀態
- 重新驗證 data 並在 UI 中傳播變化，最後處理錯誤、中斷和 race conditions（很少人會好好處理 race）


`Remix` 中的 mutation 是如何工作的:
- `Remix`使用 HTML form
- 雖然看起來像舊時代的 PHP，並不意味著它不能擴展到現代複雜的 UX

網路誕生以來，一個 mutation 的 modele 為
- 一個 form 和一個 server page 來處理它。(不考慮 `Remix`，它看起來像這樣)

```html
<form method="post" action="/add-to-cart">
  <input type="hidden" name="productId" value="123" />
  <button>Add to Cart</button>
</form>
```

```js
// on the server at `/add-to-cart`
export async function action(request) {
  let formData = await request.formData();
  return addToCart(formData);
}
```

browser 將 form 的 serialized data post 到 `"/add-to-cart"`
- 加入 pending UI
- DB 更新後，render new page with fresh data

`Remix` 的做法與 HTML form 相同
- 只是用大寫字母 `<Form>` 和一個名為 `action` 的 route function 進行了優化
- 它用 post 來 fetch，而不是 reload document
- 然後用 server 重新驗證頁面上的所有 data，以保持 UI 與後端同步
- 這與你在 SPA 中習慣做的事情相同，只是 `Remix` 管理這一切
- 除了 form 和 server side 的 `action` 外，不需要 App code 來與 server 進行溝通
- 也沒有 context, provider 或 global state management 來傳播變化到 UI 的其他部分。這就是為什麼 Remix bundles 比較小，你不需要所有的 code 來與 API route 對話
- 上面那段 code，在 Remix 其實也是 work 的，如果使用小寫的 `<form>`，browser 會處理 post，而不是 Remix
  - 這在 JavaScript fails to load 的情況下很方便（後面會有介紹）

### Remix vs Next.js: 關於 Unhandled Errors

當 User 把商品加到購物車時，後端拋出錯誤時會發生什麼？
- 在這我們 block requests 看看會發生什麼

| Next.js Failed POST |
| :---: |
| <video width="100%" muted loop autoplay loading="lazy" src="https://github.com/flameddd/blog/assets/22259196/3cfc46c9-bb23-482f-b0b2-a14e959d7f42" ></video>|
(video source: https://remix.run/blog-images/posts/remix-vs-next/next-error.mp4)  

什麼也沒發生
-  Error handling 是困難的，也是令人討厭的
- 許多開發者直接跳過它
- 我們認為這是一個糟糕的默認 UX


來看看 Remix:

| Remix Failed POST |
| :---: |
| <video width="100%" muted loop autoplay loading="lazy" src="https://github.com/flameddd/blog/assets/22259196/bc11790d-6c54-400b-8e5e-1ef000d91415" ></video>|
(video source: https://remix.run/blog-images/posts/remix-vs-next/remix-error.mp4)  

Remix 可以處理 App 中關於 data 和 render 的所有錯誤，甚至是 server 上的錯誤
- 你所要做的就是在 App 的 root 定義 [error boundary](https://remix.run/docs/en/v1/guides/errors)
- 甚至可以更細化，只拿下頁面中出現錯誤的部分。
- `Remix` 能做到，而 `Next.js` 做不到，唯一的原因是，`Remix` 的 data 抽象並沒有停留在如何將 data 引入 App，還包括如何改變它

### Remix vs Next.js: 關於 Interruptions
Users often click a button twice on accident and most apps don't deal with it very well. But sometimes you have a button that you fully expect the user to click really fast and want the UI to respond immediately.

In this app, the user can change the quantity of items in the cart. It's likely they'll click it very quickly to increment the number a few times.

Let's see how the Next.js app deals with interruptions

User 經常會點擊按鈕兩次
- 大多 App 沒有很好地處理這個問題
- 在這 App，User 可以改物品的數量。User 很可能非常迅速地點擊它，將數量增加

看看 `Next.js` 如何處理 interruption 的問題

| Next.js Interruption |
| :---: |
| <video width="100%" muted loop autoplay loading="lazy" src="https://github.com/flameddd/blog/assets/22259196/e7858242-c5cb-434e-9f8a-2e3a2448eebe" ></video> |
(video source: https://remix.run/blog-images/posts/remix-vs-next/change-quantity-next.mp4)

在中間有個從 5 到 6 再到 5 的奇怪東西。不過，最後幾秒鐘是最有趣的
- 可以看到，最後一個發送的請求落地（到 4），然後幾 frame 後，第一個發送的請求落地
  - 數字從 5 跳到了 6，再跳到 5。數字從 5 到 4，再跳到 2，沒有任何 interaction

這段 code 沒有管理 race conditions、interruptions 或 re-validation
- 所以現在的 user interface 可能與 server 不同步（取決於 2 還是 4 是最後碰到的 server side code）
- 管理 interruptions 和 mutations 後的 data re-validating 可以防止這種情況

看看 Remix 如何處理這個問題

| Remix Interruption |
| :---: |
| <video width="100%" muted loop autoplay loading="lazy" src="https://github.com/flameddd/blog/assets/22259196/89e02e33-1770-4cf7-9508-b167cc86cf57" ></video> |
(video source: https://remix.run/blog-images/posts/remix-vs-next/change-quantity-remix.mp4)

可以看到 Remix 在中斷時取消了 request
- 並在 POST 完成後重新驗證了 data。這確保了整個頁面的 user interface（不僅僅是這個表單）與 form 剛剛在 server 上所做的任何改變是同步的
- 你可能會想，也許我們的 App 比 `Next.js` App 更注重細節
- 這些行為都不在 App code 中。它都是 built-in 在 `Remix` 的 data mutation API 中的
  -  (它實際上只是在做 browser 對 HTML form 所做的事情)
- Remix 在 client side 和 server side 間的無縫集成和過渡是前所未有的

### Remix vs Next.js: Remix 擁抱 Web

在設計 Remix API 時，首先考慮到 platform，比如 mutation workflow
- 我們知道 HTML form API + server side handler 是正確的
- 所以圍繞這個建立
- 這並不是我們的主要目標，但一個非常驚人的副作用是，一個習慣性(idiomatic)的 Remix App 的核心功能不需要 JavaScript 就能工作

| Remix Without JavaScript |
| :---: |
| <video width="100%" muted loop autoplay loading="lazy" src="https://github.com/flameddd/blog/assets/22259196/971ccb88-90a7-45d9-83ab-bca316ddbb28" ></video> |
(video source: https://remix.run/blog-images/posts/remix-vs-next/no-js.mp4)  

雖然這樣用 Remix 是完全 OK 的
- 但我們並不打算讓你在沒有 JavaScript 的情況下建立網站
- Remix 團隊對建立 user interfaces 很大的野心，為此需要 JavaScript
- 與其說「`Remix` 在沒有 JavaScript 的情況下工作」，我們更願意說「`Remix` 在 JavaScript 之前工作」

Remix 在寫 server side code 時，也是著眼於 web platform
- Remix 沒有發明新的 JavaScript request/response API，而是用[Web Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- 為了處理 URL seatch params，使用 built-in `URLSearchParams`、處理 form data，我們用 built-in `FormData`

```js
export function loader({ request }) {
  // request is a standard web fetch request
  let url = new URL(request.url);

  // remix doesn't do non-standard search param parsing,
  // you use the built in URLSearchParams object
  let query = url.searchParams.get("q");
}

export function action({ request }) {
  // formData is part of the web fetch api
  let formData = await request.formData();
}
```

當你開始學習 Remix 時
- 你會在 [MDN](https://developer.mozilla.org/en-US/docs/Web) 上花費同樣多的時間，甚至比 Remix 的 doc 更多


### Remix vs Next.js: Optimizing for Change

現在你知道這兩個 frameworks 是如何做事的
- 讓我們看看 App 如何 respond to change
- 團隊在設計 Remix API 時經常談及 "optimize for change"

修改 Home Page:
- 假如你想修改 homepage 上的 production，在 `Next.js` 中有兩個選擇:

Let's consider you want to change the products on the home page, what does that look like? You have two choices in Next.js:
- 重新構建和重新部署 Aoo。build time 將隨著 production 數量的增加而線性增長
  - (每次構建都要從 Shopify 獲取每個產品的 data)
  - 單單改個 typo，就需要從 Shopify 下載每個 prodcution 來部署這個 update
  - 當商店增長到成千上萬的 production 時，這將成為一個問題
- 使用 [Incremental Static Regeneration](https://nextjs.org/docs/basic-features/data-fetching/overview#incremental-static-regeneration)
  - Vercel 意識到 SSG build time 的問題，所以有了 `ISR`
  - 當 page 被 request 時，server 會發送 cache 版本，然後在後台用新的 data rebuild 它。下位訪問者就會得到新的 cache版本
  - 如果在部署的時候，page 還沒有建立，`Next.js` 會在 server render 那個 page，然後 cache 在 CDN 上
    - 這正是 HTTP `stale-while-revalidate` 的作用，只是 ISR 有一個非標準的 API 和 vendor lock-in

在 Remix
- 只需通過 Shopify 更新 production
- production 就會在 caching TTL 內更新
- 也可以在一個下午設置一個 webhook，使 home page 查詢無效
- 這種 infrastructure 比使用 SSG 更費事
- 但它可以擴展到任何規模，任何類型的 UI（search page），並且實際上在有更多的 User 時比 SSG 更快（可以 cache 常見的 search query）
- 也不需要耦合到一個特定的主機，也幾乎不需要耦合到一個 framework，因為 Remix 主要使用 standard web APIs 來實現 App 邏輯

- 此外，Remix 認為只在 server 上用一種方式 loading data，帶來更 cleaner 的抽象。


### Remix vs Next.js: 關於 cache misses ?
Server 和 HTTP caching 只有在網站有流量的時候才起作用
- `Remix` 中 production page 的 empty cache hit 並不比 `Next.js` 的 search page（在那裡不能使用 SSG）
  - 你上次在逛線上購物時沒有搜索是什麼時候？隨著 cache 中常見 query 越來越多，它就變得更快
- 常見的 landing page 幾乎總是被引爆的，然後 Remix 的 prefetch 功能使下一個轉換瞬間完成
  - Remix 可以 prefetch 任何頁面，不管是動態的還是其他的，`Next.js` 不行
- 使用 SSG 到了一定的規模，就需要切換到 ISR。現在你不屬於上一次部署的 page 上，也會遇到同樣的 cache miss 問題
- 如果 cache miss request 在你的訪問中佔了很大一部分，那 100% cache hit rate 不能解決你的 business issue
  - 你沒有技術問題，有的只是 marketing problem

### Remix vs Next.js: Personalization
看看另一個變化。想像一下，產品團隊來找你，說 homepage 要顯示與 User 過去購買過的類似產品，而不是固定的產品列表
- 就像 search page 一樣，SSG 被淘汰了
  - 性能也被淘汰了。 SSG 確實有著有限的 use cases
- 隨著網站的發展，你將開始向 User 展示越來越多的 personalized information
  - 每次，這都會成為一個 client side fetch，漸漸的，你的 performance 就消失了
- 對 `Remix` 來說，這只是在後端進行不同的 db query


### Remix vs Next.js: Bottom Line
我們很容易忽視 Remix 簡單的 `<Form>`+`action`+`loader` APIs 的強大功能
- 將盡可能多的內容留在 server 上的設計。它改變了遊戲
- 這些 API 是 Remix 更快的 page load、更快的 transitions、圍繞 mutations（interruptions, race conditions, errors）的更好的 UX
- 以及對 developer 來說更簡單的 code

`Remix` App的速度來自 backend infrastructure 和 prefetching
- `Next.js` 則從 SSG 獲得速度。由於 SSG 的 use case 有限，特別是當功能和 data scale 時，就會失去這種速度


SSG 和 Jamstack 是 slow backend services 的很好的變通方法
- 最新一代的 platforms 和 databases 是快速的，而且只會越來越快
- 即使是支持這些 App 的 Shopify API 也可以在 200ms 內從世界任何地方發送查詢的 response
- Ryan Florence 在除了南極洲以外的每一個大洲都進行了測試


有些人說你可以用 `getServerSideProps` 做所有 Remix 的事情。這個問題來自於我們還沒有機會很好地解釋Remix!
- 如前所述，這會加快 search page 的速度。但，仍然有 data mutations 的問題要處理
- 你需要結合`getServerSideProps`, `API routes`，以及你自己的 browser code 與之溝通，以實現 mutations
  - （包括 error handling, interruptions, race conditions, redirects, and revalidation）
- 在這裡真正要說的是，的確，你可以。(Remix 已經做了）



(我: 到這邊為止就是官網 blog `Remix vs Next.js` 的內容，下面我估計會在筆記一些 Remix doc 我想看的部分)
------------------------------------------

剩下 doc 的部分主要分兩塊
- GUIDES: https://remix.run/docs/en/v1/guides/accessibility
- Remix Packages: https://remix.run/docs/en/v1/api/remix

Remix Packages 介紹 remix 自己為了 abstract 所提供的 API and react Component  
GUIDES 則是一些各主題的細節與思維，如 nest routing, error 架構等等  
這些都是比較偏實作時會用到的，這筆記就先不深入紀錄了  
