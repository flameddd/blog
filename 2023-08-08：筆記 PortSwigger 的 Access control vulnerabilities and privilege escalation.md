
# 2023-08-08：筆記 PortSwigger 的 Access control vulnerabilities and privilege escalation.md


Ref:
- https://portswigger.net/web-security/access-control
- https://portswigger.net/web-security/access-control/security-models
- https://portswigger.net/web-security/access-control/idor

---------

## 訪問控制漏洞和權限升級
在本節中，我們將討論什麼是訪問控制安全，描述權限升級和訪問控制可能出現的漏洞類型，並總結如何防止這些漏洞。

### 什麼是 access control ?
access control (或 authorization) 是對誰(或什麼)可以執行的操作或訪問的資源進行限制
- 在網路 application 中，access control 取決於 authentication 和 session management
  - Authentication identifies (身份驗證)可識別 user 身份，確認 user 的真實身份
  - session management 識別同一 user 的後續 HTTP request
  - access control 確定 user 是否被允許執行操作
  
access control 被破壞是通常是關鍵的安全漏洞。access control 設計和管理是複雜而動態的問題，它將業務、組織和法律限制應用到技術中  

從 user 角度看，access control 可分為以下幾類:  

**水平的 向 access controls:**  
- 是種機制，將資源的 access controls 限制在被明確允許訪問這些資源的 user 範圍內
- 不同的 user 可以訪問同類型的資源集。如，銀行 app 允許 user 查看自己帳號的交易和付款，但不允許查看其他 user 帳號

**與 context 相關的 access controls:**
- 根據 application 的狀態或 user 與 app 的互動情況來限制功能和資源的訪問
- 可防止 user 以錯誤的順序執行操作。如，零售網站可能會阻止 user 在付款後修改購物車的內容

------------

## 壞掉(錯誤)的  access controls 範例

**垂直權限升級(Vertical privilege escalation):**  
- 如果 user 可以訪問不允許訪問的功能，這就是 vertical privilege escalation
- 如，一個非管理員 user 可以訪問管理員頁面，可以刪除 user 帳號，這就是 vertical privilege escalation


**不受保護的功能:**  
- 如，管理員的頁面有 link 管理功能，但 user page 卻沒有。然而，user 可能只需直接瀏覽相關的管理 URL 就能訪問管理功能

例如，網站可能會在以下 URL 有敏感功能:
```
https://insecure-website.com/admin
```
 
 
事實上，任何 user 都可以訪問該 URL，而不僅僅是在 user interface 上擁有該功能管理 user
- 在某些情況下，管理 URL 可能會在其他位置公開，如 `robots.txt`:

```
https://insecure-website.com/robots.txt
```


即使 URL 沒有在任何地方公開，攻擊者也可以用 wordlist brute-force 找出敏感功能的位置  


在某些情況下，敏感功能並沒有得到有力的保護，而是通過提供一個較難預測的 URL 來加以隱藏:
- 這是所謂的 security by obscurity
- 隱藏敏感功能不能提供有效的 access control，user 有可能通過各種方式發現被混淆的 URL


例如，以下 URL 有 application 的管理功能:
```
https://insecure-website.com/administrator-panel-yb556
```



攻擊者可能無法直接猜到。但是，application 仍有可能向 user 透露 URL
- 如，URL 可能會在 JavaScript 中洩露:
  - 如果 user 是 admin，該 script 會在 interface 上加個 link
  - 不過，所有 user 都可以看到包含 URL 的 script，無論其角色如何

```html
<script>
var isAdmin = false;
if (isAdmin) {
	// ...
	var adminPanelTag = document.createElement('a');
	adminPanelTag.setAttribute('https://insecure-website.com/administrator-panel-yb556');
	adminPanelTag.innerText = 'Admin panel';
	// ...
}
</script>
```

**基於參數的 access control 方法:**  
有些 application 會在 login 時確定 user 訪問權限或角色
- 然後將資訊存在 user 可控制的位置，如隱藏 field, cookie 或 query string 參數
- application 會根據 submit 的值作出後續訪問  access control
- 這種方法從根本上說是不安全的

