# 2022-08-26：Hack Pipe for Functional Programmers: How I learned to stop worrying and love the placeholder
## September 11th, 2021 James DiGioia
### https://jamesdigioia.com/hack-pipe-for-functional-programmers-how-i-learned-to-stop-worrying-and-love-the-placeholder/

James DiGioia
- 參與 JS 的 pipe operator (`|>`) 開發已有 4 年
- 2017 年，當時它只是一個簡陋的 pipeline，做基本的函數應用
- proposal 演變成了兩種相互競爭的語法：`F#` & `Smart-Mix`
- 探索了處理 `await`、`arrow functions` 和整體組成方法的不同方式

James DiGioia 強烈支持 `F#`
- 因為 James DiGioia 更喜歡將值應用於函數的簡單性和功能性
- 且我力主張採用 `F#` 而不是 `Smart-Mix`，因為認為後者過於復雜

此後，`Smart-Mix` 語法演變為目前的 Hack iteration
- 放棄了 bare style（`x |> a.b`，沒有佔位符(placeholder)）
- 相應地簡化了其語法

大約在那個時候，這個提議的前輩 Daniel Ehrenberg 下台了，Tab Atkins-Bittner 接替了他的位置
- 雖然 Daniel 傾向於 `F# pipe`
- 但 Tab 之前傾向於 `Smart pipe`，現在則傾向於進化後的 `Hack pipe`
- 他一直在努力使 `TC39 committee` 就 `Hack pipe` 達成共識

由於 Tab 現在承擔了 champion 的責任
- James DiGioia 一直在和他們以及 champion group 的其他成員討論這個提議
- 雖然 James DiGioia 最初中主張採用 `F# pipe`，但現在相信 `Hack pipe` 是一個更好的選擇
- 在核心方面，`Hack pipe` 是連接主流和 functional JS 的一個更好的橋樑，使主流 JS 更容易採用 functional patterns

## The functional/mainstream 的分裂

現在
- functional community 與 JS 生態系統的其他部分被切斷了
- 為了實現 FP，你需要使用專門設計的函數式庫
  - `lodash/fp` 是個著名的例子；`Ramda` 是另一個

雖然這些庫的不變性和 side-effect-free 的行為對 functional programmers 和更廣泛的 JS 社群都非常有價值
- 但它也帶來了 FP 中一個更複雜的概念的 cost：currying。

## 傳統的 currying vs JS currying
傳統的 currying 是指
- 一次只接受一個參數的 function，在收到所有的參數後再返回一個新的 function

要直接模仿這一點，必須在 JS 中寫出這樣的 function

```js
const add = a => b => a + b;
add(1)(2); // 3
```

這對 functional composition 來說是很好的
- 它是一個小的、狹義的函數
- 可以和其他小的、狹義的函數結合起來，解決更大的問題
- `add` 可以部分應用於創建函數，並在更大的 composition 中使用它

```js
const doubleThenAdd2 = pipe(multiply(2), add(2));
doubleThenAdd2(5); // 12
```

但即使有這樣的好處，這顯然不是寫 functions 的正常方式
- 如果以正常的方式調用它們，它只接受第一個參數，然後丟棄後面的一切

```js
add(1, 2); // b => 1 + b;
```

這就是為什麼他們發明了 `curry` function

```js
const add = curry((a, b) => a + b);
add(1); // b => 1 + b;
add(1, 2); // 3
add(1)(2); // 3
```

curry 接收一個普通的 function，並返回一個在兩種形式下都能工作的 curry 版本
- 核心部分，curry 的工作方式是對所提供的參數進行循環計算，建立其內部的參數列表
- 如果 function 缺少參數，它會返回一個新的 function，該 function 接收剩餘的參數
- 如果提供的參數數量與 function 的長度一致，就會調用底層 function

有了 curry，使不是為 functional composition 而寫的 function 可以被組合
- curry 是更傳統的 FP 方法（currying 是內置於語言中的）和 JS 的行為預期行為之間的橋樑

## curry 是座 suboptimal 的橋樑

雖然它確實提供了一座橋樑，但
- 這座橋樑實際上加劇了 functional 和 mainstream JS之間的鴻溝
- 它是不完美的，它試圖為 JS 帶來非語言原生的功能
- 它是複雜的，使 FP 對 mainstream JS 開發者來說更加困難

最終產生了一種 FP 的方法，從根本上說是不適合 JS 的


## curry 是座不完美的橋樑
 
如果你一直在用 functional JS
- 你可能會遇到一些 curry 的變種：`curry2` 和 `curryN`
- `curryN` 需要一個額外的參數，即 function 的長度，並返回一個在提供了該參數數量後進行評估的 function
- （curry2 專門處理需要 2 個參數的 function ）
- 這是必要的，因為在 JS 中，使用 `length` 可能並不意味著你想要什麼

