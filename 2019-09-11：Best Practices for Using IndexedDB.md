## Best Practices for Using IndexedDB
### Philip Walton
#### https://developers.google.com/web/fundamentals/instant-and-offline/web-storage/indexeddb-best-practices

web 第一次載入時
- 有很多東西要 load
- 有很多 API 要打

可以靠 IndexedDB 
- 加快「重複訪問」的「加載時間」
- update the UI with new data lazily
- 採用 `stale-while-revalidate` 策略
  - 這邊有篇中文的，詳細介紹每一種`離線策略`
  - https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/?hl=zh-tw

另一個使用方向是
- 存儲用戶生成的內容
- 可以在`上傳到 server 之前作為臨時存儲`
- 也可以作為遠程數據的客戶端緩存
- 當然，兩者都存儲。

本文回答了常見問題，並討論了在IndexedDB中保存數據時要記住的一些最重要的事項。

### 保持您的應用程序可預測
IndexedDB 的許多複雜性源於 develop 無法控制的因素很多  
這邊探討了
- 使用 IndexedDB 時**必須牢記的許多問題**

不是所有平台上的所有內容都可以存在 IndexedDB 中
- 大檔案（image, video)，可以存為 `File` or `Blob`
  - 這招絕大多數都能 work
  - 但 iOS Safari 不能把 `Blob` 存到 `IndexedDB`
  
解法
- 將 `Blob`轉 `ArrayBuffer`
  - `IndexedDB` 中存 `ArrayBuffers` 支持度很好

但！
- `Blob` 具有 `MIME` 類型，而 `ArrayBuffer` 不具有 `MIME`類型。
-  You will need to store the `type` alongside the `buffer` in order to do the conversion correctly.

To convert an `ArrayBuffer` to a `Blob` you simply use the Blob constructor.

```js
function arrayBufferToBlob(buffer, type) {
  return new Blob([buffer], {type: type});
}
```

轉過去時，用 `FileReader` 讀 `blob` 當做 `ArrayBuffer`  
檔案讀完時，`loadend` 事件會被 triggered on the reader  

```js
function blobToArrayBuffer(blob) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.addEventListener('loadend', (e) => {
      resolve(reader.result);
    });
    reader.addEventListener('error', reject);
    reader.readAsArrayBuffer(blob);
  });
}
```


### Writing to storage may fail storage 寫入失敗時
有些情況，IndexedDB 還是會出錯了  
- 例如 browser 的 private mode 有些會禁止使用 IndexedDB  
- device 的「空間沒了」...，browser 也會擋

因為這些問題，所以 `error handle` 是絕對需要的，這也指出另一個概念
- keep application state in memory (in addition to storing it)

這樣 UI 才不會掛掉，例如在 private mode 的時候  

這樣 catch error
create `IDBDatabase`, `IDBTransaction` or `IDBRequest` object 的時候

```js
const request = db.open('example-db', 1);
request.addEventListener('error', (event) => {
  console.log('Request error:', request.error);
};
```


### data 可能被 Uesr 修改 or 刪除
修改可能不常見，但 data 被刪除倒是很常見  
User 用 devtool 就能操作  

所以 App 在這情況下不掛掉，也是重要問題  

### data 可能是過期的！
data 可能是`「舊的 code 寫進去的，格式、版本可能不對」`

`IndexedDB` 有 schema versions and upgrading
- `IDBOpenDBRequest.onupgradeneeded()`

升級了 schema，但還是要有 code 來處理舊的 data  

`單元測試` 來針對這些情況，會很有幫助！
- 手動測試所有可能的升級路徑和案例，大概會死人，而且不完整


### 保持 App performant

IndexedDB
- 一個關鍵特性是它的 async API
- 但！**不要誤以為在使用它時不需要擔心性能**
  - **不正確的使用「仍然可以阻止主線程」**，可能導致jank和無響應。

一般原則
- 「`IndexedDB` 的讀取和寫入」**不應該大於**「所訪問數據的讀取和寫入」。


雖然 IndexedDB 可以用來存 large, nested objects 為單個記錄  
- 從開發人員的角度來看，這樣做非常方便
- **但**應該避免這種做法

原因是當 IndexedDB 存儲對象時
- 它必須首先 create 一個目標 object 的 structured clone 
- 這個 clone 是在 main thread 上執行的，所以大檔案，就可能阻塞很久

所以，要如果規劃把 App 的 state tree persist store 到 IndexedDB，就是個大挑戰！  
所此，熱門的 state management（redux），都還是將 state tree 做為「單一一個 JS Object」做管理  

這種（單一 Object）狀態管理的方法很好管理，存到 IndexedDB 也很方便  
- 但每次更新 state 時，比較容易出錯 or 阻塞 main thread
  - 導致 browser 變慢 or 卡住
  
所以！
- 不應該把整個 state tree 存到單一 JS Object 去
- 應該要**拆開存**並且**只更新有 update 的 data 的部分**

如果存 image, video, audio 也是依樣
- 每一個 item 用一個自己的 key 來存成一個 larger object
- 這樣能取回 結構化的  data，還不會浪費 cost 去取 binary file

雖然
- IndexedDB 的「小寫操作」比「大寫操作」更好

但這只適用於
- App正在執行的對 IndexedDB 的寫入實際上會導致阻塞 main thread並降低用戶體驗的長任務

衡量這一點非常重要，才能了解自己的優化內容。

### 結論
開發人員可以利用像 IndexedDB 這樣的客戶端存儲機制
- 來改善其應用程序的用戶體驗
- 不僅可以跨會話保持狀態，還可以減少重複訪問時加載初始狀態所需的時間。
- 正確使用 IndexedDB 可以顯著改善用戶體驗
- 錯誤使用 IndexedDB 或**無法處理錯誤情**況可能會導致應用程序損壞和用戶不滿意。

由於 client 涉及控制範圍之外的許多因素
- 因此**行充分測試並正確處理錯誤至關重要**，即使最初看起來不太可能發生錯誤也是如此。