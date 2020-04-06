# frontend performance topic

## image
- lazy load
- srcset
- optimization, compress image size with unnoticeable damage
- image format
  - Vector images (SVG) tend to be smaller than images and SVG's are responsive and scale perfectly (Use vector wherever possible).
  - Raster: it's a `photographs`, JPEG is most of the time more appropriate than PNG or GIF
- cdn
  - goes further than reducing traffic on your servers and significantly decreasing response latency.
- base64 (maybe, maybe, maybe)
- Sprite: only 1 image file, then `background-position: 10px, 10px;`

## web fonts
- format: it’s most likely safe to serve WOFF2 and fall back to WOFF for older browsers.

## code 
- minified (js, html), make sure bundle size smaller
- tree shaking
- code split
- code split base on router (lazy, post-load)
- Preload
- Get rid of unnecessary dependencies
  - or use smaller one to replace bigger library
  - import what you need, but not whole library
- `preload`: Correctly prioritizing resource download and execution and reducing browser downtime during the page load is one of the main levers for improving web application performance
- upgrade library
- cdn


## http
- Minimize HTTP Requests
- HTTP/2
  - (Data packet) Binary instead of textual
  - Fully Multiplexed requests
  - One (presistent) connection instead of multiple (減少TCP連接。HTTP/2只使用一個TCP連接)
  - Uses HPACK, a header specific compression to reduce overhead
  - the requests can be sent without waiting for blocked requests to finish
- Reduce Cookie Size
- gzip, Brotli
- cache
  - `Expires`
  - `Cache-Control` 與 `max-age` (HTTP 1.1)
  - `Last-Modified` 與 `If-Modified-Since`
  - `Etag` 與 `If-None-Match`
- preload, prefetch


## storage
use storage to cache some data

- localStorage (不會過期，除非手動清除) (5MB)
- sessionStorage (每次分頁或瀏覽器關掉後就會清除) (5MB)
- cookie (4kB)
- indexedDB

Web Storage (localStorage/sessionStorage)
- is **accessible through JavaScript** on the **same domain**
- can be vulnerable to cross-site scripting (XSS) attacks

cookie
- 設定 `HttpOnly=true` 的cookie不能被 js 獲取到，無法用 `document.cookie` 讀內容
- 設定 `Secure=true` 那 cookie 只能用 `https` 協議傳送給伺服器，用http協議是不傳送


## list
- Infinite scroll
- Virtual List


## other
- Service worker
- Service worker Cache API
- WebAssembly

## pre...
- `preload` files
- `prefetch` DNS
- `preconnect`

```html
<link rel="preload" href="./nextpage.js" as="script">
<link rel="dns-prefetch" href="//yourwebsite.com">
<link rel="preconnect" href="//sample.com">
<link rel="preconnect" href="//sample.com" crossorigin>
```

## not performance but improve UX
- CSR -> pre render
- SSR -> streaming

