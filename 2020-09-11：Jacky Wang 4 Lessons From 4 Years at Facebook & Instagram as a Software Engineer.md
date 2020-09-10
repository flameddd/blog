# 4 Lessons From My 4 Years at Facebook & Instagram as a Software Engineer
## Jacky Wang
### https://medium.com/swlh/4-lessons-from-my-4-years-at-facebook-instagram-as-a-software-engineer-cc0b7c18678
  
## 作者這段時間的經歷
- 9/6/2016 加入 Facebook
- 首先加入 Menlo Park 的 Messenger 團隊之一
- 六個月後，團隊解散。轉到紐約，加入 Instagram Search and Explore team as an iOS engineer
  - 3.5 年 IG 開發經歷，參與 Discovery, Explore, Search, Location Pages, Hashtag Pages, Location/Hashtag/Sticker Stories, Save, Collections, Keyword Search, Integrity and Privacy, and more recently Shopping 
  
以下是他的 4 點最重要的學習反思

# Learnings (學習方面)
## Occam’s Razor / **Do The Simple Thing First** (先從簡單的下手)
- 最簡單的解決方案通常是正確的方案

每當有提出 overwhelming complexity in a proposed architecture or solution 時
- 總是退後想一步，重新評估真正試圖解決/構建的內容
- 並思考是否有更簡單的方法可以實現相同的目標

有時就會意識到
- 你正嘗試解決的問題，其實不是真正要解決的問題
- 提出的 complex solution 實際上繞了一大圈、花更多時間來解決更本問題
- 當你 narrow the scope of the solution to just the problem 時，所建構出來的解決方案通常相對簡單、單純非常多
- tangled complexity 實際上是一種不必要的依賴關係，應將其分別封裝和解耦

作者在工作中經常利用到的 Programming paradigms 有
- Single source of Truth
- Immutability
- One Direction Data Flow
- Single Responsibility Classes and Encapsulation

這些都遵循著
- 通過 reducing variables and dependencies 來簡化事情

沒有完美的架構，但是如果架構的所有層都遵守這一原則
- 系統會 more robust, easily understandable and extendable/delete-able


從產品的角度來看
- “首先做簡單的事情” 一直是作者在 Instagram 的核心價值觀中得到最大共鳴的最高價值觀

與 Facebook (lots of bloated features) 相比
- 作者更喜歡 Instagram 中的 simplicity，
- 作者相信也是基於 Instagram 這一產品理念
  - 通過減法(這在加法中扮演了非常積極的角色) ，認為 Instagram 做得很好
  - 通過不斷的 unshipping 不相關的 features，以簡化用戶體驗和 codebase
  - **Doing the simple also helps you move faster**

# Kill Your Darlings

斯蒂芬·金 (Stephen King)(美國恐怖大師)回憶錄
> Kill your darlings, kill your darlings, even when it breaks your egocentric little scribbler’s heart, kill your darlings.
- (就算你的自尊會受傷，幹掉你的寶貝！)
- the art of addition by subtraction


在快節奏的工作環境中的 engineer，你構建的功能，無論花費多長時間，進行了多少工作，或在計劃中看起來有多大前景
- 總有可能在完成之前 or 後 no ship, or ship and then get unshipped.

為 User 創造成功的產品的最佳方法是
- 盡快迭代並從測試和數據中學習

作者
- 在我職業生涯的早期，我為自己的作品感到非常自豪，但是卻沒有學會從流程中分離出來

你可以正確地做所有事情，構建沒有錯誤的人人都喜歡的完美產品，然後
- 了解 User 對它的喜愛程度不高 or
- it tradeoffs against a different non-negotiable metric based on company priority

以您如何構建產品而自豪，但將你的情感和身份與產品分離開來
- 從結果中做出情感反應反映出不成熟和無法做出理性且公正的決定
- 失敗的 project 就是，沒有從 project 中學到任何東西、沒有從產品和 User 那邊學到經驗、見識

Kill your darlings
- 不要讓「你的自我」妨礙團隊/公司
- 從過程中分離結果、從學習中分離出情感。

# The Underrated Importance of Soft Skills (軟技能遠比想像的更重要)

