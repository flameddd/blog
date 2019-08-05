## DDD Community 
### 2019 05 04

### Session 1 : 領域驅動設計 by Clark
- 微軟 MVP
- .net 高手

對他來說，DDD 是個開發論
- 從系統的核心來看，系統要提供哪些功能？

TDD
- 從外面往裡面看，系統要提供哪些功能？

blog
- 昏睡領域

分析設計都是假的
- 工程師自己變魔術生出來的
- 很多時候，是 SA 就沒弄清楚

#### 問題在哪邊
- 缺少開發流程
- 前面一關的產出，沒辦法給下一棒接著做
- 直到開發為止，之前都是 paper work

缺少統一的架構

#### 建立有效的開發流程 ＝＞ DDD
用專案滾你的架構  
這樣是時間累積出來的架構  
- 跟業務說，倒不如你帶我出去跟客戶談

#### 你肯踏出去嗎
你跨出去的話
- 你就不只是工程師了
- 你是顧問

你要注意哪些？
- 出去談的時候，你就先要有預期想好，你要做哪寫
- 客戶談其他功能的時候，你就可以跟對方說，你這個東西不是我們業務的範圍
  - 可能要另外談
- 這樣保持 focus 自己在業務問題上

#### 需求訪談的時候
- 問客戶，你要的這些頁面，到底面對你的什麼問題？
- 這樣找出客戶的問題，並利用自己專業來想
- 把自已從 RD 轉為顧問

從 RD 轉顧問時，你推東西（給客戶）時，很好推

#### 使用者故事對照 (博客來有書、封面一隻鳥)
- 全局瀏覽
- 餐廳營運 <- 要先寫下來！這就是標的
  - 客戶談停車場管理時，反問 這不太像是我們的範圍，妳要談嗎？（付錢，就談）

這個餐廳圖，很像 information architecture  
- 第一個是標題
- 藍色的是「故事股幹」
- 下面的就是 event，小故事
  - 這些小故事，應該要有「企業價值」
- 要知道「誰」、「在什麼地方」、「做什麼事情」

#### 系統分析
##### 1. 網路拓樸
網路拓樸最先要出來  
用系統來改變流程  

現實一點，你推 IoT、推平板
- 客人比較容易付錢
- 用硬體帶軟體，比較好帶價格
- 軟體常常被算 0 元，server 掛 50 萬，客人乖乖付錢
- 不然你只有網頁，客人就覺得沒什麼

#### 2.使用案例
要把「who」寫出來，誰來操作什麼功能

#### 3.prototype
解決方案的操作流程

#### 4.工作清單
上面的 event 就是功能了，列出所有工作項目  
描述時，每一句都會有主詞（who）、行為（動詞）  
最重要，一定要有預估人天 man/day  
有「[]」誇起來的，就時節點，平板、電腦等等  
抓名詞，就是資訊  

Man/Day 才能轉換成報價！  
業務看對方穿著  
Man/Day 一天算 7500 ~ 10000 塊！  
六日、還是領薪水  
辦公室也算錢   

列出總額後，還會乘上風險值跟加上利潤  
利潤 * 15%  
風險值，這客戶有夠煩，之前合作過，那我 * 30% 

#### DDD
有功能清單過後，接下來有領域  
抓名詞，就是資訊  
這樣就能抓出 DDD 的領域模型  
不牽扯程式，讓你腦袋想，到底有沒有符合你所想的需求

#### 詞彙表（這很重要）
統一領域模型的英文詞彙   
統一出來  
做某個產業時，要去累積那個產業的詞彙表  
不能每個工程師自己想一個英文詞

座位：Seat
顧客：Customer

#### 領域物件
領域模型、領域詞彙，轉為領域物件  
這個就跟程式相關了  
- 切割 entity
- 到底有多大的物件要進入 DB

### 我的LinkedIn 三年開發遊歷 by Ian Tsai (ZanyKing)
三年在灣區的經驗來分享  
LinkedIn 的技術架構是什麼？  
都是講 open source 的專案  
Service infra  

#### micro servicer
- 盡可能的分拆功能，每個服務都活在不同節點上

在整理這些問題、跟監控時，會遇到很多問題  
水很深的  
例如
- 你不能 share library ，這在 micro servicer 行不同，會死很快

#### code base archive
- 程式庫管理格式 `Multi-product` (就是一個 folder)
- 1 MP : 1Git Repo
- Dependency & Containers Config:production-spec.json

不同型態的服務，就 1MP : 1 git repo
不喜歡把不同服務，塞到一個 MP 裡面  
你亂塞的情況，通常是要 share code

#### code review
- git + `Review board`

review board 是商業軟體  
跟 github, gitlab 的 diff 很像  

就算只有一行，linkedin 也要有 review 才能 back master

#### 原始碼搜尋系統
- 一個超過 40 人的團隊來維護的功能，一個搜尋引擎
- LinkedIn 的 repos 有 10000+

問題是
- Share libs 哪邊被用到？
- Config file 被誰呼叫了？
- 今天 EU 發生 GDPR 了，該怎麼找到地方來修改呢？（EU 罰你營業額 4%）
- 今天被 MS 給 Bing 了，要怎改？
  - Bing 要求，你所有有關地區的儲存格式，都存 Bing 的格式
- JARVIS `go/codeSearch`
  - indexing
  - Querying

codeSearch 真正複雜的是，要怎麼排出來
- A, B, C 相互沒有依賴，但都有一個叫做 master 的成員
- 我要怎麼只搜出 A 的 master 呢？

