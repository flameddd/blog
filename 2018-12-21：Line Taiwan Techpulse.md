## Line 開發者大會
這次主題是「互動」  
原來是第三年舉辦了。  
今年有5場開發者小聚、還有 hackton。  
還有新創團隊計畫，Line protostar  
說是有「技術驚喜」阿～彩蛋？ 
   
## 技術策略佈局與跨國產品開發
下一個階段的發展方向、工作模式、技術佈局
 - 串連
 - 互惠生態圈

除了線上，也積極強化線下服務
 - 探索 更容易、更方便的找到想要的服務（新聞搜索
 - 人性化操作 語音辨識等等

AI
 - 判斷用戶的檢舉不當內容

### 互惠生態圈
 - open API 讓大家串來使用 line 功能。
 - 

Clover 到底是什麼？
有一款式 Charbot Engine 
可以訓練內容
Clova AI Tech Demo （都還沒開放，只能玩玩而已）

### Link Chain
解決效能問題？他沒講怎麼做。  
有出 line token。

### How is the cost？
明年Q2 推出官方帳號2.0、計價更彈性

###
 - Flex Messages 更好的訊息格式
 - video imagemap 支援影音的訊息格式
 - Quicj Reply 快速的回答方式。
 - Life 互動介面的優化，可以把互動放在裡面
 - 透過 line login 的方式綁定 user
 - account link 也可以綁定 user
 - 平台可以用 Matching 來 delieve 訊息，電話加密後比對，就會根據你的 target 發送訊息
 - Switcher API 切換不同 endpoint。切換真人客服，切換過去。
 - 更近一步，用 Call API 直接進入語音對話的模式
 - ICon switcher API 讓 user 知道對象是誰？

### What's next
 - New Official Account 所有的 account 都有 message API
 - 以量計價
 - ChatBot Engine

### 一卡通
 - 朋友們轉帳
 - 分攤付款 (自動計算)
 - 邀請轉帳
 - 生活繳費（一卡通專屬）
 - 乘車碼（高雄捷運開放使用）
 - Line Wallet
 - Line Shopping、travel
 - 商家的話，就要申請 Line Pay、IPass
 - Line Pay 分線上、線下 API

### 
 - smart port 業務分類
 - 民眾都是用這個入口
 - 38萬人 block 市政府服務
 - 市府網站只停留 3 秒
 
### Line IoT
 - Why Line Things
 - Service Architecture
 - LIFE BLE

產品商不用寫 App ？我有點懷疑

### Line Link
 - 韓國講者，日本沒高手嗎？
 - 164M 的 User 在全球，這是很好的基礎
 - Design for dApps and Users  這句話什麼意思？看不出具體內容
 - Contribution is the new Mining
 - Connected Economy

### 
做個台灣的 q&a 區塊鏈？
除了一般的 q&a 專業型的呢？

### 國家災害防治中心
### scrum
 - 1 ~ 4 週都被 developer 消耗光了
 - 一來一往的 debug
 - 測試都需要重頭來

我們真的在跑敏捷嗎？還是短期 water fall 而已

 - 新功能一直再加、舊功能也有在 run，整合測試才會重要

QA 要盡量了解這些構面
 - code
 - architecture
 - change scrope

在這樣才能知道，影響範圍在哪邊？才知知道要測試的重點在哪裡

 - code coverage => 寫 code 的時候，品質控管就開始了
 - code review  => 不能超過 200 行
 - 使用原生的 ios android 的 test 工具、另外有實機測試

### k8s
 - Verda
 - Ranchant

### 報名報到的機器人
 - 報名的時候
 - 報導的時候
 - 活動期間

#### Entry
 - 一般來說都會先做網頁 桌機、手機版 把 user 從習慣的桌機 導回到手機 App
 - 加入 line 好友後，就會導到了，一進來就是 rich menu
 - 這個報名的流程算是很順暢
 - 讓所有的 活動資訊，都在 Line 裡面完成。
 - 報名時，是用 LIFE 來寫的  這次是半版（希望報導時，每次看的資訊很少
 - 提供給使用者單一、簡單的入口 建立 connect、容易推播給使用者

