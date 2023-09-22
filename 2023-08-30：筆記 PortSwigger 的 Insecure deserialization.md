
# 2023-08-30：筆記 PortSwigger 的 Insecure deserialization.md


Ref:
- https://portswigger.net/web-security/deserialization
- https://portswigger.net/web-security/deserialization/exploiting

--------------

## 什麼是序列化(serialization) ?
Serialization 是將資料結構(如 object)轉為扁平格式的過程
- 這種格式可以作為連續的字節流發送和接收
- 對資料進行序列化可以簡化以下操作
  - 將複雜資料寫進 memory, file or DB
- 通過網路、application 不同組件之間或在 API call 中發送複雜資料



最重要的是，在 Serialization object 時，其 state 也會被持久化
- 換句話說，object 的屬性及值都會被保留下來

在不同的語言時，serialization 可能被稱為 `marshalling`(Ruby) or `pickling`(Python)




--------------

## 什麼是不安全的 deserialization ?
不安全的 deserialization 是指網站對 user 可控資料進行 deserialization
- 這有可能使攻擊者操縱，從而將有害資料傳到 application code 中


甚至有可能用完全不同 class 的 object 替換 serialized object
- 令人擔憂的是，網站可用的任何 class 的 object 都會被 deserialization 和實例化
- 因此，不安全的 deserialization 有時被稱為 object injection 漏洞


一個意外 class 的 object 可能會導致異常
- 不過，此時破壞可能已經造成。許多基於 deserialization 攻擊都是在反序列化完成之前完成的
- 意味著，即使網站本身的功能不與惡意 object 直接互動，deserialization 過程本身也可能發起攻擊
- 因此，strongly typed languages 的網站也容易受到這些攻擊



--------------


## deserialization 漏洞是如何產生的?
不安全的 deserialization 通常是因為人們普遍缺乏對 deserialization user 可控資料的危險性的認識
- **理想情況下，user input 根本不應該被 deserialization**


然而，有時網站會認為他們是安全的，因為他們對資料額外檢查
- **這種方法往往是無效的，因為幾乎不可能實施驗證或消毒來應對每一種可能發生的情況**
- **這些檢查從根本上說也是有缺陷的**
- 因為它們依賴於在資料被 deserialized **後**對其進行檢查，而在許多情況下，這已經太晚了，無法阻止攻擊

出現漏洞的另個原因是
- 人們通常認為 deserialized objects 是可信的
- 特別是在使用 binary serialization format 的語言時，開發人員可能會認為 user 無法有效地讀取或操作資料
  - 可能需要更多努力，但攻擊者利用 binary serialized objects 的可能性與利用基於字串格式的 object 的可能性一樣大


由於現代網站存在大量 dependencies
- 每個 library 都有自己的 dependencies，所以形成難以管理安全的龐大環境
- 由於攻擊者可以建立任何這些 class 的實例，因此很難預測在惡意資料上可以呼叫哪些 method
- 如果攻擊者能夠將一長串意想不到的 method call 串聯起來，將資料傳遞到與初始源完全無關的 sink 中，要預測並堵住每個潛在漏洞幾乎是不可能



簡而言之，可以說不可能安全的 deserialize untrusted input


--------------

## 不安全的 deserialization 會產生什麼影響?  
影響可能會非常嚴重，因為它為大大增加了攻擊面，提供一個切入點
- 它允許攻擊者以有害的方式重複使用現有的 application code，導致許多其他漏洞，通常是 remote code execution
- 即使不能 remote code execution 的情況下，不安全的 deserialization 也會導致權限升級、任意文件訪問和拒絕服務等攻擊


--------------

## 利用不安全的 deserialization 漏洞
下面以 PHP, Ruby, Java deserialization 為例，教如何利用一些常見情況
- 展示利用不安全的 deserialization 實際上比許多人認為的要容易得多
- 如果你能使用預先構建的 gadget chain (gadget chains)，在黑盒測試中甚至可以做到這一點

下面還將指導建立自己的基於 deserialization 的 high-severity 攻擊
- 雖然這些攻擊通常需要 source code 訪問權限，但一旦理解了基本概念，學習起來也會比想像中容易
- 下面特別介紹以下主題:
  - 如何識別不安全的 deserialization
  - 操作序列化對象
  - 將惡意資料傳到危險的網站功能中
  - Inject 任意 object types
  - 呼叫 Chaining method 以控制資料流進入危險 sink gadgets
  - 手動建立自己的 advanced exploits
  - PHAR deserialization

