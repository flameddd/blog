## Why Naked Promises are Not Safe For Work
### swyx August 14, 2019
#### https://www.swyx.io/writing/naked-promises

a deeper appreciation of 3 tricky cases to handle when crossing the synchronous to asynchronous boundary.  

常見的範例  
```js
function App() {
  const [msg, setMsg] = React.useState('click the button')
  const handler = () =>
    fetch('https://myapi.com/')
      .then((x) => x.json())
      .then(({ msg }) => setMsg(msg))

  return (
    <div className="App">
      <header className="App-header">
        <p>message: {msg}</p>
        <button onClick={handler}> click meeee</button>
      </header>
    </div>
  )
}
```

這邊的 Btn 的 handler function 所 reteurn 的我就稱為 **naked** Promise  
沒有被任何東西包住  

下面就是要分享一些 case  

### Promises Fail: The Error State
Promises fail 太常見了  
應該要用 catch 來處理  

ESlint 有條 rule 可以幫忙檢查，有沒有 `.catch`
- https://github.com/xjamundx/eslint-plugin-promise/blob/HEAD/docs/rules/catch-or-return.md

code 大概就變這樣  
```js
function App() {
  const [msg, setMsg] = React.useState('click the button')
  const [err, setErr] = React.useState(null)
  const handler = () => {
    setErr(null)  // <--------
    fetch('https://myapi.com/')
      .then((x) => x.json())
      .then(({ msg }) => setMsg(msg))
      .catch((err) => setErr(err))  // <--------
  }

  return (
    <div className="App">
      <header className="App-header">
        <p>message: {msg}</p>
        {err && <pre>{err}</pre>}
        <button onClick={handler}>click meeee</button>
      </header>
    </div>
  )
}
```

有兩個 `state` 來 hanel 每次的 async 操作

### Promises in Progress: The Loading State
async 總是要等待的，多加一個 `loading state` 來協同 UI 邏輯  

這樣就有三個狀態來處理每個 async 操作了
- `result`, `loading`, and `error` state

```js
function App() {
  const [msg, setMsg] = React.useState('click the button')
  const [loading, setLoading] = React.useState(false)
  const handler = () => {
    setLoading(true)  // <--------
    fetch('https://myapi.com/')
      .then((x) => x.json())
      .then(({ msg }) => setMsg(msg))
      .finally(() => setLoading(false))
  }

  return (
    <div className="App">
      <header className="App-header">
        <p>message: {msg}</p>
        {loading && <pre>loading...</pre>} 
        <button onClick={handler} disabled={loading}>
          click meeee
        </button>
      </header>
    </div>
  )
}
```

### Promises are dumb: The Component's State
promises 一旦開始了，就`沒辦法被 canceled`  
有些 workaround 的方法像是
- abortable fetch
  - https://developers.google.com/web/updates/2017/09/abortable-fetch

從來都不需要有 可以 `cancelable` 的 `promises`  
因為一旦開始了 promises，我們就不需要他了

有時候 react 會看到這樣的警告  
```
Warning: Can only update a mounted or mounting component. This usually means you called setState, replaceState, or forceUpdate on an unmounted component. This is a no-op.

# or

Warning: Can’t call setState (or forceUpdate) on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in the componentWillUnmount method.
```

如果需要需要避免 memory leak 的話，可以觀察 component 是否為 mount 的狀態  

```js
function App() {
  const [msg, setMsg] = React.useState('click the button')
  const isMounted = React.useRef(true)  // <--------
  const handler = () => {
    setLoading(true)
    fetch('https://myapi.com/')
      .then((x) => x.json())
      .then(({ msg }) => {
        if (isMounted.current) {  // <--------
          setMsg(msg)
        }
      })
  }
  React.useEffect(() => {
    return () => (isMounted.current = false)  // <--------
  })

  return (
    <div className="App">
      <header className="App-header">
        <p>message: {msg}</p>
        <button onClick={handler}>click meeee</button>
      </header>
    </div>
  )
}
```

利用 `Ref` 來像是判斷一個  instance 是否存在  
雖然 `isMounted` 是一種 `antipattern` (官方文件)  
- https://reactjs.org/blog/2015/12/16/ismounted-antipattern.html

但還是建議這樣做，取代 `cancellable promises` 的做法

從上面看下來，現在需要 `4 個 states` 來支持 `component 的 1 個 async 操作`


### Solution: Just Wrap It

在 production situation，就會一直需要這些
- error handling
- loading
- mounting tracker states

這先情況，有些 library 可以協助

`react-async` 的 `useAsync` hook 可以傳一個 promiseFn
還有對應的 cb

```js
import { useAsync } from 'react-async'

const loadCustomer = async ({ customerId }, { signal }) => {
  const res = await fetch(`/api/customers/${customerId}`, { signal })
  if (!res.ok) throw new Error(res)
  return res.json()
}

const MyComponent = () => {
  const { data, error, isLoading } = useAsync({
    promiseFn: loadCustomer,
    customerId: 1
  })
  if (isLoading) return 'Loading...'
  if (error) return `Something went wrong: ${error.message}`
  if (data)
    return (
      <div>
        <strong>Loaded some data:</strong>
        <pre>{JSON.stringify(data, null, 2)}</pre>
      </div>
    )
  return null
}
```

另外有 `useFetch` 可以用
```js
import { useFetch } from "react-async"

const MyComponent = () => {
  const headers = { Accept: "application/json" }
  const { data, error, isLoading, run } = useFetch(
    "/api/example",
    { headers },
    options
  )
  // This will setup a promiseFn with a fetch request and JSON deserialization.

  // you can later call `run` with an optional callback argument to
  // last-minute modify the `init` parameter that is passed to `fetch`
  function clickHandler() {
    run(init => ({
      ...init,
      headers: {
        ...init.headers,
        authentication: "...",
      },
    }))
  }

  // alternatively, you can also just use an object that will be spread over `init`.
  // please note that this is not deep-merged, so you might override properties present in the
  // original `init` parameter
  function clickHandler2() {
    run({ body: JSON.stringify(formValues) })
  }
}
```



`react-use` 也是類似的 library
```js
import { useAsync } from 'react-use'

const Demo = ({ url }) => {
  const state = useAsync(async () => {
    const response = await fetch(url)
    const result = await response.text()
    return result
  }, [url])

  return (
    <div>
      {state.loading ? (
        <div>Loading...</div>
      ) : state.error ? (
        <div>Error: {state.error.message}</div>
      ) : (
        <div>Value: {state.value}</div>
      )}
    </div>
  )
}
```

最後 `Daishi Kato` 的 `react-hooks-async`
- 有很棒的 abort controller for any promises

```js
import React from 'react'
import { useFetch } from 'react-hooks-async'

const UserInfo = ({ id }) => {
  const url = `https://reqres.in/api/users/${id}?delay=1`
  const { pending, error, result, abort } = useFetch(url)
  if (pending)
    return (
      <div>
        Loading...<button onClick={abort}>Abort</button>
      </div>
    )
  if (error)
    return (
      <div>
        Error: {error.name} {error.message}
      </div>
    )
  if (!result) return <div>No result</div>
  return <div>First Name: {result.data.first_name}</div>
}

const App = () => (
  <div>
    <UserInfo id={'1'} />
    <UserInfo id={'2'} />
  </div>
)
```
