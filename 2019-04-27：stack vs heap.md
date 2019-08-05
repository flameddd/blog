## stack vs heap

變數會佔用記憶體，記憶體分為三個部份來存這些變數
- global: 用來放全域變數、靜態變數(static)等等。
- stack: 堆疊（中國棧）
- heap: 堆積（中國堆）

### stack: 堆疊（中國棧）
區域變數、函式的參數與函式的位址等等，由系統管理，**必須在編譯時期為已知**  
- 變數的回收會發生在它從堆疊pop出去
- 因為個數、大小什麼的都是已知，所以系統知道怎麼進行配置與回收
- 用於靜態記憶體配置
- 在編譯時期系統已經可以預知 x 從何時開始配置跟何時結束回收(當然就是看所屬block結束就跟著回收)
- 由於配置跟回收的規則明確，當然就往stack擺囉。
- 為何是存放在stack呢，因為函式的呼叫有後進先出的概念
  - 當method1()被呼叫而開始執行
  - 待結束時必定會查找該返回何處
  - 故最後一定會讀取函式的返回位址，既然如此明確而有條理
  - 當然也是往stack放！
- 可預測性外加後進先出的生存模式，令stack無疑是最佳的存放策略
- 程式語言中變數跟函式的生命週期皆為後進先出的概念
  - 也就是越晚產生的會越先被回收或銷毀
- 正因如此只要是可預測性的相關資訊都是往stack存放

### heap: 堆積（中國堆）
heap記憶體
- 用於動態記憶體配置
- 由使用者負責進行回收
- 配置則是由malloc或是new來負責
- 為何在寫Java都不需要注意 heap 回收空間的問題？
  - 因為Java GC 的機制自動檢查Heap中哪些資料已經沒有被使用
  - 當確認資料已經沒有使用會自動將空間回收

使用這裡的記憶體
- 主要是用在編譯時期還不知道大小或個數的變數

例如說，需要用一個陣列，這個陣列的大小要
- 在執行的時候由使用者的輸入來決定
  - 那你就只能使用動態配置，也就是把這個陣列配置在heap中

### 為什麼stack快
高手經驗，當記憶體用**到 1G 後，stack 比 heap 快很多**


### 網路上一些總結
- value type 的變數, 包括「指標」變數會放在 stack
  - reference type 的變數 (如 string, object) 本身也會放 stack
  - 然而他的值 (value) 則是放 heap
- box 就是 `value types` to `reference types` 的過程
  - 所以 value 會被放到 heap 中
  - 而產生一個 object 變數來`指向這個 value`
    - 變數指標則是在 stack
- unbox 是 `reference types` to `value types` 的過程
  - 原本 object 所指向的值 (heap 中) 會被複製到 stack 中
  - 並賦予明確 value type 型別

#### 深入 Stack
#### 記憶體 - 資料存放的角度
stack 的操作特色是
 - LIFO (last in, first out)
   - (相反的叫 FIFO, 一般的 queue 就是這個行為)

#### 記憶體 - 程式運行的角度
進入 main() → 呼叫 foo() → 結束 main() 的過程  
細節來說, 是這樣的 :
1. 進入 main() 
2. 儲存 main 區域變數與呼叫參數 
3. 呼叫 foo() 
4. 儲存 foo 區域變數與呼叫參數
5. 結束 foo()
6. 結束 main()

精緻版 : 
1. 進入 main()
2. 儲存 main 區域變數與呼叫參數
3. 配置 foo 回傳結果位址
4. 呼叫 foo()
5. 儲存 foo 區域變數與呼叫參數
6. 結束 foo()
7. 回存 foo 回傳結
8. 接續執行 main()
9. 結束 main()
 
### javascript
- Does JS uses stack or heap for memory allocation or both?
- I know JS uses heap for achieving closures
  - does it also use stack?
  - What are the use cases for `what` it uses and `when` it uses? 

### V8
V8 uses a heap similar to JVM and most other languages
however, means that
- local variables (as a general rule) are put on the stack
- objects in the heap.

### how differ, when are usually used in classic system-languages, like C. 
#### Stack
- First-In-Last-Out
- When a new thread is started by a computer program a new stack is created
  - (usually the main thread first ;) ), 
- It is small and can be used to quickly store and retrieve temporary data
- In C, The stack is usually used for low-cost data
  - like simple data types (integer, float, pointer ( is also just an integer))
  - to keep it speedy and prevent a stack-overflow.

#### Heap
- called "free store"
- used to store arbitrary data in an unordered fashion
  - that's why it's a lot slower
- used for data structures in classic languages
- In order to quickly find heap-data when doing operations, a pointer to it is stored on the stack.



#### V8
- JS is a garbage-collected language
- GC means that there is data scattered around which has to be cleaned up

Does that sound like the data is stored on the stack?Rather... no, because
- stack has strictly ordered data
- So, variables are allocated on the heap

從 V8 原始碼找答案
- a class called "Boolean" is used
- 當使用一個 boolean 值時，"Boolean" class 被 instantiated
- C++ land. C++, like C 儲存 data structures on the heap
  - and only works with pointers to it in the stack

所以當 V8 需要 allocate 一個新 variable 時
  - V8 allocates a structure holding the data 
  - 加上額外的 information on the heap

有趣的是 V8 後面的 optimizer
because if a variable is always used as a boolean, it might as well drop the structure and use it as a boolean on the stack, which would bring us to system-performance without all that soft abstraction stuff on top.
（這段沒法想像，翻中文更不懂，直接貼了）
