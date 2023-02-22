## Developing Microservice Apps

## Naming Developer Environments
## https://cloud.google.com/appengine/docs/standard/java/creating-separate-dev-environments

cloud base 的專案時，應該部署多個環境，每個環境可以命名為
- dev
- qa
- staging
- prod

每個環境應要獨立、互相隔離。(使用多個 GCP projects)  
如果你用 micro service 架構的話，你的環境可以取名為：
- web-app-dev
- web-app-qa
- web-app-prod

如果你的 application 直接是 for 多個 project ，也可這樣命名：
- web-app-dev
- web-app-prod
- user-service-dev
- user-service-prod

## Contracts, Addressing, and APIs for Microservices
## https://cloud.google.com/appengine/docs/standard/java/designing-microservice-api

micro service 之間基本上都是用 HTTP-based RESTFUL 來溝通，follow 好的 pattern 來確保穩定、安全和效能。

microservices-based 的一大重點是
>  部署 microservices 跟其他的 microservice 是完全獨立的。

#### Using strong contracts
microservices 一定要提供與其他 client 溝通的`詳細定義規範`並且有`版本控管`。  
services 一定要 follow 這些 `versioned contracts`，直到肯定確認其他 service 沒有再依賴某些特定版本規範。（甚至其他 service 還有可能 roll back 來使用舊版本 contracts）  

這個 `versioned contracts` 會是團隊內的挑戰，確保內部開發團隊知道接下來是 `breaking change` 還是 `non breaking change`，他們才知道需不需要 release new major。所以 team 內部的溝通很重要。

#### Addressing microservices
services 跟 code 可以直接被訪問，你可以分別部署新的 code。每個 App Engin project 有 default 的 service、每個 service 有 defautl 的 code 版本。  
要連到這些 `default 版本的 defualt service`，可以利用 URL 帶著 app ID 就好  
例如：  
> http://my-app.appspot.com

如果把 service 命名為 `user-service`，那就這樣訪問 default serving version of that service：
> http://user-service.my-app.appspot.com

如果 deploy 另一個 非default 的 code version 命名為 `banana` 為 `user-service`
> http://banana.user-service.my-app.appspot.com

如果 deploy 另一個 非default 的 code version 命名為 `cherry` 為 default service
> http://cherry.my-app.appspot.com

> 重要！在 micro service 架構裡，client 永遠不應該能夠直接指定**特定版本號**
直接指定**特定版本號**的做法應該只有在：
- smoke testing 
- facilitate A/B testing
- rolling forward
- rolling back

client 端只能打到 defautl service 的 default version 或者 特定服務的 default version  
- http://my-app.appspot.com
- http://user-service.my-app.appspot.com

這種方式可以幫 service 部署新版本，包含 bug fix，這樣 client 端可以不用改。

### Using API versions
每個 microservice API 在 URL 上應該要有**大版本號**
> /user-service/v1/

新舊版本可以一起服務、不同版本到時候也有清楚的 log。
```
/user-service/v1/
/user-service/v2/
```

不包含 `minor API version`，因為（定義上）`minor API version` 改動不會有**breaking chages**。小版本號會讓 URL 暴增，而且 client 也會有不確定性、隨意轉換到任意小版本。  

這篇文章假定的環境是有 CI/CD 環境、master branch 總是部署在線上，所以這邊有兩個 `version` 的概念在這篇文章裡。

#### Code version
直接 map App Engine service verions、代表著 master branch 某次 commit 的 code。

#### API version
直接對應 API URL、代表著：
- request 的參數接受的內容
- responese 回覆的 docuement
- API行為

當要推新功能時，可能這樣部署 master branch 的新舊版本：
- /user-service/v1/
- /user-service/v2/

為了分流，可能需要從主要 API version 移到 service version，舉例 client 可能會用到：
- http://user-service-v1.my-app.appspot.com/user-service/v1/
- http://user-service-v2.my-app.appspot.com/user-service/v2/

(上面這例子 /v1/ /v2/ 是多餘的，但在分析 log 時也是有幫助的)  
另外順便要注意 每個 App Engine application 允許的 services 最大上限。

### Breaking v.s. non-breaking changes
了解到 breaking change 跟 non-breaking change 是很重要的。  
breaking change 通常可能（常常是刪減東西）：
- 移除掉 `request` or `response` 某些參數
- 改變 document 的內容
- 調整某些 key 的 name

新的 feature 常常都是 breaking change、micro service 改變行為時，通常也幾乎都是 breaking change。  

non-breaking change 通常是新增點什麼：
- 新增 optional 的 request 參數
- response 新增回傳點東西

為了要做到 non-breaking changes，`on-the-wire serialization` 是必要的！

### on-the-wire serialization
很多 serializations 是適用的、non breaking changes：
- JSON
- Protocal Buffers
- Thrift

這些 serializations 都會 silently 忽視多出的、非期望的 information。  
在動態語言上，多出的資訊簡明的出現在 deserialized 的 object 上。

