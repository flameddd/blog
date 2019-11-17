## google：The CL author’s guide to getting through code review
#### https://google.github.io/eng-practices/review/developer/

雖然這邊是講 change log，但整篇也有偏向提 PR/MR 的面相

### Writing good CL descriptions
Change Log 是：
- 一份公開說明紀錄
  - 紀錄改變了什麼？
  - 紀錄為什麼改變？  
- 這會是 version control 的歷史紀錄

後續的 developers 有機會來查詢問題與原因。  

### First Line
第一行
- 簡短的 summary 說明完成了什麼事情
- 使用完整的句子與順序 (an imperative sentence)
- Follow by empty line.

第一行（或者應該說第一段？）應該要有足夠的資訊
- 讓看的人，即使不用全部的 CL 都看，看 first line 就能大概了解這次的 feature 內容

it were an order (an imperative sentence)  
For example, say
```
"Delete the FizzBuzz RPC and replace it with the new system.”  
```
instead of  
```
"Deleting the FizzBuzz RPC and replacing it with the new system.”
```

### Body is Informative

The rest of the description should be informative.  

It might include a brief description of the problem that’s being solved, and why this is the best approach.  

If there are any shortcomings to the approach, they should be mentioned.  

If relevant, include background information
- such as bug numbers, benchmark results, and links to design documents.

Even small CLs deserve a little attention to detail. Put the CL in context.

### Bad CL Descriptions

“Fix bug” is an inadequate CL description.
- What bug?
- What did you do to fix it?

Other similarly bad descriptions include:

```
“Fix build.”
“Add patch.”
“Moving code from A to B.”
“Phase 1.”
“Add convenience functions.”
“kill weird URLs.”
Some of those are real CL descriptions. Their authors may believe they are providing useful information, but they are not serving the purpose of a CL description.
```

### Good CL Descriptions

Here are some examples of good descriptions.  

Functionality change
```
rpc: remove size limit on RPC server message freelist.

Servers like FizzBuzz have very large messages and would benefit from reuse. Make the freelist larger, and add a goroutine that frees the freelist entries slowly over time, so that idle servers eventually release all freelist entries.
```
第一行說明 CL 做了什麼  
後面講這解決什麼問題  
為什麼需要這樣解決？有什麼好處

Refactoring 
```
Construct a Task with a TimeKeeper to use its TimeStr and Now methods.

Add a Now method to Task, so the borglet() getter method can be removed (which was only used by OOMCandidate to call borglet’s Now method). This replaces the methods on Borglet that delegate to a TimeKeeper.

Allowing Tasks to supply Now is a step toward eliminating the dependency on Borglet. Eventually, collaborators that depend on getting Now from the Task should be changed to use a TimeKeeper directly, but this has been an accommodation to refactoring in small steps.

Continuing the long-range goal of refactoring the Borglet Hierarchy.
```

這是個 refactoring 重構的 CL  
第一行說明 CL 做了什麼，並且怎麼改變舊的方法  
後面說明具體的作法、補充了目前的解法不是最理想的，後續可能有其他方向  
也說明為什麼需要這個 change  

小的 CL 也需要一些 context 的  
Small CL that needs some context  
```
Create a Python3 build rule for status.py.

This allows consumers who are already using this as in Python3 to depend on a rule that is next to the original status build rule instead of somewhere in their own tree. It encourages new consumers to use Python3 if they can, instead of Python2, and significantly simplifies some automated build file refactoring tools being worked on currently.
```

The first sentence describes what’s actually being done. The rest of the description explains why the change is being made and gives the reviewer a lot of context.
  

### Review the description before submitting the CL
review 期間，CLs 也可大改的  
送出 CL 前，也最好重新 review CL、描述，確保正確。  


### Small CLs
短、精簡的 CL 是為了
- review 快一點
- 讓 reviewer 在短時間內理解
- 太大的改變，讓 reivew 很吃力、灰心，通常需要反覆重複看好多次才能理解
  - 這樣就容易 miss 問題
- 更少的 bug，因為你改動比較少，所以不容易有 bug
  - reviewer 也更容易看出問題
- 更有效率，小 CL 被 reject 也不會浪費太多時間  
- 更容易被 merge。大的 CL 要處理的 conflict 就越多
- Easier to design well，小的地方比較容易改，code 也會比較乾淨
- 比較不會被 review blocking，不 block 自己，也不 block reviewer
- 因為改動少、改動的檔案也少，所以 roll back 簡單


