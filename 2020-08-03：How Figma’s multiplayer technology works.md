# How Figma’s multiplayer technology works
## Evan Wallace, CTO at Figma October 16, 2019
### https://www.figma.com/blog/how-figmas-multiplayer-technology-works/

有料的一篇，沒有細講怎麼實作，但把概念、方向都說出來了  
不詳細記錄了，簡單一些關鍵字，未來有需要再重讀  

Figma 實作線上線上多人編輯的概念  
- 出發點，以 web 為 base 的產品，沒有多人線上共同編輯，it just felt wrong
  - 否則 User 就必須 export, sync, or email copies of files 來溝通
  - engineer 就要 git push, pull

## operational transforms (OTs)
- Google Docs 就是用這個演算法來實現 multiplayer system
- Figma 覺得這架構太過複雜，不適合它們

> OTs are a great way of editing long text documents with low memory and performance overhead, but they are very complicated and hard to implement correctly. They result in a combinatorial explosion of possible states which is very difficult to reason about.

## WebSockets
- Figma 用 WebSocket 來溝通同步的動作
- 另外有一篇寫到他們 scaling WebSocket 的經驗 （切換成 rust，算是大推 rust）
  - https://www.figma.com/blog/rust-in-production-at-figma/
  - 前一個方案是 typescript （文章好像只提到這個）
    - 那代表是 nodejs base，所以文章中特別有說，前一個方案是 single thread
  - 另外一套知名的 uWebSockets (c++ base)
    - https://github.com/uNetworking/uWebSockets

## Conflict-free Replicated Data Types (CRDTs)
- Figma 是參考 CRDTs
- 是一組常用在 distributed systems 的資料結構
- 設計上是 for distributed systems，但 Figma 不是
  - There is some unavoidable performance and memory overhead with doing this
- Figma is centralized (server is the central authority)

所以 Figma 只是參考 CRDTs，不是實作它  

一些 CRDTs 的類別
- **Grow-only set**: This is a set of elements. The only type of update is to add something to the set. Adding something twice is a no-op, so you can determine the contents of the set by just applying all of the updates in any order.
- **Last-writer-wins register**: This is a container for a single value. Updates can be implemented as a new value, a timestamp, and a peer ID. You can determine the value of the register by just taking the value of the latest update (using the peer ID to break a tie).


文章後面就是多個例子，在不同情境下 Figma 怎麼同步這些行為、怎樣同步才合理