
# 2023-08-14：學習 Typescript，React 與 TS 相關的幾個學習.md
## Matt Pocock


Ref:
- https://www.totaltypescript.com/react-component-props-type-helper
- https://www.totaltypescript.com/jsx-element-vs-react-reactnode
- https://www.totaltypescript.com/strongly-type-useref-with-elementref
- https://www.totaltypescript.com/react-refers-to-a-umd-global
- https://www.totaltypescript.com/what-is-jsx-intrinsicelements

---------

看到高手寫幾篇 Typescript 跟 React 相關的文章，學習看看  
文章標題分別為
- ComponentProps: React's Most Useful Type Helper
- React.ReactNode vs JSX.Element vs React.ReactElement
- Strongly Type useRef with ElementRef
- What is JSX.IntrinsicElements ?
- Explained: 'React' refers to a UMD global

很多可能或多或少我之前都知道了，但就是看看高手怎麼解釋 or 說明這些，說不定有些思維能學學

---------


## 最實用的 React type helper: `ComponentProps`

在 React 和 TypeScript 時，會經常發現有很多問題：
- 如何確定 component props 的 type ?
- 如何讓 `div` 或 `span` 接受的所有 type ?
- 這兩個問題都可以靠: `ComponentProps`


使用 `ComponentProps` type 三種最常見的方法:
1. Get the Props of Elements
2. Get the Props of a Component
3. Get the Props of an Element with the Associated Ref

---------

## 1. 取得 Elements 的 Props
需要用按 buttons, inputs, and forms 等 native HTML 時，用 `ComponentProps` 提取這些 elements 的 props，輕鬆進行 type check 並確保其正確性
- `ButtonProps` type 提取了 `button` element 的 props type
- 你可以傳入任何 DOM elements，`span` 到 `a`，甚至可以在 `ComponentProps` 本身中獲得 autocomplete 功能

```js
import { ComponentProps } from "react";
type ButtonProps = ComponentProps<"button">;
```

當想建立既能接受 `div` 的所有 props，又能加一些自己的 props 時，尤其有用:
 
```js
import { ComponentProps } from "react";

type MyDivProps = ComponentProps<"div"> & {
  myProp: string;
};

const MyDiv = ({ myProp, ...props }: MyDivProps) => {
  console.log(myProp!);
  return <div {...props} />;
};

```

---------

## 取得 Component 的 Props

`React.ComponentProps` 可以用它從現有 components 中取 props
- `SubmitButtonProps`  取了 `SubmitButton` 的 props
  - 這些 props 可用於在整個 app 中對 components 的進行 type check
 
```js
const SubmitButton = (props: { onClick: () => void }) => {
  return <button onClick={props.onClick}>Submit</button>;
};

type SubmitButtonProps = ComponentProps<
  typeof SubmitButton
>;
```


這對從無法控制的 components (third-party) 中提取 props 非常有用:
- `some-external-library` 可能沒有 export `ButtonProps` type，但你仍然可以使用 `ComponentProps` 取得

```js
import { ComponentProps } from "react";
import { Button } from "some-external-library";

type MyButtonProps = ComponentProps<typeof Button>;

```

-----------

## 通過相關聯的 Ref 取得 Element 的 Props
React 中的 Ref 可以訪問 element 的屬性並與之互動
- 通常用於 form element (如 input, button)

`ComponentPropsWithRef` 提供 components props 和其關聯的 `ref`
- `InputProps` 提取了 `input` element props，包括關聯的 ref

```js
type InputProps = ComponentPropsWithRef<"input">;
```


---------

## `React.ReactNode` vs `JSX.Element` vs `React.ReactElement`

快速解釋:
- `JSX.Element` 和 `React.ReactElement` 在功能上是相同的 type
  - 它們可以互換使用。它們代表 JSX expression 建立的東西

```js
const node: JSX.Element = <div />;
 
const node2: React.ReactElement = <div />;
```


它們(`JSX.Element`, `React.ReactElement`)不能用來表示 React 所有可呈現的內容
- 如 string 和 number 數字

```js
const node: React.ReactNode = <div />;
const node2: React.ReactNode = "hello world";
const node3: React.ReactNode = 123;
const node4: React.ReactNode = undefined;
const node5: React.ReactNode = null;
 
const node6: JSX.Element = "hello world";
// Type 'string' is not assignable to type 'Element'.
```



所以，就使用 `React.ReactNode` 吧
- 在日常使用中，應該使用 `React.ReactNode`
- 很少需要使用更特殊的 `JSX.Element` type



----------


