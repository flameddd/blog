# A Small Guide for Naming Stuff in Front-end Code
## October 21, 2021 by paceaux
### https://blog.frankmtaylor.com/2021/10/21/a-small-guide-for-naming-stuff-in-front-end-code/

這篇
- Write good code that won’t piss off your teammates or a future version of you.  
- 幾個命名原則還不錯，尤其是 css 的部分

## 命名 CSS Classes

命名「狀態」的 Class names
- 這個狀態只有兩種情況嗎？例如 boolean，是的話
  - 用 `is/has` 做 prefix
- 是某個 element 本身的來決定的狀態？
  - 用 `.is` 來描述元素本身的狀態
  - 嘗試用名詞來做結尾

```css
.isOverflowHidden
.isHidden
.isScrollable
```

- 狀態是被自己的 children 來決定的
  - 用 `has` 來描述

```css
hasOverflowHidden
.hasHiddenChildren
.hasScrollableChildren
```

- 是一個「負面」的狀態嗎？
  - 盡可能的避免，因為可能很難使用；嘗試重寫為正面的狀態
  - 使用 `isNot` 或 `hasNo`
  - 把 `is` 當形容詞使用
  - 嘗試在 `has` 後面加上名詞、加一個 `Els` (element 的說寫)也沒關係
  - 如果上面都沒辦法，那就用 `able` 吧  

```css
.isNotHidden
.hasNoHiddenEls
.isNotScrollable
.hasNoScrollable
.hasNoScrollableEls
```

- 是 `JavaScript` 所需要的 CSS class 嗎？
  - 用 `js/ui` 做 prefix
- `JavaScript` 是否只是為了來找到這個元素，而不是為了要做任合 css or style
  - 用 `.js`
  - `.jsSubmitButton` `.jsVideoPlay`
- 需要用 `JavaScript` 去改變它，使其能改變 style ?
  - 用 `.ui`
  - 用形容詞 or 過去式
  - `.uiSubmitted` `.uiVideoPlayed`
- 是 test 所需要的嗎？
  - 用 `.qa`

這些 class name 是針對 component or module 的嗎？
- 避免詞彙去描述/指定 content
- 避免詞彙去描述/指定 markup 
- 避免詞彙模​​棱兩可，描述 scope or presentation
- 避免詞彙去描述 variation or relationships

舉例，避免類似的
```css
.logos // (prescribed content)，建議這個只能用在 logo 上
.logoList // (prescribed markup) 建議只能用在 ul 、ol 、dl
.logoGridList // (ambiguous presentation)
.darkLogoGridList // (words describe variation or relation) 不明顯相關
.logoGridListBlack // (ambiguous scope) 不清楚什麼是黑色
```

應該要
- 描述目的的詞彙
- 描述出範圍
- 如果含義很明確，那考慮縮小範圍
- Prefer convention to prescribe variations
- Prefer convention to describe relationships

```css
.mediaSet // 'Set' ->; purpose, 'media' -> scope (could apply to video, picture, img)
.mediaSet--grid-md // '--' ->; variation by convention, 'grid' -> purpose  , 'md' -> scope (abbreviated)
.mediaSet--theme-dark // '--' ->; variation by convention, 'theme' -> scope, '-dark' -> variation
.mediaSet__media // '__' ->; relationship by convention , 'media' ->  purpose
```

應該使用 `-` 還是 `_` 來分隔前綴？
- 保持一致就好，如果其他人都在使用連字符，那麼您也應該使用連字符。
- 如果你是該項目的第一人，請使用 camelCase；在 IDE 中選擇駝峰式比選擇連字符文本更容易
一般來說，選擇對團隊影響最小的約定。

這是一個 SRP（single responsibility principle）類別嗎？
- Don’t use the value of a specific CSS property
- Do give the purpose of that specific CSS property
- Avoid suggesting usage with only specific HTML elements

避免
```css
.button--yellow {
  color: yellow;
}

.imgFloatLeft {
  float: left;
}
 
.imgFloatRight {
  float: right;
}
```

應該要
```css
.button--warn {
   color: yellow 
}

.rtfMedia--left {
 float: left;
}

.rtfMedia--right {
 float: right;
}
```

