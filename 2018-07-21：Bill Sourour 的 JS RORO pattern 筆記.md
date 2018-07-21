# Bill Sourour 的 JS RORO pattern 筆記
原文標題：Elegant patterns in modern JavaScript: RORO
來源：[https://medium.freecodecamp.org/elegant-patterns-in-modern-javascript-roro-be01e7669cbd](https://medium.freecodecamp.org/elegant-patterns-in-modern-javascript-roro-be01e7669cbd)

## RORO 就是指 接 Object、回也回 Object 的設計 function 方式

長知識，除了 function 用 Object 傳入，我本來知道以外，其他這樣的用法跟方式，感覺未來有機會可以學學。

function 改接收 Object 這樣可以讓程式碼看起來更清楚，更了解目的。
```javascript
// define funciont
function findUsersByRole ({
  role,
  withContactInfo, 
  includeInactive
} = {}) {...}
// call function
findUsersByRole({
  role: 'admin', 
  withContactInfo: true, 
  includeInactive: true
})
```


除此之外！ **解構宣告** 是宣告新的變數並將變數傳進來。
這意味著你不會改到原本傳進來的 Object (長知識)。
```javascript
const options = {
  role: 'Admin',
  includeInactive: true
}
findUsersByRole(options)
function findUsersByRole ({
  role,
  withContactInfo, 
  includeInactive
} = {}) {
  role = role.toLowerCase()
  console.log(role) // 'admin'
  ...
}
console.log(options.role) // 'Admin'
```

## 更好檢查變數的 pattern
```javascript
// 之前常常這樣寫吧？
function findUsersByRole ({
  role, 
  withContactInfo, 
  includeInactive
} = {}) {
  if (role == null) { 
    throw Error(...)
  }
  ...
}
```

可以這樣改善設計
```javascript
// 宣告一個這樣的 function
function requiredParam (param) {
  const requiredParamError = new Error(
   `Required parameter, "${param}" is missing.`
  )
  // preserve original stack trace
  if (typeof Error.captureStackTrace === ‘function’) {
    Error.captureStackTrace(
      requiredParamError, 
      requiredParam
    )
  }
  throw requiredParamError
}
// 然後定義時這樣定義
function findUsersByRole ({
  role = requiredParam('role'),
  withContactInfo, 
  includeInactive
} = {}) {...}
```
結果會是
 > if anyone calls findUsersByRole without supplying a role they will get an Error that says Required parameter, “role” is missing.

## 用 == 可以同時判斷 null 跟 undefined
```javascript
// 高手經驗談 長知識
if (role == null) { 
    throw Error(...)
  }
```

## return Object 的資訊更加豐富
```javascript
async saveUser({
  upsert = true,
  transaction,
  ...userInfo
}) {
  // save to the DB
  return {
    operation, // e.g 'INSERT'
    status, // e.g. 'Success'
    saved: userInfo
  }
}
```

# Easy Function Composition
這主題更讚，簡單卻很威的感覺能做到 FP 的設計
（這樣算是 FP 嗎？）

> “Function composition is the process of combining two or more functions to produce a new function. Composing functions together is like snapping together a series of pipes for our data to flow through.” — Eric Elliott

看完下面的例子，真的很優雅！

```javascript
// 先來一個這樣的 function
function pipe(...fns) { 
  return param => fns.reduce(
    (result, fn) => fn(result), 
    param
  )
}

// 在這樣呼叫就好了
function saveUser(userInfo) {
  return pipe(
    validate,
    normalize,
    persist
  )(userInfo)
}

// 這些 中間的 function 宣告時，只處理自己用到的變數就好
function validate({
  id,
  firstName,
  lastName,
  email = requiredParam(),
  username = requiredParam(),
  pass = requiredParam(),
  address,
  ...rest
}) {
  // do some validation
  return {
    id,
    firstName,
    lastName,
    email,
    username,
    pass,
    address,
    ...rest
  }
}
function normalize({
  email,
  username,
  ...rest
}) {
  // do some normalizing
  return {
    email,
    username,
    ...rest
  }
}
async function persist({
  upsert = true,
  ...info
}) {
  // save userInfo to the DB
  return {
    operation,
    status,
    saved: info
  }
}
```