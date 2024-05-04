# 2024-03-30：筆記 Joyee Cheung 談 V8 的 Memory leak regression testing.md

----------------

Joyee Cheung 是  Node.js TSC 和 V8 committer  

Ref:
- https://joyeecheung.github.io/blog/2024/03/17/memory-leak-testing-v8-node-js-1/
- https://joyeecheung.github.io/blog/2024/03/17/memory-leak-testing-v8-node-js-2/
- https://joyeecheung.github.io/blog/2024/03/17/memory-leak-testing-v8-node-js-3/

這幾篇 blog 是她
- Node.js 針對 memory leak regression 所使用的測試策略的筆記
- 對這些策略的觀察，以及為什麼我使用新的 V8 API 新增了新的測試策略
- 希望幫助讀者寫更少不可靠的 memory leak regression/memory leak reproductions 



----------------

## Measuring heap usage + `gc()`

首先看看 Node.js 用於測試 memory leaks 的最早策略之一
- `基於 memory usage 測量`
- 這可能是由於收到 user 的錯誤報告而導致的，user 透過 production 中的 memory usage 情況監控發現了 leak

`memory usage 測量`是根據這些假設
- `gc()` 應該能夠回收已經無法存取的 object 所使用的 memory
  - (V8 `--expose-gc` 能 global expose `gc` function)
- 如果操作有 leak，那操作之後 `gc()`，memory 應該不會下降，這樣應該就有 leak


