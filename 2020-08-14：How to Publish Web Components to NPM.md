# How to Publish Web Components to NPM
## Justin Fagnani | 01 Nov 2019 (lit-html, lit-element maintainer)
### https://justinfagnani.com/2019/11/01/how-to-publish-web-components-to-npm/

下面是 Justin 自己 release web components 的 checklist
- 目標是提高 compatibility, standards compliance, flexibility, and usefulness to your users.
- 這些方法會隨時間而調整，但這就是 2019 年作者目前的心得

Justin's Checklist for Publishing Web Components to NPM
- Publish `standard ES2017`
- Publish standard JavaScript modules
- Do not use `.mjs` file extensions
- Only publish a single build
- Important `package.json` fields:
  - a. Set `"type"` to `"module"`
  - b. Set `"main"` to the main entry point module
  - c. Set `"module"` to the same file as `"main"`
  - d. Include polyfills in devDependencies, not dependencies
- Do not bundle
- Do not minify
- Always self-define elements
- Export element classes
- Do not import polyfills into modules
- Import dependencies with `"bare"` or `"named"` import specifiers
- Always include file extensions in import specifiers
- Publish a custom-elements.json file documenting your elements
- Include good TypeScript typings

## Publish standard ES2017
- 與舊版 JS（如ES5）相比，modern JavaScript size 更小，速度更快，功能更強大
  - 並且絕大多數用戶都擁有支持它的 browser

因此，最好將最新的 JavaScript 發送給 User  
但是，如果沒有 modern JS
- 則無法將其發送到瀏覽器
- packages 的 User ，當有需要時，可以根據需要將 code 編譯為 lower language level
  - 但他們沒辦法把 code 反編譯為較新的語言版本 (所以我們應該包含一起發佈較新的版本)
  - 這樣是能提供最大的靈活度


選 ES2017 是因為它
- 在Chrome，Safari，Firefox 和 Edge 中得到了廣泛的支持，對於大多數瀏覽器，根本不需要編譯
- https://kangax.github.io/compat-table/es2016plus/

如果 User 需要支持 IE11 呢？
- 需要 compile `node_modules/` 裡面的 dependencies 

## Publish standard JavaScript modules
- modern borwser 和 tool-chains 都已經支援 standard JavaScript modules 了
- 如果 publish standard JavaScript modules， borwser 和 tool-chains 就能不要編譯就能載入
- 這對開發非常有幫助尤，對 code 轉換越少，debug 的體驗就越好

Native modules 非常適合在開發過程中使用，不需要 bundle
- A static file server that properly sends `304` response codes for unmodified files will take maximum advantage of the browser's cache and only send the files that changed.

## Do not use .mjs file extensions
- `.mjs` 是有爭議的，對 browser 來說毫無用處。使用它真的沒有任何好處
- browser 只關心 file 的 mime type，而不關心 file extension
  - 因此該文件 `.mjs` 不執行任何操作

`.mjs` 有一個缺點
- 不是所有工具都支援 mjs
- 某些 static file servers 可能無法正確送出 `mime-type` header
  - 如果是這樣，file won't load as a module

最好的辦法是
- avoid it altogether
- always write and publish `modules`, and always use the `.js` extension.

## Only publish a single build

在 npm 上發布多個版本很普遍
- 這是一種不良且過時的做法，可能導致 application bundle size 變大
  - 原因是多個 package 可能共同使用同一個 dependency，但是如果它們導入不同的內部版本，則 bundle 最終可能就存在多個「同一個 dependency」

對 web component，這特別危險
-  we really need there to be a single definition of a component. Multiple versions here is a bigger headache than just bloat.


我真的建議要 follow 這規則
- 一定會有 issue 來要求你 release ES5, UMD build (這時候，還是 publish a single build)
- 用 standard JS modules 的話，就能在 build time 時，建構出不同 format，就不需要 publish multiple builds

## Important `package.json` fields
### Set `"type"` to `"module"`
- `"type"` 是 npm package 的標準方式，來指出 JS  file `modules`
- 一些工具跟 `CDNs` 都會靠這個判斷、來 parse files with module parse goal
- 使用這 field，就不需要使用 `.mjs` （即使在 nodejs 上）

### Set `"main"` to the main entry point module
- 這是一個標準的做法、npm 需要 `"main"` entry

### Set `"module"` to the same file as `"main"`
`"type": "module"` 是最正確的方式指定 package 包含 `modules`

