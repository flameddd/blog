
# 2023-08-18：學習 TypeScript Generics (泛型).md
## Matt Pocock, (Jan 09, 2023), (Jul 19, 2023), (Jul 20, 2023)

Ref:
- https://www.totaltypescript.com/mental-model-for-typescript-generics
- https://www.totaltypescript.com/typescript-generics-in-three-easy-patterns
- https://www.totaltypescript.com/no-such-thing-as-a-generic

---------

對 TS 的 Generics 一直感覺沒能抓到要點、一知半解的。看到 Matt 寫了三篇文章，來好好學學看  
三篇的主題為
1. Building the Mental Model for Generics
2. TypeScript Generics in 3 Easy Patterns
3. There Is No Such Thing As A Generic


---------

## Type Helpers
首先從 type 層面來看 `generics`

想像有個 type:
```ts
type Name = 'matt'
```


這是個字串 `matt`，用 `type alias` 表示
- 這個以後不能更改，因此它的行為類似於 JavaScript 中的 `const`

```js
const name = 'matt' 
```

使用 `generics`，我們可以將 `Name` 變為一個 function
- 為 `Name` 加個 `type argument`


```ts
type Name<LastName> = 'matt' 
```


這將 `Name` 加上了一個**參數**
- 也就是說，無論何時使用 `Name`，都需要傳一個**參數**


```ts
type User = { name: Name<'pocock'> } 
```


這就把 `type` 從一個 variable 變成了 function
- 現在，它在 JavaScript 中的等價式是


```js
const name = (lastName) => 'matt' 
```

既然它現在是個 function，那就給它取個更像 function 的名稱吧：
```ts
type GetName<LastName> = 'matt' 
```

`LastName` 目前尚未使用
- 因此作為範例，我們將從 function type 中 return 一個 object


```ts
type GetNameObject<LastName> = {
  firstName: 'matt'
  lastName: LastName
}
```


JavaScript 的等價 function 是
```js
const getNameObject = (lastName) => {
  return {
    firstName: 'matt',
    lastName,
  }
}
```

然後，你可以使用 `GetNameObject` (`type function`)
- 通過傳**參數**來建立 type`


```ts
type NameObject = GetNameObject<'pocock'>
// {
//   firstName: "matt";
//   lastName: "pocock";
// }
```

這種方法，這邊稱為 `type helpers`
- 一個常見的例子就是 `Maybe` 的 `type helper` 

```ts
type Maybe<T> = T | null | undefined
type MaybeString = Maybe<string>
// string | null | undefined
```

------------

##  Generic functions
現在你知道如何在 type level 上使用 `generics`，那麼 `generic functions` 呢？
- 舉簡單的例子 `returnWhatIPassIn`:


```js
const returnWhatIPassIn = (input: any) => {
  return input
}
```


這 function 不會 return 傳入的任何內容
- 在  type level 上，它總是 return `any` type
- 這很煩，因為它破壞了你對傳入的 **autocomplete**

```js
const str = returnWhatIPassIn('matt')

str.touppercase() // No error here!
```


試想，如果你想在 type level 上建立這個 function
- 你可以建立一個名為 `ReturnWhatIPassIn` 的 type helper



```ts
type ReturnWhatIPassIn<TInput> = TInput

