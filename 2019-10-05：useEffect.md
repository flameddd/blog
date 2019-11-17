## useEffect
### 2019年3月9日 Dan Abramov
#### https://overreacted.io/zh-hant/a-complete-guide-to-useeffect/

這篇文章的目標是為了幫助你真的「深入理解」useEffect  
直到我停止透過 class 生命週期的濾鏡觀看 useEffect Hook，所有東西才在我眼中匯聚在一起。  

## 摘要
### 我要怎麼用 useEffect 複製 componentDidMount 的行為？

可以使用
```js
useEffect(fn, [])
```

它並**不是完全相等**  

與 `componentDidMount` 不同
- 他會捕捉 `props` 和 `state`

所以即使在 callbacks 裡面
- 你將會看到**初始**的 `props` 和 `state`

如果
- 想要看到最新的的東西
- 可以把它寫到一個 `ref`
- 但通常有更簡單的方法來架構你的程式碼，所以你並不需要這麼做

記住
- effects 的心理模型跟 componentDidMount 和其他的生命週期是不同的
- 嘗試想要找出它們相等的地方並不會幫助到你，反而只會讓你更困惑
- 為了能夠更有效率，你必須要「想著 effects」
  - 他們的心理模型跟回應生命週期的事件比較起來更接近實作同步化。

### 我該怎麼正確的在 useEffect 裡拿到資料？[] 是什麼？

這篇文章是一個關於用 useEffect 獲取資料的不錯的入門文章。確定你把它完全讀完
- https://www.robinwieruch.de/react-hooks-fetch-data/

`[]` 表示
- `effect` 沒有用任何參與 React 資料流的值
- 並且因此而安全的只使用一次

當那個值真的被用到的時候
- 它也是常見的錯誤來源
- 你將會需要學習幾個策略來為了依屬 (dependencies) 移除這個必要，而不是錯誤的忽略它。
  - 主要是 useReducer 和 useCallback

### 我應該要把用到的函示指定成 effect 的依屬 (dependencies) 嗎？

建議的做法是
- 把不需要 `props` 或 `state` 的函式提升到元件外面
- 並且把「**只被某個 effect 用到的函式**」放到 effect 裡面

如果在那之後你的 effect 仍需要在渲染的範圍（包含了 props 傳進來的函式）
- 使用函式，在定義它們的地方把它們包進 useCallback，並重複這個過程

為什麼這個重要？
- 函式可以從 props 和 state「看見」值 — 所以它們會參與資料流

### 為什麼我有時候會進入重複拿資料的無窮迴圈？

這可能發生在
- 當你沒有第二個依屬參數「卻」想要在 effect 裡獲取資料的時候

沒有它
- effects 會在每次渲染的時候發生
- 並且設定 state 這件事會再度觸發 effects

一個無窮迴圈
- 也可能會在你想要在依屬的陣列裡指定一個永遠都會變化的值

可以藉由
- 一個一個移除來發現到底是哪個值
- 然而，移除一個依屬（或盲目地使用 []）通常是錯誤的修正方式

相反的，你應該要修正這個問題的根源。例如
- 函式可能造成這個問題，將它們放到 effects 裡
- 抽出他們到上層
- 或是將他們包在 useCallback 裡可能有幫助
- 誤了避免重複產生新的物件，useMemo 可以達到相同的目的

### 為什麼我有時候會在 effect 裡拿到舊的 state 或 prop 的值？

Effects
- 永遠都會在它們被定義的渲染的時候「看見」 props 跟 state
  - 這樣能夠幫助避免錯誤
    - https://overreacted.io/how-are-function-components-different-from-classes/
  
但某些情況下它很惱人。在那些情況下
- 可以在一個 mutable 的 ref 特別維護某些值
- 試著用 lint rule 來訓練如何看它們
- 一些日子後，它會變得像是第二自然的事情

ref: https://reactjs.org/docs/hooks-faq.html#why-am-i-seeing-stale-props-or-state-inside-my-function


### 每次渲染都有自己的 Props 和 State
討論 effects 前，需要聊聊渲染。

```js
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p> // <---------- 仔細看看
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

count 有「觀察」著我們的 state 的變化然後自動更新嗎？  

在這個例子裡，count 只是一個數
- 它不是神奇的「data binding」、「watcher」、「proxy」或其他東西

當
- 元件第一次渲染的時候，我們從 `useState()` 拿到的 count 變數是 0
- 當我們呼叫 `setCount(1)` 之後，**React 再次呼叫我們的元件**
  - 這次，count 將變成 1

```js
// 在第一次渲染時
function Counter() {
  const count = 0; // 被 useState() 回傳
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// 經過一次點擊，我們的函式再次被呼叫
function Counter() {
  const count = 1; // 被 useState() 回傳
  // ...
  <p>You clicked {count} times</p>
  // ...
}

// 經過另一次點擊，我們的函式再次被呼叫
function Counter() {
  const count = 2; // 被 useState() 回傳
  // ...
  <p>You clicked {count} times</p>
  // ...
}
```


每當我們更新 state
- React 呼叫我們的元件
- 每次渲染的結果會「看見」他自己的 counter state 的值
- 而在我們的函式裡它是個常數。
- 它只會將一個數字的值放進我們渲染的輸出結果

所以這行不會做任何特別的 data binding
```js
<p>You clicked {count} times</p>
```


那個數字是由 React 所提供的
- 當執行 `setCount` 時，React **帶著不同的 count 值再次呼叫我們的元件**
  - 然後 React 更新 DOM 來對應我們最新的渲染結果。

關鍵要點是
- 在任何渲染裡面的 count 常數**不會經由時間而改變**
- 是我們的元件再次被呼叫，然後**每次的渲染「看見」它自己的 count 值**
  - 這個**值是孤立於每次的渲染**的

### 每次渲染都有它自己的 Event Handlers
那 `event handler` 呢？

看看以下的例子，它在三秒之後呈現了一個 count 的警告（alert）：

```js
function Counter() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```

假設做了以下幾個步驟：

1. 增加 計數器到 3
2. 按下 「Show alert」
3. 增加 計數器到 5 （在 timeout 發生以前）

但它是怎麼運作的？

討論了 count 值對我們函式的每個特定的呼叫是常數
- 這個是值得強調的
- 我們的函式被呼叫了很多次（每次渲染一次）
  - 但每次裡面的 count 值都是常數，並且被設定到某個特定的值（渲染的 state）

這並不是針對 React，正常的函式也有類似的運作方式：

```js
function sayHi(person) {
  const name = person.name;
  setTimeout(() => {
    alert('Hello, ' + name);
  }, 3000);
}

let someone = {name: 'Dan'};
sayHi(someone);

someone = {name: 'Yuzhi'};
sayHi(someone);

someone = {name: 'Dominic'};
sayHi(someone);
```

範例裡面
- 外面的 someone 變數被多次重新賦值
  - 就如同 React 的某些地方，現在的 state 可以改變
- 然而，在 sayHi 裡面，有個在特定呼叫裡跟本地的 name 常數關聯的 person
  - 這個常數是本地的，所以**它在每次的呼叫之間都是孤立的**
- 因此，每當 timeout 觸發的時候，每個警告會「記得」它自己的 name。

這個解釋了我們的 `event handler` **捕捉了點選時的 count**  
- 應用相同的代換原則，每次的選染會「看到」它自己的 count  

事實上
- 每次渲染**會回傳它自己「版本」的 handleAlertClick**
- 每個版本**會「記得」它自己的 count**
