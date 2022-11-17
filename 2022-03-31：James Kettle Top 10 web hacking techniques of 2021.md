# 2022-03-31：Top 10 web hacking techniques of 2021
## James Kettle Published: 09 February 2022 at 13:59 UTCUpdated: 10 February 2022 at 15:20 UTC
### https://portswigger.net/research/top-10-web-hacking-techniques-of-2021

社群每年選出 top 10 hacking 技術，這篇我想很適合入門
- 歷年: https://portswigger.net/research/top-10-web-hacking-techniques

-------------------

今年有一個特別的主題佔據主導地位。在提名和最後的前十名中
- 我們看到了對 [HTTP Request Smuggling](https://portswigger.net/web-security/request-smuggling) 和對 `attacks on parser inconsistency` 的攻擊的高度關注


## 10 - Fuzzing for XSS via nested parsers
- XSS 似乎是老話題，但 `Fuzzing for XSS via nested parsers` 證明了，這還是高風險的問題
  - https://swarm.ptsecurity.com/fuzzing-for-xss-via-nested-parsers-condition/
- [@Psych0tr1a](https://twitter.com/Psych0tr1a) 展示了如何 stacked HTML sanitization rules against each other with specular results.

## 9 - 通過更高的 HTTP 版本進行 HTTP 走私 (Smuggling)
- [@Emil Lerner](https://twitter.com/emil_lerner) 的研究 [HTTP Smuggling via Higher HTTP Versions](https://www.slideshare.net/neexemil/http-request-smuggling-via-higher-http-versions)
  - 作者另一篇針對 http3 的 https://medium.com/@emil.lerner/leaking-uninitialized-memory-from-fastly-83327bcbee1f

這，看不太懂，slide 解釋各種不同版本 http 解析封包的差異，然後講解案例  
這要堆封包、http 協定摸熟到一定程度，才有可能挖  

## 8 - 實用的 HTTP Header Smuggling
- `CL.CL request smuggling`
- [Daniel Thatcher](https://twitter.com/_danielthatcher) - [Practical HTTP Header Smuggling: Sneaking Past Reverse Proxies to Attack AWS and Beyond](https://www.intruder.io/research/practical-http-header-smuggling)
- Daniel Thatcher 分離了 HTTP Request Smuggling 的 core component，並將其巧妙地重新組合成一種策略，可以識別 CL.CL 漏洞和通用隱藏頭攻擊

> 看不懂，看起來是不斷嘗試玩弄 `Content-Length`，藉由找機會繞過 API gate way 的檢查

## 7 - JSON Interoperability Vulnerabilities
- [Jake Miller](https://twitter.com/theBumbleSec) - [JSON Interoperability Vulnerabilities](https://bishopfox.com/blog/json-interoperability-vulnerabilities)
- 觸發 JSON 解析器不一致，使得有其機會找到攻擊點

為什麼會出現解析不一致？
- 官方和替代規格，即使在最好的情況下，也會出現與規範的細微的、無意的偏差
- 即使在官方 JSON RFC 中，也有關於一些主題的開放式指南，例如如何處理重複鍵和表示數字

Interpreter
- [IETF JSON RFC（8259 和更早版本）](https://tools.ietf.org/html/rfc8259)：官方的 Internet 工程任務組 (IETF) 規範。
- [ECMAScript](https://262.ecma-international.org/)：對 JSON 的更改與 RFC 版本同步發布，該標準參考 RFC然而，JavaScript interpreter 提供的非規範便利，例如無引號字符串和註釋
- [JSON5](https://json5.org/)：superset，添加了便利特性（例如，註釋、替代引號、無引號字符串、尾隨逗號）來擴充官方規範。
- HJSON： HJSON 在精神上與 JSON5 相似，但設計選擇不同。
- 還有非常多不同的 interpreter  

作者 把JSON 互操作性安全風險分為五類：
- 不一致的重複鍵優先級
- Key Collision：字符截斷和註釋
- JSON 序列化怪癖
- 浮點數和整數表示
- 許可解析和其他錯誤

> 作者都是設想被攻擊的可能性。雖然很小，但感覺確實有可能發生  

## 6 - 大規模 cache 中毒
- [Youstin](https://twitter.com/iustinBB) - [Cache Poisoning at Scale](https://youst.in/posts/cache-poisoning-at-scale/)
- Youstin 說明了 web cache poisoning 仍然是個問題，而且被廣泛地忽略了
  - 小小的 cache poisoning 累積起來，最後造成的破壞可能很大
  - Youstin 說如果不懂 `web cache poisoning`，強烈建議先讀篇 https://portswigger.net/research/practical-web-cache-poisoning

Apache Traffic Server (ATS)
- 是 caching HTTP proxy，Yahoo 和 Apple 都廣泛使用
- Apache Traffic Server 中的 URL 片段處理不正確 ( CVE-2021-27577 )
- 當發送到 ATS 的請求包含 url 片段時，ATS 會轉發它而不移除 fragment。根據RFC7230，ATS 轉發的請求是無效的，因為 origin-form 應該只由絕對路徑和查詢組成。
- 此外，ATS 通過主機、路徑和查詢來產生 cache key，忽略 url fragment。因此，這意味著下面的兩個請求將共享相同的 cache key
  - GET /index.html
  - GET /index.html#Fregment

GitHub CP-DoS
- Youstin: `Cache Poisoning vulnerabilities` 很大的主因是因為 `unkeyed headers`
- 它寫了工具去暴力掃描 unkeyed headers
- 發現 Github 將 Authentication cookie 包含在 cache key 中。可以刪除所有訪問它們的未經身份驗證的用戶的存儲庫，因為他們都共享相同的 cache 密鑰。獲得了 7500 鎂  

整篇文章，Youstin 找到超多大廠都有這樣的漏洞，真是驚人  
- cache Poisoning 感覺是個很好的主題！！  

## 5 - 隱藏的 OAuth 攻擊媒介
- [Michael Stepankin](https://twitter.com/artsploit) - [Hidden OAuth attack vectors](https://portswigger.net/research/hidden-oauth-attack-vectors)
- 深入研究了 OAuth 和 OpenID 規範，以發現隱藏的 endpoints 和設計缺陷

介紹三個全新的 OAuth2 和 OpenID Connect 漏洞
- Dynamic Client Registration: SSRF by design
- redirect_uri Session Poisoning
- WebFinger User Enumeration

通常使用 OAuth 的目標
- 極大可能也支持 OpenID，這大大擴展了可用的攻擊面
- 所以每當你測試 OAuth 時，都應該試試看 `/.well-known/openid-configuration` endpoints
  - e.x. `https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration`
  - 能獲取一些有用的訊息  
  
> 文章中針對每一項都有詳細描述的樣子，這，每一點都需要好幾天研究，才有機會搞東 = =||| 

## 4 - Exploiting Client-Side Prototype Pollution in the wild
- [s1r1us](https://twitter.com/S1r1u5_) - [A tale of making internet pollution free - Exploiting Client-Side Prototype Pollution in the wild](https://blog.s1r1us.ninja/research/PP)

文章很長，利用 JS Prototype Pollution 來達成 xss  
重點是這是一個團隊在做事，不是一個人，另外他們寫了很多工具了掃描  
第一次聽說 CodeQL，感覺可以試試看  
Prototype Pollution + xss 也是個我有機會的方向  





## 3 - MS Exchange 上新的攻擊點 (Orange Tsai 那個)
- [Orange Tsai](https://twitter.com/orange_8361) - [A New Attack Surface on MS Exchange](https://blog.orange.tw/2021/08/proxylogon-a-new-attack-surface-on-ms-exchange-part-1.html)

Exchange 為了相容性，做了很多努力，相對攻擊就有機可趁。他們選定這麼目標後，一直往下挖  
拿到超過 20萬鎂 ...  



## 2 - HTTP/2: The Sequel is Always Worse
- [James Kettle](https://twitter.com/albinowax) - [HTTP/2: The Sequel is Always Worse](https://portswigger.net/research/http2)
- 關於 HTTP2 如何極大地增加整個情況的複雜性的研究
- 由於 HTTP2 的使用仍在被採用， 對 HTTP (down)upgrade 而言，HTTP Request Smuggling 是永無止境的

也是一個關於 HTTP Request Smuggling 的主題


## 1 - Dependency Confusion
- [Alex Birsan](https://twitter.com/alxbrsn) - [Dependency Confusion](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610)

靠邀，前幾天剛好看了這篇，沒想到是我最能讀懂的一篇  --> [我的中文筆記](<./2022-03-30：Alex Birsan Dependency Confusion: How I Hacked Into Apple, Microsoft and Dozens of Other Companies.md>)  

