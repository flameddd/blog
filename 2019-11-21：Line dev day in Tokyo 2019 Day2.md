## Line dev day in Tokyo 2019 Day 2

https://linedevday.linecorp.com/jp/2019/sessions


### "LINE-like" Product Management of Smart Channel
- Daisuke Asai
- LINE Planning Department Deputy Senior Manager

slide
- https://speakerdeck.com/line_devday2019/line-like-product-management-of-smart-channel

一開始是聽別場，結果覺得太沒料。所以後來跑來這邊聽，聽到這場的後半部。  
這場就是當時 `Daisuke Asai` 被指派要做 Line 的廣告  

就是這個廣告  
![Line_ad](/assets/img/Line_ad.png)  

一開始他心裡是排斥反對的，因爲 User 最討厭廣告  
後來他自己告訴自己，先**別有預設立場**，看 data 說故事吧，所以先來收集資料  

![LineDevDay2019_Tokyo_Day2_125](/assets/img/LineDevDay2019_Tokyo_Day2_125.JPG)  
![LineDevDay2019_Tokyo_Day2_123](/assets/img/LineDevDay2019_Tokyo_Day2_123.JPG)  

調查過後，似乎 Uesr 不會非常厭煩
![LineDevDay2019_Tokyo_Day2_120](/assets/img/LineDevDay2019_Tokyo_Day2_120.JPG)  

也有先做初步 User 測試，找少數幾人來使用看看
![LineDevDay2019_Tokyo_Day2_115](/assets/img/LineDevDay2019_Tokyo_Day2_115.JPG)  

針對「不同的廣告內容」了解 User 的反應  
![LineDevDay2019_Tokyo_Day2_113](/assets/img/LineDevDay2019_Tokyo_Day2_113.JPG)  

這兩張好像是說，他們的廣告是根據 User 來特定投放的。  
那些內容要透過他們的 `smart channel` 來取得  
兩天的 Line dev day 中，不時有提到這個 `smart channel`  
看來是從各系統中的資料來集中、過濾整理的東西，再讓各系統可以取用  
應該是很複雜的架構系統  
![LineDevDay2019_Tokyo_Day2_112](/assets/img/LineDevDay2019_Tokyo_Day2_112.JPG)  
![LineDevDay2019_Tokyo_Day2_110](/assets/img/LineDevDay2019_Tokyo_Day2_110.JPG)  


雖然 Daisuke Asai 被上頭指派這任務時，也是頭大、也是抱怨，也是要硬著頭皮上  
單單看這個功能、這個專案而言，可以看到 Lien dev team 在做事情的深度
- 不是強推功能
- 好好做事前研究
- 也有 pre test
- 一步一步推展到其他地區

### Looking for a silver bullet to edit videos on Android.
她們應該花不少時間研究才有這次的結果  

slide
- https://speakerdeck.com/line_devday2019/looking-for-a-silver-bullet-to-edit-videos-on-android

why? 為什麼會有這個問題
- 想在 Line 上做影音編輯
- 各家 Android 手機、各種版本、各種 API 都有些差別
  - 導致編輯影片、編輯後的影片播放都有各種問題


![LineDevDay2019_Tokyo_Day2_106](/assets/img/LineDevDay2019_Tokyo_Day2_106.JPG)  
![LineDevDay2019_Tokyo_Day2_105](/assets/img/LineDevDay2019_Tokyo_Day2_105.JPG)  
![LineDevDay2019_Tokyo_Day2_99](/assets/img/LineDevDay2019_Tokyo_Day2_99.JPG)  

最一開始的解法是 `黑名單`  
- 但才沒多久後，就發現這 `黑名單` 很難維護、越來越多，而且這樣 Uesr 也享用不到功能

![LineDevDay2019_Tokyo_Day2_98](/assets/img/LineDevDay2019_Tokyo_Day2_98.JPG)  
![LineDevDay2019_Tokyo_Day2_95](/assets/img/LineDevDay2019_Tokyo_Day2_95.JPG)  

所以找其他方案
- 找到有人 Java 開源影音
- 等於不依賴手機內部的 lib 的意思

![LineDevDay2019_Tokyo_Day2_93](/assets/img/LineDevDay2019_Tokyo_Day2_93.JPG)  

