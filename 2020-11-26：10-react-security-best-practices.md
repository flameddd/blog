# 10-react-security-best-practices
## Ron Perris October 28, 2020
### https://snyk.io/blog/10-react-security-best-practices/


## 1.Default XSS Protection with Data Binding

使用curly braces `{}` 來做 data binding
- **這樣做的話，react 會自動幫忙做 escape values，來避免 XSS attacks**
- 這個 escape values 的行為
  - 只有在 **rendering textContent**
  - 不會在  **rendering HTML attributes**

所以
```js
// Use JSX data binding syntax {} to place data in your elements. 
// Do this:

<div>{data}</div>
```

```js
// Avoid dynamic attribute values without custom validation.
// Don’t do this:

<form action={data}>
```


## 2.Dangerous URLs

URLs can contain dynamic script content via javascript:
- Use validation to assure your links are **http:** or **https:** to avoid javascript URL based script injection.

```js
// Do this:

function validateURL(url) {
  const parsed = new URL(url)
  return ['https:', 'http:'].includes(parsed.protocol)
}

<a href={validateURL(url) ? url : ''}>Click here!</a>

// Don’t do this:
<a href={attackerControlled}>Click here!</a>
```


## 3.Rendering HTML
使用 `dangerouslySetInnerHTML` insert HTML 時，一定要先消毒
- Use a sanitization library like [dompurify](https://www.npmjs.com/package/dompurify) on any values before placing them into the `dangerouslySetInnerHTML` prop.

```js
import purify from "dompurify";
<div dangerouslySetInnerHTML={{ __html:purify.sanitize(data) }} />
```


## 4.Direct DOM Access
避免直接對 DOM 操作、inject content 之類的

如果一定需要 inject HTM 的話
- 使用 `dangerouslySetInnerHTML`
- 並且先消毒再傳進去 (用 [dompurify](https://www.npmjs.com/package/dompurify) 處理)
  - (dompurify minified + gzip 過後 6.4k，稍微有點大)

```js
// Do this:
import purify from "dompurify";
<div dangerouslySetInnerHTML={{__html:purify.sanitize(data) }} />


// Don’t do this:
this.myRef.current.innerHTML = attackerControlledValue;
```


## 5.Server-side Rendering
server-side render 做 data binding 時，都會 automatic content escaping
- 像 `ReactDOMServer.renderToString()` and `ReactDOMServer.renderToStaticMarkup()` 

避免在送出前，對 `renderToStaticMarkup()` 的 output 去做 concat，避免 **XSS**

```js
// BAD, DO NOT
app.get("/", function (req, res) {
  return res.send(
    ReactDOMServer.renderToStaticMarkup(
      React.createElement("h1", null, "Hello World!")
    ) + otherData;  // <-- DO NOT do this. 
  );
});
```

## 6.Detecting Vulnerabilities in Dependencies
third-party components/library 可能會有 security issues
- Check your dependencies and update when better versions become available.

使用 `Snyk CLI` 撿查
```shell
npx snyk test
```

## 7.Injecting JSON State
server-side render 時，很常 send **JSON** data
- **一定要 escape "<" 符號 來避免 injection attacks**

```js
window.__PRELOADED_STATE__ = ${JSON.stringify(preloadedState).replace( /</g, '\\u003c')}
```

為什麼是 `\\u003c`，而不是 `\u003C` ?
- `U+003C` 是 < Less-than sign
- `U+003E` 是 > Greater-than sign

看來就是換成 unicode，然後 SSR 到 browser 時，能避免 injection，但還是能顯示 "<" 的樣子出來。


## 8.Detecting Vulnerable Versions of React
React 也是有幾版存在高風險的安全性問題的
- 有機會就升到最新版
- 或者至少「小版號」和「patch」追到最新

## 9.Configuring Security Linters
使用 linter 來幫忙自動找出安全問題
- ESLint React security config: https://github.com/snyk-labs/eslint-config-react-security/
- 直接進 repo 看 `index.js` 上面建議的 rule setting
  - https://github.com/snyk-labs/eslint-config-react-security/blob/master/index.js

```js
{
  rules: {
    // https://github.com/yannickcr/eslint-plugin-react/tree/master/docs/rules
    "no-danger": "warn",
    "no-find-dom-node": "warn",
    "jsx-no-script-url": "warn",
    "jsx-no-target-blank": "warn",
    "jsx-props-no-spreading": "warn",
    // https://github.com/snyk-labs/eslint-plugin-react-security
    "no-refs": "warn",
    // https://github.com/mozilla/eslint-plugin-no-unsanitized
    "no-unsanitized/method": "error",
    "no-unsanitized/property": "error",
  },
}
```

上面的網址應該才是 plugin，要去安裝，然後設定這些 rule 在自己專案的 config 上

最後是說 `Snyk` 能自動幫忙升級新版
- https://support.snyk.io/hc/en-us/articles/360011450338-Integrate-with-your-SCM
- (這段不太懂。但一般我會手動查哪些 library 版本太低，在自己研究 upgrade 並且處理 breaking change)

## 10.Avoiding Dangerous Library Code
有些 library 可能會有些危險的操作
- 例如直接 inserting HTML

這種情況只能
- Review library code manually
- 或者用 `linters` 嘗試找出不安全的地方
Library code is often used to perform dangerous operations like directly inserting HTML into the DOM. Review library code manually or with linters to detect unsafe usage of React’s security mechanisms.

就是要想辦法避免其他 lib 去操作
- `dangerouslySetInnerHTML`, `innerHTML`, (redirect/forward) `unvalidated URLs` 這類危險操作
- Use security Linters on your node_modules folder to detect unsafe patterns in your library code
  - (這點應該是指用 `Snyk`)
