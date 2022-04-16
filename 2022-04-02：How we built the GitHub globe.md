# 2022-04-02：How we built the GitHub globe.md
##  Tobias Ahlin December 21, 2020
### https://github.blog/2020-12-21-how-we-built-the-github-globe/
---------------------

超長的系列文，講講怎麼建構 new homepage 跟那顆地球 ...  
- (單純實作方面，網路上找到一位，嘗試自己做一個出來一模一樣的)
  - https://blog.csdn.net/iamlegendary/article/details/123542040
  - source code: https://github.com/ChuckieLei/githubhome

![](https://github.blog/wp-content/uploads/2020/12/102573836-33a1fb80-40a4-11eb-8c77-e2d328f0a570.gif)

## 1. How our globe is built
- https://github.blog/2020-12-21-how-we-built-the-github-globe/

2019 年的 Satellite 中，CEO Nat 在 GitHub 上展示了 30 天內 open source 活動的visualization  
龐大的數量和全球影響力是驚人的，Github 想在這個故事的基礎上再接再厲  

[影片](https://github.blog/wp-content/uploads/2020/12/globe-longer.mp4)  

Github 在 globe 的 design and development 設定的主要目標是：
- 呈現一個 interconnected 的 community，最後選用 PR，一個在一個地方打開、另一頭關閉，這個點子能夠很漂亮的被視覺化
- 呈現正在發生的工作。Github 想了很多，和測試了很多細節來希望呈現最好的視覺話跟互動性的效果
- 注重細節與效能，希望在所有 device 上效果都好





### 用 WebGL render 地球
- 地球是用 `theee.js`、WebGL render 的
- 最近的 PR 是用 JSON file 來喂
- 由五個層面來組成
    1. 光暈 (halo)
    2. 地球
    3. 地球上的區域
    4. 藍色尖峰 (blue spikes)，代表 open PR
    5. 粉紅色弧線 (pink arcs)，代表 merged PR
- 沒有使用任何 `textures`
- 四道光指向球體
- 大約用 12000 個五邊形圓來 render 地球上的區域
- 球體的背面使用簡單的 custom shader 繪製一個光暈


![](https://github.blog/wp-content/uploads/2020/12/layers-loop.h264.2020-12-21-11_16_56.gif)


為了繪製地球的區域
- 首先定義所需的圓密度（這將取決於機器的性能 - 後面還有詳細介紹）
- 然後在 nested for-loop 中遍歷「經度」和「緯度」
- 從南極開始向上走，計算每個緯度的周長，然後沿著那條線均勻分佈圓圈，環繞球體

```js
for (let lat = -90; lat <= 90; lat += 180/rows) {
  const radius = Math.cos(Math.abs(lat) * DEG2RAD) * GLOBE_RADIUS;
  const circumference = radius * Math.PI * 2;
  const dotsForLat = circumference * dotDensity;
  for (let x = 0; x < dotsForLat; x++) {
    const long = -180 + x*360/dotsForLat;
    if (!this.visibilityForCoordinate(long, lat)) continue;

    // Setup and save circle matrix data
  }
}
```

為了確定一個圓圈是否可見（是水還是陸地？）
- 我們載入一個包含世界地圖的小 PNG，通過 canvase 的 `context.getImageData()` 解析圖
  - 將每個圓圈，映射到通過 `visibilityForCoordinate(long, lat)` 方法繪製地圖
  - 如果該像素的 alpha 至少為 90（共 255 個），我們繪製圓；如果沒有，我們跳到下一個

收集了通過這些小圓圈可視化地球區域所需的所有數據後
- 我們創建了 `CircleBufferGeometry` instance 並使用 `InstancedMesh` 來渲染所有幾何圖形
  - https://threejs.org/docs/#api/en/geometries/CircleGeometry
  - https://threejs.org/docs/#api/en/objects/InstancedMesh




當您進入新的 GitHub  homepage時，我們希望確保您可以在地球出現時看到自己的位置，這意味著我們需要確定您在地球上的位置。我們希望在不延遲 IP 查找後的第一次渲染的情況下實現此效果，因此我們將地球的起始角度設置為以格林威治為中心，查看您設備的時區偏移量，並將該偏移量轉換為圍繞地球自身軸的旋轉 (以弧度為單位）：


### 確保 User 看到自己的位置
Github 希望 User 打開 homepage 時，能在地球上看到自己的位置
- 這個需要知道 User 的位置，但不希望透過 ip 查詢，這位造成 delay

所以
1. 將地球的起始角度設定「格林威治」中心
2. 確認 User device 的 timezone offset
3. 將該偏移量轉換為圍繞地球的軸來旋轉，以弧度為單位

這不是 User 所在位置的精確測量，但它很快

```js
const date = new Date();
const timeZoneOffset = date.getTimezoneOffset() || 0;
const timeZoneMaxOffset = 60 * 12;
rotationOffset.y = ROTATION_OFFSET.y + Math.PI * (timeZoneOffset / timeZoneMaxOffset);
```

### Visualizing PR

在這概述如何可視化 User 所有的 PR
- (下一篇才會更深入講解如何透過 data engineering 來收集 PR 資料)

[影片](https://github.blog/wp-content/uploads/2020/12/zoomed-in-arcs.h264.mp4)

這邊專注講解「merged 的 PR（粉紅色的弧線）」，因為比較有趣
- 每個「merged 的 PR」都有兩個位置
    1. open PR location
    2. merged PR location

我們將這些位置映射到地球，並在這兩個位置之間繪製貝茲曲線 (`bezier curve`)
- ([CubicBezierCurve3](https://threejs.org/docs/#api/en/extras/curves/CubicBezierCurve3) 是 three.js 的 API)

```js
const curve = new CubicBezierCurve3(startLocation, ctrl1, ctrl2, endLocation);
```


這曲線有三個不同的軌道
- 兩點相距越長，我們就會將任何特定的弧線拉到太空中穿越。然後，使用 [TubeBufferGeometry](https://threejs.org/docs/#api/en/geometries/TubeGeometry) 沿這些路徑生成幾何圖形
  - (`TubeGeometry` 在 threejs 也是有 API)
- 這樣就可以使用 `setDrawRange()` 來對出現和消失的線條進行 animation 處理

隨著每條線的 animation 達 merge location
- 我們產生一個實心圓圈的 animation ，該圓圈在該線存在時保持不變
- 另外還有一個 ring 會按比例放大並立即淡出


這些 animation 的 `ease out` 是通過將速度（此處為 0.06）乘以目標 (1) 和當前值 (animated.dot.scale.x) 之間的差異，並將其加到現有比例值來建利的
- 對每一 frame，都會向目標靠近 6%，隨著越來越靠近目標， animation 自然會變慢

```js
// The solid circle
const scale = animated.dot.scale.x + (1 - animated.dot.scale.x) * 0.06;
animated.dot.scale.set(scale, scale, 1);

// The landing effect that fades out
const scaleUpFade = animated.dotFade.scale.x + (1 - animated.dotFade.scale.x) * 0.06;
animated.dotFade.scale.set(scaleUpFade, scaleUpFade, 1);
animated.dotFade.material.opacity = 1 - scaleUpFade;
```


 homepage和地球需要在各種設備和平台上表現良好，這在早期為我們創造了一些創意限制，並使我們廣泛專注於創建一個優化良好的頁面。儘管一些現代計算機和平板電腦可以在開啟抗鋸齒的情況下以 60 FPS 的速度渲染地球，但並非所有設備都是如此，我們很早就決定關閉抗鋸齒並優化性能。這給我們留下了一條沿著地球左上邊緣的清晰像素化線，因為地球的突出邊緣與背景的較暗顏色相遇：

### performance optimizations 限制了一些創意
因為目標是在各種平台都有好的 perf，即時在一些 modern computers and tablets 都能打開抗鋸齒的情況下以 60 FPS render 地球，但有些裝置就沒辦法
- 所以很早就決定關閉抗鋸齒 (antialias) 並優化 perf 
- 所以地球左邊上邊緣，有很明顯的 pixel 線（加上那邊背景是暗色的，更明顯了）

![](https://github.blog/wp-content/uploads/2020/12/102573561-8e872300-40a3-11eb-9feb-b480aeae0564.png)  

這問題，讓我們嘗試隱藏 pixelated 邊緣的光環效應
- 我們通過使用 custom shader 繪製一個比地球稍大的球體，繪製漸變，將其放在地球後面，並將其稍微傾斜以強調左上角的效果

```js
const halo = new Mesh(haloGeometry, haloMaterial);
halo.scale.multiplyScalar(1.15);
halo.rotateX(Math.PI*0.03);
halo.rotateY(Math.PI*0.03);
this.haloContainer.add(halo);
```

![](https://github.blog/wp-content/uploads/2020/12/2-2.png)  

這 sharp edge 變平滑了，同時比開啟 antialias 效能更好。不幸的是
- 關閉 antialias 也會產生相當明顯的摩爾波紋 ([moiré effect](https://zh.wikipedia.org/wiki/%E8%8E%AB%E5%88%97%E6%B3%A2%E7%B4%8B))，因為構成世界的所有圓圈在接近地球邊緣時彼此越來越近
- 我們通過對圓圈使用[片段著色器(fragment shader)](https://developer.mozilla.org/zh-TW/docs/Web/API/WebGL_API/Tutorial/Using_shaders_to_apply_color_in_WebGL)來減少這種效果並模擬較厚大氣的外觀，其中每個圓圈的 alpha 是其與相機距離的函數，隨著每個圓圈的移動越來越遠，它會逐漸消失

```js
if (gl_FragCoord.z > fadeThreshold) {
  gl_FragColor.a = 1.0 + (fadeThreshold - gl_FragCoord.z ) * alphaFallOff;
}
```


我們不知道地球在特定設備上加載的速度（或緩慢），但我們希望確保 homepage上的標題組合始終保持平衡，並且您會得到地球加載速度快的印象即使在我們渲染第一幀之前有一點延遲。

我們在Figma中僅使用漸變創建了一個裸露版本的地球，並將其導出為 SVG。在 HTML 文檔中嵌入這個 SVG 幾乎不會增加開銷，但可以確保在頁面加載時某些內容立即可見。一旦我們準備好渲染地球的第一幀，我們通過使用Web Animations API在兩個元素之間交叉淡入淡出和放大這兩個元素來在 SVG 和畫布元素之間進行轉換。使用 Web Animations API 使我們能夠在過渡期間完全不接觸 DOM，確保它盡可能不卡頓。

### 改善 perceived speed
Github 先在 Figma，用 gradients 建立了一個 bare version 的地球
- export 成 SVG
- 放到 HTML 中
- 這幾乎不會有負擔，也確保了 User 第一眼就能看到東西

一旦準備好 render 地球的第一個 frame 時，就用 `Web Animations API` 來交叉淡入淡出和放大這兩個元素  
讓 SVG 和 canvas 作轉換  

```js
const keyframesIn = [
  { opacity: 0, transform: 'scale(0.8)' },
  { opacity: 1, transform: 'scale(1)' }
];
const keyframesOut = [
  { opacity: 1, transform: 'scale(0.8)' },
  { opacity: 0, transform: 'scale(1)' }
];
const options = { fill: 'both', duration: 600, easing: 'ease' };

this.renderer.domElement.animate(keyframesIn, options);
const placeHolderAnim = placeholder.animate(keyframesOut, options);
placeHolderAnim.addEventListener('finish', () => {
  placeholder.remove();
});
```

(我有點興趣這個 svg，我把它 copy 出來，貼到最下面)  


### Graceful degradation with quality tiers
目標時盡可能的用 60FPS render 一顆漂亮的地球
- 監控的 FPS，如果沒辦法在在過去 50 frams 中，保持 55.5 FPS 的數度，我們就會開始降低品質

有分 4 層 quality 等級，每降一級，就減少計算量，這包含
- 減少 pixel 密度
- 多久進行一次光線投射 (raycast) (弄清楚 User hovering 哪個位置)
- 在螢幕上繪製的幾何圖形數量


這使我們回到了構成地球區域的圓圈，我們會降低所需的圓圈密度並重建地球區域
- 從原來的 `~12 000` 個圓圈變為 `~8 000` 個

```js
// Reduce pixel density to 1.5 (down from 2.0)
this.renderer.setPixelRatio(Math.min(AppProps.pixelRatio, 1.5));
// Reduce the amount of PRs visualized at any given time
this.indexIncrementSpeed = VISIBLE_INCREMENT_SPEED / 3 * 2;
// Raycast less often (wait for 4 additional frames)
this.raycastTrigger = RAYCAST_TRIGGER + 4;
// Draw less geometry for the Earth’s regions
this.worldDotDensity = WORLD_DOT_DENSITY * 0.65;
// Remove the world
this.resetWorldMap();
// Generate world anew from new settings
this.buildWorldGeometry();
```

### A small part of a wide-ranging effort
直到這邊，都只是這個地球和這個 new homepage 的故事的一小部分，整個過程跨足多個團隊，包含了  design, brand, engineering, product, and communications teams。（所以才拆分層系列文）

----------------------------------------
## 2. How we collect and use the data behind the globe
- Visualizing GitHub’s global community
- Tal Safran December 21, 2020
- https://github.blog/2020-12-21-visualizing-githubs-global-community/



### Data goals
當開始這個 project 時
- 我們不想只製作另一個地球 animation 
- 我們希望數據有趣且引人入勝
- 我們希望它是真實的，最重要的是，我們希望它是即時的。

挑戰是：
1. 我們如何查詢海量數據？
2. 我們如何展示最有趣的部分？
3. 我們如何用隱私的方式對用戶位置進行 geocode？
4. 我們如何將計算的 data 公開給 monolith？
5. 我們如何不破壞 GitHub？

### Querying GitHub

由於
- 每天在 GitHub 上產生的資料量
- 我們 db 的大小以
- 保持 GitHub 快速可靠的重要性

我們知道我們無法直接查詢我們的 production db  

幸運的是
- 我們有 warehouse 和一個出色的團隊來維護它
- 定期從 production 中拿、清理 data 並將其打包到 warehouse 中
- 然後使用 [Presto](https://prestodb.io/) 查詢

我們還希望
- data 盡可能新 (fresh)
- 因此，我們能夠查詢來自 [Apache Kafka](https://kafka.apache.org/) event stream 流的 data
  - 而不是查詢我們每天只複製一次的 MySQL table 的 snapshots

例如，每次 merge PR 時都會 report 一個 event
- 該 event 為 [protobuf](https://github.com/protocolbuffers/protobuf) 的格式

以下是 merge PR 事件的 protobuf
```
message PullRequestMerge {
  github.v1.entities.User actor = 1;
  github.v1.entities.Repository repository = 2;
  github.v1.entities.User repository_owner = 3;
  github.v1.entities.PullRequest pull_request = 4;
  github.v1.entities.Issue issue = 5;
}
```

每行對應一個 entity
- 每個 entity 都在自己的 protobuf 文件中定義

這是 PR entity 定義中的一個片段：

```
message PullRequest {
  uint64 id = 1;
  string global_relay_id = 2;
  uint64 author_id = 3;

  enum PullRequestState {
    UNKNOWN = 0;
    OPEN = 1;
    CLOSED = 2;
    MERGED = 3;
  }
  PullRequestState pull_request_state = 4;

  google.protobuf.Timestamp created_at = 5;
  google.protobuf.Timestamp updated_at = 6;
}
```

event 中包含 entity 將傳遞為其定義的所有屬性
- 對於每個 merged PR，所有這些 data 都會到 warehouse 中

過去一天 merged 的 PR，Presto 查詢可能如下所示：

```sql
SELECT
  pull_request.created_at,
  pull_request.updated_at,
  pull_request.id,
  issue.number,
  repository.id
FROM kafka.github.pull_request_merge
WHERE
  day >= CAST((CURRENT_DATE - INTERVAL '1' DAY) AS VARCHAR)
```

我們還進行了一些其他查詢來取我們需要的所有 data
- 這幾乎是標準 SQL
- 它從 event stream 的最後一天拉 merged PR

### 呈現有趣的數據

我們希望
- 展示有趣的 data
- 有趣的 data 並且適合在 GitHub homepage 上受到關注
- 如 data 很好，user 就有機會被吸引過去

那麼我們如何找到好的數據呢？
- 幸運的是， data team 再次前來救援
- 幾年前，Data Science Team 建立了一個模型
  - 根據 30 多個按重要性加權的特徵對 repo 的「健康」進行排名
  - 健康的 repo 並不一定有很多星星
  - 還考慮了當前正在發生的活動量以及為項目做出貢獻的難易程度(僅舉幾例)

結果是我們可以在 warehouse 中查詢的 health score

```sql
SELECT repository_id
FROM data_science.github.repository_health_scores
WHERE 
  score > 0.75
```

將此查詢與上述查詢相結合
- 現在可以從運行狀況評分高於某個 threshold 的 repo 中拿 merged PR

```sql
WITH
healthy_repositories AS (
  SELECT repository_id
  FROM data_science.github.repository_health_scores
  WHERE 
    score > 0.75
)

SELECT
  a.pull_request.created_at,
  a.pull_request.updated_at,
  a.pull_request.id,
  a.issue.number,
  a.repository.id
FROM kafka.github.pull_request_merge a
JOIN healthy_repositories b
ON a.repository.id = b.repository_id
WHERE
  day >= CAST((CURRENT_DATE - INTERVAL '1' DAY) AS VARCHAR)
```

我們也做一些其他的事情來確保 data 是好的
- 比如過濾掉有垃圾郵件行為的 account
- 但是 repo 健康評分絕對是一個關鍵因素

### 對 User 提供的位置進行地理編碼 (Geocoding)

User 的 `GitHub profile` 有個自由填寫的欄位是你的區域
- 有些人寫實際位置
- 而另一些人寫假的 or 虛擬的位置
- 三分之二的用戶沒有輸入任何內容

對於確實輸入內容的 User
- 我們嘗試把它對應到真實位置
- 這比使用 IP 地址作為位置代理要難一些
- 但重要的是，只包含用戶一開始就願意公開的 data

Github 使用 [Mapbox’s forward geocoding API](https://docs.mapbox.com/api/search/geocoding/#forward-geocoding) 跟他們的 [Ruby SDK](https://github.com/mapbox/mapbox-sdk-rb) 來找出對應的緯度和經度  

例如 geocoding `New York City`  

```
MAPBOX_OPTIONS = {
  limit: 1,
  types: %w(region place country),
  language: "en"
}

Mapbox::Geocoder.geocode_forward("New York City", MAPBOX_OPTIONS)

=> [{
  "type" => "FeatureCollection",
  "query" => ["new", "york", "city"],
  "features" => [{
    "id" => "place.15278078705964500",
    "type" => "Feature",
    "place_type" => ["place"],
    "relevance" => 1,
    "properties" => {
      "wikidata" => "Q60"
    },
    "text_en" => "New York City",
    "language_en" => "en",
    "place_name_en" => "New York City, New York, United States",
    "text" => "New York City",
    "language" => "en",
    "place_name" => "New York City, New York, United States",
    "bbox" => [-74.2590879797556, 40.477399, -73.7008392055224, 40.917576401307],
    "center" => [-73.9808, 40.7648],
    "geometry" => {
      "type" => "Point", "coordinates" => [-73.9808, 40.7648]
    },
    "context" => [{
      "id" => "region.17349986251855570",
      "wikidata" => "Q1384",
      "short_code" => "US-NY",
      "text_en" => "New York",
      "language_en" => "en",
      "text" => "New York",
      "language" => "en"
    }, {
      "id" => "country.19678805456372290",
      "wikidata" => "Q30",
      "short_code" => "us",
      "text_en" => "United States",
      "language_en" => "en",
      "text" => "United States",
      "language" => "en"
    }]
  }],
  "attribution" => "NOTICE: (c) 2020 Mapbox and its suppliers. All rights reserved. Use of this data is subject to the Mapbox Terms of Service (https://www.mapbox.com/about/maps/). This response and the information it contains may not be retained. POI(s) provided by Foursquare."
}, {}]
```

很大一串，但這邊我們關注 `text`, `relevance`, and `center`
- 下面是 `New York City` 的這些欄位
- 如果用 `NYC` 查，會得到完全相同的結果
  - `text` 一樣會顯示 `New York City`，這是 Mapbox format 的，這可以處理一些大小寫 or 拼寫錯誤

```
result = Mapbox::Geocoder.geocode_forward("New York City", MAPBOX_OPTIONS)
result[0]["features"][0].slice("text", "relevance", "center")

=> {"text"=>"New York City", "relevance"=>1, "center"=>[-73.9808, 40.7648]}
```


`center` 是位置的經度和緯度的數組  
`relevance` 是 Mapox 對結果的信心的一個指標。相關性得分為 1 是最高的，但有時 User 會輸入 Mapbox 不太確定的位置


```
result = Mapbox::Geocoder.geocode_forward("Middle Earth", MAPBOX_OPTIONS)
result[0]["features"][0].slice("text", "relevance", "center")

=> {"text"=>"Earth City", "relevance"=>0.5, "center"=>[-90.4682, 38.7689]}
```

我們不管任何得分低於 1 的東西，為了確信我們顯示正確位置  
Mapbox 還提供了[batch geocoding endpoint](https://docs.mapbox.com/api/search/geocoding/#batch-geocoding)，允許在一個請求中查詢多個位置

在我們對所有結果進行 geocoded 和 normalized 之後
- 我們建立 PR 和 location 的 JSON，以便我們全球 JavaScript client 知道如何解析它

這是最近在舊金山打開並在東京合併的一個 [pull request](https://github.com/mdn/browser-compat-data/pull/7937)

```json
{
   "uml":"Tokyo",
   "gm":{
      "lat":35.68,
      "lon":139.77
   },
   "uol":"San Francisco",
   "gop":{
      "lat":37.7648,
      "lon":-122.463
   },
   "l":"JavaScript",
   "nwo":"mdn/browser-compat-data",
   "pr":7937,
   "ma":"2020-12-17 04:00:48.000",
   "oa":"2020-12-16 10:02:31.000"
}
```

最終的 JSON 有對 key 做簡寫，刪掉一些字，讓 loading 更快點



### Airflow, HDFS, and Munger
我們全天運行 warehouse queries and geocoding，來確保 homepage 上的 data 始終是最新的。

為了 scheduling 這個 task
- Github 使用 [Apache Airflow](https://airflow.apache.org/)
  - Airflow 將這些工作流程「有向無循環圖（DAG, Directed Acyclic Graph）」 ，就是有方向性且無回向
  
  
我們之前介紹了前兩個步驟。為了編寫文件，我們使用HDFS，這是一個分佈式文件系統，是 Apache Hadoop 項目的一部分。然後將該文件上傳到 Munger，這是我們用來將數據科學管道的結果公開回支持 github.com 的 GitHub Rails 應用程序的內部服務。

我們的 DAG 執行以下任務：
1. Query data warehouse.
2. Geocode locations from the results.
3. Write the results to a file.
4. Expose the results to the GitHub Rails app.

For writing the file
- Github 使用 [HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html#Introduction)
- 是一個分佈式文件系統，是 Apache Hadoop 的一部分
- 然後將該文件上傳到 `Munger`，這是 Github 內部服務，用來將 data science pipeline 的結果公開回支持 github.com 的 GitHub Rails application

Airflow UI:  
![](https://github.blog/wp-content/uploads/2020/12/102573671-c8f0c000-40a3-11eb-809f-94469c4b53df.png)  

![](https://github.blog/wp-content/uploads/2020/12/102573680-cbebb080-40a3-11eb-948d-84d25b0c118a.png)  

----------------------------------------
## 3. How we made the page fast and performant
- Making GitHub’s new homepage fast and performant
-  Tobias Ahlin January 29, 2021
- https://github.blog/2021-01-29-making-githubs-new-homepage-fast-and-performant/

建立一個充滿 image、animation 和 video 的頁面，頁面仍然可以快速載入並且表現良好，這會很困難。我們在這裡深入探討對我們產生最大整體 perf 影響的兩個策略：製作  high performance animations 和提供 perfect image


### High performance animation 和交互性

向下 scroll GitHub homepage 時，我們會在某些 element 中加 animation 以引起 Usert 的注意：

![](https://github.blog/wp-content/uploads/2021/01/106009518-b6ad6d00-60b8-11eb-980d-061d5f414a37.gif)  

以前，都是依賴監聽 `scroll` event，然後計算所有 element 的 visibility，來決定觸發 animation

```js
// Old-school scroll event listening (avoid)
window.addEventListener('scroll', () => checkForVisibility)
window.addEventListener('resize', () => checkForVisibility)

function checkForVisibility() {
  animatedElements.map(element => {
    const distTop = element.getBoundingClientRect().top
    const distBottom = element.getBoundingClientRect().bottom
    const distPercentTop = Math.round((distTop / window.innerHeight) * 100)
    const distPercentBottom = Math.round((distBottom / window.innerHeight) * 100)
    // Based on this position, animate element accordingly
  }
}
```

這方法有很大的 perf 問題
- 呼叫 `getBoundingClientRect()` 會 [trigger reflows](https://gist.github.com/paulirish/5d52fb081b3570c81e3a) 

現在都用 `IntersectionObservers`
- 大部分 browser 都支援了

```js
// Create an intersection observer with default options, that 
// triggers a class on/off depending on an element’s visibility 
// in the viewport
const animationObserver = new IntersectionObserver((entries, observer) => {
  for (const entry of entries) {
    entry.target.classList.toggle('build-in-animate', entry.isIntersecting)
  }
});

// Use that IntersectionObserver to observe the visibility
// of some elements
for (const element of querySelectorAll('.js-build-in')) {
  animationObserver.observe(element);
}
```

### 避免 animation pollution

當我們開始 `IntersectionObservers` animation 時
- 我們檢查了所有 animation ，並特別注重優化 animation 的核心原則之一
- 僅對 `transform` 和 `opacity` 進行 animation 處理

我們認為已經在遵循這原則做得相當好，但發現在某些情況下我們沒有做到
- 因為當元 element 變狀態時，意想不到的屬性滲透到 `transitions` 中並污染它

```css
// Don’t do this
.animated {
  opacity: 0;
  transform: translateY(10px);
  transition: * 0.6s ease;
}

.animated:hover {
  opacity: 1;
  transform: translateY(0);
}
```

換句話說
- 我們只指定了 `opacity` 和 `transform`
- 但，我們卻定義 `transition` 為 `*`

這些 `transition` 會有 perf issue
- 因為其他屬性更改可能會污染 `transition`

為了避免這問題，要明確指出屬性

```css
// Be explicit about what can animate (and not)
.animated {
  opacity: 0;
  transform: translateY(10px);
  transition: opacity 0.6s ease, transform 0.6s ease;
}

.animated:hover {
  opacity: 0;
  transform: translateY(0);
}
```

當把所有 animation 重新調整過後，`CPU` 跟 `style recalculations` 大幅改善了  
這也改善 `Cumulative Layout Shift` 的分數  

![](https://github.blog/wp-content/uploads/2021/01/106009495-af865f00-60b8-11eb-84de-b28f3281f5de.png)  

### 用 IntersectionObservers 來 Lazy-loading videos

通常，當我們想用 video 來加強 animation (UX) 時，可能會想做兩種事情
1. 只有當 vidoe 出現在畫面時，才播放 video
2. lazy load video

但 video tag 不支援 [lazy load attribute](https://developer.mozilla.org/en-US/docs/Web/Performance/Lazy_loading#images_and_iframes)  

這邊靠 `IntersectionObservers` 來完成上面兩個需求


```html
<!-- HTML: A video that plays inline, muted, w/o autoplay & preload -->
<video loop muted playsinline preload="none" class="js-viewport-aware-video" poster="video-first-frame.jpg">
  <source type="video/mp4" src="video.h264.mp4">
</video>
```
```js
// JS: Play videos while they are visible in the viewport
const videoObserver = new IntersectionObserver((entries, observer) => {
  for (const entry of entries) entry.isIntersecting ? video.play() : video.pause();
});

for (const element of querySelectorAll('.js-viewport-aware-video')) {
  videoObserver.observe(element);
}
```

搭配 `preload="none"`，`IntersectionObserver` 這簡單的 observer 讓 github 每一頁都省下好幾 mb


### 完美的 image

Github 有 user case 有超多不同的 device、screen size 和 browsers  
另外，我們特定的插圖有 JPG、PNG 和 SVG 格式  

用這張圖當例子，這張圖是在主要內容跟 footer 之間的過渡  

![](https://github.blog/wp-content/uploads/2021/01/106011272-99799e00-60ba-11eb-8c47-b636c1619353.jpg)  

為了渲染這個插圖，理想情況下
- 我們需要 PNG 的透明度
- 但將其與 JPG 的壓縮相結合
  - 因為 PNG 會佔空間
  
幸運的是
- 從 iOS 14 和 macOS Big Sur 開始，桌面和手機上的 Safari 都支持 [WebP](https://developers.google.com/speed/webp)
- 瀏覽器的支持率高達 +90%
- WebP 確實提供了兩全其美的優勢：可以創建具有透明度的壓縮、有損圖

對舊瀏覽器如何？
- 即使是在 macOS Catalina 上運行最新版 Safari 的新 Mac 也無法渲染 WebP


這挑戰最終導致我們開發了一個有點模糊的解決方案：
- 兩個 JPG 嵌入到一個 SVG 中（一是 image data，另一是 mask）
- 用 base64 放進去 svg
- 本質上是通過一個 HTTP request create 一個透明的 JPG

看看這張圖片。下載它，打開它，然後檢查它
- 它是一個透明的 JPG，用 base64 編碼，包裝在 SVG 中
- https://github.githubassets.com/images/modules/site/home/footer-illustration.svg

[mask](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/mask) 是 SVG 的，可以用它來遮蓋 SVG 的某部分
- 用 mask + image 我們能 render 具有 transparency 的 image

```html
<svg viewBox="0 0 300 300">
  <defs>
    <mask id="mask">
      <image width="300" height="300" href="mask.jpg"></image>
    </mask>
  </defs>
  <image mask="url(#mask)" width="300" height="300" href="image.jpg"></image>
</svg>
```

這樣還無法取代 WebP，因為 image 中，path 是動態的 (上面的 `href`)
- 因此 SVG 需要 embedded inside the document

將 SVG 存成一個文件中並將 img 的 src 設 constant，則不會載入圖片，我們將什麼也看不到  
- 我們把 image 轉 base64 放到 SVG 中來解決這個限制

Mac 預設就有 `base64` cli tool
```shell
base64 -i <in-file> -o <outfile>
```

其中 in-file 是您選擇的圖像，outfile 是一個文本文件，您將在其中保存 base64 數據。使用這種技術，我們可以將圖像嵌入到 SVG 中，並將 SVG 用作常規圖像的 src。

這是我們用來構建 footer 插圖的兩張圖片
- 一張用於 data imate
- 一張用於 mask（黑色完全透明，白色完全不透明

![](https://github.blog/wp-content/uploads/2021/01/106009473-ab5a4180-60b8-11eb-8da9-dd432a5c832b.jpg)  


```html
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 2900 1494">
  <defs>
    <mask id="mask">
      <image width="300" height="300" href="data:image/png;base64,/* your image in base64 */”></image>
    </mask>
  </defs>
  <image mask="url(#mask)" width="300" height="300" href="data:image/jpeg;base64,/* your image in base64 */”></image>
</svg>
```

存成 SVG 並像常見的 image 檔案一樣的方式使用它
- 這樣可以放心使用 WebP
- 另外搭配有 svg，這樣搭配，browser 就能判斷，若不支援，就改用 svg
  - 而且 svg 還是 lazy 的

```html
<picture>
  <source srcset="compressed-transparent-image.webp" type="image/webp">
  <img src="compressed-transparent-image.svg" loading="lazy">
</picture>
```

雖然費工，但這樣每次 load page 都省下幾百 KB

----------------------------------------
## 4. How we illustrate at GitHub
-  Tony Jaramillo February 4, 2021
- https://github.blog/2021-02-04-how-we-illustrate-at-github/

### 協同設計 Collaborative design

從表面上看
- 我們的新 homepage 體現了設計師、工程師等的精神和獨創性
- 但是再深入一點，你就會解開軟體開發相互連結的世界
- 從我們的可互動的 WebGL 地球儀到人物插圖再到令人著迷的產品照片，有很多令人驚嘆的東西

它展示了 GitHub 的一種新設計方式


### Story
![](https://github.blog/wp-content/uploads/2021/02/story-illustration.jpg)  

我們有獨特的機會通過設計、產品和角色 IP 講述引人入勝的品牌故事。為了應對這一挑戰
- 我們通過協作將各個團隊的才能結合在一起
- 這個頁面是一個完美的機會，可以將我們的講故事技巧與網站團隊結合起來

我們從兩個角度處理同一個故事：
- 角色和 UX/UI

從插圖的角度來看
- 我們將該頁面視為解釋 GitHub 強大功能的視覺敘事
- 同時突出我們最重要的資產，即開發人員。


octocats (八角貓)
- 代表開發人員的想法
- 貓科頭足類動物永遠頑強，並在尋找更多東西
- 就像他們所代表的人類開發者一樣
- 他們的好奇心推動了這種對創新和更美好未來的推動

同樣的情感是我們品牌的核心：
- 無論是 octocats、conference motion graphics 還是  office branding

因此
- 我們用我們的 octo-avatar,  [Mona](https://myoctocat.com/) 解釋 GitHub 的好處
- 以及我們如何提供任何 code 探索所需的工具
- 就像她一樣，你也可以在知道我們支持您的情況下安全地冒險進入未知世界。

![](https://github.blog/wp-content/uploads/2021/02/header.png)  

一開始
- Mona 站在一個空白的星球上，盯著標題 `Where the world builds software.`
- 她身後是一個充滿實際 GitHub 活動的地球，突出了全球開發社群
- 地球的動能與 Mona 的潛力並列。她腳邊的新芽和 Git 線條暗示著成長和可能性。



當向下滾動頁面時
- 我們會突出顯示 GitHub 許多獨一無二的功能
- 協作、社群、自動化和安全性
- 我們強調的一些元素，它們可以改善您的開發工作流程，為您提供創新所需的安全性和信心。

![](https://github.blog/wp-content/uploads/2021/02/footer.png)  

沿著 git line 到 foorer
- 會發現一個太空著陸器
- 裡面有更多的 octocats
- 如果人類是創造改變我們星球的 code 的先驅，那麼 octocats 就是在他們的宇宙中創造物理世界的探險家
- footer 插圖顯示了這種冒險精神。各種八角貓正在勘測土地，利用他們的機器人，並共同努力創造新的東西
- 在這裡，我們向觀眾伸出手，詢問你是否願意為社群做出貢獻並突出其他人

### 視覺開發

![](https://github.blog/wp-content/uploads/2021/02/element-illustration.jpg)  

Building is key to the story.
- 為了將人類和 Octocat 世界編織在一起，我們必須開發一種通用的形狀語言
- Git lines 是一個完美的選擇，因為它們是獨一無二的
- 我們可以將它們操縱成各種形狀和大小，並且它們是 code 開發的象徵
  - push commit、PR 或 create new branch
- 這些操作中的每一個都會將 git 行擴展一個節點
- 我們覺得這個符號是我們之間隱喻線索的最好例子
- 你可以在岩石、植物等地方找到它們。這讓人想起地球森林下廣闊的真菌網絡

### Illustration 插圖
![](https://github.blog/wp-content/uploads/2021/02/footer-concept-illustration.jpg)  


然後，我們使用該 visual framework 來創造各種 octocat 組合
- 我們的插圖過程很簡單
  - Thumbnails (縮略圖)、concepts (概念)、line art、color 和final asset creation 都基於共享的概要
- 我們的角色是利用現有的模型和表情表繪製的
- 以確保一致性

### Globe
![](https://github.blog/wp-content/uploads/2021/02/3-globes.jpg)  

展示我們開發社群的實際活動
- 我們為我們的用戶感到自豪，並抓住機會突出他們的貢獻
- 地球設計需要引人入勝、信息豐富且易於閱讀
- 它也體現了 Octocat 世界的驅動力，因此建立這種聯繫至關重要

我們的藝術家
- 通過形狀和顏色來傳播這種敘事的概念
- 我們需要展示我們平台的潛力和廣泛的 code collaboration
- 需要多次迭代才能找到正確的平衡
- 這些圖紙為我們的工程師提供了構建動態 WebGL 交互體驗的基準
- 最後，這種協作努力將最終產品提煉為最重要的部分：視覺吸引力、資料可視化、interactivity 和速度


### Color

![](https://github.blog/wp-content/uploads/2021/02/color.jpg)  

顏色是一個有趣的挑戰
- 我們的目標是在 marketing 中盡可能準確地表示 GitHub UI
- 因此需要考慮特定的 UI 顏色
- 當然，我們的 color palettes 是為實用而設計的
- 但 marketing 使我們能夠擴展和靈活利用這些基礎知識，以提供更具吸引力的故事


視覺敘事的核心是能量
- 我們需要一個能夠體現平台和我們社群內在潛力的 palette
- 我們通過調高飽和度進行迭代
- 而不會偏離我們的 UI 色調太遠。通過顏色將地球設計、角色插圖、排版和產品照片融合在一起

這個新的 palette 基於綠色、紫色和橙色三色
- 但是我們需要更多選項來區分頁面上的每個功能部分
- 因此我們在頂部分層了一個拆分互補。您可能會注意到一些額外的 `cheat colors`，這也是 UI 所需要的


之後，我們開發了 color scripts
- 將頁面視為 narrative arc
- 以引導觀眾，標題通過在焦點元素上使用那些主要顏色
- Mona 臉上的橙色調、地球上的紫羅蘭色和綠色的 CTA 按鈕
- 為我們的三合會設置了框架
- 當我們向下滾動頁面時，我們使用調色板的專用部分圍繞色輪旋轉
- 以直觀地定義每個區域。然後，footer 通過再次錨定回主要的三重顏色來解析頁面。

----------------------------------------
## 5. How we designed the homepage and wrote the narrative
- Amanda Swan February 11, 2021
- https://github.blog/2021-02-11-how-we-designed-and-wrote-the-narrative-for-our-homepage/

重新設計 GitHub 的 homepage 
- 需要的不僅僅是視覺上的更新
- 我們必須從根本上重新構想我們想要與社群分享的故事
- 我們面臨著在視覺上傳達我們平台的全球規模以及使用它的個人體驗的挑戰
- 經過漫長的探索階段，我們決定展示開發人員在使用 GitHub 時的一天。

開發人員以多種方式與 GitHub 互動
- writing code, maintaining projects, updating dependencies, reviewing pull requests 等等

這篇文章提供了設計的幕後花絮，以及我們用來實現目標的過程

### 尋找真實的外觀 (Finding an authentic look)


我們不是從頭開始，而是迭代。[Primer](https://primer.style/) 是 GitHub 的 open source design system 
- 用於GitHub 的所有用戶界面
- 多年來，GitHub 的 marketing 團隊也一直在使用 Primer 和一般設計原則來構建網站

但是隨著我們的 marketing design and branding 的發展
- 我們需要擴展這個系統，以允許我們的登出頁面出現分歧並更好地實現我們的願景。

為了 visualize 我們如何做到這一點
- 我們查看了一系列 `mood boards` 來幫助我們的團隊探索
- 我們隨著我們不斷發展的 marketing aesthetic 走向的各個方向
- 早期的視覺效果涉及不同的品牌和主題，目的在大膽並包含一些瘋狂的想法來測試我們品牌的各方面

![](https://github.blog/wp-content/uploads/2021/02/Hompage-narrative-1.png)  
![](https://github.blog/wp-content/uploads/2021/02/Homepage-narrative-2.png)  


在向 leadership 的早期 demo 中
- 我們吸引了一群對這些想法非常興奮的聽眾
- 然而，一旦想法深入人心，它們就給了我們最直接、最善意的反饋： `I hate it.`
- 當你在探索一些瘋狂的想法時，它一定會發生，而且它仍然是有用的反饋
- 他們的回應顯然鼓勵我們重新考慮我們的方法。我們回到了 GitHub 的根源，感覺是對的。 


對我們來說最重要的是
- 向開發人員展示我們產品和故事的高品質、真實的視覺表現
- 我們決定使用我們的內部專業知識為我們的 marketing 頁面重新建 HTML、CSS 和 JavaScript 的關鍵產品界面


同樣，我們必須為手工 illustrations 回歸本源
- 自從我們在網站上突出展示我們的吉祥物 Mona the Octocat 和她的 octocat 朋友以來
- 我們已經從這些高度詳細的說明性吉祥物轉向更容易、更快地製作輪廓 octocat
- 但在此過程中，我們失去了一些靈魂和想像力
- 為了真正講述我們的故事和各地開發者的故事，我們必須讓產品故事和插圖都恰到好處。

![](https://github.blog/wp-content/uploads/2021/02/Homepage-image-3.png?)  

我們引入了一種新字體 [Alliance](https://degarism.com/Alliance) 來幫助為我們的故事定下基調
- 它被用於我們所有的新 marketing 網站和品牌計劃，甚至包括擴大的類型規模
- 我們甚至重新構想了我們的 color palette，根據我們的產品顏色創建了一套更飽和的顏色
- 使插圖和產品可以在同一頁面上無縫共存
- 新字體和更新的 color palette 共同提供了廣泛的風格變化
- 使我們能夠以創造性但有凝聚力的方式擴展我們的品牌


我們之所以選擇 Alliance，是因為它具有多種 weight 和風格變體
- 非常適合品牌推廣的易讀性和靈活性

![](https://github.blog/wp-content/uploads/2021/02/Hompage-narrative-4-2.png)  

### 我們用於協作的工具

我們的大部分工作都是在 GitHub 上異步進行的，新 homepage 專案
- 主要工具是 Figma、Slack 和 GitHub
- Figma 是我們所有設計模型、模板和組件、過去迭代和收集反饋的一站式商店
- 我們還有一群非常有才華的人（主要工作人員包括產品設計，網站設計和品牌和內容）
  - 每週見面以定期更新狀態
- 對於更大的里程碑，GitHub 是我們在 project board 中發布最終設計、記錄決策和委派問題的地方
- 當然，GitHub 是我們所有 code 和開發協作的所在地。


我們的團隊在很早的階段就在 code 中製作原型
- 以在實際存在的媒介中驗證設計
- 我們儘早建立 concurrent 的工作流程
- 針對緊密的反饋循環和快速迭代進行優化
- 而不是其他 marketing 團隊中常見的瀑布式工作方式
- 我們的設計師也是開發人員，並在 GitHub 中工作，這會帶來更快的速度、更深入的主題專業知識
- 以及在 design assets 和 code 之間的轉換過程中更少遺漏某些細節

GitHub 的設計師維護了許多為 GiHub.com
- 提供支持的前端 code，構建和維護了 Primer 等 design systems 和我們擴展的 marketing 系統
- 但他們也有使用我們的 Rails monolith 和使用我們的 API 的經驗
- 他們與工程師一起打開和 PR，deploy 到 production，甚至寫 feature flags，使我們能夠在內部配備人員來交付功能，以便更快地測試和更緊密的回饋循環。

此頁面反映了對產品本身的深入了解
- 並講述了有關其工作原理的故事
- 與許多技術 marketing 頁面不同，您不能插入其他產品的 screenshots 並期望 layout 仍然有意義
- 它是專為 GitHub 製作的
- 我們對技術和設計與工程交叉領域工作人員的投資使我們能夠更好地講述真實的故事並進行大規模創新


### 概述

我們的故事開始真正在 Markdown 大綱中形成
- 我們每天都在 GitHub 使用 Markdown 處理問題、PR 和我們自己的 doc
- 對於我們的新 homepage，它確實讓我們能夠查看內容中的主要標誌：標題和 calls to action（aka, our buttons）
  - 以了解我們故事的節奏。儘管 low fidelity of plaintext，但它使我們的思維保持在高 level


![](https://github.blog/wp-content/uploads/2021/02/Hompage-narrative-5.png)  

每個標題都必須打出正確的註釋
- 以幫助構建一個關於開發人員生活的真正引人入勝且真實的故事
- 這意味著要討論開發人員體驗
  - Git、code 託管、open source 等
  - 以及加強開發人員工作流程的新功能 workflows—Actions, Codespaces 等
- 我們編寫的內容將直接與訪問我們 homepage 的人交流，開發人員對開發人員
- 這意味著，不要用 marketing 流行語來淡化它
  - 讓功能、視覺效果、文案和品牌發揮作用。

![](https://github.blog/wp-content/uploads/2021/02/Homepage-image-6.png)  

隨著設計師繼續致力於頁面佈局，我們的內容也在不斷發展，反之亦然。這是一個中等保真度的 Figma，其中包含一些早期的粉紅色內容迭代作為建議。  

隨著我們在大綱上的迭代
- 我們逐漸提高了書面的保真度
- 從功能名稱列表轉移到完整的標題和句子，以解釋每個功能在 GitHub 上的出色、強大或創新程度
- 隨著我們的話語成形，設計迭代也隨之成形，隨著保真度的提高，我們團隊之間開始了反覆協作
- 最終加速並擴大了我們的工作。


### 我們的探索設計

我們採用了開發者故事的初步大綱
- 並開始探索基於前期品牌工作的概念設計
- 我們最早的線框探索只關注視覺示例、佈局和排版處理。

![](https://github.blog/wp-content/uploads/2021/02/Hompage-image-7.png)  

使用來自 GitHub 社群的 PR data
- 我們對後來成為交互式地球的概念的早期說明

![](https://github.blog/wp-content/uploads/2021/02/Hompage-narrative-8.png)  


我們知道
- 我們想讓開發者故事的每個部分都像 visitor 向下 scroll homepage 時栩栩如生
- 我們也知道頁面的內容很長，我們必須創造視覺興趣和變化來保持 visitor 的興趣
- 這為大膽而獨特的佈局打開了大門，真正讓產品栩栩如生，同時仍然基於底層 design system 。

它還讓我們有機會創建一個新的動畫框架
- 用於根據滾動位置觸發動畫
- 其結果不僅是實現了及時的視覺效果
- but also performance gains from not taxing the GPU for animations running outside the viewport.

![](https://github.blog/wp-content/uploads/2021/02/Hompage-narrative-9.png)  

一些早期的探索使用貫穿整個故事的時間線來突出 commit history
- 我們最終選擇了一種不同的方法來分解這些部分，使它們在視覺上更加清晰和更少重複。

### 整合

我們希望第一印像是大膽的
- 這就是為什麼一個令人驚嘆的互動地球迎接我們 homepage 的每一位訪問者
- 它可視化了 GitHub 獨有的全球協作規模
- 顯示了在一個國家打開並在另一個國家合併的真實 PR
- 來自世界各地的超過 5600 萬開發人員和超過 300 萬個組織在 GitHub 上構建他們的軟體

然而，地球只是一個開始
- homepage的 其餘部分放大了我們用戶的一些日常體驗

每個部分都是一個章節，講述了開發人員使用 GitHub 完成最佳工作時的故事
- Hosting your code on GitHub and building on open source
- Collaborating with developers through PR and code review
- New and exciting tools to be your best developer, no matter where you are
- Changing the way we develop with Codespaces
- Automating our problems away with GitHub Actions
- Securing your software before it’s shipped to production
- Being the home for all developers and their many communities

每個部分
- 都通過一個高度可視化但非常簡潔的故事來增強和擴展開發人員可能認為的可能性
- 解決這些章節需要一些時間，但最終它們反映了開發人員在他們的團隊中扮演的許多角色以及我們自己的故事。


我們還將頁面上的一些時刻視為 scratches—the times
- 我們希望 visitor 仔細查看並說 `Wait, what!?` 的時間
- 這兩個時刻在我們的 homepage 中間，我們展示了開發人員如何在任何地方使用我們令人驚嘆的應用程序進行工作
- 以及 Codespaces 如何取消了長達數天的開發環境設置過程
- 我們的故事與這裡的開發人員息息相關
- 我們一起經歷了我們 write, review, ship, and maintai 軟體的方式的根本轉變。


在整個項目中
- 我們 AB test live site 了想法，以減輕對註冊產生負面影響的風險
- 我們的實驗結果有助於指導我們的設計和敘述
- 因為我們更好地了解了 visite 的行為
- 但我們並沒有讓數據決定我們的選擇
- 我們基於為開發人員創造令人驚嘆的體驗並讓我們的價值觀驅動設計過程的目標做出決策。


## Fake globe svg
```js
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" fill="none" height="704" viewBox="0 0 704 704" width="704" class="js-webgl-globe-loading position-absolute left-0 right-0 top-0 bottom-0" style="margin:auto;transform:scale(0.8)">
  <filter id="a" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse" height="560" width="560" x="70" y="70">
    <feFlood flood-opacity="0" result="BackgroundImageFix"/>
    <feBlend in="SourceGraphic" in2="BackgroundImageFix" mode="normal" result="shape"/>
    <feColorMatrix in="SourceAlpha" result="hardAlpha" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 127 0"/>
    <feOffset dx="2" dy="2"/>
    <feGaussianBlur stdDeviation="7.5"/>
    <feComposite in2="hardAlpha" k2="-1" k3="1" operator="arithmetic"/>
    <feColorMatrix type="matrix" values="0 0 0 0 0.447059 0 0 0 0 0.643137 0 0 0 0 0.988235 0 0 0 0.49 0"/>
    <feBlend in2="shape" mode="normal" result="effect1_innerShadow"/>
    <feColorMatrix in="SourceAlpha" result="hardAlpha" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 127 0"/>
    <feOffset dx="1" dy="1"/>
    <feGaussianBlur stdDeviation="3"/>
    <feComposite in2="hardAlpha" k2="-1" k3="1" operator="arithmetic"/>
    <feColorMatrix type="matrix" values="0 0 0 0 0.625 0 0 0 0 0.9325 0 0 0 0 1 0 0 0 0.32 0"/>
    <feBlend in2="effect1_innerShadow" mode="normal" result="effect2_innerShadow"/>
    <feColorMatrix in="SourceAlpha" result="hardAlpha" type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 127 0"/>
    <feOffset dx="-10" dy="-10"/>
    <feGaussianBlur stdDeviation="3"/>
    <feComposite in2="hardAlpha" k2="-1" k3="1" operator="arithmetic"/>
    <feColorMatrix type="matrix" values="0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0.25 0"/>
    <feBlend in2="effect2_innerShadow" mode="normal" result="effect3_innerShadow"/>
  </filter>
  <radialGradient id="b" cx="0" cy="0" gradientTransform="matrix(-199.20400108 -199.20400108 199.20400108 -199.20400108 332.08 338.37)" gradientUnits="userSpaceOnUse" r="1">
    <stop offset=".875" stop-color="#fff"/>
    <stop offset=".937507" stop-color="#3e68ff"/>
    <stop offset="1" stop-color="#03009f" stop-opacity="0"/>
  </radialGradient>
  <linearGradient id="c" gradientUnits="userSpaceOnUse" x1="352" x2="352" y1="331" y2="628">
    <stop offset="0" stop-color="#06060e"/>
    <stop offset="1" stop-color="#0f0e20"/>
  </linearGradient>
  <radialGradient id="d" cx="0" cy="0" gradientTransform="matrix(-5.99972278 523.99965313 -523.99965313 -5.99972278 170 147)" gradientUnits="userSpaceOnUse" r="1">
    <stop offset="0" stop-color="#4b60fb"/>
    <stop offset=".565687" stop-color="#33205d"/>
    <stop offset="1" stop-color="#33205d" stop-opacity="0"/>
  </radialGradient>
  <radialGradient id="e" cx="0" cy="0" gradientTransform="matrix(41.99992987 206.0000547 -206.0000547 41.99992987 292 327)" gradientUnits="userSpaceOnUse" r="1">
    <stop offset="0" stop-color="#354097"/>
    <stop offset="1" stop-color="#243273" stop-opacity="0"/>
  </radialGradient>
  <radialGradient id="f" cx="0" cy="0" gradientTransform="matrix(-84.00137423 185.99914213 -185.99914213 -84.00137423 462 399)" gradientUnits="userSpaceOnUse" r="1">
    <stop offset="0" stop-color="#040d20"/>
    <stop offset="1" stop-color="#040d20" stop-opacity="0"/>
  </radialGradient>
  <circle cx="352" cy="352" fill="url(#b)" r="303" transform="matrix(.98453041 .1752138 -.1752138 .98453041 67.120553 -56.22996)"/>
  <g filter="url(#a)">
    <circle cx="352" cy="352" fill="url(#c)" r="276"/>
    <circle cx="352" cy="352" fill="url(#d)" r="276"/>
    <circle cx="352" cy="352" fill="url(#e)" r="276"/>
    <circle cx="352" cy="352" fill="url(#f)" r="276"/>
  </g>
</svg>
```