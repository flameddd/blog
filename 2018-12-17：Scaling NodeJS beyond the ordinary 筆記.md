## JSConf Iceland 2018：Scaling NodeJS beyond the ordinary
## 講者 Abhinav Rastogi
## 來源 https://www.youtube.com/watch?v=K8spO4hHMhg

Abhinav Rastogi 處理的電商有這樣的規模：
 - 1億5000萬的註冊會員
 - 每15秒賣出50萬支手機
 - 年度特賣活動時，超過5天有2億的瀏覽次數。

這邊要講的是，除了`水平擴容`、`垂直擴容`之外，Application layer 可以做些什麼？

能下手分析的基本上就這4個
 - network
 - cpu
 - memory
 - disk

## network bandwidth
現在用 nodejs 來當 server時，
1000kb per page、100 rps per machine、100 machines (cluster) = 10gbps  
用 compression 來 reduce data 輕鬆解決此問題。
```javascript
const app = express()
app.use(compression)
```
但這不是好方法，因為 compression 是 high cpu cost 的操作、NodeJS 是 single thread。  
NodeJS 在執行 compression 時無法處理其他 request。

解法是 `co-hosted nginx`，也就是做，另外架一層（通常是順便用來反向代理）來幫忙 compression。
epxressjs 官網也有提到，用 nginx 來做處理效能會更好。(static file 也是讓反向代理處理會更好)

> [expressjs best-practice-performance](https://expressjs.com/zh-tw/advanced/best-practice-performance.html)：
在正式作業中，如果網站的資料流量極高，落實執行壓縮最好的作法是在反向 Proxy 層次實作它（請參閱使用反向 Proxy）。在該情況下，就不需使用壓縮中介軟體。如需在 Nginx 中啟用 gzip 壓縮的詳細資料，請參閱 Nginx 說明文件中的 ngx_http_gzip_module 模組。

另外還有一招，是打包檔案時，直接打包成 gzip file，而 expressjs 加入個 route 就行了。
```javascript
// use the gzipped bundle
app.get('*.js', (req, res, next) => {
  req.url = req.url + '.gz'; // eslint-disable-line
  res.set('Content-Encoding', 'gzip');
  next();
});
```

另外這些 js file 也要做 cache。（甚至用server worker）

![image info](./assets/img/co-hosted_nginx.png)

重點還是要做 `network profiling` 來看數據、效能。
```
netstat / ss
lsof
watch
```
原來有一個 `Socket Statistics（簡稱 ss）` 的強大工具的樣子。大家都說，比起 `netstat` 更強大，尤其當 socket 連線數上萬時。
如果沒辦法用 `ss` 指令，可能要裝一下 `iproute`。
```
ss -atr
```
另外有一套 `iproute2` 看來也不錯。  
結果 mac 好像沒有 ss  
總之 上面的結論是，你需要有做 `network profiling`

## utlimit
- max users processes

increase the limits / reduce the usage  
調整 utlimit 的 limit。

## keep-alive header
讓 user 不用一直重複 request `hand shake`。server 這邊設 header。

```js
Object.assign(header, {
  'Connection': 'keep-alive',
  'Keep-Alive': 'timeout=200'
})
```

## connection pooling
connection on server side。
減少連 server 的次數
```js
const http = require('http')

fetch(url, {
  agent: new http.Agent({
    keepAlive: true,
    maxSockets: 24
  })
})
```

## ephemeral ports
tcp connection states  
聽不懂...他提到 linux 的 tcp 運作機制，但這要怎麼改善呢？  
接續就講到 cpu 了。

## cpu profiling
用 nodejs 自身的功能 profiling，加 prof flag 來執行。
> node --prof app.js

執行後，目錄底下會產生一個這樣的檔案
> isolate-0xnnnnnnnnnnnn-v8.log

這時候用 prof-process 轉換統計數字
> node --prof-process ./isolate-0x103800000-v8.log > processed.txt

但我執行到邊，轉出來檔案都有問題。似乎是 nodejs 版本更換，格式也不同了。一時還沒找到解法。
有人說可以用 Chrome 的 `chrome://tracing` 來載入 log 欓，但我載入了也是沒東西ＱＱ

影片中用這段程式碼舉例
```js
import crypto from 'crypto';

app.get('/auth', (req, res) => {
  const hash = crypto.pbkdf2Sync(password, users[username].salt, 100, 512);

  if(users[username].hash.toString() === hash.toString()) {
    res.sendStatus(200)
  } else {
    res.sendStatus(401)
  }
})
```

用 prof 最後轉換出的 processed.txt 有這樣的資料
[Summary]:
| ticks | total | nonlib | name |
| ------: | ------: | ------: | :------ |
|79|0.2%|0.2%|JavaScript|
|36703|97.2%|99.2%|C++|
|7|0.0%|0.0%|GC|
|767|2.0%||Shared libraries|
|215|0.6%||Unaccounted|

這個告訴我們，百分之 99 的 CPU 時間都是 C++ 那邊的處理用掉的，不是 JavsScript 這塊。  

繼續挖  
[C++]:
| ticks | total | nonlib | name |
| ------: | ------: | ------: | :------ |
|19557|51.8%|52.9%|node::crypto::PBKDF2(v8)|
|4510|11.9%|12.2%|_sha1_block_data_order|
|2165|8.4%|8.6%|_malloc_zone_malloc|

看到這個，說不定我們改用其他 lib 說不定效果能改善 or 把功能拆出成另一個獨立 server，這樣我們原本的 server 能接更多的 request。

## 0x

```sh
npm install -g 0x
0x app.js
```
產火焰圖

## node-clinic https://github.com/nearform/node-clinic
之前自己玩的時候，是用這套
產火焰圖、另外還有分析圖可以看！！

## ApacheBench 
測 connection 的話，就還是老牌 ab
1. 同時 10 個連線，連續點擊 10000 次 ( 每個 Request 執行完畢後都會自動斷線，然後再重新連線 )
> ab -n 10000 -c 10 http://www.example.com/index.aspx

2. 同時 10 個連線，連續點擊 10000 次，並且使用 Keep-Alive 方式連線（當 Web Server 有支援 Keep-Alive 功能時 ApacheBench 會在同一個連線下連續點擊該網頁）
> ab -n 10000 -c 10 -k http://www.example.com/index.aspx

3. 將測試的效能原始資料匯出成 CSV 檔
> ab -e output.csv -n 10000 -c 10 http://www.example.com/index.aspx

## disk
## memory
聽不懂。 監控 I/O 狀況來看結果
監控有沒有 memory leak

## real-time monitoring
確保 CPU 不是 100%，至少要有 10% 可用。

> load, profile, fix, repeat！