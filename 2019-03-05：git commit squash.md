## git commit squash

1. `git log --oneline`  
先執行 log 查看 commit  
假如有 14 筆 commit 我要整併，那就下  

2. `git rebase -i HEAD~14`  
接著編輯  
不要的 commit 好像是設定 f  
改寫的好像是設定 r  
編輯完後存檔 wq  

3. `git push -f`
若 remote 有的話，要 force 