備註:
- 許多範例都基於 PHP，但大多數漏洞利用技術對其他語言也同樣有效


--------------

### 如何識別不安全的 deserialization
不管是 whitebox/blackbox，識別不安全的 deserialization 都相對簡單
- 你應查看傳入網站的所有資料，並嘗試識別任何看似 serialized data 的內容
- 如果你知道不同語言使用的格式，serialized data 就能比較容易地識別出來
- 一旦識別出 serialized data，就可以測試是否能夠控制它
- 對於 `Burp Suite Professional` 的 `Burp Scanner` 會自動標記任何看起來包含序列化 object 的 HTTP message




**PHP serialization format:**  
- PHP 大多使用人類可讀的字串格式，字母代表資料類型，數字代表長度
- 例如，一個 user object 的屬性為

```php
$user->name = "carlos";
$user->isLoggedIn = true;
```


序列化後，object 可能看起來像這樣:
```
O:4:"User":2:{s:4:"name":s:6:"carlos"; s:10:"isLoggedIn":b:1;}
```

這可以解釋如下：
- `O:4: "User"`: 具有 4 個字符 class name "User" 的 object
- `2`: object 有 2 個屬性
- `s:4: "name"`: 第一個屬性的 key 是 4 個字的字串 `"name"`
- `s:6: "carlos"`: 第一個屬性的 value 是 6 個字的字串 `"carlos"`
- `s:10: "isLoggedIn"`: 第二屬性的 key 是 10 個字的字串 `"isLoggedIn"`
- `b:1`: 第二個屬性的 value 是 boolean `true`


PHP 序列化的 native method 是 `serialize()` 和 `unserialize()`
- 如果可以訪問 source code，則應首先查找 `unserialize()`，然後進一步研究

**Java serialization format:**  
- 有些語言(如 Java)使用 binary serialization formats，這種格式更難讀取，但還是可以識別
- 例如，序列化的 Java object 總是以相同的字節開始，在十六進制中編碼為 `ac ed`，在 Base64 中為 `rO0`
- 任何實現 interface `java.io.Serializable` 的 class 都可以被 serialized 和 deserialized
- 如果可以訪問 source code，注意任何使用 `readObject()` 的 code，該 method 用於從 `InputStream` 中讀取和 deserialize 資料


--------------

## 操作序列化對象
利用某些 deserialization 漏洞就像更改序列化 object 中的 attribute 一樣簡單
- 由於 object 狀態是持久化的，因此可以通過研究序列化資料來識別和編輯有趣的屬性值
- 然後，你就可以通過網站的 deserialization 過程將惡意 object 傳入網站。這就是利用漏洞的第一步



一般來說，在操作序列化對象時有兩種方法
1. 是直接編輯 byte stream 形式的 object
2. 用相應的語言寫簡短的 script，建立新 object 並將其序列化
    - 在使用 binary serialization formats 時，第二種方法通常更容易操作




**修改 object 屬性:**  
- 在篡改資料時，只要攻擊者保留了有效的序列化對象，deserialization 過程就會使用修改後的屬性值建立 server side object
- 簡單的例子，一個網站使用序列化的 `User` object 在 cookie 中存 user session data
- 如果攻擊者在 HTTP request 中發現了這個序列化對象，可能會對其進行解碼，發現以 byte stream:

```
O:4:"User":2:{s:8:"username";s:6:"carlos";s:7:"isAdmin";b:0;}
```

`isAdmin` 屬性是明顯的關注點
- 攻擊者只需將該值改為 `1(true)`，用修改後的值覆蓋當前 cookie 即可
- 單獨來看，這不會產生任何影響。但是，假設網站使用此 cookie 檢查當前 user 是否有訪問某些管理功能的權限:


```php
$user = unserialize($_COOKIE);
if ($user->isAdmin === true) {
// allow access to admin interface
}
```


這種易受攻擊的 code 根據 cookie 中的資料，就可以輕鬆實現權限升級(privilege escalation)  


這種簡單的情況在實際中並不常見
- 不過，以這種方式編輯屬性值表明，不安全的 deserialization 暴露了大量攻擊面，而這正是訪問這些攻擊面的第一步


