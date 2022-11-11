# 2022-09-13：OSKAR DUDYCZ: How to successfully do documentation without a maintenance burden?
## OSKAR DUDYCZ, 2021-03-24
### https://event-driven.io/en/how_to_successfully_do_documentation_without_maintenance_burden/

前陣子在思考寫文件的事情  
覺得自己對文件維護好像沒有太多心得，剛好看到這篇讀讀看  

------------------------------------

Developers
- 喜歡抱怨缺乏文件
- 當他們不得不寫文件時，他們抱怨得更多
- 很少有專案能夠保持文件的更新
- 與其他任務相比，優先度通常很低。很多時候，它被留到了 "以後"

但，當你加入一個專案時
- 沒有什麼比保存良好的文件更好了
- 此外，當你回到一個幾個月沒有接觸過的老話題時，沒有什麼比描述性的文件更好了



OSKAR 分享些幫助保持最新文件的想法

## 1. 把你的文件放在資源庫裡。
- 不需要 Confluence、Sharepoint，沒有網絡共享
- 更重要的是，保持簡單，所以不要用 Word 等
- 最直接的方法是用 Markdown，對於文件來說應該是足夠好的
- 像 GitHub 和 GitLab 可以 preview Markdown 格式了
  - 還允許通過對其進行編輯和共同創作
- 此外，與 Word（和其他二進制格式）相反，它可以很容易地進行版本管理和差異管理
- 許多免費的工具允許生成靜態 HTML 網站，甚至可以託管在最簡單的網絡服務器上
  - 如果用 Github，可以用 GitHub Page
  - 也可以使用 Netlify，免費靜態 HTML 託管服務
- EventStoreDB (作者公司的產品) 使用 `VuePress`，在 `Marten` 中使用 `Storyteller`
  - 這兩個工具都允許直接從 source code 中動態地附加 code snippets

## 2. 把 documentation 納入 Code Review process 中 (Include documentation in the Code Review process): 
單單把文件和原始碼放在一起還不夠
- 與 Confluence 等相比，它的實際優勢是什麼？
- 如果它在 repository 中，可以讓你驗證改變的邏輯是否在文件中得到更新
- 可以查看變化的歷史，並檢查是什麼程式碼變化觸發了更新，或者文件是如何演變的
- 定期維護文件比時不時地分批維護要容易得多
- 使用 code review 工具可以讓我們進行高效的異步討論
- 對我來說，在 Confluence 上進行評論（進行 thread 對話）是一個悲劇
  - 在 Github 或 Gitlab 中參考特定的程式碼行要容易得多
  - 把所有的討論放在同一個地方，通過對技術和商業決策的單一真相來源來提高可發現性

## 3. Create immutable documentation:
幾年前，`Michael Nygard` 寫了一篇文章 [記錄架構決策 (Documenting architectural decisions)](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)  

他提出了架構決策記錄的想法，一個關於架構決策的簡潔的記錄應該包括:

- **標題(Title)**: 這個改變是關於什麼的
- **背景(Context)**: 需求來自哪裡
  - 在這裡描述決策的背景和關於當前狀態的事實
  - 可以同時描述你的商業案例和技術案例
- **決策(Decision)**: 為我們的問題提出的解決方案
- **狀態(Status)**: 例如，`proposed`, `accepted`, `rejected`
  - 當然也有 `rejected`， `rejected` 帶有關於我們思考過程的實質性價值。當 context 發生變化時，我們也可以回到被拒絕的想法，再次分析它們
- **後果(Consequences)**
  - 決策帶來的 positive and negative effects
  - 一定不要淡化 or 粉飾負面後果，我們應該展示 proposed 的解決方案的對立面和風險

同樣重要的是
- 要做到透明，並邀請所有的 stakeholders 來審查它
- 在我之前的專案中，我們將此類專案發到 forum（slack）
  - 每個人都被告知並被邀請在審查中
- 最好讓其他團隊的人也發言。這個變化往往可能直接關係到他們（例如 shared contracts 的變化）或間接關係到他們（一個全局性的架構變化，例如**授權**）
- 另方面是，我們可能會在一個團隊中陷入隧道思維。一個新的視角可以發現錯誤或缺失的部分

最後的、關鍵的部分。經批准的文件不能被改變
- 如果發現需要更新決定，應該發送一個新的ADR
- 得益於此，一旦添加，文件就不需要更新和維護。我們只是在記錄下一組決定

## 4. 過程自動化 (Automate the process)
有很多工具可以使這個過程不費力氣。沒有必要寫一些自定義的腳本，或者全部由你自己完成
- 如果你構建的工具希望被他人使用，擁有良好的文件也是至關重要的。如果沒有好的文件，它被使用的機會幾乎為零
- 鼓勵你看看我的《[架構師宣言](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)》，我在其中描述瞭如何把它放入整個架構過程中

