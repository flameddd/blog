# 2020 重讀 Redux Style Guide
### https://redux.js.org/style-guide/style-guide/

前陣子，redux 重新整理了一版 **Style Guide**，這次來重讀、思考思考。  

this list of recommendations to help you
- avoid errors
- bikeshedding
- anti-patterns

## Rule Categories
下面的主題，分三種建議等級
- **Priority A**: Essential
  - These rules help prevent errors
  - learn and abide by them at all costs
  - Exceptions may exist, but should be very rare and only be made by those with expert knowledge of both JavaScript and Redux.
- **Priority B**: Strongly Recommended
  - These rules to improve readability and/or developer experience in most projects.
  - code will still run if you violate them, but violations should be rare and well-justified.
  - Follow these rules whenever it is reasonably possible.
- **Priority C**: Recommended
  - In these rules, we describe each acceptable option and suggest a default choice.
  - That means you can feel free to make a different choice in your own codebase

# **Priority A** (絕對要 follow 的原則)
## Do Not Mutate State (永遠禁止 mutate state)
- state 被 mutate 是最常見造成 redux bug 的原因
- 用 `Immer` 來避免 mutate （找一個你熟悉的 immutable library 來用也行）

## Reducers Must Not Have Side Effects (reducers 絕對不能有 side effects)
- Reducer should **only** depend on their **state** and **action** arguments
  - only calculate and return a new state value based on those arguments.
- must not execute
  - any kind of asynchronous (AJAX calls, timeouts, promises)
  - generate random values (Date.now(), Math.random())
  - modify variables outside the reducer
  - run other code that affects things outside the scope of the reducer function

reducer 裡面去呼叫其他在外面定義的 function 是ＯＫ的！
- such as imports from libraries or utility functions
- 只要有符合上面的規定就好

## Do Not Put Non-Serializable Values in State or Actions
- 不允許存 such as `Promises`, `Symbols`, `Maps/Sets`, `functions`, or `class instances` 到 **store state** or **dispatched actions** 裡面
- 這能保證 capabilities, such as
  - debugging via the Redux DevTools will work as expected
  - UI will update as expected

### Serializable
- Serializable means that the **data can be converted to pure text without losing information**.
- in JS, usually mean that one can do the following  

```js
const data2 = JSON.parse(JSON.stringify(data));
```

放 non-serializable value 技術上可行，但
- doing so can break the ability to **persist** and **rehydrate** the contents of a store
- as well as interfere with time-travel debugging.

所以，如果
- 能接受類似 **persistence** and **time-travel debugging** 可能會掛掉的話，是可以存 non-serializable value 的

或者，這種情況
- 放 non-serializable values 到 action 中，然後中間的 middleware (`redux-thunk`, `redux-promise`, `redux-saga`) 攔截它，讓 non-serializable values 不會進到 reducers 裡面也行。

## Only One Redux Store Per App (只能有一個 redux store)
- 標準有 redux 的 application 只會有一個 redux store instance
  - (It should typically be defined in a separate file such as store.js)


# **Priority B** (強烈建議要 follow 的原則)

## 使用 `Redux Toolkit` 來寫 Redux 的 Logic
- Redux Toolkit 是官方推薦使用的 toolset
  - 內建的方法就是官方推薦的 best practices
  - 包含 setting up the store to catch mutations
  - enable the Redux DevTools Extension
  - simplifying immutable update logic with Immer 等等
- RTK 不是必要的，但官方認為使用它，可以 simplify 邏輯，確保 application is set up with good defaults
- https://redux.js.org/redux-toolkit/overview

## 用 `Immer` 來實現 Immutable Updates
- 自己實作 immutable 方式的 update logic 是困難、容易出錯的
- 靠 `Immer` 相對簡單很多。`Immer` 是 redux 官方推薦

## 檔案結構 (Structure Files) 採用 Feature Folders 或 Ducks pattern
- 集中存放特定的 feature 到同個地方，方便 maintain code
  - 所以推薦 feature folder (all files for a feature **in the same folder**)
  - 或者 "ducks" pattern (all Redux logic for a feature **in a single file**)

不推薦 by "type" of code
- 例如 集中一個 folder 叫做 reducers, actions 之類的

## 盡可能把 Logic 放在 Reducers 中
- 「計算 new state 的邏輯」都盡量放到適當的 reducer 中
  - 而不是「prepares 的 codes」，也不是「dispatches the action」這邊 (like a click handler)
- 這樣能更方便的 test、time-travel debugging
- 能幫助避免常見的錯誤，導致 mutations and bugs

