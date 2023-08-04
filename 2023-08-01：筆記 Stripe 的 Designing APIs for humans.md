# 2023-08-01：筆記 Stripe 的 Designing APIs for humans.md
## Paul Asjes, 2022/8/30
### Ref: https://dev.to/stripe/designing-apis-for-humans-object-ids-3o5a


---

Paul Asjes: Stripe 工程師，他分享 4 篇關於 API design 的文章，討論改善 API 可讀性/易用性等等

1. Designing APIs for humans: Object IDs
2. Designing APIs for humans: Error messages
3. Designing APIs for humans: Design patterns
4. Common design patterns at Stripe

Ref:
- https://dev.to/stripe/designing-apis-for-humans-object-ids-3o5a
- https://dev.to/stripe/designing-apis-for-humans-error-messages-94p
- https://dev.to/stripe/designing-apis-for-humans-design-patterns-5847
- https://dev.to/stripe/common-design-patterns-at-stripe-1hb4


---
 
## 1. 為人類設計 API: Object IDs（選擇 ID 的類型）
無論哪種類型的業務
- 都需要 DB 來存客戶資訊或訂單狀態等重要資料
- 存儲只是其中一部分，你還需要快速搜尋資料的方法，這就是 ID (primary key) 的作用


在用 relational database 時
- 最簡單的 ID 就是 row ID
- 每加一新 row 時，ID 就會是下一個順序號
  
但在實際操作中
- 這有嚴重安全性問題
- 攻擊者很容易就能猜到 ID，因為 ID 是連續的
- 如果 ID 無法猜測的，攻擊者利用 API 的難度就會大大增加
- 整數 ID 還很可能洩露公司一些非常敏感的資訊
  - 比如根據客戶和訂單數量來判斷公司的規模和成功與否
  - 在註冊並看到我只是第 42 個用戶後，可能會懷疑你所說的你的業務規模有多大。


所以，你需要確保你的 ID 是唯一的，不可能被猜到
- UUID: 32 位字母數字字符的混合體(字串)
  - 產生速度快，被廣泛採用，而且碰撞非常罕見，因此被認為是唯一性非常重要的系統中唯一標識對象的最佳方法之一

```
4c4a82ed-a3e1-4c56-aa0a-26962ddd0425
```


另一方面，下面是一個 Stripe object ID:
- 為什麼 Stripe 特別使用這種格式？下面來了解一下 Stripe ID 的結構和原因

```
pi_3LKQhvGUcADgqoEM3bh6pslE
```


-------

## Make it human readable

```
pi_3LKQhvGUcADgqoEM3bh6pslE
└─┘└──────────────────────┘
 └─ Prefix    └─ Randomly generated characters
```




所有 Stripe Objects ID 都有 prefix
- 有 prefix 的 ID 就有可讀性
  - `pi_` 這 prefix，我們就可以在不知道 ID 其他資訊的情況下，立即確認我們談論的是一個 PaymentIntent object

通過 API 建立 PaymentIntent 時
- 實際上會建立或引用多個其他 object，包括客戶 `cus_`、支付方式 `pm_` 和費用 `ch_`
- 有了前綴，只需看一眼就能立即區分所有這些不同的 object

```php
$pi = $stripe->paymentIntents->create([
  'amount' => 1000,
  'currency' => 'usd',
  'customer' => 'cus_MJA953cFzEuO1z',
  'payment_method' => 'pm_1LaXpKGUcADgqoEMl0Cx0Ygg',
]);
```

這對 Stripe 內部員工和與使用 Stripe 的開發人員都有幫助
- 例如，下面是以前在被要求幫助 debug integration 時看到的 code

 

```php
$pi = $stripe->paymentIntents->retrieve(
  $id,
  [],
  ['stripe_account' => 'cus_1KrJdMGUcADgqoEM']
);
```


