## 關於 firebase functions counter (not database counter)

卡了一天  
對於 `firebase 新手` 還是需要探索  

註：functions 我是新手，可能會有更好的寫法。另外寫這篇時，我還沒實作 recount （參考官方 `child-count` 裡面的 `recountlikes`）功能

### `database` !== `firestore`

後面才意識到，`firebase database` 跟 `firebsae firestore` 是不同的東西  
一開始看文件知道這兩者，但在使用 API 時，沒意識到！！！  
因為 後台 console 都在 `database` 裡面啊！！！！  
所以官方的 sample `functions-samples child-count`
- https://github.com/firebase/functions-samples/blob/master/child-count/functions/index.js

其實是用 `database` 作為範例  
我想 count 的是 `firestore` 所以一直沒 trigger 到  

### example about `firestore`
總之，API 就是跟**官方範例的 child-count** 不太一樣

一些我參考到的 ref

`firestore`, `increment`
- https://gist.github.com/saintplay/3f965e0aea933a1129cc2c9a823e74d7

`update`, `increment`
- https://fireship.io/snippets/firestore-increment-tips/#increase-a-counter

`batch`, `increment`
- https://stackoverflow.com/questions/55816018/firebase-cloud-function-unhandled-error-error-update
 
部署失敗：Cloud Functions for Firebase error: “400, Change of function trigger type or event provider is not allowed”
- firebase functions:delete yourFunction
- firebase deploy
- https://stackoverflow.com/questions/46530361/cloud-functions-for-firebase-error-400-change-of-function-trigger-type-or-eve

我的檔案結構
db
- customers
  - customerId1
  - customerId2
  - customerId3
  - customerId4
  - customerId5
- counters
  - customers: {
    docCount: 0
  }

會這樣設計 docCount，是因為後續我想另外加入 activeCount 在這邊，一般可能不需要這麼複雜

最後我的版本
```js
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

admin.initializeApp();
const db = admin.firestore();

exports.countCustomers = functions.firestore
  .document('customers/{customerId}')
  .onWrite(async (change) => {
    const countRef = db.collection('counters').doc('customers');
    const batch = db.batch();

    if (change.after.exists && !change.before.exists) {
      batch.update(countRef, {
        docCount: admin.firestore.FieldValue.increment(1)
      });
    } else if (!change.after.exists && change.before.exists) {
      batch.update(countRef, {
        docCount: admin.firestore.FieldValue.increment(-1)
      });
    }

    batch
      .commit()
      .then(function () {
        console.log('Counter updated.');
      })
      .catch(error => {
        console.log('Error writing document: ' + error);
      })  
  });
```