## gitlab Code Review Guidelines
## https://docs.gitlab.com/ee/development/code_review.html

### for 每個人
- 欣然接受許多的`程式決策`都是種`意見`，討論其中的 `tradeoffs`跟你 prefer 的方向，快速做決定
- 問問題；don’t make demands
> (“What do you think about naming this :user_id?”)
- 問問題來離清疑問
> (“I didn’t understand. Can you clarify?”)
- 避免去選擇性的歸屬 code
> (“mine”, “not mine”, “yours”)
- 避免用特定詞彙去談人他人特徵
  - 假設每人都是 有吸引力的、有智慧的、well-meaning
> (“dumb”, “stupid”)
- 應該清楚的、詳盡的。大家在線上絕不總是能了解你的想法
- 謙虛
> (“I’m not sure - let’s look it up.”)
- 別誇大
> (“always”, “never”, “endlessly”, “nothing”)
- 別挖苦
- 如果出現很多 `I didn’t understand` or `Alternative solution` 的留言，那就考慮 `one-on-one chats` or `video。事後補上 follow up comment 總結。
- 如果你問特定人問題，always 在 comment 一開始就提到他。（其他人也會因此知道，他們不用回答）

### Having your code reviewed
請記得，code reivew 是一個多次迭代的過程。  
- 第一位 reviewer 一定是你自己。 push 之前，看看 entire diff
  - Does diff make sense?
  - 有沒有多了什麼不相關的、跟此 MR 目的沒關係的
  - debug code 移除了嗎？
- 感謝 reviewer 的建議
> (“Good call. I’ll make that change.”)
- Don’t take it personally、不是針對你，review 是針對 code。
- 解釋為什麼 (某些) code exists
> (“It’s like that because of these reasons. Would it be more clear if I rename this class/file/method/variable?”)
- 抽出不相關的 changes，整併到未來其他的 MR/issues
- 去了解 reviewer 的觀點
- 盡量回覆每一篇 comment
- 只有當 discussions 被 fully addressed 的，才可以被 MR 的 author resolves，其他...
  - open reply, an open discussion, a suggestion, a question 要讓 reviewer resolves 
- branch 要 merge 前才 squash。

### Reviewing code
弄清楚這 change 是為什麼？是否必要？
> (fixes a bug, improves the user experience, refactors the existing code)
接著：

- 透過你的 review 來減少反覆的次數。
- 溝通那些你強烈覺得... 和那些你不覺得該...(作法、方法)
- 指出有更簡單的方法、code 也能解決問題
- 提供其他解決方案（但假設 author 已經有考慮這些方案了）
> (“What do you think about using a custom validator here?”)
- 嘗試去了解 author 的觀點
- 如果你不懂某段 code，就提出來，因為很有可能其他人也不懂。
- 一輪下來後，做個 post summary notes 很有幫助
>  “LGTM :thumbsup:”, or “Just a couple things to address.”
- 你 review 過後，如果有 changes 必要的話，MR assign 回給 author。
- merge 前，要設 `milestone`。
- `job succeeds` 後才能 merge (“Merge When Pipeline Succeeds” (MWPS) 的話ＯＫ)
- Consider using the Squash and merge feature


### The right balance
`code review` 時，掌握平衡是門藝術。到底該多 deep？

學會找到平衡是需要時間的。這就是為什麼我們(Gitlab)的 reviewers 在 reviewing 需多 MRs 後，能成為 maintainers 。
- 找到 bug 跟 改善 code style 很重要
- 但思考 good design 也很重要
> 建構`抽象化`和 `good design` 能夠降低法雜度、提高可維護性。

要求改變 design 經常等於整個重寫，這種時候，也可以先問問其他的 maintainer or reviewer。  
但覺得這樣調整很重要時，就積極去做吧。

`doing things right` 跟 `doing things right now` 有所區別，但實務上...，例如  
有一個 security fix 要 release，同MR 中的重構，你就 avoided 掉吧，下次處理。  

- `Doing things well today` 經常 >>> `doing something perfectly tomorrow`
- `Shipping a kludge today` 經常 <<< `doing something well tomorrow`
無法拿捏時，問問同伴吧。