第一個例子是在 JS 中如何處理默認參數。在下面的例子中，sort 的長度是 1，而不是 2
```js
const sort = (arr, algo = 'bubble') => // ... implementation
```

默認參數不計入 function 的 length
- 所以你需要明確告訴它，因為 JS 認為這是一個只有一個參數的 function

另一個不好用的模式是 options-bag parameters  
下面的例子如果不經過一些重要的跳轉，根本就不能用 curry

```js
const api = ({ url, method = 'GET', body = {}, headers = {} }) => // ... implementation
```

為了把這些東西變成 composable functions
- 必須把它們包裝在一次次接受這些選項的 function 中，這充其量是很尷尬的
- 如果第二個參數是 options-bag argument，再加上一些默認參數，情況就更糟糕了

這些都不是傳統意義上的 curry
- 在大多數情況下，可以通過將 api 包裝在一個 arrow function 中來解決，讓它做你想做的事情
- curry 沒辦法把它變成可 composable


## curry 是單向的橋樑

curry 是一座單向的橋樑
- 那些不那麼容易 composable 的 functions 需要被做成可 composable，以便與更廣泛的 functional 生態系統對接

但這是不對等的  
- 以上面的排序函數為例，它是以 mainstream 的習慣方式編寫的
  - data 在前，可選的、默認的參數在後
- 但為了使其可 composable，需要使第一個參數成為算法

```js
const sort = curry((algo, arr) => // ... implementation );
sort('bubble', [5, 214, 23, 6]); // [5, 6, 23, 213]
const sortBubble = sort('bubble');
```


這很奇怪!
- 現在默認值是必須的，而 data 是第二位的
- 在這情況下，你可以為每種算法創建一個單獨的函數
- 但對於其他有默認值的函數，這可能沒有意義
- 真的想撇開這個值，但讓它變成 curried 會使編寫一個 mainstream function 更加困難

在這種情況下，默認參數只是字符串，但如果它是個 options-bag 呢？這就變得更奇怪、更複雜了。


所有這些都意味著
- 必須專門為這類 function 專門寫一個 curry 式的 function
- 而這些 function 往往跟 mainstream style 不一樣
- curry bridge 只是把 function 帶入了 functional JS style；並沒有把 function 送回 mainstream style
  - 而寫 mainstream JS 往往因為這種不匹配而難以採用 FP 模式。


## curry 是一個複雜的橋樑

雖然 curried functions & compositions 一旦寫出來就會成為非常優雅的代碼
- 但出錯時，debug 可能是...很有挑戰性的
- 你失去了對參數的實際來源的可見性，pipelines 可能很難在 debugger 中穿行
- 如果沒有對每個底層 function 的作用的強烈意識，系統的整體行為可能會變得不透明

所有這些都使得 JS 中的 FP 對初學者來說很難
- 即使是基本的 add function，看起來很簡單，但 new programmers 也要花上一秒鐘才能意識到這是一個返回 function 的 function

Functional composition 是種非常強大的思考問題的方式
- 而 functional 成語讓 non-functional programmers 更難理解
- 因為他們在利用它之前需要了解幾個概念
- 有些工具，如 `ramda-debug`，使 debug 這些 pipelines 更容易
  - 但卻加劇了需要選擇進入一個全新的生態系統來使用這些工具的問題，並加深了 functional 和 mainstream JS 之間的分歧


## Curried (& point-free) code is un-JS-y

以上述方式組成的 functions 在 FP JS 中被廣泛使用，而在它之外很少使用  
每當初學者遇到像這樣的代碼時

```js
const doubleThenAdd2 = pipe(multiply(2), add(2));
doubleThenAdd2(5); // 12
```

為了理解這裡發生的事情
- 需要學習 4 個以上的概念
- 再加上驗證和 debug 這些組合的新工具和技術
- developers 發現它「難以掌握，難以閱讀」，需要一些時間和經驗，才能使上述的功能變得直觀

James 早期廣泛使用 `Ramda`
- 並試圖把它和它的相關概念介紹給團隊。儘管一些人是 senior，但這需要大量的工作來解釋
- 這從未成為他們的 natural approach

在這個過程中，James 也為調試自己的 pipelines 而掙扎
- 後來放棄了 `Ramda` 及其相關技術，將其作為 library

