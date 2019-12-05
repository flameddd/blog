## jsconfjp (Javascript Conferences Japan) 2019

ref: https://jsconf.jp/2019/schedule/

### Day1 
#### 13:00-13:30 Opening talk
![JsConfJap2019_Tokyo_Day1_1](/assets/img/JsConfJap2019_Tokyo_Day1_1.jpg)  
![JsConfJap2019_Tokyo_Day1_3](/assets/img/JsConfJap2019_Tokyo_Day1_3.jpg)  

opening 就是介紹一些行程而已  
讓我意外的是，**第一件事情是講 harassment，代表 jsconf (japan) 非常非常重視這件事情**

#### 13:30-14:00 The State of JavaScript
- RoomA by (Raphaël Benitte and Sacha Greif)

搶先聽到 `State of JavaScript` 的目前統計結果，後續可能會有些變動  
不過也是聽聽 `Raphaël Benitte` 本人親口講講他的解讀  

![JsConfJap2019_Tokyo_Day1_6](/assets/img/JsConfJap2019_Tokyo_Day1_6.jpg)  

Raphaël Benitte 當時也是感到**混亂**，覺得怎麼 frontend 變得越來越複雜  
他對這篇 `How it feels to learn Javascript in 2016` 深有同感，所以他決定弄個 survey  
- https://hackernoon.com/how-it-feels-to-learn-javascript-in-2016-d3a717dd577f

![JsConfJap2019_Tokyo_Day1_7](/assets/img/JsConfJap2019_Tokyo_Day1_7.jpg)  

這三年，`typescript` 的發展驚人  
![JsConfJap2019_Tokyo_Day1_9](/assets/img/JsConfJap2019_Tokyo_Day1_9.jpg)  

`Elm` 我算是偶而會看到，`Reason` 好像是 FB 的  
關於 `Reason` 感覺現在 fb 主力偏向 `Relay`  
個人感覺 `Reason` 應該是沒有未來了  
- https://reasonml.github.io/

`Elm` 確實有看過一些人說，使是起來很滿意  
後面的 talk Mercari 的 Frontend architecture `Jorge Bucaran` 順手講解它的特性時，是有讓我產生興趣想寫寫看  
他說，`Elm` 是個完全 pure functional, declarative 的架構  

個人感覺未來還是不會紅，但玩看看，可能可以加強我對 `declarative language` 的感覺  
有想到時，在來玩個 demo 看看吧
- https://elm-lang.org/

![JsConfJap2019_Tokyo_Day1_10](/assets/img/JsConfJap2019_Tokyo_Day1_10.jpg)  

`react` 跟 `vuejs` 的使用度差這麼多？  
我是覺得結果還會變  
但他有說出，兩個的 `滿意度都非常高`（年度下面灰色圈圈的百分比數字）  
所以可以預期後續幾年還是會繼續熱門

![JsConfJap2019_Tokyo_Day1_11](/assets/img/JsConfJap2019_Tokyo_Day1_11.jpg)
![JsConfJap2019_Tokyo_Day1_12](/assets/img/JsConfJap2019_Tokyo_Day1_12.jpg)  

`Svelte` 才剛出，但可以看到 **滿意度** 很高  
![JsConfJap2019_Tokyo_Day1_13](/assets/img/JsConfJap2019_Tokyo_Day1_13.jpg)  

這邊提到一部分稱為 `infra` 的主題  
這是 `frontend infra` 的概念，這主題算是今天蠻吸引我的  
之前沒有好好思考這問題，後面有 JAMstack talk  
![JsConfJap2019_Tokyo_Day1_15](/assets/img/JsConfJap2019_Tokyo_Day1_15.jpg)  
![JsConfJap2019_Tokyo_Day1_17](/assets/img/JsConfJap2019_Tokyo_Day1_17.jpg)  

`netlify`, `zeit`, `gatsby` 這三間都是在做類似的事情（嗎？）  
可以 google 看看，這算是有意思的主題  
![JsConfJap2019_Tokyo_Day1_18](/assets/img/JsConfJap2019_Tokyo_Day1_18.jpg)  


#### 14:15-14:45 Visualizing Connections
- RoomA by (`Nadieh Bremer`)

`Nadieh Bremer` 真是神級的 `data visualization` 大神！  
這些呈現真的都神到爆  
跟 `Shirley Wu` 兩人之前是同事嗎？  
兩位都是 `data visualization` 大神等級
- Nadieh Bremer: https://www.visualcinnamon.com/portfolio/
- Shirley Wu: https://sxywu.com/

這 talk 講她多個 project 的構思過程  

![JsConfJap2019_Tokyo_Day1_20](/assets/img/JsConfJap2019_Tokyo_Day1_20.jpg)  

構思時，她常常會這樣畫下來幫助自己設計發想  
![JsConfJap2019_Tokyo_Day1_23](/assets/img/JsConfJap2019_Tokyo_Day1_23.jpg)  

我忘了這個 data 是表示什麼的了，但也是經過好幾次調整  
- 一開始是純碎的圓圈點代表資料
- 後來好像**加入顏色代表某些年代的屬性**（但資料點太多，還是很難看）
- 接著底下**加入時間軸來分群**才好些

![JsConfJap2019_Tokyo_Day1_24](/assets/img/JsConfJap2019_Tokyo_Day1_24.jpg)  

Royal Constellations
- 一個追朔歐洲皇室過去 1000 年的族譜專案
- 也是花了超多時間查 wiki 歷史資料
- 相信弄 data set 也是超多時間

link 點去玩玩吧
- http://www.datasketch.es/october/code/nadieh/
  - 用星宿的 style 呈現
  - hover 會呈現 6 代關聯的親人
  - onClick 任意兩個點，會呈現相關聯的線

![JsConfJap2019_Tokyo_Day1_31](/assets/img/JsConfJap2019_Tokyo_Day1_31.jpg)  

這是`真實網頁截圖` = = |||，不是 ps 喔

![Royal_Constellations_01](/assets/img/Royal_Constellations_01.jpg)  
![Royal_Constellations_02](/assets/img/Royal_Constellations_02.jpg)
![Royal_Constellations_03](/assets/img/Royal_Constellations_03.jpg)  

後面這個專案好像是關於 homeless people 的
- 也是花了大量時間清洗資料
- 清完資料後，先嘗試做各種資料視覺化的圖表看看
- 後續想呈現**隨時間流動的呈現** （一樣也是先用畫的想看看，slide 上有好幾張草圖）

