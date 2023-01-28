# 2022-12-30：Watermelontools: 6 steps to do better code reviews
### https://www.watermelontools.com/post/6-steps-to-do-better-code-reviews

---------------------------------------

前陣子面試被問「你都怎麼做 code review？」，事後回想起來，自己的回答不太有條理/系統性  
所以想再來重新看看幾篇 code review 的文章來思考一下  
這篇很短，之前掃過一遍後，覺得好像條列的不錯，所以來讀讀  

(讀完後，這篇提的大方向是對的，但細節提的非常少。另外回想起來我自己好幾年前看過 Gitlab 的文件寫的很好，應該要去複習那篇才對)
- [2019-03-05：Merge request guidelines.md](<./2019-03-05：Merge request guidelines.md>)

---------------------------------------

## TLDR

- 讓每個人都做 `code review` (包括 junior)
- 檢查 code 的 `readability`
- 確保 PR 是在創造 passive documentation
- 測試 performance
- 檢查潛在的安全風險
- 檢查 `testability`

免責聲明: 這份指南適用於 `Product-Market-Fit` 之後的公司
- 如果你的公司是 `pre-PMF`，你應該優先優化迭代的速度，而不是其他的東西
- 如果你是 `pre-PMF` 的公司，你的員工數也應該盡可能的少
- 因此這裡的很多準則並不適用。根據需要進行調整

我們以前都見過類似的事情發生:
<p align="center">
  <img width="40%"  src="https://assets.website-files.com/62a268cb939b7c3b0b3bfb60/637b0f39d6cd8c3979f5385c_zsFeyYwZv6KqhJVmnxtixYbCLbzqHmBEeav1a6lHZzneynwe3Deyb6XlCg60Ccc2WyjKr2EbbODXXk7D7Q1o3dL1n1jYPNaL0nSz42kCh3EtHefG8dqWUzSkDGlEAzR_iRIRcqWgKTDwiPpnj8bdBlu0B4RELcYDGdlcjb_hze-TGV8aMWheaDykAu90LQ.png">
</p>

這不怪你
- 管理層總是有壓力，要求盡快 ship，以最終讓產品產生更多的收入或牽引力
- 事實上，這是作為開發者的最終目標
- 然而，不穩定或無效的 `code review` 文化會導致 technical debt 的累積

---------------------------------------

## 讓每個人都做 `code review`

即使是最 junior 的人也應該 review technical lead 的 PR
- 你可能認為這有悖常理。這樣做不是會有很多隱患嗎？

但實際上的一個好處是
- 一個較 junior 不一定了解 code 的上下文的所有內容
- 可能無法預見所有潛在的問題
- 而且一般來說，不像 senior 能抓到很多錯誤
- 但，這就迫使 senior 員更好地解釋他的 code/changes

有 2 種技巧來分配 PR code review，GitHub 提供了自動分配 PR 選項採用這兩種方法
1. Round-robin: 輪流分配數量，而不管他們有多少未完成的 PR
2. Load balancing: 根據每位成員擁有的未完成 PR 來分配
    - 試圖確保每個人在任何 30 天內 review 數量相同

可以參考 GitHub doc
- https://docs.github.com/en/organizations/organizing-members-into-teams/managing-code-review-settings-for-your-team#about-auto-assignment

建議
- 如果開發人員的資歷差不多，就採用輪流 (Round-robin)
- 如果資歷有差，就採 load balancing

---------------------------------------

## 檢查 readability

Code 必須是可讀的，才是可維護的。Code 必須是可維護的，才能對開發團隊有用。下面是一份代碼可讀性的檢查表:
- Code 是否符合組織的標準？
  - 標準對於每個團隊來說都是非常具體的，可以是圍繞著命名、提示等等
  - 一般來說，是團隊中的人同意的一系列規則
- 變數和 function name 是否具有描述性？
  - 使用可讀、可搜索的名字
- Function 是否可以分解成更小的 function？為了達到這個目的，值不值得對進行重構？
- 這段代碼需要 comments 嗎？

(這段也是講的很少，但大方向是對的)

---------------------------------------

## PR 是否包括有用的 passive documentation?
- Title:
  - Title 是否給了修改的資訊？
  - 一個做法是在標題上包括 (Jira, Linear, Notion, ...）ticket code，以連接資訊
  - 另個做法是在標題加入一個命令式的動詞(如: `move the module ...`, `create the function ...`, `debug the feature ...`)
- Description:
  - 作者是在解釋問題嗎？
  - 預期的解決方案？
  - 是否有一個測試計劃的概述？
  - 描述中是否包括 scrrentshot 或 screen record ?
- Description 中是否標示類型？
  - 這資訊給團隊帶來更好的可搜索性
- (可以多參考網路上別人的 PR template)

(這段提的太少了，可以去參考 Gitlab 的文件)
- [2019-03-05：Merge request guidelines.md](<./2019-03-05：Merge request guidelines.md>)

---------------------------------------

## 測試 performance

測試 performance 該有多嚴格，100% 取決於公司所處的階段
- 成熟/數百萬 User 的公司的 testability requirements 與 startup 的要求是不同的
- 後者的生存需求是找到 `Product-Market-Fit`，否則就會沒錢了
- 前者需要對嚴格性進行優化
- 後者則需要對迭代速度進行優化

當你想對嚴格性進行 optimize 時，需要測試:
- 檢查不必要的 logging: 在錯誤的地方做 code logging 會大大降低 performance，並暴露出應該保持隱私的資訊
- 檢查 data types: 基於預期值，DB types 可以進行性能優化
  - 如在 SQL 中，data 的長度不一樣，使用 varchar 比 char 更有效
  - 如果一個列要存 1 到 5 之間的值，用 tinyint 而不是 int
- 是否可以通過 add/del indexing 來提高 performance ?: 根據不同的情況，是可以的
- 可以在 server-side render 時 fetch data 嗎？
- Queries 能不能 return 更少的 data ？

檢查潛在的 security failures
- 檢查是否有過度的資訊性錯誤提示: 要有資訊量，但不要過度。有時資訊會給 attacker 提示來入侵
- 檢查是否有暴露的 env secrets:在組織中鼓勵良好的 secret management 做法
  - 一旦 secrets 在 remote branch 上暴露，必須立即更新
- 確保 authentication 和 authentication 正確處理:
  - 始終用 HTTPS
  - 考慮為不同的角色和權限級別設置不同的 API keys
    - (可以看看 GCP: https://cloud.google.com/api-keys/docs/access-control )


Testability (可測試性)
- 測試是否通過？
- 無論是 unit/e2e test，確保它們都能通過
  - 如果已存在 flow 改變了，相應地更新測試