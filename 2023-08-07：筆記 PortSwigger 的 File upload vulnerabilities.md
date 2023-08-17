
# 2023-08-07：筆記 PortSwigger 的 File upload vulnerabilities.md
### https://portswigger.net/web-security/file-upload#uploading-files-using-put

---------


## file upload 漏洞
指 server 允許 user 在未充分驗證 `filesystem`, `type`, `contents` or `upload` 的情況下上傳至 filesystem
- 如果不能正確執行這些限制，上傳可能被用來上傳任意的、具有潛在危險的檔案。可能包括 remote code execution 的 server side script


---------


### file upload 漏洞的影響是什麼？
影響通常取決於兩個關鍵因素:
- 網站未能正確驗證檔案的某些方面，無論是大小、類型還是內容等
- 檔案上傳成功後，對檔案施加了哪些限制？


最壞的情況下，檔案類型未得到正確驗證，server 允許某些類型(如 `.php`, `.jsp`)作為 code 執行
- 這情況，攻擊者可能上傳一個 server side code，作為 Web shell 使用，而有效地完全控制 server


如果 `filename` 未正確驗證，只需上傳同 `filename` 檔案，就能覆蓋關鍵檔案
- 如果 server 還存在 directory traversal 漏洞，攻擊者甚至可以將檔案上傳到意想不到的位置

如果不能確保檔案大小在預期範圍內，可能導致某種形式的 DoS 攻擊
- 即攻擊者佔滿可用 disk space

-----------

## file upload 漏洞是如何產生的？
考慮到相當明顯的危險性，網站很少會對允許 user 沒有任何限制的上傳檔案
- 常見的情況是，開發人員實施了驗證，但這些驗證存在缺陷 or 容易被繞過

例如，他們可能試圖將危險檔案類型列入黑名單
- 但檢查檔案副檔名時卻沒有考慮到解析差異
- 黑名單也很容易遺漏更隱蔽的檔案類型，而這些檔案類型可能仍然具有危險性

在其他情況下
- 網站可能試圖通過驗證屬性來檢查檔案類型，而這些屬性很容易用 Burp Proxy 或 Repeater 等工具的攻擊者操縱

最後
- 即使是穩健的驗證措施，也可能在構成網站的主機和目錄網路中 application 不一致，從而導致可被利用的差異

------------

## How do web servers handle requests for static files?
Before we look at how to exploit file upload vulnerabilities, it's important that you have a basic understanding of how servers handle requests for static files.

## server 如何處理 static file request?
在了解如何利用漏洞前，要對 server 如何處理 static file request 有基本的了解

server 幾乎完全由 static file 組成
- 當 user 提出 request 時，就會向 user 提供 static file
- 因此，每個 request path 都可以與 server filesystem 上的目錄和檔案層次結構進行 1:1 的 map
- 如今，網站越來越動態化，request path 往往與 filesystem 沒有任何直接關係
- 不過，server 仍要處理一些 static 的 request，包括 stylesheets, image

處理這些 static files 的流程大致相同
- 在某些時候，server 會解析 request path，以識別檔案副檔名
- 然後，server 根據副檔名和 MIME types 之間的預配置 map list 來確定 request 的檔案類型
- 接下來會發生什麼取決於檔案類型和 server 配置


根據檔案類型
- 如果檔案類型是不可執行的，如 image, static HTML page，server 可能會在 HTTP response 中向 client 發送檔案內容
- 如果檔案類型是可執行的，如 PHP 檔案，且 server 被配置為執行這種類型的檔案，那麼在 run script 前，server 會根據 HTTP request 中的 header 和參數來分配變數。由此產生的輸出會在 HTTP response 中發送給 client
- 如果檔案類型是可執行的，但 server 未配置為執行該類型的檔案，則一般會以錯誤作出 response
  - 不過，在某些情況下，檔案內容仍會以 plain text 形式發送給 client。這種 misconfig 偶爾會被用來洩露 source code 和其他敏感資訊

提示  
- `Content-Type` response header 可提供 server 檔案類型的線索
- 如果 application 沒有明確設置該 header，它通常包含 file extension/MIME type mapping 的結果

----------

## 利用不受限制的檔案上傳來部署 web shell
從安全角度來看
- 最糟糕的情況是網站允許你上傳 server 端 script，如 PHP, Java or Python 檔案
- 而且還配置為以 code 形式執行這些 script。這使得在 server 上能輕而易舉建立自己的 Web shell


