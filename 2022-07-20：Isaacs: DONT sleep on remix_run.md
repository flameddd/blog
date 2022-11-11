# Isaacs: DONT sleep on remin_run
## 2022/04/23
### https://twitter.com/izs/status/1517756753378643968

Isaacs 是 npm 的 creator，他在 twitter 上大力推薦 [remix_run](https://remix.run/)  

他說
- 雖然他才剛開始碰，但
- 上一次讓他有「就是這個，這就是如何在 JS 中，最好的方法來做好某件事」的感覺的是 `NodeJS`
- 整體上來講，react 是他最不喜歡的部分
  - 他說他明白 react 用 state-driven 方法來更新 UX，這讓 CSR and SSR 都能用相同的 code
  - 這種做法確實令人印象深刻，但這絕對不是有趣的東西
- 長久以來，他在「用 JavaScript 做 facny websites」領域裏面，remix 是他看到第一個認為，用了適當的方法來 SSR
- `Nested routes` 是天才之舉，回到 PHP/ASP 的方法
  - 每一個 URL path 都有一個 file，並且它處理該 URL 的所有請求
  - 使用 Remix，每個 router handlers 為 GET/HEAD 提供一個 function，為其他方法提供另一個函數
- Loader (GET/HEAD) functions stack, Action functions 是第一個獲勝的地方
  - 你能決定整個網站的某些區域能存取某些特定 data，你只需要 hooks 到 trees 中特定的位置就好
- 但是！`route` 也可以是 page 裡的 `div` 的 content，另外，這些相同的 function 也能轉為 react，能跟它互動
- 所以，default 情況下
  - remix 就是純粹的 HTML，不需要 JS
  - 然後另外用 JS 來 fetch 少量必要的資料來 respond 任意 navigation, and update
- 這是 killer feature: https://remix.run/docs/en/v1/guides/disabling-javascript 
- 缺點是，仍然使用 client side bundling，畢竟是 react
  - 另外有完整支援 TS，所以這還是能接受的事情
  - 但，實在太多 code 要轉換了
- Discord 上面非常活躍
- 他說他不是 react 的粉絲，transpilation 永遠是可怕的
  - 關於 transpilation 的議題需要一系列文章 or 一整夜加上酒才能好好的抱怨出來
  - tldr，它總是驚人的讓 `要運行的 code` 偏離 `我們寫的 code`，而且總是混淆
    - 他已經記不清楚有多少次他必須去打開所謂的 `./build/_chunk-JDHSOFJ.js` 之類的，然後找到 3148 lines
    - 去找出這些 machine generated code 去弄清楚怎麼回事
- "But source maps!" shout a crowd of people who'd try to save a drowning man by spraying him with a firehose.
- I've been waiting a long time for a react framework that could do this. But I'm guessing react will still be good a year from now. It's worth taking a close look at if only for some of the smart approaches to things, whatever replaces it will likely use similar tricks.


另外 remix 官網有寫一篇詳細的比較 remix vs next
- https://remix.run/blog/remix-vs-next
