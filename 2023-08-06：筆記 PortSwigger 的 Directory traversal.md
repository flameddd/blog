
# 2023-08-06：筆記 PortSwigger 的 Directory traversal.md
### Ref: https://portswigger.net/web-security/file-path-traversal#how-to-prevent-a-directory-traversal-attack


---------------

## Directory traversal (目錄遍歷)
Directory traversal 允許攻擊者讀取運行 application 的 server 上的任意文件
- 這可能包括 aoolication code 和資料、後端系統的憑證以及敏感的操作系統文件
- 某些情況下，攻擊者可能能夠寫入 server 上的任意文件，從而修改 application 資料或行為，並最終完全控制 server



----------


## 通過 directory traversal 讀取任意文件
考慮一個顯示待售商品圖片的購物 app。圖片通過類似下面的 HTML 顯示:

```html
<img src="/loadImage?filename=218.png">
```


`loadImage` URL
- 接收 `filename` 參數，並返回指定文件的內容
- 圖本身存在 disk 的 `/var/www/images/` 位置
- 要返回圖，application 會將請求的文件名追加到該基本目錄，並使用文件系統 API 讀取文件內容

在上述情況中，application 從以下文件路徑讀取內容:
```
/var/www/images/218.png
```


該 application 沒有對 directory traversal 攻擊的防禦
- 攻擊者可以 request 這 URL 從 server 器的文件系統中搜索任意文件:


```
https://insecure-website.com/loadImage?filename=../../../etc/passwd
```


這會導致 application 從以下文件路徑讀取資料: 
```
/var/www/images/../../../etc/passwd
```

在 file path 中
- `../` 是有效的，表示在目錄結構中向上移動一級
- 連續三個的 `../` 從 `/var/www/images/` 到 filesystem root 目錄

因此實際讀取的文件是
```
/etc/passwd
```


在 Unix 系統上，這是個標准文件，包含 server 上註冊用戶的詳細資訊  

在 Windows 中
- `../` 和 `..\`都 是有效的 directory traversal

windows 的攻擊會像這樣:
```
https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini
```

-------

## 利用 file path traversal 的常見障礙
許多防禦措施可以被規避  


如果 application 刪除或阻止 user 提供 filename 中的 directory traversal 序列，那就有可能使用各種技術繞過防禦  
- 用 filesystem root 目錄的絕對路徑，如 `filename=/etc/passwd` 直接引用文件而不使用任何遍歷序列
- 也可以使用嵌套遍歷序列(nested traversal sequences)，如 `....//` or `....\/`，當內部序列被剝離時，它們將恢復為簡單的遍歷序列


在某些情況下，例如 URL 或 `multipart/form-data` request 的 `filename` 參數中，server 可能會在將你的 input 傳遞給 application 之前剝離任何 directory traversal 序列
- 有時，可以通過 URL encode，甚至是 double URL encode 來繞過這種消毒
  - 將 `../` encode 為 `%2e%2e%2f` 或 `%252e%252e%252f`
  - 各種非標準 encode，如 `..%c0%af` 或 `..%ef%bc%8f`，也可能起到作用


**Burp Suite Professional:**  
- Burp Intruder 提供了 predefined payload list (Fuzzing - path traversal)，其中包含各種編碼路徑遍歷序列，可以嘗試使用


如果 application 要求filename 必須以預期的 base folder (如 `/var/www/images`)開頭
- 那可以在所需的基本 folder 後加入遍歷序列

如
```
filename=/var/www/images/../../../etc/passwd
```

如果 application 要求 filename 必須以預期的副檔名(如 `.png`)結尾
- 可以用 null byte 在所需副檔名前有效地終止文件路徑

如
```
filename=../../../etc/passwd%00.png
```

------

## 如何防止 directory traversal 攻擊
防止 directory traversal 漏洞最有效方法是完全避免將 user 提供的 input 傳給 filesystem API
- 這樣做的 application function 都重寫，以更安全的方式提供相同的行為

如果認為向 filesystem API 傳 user 提供的 input 是不可避免的，則應使用兩層防禦措施來防止攻擊:
-  application 在處理 user input 之前對其進行驗證
  - 理想情況下，驗證應與允許值白名單進行比較
  - 如果對於所需的功能不可能做到這點，那驗證就應確認 input 只包含允許的內容，如純英文字母數字等
- 驗證 input 內容後，application 應將 input 內容追加到 base directory，並用 platform filesystem API 對路徑進行規範化
  - application 應驗證規範化後的路徑是否以預期的 base directory 為起點


根據用 user input驗證文件規範路徑的簡單 Java code example:
```java
File file = new File(BASE_DIRECTORY, userInput);
if (file.getCanonicalPath().startsWith(BASE_DIRECTORY)) {
  // process file
}
```