假設這 JSON 定義是 `/user-service/v1/`:
```json
{
  "userId": "UID-123",
  "firstName": "Jake",
  "lastName": "Cole",
  "username": "jcole@example.com"
}
```
下面這樣的改動就是 breaking chage  
你就需要 re versioning 這 service 到 `/user-service/v2/`:  
```json
{
  "userId": "UID-123",
  "name": "Jake Cole",  # combined fields
  "email": "jcole@example.com"  # key change
}
```
這種就是 non-breaking change，所以不需要 new version：
```json
{
  "userId": "UID-123",
  "firstName": "Jake",
  "lastName": "Cole",
  "username": "jcole@example.com",
  "company": "Acme Corp."  # new key
}
```

### 部署新的 non-breaking minor API versions
當部署新的小版本號 API version，`App Engine(google)` 允許新的版本號發布 side-by-side 舊的版本。`App Engine` 可以存取任意版本、但只有一個 version 可以當作 default 版本。

> 回顧：每一個 service 都有一個 default serving version

上面這舉例，我們可以有一個 old code version，是目前 default serving version，稱為為 `apple`。  
接著我們 side by side 部署新的版本（稱為 `banana`）。這邊兩個的 URLs 還是一樣的  
> /user-service/v1/

因為我們部署的為 non-breaking minor API change。

> 這面這兩段的舉例，情境都是用 GCP 的 micro service App Engin 的講解一些事情。畢竟這是 google 的文件，但這些思維，還是可以讓我來學習一些版本 and 部署上的設計。

`App Engine` 有機制可以自動整合流量把 `apple` 導到 `banana`（就改把 banana 當作 default version）新的 defualt 被定下後，接著的 request 都會到 `banana` 了。  
上面這就是部署新的 (minor or patch) API 的過程、不會影響 client 的 microservices

如果有出錯，可以用 rolling back 來 reversing 上面的步驟，就改回 apple 成為 default version。
> in-progress 的 request 還是會讓它完成

`App Engin` 允許設定 多少% 跑到哪個 code version（金絲雀 release, canary release）、也就是 `App Engin` 裡面的 `traffic splitting`，這還可以設定時間機制，例如隨時間改動分配比例，15分鐘後，比例調整等等。

>> 此用 traffic splitting 時，（同一架構底下的）其他 micro service 要注意。因為 microservices （應該）是 stateless 的，所以它（應該）會隨機打 microservices。（大概是這樣的概念）

### 部署 new breaking major API versions
部署/回朔 breaking change API 的流程，跟 non-breaking 是一樣的。  
但，基本上你不會去做 `traffic splitting` 或 `A/B tests`，因為 breaking release 會是一個全新的 URL，例如
> /user-service/v2/

當部署新 API 時，很重要的是記得
> 舊 API 可能還在 work 中（`/user-service/v1/`）

當確定 old API 確定沒有用的時候，才下架。  
舉例，你有個 microservice `web-app`，依賴其他的 microservice `user-service`。  
假設 `user-service` 要改大版了，`web-app` 目前沒法支援新版。這我們就
- 1. `user-service` 部署 `/user-service/v2/`
- 2. `/user-service/v1/` 還是 run 著，繼續服務。
- 3. `web-app` 就要開發改 code，讓依賴從 `v1` 轉到 `v2`
- 4. 驗證過認定 `web-app` 不再需要 `v1` 後，才移除 `v1` （跟移掉一些 temporary code）

這種部署可能需要寫一些 temporary code 來支持。  
所有的操作看起來都變更麻煩了，但這就是 micro service 為 base 的 application 作法，為了達到 independent 的 development release cycles。
 
## Best Practices for Microservice Performance 
## https://cloud.google.com/appengine/docs/standard/java/microservice-performance

microservices 總是會有些效能上的問題，這邊一些建議來 minimize 這些影響。

### Turn CRUD operations into microservices
Microservices 特別適合 CRUD pattern。這種運作模式典型的做法是，一次只使用一個 entity。例如 **一位使用者**、一次只執行一個 CRUD 操作。  
因此你只需要 **單一的microservice** 來執行此操作。找出一個有 CRUD 的 entities 加上一組 bussiness methods 操作，這樣就能讓你的 application 使用了。這上面所提的 entities 就可以靠 microservices。

### 提供 batch APIs
除了 CRUD-style APIs，另外可以提供 batch 式的 API 來讓 microservice 提供給 entities們 更好的 performance。  

舉例來說，不止是提供 API 給**單位 user**，而提供 API，可以接收**一組user IDs** 並直接 return 給相對應的 users。

#### Request 範例:
> /user-service/v1/?userId=ABC123&userId=DEF456&userId=GHI789

#### Response 範例:
```json
{
  "ABC123": {
    "userId": "ABC123",
    "firstName": "Jake",
    … },
  "DEF456": {
    "userId": "DEF456",
    "firstName": "Sue",
    … },
  "GHI789": {
    "userId": "GHI789",
    "firstName": "Ted",
    … }
}
```
(`App Engine SDK(google的)` 支持很多 batch APIs)

