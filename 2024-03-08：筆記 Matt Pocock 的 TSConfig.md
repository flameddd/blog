# 2024-03-08：筆記 Matt Pocock 的 TSConfig.md
## Matt Pocock
### https://www.totaltypescript.com/tsconfig-cheat-sheet

----------------

## TL;TR

```json
{
  "compilerOptions": {
    /* Base Options: */
    "esModuleInterop": true,
    "skipLibCheck": true,
    "target": "es2022",
    "allowJs": true,
    "resolveJsonModule": true,
    "moduleDetection": "force",
    "isolatedModules": true,

    /* Strictness */
    "strict": true,
    "noUncheckedIndexedAccess": true,

    /* If transpiling with TypeScript: */
    "moduleResolution": "NodeNext",
    "module": "NodeNext",
    "outDir": "dist",
    "sourceMap": true,

    /* AND if you're building for a library: */
    "declaration": true,

    /* AND if you're building for a library in a monorepo: */
    "composite": true,
    "declarationMap": true,

    /* If NOT transpiling with TypeScript: */
    "moduleResolution": "Bundler",
    "module": "preserve",
    "noEmit": true,

    /* If your code runs in the DOM: */
    "lib": ["es2022", "dom", "dom.iterable"],

    /* If your code doesn't run in the DOM: */
    "lib": ["es2022"]
  }
}
```

-----------

## 幾個基本選項的說明

```json
{
  "compilerOptions": {
    // 幫助修補 CommonJS 和 ES Module 之間的一些界限
    "esModuleInterop": true,
    // 跳過檢查 `.d.ts` 文件的 type。這對 perf 很重要，否則所有 `node_modules` 會被檢查
    "skipLibCheck": true,
    // 針對的 JavaScript 版本。建議 `es2022` (`esnext` 中提供了其他 API 會隨著 JavaScript spec 發展而改動)
    "target": "es2022",
    // allowJs 和 resolveJsonModule 允許匯入 `.js` 和 `.json` 文件
    "allowJs": true,
    "resolveJsonModule": true,
    // 強制 TypeScript 將所有檔案視為 modules。有助於避免 "cannot redeclare block-scoped variable error"
    "moduleDetection": "force",
    // 這選項可阻止一些 TS features，這些 features 在將 modules 視為獨立檔案時不安全
    "isolatedModules": true
  }
}
```

Ref: [cannot redeclare block-scoped variable error](https://www.totaltypescript.com/tsconfig-cheat-sheet)  


-----------

## 一些限制選項的說明

```json
{
  "compilerOptions": {
    // 啟用嚴格的 type checking
    "strict": true,
    // 阻止在未先檢查 array/object 是否已定義的情況下存取該 array/object。這是防止 run time error 的好方法
    "noUncheckedIndexedAccess": true
  }
}
```

網路上有很多人會推薦官方的 `strictest` config
- https://github.com/tsconfig/bases/blob/main/bases/strictest.json
- 裡面還有好幾個限制，如 `noUnusedLocals`, `noImplicitReturns`, `noUnusedParameters`, `noFallthroughCasesInSwitch`，建議自己了解需求後加入這些
- TS 官方的 config 還有針對很多其他環境給出 base config

------------

## 轉譯 TS 的相關選項

```json
{
  "compilerOptions": {
    // 如何解析 module。如果 code 要在 Node 中跑，則最佳為 `NodeNext`
    "moduleResolution": "NodeNext",
    // 使用什麼 module 語法。 `NodeNext `是 Node 的最佳選擇
    "module": "NodeNext",
    // 編譯後的 JavaScript 檔案放在哪裡
    "outDir": "dist"
  }
}
```

### 如果是在打造 Library 的話 
```json
{
  "compilerOptions": {
    // 告訴 TypeScript 要發行 `.d.ts` 文件。這是必要的，以便 library 可以在 `.js` 檔案中被使用時，能有 **auto complete** 功能
    "declaration": true
  }
}
```

### 如果 Library 是在 Monorepo 底下的話 

```json
{
  "compilerOptions": {
    "declaration": true,
    // 告訴 TypeScript 要發行 `.tsbuildinfo` 文件。這告訴 TypeScript 你的專案是 `monorepo` 的一部分，並且有助於 cache builds 以加快速度
    "composite": true,
    // sourceMap 和 declarationMap 吿訴 TypeScript 要發行 `source map` 和 `declaration map`。以便當 library 的 user  debug 時，可以使用 `go-to-definition` 跳到 source code
    "sourceMap": true,
    "declarationMap": true
  }
}
```

-----------------

## 當不使用 TypeScript 進行轉譯時
例如只是拿來做 linter 之類的時候

```json
{
  "compilerOptions": {
    // 如果 code 用 Webpack, Rollup, Babel, SWC or ESBuild 等工具捆綁，那 `Bundler` 是最佳選擇
    "moduleResolution": "Bundler",
    // `preserve` 是最好的選擇，因為它最接近 bundlers 處理 module 的方式
    "module": "preserve",
    "noEmit": true
  }
}
```

------------

## Code 要跑在 DOM 上面時
```json
{
  "compilerOptions": {
    // `es2022` 是最穩定的選擇，`dom` 和 `dom.iterable` 會給你 `window`, `document` 這些的 type 
    "lib": ["es2022", "dom", "dom.iterable"]
  }
}
```

### Code 不是跑在 DOM 上面時
```json
{
  "compilerOptions": {
    "lib": ["es2022"]
  }
}
```