練習題: [Lab: Modifying serialized objects](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-objects)  



**修改 data types:**  
- 在比較不同資料類型時，PHP 的鬆散比較 (`===`) 行為而特別容易受到此類操作的影響
- 例如，整數和字串鬆散比較，PHP 會將字串轉換為整數，這意味著 `5 == "5"` 為 `true`


不同尋常的是，這也適用於任何以數字開頭的字母數字字串
- PHP 將根據開頭的數字有效地將整個字串轉換為整數值
- 字串的其餘部分將被完全忽略。`5 == "5 of something"` 實際上被視為 `5 == 5`

當把字串與整數 `0` 比較時，這種情況就更奇怪了:
- 為什麼？因為字串中沒有數字，即 `0` 個數字。PHP 將整個字串視為整數 `0`

```php
0 == "Example string" // true
```


考慮一下這種鬆散的比較運算符與來自 deserialized 對象的用戶可控資料結合使用的情況。這有可能導致危險的邏輯缺陷:

```php
$login = unserialize($_COOKIE)
if ($login['password'] == $password) {
// log in successfully
}
```

假設攻擊者修改了 password attribute，使其包含整數 `0`，而不是預期的字串
- 只要存儲的密碼不是以數字開頭，該條件就會始終返回 `true`，繞過身份驗證
- 注意，這只是因為 deserialization 保留了資料類型。如果直接從 request 中取密碼，`0` 將轉換為字串，條件將返回 `false`
- 在修改任何序列化對象格式中的資料類型時，一定要記得更新序列化資料中的任何 tpye 標籤和長度。否則，object 將被損壞並無法 deserialized


練習題: [Lab: Modifying serialized data types](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-modifying-serialized-data-types)  



在處理 binary formats 時，建議用 `Hackvertor` extension
- Hackvertor 可以將序列化資料修改為字串，它會自動更新 binary formats，並相應調整偏移量。可以節省大量的手動操作




--------------

## 使用 application 功能
除了簡單地檢查 attribute value 外，網站功能還可能對來自 deserialized object 的資料執行危險操作
- 在這情況，你可以使用不安全的 deserialization 傳遞意外資料，並利用相關功能進行破壞



例如，網站 `Delete user` 功能的一部分
- user 個人資料圖片是通過 `$user->image_location` 屬性中的 file path 來刪除的
- 如果 `$user` 是通過序列化對象建立的，攻擊者就可以通過將 `image_location` 設為任意 file path 的修改後對像傳入來利用這一漏洞
- 刪除自己的 user 帳號後，也會刪除這個任意檔案




練習題: [Lab: Using application functionality to exploit insecure deserialization](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-using-application-functionality-to-exploit-insecure-deserialization)  

 
這個範例依賴於攻擊者通過 user 可訪問的功能手動呼叫危險方法
- 如果利用漏洞自動將資料傳到危險方法中，不安全的 deserialization 就更加有趣。用 magic methods 可以實現這點

--------------

## Magic methods
`Magic methods` 是種特殊的方法子集，無需顯式呼叫
- 相反，每當特定事件或場景發生時，它們就會被自動呼叫
- `Magic methods` 是各種語言中 object-oriented programming 的常見 feature
- 有時會在方法名稱的 prefix 或周圍加上雙引號來表示它們



開發人員可以在 class 中加上 magic methods，以便預先確定在相應事件或情況發生時應執行哪些 code
- 呼叫 magic methods 的具體時間和原因因方法而異
- PHP 中最常見的例子之一是 `__construct()`，它在 class 的 object 實例化時被呼叫
  - 類似於 Python 的 `__init__`
- 通常情況下，constructor magic methods 包含初始化實例屬性的 code。不過，開發者可以自定義 magic methods，以執行他們想要的任何 code



Magic methods 被廣泛使用，其本身不代表漏洞
- 但是，當執行的 code 處理攻擊者可控制的資料時，它們就會變得危險
- 攻擊者可以在滿足相應條件時自動呼叫 deserialized data 上的方法
- 最重要的是，有些語言有在 deserialization 過程中自動呼叫 magic methods
  - 如 PHP `unserialize()` 會查找並呼叫 object 的 `__wakeup()` magic methods



