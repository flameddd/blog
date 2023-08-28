
# 2023-08-22：學習 TypeScript satisfies.md
## Matt Pocock, Mar 14, 2023

### Ref: https://www.totaltypescript.com/clarifying-the-satisfies-operator


--------

`satisfies` 出來一陣子了，還不清楚確切的使用場景，讀讀這篇學習一下。


--------


將 satisfies 視為另種 assign types to values 的方法
- 在研究之前，我們先回顧如何 assign types




首先是「冒號(colon annotation)」，冒號表示，「這個變數一直都是這種 type」:
```ts
const obj: Record<string, string> = {};

obj.id = "123";
```

當使用冒號時，就是在宣告該變數是這種 type
- 這意味著你賦值給變數的東西必須是這種 type:

```js
// Type 'number' is not assignable to type 'string'.
const str: string = 123;
```


你可以給一個變數賦予比你最初分配的 type 更寬的 type

```ts
let id: string | number = "123";

if (typeof numericId !== "undefined") {
  id = numericId;
}
```
T

當你想要個 default value，而這個 default 以後可能會被重新賦值時，這就很有用了
- 但 colon 也有其缺點


**當使用 colon 時，type 會與 value 相衝突**
- 換句話說，如果宣告了一個比你想要的更寬的 type，你就只能使用這個更寬的 type

例如，在這，routes object 無法 autocomplete:

```ts
const routes: Record<string, {}> = {
  "/": {},
  "/users": {},
  "/admin/users": {},
};

// No error!
routes.awdkjanwdkjn;
```


這就是 `satisfies` 要解決的問題
-  **使用 `satisfies` 時，value 會擊敗 type**
- 這意味著它推斷的是可能的最窄 type，而不是你指定的更寬 type

```ts
const routes = {
  "/": {},
  "/users": {},
  "/admin/users": {},
} satisfies Record<string, {}>;

// Property 'awdkjanwdkjn' does not exist on type
// '{ "/": {}; "/users": {}; "/admin/users": {}; }'
routes.awdkjanwdkjn;
```


`satisfies` 也能防止你在 config object 中指定錯誤的內容
- 因此，colon 和 `satisfies` 同樣安全

```ts
const routes = {
  // Type 'null' is not assignable to type '{}'
  "/": null,
} satisfies Record<string, {}>;
```



為變數指定 type 的另一種方法是 `as`
- 與 `satisfies` 和 `colon` 不同，使用 `as` **可以讓你對 TypeScript 撒謊**

在這邊中，你不會在 IDE 中看到錯誤，但執行時會出錯:
```ts
type User = {
  id: string;
  name: {
    first: string;
    last: string;
  };
};

const user = {} as User;

// No error! But this will break at runtime
user.name.first;

```


謊言有些限制
- 你可以為 object 新增屬性，但不能在基本類型間轉換




例如，無法強制 TypeScript 將 string 轉換為 number
- (除非你使用畸形的 as-as)



```ts
// Conversion of type 'string' to type 'number'
// may be a mistake because neither type
// sufficiently overlaps with the other.
const str = "my-string" as number;
const str2 = "my-string" as unknown as number;
```


下面是 `as` 的合法用法，用於將 object 轉換為尚未構建的已知類型:



```ts
type User = {
  id: string;
  name: string;
};

// The user hasn't been constructed yet, so we need
// to use 'as' here
const userToBeBuilt = {} as User;

(["name", "id"] as const).forEach((key) => {
  // Assigning to a dynamic key!
  userToBeBuilt[key] = "default";
});
```
A word of caution: if you’re using `as` as your default way to annotate variables, that’s almost certainly wrong!

The code below might look safe, but as soon as you add another property to the `User` type, the `defaultUser` will be out of date, and it will not show you an error!



需要注意的是
- 如果使用 `as` 作為註解變數的默認方式，那幾乎肯定是錯誤的！

下面的 code 可能看起來很安全，但只要你在 User type 中加上另一個屬性，`defaultUser` 就會過時，**而且不會顯示錯誤**
```ts
type User = {
  id: string;
  name: string;
};

const defaultUser = {
  id: "123",
  name: "Matt",
} as User;
```


還有最後一種為變數賦予類型的方法: 「不要 assign type 給變數」
- TypeScript 在為變數推斷 type 方面做得非常出色
-  大多數情況下，你根本不需要為變數指定 type


```ts
const routes = {
  "/": {},
  "/users": {},
  "/admin/users": {},
};

// OK!
routes["/"];

// Property 'awdahwdbjhbawd' does not exist on type
// { "/": {}; "/users": {}; "/admin/users": {}; }
routes["awdahwdbjhbawd"];
```


簡而言之，我們有四種為變數指定 type 的方法:
- 冒號註釋
- `satisfies`
- `as` 註釋
- 不註釋，讓 TS 推斷





有了不同的方法來做類似的事情，什麼時候使用什麼方法可能會讓人有點困惑
- 簡單的 case 中使用 `satisfies` 效果最好:




```ts
type User = {
  id: string;
  name: string;
};

const defaultUser = {
  id: "123",
  name: "Matt",
} satisfies User;
```




但在大多數情況下，當你想為變數指定類型時，你可能希望類型更寬泛
- 如果這個例子使用 `satisfies`，你就無法將 `numericId` 賦值給 `id`



```ts
// colon annotation

let id: string | number = "123";

if (typeof numericId !== "undefined") {
  id = numericId;
}

// satisfies

let id = "123" satisfies string | number;

if (typeof numericId !== "undefined") {
  // Type 'number' is not assignable to type 'string'.
  id = numericId;
}
```


經驗法則是，只有在兩種特定情況下才能使用 `satisfies`:
- 你需要的是變數的精確 type，而不是更寬的 type
- 變數 type 足夠複雜，需要確保不會弄錯