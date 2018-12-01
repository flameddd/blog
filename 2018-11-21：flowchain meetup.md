## flowchain meetup
看來今天還是 Jollan 講東西
先想想我自己先想想我自己
如果要做 stream 的話，單單 ipfs 也可以實現吧？
flowchain + ipfs 的結合來做 stream 有什麼優勢嗎？獎勵機制？

flowchain 號稱 結合 ipfs 做 stream 很好？
事實是這樣子，哪

怎麼定義成熟 的區塊鏈
John 說還沒有
但很多概念已經實現了，正在概念驗證的階段。

分散式
跟 
去中心化
這兩個定義是不同的  天差地遠
上次有講過，但我忘了
p2p 理想中的網路效能是要 5G

## demo
IPFS
HTTP Live Streaming 做視訊串流
FFMPEG 檔案格式轉換吧
flowchain

http 206 做 stream 的技術 partial content 漸進式下載
這不是真正的串流！
這個一定會有延遲 206 這技術

1. 一定會有延遲
2. 安全性、資料隱私問題 （and 資料所有權）

http 206 攔截超簡單。 ！？有辦法？

去中心化的網路 比較沒有 broker、fowarder （資訊橋接者）  
分散式的才比較有  
broker A 跟 broker B 是不需要同步的。  
搜尋時，就去問所有 broker 有沒有這個 hash。  
資料給 brokder ，hash 是 broker 做的。

步驟是
1. 先上 flowchain 做驗證
2. 上 ipfs
3. 給 user 資料

現在的 stream 都是用推送的
而且是輪詢的，所以後面的人都會 delay

電商 + ipfs ！？
IPFS 的容易被攻擊的是 假造攻擊、假身份。
中間人攻擊等等。
flowchain 能防護這些問題
Merkle trees / DAG  <== 第一次聽到 DAG

DAG 其實問題很多，所以落地應該很難。

今天有一個重點
這東西我自己搞得定吧？
1. ipfs daemon
2. flowchain-ledger (邊緣運算最主要的節點 flowchain.co/mining.html)
3. 要先起 computing pool ?
(flcProxy.js) 
4. 開 ipfs 節點？ （那步驟一是？）

挖礦要產生 Lambda 跟 Puzzle

做一個 ipfs 當作自己的雲端硬碟？

混合鏈一定會討論安全性問題。我怎麼確認外面的私鏈是安全的。  

公鏈在 Ethereum Classic 以太經典

flowchain 總共有 3 種類型的節點。

驗證要多快、ipfs 就需要多快。  
視訊，是個 不停的 real time 的 transition 這是個問題  
一個大檔案要做處理都很容易。  

linux 有個限制是 同時能開多少檔案。

下個網路是什麼時代
stream 時代
還是維持
檔案時代

現在網路的儲存成本越來越高了？
這方案的儲存成本降低了？ 想不通？

flowchain 最重要是可以做驗證。

接到公有鏈 就是信任

一個無限空間的資料庫！聽起來很爽
有七個
51% 攻擊
BCH 最近要有 51%攻擊

上鏈 能驗證 影音資料！安全性問題  
聽他講 Merkle tree  
DAO 被偷錢？  
flowchain 跟 filecoin 有什麼關係？  
Jollan 認為上公鏈後 團隊應該要 24h 監控，而且有問題應該 30min 內 patch。

hash 的 function 選擇非常重要
大部分區塊鏈在設計的時候 都是用同一種 hash 方式
也有開始是用不同的 hash 方式。

sha256 可以用 browser 處理很有效率。
webgl 做 sha256 效果很好。

ethereum 有幾種 node ?

IOT 最大的問題就是 信任驗證
IOT 的世界裡面 Pirvate key 的方式已經公認很沒效率的
（ 因為IOT沒有RAM、CPU ）
PPKI
會有個數字的參數 (Lambda)  Lambda 要符合三個要素 真正亂數 用完就不能再用的 magic
公用鏈真正目的是 Lambda 計算

shareshift.io

coinmarket cap
他的 roadmap 很清楚
12月礦區
明年 企業 NAS
接著轉換 erc20 => Dextoken
上交易所

argon2 目前最強的 memory hard 算法


一個 ipfsDriver ?

簡單來說，就是要記住所有的 ipfshash 吧？
然後抓下來用？
能夠 update 嗎？
能夠 update 才是最困難的吧？
我能先做什麼？
先把 ui 簡單弄出來？


流程是？
我 load 不到 ipfs 吧？
所謂的 object 是？  
https://github.com/ipfs/faq/issues/163

進去很簡單，左邊 list 右邊 ipfs ？
ui 方面可以參考 googledrive onedrive megadirve 這三套應該很夠了
另外常駐功能大概很難測
功能實現！
紀錄著 hash table 甚至要 備份起來
DAG ?
上傳 ipfs
下載 ipfs
預覽應該很難

上傳 要有 progress bar
下載要有 progress

我之前碰的 demo 是透過 區塊鏈
但這個專案其實不需要吧？

dnd 功能當然也是要的。

update 這個怎麼判斷？

landing page gateby aws 12 factors
ipfsdriver
東西越來越多

這些都沒有 business sence

付款管道