# 2023-05-23：筆記 PortSwigger 的 Clickjacking.md

Ref:
- https://portswigger.net/web-security/clickjacking

---

## 什麼是 click jacking ?

是種基於 interface 的攻擊
- 即通過點擊誘餌網站中的一些其他內容，誘使使用者點擊隱藏網站中的可操作內容


使用者訪問了一個誘餌網站（也許是由 email 提供的連結)，並點擊一個按鈕來贏取一個獎項
- 在不知情的情況下，他們被攻擊者欺騙，按了另個隱藏的按鈕，結果是在另個網站上支付一筆費用

這是 click jacking 攻擊的例子
- 這技術取決於將一個看不見的、可操作的網頁(或多個網頁)納入其中，其中包含一個按鈕或隱藏連結
  - 如，在一個 iframe 中。該 iframe 被疊加在使用者預期的誘餌網頁內容之上
- 這攻擊與 CSRF 攻擊的不同之處在於，**使用者需要執行一個動作**
  - 如點按鈕，而 CSRF 攻擊則取決於在使用者不知情或不輸入的情況下偽造整個請求


CSRF 的保護通常是 CSRF token: 一個針對 session 的、一次性使用的數字或 nonce
- CSRF token 不能緩解 click jacking 攻擊，因為目標 session 是通過從真實網站載入內容建立的，而且所有 request 都是在 domain 內發生的
- CSRF token 被置於 request 中，並作為正常行為 session 的一部分傳遞給 server
- 與正常使用者 seesion 不同的是，該過程發生在個隱藏的 iframe 中

---

## 如何構建基本的 clickjacking 攻擊

clickjacking 攻擊使用 CSS
- 攻擊者將目標網站作為一個 iframe 疊加在誘餌網站上

例子如下：
```html
<head>
  <style>
    #target_website {
      position:relative;
      width:128px;
      height:128px;
      opacity:0.00001;
      z-index:2;
      }
    #decoy_website {
      position:absolute;
      width:300px;
      height:400px;
      z-index:1;
      }
  </style>
</head>
...
<body>
  <div id="decoy_website">
  ...decoy web content here...
  </div>
  <iframe id="target_website" src="https://vulnerable-website.com">
  </iframe>
</body>
```


目標網站的 iframe 在 borwser 中被定位，使用適當的寬度和高度位置值使目標行動與誘餌網站重疊
- `z-index` 決定了 iframe 的堆疊順序。不透明度值被定義為 `0.0` (或接近 `0.0`)，以便 `iframe` 對使用者是透明的
- Browser clickjacking 保護可能會對基於閾值的 iframe 透明度檢測(如，Chrome，但 Firefox 沒有(要進一步看看有沒有改進))
  - 攻擊者選擇不透明度值，以便在不觸發保護行為的情況下達到預期效果

---

## Clickbandit (點擊土匪? )

