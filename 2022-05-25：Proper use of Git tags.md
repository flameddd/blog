# Proper use of Git tags
## Dan Aloni 23, 2022
### https://blog.aloni.org/posts/proper-use-of-git-tags/

Git tag 使用上的一些建議

---------------------------

### Tag push permissions

一個常見的陷阱是讓任何人推 Git tag，以下情況經常：
- 有人推了一個 `test` tag
- 其他人根目錄中創建了 `test` subdir

這時候，`git log test` 就混亂了
- 它應該顯示從最初 `test` 的 commit ?
- 還是來自 HEAD 修改該 test directory?
- 或者也許 local 也有一個 test branch?

所以，可以的話，就限制 Git server 只接收 release managers 送的 tag

### Tag naming

tag 是獨一無二的，並且永遠存在於 repo 中。
- 選擇不會與 branch name 產生歧義名稱

好的 tag convention:
- 要標記版本
- `v` 起始 (有利於 shell auto complete)
  - 如果立即使用版本號 `.eg 1`，auto complete 將同時顯示所有 tag and git commit hash 開頭為 `1` 的 data


### 只用帶註釋的 tag ，不使用 lightweight tag 


默認情況下 `git tag` 會 create lightweight tag (non-annotated tags)
- 用 `-a` create annotated tags 
- meta-data 會一起被 registered
- `git describe` 也不需要使用 `--tags` 了


### 不要單獨依賴 tag 進行版本控制

Git tag 只是 Git 的補充資訊
- 它本身不是 source
- 要同時用 `Git tags` and `source tree file` 來做版本控制
  - 這通常是 ecosystem 的 package manager manifest 中的 version string
  
但該 tag 哪些 commit ? 看下一點


### 改 source 中 version 的 commit 是要 tag 的 commit

按照慣例，更改 source 中 version 的 commit 是應該 tag 的 version
- 而不是將其合併到 main branch 的 commit
- 這樣，`git describe` 才能顯示了預期的結果