# 2023-09-13：筆記 stripe 談 Payment.md
## Charles Watkins (2021/10/29), Paul Asjes (2021/11/16), Charles Watkins (2022/2/2), Paul Asjes (2022/3/4)

----------------

看到 stripe 工程師分享一些關於他們 Payment 技術的文章，共 4 篇:
1. [Fundamentals of the PaymentIntents and PaymentMethods APIs](https://dev.to/stripe/fundamentals-of-the-paymentintents-and-paymentmethods-apis-3646)
2. [The PaymentIntents Lifecycle](https://dev.to/stripe/the-paymentintents-lifecycle-4f5o)
3. [Making sense of Stripe Checkout, Payment Links, and the Payment Element](https://dev.to/stripe/making-sense-of-stripe-checkout-payment-links-and-the-payment-element-23o5)
4. [Level up your integration with SetupIntents](https://dev.to/stripe/level-up-your-integration-with-setupintents-2fl4)

(以為是講 payment 相關的程式主題，但其實是講 stripe 自身的功能與工具，也是不錯啦)  


----------------


## 1. `PaymentIntents` 和 `PaymentMethods` API 的基本原理
了解 payments 工作原理可能會讓人不知所措。開發者必須學習 payment 術語、測試 API 並選擇 revenue model，才能成功完成交易
- 為了提供指導，stripe 推出 Payment 基礎知識，探討開發者的基本話題並解釋其工作原理

 

下面將介紹兩個 Stripe API，做為系列文章的序幕:
- [PaymentIntents](https://stripe.com/docs/payments/payment-intents) 
- [PaymentMethods](https://stripe.com/docs/payments/payment-methods)



---------------

## `Payment` APIs 及這 API 使用方法
要接受付款，你需要一種方法來表示 payment details 和處理交易本身
- 這就是 [PaymentMethods](https://stripe.com/docs/payments/payment-methods) 和 [PaymentIntents](https://stripe.com/docs/payments/payment-intents) 的作用

你的第一個想法可能是
- 我能二選一嗎？
- `PaymentMethods` 和 `PaymentIntents` 相互配合，將 payment details 轉為現實世界中的資金轉賬


讓我們來了解一下它們是什麼，以及它們是如何運作的。

---------------

## 什麼是 `PaymentMethods` API？
[PaymentMethods](https://stripe.com/docs/payments/payment-methods) 和 [PaymentIntents](https://stripe.com/docs/payments/payment-intents) API 讓你能接受各種付款方式，如
- 可以接受從信用卡、銀行帳號到代金券等各種付款方式 

![](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/orci0pmuhlhhufewwrrj.png)  

---------------


## `PaymentMethods` API 如何運作:
1. 客戶在網站上輸入 payment details
2. payment details 發送到 Stripe
3. Stripe 的 `PaymentMethods` API 建立一個 `PaymentMethod object` 
4. `PaymentMethod object` 可用於引用現實生活中的 payment details



---------------


## `PaymentIntents` API 是什麼？
`PaymentIntents` API 會建立 [PaymentIntent object](https://stripe.com/docs/api/payment_intents/object) 
- 用於管理交易細節(如交易金額和貨幣)及 payment flow

不同的付款方式有不同的要求和時限，`PaymentIntents` API 幫助我們管理它們
 
![](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gq5iqppt0hqbkajscpzv.png)  

---------------


## 它們如何協同工作？
當客戶向 Stripe submits payment details 時
- 你會使用 `PaymentMethods` API 來存這些資訊
- 然後，建立 `PaymentIntent object`，並將 `PaymentMethod` 附加到 `PaymentIntent`
- 最後，通過確認完成 `PaymentIntent`
- 確認 `PaymentIntent` 會授權銀行卡和銀行網路進行付款

`PaymentIntent` 會追蹤整個付款生命週期中的狀態，以便在必要時採取適當的下一步措施來完成付款  


---------------

## 為什麼追蹤付款的生命週期很重要？
不同的付款方式完成付款的時間可能不同
- 例如，信用卡交易可以立即完成，但 bank debit 可能需要數天時間
- 此外，付款流程也因存在地區差異
  - 例如歐洲的一些信用卡交易包含 [3D Secure authentication](https://stripe.com/docs/strong-customer-authentication)
- `PaymentIntent object` 通過追蹤付款狀態並指出完成付款所需的下一步操作，來說明付款方法的差異


(如果你用 [Stripe Checkout](https://stripe.com/docs/payments/checkout) 或 [Payment Links](https://stripe.com/docs/payment-links)，Stripe 會在後台處理 `PaymentIntents` 和 `PaymentMethods` 的建立和管理！)  

![](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tamsq3af8s7bijt5ee7w.png)  

---------------


## 付款的 Life cycle:
1. 客戶嘗試交易並輸入 payment details
2. 從 application 向 Stripe 發送 Transaction 和 payment method details
3. Transaction details 發送到 `PaymentIntents` API; payment method details 送到 `PaymentMethods` API
4. `PaymentIntents` API 建立 `PaymentIntent object`，而 `PaymentMethods` API 建立 `PaymentMethod object`
5. `PaymentMethod object` 附加到 `PaymentIntent object` 上
6. PaymentIntents API 將 Transaction 發送到銀行卡或銀行網路，以確認或完成交易


現在你已經了解了 `PaymentIntents` 和 `PaymentMethods` API，是時候整合:
- Stripe 的 [quick start integration builder](https://stripe.com/docs/payments/quickstart) 為 frontend/backend 提供了多種支持語言
- 影片 [Stripe Developers video](https://www.youtube.com/watch?v=WG4ehXSEpz4) 了解整合 `PaymentIntents` 的更多資訊


---------------

## 2. `PaymentIntents` Life cycle

Stripe 希望讓付款盡可能簡單，但所有行業的術語和 API 會讓你有點不知所措
- 這邊提的 `lifecycles` 是什麼意思？
- 各種 PaymentIntent statuses 是什麼意思？
- 什麼是 3DS，它為什麼重要？

下面逐一介紹:
 

---------------



## `lifecycle` 是什麼意思？
在 `PaymentIntents` 的 context 中
- [lifecycle](https://stripe.com/docs/payments/paymentintents/lifecycle) 是 `PaymentIntent` 可能處於的 state list
- 以及它在這些 statuses 中的 progresses 情況
- 由於 `PaymentIntents` 是付款的真實來源，了解這些 statuses 將有助於識別 application 中的付款中的各種問題


先列出 `PaymentIntent` 可能具有的所有 statuses:
- `requires_payment_method`
- `requires_confirmation`
- `requires_action`
- `processing`
- `requires_capture`
- `succeeded`
- `canceled`


首先來看看最基本的 payment lifecycle 是怎樣的:  


![](https://paper-attachments.dropbox.com/s_3A9E8C52EB99000F249DD8F231C764E3985699AD2C6AEB0D48DAA24E5A18EC0F_1636713281761_image.png)  


我們來看看它意味著什麼:  

 

![](https://paper-attachments.dropbox.com/s_3A9E8C52EB99000F249DD8F231C764E3985699AD2C6AEB0D48DAA24E5A18EC0F_1636713487519_image.png)  


`PaymentIntent` 被建立了
- 它的起始 state 是 `requires_payment_method`，表示需要一個 `PaymentMethod` 才能繼續。

 

![](https://paper-attachments.dropbox.com/s_3A9E8C52EB99000F249DD8F231C764E3985699AD2C6AEB0D48DAA24E5A18EC0F_1636713503421_image.png)   



`PaymentMethod` 被附加到 `PaymentIntent` 上
- `PaymentIntent` 已被確認了
- payment details、金額和貨幣已經送出，我們正在 `processing` 狀態處理所顯示的結果


在建立 card payments 時
- 你不太可能在 `processing` 狀態下捕捉到 `PaymentIntent`
- 因為銀行卡 transactions 往往會在幾秒鐘內處理完畢
- 其他付款方式可能需要幾天時間才能完成
 

![](https://paper-attachments.dropbox.com/s_3A9E8C52EB99000F249DD8F231C764E3985699AD2C6AEB0D48DAA24E5A18EC0F_1636713541968_image.png)   




付款已完成，我們已過渡到成功的結束狀態，即 `succeeded`
- 一旦 `PaymentIntent` 達到結束狀態(`succeeded` or `canceled`)，狀態就不能再改變，只能編輯 `metadata` 屬性


這就是我們所期望的無需任何 authentication 的銀行卡付款流程
- 事實上，只需呼叫一次 API 即可完成:

```shell
curl https://api.stripe.com/v1/payment_intents \
  -u sk_test_xxxxxxxxxxxx: \
  -d amount=1000 \
  -d currency=usd \
  -d "payment_method_types[]"=card 
  -d payment_method=pm_xxxxxxxxxxxx
  -d confirm=true
```


這 API call 建立了帶有金額和貨幣的 `PaymentIntent`
- 立即指定了 `PaymentMethod`，並指示 API 確認付款
- 所有這些都在一個請求中完成

下面是個稍微複雜的流程:  


![](https://paper-attachments.dropbox.com/s_3A9E8C52EB99000F249DD8F231C764E3985699AD2C6AEB0D48DAA24E5A18EC0F_1636979218961_image.png)  



跟上個流程幾乎完全相同，但增加了一個新的 `requirements_action` 狀態
 

![](https://paper-attachments.dropbox.com/s_3A9E8C52EB99000F249DD8F231C764E3985699AD2C6AEB0D48DAA24E5A18EC0F_1636979237001_image.png)  

任何以 `requires` 開頭的 state 都表明，除非採取某些行動，否則 `PaymentIntent` 不會進入不同的狀態
- 如果付款在處理之前需要驗證，`PaymentIntent` 就會進入這個狀態
  - 在處理銀行卡時，最常見的身份驗證類型是 `3D Secure(3DS)`


---------------



## 3DS 和 SCA
你可以將 3DS 視為「信用卡的 2FA」
- 當你嘗試付款，但發卡銀行希望確保提出請求的人確實是你時，就會出現這種情況


2019 年 9 月，歐盟實施[強客戶身份驗證(Strong Customer Authentication, SCA)](https://stripe.com/docs/strong-customer-authentication) 
- 這意味著加入該規則的歐洲銀行卡在完成付款前必須進行 `3DS` 檢查
- 如果不支持該流程或客戶未通過身份驗證檢查而導致上述檢查無法完成，發卡銀行將拒絕付款


即使你不經常在歐盟開展業務，也強烈建議整合 Stripe 時將 3DS 考慮在內
- 好處是，如果客戶所在地區實施了類似於 SCA 的新法律，可以確保你已做好準備
- 如果你整合 `Checkout` 或 `Payment Links`，那麼 3DS 會自動為你處理



---------------


## 但所有狀態是什麼意思？
下面對每種狀態進行簡單的解釋:

- `requires_payment_method`: `PaymentIntent` 需要附加 `PaymentMethod` 才能繼續。
- `requires_confirmation`: 已附加 `PaymentMethod`。需要確認 `PaymentIntent`
- `requires_action`: 確認後，需要採取進一步行動。可能是 3DS 檢查
- `processing`: 付款正在處理中，此狀態將自動轉移到新狀態，無需輸入
- `requires_capture`: 只有當的 `PaymentIntent` 處於 [manual confirmation mode](https://stripe.com/docs/payments/place-a-hold-on-a-payment-method) 時才會出現的特殊狀態
  - 需要指示 `PaymentIntent` capture the funds
- `succeeded`: 付款成功。這是種結束狀態
- `canceled`: 付款已取消。這是結束狀態


注意
- 唯一的結束狀態是付款成功 or 被你明確取消
- 在付款被拒絕的情況下，`PaymentIntent` 會自動分離 `PaymentMethod` 並轉換回 `requires_payment_method` 狀態


在檢查帳號上的付款時
- 查看被遺棄的付款處於何種狀態可以幫助你發現結帳體驗中的問題
- 例如，如果大量 `PaymentIntents` 處於 `requires_action` 狀態，表示 user 在面對 3DS 時正在流失
  - 同樣的情況，如果是 `requires_payment_method`，則意味著你的整合出現了問題，沒有正確建立和附加 `PaymentMethod`


---------------
 
## 3. 了解 `Stripe Checkout`, `Payment Links` 和 `Payment Element`

 
你要做的最重要的決定之一就是「**決定如何向客戶展示付款方式**」，請考慮一下:
- application 應該連到付款頁面還是嵌入付款表單？
- 是使用 API 還是選擇 no-code 解決方案？

在回答問題之前，我們先來了解一下 `Stripe Checkout`, `Payment Links` 和 `Payment Element`
- 都是 Stripe payments 
- 符合 PCI-compliant
- 每個界面都支持多種貨幣付款、一次性付款和訂閱
- 此外，它們都支持自動發送 Stripe receipts 和在 `Stripe Dashboard` 中管理退款


要用哪一種？這需要先深入了解每一種的特性  

---------------

## `Payment Links`
`Payment Links` 是 Stripe 接受一次性付款和訂閱的最快、最簡單的方式
- 設計上是 no-code，也為開發者提供了 API: [Payment Link](https://stripe.com/docs/api/payment_links/payment_links)


![](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xga5clasyk0mgtt7y4p1.png)  


使用 [Payment Links](https://stripe.com/payments/payment-links)
- 你可以為產品建立 customer hyperlink，連到由 Stripe 託管的付款頁面
- 你可以在 [Stripe dashboard](https://stripe.com/docs/payment-links) 中設定連結
  - 啟動以後，可用接受 195 個不同國家/地區的付款
- [Payment Links 支援 20+ 不同的全球付款方式](https://stripe.com/docs/payments/payment-methods/integration-options#payment-method-product-support)，可以透過 Stripe dashboard 啟動
- 其他功能還包括支援 [企業間拆分資金和自定義付款頁面](https://stripe.com/docs/connect/collect-then-transfer-guide?platform=no-code)，以匹配你的品牌顏色
- 建立後，Stripe 會提供一個永久連結


記住
- 你不僅可以發送 payment link
- payment link 可以嵌入 QR code，用於接受現場付款
- payment link 可以在任何使用標準連結的地方使用，包括你的網站

---------------


## 何時使用 `payment link` ?
如果你正在啟動一項業務、構建 MVP，或者只是提供一種不會因客戶而異的產品或服務
- 那麼 `payment link` 就是不錯的選擇
- 整合 `payment link` 連結非常簡單，只需在 Stripe dashboard 建立一個產品
- 然後將連結加到你的網站或其他需要與客戶聯繫的地方即可

最適合
- MVP
- 首次接受付款的新創公司
- 嘗試定價的成長型公司
- 沒有網站和/或服務數量有限的企業



---------------

## `Stripe Checkout`
[Stripe Checkout](https://stripe.com/payments/checkout) 是一個託管付款整合
- 可建利一個短期、預先建立的付款頁面
- `Checkout` 輕量，並提供了大量功能，如
  - [訂閱追加銷售(subscription upsells)](https://stripe.com/docs/payments/checkout/upsells)
  - [促銷代碼(promotion codes)](https://stripe.com/docs/payments/checkout/discounts)
  - [發貨選項(shipping options)](https://stripe.com/docs/payments/during-payment/charge-shipping)
- 因此可以與業務共同發展

![](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8pit50gcggpwo6b8mowd.png)

---------------

## 開始使用 `Stripe Checkout`
使用 Checkout，需要 backend 整合 [Stripe’s Checkout API](https://stripe.com/docs/payments/accept-a-payment)
- 用程式方式建立短期連結，可以將用戶重定導向到這些連結
- 設置完成後，可以用 [Stripe dashboard brand settings](https://stripe.com/docs/payments/checkout/customization) 自定義 付款頁面的顏色和品牌
- 此外，還可以[新增和管理付款方式](https://stripe.com/docs/payments/dashboard-payment-methods#view-dash)，為客戶提供 20 多種付款方式


---------------

## 何時使用 `Stripe Checkout`？
你可以用 Checkout 來支援網路商店、創作者平台、訂閱型業務、電子商務市場以及介於兩者之間的任何業務的付款

最適合
- 大多數企業和網路應用程序
- 需要可擴展和優化的開箱即用結帳平台


---------------

## `Payment Element`
[Payment Element](https://stripe.com/docs/payments/payment-element) 是個可嵌入的 UI component
- 用來做自定義付款頁面
- 它是 Stripe 最 customizable 的工具，需要專門的 coding
  - 提供了最大的靈活性，但也需要最多的整合工作
- 你可以在與 `Checkout` 相同的付款場景中使用 `Payment Element`
  - 它額外優勢是可以顯示特殊條款、多步驟付款流程，並在結帳過程中收集額外資訊


你需要用 [Stripe’s SDKs](https://stripe.com/docs/libraries#client-side-and-ui-libraries)
- 在 frontend 和 backend 上整合 Stripe
- 可以使用 [Appearance API](https://stripe.com/docs/elements/appearance-api) 更新外觀，使網站外觀和感覺相匹配


![](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5lquuei91g7k6jq2sjyh.gif)  

---------------

## 開始使用 `Payment Element`
使用 `Payment Element`
1. 你需要在 frontend 使用 [Elements library](https://stripe.com/docs/payments/elements) 
2. 在 backend 上用 [Payment Intents API](https://stripe.com/docs/payments/payment-intents) 進行設定
3. 整合好之後，就可以在 app 上託管一個付款表單

---------------
 
## 何時使用 `Payment Element` ?
`Payment Element` 最適合那些希望或需要擁有 end-to-end 的付款體驗，並願意花時間開發此功能的公司

最適合:
- 付款平台(Payment platforms)
- Mobile app
- 收集附加資訊的 applcation
- 希望 end-to-end 整個環節都控制結帳體驗的公司

---------------


## 4. 提升與 `SetupIntents` 的整合水平
什麼是 `SetupIntents`?
- 還記得 `PaymentIntents` 嗎？如果你想立即付款，就會用到它們
- `SetupIntents` 與之類似，但解決不同的 case
- 你可以使用它來設定未來付款，而不是立即付款
- 為將來使用而使用意味著你可以在建立付款之前**處理最終付款可能需要的任何身份驗證**



如何使用它們？
- 幾乎與使用 PaymentIntent [的方法](https://stripe.com/docs/payments/save-and-reuse)相同

先來考慮以下幾種情境:  

---------------

## 情景 A: 現在購買，稍後付款
假設你的商業模式是
- 要求客戶可以下訂單，但只有在貨物實際發貨後才付款
- 如果在下訂單時不能確定庫存，在這種情況下，你希望立即收集客戶的付款方式詳細資訊，但又不想立即向客戶收錢


收集詳細資訊，[存到客戶資訊](https://stripe.com/docs/payments/save-during-payment)，然後稍後在建立 `PaymentIntent`，這樣很容易
- 但如果持卡人的銀行要求 [Strong Customer Authentication (SCA)](https://stripe.com/docs/strong-customer-authentication) 才能成功付款呢？
- 這時，你需要讓客戶回到 session 狀態(即回到網站上)，以進行身份驗證並確認付款
  - (更多資訊: [SCA and 3DS](https://dev.to/stripe/the-paymentintents-lifecycle-4f5o#:~:text=3D%20Secure%20(3DS).-,3DS%20and%20SCA,-You%20can%20think))


這就是 `SetupIntents` 的使用時機
- 與 `PaymentIntents` 一樣，`SetupIntents` 也可以進行確認並觸發 3DS，讓客戶進行身份驗證
- 主要區別在於，`PaymentIntents` 實際上是在轉移資金，而 `SetupIntents` 則不是
- 相反，`SetupIntents` 可確保下次使用該 `PaymentMethod` 進行的非零付款，而且**不需要 3DS**
  - 這意味著可以建立並確認 `PaymentIntent`(將轉移資金)，而不需要客戶參與
 

![](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ppqsknnjttxp8yyqe92n.png)  



注意: off-session 流程不需要客戶的任何輸入  


表格進一步說明 `PaymentIntents` 和 `SetupIntents` 的區別:

||**PaymentIntents**|**SetupIntents**|
| :---- | :---- | :---- |
| Triggers 3DS | Yes | Yes |
| Lets you defer payment | No | Yes |
| Used to move funds | Yes | No |


延期(Deferment)與 [separate auth & capture](https://stripe.com/docs/payments/place-a-hold-on-a-payment-method) 不同
- 後者的流程中，你將授權資金並在稍後捕獲它們
- 在 `SetupIntents` 流程中，你將授權任何 non-zero 的未來付款，而不是授權特定金額



![](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6r7540xtdfh2k4el21ya.gif)  



上圖顯示了 [tinydemo example](https://stripe-tinydemos-setup-cards.glitch.me/)
- 向客戶展示了 `SetupIntent` 流程的模樣

我們再考慮一種情況。下個方案建立在上個方案的基礎上，展示如何混合使用 `SetupIntents` 和 `PaymentIntents` 來簡化付款頁面

---------------


## 情景 B: 動態決定付款時間
回想一下方案 A，公司只希望在訂單發貨後才接受付款
- 雖然可行，但你可能希望先檢查庫存，從而將付款失敗的風險降到最低
- 假設客戶點擊購買按鈕，那你的業務邏輯可以首先檢查相關商品是否有庫存並準備發貨
  - 如果有，則建立 `PaymentIntent`，讓客戶立刻付款
  - 如果沒有庫存(但很快就會有庫存)，則應建立 `SetupIntent`，以便稍後在 session 外完成付款


![](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/28ug5shyi4w6scvhrho4.png)  


---------------

 
## 情景 C: 訂閱試用期
如果公司提供的是基於訂閱的產品
- 為了吸引更多付費客戶，你決定新訂閱客戶第一個月免費
- 在正常流程中，你會建立包含 payment details 的訂閱，第一張發票會立即觸發付款
  - 如果成功付款，訂購就會啟動，一切順利
- 但在試用期的情況下，你不想預先收取任何費用
- **相反，你需要收集這些 payment details，要等到第一個免費結束後，才在第一張發票上進行會外付款**


這就是 `SetupIntents` 再次發揮作用的地方
- 如果沒有用 `SetupIntents`，那當試用期結束並建立第一張發票時，就有可能由於銀行卡需要 3DS 而導致付款失敗
- 在這情況，你就必須讓客戶返回 session 來完成付款，不僅增加了麻煩，還很有可能導致客戶懶得付款
- 在建立訂閱時使用 `SetupIntent` 可以確保在試用期結束後，當第一張發票出現時，可以在 off session 成功付款

![](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/21v74bmau9sagxgdvq1a.png)


`SetupIntents` 在上面情況下發揮的很好，但在實作時也有些需要注意的地方  

---------------

## 發卡銀行才是 authorization 的權威機構
`SetupIntents` 的好處是
- 可以在不實際建立付款的情況下完成驗證付款的工作
- 但需要注意的是，雖然 Stripe 會在成功確認 `SetupIntent` 後盡力申請例外情況
- 但交易是否需要身份驗證的最終決定權在發卡銀行


這意味著
- 即使你正確設定了稍後使用的 `PaymentMethod`，仍有可能(但很少見)在完成付款前需要進行身份驗證

因此，即使你使用 `SetupIntents` 正確完成所有操作
- **你還是必須要有一個如何讓客戶返回 session 以完成付款的流程，如果沒有，就會面臨客戶放棄付款，你損失收入的風險**


---------------


## `PaymentIntents` 也可以用來設定未來的付款！
並不總是需要 `SetupIntents` 來處理未來付款
- 如果你想 take a payment immediately up front, but then take subsequent payments at a later date off-session
- 可以用 [instructing Stripe when creating the initial PaymentIntent](https://stripe.com/docs/payments/save-during-payment)