如
```
https://insecure-website.com/login/home.jsp?admin=true
https://insecure-website.com/login/home.jsp?role=1
```


**平台 config 錯誤導致 access control 被破壞:**  
- 一些 application 根據 user 的角色限制對特定 URL 和 HTTP method 的訪問
- 從而執行 access controls


如，application 可能會配置如下規則:
```
DENY: POST, /admin/deleteUser, managers
```


規則拒絕 managers group 訪問 URL `/admin/deleteUser` 上的 `POST`
- 在這情況，可能會出現各種問題，導致繞過控制


一些 framework 支持各種非標準 HTTP header
- 可用於覆蓋原始請求中的 URL，如 `X-Original-URL` 和 `X-Rewrite-URL`
- 如果網站使用嚴格的前端控制來限制基於 URL 的訪問，但 application 允許通過 request header 覆蓋 URL，那麼下面的 request 就有可能繞過控制:

```
POST / HTTP/1.1
X-Original-URL: /admin/deleteUser
...
```

上述前端控制根據 URL 和 HTTP method 限制訪問
- 另種攻擊可能，有些網站在執行操作時允許使用其他 HTTP method 方法
- 如果攻擊者可以使用 `GET` (或其他)方法在受限 URL 上執行操作，就可以規避平台層實作的控制


**因 URL 匹配不一致而導致控制被破壞：**  
傳入 request 時，網站對 path 與定義 endpoint 匹配的嚴格程度各不相同
- 如，網站可能會容忍大小寫不一致的情況此向 `/ADMIN/DELETEUSER` request 仍可能被 map 到相同的 `/admin/deleteUser`
- 但如果控制機制的容忍度較低，可能會將 request 視為兩個不同的 endpoint，從而無法執行限制
  - 如果使用 Spring 啟用了 `useSuffixPatternMatch`，也會出現類似的差異
    - 這允許將帶有任意文件副檔名的路徑對應到不帶文件副檔名的等效 endpoint
    - 換句話說，`/admin/deleteUser.anything` request 將匹配 `/admin/deleteUser`
    - 在 Spring 5.3 之前，該選項默認為啟用
- 其他系統中，可能會遇到 `/admin/deleteUser` 和 `/admin/deleteUser/` 是否被視為不同 endpoint
  - 在這情況，只需在 path 後加斜線，就可以繞過控制



水平權限升級(Horizontal privilege escalation)攻擊會使用與垂直權限升級類似類型的利用方法
- 如，user 通常可以使用下 URL 訪問自己的帳號頁面


```
https://insecure-website.com/myaccount?id=123
```


如果攻擊者將 `id` 改為另個 user，攻擊者就可以訪問另個 user 帳號，並獲得相關資料和功能  



在某些 application 中，可利用的參數沒有可預測的值
- 如，用 GUID 來識別 user，而不是遞增數字。這情況，攻擊者可能無法猜測或預測另個 user id
- 但是，屬於其他 user 的 GUID 可能會在 application 中引用 user 的其他地方洩露，如 user message 或評論


在某些情況下，application 會檢測到 user 未被允許訪問資源，並返回 302 到 login page
- 但是，包含 302 的 response 仍可能包含目標 user 的一些敏感資料，因此攻擊仍然成功。



**從橫向到縱向的權限升級:**    
通常情況下，通過入侵權限較高的 user，水平攻擊可以轉變為垂直
- 如，水平可讓攻擊者重置或獲取屬於其他 user 的密碼
- 如果攻擊者以管理 user 為目標併入侵其帳號，那麼就可以獲得管理訪問權限，從而實作縱向權限升級
  - 如果目標 user 是 admit，攻擊者就可以訪問管理頁面
  - 該頁面可能會洩露管理員密碼或提供更改密碼的方法，也可能提供直接訪問特權功能的權限

如，攻擊者可以使用已描述過的橫向權限升級參數修改，訪問另個 user 的帳號頁面:
```
https://insecure-website.com/myaccount?id=456
```