上述 code 試圖從一個已連接的帳號中獲取 PaymentIntent
- 但不用看 code 就能立即發現錯誤：使用的是客戶 ID `cus_` 而不是帳號 ID `acct_`
- 如果沒有 prefix，debug 就困難得多
- 如果 Stripe 使用 UUID，我們就必須查 ID，以確定它是什麼類型的對象，以及它是否有效
- (在 Stripe，我們甚至開發了內部 browser extension，可以根據 ID 自動查找 Stripe Object。因為可以通過 prefix 來推斷類型，所以三次點擊一個 ID 就能自動打開相關的內部頁面，使 debug 變更容易)

-------

## 查詢多種狀態 (Polymorphic lookups)
說到推斷 object 類型，這在設計考慮到向後兼容性的 API 時尤為重要  

建立 `PaymentIntent` 時，你可以選擇提供一個 `payment_method` 參數，以指明想使用的支付工具類型
- 你可能不知道，在這你實際上可以選擇提供 Source (`src_`) 或 Card (`card_`) ID，而不是 PaymentMethod (`pm_`) ID
- PaymentMethods 取代了 Sources 和 Cards，成為在 Stripe 中表示支付工具的標準方式
- 但出於向後兼容的原因，我們仍然需要能夠支持這些舊對象

 
```php
$pi = $stripe->paymentIntents->create([
  'amount' => 1000,
  'currency' => 'usd',
  // This could be a PaymentMethod, Card or Source ID
  'payment_method' => 'card_1LaRQ7GUcADgqoEMV11wEUxU',
]);
```

如果沒有 prefix
- 就無法知道 ID 代表哪種對象，意味著不知道該查詢哪個表來獲取對像資料
- 查詢每個表來查 ID 的效率極低，因此我們需要更好的方法。其中一種方法是需要額外的 `type` 參數

 

```php
$pi = $stripe->paymentIntents->create([
  'amount' => 1000,
  'currency' => 'usd',
  // Without prefixes, we'd have to supply a 'type'
  'payment_method' => [
    'type' => 'card',
    'id' => '1LaRQ7GUcADgqoEMV11wEUxU'
  ],
]);
```


方法可行，但會使 API 複雜化，而且沒有額外的好處
- Payment_method 不再是一個簡單的字串，而是一個 hash
- 另外，這裡沒有任何額外資訊不能合併成一個字串
- 每當使用一個 ID 時，我們都想知道它代表的是哪種類型的對象，因此整合是比需要額外的 type 參數更好的解決方案

有了 prefix
- 就能立即推斷出支付工具是 PaymentMethod, Source 還是 Card，並知道要查詢哪個表，儘管它們是完全不同類型的對象

-------


## 防止人為錯誤
Prefix 還有其他些不那麼明顯的好處
- 其中之一就是當你可以通過前幾個字推斷出 ID 的類型時，就可以輕鬆處理 ID
- 例如，在 Stripe Discord server 上，用 Discord 的 `AutoMod` 來自動標記和阻止包含 `Stripe live` secret API key 的消息洩露
  - (`sk_live_` 開頭)


