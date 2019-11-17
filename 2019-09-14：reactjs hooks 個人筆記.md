## reactjs hooks 個人筆記

太少用了，做個筆記方便未來自己查  


### `componentDidMount` 實作
```js
function Example() {
  useEffect(
    () => console.log('mounted'),
    []
  );
  return null;
}
```

### functions 宣告，避免每次重複 create function 實作
變數不變的話， function 不會重新產生
```js
// Will not change unless `a` or `b` changes
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```


### `shouldComponentUpdate`、`PureComponent` 實作
- `React.memo`
- `useMemo`

```js
// 簡化版的 PureComponent，只會判斷 props，不會判斷 state
const Button = React.memo((props) => {
  // your component
});

// more example
function MyComponent(props) {
  /* render using props */
}
function areEqual(prevProps, nextProps) {
  /*
  return true if passing nextProps to render would return
  the same result as passing prevProps to render,
  otherwise return false
  */
}
export default React.memo(MyComponent, areEqual);
```

```js
// a or b 改變的話，會重新 create render
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);

// more example
function Parent({ a, b }) {
  // Only re-rendered if `a` changes:
  const child1 = useMemo(() => <Child1 a={a} />, [a]);
  // Only re-rendered if `b` changes:
  const child2 = useMemo(() => <Child2 b={b} />, [b]);
  return (
    <>
      {child1}
      {child2}
    </>
  )
}
```

### `useEffect` + `async/await

```js
export default function Example() { 
  const [data, dataSet] = useState(false)

  async function fetchMyAPI() {
    let response = await fetch('api/data')
    response = await res.json()
    console.log(response);
    dataSet(response)
  }

  useEffect(() => {
    fetchMyAPI();
  }, []);

  return <div>{data}</div>
}
```