# 2022-04-02：How GitHub Actions renders large-scale logs
## Alberto Gimeno March 25, 2021
### https://github.blog/2020-09-23-a-better-logs-experience-with-github-actions/
---------------------

整篇並沒有寫什麼技術細節，偏向列出做了哪些改動。


## Why these changes matter
在思考什麼是成功的 automation 時，我們思考的目標是
- 目標是花最少的時間查看自動化的內容
- 這樣就可以將注意力集中在相關的內容上

但結果沒有如我們所料，所以我們需要 review 目前狀況
- debugging 的過程可能令人沮喪
- 這就是為什麼我們引入一系列改進來提高 perf 和 UX 的原因

改動
- 簡化 layout 結構
- 引入了單一且更快的 virtualized scrolling
- search 現在更 responsive
- 更好的 ANSI、8 bit 和 24 bit 顏色支持
- URL 現在是交互式的
- 全新的 full-screen view mode
- 更新了的UI，提高了 readability and overall interactions


![](https://github.blog/wp-content/uploads/2020/09/preview.png)  


## Layout structure and a single scroll
Github
- 建立了一個清楚的 layout 結構
- 讓 User 能在 debug step 時，保持當下的 context 下，能快速地切換 jobs

![](https://github.blog/wp-content/uploads/2020/09/structure.png)  

當 logs 密集時，這些都非常重要。我們希望 User 追蹤 step 時，能在不同的 search results 間輕鬆切換


## 改善 readability and interactions
所有 log 中的 element 都改善了對比度、對齊方式、accessibility，並且 desktop 和 mobile 都改善了 interactions

![](https://github.blog/wp-content/uploads/2020/09/readability.png)  

這些改變，改善了 reading experience，User 在掃視 log 時更快能找到目標  

search 也有微小的視覺上改動和 interaction 的更新，另外，我們定義了一個 threshold，來避免系統搜尋了 unintentional matches 來改善 perf。這些改動，在 content-heavy logs 時，效果非常好


![](https://github.blog/wp-content/uploads/2020/09/search.png)  

顏色 是強大的視覺資源，我們希望能更加 mindful 的使用它，來引導 User 到正確的位置
- 因此重新設計了在 log 輸出中使用顏色的方式
- 如果 task 正常，log 看起來將大致相同
- 如果 task 正在進行中、等待 User的輸入或者甚至失敗，現在可以快速找到需要關注的東西並開始對下一個任務採取行動。

![](https://github.blog/wp-content/uploads/2020/09/steps.png)  

## More colorful experience
我們希望更加注意顏色上的使用，需要讓 User 能建立自己的 scripts, commands, and tools to output useful information.

所以支援了這些顏色
- ANSI colors
- 8-bit colors
- 24-bit colors

這些顏色，讓呈現來自第三方來源的 data 時，有更豐富的內容和更好的整合


![](https://github.blog/wp-content/uploads/2020/09/color.png)


