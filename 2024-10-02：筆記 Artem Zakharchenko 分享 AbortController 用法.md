# 2024-10-02：筆記 Artem Zakharchenko 分享 AbortController 用法.md

## Artem Zakharchenko 18 SEPTEMBER 2024,

### ref: https://kettanaito.com/blog/dont-sleep-on-abort-controller

---

我的經驗，只有很少數地方會用到 `AbortController`。看到這篇，好奇看看還有哪些使用機會跟用法

---

### AbortController 是什麼？

`AbortController` 是 JavaScript 的一個 global class，可以用它來中止任何事情

```js
const controller = new AbortController();

controller.signal;
controller.abort();
```

一旦建立 controller instance，就會得到兩個東西:

1. `signal` 屬性，它是 `AbortSignal` 的一個 instance
   - 這是一個 pluggable 的部分，你可以提供給任何 API 來反應 abort
   - 例如，向 `fetch()` request 提供它就會中止該 request
2. `.abort()`方法，當被 call 時，會觸發 `signal` 上的 `abort` event
   - 它也會更新 `signal`，將其標示為 `aborted`

但實際的 abort 邏輯在哪裡呢？

- 它是由 consumer 定義的
- abort handling 可歸結為 listening `abort` event 聆聽中止事件，並以適合相關邏輯的方式實作中止

```js
controller.signal.addEventListener("abort", () => {
  // Implement the abort logic.
});
```

---

## 用法、使用時機: Event listeners

可以在新增 event listener 時提供 `abort` signal

- 讓它在 `abort` 發生時自動移除

```js
const controller = new AbortController();

window.addEventListener("resize", listener, { signal: controller.signal });

controller.abort();
```

執行 `controller.abort()` 會移除 `window` 的 `resize` listener

- 這是種非常優雅的 event listeners 處理方式，因為不再需要抽象 handler function (不需要拉出來定義在外面)

```js
// const listener = () => {}
// window.addEventListener('resize', listener)
// window.removeEventListener('resize', listener)

const controller = new AbortController();
window.addEventListener("resize", () => {}, { signal: controller.signal });
controller.abort();
```

如果 application 的不同部分負責移除 listener，`AbortController` instance 也更容易傳遞

- 當發現可以使用單一的 `signal` 移除多個 event listener，時，是一個偉大的 aha 時刻！

```js
useEffect(() => {
  const controller = new AbortController();

  window.addEventListener("resize", handleResize, {
    signal: controller.signal,
  });
  window.addEventListener("hashchange", handleHashChange, {
    signal: controller.signal,
  });
  window.addEventListener("storage", handleStorageChange, {
    signal: controller.signal,
  });

  return () => {
    // Calling `.abort()` removes ALL event listeners
    // associated with `controller.signal`. Gone!
    controller.abort();
  };
}, []);
```

只要呼叫一次 `controller.abort()` 就可以移除所有的 event listeners

### 用法、使用時機: Fetch requests

`fetch()` 也支援 `AbortSignal`

- 一旦 `abort` event 發出，從 `fetch()` 回傳的 request promise 將會 `reject`，代表 `aborted` 的 request

```js
function uploadFile(file: File) {
  const controller = new AbortController();

  // Provide the abort signal to this fetch request
  // so it can be aborted anytime be calling `controller.abort()`.
  const response = fetch(
    "/upload",
    { method: "POST", body: file },
    { signal: controller.signal }
  );

  return { response, controller };
}
```

`uploadFile()` function 會啟動一個 `POST /upload` request

- 回傳相關的 response promise，但也回傳一個 controller reference，以便在任何時候中止該 request
- 如果需要取消未完成的上傳，如，當 User 按下 `Cancel` 按鈕時，這會很方便

Node.js 中的 `http` module 的 `Requests` 也支援 `signal` 屬性

---

### `AbortSignal` class 也附帶了一些 static method 來簡化 JavaScript 中的 request 處理

#### `AbortSignal.timeout`

可以用 `AbortSignal.timeout()`，來建立 `signal`

- 在特定的逾時長度過後 dispatch `abort` event

如果你只想在超過逾時時間後取消 request，就不需要建立 `AbortController`

```js
fetch(url, {
  // Abort this request automatically if it takes
  // more than 3000ms to complete.
  signal: AbortSignal.timeout(3000),
});
```

