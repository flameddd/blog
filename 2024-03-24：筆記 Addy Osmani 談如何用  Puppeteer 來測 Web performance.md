# 2024-03-24：筆記 Addy Osmani 談如何用  Puppeteer 來測 Web performance
## Addy Osmani, APRIL 28, 2020
### https://addyosmani.com/blog/puppeteer-recipes/

----------------


## Trace page load 時的 Web performance

用 `tracing.start()` 就能對頁面做基本的 Web performance 測量
- 產生出來的 `profile.json` 能用 devtool `performance` tab 去載入來看
- 要 screenshot 的話，多下個 `screenshots` 參數

```js
import puppeteer from 'puppeteer';

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  await page.tracing.start({ path: 'profile.json' });
  // 如果要 screenshot 的話，多給個參數就可以了
  // await page.tracing.start({ path: 'profile.json', screenshots: true });

  await page.goto('https://pptr.dev');
  await page.tracing.stop();

  await browser.close();
})();
```

![](https://addyosmani.com/assets/images/Performance-tracing0.png)  



------------

## 匯出 filmstrip 的 screenshots

上面的 code，只會輸出一個 json 檔案，如果想要把 screenshot 弄出來的話，可以參考這段 code
- 工作原理是過濾 screenshot entries 的 trace events

```js

const fs = require('fs');

// Extract data from the trace
const tracing = JSON.parse(fs.readFileSync('./trace.json', 'utf8'));
const traceScreenshots = tracing.traceEvents.filter(x => (
    x.cat === 'disabled-by-default-devtools.screenshot' &&
    x.name === 'Screenshot' &&
    typeof x.args !== 'undefined' &&
    typeof x.args.snapshot !== 'undefined'
));

traceScreenshots.forEach(function(snap, index) {
  fs.writeFile(`trace-screenshot-${index}.png`, snap.args.snapshot, 'base64', function(err) {
    if (err) {
      console.log('writeFile error', err);
    }
  });
});
```

------------

## 測量某些 user 行為
其實就是基本 puppeteer 基本操作，只是一樣利用 `tracing` 把動作行為包起來

```js
import puppeteer from 'puppeteer';

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  const navigationPromise = page.waitForNavigation();
  await page.goto('https://pptr.dev/#?product=Puppeteer&version=v2.1.1&show=outline');
  await page.setViewport({ width: 1440, height: 714 });

  await navigationPromise;
  const selector = 'body > sidebar-component > sidebar-item:nth-child(3) > .pptr-sidebar-item';
  await page.waitForSelector(selector);

  // 重點就這邊
  await page.tracing.start({path: 'trace.json', screenshots: true});
  await page.click(selector);
  await page.tracing.stop();

  await browser.close();
})();
```

------------

## 測量 Runtime performance

`page.metrics()`
- 這是利用 `Chrome DevTools Protocol` 的 `Performance.getMetrics()`
- 會回傳一些 runtime performance 數值
  - 如 `layout duration`, `recalc-style durations` and JS event listeners.
  
```js
import puppeteer from 'puppeteer';

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://pptr.dev');

  const metrics = await page.metrics();
  console.info(metrics);

  await browser.close();
})();
```

結果
```
{
  Timestamp: 843338.01459,
  Documents: 3,
  Frames: 1,
  JSEventListeners: 0,
  Nodes: 1064,
  LayoutCount: 2,
  RecalcStyleCount: 2,
  LayoutDuration: 0.016219,
  RecalcStyleDuration: 0.002081,
  ScriptDuration: 0.00777,
  TaskDuration: 0.066671,
  JSHeapUsedSize: 1323820,
  JSHeapTotalSize: 3866624
}
```


------------

## 產生 Lighthouse report
下面只是基本舉例，有更多的操作可以操作，可以參考文件
- ref: https://github.com/GoogleChrome/lighthouse/tree/main/docs

(需要另外安裝 `lighthouse` 和 `request` package)

```js
import fs from 'fs';
import lighthouse from 'lighthouse';
import * as chromeLauncher from 'chrome-launcher';

const chrome = await chromeLauncher.launch({ chromeFlags: ['--headless'] });
// const options = { logLevel: 'info', output: 'json', onlyCategories: ['performance'], port: chrome.port };
const options = { logLevel: 'info', output: 'html', onlyCategories: ['performance'], port: chrome.port };
const runnerResult = await lighthouse('https://remix-clone-hacker-news.flameddd1.workers.dev/', options);

// `.report` is the HTML report as a string
const reportHtml = runnerResult.report;
// fs.writeFileSync('lhreport.json', reportHtml);
fs.writeFileSync('lhreport.html', reportHtml);



// `.lhr` is the Lighthouse Result as a JS object
console.log('Report is done for', runnerResult.lhr.finalDisplayedUrl);
console.log('Performance score was', runnerResult.lhr.categories.performance.score * 100);

await chrome.kill();

```

許多細部參數可以調整，調整畫面大小、限制網路速度等等
```js
{
  extends: 'lighthouse:default',
  settings: {
    onlyAudits: [
      'first-meaningful-paint',
      'speed-index',
      'interactive',
    ],
  },
}
```

`output` 可以改為 `json`，這樣就方便用 code 去存取某些參數

```js
  const audits = JSON.parse(runnerResult.report).audits; // Lighthouse audits
  const first_contentful_paint = audits['first-contentful-paint'].displayValue;
  const total_blocking_time = audits['total-blocking-time'].displayValue;
  const time_to_interactive = audits['interactive'].displayValue;
```


------------

## 模擬 `Slow 3G` 網路速度的情況 

```js
import puppeteer from 'puppeteer';

(async () => {
  let browser = await puppeteer.launch();
  let page = await browser.newPage();

  const client = await page.createCDPSession();
  await client.send('Network.enable');
  // Simulated network throttling (Slow 3G)
  await client.send('Network.emulateNetworkConditions', {
  // Network connectivity is absent
  'offline': false,
  // Download speed (bytes/s)
  'downloadThroughput': 500 * 1024 / 8 * .8,
  // Upload speed (bytes/s)
  'uploadThroughput': 500 * 1024 / 8 * .8,
  // Latency (ms)
  'latency': 400 * 5
});
  await browser.close();
})();
```

其他網路選項的設定可以參考
- https://github.com/ChromeDevTools/devtools-frontend/blob/c4d1bd47f7e6244b38431b3409ed5cb6539ad957/front_end/core/sdk/NetworkManager.ts#L388-L410



`Slow 3G` 加上 `CPU throttling` (速度減慢 4 倍 - 接近 Moto G4 等中等品質的裝置)

```js
import puppeteer from 'puppeteer';

(async () => {
  let browser = await puppeteer.launch();
  let page = await browser.newPage();

  const client = await page.createCDPSession();
  await client.send('Network.enable');
  // Simulated network throttling (Slow 3G)
  await client.send('Network.emulateNetworkConditions', {
  // Network connectivity is absent
  'offline': false,
  // Download speed (bytes/s)
  'downloadThroughput': 500 * 1024 / 8 * .8,
  // Upload speed (bytes/s)
  'uploadThroughput': 500 * 1024 / 8 * .8,
  // Latency (ms)
  'latency': 400 * 5
});
  await client.send('Emulation.setCPUThrottlingRate', { rate: 4 });
  await browser.close();
})();
```

------------

## Test 網站在 JavaScript disabled 的情況下 render 的情況如何

```js
import puppeteer from 'puppeteer';

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.setRequestInterception(true);

  // 把 script 全部 abort 掉
  page.on('request', request => {
      if (request.resourceType() === 'script') {
          request.abort();
      } else {
          request.continue();
      }
  });

  await page.goto('https://reddit.com');
  await page.screenshot({ path: 'pptr-nojs.png' });

  await browser.close();
})();
```


------------

## Navigation Timing API metrics

```js
import puppeteer from 'puppeteer';

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://remix-clone-hacker-news.flameddd1.workers.dev/');

  // window.performance.timing 已經被標記 Deprecated 了
  const performanceTiming = JSON.parse(
      await page.evaluate(() => JSON.stringify(window.performance.timing))
  );

  const PerformanceNavigationTiming = JSON.parse(
    await page.evaluate(() => JSON.stringify(window.performance.getEntries()[0]))
  );

  console.log('performanceTiming', performanceTiming);
  console.log('PerformanceNavigationTiming', PerformanceNavigationTiming);
  await browser.close();
})();
```

### `window.performance.timing`:  
- `navigationStart`: browser 開始取得目前 document 的時間
- `unloadEventStart`: 目前 document unload event 拋出的時間，如果沒有則回傳 0
- `unloadEventEnd`: 目前 document unload event 完成的時間，如果沒有則回傳 0
- `redirectStart`: 重定向開始的時間，如果沒有則回傳 0
- `redirectEnd`: 重定向結束的時間，如果沒有則回傳 0
- `fetchStart`: 開始取得 document 的時間，該時間通常與 navigationStart 相同
- `domainLookupStart`: 開始進行 DNS 查詢的時間，如果使用了 cache，則與 `fetchStart` 相同
- `domainLookupEnd`: DNS 查詢結束的時間，如果使用了 cache，則與 `fetchStart` 相同
- `connectStart`: 開始建立連接的時間，如果有多個連接，則傳回最早的時間
- `connectEnd`: 連接建立完成的時間，如果有多個連接，則傳回最晚的時間
- `secureConnectionStart`: 開始安全連線的時間，如果不是安全連線則回傳 0
- `requestStart`: 向 server 發送請求的時間
- `responseStart`: 接收到 response 的時間，如果沒有則回傳 0
- `responseEnd`: response 結束的時間，如果沒有則回傳 0
- `domLoading`: 開始解析文檔的時間
- `domInteractive`:  document 解析完成且所有子資源（如圖片、樣式表）已下載完成的時間
- `domContentLoadedEventStart`: DOMContentLoaded event 拋出的時間
- `domContentLoadedEventEnd`: DOMContentLoaded event 完成的時間
- `domComplete`: DOM 樹建置完成的時間，但可能包括資源的載入
- `loadEventStart`: load event 拋出的時間
- `loadEventEnd`: load event 完成的時間

可以這樣解讀:
- DNS lookup cost: `domainLookupEnd` - `domainLookupStart`
- TCP connect cost: `connectEnd` - `connectStart`
- request cost : `responseEnd` - `responseStart`
- 解析 dom tree 的 cost: `domComplete` - `domInteractive`
- 白畫面的時間: `domloadng` - `fetchStart`
- domready 的時間: `domContentLoadedEventEnd` - `fetchStart`
- onload 的時間: `loadEventEnd` - `fetchStart`

![](https://global.discourse-cdn.com/business6/uploads/snowplowanalytics/original/1X/9842ec9f3dae5610cc6c5b32c5aca47ee7445b82.png)

### `PerformanceNavigationTiming`:
跟 `performance.timing` 基本類似
- `PerformanceNavigationTiming` 沒有 `navigationStart`，取而代之的是 `startTime` (其實就是0)
- 其他屬性不再以 `navigationStart` 為基準，而是以 `startTime`

常用運算效能指標
- DNS lookup cost: `domainLookupEnd` - `domainLookupStart`
- TCP connect cost: `connectEnd` - `connectStart`
- request 的時間: `responseEnd` - `requestStart`
- 解析 dom tree 的時間: `domComplete` - `domInteractive`
- dom 載入完成的時間: `domContentLoadedEventEnd`
- page load 的總時間: `duration`

![](https://developer.mozilla.org/zh-CN/docs/Web/API/PerformanceNavigationTiming/timestamp-diagram.svg)  

ref: https://web.dev/articles/ttfb?hl=zh-tw

------------

## 測量 First Paint (FP) 和 First Contentful Paint (FCP)

```js
import puppeteer from 'puppeteer';

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  const navigationPromise = page.waitForNavigation();
  await page.goto('https://pptr.dev');

  await navigationPromise;

  let firstPaint = JSON.parse(
    await page.evaluate(() =>
      JSON.stringify(performance.getEntriesByName('first-paint'))
    )
  );

  let firstContentfulPaint = JSON.parse(
    await page.evaluate(() =>
      JSON.stringify(performance.getEntriesByName('first-contentful-paint'))
    )
  );

  console.log(`First paint: ${firstPaint[0].startTime}`);
  console.log(`First paint: ${firstContentfulPaint[0].startTime}`);

  await browser.close();
})();
```


------------

## 測量 Largest Contentful Paint (LCP)
ref: https://web.dev/articles/lcp?hl=zh-tw

```js
import puppeteer, { KnownDevices } from 'puppeteer';

const Good3G = {
  'offline': false,
  'downloadThroughput': 1.5 * 1024 * 1024 / 8,
  'uploadThroughput': 750 * 1024 / 8,
  'latency': 40
};

const phone = KnownDevices['Nexus 5X'];

function calcLCP() {
  window.largestContentfulPaint = 0;

  const observer = new PerformanceObserver((entryList) => {
    const entries = entryList.getEntries();
    const lastEntry = entries[entries.length - 1];
    window.largestContentfulPaint = lastEntry.renderTime || lastEntry.loadTime;
  });

  observer.observe({ type: 'largest-contentful-paint', buffered: true });

  document.addEventListener('visibilitychange', () => {
    if (document.visibilityState === 'hidden') {
      observer.takeRecords();
      observer.disconnect();
      console.log('LCP:', window.largestContentfulPaint);
    }
  });
}

async function getLCP(url) {
  const browser = await puppeteer.launch({
    args: ['--no-sandbox'],
    timeout: 10000
  });

  try {
    const page = await browser.newPage();
    const client = await page.createCDPSession();

    await client.send('Network.enable');
    await client.send('ServiceWorker.enable');
    await client.send('Network.emulateNetworkConditions', Good3G);
    await client.send('Emulation.setCPUThrottlingRate', { rate: 4 });
    await page.emulate(phone);

    await page.evaluateOnNewDocument(calcLCP);
    await page.goto(url, { waitUntil: 'load', timeout: 60000 });

    let lcp = await page.evaluate(() => {
      return window.largestContentfulPaint;
    });

    browser.close();
    return lcp;

  } catch (error) {
    console.log(error);
    browser.close();
  }
}

getLCP("https://remix-clone-hacker-news.flameddd1.workers.dev/")
  .then(lcp => console.log("LCP is: " + lcp));
```

------------

## 測量 Cumulative Layout Shift (CLS)
ref: https://web.dev/articles/cls?hl=zh-tw

```js
import puppeteer, { KnownDevices } from 'puppeteer';

const Good3G = {
  'offline': false,
  'downloadThroughput': 1.5 * 1024 * 1024 / 8,
  'uploadThroughput': 750 * 1024 / 8,
  'latency': 40
};

const phone = KnownDevices['Nexus 5X'];


function calcJank() {
  window.cumulativeLayoutShiftScore = 0;

  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      if (!entry.hadRecentInput) {
        console.log("New observer entry for cls: " + entry.value);
        window.cumulativeLayoutShiftScore += entry.value;
      }
    }
  });

  observer.observe({ type: 'layout-shift', buffered: true });

  document.addEventListener('visibilitychange', () => {
    if (document.visibilityState === 'hidden') {
      observer.takeRecords();
      observer.disconnect();
      console.log('CLS:', window.cumulativeLayoutShiftScore);
    }
  });
}


async function getCLS(url) {
  const browser = await puppeteer.launch({
    args: ['--no-sandbox'],
    timeout: 10000
  });

  try {
    const page = await browser.newPage();
    const client = await page.createCDPSession();

    await client.send('Network.enable');
    await client.send('ServiceWorker.enable');
    await client.send('Network.emulateNetworkConditions', Good3G);
    await client.send('Emulation.setCPUThrottlingRate', { rate: 4 });
    await page.emulate(phone);
    // inject a function with the code from https://web.dev/cls/#measure-cls-in-javascript
    await page.evaluateOnNewDocument(calcJank);
    await page.goto(url, { waitUntil: 'networkidle2', timeout: 60000 });
    //await page.evaluate(calcJank);
    let cls = await page.evaluate(function () { return window.cumulativeLayoutShiftScore; });
    return cls;
    browser.close();
  } catch (error) {
    console.log(error);
    browser.close();
  }
}

getCLS("https://www.thinkwithgoogle.com").then(cls => console.log("CLS is: " + cls));
```

------------

## 測量 Frames Per Second (FPS)
不過這個範例的實用性很低，只會錄到 FPS 只有 1x 而已，好像是因為畫面沒有在動，x下面的 code 應該還需要改善。

```js
import puppeteer from 'puppeteer'

(async () => {
  const args = await puppeteer.defaultArgs().filter(flag => flag !== '--enable-automation');
  const browser = await puppeteer.launch({
    headless: false,
    devtools: true,
    ignoreDefaultArgs: true,
    args
  });
  const page = await browser.newPage();
  const devtoolsProtocolClient = await page.createCDPSession();
  await devtoolsProtocolClient.send('Overlay.setShowFPSCounter', { show: true });
  await page.goto('https://remix-clone-hacker-news.flameddd1.workers.dev/');
  await page.screenshot({ path: './FPS_image.jpg', type: 'jpeg' });
  await page.close();
  await browser.close();
})();
```

------------

## 測量 memory leaks
ref: https://media-codings.com/articles/automatically-detect-memory-leaks-with-puppeteer
- (code: https://github.com/chrisguttandin/standardized-audio-context/blob/master/test/memory/module.js)

```js
import puppeteer from 'puppeteer'

// Helper by @chrisguttandin
const countObjects = async (page) => {
  const prototypeHandle = await page.evaluateHandle(() => Object.prototype);
  const objectsHandle = await page.queryObjects(prototypeHandle);
  const numberOfObjects = await page.evaluate((instances) => instances.length, objectsHandle);

  await Promise.all([
    prototypeHandle.dispose(),
    objectsHandle.dispose()
  ]);

  return numberOfObjects;
};

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.createIncognitoBrowserContext();

  const numberOfObjects = await countObjects(page);
  console.log(numberOfObjects);

  await page.evaluate(() => {
    class SomeObject {
      constructor() {
        this.numbers = {}
        for (let i = 0; i < 1000; i++) {
          this.numbers[Math.random()] = Math.random()
        }
      }
    }
    const someObject = new SomeObject();
    const onMessage = () => { /* ... */ };
    window.addEventListener('message', onMessage);
  });

  const numberOfObjectsAfter = await countObjects(page);
  console.log(numberOfObjectsAfter);

  // Check if the number of retained objects is expected
  // expect(await countObjects(page)).to.equal(0);

  await browser.close();
})();
```

------------

## 攔截 `Request` 來做其他操作

如，阻止 iamge request:
```js
import puppeteer from 'puppeteer';

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  await page.setRequestInterception(true);

  page.on('request', (req) => {
    if (req.resourceType() === 'image'){
      req.abort();
    }
    else {
      req.continue();
    }
  });

  await page.goto('https://bbc.com');
  await page.screenshot({path: 'no-images.png', fullPage: true});
  await browser.close();
})();
```

如，把 resource 改為 local 的
```js
import puppeteer from 'puppeteer'
import fs from 'fs'
import path from 'path'

(async () => {

  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  // URL to test
  const remoteURL = 'https://pptr.dev/index.html';
  // URL to replace
  const remoteFilePath = 'https://pptr.dev/style.css';
  // Local (override) file to use instead
  const localFilePath = path.join(__dirname, "./assets/style.css");

  await page.setRequestInterception(true);

  page.on("request", interceptedRequest => {
    const url = interceptedRequest.url();
    console.log(`Intercepted ${url}`);

    if (url === remoteFilePath && !url.match(localFilePath)) {
      interceptedRequest.respond({
        body: fs.readFileSync(
          localFilePath
        )
      });
    } else {
      interceptedRequest.continue();
    }
  });

  await page.goto(`https://pptr.dev/index.html`, {
    waitUntil: "networkidle2"
  });

  await page.screenshot({path: 'override.png', fullPage: true});
  await browser.close();

})();
```

如，阻擋某些特定第三方 domains
```js
import puppeteer from 'puppeteer'

