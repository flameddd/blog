
# 2023-08-15：學習 Typescript，2023 Type 和 Interface 怎麼選？.md
## Matt Pocock
### Ref: https://www.totaltypescript.com/type-vs-interface-which-should-you-use


---------

看看高手談 Type 和 Interface 的使用時機

---------

## 快速解釋
默認情況下，應該用 `type`，直到你需要 `interface` 的特定功能，如 `extends`
- `interface` 不能表達 `unions`, `mapped` type 或 conditional types
  - `type aliases` 可以表達任何 type
- `interface` 可以使用 extends，而 `type` 不能。
- `extends` 可以讓 TypeScript 的 type check 執行時，比用 `&` 更快
- 在同一 scrope 中，相同名稱的 `interface` 會合併其 declarations，從而導致意外錯誤
- `Type aliases` have an implicit index signature of `Record<PropertyKey, unknown>`, which comes up occasionally.



---------


## 完整解釋
`interface` 從 TypeScript 的第一個版本開始就存在了
- `interface` 的靈感來源於 object-oriented programming ，它允許使用繼承來創建 type

```ts
interface WithId {
  id: string;
}
 
interface User extends WithId {
  name: string;
}
 
const user: User = {
  id: "123",
  name: "Karl",
  wrongProperty: 123,
  //Type '{ id: string; name: string; wrongProperty: number; }' is not assignable to type 'User'. Object literal may only specify known properties, and 'wrongProperty' does not exist in type 'User'.
};
```

不過，它有個內建替代方案
- `type aliases`: 使用 type 聲明
- 在 TypeScript 中，type 關鍵字可用於表示任何 type，而不僅僅是 object types


比方，我們想表示一個字串或數字 type
- 我們不能用 `interface` 來表示，但可以用 type 來表示:


```ts
type StringOrNumber = string | number;
 
const func = (arg: StringOrNumber) => {};
 
func("hello");
func(123);
 
func(true);
// Argument of type 'boolean' is not assignable to parameter of type 'StringOrNumber'.
```


當然， `type aliases` 也可以用來表達 objects
- 這引起了很多爭論
- 在宣告 object type 時，應該用 `interface` 還是 `type` ?


-----------------

## 用 `interface` 來實現 Object Inheritance
如果要處理相互繼承的 object
- 用 `interface`


```ts
type WithId = {
  id: string;
};
 
type User = WithId & {
  name: string;
};
 
const user: User = {
  id: "123",
  name: "Karl",
  wrongProperty: 123,
  // Type '{ id: string; name: string; wrongProperty: number; }' is not assignable to type 'User'. Object literal may only specify known properties, and 'wrongProperty' does not exist in type 'User'.
};
```

這是完全正確的 code
- 但稍顯不足的原因在於 TypeScript 檢查 type 的速度
- 當使用 `extends` 建立 `interface` 時，TypeScript 可以在 cache `interface` 的 name
- 這意味著將來對它的檢查可以更快
- 對於使用 `&` 的 intersection type，TypeScript 無法通過 name 來 cache，因此幾乎每次都要計算它


這只是很小的優化
- 但如果 `interface` 被多次使用，它就會增加
- 這就是為什麼 [TypeScript performance wiki](https://github.com/microsoft/TypeScript/wiki/Performance#preferring-interfaces-over-intersections) 推薦使用 `interface` 來實現 object inheritance



不過
- 還是不建議你默認使用 `interface`，為什麼？



---------------

### `interface` 會宣告合併
`interface` 還有個特點
- 當兩個同名的 `interface` 在同一 scope 中宣告時，它們的宣告會合併。


```ts
interface User {
  name: string;
}
 
interface User {
  id: string;
}
 
const user: User = {
  // Property 'name' is missing in type '{ id: string; }' but required in type 'User'.
  id: "123",
};
```



`Type` 就不能這樣做:
```ts
type User = {
// Duplicate identifier 'User'.
  name: string;
};
 
type User = {
// Duplicate identifier 'User'.
  id: string;
};
```

這是預期的行為，也是必要的語言特性
- 它用於規範修改 global objects 的 JavaScript libraries
  - 如為 `string` 的 prototypes 加入新的 method



但是，如果你沒有做好準備，它會導致非常令人困惑的錯誤
- 如果想避免，建議 ESLint 開啟 [no-redeclare](https://typescript-eslint.io/rules/no-redeclare/) rule


---------

## Index Signatures in Types vs Interfaces
Another difference between interfaces and types is a subtle one.

Type aliases have an implicit index signature, but interfaces don't. This means that they're assignable to types that have an index signature, but interfaces aren't. This can lead to errors like:



## `Types` 與 `interface` 中的索引簽名
`interface` 與 `types` 間還有一個微妙的區別
- `Type aliases` 具有隱式索引簽名，但 `interface` 沒有
- 這意味著它們可以賦值給有索引簽名的 type，但 `interface` 沒有

這可能導致以下錯誤
> Index signature for type 'string' is missing in type 'x'.

```ts
interface KnownAttributes {
  x: number;
  y: number;
}
 
const knownAttributes: KnownAttributes = {
  x: 1,
  y: 2,
};
 
type RecordType = Record<string, number>;
 
const oi: RecordType = knownAttributes;
// Type 'KnownAttributes' is not assignable to type 'RecordType'.
//   Index signature for type 'string' is missing in type 'KnownAttributes'.
```

The reason this errors is that an interface could later be extended. It might have a property added that doesn't match the key of `string` or the value of `number`.

出現這種錯誤的原因是，`interface` 以後可能會 extended
- 它可能會加一個與字串或數字不匹配的 type


可以在 `interface` 中加顯式索引簽名(explicit index signature)來解決這個問題：

```ts
interface KnownAttributes {
  x: number;
  y: number;
  [index: string]: unknown; // new!
}
```

或者乾脆用 type:
```ts
type KnownAttributes = {
  x: number;
  y: number;
};
 
const knownAttributes: KnownAttributes = {
  x: 1,
  y: 2,
};
 
type RecordType = Record<string, number>;
 
const oi: RecordType = knownAttributes;
```

這不是很奇怪嗎？



----------------

## Default to type, not interface
The TypeScript documentation has a [great guide](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces) on this. They cover each feature (although not the implicit index signature), but they reach a different conclusion than me.



## 默認使用 `type`，而非 `interface` 
TypeScript doc 對此有個[很詳細的說明文件](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces)
- 他們涵蓋了每種特性（雖然不包括隱式索引簽名），但得出的結論與我不同


TypeScript 建議你根據個人喜好進行選擇
- `Type` 和 `interface` 間的差別很小，使用任何一種都不會有太大問題
- 但 TS 團隊建議你默認使用 `interface`，只有在需要時才使用 `type`
- 我的建議恰恰相反。`declaration merging` 和隱式索引簽名(implicit index signatures)的特性足以讓人驚訝，它們應該會讓你不敢默認使用 `interface` 
- `interface` 仍然是我推薦的 object inheritance 方式，但我建議你默認使用 `type`。這樣會更靈活一些，也不會那麼令人驚訝
