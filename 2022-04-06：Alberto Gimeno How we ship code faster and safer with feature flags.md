# 2022-04-06：Alberto Gimeno How we ship code faster and safer with feature flags
## Alberto Gimeno April 27, 2021
### https://github.blog/2021-04-27-ship-code-faster-safer-feature-flags/
---------------------

Github 談怎麼做 feature flags  
可惜沒有太多實作面的內容  

## 降低 deployment risk
Github 整天都有部署，而且要確保 service 沒有中斷。所以 deployment 必須是一個沒有風險的過程。任何 deployment 出問題，都會 需要 rollback。  

但 rollback 期間，User 就會受到各式各樣的影響。所以用 feature flags 來降低 deployment 風險
- feature flags 隔離並降低了部署 deployment risk
- 能夠在幾秒內就 disable feature，不需要花幾分鐘來 rollback

舉例來說
- db 查詢的更改可能會使現有功能變慢
- 現有功能的邏輯更改可能會產生意外行為、未考慮的 edge 情況等等
- 儲存或處理 data 的方式發生變化，例如在 db 中保存新列，可能會意外地開始插入無效值或生成過多的 db 寫入

這些時候，用 feature flag 可以很快先暫停功能，而不用等 rollback


致力於新功能
功能標誌不僅有助於在修復或改進現有功能時降低部署風險，我們還在開發新功能時使用它們！

使用功能標誌允許我們以增量方式處理功能。只有從事該項目的工作人員啟用了相應的功能標誌，因此他們可以看到正在開發的新功能，而其他用戶看不到。我們不使用長期存在的功能分支。相反，功能標誌允許我們處理小批量，這給我們帶來了很多好處：

## 開發新功能時
Feature flags 在開發新功能時也有用  

feature flags 能做到，只有該功能負責的人才啟用特定功能，而一般 User 無法使用、看不到。Github 不採用長期的 feature branch，而是使用 feature flags，這樣能讓 develop 採用小、多個 branch 來開發功能  
- 小 PR 更容易 review
- 越少 change，production deployment 出問題的機率就更小
- 沒有長期的 feature branch，就減少 conflict 的機會

為了採用這種模式，需要些額外的計畫來劃分工作，例如
- new feature 常常需要新的 data modal or 挑整 data modal，如果 developer 只針對 update data modal 就提 PR，那這 PR 大概會被擋住，直到所有的 change 一起被 merge  

Github 會做些事情來避免這些情況
- create 一個 main PR。把 data modal update 的 change 放進來，讓後接著繼續做其他改動，像是 UI、API or background job。接著，把這些 update 都拆出去，成為小的 PR，給其他人 review
- 或者ㄧ樣為 data modal update 建立一個 PR，然後用這個 PR checkout 出去另一個 branch 繼續後續的開發。如果第一個 branch 有修改，那後面 co 出去的要重置


由於必須更頻繁地要求 review，這種工作方式可能會帶來一些摩擦  
有時，團隊 revoew 你的 PR 可能需要幾天時間。為了防止這成為阻礙，我們遵循以下一種或多種策略：
- 即使更改沒有合併到主分支中，也要繼續工作。無論是在那個 main PR 上，還是從等待 review 的分支上分支出來。
- 每個團隊都有一個第一響應者，專注於快速review PR，以疏通他們自己團隊或其他團隊的進度。有時，如果一個關鍵的 PR 準備好，要盡快部署了，但有非常好、但非重要問題的 feedback 要解決，我們會在後續 PR 中處理（先放過不重要的議題就對了）



## 測試 features
為了必須確保 new feature 按預期工作，同時保持現有功能。為了做到這一點，我們有許多機制來啟用或禁用所有 level 的 feature flags：
- 在 development environments 中，developer 可以從 cli 切換 feature flags
- 在 automated tests 中，可以在 code 中啟用或禁用 feature flags
- 在我們的 CI 中，有兩種不同的構建：一種在默認情況下禁用所有 feature flags 運行，另一種在默認啟用所有 feature flags 的情況下運行
  - 這大大減少了無法正確覆蓋自動化測試中大多數代碼路徑的機會
- 在 production 中，我們可以用 request 和 query string 來啟用或禁用 feature flags

不同的運輸策略
我們已經提到我們可以標記特定的演員，或者我們可以為一定比例的演員啟用功能標記。這些是最常見的選項，但其他選項包括：

## 不同的 shipping strategies
上面提到，feature flags 能針對某些人設定，另外還有其他功能
- 個別參與者：我們使用這種機制來標記從事某項功能的員工，或者為正在試驗某項功能的客戶啟用該標記，或者如果他們有我們正在嘗試解決的問題。
- Staff shipping：當一個功能幾乎準備好發佈時，我們首先為所有 GitHub 員工啟用它，發佈內部公告以提高意識，並創建內部問題以收集反饋或可能的錯誤。
- 早期訪問維護者或其他 beta 組：當我們即將發布一個重要功能，會影響開源軟件 (OSS) 維護者或其他類型用戶的工作流程時，我們會先與一小群人一起測試該功能。一段時間後，我們可能會採訪他們並收集他們的反饋，以確認我們的假設，並驗證實施。
- 參與者百分比：可以指定啟用功能標誌的參與者百分比。在這種情況下，當一個 actor 被標記時，它會保持標記，除非特徵標記被回滾。當參與者的百分比發生變化時，會向我們在 Slack 上的 deployment channel 發送消息，以便其他工程師知道這變化。
- Dark shipping：這允許我們為一定比例的調用啟用功能標誌。這與之前的機制不同，因為參與者可以在一次調用（例如一個請求）中啟用該功能，但不能在下一次調用中啟用。此機制不適用於用戶可見的功能，而是用於內部更改，例如查詢中的性能改進。

