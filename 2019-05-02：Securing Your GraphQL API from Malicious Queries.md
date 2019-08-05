## Securing Your GraphQL API from Malicious Queries
### Max Stoiber Feb 22, 2018
#### https://blog.apollographql.com/securing-your-graphql-api-from-malicious-queries-16130a324a6b

GraphQL 雖然方便，但惡意 actor could submit 也是 expensive 的  
nested 的 query 給 server, database, network 負擔很大

### example1

```js
type Thread {
  messages(
    first: Int,
    after: String
  ): [Message]
}

type Message {
  thread: Thread
}

type Query {
  thread(id: ID!): Thread
}
```

可以 query
- the messages of a thread

或者
- the thread of a message

這種`循環關係`就能讓 `bad actor` 去 construct 一個 an expensive nested query  

```js
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

上面這種結果非常差，三兩下就會 crash server

### Size Limiting
最直接的方法，limit incoming query size by raw bytes
因為 `query` 是用 `string` 發送，所以，檢查長度就有效
```js
app.use('*', (req, res, next) => {
  const query = req.query.query || req.body.query || '';
  if (query.length > 2000) {
    throw new Error('Query too large');
  }
  next();
});
```

但這樣還不夠

### Query Whitelisting
第二招，`whitelist`，過濾 query  

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

`whitelist` 用 `persistgraphql` 來自動產生  
automatically extracts all queries from your client-side code and generates a nice JSON file out of it  
```json
{
  "scripts": {
    "postbuild": "persistgraphql src api/query-whitelist.json"
  }
}
```

### Unfortunately, it also has two major tradeoffs

1. 我們沒辦法 change or delete queries，只能新增，因為如果有 Uesr 用 舊的 client 就會有問題
2. 沒辦法 open API 出去，whitelist 等於限制了其他 develop 的開發的可能性。

所以考慮下面這些方式

### Depth Limiting
nesting 就是其中一種差的 query 方式，每多一層，後端負擔就更大  
用 `graphql-depth-limit` 限制 queries 的 depth  
作者確認他們的應用最多7層，所以這邊設 10 層  
```js
app.use('/api', graphqlServer({
  validationRules: [depthLimit(10)]
}));
```
 
### Amount Limiting
另一種有害的就是 fetching 99999 of an object  
第一個參數 `Int` 可以設定，作者自己寫了一個 `graphql-input-number`  
```js
const PaginationAmount = GraphQLInputInt({
  name: 'PaginationAmount',
  min: 1,
  max: 100,
});
```

當 queries 超過 100 objects 時，就會 will throw an error  

```js
type Thread {
  messages(first: PaginationAmount, after: String): [Message]
}
```

### Query Cost Analysis
還是有特定的 queries that are neither too deep nor request too many objects but are still very expensive

```js
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

他們的 case ，這個查訊還是有可能查詢大量資料  
要避免這個，需要做 `analyze queries`  
這個工比較多，這 100% 可以避免惡意 query  
**需要花很多時間來執行 query cost analysis**  
試 nasty query 去 crash or slow dowe 你的 `staging API`  
也許你的 API 沒有這種 nested relationships，或者你的機器受的了大量查詢  

github 也有做做過分析
- https://developer.github.com/v4/guides/resource-limitations/

### Implementing Query Cost Analysis
有很多套件可以分析
- graphql-validation-complexity
- graphql-cost-analysis

作者選 `graphql-cost-analysis` 這套，說是因為他們 case 最快、最慢差很多
- 20μs, 10s+

所以需要能夠 control we got from it (大概是這套能更多控制)  
This is what the @cost directive looks like in practice

```js
type Participant {
  # The complexity of getting one thread in a thread connection is 3, and multiply that by the amount of threads fetched
  threadConnection(first: PaginationAmount, after: String): ThreadConnection @cost(complexity: 3, multipliers: ["first"])
}

type Thread {
  author: Author @cost(complexity: 1)
  participants(first: PaginationAmount,...): [Participant] @cost(complexity: 2, multipliers: ["first"])
}
```

### Summary
用
- Depth Limiting
- Amount Limiting
作為最基本的保護。

根據你自己的情況，某些特定的  security requirements and schema 就需要
- Query Cost Analysis

這需要花時間研究，但確保能避免 malicious actors