後面都在講解細節
- 影片就是一張張圖片，所以轉回來圖片後，編輯，最後再轉回影片
- 其實遇到很多問題，影片轉向、影片品質跑掉、聲音不同步等等，只好一個個解決
  - 後面都在講一個個問題怎麼解決的

![LineDevDay2019_Tokyo_Day2_92](/assets/img/LineDevDay2019_Tokyo_Day2_92.JPG)  
![LineDevDay2019_Tokyo_Day2_90](/assets/img/LineDevDay2019_Tokyo_Day2_90.JPG)  
![LineDevDay2019_Tokyo_Day2_88](/assets/img/LineDevDay2019_Tokyo_Day2_88.JPG)  
![LineDevDay2019_Tokyo_Day2_87](/assets/img/LineDevDay2019_Tokyo_Day2_87.JPG)  
![LineDevDay2019_Tokyo_Day2_86](/assets/img/LineDevDay2019_Tokyo_Day2_86.JPG)  
![LineDevDay2019_Tokyo_Day2_85](/assets/img/LineDevDay2019_Tokyo_Day2_85.JPG)  
![LineDevDay2019_Tokyo_Day2_84](/assets/img/LineDevDay2019_Tokyo_Day2_84.JPG)  
![LineDevDay2019_Tokyo_Day2_1](/assets/img/LineDevDay2019_Tokyo_Day2_1.JPG)  


### LINE Design System: making LINE Product faster without losing consistency
slide
- https://speakerdeck.com/line_devday2019/line-design-system-making-line-product-faster-without-losing-consistency

這場講述 Line 走向 `Design System` 的一些過程  
先講一個大重點
- Line 花多少人來做 `Design System` 呢？ -> `30 人`...，30 我的天啊


![LineDevDay2019_Tokyo_Day2_83](/assets/img/LineDevDay2019_Tokyo_Day2_83.JPG)  

這張投影片有意思
- 他們說，`Design System` 是 design 演進的結果
- 是最新的解決 guide
- 以前我就只知道是個方法論，但沒想到是有這樣的過程！

![LineDevDay2019_Tokyo_Day2_81](/assets/img/LineDevDay2019_Tokyo_Day2_81.JPG)  

用 `atomic design` 的方式建構底層 component
- 講者介紹時，也是講 `atomic design` 這個說法

![LineDevDay2019_Tokyo_Day2_79](/assets/img/LineDevDay2019_Tokyo_Day2_79.JPG)  

希望獲得的改善有  
對 Planner
- 已 UX flow 為基礎下幫助決策（更快構思系統吧，這點我忘了主要在說什麼）
- 聚焦在 `business goal` 上面，UI 部份別在花更多時間
對 Design
- 提升 `consistancy`
- 加速產出（減少重複工作，例如重新設計 tabs, btn 之類的）
對 dev
- 聚焦`核心邏輯(business login)`（因為使用舊有元件來組合 UI，不需要重新設計)
- 提升 `consistancy` (因為都使用一樣的 component)
- 減少 `QA cost` (都使用舊的 component，這些之前都 test 過了，理論上不會有問題了)

![LineDevDay2019_Tokyo_Day2_78](/assets/img/LineDevDay2019_Tokyo_Day2_78.JPG)  
![LineDevDay2019_Tokyo_Day2_76](/assets/img/LineDevDay2019_Tokyo_Day2_76.JPG)  

整場 talk 舉例很多原先 Line `不 consistancy` 的情況  
可以看到 `tabs` 樣式一大堆  
![LineDevDay2019_Tokyo_Day2_74](/assets/img/LineDevDay2019_Tokyo_Day2_74.JPG)  

希望用 `Design System` 改善的問題  
![LineDevDay2019_Tokyo_Day2_73](/assets/img/LineDevDay2019_Tokyo_Day2_73.JPG)  

這張圖說明，採用 `Design System` 能縮短開發時間
- 不過我當時第一眼看圖，就覺得**這張圖在講幹話**...
  - 不過就是把長方形畫短一點 @@|||

真正要透過 `Design System` 達到此效果的話
- 真的要等 `Design System` 融入整個開發體系後，才比較能體現出來

![LineDevDay2019_Tokyo_Day2_72](/assets/img/LineDevDay2019_Tokyo_Day2_72.JPG)  

`Design System` 要做的事情  
![LineDevDay2019_Tokyo_Day2_70](/assets/img/LineDevDay2019_Tokyo_Day2_70.JPG)  