type Str = ReturnWhatIPassIn<'matt'>
//   "matt"
```


在 type level，你將接收 `TInput` (無論它是什麼)並返回它
- 你可以使用非常類似的語法來註解 function


```ts
const returnWhatIPassIn = <TInput>(input: TInput): TInput => {
  return input
}
```


現在這是一個 `generic function`
- 這意味著當呼叫它時，`TInput` 的 `type` 參數會從傳入的內容中推測出來

```js
const str = returnWhatIPassIn('matt')
// "matt"
```


簡單來說就是:
- **`generic function` 是個疊在「一般 function」上的 type helper**
- 這意味著當呼叫 function 時，你也在向 type helper 傳遞一個 type


從 JavaScript function 開始，再次逐段構建該 function，使其更加清晰:
```ts
const returnWhatIPassIn = (input: any) => {
  return input
}
```


首先建立 `type helper` 的結構
- 你需要推測出一個 `TInput` 參數，並返回該參數


```ts
const returnWhatIPassIn = <TInput>(input: any): TInput => {
  return input
}
```



如果現在嘗試呼叫 function，它會將 return 的內容推測為 `unknown`
```js
const str = returnWhatIPassIn('matt')
// unknown
```


你還沒有告訴 `type helper` 要推測哪些參數
- 你需要它從輸入參數中推測出 `TInput` 的 type

像這樣解決這個問題:
```ts
const returnWhatIPassIn = <TInput>(input: TInput): TInput => {
  return input
}
```

因此，在 `(input: TInput)` 中
- 「執行時的 `input` 參數」和希望推測「`TInput` 的 type」之間有了 mapping


這就是思考 `generics` 的方法
- 將其作為一個 type helper，鋪設在 function 之上，**並在它們之間建立映射關係**


---------

## TypeScript Generics in 3 Easy Patterns

What you might think of as generics in TypeScript is actually three separate concepts:


## 3 種 Generics 的模式

你可能認為 TypeScript 中的 `generics` 實際上是三個獨立的概念:
- Passing types to types
- Passing types to functions
- Inferring types from arguments passed to functions


```ts
// 1. Passing types to types
type PartialUser = Partial<{ id: string; name: string }>;

// 2. Passing types to functions
const stringSet = new Set<string>();

// 3. Inferring types from arguments passed to functions
const objKeys = <T extends object>(obj: T): Array<keyof T> => {
  return Object.keys(obj) as Array<keyof T>;
};

const keys = objKeys({ a: 1, b: 2 });
//    ^? const keys: ("a" | "b")[]
```


先從 `Passing types to types` 開始
- 在 TypeScript，你可以宣告一個代表 object, primitive, function 的 type
- 你可以隨心所欲地宣告任何類型

 

```ts
export type User = {
  id: string;
  name: string;
};

export type ToString = (input: any) => string;
```


但假設你需要建立幾個結構類似的 type。如，資料結構
- 下面的 code 蠻亂的，我們整理看看
 

```ts
type ErrorShape = {
  message: string;
  code: number;
};

type GetUserData = {
  data: {
    id: string;
    name: string;
  };
  error?: ErrorShape;
};

type GetPostsData = {
  data: {
    id: string;
    title: string;
  };
  error?: ErrorShape;
};

type GetCommentsData = {
  data: {
    id: string;
    content: string;
  };
  error?: ErrorShape;
};
```


如果你喜歡 OOP，可以使用 `interface` 來實現:
 

```ts
interface DataBaseInterface {
  error?: ErrorShape;
}

interface GetUserData extends DataBaseInterface {
  data: {
    id: string;
    name: string;
  };
}

interface GetPostsData extends DataBaseInterface {
  data: {
    id: string;
    title: string;
  };
}

interface GetCommentsData extends DataBaseInterface {
  data: {
    id: string;
    content: string;
  };
}
```

但是，`type function` 會更簡潔
- 它接收 data type 並返回新的 data shape:
 

```ts
// Our new type function!
type DataShape<TData> = {
  data: TData;
  error?: ErrorShape;
};

type GetUserData = DataShape<{
  id: string;
  name: string;
}>;

type GetPostsData = DataShape<{
  id: string;
  title: string;
}>;

type GetCommentsData = DataShape<{
  id: string;
  content: string;
}>;
```

理解這種語法很重要
- `<TData>` 的 `<>` 告訴 TypeScript，我們要為這個 type 加一個 `type argument`
- `TData` 可以為任何名字，它就像 function 的參數

這是個 `generic type`:
```ts
// Generic type
type DataShape<TData> = {
  data: TData;
  error?: ErrorShape;
};

// Passing our generic type another type
type GetPostsData = DataShape<{
  id: string;
  title: string;
}>;
```

`Generic types` 可以接受多個 type 參數
- 可以為 `type 參數`提供默認值
- 也可以設置限制條件，這樣就只能傳遞特定 type 的參數



不過，可以傳給 `type 參數`的不僅僅只有 tpye:

```ts
const createSet = <T>() => {
  return new Set<T>();
};

const stringSet = createSet<string>();
// ^? const stringSet: Set<string>

