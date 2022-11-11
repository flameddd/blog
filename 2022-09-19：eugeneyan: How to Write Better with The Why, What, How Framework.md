# 2022-09-19：eugeneyan: How to Write Better with The Why, What, How Framework
## eugeneyan, 2021-03-24
### https://eugeneyan.com/writing/writing-docs-why-what-how/

之前就有聽說過 AWS 裏面，啟動產品時，會先寫文件  
這是另一個寫關於這件事，所以讀看看  

另一篇:
- [Werner Vogels (CTO - Amazon.com): Working Backwards](2020-02-16：Working&#32;Backwards.md)

------------------------------------

這是 AWS **早期的故事**
- 在寫任何 code 之前，工程師們花了 18 個月的時間來思考和編寫關於如何最好地服務客戶的文件
- 亞馬遜認為這是最快的工作方式，深入思考客戶的需求，然後再嚴格執行這個完善的願景

同樣，作為一名數據科學家，雖然通過程式解決問題
- 但很多工作都是在編寫任何程式之前發生的。這種工作的形式是思考和/或通過編寫 docs
- 在以寫作文化而聞名的亞馬遜，情況更是如此

eugeneyan 將分享寫過的三種文件：`one-pagers`、`design documents` 和行動後的回顧(after action reviews)  
- 這篇文章將揭示用來組織 eugeneyan 大部分寫作的框架
- 在下一篇文章中，將討論 `design docs` (How to write design documents for data science/machine learning projects?)
  - https://eugeneyan.com/writing/ml-design-docs/


![](https://eugeneyan.com/assets/work-docs.jpg)  

## 單頁文件(`One-pagers`)、設計文件(`design docs`)、事後回顧(`after-action reviews`)
在建立/運作一個系統時，通常會寫三種類型的文件
- 前兩種有助於獲得一致性和反饋
- 最後一種用於反思
- 這三種都有助於深入思考和改善結果

![](https://eugeneyan.com/assets/work-docs.jpg)  

**單頁文件(`One-pagers`)**:
- 用這些來實現與業務/產品利益相關者的一致
- 也用來作為季度/年度優先次序的背景備忘錄
- 在 `One-pagers` 中，它應該讓讀者迅速了解問題、預期結果、建議的解決方案和高層次的方法
- 當你深陷專案的雜亂中，或遇到範圍改變時，參考這些文件是非常有用的

**設計文件(`design docs`)**:
- 使用這些文件來獲得來自科學家和工程師同事的反饋
- 它有助於在過程的早期識別設計問題
- 此外，可以在設計文件上比在系統上更快地進行迭代，特別是當上述系統已經在 production 中時
- 它通常涵蓋方法論和系統設計，並包括實驗結果和技術基準（如果有的話）

`design docs` 在 engineering projects 中更常見
- 而在 science/machine learning 中則不多
- 儘管如此，我發現它對於建立更好的 ML 系統和產品是非常有價值的

**事後回顧(`after-action reviews`)**:
- 用這個來反映在一個專案 release 後，或在一個重大錯誤後
- 如果是專案回顧，我們會討論哪些地方做得好（和哪些地方做得不好），後續行動，以及下次如何做得更好
- 就像 Scrum retrospective，只是有更多的時間來思考，並寫成文件。然後，可以與其他團隊分享

如果是錯誤回顧（例如，系統癱瘓）
- 我們會診斷出根本原因，並確定後續行動以防止再次發生
- 不會去責備任何一個人
- 其目的是討論我們可以做得更好，並與更大的組織分享（有時是痛苦的）教訓。亞馬遜稱這些為 [Correction of Errors](https://wa.aws.amazon.com/wat.concept.coe.en.html)
  - Correction of Errors 舉例: https://github.com/JDHarris007/coe/blob/master/CoE.md


## 寫作框架: Why, What, How, (Who)
`Why-What-How`，聽起來像是一年級學生的閱讀/寫作課
- 然而，它指導著我大部分工作文件
- 我在這個網站上的寫作也遵循它，如
  - [A Practical Guide to Maintaining Machine Learning in Production](https://eugeneyan.com/writing/practical-guide-to-maintaining-machine-learning/)
  - [Growing and Running Your Data Science Team](https://eugeneyan.com/writing/data-science-teams/)

**為什麼(Why)**:
- 首先解釋為什麼文件很重要
- 這通常是圍繞著我們想要解決的問題或機會，以及預期的好處
- 也可以回答 `Why now` 問題

把這看作是你文件的鉤子。在閱讀了 `Why` 後:
- 讀者應該感到有必要閱讀文件的其餘部分（並希望能對你的建議作出承諾）
- 在資源緊張的環境中（如，startups），這部分要說服決策者為你的想法投入資源


因此，至關重要的是，在閱讀這一部分後
- 你的聽眾要理解問題和背景
- 用他們的術語簡單描述: 客戶利益、業務收益、生產力的提高

對比下面的兩個 Whys，哪個更適合商業受眾？
> 1. 我們需要採購GPU clusters，用於 SOTA 深度學習模型的分佈式訓練，將 nDCG@10 提高20%。
> 2. 我們需要投資基礎設施，以改善客戶推薦，預計轉換和收入提升 5%。


第一個可能有點誇張，但我見過像這樣開頭的 Why
- 這是個**從一開始就失去觀眾的好方法**

**What:**
- 在聽眾相信我們應該解決問題之後，分享一個好的解決方案是什麼樣子的
- 預期的結果是什麼以及衡量它們的方法

構建 `What` 的一種方式是
- 通過「成功的衡量標準」和「約束條件」
- 「成功的衡量標准」定義了一個好的（或壞的）解決方案是什麼樣子
- 「約束條件」定義了解決方案可以（或不能）做什麼
- 它們一起使讀者能夠評估和決定提案，進行權衡，並提供反饋。

另一種方法是通過「需求」來描述
- 業務需求規定了預期的客戶體驗、對業務指標的提升（成功措施）和預算（限制）
- 它們也可以被稱為產品或功能需求
  - 技術要求規定了吞吐量、延遲、安全、隱私等，通常作為約束條件

**如何實現(How)**:
- 最後，解釋你將如何實現 `Why` 和 `What`
- 這包括方法論、high-level design、技術決策等
- 加入你如何不實現它（即超出範圍）也很有用
- 這一部分的深度取決於文件的內容


對於 `One-pagers`
- 它可以是關於可交付成果的一兩段話，細節在附錄中

對於 `design docs`
- 可能想包括一個系統背景圖，技術決定（例如，集中式與分佈式，EC2 與 EM R與 SageMaker）
- 離線實驗結果（例如，命中率，nDCG），和基準（例如，吞吐量，延遲，實例數）

有了 solid `Why` 和 `What`，就可以提供 context
- 使讀者更容易對你的想法進行評估和提供反饋
- 反之，如果意圖和要求表達不清楚，很難發現一個好的解決方案


**(Who)**:
- 寫 docs 時，要牢記聽眾
- 儘管「誰」可能不會在 docs 中顯示出來，但它會影響 docs 的結果（主題、深度、語言）
- 給 business leaders 的文件與給 RD 的文件看起來會非常不同
- 不同的受眾會關注不同的方面：客戶痛點、商業成果、投資回報率與技術要求、設計選擇、API spec
- 在寫作時考慮到你的對象，可以使討論和反饋更有成效
  - 我們不要求業務領導對基礎設施的選擇進行反饋，也不要求開發 RD 對業務戰略進行指導


## 怎麼用 framework 來架構你的 docs
下面是一些使用 `Why-What-How` 來架構 `One-pagers`、`design docs`、`after-action reviews` 以及我在這個網站上的寫的例子


| | **Why?** | **What?** | **How?** |
| :------| :------ |:------ |:------ |
| One-Pager | • 問題或機會 <br>• 假設的好處 | • 衡量成功的標準<br>• 限制因素 | • 可交付的成果<br>• 界定範圍外的內容 |
| Design Doc | • 為什麼這個問題很重要<br>• 預期的 ROI | • 業務/產品要求<br>• 技術要求和限制 | • 方法和系統設計<br>• 圖解、實驗結果、技術選擇、整合 |
| After-action Review | • 事件的背景<br>• Root cause 分析 (5 Whys) | • 有形和無形的影響<br>• 估算(如，停機時間，費用) | 後續行動和 owners |
| Writing on this site | 為什麼讀這 post 很重要 (e.g., anecdotes) | 討論的主題 (e.g., documents we write at work) | 分享的見解 (e.g., `Why-What-How`, examples) |


## **單頁文件(`One-pagers`)** 範例:
- **原因(Why)**:
  - 數據科學團隊（在一家電子商務公司）面臨的挑戰是幫助客戶更容易發現產品
  - 高級領導假設，「更好的產品發現」將提高客戶參與度和業務成果
- **What**:
  - First-order  指標是參與度（如 CTR）和收入（如轉換，每季的收入）
  - Second-order 指標包括 App 的使用（例如，每日活躍用戶）和保留（例如，每月活躍用戶）
  - 通過預算和時間表設定約束條件
- **方法(HOw)**:
  - 團隊考慮了幾種 online（如搜索、推薦）和 offline（如目標郵件、推送通知）的方法
  - 分析表明，大部分的客戶活動發生在產品頁面上
  - 因此，假設產品頁面上的專案對專案（`i2i`）recommender 能夠產生最大的投資回報率
- **附錄**:
  - 入站渠道和網站活動的細分
  - 各種方法的概述
  - 關於推薦系統的詳細解釋

## **設計文件(`design docs`)** 範例:
- **原因(Why)**: 
  - 目前，產品頁面缺乏一種讓用戶發現類似產品的方式
  - 為了解決這個問題，我們正在建立一個 `i2i` recommender 來提高產品的可發現性和客戶參與度
- **What**:
  - 業務需求與 `One-pagers` 中規定的相似，儘管更加詳細
  - 我們與網絡和移動應用團隊合作，定義技術要求，如
    - 吞吐量(throughput)（`>1,000` requests per second）
    - latency（`<150ms` at `p99`）
    - availability（`99%` 的正常運行時間）
    - 我們的限制條件包括成本（`<10%` 的創收，有一個絕對的門檻）和集成點
- **方法(How)**:
  - **這是 `design docs` 中最重要的部分**
  - 將分享方法和高層次的設計
    - 包括系統背景圖、技術選擇、最初的離線評估指標（用於ML）
    - 並解決吞吐量、延遲、成本、安全、數據隱私、集成等方面的問題
- **附錄**:
  - Trade-offs
  - 考慮但不包括的內容，API 規格，UI 等


## **事後回顧(`after-action reviews`)** 範例:
- **背景(Context)**
  - 在一個銷售高峰日（11/11），`i2i` recommender 在產品頁面上有一段時間不可見
  - 這是 category managers 在檢查產品的折扣時發現的
- **Why (5 Whys)**:
  流量的激增導致提供推薦服務時的延遲增加（>150ms）。延遲的增加導致了推薦者小部件的超時--在產品頁面上無法顯示。雖然啟用了自動縮放功能，但它達到了實例配額，無法超越該配額進行擴展。雖然我們進行了3倍於正常流量的負載測試，但這是不夠的，因為峰值流量是正常流量的30倍。此外，由於我們的警報沒有考慮到結果沒有顯示，所以沒有提前發現。
- **結果(What)**:
  - 客戶體驗沒有受到影響，因為產品頁面繼續在預期延遲內加載
  - 然而，不提供推薦服務導致了預期收入的損失
  - 根據當天剩餘時間內歸屬於推薦者的收入，估計損失為 x 美元
- **如何處理(How)？**:
  - 我們將採取 follow-up 行動，以防止事件重複發生，並儘早發現類似問題
  - These are their respective owners
- **附錄**:
  - 事件的時間線
  - 整體學習和建議

## Personal writing 範例
- **Why**: 為什麼寫文件很重要？分享奇聞軼事。提到這是被高度認可的
- **What**: 我寫什麼文件？分享一些例子。
- **How**: 解釋 `Why-What-How` 方法，並分享我如何使用它的例子


## 寫 docs 非常耗時（很貴），但也很便宜
- 寫文件是要花錢的，需要花時間來寫、審查和迭代，這些時間本可以用在實作上

儘管如此
- 寫作是一種便宜的方式
- 可以確保以正確的方式解決正確的問題
- 幫助團隊避免 rabbit holes 或建立不被使用的系統來節省錢
- 它們還能幫助協調利益相關者，改進最初的想法，並擴大知識範圍

如果問題是模糊
- 建議的解決方案是有爭議的
- 所需的努力是高的（>3-6個月）
- 和/或需要在多個團隊中達成共識，從文件開始將在**中長期內節省努力**


這裡有更多關於 `one-pagers` 的細節（以及其他我在開始專案前做的事情）。
- https://eugeneyan.com/writing/what-i-do-before-a-data-science-project-to-ensure-success/#first-draw-the-map-to-the-destination-one-pager