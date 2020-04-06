# HTTP Archive + BigQuery data import
## https://www.jkyoutube.com/watch?v=n6hIa-fPx0M

ref link
- https://github.com/HTTPArchive/bigquery
  - https://httparchive.org/faq#how-do-i-use-bigquery-to-write-custom-queries-over-the-data
  - https://github.com/HTTPArchive/httparchive.org/blob/master/docs/gettingstarted_bigquery.md

影片中，Surma 要修一個 chrome 的 bug，但
- 是一個不符合 W3C 規範的邏輯
- 可能會把 User `原本 OK 的網站，弄的 不OK`（雖然你是修 bug）

所以 Surma 想要了解
- 這 bug `潛在的影響範圍` 是多大？(像我這種 lvl 的開發者，問題想到這邊就止步了)

首先，這個 bug 是 chrome  
- 去 get 一個 `.wasm` 檔案時，回來的 `content-type` 錯誤（不符合 W3C 規範）

![content_type_application_wasm](/assets/img/content_type_application_wasm.jpg)  

所以如果網路上，有人這樣去使用，就有機會有問題

## HTTP Archive with BigQuery
chrome team 本來就有收集資料，但剛好沒有這種 content-type
- 要加也可能，但加上去可能再等 6 weeks

Surma 就利用 BigQuery 去查詢 HTTP Archive
- HTTP Archive 有 archive 5百萬 的 page data
  - 資料很詳細，所以能查到 `content-type` 有什麼！

我進去到 BigQuery 查不到 dataset，後來時直接點 ref 文件裡面的 link 才看到 data set
- https://console.cloud.google.com/bigquery?p=httparchive

![BigQuery_HTTP_Archive_01](/assets/img/BigQuery_HTTP_Archive_01.jpg)  

```sql
SELECT resp_content_type, COUNT(*) as count
FROM httparchive.summary_requests.2020_02_01_mobile
WHERE url LIKE `%.wasm`
GROUP BY resp_content_type
ORDER BY count
```

![BigQuery_HTTP_Archive_02](/assets/img/BigQuery_HTTP_Archive_02.jpg)  
![BigQuery_HTTP_Archive_03](/assets/img/BigQuery_HTTP_Archive_03.jpg)  

最後得到結論， 5 million 最熱門的 page 當中，只有 1 個 page 有可能會有問題
- 只有 `0.00002%` 的機率，會被這個 bug fix 影響到

而 chrome 這邊的規定
- 當有機率影響 `> 0.03%` 的 user 時，就要深入調查 (closely investigated)
  - 這樣就不需要 application notice like deprecation or breaking change

### other
BigQuery Demo - The State of the Web
- https://www.youtube.com/watch?v=G2jO_uNIdbY

這影片有示範用 BigQuery 去 select 出所有頁面的 css file (透過指定 `mimeType=text/css`)  
（但沒有跑出結果，join table 太大，所以太久，沒 show 出來 = =）  
但影片的過程還是值得參考。