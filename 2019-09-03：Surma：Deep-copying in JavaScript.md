## Surma：Deep-copying in JavaScript
### January 25, 2018
#### https://dassur.ma/things/deep-copy/

### Shallow copy

這樣可以 copy object
- 是種 shallow copy

```js
const obj = /* ... */;
const copy = Object.assign({}, obj);
```

`object spread` 的方式，也是一種 shallow copy  

### JSON.parse
- One of the oldest way

```js
const obj = /* ... */;
const copy = JSON.parse(JSON.stringify(obj));
```


缺點
- you create a temporary, potentially big string just to pipe it back into a parser.
- this approach cannot deal with cyclic objects.

像這種就會掛掉
```js
const x = {};
const y = {x};
x.y = y; // Cycle: x.y.x.y.x.y.x.y.x...
const copy = JSON.parse(JSON.stringify(x)); // throws!
```

而且，像是
- `Maps`
- `Sets`
- `RegExps`
- `Dates`
- `ArrayBuffers`

和其他內建的 types just get lost at serialization  
有些會失敗  


### Structured Clone

Structured cloning 是個已經存在的演算法
- https://html.spec.whatwg.org/multipage/structured-data.html#structuredserializeinternal
- 像是用在 `postMessage` 要傳東西到另一個 `window` or `WebWorker`
- 所以它支援 `cyclic objects` and supports a wide set of `built-in types`
  - https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm#Supported_types

但它沒有被直接 export 出來  

MessageChannel
- 就像上面提到的，`postMessage` 有用到這方法，所以...
- create a `MessageChannel` and send ourselves a message

```js
function structuralClone(obj) {
  return new Promise(resolve => {
    const {port1, port2} = new MessageChannel();
    port2.onmessage = ev => resolve(ev.data);
    port1.postMessage(obj);
  });
}

const obj = /* ... */;
const clone = await structuralClone(obj);
```

這方法個缺點是
- it is asynchronous

不是大問題，但有時候，我們還是需要 `synchronous way` 來 deep copy

### History API

`history.replaceState` 可以做到，而且是 sync 的  

```js
function structuralClone(obj) {
  const oldState = history.state;
  history.replaceState(obj, document.title);
  const copy = history.state;
  history.replaceState(oldState, document.title);
  return copy;
}

const obj = /* ... */;
const clone = structuralClone(obj);
```

但
- it feels a bit heavy-handed to tap into the browser’s engine just to copy an object
- Also, `Safari` limits **the amount of calls** to replaceState to 100 within a 30 second window.

### Notification API
`Notification` 也能做到 structural cloning

```js
function structuralClone(obj) {
  return new Notification('', {data: obj, silent: true}).data;
}

const obj = {};
const clone = structuralClone(obj);
```

這方式很簡端，但
- 這可能需要 `browser` 的 `permission`
- `Safari` 會有問題（always returns undefined for the data object）

## Conclusion

如果不需要處理
- cyclic objects
- built-in types

那就用 `JSON.parse(JSON.stringify())` 就好  

如果需要，那
- `MessageChannel`
- 是唯一可靠，而且 `cross-browser` 的選擇