在 Java deserialization 中，`ObjectInputStream.readObject()` 也是如此
- 該方法用於從 initial byte stream 中讀取資料，本質上類似於重新初始化序列化對象的構造 function
- 不過，`Serializable` class 也可以宣告自己的 `readObject()` method，如下:


```java
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException
{
  // implementation
}
```


以這種方式聲明的 `readObject()` 方法就像 magic method
- 在 deserialization 過程中會被呼叫。這允許 class 更緊密地控制其自身 field 的 deserialization 


你應密切關注任何包含此 magic methods 的 class
- 它們允許在 object fully deserialized 之前將資料從序列化 object 傳到網站 code 中。這是建立更高級漏洞的起點

--------------

##   Injecting 任意 objects

在 object-oriented programming 中，object 可用的方法由其 class 決定
- 因此，如果攻擊者可以操縱作為 serialized data 傳入的  class of object，就可以影響 deserialization 後甚至 deserialization 過程中執行的 code

Deserialization 方法通常不會檢查對象
- 這意味著你可以傳入網站可用的任何可序列化類的對象，而該對象將被 deserialization
- 這實際上允許攻擊者建立任意 class 的實例。這個 object 不是預期的 class 並不重要
- 意外的 object 類型可能會導致 application 邏輯出現異常，但此時惡意 object 已經實例化


如果攻擊者可以訪問 source code，就可以詳細研究所有可用的 class
- 他們可以查找包含 deserialization 方法的 class，檢查是否有對資料執行危險操作的 class
- 攻擊者就可以傳入該 class 的序列化對象，進行攻擊


練習題: [LAB Arbitrary object injection in PHP](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-arbitrary-object-injection-in-php)  



包含這些 deserialization 方法的 class 還可用於發起更複雜的攻擊
- 其中涉及一長串方法呼叫，即所謂的 gadget chain (`gadget chain`)



--------------

##  Gadget chains
gadget 是存在於 application 中的片段程式碼，可以幫助攻擊者實現特定目標
- 單個 gadget 可能不會直接對 user input 進行任何有害操作


然而，攻擊者的目的可能只是呼叫一個 method，將他們的輸入傳到另個 gadget 中
- 通過這種方式將多個 gadgets 串在一起，攻擊者就有可能將 input 傳到一個危險的 sink gadget 中，從而造成最大的破壞


重要的是，與其他一些利用漏洞的類型不同，gadget chain 不是由攻擊者構建的鍊式方法的 payload
- 所有 code 都已存在於網站上。攻擊者唯一能控制的是傳入的資料
- 這通常是通過在 deserialization 過程中調用的 magic method 來實現的，有時也稱為 kick-off gadget



**使用預構建的 gadget chains:**  
手動識別 gadget chains 是相當艱鉅的過程
- 沒有 source code 訪問權限幾乎是不可能的
- 幸運的是，你可以先嘗試使用一些預構建的 gadget chains


有幾種工具可以提供一系列預先發現的 pre-discovered chains
- 這些 chains 已在其他網站上被成功利用。即使你沒有訪問 source code 的權限，也可以用工具來識別和利用不安全 deserialization 漏洞
- 之所以能採用這種方法，是因為廣泛使用了包含可利用 gadget chains 的 library
  - 如，如果 Java 的 Apache Commons Collections library 中的 gadget chains 可以在某網站上被利用，那麼任何其他網站也可以利用


**ysoserial:**  
- Java deserialization 工具之一是 `ysoserial`
- 它可以讓你從提供的 gadget chains 中選擇一個你認為目標 application 正在使用的 library，然後傳入想執行的命令
- 它會根據所選 chain 建立一個合適的 serialized object
- 這仍然需要一定的嘗試和錯誤，但比手動構建自己的 gadget chains 要省力得多。
- https://github.com/frohoff/ysoserial


在 Java 16 及以上版本中，你需要設置一系列命令行參數，以便 Java 運行 ysoserial。如:
```shell
java -jar ysoserial-all.jar \
   --add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.trax=ALL-UNNAMED \
   --add-opens=java.xml/com.sun.org.apache.xalan.internal.xsltc.runtime=ALL-UNNAMED \
   --add-opens=java.base/java.net=ALL-UNNAMED \
   --add-opens=java.base/java.util=ALL-UNNAMED \
   [payload] '[command]'
```