**Web shell:**  
Web shell 是種惡意 script，攻擊者只需向正確的 endpoint 點發送 HTTP request，就能在 remote web server 上執行任意命令


如果成功上傳 web shell
- 就能完全控制 server
- 意味著你可以讀寫任意檔案、外洩敏感資料，甚至利用 server 對內部基礎設施和網路外的其他 server 進行透視攻擊
- 上傳後，發送對該惡意檔案的 request 將在 response 中返回目標檔案的內容

例如，可以使用下面的 PHP 從 server 的檔案系統中讀取任意檔案:
```php
<?php echo file_get_contents('/path/to/target/file'); ?>
```



通用性更強的  web shell 可能是這樣的:
- script 可通過查詢參數傳遞任意系統命令
```php
<?php echo system($_GET['command']); ?>
```

如下:
```
GET /example/exploit.php?command=id HTTP/1.1
```

----------


## 利用 file uploads 驗證漏洞
你不太可能發現一個對檔案上傳沒有任何防護的網站
- 但是，防禦措施到位並不意味著它們防禦很強

下面介紹 server 嘗試驗證和消毒的一些方法，以及如何利用這些機制中的漏洞獲得 web shell 以執行 remote code  

**缺陷的 file type 驗證:**  
送出 HTML form 時，content type 通常為 `application/x-www-form-url-encoded` 的 `POST` request 中提供資料
- 這 tpye 適合發送簡單的內容，如姓名、地址等，但不適合大量二進制資料，如整個 image 或 PDF
- 這種情況下，content type `multipart/form-data` 是首選


考慮一個包含上傳 image、提供描述和輸入 username 等 field 的 form。form 可能會產生類似下面的 request:

```
POST /images HTTP/1.1
Host: normal-website.com
Content-Length: 12345
Content-Type: multipart/form-data; boundary=---------------------------012345678901234567890123456

---------------------------012345678901234567890123456
Content-Disposition: form-data; name="image"; filename="example.jpg"
Content-Type: image/jpeg

[...binary content of example.jpg...]

---------------------------012345678901234567890123456
Content-Disposition: form-data; name="description"

This is an interesting description of my image.

---------------------------012345678901234567890123456
Content-Disposition: form-data; name="username"

wiener
---------------------------012345678901234567890123456--
```

message body 被分割成不同的部分
- 分別用於 form 的每個 input
- 每個部分都包含一個 `Content-Disposition` header
- 它提供了 input field 相關的基本資訊
- 這些單獨的部分還可能包含自己的 `Content-Type` header，告訴 server 使用該 input 的資料的 MIME type

驗證檔案上傳的方法是檢查特定於 input 的 `Content-Type` header 是否與預期的 MIME type match。例如
- 如果 server 只收 image，則可能只允許使用 `image/jpeg` 和 `image/png` 等類型
- 如果 server 默認信任該 header 的值，就會出現問題
- 如果不進一步的驗證以檢查檔案內容是否確實與 MIME type 相符，就可以使用 Burp Repeater 等工具輕鬆繞

-----------

## 防止在 user 可訪問的目錄中執行檔案
雖然從一開始就防止上傳危險檔案類型更好，但第二道防線是阻止 server 執行任何漏網的 script


作為預防措施
- server 通常只運行其 MIME type 被明確配置為可執行的 script
- 否則，它們可能只會返回某種錯誤資訊，或者在某些情況下，以 plain text 形式提供檔案內容

```
GET /static/exploit.php?command=id HTTP/1.1
Host: normal-website.com


HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 39

<?php echo system($_GET['command']); ?>
```


這種行為可能很有趣，因為它提供了一種 leak source code 的方法，但它使任何建立 web shell 的嘗試都化為烏有  


這種配置通常因目錄而異
- 上傳 user 提供檔案的目錄可能比 filesystem 中其他位置的控制要嚴格得多
- 因為其他位置被認為是 end user 無法訪問的
- 如果能找到方法，將 script 上傳到另個不應該包含 user 提供檔案的目錄，那麼 server 終究可能會執行你的 script


Tip:
- web server 通常使用 `multipart/form-data` request 中的 `filename` field 來確定檔案的名稱和儲存位置


還應注意
- 儘管你可能會將所有 request 發送到同 domain，但這通常指向某種 reverse proxy server，如 load balancer
- 你的 request 通常會由其他 server 在幕後處理，這些 server 的配置也可能不同


