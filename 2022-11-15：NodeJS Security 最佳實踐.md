# 2022-11-15：Node.js Security Best Practices
## Node.js Security WorkGroup, Oct 28, 2022
### https://nodejs.org/zh-tw/docs/guides/security/
### https://github.com/nodejs/security-wg/issues/799

最近，Node.js Security WorkGroup 的一些關於 security 的討論，而目前為止的討論內容被官方彙整成文件  
看起來有針對特定的 security issue 做討論  
未來可能還會新增內容，可以再回去確認看看官網文件  

---------------------------------------

## 關於這份 Document
- Best practices: 簡化、濃縮方式來看待 best practices。可以使用[這個 issue](https://github.com/nodejs/security-wg/issues/488) 或 [nodebestpractices](https://github.com/goldbergyoni/nodebestpractices) 作為起點
  - 需要注意的是，這 doc 是專門針對 Node.js 的
  - 如果想找一些更廣泛的東西，可以看看 [OSSF Best Practices](https://github.com/ossf/wg-best-practices-os-developers)
- 攻擊解釋: 用通俗的說明，記錄在威脅模型中提到的攻擊（如果可能的話）和一些 code example
- Third-Party Libraries: 定義威脅（typo 攻擊、malicious packages...）和關於 node modules dependencies 的 best practices 等等

------------------------------------------------------------

## 威脅清單
### 1. 拒絕 HTTP server 的服務 (Denial of Service of HTTP server, CWE-400)
這是一種攻擊方法
- 由於 application 處理傳入的 HTTP requests 的方式，使其無法達到設計的目的
- 這些 requests 不需要由 attacker 刻意製作，一個 config 錯誤或有缺陷的 client 也可以向 server 發送一個 pattern of requests，從而導致拒絕服務

HTTP requests 由 Node.js HTTP server 接收，並通過註冊的請求處理程序移交給 application code
- server 並不解析 requests 內容。因此，任何由主體內容在移交給 requests 處理程序後引起的 DoS 都不是 Node.js 本身的漏洞
- 因為正確處理它是 application code 的責任

確保 WebServer 正確處理 `socket errors`，例如，當 server create 時沒有 error handling，將容易受到 DoS 攻擊
```js
const net = require('net');

const server = net.createServer(function(socket) {
  // socket.on('error', console.error) // this prevents the server to crash
  socket.write('Echo server\r\n');
  socket.pipe(socket);
});

server.listen(5000, '0.0.0.0');
```

如果執行了 bad request，server 可能會 crash  

[Slowloris](https://en.wikipedia.org/wiki/Slowloris_(computer_security))，一個不是由 request's contents 引起的 DoS 攻擊的例子
- 在這攻擊中，HTTP requests 被緩慢地、碎片化地發送，一次一個碎片
- 在完整的 request 被送達之前，server 將保持資源專門用於正在進行的 requests
- 如果在同一時間發送足夠多的這些 requests，concurrent connections 的數量將很快達到最大值，導致拒絕服務
- 這攻擊的方式，不取決於 requests 的內容，而是**取決於向 server 發送 requests 的時間和模式**

**緩解措施:**
- 用 reverse proxy 來接收和轉發對 Node.js application 的 requests
  - reverse proxy 可提供 caching, load balancing, IP blacklisting 等，這些都可以減少 DoS 攻擊的機率
- 正確的 config `server timeouts`，這樣就可以放棄空閒或 requests 到達過慢的連接
  - 看看 [http.Server](https://nodejs.org/api/http.html#class-httpserver)，尤其是 `headersTimeout`, `requestTimeout`, `timeout`, and `keepAliveTimeout`.
- 限制 the number of open sockets per host 和總數
  - 看看 [http docs](https://nodejs.org/api/http.html), 尤其是 `agent.maxSockets`, `agent.maxTotalSockets`, `agent.maxFreeSockets` and `server.maxRequestsPerSocket`.

------------------------------------------------------------

### 2. DNS Rebinding (CWE-346)
這攻擊針對正在使用 [--inspect switch](https://nodejs.org/en/docs/guides/debugging-getting-started/) 啟用 debugging inspector 的 Node.js application

由於 browser 打開的網站可以進行 WebSocket 和 HTTP requests
- 它們可以針對本地運行的 debugging inspector
- 這通常被 browser 的 [same-origin policy](https://nodejs.org/en/docs/guides/debugging-getting-started/) 所阻止
  - 該策略禁止 script 從不同的 origins 來取得資源

然而，通過 DNS Rebinding
- attacker 可以暫時控制其請求的來源，使其看起來是來自本地的 IP
- 這是通過控制一個網站和用於解決其 IP 的 DNS server 來實現的
  - ([DNS Rebinding wiki](https://en.wikipedia.org/wiki/DNS_rebinding))

**緩解措施:**
- 加一個 `process.on(‘SIGUSR1’, …)` 監聽器，在 `SIGUSR1` 訊號時，disable inspector
- 不要在 production 中執行 inspector

------------------------------------------------------------

### 3. 將敏感資訊暴露給未經授權的人 (CWE-552)
在 package publish 過程中，所有包含在當前目錄中的檔案和資料夾都被送到 npm registry
- 通過 `.npmignore` and `.gitignore` or by defining an allowlist in the `package.json` 可以控制這種行為

**緩解措施:**
- 使用 `npm publish --dry-run` 列出所有要 publish 的檔案，確保在 publish 前確認過內容
- 建立和維護忽略文件也很重要，如 `.gitignore` 和 `.npmignore`。通過這些文件，你可以指定哪些不應該被發布
  - `package.json` 中的 [files property](https://docs.npmjs.com/cli/v8/configuring-npm/package-json#files) 允許反向操作
- 如果在已經暴露的情況下，確保 [unpublish the package](https://docs.npmjs.com/unpublishing-packages-from-the-registry)

------------------------------------------------------------

### 4. HTTP Request Smuggling (CWE-444)
這是一種涉及兩個 HTTP servers（通常是一個 proxy 和一個 Node.js application）的攻擊
- client 發送的 HTTP request 首先經過 proxy，然後被 redirect 到 backend server
- 當兩台 server 對模棱兩可的 HTTP request 有不同的解釋時，attacker 就有可能發送一個惡意 message
- 該 message 不會被第一台看到，但會被第二台看到，有效的「偷渡(Smuggling)」過 proxy

看看 [CWE-444](https://cwe.mitre.org/data/definitions/444.html)，更詳細的描述和例子  
(之前讀的 [2021 Top10 web hacking](<./2022-03-31：James Kettle Top 10 web hacking techniques of 2021.md>) 有提到，James Kettle 說在 2021 年時，這漏洞有明顯的成長趨勢)


由於這攻擊依賴 Node.js 對 HTTP request 的解釋與（任意的）HTTP server 不同
- 成功的攻擊可能是由於 Node.js、前端服務器或兩者中的漏洞
- 如果 Node.js 解釋 request 的方式與 HTTP 規範一致(see [RFC7230](https://datatracker.ietf.org/doc/html/rfc7230#section-3))，那麼就不認為這是 Node.js 的漏洞
- (我: 還有一種是針對 `HTTP/1` 與 `HTTP/2` 的不一致所達成的 Smuggling，上面 Top10 的文章裡就有收入一篇研究 case

**緩解措施:**
- 啟動 HTTP server 時不要使用 `insecureHTTPParser` option
- Config 前端 server，來做到 normalize 模棱兩可的 request
- 在 Node.js 和選擇的前端 server 中持續監測新的 HTTP request smuggling vulnerabilities
- 在 end to end 時，都使用 `HTTP/2`，可能的話，禁用 HTTP downgrading


------------------------------------------------------------

### 5. 透過 Timing Attacks 來揭露 Information (CWE-208)
這是種允許 attacker 了解潛在的敏感資訊的攻擊
- 例如，測量 application response request 所需的時間
- 這攻擊不是專門針對 Node.js 的

只要 application 在 timing-sensitive operation (如分支)中有使用到 secret，就有可能受到攻擊
- 考慮在典型的 application 中處理認證。這裡，一個基本的認證方法包括電子郵件和密碼作為憑證
- User 資訊從 User 提供的輸入中檢索出來，最好是在 DBMS 中
- 在檢索 User 資訊時，密碼要與從 DB 中檢索的 User 資訊進行比較
- 對於相同長度的值，使用 built-in string comparison 需要更長的時間
- 這種比較，當運行到一個可接受的數量時，會無可避免的增加 request 的 response 時間
- 通過比較 request 的 response 時間，attacker 可以在大量的 request 中猜出密碼的長度和值

**補救措施:**
- `crypto` API 有個 function [timingSafeEqual](https://nodejs.org/api/crypto.html#cryptotimingsafeequala-b)，使用恆定時間算法比較實際和預期的敏感值
- 對於密碼比較，可以使用 [scrypt](https://nodejs.org/api/crypto.html#cryptoscryptpassword-salt-keylen-options-callback)
- 避免在 variable-time operations 中使用 secrets。這包括在秘密上的 branching，以及當 attacker 可能在同一基礎設施（例如，同一 cloud）上共處時，使用 secrets 作為 memory 的 index。在 JavaScript 中寫恆定時間 code 是困難的（部分原因是 JIT）
  - 對於 crypto 應用，使用built-in crypto APIs or WebAssembly（針對原生還沒有實現的 algorithms）


------------------------------------------------------------

### 6. 惡意的 Third-Party Modules (CWE-1357)
Node.js 中，任何 package 都可以訪問網路，package 還可以訪問文件系統，因此可以將任何 data 發送到任何地方  

所有運行到 node process 中的 code 都能使用 `eval()` (或其等價物)載入和執行額外的任意 code
- 所有具有 file system write 權限的 code 都可以通過寫入被載入的新文件或現有文件來實現同樣的事情
- Node.js 有個實驗性的 [policy mechanism](https://nodejs.org/api/permissions.html#policies) 來聲明載入的資源可不可信任
  - 這個策略在**默認情況下是不啟用的**
- 確保鎖住 dependency versions，並使用常見的工作流程或 npm scripts 自動檢查漏洞
- 安裝 package 之前，確保這個 package 得到了維護，並且包括了你預期的所有內容
- 要小心，Github 的 source code 不一定和發布的一樣，在 `node_modules` 中驗證一下

#### Supply chain 攻擊
供應鏈攻擊發生在 Node.js application 的 dependencies（無論是直接的還是橫向的）被破壞時
- 這可能是 application 對 dependencies 的規範過於寬鬆(允許不需要的更新)和/或常見的 typos
  - （[typosquatting wiki](https://en.wikipedia.org/wiki/Typosquatting)）

attacker 如果控制了一個上游 package
- 就可以發布含有惡意 code 的新版本
- 如果 Node.js application 依賴該 package，而沒有嚴格規定使用哪個版本，那 package 可以自動更新到最新的惡意版本，危及該 application

在 `package.json` 中指定 dependencies 確切的版本號或範圍
- 然而，當釘住 dependencies 的確切版本時，它的交叉 dependencies 本身並沒有被釘住
- 這使 application 容易受到不需要的/意外的更新

Possible vector attacks:
- Typosquatting
- Lockfile poisoning
- 惡Malicious Packages
- 被破壞的維護者(Compromised maintainers)
- 依賴性混淆(Dependency Confusions)

**補救措施:**
- 用 `--ignore-scripts` 防止 npm 執行任意 scripts
  - 此外，可以用 `npm config set ignore-scripts true` disable it globally
- 將 dependency 固定在特定不可改變的版本上，而不是一個範圍內的版本
- 用 `lockfiles` 釘住每一個 dependency (direct and transitive)
  - 即使有使用了 `lockfiles` 也不是萬無一失: [lockfile poisoning](https://blog.ulisesgascon.com/lockfile-posioned)
- 使用 CI 自動檢查新的漏洞，如 `npm-audit`
  - 使用 [Socket](https://socket.dev/) 等工具對 packages 進行靜態分析，發現網路或文 filesystem access 等危險行為
- 用 [npm ci](https://docs.npmjs.com/cli/v8/commands/npm-ci) 而不是`npm install`
  - 這將強制執行 lockfile，這樣它和 `package.json` 間的不一致會導致錯誤，而不是默默地忽略 lockfile
- 仔細檢查 `package.json` 中的錯誤/依賴的名字

------------------------------------------------------------

### 7. 違規的 Memory Access (CWE-284)
Memory-based 或 heap-based 的 attacks 依賴在 memory management 錯誤和可利用的 memory allocator 的相結合
- 像所有的 runtimes 一樣，如果 projects 運行在共享機器上，Node.js 就容易受到這些攻擊
- `secure heap` 對於防止敏感資訊因 pointer overruns and underruns 而洩露是很有用的

不幸的是，Windows 上不能用 `secure heap`
- 更多資訊: [secure-heap documentation](https://nodejs.org/dist/latest-v18.x/docs/api/cli.html#--secure-heapn)

**補救措施:**
- 根據你的 application 使用 `--secure-heap=n`，其中 n 是分配的 maximum byte size
- 不要在共享機上運行你的 production runtime


------------------------------------------------------------

### 8. Monkey Patching (CWE-349)

`Monkey patching` (猴子補丁)是指在運行時修改屬性，以改變現有行為。例如:

```js
// eslint-disable-next-line no-extend-native
Array.prototype.push = function (item) {
  // overriding the global [].push
};
```

**補救措施:**
- `--frozen-intrinsics` 啟用了實驗性 frozen intrinsics，所有 built-in JavaScript objects and functions 被遞歸 frozen

因此，下面不會覆蓋 `Array.prototype.push`:
```js
// eslint-disable-next-line no-extend-native
Array.prototype.push = function (item) {
  // overriding the global [].push
};

// Uncaught:
// TypeError <Object <Object <[Object: null prototype] {}>>>:
// Cannot assign to read only property 'push' of object ''
```

但，你仍然可以使用 `globalThis` 來定義新的 globals 和替換現有的 globals

```
> globalThis.foo = 3; foo; // you can still define new globals
3
> globalThis.Array = 4; Array; // However, you can also replace existing globals
4
```

因此，`Object.freeze(globalThis)` 可以用來保證沒有 globals 被替換

------------------------------------------------------------

### 9. Prototype Pollution 攻擊 (CWE-1321)

Prototype pollution 是指通過濫用 `_proto_`, `constructor`, `prototype` 和其他從 built-in prototypes 繼承的屬性，來修改或 inject Javascript 的可能性。

```js
const a = {"a": 1, "b": 2};
const data = JSON.parse('{"__proto__": { "polluted": true}}');

const c = Object.assign({}, a, data);
console.log(c.polluted); // true

// Potential DoS
const data2 = JSON.parse('{"__proto__": null}');
const d = Object.assign(a, data2);
d.hasOwnProperty('b'); // Uncaught TypeError: d.hasOwnProperty is not a function
```

**Examples:**
- [CVE-2022-21824](https://www.cvedetails.com/cve/CVE-2022-21824/) (Node.js)
- [CVE-2018-3721](https://www.cvedetails.com/cve/CVE-2018-3721/) (3rd Party library: Lodash)

**補救措施:**
- 避免 [insecure recursive merges](https://gist.github.com/DaniAkash/b3d7159fddcff0a9ee035bd10e34b277#file-unsafe-merge-js)
  - see [CVE-2018-16487](https://www.cve.org/CVERecord?id=CVE-2018-16487)
- 對外部/不信任的 requests 做 JSON Schema 驗證
- 使用 `Object.create(null)` 建立沒有 prototype 的 Objects
- 凍結 prototype `Object.freeze(MyObject.prototype)`
- 使用 `--disable-proto` 禁用 `Object.prototype.__proto__` 屬性
- 使  `Object.hasOwn(obj, keyFromObj)` 檢查該屬性是否直接存在於對像上，而不是來自 prototype
- 避免使用 `Object.prototype` 中的 methods

------------------------------------------------------------

### 10. 不受控制的 Search Path Element (CWE-427)
Node.js 按照 [Module Resolution Algorithm](https://nodejs.org/api/modules.html#modules_all_together) 來載入 modules
- 因此，它假設 requested module 的目錄(require)是可信的

通過這一點，它意味著以下 application 行為是可以預期的。假設目錄結構如下:
```
app/
  server.js
  auth.js
  auth
```
如果 server.js 使用 `require('./auth')`，它將遵循 `Module Resolution Algorithm`，載入 `auth` 而不是 `auth.js`


**補救措施:**
- 使用實驗性的 [policy mechanism with integrity checking](https://nodejs.org/api/permissions.html#integrity-checks) 
  - 可以避免上述威脅。對於上述的目錄，可以使用以下的 `policy.json`

```json
{
  "resources": {
    "./app/auth.js": {
      "integrity": "sha256-iuGZ6SFVFpMuHUcJciQTIKpIyaQVigMZlvg9Lx66HV8="
    },
    "./app/server.js": {
      "dependencies": {
        "./auth" : "./app/auth.js"
      },
      "integrity": "sha256-NPtLCQ0ntPPWgfVEgX46ryTNpdvTWdQPoZO3kHo0bKI="
    }
  }
}
```

因此，當要求使用 auth module 時，系統將驗證完整性
- 如果與預期的不一致，則拋出一個錯誤

```
» node --experimental-policy=policy.json app/server.js
node:internal/policy/sri:65
      throw new ERR_SRI_PARSE(str, str[prevIndex], prevIndex);
      ^

SyntaxError [ERR_SRI_PARSE]: Subresource Integrity string "sha256-iuGZ6SFVFpMuHUcJciQTIKpIyaQVigMZlvg9Lx66HV8=%" had an unexpected "%" at position 51
    at new NodeError (node:internal/errors:393:5)
    at Object.parse (node:internal/policy/sri:65:13)
    at processEntry (node:internal/policy/manifest:581:38)
    at Manifest.assertIntegrity (node:internal/policy/manifest:588:32)
    at Module._compile (node:internal/modules/cjs/loader:1119:21)
    at Module._extensions..js (node:internal/modules/cjs/loader:1213:10)
    at Module.load (node:internal/modules/cjs/loader:1037:32)
    at Module._load (node:internal/modules/cjs/loader:878:12)
    at Module.require (node:internal/modules/cjs/loader:1061:19)
    at require (node:internal/modules/cjs/helpers:99:18) {
  code: 'ERR_SRI_PARSE'
}
```


### Production 中使用 experimental features
不建議在 production 中使用 `experimental features`
- `Experimental features` 在需要時可能會遭受破壞性的改變
- 而且它們的功能並不安全穩定
- 建議使用 `--policy-integrity` 來避免 policy mutations