這個 [test](https://github.com/nodejs/node/blob/6ea5f6efa5e7dee7b0771fddb204ed6c75f2c9a0/test/parallel/test-diagnostics-channel-memory-leak.js) 來舉例
- (這個 test 已經失效了，已經修改成另一種策略，後面會提)

```js
const { ok } = require('assert');
const { subscribe, unsubscribe } = require('diagnostics_channel');

function noop() {}

const heapUsedBefore = process.memoryUsage().heapUsed;

for (let i = 0; i < 1000; i++) {
  subscribe(String(i), noop);
  unsubscribe(String(i), noop);
}

global.gc();

const heapUsedAfter = process.memoryUsage().heapUsed;

ok(heapUsedBefore >= heapUsedAfter);
```

測試流程基本上為:
1. 在 allocation 開始之前測量 memory usage heap
    - 這個 case 就是從 `v8::HeapStatistics::used_heap_size()` 產出的 `heapUsed` 和 
    - `v8::Isolate::GetHeapStatistics()` 的統計資料
2. 執行可能 leak 的操作
    - (盡量 allocate 多次大量的記憶體)
3. 執行 `gc()`，然後再測量  memory usage heap 一次
4. 如果 memory usage 不下降，就是 leak，反之，沒有 leak


有幾個問題會使測試不可靠
- 其中之一是假設 `gc()` 後會**立即回收**足夠的無法存取的 memory
- 但實際上 `gc()` 不是這樣運作的
- 真正降低 memory usage 的 GC task 可能會延遲，直到 thread idle，也就是不執行 JavaScript 的時候
  - (或者，從概念上講，可以說它是 async)

----------------

## `gc()` multiple times asynchronously


為了應對 delay `gc()` 效應
- Node.js core’s test suite 有一個 [gcUntil](https://github.com/nodejs/node/blob/c975384264dc553de62398be814d0c66fc1fc1fb/test/common/index.js#L838-L860)，用 `setImmediate()` 執行 `gc()` 10 次，直到某個條件為 `true`
- 選用 `setImmediate()` 是因為 callback 將在 event loop 的下一次迭代中執行。到那時，thread 已經完成了 stack 上 JavaScript 的執行，並且可能已經處理了一些 GC task


```js
function gcUntil(name, condition) {
  return new Promise((resolve, reject) => {
    let count = 0;

    function gcAndCheck() {
      setImmediate(() => {
        count++;
        global.gc();
        if (condition()) {
          resolve();
        } else if (count < 10) {
          gcAndCheck();
        } else {
          reject(name);
        }
      });
    }

    gcAndCheck();
  });
}
```

所以在上面提到的第 3 步驟中，我們要改成這樣:
- 3. 執行 `gc()`，等 JS 執行完後，測量 memory usage 
- 4. 如果 memory usage 沒有下降，再執行 `gc()`，最多重複 10 次。如果 10  memory usage 都沒有下降足夠的量，那就是 leak，反之沒有

最前面的範例調整成這樣:
```js
const { subscribe, unsubscribe } = require('diagnostics_channel');

function noop() {}

async function main() {
  const heapUsedBefore = process.memoryUsage().heapUsed;

  for (let i = 0; i < 1000; i++) {
    subscribe(String(i), noop);
    unsubscribe(String(i), noop);
  }

  await gcUntil('heap usage should go down', () => {
    const heapUsedAfter = process.memoryUsage().heapUsed;
    return heapUsedBefore >= heapUsedAfter;
  });
}

main();
```


但是，上面的 test 的結果還是不可靠，還有特殊的情況
- 這裡有個 [Node.js core’s test suite 例子](https://github.com/nodejs/node/blob/1f2ad0560063f2870d18cf649b285112de84eb64/test/parallel/test-v8-serialize-leak.js#L18-L31)
- 它在測量 RSS ([resident set size](https://en.wikipedia.org/wiki/Resident_set_size))，因為正在測試的 leak 來自 native side
- 它透過將測量值與 local run 中看起來合理的乘數進行比較，來檢查記憶體開銷是否會消失
- (這是個相當粗糙的測試，但能完成工作)

```js
const v8 = require('v8');

const before = process.memoryUsage.rss();

for (let i = 0; i < 1000000; i++) {
  v8.serialize('');
}

async function main() {
  await gcUntil('RSS should go down', () => {
    const after = process.memoryUsage.rss();
    return after < before * 10;
  });
}

main();
```

到目前為止，這個測試在 CI 中可靠，但它仍依賴於一個不穩定的假設
- `如果 OS 可以回收 native memory，最終 process.memoryUsage.rss() 應該會下降`
- `Resident set size` 是分配給 process 的實體記憶體量。你可能會認為，只要釋放分配的 memory，它就會立即下降
- 實際情況並非如此，主要由 `memory allocator` 來決定何時實際將 memory 傳回給 OS


有時
- 可能存在大量碎片，而且系統沒有記憶體壓力，因此 `memory allocator` 可能認為整理碎片以將未使用的 memory 傳回給 OS 的成本太高，寧願保留它以備 process 需要
- 例如，在最新版本的 `glibc` 中，這種情況經常發生
- 發生這種情況時，根據 `Resident set size` 是否下降來判斷 leak 也會產生誤報
  - 這論點，對基於 heapUsed 的測試也是一樣的

為了解決這個問題
- 我們可以給 V8 多點 memory pressure，並鼓勵它回收更多 memory


----------------

## Small heap + pressure test for OOM failure

這可能是 Node.js 中最常用的 memory leak testing 試策略之一
- (即是是因為 V8 更新而改動了 GC，使它變得越來越不可靠)
- (如果不熟悉 V8 GC 的設計和 generation layout，可以看看這篇 [blog](https://v8.dev/blog/trash-talk))

這個想法本質上是:
1. 將 `maxium heap size` 設為相對較小的值
    - default 下，minial Node.js instance 使用的 V8 heap 約為 3-4 MB
    - 通常使用此策略的測試將 old space 的大小限制為 16-20MB
      - (當存在 leak 時，leak 的 object 及其保留的 graph 通常最終位於 old space 中)
2. 重複測試的操作並確保其 total memory consumption 明顯高於 heap size limit
    - 為了讓測試快點，1 中設定的 heap size 通常較小，以便測試可以透過運行較少的操作快速達到 heap size limit
3. 如果 test 因 Out-Of-Memory 而 crash潰，則表示測試的操作留下了一個 reachable graph，V8 的 GC 即使在壓力下也無法清除 memory，代表存在 leak



大致的範例 如下:
- https://github.com/nodejs/node/blob/2d1bc3d130bdd0c948f5ad5874387ab8ffd04a33/test/parallel/test-shadow-realm-gc.js

```js
// Flags: --experimental-shadow-realm --max-old-space-size=20

for (let i = 0; i < 100; i++) {
  const realm = new ShadowRealm();
  realm.evaluate('new TextEncoder(); 1;');
}
```


V8 的 GC 運作方式也導致了另一個問題
- 通常，步驟 2 是在緊密循環中完成的，上面的範例也是如此
- 在 V8 中，舊的 garbage collection 被設計為在 JS thread 空閒時啟動，避免損害 JS 效能
- 據觀察，在緊密循環中分配記憶體可能會為 GC 留下非常小的運作空間，從而導致測試不穩定


----------------

## Pressure test for OOM failure with room for GC
為了給 V8 的 GC 有點啟動空間並避免誤報，導入了另個實用程式
- https://github.com/nodejs/node/blob/58491eb8097b0b521a1a6e3d8e3d30d0409605df/test/common/gc.js#L71-L76



```js
const wait = require('timers/promises').setTimeout;

// Repeat an operation and give GC some breathing room at every iteration.
async function runAndBreathe(fn, repeat, waitTime = 20) {
  for (let i = 0; i < repeat; i++) {
    await fn();
    await wait(waitTime);
  }
}
```

更新後的測試如下:
- https://github.com/nodejs/node/blob/c5eb2c41902290cdf7e1ee5d6d90fd74e1f3377e/test/parallel/test-shadow-realm-gc.js#L13

```js
// Flags: --experimental-shadow-realm --max-old-space-size=20
'use strict';

runAndBreathe(() => {
  const realm = new ShadowRealm();
  realm.evaluate('new TextEncoder(); 1;');
}, 100);
```

這裡用 `setTimeout()` 給 GC 足夠的時間來啟動
- 這使測試運行稍微慢一些，但可以接受，並且更新後的測試在 CI 中已經足夠穩定



我在這種方法中觀察到的另一個警告是:
- 一旦啟用了 V8 native coverage collection (如 `NODE_V8_COVERAGE`)
- 新編譯的 code 中的 feedback vectors 可以比平常存在得更久
- 因為 V8 需要它們來追蹤呼叫計數
- 如果操作涉及 compiling，步驟 1 中選擇的 heap size limit 必須足夠大以解決此開銷
- 否則即使測試的操作會產生最終可收集的 graph，測試仍然可能會耗盡記憶體

----------------

## Next: finalizer-based testing
As it turns out, testing against memory leaks using memory usage measurements can sometimes be quite tricky. In the next post, I will talk about a different strategy used by Node.js for testing against memory leaks.

下一步：基於終結器的測試
事實證明，使用記憶體使用測量來測試記憶體洩漏有時可能非常棘手。 在下 一篇文章 ，我將討論 Node.js 用於測試記憶體洩漏的不同策略。


----------------

上面討論了如何靠 memory usage measurement 來檢測 memory leaks
- 但有時我們希望測試更加精確、專注於特定物件的狀態
- 例如某些可以跟 V8 garbage collector 互動的 object，上面的方法就很棘手了


----------------

## Weak callback + `gc()`

一個 Node.js 核心測試套件常使用的策略是依賴
-  native [v8::PersistentBase::SetWeak()](https://github.com/nodejs/node/blob/d1d60b297d843b830469aa6cd8730bc972283e84/deps/v8/include/v8-persistent-handle.h#L149-L152) 
- 當觀察到 object 被 GC 了，API 會呼叫 `finalizer`
  - 在 `Node-API` 中，這被抽象化為 [napi_add_finalizer()](https://nodejs.org/api/n-api.html#napi_add_finalizer)

測試的流程大概是這樣:
1. 註冊一個 `process.on('exit') callback` 來檢查對象的 `finalizer` 是否如預期被呼叫
2. 不斷的 allocate 容易洩漏的 object，並且每次分配新 object 時，為其註冊一個 `finalizer`，追蹤該 `finalizer` 是否被呼叫/呼叫了多少次
3. 在 process exit 時，如果 step1 中設定的 callback 發現 `finalizer` 沒有被呼叫足夠多的次數，則該物件被認為 leaking


Node.js core test 中的[範例](https://github.com/nodejs/node/blob/0c30d0e4e02e5b2b125a4eccb29ae8f48b457b8a/test/parallel/test-zlib-invalid-input-memory.js)

下面是簡化的版本:

```js
// Flags: --expose-gc
const assert = require('assert');
const http = require('http');

const max = 100;
let called = 0;

function ongc() { called++; }
process.on('exit', () => { assert.strictEqual(called, max); });

// Checks that server that doesn't listen can still be GC'ed.
for (let i = 0; i < max; i++) {
  const server = http.createServer((req, res) => {});
  onGC(server, { ongc });
}

setImmediate(() => {
  global.gc();
});
```



`onGC()` helper 是在 [FinalizationRegistry](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/FinalizationRegistry) API 可用於 JavaScript 之前引入的
- 它本質上與“ `FinalizationRegistry` 具有相同的用途，並為第一個參數呼叫 `ongc()` callback 作為 `finializer`
- 它是透過 [Node.js’s destroy async](https://nodejs.org/api/async_hooks.html#destroyasyncid) hook 實現的
- 而該 hook 又是透過前面提到的`v8::PersistentBase::SetWeak()` API 實現的


`onGC()` helper 的簡化版本應如下所示:

```js
const async_hooks = require('async_hooks');
const gcTrackerMap = new WeakMap();
function onGC(obj, gcListener) {
  const onGcAsyncHook = async_hooks.createHook({
    init(id, type) {
      if (this.trackedId === undefined) {
        this.trackedId = id;
      }
    },
    destroy(id) {
      if (id === this.trackedId) {
        this.gcListener.ongc();
        onGcAsyncHook.disable();
      }
    },
  }).enable();
  onGcAsyncHook.gcListener = gcListener;

  // Link the lifetime of an async resource with obj.
  // When obj is garbage collected, resource can be
  // garbage collected too, and when resource is gone
  // the destroy hook would be triggered to call
  // gcListener.ongc().
  const resource = new async_hooks.AsyncResource('GC');
  gcTrackerMap.set(obj, resource);
  obj = null;  // Don't keep obj alive in the closure.
}
```

**`FinalizationRegistry` + `gc()`**

`FinalizationRegistry` API
- 大致與上面描述的 `onGC()` helper 具有相同的目的
- 但是 callback 是透過的是[這個機制](https://github.com/nodejs/node/blob/d8c97e48573ef9679751be0c7a308e08d0101c47/deps/v8/src/heap/heap.cc#L6615-L6624)
  - 這機制與 [weak callbacks](https://github.com/nodejs/node/blob/d8c97e48573ef9679751be0c7a308e08d0101c47/deps/v8/src/handles/global-handles.cc#L825-L854) 不相同
- 與 `weak callbacks` 相比，finalization registry callbacks 的呼叫通常發生得較晚且難以預測
- 這是有意設計的，目的是為了讓 JS engine 在 callbacks 的調度方面有更多的餘地，並避免損害性能
- 從技術上講，JS engine 甚至不必呼叫 callbacks (`weak callbacks` 也是如此，但無論如何它們都不那麼複雜)



[proposal 的解釋](https://tc39.es/ecma262/#sec-weakref-invariants):
> `Finalizers` 是一件棘手的事情，最好避免它們。它們可以在意想不到的時間被呼叫，或者根本不被呼叫 ...
> 提議的規範允許 implementations 以任何理由或無理由跳過呼叫 finalization callbacks


如果我們將上面的 test 改用 `FinalizationRegistry`，如下所示:
```js
const assert = require('assert');
const http = require('http');

const max = 100;
let called = 0;

function ongc() { called++; }
process.on('exit', () => { assert.strictEqual(called, max); });

const f = new FinalizationRegistry(ongc);
for (let i = 0; i < max; i++) {
  const server = http.createServer(() => {});
  f.register(server);
}

// Here we must do gc() before run a setImmediate() to
// keep the event loop running for at least another
// iteration, otherwise no tasks scheduled for the finalization
// registry callback would have a chance to run before
// process shutdown.
global.gc();
setImmediate(() => {});
```


但實際上，在發出 `exit` event 時
- callback 只會被呼叫 99 次，至少在作者 local 測試時是這樣
- 如[作者另篇文章分析的](https://joyeecheung.github.io/blog/2023/12/30/fixing-nodejs-vm-apis-3/) Jest 誤報 `--deteck-leaks`  (基於 `FinalizatioRegistry`)表明，你不能使用 `gc()` 來確保在 GC 時為曾經註冊的每個 object callback 呼叫 finalization registry callbacks，即使執行 async `gc()` 10 次
- 因為這不是它們最初設計的目的


一個更 flake 的 test case 可以改變這行:

```js
process.on('exit', () => { assert.strictEqual(called, max); });
```

改成:  
```js
process.on('exit', () => { assert(called > 0); });
```


最終，這取決於你要測試的回歸
- 如果正在測試的每一次重複操作都能可靠地重現 leaking，那麼一份沒有 leaking 的樣本可能已經讓你有 90% 的信心相信已經修復了該問題並且不會再次出現問題
- 當然，你可能希望有 100% 的可信度，並在每個樣本中確認這一點，但考慮到使用 GC 觀察 finalization 可能會在設計上給你帶來誤報
- 因此誤報較少的不太精確的測試比更精確的測試好誤報更多

----------------

## 濫用 heap snapshots 來進行更激進的 GC

濫用堆快照進行更激進的 GC
簡單的 `gc()` 通常不足以清理盡可能多的 objects 並呼叫盡可能多的 callback
- 因為它根本不是為此設計的

多次運行它或保持 thread 運行一段時間(在 Node.js 中，用 `setImmediate()` 來保持 event loop 處於活動狀態)有時可以給 V8 足夠的推動來運行無法訪問的 object 的 `finalizers` (這就是 `Jest` 的 `--detect-leaks` 的作用)
- 但有時這些技巧仍然不夠
- 在這種情況下，依靠 `finalizers` 來告訴你 ojbect 是否可以被收集，並認為 `finalizer` 沒有被呼叫是 leaking 的指示，那麼將出現誤報

`gc()` 還有另個警告
- 如果正在檢查的圖形涉及新編譯的 functino/script，並且你假設 V8 可以在使用者無法存取它們時收集它們(這通常會發生)，那麼 `gc()` 可能會咬你一口
- 因為僅由 `gc()` 引發的強制 GC 就可以阻止它們被回收
- 這是故意的，因為 `gc()` 是 V8 內部 API，僅滿足 V8 本身的測試需求，其中包括此行為




也就是說，有時 regression tests 仍然不可避免地以某種方式強制進行垃圾收集
- 有沒有比 `gc()` 更可靠的替代方法？
- 一些 Node.js 測試用的一種技巧以及後來對 `Jest` 的 `--detect-leaks` 的修復是用 heap snapshot 來執行某種最後手段的垃圾收集
- 按照設計，heap snapshot 的目的是盡可能準確地捕捉 heap 上的活動內容
- 因此獲取它會促使 V8 開始垃圾收集，並進行一些額外的操作，以運行盡可能多的 `finalizers`
- heap snapshot 產生過程也會清除編譯 cache，這有助於清除如果透過 `gc()` 強制進行 GC 則不會收集的 script


這裡有另一個 Node.js [helper](https://github.com/nodejs/node/blob/58491eb8097b0b521a1a6e3d8e3d30d0409605df/test/common/gc.js#L33-L68)  
- 簡化版的如下


```js
async function checkIfCollectable(fn, maxCount, generateSnapshotAt) {
  let anyFinalized = false;
  let count = 0;

  const f = new FinalizationRegistry(() => {
    anyFinalized = true;
  });

  async function createObject() {
    const obj = await fn();
    f.register(obj);
    if (count++ < maxCount && !anyFinalized) {
      setImmediate(createObject, 1);
    }
    // This can force a more thorough GC, but can slow the test down
    // significantly in a big heap. Use it with care.
    if (count % generateSnapshotAt === 0) {
      require('v8').getHeapSnapshot().pause().read();
    }
  }

  createObject();
}
```



上面這個 helper 用了一個 object factory `fn()`，並運行它最多 `maxCount` 次
- 理想情況下，heap 大小限制也應該設較小的值，以便 V8 在分配發生時有某種緊急感來清理 constructed objects
- 如果在處理過程中呼叫了從 `fn()` 傳回的任何 object 的 `FinalizationRegistry` callback，我們就知道至少其中一些 ㄖobject 在記憶體壓力下是可收集的
- 那麼我們就有足夠的信心來反駁 leaking
- 為了給 V8 額外的推動來呼叫 `finalizer`，我們也將以指定的頻率取得 heap snapshot



要使用這個 helper，要拿這個[測試](https://github.com/nodejs/node/blob/017998971bffd8cb35f384de3a6da40290aafd61/test/es-module/test-vm-contextified-script-leak.js#L1-L16)為例
- (有關這個錯誤的詳細分析，[查看其他文章](https://joyeecheung.github.io/blog/2023/12/30/fixing-nodejs-vm-apis-3/))

```js
// Flags: --max-old-space-size=16 --trace-gc
const vm = require('vm');

// Tests that vm.Script compiled with a custom
// importModuleDynamically() callback doesn't leak.
async function createContextifyScript() {
  // Try to reach the maximum old space size.
  return new vm.Script(`"${Math.random().toString().repeat(512)}";`, {
    async importModuleDynamically() {},
  });
}
checkIfCollectable(createContextifyScript, 2048, 512);
```


這種方法在一段時間內將這些問題排除在 CI 之外，直到 Node.js 將 V8 更新到了新版本


----------------


## 基於堆疊迭代(heap iteration-based)的測試
下面將討論另種更可靠的策略，該策略用於修復來自較新版本 V8 的碎片  


在上面介紹的策略將 heap snapshot 技巧描述為對 API 的濫用(abuse)
- 因為 heap snapshot 並非設計用於與 heap 中運行的 `finalizers` 進行互動
- 但使用它來識別/反駁 leaking 的概念本身並不是一種濫用
  - 這正是它們的設計目的
  
那麼，可以使用 heap snapshots 或其更簡單的版本來測試最大的 memory leaks 嗎？

----------------

## Chrome DevTools console API
Technically, we can already do the testing using existing APIs and there is no need to use the finalizers. Just take two heap snapshots before and after a certain amount of allocation, and find the difference in them to learn whether certain kind of objects can be garbage collected - this is an intended & documented use case of heap snapshots. It’s just that using finalizers to monitor specific objects is simpler and faster than parsing & diffing the heap snapshots generated, so we took the shortcut here. But what if we can do the diffing without generating a heap snapshot at all?

## Chrome  DevTools console API
從技術上講，我們已經可以使用現有的 API 進行測試，而無需使用 `finalizers`
- 只要在一定量的分配的前、後截取兩個 heap snapshots，然後找出之間的差異，就能知道某 objects 是否可以被 GC
  - 這就是 heap snapshots 的預期 use case
- 只是，使用 `finalizers` 監控特定 object 比解析和比較生成的 heap snapshots 更簡單、更快速
  - 所以在這裡走了捷徑
  
但如果可以在不產生 heap snapshots 的情況下進行差異處理呢？  


Chromium DevTools 中有個 console API 可以實現類似的 use case
- [queryObject(constructor)](https://developer.chrome.com/docs/devtools/console/utilities?hl=zh-tw)
- 它可以執行與 heap snapshots 相同的相當激進的 GC，並搜尋當前執行 context 以查找其 prototype chain 包含該 constructor’s prototype 的 object

```js
// In the DevTools console:

class A {}
const a = new A();
// Returns undefined, then logs an array that is just [a]
queryObjects(A);
```


----------------

### Node.js 中的內部 API 和新的 helper
現在我們有了一個新的 V8 API，可以在 aggressive 的 GC 之後搜尋 heap 上的 object
- [v8::HeapProfiler API](https://chromium-review.googlesource.com/c/v8/v8/+/5006373)
  - 允許 embedders 進行類似的搜索，並收集符合 customized predicate 的 object references
- 下一步就是利用它設計 memory leak testing 策略

首先
- Node.js 加入了一個 [countObjectsWithPrototype(prototype) helper](https://github.com/nodejs/node/pull/50572)
  - (將只用於 Node.js 自身的測試，以檢查該策略是否足以替代)
- 這與 `queryObjects()` 類似，只是它直接取得 prototype，並傳回找到的 object 數量，這正是測試所需要的




```js
class A {}
const a = new A();
countObjectsWithPrototype(A.prototype);  // 1
```


使用新的 API，一個 native 的 memory leak checker 可以像這樣實作:

```js
async function checkIfCollectableByCounting(fn, klass, count) {
  const initialCount = countObjectsWithPrototype(klass.prototype);

  for (let i = 0; i < count; ++i) {
    // Here, fn() should create one and only one object
    // of klass.
    const obj = await fn(i);
  }
  const remainingCount = countObjectsWithPrototype(klass.prototype);
  const collected = initialCount + count - remainingCount;
  if (collected > 0) {
     console.log(`${klass.name} is collectable (${collected} collected)`);
     return;
  }
  throw new Error(`${klass.name} cannot be collected`);
}

// Usage:
const leakMe = [];
class A { constructor() { leakMe.push(this); } }
function factory() { return new A; }
// It will throw because all the A created are put into an array
// that's still alive and therefore not collectable.
checkIfCollectableByCounting(factory, A, 1000);
```



但是，當嘗試使用這種策略來 deflake [一個特別脆弱的 memory leak test](https://github.com/nodejs/node/pull/50572)時，效果並不理想
- 如果 `fn()` 產生的圖充滿了 weak referenes，而 object 建立循環又使 heap 增長過快，一旦將 heap size 限制為一個較小的值，V8 執行的最後手段 GC 就不足以在我們呼叫第二個 `countObjectsWithPrototype()` 之前阻止程式耗盡 memory
- 此外還會遇到誤報
  - (在正常 app中，成長速度不應該如此之快，因為 user 很少會在 loop 中建立如此多的  weak references)




那如果
- 在 object creation 的 loop 的每次迭代中進行 count
- 並在檢測到建立的任何 object 被收集時提前結束測試
- 但由於 `checkIfCollectableByCounting()` 會產生 GC 並遍歷 heap，會耗費大量時間，大幅降低測試速度


經過一些調整，加以下技巧似乎可以讓測試足夠可靠:
1. 不要每次迭代只建立一個 object，也不要在一個循環中建立所有 object，而是分批建立
    - 這樣可以讓 heap 長得足夠快，以便測試能快速完成
    - 但也不會太快，以免在生成的圖形過於複雜而無法有效 GC 時，V8 的 GC 無法跟上
2. 每次批次後都給 GC 一點時間完成工作
3. 在每個批次後檢查是否有 object 被收集，而不是等到所有都建立完成後再檢查。有助於更快完成測試

最終版本如下

```js
const wait = require('timers/promises').setTimeout;

async function checkIfCollectableByCounting(fn, klass, count, waitTime = 20) {
  const initialCount = countObjectsWithPrototype(klass.prototype);

  let totalCreated = 0;
  for (let i = 0; i < count; ++i) {
    // Here, fn() create the objects in batches, and return the count
    // of objects created in the batch.
    const created = await fn(i);
    totalCreated += created;
    await wait(waitTime);  // give GC some breathing room.

    // If we already find some collected objects after
    // processing a batch, it's good enough to stop.
    const currentCount = countObjectsWithPrototype(klass.prototype);
    const collected = initialCount + totalCreated - currentCount;

    if (collected > 0) {
      console.log(`Detected ${collected} collected ${name}, finish early`);
      return;
    }
  }

  // Final check.
  await wait(waitTime);  // give GC some breathing room.
  const currentCount = countObjectsWithPrototype(klass.prototype);
  const collected = initialCount + totalCreated - currentCount;

  if (collected > 0) {
    console.log(`Detected ${collected} collected ${klass.name}`);
    return;
  }

  throw new Error(`${klass.name} cannot be collected`);
}

// Usage:
const leakMe = [];
class A { constructor() { leakMe.push(this); } }
function factory() {
  for (let i = 0; i < 100; ++i) new A;
  return 100;
}
// It will throw because all the A created are put into an array
// that's still alive and therefore not collectable.
checkIfCollectableByCounting(factory, A, 10);
```

----------------



## 新的 Node.js  API: `v8.queryObjects()`
到目前為止，上述版本運行得相當不錯
- CI 中不再有 flakes
- 這似乎對 Node.js User 也很有用
  - 因此，[將其公開到 Node.js 的 v8 library 中](https://github.com/nodejs/node/pull/51927)


這 API 與 Chrome DevTools console API 更為相似
- 只不過它傳回的是資訊而非僅記錄資訊
- 為避免意外 leaking 找到的 object reference，API 不回傳 reference
- 預設情況下，它會返回 count，這對於測試來說通常已經足夠
- 為了幫助 debug，它可以傳回找到的 object 的  string summaries

```js
const { queryObjects } = require('v8');
class A { foo = 'bar'; }
console.log(queryObjects(A)); // 0
const a = new A();
console.log(queryObjects(A)); // 1
// [ "A { foo: 'bar' }" ]
console.log(queryObjects(A, { format: 'summary' }));
```



由於此 API 是基於 prototype 行走(prototype walking)
- 類似於 Chrome DevTools API
- 所以要注意，如果 class 是繼承的，則 child class 的 prototype chain 含傳入 API 的 constructor 的 prototype
  - 因此  child class 的 prototype 被視為匹配(match)


```js
class B extends A { bar = 'qux'; }
const b = new B();
console.log(queryObjects(B)); // 1
// [ "B { foo: 'bar', bar: 'qux' }" ]
console.log(queryObjects(B, { format: 'summary' }));

console.log(queryObjects(A));  // 3
// [ "B { foo: 'bar', bar: 'qux' }", 'A {}', "A { foo: 'bar' }" ]
console.log(queryObjects(A, { format: 'summary' }));

```



使用新 API 後，helper 只需將 `countObjectsWithPrototype(klass.prototype)` 改為 `v8.queryObjects(klass)`，而不再依賴內部 API 即可運作
- 希望這新 API 也能幫助其他人在 memory leak regression test 中減少誤報