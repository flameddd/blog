
# 2023-08-22：學習 TypeScript，TypeScript Discriminated Unions for Frontend Developers.md
## Matt Pocock, Mar 29, 2023
### Ref: https://www.totaltypescript.com/discriminated-unions-are-a-devs-best-friend 


--------

這篇不知道該怎麼下標題，總之就是 Matt 談一個「可區分的 unions」用法，來處理 call API 的情境


frontend 大部分的複雜性來自於處理 application 可能處於的各種狀態
- 它可能正在 loading date、等待填寫 form 或 send event，或者同時處於這三種狀態
- 如果沒有正確處理狀態，可能會陷入困境
- 處理狀態首先要看如何在 type 中表示它們


--------


## 一袋可選選的項目
假設你正在構建簡單的  data loader
- 你可能會選擇使用這樣的 type 來表示它的 `state`:

```ts
interface State {
  status: "loading" | "error" | "success";
  error?: Error;
  data?: { id: string };
}

// Some examples:
const example: State = {
    status: "loading"
};

const example2: State = {
    status: "error",
    error: new Error("Oh no!")
};
```

這看起來很不錯
- 我們可以通過檢查 `status` 來了解我們應該在畫面上顯示什麼



下面進一步討論情境:
- 在這裡，我們處於 `success` state。應該可以訪問資料。但它並不存在！
```ts
const example3: State = {
  status: "success",
  // Where's the data?!
};
```






這裡，處於 `loading` state，但 object 中仍然存在 error
```ts
const example4: State = {
  status: "loading",
  // We're loading, but we still have an error?!
  error: new Error("Eek!"),
};
```


這是因為我們選擇了使用我所說的 bag of optionals 來表示我們的 state
- 一個充滿 optional properties 的 object



當一個值可能存在也可能不存在時，最好使用 optional 屬性。在上面例中，是不正確的
- 當狀態為 `loading` 時，data 或 error 永遠不會出現
- 狀態為 `success` 時，data 始終存在
- 狀態為 `error`時，error 始終存在



------------------



## 有區分的聯集(Discriminated unions)
更準確的方法是用 discriminated union
- 我們先將 state 改為 object union，每個 object 都包含一個 status

```ts
type State =
  | {
      status: "loading";
    }
  | {
      status: "success";
    }
  | {
      status: "error";
    };
```


現在可以開始向 union 的每個分支加元素了
- 重新加 error 和 data type



```ts
type State =
  | {
      status: "loading";
    }
  | {
      status: "success";
      data: {
        id: string;
      };
    }
  | {
      status: "error";
      error: Error;
    };
```


現在，上面的範例將開始出錯了
- (我們還沒有完成)

```ts
// Error: Property 'data' is missing
const example3: State = {
  status: "success",
};

const example4: State = {
  status: "loading",
  // Error: Object literal may only specify known
  // properties, and 'error' does not exist
  error: new Error("Eek!"),
};
```

---------------

## 重構有區分的聯集(Discriminated unions)
想像一下，我們正在 code 中的一個 component 中
- 我們收到了狀態片段，並希望用它來 render染一些 JSX


```ts
const Component = () => {
  const [state, setState] = useState<State>({
    status: "loading",
  });
  const {
    status,
    // Error: Property 'data' does not exist on type 'State'.
    data,
    // Error: Property 'error' does not exist on type 'State'.
    error,
  } = state;
};
```



這是個棘手的問題
- `data`和 `error` 都可以存在 `State` 中，這邊為什麼會出錯？



原因是還沒有嘗試 discriminate the union
- 我們不知道自己處於哪種 `state`，因此唯一可用的屬性是所有成員共享的屬性。也就是 `status`



一旦檢查了所處的分支，就可以安全地重組 `state` 了
```ts
if (state.status === "success") {
  const { data } = state;
}
```



這種嚴格性是種 feature，不是缺陷
- 通過確保只有在 state 等於 `success` 時才能訪問資料
  - 鼓勵你根據 application 的 state 來思考，並只在可用 state 下訪問資料



-----------



## 總結
當你開始從區分 state 的角度來考慮 application 時，很多事情都變得簡單了
- 你將開始理解資料和畫面之間的聯繫，而不是一個可有可無的資料包


不僅如此，你還能以一種全新的方式來思考 props
- 如果需要以兩種略有不同的方式顯示一個 component，該怎麼辦？使用 discriminated union:


```ts
type ModalProps =
  | {
      variant: "with-description-and-button";
      buttonText: string;
      description: string;
      title: string;
    }
  | {
      variant: "base";
      title: string;
    };
```


在這，只有當傳入的是 `with-description-and-button` 時，才需要 `buttonText` 和 `description`
