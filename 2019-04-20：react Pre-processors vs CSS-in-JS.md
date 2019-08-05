## react Pre-processors vs CSS-in-JS
cross 團隊要選定一種技術來 build common components

> 個人（想法、偏好）一個大前提是，不要專案內有 2 種 style 的方式。
所以勢必想要選一種

大問題之一
SASS 能自動 vendor prefixes 嗎？



### 參考資料
- X個方向
- Sarrah Vesselov (former gitlab UX manager、designer)
- styled-component 的作者找看看 blog 跟 twitter
- UI must read
- 丟到社群去
- styled-component 的首頁介紹

### 我自己的前提偏好
不想到時候在 App 裡面**同時有兩套 style 維護方式**  

```js

const Container = styled.div`
//...
` 
return (
  <Container
    className={classNames(
      'folder',
      { 'state-folder-open': this.state.menu }
    )}
  >
  </Container>
)
```
:cry: :cry: :cry: 

## Pre-processors 
### Pre-processors 優點
- The power of variables, mixins, and nesting
- write less code
- easy to pickup （對 css 熟悉的人，進入 SASS 的成本不高）
- easy create **modular CSS** that compiles into one sheet (也有modular CSS 的功能)

### Pre-processors 缺點
- CSS selectors 全部都在 global scope
  - selector 就有可能相互影響、碰撞 
- Any time we make a change to a CSS file, we need to carefully consider the global environment in which our styles will sit. 
- need so much discipline just to keep the code at a minimum level of maintainability.
- dependencies (SASS 好像需要一些額外的 config)
- Nesting 對 scalability 是個挑戰，極有可能不斷的 nest css 下去
  - nesting can be out of control very very quickly; 10 deepth
- Component Composition reduces need for mixins/nesting （開發時，應該要盡量創照 very small 的 component）
- 需要去命名一堆 className
 - common component 就會需要 follow 一套命名原則 (BEM, SMACSS)
- dead CSS code
  - 很難找到這些 code 並刪除（更深的情況 multiple code bases）

## CSS-in-JS
### CSS-in-JS 優點
- All ths CSS (寫 css 就好，不用學寫其他 sass 之類的)
- independent and reusable (所以都是小的、獨立的 component，可重複利用)
  - 更小的元件（粒度）組合性、複用性更高
- Single use class names (不用命名 classname、有 scope 存在，不會有 css collisions)
- basic Sass support (Sass 的特性都能支援、variables, mixins, and nesting)
- Theme support （可以用 provider provide theme)
- 清楚知道改動範圍、不太會有 dead code，可以明確知道 css 的 style 是什麼物件
- Automatic critical CSS (不再是載入整包 css file，而是像有被 code splite 後元件單獨的 style）

### CSS-in-JS 缺點
- more choices (用 CSS-in-JS 代表要選一套 library）
  - 基本上我們就是選用 styled-component，選定一套 library 對未來會不會有問題，是個點
  - 新的 lib 一直出、改變太快，今天選一種，未來要轉也是個困難
- Learning curve (新的 lib === 學習曲線)

### release問題
- 做不到 UI team release，所有的 common component 自動 upgrade
- 頂多做到 UI team release css，所有 common component 的 css upgrade
  - 這問題又延伸，事前怎麼測試（怎麼測試所有（已知）有用到的產品 upgrade 後的結果）

還是希望是 npm library，這樣 production Owner team 才能掌控。當然如果 UX 希望掌控這段，上述問題就是 UX 要回答的

### 維護問題
之前是說
 - TW, BJ, HK_UX 各派 1~2 位出來 維護
 - 相互 code review

個人覺得**這流程很糟**  
1. 我有一個 task 要進行，所以我去改 common component
2. 然後要求 BJ or HK 的人去 code review 並且確認 **這樣改動會不會影響（改壞）他們產品**

**這就等於 我們 team 有 task，其他 BJ, HK 也要花人力進來進行**  
**是不是未來統一由 UX 管理？（但現在又是 BJ team 想先斬後奏，先弄一個逼 HK...）**  

### bootstrap
- 初版 follow bootstrap css 做為 base，應該ＯＫ，但後續一律跟 bootstrap 脫鉤。
  - 不會因為 bootstrap 改版，我們就想跟著改
  - 而是看到 缺點 or 某些 UI framework 做什麼好的 feature 我們自己來實現

這樣應該就不涉及到 bootstrap 升級問題

### 轉換成本
- 若決定 SASS 為 base 的話，那 TW team 所有 project 都要重構（BJ team 其實也需要吧）
  - 例如 build css, import css, refactor code (e.x. dropdown 就要重新用 classnames 去判斷之類的）
- 轉換的過程肯定是很慢的。

### styled-Component 版本問題
- 版本不一致問題也是要解決，Yui 之前有內部 npm 發佈一個版本，大家去裝這個版本應該可行
  - 舊產品的目前版本太低的，需要轉換時間

### 一些想法
- 藉機這次訂出 style guide (design system)，release 後，common component 跟後需所有產品 team 開發時 follow

也許 UX team 已經有了，但我好像從來沒看到完整的 guide

- 依粒度來優先開發（低到高） common component
  - 才能用**低**的來組合出**高**的，不然到時候變成先做高的，然後去**拆成低**的，甚至去**改動低粒度**的元件
- 高粒度先在各產品開發  

![image](/uploads/3e76f94e17bba6a12bf79bd74b6f7880/image.png)  
- `Low` complexity & `High` reusability: **Buttons, Text, Labels & Grid**
- `Medium` complexity & `Medium` reusability: **Input fields with labels and validation text**
- `High` complexity & `Low` reusability: **Date picker**

(source: https://medium.com/styled-components/build-better-component-libraries-with-styled-system-4951653d54ee)

### refs
### Sarrah Vesselov (former gitlab UX manager、designer)
#### https://www.youtube.com/watch?v=V-flI7ui52M
影片把 styel 分成 4 方式
- Standardization （純 css）
- Pre-processors (SASS etc)
- inline Styles (react inline style)
- CSS-in-JS (styled-component etc)

分別講解，最後條列出她覺得的優缺點  
最後選用哪種，她還是認為看你 team 跟 產品需求（大小）


### 精彩討論
- https://news.ycombinator.com/item?id=19190641

### others
#### https://medium.com/styled-components/getting-sassy-with-sass-styled-theme-9a375cfb78e8

還是有人嘗試轉換、結合 `sass` + `styled-components`，做法是
- 先把基本的 css 手動轉成 styled-components 的元件
- 用套件把 sass 的`變數`抽出來當作 theme，傳給`<ThemeProvider />` (styled-componentns)
  - 這樣 `styled-componentns` 就能取到 `sass` 定義的`變數`。

