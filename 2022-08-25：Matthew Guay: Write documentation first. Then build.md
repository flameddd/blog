# 2022-08-25：Matthew Guay: Write documentation first. Then build.md
## June 10, 2022
### https://reproof.app/blog/document-first-then-build


> "49年前，Kettering University’s Marvin Swift 在《哈佛商業評論》："寫作是一種發現的方式。找出思考的內容，以及建立的內容。


## 測量兩次，切割一次 (Measure twice, cut once)

這一切都從一個想法開始
- 你可以想像它；它幾乎感覺真實。現在你只需要把你的設想轉化為現實

所以掏出筆來。開始寫
- 誘惑是直接跳入建設模式，寫代碼或建立線框或草圖你想要創建的東西
- 更好的想法是把你的想法寫下來，作為一種思考和改進的方式

然後，當你開始構建時
- 你會清楚地知道要構建什麼，以及為什麼，你會在啟動時有預先寫好的文件

這也不是你一個人的事
- 你會有一個團隊，至少有員工或測試人員或客戶
- 你永遠無法傳達你對一個想法的所有想法；你每次講故事時都會忘記一些東西，再加一點，一個隱含的承諾，你的最終產品可能無法遵守

先寫出你的想法
- 然後你就有了一個可遵循的腳本
- 一個與他人分享你的故事的一致方式。我們是這個，不是那個，這就是原因。

最初的 PC spreadsheet，VisiCalc，是 Dan Bricklin 夢想出來的，由 Bob Frankston 在馬薩諸塞州的閣樓上開發。Bricklin 在 TED 中說：「我做了更多的工作來弄清楚這個程序到底應該如何運作。」
- https://www.ted.com/talks/dan_bricklin_meet_the_inventor_of_the_electronic_spreadsheet/

第一個電子表格首先是作為文件出現的。Bricklin 說：
- 我寫了一張參考卡，作為文件
- 上面有一張手繪和註釋的 VisiCalc 未來界面的傳真件
- 寫作幫助整合想法，把原始電子表格的所有功能都寫在 5 頁簡明的文件中
  - (http://www.bricklin.com/history/refcard1.htm)
- 這些文件是軟件的最初宣傳
- 它幫助我確保我所定義的用戶界面能夠簡潔明了地解釋給普通人
- 它指導了程序的開發，因為團隊提前決定了一些東西應該如何工作，而不是僅僅詳細說明程序如何工作，希望人們能夠理解

Frankston 補充說
- 如果我們想不出如何解釋參考卡上的一個功能，我們就會改變程序
  - (https://www.landley.net/history/mirror/apple2/implementingvisicalc.html)

而當要發布 VisiCalc 的時候，文件已經準備好了，只等著把 Bricklin 的手繪界面換成真實的屏幕截圖了


Nest 恆溫器的誕生首先是一個簡單得多的東西：一份新聞稿和包裝  
Tony Fadell 在 [Build](https://www.amazon.com/Build-Unorthodox-Guide-Making-Things/dp/B09LGNR2TW/) 中寫道。他解釋說
- 你已經有了這個巨大的願景，比你可能建造的還要多
- 而在一份新聞稿中，你只有 30 秒的時間來吸引別人的注意。只有最重要的想法才會被關注

所以 Fadell 先寫了新聞稿
- 簡化、精簡的設想是評判 Nest 的標準
- 如果有一個新的功能想法不適合，它就會被砍掉，至少現在是這樣。新聞發布稿是需要發布的內容，不多，也不少


然後是包裝。有了對 Nest 的宣傳
- 團隊的下一件事就是製作包裝盒
- Fadell說：「包裝引領了一切。盒子的物理限制迫使我們把我們想讓人們了解的第一、第二、第三方面的內容準確地歸納出來」
- 你從一頁的新聞稿到幾平方英寸，從句子到摘要。在這一過程中，你完善了你的思維，直到你把產品的本質提煉為你會從貨架上拿起併購買的東西

Jeff Bezos 著名的是堅持每次開會都要先寫4頁的備忘錄，而不是 PPT，Bezos 在一封內部郵件中寫:
- 「寫一份好的4頁備忘錄比'寫'一份20頁的 powerpoint 更難」
- 「因為一份好的備忘錄的敘述結構迫使人們更好地思考，更好地理解什麼比什麼更重要，以及事情是如何關聯的。」

Bezos 在一份可能是天書的 128 字備忘錄中寫道：
- 「所有團隊今後都將通過服務接口暴露他們的 data 和功能」
- 這是一個集中的想法，導致亞馬遜建立網絡服務，它將成為它自己最好的客戶，並將 AWS 變成今天的雲計算巨頭

這一傳統延續到了最新的 AWS 服務。亞馬遜 ML 工程師 Eugene Yan 說
- 當亞馬遜著手建立一個新的服務時，「在編寫任何代碼之前，工程師們花了 18 個月的時間編寫關於如何最好地服務客戶的文件」
- 一旦他們開始編碼，他們就會清楚地知道誰會使用該產品，他們為什麼要使用它，以及他們預計該服務將提供什麼

## Write. Rewrite. Revise. Then do.
Swift 在幾十年前寫道「寫作是一種發現的方式」，但他並沒有就此打住，Swift 繼續說，「重寫，是改善思維的關鍵。」  
- 在寫和重寫你的備忘錄，你的想法時，你把你的思考和寫作一起完善
- 如果你在開始建設之前，在寫下第一行代碼或鋪設第一塊磚頭之前，就把這些思考弄清楚了，你就會有一個更好的機會創造出偉大的東西。你也會有很好的文件，從第一天起就準備好幫助你的客戶。