#### `AbortSignal.any`

可以用 `AbortSignal.any()` 將多個 abort signals 組合為一個

- 類似於 `Promise.race()` 先到先得的方式來處理多個 promise

```js
const publicController = new AbortController();
const internalController = new AbortController();

channel.addEventListener("message", handleMessage, {
  signal: Abort.any([publicController.signal, internalController.signal]),
});
```

上面的範例中

- 引入了兩個 abort controllers
- 公開的 controller 給使用者，允許他們觸發 abort，觸發 event listener 被移除
- 而內部 controller 則允許我在不干擾公開 abort controller 的情況下移除 listener

如果提供給 `AbortSignal.any()` 的任何 `abort` event，那個 parent signal 也會 dispatch `abort` event

- 超過這一時間點的任何其他 `abort` event 都會被忽略

---

### 用法、使用時機: Streams

也可以使用 `AbortController` 和 `AbortSignal` 來取消 streams

```js
const stream = new WritableStream({
  write(chunk, controller) {
    controller.signal.addEventListener("abort", () => {
      // Handle stream abort here.
    });
  },
});

const writer = stream.getWriter();
await writer.abort();
```

`WritableStream` 也有 expose `signal` 屬性

- 這樣就可以呼叫 `writer.abort()`，這會 bubble 到 stream 中的 `write()` 方法中的 `controller.signal` 上的 abort event

---

## 製作任何 abortable 的東西

關於 `AbortController` API 是它非常多樣化。以至於可以做到任何邏輯成為可中止的！

- 可以強化你使用原生不支援中止/取消的第三方 library 上

舉例將 `AbortController` 加入 `drizzle-orm` 的 `transactions` 中

- 這樣就可以一次取消多個 transactions

```js
import { TransactionRollbackError } from "drizzle-orm";

function makeCancelableTransaction(db) {
  return (callback, options = {}) => {
    return db.transaction((tx) => {
      return new Promise((resolve, reject) => {
        // Rollback this transaction if the abort event is dispatched.
        options.signal?.addEventListener("abort", async () => {
          reject(new TransactionRollbackError());
        });

        return Promise.resolve(callback.call(this, tx)).then(resolve, reject);
      });
    });
  };
}
```

`makeCancelableTransaction()` function

- 會接受 DB instance，並傳回一個 higher-order transaction function
  - 該 function 現在接受一個 `abort` signal 作為參數

為了知道 `abort` 發生的時間

- 我在 `signal` 上新增了 `abort` event listener
- 每當 `abort` event 發生時，即 `controller.abort()` 被呼叫時，該 handler 就會被呼叫
  - 因此，當發生這種情況時，我可以用一個 `TransactionRollbackError` error 來 reject transaction promise，以回溯整個 transaction
  - (這與呼叫 `tx.rollback()` 產生相同的錯誤是一樣意思的)

現在，在 Drizzle 中可以這樣用它:

```js
const db = drizzle(options);

const controller = new AbortController();
const transaction = makeCancelableTransaction(db);

await transaction(
  async (tx) => {
    await tx
      .update(accounts)
      .set({ balance: sql`${accounts.balance} - 100.00` })
      .where(eq(users.name, "Dan"));
    await tx
      .update(accounts)
      .set({ balance: sql`${accounts.balance} + 100.00` })
      .where(eq(users.name, "Andrew"));
  },
  { signal: controller.signal }
);
```

這樣就可以用自訂 `transaction` 來執行多個 database operations

- 但也可以為它提供一個 `abort` signal 來一次取消所有操作

---

## Abort error handling

每個 abort event 都附有中止的 `reason`。這樣更有 customizability

- 因為可以對不同的 `reason` 做出不同的對應
- `reason` 是 `controller.abort()` 方法的 optional argument
- 可以在任何 AbortSignal instance 的 `reason` 屬性中存取 abort reason

```js
const controller = new AbortController();

controller.signal.addEventListener("abort", () => {
  console.log(controller.signal.reason); // "user cancellation"
});

// Provide a custom reason to this abort.
controller.abort("user cancellation");
```

`reason` 可以是任何 JavaScript value

- 因此可以傳入 string, Error or Object

---

## 結論

如果你開發的 library 中需要有 aborting 或 cancelling 操作功能，強烈建議使用 `AbortController` API