### 採用 asynchronous 的 requests
實務上會**需要跟多個 microservices 互動來拿到東西來組合**出我們要的東西。例如  
你需要抓`目前已登入的 User 的「偏好」和他們「公司的細節」`。這些資料彼此獨立，所以應該要能`平行呼叫 microservices 來取回些資料`。（Urlfetch library、Google App Engine SDK 有支援。） 

### Use *.appspot.com, not a custom domain
對 (GCP) google infrastructure 的 routing 機制而言，custom domain 會有不同的 route。  
如果你使用 `my-app.appspot.com` 為 hostname，你的 microservice 呼叫是內網的，所以 performs 會更好。

### Set follow_redirects to False
這邊是說 `Urlfetch` 的設定 `follow_redirects` 可以關掉。原因是因為 microservice 的架構，你應該不需要用到 redirect 的機制， http status code 應該只會有 2xx 4xx 跟 5xx。

### Prefer services within a project over multiple projects
雖然設計上，microservice 部署在不同的 project(GCP的) 是好的設計。但如果效能是最主要的目標的話，就設在同一個 project。因為 GPC 來說，同一個 project 代表他們實體上在同一個 datacenter 中。

### Avoid chatter during security enforcement
在有 security 機制環境下的時候，如果有大量的溝通 call API 時做的驗證(auth)，performancer 就會差。

> 舉例，如果 microservice 的 application 需要經過檢驗 ticket 後才能拿到結果的話，這樣需要來來回回的驗證過後才能拿到我們要的資料。

`OAuth2` 的機制(`refresh tokens` 跟 `caching access token`(`Urlfetch`的設計))可以分攤這個 cost。 但是如果 `cached access token` 是存在機器裡面，你就需要先做 fetch 一次memcache，為了要避免這個 overhead，你可能需要 cache 這個 access token 在 instance 的 memory 中。  

但你仍然會頻繁的經歷到 OAuth2 的操作，像是每一個全新的 instance 協調一個 access token。  
（App Engine instances 的頻繁 spin up and down 就會這樣）  
有一些 hybrid of memcache and instance cache 可以減少這樣問題，但實作上又更困難了。  

另一種方式是 microservices 之間共享 secret token。舉例，傳送一個 custom 的 HTTP header。  
這方法，每一個 microservice 可能給每一個 caller 都有一個 unique token。  
基本上，shared secrets 的做法會是個安全的疑慮。但因為所有的 microservices 都是一樣的 application，所以這樣安全性問題比較不大，而換取到 performance。  
這種 shared secret 的做法，microservice 只需要做**字串比較**，拿傳進來的 secret 跟 presumably in-memory dictionary 比較。這種 security 的方式就很 light 了。  
 
（如果你所有的 microservices 都在 `App Engine`，你可以檢查進來的 header `X-Appengine-Inbound-Appid`，這 header 是 `Urlfetch` 加上的，使 request 其他 `App Engin` project 並且不讓第三方去設定。）  

根據你自己的 security 需求，你的 microservices 可以去檢查 `header` 來執行你要的 security policy。

### API responses
API 的 responses 在 response 帶上一個 unique request ID 回去是很有幫助的。這樣能辨別 client 的 request 是跟哪個  microservice 的 responses。之後要 debug 之類的很有幫助。

```js
// 舉例 For example (in pseudo-code):
response = {
  'status': 200,
  'apiVersion': '1.2',  // helpful to show the formal minor version to the client
  'hostname': os.environ.get('HTTP_HOST'),
  'path': os.environ.get('PATH_INFO'),
  'instanceId': os.environ.get('INSTANCE_ID'),
  'codeVersion': os.environ.get('CURRENT_VERSION_ID'),
  'requestId': os.environ.get('REQUEST_LOG_ID'),
  'data': {
    // your actual API response goes here
  }
}
```

## Microservices Architecture on Google App Engine
Microservices 是開發 Application 的一種參考架構，它能讓大型 application 解耦合變成**獨立構成的部位**。每個部位有自己層面的責任，為了要服務單一 user 或者 API request，microservices-based 的 Application 可能會有很多 internal microservices 來組成來 response。  

好的 microservices-based application 能達到：
- 不同的 microservices 之間定定義優秀的  contracts。
- 允許獨立的部署流程、含 rollback。
- 容易 concurrent, A/B release testing on subsystems.
- Minimize test automation and quality-assurance overhead.
- 改善 logging 和 monitoring 的資訊的清楚。
- Provide fine-grained cost accounting.
- 提升 App 整體的 scalability 和 reliability.

## Migrating to Microservices from a Monolithic App 
從傳統的單一 App 開始時，要先找到可以單獨出來成為 microservices 的部分。通常有良好架構的 單一App 能很好拆分

- 找出 App 能拆出來的 bussinsee 邏輯
- 找那些已經是 isolated 的 code。
- 檢查 App 的邏輯，找出那些如果能 scaling configuration settings 或 memory 就對 App 有幫助的部分。

要拆分，是需要重構來移除一些 dependencies，所以建議：
> 拆服務之前，先重構一些 legacy code，然後部署 prod，確認是否 OK。

一些常見的 microservices 地方：
- User or account information
- Authorization and session management
- Preferences or configuration settings
- Notifications and communications services
- Photos and media, especially metadata
- Task queue workers