## 命名 CSS Pre-processor (SCSS, Sass, Less, Stylus)

這些變數是 colors 嗎？
- 不要讓變數名稱指出變數的值（如果值改變了就麻煩了）
- 有意義的成稱，例如 hue, lightness, or darkness
- 色調方面，使用 relatives and superlatives (er, est) 

```css
--colorNeutralDarkest
--colorNeutralDarker
--colorNeutralDark
--colorWarmDarker
--colorWarmerDarkest
```

通用的變數
- 這個變數會被多次使用嗎？
- 從 UI element 元素名稱開始
- 中間採用具有描述性的詞彙
- 它所影響的 css 屬性最結尾
- 避免將任何有關 value 的東西放到名字中

通用變量
- 變量會被多次使用嗎？
- 從 UI 元素名稱開始
- 在中間更具有描述性
- 以它影響的 CSS 屬性結束
- 避免將有關變量值的任何信息放在名稱中

```css
$linkColor: rgb(165,220,220,220);
--linkColor: var(--colorCoolDark);
$formControlVerticalPadding: .375em;
```

這個變量是否多次使用，但 pseudo 類有不同的變量？
- 從 UI 元素名稱開始
- 加它影響的 CSS 屬性
- 以狀態結束

```css
$linkColor: rgb(165,220,220); 
$linkColorHover: rgb(165,165,220); 
--linkColorFocus: var(--colorCoolDarker);
```

變量是否使用過一次？
- 從使用變數的 class name 開始
- 以它影響的 CSS 屬性結束

```css
.foo { 
  $fooHeight: $containerHeight / 3; 
  width: $fooHeight / 2; 
  height: $fooHeight; 
}
```

## JavaScript 命名

命名某個會 return value 的 function  
function return boolean 嗎
- 用 prefix `is` or `has`
  - `is` 搭配形容詞、`has` 搭配名詞
- 避免負面命名

```js
function isHot() {
 if (temperature > 100 ) {
   return true;
 } else {
   return false;
 }
}
function hasEggs(carton) { 
  if (carton.eggs.length > 0 ) { 
    return true;
  } else { 
    return false;
  }
}
```

function 回傳其他種類的值？
- 用 `get` 做 prefix
- 結尾寫取回什麼東西
- 如果回傳是 enumerable，就用複數形式

```js
function getTemperature () { 
  return temperature; 
} 
function getEggs() { 
  const eggs = []; 
  return eggs; 
}
```

是透過這 function 回傳的值來宣告變數嗎？
- 用名詞的部分來作為 function name 的部分

```js
const foods = getFoods();
const messages= prepareMessages();
```

function 沒有 return value
- 用動詞開頭、名詞結尾
- 使名詞成為動作的接受者

```js
calculateTemperature()
findThermometer();
```


用來 Looping 某些東西
- 避免用單一字母命名變數 (e.g. i);
- 使用至少三個字母
- 採用一個容易指出它代表什麼的名字

looping 是 forEach 用在 data set 上 or iterable 的結構上嗎？
- data set or iterable 採用複數名詞
- item 用單數

```js
const fruits = ['banana', 'avocado', 'tomato']; 
fruits.forEach((fruit) => { });
```

這個 `.forEach` 需要用到 `index` 嗎?
- 至少使用 `idx`
- 避免單一單字

```js
const foodGroups = [ ['banana', 'avocado', 'tomato'], ['steak', 'sausage' ] ];

foodGroups.forEach((foodGroup, idx ) => {
  foodGroup.forEach((food, foodIdx) => {});
});
```

是使用 `for in` or `for of` 嗎？
- object/array 的變數名稱用複數名詞
- 有用到 counter 的話，用 `idx`
  - 有多層的話，把 `idx` 放結尾

```js
for (let idx = 0, fruit; idx < fruits.length; ++idx) {
  fruit = fruits[idx];
  for (let goodFruitIdx = 0; goodFruit; goodFruitIdx < goodFruits.length; ++goodFruitIdx) {
    goodFruit = goodFruits[goodFruitIdx]
  }
}
```