大多 tools 已經支援 `"module"` field 了
- 但直到所有 tools 都支持 `"type"` field 之前，我們都應該設定 `"module"`

## Include polyfills in `devDependencies`, not `dependencies`
- Polyfills 是 application 關心的東西，如果 app 有需要，那應該**直接**依賴它們
- Packages 用到 polyfills 有時候只是為了 tests and demos

所以對 packages 來說
- Polyfills should only go in `devDependencies`

## Do not bundle
- 這點需要考慮到 package structure
- 最重要的一點是，**不要 bundle dependencies**
  - This keeps you from causing bloat by duplicating dependencies into multiple package bundles.
- As long as you don't bundle dependencies, you may decide to bundle your package in order to hide implementation modules. If you do, make sure you preserve all the valid entry points of your package as separate entry point files into a set of bundles with shared chunks.
  - (沒有很理解這句話，不過把所有的東西分開有單一路口，這也是很多 UI framework 都有做的。就能單獨 import 所需要的 component)

如果您支持 import `my-library/element-a.js` 並且 `my-library/element-b.js` 沒有 boundle 在一起
- browser's module loader 就會是 tree-shaker，只 load 有需要的

重要的是
- (設計層面上) keep modules relatively small and single-purposed
- 讓 User 能只 load 需要的部分
- Bundlers 也能因此更快的 bundle 出更小的檔案

我發現完全不 bundle libraries 更為簡單
- Application 的 build 會去採用它需要的方式

## Do not minify
- minification 應該是 application 要去關心的
- non-minified code 方便 debug
- minifiers 會越來越好、output 更小（所以讓 application 要去 build 時再去做就好）

so don't bake this into your published files.

## Always self-define elements

(這一點就是完全關於 `web component` 的事情)
- 宣告 `web component` class 時，一定要 `customElements.define()` 來宣告 element
  - https://developers.google.com/web/fundamentals/web-components/customelements?hl=zh-tw
  - 這是一個有爭議的點，但目前也沒有更好的方法
  - 有些人希望允許 component 的 user 能選擇標籤名稱，因此他們主張放棄 `customElements.define()` 或將其放在單獨的 module 中，但這實際上效果不佳。


Custom elements 現在需要 `tag names` 跟 `classes` 有 strict one-to-one relationship
- 所以，如果 custom element 沒有 self-define 時，可能會有多個 user 嘗試去 define，但最後只有第一個去 define 的人會成功

如果 user 真的想選用不同的 `tag name`，可以
- create a trivial subclass of the element and register

像這樣，就有 `<my-some-element />` 可以用了
```js
import {SomeElement} from 'some-element';

customElements.define('my-some-element', class extends SomeElement{});
```

關於這點，未來絕對會有所改變的，一旦支援了 `scoped custom element registries`
- https://github.com/w3c/webcomponents/issues/716


## Export element classes
為了要支援上面提到的 trivial subclassing pattern, subclassing
你應該要 export the element class

```js
export class MyElement extends LitElement {
  // ...
}
customElements.define('my-element', MyElement);
```

## Do not import polyfills into modules
這是很重要的一個基本準則，即使這篇是談 web component 也值得提出來
- 不要將 `Polyfill` import 到 modules

跟上面很多原則一樣
- 需不需要 polyfill 只有 Application 知道
- 大多 User 不需要 web component polyfill
  - (如果你直接 import polyfill 了，當 User 不需要時，User 很難處理)
- 或者使用 `webcomponents-loader` 動態 load，只在有需要時載入
  - https://github.com/webcomponents/polyfills/tree/master/packages/webcomponentsjs

## Import dependencies with "bare" or "named" import specifiers
- (這點後半段，也是搞不懂想表達什麼，未來能改成 "import maps" 的好處是？可以不經過 bundler 處理過就能使用這樣嗎？)

Nodejs 本來就支持 CommonJS modules with require() and node module resolution

```js
const otherLib = require('other-lib');
```

現在，developers 已經非常習慣用 standard JS module syntax

```js
import * as otherLib from 'other-lib';
```

