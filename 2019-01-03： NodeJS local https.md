## local 測試時，使用 https
有些時候可能需要，例如我這次測試`微信desktop`時，要用內建的功能開啟網頁時，它只支援 `https`

### 產生 key
先在該目錄下，先產生 key （指令如下）
> openssl req -nodes -new -x509 -keyout server.key -out server.cert

### NodeJS 部分
除了 http 之外，另外起 https server 並 proxy 過去
```js
const https = require('https');
const httpProxy = require('http-proxy');
const fs = require('fs');

const proxy = httpProxy.createProxyServer({});

https.createServer({
  key: fs.readFileSync('./server.key'), // 確認這目錄是上個步驟所產生的檔案
  cert: fs.readFileSync('./server.cert'), // Windows、Linux 的目錄位置可能不同
},
(req, res) => {
  proxy.web(req, res, {
    target: 'http://localhost:3014', // 指到 http server 的 位置去
  });
},
app)
.listen(8009, function () {})
```

### 最後這樣訪問
 > https://127.0.0.1:8009


### 完整 code
```js
/* eslint consistent-return:0 */

const express = require('express');
const https = require('https');
const httpProxy = require('http-proxy');
const fs = require('fs');
const path = require('path');
const session = require('express-session');
const bodyParser = require('body-parser');
const cookieParser = require('cookie-parser');
const passport = require('passport');
const logger = require('./logger');
const argv = require('./argv');
const port = require('./port');
const setup = require('./middlewares/frontendMiddleware');
const apiRouter = require('./api');
const auth = require('./middlewares/authMiddleware');
const detailRouter = require('./detail');
const { resolve } = require('path');

const RedisStore = require('connect-redis')(session);

const redisHost = process.env.REDIS_HOST || 'localhost';
const redisPort = process.env.REDIS_PORT || '6379';
const redisPass = process.env.REDIS_PASS || '';

const proxy = httpProxy.createProxyServer({});


const app = express();
app.enable('trust proxy');

app.use((req, res, next) => {
  if (/.chunk.js$/.test(req.url) && process.env.NODE_ENV !== 'development') {
    res.set({ 'Cache-Control': 'public, max-age=2592000000' });
  } else {
    res.set({ 'Cache-Control': 'public, max-age=0' });
  }
  next();
});

app.use(
  '/assets',
  express.static(path.resolve(process.cwd(), 'assets'), {
    maxAge: 24 * 3600,
  }),
);

app.set('etag', false);
app.use(
  session({
    store: new RedisStore({
      host: redisHost,
      port: redisPort,
      pass: redisPass,
    }),
    resave: false,
    saveUninitialized: false,
    secret: 'simple-reportw',
    cookie: {
      secure: false,
    },
  }),
);
app.use((req, res, next) => {
  if (!req.session) {
    throw new Error('Redis connect error');
  }
  next();
});

app.use(bodyParser.urlencoded({ extended: true, limit: '50mb' }));
app.use(bodyParser.json());
app.use(cookieParser());


// If you need a backend, e.g. an API, add your custom backend-specific middleware here
app.use('/api', apiRouter);
app.use('/articles', detailRouter);

app.use('/reports/:reportId', auth());

// In production we need to pass these values in instead of relying on webpack
setup(app, {
  outputPath: resolve(process.cwd(), 'build'),
  publicPath: '/',
});

// get the intended host and port number, use localhost and port 3014 if not provided
const customHost = argv.host || process.env.SR_HOST;
const host = customHost || null; // Let http.Server use its default IPv6/4 host
const prettyHost = customHost || 'localhost';

// Start your app.
app.listen(port, host, async err => {
  if (err) {
    return logger.error(err.message);
  }

  logger.appStarted(port, prettyHost);
});

https.createServer({
  key: fs.readFileSync('./server.key'),
  cert: fs.readFileSync('./server.cert')
},
(req, res) => {
  proxy.web(req, res, {
    target: 'http://localhost:3014',
  });
},
app)
.listen(8009, function () {})

```
