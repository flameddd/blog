## Line dev day in Tokyo 2019

![LineDevDay2019_Tokyo_3](/assets/img/LineDevDay2019_Tokyo_3.JPG)  


### Keynote
沒有講太多東西，巴拉巴拉說有 Line 有價值之類的  
不過可以看出
- 很快速的推出各種服務，像 finacial 類得過去12個月就推出很多東西！
  - Line Pay 日本的 User 將近 **4千萬** 了
- 要推出 `mini app` 了！這個後續影響應該蠻大的
- `smart channel` 看來是要為了提供旗下各個服務資料的架構，下面圖說明資料收集 flow
- Line 每天訊息量約 `50億` 封訊息。好奇那 `facebook` 呢？
  - 這讓我想到一個問題，Line 是甚麼規模的公司呢？跟 fb 相比呢？ -> 「對錢沒有概念」問題
- 每天 `390TB(壓縮後)` 的資料量
- `bug bounty` 最近才啟動的

![LineDevDay2019_Tokyo_42_opening](/assets/img/LineDevDay2019_Tokyo_42_opening.JPG)  
![LineDevDay2019_Tokyo_1](/assets/img/LineDevDay2019_Tokyo_1.JPG)  
![LineDevDay2019_Tokyo_44_opening_smart_channel](/assets/img/LineDevDay2019_Tokyo_44_opening_smart_channel.JPG)  



### Strong customer authentication & biometrics using FIDO
- 聽過 `FIDO`
- 根本上就是用 `Private/Public key`
- 加上可以用生物辨識（目前主流應該就是 `face/figureprint`）
- Line Pay （Japan）已經採用了，password less，UX 更好
- 一些流程與 Line 的架構圖
- `W3C WebAuth` <-- google 看看
- `FIDO` 跟 `FIDO2` 是不一樣的嗎？  <-- google 看看

![LineDevDay2019_Tokyo_35_FIDO](/assets/img/LineDevDay2019_Tokyo_35_FIDO.JPG)  
![LineDevDay2019_Tokyo_36_FIDO](/assets/img/LineDevDay2019_Tokyo_36_FIDO.JPG)  
![LineDevDay2019_Tokyo_38_FIDO](/assets/img/LineDevDay2019_Tokyo_38_FIDO.JPG)  
![LineDevDay2019_Tokyo_39_FIDO](/assets/img/LineDevDay2019_Tokyo_39_FIDO.JPG)  
![LineDevDay2019_Tokyo_40_FIDO](/assets/img/LineDevDay2019_Tokyo_40_FIDO.JPG)  



### Inside of Blog; Light and shadow of the service matured for 15 years and challenge chaos and legacy
- 沒看清楚主題，原來是要講 blog 的系統
- 看到別人閃人，才注意到主題，結果我也閃了
- 這場等於沒聽

### LINE's next-generation SDN architecture
- 完全忘了.... 
- 什麼是 `SDN architecture` ？回去 google 看看
- 什麼是 `SRv6 (IPv6 Segment Routing)` ？回去 google 看看
- 也看不懂什麼是 `NW device` ？回去 google 看看

![LineDevDay2019_Tokyo_30_SDN_architecture](/assets/img/LineDevDay2019_Tokyo_30_SDN_architecture.JPG)  

### Security Development Team Senior Security Engineer
- 我有聽這場嗎？

### Building a data-consistent multi-master commerce platform
- 這場是 `LinkedIn` 的人
- 從 User 就 loadbalancer，而且是類似 `sticky session` 的方法把 User 導去某台 db
- 企業版用戶有不同的規則
- 主力還是 `oracle` 因為他們希望要用 RDB
- 透過各種 DB 來架構、`expresso` 是 LinkedIn 自己開發的 `document base DB`

![LineDevDay2019_Tokyo_29_LinkedIn](/assets/img/LineDevDay2019_Tokyo_29_LinkedIn.JPG)  
![LineDevDay2019_Tokyo_28_LinkedIn](/assets/img/LineDevDay2019_Tokyo_28_LinkedIn.JPG)  
![LineDevDay2019_Tokyo_27_LinkedIn](/assets/img/LineDevDay2019_Tokyo_27_LinkedIn.JPG)  


### Adventures in Hardening the Security of LINE's Infra
- 從 `infra` 角度來面對 `Security` 問題
- 從 router 來擋
- 好像是說，事前要先掃描公司內個種 db 服務，盡量挖出來
- iptable 難維護，難共同維護，人為出錯機率高
- 其中目標之一是，`不能停止服務`
- 需要跟其他 team 一起合作
- 需要靠監控系統
- 有 alert system 功能，當某一台大量問題時，例如 access deny 就會有 slack 訊息
  - 這能處理上線後，有人為的失誤，例如 **新機器沒有註冊到此 rule規則中**