const numberSet = createSet<number>();
// ^? const numberSet: Set<number>
```


就像 type 可以接受 type 作為參數一樣，function 也可以


下面的例子
- 宣告 `createSet` 時加了 `<T>`
- 然後，我們將 `<T>` 傳給 `Set()`
- 而 `Set()` 本身允許傳一個 `type argument`


```ts
export const createSet = <T>() => {
  return new Set<T>();
}
```


下面這意味著，當呼叫它時，可以向 `createSet` 傳一個 `<string>` type 的參數
- 現在，我們得到了一個只能傳遞字串的 `Set`
```ts
const stringSet = createSet<string>();
// Argument of type 'number' is not assignable to parameter of type 'string'.
stringSet.add(123);
```



這就是: 手動將 type 傳遞給 function
- 如果你使用 React，並且需要向 `useState` 傳 type 參數，你就會看到這一點

```ts
const [index, setIndex] = useState<number>(0);
       // ^? const index: number
```


但是，你也會注意到 React 中，即在某些情況下，不需要傳 type 參數就能推測出 tyoe:

```ts
const [index, setIndex] = useState(0);
// ^? const index: number

// WHAT IS THIS SORCERY
```

為了了解這裡發生了什麼，再次看看 `createSet` function
- 注意，它不接受實際參數，只接受 type 參數

```ts
export const createSet = <T>() => {
  return new Set<T>();
}
```

如果我們改變 function，讓它接受 Set 的初始值，會發生什麼呢？
- 我們知道，初始值必須與 Set 的 type 相同，所以我們把它設為 `T`

```ts
export const createSet = <T>(initial: T) => {
  return new Set<T>([initial]);
}
```


現在，當呼叫它時，我們需要傳入一個初始值
- 這個初始值需要與我們傳給 `createSet` 的 type 類型相同

```ts
const stringSet = createSet<string>("matt");
//    ^? const stringSet: Set<string>

// Argument of type 'string' is not assignable to parameter of type 'number'.
const numberSet = createSet<number>("pocock");
```


TypeScript 可以從 initial 中推測出 T 的 type
- 換句話說，type 參數將從執行時 type 中推測出來

```ts
const stringSet = createSet("matt");
//    ^? const stringSet: Set<string>

const numberSet = createSet(123);
//    ^? const numberSet: Set<number>
```


這個 objKeys function 也是如此，其中還包括一些額外的好處:
- 我們將 T 限定為 object，這樣它(`T`)就可以傳給 `Object.keys` (只接受 object)
- 我們強制 `Object.keys` 的返回 type 為 `Array<keyof T>`


```ts
const objKeys = <T extends object>(obj: T) => {
  return Object.keys(obj) as Array<keyof T>;
};
```


總而言之，你所認為的 `generics` 實際上是三種不同的模式:
- 將 type 傳遞給 type: `DataShape<T>`
- 將 type 傳遞給 function: `createSet<string>()`
- 從傳給 function 的參數中，推測 type: `useState(0)`


-----------

## There Is No Such Thing As A Generic

如果你不理解 TypeScript 中的 `generics`，你一定是誤解了什麼
- 並不存在所謂的 `generics`
- 這裡有 **generic types**, **generic functions** 和 **generic classes**
- 這裡有 **type arguments** 和 **type parameters**.

你不能傳遞一個 `generic`，也不能宣告或推測它
- 換句話說，`generic` 不是名詞，而是形容詞




人們認為 `generic` 是 TypeScript 中的某樣東西
- 看看下面的 code，你可能會說「我們向 `useState` 傳了一個 `generic`」
```ts
import { useState } from "react";
 
useState<string>();
```


你也可以說「我們向 `Record` 傳遞了兩個 `generic`」
```ts
type NumberRecord = Record<string, number>;
```


那 `Maybe` 怎樣能有兩個 `generics` 呢?


```ts
type Maybe<T> = T | null | undefined;
```


大家看到 `<>` 語法就會認為這是個 `generic`
- 但因為 `generic` 可以出現在 functions, function calls, types, and type declarations 中
- 所以 `generic` 到底是什麼並不清楚

這也是為什麼人們很難理解 `generic` 這個概念的原因，這個詞太過繁瑣
- 那應該用什麼術語來代替它呢？
----------

## Type Arguments
如果不能使用 "通用(`generic`)" 一詞，我們該如何描述這段 code 呢？


```ts
import { useState } from "react";
 
