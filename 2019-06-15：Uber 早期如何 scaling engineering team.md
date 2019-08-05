### Uber 早期如何 scaling engineering team
### Gergely Orosz 03 OCTOBER 2018
#### https://blog.pragmaticengineer.com/scaling-engineering-teams-via-writing-things-down-rfcs/

Gergely Orosz 分享在 Uber 的 best practice  

當像微軟這樣的大公司或像Skyscanner這樣的小公司工作時，有兩件與計劃相關的事情總是讓我煩惱
- 首先，他人建立或與我的團隊建立相同的東西缺乏可見性
- 其次，由於不同的團隊以非常不同的方式構建事物而產生的技術和架構債務，無論是方法還是質量方面。

如果我說有一種方法可以很好地解決這兩個問題，使用幾個簡單的步驟怎麼辦？
(警告，其中步驟聽起來有點瘋狂）  

#### 1. Do planning before building something new
在構建新事物之前做好計劃
- 這可以是面對面的白板討論
- or just talking it through with the team members

只要你清楚知道如何完成工作

#### Capture this plan in a short、寫下文件
團隊一旦清楚 **how you do** 以及 **what yout do**，就應該相對快速地寫下
- "how". Don't go overboard.


#### 開始工作之前 選 (few) 幾個人來 approve this plan
就像 PR 要有人 review 過後、對 code quality 有保證後，才能 merge
- 在開始項目之前，一些相關人員驗證計劃的工作，則會產生巨大差異。 
  - 這些人可以是 senior engineers、團隊中將使用該功能的人員等等。

#### 把 planning document mail 給公司的所有工程師
- 並且讓任何人和每個人對其進行評論

是的，這可能聽起來很瘋狂。

#### 讓每人都按照上述步驟
- for every project that is beyond some super trivial complexity
- and iterate on a lightweight RFC-like process

##### RFC 請求意見稿
Request For Comments (RFC）是由網際網路工程任務組（IETF）發布的一系列備忘錄。檔案收集了有關網際網路相關資訊，以及UNIX和網際網路社群的軟體檔案，以編號排定。目前RFC檔案是由網際網路協會（ISOC）贊助發行。

聽起來不太可能，上述過程既可以工作也可以很好地擴展，從少數工程師到數千人團隊
- 它不僅解決了關於**可見性**或**減少技術/架構債務**的問題
- 還解決了傳播知識和讓工程師日常參與的問題

這是我建議任何中小型技術團隊做的一個簡單的過程，特別是如果他們處於成長階段


### 把事情寫下來的力量
- 寫作和與他人分享寫作會產生責任感
- 它也幾乎總能導致更徹底的決策

提高代碼質量的簡單方法？
- 在合併之前，請以書面形式進行代碼審查

舉行會議的簡單方法是減少浪費時間？
- 在會議之前有書面議程
- 然後寫下來
- 然後發出決定和行動

一個簡單的方法來運行具有較少意外的項目？ 讓團隊記下他們計劃做的事情並與他人分享。  


工程師討厭時間浪費
- document 通常被視為浪費時間之一
- 主要是因為它很無聊

因此有一種自然趨勢是跳過這一步以提高效率
- 不過，我要推翻「節省時間」這論點



如果每個人都同意如何完成項目，那麼寫下這個方法應該是件小事
- 文件只需抓到對方法的共同理解
- 將要進行的架構更改
- 以及棘手的部分

團隊中的某個人應該能夠在幾個小時內完成此任務，其他團隊成員在閱讀完之後會豎起大拇指  
通常
- 我認為這不那麼簡單
- “這不是我們談話時的意思
- 或者“我們忘記了這個重要的邊緣情況怎麼樣？
- “如果我們在這裡改變它，它可以打破系統的其他部分”

是在寫計劃時經常出現的事情
- 在項目完成一半時，進行這些討論是很棒的

當有人不同意 project 該被如何完成 或者有很多改動時
- 寫下來，至少能給出更 clearer picture

### Reviewers and spreading knowledge across the organization

document 寫給別人時，是不太一樣的
- 最好指定誰需要閱讀這份 document
  - 並且要寫到最後這些人看完 document 後，會豎起大拇指
- 最可靠的確保人有閱讀這 document 的方法是以書面形式
  - 例如 一條評論
  
mail 給所有 engineering organization 聽起來很誇張
- 有些人會擔心這產生太多干擾
- 干擾肯定會增加一些，但 mail 過濾很簡單就能處理
- 還有，當 RD 團隊到數千人的規模時，project 項目其實比想像中的少很多

這樣推 information 到全公司，對於塑造文化有極大的幫助
- 通過電子郵件
  - 並邀請任何人發表評論，樹立信任和責任感
  
最後
- 允許任何人和每個人都參與進來是整個組織中保持一致的 consistent engineering bar 的關鍵部分

在Uber，我已經看到多個案例人員意識到組織另一方的另一個團隊正計劃做一些與他們所做的類似的事情  
但採用的方法卻截然不同，往往是不同的品質  
例如  
- 建立新功能的美國團隊可能沒有考慮過世界其他地區
- 印度的團隊也指出了他們的本地化方法存在差距

具有透明度，自我平衡和自我糾正的團隊很自然地發生

### Tailoring the process via iteration
通過迭代調整流程

隨著公司的成長和成熟，這一規劃和審核流程如何運作的細節已得到完善
- 最初版本的 mail 已經進化為針對不同 domain (backend, mobile, web) 專屬的模板
  - 以幫助更一致的方式傳達信息
- 更多的工具正在構建，以使搜索和批准過程變得更加容易。


我見過的迭代的一個有趣部分是模板的演變  
審核許多工程提案的人通常會遇到相同類型的問題。 諸如
- “做這項工作的動機是什麼？”之類的問題
- 或“這將如何測試？”
- 或“這裡會有任何改變嗎？”

是非常常見的問題。 看到重複這些，工程師提出了不斷更新的模板，使閱讀和編寫這些計劃變得容易。 舉一個例子，這大致是一年前後端和前端（移動/網絡）模板的演變，從我們切換到生成的模板時：

#### Backend
- List of approvers
- Abstract (what is the project about?)
- Architecture changes
- Service SLAs
- Service dependencies
- Load & performance testing
- Multi data-center concerns
- Security considerations
- Testing & rollout
- Metrics & monitoring
- Customer support considerations

#### Mobile/Web
- List of approvers
- Abstract (what is the project about?)
- UI & UX
- Architecture changes
- Network interactions detailed
- Library dependencies
- Security concerns
- Testing & rollout
- Analytics
- Customer support considerations
- Accessiblity


Iterating and customizing 達到工程團隊的需求是關鍵  
在我們的例子中，模板開始包含我們關心的重要事項
- 可靠性
- 規模
- 安全性等

在Uber，我們已經構建了許多小型服務，因此在執行此類操作時需要考慮的事項（如負載和性能測試或SLA）是該域的一部分。 當可訪問性成為移動設備的一個重點時，該部分將其作為模板。你明白了


### A process that scales
Uber 用 RFC (跟傳統的 RFC 相似)

為了防止這些過於理論化，可以看我們的一些開源項目（例如BaseUI）來看到我們的RFC過程是公開發生的。 BaseUI是一個由現代，響應，生活組件組成的網絡設計系統
- 所有RFC均在此處和此處發布和審核
- 開發僅在RFC批准後開始


像谷歌，Facebook，微軟或亞馬遜等大型科技公司都提出了類似於某種程度的RFC流程的東西  
對於所有公司而言
- document 應該是多麼徹底
- document 被推送到誰
- 如何嚴格審查或者在不同組織中如何標準化的比例都是不同的

如果你的科技公司仍在計劃中，而且沒有結構化的方式來記錄/分發它，那麼有助於工程規模化的一件事就是儘早建立一種RFC流程。 試一試，然後重複一遍如何做。 這就是它總是如何開始。