### What is Small? 怎樣算 small
CL 是 self-contained change 就算 small size

- 最小的改變來針對一個事情
- 只是 feature 的一個部分，而不是整個 feature 一次丟上來
- 跟 reviewer 一起找出能接受的 size

所以你要了解的東西，只要能單單從
- CL
- CL’s description
- existing codebase

這些內容，你就能了解這個 CL 的話，那基本上就算是個 small CL

CL 很難理解的話，那就不算 small 了  
不存在具體的規則能指出大小，但
- 100 lines 通常是合理的大小
- 1000 lines 就太大了（但這要考量到 reviewer）

有時候也評估到 file change
- 200 lines 在 1 隻檔案裡面，這樣還ＯＫ
- 200 lines 在 50 隻檔案裡面，這樣**不ＯＫ**，太大了

Reviewers 很少會抱怨 CL too small ㄉㄚ

### When are Large CLs Okay?
有些情況的 大CL 都沒關係

- 刪除檔案（很多 lines change）通常沒關係，不會浪費 reviewer 時間
- 一些 大CL 的 change 是因為 auto tool refactor 之類導致的
  - 基本上對這些 auto tool 都是 100% 信任的

### Separate Out Refactorings
refactorings 的 CL best practice 就是
- 從 feature CL or bug fix 中拆出來，獨立 CL

舉例
- 搬 code 或 rename 的 CL 別跟 bug fix 一起
- reviewer 會輕鬆很多

小的 cleanups，例如修改 local 變數 name
- 就合在 feature change or bug fix CL 中

Refactorings 要不要拆出成獨立 CL，這需要 developers and reviewers 決定  
考量 review 會不會困難  

### Keep related test code in the same CL
test 的 code 應該要跟 feature 在同一個 CL 中

但，如果是其他**獨立的test**
- 那可以拆到不同 CL
- 還有一些 refactoring 的 CL 也可以拆（見下）
  - validating pre-existing, submitted code with new tests.
  - refactoring the test code (e.g. introduce helper functions).
  - introducing larger test framework code (e.g. an integration test).


### Don’t Break the Build
多個 CL 相互有關聯時，確保後續的 CL 不會壞 or merge 進去不會壞

### Can’t Make it Small Enough
還是有情況是，CL 沒法再小了，就是會這麼肥。  
但作者自己的經驗是，這種情況其實很少見
- 總是能找到方法把 CL decompose functionality 到各個小的 CL 中

提 大CL 前，先試試看
- 先提「單純 refactoring only」的 CL
  - 這可以讓 code 更清楚
- 跟 teammate 討論看看，有沒有想法可以拆這些 CL

真的沒辦法，就提吧（這情況應該要非常少見）
- 先跟 review 討論，取得他們的同意
- 讓他們有準備、有預期這個 review 會花很多時間

### How to handle reviewer comments 
該怎麼面對 reviewer 的 comments

### Don’t Take it Personally 對 code 不對人

review 的目的
- 維持更好的 code 跟 production
- 絕對不是對你 or 你的能力的 personal attack

有時候 reviewer
- 表現出他們的 frustration 或寫下 frustration 的 comments
- 這其實是不夠強的 reviewer 的行為

但身為 developer
- 你要有準備面對這些事情
- 問問自己，到底 reviewer 想要跟我溝通什麼事情？
- 絕對不要憤怒 or 情緒的回覆這些 comment
  - 這很嚴重的違法專業態度，而且這情緒會永遠殘留在 code review tool 中
  - 如果你太生氣了，那起來走走。冷靜後再正式回覆
  
如果 reviewer 沒有提供有建設性 or 禮貌的 feedback 2惡化
- 當面跟他說
- 沒法當面就 video call 或 私下 email
- 解釋說明他們的方式你不喜歡，說明哪些你喜歡的差異
- 如果他們還是持續回覆**沒有建設性**的 feedback 的話，適當的逐步往 manager 方向提吧

### Think for Yourself

無論你對此議題有什麼看法
- 都要退一步思考 reviewer 的 feedback 的價值，是否有幫助
- 所以你的第一個問題一定是「reviewer 說得對嗎？」

如果你沒法回答問題的話
- 可能是需要 reviewer 的提問更清楚一些

如果思考過後，你還是覺得你的想法是對的
- feel free 去回答並解釋為什麼你的方法是更好的
  - 從 codebase 面向 or
  - User 面向等等出發點
  
持續跟 reviewer 交換想法、資訊，直到達成共識

