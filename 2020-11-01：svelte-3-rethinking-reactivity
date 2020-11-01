# Svelte3 rethinking reactivity
## Rich_Harris Mon Apr 22 2019
### https://svelte.dev/blog/svelte-3-rethinking-reactivity

這篇文章內容就是在聲明，modern framework 太 overhead 了，framework 的部分有機會在 build time 實現，不需要拉到 run time。**建議看影片**，影片有些 demo

Rich Harris 對此主題有 30mins 的 talk
- https://www.youtube.com/watch?&v=AdNJ3fydeao
- **建議看影片**，影片有些 demo


## What is Svelte?
- Svelte is a component framework
- 跟 React or Vue 相似，但有一點決定性的不同

傳統框架允許寫聲明式的狀態驅動 code (declarative state-driven code) 
- 但要付出代價： browser 必須做更多的工作來 convert those declarative structures into DOM operations
- 技術上就是 [Virtual DOM diff](https://svelte.dev/blog/virtual-dom-is-pure-overhead)
- 這些都吃掉你的 frame budget 和增加 GC 的負擔

跟 react, vue 不一樣
- **Svelte runs at build time**
- 把 code 轉換成 imperative code 來 update DOM
- 沒有 run time 的 framework (Virtual DOM diff)、負擔小，就能有更好的 performance
 
## Svelte 第一版的時候
- 目標是測試一個假說 [Frameworks without the framework: why didn't we think of this sooner?](https://svelte.dev/blog/frameworks-without-the-framework)
- 是一個 compiler，能產生好的 code，從而提供出色給 developer 好的 develop experience
  - (第二版只是升級一些東西)

## Svelte3
- 寫 components 不需要這麼多 boilerplate ([significantly less boilerplate](https://svelte.dev/blog/write-less-code))
- 玩tutorial 體驗看看 -> https://svelte.dev/tutorial/basics

要做到 **significantly less boilerplate** 這點
- needed to rethink the concept at **the heart of modern UI frameworks: reactivity**
- (上面的影片連結就是在談這個)


## Moving reactivity into the language
舊版的 Svelte，需要告訴電腦某些 state 改變了，所以需要呼叫 `this.set`
```js
const { count } = this.get();
this.set({
  count: count + 1
});
```
（這跟 react 幾乎一樣。概念相同，但技術上不一樣。影片有說明。）  
 
hooks 的出現，改變了一切
- 許多框架開始嘗試實現自己的 hooks
- 但是我們很快得出結論，這不是我們想要的方向
- hooks 具有一些吸引人的屬性，但是它們還包含一些不自然的代碼，並為 GC 帶來了不必要的工作
- 對於嵌入式設備以及繁重的動畫交互中使用的框架，都是不好的。

重新思考過後，認為
- 最好的 API 根本就不是 API
- We can just use the language

Updating some count value — and all the things that depend on it — should be as simple as this

```js
count += 1;
```

因為 Svelte 是 compiler，因此可以在背後來實現成這樣：

```js
count += 1; $$invalidate('count', count);
```

重要的是，我們可以做到所有這些(單靠這樣就能 update value)，而無需使用 `proxies` or `accessors` 的開銷和復雜性。It's just a variable.
 