Reducers
- 總是比較容易測試，因為全部都是 pure functions
  - 只要 `const result = reducer(testState, action)`，然後 `assert().equal()`
  - 所以盡量放 reducer，你就更容易 test
- 大部分的 redux users 都知道，reducer state update 需要 follow "**the rules of immutable updates**"，但寫在 reducer 之外的，就沒有這種觀念。
  - 如果把 logic 寫在外面 reducer，就比較容易犯錯（違反這條 rule）
- `Immer` 就比較方便 update state，另外 `Immer` 會 freeze the state and catch accidental mutations.
- Time-travel debugging 是讓你執行一個 `undo` 的 "dispatched action"，然後(一樣是透過 redux debug) 執行其他的 action 或者 `redo` action
  - 當有問題時，就去 reducer debug，所以集中放在邊，方便我們集中地方 debug
- 放 logic 在 reducers 中，也就讓人知道，未來你要去哪找 update 的邏輯在哪邊。而不是隨便亂猜 or 從 UI component 從頭追蹤

## Reducers Should Own the State Shape
- 為了 maintainability，reducer 是個被用 key/value "slices" 拆開的
  - 每一個 "slice reducer" 負責提供
    - initial value 和
    - update 的 state 片段

此外，slice reducers 應該
- 嚴格控制「other value」
  - 這應可以避免其他人隨便的使用 `spreads/returns`
    - e.x. `({ ...state })`，所傳來不正確的 or 不相關的欄位
    - return `action.payload` or return `{...state, ...action.payload}`
- 這樣可以減少 bug

A "spread return" reducer may be a reasonable
- 當處理類似 **form** 的資料時，spread 可能是合理的
  - 要不然，如果這種時候還堅持**不用 spread** 的話，可能很花時間，也沒有太多好處

```js
const initialState = {
  firstName: null,
  lastName: null,
  age: null,
};

export default usersReducer = (state = initialState, action) {
  switch(action.type) {
    case "users/userLoggedIn": {
      return action.payload;  // 這邊的 return 就是完全信任 action 傳來的東西
    }
    default: return state;
  }
}

// 如果有人這樣亂傳，就 bug 了
// dispatch({  
//   type: 'users/userLoggedIn',
//   payload: {
//     id: 42,
//     text: 'Buy milk'
//   }
// })
```

## Name State Slices Based On the Stored Data
- 如同上面一條建議，標準的 splitting reducer logic 方法是用 "`slices`" of state
- `combineReducers` 就是標準的方法來合併這些 slice reducers
  - 傳進去的 key names 就定義了 key
  - key name 中避免使用 `"reducer"` 這單字

```js
// bad
{
  usersReducer: {},
  postsReducer: {}
}
```
```js
// good
{
  users: {},
  posts: {}
}
```

## 把 Reducers 當作是 State Machines
舉例
- `"request succeeded"` 的 action，只會在狀態已經是 `"loading"` 才會計算新的 value
- 只有在狀態是 `"being edited"` 時，才會發生 dispatch `"update this item"` 的 action

For example 
- 我們有一個 `fetchUserReducer`，與對應的 finite states （對應的情境、情況）可能會是這些
  - "idle" (fetching not started yet)
  - "loading" (currently fetching the user)
  - "success" (user fetched successfully)
  - "failure" (user failed to fetch)

我們可以這樣具體的指定 status
```js
const initialUserState = {
  status: 'idle', // explicit finite state
  user: null,
  error: null
}
```

寫 typescrpit 的話，可以用 `discriminated unions` 清楚定義不同的 state 時  
```ts
interface Square {
    kind: "square";
    size: number;
}

interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
type Shape = Square | Rectangle;
```

當 `state.status === 'success'` 時，我們會
- 期待 `state.user` 有東西，並且認為 `state.error` 不會是 `true`

```js
import {
  FETCH_USER,
  // ...
} from './actions'

const IDLE_STATUS = 'idle';
const LOADING_STATUS = 'loading';
const SUCCESS_STATUS = 'success';
const FAILURE_STATUS = 'failure';

const fetchIdleUserReducer = (state, action) => {
  // state.status is "idle"
  switch (action.type) {
    case FETCH_USER:
      return {
        ...state,
        status: LOADING_STATUS
      }
    }
    default:
      return state;
  }
}

// ... other reducers

const fetchUserReducer = (state, action) => {
  switch (state.status) {
    case IDLE_STATUS:
      return fetchIdleUserReducer(state, action);
    case LOADING_STATUS:
      return fetchLoadingUserReducer(state, action);
    case SUCCESS_STATUS:
      return fetchSuccessUserReducer(state, action);
    case FAILURE_STATUS:
      return fetchFailureUserReducer(state, action);
    default:
      // this should never be reached
      return state;
  }
}
```