練習題: [Lab: Exploiting Java deserialization with Apache Commons](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-exploiting-java-deserialization-with-apache-commons)  



使用以下工具來幫助你快速檢測幾乎所有 server 上不安全的 deserialization:
- `URLDNS` chain:
  - 觸發所提供 URL 的 DNS lookup
  - 最重要的是，它不依賴於使用特定易受攻擊 library 的目標 application，而且可以在任何已知 Java 版本中執行
  - 這使它成為最通用的檢測 gadget chain
  - 如果在流量中發現序列化對象，可以產生一個 object，觸發與 Burp Collaborator server 的 DNS 互動
    - 如果是這樣，就可以確定目標上發生了 deserialization
- `JRMPClient` 是另一個可用於初始檢測的通用 chain:
  - 它會使 seerver 嘗試與提供的 IP 建立 TCP connection。你要提供原始 IP 而不是 hostname
  - 在所有出站流量(包括 DNS lookwups)都被防火牆隔離的環境中
  - 可以嘗試用兩個不同的 IP 產生 payload:
    - 一個本地 IP 和一個防火牆外部 IP
  - 如果 application 立即 response 本地地址的 payload，但外部地址的 payload，導致 response 延遲，這表明 gadget chain 起作用是，因為 server 試圖連接到防火牆地址
  - 在這種情況下，即使是 blind，response 中細微的時間差也能幫助你檢測 server 上是否發生了 deserialization



**PHP通用 Gadget Chains:**  
- 大多數經常出現不安全 deserialization 漏洞的語言都有相應的概念驗證工具
- PHP 可以使用 `PHP Generic Gadget Chains` (PHPGGC)



練習題: [Lab: Exploiting PHP deserialization with a pre-built gadget chain](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-exploiting-php-deserialization-with-a-pre-built-gadget-chain)  


注意
- 漏洞是 user 可控資料的 deserialization，而不僅僅是網站 code 或任何 library 中存在 gadget chain
- gadget chain 只是 inject 有害資料後操縱 data flow 的一種手段。這也適用於依賴於 deserialization 不受信任資料的各種內存損壞漏洞
- 換句話說，即使網站設法堵住了所有可能的 gadget chain，它仍然可能存在漏洞


**使用已經被挖掘的 gadget chains:**  
- 在目標 application 使用的 framework 中，不總是有專門的工具可用來利用已知的 gadget chains
- 在這情況，上網查找是否有任何記錄在案的漏洞利用方法，以便手動調整
- 調整 code 可能需要對語言和 framework 有基本了解，有時可能還需要自己序列化對象，但這種方法仍然比從頭開始省力得多
 

練習題: [Lab: Exploiting Ruby deserialization using a documented gadget chain](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-exploiting-ruby-deserialization-using-a-documented-gadget-chain)  




--------------

## 創造自己的漏洞
當現成的 gadget chains 和網路上分享的漏洞利用不成功時，你需要建立自己的 exploit


要成功建立自己的 gadget chain，幾乎肯定需要訪問 source code
- 第一步是研究 source code，找出包含在 deserialization 過程中呼叫的 magic method 的 class
- 評估該 magic method 的 code，看看它是否直接對 user 可控屬性執行了任何危險操作



如果 magic method 本身不可利用，它可以作為 gadget chain 的啟動工具
- 研究啟動小工具呼叫的任何 method
- 這些 method 中是否有對你控制的資料進行危險操作的？
- 如果沒有，請仔細研究它們隨後呼叫的每個 method，依此類推


重複這個過程，跟踪你可以訪問的值，直到你走到死胡同，或者發現一個危險的 sink gadget，你的可控資料被傳到這個 gadget 中  


一旦想出如何在 application code 中成功構建 gadget chain
- 下一步就是建立包含 payload 的序列化對象
- 這只需研究 source code 中的 class declaration，並建立一個有效的序列化對象


處理 binary formats 可能特別麻煩
- 在對現有 object 進行細微修改時，可以直接處理 bytes
- 如果要進行更重大的更改，例如傳入全新的 object，這樣做很不切實際
- 通常情況下，用目標語言寫自己的 code 來產生資料並將其序列化要簡單得多

在建立自己的 gadget chain 時，要留意是否有機會利用這個額外的攻擊面來觸發二次漏洞