所有這些策略和 feature flags 的完整管理都是通過 Web UI 完成的
- 並非所有員工都可以訪問此 UI
- 但大多數工程師可以
- 該界面允許管理 feature flags 的 shipping status、feature flags 的新增或刪除
  - 此外可以看到對 feature flags 所做的更改歷史記錄：誰進行了更改，何時進行了哪些更改，以及其他元數據，例如哪個團隊擁有它。


標記什麼
當我們說我們可以標記演員時，我們的意思是我們不僅可以標記用戶，還可以標記其他實體。特別是，我們可以標記用戶、組織、團隊、企業、存儲庫或 GitHub 應用程序。在編寫檢查標誌是否啟用的代碼時，我們需要指定哪個參與者啟用了標誌。這些是一些例子：

## What to flag
上面提到 flag an actor 時，這邊是指，flag 不只針對 users，另外還有一些實體，例如
- 組織、團隊、企業、repositories 或 GitHub Apps

在 code 裡面檢查 flag 時，大概需要確認這些東西  
- 如果 flag 是純視覺相關的，我們 flag 用戶，並檢查當前登入的用戶
  - 舉例：dark mode、layout or navigation changes, and new components in the UI that don’t depend on data changes.
- 如果 flag 改變了我們存儲 data 的方式，通常最好 flag repository，以便所有 repository 的User 保持一致
  - 舉例，如果開始存儲特定操作的 tiempstamps，我們不希望某些用戶生成新的 timestamps，而其他用戶則不會。
- 對於 API 更改，我們還會 flag GitHub App
- 有時，特別是對於一些有風險的更改，我們會創建自定義的、特定於上下文的參與者類型，而現有的參與者類型根本無法針處理需求

有時我們會進行更複雜的檢查，例如
- 檢查多個 actors 或檢查 repository 是否是 public 的
  - 例如，當員工發布一個功能時，它可能會為員工的所有 repository 提供一個新功能。如果該功能不能對其他用戶可見，我們希望只為 private repo 啟用該功能，以防止員工的 public repo 洩露新功能。


首先，我們有運行時成本。當我們檢查一個特徵標誌是否對一個actor啟用時，我們需要首先加載該標誌的元數據信息，這些元數據信息存儲在MySQL中，但緩存在memcached中。我們現在正在考慮讓特徵元數據和演員列表始終在內存中，以減少這種開銷。有一個例外：我們認為某些功能標誌“很大”。這方面的一個例子是 GitHub Actions，這是我們每周向數万用戶推出的一項功能。對於這些大的功能標誌，當它們還沒有完全啟用時，


## The cost of a feature flag
Feature flag 改變了很多原有的工作方式，這是需要一些額外的計劃和協調。另外還有一些需要知道的額外成本  

首先是 **runtime cost**，當需要檢查某個 feature flag 是否對某位 actor 啟用時
- 需要首先載入該 flag 的 metadata，這些 metadata 在 MySQL中，但也有 cache 在 memcached 中
- Github 現在正在考慮讓 flage metadata 和 actor list 始終在 cache 中，以減少這種開銷

有一個例外：
- Github 認為某些 feature flag 很大，例如 `GitHub Actions`，這是每周向數萬 User 用戶推出的一項功能。

對於這些大的 feature flag
- 當它們還沒有完整啟用時: we need to do a per-actor query to MySQL to check if the actor is in the list of enabled actors.
- For runtime
  - 有一些技術債所造成的成本
  - 一旦某個 feature flag 完全啟動，通常會殘留很多 dead code and old tests 在 repo 中需要被刪除
  - 通常 Github 會等 feature 推出幾週後，手動執行 script 來處理
  
![](https://github.blog/wp-content/uploads/2021/04/feature-flags-fig-1.png)  

這 script 用 `git grep` 和 regular expressions 找到特定的 feature flag，然後被找到的 ruby 檔案，能用 `rubocop-ast` 來調整 or 刪除 code 等等  
現在還有部分手動，未來會盡量自動化這過程  


## Conclusion
Feature flags allow us to ship code faster with higher confidence. Using them we reduce deployment risk and make code reviews and development on new features easier, because we can work on small batches. There are also associated costs, but the benefits highly exceed them.

## 結論
- Feature flags 讓 developer ship code faster with higher confidence
- reduce deployment risk
- code reviews and development on new features easier, because we can work on small batches.
- 也有相關的成本，但收益遠遠超過它們

