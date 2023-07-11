
# 2023-05-23：筆記 PortSwigger 的 CSRF.md

Ref:
- https://portswigger.net/web-security/csrf
- https://portswigger.net/web-security/csrf/xss-vs-csrf
- https://portswigger.net/web-security/csrf/bypassing-token-validation
- https://portswigger.net/web-security/csrf/bypassing-samesite-restrictions
- https://portswigger.net/web-security/csrf/bypassing-referer-based-defenses
- https://portswigger.net/web-security/csrf/preventing


---------------

## 什麼是 CSRF？
跨站請求偽造 (Cross-site request forgery) 是個網路安全漏洞
- 攻擊者誘導使用者執行他們不打算執行的操作
- 它允許攻擊者部分規避 `same origin` policy

 

---------------

## CSRF 攻擊的影響是什麼？
CSRF 攻擊中，攻擊者會使受害使用者無意中執行一個動作
- 例如，改變他們帳號上的電子郵件地址，改變密碼，或進行資金轉移
- 根據行動的性質，攻擊者可能會獲得對使用者帳號的完全控制
- 如果被攻擊的使用者在 app 中擁有特權角色，那麼攻擊者可能能夠完全控制 app 的所有資料和功能


---------------



## CSRF 是如何工作的?
CSRF 攻擊，必須具備三個條件：
- **一個動作**:在 app 中，有個攻擊者有理由誘導的行動
  - 這可能是一個特權行動(如修改其他 user 的權限)或對 user 特定資料的任何行動(如改變 user 自己的密碼)
- **基於 Cookie 的 session 處理**: 執行該動作涉及發出一個或多個 HTTP request
  - app 完全依靠 session cookie 來識別發出 request 的 user。沒有其他機制來跟踪 session 或驗證使用者 request
- **不存在不可預期的 request parameters**: 執行動作的 request 不包含任何攻擊者無法確定或猜測其值的參數
  - 例如，當 user 改變密碼時，如果攻擊者需要知道現有密碼的值，該功能就沒有漏洞



假設 app 包含一個讓 user 改變 email 的 function。當 user 執行時，會發出類似下面的 HTTP request:

```
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE

email=wiener@normal-user.com
```

這符合 CSRF 所需的條件:

- 攻擊者對改變 user email 的行為感興趣。在這個動作之後，攻擊者通常能夠觸發密碼重置並完全控制該 user 的帳號
- 該 app 使用 session cookie 來識別哪個 user 發出了 request。沒有其他 token 或機制來跟踪 user session
- 攻擊者可以很容易地確定執行動作所需的 request 參數的值



有了這些條件，攻擊者可以構建包含以下 HTML 的網頁:

```html
<html>
  <body>
    <form action="https://vulnerable-website.com/email/change" method="POST">
      <input type="hidden" name="email" value="pwned@evil-user.net" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```


如果受害者使用者訪問攻擊者的網頁，將發生以下情況：
- 攻擊者的頁面將觸發一個 HTTP request 到易受攻擊的網站
- 如果 user 登錄了易受攻擊的網站，他們的瀏覽器將自動在 request 中包含他們的 session cookie (假設沒有使用 `SameSite cookies`)
- 受攻擊的網站將以正常方式處理該 request，將其視為由受害使用者發出的 request，並更改其 email 地址

注意
- 儘管 CSRF 通常被描述為與 cookie-based session handling 有關，但它也出現在其他情況下
- 即 app 自動加一些 user auth 到 request 中，如 HTTP HTTP Basic authentication and certificate-based authentication

