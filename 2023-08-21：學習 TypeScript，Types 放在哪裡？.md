
# 2023-08-21：學習 TypeScript，Types 放在哪裡？.md
## Matt Pocock, May 24, 2023
### Ref: https://www.totaltypescript.com/where-to-put-your-types-in-application-code


--------

## 規則 1: 當一個 type 只在一個地方使用時，就放在使用它的檔案中

大多 applcation 都是由 function 和 class 構建的
- 而當在處理它們時，往往需要對它們的 type 進行修改，以保持它們的運行。

最典型的例子就是 component props
- 如果有個 `MyComponent`，需要使用 props `foo` 和 `bar`，建議將其 type 放在同個檔案中


```ts
// MyComponent.tsx

interface Props {
  foo: string
  bar: number
}

export const MyComponent = (props: Props) => {
  // ...
}
```



另種方法是將 type 移到一個單獨的檔案中：
```
|- src
  |- components
    |- MyComponent.tsx
    |- MyComponent.types.ts
```


但這種方法非常不直觀
- 當在處理 MyComponent 時，經常需要同時編輯它和它的 type
- 將它們放在不同的檔案中只會增加工作難度


當 type 確實是一次性使用時，不要害怕 inline:
```ts
// MyComponent.tsx

export const MyComponent = (props: {foo: string; bar: number}) => {
  // ...
}
```

很多團隊不願意 inline type
- 因為感覺太混亂
- 但不要覺得你總是需要將一個 type 取為一個單獨的 type 或 interface
- inline 是絕對沒問題的，而且如果需要的話，以後再重構成本也非常低


--------

## 規則 2: 在多個地方所使用的 type，應該移到共享位置


應該把 application 中的 type 看作 function
- 如果 function 在多個地方中使用，你會想把它移到共享位置，type 也是如此。


這通常意味著在適當位置建立一個 `*.types.ts` 檔案
- 如果它們在整個 application 中共享，把它們放在 `src` folder 下


```
|- src
  |- components
    |- MyComponent.tsx
  |- shared.types.ts
```


如果只在 `components` folder 中使用，就放在那裡：
```
|- src
  |- components-
    |- MyComponent.tsx
    |- components.types.ts
```


--------

## 規則 3: 在 monorepo 中不止一個 package 中使用的 type 應該移到 shared package 中

如果你正在處理包含多個 package 的 monorepo
- 在這情況，應將 shared type 移到 shared package

```
|- apps
  |- app
  |- website
  |- docs
|- packages
  |- types
    |- src
      |- shared.types.ts
```


上面的例子
- 有個包含三個 application 的 monorepo: `app`, `website`, `docs`
- 還有一個 `types` package，含整個 monorepo 共享的 type


根據你的 monorepo 結構，實現方式可能會有所不同
- Turborepo docs 的一篇指南，也許能幫助起頭
  - [internal packages](https://turbo.build/repo/docs/handbook/sharing-code/internal-packages)




--------



## 結論: 三條規則
1. 當一個 type 只在一個地方使用時，就放在使用它的檔案中
2. 在多個地方所使用的 type，應該移到共享位置
3. 在 monorepo 中不止一個 package 中使用的 type 應該移到 shared package 中