useState<string>();

```



我們並沒有傳 `generic` 給 `useState`
- 我們傳給它的是一個 `type argument`。傳的 `type argument` 是 `string`

那 `Record` 呢？
```ts
type NumberRecord = Record<string, number>;
```


我們傳了兩個 `type arguments` 給 Record
- 第一個 `type arguments` 是 `string`
- 第二個 `type arguments` 是 `number`


因此，`type argument` 的作用就像 function 參數一樣
- 我們可以將它傳遞給 function, class 或 type



但並非所有 type, function, class 都能接收 `type arguments`
- 那我們如何知道哪些可以接收 `type arguments` 呢?

```ts
type Example = PropertyKey<string>;
//Type 'PropertyKey' is not generic.
 
encodeURIComponent<string>();
//Expected 0 type arguments, but got 1.
 
new Event<string>();
//Expected 0 type arguments, but got 1.
```

----------

## Type Parameters
我們看看前面的 `Maybe<T>`

```ts
type Maybe<T> = T | null | undefined;
```




Maybe 宣告了一個 `type parameter` `T`
- 意味著必須給 `Maybe` 傳遞一個 `type argument`。如果不傳遞 `type argument`，就會出錯

```ts
type Example = Maybe;
// Generic type 'Maybe' requires 1 type argument(s).
```

因此，`type parameter` 就像 function 參數
- 它宣告你可以傳給 type, function, class 的 `type argument`

----------

## Generic Types
Let's bring the phrase 'generic' back into our vocabulary and give it a proper definition.


## 通用類型
讓我們把 "泛型 "這個短語帶回我們的詞彙表，並給它一個正確的定義。

> `generic`: 形容詞、宣告了一個或多個 `type parameters` 的 type, function, or class

所以，`Maybe` 是個 `generic type`，因為它宣告了一個 `type parameter`
- 而之前看到的 `Type 'PropertyKey' is not generic`


```ts
type Example = PropertyKey<string>;
// Type 'PropertyKey' is not generic.
```

因此，`generic types` 只是宣告了 `type parameters` 的 type



------------

## `Generic Functions` and `Generic Classes`

Function 和 Class 也可以宣告 `type parameters`
- 這樣，它們就成為 `generic functions` 和 `generic classes`，並可以接收 `type arguments`



```ts
const myFunc = <T>() => {
  // implementation...
};
 
myFunc<string>();
 
class MyClass<T> {
  // implementation...
}
 
new MyClass<string>();
```

這可以有各種各樣的用途
- 最常用的是向第三方 library 提供 type 資訊

在下面的範例中，`useState` 如何知道它應該返回什麼 type ? 
```js
import { useState } from "react";
 
const [message, setMessage] = useState();
    // const message: undefined


// It can't - so we have to pass a `type argument` to it.
const [message2, setMessage2] = useState<string>();
        // const message: string | undefined
```


----------

## 推測 `Type Arguments`
你會注意到，`generic functions` 和 `generic classes` 的行為與 type 不同
- 它們不需要傳 `type argument`


```js
// No error!
myFunc();
 
new MyClass();
```

但是 `generic types` 卻要求你傳一個 `type argument`

```ts
type Example = Maybe;
// Generic type 'Maybe' requires 1 type argument(s).
```


這是為什麼呢？
- 因為，如果不向 `generic function` 或 `generic class` 傳 `type argument`，它就會嘗試從執行參數中推測
- 這就是 `useState` 的工作原理。它的(簡化)宣告看起來像這樣
  - ([https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts#L940](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts#L940))

```ts
declare function useState<T>(
  initial?: T
): [T, (newValue: T) => void];
```



你可以看到
- 它接受 `T` 作為 `type parameter`，並返回一個包含 `T` 和更新 `T` 的 function 的 array
- 如果我們傳執行時參數，TypeScript 可以從中推測出 `type argument`

```ts
// T is inferred as string!
const [message, setMessage] = useState("Hello!");
      // const message: string
 
// T is inferred as number!
const [id, setId] = useState(1);
      // const id: number
```