link
- https://www.theguardian.com/us-news/ng-interactive/2017/dec/20/bussed-out-america-moves-homeless-people-country-study
  - 地圖流動的效果，要 scroll 到約 1/10 的地方才有

最後的效果...，這要做出來，真的要 d3 專業才行  

![JsConfJap2019_Tokyo_Day1_33](/assets/img/JsConfJap2019_Tokyo_Day1_33.jpg)  
![JsConfJap2019_Tokyo_Day1_34](/assets/img/JsConfJap2019_Tokyo_Day1_34.jpg)
![Bussed_out_01](/assets/img/Bussed_out_01.gif)  

Intangible Cultural Heritage （非物質文化遺產）
- 這專案的資料量非常複雜，**connection 超過 12,0000** (這麼大，要怎麼做 datavis)
- 光是現有資料變成 資料點，畫面就完全密密麻麻了，加上線表示 connection 就更嚴重
- 後續把某些 group 再一起變成 大資料點，但還是密密麻麻
- 在分群，變且用圓形的 layout 概念呈現

link
- https://ich.unesco.org/en/dive&display=biome#tabs
  - 這麼大量的 data，最後效果真的很好

![JsConfJap2019_Tokyo_Day1_36](/assets/img/JsConfJap2019_Tokyo_Day1_36.jpg)  
![JsConfJap2019_Tokyo_Day1_38](/assets/img/JsConfJap2019_Tokyo_Day1_38.jpg)
![JsConfJap2019_Tokyo_Day1_39](/assets/img/JsConfJap2019_Tokyo_Day1_39.jpg)  
![IntangibleCulturalHeritage_01](/assets/img/Intangible_Cultural_Heritage_01.jpg)  
![IntangibleCulturalHeritage_02](/assets/img/Intangible_Cultural_Heritage_02.jpg)  

接著是一個關於貓的專案
- 她好奇網路上大家都問些什麼關於貓的問題
- 千千百種問題，她也是花一堆時間整理、歸類問題
- 還有良好的 mobile (RWD) 呈現，可以在變態一點嗎 = =||| （上面的專案不知道有沒有支援)

link
- 設計過程: https://www.visualcinnamon.com/2019/04/designing-google-cats-and-dogs
  - 設計過程簡直天書....
- 專案連結: https://whydocatsanddogs.com/cats



![JsConfJap2019_Tokyo_Day1_43](/assets/img/JsConfJap2019_Tokyo_Day1_43.jpg)
![JsConfJap2019_Tokyo_Day1_45](/assets/img/JsConfJap2019_Tokyo_Day1_45.jpg)  

`實際畫面截圖`:  

