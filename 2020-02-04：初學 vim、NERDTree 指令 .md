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
