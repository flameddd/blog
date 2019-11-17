## firebase pagination

相關參考
- https://www.youtube.com/watch?v=poqTHxtDXwU
- https://firebase.google.com/docs/firestore/query-data/query-cursors

影片的方法，我思考過後這應該用在 client 端的做法  
如果是用在 `functions` 應該沒辦法  
因為沒有 `myQuery` ref  

影片有特別提到，你的 pagination 到底要多麼 `realtime`？  
換頁時，data 有變動的話，就會有些 miss 了  
這很重要，會影響實作，當然也要付出成本  

像是 chat 就沒關係  
- 因為過去的對話，是不會變順序的

```js
myQuery = restaurantRef
  .whereField("city", isEqualTo: "Tokyo")
  .whereField("category", isEqualTo: "tempura")
  .order(by: "rating", descending: true)
  .limit(to: 20)


myQuery = restaurantRef
  .whereField("city", isEqualTo: "Tokyo")
  .whereField("category", isEqualTo: "tempura")
  .order(by: "rating", descending: true)
  .limit(to: 20)
  .start(after: ["Tokyo", "tempure", 4.9])

nextBatch = myQuery.start(after:
  ["Tokyo", "tempure", 4.9]
)

myQuery = myQuery.start(after:
  ["Tokyo", "tempure", 4.9]
)

myQuery = myQuery.start(after: previousDoc)

// 要維持 real time feature 的話
// .limit(to: 20)
// .limit(to: 40)
// .limit(to: 60)
// .limit(to: 80)
// 就要這樣全部 select 出來了，但這樣也是成本問題！！
```


### `functions`
上面的方式 `firebase functions` 估計是沒辦法  
因為 client 每次呼叫，都是一個 stateless 的狀態行為  
所以我的 case 用文件的作法可能才是解法  

下面這些 API 應該能 work
```js
let startAt = db.collection('cities')
  .orderBy('population')
  .startAt(1000000);

let endAt = db.collection('cities')
  .orderBy('population')
  .endAt(1000000); // endBefore()

let docRef = db.collection('cities').doc('SF');
return docRef.get().then(snapshot => {
  let startAtSnapshot = db.collection('cities')
    .orderBy('population')
    .startAt(snapshot);

  return startAtSnapshot.limit(10).get();
});
```

最後我個人的問題是  
現在我採用 `material table`  
這要怎麼 `async pagination`  ＝ = ？？？


### `startAt`
後來了解到後，真是傻眼  
這 `startAt` 不是 `index`，它是 `key`  
所以 firebase 是不支援 index range select （performance issue）  
- [startAt() and startAfter() methods don't work #1235](https://github.com/firebase/firebase-js-sdk/issues/1235)


未來要設計時，一定要有個到時預計來 `sort` 的 `key` 用

20190918 第一次算是成功的版本  

這是不算好的例子，因為 name 會重複  
```js
//client
const getCustomerData = async () => {
    const lastName = customerData.length > 0
      ? customerData[customerData.length -1].name
      : ''
    const { data } = await functions
      .httpsCallable('getCustomers')({ page, pageSize: rowsPerPage, lastName })

    setCustomerData(data)
  }

React.useEffect(
  () => {
    getCustomerData()
  },
  [page, rowsPerPage]
);
```

```js
// backend functions

export const getCustomers = tokyoFunctions
  .https
  .onCall(async (data, context) => {
    try {
      const { pageSize, lastName } = data;
      const customerSnapshot = await db.collection('customers')
        .orderBy('name')
        .startAfter(lastName) // <-- 靠 client 告訴我，上次最後一筆是哪一筆
        .limit(pageSize)
        .get();

      if (customerSnapshot.empty) {
        return []
      }
      
      return customerSnapshot.docs.map(snapshot => snapshot.data())
      

    } catch (error) {
      console.log('======= Err =========');
      console.log(error);
      return error
    }
  });
```