這邊有一個亮點，之前我沒看過這種觀點
- 先定義出自己團隊的 `Design Principles`
- 後續的設計(`Design System`)要確保有考慮這些 Principles
  - 很特別的是 `chat first`。他說，這顯而易見，因為我們是 IM 軟體～

![LineDevDay2019_Tokyo_Day2_68](/assets/img/LineDevDay2019_Tokyo_Day2_68.JPG)  

定義 `color palette`  
![LineDevDay2019_Tokyo_Day2_67](/assets/img/LineDevDay2019_Tokyo_Day2_67.JPG)  

`LDS` 應該是指 `Line Design System`  
這張圖表示說
- 以前 color 都是寫上代碼 
  - 寫代碼的方式，就算是種錯誤的寫法，會有 test 抓出來
- 現在都採用 `LDS 所定義的 color`

![LineDevDay2019_Tokyo_Day2_66](/assets/img/LineDevDay2019_Tokyo_Day2_66.JPG)  

`semantic color` 這個 term 應該是我第一次看到
- 但這個概念有看過，有看過些文章說明配色時提過
- 總之就針對 `semantic` 的地方特別選定一些 color
  - 例如一些 meta data 地方可能會比較淺黑色

![LineDevDay2019_Tokyo_Day2_65](/assets/img/LineDevDay2019_Tokyo_Day2_65.JPG)  
![LineDevDay2019_Tokyo_Day2_64](/assets/img/LineDevDay2019_Tokyo_Day2_64.JPG)  

這個 a11y 的觀念我一直沒有很懂
- 在 jsconf japan 的 a11y talk 應該也是提到這個
- 但對於這個`比例數字`要怎麼真正運應，我實在不懂
  - https://www.canva.com/design/DADnrZSkJec/OnMFRrcsA2ybW0zzI3-IqA/view?utm_content=DADnrZSkJec&utm_campaign=designshare&utm_medium=link&utm_source=publishsharelink#25

![LineDevDay2019_Tokyo_Day2_63](/assets/img/LineDevDay2019_Tokyo_Day2_63.JPG)  
![LineDevDay2019_Tokyo_Day2_62](/assets/img/LineDevDay2019_Tokyo_Day2_62.JPG)  

`atomic design` 的 component  
![LineDevDay2019_Tokyo_Day2_60](/assets/img/LineDevDay2019_Tokyo_Day2_60.JPG)  
![LineDevDay2019_Tokyo_Day2_58](/assets/img/LineDevDay2019_Tokyo_Day2_58.JPG)  

有了這些 component，就能快速 `prototyping`  
![LineDevDay2019_Tokyo_Day2_57](/assets/img/LineDevDay2019_Tokyo_Day2_57.JPG)  

內部也有 `LDS 的 Demo App`，讓設計、開發能快速看 on device 的效果  
![LineDevDay2019_Tokyo_Day2_56](/assets/img/LineDevDay2019_Tokyo_Day2_56.JPG)  

一些可以互動的元件，設計上要考慮的原則
- 不過聽聽而已，聽不出內容

![LineDevDay2019_Tokyo_Day2_55](/assets/img/LineDevDay2019_Tokyo_Day2_55.JPG)  

這個是說明類似 `animation` 的東西
- 之前也都有不一致
- 統一由 `Design system` 統一設計規定

![LineDevDay2019_Tokyo_Day2_54](/assets/img/LineDevDay2019_Tokyo_Day2_54.JPG)  

也有提供 `code snippet` 給 develop 使用  
![LineDevDay2019_Tokyo_Day2_52](/assets/img/LineDevDay2019_Tokyo_Day2_52.JPG)  

線上文檔當然也都有  
![LineDevDay2019_Tokyo_Day2_51](/assets/img/LineDevDay2019_Tokyo_Day2_51.JPG)  


### Front-end in Design Systems
slide
- https://speakerdeck.com/line_devday2019/front-end-in-design-system

`Front-end in Design Systems` 是 `LINE Design System` 的延續主題
- Line 的 frontend 有 160 人...
- 每年 frontend team 有一次大聚會

![LineDevDay2019_Tokyo_Day2_50](/assets/img/LineDevDay2019_Tokyo_Day2_50.JPG)  
![LineDevDay2019_Tokyo_Day2_49](/assets/img/LineDevDay2019_Tokyo_Day2_49.JPG)  

