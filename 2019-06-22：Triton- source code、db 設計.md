### Triton: source code、db 設計
幾篇文長的整理，學習些思維

### 有人問：應該要如何寫出又好又快的source code。
這是工作上的實戰心得，會寫得有點散亂的。  
很多人（包括我）都說：
- 寫程式不太難，需求改來改去下寫程式才真的他媽的難……  
- 很多的爛程式，原因都是在寫到一半時才發現需求根本不是這樣子，然後專案限期不能改。

結果就是暴力亂來不管品質亂寫算了。當然，
- 有５０％情況是邪惡老闆／用戶覺得「不用自己動手，所以改一改很容易～」
- 但是有另外５０％是：用戶說不出自己的真正需求。

跟用戶談需求時
- 你不應該覺得１００％理解用戶說什麼便以為談好需求了！
- 因為這份需求有９９％不是用戶真正想要的東西！

給一個例子：  
在前公司工作時，預計有大型活動會引發突然性的超高流量，可能會觸發系統趴掉。  

```
主管：「我們需要做一個ratelimiting來保護系統！」
我：「用nginx做就可以。但是這樣子，在爆流時，現有正在用的用戶不就會有可能機會均等地被擋住了嗎？」
主管：「對啊，這會影響到用戶感受的（User experience），不行不行～」
我：「所以，當超高流量出現時。現有在系統中的用戶要不受影響，但是新的用戶不受影響？」
我：「新的用戶，是指之前三十分鐘都沒有什麼API Request的用戶。這定義可以嗎？」
主管：「對～～～」
我：「所以，我的新功能是要紀錄最近３０分鐘的用戶的Auth Token。系統流量過高時，如果用戶的Auth Token是之前三十分鐘出現過，就讓他正常使用系統，否則就擋住他吧。」
主管：「對～～～」
```

別當一個單純聽指令的coder，身為一個成熟的developer，你應該進一步去推想：
- 有沒有邏輯矛盾？
- 用戶體驗是否不受影響？

很可能，在談需求的過程
- 你要用 Excel、小畫家之類的工具去做wireframe
- 跟用戶一步一步做role-play去引導用戶吐出真正的需求。

這樣子，你才會在之後的開發中減少需求改動的可能性的。

### 好的程式，應該高度重視：single responsibility principle

很多人口中說要寫好程式，然後現實卻爛得不知謂的原因就是沒重視這個。（唉）  
例子：  

services/user 這個package，是
- 用來處理用戶有關的 Request 的
- 用戶的個人頁面的 RequestHandler 放這裡。
- -> 然後嘛，負責user_balance改動也放這裡囉

所以嘛，services/itemPurchase
- 在購買東西時要改動 service/user 就自然要 import services/user

別忘記services/user也有一堆的RequestHandler，所以最終不同的services packages就是互相import，整個dependency graph像蜘蛛網一樣  
最後你的程式就會死於cyclic import沒法comile囉  


以下是違反 single responsibility principle 時，你會感覺到的臭味道：

1. dependency graph很亂很雜，你要很多「hacky」邪惡手段。
 - 例子：ObjectA 是需要用上 ObjectB 的
 - 但是在 objA 建立時 objB 因為各種原因還未存在
 - 需要之後 objA.SetB(objB) 來邪惡地丟進去

2. 同一 package 內，有不同面向的程式碼。import這個package時，你常常只會用到其中一個面對
 - 例子：剛才 services/user，RequestHandler 是讓 framework 來把 Request 丟進去處理的
 - 而 UpdateUserBalance 則是被其他的 services 使用

3. unit testing 時，需要建立一起根本沒被用上的 object
4. 統中有大量god-object，（不計testcase）你有一堆要數千行的class

### 不單單是「把package拆小」這麼簡單的
single responsibility principle 不單單是「把package拆小」這麼簡單的  
你還應該養成良好的紀律  
例如：  

你發現一堆 services 都在使用 services/user 的 UpdateUserBalance()
- 那麼這就代表 UpdateUserBalance() 應該是一個 library，而不是 service package
 - 應該把 UpdateUserBalance() 抽離放到「獨立的package」

這樣子才能避開 services/user 內的 RequestHandler，被不合理地 depends on  

例如：  
business logic 應該集中在 service package（和其相關的支援性的 helper package ）
- 然後 utils 是存放 non-business logic 的程式碼

這樣才可以分隔
- 「為追求極限效能而寫得很髒的utils」跟
- 「很麻煩的business logic的services」

然後才可以跟據 package location 來大約猜到這個 package 的功能性的 像是：
- services/money 大約是用來處理跟用戶的錢包有關的 Request 吧
- 而 utils/money 就可能是提供十進位運算的 library，用來支援跟錢有關的運算


### 因為高流量而當機

有正確設定好的系統，其實是不會因為高流量而當掉的
- 只是大部份用戶連不上網頁，等到連上後球鞋已經被搶光光了
- 主要系統停止服務的原因也不是因為流量。

基本上我這３年內見證過的系統趴掉的原因：

1. 程式有 bug
 - ６０％，像是寫了 while 1 loop 啦，開了db/file/http connection 忘了關

