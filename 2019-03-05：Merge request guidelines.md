## Merge request guidelines
## https://docs.gitlab.com/ee/development/contributing/merge_request_workflow.html#contribution-acceptance-criteria

### Merge request guidelines
如果你有能力，你提的 MR 請包括 tests。如果你不知道該怎麼 fix or improvements，你也可以只提 tests 的 MR，tests 能 exposes 出某些 issues。有 tests 才能加速 MR 的 merge。

#### MR 的 workflow
1. fork project 到你 personal space。
2. 從 master checkout 出 feature branch。
3. 寫 tests 和 code
4. 產生一個 changelog (gitlab 有用工具產生 changelog)
5. 如果有寫 documentation，確保有 follow documentation guidelines
6. local 就 squash commits 整理一下。
7. push commits 到你的 fork project 去
8. 提 MR 到 master branch (MR 至少要有一位 approval、可以多位。)
9. MR title 應該要描述你想要改的東西
10. MR description 寫出改的動機、跟解決方案。
  - 如果你是貢獻 code 的人，用 description 裡面提供的方案來寫。
  - 如果你貢獻 documentation，那選用 `Choose a template(gitlab 的功能)`
  - 提及 MR 要解決的 issue(s)。（gitlab 用`Solves #XXX` or `Closes #XXX` 會自動 mapping 跟 close issue）

11. 設相關的 `milestone` 跟 `labels`（如果你有權利）
12. MR 改動到 UI，那要附 `Before and After screenshots`
13. MR 改動到 CSS 那要列出影響的 page
> grep css-class ./app -R
14. 準備回答問題（包含不適當的 feedback）。討論完的問題就 resolve 掉。
15. 如果 MR code 有關
  - 執行 shell commands
  - 讀 or 開檔案
  - 處理硬碟相關檔案路徑等等  

確保這些有 follow `shell command guidelines`  
16. 如果 code 有在硬碟上 `creates new files`，follow `shared files guidelines`
17. commit messages follow 這些指南
  - [A Note About Git Commit Messages](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
  - [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)
18. 如果 MR 包含一or多個 migrations，確保code review 前有執行所有的 migrations 在 fresh database。當 review 改很大的話，重新測一次。
19. 複雜的 migrations 要寫 tests。
20. MR 一定要遵守 [(gitlab)merge request performance guidelines](https://docs.gitlab.com/ee/development/merge_request_performance_guidelines.html).
21. 用 `Capybara` or `PhantomJS` 測試的話，參考這篇文章寫出 reliable asynchronous tests
  - https://thoughtbot.com/blog/write-reliable-asynchronous-integration-tests-with-capybara
22. 如果 MR 改動，你的 intall 方式有特別步驟的話，要寫清楚。
23. 如果 MR 改動，你的 upgrade 方式有特別步驟的話，要寫清楚。

記住，**MR 要越小越好**。大的 MR 你可以想看看
- 能不能拆成各自的 functionality？
- 只提 backend/API code ?
- 從簡單的 UI 開始？
- 重構只針對特定某部分？

小的 MR 增加可讀性、對提升品質很有幫助，也會有最小的 commit log、也最快能合併。k8s 這邊也有一些觀點可以參考
- [How to get faster PR reviews’ document of Kubernetes](https://github.com/kubernetes/kubernetes/blob/release-1.1/docs/devel/faster_reviews.md)


### Contribution acceptance criteria
1. 改變越小越好
2. 加入適當的測試並且 all tests pass（除非現有的 code 就是有 bug）、所有新 class 都要有對應的 test，甚至要有 feature test
3. 如果懷疑 CI build 失敗，跟你的改變沒關係，restart CI。
4. 你的 MR 最初應該只有一次 commit (git squash)
5. 你的 change 要能順利 merge，不行的話要 rebase
6. 別搞壞現有功能 or function
7. fix 一個特定的 issue or 實作一個特定的 feature（別 combone 再一起、拆開 MR）
8. 做 Migrations 應該只做一件事情 either
  - create table
  - move data 到新 table
  - remove old table  
來在 failure 幫忙 retry  
9. Keeps the (GitLab) code base clean and well structured
10. （可能的話，就多多）functionality ，有利於其他人（理解）
11. 別新增配置與設定，因為這些複雜的東西會讓 test 變更困難、日後也難改變
12. Changes 不要造成 performance 變差
  - 避免重複的 polling endpoints (require 次數、量會很大)
  - 透過 `SQL log` or `QueryRecorder` 確認 `N+1 queries`
  - 避免重複存取 filesystem。
13. 如果你需要 polling 來實踐 real-time 的 features，使用帶有 `ETag cache` 的 polling
14. 已經提出的 MR 後才做的 changes commit 就保持 separate (no squashing)
15. 如果 MR 有新增 libraries 的話，這些 libs 要 conform `Licensing guidelines(gitlab)`
16. MR 要做到 definition of done.

### Definition of done 
(這些都是 gitlab 的定義而已，其他人參考看看就好)  
Please ensure you support the feature you contribute through all of these steps.  

1. `Description` 適當的解釋
2. Working 跟 clean code，需要的地方，就有 commented。
3. CI 上有 單元、整合、系統測試都過
4. Performance/scalability 相關的地方都有被考慮到、被測試
5. Documented (有寫文件歸檔了)
6. 如果有需要，要新增 Changelog entry。
7. 被 UX/FE/BE 給 review 了，所有被提出的地方都被處理過了。
8. MR 被 project maintainer merge 了
9. Added to the release blog article, if relevant
10. Added to the website, if relevant
11. 社群上的問題都回答了
12. Answers to questions radiated (in docs/wiki/support etc.)
13. Black-box tests/end-to-end tests added if required. Please contact the quality team with any questions

### CHANG LOG
mozilla 的 project 有一個產生的方案，去查看看。  
一時忘記了