### chenk in  原來 Beacon 是 藍芽 4.0
 - Beacon + QRCode 兩招一起來
 - simple Beacon 開發者範例
 - 大型活動，可以跟 Line 合作

### Events 活動中
 - realtime information
 - interactive Booths
 - Questionnaire

#### real time information
 - Link 的 airdrop  這個可以讓講者能即時提供訊息
 - Flex Message 是 JSON 的 file 寫作

#### interactive Booth
 - 外面的遊戲 廣告大型看板的範例
 - smart city 

#### Questionnair
 - 

#### RECAP
 - LIFE
 - Beacons
 - Message Typs

### 日本講者 short speak 快速開發
  快速開發、很短時間，針對既有服務推出 App。推出 Today Service (現在是全方面新聞平台)。
主要是三個步驟  
 - Release Fast
 - Find Pain Points (Fix it)
 - Iterate

v1.0 => 介紹頁面、 login 頁面、web view 串後端內容  
v2.0 更豐富的內容  只增加了 bottom navigator  
v3.0 加強影音  用原生方式開發  

### Today adn Buzz
 - Today 提供各家新聞媒體
 - Buzz 對 content 有不同的方式呈現 （印尼那邊提供）
   這有點猛，讓 User 幫你做"使用者精選"

用 HBase 當 DB
考慮到
 - High wreite throughput
 - high 可擴充性

遇到的問題
 - 只有 row key 有 index
 - 沒有 transaction 

但我們需要 secondary index  
因為我們要排序 author or 熱門程度等等  
這些我們都需要排序  
當我們進入 User proile page 時，author 才是這時候的條件  
那就需要 full table scan 了，這樣很沒效  
所以要設計 secondary index  
也就是 Secondary Table 處理！  

KafKa 用來處理 交易 問題。  
不對噎，他們是想確保 task queue 跟管理 task、確保 task 完成。  

### 分享 Line Beacon 的應用
 - new digest 台北通勤時間約 50 mins
 - 捷運新聞報
 - COUPON => Beacon 很重要的特性是，知道 User 在哪個地點、位置
 - Puzzle game 拼圖遊戲，四片拼圖完成後 能有貼圖
 - Bus Info

不同的 Beacon 設置不同的任務
怎麼管理大量的 Beacon events
 - 所有 Beacon 都會測到 這位 user 進來，Server 會管理由哪個 Beacon 處理推送

會先把很多 User 的狀態，先分別放到 db 的 Cache 中。
 - 這位 User 有沒有同意過、上次有沒有經過這邊捷運站。

### 新創計畫
 - 企業戰略部 ！原來有這單位

分成三類問題
 - 我有很好的技術團隊，但我公司只有2個人 我該怎麼開發 資源不足
 - 我有很好的 bussiness modal ，也有 prototype 了，但接下來呢？行銷、分紅？如何成長？沒有導師
 - 已經證實 modal 可行，但是沒有合適的投資者、創投等等

提供
 - 免費的官方帳號、沒有好友上限
 - Message API push message 無上限
 - line login 等等
 - 最重要的是 技術顧問 ，會打從一開始就協助

Taix GO 如果沒有用 Line 會需要...
 - 客戶端 ios android 的 APP
 - 司機端 ios android 的 APP

四倍... 的節省  

 - 中華開發的專業經理人協助(含會計、法律事務所服務)
 - 教你怎麼募資

### 社群發展
 - API expexter
 - cool 有一個專屬的開發者部門
 - 有提供什麼資訊？
  - 所有的 document、SDK
  - Blog 有中文的？
  - github starter login、Things、SimpleBeacon
  - Line Hack2018 好激烈啊
  - Line Boot Awards 2018