儘管可以如上所述手動建立 clickjacking 的 POC
- 但實踐中是相當繁瑣和耗時的。當在野外測試 clickjacking 時，建議使用 [Burp's Clickbandit](https://portswigger.net/burp/documentation/desktop/tools/clickbandit) 工具
- 它用 browser 在可框架的頁面上執行所需的動作，建立一個包含合適的 clickjacking 覆蓋的 HTML 文件
  - 可以在幾秒鐘內產生一個互動的 POC，不需要寫 HTML 或 CSS

---

## 用預填充的 form input 進行 Clickjacking

一些需要填寫和 form 的網站允許在 submit 前使用 GET 參數預填充 input
- 其他網站可能要求在 form 提交前提供內容
- 由於 GET 值是 URL 的一部分，那麼目標 URL 可以被修改，以納入攻擊者選擇的值
- 透明的 "submit" 按鈕被覆蓋在誘餌網站上，是基本的 clickjacking 例子。

---

## Frame busting scripts (框架破壞腳本)

Clickjacking attacks are possible whenever websites can be framed. Therefore,
preventative techniques are based upon restricting the framing capability for
websites. A common client-side protection enacted through the web browser is to
use frame busting or frame breaking scripts. These can be implemented via
proprietary browser JavaScript add-ons or extensions such as NoScript. Scripts
are often crafted so that they perform some or all of the following behaviors:

## 框架破壞腳本

只要網站有 framed，就有可能發生 Clickjacking 攻擊
- 因此，預防技術的基礎是限製網站的 framing capability
- 通過 browser 實施的一種常見的 client side 保護是使用框架破壞 script
  - 可以通過專有的 browser JavaScript extensions (如 NoScript )來實現
  - script 通常是精心設計的，以便它們執行以下一些或全部行為
    - 檢查並強制執行當前的 App window 是 main window 或 top window
    - 使所有 frames 可見
    - 防止點擊不可見的 frames
    - 攔截並向使用者提示潛在的 clickjacking 攻擊

框架破壞技術通常是針對 browser 和平台的，由於 HTML 的靈活性，它們通常可以被攻擊者規避
- 由於框架破壞是JavaScript，那麼 browser 的安全設置可能會阻止其操作，或者 browser 甚至可能不支持 JavaScript
- 攻擊者對付框架破壞者的一個有效辦法是用 HTML5 的 iframe `sandbox` 屬性
  - 這個屬性與 `"allow-forms"` 或 `"allow-scripts"` 一起設置，並省略 `"allow-top-navigation"`，那框架破壞 script 就可以被化解
  - 因為 iframe 無法檢查它是否是 top window

```html
<iframe id="victim_website" src="https://victim-website.com" sandbox="allow-forms"></iframe>
```

`allow-forms` 和 `allow-scripts` 這兩個值都允許在 iframe 中指定
- 但 top-level navigation 被禁用。這抑制了破壞框架的行為，同時允許在目標網站內的功能

---

## 結合 clickjacking 和 DOM XSS 攻擊
從歷史上看，clickjacking 被用來執行一些行為，如提高 Facebook 頁面上的 `like`
- 然而，clickjacking 被用作另種攻擊(如 DOM XSS)的載體時，它的真正威力就顯現出來了
- 假設攻擊者首先確定了 XSS，這種聯合攻擊的實施就相對簡單了
- 將 XSS 與 iframe 目標 URL 相結合，使使用者點擊按鈕或連結，從而執行 DOM XSS 攻擊

---

## 多步驟 clickjacking
攻擊者對目標網站輸入的操縱可能需要多個動作
- 如，攻擊者可能想欺騙使用者在零售網站上購買東西，因此在下訂單之前需要將物品加到購物籃中
- 這些行動可以由攻擊者使用多個分部或 iframe 來實現
- 如果要做到有效和隱藏，從攻擊者的角度來看，這種攻擊需要相當精確和謹慎

---


## 如何防止 clickjacking 攻擊
我們已經討論了一個經常遇到的 browser 端預防機制，即框架破壞 script
- 然而，攻擊者往往可以直接規避這些保護措施
- 因此，server side 驅動的協議已經被設計出來，以限制 browser iframe 的使用並減輕對點擊劫持的影響

Clickjacking 是種 browser 端行為，其成功與否取決於 browser 的功能和對現行網路標準和 best practice 的遵守情況
- server side 對 Clickjacking 的保護是通過定義和交流對組件(如 iframe)使用的限制來實現的
  - 然而，保護的實施取決於 browser 對這些約束的遵守和執行
- server side 保護的兩個機制是 `X-Frame-Options`和 `Content Security Policy`

---

## X-Frame-Options
該 header 為網站提供了對 iframes 或對象使用的控制，可以用 `deny` 禁止在 frame 中包含網頁

```
X-Frame-Options: deny
```

可以用 `sameorigin` 將 frame 限制在與網站相同的 origin 上

```
X-Frame-Options: sameorigin
```


使用 `"allow-from"` 限制指定的網站

```
X-Frame-Options: allow-from https://normal-website.com
```

`X-Frame-Options` 在各 browser 中的實現不一致(如，Chrome 76 和 Safari 12 不支持 `allow-from`）
- 如果正確地與 CSP 一起用，作為多層防禦策略的一部分，它可以提供有效的保護，防止 clickjacking 攻擊

---

## Content Security Policy (CSP)

CSP 對 XSS 和 clickjacking 等攻擊提供緩解

```
Content-Security-Policy: policy
```

`policy` 是一串由分號分隔的指令
- 推薦的 clickjacking 保護是加入 `"frame-ancestors"`
- `frame-ancestors 'none'` 的行為類似於 `X-Frame-Options deny`
- `frame-ancestors 'self'` 大致相當於 `X-Frame-Options sameorigin`


以下的 CSP 白名單只針對同一 origin 的 frame：
```
Content-Security-Policy: frame-ancestors 'self';
```


也可以將 frame 限制在指定的網站上：

```
Content-Security-Policy: frame-ancestors normal-website.com;
```

為了有效防止 clickjacking 和 XSS，CSP 需要仔細開發、實施和測試，並應作為多層防禦策略的一部分

Read more
- [Protecting against clickjacking using CSP](https://portswigger.net/web-security/cross-site-scripting/content-security-policy#protecting-against-clickjacking-using-csp)