![why_do_cats_and_dogs_phone_detail_3](https://d33wubrfki0l68.cloudfront.net/a11c2b16fa7980189e27c8860f6ff1a2691947d2/44e60/img/portfolio/2019/why-do-cats-and-dogs/why_do_cats_and_dogs_phone_detail_3.png)  
![why_do_cats_and_dogs_detail_3](https://d33wubrfki0l68.cloudfront.net/eb08f9da6fa4c5219229e778a8bb1fec6d855520/f7c23/img/portfolio/2019/why-do-cats-and-dogs/why_do_cats_and_dogs_detail_3.png)  

#### 14:45-15:15 Defining Open Source
- `Henry Zhu` 現在 `babeljs` 其中 main contributor
- Open Source 的定義說了一些，但他說他其實也沒有答案
- 投入 OS 大概只是因為 fun, cool
- 有寫一篇他的心路歷程 why left job, and embrace Open source
  - https://www.henryzoo.com/in-pursuit-of-open-source-part-1/
  - https://www.henryzoo.com/in-pursuit-of-open-source-part-2/
  - 改天用 google 翻譯快速看看吧
- 有一個 podcast 他聊他對 Open Source 的想法 https://hopeinsource.com/
- 提到 maintainer III 這個社群，說 maintainer 其實容易被忽略，該怎麼應對之類的
- 提一本書 How to talk about books you haven't read
- 提到 The Backwards Brain Bicycle - Smarter Every Day 133
  - https://www.youtube.com/watch?v=MFzDaBzBlL0 非常有意思的影片
- 聊到投入 open Source 某些層面上對他來說是 freedom

![JsConfJap2019_Tokyo_Day1_53](/assets/img/JsConfJap2019_Tokyo_Day1_53.jpg)  

忘了他為什麼說這句話了
- 好像是指他要走出 code，來跟 user、社群 互動

![JsConfJap2019_Tokyo_Day1_56](/assets/img/JsConfJap2019_Tokyo_Day1_56.jpg)  


#### 15:30-16:00 Building and Deploying for the Modern Web with JAMstack
- RoomA by (Guillermo Rauch)
- `Next.js` 創辦人之一的樣子
- 讓我想起來，今年 `google firebase summit` 有類似的主題
  - Architecting Mobile Web Apps (Google I/O'19)
  - https://www.youtube.com/watch?v=NwY6jkohseg
- 其中一個精神，就上讓 developer 能更快速的上線系統  
- `modern web architecture` 這個值得當作一個主題來更深入思考一下  
- `netlify`, `zeit`, `gatsby` 應該都是這個方向
- `gatsby` 的那三篇 blog 要去看看
  - https://www.gatsbyjs.org/blog/2018-10-04-journey-to-the-content-mesh/
  - https://www.gatsbyjs.org/blog/2018-10-10-unbundling-of-the-cms/
  - https://www.gatsbyjs.org/blog/2018-10-11-rise-of-modern-web-development/
- 這邊的 `chrome collab` 是什麼意思？
- plugin 指的是，已經有套件可以快速整併其他功能了
  - e.x. payment (stripe), fulltext search

他們稱為 `JAMstack` 架構
- frontend 就 static site，部署上去 serverless (firebase host 這樣應該也算)
- api 就 serverless
- `markup` 是我最不太懂的概念，請起來就是剛進入頁面，沒 data 時，要有 holder 在上面
  - 為什麼這要稱為所謂的 serverless 呢？

後面再說，現在主流服務的某些 feature 都可以透過第三方來提供  
e.x. firebase, Auth0, firebase Auth, stripe  
developer 就不需要花時間開發了，直接串吧  

後面條列出他認為的 static site 優點
- 其中 security 方面的思維有點特別
  - 以前你要顧 frontend, backend 的兩邊 security，但現在你只需要顧 frontend
  - 因為都變 serverless 了，你不需要顧（或者說，交給第三方顧安全）

列舉 static site 的缺點  

pre page render 這邊是在講
- 你可以先取到某些資料，他這邊用 auth 舉例
- 第一次載入資料，你就已經完成 auth、有 auth 資料
- `getInitialProps` 是 `next.js` 的功能，也就是他們倡導的，第一次打頁面時，就一併完成第一次 api 請求。 `next.js` 視這個功能為主要改善 `CSR` 的關鍵
  - 有興趣可以去看看 `next.js` 的 github `README`，最下面有寫 `why create next.js`，可以體會一下他們的追求與精神（他們說，create `next.js` 這樣的 SSR 主要目的是為了改散 CSR 的效能）

go live 跟相關提到的
- 由 code review 轉變為一個類似 release review 這概念我不是很懂
  - 好像是說，因為部署輕鬆，所以部署上去直接看結果？

![JsConfJap2019_Tokyo_Day1_65](/assets/img/JsConfJap2019_Tokyo_Day1_65.jpg)  
![JsConfJap2019_Tokyo_Day1_68](/assets/img/JsConfJap2019_Tokyo_Day1_68.jpg)  
![JsConfJap2019_Tokyo_Day1_71](/assets/img/JsConfJap2019_Tokyo_Day1_71.jpg)  
![JsConfJap2019_Tokyo_Day1_73](/assets/img/JsConfJap2019_Tokyo_Day1_73.jpg)
![JsConfJap2019_Tokyo_Day1_74](/assets/img/JsConfJap2019_Tokyo_Day1_74.jpg)  
![JsConfJap2019_Tokyo_Day1_77](/assets/img/JsConfJap2019_Tokyo_Day1_77.jpg)  
![JsConfJap2019_Tokyo_Day1_78](/assets/img/JsConfJap2019_Tokyo_Day1_78.jpg)  

#### 16:00-16:30 Write What Not How
- RoomA by `Jorge Bucaran`
- Mercari 的 Frontend architecture
- 整場都講他對 `declarative language` 的想法
- 先花很多時間舉例 `declarative` 跟 `imperative`
- 其中提到一個關於 functional programing (FP) 的描述我蠻喜歡的
  - FP 就是 math，你不會有聽到有人說 or 嫌 math 不夠好，所以不應該用
  - 所以他就覺得，網路上有人在那邊戰 FP 的這些方面的人有莫名上妙，它就是 math 表示而已
- 這邊提到 `elm` 是 whole declarative language 讓我對 `elm` 有點興趣玩個 demo
- FP 其中一個重點是 **no side effect**，但其實實務上 JS 不可能不靠 side effect 來完成開發

整場都是在聊語言設計
- 我個人確實用了類似 react, redux 這樣的 `declarative` 就回不去了
- 這場給我的感觸比較低，比較沒重心關注 or 思考這方面思維

![JsConfJap2019_Tokyo_Day1_79](/assets/img/JsConfJap2019_Tokyo_Day1_79.jpg)  
![JsConfJap2019_Tokyo_Day1_80](/assets/img/JsConfJap2019_Tokyo_Day1_80.jpg)  
![JsConfJap2019_Tokyo_Day1_81](/assets/img/JsConfJap2019_Tokyo_Day1_81.jpg)  

#### 16:45-17:15 Deno - A new way to JavaScript
- RoomA by Kitson Kelly
- `Kitson Kelly` 的經歷也是 js 高手大神了，ts core contributor

link
- slide: https://docs.google.com/presentation/d/1HtVc8fepsX6CoKqldGj7uXf5Fp7N4PcThqfiXGRxVAE/edit#slide=id.p

這場
- 說說 why create DENO -> nodejs 是舊設計了，有很多包袱
  - 前陣子才看 nodejs team 的 talk，才知道 **nodejs 的 source code 裡面還是用 callback 來處理 async**，主因是 **promise** 是針對 browser 環境的樣子，他們幾乎沒辦法使用在 nodejs 上（但已經有努力，而且有進展了）
- secure by default 這個之前我自己有看過一些
  - 但我好奇，這個機制，在 nodejs 不能加嗎？
  - 10 things regred 上，security 是其中一件事情，所以 deno 要改善這問題
- 第三方 library 改分散式了，非集中化模式，跟 golang 一樣吧？
- 不會支援所有 web 的 api
- 另外讓我驚艷的是，**可以直接把第三方 library bundle 起來到 local，這樣就能直接引用 local 的 file 了**
- 針對 v8 有改善對 typescript 的效能（說是利用 v8 對 ts 的 snapshot），讓載入速度更快（應該是指 develop 的時候）
- roadmap 1.0 快了！

![JsConfJap2019_Tokyo_Day1_82](/assets/img/JsConfJap2019_Tokyo_Day1_82.jpg)  
![JsConfJap2019_Tokyo_Day1_83](/assets/img/JsConfJap2019_Tokyo_Day1_83.jpg)  
![JsConfJap2019_Tokyo_Day1_85](/assets/img/JsConfJap2019_Tokyo_Day1_85.jpg)  
![JsConfJap2019_Tokyo_Day1_87](/assets/img/JsConfJap2019_Tokyo_Day1_87.jpg)  
![JsConfJap2019_Tokyo_Day1_88](/assets/img/JsConfJap2019_Tokyo_Day1_88.jpg)  

#### 17:15-17:45 Web Accessibility in a Nutshell
- RoomC by Nazanin Delam (`netflix` 的人)

slide
- https://www.canva.com/design/DADnrZSkJec/OnMFRrcsA2ybW0zzI3-IqA/view?utm_content=DADnrZSkJec&utm_campaign=designshare&utm_medium=link&utm_source=publishsharelink

why a11y
- 她做些研究，全球約 `15%` 的人有些不方便 -> 這等於 `10E 人` -> 這還是值得做
  - 但我個人而言，產品要到什麼規模才開始要弄這些呢？ a11y 一定不是主要 feature
- 提了 `NEF` 三項原則 (No Extra effort)
  - 第一是 semantic tag
    - 這個我也看過一些文章說明重要性，但我還是對 when? where? use which tag 還是沒概念
    - 不過記得一件事情 `div`, `span` 都是沒意義的 tag
  - 第二提 color 的對比程度比例設計（也是有看過一些文章，有的網站可以告訴你哪些相對對比顏色）
    - `color-adjust: exact` 說這是未來 browser 會支援的功能，browser 直接幫你決定好的對比色
    - devtool `lighthouse` 好像也會檢查
  - 第三是 a11y tree，可以透過 dev tool 來看、幫助開發 a11y tree
- 提了 `MEF` 原則 (Minimal Effort A11y Steps) 這邊講的東西我不太記得了
  - no mouse -> 關掉 mouse 功能一天看看，來玩你的 App 就知道程度
    - 測試你的 interactive 元件、有沒有 skip link(例如你 header 有 30 個選項，應該要 skip 掉一些 interactive 功能，不然 user 會一直切 tab 切到手軟)
  - keyboard navigation、Meaningful Announcements
  - 包容的體驗

![JsConfJap2019_Tokyo_Day1_93](/assets/img/JsConfJap2019_Tokyo_Day1_93.jpg)  
![JsConfJap2019_Tokyo_Day1_94](/assets/img/JsConfJap2019_Tokyo_Day1_94.jpg)  
![semantic_01](/assets/img/semantic_01.jpg)  
![semantic_02](/assets/img/semantic_02.jpg)  
![JsConfJap2019_Tokyo_Day1_99](/assets/img/JsConfJap2019_Tokyo_Day1_99.jpg)  
![JsConfJap2019_Tokyo_Day1_96](/assets/img/JsConfJap2019_Tokyo_Day1_96.jpg)  
![JsConfJap2019_Tokyo_Day1_97](/assets/img/JsConfJap2019_Tokyo_Day1_97.jpg)  
![a11y_tree](/assets/img/a11y_tree.jpg)  
![JsConfJap2019_Tokyo_Day1_102](/assets/img/JsConfJap2019_Tokyo_Day1_102.jpg)  

#### 18:15-18:45 Sponsor talk (mediba, Mercari, Cybozu, CureApp)
- Mercari frontend team 有 350 人！哇塞，
  - microservice 轉變期，It's work (on fire) 中，所以還繼續在徵人
- Cybozu 沒聽過，但說台灣有辦公室，有空可以查一下

#### 18:45-18:55 Predictive Prefetching for improved performance
主題聊 JS 是 site loading 最重的東西
- 基本解法是 code splite
- lazy load by router

更近一步的招式 `prefetch`
- 這 talk 主軸是，通常的 `prefetch` 都是人為去設定，哪個檔案先做
  - 但這樣其實不準，也有可能人為誤差

所以這邊教一招
1. 利用 `GA` 去了解哪個 router 的使用率最高（你就知道哪個 bundle 更高機率被使用
2. 利用 `Guess.js` 去決定
  -  之前看過 `Guess.js`，但一直不知道 user case 用在哪邊，這應該算是第一次看到實際 case

所以概念是，利用 `GA` 來收集資料給 `Guess.js`，搭配 `webpack` 來處理  
這樣自動做到
- prefetch，但是是自動設定哪個 bundle 要先去 prefetch !

Cool 這應該是我第一次聽到 `prefetch` 的新玩法


![JsConfJap2019_Tokyo_Day1_104](/assets/img/JsConfJap2019_Tokyo_Day1_104.jpg)
![JsConfJap2019_Tokyo_Day1_105](/assets/img/JsConfJap2019_Tokyo_Day1_105.jpg)  
![JsConfJap2019_Tokyo_Day1_106](/assets/img/JsConfJap2019_Tokyo_Day1_106.jpg)  
![JsConfJap2019_Tokyo_Day1_109](/assets/img/JsConfJap2019_Tokyo_Day1_109.jpg)  
![JsConfJap2019_Tokyo_Day1_110](/assets/img/JsConfJap2019_Tokyo_Day1_110.jpg)  
![JsConfJap2019_Tokyo_Day1_112](/assets/img/JsConfJap2019_Tokyo_Day1_112.jpg)  
![JsConfJap2019_Tokyo_Day1_114](/assets/img/JsConfJap2019_Tokyo_Day1_114.jpg)  
![JsConfJap2019_Tokyo_Day1_115](/assets/img/JsConfJap2019_Tokyo_Day1_115.jpg)  
![JsConfJap2019_Tokyo_Day1_116](/assets/img/JsConfJap2019_Tokyo_Day1_116.jpg)  
![JsConfJap2019_Tokyo_Day1_117](/assets/img/JsConfJap2019_Tokyo_Day1_117.jpg)  
![JsConfJap2019_Tokyo_Day1_118](/assets/img/JsConfJap2019_Tokyo_Day1_118.jpg)  

#### 19:05-19:15 Cache Me If You Can by Maxi Ferreira
奇怪！他解釋的 `cache-control: no-cache` 怎麼跟我印象的不一樣！！？  
回去 google 看看  

真的耶，我記錯 `no-cache` 的意思了  
no-cache: 快取需存取，但是要重新驗證  
- 快取伺服器在把已儲存的複製版本傳給請求者之前，先會送一個請求給網頁伺服器做驗證。


這邊結論的最優解法應該就是
- 用 expire 然後，打包新資料時，用 hash file name
  - 這樣 cache 就不會過期
  - 然後 新資源也確定拿得到，也會有新的 cache


![JsConfJap2019_Tokyo_Day1_120](/assets/img/JsConfJap2019_Tokyo_Day1_120.jpg)  
![JsConfJap2019_Tokyo_Day1_126](/assets/img/JsConfJap2019_Tokyo_Day1_126.jpg)  

### Day2
  
#### 11:00-11:30 Time Is But an Illusion… in JavaScript
- RoomA by Jennifer Wong
- Mozilla 的人
- `recurrence rule (rrule)` 我第一次聽說這名詞，google 過後知道這算是一種專業的說法
  - 要設定某事情在某固定時間重複執行，如`每週一`，這就叫 `rrule`
- 她需要開發某功能，但現有的 `rrule` library 都沒人更新了
- `Brendan Eich` 我只有 10 天時間能開發 javascript，所以 js 的 date 我就直接抄 `java` 了
  - 這也是為什麼 js 的 month 的 index 跟 java 一樣，都是 `0 為 base`

問題比想像中複雜
- 有**時區**問題
- local 跟 ci 測試的機器不一樣，時區也不一樣，所以 test 要特別處理
- 還有**日光節約時間**

我感覺她最後沒有講太多她怎麼完成的
- 舉例了一些 native JS date 跟 momentjs 的 api 比較


![JsConfJap2019_Tokyo_Day2_1](/assets/img/JsConfJap2019_Tokyo_Day2_1.jpg)  

這就是 `rrule` 想實做的東西  
![JsConfJap2019_Tokyo_Day2_2](/assets/img/JsConfJap2019_Tokyo_Day2_2.jpg)  
![JsConfJap2019_Tokyo_Day2_4](/assets/img/JsConfJap2019_Tokyo_Day2_4.jpg)  
![JsConfJap2019_Tokyo_Day2_5](/assets/img/JsConfJap2019_Tokyo_Day2_5.jpg)  
![JsConfJap2019_Tokyo_Day2_6](/assets/img/JsConfJap2019_Tokyo_Day2_6.jpg)  
![JsConfJap2019_Tokyo_Day2_7](/assets/img/JsConfJap2019_Tokyo_Day2_7.jpg)  
![JsConfJap2019_Tokyo_Day2_9](/assets/img/JsConfJap2019_Tokyo_Day2_9.jpg)  
![JsConfJap2019_Tokyo_Day2_10](/assets/img/JsConfJap2019_Tokyo_Day2_10.jpg)  
![JsConfJap2019_Tokyo_Day2_11](/assets/img/JsConfJap2019_Tokyo_Day2_11.jpg)  
![JsConfJap2019_Tokyo_Day2_14](/assets/img/JsConfJap2019_Tokyo_Day2_14.jpg)  
![JsConfJap2019_Tokyo_Day2_15](/assets/img/JsConfJap2019_Tokyo_Day2_15.jpg)  
![JsConfJap2019_Tokyo_Day2_17](/assets/img/JsConfJap2019_Tokyo_Day2_17.jpg)  
![JsConfJap2019_Tokyo_Day2_18](/assets/img/JsConfJap2019_Tokyo_Day2_18.jpg)  
![JsConfJap2019_Tokyo_Day2_22](/assets/img/JsConfJap2019_Tokyo_Day2_22.jpg)  

#### 11:30-12:00 Life of Streams
- RoomA by Dominic Tarr
- 這位講者似乎對 nodejs stream 非常熟
- nodejs stream 相關的 code 約 2000 行
- 他自己寫一個 stream 的 library，約不到 500 行的樣子
- 他認爲更好，但他沒有想要取代 nodejs core stream 的意思
- 後來直接看 code，但我實在不熟這塊，看不懂... 所以也聽不懂

他有寫過一篇文章，有人引用的樣子，下面也有翻譯
Node.js Streams: Everything you need to know
- https://www.freecodecamp.org/news/node-js-streams-everything-you-need-to-know-c9141306be93/
- 翻譯: https://juejin.im/post/5ad05b7bf265da237e0a2308


![JsConfJap2019_Tokyo_Day2_26](/assets/img/JsConfJap2019_Tokyo_Day2_26.jpg)  
![JsConfJap2019_Tokyo_Day2_27](/assets/img/JsConfJap2019_Tokyo_Day2_27.jpg)  
![JsConfJap2019_Tokyo_Day2_28](/assets/img/JsConfJap2019_Tokyo_Day2_28.jpg)  
![JsConfJap2019_Tokyo_Day2_29](/assets/img/JsConfJap2019_Tokyo_Day2_29.jpg)  
![JsConfJap2019_Tokyo_Day2_30](/assets/img/JsConfJap2019_Tokyo_Day2_30.jpg)  
![JsConfJap2019_Tokyo_Day2_32](/assets/img/JsConfJap2019_Tokyo_Day2_32.jpg)  

#### 13:00-13:30 Passwords are so 1990
- RoomC by Sam Bellen
- 講者是 `Auth0` 的人
- 主要就是講 `WebAuthn` (1990.sambego.tech)
- 有趣的是，`LineDevDay` 跟 `jsconf` 都有這個 auth 的 talk
- 最常見的是搭配 **指紋** （但其實還是有人做 face, voice 辨識)
- 除了市面上的這些裝置，其實近 5 年的 apply 產品都有支援了（`touch ID`）
- API 其實很強大了，甚至可以指定，連線裝置是 藍芽的 才准使用驗證功能
- 但 `WebAuthn` 還是有缺點
  - 無法跨裝置（你要建立兩組的樣子）
  - Usr cred management 好像是說沒辦法多帳號切換管理之類的


![JsConfJap2019_Tokyo_Day2_47](/assets/img/JsConfJap2019_Tokyo_Day2_47.jpg)
![JsConfJap2019_Tokyo_Day2_48](/assets/img/JsConfJap2019_Tokyo_Day2_48.jpg)  
![JsConfJap2019_Tokyo_Day2_49](/assets/img/JsConfJap2019_Tokyo_Day2_49.jpg)
![JsConfJap2019_Tokyo_Day2_52](/assets/img/JsConfJap2019_Tokyo_Day2_52.jpg)  
![JsConfJap2019_Tokyo_Day2_53](/assets/img/JsConfJap2019_Tokyo_Day2_53.jpg)  
![JsConfJap2019_Tokyo_Day2_55](/assets/img/JsConfJap2019_Tokyo_Day2_55.jpg)  
![JsConfJap2019_Tokyo_Day2_56](/assets/img/JsConfJap2019_Tokyo_Day2_56.jpg)  
![JsConfJap2019_Tokyo_Day2_57](/assets/img/JsConfJap2019_Tokyo_Day2_57.jpg)  
![JsConfJap2019_Tokyo_Day2_58](/assets/img/JsConfJap2019_Tokyo_Day2_58.jpg)  
![JsConfJap2019_Tokyo_Day2_59](/assets/img/JsConfJap2019_Tokyo_Day2_59.jpg)  

```js

// register / create a new credential
navigator.credentials.create(createCredentialDefaultArgs)
  .then((cred) => {
    console.log("NEW CREDENTIAL", cred);

    // normally the credential IDs available for an account would come from a server
    // but we can just copy them from above...
    var idList = [{
        id: cred.rawId,
        transports: ["usb", "nfc", "ble"],
        type: "public-key"
    }];
    getCredentialDefaultArgs.publicKey.allowCredentials = idList;
    return navigator.credentials.get(getCredentialDefaultArgs);
  })
  .then((assertion) => {
    console.log("ASSERTION", assertion);
  })
  .catch((err) => {
    console.log("ERROR", err);
  });
```
  
#### 13:30-14:00 npm i -g @next-and-beyond: Building the future of package management
- RoomC by Claudia Hernández
- 講者是 npm 的人
- npm 還在積極開發 v6 跟 v7，v7是大改版
- `pacote` 是 npm 裡面負責 fetch library 的模組
  - 只要關於 fetch library 就是靠這個
- `arborist` 這是 v7 的代號，改善效能
  - 而且開始支援 `monorepo` <--- 這個好像是大公司的趨勢
  - 有提到 `babel` 當時的討論串 https://github.com/babel/babel/blob/master/doc/design/monorepo.md
    - 裡面的下面有好多公司的參考：`apple`, `twitter`, `fb` 說他們怎麼處理 scale repo 的問題，好像都是 `monorepo` 方向
- 忘了是 v6 還是 v7 要開始支援 `yarn.lock.json` 檔，這樣以後 yarn 的專案，我也能用 `npm i`

![JsConfJap2019_Tokyo_Day2_60](/assets/img/JsConfJap2019_Tokyo_Day2_60.jpg)
![JsConfJap2019_Tokyo_Day2_62](/assets/img/JsConfJap2019_Tokyo_Day2_62.jpg)
![JsConfJap2019_Tokyo_Day2_64](/assets/img/JsConfJap2019_Tokyo_Day2_64.jpg)
![JsConfJap2019_Tokyo_Day2_65](/assets/img/JsConfJap2019_Tokyo_Day2_65.jpg)
![JsConfJap2019_Tokyo_Day2_66](/assets/img/JsConfJap2019_Tokyo_Day2_66.jpg)

#### 同一時間還有另一場關於 WebAssembly 的

這場其實技術含量高，我沒去聽是因為事前 google 到，她之前有講過很類似這個題目  
我事前看 youtube 看完了
- RustConf 2019 - From Electron, to Wasm, to Rust (Aaand Back to Electron) by Irina Shestak
  - https://www.youtube.com/watch?v=Yo83yhRN_Ss
- 講者是 `MongoDB` 的人
- 用 electron (nodejs) 開發一個 `MongoDB` 工具
- 遇到效能問題，用 `WebAssembly` 處理
- 從她整場分享下來，要用 `WebAssembly` 改善效能絕對ＯＫ，但也覺對不輕鬆
  - 舉例 `js` 的數字型態只有 `number` 一個
  - `rust` 的數字型態有超過 15 個樣子 = =|||，她光決定要轉哪個就很頭大
- debug 也是都一塊西一塊找

她的結論 `WebAssembly` 不是未來，它只適合處理特定議題


#### 14:15-14:45 JSConf Panel Talks
- RoomA by Jan Lehnardt and Yosuke Furukawa

純粹聊聊 JSConf japan 的事情  
我聽前面而已就走了  
有意思是的，有人問會什麼 jsconf japan 票價這麼便宜
- `8000 日圓`對日本人平常的聚會來說，這算很貴的價格
  - 為了不勸退日本技術人，所以這個定價
  - 但相對場地、相關配套也有限

8000 日圓 -> 2300  NT$

順手查其他 jsconf 的價格，jsconf EU 為
Name	End Date	Gross Price
- Early pricing	27th December 2018	699€
- Regular pricing	27th February 2019	799€
- Late pricing 1	13th March 2019	799€
- Late pricing 2	27th March 2019	809€
- Late pricing 3	10th April 2019	819€
- Late pricing 4	24th April 2019	829€
- Late pricing 5	8th May 2019	839€
- Late pricing 6	22nd May 2019	849€

換算台幣 1:33
- 699€ 約 23000 NT$ .....
- 799€ 約 26000 NT$ .....

Oh my god, 歐洲這種技術活動真是夭壽貴  
前陣子來日本舉辦的 乙太訪 conference 印象約 1000€  

![JsConfJap2019_Tokyo_Day2_67](/assets/img/JsConfJap2019_Tokyo_Day2_67.jpg)  

#### 15:30-16:00 Browser APIs: the unknown Super Heroes
- RoomA by Rowdy Rabouw
有趣的一場，介紹很多 api  
這些絕對都是實用的 api  
只是平常自己開發的東西，沒能接觸這些而已  
- `geolocation` 地理位置，能知道經緯度
  - 最實用的應該是直接把這個經緯度，丟給 google map，這樣就能直接開始導航
- `battery` 電池狀況、是否正在充電中
  - 這類的 API 有支援議題，所以寫法上要防呆
- `connection` 網路狀況
  - 最常見的就是 streaming 時，網路差，就改低畫質給你
  - 連上網路 **不等於** 連上 internet
- media: `<audio />`
  - 要注意 `preload` 屬性，如果什麼都不下的話，畫面一 load 完就會開始下載 audio
  - 這樣不好，應該是 user 要聽的時候，才去下載
  - `preload` 可以設 none
  - 或者另一個 `metadata` 的更好，它會下載基本資訊，例如長度。這樣 User 看到長度等等，決定要不要聽
    - 要聽時，點擊後才開始下載
  - media 還可以 recorder
    - 可以用在 語音辨識、voice auth 時
- `震動` 的 API
- `SpeechSynthesis` 說話的 API，可以加強 a11y
- `Web blueTooth` 藍芽 API，現場 demo 連結裝置看起來很輕鬆
  - 應該是很強大的 API

```js
if ("geolocation" in navigator) {
  /* geolocation is available */
  navigator.geolocation.getCurrentPosition(function(position) {
    do_something(position.coords.latitude, position.coords.longitude);
  });
} else {
  /* geolocation IS NOT available */
}
```

![JsConfJap2019_Tokyo_Day2_69](/assets/img/JsConfJap2019_Tokyo_Day2_69.jpg)  

```js
const battery = navigator.battery
  || navigator.mozBattery
  || navigator.webkitBattery;

function updateBatteryStatus() {
  console.log("Battery status: " + battery.level * 100 + " %");

  if (battery.charging) {
    console.log("Battery is charging"); 
  }
}

battery.addEventListener("chargingchange", updateBatteryStatus);
battery.addEventListener("levelchange", updateBatteryStatus);
```

![JsConfJap2019_Tokyo_Day2_72](/assets/img/JsConfJap2019_Tokyo_Day2_72.jpg)  
![JsConfJap2019_Tokyo_Day2_73](/assets/img/JsConfJap2019_Tokyo_Day2_73.jpg)  

```js
const connection = navigator.connection
  || navigator.mozConnection
  || navigator.webkitConnection;

function updateConnectionStatus() {
  alert("Connection bandwidth: " + connection.bandwidth + " MB/s");
  if (connection.metered) {
    alert("The connection is metered!");
  }
}

connection.addEventListener("change", updateConnectionStatus);
updateConnectionStatus();
```

![JsConfJap2019_Tokyo_Day2_74](/assets/img/JsConfJap2019_Tokyo_Day2_74.jpg)  
![JsConfJap2019_Tokyo_Day2_75](/assets/img/JsConfJap2019_Tokyo_Day2_75.jpg)  
![JsConfJap2019_Tokyo_Day2_76](/assets/img/JsConfJap2019_Tokyo_Day2_76.jpg)  
![JsConfJap2019_Tokyo_Day2_77](/assets/img/JsConfJap2019_Tokyo_Day2_77.jpg)  
![JsConfJap2019_Tokyo_Day2_78](/assets/img/JsConfJap2019_Tokyo_Day2_78.jpg)  
![JsConfJap2019_Tokyo_Day2_79](/assets/img/JsConfJap2019_Tokyo_Day2_79.jpg)  
![JsConfJap2019_Tokyo_Day2_80](/assets/img/JsConfJap2019_Tokyo_Day2_80.jpg)  
![JsConfJap2019_Tokyo_Day2_81](/assets/img/JsConfJap2019_Tokyo_Day2_81.jpg)  
![JsConfJap2019_Tokyo_Day2_83](/assets/img/JsConfJap2019_Tokyo_Day2_83.jpg)  
![JsConfJap2019_Tokyo_Day2_87](/assets/img/JsConfJap2019_Tokyo_Day2_87.jpg)  


#### 16:00-16:30 Anatomy of a Click
- RoomA by `Benjamin Gruenbaum`

齁，原來這人是 nodejs core team 的人，難怪聽他私下聊的時候，對 `V8` 這麼熟  
各種奇耙舉例例子  

後來知道，這 talk 主題他有在別的地方講過
- Benjamin Gruenbaum - The Anatomy of a Click
- https://www.youtube.com/watch?v=pGSYpKPjD6I

他目前好像在開發一個新的 frontend test 框架 -> `testim`  
整場 talk 算是他在探索 `onclick 行為`

![JsConfJap2019_Tokyo_Day2_91](/assets/img/JsConfJap2019_Tokyo_Day2_91.jpg)  
![JsConfJap2019_Tokyo_Day2_92](/assets/img/JsConfJap2019_Tokyo_Day2_92.jpg)  
![JsConfJap2019_Tokyo_Day2_93](/assets/img/JsConfJap2019_Tokyo_Day2_93.jpg)  

聽他講解 onClick event 講的真是清楚，讓我複習了整個過程  
onClick event dispatch 後，有三個階段
  1. 從外到內 -> `Capture phase` -> 從 `HTML` 一層層往下傳
  2. 目標收到 -> `Target receives event` -> DOM 收到事件
  3. 往上冒泡 -> `Bubbling phase` -> 從 `目標` 一路往外到 `HTML`


![JsConfJap2019_Tokyo_Day2_95](/assets/img/JsConfJap2019_Tokyo_Day2_95.jpg)  
![JsConfJap2019_Tokyo_Day2_96](/assets/img/JsConfJap2019_Tokyo_Day2_96.jpg)  
![JsConfJap2019_Tokyo_Day2_97](/assets/img/JsConfJap2019_Tokyo_Day2_97.jpg)  
![JsConfJap2019_Tokyo_Day2_98](/assets/img/JsConfJap2019_Tokyo_Day2_98.jpg)
![JsConfJap2019_Tokyo_Day2_99](/assets/img/JsConfJap2019_Tokyo_Day2_99.jpg)  

`React` 為了改善效能
- 有做 event delegation，而且下很多功夫

![JsConfJap2019_Tokyo_Day2_100](/assets/img/JsConfJap2019_Tokyo_Day2_100.jpg)
![JsConfJap2019_Tokyo_Day2_101](/assets/img/JsConfJap2019_Tokyo_Day2_101.jpg)  

我聽不懂他說的 dispatch event React 會 flaky
- 應該是指，因為 react 自己有做 event delegation，所以測試某個 target 時，有時候 listen 在這個 target 會有問題
- 可能因為我們都是直接用官方的 react test framework，所以沒遇過這問題
- 應該是他剛好正在開發 test framework，然後用標準的 browsers 行為去期待，結果反而有問題
- `angular`、`Svelte`、`vuejs` 沒有這樣做，所以比較沒問題
  - 說這樣做才能給 browsers 更多優化的機會

![JsConfJap2019_Tokyo_Day2_102](/assets/img/JsConfJap2019_Tokyo_Day2_102.jpg)
![JsConfJap2019_Tokyo_Day2_103](/assets/img/JsConfJap2019_Tokyo_Day2_103.jpg)  

其實滑鼠按下 click 時
- OS 層級會先有動作
  - 如 USB 會先接收到訊號判斷

總之就是他從很底層部分研究說明，這應該聽聽就好  

他極度推薦 chrome 官方的文件網站
- 說搜尋功能超強
- 非常多都有詳細的流程圖表說明細節的內容
- https://cs.chromium.org
- 他找這個問題時，就搜尋 `MouseEvent` 就找到文件了


![JsConfJap2019_Tokyo_Day2_104](/assets/img/JsConfJap2019_Tokyo_Day2_104.jpg)  
![JsConfJap2019_Tokyo_Day2_106](/assets/img/JsConfJap2019_Tokyo_Day2_106.jpg)  
![JsConfJap2019_Tokyo_Day2_107](/assets/img/JsConfJap2019_Tokyo_Day2_107.jpg)  

他有提到 `cypress`
- 但我沒有很懂他的意思，好像意思是 `cypress` 的 event 有跳過一段
- 這樣跳過一段，而不是從正常的 flow 進去的話，cross browser 就會有差別
- 所以他覺得這是其中一個主因，為什麼大家比較推崇 `selenium`

![JsConfJap2019_Tokyo_Day2_110](/assets/img/JsConfJap2019_Tokyo_Day2_110.jpg)  
![JsConfJap2019_Tokyo_Day2_111](/assets/img/JsConfJap2019_Tokyo_Day2_111.jpg)  
![JsConfJap2019_Tokyo_Day2_112](/assets/img/JsConfJap2019_Tokyo_Day2_112.jpg)  

整場可能對我來說太深了
- 整個 talk 應該是偏向，分享一些東西給我們聽
- 知道這些事情後，目前他說也沒有銀彈
  - 自動化 event test (cross browser) 就是很困難

 `Benjamin Gruenbaum` 在 stackoverflow 那邊有回答很多 js 的問題  
看來是蠻值得看看的

![JsConfJap2019_Tokyo_Day2_113](/assets/img/JsConfJap2019_Tokyo_Day2_113.jpg)  


#### 16:45-17:15 Discovering Animals with AI and Javascript
- RoomA by Jonny Kalambay

很中規中矩的介紹怎麼開發 `tfjs`
- `client 端`辨識的話，他認為有三大好處，cost, Speed, Privacy
- `tfjs` 在 mobile 上，就是用 `webGL`、在 nodejs 上，會直接用 `GPU`
- `tfjs` 官方已經有好幾個很實用的 pre-traing modal 可以使用
- 如果目標中有多個對象的話，需要再加一層 `object detect` 辨識的模型
  - 先辨識有幾個目標，再把目標一個一個丟進去辨識

最好的文件
- 官方教學

![JsConfJap2019_Tokyo_Day2_117](/assets/img/JsConfJap2019_Tokyo_Day2_117.jpg)  
![JsConfJap2019_Tokyo_Day2_119](/assets/img/JsConfJap2019_Tokyo_Day2_119.jpg)  
![JsConfJap2019_Tokyo_Day2_120](/assets/img/JsConfJap2019_Tokyo_Day2_120.jpg)  
![JsConfJap2019_Tokyo_Day2_121](/assets/img/JsConfJap2019_Tokyo_Day2_121.jpg)  
![JsConfJap2019_Tokyo_Day2_122](/assets/img/JsConfJap2019_Tokyo_Day2_122.jpg)  
![JsConfJap2019_Tokyo_Day2_125](/assets/img/JsConfJap2019_Tokyo_Day2_125.jpg)  

另外提了三個點，是開發心得
- modal 剛 download 完後，第一次辨識會非常慢（沒聽懂為什麼會慢）
  - 所以一旦 download 完後，可以先 `warmupResult` -> 暖機就是了
- 當有多個 modal 時，你要注意 file size。每個 modal 可能檔案都不小
  - 他的例子，`object detect` + `他自己訓練的 modal` 可能就快 10mb 了
  - 所以有選擇的時候，**要盡可能選小 size**
- 因為是使用 GPU，所以`要自己做 gc (garbage collection)`

![JsConfJap2019_Tokyo_Day2_129](/assets/img/JsConfJap2019_Tokyo_Day2_129.jpg)  
![JsConfJap2019_Tokyo_Day2_131](/assets/img/JsConfJap2019_Tokyo_Day2_131.jpg)  
![JsConfJap2019_Tokyo_Day2_133](/assets/img/JsConfJap2019_Tokyo_Day2_133.jpg)  

#### 17:15-17:45 Pika: Reimagining the Registry
- RoomA by Fred K. Schott
- 說 Pika 專案是個大賭注
- 希望未來不需要把 js file bundle 起來
  - 現在的做法就是 code splite 然後傳需要的 chunk
  - 但這樣很有機會有很多重複的 code

![JsConfJap2019_Tokyo_Day2_134](/assets/img/JsConfJap2019_Tokyo_Day2_134.jpg)  
![JsConfJap2019_Tokyo_Day2_135](/assets/img/JsConfJap2019_Tokyo_Day2_135.jpg)  
![JsConfJap2019_Tokyo_Day2_136](/assets/img/JsConfJap2019_Tokyo_Day2_136.jpg)  
![JsConfJap2019_Tokyo_Day2_137](/assets/img/JsConfJap2019_Tokyo_Day2_137.jpg)  

如果能幹掉這些重複的 code，那未來可能會
- 減少 80% 的 tool
- 開發快 10 倍
- web 快 10 倍

![JsConfJap2019_Tokyo_Day2_138](/assets/img/JsConfJap2019_Tokyo_Day2_138.jpg)  
![JsConfJap2019_Tokyo_Day2_139](/assets/img/JsConfJap2019_Tokyo_Day2_139.jpg)  
![JsConfJap2019_Tokyo_Day2_142](/assets/img/JsConfJap2019_Tokyo_Day2_142.jpg)  

他說了一個很好的現況例子
- 假如我們現在去分別訪問
  - fb, reddit, hotmail
- 這三個網站，我們進去的第一件事情會是
  - download reactjs
  - download reactjs
  - download reactjs

就有三份同樣的東西 cache 在 browsers 裡面了
- 如果以後 browsers 能夠 cache 類似 `import https://cdn.xxxxxx./reactjs`
- 這樣可以大幅減少重複的 code
- cache 了，速度也變快

總之這是一個大膽的賭注，他們剛 release 1.0  
有趣的思考

![JsConfJap2019_Tokyo_Day2_145](/assets/img/JsConfJap2019_Tokyo_Day2_145.jpg)  
![JsConfJap2019_Tokyo_Day2_147](/assets/img/JsConfJap2019_Tokyo_Day2_147.jpg)  


####  Sponsor talk Yahoo! Japan, Recruit, Dwango DMM, Twilio
- Yahoo Japan 也是前幾年開始淘汰 angular 了
  - 跟 Line 一樣 Yahoo Japan 內部 vuejs 比 react 多一點點
- Twilio 這個簡訊服務好像很紅，蠻多地方都聽到的  看來是簡訊的一個主流解決方案

####  points at random
- MARIKO KOSAKA <-- Google 的人、google IO 介紹踩地雷那位

她講一些當年她在日本進入社群的事情  
社群是大家一起加入進來成形的之類的  
總之就是鼓勵大家、也稱讚大家是社群的一份子  

