
# 2023-08-30：筆記 PortSwigger 的 Authentication vulnerabilities.md


Ref:
- https://portswigger.net/web-security/authentication
- https://portswigger.net/web-security/authentication/password-based
- https://portswigger.net/web-security/authentication/multi-factor
- https://portswigger.net/web-security/authentication/other-mechanisms
- https://portswigger.net/web-security/authentication/securing

-------------------

這章節比較偏向概念，沒有太多具體的案例講解。  
但還是有幾題實際的練習題可以練習。
- https://portswigger.net/web-security/all-labs#authentication

-------------------

## 認證漏洞
從概念上講，身份驗證漏洞是最容易理解的問題
- 然而，由於身份驗證與安全之間的明顯關係，可能是最關鍵的問題之一
- 除了可能允許攻擊者直接訪問敏感資料和功能外，它們還暴露了更多的攻擊面，可被進一步利用
- 因此，學習如何識別和利用身份驗證漏洞(包括如何繞過常見的保護措施)是一項基本技能。


下面將介紹網站最常用的身份驗證機制，並討論其中的潛在漏洞
1. 重點介紹不同身份驗證機制的固有漏洞
2. 以及因實施不當而引入的一些典型漏洞
3. 最後，我們將就如何確保你自己的身份驗證機制盡可能強大提供些基本指導


---------------

## 什麼是身份驗證(authentication)？
身份驗證是驗證特定 user 或 client side 身份的過程
- 換句話說，就是要確保他們確實是自己聲稱的那個人
- 強大的身份驗證機制是有效網路安全不可或缺的一面


不同類型的身份驗證可分為三種身份驗證(authentication)因素:

- 你知道的東西，如密碼或安全問題的答案。有時被稱為知識因素(knowledge factors)
- 你擁有的東西，即手機或安全令牌等實物。有時被稱為佔有因素(possession factors)
- 你現在或正在做的事情，例如你的生物特徵或行為模式。有時被稱為固有因素(inherence factors)

身份驗證機制依靠一系列技術來驗證其中一個或多個因素



**身份驗證(authentication)與授權(authorization)有何區別？**
- 身份驗證(authentication)是驗證 user 是否真實身份的過程
- 授權(authorization)則是驗證 user 是否被允許做某事


在網站中
- 身份驗證(authentication)確定的是試圖 username `Carlos123` 訪問網站的人是否真的是建立該帳號的人
- 一旦 `Carlos123` 通過身份驗證，他的權限將決定他是否有權訪問其他 user 的個人資訊或執行刪除其他 user 帳號等操作

---------------

## 認證漏洞是如何產生的？
身份驗證機制大多數漏洞都是通過以下兩種方式之一產生的:

1. 身份驗證機制薄弱，它們無法充分抵禦暴力破解攻擊
2. 實現過程中存在邏輯缺陷或 code 不完善，使攻擊者可以完全繞過身份驗證機制。這有時被稱為 broken authentication


---------------

## 身份驗證漏洞有什麼影響？
影響可能非常嚴重
- 一旦繞過身份驗證或暴力進入其他 user 的帳號，就可以訪問被入侵帳號的所有資料和功能
- 如果能入侵系統管理員等高權限帳號，就可以完全控制整個 application，並有可能訪問內部基礎設施


即使入侵的是低權限帳號
- 攻擊者仍有可能訪問本不該訪問的資料，如商業敏感資訊
- 即使帳號無法訪問任何敏感資料，也可能允許攻擊者訪問更多頁面，從而提供更多攻擊面
  - 某些高嚴重性攻擊不可能從公開訪問的頁面發起，但可能從內部頁面發起。


---------------

## 認證機制中的漏洞
身份驗證系統通常由幾種不同的機制組成，在這些機制中可能會出現漏洞，我們將更仔細地研究以下領域中最常見的漏洞:

