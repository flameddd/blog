## 和 CORS 跟 cookie 打交道
### 愷開 2019 Jan 18
#### https://medium.com/d-d-mag/%E5%92%8C-cors-%E8%B7%9F-cookie-%E6%89%93%E4%BA%A4%E9%81%93-dd420ccc7399


`Access-Control-Allow-Origin: *` 開好開滿就對了？

### 同源政策（same-origin policy）
同源政策規定
- 某些特定的資源
- 程式碼

必須在同源的情況下才可以存取。

#### 什麼是同源呢？
一份 document 的來源，由
- protocol
- host
- port

來定義
- http://kalan.com
- https://kalan.com 

就不算是同源。
subdomain 
- https://api.foobar.com 
- https://app.foobar.com

因為 host 不同，所以也不算

有些資源是能夠透過跨來源取得的
- `<img />`
- `<video />`, `<audio />`
- `<iframe />`：可以透過定義 header 來防止他人嵌入
- 透過 `<link rel="stylesheet" href />` 載入的 CSS 腳本
- `<script src="" />` 載入的 Javascript

透過程式碼發出的`跨來源請求`則會`受到同源政策的限制`如
- Fetch
- XHR

如果都要限制在同源政策下的話，前後端開發會非常難以進行  
也沒辦法用 XHR 的方式套用其他 SDK 的 API  
因此出現了 CORS（Cross-Origin Resource Sharing）的機制  

### CORS（Cross-Origin Resource Sharing）
CORS 通常需要後端設定相關的 header，並且了解背後所具備的含義才有辦法正確運作  


CORS 是由**兩個 Header** 來做相對的存取控制
- Request 當中的 Origin 
- Response 中的 Access-Control-Allow-Origin。

當
- Request 的 Origin 和
- Response 的 header 中 Access-Control-Allow-Origin
的值相同
- 或是Access-Control-Allow-Origin: * （代表允許任何網域存取資源）

此時就會放寬 CORS 的限制，允許存取跨域資源  
如果不符合 CORS policy 的話，會顯示警告錯誤訊息  

如果你嘗試去讀取回傳的物件，還會得到 warning。

首先，如果按照提示所講的，將
- fetch mode 改成 no-cors 會發生什麼事呢？

的確把錯誤訊息給處理掉，但是情況似乎沒有比較好一點
- 就算使用 no-cors  mode，CORS 也不會因為這樣就打開大門，請求並不會成功送出
- 也因此出現了 SyntaxError: Unexpected end of input 錯誤
- 這個 mode 通常是跟 `service worker` 搭配使用的。

要解除 CORS 只有一招，就是在
- server 端加上正確 Control-Access-Allow-Origin（host 必須跟 origin 相同或是 *）。

另外
- CORS 這個機制只會運作在 javascript 送出 XHR 或 fetch 時
- 一般像是 curl 或 postman 並沒有這個機制
- 所以常常在測試 API endpoint 時會忽略這項事情，導致前後端在測試 API 時有出入發生

有些跨來源請求不會發生 「預檢請求(preflight)」，而有些請求則會：

1. 必須是 GET, HEAD, POST 其中一種方法
2. 除了 user-agent 自動設置的 header 和特定的 header 之外，不包含其他 header。可接受的 headers
2. 若有 Content-Type（注意是 request header ，不是 response header），則必須是下列的值 application/x-www-form-encoded, text/plain, multipart/form-data

如果不滿足以上條件的話，就會發出 preflight 請求。

以下試著 Content-Type 為 application/json 來達成 preflight 的要件
- 不為 application/x-www-form-encoded, text/plain, multipart/form-data


1. 必須加入一個 OPTIONS 的相同 api endpoint，並且設定 Access-Control-Allow-Origin 來符合 CORS 要件
2. 必須加入 Access-Control-Allow-Headers，且必須包含所有不在條件內 header，否則無法通過

沒有通過 preflight check 的話，會得到錯誤訊息如下：
```
Access to fetch at 'http://localhost:3001/trigger-preflight' from origin 'http://localhost:3000' has been blocked by CORS policy: Request header field content-type is not allowed by Access-Control-Allow-Headers in preflight response.
```

或是沒有在 OPTIONS 的回應標頭裡加上 Access-Control-Allow-Origin:
```
Access to fetch at 'http://localhost:3001/trigger-preflight' from origin 'http://localhost:3000' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
```

如果成功的話，你會看到 network 裡頭有兩個請求
- 一個是 OPTIONS
- 另一個則是真正的請求。

### 附帶身份驗證的請求

cookie 並不能跨域傳遞  
不同 origin 中的 cookie 沒辦法互相傳遞及存取  

不過如果
- 在 a 網域送出了 b 網域的請求
- 且 b 網域回傳了 cookie 的訊息

那麼在 a 網域會以 b 網域的形式儲存一份 cookie  
如果
- 沒有設定 `withCredentials`
- 或是 `credentials: ‘include’` 的話

就算伺服器有回傳 Set-Cookie，一樣不會被寫入


一般情況下如果再使用 b 網域的 API
- cookie 是不會自動被送出去的。
- 必須在 XHR 設定 `withCredentials`
- 或是 `fetch` 的選項中設置 `{ credentials: 'include' }`

因為這也是一個跨域請求，所以也必須遵照 CORS 要件加入 `Access-Control-Allow-Origin`  
為了避免安全性的問題，瀏覽器還有規範 `Access-Control-Allow-Origin` 不能是 `*`  

瀏覽器會自動拒絕沒有 `Access-Control-Allow-Credentials` 的回應  
因此**如果要能夠將身份訊息傳到跨網域的伺服器**當中
- 必須額外加上 `Access-Control-Allow-Credentials: true`

在 Request Cookie 可以看到 cookie 被成功送出



#### 預檢（preflighted）
preflight 就是請求會先以 HTTP OPTION 的方式送去另外一個網域敲門  
確認沒問題後才會送出真正的請求  
一旦觸發了這個條件，要做的事情就會變得麻煩許多  
- request 會先以 HTTP 的 OPTIONS 方法送出
- request 到另一個網域，確認後續實際（actual） request 是否可安全送出

由於跨站 request 可能會攜帶使用者資料，所以要先進行預檢 request


#### HTTP OPTIONS
OPTIONS 方法 用於
- 獲取目的資源所支持的通信選項
- 客戶端可以對特定的 URL 使用 OPTIONS 方法

```
OPTIONS /index.html HTTP/1.1
OPTIONS * HTTP/1.1
```

```
curl -X OPTIONS http://example.org -i
```

```
HTTP/1.1 200 OK
Allow: OPTIONS, GET, HEAD, POST
Cache-Control: max-age=604800
Date: Thu, 13 Oct 2016 11:45:00 GMT
Expires: Thu, 20 Oct 2016 11:45:00 GMT
Server: EOS (lax004/2813)
x-ec-custom-error: 1
Content-Length: 0
```