coding skills is fundamental to becoming a good software engineer, but
- the best software engineers are often ones who have the combination of coding and **very strong soft skills**.

Engineering
- it’s also figuring out what’s the right thing to build and also **convincing others**

特別是在 Facebook, where engineers are expected to drive and own a big part of the roadmap and direction on projects
- 不僅要與不同 stacks 的其他 engineers 合作，還要與設計師，內容，產品，研究，管理，其他團隊的人合作
- 通常，要對要構建的內容進行調整可能比實際構建更具挑戰性
- You need soft skills to make sure what you build can actually be shipped


在提出構想和買入要建造的東西時，我認為最好的工程師要做的就是迅速將原型拼湊在一起，然後交到人們手中進行嘗試。 投一個想法是“做起來比說容易”，而不是“說起來比做起來容易”。 玩弄手中的東西至少比看到模擬和聽見的話更具吸引力。 黑客馬拉松是鍛煉肌肉的好地方。

當在想 ideas 時，並且尋求其他人的支持來同意 build 時，這時候，best engineers 會這樣做
- 快速 hack a prototype、把 prototype 拼湊在一起，然後丟給別人去玩看看

Pitching an idea
- 要的是 **“easier done than said”**
- 而不是 **“easier said than done”**. 

能有東西可以實際玩玩看
- 遠遠強過只用嘴巴說、心理想去模擬畫面  

當要 ship a feature 時，不單單只需要 develop feature
- 還需要說服所有 stakeholders 和 team members
  - 要描述 user 的使用體驗、經驗來說明為什麼要開發、啟用這個 feature
  - 這些都需要用友說服力的 data 來 support 你的論點
  為什麼這個的功能應以用戶體驗，強大的數據支持和買入為基礎進行敘述 來自領導。


Soft skills
- 不僅與口頭交流有關
- 寫作技能是生活中最被低估的技能
  -  能夠簡潔明了地指出要點，並寫 notes / documentation 以在被問到之前回答問題

# Extreme Ownership (完全負責)

作者
- 在我們的團隊/組織中，有一個 **Directly Responsible Individual (DRI)** 的角色
- 是給 engineers 從專案的 planning 到 launch 的 full end-to-end ownership
- 目標是 empower individuals，建立問責制並減少依賴
  - 避免依賴 managers 來告訴團隊該做什麼
  - 提高團隊的自我組織並弄清楚如何進行
- 成功的 DRI 會讓 managers 知道，在 assigning the project 後，managers 就不需要 ownership

作者: 一位經理推薦 Jocko Willink Extreme Ownership
- 這本書跟這個主題吻合
- 這條 rule 在我工作上、工作外發展都非常有幫助
- Extreme Ownership | Jocko Willink | TEDxUniversityofNevada
  - https://www.youtube.com/watch?v=ljqra3BcqWM

核心概念
- 發生在你身上的事情並不總是你的錯，但 always 是你的全部責任

for example:

別人 commit 的 code 有 bug，這不是我的錯
- 但是我的責任是思考
- 我該如何把 component 的架構設計更好，來避免這種 bug?
- 能不能加入某些 test 讓 bug 更早的被抓到，而不是等到 regression or production 時才抓到

我 own 的 project 因為 x, y, z 原因 fail 掉了，這不是我的錯
- 但我有責任盡力提前看到 project 會 fail，甚至避免 project fail。

隊友們努力奮鬥了，但還是無法達到預期，這不是我的錯
- 我可以做些什麼來創造一個更令人鼓舞的環境或疏通它們，而不是將 project 的 fail 歸咎於他們
- The failure is just as much on me as it was on them.

It’s not entirely my fault that my life is the way it is,
- but it is entirely my responsibility to make it the way I want.


對你的 code、project、career、mindfulness、relationships 完全負責，這將使你和你周圍的每個人都變得更加輕鬆。


# Conclusion

At four years of Facebook & Instagram, I learned to
- Do the simple thing first
- Kill your darlings
- Value soft skills
- Take extreme ownership over my work and my life.

These lessons not only helped me in my career but also had an enormous impact on my life outside of work.
