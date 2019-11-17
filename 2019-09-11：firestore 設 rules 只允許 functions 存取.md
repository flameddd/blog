## firestore 設 rules 只允許 functions 存取
### Doug Stevenson Jul 18 '18
#### https://stackoverflow.com/questions/51405539/cloud-firestore-security-rules-allow-write-only-from-firebase-function

## 我自己還沒驗證過這方法
先寫下來後續可能會用到  

-----------------------------

`Functions` 為 firebase 體系
- 自然就有權取用 `Firebase Admin SDK` 去存取其他 firebase 產品  
  - 不需要設定 `firestore` 的 `rules` 了

如果想要，擋住 `client（web）` 端存取 `firestore`
- simply reject access to all clients entirely
- let the Admin SDK do its work

就可以了  

例如
```js
service cloud.firestore {
  match /databases/{database}/documents {

    // Match any document in the 'cities' collection
    match /cities/{city} {
      allow read: if false;
      allow write: if false;
    }
  }
}
```

例如
```js
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read: if false;
      allow write: if false;
    }
  }
}
```