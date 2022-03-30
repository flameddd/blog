# Blogged Answers: A (Mostly) Complete Guide to React Rendering Behavior
##  Mark Erikson May 17, 2020
### https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior

Mark Erikson
- redux 的主要 maintainer
- 非常認真幫忙改善 redux 的開發體驗、文件。Twitter 上都會主動幫忙解 redux 問題

這篇我估計 7 成內容我都已經很懂了，但我還是很想讀，看看 Mark 會怎麼寫，加上另外 3 成  

## What is "Rendering"?
就是 React 「問 UI Component（基於 props and state）要顯示成怎麼樣子」的過程

## Rendering Process Overview
在 rendering process 過程中，React 會
- 從 component tree 的 root 開始，向下循環找出所有被標記需要 update 的 Component
- 每一個被標記需要 update 的 Component，React 會呼叫
  - `classComponentInstance.render()` (for class components) 或者
  - `FunctionComponent()` (for function components)
- and save the render output

component's render output
- 通常是寫成 `JSX` 的語法
- 然後呼叫 `React.createElement()` 轉換成 object 的結構來描述 UI 的樣子

```js
// This JSX syntax:
return <SomeComponent a={42} b="testing">Text here</SomeComponent>

// is converted to this call:
return React.createElement(SomeComponent, {a: 42, b: "testing"}, "Text Here")

// and that becomes this element object:
{ type: SomeComponent, props: {a: 42, b: "testing"}, children: ["Text Here"] }
```