frontend team 的資深分佈  
![LineDevDay2019_Tokyo_Day2_47](/assets/img/LineDevDay2019_Tokyo_Day2_47.JPG)  

UI framework 的使用分佈  
![LineDevDay2019_Tokyo_Day2_46](/assets/img/LineDevDay2019_Tokyo_Day2_46.JPG)  

frontend framework 使用分佈
- 這邊跟 Yahoo Japan 有一點一樣，就是`這兩年左右內部就漸漸停用 angular` 了
  - 頂多剩下一些 `angular` 老系統要維護
  - 而且都是 `vuejs` 稍微領先 `react` 一些

![LineDevDay2019_Tokyo_Day2_45](/assets/img/LineDevDay2019_Tokyo_Day2_45.JPG)  

`vuejs` 這邊，針對 Bootstrap 有做 component  
![LineDevDay2019_Tokyo_Day2_44](/assets/img/LineDevDay2019_Tokyo_Day2_44.JPG)  
![LineDevDay2019_Tokyo_Day2_43](/assets/img/LineDevDay2019_Tokyo_Day2_43.JPG)  

`Icon` 方面也是一堆`不 consistancy` 的情況
- 樣子不一致
- 命名也不一致

![LineDevDay2019_Tokyo_Day2_42](/assets/img/LineDevDay2019_Tokyo_Day2_42.JPG)  

`命名`方面
- **不在揣測行為**
- 而是單純**依據 Icon 樣式**就好

![LineDevDay2019_Tokyo_Day2_40](/assets/img/LineDevDay2019_Tokyo_Day2_40.JPG)  
![LineDevDay2019_Tokyo_Day2_39](/assets/img/LineDevDay2019_Tokyo_Day2_39.JPG)  
![LineDevDay2019_Tokyo_Day2_38](/assets/img/LineDevDay2019_Tokyo_Day2_38.JPG)  

但是，因為有不同的實作方法
- 所以使用方法也都不同

後面帶出，為了要解決這問題，所以採用 `Web Component`

![LineDevDay2019_Tokyo_Day2_37](/assets/img/LineDevDay2019_Tokyo_Day2_37.JPG)  

現在 and 未來要走的部屬流程
- 希望每位設計師獨立改完，經過 review 後，就能順利推出  

![LineDevDay2019_Tokyo_Day2_36](/assets/img/LineDevDay2019_Tokyo_Day2_36.JPG)  
![LineDevDay2019_Tokyo_Day2_35](/assets/img/LineDevDay2019_Tokyo_Day2_35.JPG)  

Icon 的一些制定標準  
![LineDevDay2019_Tokyo_Day2_34](/assets/img/LineDevDay2019_Tokyo_Day2_34.JPG)  
![LineDevDay2019_Tokyo_Day2_33](/assets/img/LineDevDay2019_Tokyo_Day2_33.JPG)  
![LineDevDay2019_Tokyo_Day2_32](/assets/img/LineDevDay2019_Tokyo_Day2_32.JPG)  

這個我不太懂，總之是某種產生各種 css color 顏色的做法
- 應該是說，不要靠 pre-processor 處理，而是事前 or 某種引入來完成

![LineDevDay2019_Tokyo_Day2_31](/assets/img/LineDevDay2019_Tokyo_Day2_31.JPG)  
![LineDevDay2019_Tokyo_Day2_30](/assets/img/LineDevDay2019_Tokyo_Day2_30.JPG)  

RWD 的 width 界定範圍
- 另外這陣子，有看到別人界定這東西時，連 Apple Watch 也規範進去...，真的需要嗎 = =|||

![LineDevDay2019_Tokyo_Day2_29](/assets/img/LineDevDay2019_Tokyo_Day2_29.JPG)  

這邊就是開始提，因為 Line 內部使用不同的 framework
- 各自元件不能互用，你要寫三遍
- 所以改用 `Web Component` 來解決

![LineDevDay2019_Tokyo_Day2_27](/assets/img/LineDevDay2019_Tokyo_Day2_27.JPG)  

`30 人`，夭壽，30 人來搞 `Design System`
- 難怪成果似乎很不錯
- 圖上可以注意到，有分 `UI Designer` 跟 `UX Designer` ~