這叫做 `bare import specifiers` 或者叫 `named imports`，但 browsers 還沒有支援  
Browsers 只支援 importing by URL，所以所有的 import 一定是
- 完整的 URL (http://) 或者
- 相對的 URL (starting with `/`, `./` or `../`

`bare import specifiers` 在 browsers 能 work 是因為經過 bundler (Rollup, Webpack) 處理過  

那如果我們要 publish standard modules 到 npm，User 使用時，不需要透過 compilation，並且我們還有相對的 dependencies 時，我們該怎麼做？  

可以用**相對路徑**

```js
import * as otherLib from '../other-lib/index.js';
```

但這種做法需要我們準確的知道某一個 `other-lib` 的系統相對位置
- 但！npm 允許 install modules 在好幾個不同的位置，所以我們不可能知道

所以，目前最好的做法還是
-  import dependencies by name and let tools rewrite the import specifiers before they reach the browser

In the near future
- browsers will support "import maps" that will tell them how to translate names into URLs, so they too will support named imports.

## Always include file extensions in import specifiers
- (這點我看了很久，實在不太懂這個目的。我們一般在使用時，也是不需要 extensions，也能順利在 browser 上運作，這可能是靠 bundler 的幫忙。那如果有 bundler 存在，還需要這點嗎？)

Nodejs module 不需要 extensions，因為它會搜尋 file system 找各種 extensions  
- 當 `import some-package/foo`，Nodejs 會 `import some-package/foo.js`
- 但，這**不適用 browser**

`Import maps` 可以 mapping names to URLs，但只有兩種 mapping 方式
- `exact` and `prefix`
-  more info: https://dev.to/diegosanchezp/using-nodejs-packages-in-the-browser-with-import-maps-38ei

example for lodash
```json
{
  "imports": {
    "lodash": "/node_modules/lodash-es/lodash.js",
    "lodash/": "/node_modules/lodash-es/"
  }
}
```

This means that we can easily map a bare specifier like lodash, or a prefix + full file path, like `lodash/forEach.js`, etc., but to support extensionless imports like `lodash/forEach`, we'd have to map every one to the full path, like:

```json
{
  "imports": {
    "lodash": "/node_modules/lodash-es/lodash.js",
    "lodash/": "/node_modules/lodash-es/",
    "lodash/forEach": "/node_modules/lodash-es/forEach.js"
  }
}
```

We would have to do this for every extensionless import in the entire app. lodash-es has 341 modules that could be imported. Creating entries for each one that's imported without extensions would make for a bloated import map, **so it's much better to just use extensions in imports and have only a prefix import map entry.**


## Publish a `custom-elements.json` file documenting your elements
- (關於這點，我嘗試去找了幾個知名的 web component library，都沒看到這檔案，這用途可能還不是很有用？ or 實作的方法不一樣？)

Web components tools，像是
- IDE plugins: https://marketplace.visualstudio.com/items?itemName=runem.lit-plugin
- catalogs: https://catalog.open-wc.org/

都開始使用一種通用的格式來描述發佈在 npm 上的 custom elements

`custom-elements.json` 描述了
- `tag names`
- `attributes`
- `properties`
- `events` 等等

這些能讓 IDE 提供
- auto-completion
- hover-over docs
  - (看一下上面的 lit-plugin 展示就知道了，就是在 code 上 hover 就有提示出現)

linters 可以檢查您是否正在使用已定義的屬性  
type checkers 可以確保 property 綁定的類型正確  
documentation viewers can display the information for human consumption  
catalogs like Storybook can generate "knobs" for components automatically
  - (`knobs` 未來會淘汰了，不過相信 storybook 內建的也能很好支援)

總之，`custom-elements.json` 能提升 develop exp

## Include good TypeScript typings
作者最喜歡 typeScript 一點是
- **type system** 能夠用 string keys 來 type APIs

用 `document.createElement()` 舉例
- `createElement` 的 return 類型取決於出輸入的 string
- `document.createElement('div')` 會 return `HTMLDivElement`
- `document.createElement('img')` 會 return `HTMLImageElement`

這就能讓我們做這樣的檢查
```js
document.createElement('div').src = './image.jpg'; // error!
document.createElement('img').src = './image.jpg'; // fine :)
```

最好的地方就是，我們可以 extend 這個基於從 tag name 到 class 的 mapping
- 這叫做 `HTMLElementTagNameMap`
- 所以要 extend element 時，就使用 TypeScript's interface augmentation

TypeScript 中完整自定義元素定義，應該像這樣：
```js
export class MyElement extends LitElement {
  // ...
}

customElements.define('my-element', MyElement);

declare global {
  interface HTMLElementTagNameMap {
    "my-element": MyElement,
  }
}
```