2. 有人直接連進 production database/redis，然後做了笨事
 - ２０％，像是跑一些要一小時以上的報表，過程中把資料鎖住了
 - 然後引發滾雪球最後整個資料庫全部資料都被鎖住。

3. devops 流程錯誤
 - １０％，像是只把程式上了production，但相關的 schema change 卻沒執行啦（或是反過來……）

4. amazon 底層的內部 bug
 - １０％，像是 aws ec2 找不到 aws RDS 的 DNS
 - 常常不明情況 CPU 100% 的 aws aurora Mysql


我多次強調：
- database connection pool 不止是用來回收用完的 db connection，讓效能提升的。
- database connection pool 是系統爆炸性高流量時，其 max_conn 設定讓資料庫無法負擔的流量卡在application layer乖乖排隊
 - 而不是全衝一起把資料庫炸掉！
 - 剛才的小店１０００人全都跑入去……最後結果就是全部人都沒東西吃

正確設定database connection pool，就是
- 「別」在４核伺服器上設什麼 max_conn = 100 或 1000這種跟「無限」一樣瘋狂的數字啦
- 一般而言：你的 application 有多少個 thread 能同時在跑
  - 把這個數字除２到除４就是「一般」系統的default setting了。

例子：你家是用Intel I7 7700架laravel伺服器，I7 7700應該是4 core 8thread的（希望沒錯），那麼其connection pool的max_conn應該就是２到４之間。

把以上東西再延伸：  
- 不止database 要這樣子保護的
 - redis也要
 - rabbitMQ也要
 - amazon SQS其實也要的～
這樣子你系統底層的component才是不容易當掉的。  

然後把以上觀念再延伸一層：
- database 要不炸掉是要 application layer 去保護
- 那麼要 application layer 不被大量在排隊等待的request炸掉？
  - 當然是更上方的load balancer那一層囉
- 你有好好設定 nginx，別讓同一時間太多 Request 跑進 application layer 嗎？
- 有直接發好人卡（HTTP 503）給連排隊空間也滿掉的過多的request嗎？

要系統不趴掉，就是忍住沉悶，平日做stress testing去正確設定好系統容量，和設定好每一層的max_conn之類的參數囉  

在 App. Server 上設定 database connection pool，並且把 pool max size 設定為cpu running threads的25% - 50%就好  

有人問：
postgresql/oracle/mariadb 這堆主流的 RDBMS，都是可以設定其連線上限的
- 為何不是 stress test 看看 RDBMS 能頂住多少 concurrent connection
- 然後在 RDBMS 設定 max_connection 參數不就完了嗎？

我的回答：  
App. Server 層的 database connection pool，是為了
- Ａ：效能
- Ｂ：防止底層資源被流量炸掉

身為資料庫的管理員，正常都會期待 App. Server 層是有在用 connection pool 的

然後 connection pool 在一個 Request 用完其 db connection 時
- 會單純讓 connection 保持 idle 而不是立即斷掉的

重點：
- 在 App. Server 有打開 connection pool 下，絕大部份 db connection 都是 idle 的
- 到底會有多少是 idle，其很看 connection pool 內部怎麼寫和具體設定
- 所以現在有多少 db connection，只反映出你開了多少個 docker container
- 而不是反映現在流量多大。

如果 RDBMS 有設定剛剛好的 max_connection，然後你的RDBMS是多個系統共用的
- 你的 max_connection = 90，然後某一天有一個新手以ASP .NET開發了一個小專案連上RDBMS
- 當下次要deploy新版系統時，就會發現除了ASP .NET開發的那模組，其他的都連不上RDBMS了
- 補充：ASP .NET內建的connection pool，其default pool size為100
  - 所以他很快便把所有connection佔光光了～

簡單而言：設定max_connection，這樣子一個App. Server的設定錯誤就能讓癱瘓掉  

如果以 RDBMS max_connection 來保護系統
- 但沒在 App. Server 層使用 connection pool
- 高流量時就會有 Request 沒法拿到Connection
- 然後你的end user就會收到一大堆的500 Internal Server Error了

又有人問：
- 為何在 App. Server 的 Connection pool 的 max pool size 應該是 App. Server CPU thread 的25% - 50%之間？

「25% - 50%之間」是根據我之前實戰的經驗的，你的數字可能跟我不完全相同  
我之前在hypebeast.com的系統嘛：  
- Average Response Time大約是200ms，然後其中１／３時間是用在RDBMS上面的
- 在系統正常時等待DB的時間佔了 1/3，那就代表
 - 每２個在 App. Server 運作中的 CPU thread
 - 就有一個相對應的DB connection。（這是簡單算術吧）

當 DB 變成 performance bottleneck 時
- 每一 Request 在 DB 所要用上的時間只會更多的
- 所以這樣子 DB 時間就從 33% 慢慢升到最後的 90%

所以如果 connection pool 沒設上限
- CPU thread : DB connection 就會從本來的 2 : 1 慢慢瘋到 1 : 10

把 Connection pool 的 pool size 設定成「App. Server CPU thread的25% - 50%之間」
- 就是當 DB 開始變慢
  - CPU thread : DB connection 試圖從本來理想的 2 : 1 比例要掉下去時
  - 讓 Request 因為等候 connection pool 而進不了RDBMS
 