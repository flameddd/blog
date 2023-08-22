
# 2023-08-21：學習 TypeScript，兩篇關於 object 的案例.md
## Matt Pocock, (Jun 15, 2023), (Jul 18, 2023)

Ref: 
- https://www.totaltypescript.com/iterate-over-object-keys-in-typescript
- https://www.totaltypescript.com/get-keys-of-an-object-where-values-are-of-a-given-type

--------

兩篇關於 TypeScript 處理 object 時的 user case
1. Get Keys of an Object Where Values Are of a Given Type
2. How to Iterate Over Object Keys in TypeScript


--------


## 如何用 TypeScript iterate Object 的 keys

在 TS 中 iterating object keys 非常麻煩，下面介紹幾種方案


快速解釋
- Iterating `Object.keys` 不起作用，是因為 `Object.keys` 返回的是 `string[]`，而不是所有 keys 的 union




```ts
function printUser(user: User) {
  Object.keys(user).forEach((key) => {
    // Doesn't work!
    console.log(user[key]);
// Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'User'.
  // No index signature with a parameter of type 'string' was found on type 'User'.
  });
}

```


在正確的位置將 key 值轉為 `keyof typeof` 就可以了:

```ts
const user = {
  name: "Daniel",
  age: 26,
};
 
const keys = Object.keys(user);
 
keys.forEach((key) => {
              //(parameter) key: string

  console.log(user[key as keyof typeof user]);
});
```

通過縮小 inline type 的範圍，custom type predicate 也可以起作用

```ts
function isKey<T extends object>(
  x: T,
  k: PropertyKey
): k is keyof T {
  return k in x;
}
 
keys.forEach((key) => {
  if (isKey(user, key)) {
    console.log(user[key]);
                    // (parameter) key: "name" | "age"
  }
});
```


--------


## 更詳細的解釋

問題在，使用 `Object.keys` 並不像期望的那樣有效
- 因為它沒有返回你所需要的 type
- 它沒有返回包含所有 key 的 type，而是 `string[]`



```ts
const user = {
  name: "Daniel",
  age: 26,
};
 
const keys = Object.keys(user);
       // const keys: string[]
```



這意味著你無法使用 key 訪問 object 上的 value:


```ts
const nameKey = keys[0];
        // const nameKey: string
 
user[nameKey];
// Element implicitly has an 'any' type because expression of type 'string' can't be used to index type '{ name: string; age: number; }'.
//   No index signature with a parameter of type 'string' was found on type '{ name: string; age: number; }'.
```