最後不行的時候，靠 elasterSearch

其他能考慮的 這些都是 Enterprise Code Search 的方案
- OpenGrok
- Zoekt

今天走入 micro service  
- repo 會急數成長

所以這是很重要的 infro

#### 監控
真正難的是
- 基於數計產生圖片
- 基於圖表來設定 threadhold, alert

LinkedIn 的 open source
搭配 `iris` + Oncall 制定管理 (sns, email)

#### unified Data Exchages format
所有的 data transfor 的 object 
- 統一規定、強制要求
- follow `Avro Schema` <- 因為這可以跑 validator，檢查能不能前後相容
- 系統一定是向前、向後相容，新系統，舊格式要能處理、舊系統，新格式要能處理
  - 這就牽涉到，怎麼設計 format

舊系統，吃到新格式時，資料不能掉，要能繼續往下傳 因為很多 obj 會一直往下傳
這個東西在 micro servicer 很重要
- 很容易遇到，一更新上新版，其他服務就沒辦法合作了

### Restful container 
link 選用這套來產生 restful code
- `Restli`，code gen 產出來的 function
- 強制要求只能用這個，這樣才能保證向前、向後相容

### Tracing System call tree
- 每一次要 call 前，要先有 call tree ID
- 只要這個 call 是 block 的（要等 response 才能往下的）
  - 這個 call tree ID 一定要往下帶
  - 這樣才能知道中間哪個 service 死掉的，每台 service 就能用這個 ip 發送 log

other sol
- `Zinkin`

### Dynamic Discovery
Context Aware: Cluster sticking on memberId, ContractId


我不知道是誰要收我這個 request
- 去問 zookeep 問誰要收，這時候才把 d2:// 轉成 url

### Event Queue: Kafka
在 micro service 很多要用 event 來完成
- 交易
- rollback

When to use?
- Derived Biz Model update (new role assignment, new email or Phone number)

（很多時候會搭配 zookeep?）
光是要管理
- 所有 Kafka 的節點

就需要一個 manager (confluent)
- 這就需要 20+ 以上的 RD 在管理了
- 這是巨型公司會需要的

### linkedIn
上面這麼多東西，都是 link 為了實踐 micro service 所做的功
- 這些都是基礎的
- 你可以有窮人版的 sol
- 不然你乾脆別拆，直接用很強的機器處理

### Ian 的心得
#### review board 
很好的產品，使用流程非常順
搭配 code serch engine 整合起來會更好

像 google 很多東西，都不需要開 IDE 了，直接 Browser 修改，就能開 ticket 了

#### R2D2
link 的 micro service 能做得好，是因為這套架構做得很好
SOA 架構 (service ori architecture)

#### Kafka
用起來簡單，一但大了 管起來很難

confluent 的版本現在還是最好的

#### 小總結技術架構
- SOA Compliant
- DTO share libs 嚴重綁定 java

#### CI/CD
- stage 超長

今天寫一段程式碼
1. 2位 PO review
2. 如果有關係 DB or Service 需要 Data model Review Committee
  - 需要領號碼牌...，像看病...有 office hour
3. new API exposed to Internet 的話，需要 Security review - Penetration test
4. new function impl: ACL approval to all downstream servicers
  - 一個一個去找那個 downstream servicer 的 PO，你要請他簽過，每一個都是一張 issue
5. verifying GDPR comliance
  - 最痛苦的，每一個新的 DB 都要檢查

GDPR 檢核的工具，也是個 30+ 的 team 來處理的

After Approval
1. Make sure shared Staging ENV is ready (可能其他 team 正在調整)
2. Using web UI tool
3. test it in staging
4. semi-auto push it to Canary `go/EKG`
5. (after Canary all green）release to production env

上金絲雀後，開始會有數據  
拿這些數據，去跟還沒 update 的機器比較  
有沒有流量 double 等等
（java JVM 要暖身，等一下才收集資料）

#### start yout own Multi production
##### In your team
- Design Doc
- Production team Design Review meeting
- Product SRE team acceptance
....

這邊太多了，到時候看 PTT

#### 個人心得
這些 CI/CD 開發流程真的真的很痛苦  
When you MUST go through?
- Service API Schema change
- DB Schema change

收費員只能對設計合理性的粗略審查  
link 甚至有被某國軍隊資安攻擊  

### 從 Micro service 看產品
- micro service 是個非常貴的方案
- 這是一個演化過後的方案

有時候，看 repo 就能知道你是不是 micro service
- XXX-frontend
- XXX-backend

XXX 是 team name or prodcution name 的話  
8成是 3層（就不是 micro service）  
這樣開，我才可以降低開 repo 的成本  

如果你要 micro service
- 要盡可能把 開 repo 的成本降低
- 步驟能自動化，就一定要自動化


new:
- XXX-web
- XXX-api
- XXX-mt (middle tier)
- XXX-backend
- XXX-samza
- XXX-common

開 repo 需要三張投影片、多個步驟才能開  
上面這些都是 anti pattern  
都是流程搞砸的結果  

建議是 team name, service name

- we6-frontend

#### 造成 distri Monolith 的成因
今天新多一個 service 的成本太高了  
多一個，可能要多個 team  

#### Knowledge manager
超過 50+ 時，就要有這東西了  
過1年就很痛了  
穀倉效應就出來了  
forum 的重要性（it's very good Q, I found the ANS not only benefit to you but else. Pls ask on forum, I will rely in 5 mins)
- slack, email, office hours 的知識流通率極低

#### others
積極透過程式碼一致性提升生產力： Java, Avro