因為我們已經
- 為每一個 state 做出定義了
- 這樣就避免影響
  - 舉例 `FETCH_USER` 的 action，就不回影響到 `status === LOADING_STATUS` 的情況

## Normalize Complex Nested/Relational State
- Normalize nested data
  - 方便 lookup, update data
- https://redux.js.org/recipes/structuring-reducers/normalizing-state-shape

這點有做真的有差，或者應該說，nested data 要 update 本來就非常麻煩

## 把 Model Actions 當作 Events，而不是 Setters
Redux 程式上，是不管 action.type 的內容是什麼，不管怎麼寫都合法
- 現在式 `"users/update"`
- 過去式 `"users/updated"`
- 描述成 event `"upload/progress"`
- 當作 setter `"users/setUserName"`

這些都合法，但，官方推薦還是盡量
- 採用「描述 event 發生」的方式，這種 event 的形式能夠
  - 會有更有意義的 action names 和減少 dispatch actions 的次數
    - (如果是 setter 的話，一個事情的處理，我可能要 setter 好幾個不同的 state，那就有可能要 dispatch 多個 actions。視為 event 的話，思維上，就是 emit 一個 event，然後所有的 handlers 一起一次處理)
- setters 的寫法，常會導致發展出很多個獨立的 action type
  - 太多 dispatch、也會有太多 action log，這樣就比較沒意義

舉例
- 餐廳的 App、有人點 pizza 跟飲料

這時候可以
```js
{
  type: "food/orderAdded",
  payload: { pizza: 1, coke: 1 }
}

// setter 的話，可能會是這樣
{
  type: "orders/setPizzasOrdered",
  payload: {
    amount: getState().orders.pizza + 1,
  }
}

{
  type: "orders/setCokesOrdered",
  payload: {
    amount: getState().orders.coke + 1,
  }
}
```

第一個例子
- 是一個 event
  - 某一個人點了餐點

第二個例子
- setter
  - 我知道 pizza 要多一個、coka 多一杯
  - 你去把這些數字算出來
  
第一個例子，你就只要 dispatch 一次就好

## 命名有意義的 Action 名稱
`action.type` 欄位有兩個目的
  - Reducer 用來判斷是否需要處理這個 action
  - 顯示在 Redux DevTools history log 上，讓 dev 讀這些 action (就像 events 一樣) 

理想狀況
- 你應該要能讀一串 dispatch 的 action type，然後就能帶大概理解這 action 是做什麼的
- 避免通俗的動作名稱，如 `SET_DATA`, `UPDATE_STORE`，這種其實看不出來有什麼意義


## 允許很多個 Reducers 來回應同一個 Action
- Redux 的 reducer logic 是拆分成很多小的 reducers
  - 每一個 reducer 獨立 update 自己部分的 state tree，最後合併起來
  - 當一個 action 被 dispatch 時，可能被所有 reducer 處理，也可能一個 reducer 都沒有

關於這部分
- 官方鼓勵，如果可以的話，那就朝多個分別 reducer functions 全都 handle 同一個 actions
- 很多時候，一個 actions 通常只會被一個 reducer function 來 handle

但
- redux 是 modeling actions as "events"
- 允許多個 reducers 處理這些 action 對 codebase 的 scale 比較好
  - 而且能 minimize dispatch action 的次數，避免 performance 議題

## 避免**同時** Dispatching 多個 Actions
- 這樣做是ＯＫ的，但常常會導致多次相關的 UI expensive updates
- 有些 intermediate state 可能會違反某些 application 邏輯
  - (應該是指多次 actions 中改變的 state，但還沒到最終結果的 state)
- 最好是 dispatching 單個 "event"-type 的 action，一次把所有 state update 到位
  - 不然考慮使用 action batching addons 來 dispatch multiple actions


關於 react render 機制
- 從 React event handlers 排隊的 UI 更新，通常會被批次到一個 React render pass 中
- 但在這些 event handlers 之外（不是 React event handlers 的那些）排隊的更新則不會
  - 包括 dispatches from most async functions, timeout callbacks, non-React code
  - 這些情況，每次 dispatch 都會導致一次「complete synchronous React render pass」
  - 這就會影響 performance
  來自大多數異步函數的調度，超時回調和非反應代碼。 在這種情況下，每次調度都會在調度完成之前導致完整的同步React渲染過程，這會降低性能。

另外
- multiple dispatches 就像是一種大的 "transaction"-style 來 update sequence
  - 這會導致 intermediate states，因而可能會出錯
