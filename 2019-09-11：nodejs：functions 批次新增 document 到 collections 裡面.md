## nodejs：functions 批次新增 document 到 collections 裡面
### nodejs how functions to create documents into collections with auto generate ID

### keyword
`firebase`, `functions`, `firestore`, `batch`, `add`, `set`, `gener`, `automatically-generated unique ID`, `nodejs`

### 相關參考文件
- https://googleapis.dev/nodejs/firestore/latest/WriteBatch.html
- https://firebase.google.com/docs/firestore/manage-data/add-data?hl=zh-cn
- https://firebase.google.com/docs/firestore/manage-data/transactions

指定 credential 的部分參考
- 「2019-09-09：Firebase：nodejs 設定 server 對 server 生產應用程式的驗證作業.md」


用
- `batch` 做批次操作  
- `doc()` 讓 firestore 自己產生 key (要指定 key 的話，就傳字串進去)

```js
const admin = require('firebase-admin')

const faker = require('faker/locale/zh_TW');
const dateformat = require('date-fns/format');
const parseISO = require('date-fns/parseISO');

const firebaseConfig = {
  "databaseURL": "https://xxxxxxxxx.firebaseio.com",
}

const serviceAccount = require("./xxxxxxxxx-2f9b5f40b0e8.json");

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount),
  ...firebaseConfig
});

const db = admin.firestore();
const batch = db.batch();

const customerColRef = db.collection('customers')

const addr = `${faker.address.zipCode("#####")}${faker.address.cityPrefix()}${faker.address.citySuffix()}${faker.address.streetAddress()}`

Array.from({ length: 5 }, () => {
  batch.set(
    customerColRef.doc(),
    {
      _delete: false,
      name: `${faker.name.findName()}`,
      title: `${faker.name.prefix()}`,
      phone: `${faker.phone.phoneNumberFormat()}`,
      email: faker.internet.email(),
      birth: parseISO(dateformat(new Date(faker.date.past()), 'yyyy-MM-dd')),
      age: faker.random.number({ min: 20, max: 75 }),
      addresses: [addr],
      passport: `${faker.internet.password(12)}`,
    }
  )
})

batch
  .commit()
  .then(function (res) {
    console.log('Batched 執行完成：');
    console.log(res)
    
  })
  .catch(error => {
    console.log('有問題，Error：')
    console.log(error)
  })

console.log('==========================')
```