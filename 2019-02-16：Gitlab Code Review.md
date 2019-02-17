## Gitlab Code Review
## https://about.gitlab.com/handbook/engineering/workflow/code-review/

## Overview
每一個 MR/PR 都要 code review，要遵循 [Code Review Guidelines](https://docs.gitlab.com/ee/development/code_review.html)  

誰需要來 review、approve 和 merge MR。

## Reviewer
所有 GitLab engineers 都可以，而且也鼓勵這去 perform code review。你想 review MR 的話，可以等某人 assign 給你，或者歡迎去瀏覽 open MR list並留下你的 feedback 或問題。
> 為了讓其他 engineers 知道你有興趣去 code review，你可以這邊加入你的資訊

Gitlab 的做法是維護一份文件，讓所有人看到。

> all engineers 可以 review 所有 MR，但只有 maintainers 才能 accept MR。

## Maintainer
Maintainers 是 GitLab engineers 而且也是 code review 的專家。需要對 product and code base 非常了解。Maintainers 被授權能夠 accept GitLab Engineering Projects 的 MR。  

每一個 project 至少有一位 maintainer，但大多是多位 maintainer。某些 project 有分 frontend 跟 backend maintainer。

傑出的 engineers 通常也是傑出的 reviewers。但 code review 本質上也是個 skill，不是每一位 engineer (無論 engineer 的資深程度)都有同樣的機會磨練這項 skill。另外一點也很重要是 good maintainer 是對 existing product and code base 有非常深入了解的人。這樣 maintainer 能夠辨識出（容易被 miss的） inconsistencies、極端案例（edge cases）或與跟其他（不明顯） feature 的 interactions。
為了要保證 code base 的品質和產品的完整，要成為 maintainer 的人一定要展現出令人有說服力（與現有的 maintainers 有相同水平）的 reviewing skills。

> Gitlab 一樣用一份文件來列出哪些人是 maintainers

## How to become a maintainer
參與 MRs 就是個提昇 reviewing skills 的好方法。把你的 review notes 傳遞給 maintainers，然後 follow MR 的交談直到 MR close。
- 如果你覺得 comment 不合理 or 想不通，就要求對方近一步說明。
- 如果你自己 miss 掉某些 review，就弄清楚會什麼自己會 miss 掉、記下來，下次就會（能夠點出了）

這邊有 2 點指南可以做參考，但這不是死板的規則（no concrete rules）：

- Junior（比起成為maintainers）應該更專注在提升自己成為 中階工程師。
  - Junior 更往下走的話，我們預期 Junior 就更能成為有能力的 maintainer。
- 申請成為 maintainer 前，希望能至少待在 GitLab 一年。這樣才對 codebase 有好的了解、專精一or多個領域，並且深入理解 coding standards。（如果職位比較高的員工，這時間會短點）

除此之外，下面都符合的話，也可能被視為 maintainer：
- 他們**看過**的 MR，透過 maintainer 看過之後沒有什麼 significant 需要改動的東西。
- 他們**提出**的 MR，透過 maintainer 看過之後沒有什麼 significant 需要改動的東西。
