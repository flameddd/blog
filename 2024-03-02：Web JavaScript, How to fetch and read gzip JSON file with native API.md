# 2024-03-02：Web JavaScript, How to fetch and read gzip JSON file with native API.md


----------------

```js
const response = await fetch('.../users_10k.json.gz')
const ds = new DecompressionStream('gzip');
const decompressed_stream = response.body.pipeThrough(ds);
const decompressed_text = await new Response(decompressed_stream).text();
console.log(decompressed_text) // { ... }
```
