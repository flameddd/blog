# FrontEnd: Stop using isLoading booleans (use status field)
## Kent
### https://kentcdodds.com/blog/stop-using-isloading-booleans

寫 component state 時，很常見用 `isLoading` 來表示狀態

```js
function geoPositionReducer(state, action) {
  switch (action.type) {
    case 'error': {
      return {
        ...state,
        isLoading: false,
        error: action.error,
      }
    }
    case 'success': {
      return {
        ...state,
        isLoading: false,
        position: action.position,
      }
    }
    default: {
      throw new Error(`Unhandled action type: ${action.type}`)
    }
  }
}
function useGeoPosition() {
  const [state, dispatch] = React.useReducer(geoPositionReducer, {
    isLoading: true,
    position: null,
    error: null,
  })
  React.useEffect(() => {
    if (!navigator.geolocation) {
      dispatch({
        type: 'error',
        error: new Error('Geolocation is not supported'),
      })
      return
    }
    const geoWatch = navigator.geolocation.watchPosition(
      position => dispatch({type: 'success', position}),
      error => dispatch({type: 'error', error}),
    )
    return () => navigator.geolocation.clearWatch(geoWatch)
  }, [])
  return state
}
```

使用時  
1. 一開始 `isLoading` = true，然後顯示 loading，`useEffect` 取 data 後，set `isLoading` = false
2. `isLoading` = false 時，就有 `position` 拿 data

我習慣是 error 的判斷放比較前面，這樣讀 code 感覺比較順，下面的 case 討論也有涉及這方面  
- 如果今天出現 error 時，你要給 user 看到什麼？純 error 畫面？
- 這個 case 會是出現 `position data`，而看不到 `error`
- 又或者我們應該 show error，但還是給 user 看到最後一刻的 `position data`？
  - 想想 google map or apple map，假如今天 network 出問題，畫面有跳掉嗎？
  - 這要看 compnent 的 UI 邏輯設計，沒有絕對答案

```js
function YourPosition() {
  const {isLoading, position, error} = useGeoPosition()
  if (isLoading) {
    return <div>Loading your position...</div>
  }
  if (position) {
    return (
      <div>
        Lat: {position.coords.latitude}, Long: {position.coords.longitude}
      </div>
    )
  }
  if (error) {
    return (
      <div>
        <div>Oh no, there was a problem getting your position:</div>
        <pre>{error.message}</pre>
      </div>
    )
  }
}
```

假設取完 data 後，後續使用出現這類 error 時
- `GeolocationPositionError.POSITION_UNAVAILABLE`
- `GeolocationPositionError.TIMEOUT`
- `GeolocationPositionError.PERMISSION_DENIED`

上面的範例，結果會是 show last recorded position，並不會 show `error`
- (error 跟 position 順序對調，就是只 show error，不 show position)

改用 `status` ('idle', 'pending', 'resolved', 'rejected')

```js
function geoPositionReducer(state, action) {
  switch (action.type) {
    case 'error': {
      return {
        ...state,
        status: 'rejected',
        error: action.error,
      }
    }
    case 'success': {
      return {
        ...state,
        status: 'resolved',
        position: action.position,
      }
    }
    case 'started': {
      return {
        ...state,
        status: 'pending',
      }
    }
    default: {
      throw new Error(`Unhandled action type: ${action.type}`)
    }
  }
}
function useGeoPosition() {
  const [state, dispatch] = React.useReducer(geoPositionReducer, {
    status: 'idle',
    position: null,
    error: null,
  })
  React.useEffect(() => {
    if (!navigator.geolocation) {
      dispatch({
        type: 'error',
        error: new Error('Geolocation is not supported'),
      })
      return
    }
    dispatch({type: 'started'})
    const geoWatch = navigator.geolocation.watchPosition(
      position => dispatch({type: 'success', position}),
      error => dispatch({type: 'error', error}),
    )
    return () => navigator.geolocation.clearWatch(geoWatch)
  }, [])
  return state
}
```

使用時
```js
function YourPosition() {
  const {status, position, error} = useGeoPosition()
  if (status === 'idle' || status === 'pending') {
    return <div>Loading your position...</div>
  }
  if (status === 'resolved') {
    return (
      <div>
        Lat: {position.coords.latitude}, Long: {position.coords.longitude}
      </div>
    )
  }
  if (status === 'rejected') {
    return (
      <div>
        <div>Oh no, there was a problem getting your position:</div>
        <pre>{error.message}</pre>
      </div>
    )
  }
  // could also use a switch or nested ternary if that's your jam...
}
```

這種微小差別的好處是，讓 User(programmer) 比較好寫有彈性、調整不同狀態的顯示  

如果嫌 Container 這樣寫太麻煩，`hooks` 可以再擴充 status  
```js
function useGeoPosition() {
  // ... clipped out for brevity ...
  return {
    isLoading: status === 'idle' || status === 'pending',
    isIdle: status === 'idle',
    isPending: status === 'pending',
    isResolved: status === 'resolved',
    isRejected: status === 'rejected',
    ...state,
  }
}
```

這樣寫的時候，我們就能夠自己寫判斷
- 而不是一個 `isloading` or `position` or `error` 就只能讓我們寫死 render 的東西

```js
if (status === 'idle' || status === 'pending') {}
if (status === 'resolved') {}
if (status === 'rejected') {}
if (status === 'rejectedNotSupported') {}
```