---------------


 
## 如何構建 CSRF 攻擊
手動建立 CSRF 攻擊所需的 HTML 可能很麻煩，特別是當 request 包含大量參數，或者 request 中存在其他怪異現象時
- 使用 Burp Suite Professional 的 [CSRF PoC generator](https://portswigger.net/burp/documentation/desktop/tools/engagement-tools/generate-csrf-poc)

- 在 Burp Suite Professional 中選擇想測試或利用的 request
- 從右鍵選單中，選擇 `Engagement tools / Generate CSRF PoC`
- Burp Suite 將產生一些觸發所選 request 的 HTML(除去 cookie，這將由受害者的 browser 自動加上)
- 你可以調整 `CSRF PoC generator` 的各種選項，對攻擊的各個方面進行微調
  - 可能需要在一些不尋常的情況下這樣做，以處理 request 的古怪特徵
- 將產生的 HTML 複製到一個網頁中，在一個登錄到易受攻擊的網站的 browser 中查看，並測試是否成功發出了預期的請求並發生了預期的動作




---------------


## 如何達成 CSRF 漏洞
CSRF 攻擊的傳遞機制與 reflected XSS 基本相同
- 攻擊者將惡意 HTML 放到他們控制的網站上，然後誘使受害者訪問該網站
- 可能是通過 email 或 socil media 向 user 提供網站連結
- 或者，攻擊被放在受歡迎的網站上(例如，在使用者評論中)，等待使用者訪問該網站



注意
- 一些簡單的 CSRF 攻擊用 GET 方法，並且可以通過易受攻擊網站上的一個 URL 而完全自成一體
- 在這情況下，攻擊者可能不需要使用外部網站，而可以直接向受害者提供易受攻擊 domain 的惡意 URL

在前面的例子中，如果更改 email 地址的 request 可以用 GET 執行，那麼一個自足的攻擊就會是這樣的
```html
<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">
```

---------------

## 針對 CSRF 的常見防禦措施
現在，成功發現和利用 CSRF 漏洞往往需要繞過目標網站、受害者的 browser 或兩者部署的反 CSRF 措施  

最常見的防禦措施如下:  

- CSRF token
  - CSRF token 是個獨特的、秘密的、不可預測的值，由 server side 產生並與 client side 共享
  - 當試圖執行敏感的動作時，如 submit form，client side 必須在 request 中包含正確的 CSRF token
  - 這使得攻擊者很難代表受害者構建一個有效的 request
- SameSite cookies
  - SameSite 是 browser 安全機制，它決定了網站的 cookies 何時被包含在來自其他網站的 request 中
  - 由於執行敏感 request 通常需要經過驗證的 session cookie，適當的 SameSite 限制可以防止攻擊者跨網站觸發這些操作
  - 2021 年以來，Chrome 默認 Lax SameSite 限制。由於這提議的標準，預計其他主要 browser 在未來會採用這種行為
- 基於 `Referer` 的驗證
  - 一些 app 利用 HTTP Referer header 試圖防禦 CSRF 攻擊，通常是通過驗證 request 是否來自 app 自己的 domain
  - 這通常不如 CSRF token 有效。


 

---------------



## XSS 和 CSRF 的區別是什麼？
- XSS 允許攻擊者在受害使用者的 browser 中執行任意的 JavaScript
- CSRF 允許攻擊者誘導受害使用者執行他們不打算執行的操作

XSS 的後果通常比 CSRF 更嚴重:
- CSRF 通常只適用於 user 能夠執行的行動的一個子集
  - 許多 app 總體上實現了 CSRF 防禦，但忽略了一兩個被暴露的操作
  - 相反，一個成功的 XSS 通常可以誘導 user 執行任何動作
- CSRF 可以說是一個**單向**的漏洞，因為雖然攻擊者可以誘導受害者發出 HTTP request，但他們不能從該 request 中獲取 response
  - 相反，XSS 是**雙向**的，因為攻擊者 inject 的 script 可以發出任意 request，讀取 response，並將資料 leak 到攻擊者選擇的外部 domain


---------------


## CSRF token 可以防止 XSS 嗎？
有些 XSS 確實可以通過有效使用 CSRF token 來防止

考慮一個 reflected XSS，可以像這樣利用：
```
https://insecure-website.com/status?message=<script>/*+Bad+stuff+here...+*/</script>
```


現在，假設這個易受攻擊的 function 包括一個 CSRF token:

```
https://insecure-website.com/status?csrf-token=CIwNZNlR4XbisJF39I8yWnWX9wX4WFoz&message=<script>/*+Bad+stuff+here...+*/</script>
```

假設 server 正確驗證 CSRF token，拒絕沒有有效 token 的 request
- 那該 token 確實可以防止對 XSS 漏洞的利用
- 這裡涉及到跨站 request。通過防止攻擊者 CSRF，app 可以防止對 XSS 的小規模利用

這裡有些重點：
- 如果一個 reflected XSS 漏洞存在於網站的其他地方，在沒有 CSRF token 保護的功能中，那這個 XSS 可以被利用
- 如果一個可利用的 XSS 漏洞存在於網站的任何地方，那該漏洞可以被利用，即使這些操作本身是受 CSRF token 保護的
  - 在這情況下，攻擊者的 script 可以 request 相關頁面獲得有效的 CSRF token，然後用該 token 來執行受保護的操作
- CSRF token 不能保護 stored XSS 漏洞。如果一個受 CSRF token 保護的頁面也是一個 stored XSS 漏洞的輸出點，那該 XSS 可以以通常的方式被利用，當使用者訪問該頁面時，XSS payload 將執行


與 client side 共享 CSRF token 常見方法是作為隱藏參數包含在 HTML form 中，如:

```html
<form name="change-email-form" action="/my-account/change-email" method="POST">
  <label>Email</label>
  <input required type="email" name="email" value="example@normal-website.com">
  <input required type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">
  <button class='button' type='submit'> Update email </button>
</form>

```

送出會像這樣
```
POST /my-account/change-email HTTP/1.1
Host: normal-website.com
Content-Length: 70
Content-Type: application/x-www-form-urlencoded

csrf=50FaWgdOhi9M9wyna8taR1k3ODOR8d6u&email=example@normal-website.com
```

正確實作 CSRF token 有助於防止 CSRF 攻擊
- 使攻擊者難以代表受害者構建有效的 request
- 由於攻擊者沒有辦法預測 CSRF token 的正確值，他們將無法在惡意 request 中包含它

注意
- CSRF token 不一定要作為 POST request 的隱藏參數發送
- 例如，一些 app 將 token 在 HTTP header 中。token 的傳輸方式對整個機制的安全性有很大影響




---------------

## 常見 CSRF token 驗證的缺陷
CSRF 漏洞通常是由於 CSRF token 驗證有缺陷而產生的。下面將介紹常見的問題，使攻擊者能夠繞過這些防禦


---------------

## CSRF token 的驗證取決於 request 方法
一些 app 在 request 使用 POST 方法時正確驗證 token，但在使用 GET 方法時跳過驗證
- 在這種情況下，攻擊者可以切換到 GET 來繞過驗證，並提供 CSRF 攻擊：

```
GET /email/change?email=pwned@evil-user.net HTTP/1.1
Host: vulnerable-website.com
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm
```

---------------


## CSRF token 的驗證取決於 token 的存在
一些 app 在 token 存在的情況下會正確驗證令牌，但如果 token 被省略，則會跳過驗證。

在這種情況下，攻擊者可以刪除包含 token 的整個參數(而不僅僅是它的值）來繞過驗證

```
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 25
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm

email=pwned@evil-user.net
```

---------------

## CSRF token 不與 user session 綁定
一些 app 不驗證 token 是否與提出 request 的 user 屬於同一個 sesseion
- 相反，app 維護一個它所發出的 global token pool，並接受出現在該 pool 中的任何 token
- 在這種情況，攻擊者可以使用他們自己的帳號登錄 app，獲得有效的 token，然後在 CSRF 攻擊中把該 token 提供給受害使用者



---------------

## CSRF token 與  non-session cookie 綁定在一起
上一個漏洞的變種，一些 app 確實將 token 與 cookie 綁定，但不是與用於跟踪 session 的同一個 cookie 綁定
- 當 app 採用兩個不同的框架，一個用於 session 處理，一個用於 CSRF 保護，而這兩個框架沒有整合在一起時，就很容易發生這種情況：

```
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68
Cookie: session=pSJYSScWKpmC60LpFOAHKixuFuM4uXWF; csrfKey=rZHCnSzEp8dbI6atzagGoSYyqJqTz5dv

csrf=RhV7yQDO0xcq9gLEah2WVbmuFqyOq7tY&email=wiener@normal-user.com
```

這種情況更難被利用，但仍然是脆弱的
- 如果網站包含任何允許攻擊者在受害者的 browser 中設置 cookie 的行為，那麼攻擊就有可能發生
- 攻擊者可以用自己的帳號登錄 app，獲得有效 token 和相關的 cookie，利用 cookie 設置行為將他們的 cookie 放入受害者的 browser，並在CSRF 攻擊中向受害者提供他們的 token

注意
- 設置 cookie 的行為甚至不需要存在於與 CSRF 漏洞相同的 web app 中
- 如果被控制的 cookie 有合適的範圍，在同一整體 DNS domain 中的任何其他 app 都有可能被利用來設置被攻擊的 app 中的 cookie
- 例如，`staging.demo.normal-website.com` 上的 cookie 設置功能可以被利用來給 `secure.normal-website.com` 的 cookie


---------------
 

## CSRF token 只是在 cookie 中單純的重複使用
一些 app 不維護任何已發出的 token 的 server side 記錄，而是在 cookie 和 request 參數中重複每個 token
- 當 request 被驗證時，app 只是驗證在參數中的 token 與在 cookie 中的值相 match
- 這有時被稱為對 CSRF 的 `double submit` 防禦，並被提倡，因為它很容易實現，並避免對任何 server side 狀態的需要：

```
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68
Cookie: session=1DQGdzYbOJQzLP7460tfyiv3do7MjyPw; csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa

csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa&email=wiener@normal-user.com
```

在這情況，如果網站包含任何 cookie 設置功能，攻擊者可以再次進行 CSRF 攻擊
- 在這裡，攻擊者不需要獲得他們自己的有效 token。他們只需發明一個 token (也許是所需的格式，如果這被檢查的話)
- 利用 cookie 設置行為將他們的 cookie 放入受害者的 browser，並在 CSRF 攻擊中將他們的 token 提供給受害者


---------------

## 繞過 SameSite cookie 限制


自 2021 年以來，Chrome 默認 `Lax` SameSite 限制
- 這是一個擬議的標準
- 希望其他主要 browser 在未來也能採用這種行為
- 因此，為了徹底測試 cross-site attack vectors，必須牢固掌握這些限制是如何工作的，以及它們可能被繞過的情況

下面介紹 SameSite 是如何工作的，並澄清一些相關術語
- 將看一些最常見的方法，可能會繞過這些限制



---------------


## 在 SameSite cookies 的背景下，什麼是 `site` ?
在 SameSite cookie 限制下，`site` 被定義為 `top-level domain` (TLD)
- 通常像 `.com` 或 `.net`，加上 domain 的一個附加級別。這通常被稱為 `TLD+1`


當確定一個 request 是否是 `same-site` 時，URL scheme 也被考慮在內
- 從 `http://app.example.com` 到 `https://app.example.com` 的連結被大多數瀏覽器視為 cross-site

![](https://portswigger.net/web-security/csrf/images/site-definition.png)  


另外
- `"effective top-level domain" (eTLD)` just a way of accounting for the reserved multipart suffixes that are treated as top-level domains in practice, such as `.co.uk`.
 

---------------

##  site 和 origin 之間有什麼區別？
區別在於它們的範圍
- 一個 site 包括多個 domain
- origin 只包括一個 domain
- 重要的是不要交替使用這兩個術語，因為將兩者混為一談會產生嚴重的安全問題


如果兩個 URL 共享完全相同的 scheme、domain 和 port，它們就被認為具有相同的 origin
- 不過請注意，port 通常是從 scheme 中推斷出來的。

![](https://portswigger.net/web-security/csrf/images/site-vs-origin.png)  

從例子中可以看出，`site` 這個詞沒有那麼具體，因為它只考慮 scheme 和 domain 的最後部分
- 最重要的是，這意味著一個 `cross-origin` request 仍然可以是 `same-site` 的，而不是相反的

| Request from | Request to | Same-site? | Same-origin? |
| :--- | :--- | :--- | :--- |
| https://example.com | https://example.com | Yes | Yes |
| https://app.example.com | https://intranet.example.com | Yes | No: mismatched domain name |
| https://example.com | https://example.com:8080 | Yes | No: mismatched port |
| https://example.com | https://example.co.uk | No: mismatched eTLD | No: mismatched domain name |
| https://example.com | http://example.com | No: mismatched scheme | No: mismatched scheme |


這是重要的區別
- 因為它意味著任何能夠執行任意 JavaScript 的漏洞都可以被濫用，以繞過屬於  same site 的其他 doamin 上的防禦


---------------

## SameSite 是如何工作的？
SameSite 是使 browser 和網站所有者能夠限制哪些 cross-site requests
- 應包括特定的 cookies
- 有助於減少 CSRF 攻擊，CSRF 攻擊會誘使受害者的 browser 發出 request，從而在易受攻擊的網站上觸發有害行為
- 由於這些 request 通常需要受害者的 auth sessione 相關的 cookie，如果不包括這個 cookie，攻擊就會失敗


所有主要瀏覽器目前都支持以下 SameSite：
- Strict
- Lax
- None

開發人員可以為每個 cookie 手動配置級別，只需在 `"Set-Cookie"` response header 中包含 `"SameSite"`:  
```
Set-Cookie: session=0F8tgdOhi9ynR1M9wa3ODa; SameSite=Strict
```

雖然這對 CSRF 攻擊提供了一些保護，但這些限制都不能保證免疫

---------------


## `Strict`
如果 cookie 設為 `SameSite=Strict`
- browser 將不會在任何 cross-site requests 中發送它
- 簡單地說，如果 request 的目標站點與 browser 地址中不一致，它將不包括該 cookie
- 在設置能夠使攜帶者修改資料或執行其他敏感行動的 cookie 時，如訪問只有經過認證的使用者才能使用的特定頁面，建議採用這種方法
- 雖然這是最安全的選擇，但在希望有 cross-site 功能的情況下，它可能會對使用者體驗產生負面影響


---------------
 
## `Lax`
`Lax` SameSite 限制意味著 browser 將在 cross-site requests 中發送 cookie，但只有在滿足以下兩個條件的情況下：
- request 使用 `GET`
- request 是 user 的 top-level navigation 導致的，例如點擊一個連結。

這意味著 cookie 不包括在跨網站的 `POST` 請求中
- 由於 POST 通常用於執行修改資料或狀態的操作，它們更有可能為 CSRF 攻擊的目標
- 同樣地，cookie 也不包括在 background request，例如那些由 script, iframe or image和其他資源的引用發起的 request



---------------

## `None`
`"SameSite=None"` 將完全禁用 SameSite 限制
- 因此，browser 將在所有 request 中發送 cookie，即使是那些由完全不相關的第三方網站引發的 request
- 除了 Chrome 之外，如果在設置 cookie 時沒有提供 SameSite，這就是 default 
  - (這點要再確認看看其他 browser，畢竟這篇文章可能很久了)


禁用 SameSite 是有理由的
- 例如當 cookie 旨在從第三方環境中使用，並且不允許持有者訪問任何敏感資料或功能時
  - 追踪 cookie 是個典型的例子

如果你遇到設置為 `"SameSite=None"` 或沒有明確限制的 cookie
- 值得調查它是否有任何用途
- 當 `"Lax-by-default"` 首次被 Chrome 採用時，它的副作用是破壞了許多現有的網路功能
- 一種快速的變通方法，一些網站選擇簡單地禁用對所有 cookie 的 SameSite，包括潛在的敏感 cookie


當設置 `"SameSite=None"` 的 cookie 時
- 網站還必須包括 `"Secure"`，確保 cookie 只通過 HTTPS 發送。否則，browser 將拒絕該 cookie，它將不會被設置

```
Set-Cookie: trackingId=0F8tgdOhi9ynR1M9wa3ODa; SameSite=None; Secure
```

---------------

## 用 GET request 繞過 SameSite Lax 限制
實作中，server 並不總是對它們是否收到一個特定 endpoint 的 GET 或 POST request 很警慎，即使是那些期待著提交 form 的 request
- 如果也對 session cookie 用了 `Lax`，無論是明確的還是由於 browser 默認的，你仍然可以通過從受害者的 browser 引出 GET request 來執行 CSRF 攻擊


只要該 request 涉及 top-level navigation，browser 仍將包括受害者的 sesseion cookie
- 以下是發起這種攻擊的最簡單方法之一

```html
<script>
  document.location = 'https://vulnerable-website.com/account/transfer-payment?recipient=hacker&amount=1000000';
</script>
```

即使普通的 `GET` request 不被允許，一些框架也提供了覆蓋 request 的方式
-  如，Symfony 支持 form 中的 `_method` 參數，在路由方面，它優先於普通方法

<form action="https://vulnerable-website.com/account/transfer-payment" method="POST">
  <input type="hidden" name="_method" value="GET">
  <input type="hidden" name="recipient" value="hacker">
  <input type="hidden" name="amount" value="1000000">
</form>


---------------

## 使用現場小工具繞過 SameSite 限制
如果有個 cookie 被設為 `SameSite=Strict`
- browser 不會在任何 cross-site requests 中包含它。如果能找到個小工具，導致同一站點內的二次 request，你可能能夠繞過這個限制


一個可能的小工具是 client side redirect，它使用攻擊者可控制的 input(如 URL 參數)動態構建 redirection 目標
- 關於一些例子，參考 [DOM-based open redirection](https://portswigger.net/web-security/dom-based/open-redirection)



就 browser 而言，這些 client side redirects 根本就不是真正的 redirect
- 所產生的 request 只是被當作普通的、獨立的 request
- 最重要的是，這是 same-site request，因此，將包括所有與該 site 有關的 cookies，不管有什麼限制
- 如果你能操縱這個小工具，引出惡意的二次 request，可以使你完全繞過任何 SameSite cookie 限制


注意
- server side 的 redirects 不可能進行攻擊
- 在這種情況下，browser 會認識到跟 redirect request 最初來自於 cross-site request，所以仍然會採用適當的 cookie 限制




---------------



## sibling domains 繞過 SameSite 限制
必須記住，即使 request  是 cross-origin 發出的，也可能是 same-site 的


確保徹底檢查所有可用的攻擊面，包括任何同級別的 domain
- 特別是，能夠引起任意二次 request 的漏洞，如 XSS，可以完全破壞 site-based 的防禦，使網站的所有 domain 暴露於 XSS 攻擊



除了經典的 CSRF，如果目標網站支持 [WebSockets](https://portswigger.net/web-security/websockets)
- 這種功能可能會受到 [cross-site WebSocket hijacking (CSWSH)](https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking) 攻擊
- 這本質上只是針對 WebSocket 握手的 CSRF 攻擊
  - [WebSocket vulnerabilities](https://portswigger.net/web-security/websockets) 專題

---------------


## 用新發布的 cookies 繞過 SameSite Lax 限制
帶有 `Lax` SameSite 限制的 cookie 通常不會在任何跨站 POST request 中發送，但也有些例外


如果設置 cookie 時沒有包含 `SameSite`，Chrome 會默認用 `Lax`
- 然而，為了避免破壞單點登錄(SSO)機制，它實際上不會在 top-level POST request 的前 120 秒內執行這些限制
- 因此，有兩分鐘的時間，使用者可能很容易受到 cross-site attacks
- 注意: 這個兩分鐘的窗口並不適用於明確設置 `SameSite=Lax` 的 cookies


試圖將攻擊的時間安排在這個短暫的窗口內是有些不切實際的
- 另方面，如果你能在網站上找到個小工具，使你能夠強迫受害者獲得一個新的 session cookie
- 你就可以在後續的主要攻擊之前先期刷新他們的 cookie
- 例如，完成一個基於 OAuth 的登錄流程可能會導致每次都有一個新的 session
  - 因為 OAuth 服務不一定知道使用者是否還在登錄目標網站


為了觸發 cookie 的刷新，而不需要受害者再次手動登錄，需要使用 top-level navigation
- 這樣可以確保與他們當前[ OAuth](https://portswigger.net/web-security/oauth) session 相關的 cookie 被包括在內
- 這帶來了額外的挑戰，因為你需要將使用者重定向到你的網站，以便你可以發起 CSRF 攻擊


另外，你可以從一個新的標籤頁觸發 cookie 刷新
- 這樣 browser 就不會在你能夠進行最終攻擊之前離開頁面
- 這種方法的小問題是，browser 會阻止彈出式 tab，除非它們是通過手動交互打開的

例如，browser 默認會阻止以下彈出 tab 方式
```js
window.open('https://vulnerable-website.com/login/sso');
```


替代方案，可以把 `onclick` 綁在 window
- 這樣，`window.open()` 只有在 user 點擊頁面某處時才會被呼叫
```js
window.onclick = () => {
  window.open('https://vulnerable-website.com/login/sso');
}
```



---------------

## 繞過 Referer-based CSRF 防禦
除了採用 CSRF token 的防禦外，一些 app 利用 HTTP Referer header 防禦 CSRF 攻擊
- 通常是通過驗證 request 是否來自 app 自己的 domain。這種方法通常不太有效，而且經常被繞過


參考者 "頭
HTTP `Referer` header 是個 request header
- 它包含連結到被 request 資源的網頁的 URL
- 當 user 觸發 HTTP request 時，包括點擊一個連結或提交表單，它通常由 browser 自動添加
- 有各種方法允許連結頁面保留或修改 `Referer` header，這通常是出於隱私原因。



---------------


## Referer 的驗證取決於 header 的存在
一些 app 在 request 中存在 `"Referer"` header時，會驗證該 header，但如果該 header 被省略，則跳過驗證

在這種情況下
- 攻擊者可以製作他們的 CSRF 漏洞
- 使受害使用者的 browser 在結果 request 中放棄 Referer header
- 有各種方法來實現這一點，但最簡單的是在承載 CSRF 攻擊的 HTML 中使用 META tag

```html
<meta name="referrer" content="never">
```

---------------



## Referer 的驗證可以被繞過
有些 app 以天真的方式驗證 `Referer` header
- 這可以被繞過
- 例如，如果 app 驗證了 `Referer` 中的 domain 是以預期值開始的，那麼攻擊者可以將其作為自己 domain 的一個 sub domain


```
http://vulnerable-website.com.attacker-website.com/csrf-attack
```



同樣，如果 app 只是驗證了 `Referer` 包含自己的 domain
- 攻擊者可以在 URL 的其他地方放置所需的值

```
http://attacker-website.com/csrf-attack?vulnerable-website.com
```

注意
- 儘管可以用 Burp 來識別這種行為，但當在 borwser 中測試你的 poc 時，你會發現這種方法不再起作用
- 為了減少敏感資料以這種方式被洩露的風險，許多 borwser 現在默認從`Referer` header 中剝離 query string


你可以通過確保包含你的漏洞的 response 有 `"Referrer-Policy: unsafe-url"` header 的來覆蓋這行為
- (注意在這種情況下 `Referrer` 的拼寫是正確的！)。這確保了完整的 URL 將被發送，包括 query string



---------------

## 如何防止 CSRF： 用 CSRF token
抵禦 CSRF 的最有力的方法是 CSRF token。token 必須滿足以下條件：
- 與一般的 session token 一樣，具有高熵的不可預測性
- 與 user 的 session 相聯繫
- 在每一種情況下，在執行相關動作之前都要嚴格驗證

---------------


## 應該如何產生 CSRF token?
CSRF token 應該包含重要的熵(entropy)
- 並具有強烈的不可預測性，與一般的 session token 具有相同的屬性
- 應該使用加密安全的偽隨機數產生器(CSPRNG)，用它產生時的時間戳加上 static secret 作為 seed


如果你需要在 CSPRNG 的強度之外的進一步保證
- 可以通過將其輸出與一些 user 特定的熵相連接來產生單個 token，並對整個結構 strong hash
- 這對那些試圖根據發給他們的樣本來分析 token 的攻擊者來說是一個額外的障礙



---------------


## 應該如何傳輸 CSRF token ?
CSRF token 應被視為 secret
- 在其整個生命週期內以安全方式處理
- 通常有效的方法是在使用 POST 提交 HTML form 的一個隱藏 input 中。然後 submit 時，token 作為 reqeust 參數包含在內


```html
<input type="hidden" name="csrf-token" value="CIwNZNlR4XbisJF39I8yWnWX9wX4WFoz" />
```

 
為提高安全性，CSRF token 應盡早放在 HTML 中
- 最好是在任何非隱藏的 input 輸入之前，以及在 HTML 中嵌入使用者可控資料的任何位置之前
- 這可以減輕攻擊者利用精心製作的資料來操縱 HTML 並捕獲其部分內容的各種技術


另種方法是將 token 放入 URL query string，這種方法的安全性較低，因為 query string：
- 被記錄在 client side 和 server side 的不同位置
- 有可能在 HTTP Referer header 中被傳送給第三方
- 可以在使用者的瀏覽器中顯示在螢幕上


一些 app 在 request header 中傳 CSRF token
- 這對設法預測或捕獲另一個使用者的 token 的攻擊者來說是種進一步的防禦
- 因為 browser 通常不允許 cross-domain 發送 custome headers
- 然而，這方法限制了 app 使用 XHR 進行 CSRF 保護的 request，並且可能被認為對許多情況過於復雜

CSRF token 不應該在 cookies 中傳輸


---------------

## 如何驗證 CSRF token ?
CSRF token 產生時，它應該被儲在 server side 的 User session 中
- 收到 request 時，server 應該驗證該 request 是否包括與 session 中的相匹配的 token
- 無論請求的 HTTP method 或內容如何，都必須進行驗證



---------------



## 使用嚴格的 `SameSite` cookie 限制
除了實作 CSRF token 外
- 建議在發出的每個 cookie 中明確設置 SameSite 限制。這樣做，可以準確地控制 cookie 將在哪些情況下被使用

即使所有的 browser 最終都採用 `"Lax-by-default"` 政策
- 這也不適合每一個 cookie，而且比 `"Strict"` 更容易被繞過
- 同時，不同瀏覽器之間的不一致性也意味著，只有一部分使用者能從任何 SameSite 保護中受益

理想情況下
- 應該默認使用 `"Strict"`
- 然後只有在有充分理由的情況下才將其降低到 Lax 之下
- 除非你完全了解安全問題，否則不要用 `"SameSite=None"`




---------------

## 警惕 `cross-origin`, `same-site` 攻擊
儘管正確配置的 SameSite 限制對 cross-site 攻擊提供了很好的保護，但必須明白它們對 `cross-origin`, `same-site` 攻擊是完全無能為力的
- 如果可能的話，建議將不安全的內容，如使用者上傳的文件，與任何敏感功能或資料隔離在一個單獨的網站上
- 在測試一個網站時，一定要徹底檢查屬於同一網站的所有可用攻擊面，包括其任何 sibling domains