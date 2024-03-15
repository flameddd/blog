# 2022-11-08：Jurn van Wissen: Let’s Make One of Those Fancy Scrolling Animations Used on Apple Product Pages
## Jurn van Wissen on May 22, 2020 (Updated on May 25, 2020)
### https://css-tricks.com/lets-make-one-of-those-fancy-scrolling-animations-used-on-apple-product-pages/

一直沒有好好想過 Apple 官網的一些 animation 實作細節  
看看這篇怎麼是怎麼實作的  
(看了之後，是講怎麼做隨這 scroll position 來帶出光影效果)  
完整的 react source code 在最下面  

---------------------------------------


Apple 的頁面一直有很動畫，當往下 scroll 時，經常會有類似
- macbook 銀幕打開
- iphone 旋轉

等等特效，下面影片是 ipad pro 頁面往下 scroll 時的效果

<p align="center">
<video width="80%" height="500px" autoplay muted loop  src="https://github.com/flameddd/blog/assets/22259196/a25a4656-a23e-41cb-adff-f51064dd786d" ></video>
</p>
(video source: https://res.cloudinary.com/css-tricks/video/upload/f_auto,q_auto/v1588108817/ipad-pro_wdpam6.mp4)  


很多效果都不是只用 `HTML` 和 `CSS`，即使用 devtools 也不一定能找到答案，因為它往往無法看透 `<canvas>`  
下面展示看看該如何實作 AirPods Pro 頁面的 shifting light effect  
 
<p align="center">
<video width="80%" height="500px" autoplay muted loop  src="https://github.com/flameddd/blog/assets/22259196/57dd58d3-2252-4630-b605-e8abb20a2872" ></video>
</p>

(video source: https://res.cloudinary.com/css-tricks/video/upload/v1588889054/airpods-animation_g2qstg.mp4)  

---------------------------------------

## 1. 基本概念
創造一個動畫，就像一個快速連續的圖片序列
- 就像翻頁書，不需要復雜的 `WebGL` 或其他 JS lib
- 將每一 frame 與 User 的 scroll 的位置同步，我們可以在 User 向下（或向上） scroll 時播放該動畫

<p align="center">
<video width="80%" height="500px" autoplay muted loop  src="https://github.com/flameddd/blog/assets/22259196/cf81bf49-18dd-4c40-b83a-af2fecbaad39" ></video>
</p>
(video source: https://res.cloudinary.com/css-tricks/video/upload/v1588889054/flipbook_tisnxf.mp4)  

---------------------------------------

## 2. 首先從 markup 和 styles 開始

這個效果的 HTML 和 CSS 很單純
- 因為魔法發生在 `<canvas>` 中，我們用 JavaScript 給一個 ID 來控制

CSS:
- 給 document `100vh` height
- 然後把 `<body>` 設 5 倍的高，讓我們能 scroll
- 設定 background color，來配合圖片
- 把 `<canvas>` 置中、限制別超過一個頁面的大小


```css
html {
  height: 100vh;
}

body {
  background: #000;
  height: 500vh;
}

canvas {
  position: fixed;
  left: 50%;
  top: 50%;
  max-height: 100vh;
  max-width: 100vw;
  transform: translate(-50%, -50%);
}

```

現在，可以向下 scroll，而 `<canvas>` 會保持在畫面上  
- 下一步是載入圖片

---------------------------------------

### 3. Fetch 正確的圖片

我們要處理一個圖片序列
- 我們將假設檔案名稱是升序編號（`0001.jpg`, `0002.jpg`, `0003.jpg`, ...）
- 圖片都在同一個目錄中

寫個 function，根據 User 的 scroll 位置
- 回傳我們想要的圖片編號的路徑
- 圖片編號是一個整數，我們把它變成 string，並用 `padStart(4, '0')` 在前面預置 `0`
  - 直到達到四位數來 match file name
- 舉例，向這個 function 傳 `1`，將會 return `0001`

```js
const currentFrame = index => (
  `https://www.apple.com/105/media/us/airpods-pro/2019/1299e2f5_9206_4470_b28e_08307a42f19b/anim/sequence/large/01-hero-lightpass/${index.toString().padStart(4, '0')}.jpg`
)
```

下面是在 `<canvas>` 上繪製的序列中的第一張圖片:
- codepen: https://codepen.io/j-v-w/pen/BaoQPeg

```js
// 基本上就是去 fetch 圖片，然後放在 canvas 上面  
const canvas = document.getElementById("hero-lightpass");
const context = canvas.getContext("2d");

const currentFrame = index => (
  `https://www.apple.com/105/media/us/airpods-pro/2019/1299e2f5_9206_4470_b28e_08307a42f19b/anim/sequence/large/01-hero-lightpass/${index.toString().padStart(4, '0')}.jpg`
)

// Set canvas dimensions
canvas.width=1158;
canvas.height=770;

// Create, load and draw the image
const img = new Image()
img.src = currentFrame(1); // we'll make this dynamic in the next step, for now we'll just load image 1 of our sequence
img.onload=function(){
  context.drawImage(img, 0, 0);
}
```

我們要做的是根據 User scroll 的位置來更新它
- 通過 fetch 另張圖片來替換掉
- 在 `<canvas>` 上繪製圖片，並以序列中的下一個圖片來更新繪製
- 現在需要做的是跟踪 User scroll postion，並確定該 scroll position 的相應 image frame


---------------------------------------

## 4. 將圖片與 User scroll 整合
為了知道需要在序列中傳遞哪個數字（也就是要加載哪個圖片）
- 我們需要計算 User 的 scroll 進度
- 做一個事件監聽器來跟踪來計算要加載哪張圖片

我們需要知道:
- scroll 的開始和結束位置
- User 的 scroll 進度（即 User 在頁面中的百分比）
- 與 User 的 scroll 進度相對應的圖片

使用 `scrollTop` 來取 element 的垂直滾動位置
- 在這例子中，這個位置剛好是 document 的頂部。這將作為起點值
- 用 `window height` 減去 `document scroll height`，得到終點（或最大值）值
- 用 scrollTop 值除以 User 可以向下滾動的最大值，就得到了 User 的滾動進度



然後要將該滾動進度轉化為一個「索引號」
- 該「索引號」與「圖片編號序列」相對應，以便我們返回該位置的正確圖片
- 可以通過將「進度號」乘以我們擁有的幀數（圖片）來實現
  - 用 `Math.floor()` 四捨五入，然後用  `Math.min()` 最大幀數，這樣它就不會超過總幀數了


```js
window.addEventListener('scroll', () => {  
  const scrollTop = html.scrollTop;
  const maxScrollTop = html.scrollHeight - window.innerHeight;
  const scrollFraction = scrollTop / maxScrollTop;
  const frameIndex = Math.min(
    frameCount - 1,
    Math.floor(scrollFraction * frameCount)
  );
});
```

---------------------------------------

## 5. 更新 `<canvas>`
`<canvas>` 其中一個功能是 [requestAnimationFrame](https://css-tricks.com/using-requestanimationframe/) 的方法
- `requestAnimationFrame` 能跟 browser 一起更新，`img` 就沒有這樣的方法
- 這就是為什麼這邊使用 `<canvas>` 而不是使用 `<img>` 元素或帶有背景圖片的 `<div>` 的原因。
- requestAnimationFrame 會同步 browser 的 refresh rate
- 並用 WebGL 啟用硬體加速
- **換句話說，我們將得到 frame 之間的超級平滑過 transitions，沒有圖片 flashes**

在 scroll 事件監聽器中呼叫這個 function
- 在 User 向上或向下 scroll 頁面時交換圖片
- `requestAnimationFrame` 需要 callback

```js
requestAnimationFrame(() => updateImage(frameIndex + 1))
```


把 `frameIndex + 1`
- 雖然圖片序列是從 `0001.jpg` 開始的
- 但 scroll 進度計算實際上是從 0 開始的。這可以確保這兩個值始終保持一致


callback function:

```js
const updateImage = index => {
  img.src = currentFrame(index);
  context.drawImage(img, 0, 0);
}
```

---------------------------------------

## 6. 更好的 image preloading

在技術上
- 我們已經完成了
- 但，快速 scroll 會導致圖片 frame 之間有一點滯後
- 因為每一個新的圖片都會發出一個新的網絡請求，需要一個新的下載

我們應該 preloading image
- 這樣，每一幀都已經下載好了，使動畫順暢

Loop 瀏覽整個圖片序列並 preload 它們:

```js
const frameCount = 147;

const preloadImages = () => {
  for (let i = 1; i < frameCount; i++) {
    const img = new Image();
    img.src = currentFrame(i);
  }
};

preloadImages();
```

目前為止的 demo (隨 scroll 的光影效果)
- https://codepen.io/j-v-w/pen/ZEbGzyv

(上面這個 code 另外有個 performace 問題，文章的留言討論有提到，修正版本跟完整的 react code 我下面有貼)  

---------------------------------------

## 7. 關於 performance 的簡短說明

雖然這個效果相當巧妙，但它也有很多圖片。確切地說，是147張
- 不管我們如何優化圖片，或者提供圖片的 CDN 速度有多快
- 加載數百張圖片總是會導致頁面臃腫
- 比方說，我們在同一個頁面上有多個這樣的實例。我們可能會得到這樣的 performance profile

![](https://i0.wp.com/css-tricks.com/wp-content/uploads/2020/05/rgnzixsg.png)  

對網路狀況好的 User，可能影響不大，但如果網路不好，這效果可能就不好  

我們可以做的些事情來平衡:
- 下載一個單一的 fallback image，而不是一次就下載整個圖片序列
- 為某些設備準備使用較小圖片序列
- 允許用戶啟用序列，也許用一個按鈕來啟動和停止序列

Apple 採用了第一個選項
- 如果在連接到 `slow 3G` 的 mobile device 上開啟 [AirPods Pro頁面](https://www.apple.com/airpods-pro/)
- 性能統計數字開始看起來好了很多

![](https://i0.wp.com/css-tricks.com/wp-content/uploads/2020/05/kJPcmdlA.png)  


仍然是一個沉重的頁面
- 但比我們在沒有任何性能考慮的情況下得到的東西要輕得多
- 這就是為什麼 Apple 能夠在一個頁面上得到這麼多複雜的序列

---------------------------------------

## 8. Further reading
如果對這些圖片序列是如何產生的感興趣
- 可以看看 AirBnB 的 [Lottie庫]（https://airbnb.design/lottie/）
- 了解用 After Effects 生成動畫的基本知識
- 同時提供了一種簡單方法

---------------------------------------

## 9. React source code
```js
import { useEffect } from 'react'

const frameCount = 147
const currentFrame = index => (
  `https://www.apple.com/105/media/us/airpods-pro/2019/1299e2f5_9206_4470_b28e_08307a42f19b/anim/sequence/large/01-hero-lightpass/${index.toString().padStart(4, '0')}.jpg`
)
const images = []

function App() {

  // preload all images
  useEffect(() => {
    const preloadImage = () => {
      for (let index = 1; index < frameCount; index++) {
        images[index] = new Image();
        images[index].src = currentFrame(index)
      }
    }
    preloadImage()
  }, [])

  useEffect(() => {
    const canvase = document.getElementById("hero-lightpass")
    const context = canvase.getContext("2d")

    canvase.width = 1158;
    canvase.height = 770;

    const img = new Image();

    // load first image
    img.src = currentFrame(1)
    img.onload = function () {
      context.drawImage(img, 0, 0);
    }

    // sync image with scroll position function
    function syncImage() {
      const scrollTop = document.documentElement.scrollTop || document.body.scrollTop
      const maxScrollTop = document.documentElement.scrollHeight - window.innerHeight;
      const scrollFraction = scrollTop / maxScrollTop
      const frameIndex = Math.min(
        frameCount - 1,
        Math.floor(scrollFraction * frameCount)
      )

      requestAnimationFrame(() => {
        context.drawImage(images[frameIndex + 1], 0, 0)
      })
    }

    window.addEventListener('scroll', syncImage);

    return () => {
      window.removeEventListener('scroll', syncImage)
    }
  }, [])


  return (
    <div>
      <canvas id="hero-lightpass" />
    </div>
  )
}

export default App

```