**危險檔案類型的黑名單不足:**  
防止 user 上傳惡意 sciprt 的一個更明顯方法是危險案副檔黑名單
- 黑名單做法本身就存在缺陷，很難明確阻止每種可能用於執行 code 的副檔名
- 有時可以用不知名但仍可執行的替代檔案副檔名（如 `.php5`、`.shtml` 等）來繞過黑名單

**覆蓋 server config:**    
如在上面所討論的，除非經過 config，否則 server 通常不會執行檔案
- 例 Apache server 執行 client side request 的 PHP 檔案前，需要在 `/etc/apache2/apache2.conf` 檔案加以下設定:


```
LoadModule php_module /usr/lib/apache2/modules/libphp.so
AddType application/x-httpd-php .php
```

許多 server 還允許開發人員在個別目錄中建立特殊 config file，覆蓋 global config
- 例如，如果存在 `.htaccess` 檔案，Apache server 將從該檔案載入特定目錄的 config



同樣
- 開發人員也可以用 `web.config` 對 IIS server 進行特定目錄 config

這些指令允許向 user 提供 JSON 檔案:
```
<staticContent>
  <mimeMap fileExtension=".json" mimeType="application/json" />
</staticContent>
```

server 會用這類 config，但通常不允許使用 HTTP request 訪問它們
- 不過，偶爾會發現 server 無法阻止你上傳自己的惡意 config
- 在這情況下，即使你需要的檔案副檔名被列入黑名單，你也可以欺騙 server 將任意的自定義副檔名 map 到可執行的 MIME type

**混淆檔案副檔名:**  
即使是詳盡的黑名單，也有可能被混淆繞過
- 比方說，驗證 code 對大小寫敏感，無法識別 `exploit.pHp` 實際上是個 `.php`
- 如果隨後將檔案副檔名 map 到 MIME type 的 code 不區分大小寫，就會使惡意 PHP 檔案通過驗證，最終執行


使用以下技術也能達到類似效果:
- 提供多種副檔名
  - 根據解析檔案名的方法，`exploit.php.jpg` 可能被解釋為 `PHP` 或 `JPG`
- 加尾部字。某些 component 會刪除或忽略尾部空格、點等字： `exploit.php.`
- 嘗試對點、正斜杠和反斜杠使用(雙重) URL encode
  - 如果在驗證副檔名時未對該值進行 decode，但隨後在 server 進行了 decode，這樣也可以上傳原本會被阻止的惡意檔案
  - `exploit%2Ephp`
- 在副檔名前加分號或 URL encode 的 null byte characters
  - 如果驗證是用 PHP 或 Java，但 server 用 C/C++ 處理檔案，可能會導致被視為檔案名結尾的內容出現差異
  - `exploit.asp;.jpg` 或 ` exploit.asp%00.jpg`
- 嘗試使用多字節 unicode 字，這些字可能會在 unicode 轉換或規範化後轉換為 null byte 和點
  - 如果檔案名被解為 UTF-8，`xC0 x2E`、`xC4 xAE` 或 `xC0 xAE` 等序列可能會被轉換為 `x2E`，但在 path 中使用前又會被轉換為 ASCII 字



其他防禦包括刪除或替換危險的副檔名，以防止檔案被執行
- 如果這種轉換不是 recursively，你可以將被禁止的字定位成這樣
- 刪除它仍然會留下有效的副檔名

例如，如果刪除 `.php`，會發生什麼情況？
```
exploit.p.phphp
```


**錯誤檔案內容的驗證:**  
更安全的 server 會嘗試驗證檔案內容是否與預期相符，而不是默認信任 request 指定的 `Content-Type`
- 就圖片上傳功能而言，server 可能會嘗試驗證圖片的某些固有屬性，如 dimensions
  - 如果嘗試上傳 PHP script，根本不會有 dimensions，server 會推斷它不是圖，並拒絕


同樣，某些檔案類型的 header 或 footer 特定的 bytes
- 這些 bytes 可用於確定檔案內容是否符合預期類型。如，JPEG 總是以 `FF D8 FF` bytes 開始

這是驗證檔案類型的更穩健的方法
- 但即便如此也並非萬無一失。使用 `ExifTool` 等，可以輕而易舉地創造一個在 metadata 中包含惡意 code 的多重 JPEG