TypeScript 返回 `string[]` 是有原因的
- TypeScript object type 是開放式的
- 在很多情況，TS 無法保證 `Object.keys` 返回的 key 確實在 object 上，因此 `string[]` 是唯一合理的解決方案
  - (詳情: https://github.com/Microsoft/TypeScript/issues/12870)


--------

## `For...in` loops
You'll also find this fails if you try to do a for...in loop. This is for the same reason - that key is inferred as a string, just like `Object.keys`.


## For...in 循環
如果你嘗試用 `for...in` loop
- 會發現這個方法也失敗
- 這是出於同樣的原因，`key` 被推斷為 `string`，就像 `Object.keys` 一樣




```ts
function printUser(user: User) {
  for (const key in user) {
    console.log(user[key]);
// Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'User'.
  // No index signature with a parameter of type 'string' was found on type 'User'.
  }
}
```



但在很多情況下，你會確信自己知道 object 的形狀，那麼，你會怎麼做呢？


--------



## 解決方案 1：轉為 `keyof typeof`
使用 `keyof typeof` 將 key 轉換為**更特定的 type**

下面，我們將 `Object.keys` 的結果轉換為包含這些 key 的 array
```ts
const user = {
  name: "Daniel",
  age: 26,
};
 
const keys = Object.keys(user) as Array<keyof typeof user>;
 
keys.forEach((key) => {
              //(parameter) key: "name" | "age"

  // No more error!
  console.log(user[key]);
});
```


也可以在使用 object index 時這樣做
- 在這，key 仍然為字符，但當 user 用 index 時，將其轉為  `keyof typeof user`

```ts
const keys = Object.keys(user);
 
keys.forEach((key) => {
              // (parameter) key: string

  console.log(user[key as keyof typeof user]);
});
```


不過，任何形式使用 `as` 通常都是不安全的，這邊也不例外
```ts
const user = {
  name: "Daniel",
  age: 26,
};
 
const nonExistentKey = "id" as keyof typeof user;
            //const nonExistentKey: "name" | "age"
 
// No error!
const value = user[nonExistentKey];
```

在這情況下，`as` 是強大的工具
- 正如你所看到的，它可以**欺騙** TypeScript 某些東西的 type



--------


## 解決方案 2: Type Predicates
來看看 Type Predicates，稍微更聰明、更安全的解決方案
- 通過用 `isKey` function 輔助，我們可以在呼叫 object index 之前檢查 key 是否確實在 object 上
- `isKey` 的返回 type 中使用 `is` 語法，可以讓 TypeScript 正確推斷



```ts
function isKey<T extends object>(
  x: T,
  k: PropertyKey
): k is keyof T {
  return k in x;
}
 
keys.forEach((key) => {
  if (isKey(user, key)) {
    console.log(user[key]);
                     // (parameter) key: "name" | "age"

  }
});
```

(這超棒的方案來自 Stefan Baumgartner)
- https://fettblog.eu/typescript-iterating-over-objects/




--------

## 解決方案 3: Generic Functions
來看個稍微奇怪的解決方案。在 `generic function` 中，使用 `in` 可以將 type 縮小到 key 的範圍
- (實際上，我也不知道為什麼這個方法有效，而 non-generic 方法無效)



```ts
function printEachKey<T extends object>(obj: T) {
  for (const key in obj) {
    console.log(obj[key]);
                    // const key: Extract<keyof T, string>
  }
}
 
// Each key gets printed!
printEachKey({
  name: "Daniel",
  age: 26,
});
```

--------

## 解決方案 4: 在 function 中封裝 `Object.keys`
將 `Object.keys` 封裝在一個 function 中，該 function 返回被轉換的 type



```ts
const objectKeys = <T extends object>(obj: T) => {
  return Object.keys(obj) as Array<keyof T>;
};
 
const keys = objectKeys({
  name: "Daniel",
  age: 26,
});
 
console.log(keys);
             // const keys: ("name" | "age")[]
```



這可能是最容易被誤用的解決方案
- 將轉換隱藏在 function 中，讓人覺得很方便，可能導致人們不加思考的用它


--------


## 結論
通常情況下，方案一(`as keyof typeof`)能很好地完成任務
- 它簡單易懂，通常也足夠安全
- 但如果你喜歡方案二(`type predicate`)或方案三(`generic`)的外觀，那就試試吧
  - (作者: `isKey` function 看起來非常有用，我會在下個項目中用它)




--------


## 取得 Object 裡，某些特定 type 的 value 的 keys
(這比較繞舌，算是個特定情境下的 user case，看看下面範例就會很清楚是什麼目的了)  


TypeScript 中，當 object 的某些 value 為特定 type 時，你想要取得這些 value 的 key
- 假設有個 object

```ts
type Obj = {
  a: string;
  b: string;
  c: number;
  d: number;
};
```



我們只想取 string 的 key
- 因此，我們要取 `"a" | "b"`
- 這很棘手




快速解決方案
- 使用 remapping 來只取 string keys

```ts
type StringKeys<T> = {
  [K in keyof T]: T[K] extends string ? K : never;
}[keyof T];
 
type StringKeysOfObj = StringKeys<Obj>;
           // type StringKeysOfObj = "a" | "b"
```


可以通過一個 generic condition 來重複使用它:

```ts
type KeysOfValue<T, TCondition> = {
  [K in keyof T]: T[K] extends TCondition
    ? K
    : never;
}[keyof T];
 
type StringKeysOfObj = KeysOfValue<Obj, string>;
           //type StringKeysOfObj = "a" | "b"
```
--------

## 解釋
來看看是如何找到這解決方案的，回顧一下我思考過程:
- 一開始，從普通 object 開始

```ts
type Obj = {
  a: string;
  b: string;
  c: number;
  d: number;
};
```



我們知道我們正在走向 union of keys，所以我們知道 `keyof` 會在某個地方出現:
```ts
type KeysOfObj = keyof Obj;
// 'a' | 'b' | 'c' | 'd'
```




你可能會想
- 現在只需要排除沒有字串值的 key，也許可以用 `Exclude` 

```ts
type StringKeys = Exclude<
  KeysOfObj,
  // What do we put here?
  ???
>;
```



當我們嘗試填 `Exclude` 第二個 type 參數時，問題就出來了
- 我們需要當前 key 的 context 來確定 `Obj[Key]` 是否為 string


因此
- 我們 type 的中間部分將是對每個 key 的 iterating


故事已經很清楚了，下面來實現它

--------

## Iterating over each key
有幾種不同的方法可以實現這一點
- 我最喜歡的是使用 [IIMT - an immediately indexed mapped type](https://www.totaltypescript.com/immediately-indexed-mapped-type)
  - (這是作者自創的名詞，不用特別記。基本上就是 Indexed access types 概念)

```ts
type KeysOfObj = {
        // type KeysOfObj = "hello_a" | "hello_b" | "hello_c" | "hello_d"

  [K in keyof Obj]: `hello_${K}`;
}[keyof Obj];

// 如果沒有最下面的 "[keyof Obj]"，KeysOfObj 的 type 會是
// {
//   a: "hello_a",
//   b: "hello_b",
//   c: "hello_c",
//   d: "hello_d"
// }
// 而 "[keyof Obj]" 就像是呼叫 Obj['a'] 一樣，最後出來的 tpye 為 "hello_a"
//                  (又或者可能是呼叫 'b', 'c', 'd')
```

這樣，就可以用 mapped type 建立 object
- 然後用 `keyof Obj` 對其進行索引，從而輸出所建立 object 的 value
- 這樣好處是，我們可以訪問同一 scope 中的當前 key 和 object，因此我們可以使用當前 key 來索引 object


上面例子中
- 我們用它建立了 string literal type，每個 value 的 prefix 都是 `hello_`
- 但如果我們想根據 value  的 type 做不同的事情，該怎麼辦呢？



--------

## Conditional types

可以用 conditional type 來實現
- conditional type 可以用條件來表達 `if/else` 邏輯:
 

```ts
type Tests = [
      // type Tests = [true, true, false, false]

  Obj["a"] extends string ? true : false,
  Obj["b"] extends string ? true : false,
  Obj["c"] extends string ? true : false,
  Obj["d"] extends string ? true : false
];
```



下面例子
- 我們檢查當前 key 的 value 是否為 string。如果不是，將返回 `never`，而不是 `false`
```ts
type KeysOfObj = {
        // type KeysOfObj = "a" | "b"

  [K in keyof Obj]: Obj[K] extends string
    ? K
    : never;
}[keyof Obj];
```



從技術上講，`KeysOfObj` 的 type 應該是 `'a' | 'b' | never | never`:

```ts
type Tests = [
      // type Tests = ["a", "b", never, never]

  Obj["a"] extends string ? "a" : never,
  Obj["b"] extends string ? "b" : never,
  Obj["c"] extends string ? "c" : never,
  Obj["d"] extends string ? "d" : never
];
```


但 TypeScript 會自動從 unions 中移除 `never`，所以我們最終拿到 `'a' | 'b'`:
```ts
type Example = "a" | "b" | never | never;
       // type Example = "a" | "b"
```

--------


## 實現 generic
最後一部分是把它變成一個可以用於任何 object 的 type helper
- 為此，我們可以為 `StringKeys` 加一個 type 參數，並刪除對 Obj 的 hard-code 引用:

```ts
type StringKeys<T> = {
  [K in keyof T]: T[K] extends string ? K : never;
}[keyof T];
 
type User = {
  firstName: string;
  lastName: string;
  age: number;
  numberOfCats: number;
};
 
type StringKeysOfObj = StringKeys<User>;
           // type StringKeysOfObj = "firstName" | "lastName"
```


也可以為 helper 加第二個 type 參數，以便指定條件:

```ts
type KeysOfValue<T, TCondition> = {
         // type KeysOfValue<T, TCondition> = { [K in keyof T]: T[K] extends TCondition ? K : never; }[keyof T]

  [K in keyof T]: T[K] extends TCondition
    ? K
    : never;
}[keyof T];
 
type StringKeysOfObj = KeysOfValue<User, string>;
           // type StringKeysOfObj = "firstName" | "lastName"
 
type NumericKeysOfObj = KeysOfValue<User, number>;
            // type NumericKeysOfObj = "age" | "numberOfCats"
```

這樣，type helper 就完成了
- 輸入一個 object ，通過 IIMT iterate object keys，然後用 conditional type 檢查當前 key 的 value 是否符合條件