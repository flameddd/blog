## Gatsbyjs 對未來 (Modern) frontend (CMS) architecture 思考與看法

這系列文章前陣子就有注意到，一直放著沒看。  
直到 2019/11/30 去 japan 參加 `jsconf japan`，現場聽 `Guillermo Rauch` (founder of ZEIT, co-creator of Now and Next.js, and former CTO) 講述這概念，回來才決定一定要看完這幾篇。  
- jsconf 我個人筆記: https://github.com/flameddd/blog/blob/master/2019-11-30%EF%BC%9Ajsconfjp%20(Javascript%20Conferences%20Japan)%202019.md#1530-1600-building-and-deploying-for-the-modern-web-with-jamstack

### Journey to the Content Mesh (共 5 篇)
1. Delivering Modern Website Experiences
    - https://www.gatsbyjs.org/blog/2018-10-04-journey-to-the-content-mesh/
2. Unbundling of the CMS
    - https://www.gatsbyjs.org/blog/2018-10-10-unbundling-of-the-cms/
3. The Rise of Modern Web Development
    - https://www.gatsbyjs.org/blog/2018-10-11-rise-of-modern-web-development/
4. Why Mobile Performance Is Crucial
    - https://www.gatsbyjs.org/blog/2018-10-16-why-mobile-performance-is-crucial/
5. Creating Compelling Content Experiences
    - https://www.gatsbyjs.org/blog/2018-10-18-creating-compelling-content-experiences/


### 1. The Journey to a Content Mesh
第一篇提到 `Content Mesh` 的概念
- 一個方向是，現在有很多優秀的第三方服務提供（類似 SaaS）
- 傳統的 CMS，什麼 feature 都要自己動手打造
- 提倡要去串這些第三方，才能更快速、提供更好的體驗

第三方
- `Shopify` 可以當作 e-commerce 商品庫存網站
- `Salsify` product listings 網站（沒聽過），什麼是 product listings 的概念？
- `Bazaarvoice` 做 product reviews，也沒聽過這概念.... 
  - 這間公司上市後，又被收購，所以下市。 2005 年成立，這樣看來算很成功
  - Bazaarvoice 為顧客評價社交商務平台提供商，通過SaaS(軟件即服務)模式，提供各式的在線評論引擎，匯集來自facebook，twitter等各式社交平台的顧客點評，且能將零售商的產品點評同步到購物比價引擎中去
- `WordPress` paywalled content site can create stories in WordPress （不懂這概念）
- `JWPlayer` store video (不懂...)
- `Auth0` store user data (我常聽到這間公司，但不知道它也有做帳號管理！？)
  - 原來網路上稱 `Auth0` 為 **身分管理平臺**