(async() => {
  const browser = await puppeteer.launch({
    headless: true
  });
  const page = await browser.newPage();
  const options = {
    waitUntil: 'networkidle2',
    timeout: 30000
  };

  // Before: Normal navigtation
  await page.goto('https://theverge.com', options);
  await page.screenshot({path: 'before.png', fullPage: true});
  const metrics = await page.metrics();
  console.info(metrics);

  // After: Navigation with some domains blocked

  // Array of third-party domains to block
  const blockedDomains = [
    'https://pagead2.googlesyndication.com',
    'https://creativecdn.com',
    'https://www.googletagmanager.com',
    'https://cdn.krxd.net',
    'https://adservice.google.com',
    'https://cdn.concert.io',
    'https://z.moatads.com',
    'https://cdn.permutive.com'];
  await page.setRequestInterception(true);
  page.on('request', request => {
    const url = request.url()
    if (blockedDomains.some(d => url.startsWith(d))) {
      request.abort();
    } else {
      request.continue();
    }
  });

  await page.goto('https://theverge.com', options);
  await page.screenshot({path: 'after.png', fullPage: true});

  const metricsAfter = await page.metrics();
  console.info(metricsAfter);

  await browser.close();
})();
```


------------

## 取得 JavaScript 和 CSS 的 Code Coverage

```js
import puppeteer from 'puppeteer'

(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  // Gather coverage for JS and CSS files
  await Promise.all([page.coverage.startJSCoverage(), page.coverage.startCSSCoverage()]);

  await page.goto('https://pptr.dev');

  // Stops the coverage gathering
  const [jsCoverage, cssCoverage] = await Promise.all([
    page.coverage.stopJSCoverage(),
    page.coverage.stopCSSCoverage(),
  ]);

  // Calculates # bytes being used based on the coverage
  const calculateUsedBytes = (type, coverage) =>
    coverage.map(({url, ranges, text}) => {
      let usedBytes = 0;

      ranges.forEach((range) => (usedBytes += range.end - range.start - 1));

      return {
        url,
        type,
        usedBytes,
        totalBytes: text.length,
        percentUsed: `${(usedBytes / text.length * 100).toFixed(2)}%`
      };
    });

  console.info([
    ...calculateUsedBytes('js', jsCoverage),
    ...calculateUsedBytes('css', cssCoverage),
  ]);

  await browsr.close();
})();

```