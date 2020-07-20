# JavaScript tree shaking
## Daniel Brain 2020 Jan 20
### https://medium.com/@bluepnume/javascript-tree-shaking-like-a-pro-7bf96e139eb7
### https://webpack.js.org/guides/tree-shaking/

JavaScript tree shaking 是
- 避免 large bundle sizes，改善 performance.

基本三個原則
1. 為每一個 modules 宣告好 `imports` 和 `exports`
2. 在 compilation 階段時，利用 bundler (Webpack, Rollup) 分析好 dependency tree
3. 所有沒用到的 code 都在 bundle 時 remove 掉


但還有很多小細節要確認，避免 bundler 跳過了 tree shaking

## 使用 ES6 style imports and exports
- 使用 `import` and `export`
- `commonjs` 跟 `require.js` 在 build 階段時，都是 non-deterministic （無法斷定的）
  - 這樣 bundlers 就沒辦法決定能不能 drop 掉

```js
// `commonjs` 跟 `require.js` 這種方式都能 import export
module.exports.foo = function foo() {}
module.exports[someVariable] function foo() {}

function exportFoo() {
  module.exports.foo = function foo()
}
function importFoo() {
  const { foo } = require(baseDirectory + '/foo. js');
}
```

ES6 style 的 `import` and `export` 相對功能限制很多
- 只能在 module 層級做 `import` and `export`，不能在 function 裡面
- `import` and `export` 只能用 static strings，不能用變數
- `importing` 的東西，一定是從某個地方有被 `export` 的

所以讓 bundler 能好好分析、決定

```js
export function foo() {}
import { foo } from './foo'
```

## 不要讓 `Babel` transpile 了 imports and exports
- 如果你有用 `Babel`，它**預設**會把所有 `import` and `export` 轉為 `commonjs`
  - 這樣 Webpack 就沒法最佳化 tree shaking 了
  - https://babeljs.io/docs/en/babel-preset-env#modules

## 盡量拆成小東西來 exports
如果是這樣 export 的話，Webpack 會保留整個部分
- export 一個很多 properties 和 methods 的 object
- export 一個很多 methods 的 class
- 用 `export default` 然後包含很多東西

```js
// 這邊的例子，只要有用到，就是整段 code 都包含
export default {
  add(a, b){
    return a + b;
  },
  subtract(a, b){
    return a b;
  }
}

export class Number {
  constructor(num) {
    this. num = num
  }
  add(othernum) {
    return this.num + othernum;
  }
  subtract(othernum 0) {
    return this.num + othernum;
  }
}
```

要像這樣，盡量拆小
```js
export function add(a, b) {
  return a + b;
}
export function subtract(a, b){
  return a - b;
}
```

這篇推薦怎麼寫更 function style 的 javascript
- https://medium.com/@bluepnume/functional-ish-javascript-205c05d0ed08

## 避免 module-level 的 side effects

```js
export function add(a, b) {
  return a + b;
}

export const memoizeAdd = window.memoize(add);
```

這種寫法，`window.memoize` 會在 module **被 import 時執行**，Webpack 會這樣看待
1. 恩，這裡有個 pure exported function `add`，如果沒人用，也許可以從 bundle 移除
2. 接著有呼叫 `window.memoize`，然後傳入 `add`
3. 不知道 `window.memoize` 做什麼用的，但有可能會呼叫 `add`，然後有觸發其他 `side-effect`
4. 為了保險，我必須保留 `add`

而 Webpack 和 Terser 最新版本在檢測是否確實會觸發副作用方面做得非常出色。
如果要改為這樣：
```js
// 多給 Webpack 一些資訊
import { memoize } from './util';

export function add (a, b) {
  return a + b;
}

export const memoizeAdd = memoize(add)
```

現在的話，流程會是
1. 這邊會在 module level 呼叫 `memoize`，這可能會有問題
2. 但，`memoize` 是一個 ES6 import，那我們可以去 `util.js` 裡面看看
3. `memoize` 看起來是個 pure function，所以不會有 side-effect 風險
4. 所以，如果沒人用 `add` 的話，可以從 bundle 移除 (我看不太懂，最後 memoize 不是有使用嗎＠＠？)

## 使用第三方 libraries 時
要研究看看有沒有辦法被 tree shaking，最好的例子就是 lodash
- import _ from 'lodash'; // 這樣就整包進去了
- import xxx from 'lodash/lib/xxx'; // 寫法是不是這樣要查一下，但大概就是這概念，只 import 需要的

## 關於 Webpack
- 記得要 deploy 時 mode 要開 production
- tree shaking 真的很難用肉眼去判斷，最保險的話，還是 bundle 打包後，跑 Analyzer 看看
  - https://github.com/webpack-contrib/webpack-bundle-analyzer

side-effect-free
如果所有 code 都不包含 side effect，可以將該屬性標記為 false，來告知 webpack，它可以安全地刪除未用到的 export。
```js
// package.json
{
  "name": "your-project",
  "sideEffects": false
}
```

"side effect" 的定義是，在導入時會執行特殊行為的代碼
而不是僅僅暴露一個 export 或多個 export。舉例說明
- 例如 polyfill，它影響全局作用域，並且通常不提供 export。

如果 code 確實有些副作用，可以 set Array：

```js
{
  "sideEffects": [
    "./src/some-side-effectful-file.js"
  ]
}
```

```js
// 如果 project 中使用類似 css-loader 並 import 一個 CSS 文件，則需要加到 side effect 列表中，以免在 production 模式中將它刪除：
{
  "sideEffects": [
    "./src/some-side-effectful-file.js",
    "*.css"
  ]
}
```

## Sean (Webpack 的維護者之一)，說明 Webpack 的 sideEffects
- whenever a module reexports all exports (regardless if used or unused) need to be evaluated and executed in the case that one of those exports created a side-effect with another.
  - 當一個 module 重新 export 出所有導出(無論是否會被用) 需要被計算和執行時，其中一個 export 對其他的 export 產生了副作用。
  - https://stackoverflow.com/questions/49160752/what-does-webpack-4-expect-from-a-package-with-sideeffects-false

在 vuejs 時，也有說明 sideEffects
- https://github.com/vuejs/vue/pull/8099
- 看起來意思是，當有人需要 import vuejs，而不是打包 bundle 時，webpack 搭配此參數，可以做更好的 tree shaking