- `Recurly` subscription data (沒聽過，也不懂...)
- `Algolia` (full text) search
- `Stripe` payments
- `Optimizely` for A/B testing。沒聽過這間
  - 或者說，我前幾天才知道原來很多人 `Google Optimize` 做 A/B test
  - 我第一次知道 `Google Optimize` 這產品，那這個 `Optimizely` 可能也是類似的概念
    - Google Optimize安裝教學- 免費又好用的A/B Test工具！
    - (https://www.awoo.com.tw/blog/google-optimize-installation/)
- `Evergage` for personalization (沒聽過，也不懂...)

確實，Gatsbyjs 提出很多功能都是我沒接觸過的。畢竟沒有接觸這類的市場與方向，如果真的有需要這些的話，串第三方真的是個選項！就像 paymant 一樣，沒人想自己去做。所以如果把上面的某些 feature 認定為自己產品的核心之一的話，串服務就是很好的選擇了（$$）


### Three Areas of Rapid Innovation
文章提了三個面 向  
the confluence of three revolutions in how we create and consume content:  

1. Content management: `modular, specialized content systems` 會取代傳統的 `Monolithic CMS applications`.
2. Development techniques: `Modern UI frameworks` 是豐富 UX 體驗的重要關鍵
3. Performance: mobile 的流量已經超過總 internet traffic 的 50%，所以 `high-performance` 變成是 `must-have`，而不是 `nice-to-have`

所以 Gatsbyjs 的結論是
- A content mesh, all of these systems can be brought together in a unified, low-cost, low-defect whole.
- In other words, the content mesh makes developers, content creators, and users all happy.

### Part 2: Unbundling of the CMS

#### Integrating modular services
現在這種 `API-driven approach` 的方式，相較 `“headless”` 之外有更好的形容方法 -> `“modular”` or `“microservices”`

感覺這篇的重點是，傳統方法就算是做得很好的公司 (Drupal7, Sitecore)，也需要你一個個去整合。  
但現在有機會讓企業一次（單一供應商）擁有、打一次交道就好 -> Gatsbyjs  
（`Guillermo Rauch` 的 talk 有說到，他們現在整合這些東西，就像 plugin 一樣，很簡單就能使用）

### Part 3: The Rise of Modern Web Development
第三篇一開始說傳統 CMS 開發的困難點

- Walled-garden (封閉) development: Development can be blocked due to CMS access restrictions or code freezes. Upgrade paths can be challenging when CMSs don’t support component UI versioning. (就像 WordPress，之前同事要改這東西，都要嘗試去挖 source code)
- Maintaining local environments: Setting up a local app server and database and keeping it up to date with team members’ changes is time-consuming. (local 開發環境本來就需要花時間，它提這點意思應該是指，它們直接提供 online 的 develope 環境(我猜的))
- Project organization. Reliably installing and managing third-party dependencies, including cross-compatibility and handling bugs in upstream, is challenging. (這一點，可能是要說 Gatsbyjs 幫你去整併第三方的東西，你不用自己玩)
- A difficult target environment. Cross-browser API incompatibilities, global DOM application state, and imperative DOM APIs make it easy to inadvertently introduce bugs. (這點我感覺不太出來，可能是要給後面強調 UI framework 的事情用的)

後面就提這幾年 UI framework (react, angular, vue) 的成整與優秀，可提供更好的開發與體驗  
其中一個看事情的觀點還不錯，它去看 `website winners of the 2018 Webby awards`，獲選的網站都是用這些框架
- https://www.webbyawards.com/


### Part 4: Why Mobile Performance Is Crucial
就講在 mobile 上，讀取超過 3 秒的就算失敗  
mobile 是主流中的主流了，Performance 是最重要的一環  
另外，今天也剛好看到 pornhub 的 2019 統計報告
- https://www.pornhub.com/insights/2019-year-in-review
- 2019 年有 420 億次的訪問
- 76.6% of that traffic was from smartphones <--  

也許我的心態上，還是太看輕 mobile 了。  
之前去聽 shopline 的活動，他們說泰國那邊商務上都是用 table  

警惕警惕！！  

Movements such as `Progressive Web Apps` and the `JAMstack` have brought attention to site performance as a first-order goal.  

後面提一些可以優化的方向，並且說這些可以靠 framework 來解決 (應該就是指 next.js or Gatsbyjs)

### Conclusion: Creating Compelling Content Experiences
最後一篇，感覺比較能抓到真正有用的知識  
最後它當然就是推薦他們自家的方案 Gatsbyjs  

> To adopt a new architecture in one area, you often need to adopt new technologies in the other two.

Writing headlines like `”Gatsby + Contentful + Netlify (and Algolia)”`
- grouping a React-based website framework, a headless CMS, a static host + CDN, and a search provider
- it’s clear these these technologies are meant to be used together.

Website teams moving to this space **have to plan four steps** moving to the content mesh

#### First, choose content systems
選擇有

1. A spreadsheet and text files (for extremely simple sites).
    - 最簡單的... (嗎？ＸＤ)
2. Specialized systems tailored to their use case, such as `Shopify + Salsify + Bazaarvoice` for an e-commerce site.
3. A headless cloud CMS with rich content modelling capabilities, such as `Contentful`.
    - `Contentful` 知名的 headless CMS (google 過後才知道)

這邊的結論是，找一個好的 serverless backend 來處理這邊的事情。  
而 Gatsbyjs 主打偏向 CMS，所以 headless CMS 就是選擇的主要方向  

#### Second, pick a UI development library
這邊就說選用 react, ng 等 modern framework 的優點，好處，能改善
- SEO
- Routing
- Accessibility
- i18n

我自己在用 react，沒能感受「沒用 react」時，要做這些有多痛苦？  
總之，有 framework 來開發 large web app 還是方便   

#### Third, choose a performance strategy
There are two main approaches to performance
- payload optimization and
  - involves performance-enhancing development practices.
    - reducing image and JS weight
    - deferring below-the-fold image calls
    - inlining critical CSS
- delivery optimization
  - means compiling websites to static files that can be served from a global CDN, rather than running servers and databases.

這裡再次強調 CDN，也許是我個人沒有開發 global 的經驗，雖然知道 CDN 的優點與重要性，但沒有太多想法
- 或者應該說，我並沒有把 CDN 當作是開發的一環節在思考。
- 現在 Gatsbyjs 所提供的服務就有 CDN。firebsae host 也有自動 scaling 的功能，也就是一種 CDN 了。

也就是說，依賴這種 serverless、hosting 的 backend，而不是自己去弄 ec2 之類的，就是一種開發環境、手段與架構的選擇！  

#### Fourth, choose your content mesh
To truly engage users, you need a modern website
- a modular CMS architecture
- modern development framework
- cutting-edge performance

The challenge for website teams is:
- how to achieve this without a lot of costly, time-intensive custom integration work?

The answer: choose a `content mesh`  
A content mesh:
- pulls in data from your headless CMS systems
- enables you to develop in your preferred UI library while providing website tooling
- automatically makes your site fast out of the box

headless CMS systems 就像我去使用 firebase db 一樣
- 不自己去 host database 了，也不用管 scaling 的問題

UI library
- 用 framework 確實方便 web app 開發

fast out of the box
- 這邊提倡的是靠 cdn，這就跟我用 firebase host 是一樣的，自動幫我 scaling


接著就提它們所提供的服務
- utilizing vendor integrations
- assorted JAMStack solutions
- Gatsby

only Gatsby comes with out-of-the-box CMS integrations, a modern development framework, and cutting-edge performance.

確實我自身沒有串過太多（自認為不算多）第三方服務，又或者沒有自身去開發這些服務的經驗  
但不知道他們所謂的 utilizing vendor integrations 是不是真的這麼強大、好用  
像 firebase 要做 full text search，就需要 `Algolia`，我拖了很久都還沒弄（不是當下最重要的事情....）


Gatsby features:
- Integrations with 120+ backends, including 15+ enterprise content systems like 
  - WordPress, Drupal, Contentful, Contentstack, Salsify and Shopify
- A React development environment
- Lightning-fast out-of-the-box performance, including both payload and delivery optimizations.
 
所以它們說，用 Gatsby 是好的 CMS solution


也算是有意思吧，但如果我沒買他們的服務，能順利整合其他第三方嗎？  
也算是學學人家的架構思維  

應該是可以，就是他們有寫好 lib 而已，例如 `contentful`  
- https://www.gatsbyjs.org/docs/sourcing-from-contentful/

```shell
npm install --save gatsby-source-contentful
```

gatsby-config.js
```js
plugins: [
  {
    resolve: `gatsby-source-contentful`,
    options: {
      spaceId: `your_space_id_grab_it_from_contentful`,
      accessToken: `your_token_id_grab_it_from_contentful`,
    },
  },
]
```