![](https://res.cloudinary.com/practicaldev/image/fetch/s--c6FoVha4--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dgcna3hdn99dholhnfor.png)  


Stripe 自 2012 年起就開始用這種 prefix 技術
- 據我所知，我們是第一個大規模使用這種技術的公司
- 2012 年之前，Stripe 的所有 ID 看起來更像傳統的 UUID




-------


## 2. 為人類設計 API: 好的 Error messages

Error message 就像來自稅務機關的信件
- 你寧願不收到它們，但當收到時，又希望它們能清楚地告訴接下來要做什麼
- 在集成一個新的 API 時，難免遇到錯誤
  - 即使完全照 doc example code 進行，也總會出錯，尤其是當你的操作已經超出了 exmaple，需要調整以適應你的 use case 時

好的 error message 是 API 中被低估和忽視的一部分
- 在指導開發人員如何用 API 的過程中，error message 與 document 或 example 一樣，都是重要的學習途徑


先看錯誤資訊的例子:

```js
{
  status: 200,
  body: {
    message: "Error"
  }
}
```

這 error message 毫無幫助，讓我們逐一分析。

-----

## 發送正確的 code

這是是一個錯誤，還是？
- message 說是，但 status code 是 200，這表明一切正常
- 令人困惑，而且非常危險！大多數錯誤監控系統首先根據 status code 進行過濾，然後再嘗試 context
- 這個錯誤很可能會被歸入「一切正常」一類，被完全忽略
  - 只有一些自然語言處理功能，才能自動檢測出這實際上是個錯誤

Status code 是為機器準備的，而 error message 則是為人類準備的
- 雖然對 code 有紮實的了解始終是個好主意，但你並不需要知道所有的代碼，尤其是有些 code 有些深奧
- 實際上，你的 API 的用戶只需了解此表即可


| Code | Message |
| :--- | :--- |
| 200 - 299 | 一切正常 |
| 400 - 499 | 你搞砸了 |
| 500 - 599 | 我們搞砸了 | 




當然可以使用更具體的 code (如 rate limit 應該送 429)
- 重點是，HTTP response status codes 是 spec 的一部分是有原因的，你應該始終確保送回的 code 是正確的

這點看似顯而易見，但很容易不小心忘記 status code，就像 `Express.js` 範例一樣:

```js
// ❌ Don't forget the error status code
app.post('/your-api-route', async (req, res) => {      
  try {
    // ... your server logic
  } catch (error) {    
    return res.send({ error: { message: error.message } });
  }  

  return res.send('ok');
});

// ✅ Do set the status correctly
app.post('/your-api-route', async (req, res) => {      
  try {
    // ... your server logic
  } catch (error) {    
    return res.status(400).send({ error: { message: error.message } });
  }  

  return res.send('ok');
});
```

在前面的 code 中
- 無論是否發生錯誤都送 200
- 在下面的 code 中，確保只在發送錯誤資訊的同時發送相應的 code
  - 注意，在 production 中，要區分 400 和 500，而不是對所有錯誤一律發送 400

----


## Be descriptive (error message 要有描述性)
接下來是 error message 本身
- "Error" 的文字和沒有資訊一樣，沒用
- Status code 應該已經告訴你是否發生了錯誤
- erroe message 需要詳細說明，才能真正解決問題

含糊不清的 message 來向 end user 掩蓋內部系統的任何細節可能很誘人(security)
- 但請記住受眾是誰。API 是為開發人員準備的，他們想知道到底哪裡出了問題
- 如果你是 end user，收到 `An error occurred` 也是可以接受的，因為你不需要去 debug
- 作為開發人員，最令人沮喪的莫過於出現故障而 API 不告訴原因

以前面那個錯誤為例，讓它變得更好:
```js
{
  status: 404,
  body: {
    error: {
      message: "Customer not found"
    }    
  }
}
```

可以看到
- 我們有個相關的 status code: `404, resource not found`
- 資訊很清楚，這是個試圖搜尋客戶的 request，但因為找不到客戶失敗了
- Error message 被封裝在錯誤對像中，這樣處理錯誤就容易多了
- 如果不依賴 code，可以簡單地檢查 `body.error` 是否存在，以確定是否發生了錯誤


這樣更好，但仍有改進的餘地。該錯誤功能實用，但沒有幫助

------

## Be helpful (error message 要有幫助)
優秀的 API 讓你知道出錯的原因是最起碼的
- 但開發人員真正想知道的是如何解決這個問題
- 有用的 API 希望與開發人員合作，消除解決問題的任何障礙或阻礙

`Customer not found` 提供一些出錯原因的線索
- 但作為 API 的設計者，我們還可以提供更多的資訊
- 首先，明確指出是哪個客戶未找到

```js
{
  status: 404,
  body: {
    error: {
      message: "Customer cus_Jop8JpEFz1lsCL not found"
    }    
  }
}
```

現在不僅知道出錯了，還能得到錯誤的 ID
- 這在查看一系列 error log 時特別有用，因為能告訴我們問題是出在特定的 ID 還是多個 ID 上
- 這就提供了線索，讓我們知道問題是出在單一客戶身上，還是出在發出 request 的 code 上
- 此外，ID 有 prefix，我們可以立即判斷是否使用了錯誤的 ID 類型

還可以進一步。在 API 方面
- 我們可以訪問對解決錯誤有幫助的資訊
- 我們可以等待開發人員自己嘗試解決，也可以向他們提供我們知道有用的附加資訊

例如，在未找到客戶的原因可能是所提供的客戶 ID 在 live mode 下存在，但我們使用的是 test mode keys
- 使用錯誤的 API key 是很容易犯的錯誤，一旦知道問題所在，解決起來也很簡單
- 如果我們在 API 進行快速查找，看看 ID 所指向的客戶對像是否存在於 live mode 中，我們就可以立即提供該資訊

```js
{
  status: 404,
  body: {
    error: {
      message: "Customer cus_Jop8JpEFz1lsCL not found; a similar object exists in live mode, but a test mode key was used to make this request."
    }    
  }
}
```

這比我們以前使用的要有用得多。它能立即找出問題所在，並提供解決問題的線索。這技巧的其他例子還有
- 在類型不匹配的情況下，說明預期的結果和收到的結果
  - (`xpected a string, got an integer`)
- 請求是否缺少權限？告訴他們如何獲得權限
  - (`Activate this payment method on your dashboard with this URL`)
- 請求缺少一個 field ? 準確說明缺少哪個 field，或許可以連到 doc 或 API 參考中的相關頁面

注意：在類似最後一點的情況下
- 請謹慎處理你所提供的資訊，因為這些資訊有可能洩露，從而帶來安全風險
- 在驗證 API 的情況下，如果你在請求中提供了 username 和 pass，那麼返回 `incorrect password` error 會讓潛在攻擊者知道，雖然密碼不正確，但 username 是正確的

--------


## 提供更多的資訊
我們可以而且應該盡可能地提供幫助，但有時這還不夠
- 可能遇到過這樣的情況：你認為 request 傳了正確的 field，但 API 卻不同意
- 找到解決方案的最簡單方法就是回過頭來看看原始 request 以及你到底傳入了哪些 field
- 如果開發人員沒有設置某種 log，那很難做到這點，但是 API 服務應該總是有 request/response log，為什麼不與開發人員分享呢？

在 Stripe，我們會在每個 response 中提供一個 request ID
- 這 ID 很容易識別，是 `req_` 開頭
- 使用該 ID 並在 dashboard 上查，就會得到詳細說明 req/res 的頁面，還有幫助 debug 的其他詳細資訊

![](https://res.cloudinary.com/practicaldev/image/fetch/s--yAB6gZQT--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vn4945cgfpcq0mhpls9w.png)  


注意，dashboard 還提供了 timestamp, API version，甚至 source code
- 本例中為 `stripe-node` 8.165



------


## 要有同理心
最令人沮喪的錯誤是 500 error
- 這意味著 API 方面出了問題，不是開發人員的錯
- 這類錯誤可能是一時的故障，也可能是 API provider 的中斷，而你當時根本無法知道
- 如果 end user 在業務關鍵路徑上依賴於你的 API，那出現這類錯誤就非常令人擔憂，尤其是當你開始接二連三地出現這類錯誤時

與其他錯誤不同
- 這裡並不希望完全透明
- 你不希望將導致 500 的 internal error 直接轉入 response，這樣會洩露系統內部運作的敏感資訊
- 應該對用戶導致錯誤的行為保持完全透明，但在導致錯誤時，需要小心分享的資訊

上面第一個例子一樣，乏善的 `500: error` 和沒有 message 一樣沒用
- 相反，通過同情的方式讓開發人員放心，確保他們知道錯誤已被確認，並且有人正在處理

舉例
- "An error occurred, the team has been informed. If this keeps happening please contact us at `{URL}`"
- "Something went wrong, please check our status page at `{URL}` if this keeps happening"
- "Something goofed, our engineers have been informed. Please try again in a few moments"

這樣做並不能解決根本問題，但可以讓用戶知道你們正在處理，如果錯誤繼續出現，他們可以選擇跟進，從而減輕打擊

------

## Putting it all together
總之，有價值的錯誤資訊應該
- 使用正確的狀態代碼
- Error message 具有描述性
- Error message 有幫助
- Error message 提供詳細的資訊
- 有同情心

下面是 Stripe API error response 範例，在嘗試使用錯誤的 API key 搜尋客戶後出現了錯誤:

```js
{
  status: 404,
  body: {
    error: {
      code: "resource_missing",
      doc_url: "https://stripe.com/docs/error-codes/resource-missing",
      message: "No such customer: 'cus_Jop8JpEFz1lsCL'; a similar object exists in live mode, but a test mode key was used to make this request.",
      param: "id",
      type: "invalid_request_error"
    }
  },
  headers: {    
    'request-id': 'req_su1OkwzKIeEoCy',
    'stripe-version': '2020-08-27',    
  }  
}
```
(為簡潔起見，省略部分 header)  

這裡有
1. 使用正確的 HTTP status code
2. 用 `error` object 封裝錯誤
3. 提供以下幫助
    1. 錯誤代碼
    2. 錯誤類型
    3. 相關 doc 連結
    4. 此 request 中使用的 API 版本
    5. 關於如何解決問題的建議
4. 提供 request ID 以查找 request/resoonse 配對

這樣，錯誤資訊中就會出現大量有用的資訊，即使是最初級的開發人員也能修復問題，並發現如何使用可用工具自行 debug

------

## 3. 為人類設計 API： Design patterns


API 之所以有趣，是因為它可以決定產品的成敗
- 如果產品令人驚嘆，但 API 卻不完整、難以使用或完全令人困惑，意味著絕大多數用戶可能永遠不會發現你的產品到底有多棒
- 如果開發人員不能在最初的 30 分鐘內取得一些有意義的進展，你可能就永遠失去他們
- 反之，擁有出色 API 體驗的平庸產品則可能表現出色



-------


## 了解你的用戶
要獲得出色的 API 體驗，在寫第一行 code 之前就開始考慮 design patterns  

在設計時，應牢記以下三點:
1. 目標受眾是誰？
2. 你希望 API 的 customizable 程度有多高？
3. 為了 customizable，你願意放棄多少 accessibility ?

-----

## 誰是目標受眾?
在確定 design pattern 前，了解受眾是絕對必要的
- 一般來說，API 是為高級用戶設計的，但你是否也想迎合 junior developers 的需求呢？
- 可以讓 API 保持簡單的交互方式，並為用戶做出許多有主見的決定

如果你是小眾產品
- 只有一部分開發人員使用(如，只使用 Java 或 C++ 的企業開發人員)，你是否會更傾向於他們偏好的 object models 和語言?



-----


## 你希望 API 的 customizable 程度如何?
功能和 accessibility 也是重要因素
- 功能強大的 API 可為 end users 提供大量選項
  - 如自定義 API request 的許多不同因素
- 但這是以 accessibility 為代價的
  - 因為選項越多，API 就越複雜
- 複雜的 API 在嘗試建立 integration 時更有可能碰壁，使得令人沮喪
  - 例如，你可能不得不在沒有足夠經驗的情況下做出複雜的決定，而這些決定可能會對 project 的未來版本產生影響

<p align="center">
  <img width="400" src="https://res.cloudinary.com/practicaldev/image/fetch/s--6ARZT4er--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a7u3nqeak3ui5ydr69l4.png">
</p> 


------

## 你願意放棄多少 accessibility 換取 customizability ?

在圖中的位置在很大程度上取決於第一個問題
- 你的目標受眾是誰?
- 記住，雖然你可以將低功耗、低複雜度的 API 轉化為強大而複雜的 API，但如果不從頭開始，幾乎不可能實現相反的效果
- 偏向於先從簡單的開始，然後再逐步完善，但盡量不要把自己設計得陷入困境而難以自拔

儘早弄清 design patterns 會帶來巨大的好處
- 有助於提高 API 的壽命和開發速度

------



## 保持一致性
首先是 consistent
- 先舉個簡單例子來說明。 API 最常見的操作通常是請求 GET 資源和 POST 建立或更新資源


```
// Create a PaymentIntent
POST /v1/payment_intents

// Update a PaymentIntent
POST /v1/payment_intents/:id

// Get a specific PaymentIntent
GET /v1/payment_intents/:id
```

在上例中，一個 GET PaymentIntent 的 route
- 但一次只能 get 一個。如果需要 PaymentIntents list 會很麻煩
- 加 list endpoint 可以很好地改善，為用戶提供靈活性，並能將操作減少到盡可能少的 API request 

```
// Get a list of PaymentIntents
GET /v1/payment_intents
```

現在想像一下
- 如果能列出 PaymentIntents，卻不能列出客戶等其他資源，那是多麼沮喪
- 作為 API 設計者，希望 API 盡可能具有可預測性，允許用戶根據以前使用 API 的經驗猜測 HTTP method 和 endpoint 的組合
- 要嚴格保持設計的一致性；新 route 的工作方式應與現有 route 基本相同
  - 讓用戶更快地掌握 API 的使用方法，感受精心設計的直觀體驗

-----


## Be intuitive
深入了解一下直觀的含義
- 假設 API 中有個客戶資源對象，它包含 3 個 field: name, email and address
- 首先，我們建立客戶

```
// Request
curl https://api.example.com/v1/customers 
  -u sk_test_123
  -d "name"="Leto Atreides"
  -d "email"="leto@houseatreides.com"
  -d "address"="1 Palace Street, Caladan"

// Response
{
  "id": "cus_123",
  "object": "customer",
  "name": "Leto Atreides",
  "email": "leto@houseatreides.com",
  "address": "1 Palace Street, Caladan"
}
```

之後，我們再更新:  

```
// Request
curl https://api.example.com/v1/customers/cus_123 
  -u sk_test_123:
  -d "address"="1 Spice Road, Arrakis"

// Response
{
  "id": "cus_123",
  "object": "customer",
  "name": undefined,
  "email": undefined,
  "address": " 1 Spice Road, Arrakis"
}
```


這是個簡單的 request，只是更新現有客戶的地址
- 但，姓名和電子郵件的值去哪兒了？
  - API 完全按照要求做了。更新地址，但由於 request 中沒有姓名和電子郵件，所以它將此理解為請求取消設置這兩個值
- 這讓人非常沮喪，但在技術上是正確的，而且清楚地表明該 API 在設計時沒有考慮到人類
- 這個 API 的設計並沒有檢查是否傳入了參數，而是用屬性值更新了對象，無論是否提供了屬性值

API 需要 be intuitive，像上面這樣的操作應該是「正常工作」的

-----

## 4. Stripe 的 common design patterns
你可能不同意 Stripe API 的 design patterns，你最終採用的設計很可能與我們不同。這沒有問題，因為不同的公司有不同的用例
- 在這介紹一些 design patterns，這些模式足夠通用，在 API 設計過程中對任何人都很有用


語言
- 命名很難，API 設計也不例外
- 這裡的問題在於，與 variable/function 命名類似，希望 API 的 routes, fields and types 簡潔明了

這在最好的情況下也很難做到，但這裡有些提示

-----

## 使用簡單的語言

這是顯而易見的建議
- 但實際操作中卻很難做到。盡量將一個概念提煉到其核心，不要害怕使用同義詞詞庫。

例如，建立 platform 時
- 不要混淆用戶和客戶的概念
  - 用戶(在 Stripe 術語中)是直接使用平台產品的一方
  - 客戶(也稱為 "最終用戶"(end-user))是最終購買用戶可能提供的商品或服務的一方
- 你不必使用這些確切的術語（`user` 和 `end-user` 都完全可以），只要用語一致即可



-----

## 避免術語
各行各業都有術語
- 不要以為用戶對特定行業瞭如指掌
- 例如，信用卡上看到的 16 位數字被稱為 `Primary Account Number`，簡稱 `PAN`
- 在金融科技圈中，人們談論 `PAN`, `DPAN`, `FPAN` 很正常的

因此你可以在支付 API 中使用類似的術語:
```js
card.pan = 4242424242424242;
```

即使你知道 `PAN` 代表什麼
- 但它與信用卡上 16 位數字之間的聯繫可能仍然不明顯


因此，避免使用術語，而使用更容易被更多人理解的語言：
```js
card.number = 4242424242424242;
```

在考慮 API 的受眾是誰時，這一點尤為重要
- 實施 API 的人的核心角色很可能是開發人員，對金融科技或任何其他專業領域一無所知
- 一般來說，最好假設人們不熟悉你所在行業的術語



------



## Structure: 首選 enums 而不是 booleans
假設有個訂閱模式的 API
- 作為 API 的一部分，我們希望用戶能夠確定相關訂閱是處於 `active` 狀態還是已被 `canceled`

以下 interface 似乎是合理的:
```js
Subscription.canceled={true, false}
```

這可行
- 但假設在實施功能一段時間後，決定推出一項新功能: 暫停訂閱 (pausing subscriptions)
- 暫停訂閱意味著我們將暫時停止付款，但訂閱仍然有效，沒有取消

為了反映這一點，我們可以新增一個 field:
```js
Subscription.canceled={true, false}
Subscription.paused={true, false}
```
 
現在，為了查看訂閱狀態，我們需要查看兩個 field 而不是一個
- 這也帶來了更多困惑：如果訂閱有 `canceled: true` 和 `paused: true`，該怎麼辦？
  - 已取消的訂閱也能暫停嗎？也許這是個錯誤，並規定取消的訂閱必須有 `paused: false`

這是否意味著已取消的訂閱可以取消暫停？
添加的 field 越多，問題就越嚴重
- 不能只檢查一個值，而是需要一堆令人困惑的 `if/else` 來弄清這個訂閱到底發生了什麼


不妨考慮以下模式:
```js
Subscription.status={"active", "canceled"}
```


用 enums，單個 field 就能知道狀態
- 這技術的另個優點是 extensibility

回到之前的例子，需要做的就是加個額外的 enums:
```js
Subscription.status={"active", "canceled", "paused"}
```

增加了功能，但 API 複雜性保持不變，同時描述性更強
- 如果決定刪除訂閱 pausing 功能，刪除 emun 總是比刪除 field 更容易

這並不意味著永遠不應該在 API 中使用 booleans
- 因為幾乎可以肯定在某些 edge 情況下 booleans 更有意義
- 相反，建議你在加 booleans 之前，考慮將來 booleans 邏輯不再有意義的可能性(例如，有第三個選項)



-----

## Structure: 使用 nested objects 實現未來可擴展性

例如，嘗試從邏輯上將 field 分組。如下所示：

```js
customer.address = {
  line1: "Main Street 123",
  city: "San Francisco",
  postal_code: "12345"
};
```

另種方式:
```js
customer.address_line1 = "Main street 123";
customer.address_city = "San Francisco";
customer.address_postal_code: "12345";
```

第一種方便於日後加其他 field
- 例如，如果決定將業務擴展到海外，可加國家 field，並確保 field 名稱不會太長
- 保持 top resource 的整潔美觀



------

## Responses: 返回 object type


在大多情況下，call API 是為了取或修改某些資料
- 修改，通常是返回被更改資源的表示形式
- 例如，更新 email address，作為 200 response 的一部分，你會期望得到帶有新的更新 email 的客戶副本

為了讓開發人員的生活更輕鬆
- 明確返回的具體內容
- 在 Stripe API 中，我們在 response 中設置了 object field，清楚說明我們正在處理的內容。例如，API route


```
/v1/customers/:customer/payment_methods/:payment_method
```


返回附加到特定客戶的 PaymentMethod
- 希望從 route 中可以明顯看出，你應該期待返回 PaymentMethod
- 但為了以防萬一，我們包含了該 object field，確保不會混淆

```json
{
  "id": "pm_123",
  "object": "payment_method",
  "created": 1672217299,
  "customer": "cus_123",
  "livemode": false,
  ...
}
```

這在篩選 log 或為集成加一些防禦判斷很有用:

```js
if (response.data.object !== 'payment_method') {
  // Not the expected object, bail
  return;
}
```

-------

## Security: 使用 permission system
假設你正在為 product dashboard 開發新功能
- 這是大客戶特別要求的。你準備讓他們以測試版的形式進行測試，以獲得一些反饋
- 因此你讓他們知道向哪條 route 提出 request 以及如何使用該 route
- 新 route 並沒有在任何地方公開記錄，除了你的客戶，應該沒人知道，所以你也不用太擔心

幾週後，你針對反饋意見對功能進行了修改
- 結果收到了其他用戶的一系列憤怒郵件，詢問為什麼他們的集成突然中斷了


災難發生了
- 原來是你的秘密 API route 被洩露了
  - 也許那位客戶對新功能感到非常興奮，於是告訴他們的開發人員朋友
  - 又或者，該客戶的 user 在 networking panel 上看到了這些對未註明 API 的 request，並決定在自己的產品中使用該功能
- 你不僅要清理當前的混亂
- 而且現在你的測試版功能實際上已經被拖入了啟動狀態
  - 由於做出任何新的更改都需要通知每一位用戶，因此開發速度已經慢到了極點


API Reverse engineering 有時候並沒有這麼困難
- 除非採取措施加以防止，否則，可以認為人們會這樣做
- [Reverse engineering an API](https://medium.com/better-practices/reverse-engineering-an-api-403fae885303)

所謂隱蔽安全，就是認為隱藏的東西就是安全的
- 如果想確保私有 API 不被洩露，就必須確保除非用戶擁有正確的權限，否則無法訪問這些 API
- 最簡單的方法就是將權限系統與 API key 綁定。如果 API key 無權使用路由，則應及早退出，並返回狀態為 403 的錯誤資訊

-------


## Security: 讓 ID 無法被猜測
最上面提過，但值得再次提及
- 如果設計的 API 會返回帶有 ID 的 object，確保這些 ID 無法被猜或逆向工程
- 如果 ID 只是簡單的序列，那最好的情況是無意中洩露業務資訊，最壞情況是 security 事故

舉例，如果在網站上購物，確認訂單 ID 為 10，那可以做出兩個假設:
1. 你的業務量可能沒有你聲稱的那麼多
2. 我可能可以獲得之前 9 個訂單（以及所有未來訂單）的資訊，但我不應該獲得這些資訊，因為我知道它們的 ID


第二個假設，可以嘗試濫 API，從而找出更多關於其他客戶的資訊:
 
```
// If the below route isn't behind a permission system, 
// I can guess the ID and get potentially private 
// information on your other customers
curl https://api.example.com/v1/orders/9 

// Response
{
    "id": "9",
    "object": "order",
    "name": "Lady Jessica",
    "email": "jessica@benegesserit.com",
    "address": "1 Palace Street, Caladan"
}
```

用 UUID 使 ID 無法猜測
- 在方便性上的損失(談訂單 42 比談訂單 123e4567-e89b-12d3-a456-426614174000 容易得多)可以在安全性上得到彌補
- 不要忘了加 prefix 使其具有可讀性，以 order_3LKQhvGUcADgqoEM3bh6pslE 將使你和使用你的 API 的 user 的生活更加輕鬆


## 作者推薦的一些資源
- [APIs you won’t hate](https://apisyouwonthate.com/)
- [Why Show Users Garbage API Errors?](https://apisyouwonthate.com/blog/why-show-users-garbage-api-errors/)
- [Creating Good API Errors in REST, GraphQL and gRPC](https://apisyouwonthate.com/blog/useful-api-errors-for-rest-graphql-and-grpc/)