**利用  file upload race conditions:***
現代 framework 對這類攻擊的防禦能力更強
- 它們一般不會直接將檔案上傳到 filesystem 上的目標位置
  - 會採取些預防措施，比如先上傳至臨時的沙盒目錄，並隨機化名稱以避免覆蓋檔案
  - 然後，會對該臨時檔案進行驗證，只有在認為安全後才將其傳輸到目的地

儘管如此，開發人員有時也會實作自己的檔案上傳處理
- 不僅相當複雜，而且還會引入危險的 race condition，使攻擊者能夠完全繞過驗證

例如，有些網站會將檔案直接上傳到主 filesystem
- 然後在未通過驗證時再次將其刪除
- 這種行為在依賴掃毒軟體等檢查的網站中非常典型
- 這可能只需要幾 ms，但檔案存在於 server 上的短時間內，攻擊者仍有可能執行該檔案


這些漏洞通常非常隱蔽，在 blackbox 測試中很難檢測到
- 除非你能找到洩露相關 source code 的方法

**在 URL-based 的 file uploads 的 race conditions:**  
通過 URL 上傳檔案的 function 中也會出現類似的 race conditions
- 在這情況下， server 必須通過網路取得檔案並建立本地 copy，然後才能執行任何驗證。


由於檔案是使用 HTTP 載入的
- 因此開發人員無法使用其 framework 的 built-in 機制對檔案進行安全驗證
- 相反，他們會手動建自己的流程來臨時儲存和驗證檔案，但這樣做可能不太安全

例如，如果檔案被載到一個具有隨機名稱的臨時目錄中，理論上攻擊者就不可能利用任何 race condition
- 如果攻擊者**不知道目錄的名稱**，就無法 request 檔案以觸發檔案的執行
- 另方面，如果使用偽隨機 function(如 PHP 的 `uniqid()`)產生隨機目錄名，就有可能被暴力破解


為了讓此類攻擊更容易得手，可以嘗試延長處理檔案的時間，從而延長暴力破解目錄名的時間窗口
- 上傳較大的檔案。如果檔案是分塊處理的，就有可能利用這點
- 產生惡意檔案，在檔案開頭寫上 payload，然後寫上大量任意填充字節



-------------


## 不需要執行 remote code ，也能利用上傳檔案漏洞
上傳 server script以執行 remote code 是檔案上傳功能最嚴重的後果，但這些漏洞仍然可以通過其他利用方式

**上傳惡意 client side script:**  
雖然可能無法在 server 上執行 script，但仍可以上傳 script 進行 client side 攻擊
- 如果可以上傳 HTML 或 SVG，就有可能用 `<script>` 產生 stored XSS payload

如果上傳檔案出現在其他 user 訪問的頁面上，他們 browser 在嘗試呈現頁面時就會執行 script
- 注意，由於 [same-origin policy](https://portswigger.net/web-security/cors/same-origin-policy) 的限制，這類攻擊只有在上傳的檔案與你上傳的檔案 same origin 才會起作用


**利用 parsing (上傳檔案)過程中的漏洞:**  
如果上傳的檔案似乎儲存和提供都很安全
- 最後的辦法就是嘗試利用解析或處理不同檔案格式的特定漏洞
- 例如，server 會解析基於 XML 的檔案，如 Microsoft Office `.doc` 或 `.xls` 檔案，這可能是 XXE injection 攻擊的潛在 vector



-----------

## 使用 PUT 上傳檔案
值得注意的是，某些 server 可能被配置為支持 `PUT` request
- 如果沒有適當的防禦，即使 frontend 上沒有上傳功能，也可以通過這種方式上傳惡意檔案

 

```
PUT /images/exploit.php HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-httpd-php
Content-Length: 49

<?php echo file_get_contents('/path/to/file'); ?>
```

 
Tip:
- 可以嘗試向不同的 endpoint 送 `OPTIONS` request，以測試是否有任何 endpoint 支持 `PUT`

------


## 如何防止 file upload 漏洞
一般來說，最有效方法是實作以下所有做法:
- 對照允許副檔名的白名單，而不是禁止副檔名的黑名單檢查檔案
  - 猜測你可能希望允許哪些副檔名要比猜測攻擊者可能試圖上傳哪些副檔名容易得多
- 確保檔案名不包含任何可能被解釋為目錄或遍歷序列（`../`）的字串
- 重新命名上傳的檔案，避免可能導致現有檔案被覆蓋的碰撞
- 在完全驗證之前，不要將檔案上傳到 server 的永久 filesystem
- 盡可能使用已建立的 framework 來預處理檔案上傳，而不是嘗試寫自己的驗證機制