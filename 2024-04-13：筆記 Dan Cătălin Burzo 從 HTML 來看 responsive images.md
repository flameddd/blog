# 2024-04-13：筆記 Dan Cătălin Burzo 從 HTML 來看 responsive images.md
## ref: https://danburzo.ro/responsive-images-html/


----------------

(原文講的很細，有些定義內容我就沒有筆記了)

----------------

## responsive image

通常在講 `responsive image` 時，主要談 css 的
- `width`
- `height`
- `aspect-ratio`
- `object-*`
- ... 各種屬係

如果是從 HTML 來看呢？那會是 `<img>` 上的 `sizes`, `srcset` 和 `<picture>` 和 `<source>` 等等話題  

----------------

## Level 1: 一張圖片，一種分辨率

先從最簡單的 case 來看

```html
<img src="puppy.jpg" alt="A black and white pupper, glancing inquisitively">
```

![](https://danburzo.ro/img/responsive-images/puppy.jpg)  


在沒有任何 attributes 或 styles 的情況下
- `puppy.jpg` 以自然寬度和高度呈現
- `120px X 150px` 以 CSS pixels 為單位呈現

`CSS pixel` 的定義是
- 是在肉眼一臂之遙的距離下，像素密度 (pixel density) 為單個 96 DPI 像素的物理尺寸

「一臂之遙」定義當然不精確、有很多解釋的空間
- 因此，不管螢幕的實際像素密度多少，通常談 `px` 時，是為了想辦法讓 96px 的長度等於 1 英寸(約 2.54 公分)

現在手機的使用習慣距離遠小於手臂的距離
- 螢幕需要具有更大的密度才能看起來更好


iPhone 15 的物理解析度為 `2556×1179` px
- 螢幕對角線尺寸為 6.1 吋
- 像素密度為 `460 ppi`
- 密度高出三倍以上 CSS display
- 因此，iPhone 可以輕鬆地使用三個 device pixels 來繪製每個 `CSS pixel`
- 而網頁內容的大小與在距離較遠的常規螢幕上查看時的大小大致相同


`device pixel` 跟 `CSS pixel` 之間的比例稱為 `device pixel ratio`
- `window.devicePixelRatio` 可以拿到值  
- `device pixel ratio` 並不是實體螢幕性能的固定衡量標準
- User 選擇的解析度或放大和縮小網頁也會影響這比率
- 放大頁面會產生更少、更大的 `CSS pixels`，因此 `device pixel ratio` 會增加
  - (可以用 `window.devicePixelRatio` 測試 )



回到圖片
- 在 `device pixel ratio` 為 2 或 3 的裝置上(通常稱為**視網膜螢幕**)圖在自然尺寸下看起來很模糊
- 在一個 `CSS pixel` 的空間內，螢幕可以使用兩到三個獨立的 `device pixels`，但我們只給它一個 image pixel 來繪製
- 在這些密度較高的螢幕上，如果把圖的密度縮小，讓它跟螢幕的密度相匹配，圖片會看起來會更好。
- 當一個 image pixel 變成一個 `device pixel`，而不是兩個或三個 `device pixel` 時，圖片就會非常清晰


```html
<img src="puppy.jpg" width="120" alt="…"> <!-- 1x -->
<img src="puppy.jpg"  width="60" alt="…"> <!-- 2x -->
<img src="puppy.jpg"  width="40" alt="…"> <!-- 3x -->
```


|`width="120"`|`width="60"`|`width="40"`|
| :---: | :---: | :---: |
| <img src="https://danburzo.ro/img/responsive-images/puppy.jpg" width="120"> | <img src="https://danburzo.ro/img/responsive-images/puppy.jpg" width="60"> | <img src="https://danburzo.ro/img/responsive-images/puppy.jpg" width="40"> |

`<img>` 最好包含明確 `width` 和 `height`
- 這樣 browser 可以預先為圖留出空間，並防止 layout shift
- 也可以防止某些情境，沒有載入(附上) CSS style，例如  RSS feeds 上常常看到 icon image 很巨大，顯然是沒有設定長寬屬性



----------------

## Level 2: 一張圖片，多種分辨率

對於各種密度的螢幕上的清晰圖片
- 將相同圖片檔案縮小到不同程度以增加其密度是不夠的
- 我們需要相應更大的圖來包含更多細節



![](https://danburzo.ro/img/responsive-images/puppy-sizes.png)  

Images with progressively larger resolutions, representing the same content.  

<img src="https://danburzo.ro/img/responsive-images/puppy.jpg" width="120">
<img src="https://danburzo.ro/img/responsive-images/puppy-hd.jpg" width="120">
<img src="https://danburzo.ro/img/responsive-images/puppy-ultra-hd.jpg" width="120">

```html
<img src="puppy.jpg" width="120" > <!-- 1x -->
<img src="puppy-hd.jpg" width="120" > <!-- 2x -->
<img src="puppy-ultra-hd.jpg" width="120" > <!-- 3x -->
```

當縮小到理想密度時，所有三個圖都以相同的尺寸渲染
- `srcset`屬性允許將所有圖打包在一個單一的 `<img/>`，讓 browser 選擇最適合每種情況的圖



<img
  srcset="
  https://danburzo.ro/img/responsive-images/puppy-ultra-hd.jpg 3x,
  https://danburzo.ro/img/responsive-images/puppy-hd.jpg 2x"
  src="https://danburzo.ro/img/responsive-images/puppy.jpg"
  alt="A black and white pupper, glancing inquisitively">


```html
<img 
	srcset="puppy-ultra-hd.jpg 3x, puppy-hd.jpg 2x" 
	src="puppy.jpg"
	alt="…"
>
```

每個項目在 `srcset` 有個 `pixel density descriptor`
- 一個浮點數後面跟著單位 `x`
- 每個圖來源旁邊的描述符聲明了該來源要渲染的 image density。如果省略，預設為 `1x`
- 如果 browser 不支援這功能，就會採用 `src` 的圖
- `src` 的圖也是 `1x` pixel density descriptor 的候選圖片



browser 將從候選圖片中選擇最合適的圖片來源
- 不僅基於 display density，還可能基於其他因素，例如網路速度和 mobile preferences
- 這意味著 browser 可以自由選擇它認為最有效的方式

在沒有 `width` 和 `height` HTML 屬性的情況下
- 無論選擇什麼圖片來源，都會以其密度校正後的自然尺寸(圖片的自然尺寸除以其聲明的密度)進行渲染
- 密度校正後的自然尺寸可透過 DOM 的 `naturalWidth` 和 `naturalHeight` 屬性存取

```js
img.naturalWidth === intrinsicWidth / density;
img.naturalHeight === intrinsicHeight / density;
```


在
- `1x` 密度下，元素的密度校正自然尺寸與檔案的自然(內在)尺寸一致
- `2x` 密度下，圖片的 CSS pixels 數量是檔案自然尺寸的一半


由於我們生成三個圖並聲明其預期密度的方式
- 無論 browser 選擇哪個來源，它們都以 `width` `120px` 來渲染
- 因此，加上 `width`, `height` 屬性都沒有問題



```html
<img 
	srcset="puppy-ultra-hd.jpg 3x, puppy-hd.jpg 2x" 
	src="puppy.jpg"
	width="120"
	height="150"
	alt="…"
>
```

要了解 browser 在任何給定點選擇了哪個圖來源
- 可以查 image 的 [currentSrc](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/currentSrc) 屬性

----------------

## Level 3: 動態圖片密度(image density)
帶有 ` pixel density descriptors` 的 `srcset` 屬性對於以(密度校正後的)自然尺寸顯示的圖非常有效
- 但是，圖通常會參與響應式佈局，並透過 CSS 使其具有流動性
- 因此圖會根據佈局以不同的密度顯示
  - 在大螢幕上，圖片可能是 three-column layout 的一部分
  - 在較小的螢幕上，佈局可能會折疊為具有全寬圖片的單列



在這種情況下
- 圖片密度會發生變化，但顯示密度不會變化
- 因此具有 `density descriptors` 的 `srcset` 並不能解決這個問題



還有第二種方法可以使用 `srcset`
- 為了幫助 browser 在 media 條件改變時選擇適當密度的圖片來源
- 我們可以將 `density descriptors` 換成 `width descriptors` 和單獨的 `sizes` 屬性的組合


`width descriptors` (使用後綴 `w`)
- 不是描述預期的圖片密度
- 而是聲明每個圖片來源的自然(內在)寬度
- 這些資訊本身並不足以讓 browser 做出有意義的選擇
- browser 還需要知道圖片將如何佈局
- 這可以透過 `sizes` 屬性來實現，它聲明了圖片在一個或多個 media 條件下的佈局寬度




<img srcset="
  https://danburzo.ro/img/responsive-images/puppy-ultra-hd.jpg 360w,
  https://danburzo.ro/img/responsive-images/puppy-hd.jpg 240w,
  https://danburzo.ro/img/responsive-images/puppy.jpg 120w"
  sizes="(min-width: 50em) 10em, 80vw"
  alt="A black and white pupper, glancing inquisitively">

```html
<img
  srcset="
    puppy-ultra-hd.jpg 360w,
    puppy-hd.jpg 240w,
    puppy.jpg 120w
  "
  sizes="(min-width: 400px) 10em, 80vw"
  alt="…"
>
```

`srcset` 用 `(w) descriptors` 聲明了三個圖片來源
- 分別為 `360px`, `240px`, `120px`
- `sizes` 描述了圖片的佈局方式
  - 在寬度大於 `400px` 的設備上，圖片將以 `10em` 寬度呈現，否則將佔據視窗寬度的 `80%`

可以使用任何 `CSS unit` 來宣告 `<length>` 的佈局寬度，並使用 `calc()` 和其他 [math functions](https://drafts.csswg.org/css-values/#math-function) 來嘗試大致匹配圖片的實際佈局寬度  


在 `sizes` 中不允許使用**百分比**
- 因為它們與通常理解的父級寬度百分比不一致
- 記住，為 [eagerly-loaded images](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/loading) 選擇圖片來源是在 layout 之前進行的
- 因此我們只能**參考事先已知**的內容，例如 viewport’s dimensions


稍後我們將看到，lazily-loaded images 是在 layout 之後 fetct 的
- [不需要處理](https://danburzo.ro/responsive-images-html/#sizes-auto) 這些尺寸問題


`sizes` 的作用是幫助將 `width descriptors` 轉換為 `density descriptors`
- `width descriptors` 透過以下方式轉換成 `density descriptors`
    1. 從 `size` 值中找出與目前 media 條件相符的 `size`
    2. 將 `size` 值轉換為 `CSS pixels`
    3. 將聲明的 width 除以 pixels 數

根據來源 `width descriptors` 和 layout `size` 計算的密度稱為來源的 ` effective density`  



在寬度為 `1920px` 的較大視窗中
- 圖片將以 `10em` 的寬度顯示，以 CSS pixels 計算則為 `10 * 16px = 160px`
- 大圖片來源的宣告寬度為 `360px`，當呈現為 `160px` 寬時，其有效密度為 `360/160 = 2.25`
- 中型和小圖片的有效密度分別為 `1.5` 和 `0.75`
- 在此視窗寬度下，帶有 `density descriptors` 的等效 `srcset` 為:

```html
<img
   srcset="
     puppy-ultra-hd.jpg 2.25x,
     puppy-hd.jpg 1.5x,
     puppy.jpg 0.75x
   "
   style="width: 10em"
   alt="…"
>
```

在寬度為 `300px` 的較小視窗中
- 圖片將以 `80vw` 顯示，計算結果為 `300px * 80/100 = 240px` CSS pixels
- 在這 media 條件下，三個圖片來源的有效密度分別為 `1.5`, `1` 和 `0.5`
- 在較小的 viewport width 上，使用 `density descriptors` 的 `srcset` 相當於


```html
<img
   srcset="
     puppy-ultra-hd.jpg 1.5x,
     puppy-hd.jpg 1x,
     puppy.jpg 0.5x
   "
   style="width: 80vw"
   alt="…"
>
```


因此，使用 `width descriptors` 的 `srcset` 與 `sizes` 結合
- 是種為圖片來源分配 **dynamic density** 的方法，大致基於圖片在各種 media 條件下的佈局方式
- 當 `srcset` 使用 `width descriptors` 時，圖片的 `src` 屬性純粹是為不支援該屬性的 browser 提供備用屬性
- 該屬性無法提供圖片來源，因為它的值無法附加 `width descriptors` 。

這兩種 `srcset` 屬性最終都會被解析為一組具有 `density descriptors` 的圖片來源
- 但它們並不能很好地混合在一起
- 在一個 `srcset` 中，不能對某些圖片來源使用 `width descriptors`，而對其他圖片來源使用 `density descriptors`
- 要嘛使用 `sizes` 的 `width descriptors` ，要嘛使用不帶 `sizes` 屬性的 `density descriptors`
- 在前一種情況下，`sizes` 屬性是必要的、而在後一種情況下，它沒有任何作用，會被忽略


**不要依賴 default**
- 當我們 `sizes` 需要 `width descriptors` 時
- 這意味著省略它的 HTML 文件[不會驗證](https://validator.w3.org/)，並且它不會成為規範
- 但 HTML 可以容忍錯誤，並且預設值為 `100vw`
- 正如 [Eric Portis 的解釋](https://alistapart.com/blog/post/article-update-dont-rely-on-default-sizes/)，不要依賴該預設值，它可能會推動 browser 獲取比需要大得多的圖片，違背了該功能的全部目的




----------------

## Level 4: `<picture>` tag
`<img>` 上的 `srcset` 單單只向 browser 提供一組候選來源，以及有關它們的足夠資訊以進行明選擇
- 正如 Mat Marquis 在[學習圖片](https://web.dev/learn/images/)中所寫，它使 `srcset` 成為 [descriptive syntax](https://web.dev/learn/images/descriptive)
- 它對 browser 說: 「這是我擁有的，現在你來選擇！」



還有另個 HTML 功能，可以[更具規範性](https://web.dev/learn/images/prescriptive) 並表示僅在滿足這些條件時才考慮這些圖片來源
- 透過將一個或多個與 `<img>` 關聯的 `<source>` 包在 `<picture>` 中來完成的

```html
<picture>
  <source …>
  <source …>
  <img …>
</picture>
```


`<picture>` 是個容器
- 透過提供更多群組來源供選擇(用 `<source>` 聲明)來增強其內部的 `<img>`
- 如果 browser 不支援這些元素，也不會有問題、它們會被忽略，就像 `<img>` 單獨工作一樣。


與 img 一樣，`<source>` 用 `srcset` 和 `sizes` 來聲明圖片來源
- 此外，還接受兩個決定其貢獻的屬性：
  - `type` 聲明圖片的類型，以便 browser 可以跳過它不理解的圖片格式
  - `media` 宣告意義的媒體條件，如果不適用，browser 會跳過這些條件


與目前 media 和 type 相符的第一個來源定義了圖片候選池
- browser 從中選擇最合適的圖片，如 `<img srcset>`
- 如果沒有適用於目前情況的來源，則使用 image 本身的 `srcset` 或 `src` 作為後備



### `type` 屬性
`type` 能夠為支援 browser 提供更新、更有效率的圖片格式，而不會對其他 browser 造成破壞
- 如果 browser 無法用 `image/avif` 或 `image/webp`，它可以忽略對應的 `<source>` 元素

```html
<picture>
  <source srcset='puppy.avif' type='image/avif'>
  <source srcset='puppy.webp' type='image/webp'>
  <img src='puppy.jpg' alt='…'>
</picture>
```




用帶有 type 屬性的 `<source>` 就能安全的提供較新的圖像格式
- 如果省略 `type`，或者直接在 image 的 `srcset` 中使用 `puppy.avif` 和 `puppy.webp`，(舊一點)的 browser 會取得它們不理解的格式，從而無法顯示圖片


### `media` 屬性

`media` 可以包含任何媒體條件
- 如，可以為深色模式提供替代圖片，以及適合列印的更高對比度版本


![](https://danburzo.ro/img/responsive-images/macos-light.png)  

![](https://danburzo.ro/img/responsive-images/macos-dark.png)  

![](https://danburzo.ro/img/responsive-images/macos-contrast.png)  







(如果你的 OS 目前是 dark scheme 的話，下面的圖片會顯示背景黑的那張(`macos-dark.png`)。而當你用 browser 去預覽列印的話，會看到預覽上面出現的高對比的圖片(`macos-contrast.png`)，這些都設計在 **1 個 `<picture />` 中** )

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://danburzo.ro/img/responsive-images/macos-dark.png">
  <source media="print" srcset="https://danburzo.ro/img/responsive-images/macos-contrast.png">
  <img style="max-width: 100%" src="https://danburzo.ro/img/responsive-images/macos-light.png" alt="The Display settings in MacOS, rendered in light mode.">
</picture>

```html
<picture>
  <source media='(prefers-color-scheme: dark)' srcset='macos-dark.png'>
  <source media='print' srcset='macos-contrast.png'>
  <img src='macos-light.png' alt='…'>
</picture>
```

`srcset` 和 `size` 相同適用於 `<source>`
- 無法在單一 `srcset` 中混合 `density descriptors` 和 `width descriptors`
- 並且 `size` 只能與 `width descriptors` 一起使用



`<source>` 也有自己的規則:
- 每個 `<source>` 通常必須附加某種 type 或 media 條件，或兩者兼而有之
- 只有當圖片本身還沒有 `srcset` 時，才允許使用一個裸露的、無條件的 `<source>`
- `<source>` 上沒有 `src` 屬性，因為[這會令人困惑](https://github.com/ResponsiveImagesCG/picture-element/issues/78)，此外，任何有效的 `src` 都可以放入 `srcset`


----------------

## Level 5: 稍微從藝術(art)的角度來談
雖然 `srcset` 旨在表示不同比例的相同圖片內容
- 但多個 `<source>` 可以完全表示不同的內容
- 在前面範例中，使用 `media` 來提供根據使用者偏好設定樣式的圖





**不同的圖片不需要具有相同的長寬比**
- 我們完全可以在不同的場景中提供完全不同的圖片
- 在大螢幕上，照片可以是拍攝對象的廣角鏡頭
- 而在較小的螢幕上，照片可以裁切得更貼近動作



這技術通常被稱為 `art-directing responsive images`
- 為較小的螢幕選擇 `200px * 200px` 的特寫鏡頭。


![](https://danburzo.ro/img/responsive-images/puppy-art-direction.png)  


由於設定 `<img>` 的 `height` 和 `width` 很重要
- 因此 `<source>` 後來也支援設定 `height`, `width`




<picture>
  <source srcset="https://danburzo.ro/img/responsive-images/puppy-closeup.jpg" width="200" height="200" media="(max-width: 40em)">
  <img src="https://danburzo.ro/img/responsive-images/puppy-ultra-hd.jpg" width="450" height="600" alt="A black and white pupper, glancing inquisitively">
</picture>

```html
<picture>
  <source srcset="puppy-closeup.jpg" width="200" height="200" media="(max-width: 40em)">
  <img src="puppy-ultra-hd.jpg" width="450" height="600">
</picture>
```

當 viewport width 小於 `40em (640px)` 時
- art-directed responsive image 會顯示特寫的圖片
  - (注意 `<source>` 上的 `width` 和 `height`)


**圖片載入後之後再裁切**:
- 可以用 CSS `object-fit`, `object-position` 和 `object-view-box` 對圖片進行裁切
- 然而，在實用的範圍內，`<picture>` 提供預先裁剪的圖片可以節省一些網路傳輸和計算






----------------



## 帶有 `sizes=auto` 的 lazy image
HTML 最近新增的內容允許，lazily-loaded images，從而放棄 `sizes` 屬性中的 hard code 近似值
- 相反，使用 `auto` 值，browser 會使用圖的實際 layout width 來計算更準確的候選圖片密度

這篇文章介紹非常詳細
- Eric Portis [covers the feature and its caveats](https://ericportis.com/posts/2023/auto-sizes-pretty-much-requires-width-and-height/) 


----------------


##  browser 如何選擇圖片
下面介紹 browser 會怎麼選擇 image source



----------------


## 在 `srcset` 中使用 `density descriptors` 

下面的圖用了多個 `300×200px` 固定大小的圖
- 但在 `srcset` 中用範圍從 `0.1x` 到 `4x` 的 `density descriptors` 進行標記和聲明

這展示了 browser 選擇的 source
- [(另開單獨測試頁面 )Test: image density selection](https://danburzo.ro/demos/responsive-images/density.html)


<img srcset="
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@0.1x.png 0.1x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@0.2x.png 0.2x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@0.3x.png 0.3x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@0.4x.png 0.4x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@0.5x.png 0.5x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@0.6x.png 0.6x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@0.7x.png 0.7x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@0.8x.png 0.8x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@0.9x.png 0.9x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@1x.png   1x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@1.1x.png 1.1x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@1.2x.png 1.2x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@1.3x.png 1.3x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@1.4x.png 1.4x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@1.5x.png 1.5x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@1.6x.png 1.6x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@1.7x.png 1.7x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@1.8x.png 1.8x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@1.9x.png 1.9x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@2x.png   2x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@2.2x.png 2.2x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@2.4x.png 2.4x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@2.6x.png 2.6x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@2.8x.png 2.8x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@3x.png   3x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@3.2x.png 3.2x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@3.4x.png 3.4x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@3.6x.png 3.6x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@3.8x.png 3.8x,
  https://danburzo.ro/demos/responsive-images/placeholders/300x200@4x.png   4x
">

```html
<img srcset="
  .../placeholders/300x200@0.1x.png 0.1x,
  .../placeholders/300x200@0.2x.png 0.2x,
  .../placeholders/300x200@0.3x.png 0.3x,
  .../placeholders/300x200@0.4x.png 0.4x,
  .../placeholders/300x200@0.5x.png 0.5x,
  .../
  .../placeholders/300x200@3x.png   3x,
  .../placeholders/300x200@3.2x.png 3.2x,
  .../placeholders/300x200@3.4x.png 3.4x,
  .../placeholders/300x200@3.6x.png 3.6x,
  .../placeholders/300x200@3.8x.png 3.8x,
  .../placeholders/300x200@4x.png   4x
">
```


**Firefox**:  
- 在 `100%` zoom 時，device pixel ratio 為 2。放大和縮小頁面會更新 DPR 並重新取得具有高於目前 DPR 的最小密度的圖，或當所有密度都太小時可用的最高密度。
- source code: [ResponsiveImageSelector.cpp#L331](https://github.com/mozilla/gecko-dev/blob/46dae934738073c4f51c2b628503fe3bf84f89d2/dom/base/ResponsiveImageSelector.cpp#L331)

**Chrome**:  
- browser 在載入頁面時會選擇密度最接近 DPR 的圖
- 放大和縮小頁面會更新 DPR，但 browser 只會在 page refresh 時取得另一個更合適的圖
- 它會更喜歡拿 cache 中最密集的 image source，即使其密度遠高於所需的密度
- source code: [html_srcset_parser.cc#L424](https://github.com/chromium/chromium/blob/3167b39925e4ea2c1983d049c8a440d023a727bb/third_party/blink/renderer/core/html/parser/html_srcset_parser.cc#L424)


**Safari**:
- DPR 在 MacBook 上固定為 2，在 iPhone 上固定為 3
- 放大和縮小頁面不會更新 DPR 或取得其他圖片
- 就像 Firefox，會拿到密度最小且高於固定 DPR 的圖
- source code: [HTMLSrcsetParser.cpp#L266](https://github.com/WebKit/WebKit/blob/4e262b07baa1b55186d79a67b35f326cfdea48b2/Source/WebCore/html/parser/HTMLSrcsetParser.cpp#L266)



總之，根據 browser 的演算法，每個 image source 會被評估
- srcset 中的順序不會影響選擇
- 每個特定密度(無論是 declared density 還是 effective density)的第一個項目都會被保留，任何重複的項目都會被刪除。



browser 目前並沒有採用 HTML spec 所設想的複雜決策
- browser 總是傾向於使用接近目前 DPR 的 image density


除了 Firefox 會對 zooming 做出反應外，其他 browser 在整個頁面期間都會堅持選擇圖片來源
- 在任何 browser 中，縮放都不會影響 DPR，因此 raster images 不會因為縮放而神奇地加強
  - (後面那句看不懂)



----------------

## 在 `srcset` 中使用 `width`

下面這張 placeholder 所顯示的圖片大小是由 browser 自己選擇的
- `<img />` 中的 `srcset` 指定了**10 種**不同大小的圖片

<img srcset="
  https://danburzo.ro/demos/responsive-images/placeholders/300x200.png    300w,
  https://danburzo.ro/demos/responsive-images/placeholders/600x400.png    600w,
  https://danburzo.ro/demos/responsive-images/placeholders/900x600.png    900w,
  https://danburzo.ro/demos/responsive-images/placeholders/1200x800.png  1200w,
  https://danburzo.ro/demos/responsive-images/placeholders/1500x1000.png 1500w,
  https://danburzo.ro/demos/responsive-images/placeholders/1800x1200.png 1800w,
  https://danburzo.ro/demos/responsive-images/placeholders/2100x1400.png 2100w,
  https://danburzo.ro/demos/responsive-images/placeholders/2400x1600.png 2400w,
  https://danburzo.ro/demos/responsive-images/placeholders/2700x1800.png 2700w,
  https://danburzo.ro/demos/responsive-images/placeholders/3000x2000.png 3000w"
sizes="100vw">

```html
<img srcset="
  .../placeholders/300x200.png    300w,
  .../placeholders/600x400.png    600w,
  .../placeholders/900x600.png    900w,
  .../placeholders/1200x800.png  1200w,
  .../placeholders/1500x1000.png 1500w,
  .../placeholders/1800x1200.png 1800w,
  .../placeholders/2100x1400.png 2100w,
  .../placeholders/2400x1600.png 2400w,
  .../placeholders/2700x1800.png 2700w,
  .../placeholders/3000x2000.png 3000w"
sizes="100vw">
```


上面用多個具有相同寬高比(aspect ratio)但不同比例的圖
- 在 `srcset` 中用從 `300w` 到 `3000w` 的 `width` 描述進行標記和聲明
- 並指定 `sizes="100vw"`

(可以用 browser devtool 切換成 mobile view，你會看到圖片會切換成別張)  

	

在 `srcset` 中使用 `width` 描述時
- 我們希望 browser 考慮 `size` 屬性(此處為 `100vw`)來計算隨 viewport 更新的 density descriptors
- 因此，browser 的行為或多或少類似於 density descriptors 也就不足為奇了
- 還有個額外的好處，就是 image sources 能更頻繁地重新選擇

**Firefox**
- 調整 browser 視窗的大小會使 browser 在任何給定時刻選擇具有適當 effective density 的圖
- Zooming in and out 頁面會重新選擇圖片，但通常不會產生任何效果
  - 雖然 `CSS pixels` 的大小增加(隨之而來的是 DPR)，`100vw` 評估為更小的 `CSS pixels`，這會導致更多或整個圖片 density 不太固定

**Chrome**
- 與 Firefox 一樣，調整 browser 視窗大小會導致 browser 根據 image sources 的 effective density 重新選擇圖片來源
  - 如 density descriptors 測試所示，Chrome 會 caches 其所獲取的 images，並始終使用最密集的可用圖
- 一旦取得 large viewport，即使縮小 viewport，也會使用 densest 的圖


**Safari** 
- 對於獲取其他圖是最保守的
- 使用 `sizes=”100vw` 時，image source 會在頁面載入時評估一次，並且調整 browser 視窗的大小不會產生任何影響
- 使用包含 `media` 條件的屬性，例如  `sizes="(max-width: 400px) 25vw, (max-width: 800px) 50vw, 100vw"`，一旦套用不同的尺寸，它就會重新選擇圖片來源


Safari 的方法代表 `sizes="100vw"`，不會像其他 browser 那樣使圖片流暢
- 其他 browser 會在每次調整大小後更新經過密度校正的自然尺寸
- 尺寸只有在 source 第一次 redenr 時計算一次


----------------

## 更多的 responsive images refs
- [Learn images](https://web.dev/learn/images/) by Mat Marquis  <-- 這邊差一個連結
- [Srcset and sizes](https://ericportis.com/posts/2014/srcset-sizes/) (2014) by Eric Portis
- [Responsive Images 101](https://cloudfour.com/thinks/responsive-images-101-definitions/) (2015), a ten-part series (and [book](https://cloudfour.com/thinks/announcing-our-new-ebook-responsive-images-101/)) by Jason Grigsby
- Eric Portis’s Observable notebook [w descriptors and sizes: Under the hood](https://observablehq.com/@eeeps/w-descriptors-and-sizes-under-the-hood) explains these concepts and behaviors with nice-looking, interactive browser tests.