## 完整解釋
當 TypeScript 團隊開始支持 React 時，JSX 是一大絆腳石
- JavaScript 不存在 JSX 語法，因此他們必須在 compiler 中建立它，才能支持 JSX
- 他們想到了 `.tsx` 文件、`tsconfig.json` 中的 `jsx` 選項，於是 JSX 突然得到了支持

但還有個有趣的問題沒有答案：這個 function 應該推斷出什麼 type ?

**JSX.Element:**  
```js
// hover 這裡的時候，我們應該要看到什麼？
const Component = () => {
  return <div>Hello world</div>;
};
```


答案是名為 `JSX.Element` 的特殊類型
- 如果你今天 hober 在一個 component 組件上，你很可能會看到下面:
```js
// const Component: () => JSX.Element
```

`JSX` 是個  global namespace
- 它就像 global scope 中的一個 object
- namespace 可以包含 type，而 `Element` 就是這些類型之一
- 這意味著，如果 React 的 type definitions 定義了 `JSX.Element`，它就會被 TypeScript 接收

下面是它在 React type definitions 中的樣子:


```js
// Puts it in the global scope
declare global {
  // Puts it in the JSX namespace
  namespace JSX {
    // Defines the Element interface
    interface Element extends React.ReactElement<any, any> {}
  }
}
```

無論它是如何定義的。它是在你寫 JSX 時建立的 type
- 我們可以將 `JSX.Element` 視為呼叫 JSX expression 所 return 的內容


**`JSX.Element` 有什麼用途？**
為什麼這些知識有用？你想用 `JSX.Element` type 做什麼？
- 最明顯的是 typing the `children` property of a component.
 

```js
const Component = (
  { children }: { children: JSX.Element; }
) => {
  return <div>{children}</div>;
};

```

當開始使用這種類型時，問題就開始變得明顯了
- 例如，如果你想呈現字串，該怎麼辦？
 

```js
// 'Component' components don't accept text as
// child elements. Text in JSX has the type
// 'string', but the expected type of 'children'
// is 'Element'.
<Component>hello world</Component>
```



上面這個是完全正確的用法
- React 可以將各種東西作為 component 的 children 來處理，如 `numbers`, `strings，甚至是` `undefined`
- 但 TypeScript 並不高興。我們將 `children` 設為 `JSX.Element`，它只接受 JSX

我們需要為 `children` 使用不同的 type definition
- 我們需要一種可接受字串、數字、`undefined` 和 JSX 的 type

**React.ReactNode:**  
這就是 `React.ReactNode` 的用武之地
- 它接受 React 可以呈現的所有內容的 type

它位於 React namespace 中:

```js
declare namespace React {
  type ReactNode =
    | ReactElement
    | string
    | number
    | ReactFragment
    | ReactPortal
    | boolean
    | null
    | undefined;
}
```


可以用它在  `children` prop 的 type:


```js
const Component = (
  { children }: { children: React.ReactNode; }
) => {
  return <div>{children}</div>;
};
```

現在可以輸入 strings, numbers, undefined, and JSX:


```js
<Component>hello world</Component>
<Component>{123}</Component>
<Component>{undefined}</Component>
<Component>
  <div>Hello world</div>
</Component>
```

**什麼情況下不應該使用 `React.ReactNode` ?**
在 TypeScript **5.1** 之前，有種特殊情況不能使用 `React.ReactNode`
- component 的 return type

```js
const Component = (): React.ReactNode => {
  return <div>Hello world</div>;
};
```


定義它時看起來沒問題，但當我們要使用它時，它就會抓狂:

> 'Component' cannot be used as a JSX component. Its return type 'ReactNode' is not a valid JSX element.

```js
// 'Component' cannot be used as a JSX component.
//  Its return type 'ReactNode' is not a valid JSX element.
<Component />
```


出現錯誤的原因是，TypeScript 用 `JSX.Element` 來檢查某些內容是否可以渲染為 JSX
- `React.ReactNode` 包含的內容不是 JSX，因此不能用作 JSX element


但是，TypeScript 5.1 以後，這功能可以完全正常工作
- TypeScript 改善從 React component 推斷 type 的方式
- https://devblogs.microsoft.com/typescript/announcing-typescript-5-1-beta/#decoupled-type-checking-between-jsx-elements-and-jsx-tag-types




**React.ReactElement:**  

接著談談 `React.ReactElement`，它是種 object type，定義如下:
```js
interface ReactElement<
  P,
  T extends string | JSXElementConstructor<any>
> {
  type: T;
  props: P;
  key: Key | null;
}
```