- 舉例，連續發出這三個 actions "UPDATE_A", "UPDATE_B", and "UPDATE_C"
  - 某些 code 期待的是 a, b, c 三個 state 一起被 update
  - 但上面這種情況，可能會是 a, b update 了，但 c 還沒完成 update
  - 這樣可能就會有問題
- 如果真的需要 multiple dispatches
  - 考慮 batching
  - (possibly using `batch()` from `React-Redux`)
  - 文件與一些討論 https://redux.js.org/faq/performance#how-can-i-reduce-the-number-of-store-update-events

## 評估每一個 state 該不該存在
- 官方文件有這樣一句話 "the state of your whole application is stored in a single tree"
  - 這句話已經被過度解讀了
  - 絕對不是指把整個 application 的 state 都存進去
- 反而要好好思考，哪些 state 才應該在 global state
  - 如果 Values 是 local 的，那應該離 UI component 越近越好

判斷條件參考
- https://redux.js.org/faq/organizing-state#do-i-have-to-put-all-my-state-into-redux-should-i-ever-use-reacts-setstate


## Connect 多一點 Components 來從 store 讀 data
- Prefer 多點 UI components subscribed redux store 來讀 data (小量的 data)
  - 一般來說，這樣 UI performance 會比較好
  - 當只有一點點 state update 時，就只有少許的 component 會 update

舉例
- 不單單是 connect `<UserList>` 元件來讀整個 Users 的 array
  - `<UserList>` 去取 all user IDs
- 然後 `<UserListItem userId={userId}>` 這邊有了 userId
  - 就用 `<UserListItem>` 去 connect 取 user 的資料！
- 這舉例，用 `connect()` or `useSelector()` 來做都可以

## connect 的 mapDispatch  建議用 Object 格式當參數
- `mapDispatch` 可以宣告成 function or object
- 強烈推薦使用 object
  - https://react-redux.js.org/using-react-redux/connect-mapdispatch#defining-mapdispatchtoprops-as-an-object
- 比要簡潔
  - 除非有特別原因，不然就直接 object 吧
    - （另外，這樣 component 裡面會沒有 dispatch prop)

## Function Components 中，呼叫 `useSelector` 多次
- 當使用 `useSelector` 時，鼓勵呼叫 `useSelector` 多次、取小筆 data
  - 而不是「call 一次，取大筆 data」

因為
- 跟 `mapState` 不同，`useSelector` 不需要 return **object**
- read smaller values 代表，比較不會因為某個特定的 state change 造成這 component render

但
- 假如有一個 component 需要所有散落在 state 的 fields 時，就寫一個 `useSelector` 來取就好
  - 不要寫多個 `useSelector` 來各自取


## Use Static Typing
- 用 `TypeScript` or `Flow`
  - 幫助抓出更多 common mistakes
  - improve the documentation of your code
  - better long-term maintainability
- redux 官網有提供好幾個 TypeScript 參考資源
- 可以參考 Redux Toolkit 怎麼寫 TypeScript 版本

## 用 Redux DevTools Extension 來 debug
- Configure your Redux store to enable debugging with the Redux DevTools - - 

這點不用太說明了，沒有它，根本沒法 debug  

## State 用 JavaScript 的 Objects 就好
- 偏好使用 JS 的 objects 和 arrays 來存 state
  - 而不是 specialized libraries (例如 `Immutable.js`)
- 官方推薦 `Immer`


`Immutable.js`
- 有段時間，算是廣為推薦來搭配 redux 的，大家說的優點有
  - Performance improvements from cheap reference comparisons
  - Performance improvements from making updates thanks to specialized data structures
  - Prevention of accidental mutations
  - Easier nested updates via APIs like setIn()

但這些優點，其實沒有這麼有幫助，甚至是 negatives 的
- Cheap reference comparisons，不只 Immutable.js 能做到
- `Immer` 也能 accidental mutations can be prevented
- `Immer` 的 update logic 更簡單，根本不需要 setIn() (這是個學習成本)
- Immutable.js 很肥、API 相對複雜
- Immutable.js 要轉回 JS 原生 objects 是 cost expensive 的
  - deep object clone
- Immutable.js 不太維護了
- Immutable.js 現在最好的優點只剩，update 非常大 object 時，很快
  - (tens of thousands of keys)

所以
- Immer is a much better option.

# **Priority C** (推薦 follow 的原則)
## Action 的 Types 建議寫成 domain/eventName 形式
- 舊版的文件是這樣建議 "SCREAMING_SNAKE_CASE"，如
  - "ADD_TODO"
  - "INCREMENT"
  - 像很多程式語言宣告 constant value 一樣
  - 缺點 -> 全部大寫很難讀

