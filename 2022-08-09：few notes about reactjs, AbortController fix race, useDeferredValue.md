# 2022-08-09：few notes about reactjs, AbortController fix race, useDeferredValue

Ref:
- https://maxrozen.com/race-conditions-fetching-data-react-with-useeffect
- https://reactjs.org/docs/hooks-reference.html

筆記一下
1. 用 `AbortController` 來 cancel request
2. `useDeferredValue` and `useTransition`

---------------------------------------------

利用 `abortController.signal` 來 cancel request
- 這樣在 `id` update 時，前面的 request 就不會進到 state 中  
  - 之前沒什麼這問題，是因為都用了 `saga` 處理
- `setTimeout` 是為了 demo 而已，不是實作的一部分

```js
useEffect(() => {
  const abortController = new AbortController();

  const fetchData = async () => {
    setTimeout(async () => {
      try {
        const response = await fetch(`https://swapi.dev/api/people/${id}/`, {
          signal: abortController.signal,
        });
        const newData = await response.json();

        setFetchedId(id);
        setData(newData);
      } catch (error) {
        if (error.name === 'AbortError') {
          // Aborting a fetch throws an error
          // So we can't update state afterwards
        }
        // Handle other request errors here
      }
    }, Math.round(Math.random() * 12000));
  };

  fetchData();
  return () => {
    abortController.abort();
  };
}, [id]);
```


`useDeferredValue` 跟 `useTransition` 的目的非常相似  
- `useDeferredValue` 偏向用在你比較沒辦法控制如何取得 value 的時候，例如 call 第三方 lib
- 兩者都是用來處理有時候要用 `debounce` 來避免 UI 太快 or 過度更新的狀況
  - 例如，輸入文字來 filter 很大的 list，這時候可以 debounce filter feature 減少不必要的過程
- 這些情況通常 User 在乎的是最終結果，而不是過程
  - filter 輸入 `12345`，User 在意的是最後 `12345` 的結果，中間打到 `123` 的結果並不在乎

`useDeferredValue`
- 需要搭配 `useMemo` 才能做到讓 UI 只有在 `deferredQuery` update 時才更新

```js
function Typeahead() {
  const query = useSearchQuery('');
  const deferredQuery = useDeferredValue(query);

  // Memoizing tells React to only re-render when deferredQuery changes,
  // not when query changes.
  const suggestions = useMemo(() =>
    <SearchSuggestions query={deferredQuery} />,
    [deferredQuery]
  );

  return (
    <>
      <SearchInput query={query} />
      <Suspense fallback="Loading results...">
        {suggestions}
      </Suspense>
    </>
  );
}
```


`useTransition`

```js
function App() {
  const [isPending, startTransition] = useTransition();
  const [count, setCount] = useState(0);
  
  function handleClick() {
    startTransition(() => {
      setCount(c => c + 1);
    })
  }

  return (
    <div>
      {isPending && <Spinner />}
      <button onClick={handleClick}>{count}</button>
    </div>
  );
}
```