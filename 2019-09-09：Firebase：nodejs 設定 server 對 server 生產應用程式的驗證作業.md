## Firebase：nodejs 設定 server 對 server 生產應用程式的驗證作業
#### https://cloud.google.com/docs/authentication/production
#### https://console.cloud.google.com/apis/credentials/serviceaccountkey
#### https://firebase.google.com/docs/reference/admin/node/admin.credential.html#cert

暫且不想設定在環境變數中，官方的範例預設為  
先設定成環境變數
```Shell
export GOOGLE_APPLICATION_CREDENTIALS="/home/user/Downloads/[FILE_NAME].json"
```

在用這指令去設定
```js
admin.initializeApp({
  credential: admin.credential.applicationDefault(),
  databaseURL: "https://<DATABASE_NAME>.firebaseio.com"
});
```

但我還不想設定這些東西  
所以用其他方法

1. 先去 serviceaccountkey 下載 key (JSON)到本機去  
2. 把 JSON 存到專案目錄去（記得 `git ignore` !!!!，避免上傳上去了）
    - `git ignore`!!!
    - `git ignore`!!!
    - `git ignore`!!!
3. 用 `admin.credential.cert` 載入 `credential`
    - https://firebase.google.com/docs/reference/admin/node/admin.credential.html#cert

### 程式碼
```js
const admin = require('firebase-admin')
const serviceAccount = require("./aaa-dev-2f9b5f40b0e8.json");

const firebaseConfig = {
  "databaseURL": "https://energyx-dev.firebaseio.com",
}

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount), // <--
  ...firebaseConfig
});

const db = admin.firestore();

let cityRef = db.collection('customers').doc('xxxxxxxxxxx');
cityRef.get()
  .then(doc => {
    if (!doc.exists) {
      console.log('No such document!');
    } else {
      console.log('Document data:', doc.data());  // <- 成功拿到 data
    }
  })
  .catch(err => {
    console.log('Error getting document', err);
  });
```