**不安全的直接對象引用：***  
- IDOR 是 access control 漏洞的子類
- 當 application 用 user 提供的 input 直接訪問 object，而攻擊者可以修改 input 以獲得未經授權的訪問時，就是 IDOR


**Multi-step processes 中的 Access control 漏洞:**    
許多網站通過一系列步驟實現重要功能。當需要各種 input 或選項，或當 user 需要在執行操作前審查和確認細節時，通常會這樣做。如，更新 user 詳細資訊的管理功能可能涉及以下步驟:
1. 載入包含特定 user 詳細資訊信息的表單
2. 送出修改
3. 查看變更並確認修改


有時，網站會對其中一些步驟實作嚴格控制，但忽略其他步驟
- 假設第一和第二步正確實作控制，但第三步卻沒有。網站假定 user 只有在完成了受到控制的第一步後，才會進入第三步
- 在這裡，攻擊者可以跳過前兩步驟，直接送出帶有所需參數的第三步 request，從而獲得未經授權的 function 訪問權限

**基於 `Referer` 的 access control:**  
一些網站控制基於 request 的 `Referer` header
- `Referer` header 一般由 browser 加到 request 中，以指示 request 是從哪個頁面發起的
  - 假設一個 application 對位於 `/admin` 的主管理頁面實作了強大的控制，但對於 `/admin/deleteUser` 等子頁面，只檢查 `Referer`
  - 如果 `Referer` 包含主 `/admin` URL，則允許 request
  - 在這情況，攻擊者可以完全控制 `Referer`，因此可以偽造對子頁面的直接 request，提供`Referer`，從而獲得未經授權的訪問


**基於 location 的控制:**  
有些會根據 user 的地理位置控制
- 這可能適用於銀行或媒體服務，服務適用國家法律或業務限制
- 通常可以用 VPN 或 client 地理定位機制來規避控制

-----------------

## 如何防止 access control 漏洞
採取深度防禦方法和應用以下原則來防止漏洞:

- 絕對不要只依賴混淆(obfuscation)來進行 access control 
- 除非某項資源打算公開訪問，否則默認情況下拒絕訪問
- 盡可能使用單一的 application 機制來執行 access control
- 在 code 方面，強制要求開發人員聲明每個資源允許的訪問權限，並在默認情況下拒絕訪問
- 徹底審核和測試 access control，確保其按設計運行

---------------


### Access control 安全模型
常見的安全模式

---------


## 程式化 access control (Programmatic access control)
user 權限矩陣存在 db 中，參照該矩陣以程序方式實作控制
- 這種控制方法可包括角色、群組或單個 user、流程集合或工作流，而且可以高度細化


----------


## 酌情 access control (Discretionary access control, DAC)
自由裁量控制
- 對資源或功能的訪問受 user 或指定 user group 的限制
- 資源或功能的所有者可以向 user 分配或委託訪問權限
- 這模式高度細化，對單個資源或功能和 user 的訪問權限進行定義。因此，設計和管理會變得非常複雜


--------

 

## 強制 access control (Mandatory access control, MAC)
是種集中控制的訪問控制
- 在這種系統中，主體對某些對象(文件或其他資源)的訪問受到限制
- 與 DAC 不同，資源的 user 和所有者沒有能力委託或修改其資源的訪問權限
- 這種模式通常與基於軍事許可的系統有關

--------

## 基於 role 角色的 access control (Role-based access control, RBAC)
定義指定訪問權限的 role
- 將 user 分配給單/多個角色
- 與其他模型相比，RBAC 提供更強的管理功能
- 如果設計得當，還能提供足夠的粒度，從而在複雜的 application 中提供可管理的控制
- 例如，採購員可以被定義為具有採購分類帳功能和資源子集訪問權限的角色
  - 當員工離開或加入組織時，訪問控制管理就簡化為定義或撤銷採購員角色的成員資格
- 當有足夠多的角色來正確調用控制時，RBAC 是最有效的，但也不能太多，以免使模型過於複雜，難以管理