- [密碼登錄的漏洞](https://portswigger.net/web-security/authentication/password-based)
- [多重身份驗證的漏洞](https://portswigger.net/web-security/authentication/multi-factor)
- [其他身份驗證機制的漏洞](https://portswigger.net/web-security/authentication/other-mechanisms)


注意，有幾個練習題要求你列舉 username 和暴力破解密碼
- 這邊提供了一份候選 [usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames) 和 [passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)
- (如果你喜歡入侵身份驗證機制，在完成主要身份驗證練習後，可以嘗試看看 OAuth 的練習題)


---------------

##  密碼登錄的漏洞(Vulnerabilities in password-based login)

對於採用密碼登的網站，user 要自己註冊帳號 or 由管理員分配帳號
- 該帳號與一個唯一的 username 和一個 password 相關聯
-  user 輸入該 username 和密碼來驗證自己的身份

在這種情況下
- **只要知道 password 就足以證明 user 的身份**
- 因此，如果攻擊者能夠取得或猜到其他 user 的登錄憑證，網站的安全性就會受到威脅

方法有很多，在下文探討

---------------

## 暴力破解攻擊
攻擊者使用試錯試圖猜測有效的 user credentials
- 這些攻擊通常使用 username 和 password 的 wordlist 自動進行
  - 將過程自動化，特別是使用專用工具，可使攻擊者高速嘗試大量登錄

暴力破解並不總是完全隨機地猜測 username 和 pass
- 通過使用基本邏輯或可公開取得的知識，可以對暴力破解攻擊進行微調，做出更有根據的猜測
- 這大大提高了此類攻擊的效率。依賴密碼登錄作為驗證 user 身份的唯一方法的網站，**如果沒有實施足夠的暴力破解保護，就會非常容易受到攻擊**


**暴力破解 username:**    
如果 username 符合可識別的模式(如 email)，就特別容易被猜中
- 例如，以 `firstname.lastname@somecompany.com` 格式登錄的企業 username 非常常見
- 不過，即使沒有明顯的模式，有時也會使用可預測的 username 建立高權限帳號，如 `admin` 或 `administrator`


在審核過程中
- 檢查網站是否公開披露了潛在的 username
- 例如，你是否能夠在不登錄的情況下訪問 user config ?
- 即使 config 的實際內容是隱藏的，但 config 中使用的名稱有時與登錄 username 相同
- 你還應該檢查 **HTTP response**，看看是否有任何 email 被披露
  - 有時，response 會包含管理員和 IT support 等高權限 user 的 email


**暴力破解密碼:**  
密碼暴力破解，難度根據密碼的強度而有所不同
- 許多網站都採用某種密碼策略，強制建立高熵密碼，從理論上講，這樣密碼更難被暴力破解。這通常包括強制使用以下密碼
  - 最少 characters 數
  - 小寫字母和大寫字母的混合
  - 至少一個特殊 characters

User 通常不會用隨機字組合來做密碼，而是用他們能記住的密碼，例如
- 如果 `mypassword` 不允許，user 可能會嘗試用 `Mypassword1!` 或 `Myp4$$w0rd` 代替


在政策要求 user 定期更改密碼的情況下， user 進行微小的、可預測的更改也是很常見的
- 例如，`Mypassword1!` 變成了 `Mypassword1?` 或 `Mypassword2!`


這種可預測模式的了解意味著暴力破解攻擊往往比簡單地遍歷每種可能的字組合要複雜得多，因此也有效得多

**枚舉 username:**  
- 枚舉 username 是指攻擊者通過觀察網站行為的變化來識別特定 username 是否有效
- 枚舉 username 通常發生在 login page
- 例如，當輸入有效 username 但密碼不正確時，或者當你在註冊一個已被佔用的 username 時
  - 這大大減少了暴力登錄所需的時間和精力，因為攻擊者能夠快速產生有效 username 的 shortlist


在嘗試暴力破解 login page 時，應特別注意以下方面的任何差異:
- **Status codes**:
  - 在暴力攻擊中，絕大多數猜測返回的 HTTP status code 可能都是相同的，因為大多數猜測都是錯誤的
  - 如果猜測返回的 status code 不同，就充分說明 username 是正確的
  - **網站的最佳做法是，無論結果如何，都返回相同的狀態代碼**
- **Error messages**: 
  - 有時，返回的 error message 會有所不同
  - 這取決於 username 和 pass 都不正確還是只有 pass 不正確
  - **網站的最佳做法是在兩種情況下都使用相同的通用 message**
  - 但有時會出現小的輸入錯誤。只要有一個字出錯，兩條資訊就會截然不同，即使該字沒有 render 在畫面上也是如此
- **Response times**: 
  - 如果處理大多數 request 的 respnse time 相似，那任何偏離時間的 request 都表明幕後發生了不同的情況
  - 這也表明所猜測的 username 可能是正確的
  - 例如，網站可能只在 username 有效的情況下才檢查密碼是否正確。這額外步驟可能會導致 response time 略有增加
  - 這可能很微妙，攻擊者可以通過輸入過長的密碼，使網站處理時間明顯延長，從而使這種延遲更加明顯



練習題:
- [Username enumeration via different responses](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses)
- [Username enumeration via subtly different responses](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-subtly-different-responses)
- [Username enumeration via response timing](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing)


---------------

## 有缺陷的暴力破解防禦
防禦暴力破解的核心是盡量降低攻擊者嘗試登錄的速度。防止暴力破解攻擊最常見的兩種方法是
- 如果 user 嘗試登錄失敗次數過多，則鎖定其嘗試訪問的帳號
- 如果 user 連續嘗試登錄的次數過多，則封鎖其 IP 地址


兩種方法都能提供不同程度的保護，但都不是無懈可擊的，尤其是在使用錯誤邏輯的情況下:
- 例如，有時如果登錄失敗次數過多，你的 IP 就會被阻擋
- 在某些實現中，如果 IP 所有者成功登錄，嘗試失敗的 counter 就會重置
- 這意味著攻擊者只需每嘗試幾次登錄自己的帳號，就能避免達到這個限制


**在這種情況下，只要在整個 wordlist 中定期加入自己的登錄，就足以讓這種防禦幾乎毫無用處**  


練習題: [Broken brute-force protection, IP block](https://portswigger.net/web-security/authentication/password-based/lab-broken-bruteforce-protection-ip-block)  



**帳號鎖定:**  
- 防止暴力破解的一種方法是，如果符合某些可疑條件(通常是登錄失敗的次數)，則鎖定帳號
- 與正常登錄錯誤一樣，server 顯示帳號已鎖定的 response 也可幫助攻擊者枚舉 username


練習題: [Username enumeration via account lock](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-account-lock)  

 
鎖定帳號可以在一定程度上防止對特定帳號進行有針對性的暴力破解
- 但是，這種方法無法充分防止暴力攻擊
- 因為在這種攻擊中，攻擊者只是試圖獲得對任意帳號的訪問權

例如，可以使用以下方法繞過這種保護：
1. 建立一個可能有效候選 username list。這可以通過`枚舉 username` 或簡單地根據`常見 username list` 來實現
2. 確定一個很小的密碼候選名單，你認為至少有一個 user 可能擁有這些密碼
    - 最重要的是，所選密碼的數量不能超過允許的登錄次數
    - 例如，如果你計算出登錄限制為 3 次，那麼最多只能猜測 3 次密碼
3. 用 Burp Intruder 等工具，用每個候選 username 嘗試每個選定的密碼
    - 你就可以嘗試暴力破解每個帳號，而不會鎖定帳號
    - 你只需要一個 user 使用三個密碼中的一個，就可以入侵一個帳號


帳號鎖定也無法抵禦憑證填充(credential stuffing)攻擊
- 這涉及使用大量的 `username:password` 配對組合字典
  - 這些字典由真實洩露資料中組成。
  - 帳號鎖定不能防止憑據填充，因為每個 username 只被嘗試使用一次
- 憑證填充(Credential stuffing)尤其危險，有時只需一次自動攻擊，攻擊者就能入侵多個不同帳號

**User rate limiting:**
- 另種方法是限制 user rate limiting
- 在短時間內發出過多登錄請求，就會導致 IP 被封鎖。通常情況下，只能通過以下一種方式解封:
  - 一段時間後自動解封
  - 由管理員手動解除
  -  user 成功完成驗證碼後手動解鎖
- 有時比帳號鎖定更受歡迎，因為它不容易受到枚舉 username 和拒絕服務攻擊
- 不過，它仍不完全安全。攻擊者可以通過多種方式操縱其表面 IP 以繞過


由於限制是基於從 user IP 發送的 HTTP request 請求的速率
- 因此，如果你能想出用一個 request 猜出多個密碼的方法，有時也能繞過這一防禦

練習題: [Broken brute-force protection, multiple credentials per request](https://portswigger.net/web-security/authentication/password-based/lab-broken-brute-force-protection-multiple-credentials-per-request)


---------------



## HTTP basic authentication
HTTP basic 身份驗證相對簡單且易於實現
- 在 HTTP basic 身份驗證中，clietn 從 server 接收身份驗證 token
- 該 token 由 username 和 pass 連接而成，並以 Base64 encode
- 該 token 由 browser 存儲和管理，browser 會自動將其加到每個後續 request 的 `Authorization` header 中


```
Authorization: Basic base64(username:password)
```


出於多種原因，這種方法通常不被認為是安全的身份驗證方法。首先
- 它在每次 request 時重複發送 user 的登錄憑據
- 除非網站也實施了 `HSTS`，否則 user 憑據就有可能被中間人攻擊捕獲
  - (強制使用 HTTPS)
- 此外，HTTP basic authentication 通常無法防禦暴力攻擊。token 完全是 static，因此很容易被暴力破解
- HTTP basic authentication 還特別容易受到 session 相關的漏洞攻擊，尤其是 CSRF，因為它本身不提供任何保護


在某些情況下，利用脆弱的 HTTP basic authentication 可能只會讓攻擊者訪問一個看似無趣的頁面
- 然而，除了提供更多攻擊面外，以這種方式暴露的 confidential 還可能在其他更機密的情況下被重複使用


---------------

## 多重因素(multi-factor)身份驗證中的漏洞
下面介紹  multi-factor 身份驗證機制中可能出現的漏洞和練習題
- 有些網站要求 user 使用多個身份驗證 factor 來證明自己的身份




對大多網站來說，驗證生物識別是不切實際的
- 不過，基於「你所知道的」和「你所擁有的」的強制性和可選的雙因素身份驗證(2FA)越來越常見
- 這通常要求 user 輸入傳統密碼和臨時驗證碼，驗證碼來自 user 所持有的帶外物理設備


攻擊者有可能取得密碼，但同時從 out-of-band source 取得另一個因素的可能性要小得多
- 因此，2FA 證顯然比 single-factor 更安全
- 不過，2FA 安全性取決於其實施情況。實施不力也會被破解，甚至完全被繞過



還值得注意的是，只有通過驗證多個不同的 factor，才能充分發揮  multi-factor  身份驗證的優勢
- 以兩種不同方式驗證同一 factor 並不是真正的雙因素身份驗證
- 基於 email 的 2FA 就是這樣一個例子
- 雖然 user 必須提供密碼和驗證碼，但取得驗證碼只依賴於 user 知道其 email 的登錄憑證。因此，知識驗證因素只是經過了兩次驗證


---------------

## 2FA tokens
- 驗證碼通常由 user 通過某設備讀取。現在，許多高安全性網站都會為此目提供專用設備，如 RSA token
- 除了為安全而設計外，這些專用設備還具有直接產生驗證碼的優勢
- 出於同樣的原因，網站使用專用 app (如 Google Authenticator) 也很常見




另方面，有些網站會將驗證碼以簡訊發給 user
- 從技術上講，這仍然是在驗證「你擁有的東西」這一 factor，但卻容易被濫用
- 首先，驗證碼是通過簡訊發送的，而不是由設備本身產生的，這就造成了 code 被截獲的可能性
- 此外還存在 SIM 卡互(swapping)換的風險，攻擊者通過欺詐獲得一張帶有受害者電話號碼的 SIM 卡
  - 這樣，攻擊者就會收到發送給受害者的所有簡訊




---------------

## 繞過 2FA
有時，2FA 實作存在缺陷，以至於可以完全繞過

如果首先提示 user 輸入密碼，然後在另一個頁面上提示 user 輸入驗證碼
- 那麼 user 在輸入驗證碼之前實際上已經處於 logged in 狀態
- 在這種情況下，值得測試能否在完成第一驗證步驟後直接跳轉到 logged-in only 頁面
- 有時會發現網站其實並不檢查是否完成了第二步


練習題: [2FA simple bypass](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass)


---------------

## 有缺陷的 2FA 驗證邏輯
有時邏輯存在缺陷，user 完成初始登錄步驟後，網站沒有充分驗證同一 user 是否在完成第二步驟

例如， user 在第一步中使用正常憑據登錄，如下所示:

```
POST /login-steps/first HTTP/1.1
Host: vulnerable-website.com
...
username=carlos&password=qwerty
```


然後，在進入登錄流程的第二步之前，會為他們分配一個與其帳號相關的 cookie:


```
HTTP/1.1 200 OK
Set-Cookie: account=carlos

GET /login-steps/second HTTP/1.1
Cookie: account=carlos
```

送出驗證碼時，request 會使用此 cookie 來確定 user 試圖訪問的帳號:
```
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=carlos
...
verification-code=123456

```

在這種情況下，攻擊者可以使用自己的憑據登錄，但在送出驗證碼時會將 `account` cookie 的值更改為任意 `username`
```
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=victim-user
...
verification-code=123456
```

 
如果攻擊者能夠暴力破解驗證碼，這是極其危險的
- 這將允許他們完全根據 username 登錄任意帳號。甚至不需要知道 user 的密碼


練習題: [2FA broken logic](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-broken-logic)  


---------------

 
## 暴力破解 2FA 驗證碼
與密碼一樣，也需要防禦暴力破解 2FA 驗證碼
- 這非常重要，因為驗證碼通常是 4 或 6 位數字。如果沒有足夠的保護，破解是輕而易舉的

一些網站試圖在 user 輸入一定數量的錯誤驗證碼後自動註銷來防止這種情況
- **這是無效的**
- 攻擊者可以通過為 `Burp Intruder` [creating macros](https://portswigger.net/burp/documentation/desktop/settings/sessions#macros) 來自動執行這一多步驟過程
  - `Turbo Intruder` extension 也可用於此目的

 
練習題: [2FA bypass using a brute-force attack](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-bypass-using-a-brute-force-attack)  


---------------

 

## 其他身份驗證機制中的漏洞
下面介紹與身份驗證相關的輔助功能，說明這些功能如何可能存在漏洞，還有練習題  


除了基本的登錄功能外，大多數網站還提供輔助功能，允許 user 管理自己的帳號
- 例如，user 通常可以更改密碼 or 重置密碼
- 這些機制也會引入攻擊者可以利用的漏洞
- 你需要採取類似措施，確保相關功能同樣強大
- 這一點在攻擊者能夠建立自己的帳號並因此能夠輕鬆訪問研究這些附加頁面的情況下尤為重要

---------------

## 保持 user 登錄狀態
一個常見的功能是在關閉 browser session 後仍能保持登錄狀態
- 通常是有 `Remember me` 或 `Keep me logged in` 等字樣


這種功能通常是 `remember me` flag 存在持久的 Cookie 中
- **由於擁有這個 cookie 就能有效繞過整個登錄過程，因此最好的做法是讓人無法猜到這個 cookie**
- 不過，有些網站會根據可預測的靜態值(如 username 和 timestamp)的連接產生 Cookie
- 有些網站甚至將密碼作為 cookie 的一部分。如果攻擊者能夠建立自己的帳號，那麼這種方法就特別危險，因為他們可以研究自己的 cookie，並可能推斷出 cookie 是如何產生的
  - 一旦推算出公式，就可以嘗試暴力破解其他 user 的 cookie，從而訪問他們的帳號


有些網站認為，如果 cookie 以某種方式加密，即使使用靜態值也無法猜測
- 如果方法正確，這可能是對的，但使用簡單的 two-way encoding (如 Base64)並不能提供任何保護
- 即使使用適當的加密和 one-way hash function 也不是完全無懈可擊的
- 如果攻擊者能輕易識別 hashing algorithm，並且沒有使用 salt，就有可能通過 wordlist 來暴力破解 cookie


如果沒有對 cookie 猜測應用類似的限制，這種方法可用於繞過登錄嘗試限制  

練習題: [Brute-forcing a stay-logged-in cookie](https://portswigger.net/web-security/authentication/other-mechanisms/lab-brute-forcing-a-stay-logged-in-cookie)  



即使攻擊者無法建立自己的帳號，可以利用這個漏洞
- 他們仍然使用 XSS 等常用技術竊取其他 user 的 `remember me` cookie，並從中推斷出 cookie 的構造
- 如果網站是使用 open source 構建的，cookie 構建關鍵細節甚至可能是公開文件


在一些罕見的情況下
- 從 cookie 中取得 user 的明文實際密碼是有可能的，即使它是 hash 的
- 網上有知名 password list 的 hash 版本，因此如果 user 的密碼出現在這些列表中，密碼像輸入搜索引擎一樣簡單
  - 這說明了 salt 在有效加密中的重要性
 

練習題: [Offline password cracking](https://portswigger.net/web-security/authentication/other-mechanisms/lab-offline-password-cracking)


---------------


## 重置 user 密碼
有些 user 通常需要重置密碼的方法
- 在這情況，基於密碼的身份驗證是不可能的，因此必須依靠其他方法來確保真正的 user 正在重置自己的密碼
- 因此，密碼重置功能本質上是危險的，需要安全地實作

這種功能通常有幾種不同的實現方式，漏洞程度也各不相同:
- **通過 email 發送密碼**
  - **如果網站要安全地處理密碼，就絕對不能向 user 發送當前密碼**
  - 相反，有些網站會產生新密碼，並通過 email 發送給 user
  - 一般來說，應避免通過不安全的渠道發送永久密碼
  - 在這種情況下，安全性依賴於產生的密碼在很短的時間內失效，或 user 立即重新更改密碼。否則，這種方法極易受到中間人攻擊
  - email 並非真正為安全存儲機密而設計，因此一般不認為 email 是安全的
    - 許多 user 還會通過不安全的渠道在多個設備之間自動同步收件箱
- **使用 URL 重置密碼**  
  - 一種穩健的重置密碼方法是向 user 發送一個唯一的 URL，將 user 帶入密碼重置頁面
  - 這種方法的安全性較低，使用帶有易猜參數的 URL 來識別正在重置的帳號

例如
- 攻擊者可以更改 user 參數，指向他們所確定的任何 username
- 然後，攻擊者就直接進入頁面，為任意 user 設置新密碼
```
http://vulnerable-website.com/reset-password?user=victim-user
```


更好方法是產生高熵的難以猜測的 token，並根據該令牌建立重置 URL
- 在最好的情況下，**該 URL 不應提示重置的是哪個 user 的密碼**
- 當 user 訪問 URL 時，backend 檢查是否存在此 token
  - 如果存在，則應檢查要重置哪個 user 的密碼
- token 應在短時間後過期，並在密碼重置後立即銷毀


```
http://vulnerable-website.com/reset-password?token=a0ba0d1cb3b63d13822572fcff1a241895d893f659164d4cc550b421ebdd48a8
```

**但是，有些網站在 submit reset 時沒有再次驗證 token**
- 在這情況，攻擊者可以通過自己的帳號訪問 reset form，刪除 token，然後利用該頁面重置任意 user 的密碼


練習題: [Password reset broken logic](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic)  



如果重置 email 中的 URL 是動態產生的，**這也可能會導緻密碼重置中毒**
- 攻擊者有可能竊取其他 user 的 token，並用它來更改自己的密碼


練習題: [Password reset poisoning via middleware](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-poisoning-via-middleware)  


---------------


## 更改 user 密碼
通常情況下，更改密碼需要**輸入兩次當前密碼和新密碼**
- 從根本上依賴於與普通登錄頁面相同的過程來檢查 username 和當前密碼是否匹配
- 因此，這些頁面很容易受到相同技術的攻擊

如果密碼更改功能允許攻擊者在不以受害 user 身份登錄的情況下直接訪問，那麼它就會特別危險
- 例如，如果 username 是在一個隱藏 field 中提供的，攻擊者就有可能編輯該值，從而攻擊任意 user
- 攻擊者可利用漏洞枚舉 username 及強制輸入密碼




練習題: [Password brute-force via password change](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-brute-force-via-password-change)  


---------------

## 如何確保 authentication mechanisms 的安全
要概述保護網站的所有可能措施顯然是不可能的。不過，有幾項一般原則應始終遵循:


## 謹慎使用 user 憑證
如果你無意中向攻擊者洩露了一套有效的登錄憑據
- 那麼即使是最強大的身份驗證機制也會失效
- 絕對不通過未加密連接發送任何登錄資料
  - 確保任何嘗試的 HTTP request redirect 到 HTTPS
- 還應該對網站進行審計，以確保 username 或 email 不會通過公開訪問的配置文件或 HTTP response 等方式洩露


---------------

### 不要依賴 user 的安全性
**嚴格的身份驗證措施通常需要 user 付出額外的努力**
- 人性使然，user 難免會想方設法讓自己省力。**因此，你需要盡可能地強制執行安全行為**


最明顯的例子就是實施有效的 password policy
- 一些傳統的 policy 以失敗，是因為人們將自己可預測的密碼強加到 policy 中。相反
- 實施某種簡單的密碼檢查器會更有效，它可以讓 user 嘗試使用密碼，並實時反饋密碼的強度
- 一個流行的例子是由 Dropbox 開發的 JavaScript 庫 `zxcvbn`
  - 通過只允許使用密碼檢查器評級較高的密碼，可以比傳統策略更有效地強制使用安全密碼
  - https://github.com/dropbox/zxcvbn



---------------

## 防止枚舉 username 
如果暴露系統中存在的 user，攻擊者就更容易破解身份驗證機制
- 由於網站的性質，在某些情況下，知道某個人擁有帳號本身就是敏感資訊
- 無論嘗試使用的 username 是否有效，**重要的是要使用相同的通用 error messages**
- 並**確保它們確實是相同的。每次登錄 request 都應返回相同的 HTTP status code**
- 最後，**盡可能縮短不同情況下的 response time**


---------------

## 實作暴力破解的強力防禦
暴力破解攻擊的構造非常簡單，確保採取措施防止或至少破壞任何暴力破解登錄的嘗試至關重要
- 更有效的方法之一是實施嚴格的、基於 IP 的 user rate limiting
  - 這應包括防止攻擊者操縱其明顯 IP 的措施
  - 理想情況下，在達到限制後，每次嘗試登錄時都應要求 user 完成驗證碼測試

記住，這並不能保證完全消除暴力威脅。但盡可能讓過程變得繁瑣和人工化，會增加攻擊者放棄的可能性，轉而尋找更軟的目標


---------------


## 不要忘記輔助功能
一定不要只關注 login page，而忽略了與身份驗證相關的其他功能
- 在攻擊者可以自由註冊帳號並探索這些功能的情況下，這一點尤為重要
- 記住，密碼重置或更改與 login 一樣，都是有效的攻擊面，因此必須擁有同樣的嚴格防禦




---------------


## 實作適當的  multi-factor 身份驗證
雖然  multi-factor 身份驗證不一定適用於每個網站，但如果得當，它比僅使用密碼登錄要安全得多
- 記住，驗證同一 factor 的多個實例並不是真正的  multi-factor 身份驗證
- 通過 email 發送驗證碼本質上只是一種更冗長的 single factor 身份驗證



從技術上講，SMS-based 2FA 驗證了兩個因素(你知道的和你擁有的)
- 但是，SIM 卡調換(swapping)等濫用的可能性意味著這種系統可能並不可靠
- 理想情況下，2FA 應使用專用設備或 app 直接產生驗證碼。它們是專門為提供安全性而設計的，因此通常更加安全
- 最後，就像主要的身份驗證邏輯一樣，確保 2FA 檢查的邏輯是合理的，不會被輕易繞過。