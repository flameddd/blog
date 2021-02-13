# HTTP 203: JavaScript counters the hard way
## Surma and Jake
### https://twitter.com/DasSurma
### https://twitter.com/jaffathecake
### https://www.youtube.com/watch?v=MCi6AZMkxcU
#### Jake's The optimal reusable solution: https://gist.github.com/jakearchibald/cb03f15670817001b1157e62a076fe95

HTTP 203 來解釋各種 counter 實作的缺點  
code: https://github.com/flameddd/http203-counter

## ex1
- :x: Accurate over time
- :heavy_check_mark: Updates visually steadily (not frame perfect, but good enough)
- :x: Avoids running in background (keep update DOM, so using CUP when don't have to)
- :heavy_check_mark: good CPU usage

```js
const intervalTime = 1000;
let seconds = 0;

setInterval(() => {
  seconds = seconds + 1;
}, intervalTime)
```

`setInterval` 跟 `setTimeouts` 都是 schedule task 到 `event queue`  
換句話說，如果 `main thread` 有 task 阻塞的話，這 task 就不會 run 了  
counter 就會有 drift 的問題  

safari best case 情況下，每次 call 都會 drift 約 millisecond  
250 秒左右時，就能看到誤差有 1 秒了  

我問了 Jake，為什麼 safari 會這樣

> Jake: Because it queues the next callback once the previous callback is complete
( https://twitter.com/jaffathecake/status/1359217154323841026 )

另外，**如果當下 browser tab 在 backgournd 的話，drift 會更嚴重**！  <-- 非常嚴重 
- https://developer.chrome.com/blog/timer-throttling-in-chrome-88/

## ex2
跟上面很相似，但這個會隨時間過去，仍然顯示正確  
- :heavy_check_mark: Accurate over time

但還是沒有到 **frame perfect** 的程度  

```js
const intervalTime = 1000;
let seconds = 0;

// ==== ex2 ====
const start = Date.now()
setInterval(() => {
  const elapsed = Date.now() -start
  seconds = Math.floor(elapsed / intervalTime)
},intervalTime)
```

此外，這些做法可以加一些 code 更省效能  

```js
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    clearInterval(id);
  } else {
    frame = setInterval(frame, intervalTime)
  }
})
```


## ex3
這個方法的好處是，animation 會在 page hidden 時候暫停，自動就有上面 `visibilitychange` 判斷的功能  

但是
- 984 ms CPU time 是浪費的，UI 根本沒有 update (59 frame time 的執行是沒意義的)

所以是
- :heavy_check_mark: Accurate over time
- :heavy_check_mark: Updates visually steadily (not frame perfect, but good enough)
- :heavy_check_mark: Avoids running in background (keep update DOM, so using CUP when don't have to)
- :x: good CPU usage

所以這方法也很耗電、可能讓你電腦風扇開始叫  

```js
const start = Date.now()
function frame() {
  const elapsed = Date.now() - start;
  seconds = Math.floor(elapsed / intervalTime);
  requestAnimationFrame(frame)
}
frame()
```

## ex4
跟上面類似的概念，但不需要每 frame 執行  
但，實際上，還是用到了 animation，所以 CPU 使用率方面，其實跟 `requestAnimationFrame` 是幾乎一樣的

- :heavy_check_mark: Accurate over time
- :heavy_check_mark: Updates visually steadily (not frame perfect, but good enough)
- :heavy_check_mark: Avoids running in background (keep update DOM, so using CUP when don't have to)
- :x: good CPU usage

```js
// time of current frame when document been created
const start = document.timeline.currentTime;
function frame() {
  const elapsed = document.timeline.currentTime - start;
  seconds = Math.floor(elapsed / intervalTime);
  
  document.body.animate(null, {
    duration: intervalTime - elapsed % intervalTime
  }).onfinish = frame
}
frame()
```

## ex5
用純 css 的方式實現。一樣有 CPU 使用率高的問題  
而且 safari not work

- :x: good CPU usage

這方法的結果讓我很驚訝，看這影片之前，我以為這會是好方法  
Jake 自己也說他這很驚訝這結果  

ex3, ex4 這兩種方法，Jake 認為 browser 應該有機會改善（甚至認為 ex3 的 CPU 這麼高，算是 bug 了）  
所以有開 issue 給各家 browser。影片下方有連結  

```html
<div>
  <span class="first"></div>
  <span class="second"></div>
</div>
```

```css
.first::before { 
  content:"0";
  animation: 100s linear infinite 5s backwards numbers;
}
.second::before {
  content:"0";
  animation: 10s linear infinite 0.5s backwards numbers;
}

@keyframes numbers {
  0% { content: '0';}
  10% { content: '1';}
  20% { content: '2';}
  30% { content: '3';}
  40% { content: '4';}
  50% { content: '5';}
  60% { content: '6';}
  70% { content: '7';}
  80% { content: '8';}
  90% { content: '9';}
}
```

## ex6
這招表現不錯，safari、Firefox 都好，但 Chrome 表現反而差  

Jake 說不知道這是正常情況，還是 bug  
看起來在 `setTimeout` 跟 `current frame time` 的 sync 方面，有些許落差  
導致沒有達到 frame perfect 的程度

```js
const start = document.timeline.currentTime;
function frame(time) {
  const elapsed = time - start;
  seconds = Math.floor(elapsed / intervalTime);
  
  setTimeout(
    () => requestAnimationFrame(frame),
    intervalTime - elapsed % intervalTime
  )
}
frame(start)
```

另外，這個例子有三個地方有著 `Math.floor` 處理/味道 
1. ` Math.floor(elapsed / intervalTime);` 
2. `setTimeout` 的第二個參數。如果你給它 `99.99`，實際上它會轉為 `99`，因為這參數是 second base 的
3. `elapsed % intervalTime` 這個是指邏輯上的不準確，這邊的邏輯應該算是給出「超過 1 秒多久」，而沒有給出「差多久到 1 秒」，有點微妙的差距。可能會拿到 0.99ms，但實際應該是 1ms  

## ex7
調整上面的做法，除此之外。
`setTimeout` 用了 `performance.now()` 來計算  
差別在哪？，原本的做法，我們給 `100` 那它就是 100 時執行  
但！不是在 100 的最後一個 frame 時執行  

`performance.now()` 的話，就是執行到 `setTimeout` 這邊時，我們才會拿到 current frame 的 time  
這樣更精準  
```js
const start = document.timeline.currentTime;
function frame(time) {
  const elapsed = time - start;
  seconds = Math.round(elapsed / intervalTime);
  const targetNext = seconds + (intervalTime / 1000) * 1000 + start;
  
  setTimeout(
    () => requestAnimationFrame(frame),
    targetNext - performance.now()
  )
}
frame(start)
```
