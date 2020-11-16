# git revert last commit single file
## https://thai-lin.blogspot.com/2018/08/gitcommit.html

## before
You may need **stash** un-commit code
```
git add .
git stash
```

## revert

```
git reset HEAD~ 
```

then, **revert** or modify the file you want to revert.
then, `add` and `commit`

```
git add .
git commit -m "xxxx"
```
