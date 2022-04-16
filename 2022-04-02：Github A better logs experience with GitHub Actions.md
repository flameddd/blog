# 2022-04-02：Github A better logs experience with GitHub Actions.md
##  Adrian Mato Gondelle September 23, 2020
### https://github.blog/2021-03-25-how-github-actions-renders-large-scale-logs/


為了 render 大量的資料，必須慘用 virtualization，目標
- 要能呈現資料的 subset
- 不能讓 User 察覺到在可視範圍 viewport 內的東西還沒 render 出來
- 需要 User scroll 時，就能看到內容，所以還沒顯示出來的資訊也需要計算 layout，來保持 scroll 時的 UX
- ...

## Github 初次嘗試
第一次嘗試時，測試了一個 react base 的 library，和另一個 vanilla JS 的 library  

有些 library 有些限制，這是因為它們實作的方式的關係，例如
- 所有的 item 必須有「固定的高」 <- 這讓 library 在計算方面，變得輕鬆很多
- 只需要大概用 `item_index * items_height` 計算，就能判斷了

但對 Actions 來說，這不適用  

當時，最後選了 vanilla JS，它有大部分所需的功能、可變動的 item 高度等等  
但，漸漸地看到一些 bug 和 bad UX  

- 要求可 scroll 的區域必須有一個固定的高度，但這使得 GA 的 UX 很差。因為每一個 steps 都有獨立的 log，所以「整個頁面」和「各自單獨的 steps」都是可 scroll 的區域
- virtualized list 從 hidden 切換到 visible 的部分，沒有被好好的測試過。在 GA 中，允許 User
  - expand and collapse steps
  - 也有時需要自動 expand logs
  - 在這些狀況時，就發現一些 bug
- User 沒法同時 scroll 和「選取字」，因為是 virtualization，那些「選取」最後都回從 DOM 上移除
- 在某些情況下，render 很慢，因為我們必須在 background 先 render 這些 log line，這樣才能正確計算高度，避免 UI 跑掉。但這對 virtualization 其實沒有幫助，我們等於是 render 了兩次


## 重新思考需求和方案
由於上面這些因素，所以開始重新思考，到底有沒有需要 virtualization
- 我們能根據實際使用情況做出決策
- 絕大多數 User 的 log 小到可以在沒有的情況下呈現它們
- 我們可以移除 virtualization，但允許單獨下載更大的日誌。


我們的 data 顯示
- 99.51% 的現有作業的行數少於 50k
- 但我們知道超過 20k 的 log ，瀏覽器就開始有狀況了
- 我們還發現，即使日誌行數很少，也可能會佔用過多的 memory

有了這些資訊
- 我們決定不需要 data virtualization
- 但我們仍然需要 UI virtualization
- In the very edge case of having a very large log file with a low number of log lines, we truncate it and provide a link to download it.

## 從頭寫自己的  virtualization library

Github 決定自己來，目標是
- 能夠 render 至少 50k 行的 log，理想情況下是在 mobile 上也能使用
- User 能不受任何限制的「選取文字」(能 scroll 著選)
- 絕大多是情況下，UI/UX 都是順的，包含「跳到搜尋結果」、點開 `permanent links` 和 `streaming logs`
- 只有一個沒有固定高度的 scrollable area
- 能讓 Steps 有 sticky header
- 使計算盡可能快、less memory，就算不是完全準確的計算也可以

為了這些目標，以下幾個方面，必須採用跟其他 library 不同的做法：  
rendering 前，預估高度
- 大多數的 virtualization library 都是依靠精準的計算、或者是固定高然後利用 `position:absolute` 來實作，這樣相對簡單很多
- Github 決定用「預估高度」的概念，來避免 costly calculations
- 當上面的預估不準時，用 `position:relative` 來避免 log line 被截斷

DOM 結構
- 為了要能夠 sticky header 和只有一個 scrollable area，如何設計 DOM 結構是另個挑戰
- scrollable container 必須包含這兩個、但 sticky headers 就絕對不是 virtualize 的

經過測試，雖然驗證了能夠自己實作 virtualization 來達到目標，但也注意到必須要小心
- 例如，我們很快意識到，盡可能少地 render DOM 節點很重要
- 在 scrolling 時盡可能減少 DOM mutations 也很重要
  - 當 User scrolling 時，我們需要 add 新的可見 DOM，並且 remove 不可以見的 DOM
  - 如果 User scroll 非常快，尤其在 mobile 上，太多的  DOM mutations，UX 就變差
  

我們通過幾種方式解決這些問題
- 例如，throttle 執行 code，這樣來 batch update，減輕執行太頻繁
  - 但這做法，讓 UI 感覺不順
- 之後想出將 log 分組到 cluster 中，而不是單獨一行行的 add/remove
  - 把 log 集中在一群裡面來 add/remove
  - 測試過後，決定 50 條 log 為一個 cluster 
  
(文章後面就沒什麼了，有點斷在這邊的感覺 ...，只寫道，還有很多 UI/UX improvements 要做。這些 UI/UX improvements 另外有一篇文章 https://github.blog/2020-09-23-a-better-logs-experience-with-github-actions/ )