它以 object 的形式來表示正在 rendering 的 element
- 如果在 console.log 中輸出 JSX expression，就會看到類似下面的內容:


```js
// { type: 'div', props: { children: [] }, key: null }
console.log(<div />);
```


可以在任何地方用它來代替 `JSX.Element`
- `React.ReactElement` 跟 `JSX.Element` 幾乎等同於別名

事實上，許多 component 都是這樣註釋的:

```js
const Component = (): React.ReactElement => {
  return <div>Hello world</div>;
};
```



但是，就像 `JSX.Element`，當試圖將字串、數字為 child 傳入時，你會得到錯誤:
```
Type 'string' is not assignable to type 'ReactElement<any, string | JSXElementConstructor<any>>'.
```

```js
const Component = (): React.ReactElement => {
  // Type 'string' is not assignable to type
  // 'ReactElement<any, string | JSXElementConstructor<any>>'.
  return "123";
};
```

因此，`React.ReactElement` 就像是 `JSX.Element` 的別名
- 同樣的規則適用，你不應該用它


----------

## 結論
你幾乎不應該在 code 中使用 `JSX.Element` 或 `React.ReactElement`
- 它們是 TypeScript 內部使用的 tpye，用於表示  JSX expressions 的 return type
- 使用 `React.ReactNode` 來表示 component 的 children type


------------



## 用 `ElementRef` 來 strong type `useRef`

使用 `useRef` 的時後，並不總是很清楚應該使用什麼類型
```js
import { useRef } from "react";
 
const Component = () => {
  // What goes here?
  const audioRef = useRef<NoIdeaWhatGoesHere>(null);
 
  return <audio ref={audioRef}>Hello</audio>;
};
```


簡單的解決辦法是 hober 在 `ref` 類型上，看它接受什麼:
```js
import { useRef } from "react";
 
const Component = () => {
  // What goes here?
  const audioRef = useRef<HTMLAudioElement>(null);
 
  return <audio ref={audioRef}>Hello</audio>;
                
  // ref ----> (property) React.ClassAttributes<HTMLAudioElement>.ref?: React.LegacyRef<HTMLAudioElement> | undefined
};
```


但還有更簡單的方法

------------

## 什麼是 `ElementRef` ?

`ElementRef` 是 React 的  type helper
- 可以輕鬆取 target element 的 type

 
```js
import { useRef, ElementRef } from "react";
 
const Component = () => {
  const audioRef = useRef<ElementRef<"audio">>(null);
 
  return <audio ref={audioRef}>Hello</audio>;
};
```

這甚至可以用在使用 `forwardRef` 的 component
- 你可以使用 `typeof` 將它們傳給 `ElementRef`，它就會取出 component 轉發到的 element 的 type
 

```js
import { OtherComponent } from "./other-component";
import React, { useRef, ElementRef } from "react";
 
// Pass it in via typeof!
type OtherComponentRef = ElementRef<typeof OtherComponent>;
 
const Component = () => {
  const ref = useRef<OtherComponentRef>(null);
 
  return <OtherComponent ref={ref}>Hello</OtherComponent>;
};
```

如果你之前用的是 `HTMLAudioElement` 或 `HTMLDivElement` 等，就沒有必要更改
- 但如果不確定使用哪種類型，`ElementRef` 是個很好的幫手

-----------


## 什麼是 `JSX.IntrinsicElements` ?

`JSX.IntrinsicElements` 是 TypeScript global scope 中的一種類型
- 它定義了哪些 native JSX element 在 scope 中，以及它們需要哪些 props

## 使用方法 

```js
declare global {
  namespace JSX {
    interface IntrinsicElements {
      "my-custom-element": {
        id: string;
      };
    }
  }
}
 
<my-custom-element />;
// Property 'id' is missing in type '{}' but required in type '{ id: string; }'.
```

上面的範例:
1. 使用 `declare global` 宣告一個 global type
2. 使用 `JSX` namespace
3. 建立 `IntrinsicElements` interface
4. 在 interface 上定義一個屬性，該屬性包含我們要定義的 element name
    - 我們在 `IntrinsicElements` interface 上定義的 property 就是我們要定義的 element name
    - 該屬性的值就是該元素所接受的 props

這樣做是因為同一 scrope 中的 interface 會合併它們的 declarations
- 這意味著我們定義的 `IntrinsicElements` interface 將與 TypeScript 已經知道的 interface 合併


如果嘗試用未在 `JSX.IntrinsicElements` 中定義的 element，則會出現錯誤：

> Property 'X' does not exist on type 'JSX.IntrinsicElements'.