當整個 component tree 的 render output 收集完之後
- React 就會做比較 (通常是跟 vDOM 比較)
- 找出有哪些是真的需要更新到 real DOM 上面的
- 這段 diffing and calculation process 就是 [reconciliation](https://reactjs.org/docs/reconciliation.html)

## Render 和 Commit 階段

React team 把這些工作區分成兩個階段:
- "Render phase" rendering 過程，和計算改變工作
- "Commit phase" 是指把 DOM 套上 changes 的過程

commit phase 之後
- class 就會同時 run `componentDidMount` and `componentDidUpdate`
- function 就會去跑 `useLayoutEffect` hooks

接著 React 會 set 的一個短暫的 timeout
- timeout 過期後
- runs all the `useEffect` hooks
- 這步驟稱為 "`Passive Effects`" phase

這邊有圖可以看（但沒有 hooks 的
- https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/

未來的 `Concurrent Mode` 中
- 能在 `rendering phase` 暫停，允許 browser 處理 events
- React 之後會在適當的時間點，採取 resume, throw away, or recalculate 其中一種動作
- 一旦 render 完成，仍然會接著執行 `commit phase`

`rendering` 跟 `updating the DOM` 是不一樣的
- component rendered 的結果，可能會沒有任何 visible 改變

當 React renders 一個 component 時:
- component 可能會 return 跟上一次一樣的結果。所以沒有改變的需要
- `Concurrent Mode`中，React might end up rendering a component multiple times, but throw away the render output each time if other updates invalidate the current work being done


## React 怎麼 Handle Renders?
## Queuing Renders
initial render 結束後，有不同的方法告訴 React 安排一次 re-render

- Class components:
  - `this.setState()`
  - `this.forceUpdate()`
- Function components:
  - `useState` setters
  - `useReducer` dispatches
- Other:
  - Calling `ReactDOM.render(<App>)` again
  - (which is equivalent to calling `forceUpdate()` on the root component)

## Standard Render Behavior
React 的預設行為是當 parent component renders 時
- 會 recursively render 底下所有的 child components

舉例，有一個 component tree 是 `A > B > C > D`，已經顯示在畫面上了  
1. user 按下 B 上的 btn 增加 counter 的數字
2. 我們呼叫 B 裡面的 `setState()` 來安排 (queues) B **re-render**
3. React 從 tree 的最上面開始 render pass
4. React 看到 A 沒有 flag 說要 update，所以略過
5. React 看到 B 有 flag 要 update，所以 renders B。B return `<C />`
6. C 沒有 flag，但是他的 parent B rendered，所以 React 也去 renders C。C return `<D />`
7. D 因為 C 的關係，所以也會 renders

另一個重點是
- React 不會管是否有沒有 **"props changed"**。React 會無條件的 render child components，就因為 parent rendered

這意味著
- 在你的 root `<App />` component 呼叫 `setState()`
- 會導致底下每一個 component 都 re-render
  - 雖然大部分的 component 都會 return 跟上一次一樣的結果
  - 但 React 還是需要去問、去比較 diffing output，這些也時間與計算

(rendering 不是壞事，這是 React 的運作方法)


## Component Types and Reconciliation
React 的 rendering logic compares elements
1. 先比較 **type field** (using `===` reference comparisons)
2. 如果 type 不一樣 (例如 `<div>` 變 `<span>`，或 `<ComponentA>` 變 `<ComponentB>`)
3. React 會假設整個 tree 都改變了，這樣來加速比較
4. 所以，React 就會 destroy 整個 existing component tree section
    - 包含 all DOM nodes
    - 再來 recreate new component instances


所以，render 的時候，**最好最好不要 create new component type**  
- 建立新 type 時，React 就有可能重複的 destroy and recreate the child component tree


```js
// bad、換句話說，別這樣做
function ParentComponent() {
  // This creates a new `ChildComponent` reference every time!
  function ChildComponent() {}
  
  return <ChildComponent /> // 這樣每次都是新的 reference
}

// good、應該要這樣做，事先建立好
// This only creates one component type reference
function ChildComponent() {}
  
function ParentComponent() {
  return <ChildComponent />
}
```


## Render Batching and Timing
一般情況，呼叫 `setState()` 會使 React 開啟新的 render pass、同步(synchronously)的執行、然後 return  

但，React 也有自動做一些優化、一種 **render batching**
- ` Render batching` 是當多個 `setState()` 在「一個 render pass」裡面排隊執行

React 文件中提到，**"state updates may be asynchronous"**
- 這就是在指 batching
- **尤其在 React event handlers 的地方**，React 會自動做 batches state updates
  - 因為 React event handlers 佔很大部分

React 為 event handlers 實做 render batching 的方法是
- 包在 `unstable_batchedUpdates` 裡面
- unstable_batchedUpdates 就會把它們執行在「一個 render pass」中
- 這方法運做的很好，因為 React 已經早就知到哪些 handlers 針對特定的 event 一定需要被呼叫

概念上來說，你可以想像 React 內部像這 pseudocode 一樣運作

```js
// PSEUDOCODE Not real, but kinda-sorta gives the idea
function internalHandleEvent(e) {
  const userProvidedEventHandler = findEventHandler(e);
  
  let batchedUpdates = [];
  
  unstable_batchedUpdates( () => {
    // any updates queued inside of here will be pushed into batchedUpdates
    userProvidedEventHandler(e);
  });
  
  renderWithQueuedStateUpdates(batchedUpdates);
}
```

反過來說，任何 state updates 不在目前正要呼叫的 updates queue stack 裡面的話，一定不會包含在 batch 裡面  

看看這範例
```js
const [counter, setCounter] = useState(0);

const onClick = async () => {
  setCounter(0);
  setCounter(1);
  
  const data = await fetchSomeData();
  
  setCounter(2);
  setCounter(3);
}
```
上面個舉例總共會有 **3 次 render passes**
1. `setCounter(0)` 跟 `setCounter(1)`
    - 因為這兩次都發生在 original event handler call stack
    - 這兩個都會放到 `unstable_batchedUpdates` 執行
2. `setCounter(2)`
    - 因為是在 `await` 之後呼叫，這代表 original synchronous call stack 已經結束了
    - 下半部的程式已經完全跟 event loop call stack 分開了
    - 所以 React 會執行 render pass **synchronously**
    - call `setCounter(2)`, finish the pass, and return from `setCounter(2)`
3. `setCounter(3)` 也是一樣
    - 已經在 original event handler 之外了，所以也不在 batch 之內

在 commit-phase lifecycle 有些 edge cases
-  `componentDidMount`, `componentDidUpdate`, and `useLayoutEffect`
- 這些能讓我們在「一個 render 之後」，但在「browser 執行 paint 之前」有機會執行一些 logic

常見的 case 是
- 第一次 render 一個 component，但只有不完整的資料
- 在　commit-phase lifecycle，利用 ref 去拿到真正 DOM 在頁面上的 size
- 把 size 的資訊 set 到 state 中
- 馬上 re-render

DOM 被修改過後，Browsers 就會重新計算 DOM structure。但在 JS script 還在執行、blocking event loop 時，不會 paint 任何東西到 screen 上。  
- 就像 `div.innerHTML = "a"; div.innerHTML = b";` 這樣，`a` 永遠不會出現  
- 如果你執行某些 React update，像 "partial->final" 的過程，最後只會有 final 的 content 會顯示在 screen 上

`unstable_batchedUpdates`
- React team 有說，這是最 stable 的 "unstable" APIs，Facebook 的 code 有一半以上都靠它
- 不像其他 React API，unstable_batchedUpdates 是從 `react-dom` 和 `react-native` export 的
- `React-Redux v7` 內部也使用了 `unstable_batchedUpdates` 做某些優化

未來 `Concurrent Mode`，React 會全面都是 `batch updates`


## Render Behavior 的 Edge Cases
在 `<StrictMode>` (StrictMode 只會生效在 development 環境) 時，React 會 double-render components  

所以
- `rendering logic` 執行的次數，不會跟 `committed render passes` 相同
- 不能依賴 `console.log()` 來看 render 次數
  - 要用 `React DevTools Profiler` 來 trace and count 整體的 committed renders 
  - 或者在 `useEffect/componentDidMount/Update` 裡面 console.log (這邊的，只有在 React 真正完成 render pass and committed 時，才會 print log)


正常時候
- 絕對不要在執行 rendering logic 時去 update state

有一種例外
- Function components 可能在 rendering 時候去 update state (call `setSomeState()`)
- 只要這情況有被某些 conditionally 來判斷停止，不要讓它每次 render 都觸發執行
- 這樣的 function 就等同於 `getDerivedStateFromProps`

如果 function component render 時 state update
- 在繼續往下做之前，React 會立刻去 update state，並且 synchronously re-render 那個 component
- 如果 component 無限 loop 去 update state，促使 react re-render，在一定次數 (目前是 50 次) 的重複嘗試 and throw error 後，會結束這個 loop

上面這個方法，能夠立刻強制 update 一個 state 來根據 props 的 change，而不需要透過
-  re-render + a call to `setSomeState()` inside of a `useEffect` 這樣的組合


## 改善 Rendering Performance
雖然 render 是 React 工作的一部分，但也確實會有 wasted 的 render (render output 沒有改變、DOM 不需要被 update)  

React component render output 應該完整根據 **current props/state**  
因此，如果我們提早的知道 props/state 沒有變動，那我們就能安心的跳過　render 的工作

優化 React render 的主要方法，就是在適當的時機點跳過 render，減少工作量

## Component Render Optimization 技巧

3 個主要的 API 能 skip render

- `shouldComponentUpdate`: If it returns false, React will skip rendering the component. the most common approach is to check if the component's props and state have changed since last time, and return false if they're unchanged.
- `PureComponent`: since that comparison of props and state is the most common way to implement shouldComponentUpdate, the PureComponent base class implements that behavior by default
- `React.memo()`: The wrapper component's default behavior is to check to see if any of the props have changed, and if not, prevent a re-render. Both function components and class components can be wrapped using React.memo(). (A custom comparison callback may be passed in, but it really can only compare the old and new props anyway, so the main use case for a custom compare callback would be only comparing specific props fields instead of all of them.)

上面這些方法的比較都是用 `shallow equality`，像這樣

```js
obj1.a === obj2.a && obj1.b === obj2.b && .........
```

另招比較少人知道的技巧
- 如果 React component return 跟上一次完全一樣的 element reference，React 就會 skip 這特定的 child re-rendering

這些上面的技巧，skipping rendering a component 也意味著 skip 掉底下整個 subtree
- 適當的用這些技巧來阻止預設的行為 (render children recursively)


## 傳 Props 時，新的 References 如何影響 render 的 Optimizations (How New Props References Affect Render Optimizations)
- React 預設行為就是會去 re-render 所有底下的 children，即使 props 沒有改變

每次 `ParentComponent` render 時，都會 create 全新的 function, object (new ref) 往下傳。   
不管是不是 new ref，NormalChildComponent 都會 re-render

```js
function ParentComponent() {
  const onClick = () => {
    console.log("Button clicked")
  }

  const data = {a: 1, b: 2}
  
  return <NormalChildComponent onClick={onClick} data={data} />
}
```

如果 child 用 `React.memo()` 來優化，那 new ref 就會使 child re-render
- 如果是新的 data，那 re-render 當然是OK的

那如果只是傳 callback function 下去呢?

```js
const MemoizedChildComponent = React.memo(ChildComponent)

function ParentComponent() {
  const onClick = () => {
    console.log("Button clicked")
  }
  
  const data = {a: 1, b: 2}
  
  return <MemoizedChildComponent onClick={onClick} data={data} />
}
```

這個範例
- `MemoizedChildComponent` 每次都會 re-render
- 比較 old/new props 的行為，在這邊就都是浪費。


## 改善 props 的 ref (Optimizing Props References)
Class components 不太需要擔心 create new callback function ref  
(因為有 instance，所以一直是同樣的 ref)  

但有些時候需要產生單一的 callback function 給底下 child 的 list item 之類的  
這時候就是 new references
這種情況，React 本身沒有內建甚麼方法能幫忙優化
(類似情況可以靠 function/event delegation 的概念處理)  


function components 可以靠
- `useMemo` 處理 data new ref (or 減少複雜運算)
- `useCallback` 處理 callback function new ref

## 全部都 cache 起來? (Memoize Everything?)
那為什麼我們不把所有的 component 都用 `React.memo()` 包起來呢?
Dan Abramov 一在強調，memoization 來做 comparing 是有代價的  
如果 hit rate 極低，那胡亂使用 `React.memo()` 在經常會更新 props 的 component 上，那根本就是浪費  

## Immutability and Rerendering

React 中 State updates 應總是用 immutabe 的方式，因為
- 能確保 component 有正常 render update 的 state


`React.memo` / `PureComponent` / `shouldComponentUpdate` 這些都使用 **shallow equality** 來確認 props 的變化  

如果用 mutate 方式 upadte，這樣有些 component 就會假設這邊沒有 state update  

另外關於 `useState`, `useReducer` hooks，每次使用 `setCounter()` or `dispatch()` 時　　
React 將會 queue up 一次 re-render，但 Reach 要求每次 update state 時，都必須傳入 new ref  

React 會在 render phase 時，套用上所有的 update state。當 React 嘗試從 hooks 採用 state update 時，就會確認 new value 是不是 same reference。如果跟前一次是 same ref，那就不會 render 了  

```js
// 這個 update 不會成功 re-render.
const [todos, setTodos] = useState(someTodosArray);

const onClick = () => {
  todos[3].completed = true;
  setTodos(todos);
}

// 技術上，只有最外層需要是 immutably updated
const onClick = () => {
  const newTodos = todos.slice();
  newTodos[3].completed = true;
  setTodos(newTodos);
}
```

class component 的 `this.setState()` 跟　function component　的 `useState`, `useReducer` hooks 的差異  

`this.setState()` 不會管是不是 mutate，一律會完成一次 re-render:

```js
const {todos} = this.state;
todos[3].completed = true;
this.setState({todos});
```


但除了這邊的差異之外，整個 React 生態系、第三方都會假設你會使用 immutabe 的方式  
所以一律使用 immutabe 的方式能夠避免一些其他地方的判斷出問題


## 測量 React Component Rendering Performance
用 `React DevTools Profiler` 來看
- 每次 commit 有哪些 component 執行 render
  - 找出那些不應該 render 的 component 來研究 why
  - 如果這些 component 有 perf issue，看有沒有機會改善

另外是，**dev builds** 的 React 執行起來比較慢，真的要測試準確時間時，還是要用 **production builds**  


## Context and Rendering Behavior
React's Context API 是種機制讓你能提供 single user-provided value available 給 subtree 的 components
- 任何 `<MyContext.Provider>` 裡面的 component 能直接從 context 裡面讀 value，不需要透過傳 props

Context 不是 "state management"
- 你需要自己去管理傳進去 context 的 values


## Updating Context Values
React 會檢查 `context provider` 是否有被給新的 value，如如果 provider value 是 new ref，底下的 components consuming 就需要 update  

```js
function GrandchildComponent() {
  const value = useContext(MyContext);
  return <div>{value.a}</div>
}

function ChildComponent() {
  return <GrandchildComponent />
}

function ParentComponent() {
  const [a, setA] = useState(0);
  const [b, setB] = useState("text");

  const contextValue = {a, b};

  return (
    <MyContext.Provider value={contextValue}>
      <ChildComponent />
    </MyContext.Provider>
  )
}
```
上面的範例，每次 `ParentComponent` render，React 都會知道 `MyContext.Provider` 被設上 new value  
然後所有 nested component that consumes 都會強制 re-render

從 React 角度，provider 只有一個 single value，不管是 object, array, or a primitive  
- 單純就是一個 context value
- 現在為只，並沒有方法能讓 component that consumes a context 能夠 skip 掉因為 new context values 所觸發的 render，即使這個 component 只用到其中一部分的 data

## State Updates, Context, and Re-Renders

整合上面提到的
- 呼叫 `setState()`　會 queues component 做一次 render
- React 預設行為是 recursively renders nested components
- `Context providers` are given a value by the component that renders them
  - 這個 value 通常來至 parent component's state

這意味著，default 情況下，parent component 做任何 state update 到 context provider 時
- 會造常所有的 descendants 都會 re-render，無論它們有沒有用到 context 裡面的 value

再去看上面的例子 **Parent/Child/Grandchild**
- `GrandchildComponent` 會 re-render，但不是因為 context update
  - 是因為 `ChildComponent` rendered 了

這例子沒有任何優化
- default 情況下，只要 `ParentComponent` render，那 `ChildComponent` 跟 `GrandchildComponent` 都會 rerender

如果 parent 放一個 new context value 到 `MyContext.Provider`
- `GrandchildComponent` 在 render 時，就會確認 new value，並使用它
- 但 **context update** 不是觸使 `GrandchildComponent` 的原因

## Context Updates and Render Optimizations
看看另種情況

```js
function GreatGrandchildComponent() {
  return <div>Hi</div>
}

function GrandchildComponent() {
    const value = useContext(MyContext);
    return (
      <div>
        {value.a}
        <GreatGrandchildComponent />
      </div>
}

function ChildComponent() {
    return <GrandchildComponent />
}

const MemoizedChildComponent = React.memo(ChildComponent);

function ParentComponent() {
  const [a, setA] = useState(0);
  const [b, setB] = useState("text");

  const contextValue = {a, b};

  return (
    <MyContext.Provider value={contextValue}>
      <MemoizedChildComponent />
    </MyContext.Provider>
  )
}

```

接著我們 call `setA(42)`

1. `ParentComponent` will render
2. new contextValue reference 被 create
3. React 看到 `MyContext.Provider` 有new context value，因此，任何 **consumers of MyContext** 都需要被 update
4. React 會嘗試render `MemoizedChildComponent`，但因為有 `React.memo()`，這時候沒有任何 props 改變（連傳都沒傳），所以 React 會 skip 整個 `ChildComponent` render
5. 然而，因為 `MyContext.Provider` update，因此有其他的 component 需要知道這件事情
6. React 會持續往下看，找到 `GrandchildComponent`，看到有讀取 `MyContext`，而 context 有 new value，因此它**必須**要 re-render，React 就會針對 `GrandchildComponent` 做 re-render

 React goes ahead and re-renders `GrandchildComponent`, specifically because of the context change.
7. 因為 `GrandchildComponent` render，React 會繼續往下執行，所以也會　re-render `GreatGrandchildComponent`.

所以說！！
- 在 Context Provider 裡面的 **React Component** 可能需要用 `React.memo`
- 這樣 parent component update state 就不會 force 所有 component re-render
  - 只有用到 context 的地方會 re-render
- 或者也能用到 **"same element reference"** 的技巧來避免這情況

然而，當 `GrandchildComponent` 因為 next context value 而 render，那 React 就會回到 default behavior  
也就是 recursively re-rendering everything  

 

## React-Redux and Rendering Behavior
社群上非常看到這兩者的比較，但這其實是錯誤的二分法，這兩者式不同的工具、做不同的事情  

(作者這邊沒詳細說明，因為他另外寫了一篇專門討論這兩者的差異)


## React-Redux Subscriptions
一堆人不斷的說，React-Redux 裡面是使用 `context`
- technically true, but
- React-Redux 是用 context 來傳遞 **Redux store instance**，而不是 **current state value**
- 意思是，每次都是傳 **same context value** 到 `<ReactReduxContext.Provider />` 

一定要記住
- 每次有 action dispatched，Redux store 就會執行**所有的 subscriber** 的 notification callbacks
- UI layers 就從 subscriber callbacks 讀到最新的 state、然後 diff the values, and force a re-render  if the relevant data has changed. 
  - `subscription callback` 的過程是在 React 之外、跟 React 無關  

視情況而定，有少數的 component 可能會不斷的 rendering
- 但，當 store state updated，React-Redux 必須要執行 `mapState/useSelector functions`
- 絕大多情況，執行這些 `selectors` 的 cost 是比 React 執行另一次 **render pass** 還低的。

但還是要注意這些，來避免 perf issue
- selectors 有沒有執行 expensive task, or
- 總是 return new value, ref 回來


## connect 跟 useSelector 差異

`connect` 是 HOC
- 包住你的 Component，然後 subscribing to the store
- 把 `mapState` 和 `mapDispatch` 做為 props 傳進去 props

connect 行為幾乎跟 `PureComponent/React.memo()` 一樣，但還是有些差異
- connect 只會當 combined props 有變動時，才會 render component

final combined props 是這樣組合的
```js
{...ownProps, ...stateProps, ...dispatchProps}
```

所以
- parent 傳 new ref 時，就會讓 component re-render
- mapState 回傳了 new ref 時，也會



`useSelector`
- 是在 component 裡面被呼叫的
- 所以 useSelector 沒辦法阻止由 parent 觸發的 render component


從效能角度談 `connect` 跟 `useSelector`
- `connect`: 每個 connected component 就像 **PureComponent**
  - 所以它就像防火牆，能夠避免 React 的 default render behavior (cascading down the entire component tree)
  - 通常 React-Redux 的 App，都會有很多的 connected component ，所以能夠很大範圍阻止這種 render

如果，你專注只使用 `useSelector`
- 很大一部分的 component tree 會因為 store update 而 re-render (相比用 `connect`)
- 所以如果當下 performance 是需要關注的議題，使用 `React.memo()` 來避免因為 parent render child

## 總結

- React 預設總是遞迴的 render component. 當 parent render 時， children 也會 render
- Rendering 本身沒有任何問題，這是讓 React 知道 DOM 有沒有需要 update
- 但是，rendering 會花時間。 "wasted renders" 加起來，就浪費很多時間了
- 大多時候，props 傳 new references like callback functions 和 objects 都不會有問題
- 像 `React.memo()` 這類的 API，props 沒變時，能幫忙避免不需要的
  - 但如果你需要每次都傳 new ref 時，就沒辦法，這時候要 memoize values
- `Context` 讓底下任一層的 component 能讀取 value
- `Context providers` 會比較 value 的 reference 來得知是否有改變
- 新的 `context values` 會強迫所有 nested `consumers` to re-render
- But, many times the child would have re-rendered anyway due to the normal parent->child render cascade process
- So you probably want to wrap the child of a context provider in React.memo(), or use {props.children}, so that the whole tree doesn't render all the time when you update the context value
- When a child component is rendered based on a new context value, React keeps cascading renders down from there too
- React-Redux uses subscriptions to the Redux store to check for updates, instead of passing store state values by context
- Those subscriptions run on every Redux store update, so they need to be as fast as possible
- React-Redux does a lot of work to ensure that only components whose data changed are forced to re-render
- connect acts like React.memo(), so having lots of connected components can minimize the total number of components that render at a time
- `useSelector` 只是 hook，它不能阻止由 parnet component 發起的 render
  - 當整個 App 到處了 `useSelector` 時，可能就要用些 `React.memo()` 避免每次都聯動 render


## Final Thoughts
總結 `context` 跟 `redux` 建議使用時機

Use `context` if:
- 當你只需要傳些簡單的值，而且不需要經常 update 這些 value
- You have some state or functions that need to be accessed through part of the app, and you don't want to pass them as props all the way down
- 你想使用 react build in 功能就好，不想額外的 library
Use `(React-)Redux` if:
- 有很多的 state，而且用在 App 的很多地方
- state 經常 update
- state 的 update logic 可能複雜
- The app has a medium or large-sized codebase, and 可能會有多個人同時開發此 App