練習題: [Lab: Developing a custom gadget chain for Java deserialization](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-developing-a-custom-gadget-chain-for-java-deserialization)  


通過仔細研究 source code，你可以發現更長的 gadget chains 可能允許構建 high-severity 性攻擊，通常包括 remote code execution  


練習題: [Lab: Developing a custom gadget chain for PHP deserialization](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-developing-a-custom-gadget-chain-for-php-deserialization)  

--------------

## PHAR deserialization
目前為止，主要研究了利用網站明確 deserialization user input 的 deserialization 漏洞
- 然而，在 PHP 中，即使沒有明顯使用 `unserialize()` 方法，有時也有可能利用 deserialization

PHP 提供了幾種 URL 風格的 wrappers，可用於在訪問 file path 時處理不同的協議
- 其中一個是 `phar://` wrappers，它為訪問 PHP Archive (`.phar`) file 提供了一個 stream interface 


文件顯示，`PHAR` manifest 包含 serialized metadata
- 如果對 `phar://` steam 執行任何 filesystem 操作，這些 metadata 都會被 implicitly deserialized
- 意味著，只要能將 `phar://` steam 傳到 filesystem 方法中，它就有可能成為利用不安全 deserialization 的載體

對 `include()` 或 `fopen()` 等明顯具有危險性的 filesystem methods，網站很可能採取了應對措施，以降低被惡意使用的可能性
- 但，像 `file_exists()` 這樣沒有明顯危險性的方法可能就沒有那麼好的保護措施了


這種技術還需要以某種方式將 `PHAR` 上傳到 server
- 一種方法是使用 image upload
- 如果能建立 polyglot file，將 `PHAR` 偽裝成 `JPG`，有時就能繞過驗證檢查
- 如果你能迫使網站從一個 `phar://` steam 中讀取這個 polyglot file `JPG`，那麼你通過 `PHAR` metadata inject 的任何有害資料都會被 deserialized
- 由於 PHP 在讀取 steam 時不會檢查副檔名，因此文件是否使用圖片副檔名並不重要


只要網站支持 object 的 class，`__wakeup()` 和 `__destruct()` 方法都可以用這種方式呼叫
- 這樣就可以用這種技術啟動一個 gadget chain 




練習題: [LAB: Using PHAR deserialization to deploy a custom gadget chain](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-using-phar-deserialization-to-deploy-a-custom-gadget-chain)  


這個別出心裁的技術曾被列入 portswigger 的 2018 年十大網路黑客技術
- https://portswigger.net/research/top-10-web-hacking-techniques-of-2018#6



--------------


## 利用 memory corruption 進行 deserialization 攻擊
通常會有公開記錄的  memory corruption vulnerabilities，通過不安全的 deserialization 來利用
- 這些漏洞通常會導致 remote code execution

Deserialization method (如 PHP `unserialize()`) 很少針對此類攻擊進行加固
- 因此暴露了大量攻擊面。這本身並不總是漏洞，因為這些方法本來就不是用來處理 user 可控 input 的

--------------


## 如何防止不安全的 deserialization 漏洞
一般來說，除非絕對必要，否則應避免對 user input 輸入進行 deserialization
- 在許多情況下，deserialization 可能帶來的嚴重漏洞以及防範這些漏洞的難度都超過了 deserialization 的好處


如果**需要對不可信來源的資料 deserialization**，則應採取強有力措施，**確保資料未被篡改**
- 例如，實施 digital signature 來檢查資料的完整性
- 記住，任何檢查都必須在開始 deserialization 流程之前進行。否則，就沒有用


如果可能，避免使用`通用 deserialization` 功能
- 這些方法的序列化資料包含原始 object 的所有屬性，可能包括敏感資訊的 private fields
- 相反，你可以建立自己特定的 serialization methods，這樣至少可以控制哪些 field 會被暴露

最後，請記住，漏洞是 user input 的 deserialization，而不是隨後處理資料的 gadget chain 的存在
- 不要依賴於試圖消除在測試過程中發現的 gadget chain 
- 由於網站幾乎肯定存在 cross-library dependencies，試圖全部消除它們是不切實際的
- 在任何時候，公開的 memory corruption exploits 也是因素，這意味著無論如何，你的 application 都可能存在漏洞
