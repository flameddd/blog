# 2023-05-04：筆記 samcurry 的 Hacking a Massive Steam Scamming and Phishing Operation for Fun and Profit
## samcurry (samwcyo), May 8, 2018
### https://samcurry.net/hacking-a-massive-steam-scamming-and-phishing-operation-for-fun-and-profit/


----------------

一篇 samcurry 的舊 blog 

----------------


我經常玩 CS:GO 或 PLAYERUNKNOWN'S BATTLEGROUNDS，玩起來真的很有趣
- 但有趣的是，它們的可交易遊戲物品非常有價值
- 由於物品的價格很高，而且往往是未成年的沒有經驗的玩家，這些社區是詐騙和欺詐的成熟之地

有一天，在接受了一個不知名的朋友請求後
- 我的帳號被更多的朋友請求所淹沒
- 所有的帳號都有 CSGO money 或 CSGO gambling 這樣的標題
- 他們會定期發送訊息，這些訊息都遵循非常相似的格式

![](https://samcurry.net/wp-content/uploads/2018/05/msg.png)  


我收到的每一條訊息都會有
1. 一個賭博或交易網站的 line，然後
2. 某種促銷獎勵。

起初我以為這只是一個可愛的賭博網站廣告方式
- 但直到有人開始給我發訊息，指責我接管了他們的帳號，我才懷疑這些網站是惡作劇或欺詐行為。

> dyoungclaus: 為什麼我把你加進去了？
> dyoungclaus：你能解釋一下你為什麼黑了我的帳號嗎？
> dyoungclaus：？

給我發訊息的人最初是通過給我發送一個隨機的朋友請求開始我們的關係
- 隨後是一個廣告，最後指責我黑了他們的帳號
- 我的猜測是，他們所宣傳的 link 以某種方式誘使使用者將帳號訪問權交給第三方，或者是對他們的憑證進行釣魚。

在這一點上，我很好奇網站上到底有什麼，所以我看了一眼:  
![](https://samcurry.net/wp-content/uploads/2018/05/csgo-1600x822.png)  


我看的第一個網站（請記住，我至少收到 50 條這樣的訊息，每條都有不同的域名）有一個有趣的前提:

1. Deposit skins 將被轉化為貨幣
2. 使用上述貨幣購買一個 "箱子(“case”)"，這將使你有機會獲得更有價值的物品
3. 將你的贏利兌現到你的帳號
    - 由於所有這些都是由未知的第三方（不管是誰擁有這個網站）控制的，而不是真正的遊戲
    - 所以人們對是否能真正贏得更稀有的物品（任何比箱子本身更貴的東西）幾乎沒有信任

|||
| :----: | :----: |
| ![](https://samcurry.net/wp-content/uploads/2018/05/csgo_2-1600x822.png) | ![](https://samcurry.net/wp-content/uploads/2018/05/csgo_3-1600x803.png) |


我甚至沒有想到的一個問題是 `can you even withdraw your items?`
- 答案當然是 `no`

![](https://samcurry.net/wp-content/uploads/2018/05/csgo_4.png)  

這個騙局相當容易上手：
- 告訴使用者他們在接受瘋狂促銷後贏得了一件物品，要求他們存款超過 20 美元，不給他們的「獎金」


在這次行動中，仍有幾件事讓我感到困惑:
1. 他們是如何得到這麼多合法帳號來做廣告的？
2. 我從私人訊息中看到至少有50個不同的域名。誰在經營這些網站？
3. 他們通過詐騙人們賺了多少錢？



## 調查
在打開兩個獨立網站後，我繼續調查，發現它們幾乎是彼此的相同版本


|||
| :----: | :----: |
| ![](https://samcurry.net/wp-content/uploads/2018/05/site1-1600x798.png) | ![](https://samcurry.net/wp-content/uploads/2018/05/site2-1600x799.png) |

在這一點上，我**意識到它們都必須以某種方式聯繫在一起**
- 但我仍然不確定這些東西是如何被設置和分發的
- 我決定做的第一件事是把它當作一個普通的漏洞賞金計劃，並試圖確定誰在託管該 app 及 server 提供給我們的訊息
- 我繼續前進並打開 Burp，然後向該網站發送一個標準的 HTTP 請求

以下是我收到的 response
```
HTTP/1.1 200 OK
X-Powered-By: SomeSpecificString
X-RateLimit-Limit: 200
X-RateLimit-Remaining: 199
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept
Content-Type: text/html; charset=utf-8
Content-Length: 53748
ETag: W/"d1f4-ZwMTxTE0SgpKtTCYdOrz9CjojXk"
set-cookie: cax9IuiMGwowstealmycookiemuch?7TY57q=s%3A0QWXsdonotstealmycookienoobQ5oDGAIoO.7K43b%2FKnVd%2Bq2gb0oT63nbtDG4; Path=/; HttpOnly
Date: Wed, 09 May 2018 00:51:18 GMT
Connection: close

```

如果你做過任何 bug bounty
- 你會注意到上述 HTTP response 有個非常獨特的 header

```
X-Powered-By: SomeSpecificString
```

這讓我很困惑，因為這是我第一次看到這個標題
- 我做了任何人都會做的事，google

![](https://samcurry.net/wp-content/uploads/2018/05/verygoodgirl-1.png)  

這個特定 header 有將近 300 個結果
- 都與 CSGO 賭博或交易網站聯繫在一起
- 這並不是對存在多少網站的真實統計，因為許多網站是像 `htmlstat.domains` 或 `ipaddress.com` 這樣的網站
- 但看到這個獨特的字符串是如何將它們聯繫在一起的，絕對是很有趣的

我採取的下一個步驟是去 [Shodan](https://www.shodan.io/)，以獲得一個更具體的想法，即這些網站有多少存在:  
![](https://samcurry.net/wp-content/uploads/2018/05/shodan-1-1600x892.png)  

看起來 Shodan 已經對這些主機中的 73 個進行了指紋識別
- 但其中許多實際上並不是賭博網站
- 下面是一個沒有遵循上述截圖中描述的格式的例子

![](https://samcurry.net/wp-content/uploads/2018/05/scam-1-1600x961.png)  


注意到上面的截圖有什麼奇怪的地方嗎？
- `ctean***********.com` 絕對不是 `steamcommunity.com` 的
- 這很可能是他們檢索所有帳號來發送垃圾郵件的方式

在對該 domain 進行了一段時間的研究後
- 我決定發送一個 `blind XSS payload`，並在離線狀態下運行幾個小時
- 希望該網站沒有對 HTML 進行消毒
- 因此允許我在騙子的 instance 中執行任意 JS，這將使我能夠看到持有 credentials 的面板

## Hacking the Hackers
從短暫的午睡中醒來後
- 我注意到使用我最喜歡的 blind XSS 工具[XSS hunter](https://xsshunter.com/#/) 執行的  multiple fires

```html
<html lang="en"><head>
 <meta charset="UTF-8">
 <title>VGGSTEAM Admin Panel</title>
 <script src="//cdn.jsdelivr.net/npm/alertifyjs@1.11.0/build/alertify.min.js"></script>
 <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css">
 <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/alertifyjs@1.11.0/build/css/alertify.min.css">
 <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/alertifyjs@1.11.0/build/css/themes/default.min.css"> 
 <link rel="stylesheet" href="/assets/css/app.css">
 <script src="/assets/js/socket.io-1.4.5.js"></script>
 <script src="/assets/js/jquery.min.js"></script>
 <script src="/assets/js/app.js"></script>
```

僅僅通過觀察 DOM，我就能確定這是一個管理員面板，用於他們所運行的任何網絡釣魚或詐騙服務。發生的情況是

1. 我提供了我的 XSS hunter payload 作為釣魚網站的使用者名
2. 釣魚網站收到了我的輸入，然後在「失敗」的訊息下將其提供給受害者的瀏覽器，因為憑證是無效的。
3. XSS hunter payload 向我發送了瀏覽器 DOM, cookies, 受害者 IP 地址和它所執行的 endpoint

![](https://samcurry.net/wp-content/uploads/2018/05/xss-1600x891.png)  



使用 XSS hunter 的釣魚網站的管理面板的截圖
- 所附的日誌顯示了登錄釣魚帳號的嘗試，以及在受害者啟用 2FA 後發送的相關代碼

所附的 XSS payload 的 endpoint 顯示為 `ctean*******.com/admin/8vya1znto4wqoyjbkqr5q8au3xgsip08`
- 我做的第一件事是訪問這個 endpoint，我立即以管理員的身份登錄了

這樣做的原因（我想）是攻擊者認為沒有人能夠預測或猜測 `8vya1znto4wqoyjbkqr5q8au3xgsip08` 的位置
- 因此認為訪問管理面板不需要認證。

在瀏覽了一段時間後，很明顯
- 被入侵的帳號立即被搜索出有價值的庫存物品
- 如果存在這些物品，它們將被交易給攻擊者，並被清空
- 僅在這個域名上就有數百個被入侵的帳號。下面是一個例子，說明這些帳號是如何組織的。

![](https://samcurry.net/wp-content/uploads/2018/05/hacked_accounts-1600x889.png)  


有趣的是，從一個新設備立即進行交易並沒有觸發 Steams 的 `new device, no trading` 政策
- 黑客們很可能找到了一個繞過這個問題的方法，這就更有意思了

該網站有額外的 interface，可以代表使用者批量發送訊息（這可能是我收到這麼多垃圾郵件的原因）
- 還有一個按鈕，可以讓我登錄到受害者的帳號，而無需在任何地方進行認證
- 該網站似乎濫用了 Steam 登錄功能的某些功能，允許攻擊者在第三方網站上存儲 session，他們可以點擊一個按鈕並完全訪問某人的帳號

我決定通過 HackerOne 向 Valve 報告這一大規模行動
- 因為我在幾個月前就被邀請作為私人參與者
- 儘管這不是 Steam 的直接安全漏洞，我仍然覺得通過 HackerOne 平台報告是最好的選擇
  - 因為我能夠恰當地描述這個問題，並就我如何批量識別這些網站進行對話
  - 而不是向他們的支持部門發送一封看起來很垃圾的電子郵件，他們很可能不理解報告的內容

## Timeline
January 29th – report issue to the private Valve HackerOne program
January 29th – receive response asking for a list of compromised accounts
January 31st – issue triaged
February 1st – $1,000 reward given for effort put into the report


一般來說這不是 Valve 會獎勵的東西，收到賞金的原因如下：
```
謝謝你的報告。雖然這個報告不涉及我們可以控制的資產，但我們已經對受影響的使用者帳號採取了行動，使他們受到的傷害盡可能小。通常情況下，關於我們不控制的東西的報告沒有資格獲得賞金，但在這種情況下，報告的詳細程度值得支付賞金。
```