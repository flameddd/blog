## http_header_cache

### Expires (HTTP 1.0)
HTTP Response Header  
> Expires: Wed, 21 Oct 2017 07:28:00 GMT

### Cache-Control 與 max-age (HTTP 1.1)
> Cache-Control: max-age=30

#### RFC2616
> If a response includes both an Expires header and a max-age directive, the max-age directive overrides the Expires header, even if the Expires header is more restrictive

> `max-age` overrides `Expires`

上面這兩個 Header 都是在關注一個 Response 的「新鮮度(freshness)」過期了、不新鮮了，就發送 Request 去跟 Server 拿新的資料。  
> 但是這邊要特別注意一點：「過期了不代表不能用」

如果還可以用，那 Server 就不必返回新content，只要跟瀏覽器說：「你快取的圖片可以繼續用一年喔」就可以了。

### Last-Modified 與 If-Modified-Since
想要做到上面的功能，必須要 `Server` 跟 `Client` 兩邊互相配合才行。  
其中一種做法就是使用 HTTP 1.0 就有的：Last-Modified與If-Modified-Since的搭配使用。

#### Last-Modified 
response header
> Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT
#### If-Modified-Since
request header
> If-Modified-Since: Wed, 21 Oct 2015 07:28:00 GMT

這樣要求，若這時間點都沒有被改變的話，server 就 return
> http return status code 304

但這招，只要檔案有被重新打開存擋就 update time 了，所以用下面這招

### Etag 與 If-None-Match
#### Etag
response header
> ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
> ETag: W/"0815"


Etag跟If-None-Match也是搭配使用的一對，就像Last-Modified跟If-Modified-Since一樣。

Server 在回傳 Response 的時候帶上Etag表示這個檔案獨有的 hash，快取過期後瀏覽器發送If-None-Match詢問 Server 是否有新的資料（不符合這個Etag的資料），有的話就回傳新的，沒有的話就只要回傳 304 就好了。


例如，當編輯MDN時，當前的wiki內容被散列，並在響應中放入Etag：
> ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4
將更改保存到Wiki頁面（發布數據）時，`POST`請求將包含有`ETag值的If-Match`頭來檢查是否為最新版本。
> If-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
如果哈希值不匹配，則意味著文檔已經被編輯，拋出`412`前提條件失敗錯誤。

服務器將客戶端的ETag（作為If-None-Match字段的值一起發送）與其當前版本的資源的ETag進行比較，如果兩個值匹配（即資源未更改），服務器將返回不帶任何內容的`304`未修改狀態，告訴客戶端緩存版本可用（新鮮）。  

### 412 Precondition Failed
在 HTTP 協議中，響應狀態碼 412 Precondition Failed（先決條件失敗）表示 client 端錯誤，意味著對於目標資源的訪問請求被拒絕。這通常發生於采用除 GET 和 HEAD 之外的方法進行條件請求時，由首部字段 If-Unmodified-Since 或 If-None-Match 規定的先決條件不成立的情況下。這時候，請求的操作——通常是上傳或修改文件——無法執行，從而返回該錯誤狀態碼。

### 快取檢查表
google 文件教學在定義快取策略時，請記住下列技巧和方法：

1. 使用一致的網址：如果您在不同的網址上提供相同的內容，將會多次取得及儲存該內容。提示：**請注意網址區分大小寫**！
2. 確認伺服器提供驗證權杖 (ETag)：透過驗證權杖，如果伺服器上的資源未曾變更，就不必傳輸相同的位元組。
3. 確定中繼快取可以快取哪些資源：對所有使用者的回應完全相同的資源很適合由 CDN 或其他中繼快取進行快取。
4. 確定每個資源的最佳快取效期：不同的資源可能有不同的更新要求。審查並確定每個資源適合的 max-age。
5. 確定網站的最佳快取階層：對 HTML 文件組合使用包含內容指紋碼的資源網址以及短時間或 no-cache 的效期，可以控制用戶端取得更新的速度。
6. 更新內容最小化：有些資源的更新頻率比其他資源高。如果資源的特定部分 (例如 JavaScript 函式或一組 CSS 樣式) 會經常更新，請考慮將其程式碼當做單獨的檔案提供。如此一來，每次擷取更新時，剩餘內容 (例如不會頻繁更新的程式庫程式碼) 可以從快取中擷取，讓需要下載的內容量降到最低。
