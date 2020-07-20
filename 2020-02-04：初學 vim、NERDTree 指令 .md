# 2020-02-04：初學 vim、NERDTree 指令

還沒決定要使用，但調教看看  
下面記錄一些，符合我使用習慣的快速鍵
- `t`: 在新 Tab 打文件，並跳到新 Tab
- `T`: 在新 Tab 打文件，但不跳到新 Tab
- `gT`: 切換前一個 tab
- `gt`: 切換後一個 tab
- `ZZ`: 關閉 tab
- `ctl + d`: 下一頁
- `ctl + u`: 上一頁


search word 後
- `n`: 切到上一個 keyword
- `N`: 切到下一個 keyword

啟動 fzf
- `:Files`: 啟動 fzf
  - `tab`: 多選檔案
  - `ctl + t`: 用 tab 開啟
- `:GFiles`: 啟動 fzf，會吃 gitignore
- `:Ag`:  啟動 The Silver Searcher

refresh
- `r`: refresh NERDtree、如果檔案有從外部改變的話，就這樣 refresh

Multiple cursors
- Vim 要 Multiple cursors 目前是需要 plugin
- 但有人建議用 Vim 的功能也行
- 所以步驟是
  1. `/` 先 search 要的 work ->  /flex (enter)
  2. `cgn` 輸入要的字 -> inline-flex
  3. 按 `.` 下一個也取代 （一直按 `.` 一直往下取代

單行間移動 (https://mropengate.blogspot.com/2015/07/vim-ch2.html)
- `w` 移至次一個字字首(英文單字)。
- `b` 移至前一個字字首(英文單字)。
- `0` 移至行首
- `$` 移至行尾

回復
- `u`: undo last change

剪下一行、貼上一行
- uppercase `V` to select whole lines
  - (Press `v` to select characters)
- Press `d` to cut (or `y` to copy).
  - Move to where you would like to paste.
- Press `P` to paste **before** the cursor, or `p` to paste **after**.

插入一行
- `O` add new line before
- `o` add new line after

選取 or 清除 "",(),<> 中間的內容
- `ci"` 清除 "" 中間的內容，並且進入 insert 模式
  - `ci(` 清除 () 中間的內容，並且進入 insert 模式
- `vi"` 進入 visual mode，並且選取 "" 中間的內容
  - `vi(` 進入 visual mode，並且選取 () 中間的內容

很清楚的 Vim Cheat Sheet
- https://vim.rtorr.com/lang/zh_tw