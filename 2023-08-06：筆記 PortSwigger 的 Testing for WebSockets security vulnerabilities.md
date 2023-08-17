
# 2023-08-06：筆記 PortSwigger 的 Testing for WebSockets security vulnerabilities.md

Ref:
- https://portswigger.net/web-security/websockets#websockets-security-vulnerabilities
- https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking


---------


## 操縱 WebSocket 流量
WebSockets 漏洞通常涉及以 application 意想不到的方式操縱 WebSockets  


可以用 Burp Suite 來:
- 攔截並修改 WebSocket message
- Replay 並產生新的 WebSocket message
- 操縱 WebSocket connection

--------

## 攔截並修改 WebSocket message
Burp 操作方法:

- 打開 Burp browser
- 瀏覽使用 WebSockets 的 application 功能。查看 `Burp Proxy` 的 WebSockets `history` tab 中出現的項目來確定 WebSockets 正在被使用
- 在 `Burp Proxy` 的 `Intercept` tab 中，確保 interception 已打開
- 當 browser 或 server 發送 WebSocket message 時，它會顯示在 Intercept tab 中，供查看或修改。按 `Forward` 按鈕轉發 message

注意
- 可以配 Burp Proxy 是 `client-to-server` 還是 `server-to-client` 的 message 
- 在 `Settings` dialog 的[WebSocket interception rules](https://portswigger.net/burp/documentation/desktop/settings/tools/proxy#websocket-interception-rules) 中設定



--------------


## Replaying 和產生新的 WebSocket message
用 Burp Repeater 可以重放單條 message 並產生新 message:
- `Burp Proxy`，選擇 WebSockets `history` 或 `Intercept` tab 中的 message，然後 context menu 選 `Send to Repeater`
- 在 `Burp Repeater`，可以編輯 message，並重複發送
- 你可以輸入新 message，然後向 client 或 server 任一方向發送
- `Burp Repeater` 的 `History` panel 中，可以查看通過 WebSocket connection 傳輸的 message history
  - 這包括在 `Burp Repeater` 中產生的 message，以及 browser 或 server 通過同一 connection 產生的 message
- 如果想編輯並重新發送 history panel 中的任何 message，選擇 message 並從 context menu 中選 `Edit and resend`

---------------

## 操作 WebSocket connection
除了操作，有時還需要 [WebSocket handshake](https://portswigger.net/web-security/websockets/what-are-websockets#how-are-websocket-connections-established) 來建立 connection

操縱 WebSocket handshake 可能是必要的:
- 它可以接觸到更多的攻擊面
- 某些攻擊可能會中斷 connection，因此需要建立新的 connection 
- 原始 handshake request 中的 token 或其他 data 可能已經過時，需要更新



用 `Burp Repeater` 操作 WebSocket handshake:
- (上面有說明如何在 `Burp Repeater` 發送 WebSocket message)
- 在 Burp Repeater 點擊 WebSocket URL 旁邊的鉛筆 icon。這會打開一個嚮導，讓你附加到現有已連接的 WebSocket、clone 已連接的 WebSocket 或重新連接到斷開的 WebSocket
- 如果選擇 clone 已連接的 WebSocket 或重新連接已斷開連接的 WebSocket，嚮導會顯示 WebSocket handshake request 的詳細資訊，你可以在執行 handshake 之前根據需要進行編輯
- 點 `Connect` 後，Burp 將嘗試執行已配置的 handshake 並顯示結果。如果成功建立新的 WebSocket coneection，你就可以用它在 Burp Repeater 中發送新 message

-----------------

### WebSockets 漏洞
原則上，幾乎所有網路安全漏洞都可能與 WebSockets 有關:
- 傳輸到 server 的 user input 可能會以不安全的方式進行處理，從而導致 SQL injection 或 XML external entity injection
- 只有用 [帶外(out-of-band,OAST)技術]（https://portswigger.net/blog/oast-out-of-band-application-security-testing）才能檢測到通過 WebSockets 到達的某些 blind 漏洞
- 如果攻擊者控制的 data 通過 WebSockets 傳輸給其他 application user，則可能導致 XSS 或其他 client side 漏洞


**操縱 WebSocket message 以利用漏洞:**  

大多基於 input 影響 WebSockets 的漏洞都可以通過篡改 message 內容來發現和利用  

假設一個聊天 app 使用 WebSockets 在 browser 和 server 間發送聊天 message
- 當 user 送出聊天資訊時，會向 server 送類似以下內容的 WebSocket message:

```json
{ "message": "Hello Carlos" }
```

message 會(再次通過 WebSockets)傳輸給另位 user，並在該 user 的 browser 中呈現如下:


```html
<td>Hello Carlos</td>
```

在這情況，如果沒有其他處理或防禦
- 攻擊者可以通過以下 WebSocket message 來驗證 XSS:


```json
{"message":"<img src=1 onerror='alert(1)'>"}
```


**操縱 WebSocket handshake 來利用漏洞:**    
有些漏洞只能通過操縱 WebSocket handshake 來發現和利用。這些漏洞往往涉及設計缺陷:
- 錯誤信任 HTTP header 來執行安全決策，如 `X-Forwarded-For` header
- Session handling 機制的缺陷，因為處理 WebSocket message 的 session context 通常由 handshake message 的 session context 決定
- application 使用的自定義 HTTP header 帶來的攻擊面


**利用 cross-site  WebSockets 漏洞:**   
當攻擊者從其控制的網站建立 cross-site WebSockets connection 時，就會一些漏洞。這被稱為 [cross-site WebSocket hijacking](https://portswigger.net/web-security/websockets/cross-site-websocket-hijacking) 攻擊
- 涉及利用 WebSocket handshake 過程中的 [cross-site request forgery](https://portswigger.net/web-security/csrf) (CSRF) 漏洞
- 這種攻擊通常會造成嚴重影響，使攻擊者可以代表受害 user 執行特權操作，或獲取受害 user 可以訪問的敏感資料



----------

## Cross-site WebSocket 劫持
Cross-site WebSocket hijacking (也稱為 cross-origin WebSocket hijacking)
- 涉及 WebSocket handshake 過程中的 CSRF 漏洞
- **當 handshake request 僅依賴 HTTP cookie 進行 session handling，且不包含任何 CSRF tokens 或其他不可預測值時，就會出現這種漏洞**

攻擊者可以
- 在自己的 domain 上建立惡意網頁，與受攻擊的 application 建立 cross-site WebSocket connection
- application 將在受害 user 與 application session 的 context 處理該 connection
- 然後，攻擊者的網頁可以通過連接向 server 發送任意 message，並讀取返回的 message
- 這意味著，與普通的 CSRF 不同，攻擊者可以獲得與被攻擊 application 的雙向交互



-------


## cross-site WebSocket hijacking 有什麼影響？
成功的劫持攻擊通常會使攻擊者能:

- **偽裝成受害者用戶執行未經授權的操作**:
  - 與普通 CSRF 一樣，攻擊者可以向 server 發送任意 message
  - 如果 application 使用 client side 產生的 WebSocket message 來執行任何敏感操作，那攻擊者就可以 cross side 產生合適的資料並觸發這些操作
- **竊取 user 能存取的敏感資料**:
  - 與普通的 CSRF 不同，hijacking 可讓攻擊者通過被劫持的 WebSocket 與受攻擊的 application 進行雙向交互
   - 如果 application 使用服 server 產生的 WebSocket message 向 user 返回任何敏感資訊，那就可以攔截這些資訊

------------


## 執行 cross-site WebSocket hijacking 攻擊
cross-site WebSocket hijacking 攻擊本質上是 WebSocket handshake 的 CSRF 漏洞
- 因此執行攻擊的第一步就是查看 application 的 WebSocket handshake，並確定它們是否受到 CSRF 防護


就 CSRF 攻擊的正常條件而言，通常需要找到一個 handshake message
- 它完全依賴 HTTP cookie 進行 session handling，並且在 request 參數中不使用任何 token 或其他不可預測的值


如下 WebSocket handshake request 很可能會受到 CSRF 的攻擊
- 因為唯一的 session token 是在 cookie 中傳輸的
- (`Sec-WebSocket-Key` header 包含一個隨機值，用於防止 caching proxies 出錯，不用於身份驗證或 session handling 目的)

```
GET /chat HTTP/1.1
Host: normal-website.com
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w==
Connection: keep-alive, Upgrade
Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2
Upgrade: websocket
```


如果 WebSocket handshake request 在 CSRF 漏洞
- 那攻擊者的網頁就可以執行 cross-site request，在存在漏洞的網站上打開 WebSocket
- 攻擊接下來會發生什麼，完全取決於 application 的邏輯和使用 WebSocket 的方式


攻擊可能包括:
- 發送 WebSocket message，代表受害用戶執行未經授權的操作
- 發送 WebSocket message 以搜尋敏感資料
- 有時，只是等待包含敏感資料的傳入 message