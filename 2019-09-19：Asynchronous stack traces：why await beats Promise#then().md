## Asynchronous stack traces：why await beats Promise#then()
### Mathias 2nd October 2017
#### https://mathiasbynens.be/notes/async-stack-traces

作者 Mathias 是 google V8 team 的人

`async/await` 不只是
- more readable

另外它在 JavaScript engines 中有機會更好的 optimizations
- one such optimization involving stack traces for asynchronous code

### 根本上的差異 `vanilla promises` vs `await`

這樣執行時，**當下的 function 是會停住的（suspends）**
```js
await X()
```

這樣的話，**function 是繼續往下執行，最後是 callback 執行最後結果**
```js
promise.then(X)
```

這兩種模式，在 `stack traces` 的地方，差別就很大
`promise chain` 就可能在任何 point 的時候 throws unhandled exception  

### Vanilla promises
Imagine a scenario where a function c is called when a call to an asynchronous function b resolves:

```js
const a = () => {
  b().then(() => c());
};
```
`a` 執行時，會有下面的 `sync` 流程
1. `b` 被呼叫，然後回傳 promise
    - promise 在未來的某個時間點，會 `resolve`
2. `.then callback (也就是 c)` 會被加入到 callback chain (V8 lingo: [...])  

接著，`a` 執行完了，`a` 永遠不會 **suspended** 直到 async time 時 呼叫了 `b resolves`  
如果 `b` (or `c`) 某 async 時間丟出 exception，那 `stack trace` 應該要包含 `a`  
因為 `b` (or `c`) 是由 `a` 呼叫的  

上面的情境，JavaScript engine 就需要特別處理來捕捉到這情境  

```
In V8, the stack trace is attached to the promise that b returns. When the promise fulfills, the stack trace is passed on so that c can use it as needed.

Capturing the stack trace takes time (i.e. **degrades performance**); storing these stack traces **requires memory**.
```

### async/await
用 async 的話
```js
const a = async () => {
  await b();
  c();
};
```

即使沒有 collect the stack trace 也能在 `await call` 的時候 restore `call chain`  
就因為 `suspended, waiting for b to resolve` 才能做到  
`b` throws an exception 時，`stack trace` 就能即時 `reconstructed`  
`c` throws an exception 時，`stack trace` 就把它當做 sync function 來 constructed  


### Recommendations
async/await 不只是單純的語法糖而已  
讓 JavaScript engines 效能更好  

所以
-  Prefer async/await over desugared promises.
- Use `@babel/preset-env` to **avoid transpiling async/await unnecessarily**.

```
In general, don’t transpile code unless you absolutely need to!  
For example, all modern browsers that support service workers also support async/await.  
Consequently, there is no need to transpile your service worker–specific code down to vanilla promises. The same argument applies to browsers with JavaScript modules support.
```