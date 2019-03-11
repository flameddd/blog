##  Git commit message

### 用一行空白行分隔標題與內容
### 限制標題最多只有 50 字元
 以 50 個字為目標，最多 72 個字為硬規則。經驗法則這比較好閱讀
### 標題開頭要大寫
超簡單的規則，如同字面上的意思，任何標題的開頭都要大寫。  
:( Bad  
> accelerate to 88 miles per hour  

:) Good  
> Accelerate to 88 miles per hour  

### 標題不以句點結尾
結尾的標點符號在標題列是多餘的，要遵守 50 字或以下的規則時，字元空間是很寶貴的。

:( Bad  
> Open the pod bay doors. 

:) Good  
> Open the pod bay doors  

### 以祈使句撰寫標題
祈使句有時聽起來有點不禮貌；因此我們並不常使用。但是對於 Git commit 標題來說卻很適合。其中一個理由是因為 Git 本身就是依照你的命令來完成一個動作。

舉例而言，`git merge` 預設的 message 是：
> Merge branch 'myfeature'

當使用 `git revert`：
> Revert "Add the thing with the stuff"
> This reverts commit cc87791524aedd593cff5a74532befe7ab69ce9d.

或是在 GitHub PR 按下 “Merge” 按鈕的時候：
> Merge pull request #123 from someuser/somebranch

一個正確的 Git commit 標題應該要能夠代入下面的句型，使之成為完整的句子：  
If applied, this commit will **<你的標題>**  
舉例而言：  

If applied, this commit will **refactor subsystem X for readability**  
If applied, this commit will **update getting started documentation**  
If applied, this commit will **remove deprecated methods**  
If applied, this commit will **release version 1.0.0**  
If applied, this commit will **merge pull request #123 from user/branch**  

### 內文每行最多 72 字
### 用內文解釋 what 以及 why vs. how

Bitcoin Core 的這個 commit 給了一個很棒的範例，解釋了什麼改變以及為什麼：
```
commit eb0b56b19017ab5c16c745e6da39c53126924ed6
Author: Pieter Wuille <pieter.wuille@gmail.com>
Date:   Fri Aug 1 22:57:55 2014 +0200
 
   Simplify serialize.h's exception handling
 
   Remove the 'state' and 'exceptmask' from serialize.h's stream
   implementations, as well as related methods.
 
   As exceptmask always included 'failbit', and setstate was always
   called with bits = failbit, all it did was immediately raise an
   exception. Get rid of those variables, and replace the setstate
   with direct exception throwing (which also removes some dead
   code).
 
   As a result, good() is never reached after a failure (there are
   only 2 calls, one of which is in tests), and can just be replaced
   by !eof().
 
   fail(), clear(n) and exceptions() are just never called. Delete
   them.
```