- 安裝時
  1. 一開始只會啟動 `detect only mode`
  2. 根據 `detect log`
  3. 最後改為 `deny mode`

方案是
- `host-based firewall` <-- 沒聽過，回去 google 看看

![LineDevDay2019_Tokyo_19_Security_Infra](/assets/img/LineDevDay2019_Tokyo_19_Security_Infra.JPG)  
![LineDevDay2019_Tokyo_21_Security_Infra](/assets/img/LineDevDay2019_Tokyo_21_Security_Infra.JPG)  
![LineDevDay2019_Tokyo_22_Security_Infra](/assets/img/LineDevDay2019_Tokyo_22_Security_Infra.JPG)  
![LineDevDay2019_Tokyo_24_Security_Infra](/assets/img/LineDevDay2019_Tokyo_24_Security_Infra.JPG)  
![LineDevDay2019_Tokyo_26_Security_Infra](/assets/img/LineDevDay2019_Tokyo_26_Security_Infra.JPG)  


### 100+PB scale Unified Hadoop cluster Federation with 2k+ nodes
- 他們要整合 3 套 Hadoop (Line 有太多異質 data 了)
- 後面查到 Hadoop 有新的機制，類似整合的樣子
  - rpc 的版本要一致，說這個好處理
  - 每一台的某個 id 要一致，這個他們研究好一陣子後才找到方法手動改

最後的方案是
- `HDFS Federation` <-- 不知道這啥，回去 google 看看

直接合併到最大的 cluster，的缺點是
![LineDevDay2019_Tokyo_11_Hadoop](/assets/img/LineDevDay2019_Tokyo_11_Hadoop.JPG)  

建一個新的，併過去也是很困難
- 2000 node，你需要 double 才能做這件事情

![LineDevDay2019_Tokyo_12_Hadoop](/assets/img/LineDevDay2019_Tokyo_12_Hadoop.JPG)  
![LineDevDay2019_Tokyo_9_Hadoop](/assets/img/LineDevDay2019_Tokyo_9_Hadoop.JPG)  
![LineDevDay2019_Tokyo_10_Hadoop](/assets/img/LineDevDay2019_Tokyo_10_Hadoop.JPG)  
![LineDevDay2019_Tokyo_14_Hadoop](/assets/img/LineDevDay2019_Tokyo_14_Hadoop.JPG)  
![LineDevDay2019_Tokyo_15_Hadoop](/assets/img/LineDevDay2019_Tokyo_15_Hadoop.JPG)  
![LineDevDay2019_Tokyo_16_Hadoop](/assets/img/LineDevDay2019_Tokyo_16_Hadoop.JPG)  


### PicCell: Flooding your service with AI
- `PicCell` 查不到這個，是 Line 自家的 liarary 嗎？  
- 他們說自己有訓練模型 F1 score 高達 9 成
- 文字辨識
- 後續做 adult content 也判斷率也很高
- 整合其他第三方產品時，準確率非常低，後來要繼續訓練，後續好像可以讓 develope 設計，什麼情況下用什麼模型。
- 後面整合其他服務時，要做文字辨識、QA code 辨識、圖片辨識、影片辨識
  - 甚至仿冒品辨識（仿冒品想辦法做歸類，例如鞋子、手錶）
  - qr code 就讀取進去，抓網站，然後用 adult context 來判斷
  - OCR 說是最難的

![LineDevDay2019_Tokyo_2_PicCell](/assets/img/LineDevDay2019_Tokyo_2_PicCell.JPG)  

架構圖
- `redis` 用來 `cache`，例如同樣一張圖片，短時間重複被請求
- `Spiderman` 說是用來 `server side` 確保有回覆給 client (確保有 callback )
  - 這我也 google 不到，估計是 Line 自家的
- `Librarian` 好像就是依情況選用不同 modal 的工具
- `Timberman` 是用來影片截圖，截片圖片出來做 adult content detect 用的
- kafka 就是針對 stream data

![LineDevDay2019_Tokyo_4_PicCell](/assets/img/LineDevDay2019_Tokyo_4_PicCell.JPG)  

不同服務的 data 來訓練  
![LineDevDay2019_Tokyo_5_PicCell](/assets/img/LineDevDay2019_Tokyo_5_PicCell.JPG)  



### Backend Architecture Design for Analyzing and Processing Massive Blockchain Traffic
- 這場前面部分太淺
- 這場沒翻譯，前半段實在太淺，後面的懶得聽了
- 沒收穫

### other
#### 車內資訊系統、導航，Line 也跨足這個了  
![LineDevDay2019_Tokyo_31_car_info](/assets/img/LineDevDay2019_Tokyo_31_car_info.JPG)  

#### 直撥，看起來目前只有 Japan 區域，感覺短時間不會跨國，感覺重心不在這邊。

![LineDevDay2019_Tokyo_34_Live](/assets/img/LineDevDay2019_Tokyo_34_Live.JPG)  