其他的 communities 用更好的方法
- 類似 the "feature" or "domain" the action is related to
- NgRx community 就使用 "[`Domain`] Action Type" 的模式
  - e.x. "[Login Page] Login"
- 另一種是 "domain:action" 也經常被採用

Redux Toolkit 的 `createSlice` function 目前產生的模式為 "domain/action"
- e.x. "todos/addTodo"

總之，這種寫法 more readability

## 用 "Flux Standard Actions" 規則寫 Actions
- 最初的 "Flux Architecture" 文件，action 的寫法應該只有 **type** 一個欄位，沒有其他
- 後來為了 consistency，Andrew Clark 提出 "Flux Standard Actions"
  - https://github.com/redux-utilities/flux-standard-action

總歸來說，FSA convention 提出，action 應該
- 把 data 放到 `payload` 欄位
- 看情況，可能會有 `meta` 欄位
- 可能會有 `error` 欄位，來表示這個 action 是某種 failure
  - FSA spec 表示，"error" 的 actions，應該要把 error 設為 `true` and use the same action type as the "valid" form of the action
  - 實際上，很多人用分別的 action types 來表達 "success" and "error" 的情況
  - 上面兩種都行

```js
{
  type: 'ADD_TODO',
  payload: {
    text: 'Do something.'  
  }
}
{
  type: 'ADD_TODO',
  payload: new Error(),
  error: true
}
```

所以，推薦用這模式，能夠更有一致性。


## 使用 Action Creators
- Redux 中，action creators 不一定要有
  - Components 可以直接 call `dispatch({type: "some/action"})` (直接包一個 object 在這邊)
- 但是，action creators 提供 consistency
  - 尤其是需要 preparation or additional logic 時，需要先填入某些 data 在 action 中
  - (such as generating a unique ID)
- Redux Toolkit 有支持，建議使用，而不是自己寫

## Use Thunks for Async Logic
- 因為太多種 async middleware 造成混淆了
- 官方建議 `Redux Thunk`
  - 這已經能處理絕大多數的 use case 了
  - 另外，使用 `async/await` 方式，使 thunks 很好讀

如果複雜的 async workflows，如
- cancelation
- debouncing
- running logic after a given action was dispatched
- "background-thread"-type behavior

這時候再考慮更強大的 lib, e.x
- `Redux-Saga` or `Redux-Observable`

## Move Complex Logic Outside Components
- 習慣上，官方建議盡可能把 logic 抽出 component 之外
  - 部分原因是因為鼓勵 `"container/presentational"` pattern
  - 許多 components 單純從 props 接受 data，然後顯示 UI
- 官方還是建議把 synchronous or async 的 logic 移出去
  - 通常會搬到 thunks 中
  - 尤其是當 data 是從 store state 中讀取

但！
- 使用了 hooks 的話，有時會更容易直接管理這些 data fetching
  - （這樣就是會直接在 component 裡面了）
  - 這也會取代掉一些 thunks 的 use case

## Use Selector Functions to Read from Store State
- "Selector functions" 用來封裝從 store state 讀出來的 data，並且能導出進一步的 data
  - 其實就是，如果 UI 某些 data 可以從 store state 的 data 推導出來時，這樣的邏輯計算可以就在這邊寫好
    - e.x. 發票明細 data，可以在這邊推導出「總金額」，這樣不用把這邊邏輯放到 UI components 那邊，增加 component 複雜度
- `Reselect` 這類 lib，可以達成 memoized selector functions 來做某些優化
  - 強烈建議使用 memoized selector functions (推薦使用 `Reselect`)

但
- 不用覺得每個欄位都必要寫 selector functions
- 可以根據這個 fields 有沒有經常被 accessed and updated 來決定要不要寫

## 避免把 Form 的 state 放到 Redux
- 大部分的 form state 都不應該進 Redux
  - 這些 data 大部分都不是 global、不被 cached，也不需要同時被多個 components 使用
- 同時，form 存到 redux 通常變成，每一次 change/update 都會 dispatching actions
  - 影響 performance、沒有好處、大概也不需要 time-travel backwards debug
    - (time-travel backwards from name: "Mark" to name: "Mar")

即使，最後 data 是丟 redux，那也是建議這樣做
- 編輯的時候，state 是放在 local component state 編輯
- 只在完成 form 時，才 dispatching an action 把最後結果 update 到 redux store 中

在某些情況下
- 把 form data 存到 redux 確實有意義
  - e.x. 在 WYSIWYG (所見即所得) 的編輯中，有個 live previews 的功能在時
- 但大部分情況是不需要的
