
# 2023-08-31：筆記 PortSwigger 的 GraphQL API vulnerabilities.md


Ref:
- https://portswigger.net/web-security/graphql
- https://portswigger.net/web-security/graphql/what-is-graphql
- https://www.apollographql.com/blog/graphql/security/securing-your-graphql-api-from-malicious-queries/


-------------------


## GraphQL API 漏洞
GraphQL 漏洞一般是實作和設計缺陷造成的
- 如 `introspection` 功能沒關，使攻擊者能夠查詢 API 以收集有關其模式的資訊。
- GraphQL 攻擊通常以惡意 request 的形式出現
  - 攻擊者可藉此取得資料或執行未經授權的操作
  
  


-------------------


## 找出 GraphQL endpoints
在測試 GraphQL API 之前，首先需要找到它的 endpoints
- 對 GraphQL API 所有 request 都使用相同的 endpoints，因此這是非常有價值的資訊


Note:
- [Burp Scanner](https://portswigger.net/burp/vulnerability-scanner) 可以自動測試 GraphQL endpoints

-------------------


## 通用查詢(universal query)
向任何 GraphQL endpoint 發送 `query{__typename}`
- response 會包含字串 `{"data": {"__typename": "query"}}`
- 這被稱為通用查詢(universal query)，是探測 URL 是否對應 GraphQL 的有用工具

該查詢之所以有效，是因為
- 每個 GraphQL endpoint 都有一個名為 `__typename` 的保留字，該字會以字串形式返回查詢 object 的類型


-------------------

## 常見 endpoint 名稱
GraphQL 服務經常使用類似的 endpoint 後綴。測試 GraphQL endpoint 時，就拿通用查詢(universal query)送到以下位置:
- 如果這些常用 endpoint 不返回 GraphQL response，也可以嘗試在路徑中加上 `/v1`

常見 endpoint:
- `/graphql`
- `/api`
- `/api/graphql`
- `/graphql/api`
- `/graphql/graphql`


Note
- GraphQL 服務通常會以 `query not present` 或類似錯誤 response 任何非 GraphQL 請求
- 測試 GraphQL endpoint 時應牢記這一點

-------------------

## Request methods
嘗試找到 GraphQL endpoint 的下一步是使用不同的 request methods 進行測試  


在 production 環境的 GraphQL endpoint 的最佳做法是只接受  `content-type` 為 `application/json` 的 POST request
- 因為這有助於防範 `CSRF` 漏洞
- 不過，有些 endpoint 可能會接受其他方法，如 `GET` request 或 `content-type` 為 `x-www-form-urlencoded` 的 POST request
- 如果向普通 endpoint 發送 POST 找不到 GraphQL endpoint 時，嘗試使用其他 HTTP method 發通用查詢(universal query)

**初始測試:**  
一旦發現了 endpoint，就可以發送些測試 request，進一步了解它是如何運作
- 如果有網站在使用這個 endpoint ，可以在 Burp browser 中探索，並用 HTTP history 檢查發送的查詢

-------------------


## 利用還沒有消毒的參數
接著可以開始找漏洞了。測試查詢參數是很好的開始  


如果 API 使用參數直接訪問 object，就可能存在 access control 漏洞
- User 只需提供與該資訊相對應的參數，就有可能訪問到不該訪問的資訊 (IDOR)


More information
- GraphQL arguments: https://portswigger.net/web-security/graphql/what-is-graphql#arguments
- [Insecure direct object references (IDOR)](https://portswigger.net/web-security/access-control/idor)


例如，下面的查詢一個商店的產品列表:

```
#Example product query

query {
  products {
    id
    name
    listed
  }
}
```


返回的產品列表只包含列出的產品  

```
#Example product response

{
  "data": {
    "products": [
      {
        "id": 1,
        "name": "Product 1",
        "listed": true
      },
      {
        "id": 2,
        "name": "Product 2",
        "listed": true
      },
      {
        "id": 4,
        "name": "Product 4",
        "listed": true
      }
    ]
  }
}
```

根據這些資訊，我們可以推斷出以下內容:
- 產品被分配有序的 ID
- 列表中缺少產品 ID 3，可能是因為該產品已被除名

通過查詢丟失產品的 ID，可能能得其詳細資訊，儘管該產品未在中列出現:

```
#Query to get missing product

query {
  product(id: 3) {
    id
    name
    listed
  }
}
```

```
# Missing product response

{
  "data": {
    "product": {
    "id": 3,
    "name": "Product 3",
    "listed": no
    }
  }
}
```

-------------------


## 研究 schema information
測試 API 的下一步是拼湊 schema 資訊
- 最好的方法是用 `introspection` 查詢
  - `introspection` 是 built-in GraphQL function，能查詢 server 中有 schema 的資訊


`Introspection` 可幫助了解如何與 GraphQL API 進行交互。它還能披露潛在的敏感資料，如 description fields


**使用 `introspection`:**    
- 查詢 `__schema` field，這個 field 在所有查詢的 root type 中都有
- 與一般查詢一樣，在 `introspection` query 時，可以指定希望返回的 response field 和 structure
  - 例如，你可能希望 response 只包含可 mutations 的 name


**檢查 `introspection` 有沒有啟用:**  
- Best practice 是在 production 中禁用 `introspection`，但也有可能沒關掉
- 可以使用下面的查詢來測試 `introspection`。如果啟用了 `introspection`， response 將返回所有可用查詢的名稱



```
#Introspection probe request

{
  "query": "{__schema{queryType{name}}}"
}
```


Note
- `Burp Scanner` 會自動檢測 `introspection`。如果發現了，會報告 `GraphQL introspection enabled` issue



**執行完整 `introspection` query:** 
- 下步是針對 endpoint 執行完整的 `introspection` query，以獲得盡可能多的 schema 資訊
- 下面的範例查詢會返回所有查詢、mutations、subscriptions、types 和 fragments 的全部詳細資訊

```
#Full introspection query

query IntrospectionQuery {
  __schema {
    queryType {
      name
    }
    mutationType {
      name
    }
    subscriptionType {
      name
    }
    types {
      ...FullType
    }
    directives {
      name
      description
      args {
        ...InputValue
    }
    onOperation  #Often needs to be deleted to run query
    onFragment   #Often needs to be deleted to run query
    onField      #Often needs to be deleted to run query
    }
  }
}

fragment FullType on __Type {
  kind
  name
  description
  fields(includeDeprecated: true) {
    name
    description
    args {
      ...InputValue
    }
    type {
      ...TypeRef
    }
    isDeprecated
    deprecationReason
  }
  inputFields {
    ...InputValue
  }
  interfaces {
    ...TypeRef
  }
  enumValues(includeDeprecated: true) {
    name
    description
    isDeprecated
    deprecationReason
  }
  possibleTypes {
    ...TypeRef
  }
}

fragment InputValue on __InputValue {
  name
  description
  type {
    ...TypeRef
  }
  defaultValue
}

fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
      }
    }
  }
}
```

Note: 如果 `introspection` 啟用了，但上面 query 無法用
- 試試從查詢結構中移除 onOperation、onFragment 和 onField
  - (許多 endpoint 的 `introspection` 不接受這些指令)


**視覺化 `introspection` 的結果:**  
- `introspection` 的 response 通常很長，難以處理，有些工具可以幫忙視覺化
- [GraphQL visualizer](http://nathanrandal.com/graphql-visualizer/)
- `InQL`，Burp 的 extension
  - More information: [Working with GraphQL in Burp Suite](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/working-with-graphql)



**`Suggestions`:**  
- 即使 `introspection` 完全禁用了，有時也可以使用 `suggestions` 來收集 API 結構資訊
- `Suggestions` 是 Apollo GraphQL 平台的功能，server 在錯誤資訊中建議對查詢進行修改
  - 這些建議通常用於查詢稍有錯誤但仍可識別的情況
  - (例如: `There is no entry for 'productInfo'. Did you mean 'productInformation' instead?`)
  - (在 Apollo 中無法直接禁用 `suggestions`。參考 [GitHub thread workaround](https://github.com/apollographql/apollo-server/issues/3919#issuecomment-836503305))
- `Burp Scanner` 會自動檢測 `suggestions`。如果發現 `suggestions` active，會報告 `GraphQL suggestions enabled` issue


**`Clairvoyance`:**
- Obtain GraphQL API schema even if the introspection is disabled
- https://github.com/nikitastupin/clairvoyance




練習題:
- [Accessing private GraphQL posts](https://portswigger.net/web-security/graphql/lab-graphql-reading-private-posts)
- [Accidental exposure of private GraphQL fields](https://portswigger.net/web-security/graphql/lab-graphql-accidental-field-exposure)

-------------------


## 繞過 GraphQL `introspection` 防禦
如果無法在測試的 API 中執行 `introspection` 查詢，那嘗試在 `__schema` 關鍵字後插入一個特殊字
- 當開發人員禁用 `introspection` 時，他們可以使用 `regex` 在查詢中排除 `__schema` 關鍵字
- 應該嘗試**空格**、**換行**和**逗號**等字，**因為 GraphQL 會忽略它們，但有缺陷的 regex 不會**


因此，如果開發人員只排除了 `__schema{`，那下面的 `introspection` 查詢就不會被排除
```
#Introspection query with newline

{
  "query": "query{__schema
  {queryType{name}}}"
}
```


如果這方法不起作用，再嘗試其他 request method
- 有可能只有通過 `POST` 才會禁用 `introspection`。試試用 `GET` 或 `content-type` 為 `x-www-form-urlencoded` 的 POST request



下面範例顯示用 `GET` 發送的 introspection probe，帶有 URL-encoded 參數
 
```
# Introspection probe as GET request

GET /graphql?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D
```


注意
- 如果 endpoint 只接受通過 GET 發 `introspection` 查詢，而你又想用 `InQL` Scanner 分析查詢結果
- 你首先需要把查詢結果存到文件中。然後，可以載入文件到 InQL 中
 

練習題: [Finding a hidden GraphQL endpoint](https://portswigger.net/web-security/graphql/lab-graphql-find-the-endpoint)

-------------------


## 用 aliases 繞過  rate limiting
通常，GraphQL objects 不能包含多個同名屬性
- aliases 可以繞過這限制，明確命名希望 API 返回的屬性。可以用 aliases 在一次 request 中返回同一類型對象的多個實例
- [GraphQL aliases](https://portswigger.net/web-security/graphql/what-is-graphql#aliases)
 


雖然 aliases 的目的是限制你需要呼叫的 API 的數量，但也可用於對 GraphQL endpoint 進行暴力破解
- 許多 endpoint 都有某種 rate limiter，防止暴力攻擊
- 有些 rate limiter 是根據接收到的 HTTP request 次數而不是 endpoint 上執行的操作數來判斷的
- **由於 aliases 可以在單個 HTTP message 中發多個查詢，因此可以繞過這一限制**


下面的簡化範例顯示了一系列檢查商店折扣碼是否有效的 aliases 查詢
- 這操作有可能繞過 rate limiting，因為它是單個 HTTP request，儘管它有可能用於同時檢查大量折扣碼

```
#Request with aliased queries

query isValidDiscount($code: Int) {
  isvalidDiscount(code:$code){
    valid
  }
  isValidDiscount2:isValidDiscount(code:$code){
    valid
  }
  isValidDiscount3:isValidDiscount(code:$code){
    valid
  }
}
```

練習題: [Bypassing GraphQL brute force protections](https://portswigger.net/web-security/graphql/lab-graphql-brute-force-protection-bypass)

-------------------



**GraphQL 的 CSRF 漏洞是如何產生的?**
如果 GraphQL endpoint 沒有驗證發送給它的請求的 content type，也沒有實施 CSRF token，那就會出現 CSRF 漏洞


只要 `content type` 經過驗證，使用 `application/json` 的 `POST` request 就可以安全地防止偽造
- 在這種情況下，即使受害者訪問了惡意網站，攻擊者也無法讓受害者的 browser 發送該 request


但 browser 可以發 `GET` 等其他方法，或 `content type` 為 `x-www-form-urlencoded` 的任何請求
- 因此，如果 endpoint 接受這些請求，user 就有可能受到攻擊，發送惡意請求

構建基於 GraphQL 的 CSRF 漏洞的步驟與一般 CSRF 漏洞相同，可以參考
- [How to construct a CSRF attack.](https://portswigger.net/web-security/csrf#how-to-construct-a-csrf-attack)


練習題: [Performing CSRF exploits over GraphQL](https://portswigger.net/web-security/graphql/lab-graphql-csrf-via-graphql-api)

-------------------


## 防止 GraphQL 攻擊
為防止 GraphQL 攻擊，在將 API 部到 production 環境時採取以下步驟:

- 如果你的 API 不供公眾使用，就關閉 `introspection`
  - 這使攻擊者更難取得有關 API 的資訊，降低不必要的資訊洩露風險
  - [how to disable introspection in the Apollo GraphQL platform](https://www.apollographql.com/blog/graphql/security/why-you-should-disable-graphql-introspection-in-production/#turning-off-introspection-in-production)
- 如果 API 主要是給公眾使用，那你可能需要啟用 `introspection`。但你要檢查 API 的 `schema`，確保不會向公眾揭露非預期 field
- 確保禁用 `suggestions`
  - 可以防止攻擊者用 `Clairvoyance` 或類似工具收集 schema
  - Apollo 無法直接禁用，參考 [GitHub thread](https://github.com/apollographql/apollo-server/issues/3919#issuecomment-836503305) 以了解解決方法
- 確保你的 API schema 不揭露任何私人 user field，如 email or User ID




**防止 GraphQL 暴力攻擊:**  
- 在用 GraphQL API 時，有時可以繞過標準 rate limiting (上面提到的 aliases)
- 有鑑於此，你可以採取一些設計步驟來保護 API 免受暴力攻擊
- 這通常涉及限制 API 接受查詢的複雜性，減少攻擊者實施 DoS 攻擊的機會

防禦暴力破解攻擊
- 限制 API 的 `query depth`
  - `query depth` 是指查詢中嵌套的層數
  - 嵌套過深的查詢會對性能重大影響，可能為 DoS 攻擊提供機會。限制 `query depth`，可以降低發生的機率
- 限制 Configure operation
  - Operation limits enable 可讓你設 API 可接受的唯一 field, `aliases` 和 root fields 的最大數量
- 設定查詢可包含的最大 bytes 數
- 考慮對 API 成本分析
  - 成本分析是指 application 在接到查詢時識別與執行查詢相關的資源成本的過程
  - 如果執行查詢的計算過於復雜，API 就會放棄該查詢

More information: 下面有筆記另一篇文章，有更多這方面的細節


**防禦 CSRF over GraphQL:**  
- 要特別防範 GraphQL CSRF 漏洞，要在設計 API 時確保以下幾點:
  - 你的 API 只接受通過 JSON 編碼的 POST 發送的查詢
  - API 會驗證所提供的內容是否與所提供的 content type 相符
  - API 具有安全的 CSRF token


-------------------

## 防禦 GraphQL API 惡意查詢

GraphQL 可以隨時查詢自己想要的內容，但也會帶來複雜的安全問題
- 攻擊者可能發出昂貴的嵌套查詢，使你的 server, db, 網路或所有這些都超載
- 沒有保護，你就會面臨 DoS 攻擊

例如，在 Spectrum 的 GraphQL API 中有這樣的 relationship:

```
type Thread {
  messages(first: Int, after: String): [Message]
}

type Message {
  thread: Thread
}

type Query {
  thread(id: ID!): Thread
}
```


既可以查詢 thread 的 messages，也可以查詢 messages 的 thread
- 這種 circular relationship 允許攻擊者構建下面這樣的 expensive nested query
- 這種查詢會成倍增加載入對象的數量，導致 server 崩潰
 

```
query maliciousQuery {
  thread(id: "some-id") {
    messages(first: 99999) {
      thread {
        messages(first: 99999) {
          thread {
            messages(first: 99999) {
              thread {
                # ...repeat times 10000...
              }
            }
          }
        }
      }
    }
  }
}
```


-------------------


## 限制 Size
第一種最簡單的方法是按 raw bytes 限制傳入查詢的大小
- 由於查詢是以字串發送的，因此快速檢查長度就足夠了:
 

```js
app.use('*', (req, res, next) => {
  const query = req.query.query || req.body.query || '';
  if (query.length > 2000) {
    throw new Error('Query too large');
  }
  next();
});
```

不幸的是，這種方法在實務中並不奏效:
- 檢查可能會允許使用短的 field name，或錯誤阻止了使用長 field name 或合法的 nested fragments 查詢


-------------------


## 查詢白名單
第二種方法是 Whitelisting
- 列出 application 中批准的查詢，告訴 server 除了這些查詢外，不允許任何查詢通過
 


```js
app.use('/api', graphqlServer((req, res) => {
  const query = req.query.query || req.body.query;
  // TODO: Get whitelist somehow
  if (!whitelist[query]) {
    throw new Error('Query is not in whitelist.');
  }
  /* ... */
}));
```


手動維護 list 顯然是件麻煩事
- 幸好 Apollo 團隊開發了 `persistgraphql`，能自動從 client code 中提取所有查詢，產生漂亮的 JSON
- https://github.com/apollographql/persistgraphql


```json
{
  "scripts": {
    "postbuild": "persistgraphql src api/query-whitelist.json"
  }
}
```



這技術效果非常好，能可靠地阻止所有惡性查詢。但不幸的是，它也有兩個主要缺點:
1. 我們永遠無法更改或刪除查詢，只能添新增新的查詢
    - 如果任何 user 用的的是舊的的 client，我們就無法阻止他們的請求
    - 我們很可能需要保存 production 中使用過的所有查詢的歷史記錄，這就複雜得多了
2. 我們不能向公眾開放 API
    - 我們希望在未來某個時候向公眾開放 API，這樣其他開發人員就可以使用。如果只允許白名單的查詢，那就已經嚴重限制了他們的選擇，也違背了 GraphQL API 的初衷

這些都是我們無法接受的限制，所以我們需要想其他方案


-------------------

## Depth Limiting

`graphql-depth-limit` 能輕鬆限制傳入查詢的最大深度
- (我們檢查了我們的 client，發現使用的最深查詢有 7 層，因此我們將最大深度設為 10(相當寬鬆))
- https://www.npmjs.com/package/graphql-depth-limit

 

```js
app.use('/api', graphqlServer({
  validationRules: [depthLimit(10)]
}));
```


depth limiting 就是這麼簡單！



-------------------


## 數量限制
另一種惡意查詢是取得一個對象的 999999
- 無論對像是什麼，取得大量對象總是很昂貴的
- (DB 方面可能有技巧 or 內建優化能減輕 db 壓力，但網路和處理壓力卻無法減輕)


使用 `graphql-input-number` 建立了一個自定義標量
- 將最大值限制為 100，而不是將第一個參數的類型設置為 Int(允許任意數字):
- https://www.npmjs.com/package/graphql-input-number


```js
const PaginationAmount = GraphQLInputInt({
  name: 'PaginationAmount',
  min: 1,
  max: 100,
});
```
 
如果有人查詢超過 100 個對象，就會出錯:  

```
type Thread {
  messages(first: PaginationAmount, after: String): [Message]
}
```

現在，已經完全阻止了上面的惡意查詢！

-------------------

## 查詢成本分析
在適當的條件下，server 仍有可能不堪重負
- 某些特定 application 的查詢既不會太深入，也不會請求太多對象，但仍然非常昂貴


對於 Spectrum 來說，這樣的查詢可能是這樣的:
```
query evilQuery {
  thread(id: "54887141-57a9-4386-807c-ed950c4d5132") {
    messageConnection(first: 100) { ... }
    participants(first: 100) {
      threadConnection(first: 100) { ... }
      communityConnection { ... }
      channelConnection { ... }
      everything(first: 100) { ... }
    }
  }
}
```


這個查詢的深度和單個數量都不是特別高，但它可能會取得數以萬計的記錄
- 意味著它對 db, server 和網路的消耗都很大，這是最糟糕的情況



為了防止，我們需要分析查詢，計算複雜性，在代價過高時對其進行阻止
- 這比之前的兩種保護措施都要費事，但它能百分之百確保沒有惡意查詢能進入解析器


**在你花費大量時間實作查詢成本分析前，請先確定你是否需要它？**
- 測試用一個討厭的查詢讓你的 staging API 崩潰或變慢，看看你能做到什麼程度
- 也許你的 API 沒有這類嵌套關係，也許它完全可以一次處理成千上萬條記錄，不需要查詢成本分析


我在  2017 MacBook Pro 上本地執行上面查詢
- 我們的 API server 花了 10 ~ 15 秒才 response 1 megabyte 的 JSON
- 我們真的需要它，因為絕不希望任何人用它來轟炸我們的 API
- ([GitHub GraphQL API 也有用 Query Cost Analysis](https://docs.github.com/en/graphql/overview/resource-limitations))


**實作查詢成本分析:**  
- npm 上有幾個，其中兩個是
  - [graphql-validation-complexity](https://github.com/4Catalyzer/graphql-validation-complexity)
    - 即插即用的
  - [graphql-cost-analysis](https://github.com/pa-bru/graphql-cost-analysis)
    - 通過指定 `@cost` 指令讓你擁有更多控制權



Spectrum 使用 [graphql-cost-analysis](https://github.com/pa-bru/graphql-cost-analysis)
- 因為我們最快的解析器(20μs)和最慢的解析器(10s+)之間的差異很大，所以我們需要從中控制
- 話雖如此，也許 [graphql-validation-complexity](https://github.com/4Catalyzer/graphql-validation-complexity) 對你來說已經足夠了



它的工作方式是指定解析某個 field 或 type 的相對成本
- 它還支持乘法
- 因此，如果你 request 一個 list，其中的任何 nested field 都將乘以 pagination amount，這非常實用

這就是 `@cost` 指令的實際效果:
```
type Participant {
  # The complexity of getting one thread in a thread connection is 3, and multiply that by the amount of threads fetched
  threadConnection(first: PaginationAmount, after: String): ThreadConnection @cost(complexity: 3, multipliers: ["first"])
}

type Thread {
  author: Author @cost(complexity: 1)
  participants(first: PaginationAmount,...): [Participant] @cost(complexity: 2, multipliers: ["first"])
}
```



這只是我們 API types 的一個片段，但你可以理解其中的意思
- 你只需指定某個 fiwls 的複雜程度、要乘以的參數以及最大成本，剩下的就交給 `graphql-cost-analysis` 來完成

通過 Apollo Studio 的 performance tracking data 確定了某些解析器的複雜程度
- 我查看了整個模式，並根據 p99 服務時間分配了一個值。然後，查看了 client 上的所有查詢，找出了最昂貴的一個，它得到了 ~500 個複雜度點。為了給未來留有餘地，我們將最大復雜度設置為 750。



**結論:**  
- 我建議將 `Depth` 和 `Amount Limiting` 做為任何 GraphQL API 的最低保護措施
  - 它們很容易實現，並能提供足夠的安全性
- 根據你的具體安全要求和模式，你可能還需要研究查詢成本分析。雖然更費事，但它確實能提供全面的保護，防止攻擊者的攻擊