![LineDevDay2019_Tokyo_Day2_26](/assets/img/LineDevDay2019_Tokyo_Day2_26.JPG)  
![LineDevDay2019_Tokyo_Day2_24](/assets/img/LineDevDay2019_Tokyo_Day2_24.JPG)  
![LineDevDay2019_Tokyo_Day2_23](/assets/img/LineDevDay2019_Tokyo_Day2_23.JPG)  

這投影片很重要的觀念，他們的經驗上，玩 `Design System` 時，你應該
- `Design System` 的 `Rule` 應該要**鬆散**，而不是嚴格定死每一細節
- `Parts` 有點忘記了，應該是指，不是所有的 Component 都一定要整合一起。
- 設計、規範、開發 `Design System` 的 `Team` 應該要**集中**，由一個的 team 來統一處理
  - Line 的組織架構上（上面第一張圖），有一個 `Frontend Standardization` 的 team。是專門規劃，制定 frontend team 的標準的。有趣（但好像沒有說 `Design System` 就完全是這 team 處理）

![LineDevDay2019_Tokyo_Day2_22](/assets/img/LineDevDay2019_Tokyo_Day2_22.JPG)  
![LineDevDay2019_Tokyo_Day2_21](/assets/img/LineDevDay2019_Tokyo_Day2_21.JPG)  
![LineDevDay2019_Tokyo_Day2_20](/assets/img/LineDevDay2019_Tokyo_Day2_20.JPG)  
![LineDevDay2019_Tokyo_Day2_19](/assets/img/LineDevDay2019_Tokyo_Day2_19.JPG)  


### Let’s experience super smart city leveraging LINE API
這場很不錯，看到 Line 為公家機關 local goverment 做的系統
- chatbot + 網頁的互動
- user 用 chatbot 來申請表單，拍自己照片證明身份
- 公務員用網頁處理，有問題的話
  - 網頁就能回覆對方
  - 網頁就能把「案件狀態」調整到某一個 stage
- 網站就是表單流程管理的工具
- 完全把 line chatbot 當 cliet 端來用

可惜 slide 沒有公佈出來的樣子  

網路上有看到別人也有寫這 talk 的整理，也值得看看
- https://medium.com/@EtrexKuo/line-dev-day-2019-lets-experience-super-smart-city-leveraging-line-api-213b4e0c5e55
```
@EtrexKuo: 
    我們可以知道他們的 CRM （資料庫的部分）用的是 salesforce 、主機用的是 Heroku 、 NLU 自然語言辨識用的是 Dialogflow 、 logger 用的是 Firestore 、前後文儲存用的是 Redis 、功能則是定義為 Skill 
```

這張圖，是他把 card 從 `受付` **拖拉**到 `修正`
- 這位市民就會收到通知

![LineDevDay2019_Tokyo_Day2_7](/assets/img/LineDevDay2019_Tokyo_Day2_7.JPG)  

右邊的網站介面，這邊可以輸入訊息給市民  
User 就會收到訊息

![LineDevDay2019_Tokyo_Day2_9](/assets/img/LineDevDay2019_Tokyo_Day2_9.JPG)  

這個是，當有些申請項目需要申請人的照片時
- line 那邊操作就會要你自已當下拍一張照片，然後上傳
- 網站這邊下面就會看到申請人的檔案

![LineDevDay2019_Tokyo_Day2_10](/assets/img/LineDevDay2019_Tokyo_Day2_10.JPG)  

這 bot 也串好了支付功能
- 印象中現場是 demo Line pay

![LineDevDay2019_Tokyo_Day2_11](/assets/img/LineDevDay2019_Tokyo_Day2_11.JPG)  
![LineDevDay2019_Tokyo_Day2_13](/assets/img/LineDevDay2019_Tokyo_Day2_13.JPG)  
![LineDevDay2019_Tokyo_Day2_14](/assets/img/LineDevDay2019_Tokyo_Day2_14.JPG)  
![LineDevDay2019_Tokyo_Day2_16](/assets/img/LineDevDay2019_Tokyo_Day2_16.JPG)  
![LineDevDay2019_Tokyo_Day2_17](/assets/img/LineDevDay2019_Tokyo_Day2_17.JPG)  


### How do you get the right balance between business, technology, and creativity?
講太多廢話，10分種左右就聽不下去了
