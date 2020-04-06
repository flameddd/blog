# DavidKPiano：React tip custom hooks
## David K. Musical (@DavidKPiano), xState 的作者 Apr 3, 2020
### https://twitter.com/DavidKPiano/status/1246059611037278208


React tip: for many custom hooks, this return signature:
- [ received, send ] = useSomeHook(...)
  - is really versatile and flexible
  - as it allows two-way communication between the hook & component!

這邊的範例是，如果今天要做一個 custom hook，包含 call API 時，順便把操作一起 return 出來給外面的 component 使用。這樣外面的 component 可以操作要不要重新 fetch or cancel

## Example custom useAPI() hook with fetching & cancellation
- https://codesandbox.io/s/exciting-sky-52nto  

```js
import React from "react";
import { useReducer, useEffect, useMemo } from "react";
import "./styles.css";

const API_URL = "https://swapi.co/api";

function fetchReducer(state, event) {
  switch (state.status) {
    case "idle":
    case "resolved":
    case "rejected":
      if (event.type === "FETCH.INIT") {
        return {
          ...state,
          status: "pending"
        };
      }
      return state;
    case "pending":
      if (event.type === "FETCH.RESOLVE") {
        return {
          ...state,
          status: "resolved",
          data: event.data
        };
      }
      if (event.type === "FETCH.REJECT") {
        return {
          ...state,
          status: "rejected",
          error: event.error
        };
      }
      if (event.type === "FETCH.CANCEL") {
        return {
          ...state,
          status: "idle"
        };
      }
      return state;
    default:
      return state;
  }
}

const initialState = {
  status: "idle",
  options: undefined,
  data: undefined,
  error: undefined
};

function useAPI(resource) {
  const [state, dispatch] = useReducer(fetchReducer, initialState);

  const fetcher = useMemo(() => {
    const fetchResource = options => {
      dispatch({
        type: "FETCH.INIT",
        options
      });
    };

    fetchResource.cancel = () => {
      dispatch({ type: "FETCH.CANCEL" });
    };

    return fetchResource;
  }, []);

  useEffect(() => {
    if (state.status === "pending") {
      let canceled = false;
      fetch(`${API_URL}/${resource}`, state.options)
        .then(response => response.json())
        .then(data => {
          if (canceled) return;
          dispatch({
            type: "FETCH.RESOLVE",
            data
          });
        })
        .catch(error => {
          if (canceled) return;
          dispatch({
            type: "FETCH.REJECT",
            error
          });
        });

      return () => {
        canceled = true;
      };
    }
  }, [state, resource]);

  return [state, fetcher];
}

function Luke() {
  const [luke, getLuke] = useAPI("people/1");
  // getLuke 就 hooks 裡面的操作，外層有機會能使用（溝通）它，不會被 hooks 裡面的邏輯綁死

  console.log(luke);

  if (luke.status === "pending") {
    // show loading screen
    // 這邊能操作把 fetch 給 cancel 掉
    return (
      <div>
        <em>Loading Luke...</em>
        <button onClick={() => getLuke.cancel()}>Cancel</button>
      </div>
    );
  }

  if (luke.status === "resolved") {
    // luke.data is available
    // 這邊能 執行 fetch
    return (
      <div>
        <strong>{luke.data.name}</strong>
        <button onClick={() => getLuke()}>Fetch again</button>
        <pre>{JSON.stringify(luke.data, null, 2)}</pre>
      </div>
    );
  }

  return (
    <div>
      <button onClick={() => getLuke()}>Get Luke!</button>
    </div>
  );
}
```