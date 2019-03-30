## Google Developers ML Summit
TensorFlow 2.0 update

Cloud ML APIs -> Cloud AutoML -> TensorFlow  
(easuer to use ---> need more ML Knowledge)  

想想 streaming 的 data 
依時間
依內容

Kaggle  google 旗下的一個網站，上面有 data 可以跑  
ML Kit for firebase  
ML study Jam  

利用 image 辨識來感知空氣污染！ cool  
洗衣店老闆自己學 ML 架構系統，讓客戶可以自動自己來  
Keras 現在 TensonFlow2.0 主要的 API  
 
Browser Side or NodeJS 這邊 TensorFlow 可以做什麼呢？  
感覺用相機、錄影是個機會啊！！  
有什麼應用場景啊？之前那個空污的？  
用來判斷，獲得推文的數量可能性？  
回去 google when use tensonflow.js 看看別人討論  
昨天看的那個 滑鼠軌跡 預測！！！  


舉證乘法  不可能快於 n^2.37  因為兩個維度就是兩維啊  
但 TPU O(3n-2)  
好神，攤平 平方、舉證乘法  
bfloat16精度、準度  
float32   
reference modal 有提供很多，所以可以用來應用  
都來聽了，你不去試試看嗎？連JS 都有得玩  

think big, start smart  
web first, mobile first, AI first  每個 generation 不一樣  

AutoML => your data, our model  

### way build your own models
Cloud TPUs + compute engin  

ML 三種模式
- your data + your model  => Cloud Dataproc + k8s Engine
- our data + our model
- your data + our model

AutoML 的主因是，專家太少，但大家還是有需求  
這連我都可以試試看！  
用影像辨識，來找流行時尚？  
辨識衣服？  

### ML @ iCook
用 google 助理來應用 iCook  
有 case study, google AI recipe-sharing  
借力使力，沒有 AI team 但也嘗試擁抱 AI  

#### ML 落地會遇到什麼問題？：
- training cost is high
- From data science to app service is messy

> You should only spend time on `domain problems`  
把時間花在你落地實際場景，通用問題對你公司的價值不大  
（specific problem）  
因為公司只有僅僅一些資源而已、人、時間、技術等等，所以都應該 focus 在我們的問題  
研發能量  

講者經驗分享，他們
- 解掉 domain 的問題時，很多時候都不是用很厲害的 model  
- 更好的是有新的 data 維度進來，更能加強結果  

From lab to app:
到底這個 train 過後的 model 怎麼在 app 上用呢？有些又不適合做成 API  
- Set up a prediction API
  - what if devices are offlien?
- Embed model in your app
  - How to update?

到底怎麼把 model 怎麼 deploy 在 app 上做 predict ?  

這類流程稱呼 `MLOps` ... 

所以有今天的主題 `ML Kit for firebase`  
> 常見 model 跟 custom model 怎麼部署在 mobile 上

在手機上就做完 perdiction 了！  
辨識完後，還有 kg id 可以打 api 去查其他語言的結果
google 右邊的區塊，就是這個 知識圖
`kg:/m/0grwl`這樣的  
靠，景點辨識也行，孔廟圖片，辨識出地點跟知識  
這 data 只有 google 才做得到  
各式各樣的 detect 都有分
- on device
- on cloud

ML kit 其實已經把 tensorFLow lite 封裝起來了  
東西丟到手機上，問題才開始...：
- 模型應該要 bundle 在 app 上嗎？ 模型可能上百 MB
  - app size 初始 size 就太大了
- 該不該 update model 呢？（哪我把 model 放在 cloud 摟）
  - 透過 cellular 還是 wifi 下載呢？

本地要不要放？雲端要不要放？兩邊的精緻度？能不能網路載？還是只能wifi 載？  
(app整包就不能超過 100mb 了，不然 3g 就沒法下載)  

> 手機上 極度建議用 ML kit 就好，tensonflow lite 很難用，kit 已經封裝了
Could Vision API 就能試試看了  
所以 firebase 是可以放 cloud model 了！！  

### google asisten 上開發 google action
應用比我想像中的還要廣  

GdMlevent329

action 串連 那個匿名發問的東西  可以嗎
dialogflow 串 github
昨天聽的 k
