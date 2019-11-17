## git 計算分別每人的 commit 次數

```
git shortlog -s -n --all --no-merges
```

Added `--no-merges` to exclude statistics from merge commits.

`--all` 這個我不太確定有什麼差別  
```
Tell the command to automatically stage files that have been modified and deleted, but new files you have not told Git about are not affected.
```