```js
const Component = () => {
  return <i-am-not-defined />;
  // Property 'i-am-not-defined' does not exist on type 'JSX.IntrinsicElements'.
};
```

-----------

## When will I use this?
Mostly, you won't modify `JSX.InstrinsicElements` yourself. Instead, the framework you're using will define the elements for you.




### 什麼時候會用到 `JSX.InstrinsicElements` ?
大多情況下，你不會自己修改 `JSX.InstrinsicElements`
- 相反，你使用的 framework 會為你定義 element
- 例如，React 使用 `JSX.IntrinsicElements` 為 native HTML element 定義 props
- 在 VSCode 中，在一個 element 上 `cmd-click`，就會跳到該 element 的定義，而定義的內容非常豐富
 

```js
declare global {
  namespace JSX {
    interface IntrinsicElements {
      a: React.DetailedHTMLProps<
        React.AnchorHTMLAttributes<HTMLAnchorElement>,
        HTMLAnchorElement
      >;
      abbr: React.DetailedHTMLProps<
        React.HTMLAttributes<HTMLElement>,
        HTMLElement
      >;
      address: React.DetailedHTMLProps<
        React.HTMLAttributes<HTMLElement>,
        HTMLElement
      >;
      // ... hundreds more
    }
  }
}
```

從根本上說，這與我們上面的 code 做了完全相同的事情
- 屬性的名稱是 element name ，值是 element 接受的 props
- `React.DetailedHTMLProps` 只是個 utility type，它建立了 `a`, `abbr` 和 `address` 所接受的 type

--------------

## 解釋: React refers 到 UMD global

如果你用 TypeScript 和 React，你可能會遇到這個錯誤:

```js
const Component = () => {
  // React refers to a UMD global, but the current file is a module. Consider adding an import instead.
  return <div>Hello world</div>;
};
```


這個錯誤是因為你在 `tsconfig.json` 中配了 `jsx` 屬性
- 根據你正在做的事情，你需要將其更改為 `preserve` 或 `react-jsx`

----------

## Quick Fix
如果你**不使用 TypeScript 來編譯**你的 app
- 你應該將 `tsconfig.json` `jsx` 改為 `preserve`
- 大多數現代 framework 都不會使用 TypeScript 來編譯。所以如果你不確定，請選擇這個選項
  - (還應該重啟 TS server)
 

```json
{
  "compilerOptions": {
    "jsx": "preserve"
  }
}
```

如果**你使用 TypeScript 編譯**你的 app
- 應該檢查一下你是否正在使用 React 17 或更高版本？
- 如果是，則 `tsconfig.json` `jsx` 改為 `react-jsx`



```json
{
  "compilerOptions": {
    "jsx": "react-jsx"
  }
}
```

如果不是，則 `tsconfig.json` `jsx` 保留為 `react`
- 並在檔案最上面加 `import React from "react";`


----------

### 解釋
當在 `tsconfig.json` 的 `compilerOptions` 中將 `jsx` 設為 `react` 時，錯誤就會發生
- 原因是，當這樣設時，它會假定它會將 `JSX` 轉換為 `React.createElement` 來呼叫

因此，下面的 code 

```js
const Component = () => {
  return <div>Hello world</div>;
};
```

轉為

```js
const Component = () => {
  return React.createElement("div", null, "Hello world");
};
```
 

錯誤就在這裡。這個 module 的 scope  中沒有 `React`，因此導致執行時錯誤
- 但自 React 17 以來，就不需要 import React 來使用 JSX 了
- 因此，這個錯誤與 React 如今的工作方式格格不入


------------

## Stopping the Error
這個錯誤說明你可能錯誤配了 `tsconfig.json`
- 大多 React framework 都不使用 TypeScript 來處理這種轉換
  - 它們使用 swc 這樣的工具，運行速度更快，但不進行類型檢查
    - (這在指 NextJS)
- TypeScript 在此處拋出錯誤，因為它假定你正在使用它來轉換 JSX，但你可能不是
- 最安全的辦法是 `tsconfig.json`  `jsx` 改為 p`reserve`
  - 這樣，TypeScript 就不會打擾你的 JSX，也不會在你忘記 import React 時出錯



--------------

## The Exception to the Rule
只有在使用 React 16 或更早版本的情況下，這個錯誤才會有用
- 在這種情況下，你需要 import React 才能使用 JSX
- 因此，應該在 `tsconfig.json` `jsx` 保留為 `react`，並在檔案最上方加 `import React from "react";`
- `UMD Global` 只是個幌子，並不是真正的問題所在。真正的問題是在使用 JSX 時沒有 import React