這些概念的 overhead 使得在更廣泛的 JS 社群中採用其他功能化概念變得更加困難
- 如果不採用整體的 functional style，它們就不能很好地整合到 non-functional code 中
- mainstream JS和 functional JS 都因此而受到影響，因為 mainstream JS 無法從 functional 的理念中獲益
- 而 FP 則被貶低為玩自己的玩具，而不是分享其他人的玩具。

## 拒絕 currying；擁抱 composition

在 JS 中引入 native pipe operator，本身就可以解決很多問題
- 不再需要 pipe 或 compose function
- 而且 pipe 的行為比這兩個 functions 都更直觀
-  `x |> func` 看起來像「x 進入 func」

此外，debug 更容易
- 有了集成到 JS 中的 pipeline，開發工具可以更容易地在 pipeline 步驟間加 breakpoints，檢查任何特定步驟的輸出，而不需要額外的工具
- 類似你可以在 one-line arrow function 之後加一個斷點
- 檢查 callstack 和 arguments。僅僅這兩個特點就使 pipeline 的價值對語言來說是值得的

總的來說，在這兩個方案之間
- 大多數 functional programmers 更喜歡 `F#` 方案
- 看一下帶有 `F#` 管道的 `doubleThenAdd2`

我們看到的是對當前用法的直接翻譯。

```js
// const doubleThenAdd2 = pipe(multiply(2), add(2));
// doubleThenAdd2(5); // 12

const doubleThenAdd2 = x => x |> multiply(2) |> add(2);
```

functional programmers 可以很直接地將當前的 pipe 轉換為 `|>`
- 因為它的工作方式與他們在 FP JS 中已經習慣的方式相同
- Currying & point-free 是 functional programmers 在 JS code 中的重要組成部分
  - 因此他們希望有個 pipe operator，能夠繼續這樣做。

但是...如果不是這樣呢？`Hack pipe` 的版本看起來是這樣的

```js
// const doubleThenAdd2 = pipe(multiply(2), add(2));
// doubleThenAdd2(5); // 12

// const doubleThenAdd2 = x => x |> multiply(2) |> add(2);

const doubleThenAdd2 = x => x |> multiply(2, ^) |> add(2, ^);
```

現在，甚至不再需要 currying
- 我們可以使用基本的、沒有 curried 的 add function，而不用修改
- 因為 Hack 允許任意的 expressions，而不是 functions，這實際上可以更簡單。

```js
const doubleThenAdd2 = x => x |> 2 * ^ |> 2 + ^;
```

甚至不需要這些運算符的 functional 版本！可以組成運算符，而不僅僅是 functions。我們可以組合運算符
- 不需要為它們引入或編寫整個 library

整個 non-curried libraries 的世界都被開放給了 functional composition
- 不再需要 `lodash/fp` 作為 lodash 的 base，因為基本 lodash functions 在 pipe 中 work

更重要的是，這 operator 比 curry 要容易理解得多
- Beginning programmers 了解 expression
- 有了Hack pipe，為了開始利用 composition 的優勢，就不再需要學習一堆概念了

Functional programmers 可以不再解釋 currying & point-free
- 而是專注於將其他功能
- Mainstream JS 可以開始使用 Maybe 和 Either 這樣的結構

因此，Hack pipe 對 FP 很有好處
- 它彌合了 FP 社群和更廣泛的 JS 社群之間的鴻溝
- 一大類互操作工具消失了，取而代之的是一個 mainstream & functional JS 都能利用的 native operator


##  FP 的未來是 Hack pipe

在最近的　TC39　會議上，Hack pipe 被推進到 **Stage 2**
- 希望這個消息，尤其是考慮到長期以來對 `F#` 的偏愛，會在 `F#` 的信徒引起一些驚愕
- 然而，Hack 是 JS 中 FP 的一個勝利，即使它意味著放棄了一些歷史悠久的 functional 傳統

生態系統肯定需要時間來適應 Hack pipe 的引入，儘管 curried functions 可以利用現有的 pipe operator


```js
// Without `curry`
1 |> add(1)(%);
// With `curry`
1 |> add(1, %);
```

一些代數 data 類型可以很好地與 Hack pipe operator 配合
- 支持 tree-shaking
- 並且對初學者來說是可以理解的
- 這將大大有助於使 FP 在 non-functional 背景下更加習慣，將他們與更複雜的函數式解決問題的方法脫鉤

JS 中的 FP 長期以來一直採用其他 functional languages 的技術和工具
- 這就是為什麼 currying & point-free programming 在 FP JS中變得如此流行
- 但 JS 不是 Haskell，currying 也不是 JS built-in 的
  - 所以相比之下，採用它總是有點不自然
 
我們應該採用一個讓人感覺是JS native 的 pipe operator，並調整我們的技術，為大眾提供更自然的 functional JS