## 一般 Functions

複雜的 function (Functions with many parts)
- 辨別出這個 function 裡面所有的 tasks（units of work that have single, testable activities）
- 對上面這些 task 都命名、建立 function，命能代表這些 task 的名稱
- 重新評估這些 function，想一個能 summarizes 這些 task 的名稱 

BAD
```js
function getLiElement(titleNode) { // function name 太廣泛
  const listEl = document.createElement('li');
  const { tagName} = titleNode;
  let type = 'biggest';
  switch (tagName) { // task that determines a part of a class name
    case 'H3':
      type = 'bigger';
      break;
    case 'H4':
      type = 'big';
      break;
    case 'H5':
      type = 'base';
      break;
    default:
      break;
  }
  const itemTypeClass = `articleTOC__item--${type}`; // operation that concatenates strings
  listEl.classList.add('articleTOC__item');
  listEl.classList.add(itemTypeClass);
  
  return listEl;
}
```

GOOD:
```js
// function that does one thing with name that reflects what it does
function getTOCItemType(titleNode) {
  if (!titleNode) return;

  const { tagName} = titleNode;
  let type = 'biggest';
  
  switch (tagName) {
    case 'H3':
      type = 'bigger';
      break;
    case 'H4':
      type = 'big';
      break;
    case 'H5':
      type = 'base';
      break;
    default:
      break;
  }

  return `articleTOC__item--${type}`;
}
function getTOCItemElement(titleNode) {
  const listEl = document.createElement('li');
  // variable that summarizes what the other function has done
  const tocElementTypeClassname = getTOCItemType(titleNode);

  listEl.classList.add('articleTOC__item');
  listEl.classList.add(tocElementTypeClassname);
  return listEl;
}
```

複雜的 function (很難理解、讀的)
- 看看有沒有辦法減少複雜度
- 看看  comparison operations (if, switch)，確認看看是否在做一些 self-evident 事情
  - 考慮 create 新變數來代表「每一個 logical comparison」 (follow 上面是如何命名 boolean 的)
  - 考慮 create 一個變數來 summarizes 所有的 logical comparisons 
- 減少 **magic numbers** and **magic strings**

Determine if it is complex. If it is, make it less so.
Look at comparison operations (if, switch) and determine if it is self-evident what the logic is doing
Consider creating a variable for each logical comparison (follow the guidance for how to name boolean variables)
Consider creating a variable that summarizes the logical comparisons
Eliminate “magic numbers” and “magic strings” (values that exist, but their reason for being those values is not self-evident)
Consider moving numbers in a setTimeout to variables
Consider making values used for validating parameters their own variables
Consider naming all of the parts of a concatenated string

BAD
```js
function getTOCItemType(titleNode) {
  if (!titleNode) return;
  const { tagName} = titleNode;
  let type = 'biggest'; // variable name whose purpose isn't explained
  switch (tagName) {
    case 'H3':
      type = 'bigger';
      break;
    case 'H4':
      type = 'big';
      break;
    case 'H5':
      type = 'base';
      break;
    default:
      break;
  }
  return `articleTOC__item--${type}`; // "magic string" ; unclear why type is added to a string
}
```

GOOD
```js
function getTOCModifierClass(titleNode) { // name more clearly explains purpose
  if (!titleNode) return;
  const { tagName} = titleNode;
  const blockName = 'articleTOC__item'; // variable that explains its value
  const modifierSeparator = '--' // variable that explains its value
  let modifierName = 'biggest'; // changed variable name that explains why it changes
  switch (tagName) {
    case 'H3':
      modifierName = 'bigger';
      break;
    case 'H4':
      modifierName = 'big';
      break;
    case 'H5':
      modifierName = 'base';
      break;
    default:
      break;
  }
  return `${blockName}${modifierSeparator}${modifierName}`; // clarity that this is returning a BEM class
}
```

Function 的 Parameter names
- 避免把 type 放到 name 中，那是 JSDOC 的事情
- 但，如果你覺得這個 type 非常重要，那還是可以放。放在開頭

```js
function addNumbers(numberFirst, numberSecond) {}
```
