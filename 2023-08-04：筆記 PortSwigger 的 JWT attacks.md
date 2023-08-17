
# 2023-08-04：筆記 PortSwigger 的 JWT attacks.md
### Ref: https://portswigger.net/web-security/jwt


---------------

(Burp Suite Professional 2022.5.1 起，有掃描 JWT 功能: **Target** > **Issued definitions** tab )  

Burp Suite 的 JWT 相關功能
- https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts

-----------

## 什麼是 JWTs?
`JSON web tokens` (JWTs):
- 是種標準化格式，用於在系統間發送加密簽名的 JSON 資料
- 理論上，它可以包含任何類型的資料，但最常用於發送有關使用者的資訊(`claims`)，作為身份驗證、session handling 和訪問權限控制機制的一部分

與傳統 session tokens 不同
- server 需要的所有資料都存儲在 JWT 本身的 client side
- 這使 JWT 成為使用者需要與多個 backend server 進行無縫互動的高度分佈式網站的首選

**JWT format:**  
A JWT consists of 3 parts: a header, a payload, and a signature. These are each separated by a dot, as shown in the following example:



**JWT 格式:**  
- JWT 由三部分組成：header, payload 和 signature
- 「點」隔開每個部分

如下:  
```
eyJraWQiOiI5MTM2ZGRiMy1jYjBhLTRhMTktYTA3ZS1lYWRmNWE0NGM4YjUiLCJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTY0ODAzNzE2NCwibmFtZSI6IkNhcmxvcyBNb250b3lhIiwic3ViIjoiY2FybG9zIiwicm9sZSI6ImJsb2dfYXV0aG9yIiwiZW1haWwiOiJjYXJsb3NAY2FybG9zLW1vbnRveWEubmV0IiwiaWF0IjoxNTE2MjM5MDIyfQ.SYZBPIBg2CRjXAJ8vCER0LA_ENjII1JakvNQoP-Hw6GG1zfl4JyngsZReIfqRvIAEi5L4HV0q7_9qGhQZvy9ZdxEJbwTxRs_6Lb-fZTDpW6lKYNdMyjw45_alSCZ1fypsMWz_2mTpQzil0lOtps5Ei_z7mM7M8gCwe_AGpI53JxduQOaB5HkT5gVrv9cKu9CsW5MS6ZbqYXpGyOG5ehoxqm8DL5tFYaW3lB50ELxi0KsuTKEbD0t5BCl0aCR2MBJWAbN-xeLwEenaqBiwPVvKixYleeDQiBEIylFdNNIMviKRgXiYuAvMziVPbwSgkZVHeEdF5MQP1Oe2Spac-6IfA
```




header 和 payload 部分只是 `base64url` encode 的 JSON
- header 包含有關 token 本身的 metadata
- payload 則包含有關使用者的實際 `claims`


例如，可以 decode 上面 token 的 payload，以顯示以下 `claims`: 
```json
{
  "iss": "portswigger",
  "exp": 1648037164,
  "name": "Carlos Montoya",
  "sub": "carlos",
  "role": "blog_author",
  "email": "carlos@carlos-montoya.net",
  "iat": 1516239022
}
```



大多情況下
- 任何可以訪問 token 的人都可以輕易讀取或修改這些資料
- 因此，JWT 的安全性都在**很大程度上依賴於加密簽名**

**JWT 簽名:**
- 簽發 token 的 server 通常通過對 header 和 payload 進行 hashing 來生成簽名
- 某些情況下，還會對 hash 進行加密
- 無論採用哪種方式，過程都需要 `secret signing key`
- 這種機制為 server 提供方法，以驗證 token 資料是自己所簽發，沒有被篡改過:
  - 由於簽名直接來源於 token 的其他部分，因此更改 header 或 payload 的任一個字都會導致簽名不匹配
  - 如果不知道 server 的 `secret signing key`，就不可能為給 header 或 payload 產生正確的簽名


**JWT vs JWS vs JWE**:  
JWT 規範實際上非常有限
- 它只定義一種格式，用於將資訊 `claims` 表示為可在雙方之間傳輸的 JSON object
- 實務中，JWT 並沒有真正作為一個獨立的實體使用
- `JSON Web Signature` (JWS) 和 `JSON Web Encryption` (JWE) 規範擴展 JWT 規範，定義了實際實施 JWT 的具體方法


換句話說
- JWT 通常是一種 JWS token 或 JWE token
- 當使用 JWT 一詞時，幾乎總是指 JWS token
  - JWE 非常相似，只是 token 的實際內容是加密的，而不僅僅是 encode

--------

### 什麼是 JWT 攻擊？
JWT 攻擊涉及使用者向 server 送修改過的 JWT，以達到惡意目的
- 通常，這目標是通過冒充另一個已通過身份驗證的使用者來 bypass authentication 和 [access controls](https://portswigger.net/web-security/access-control)。



--------


## JWT 攻擊有什麼影響？
JWT 攻擊通常很嚴重
- 如果攻擊者能用任意值建立自己的有效 token，就有可能提升自己的權限或冒充其他使用者，完全控制他們的帳號


--------

## JWT 攻擊漏洞是如何產生的？
JWT 漏洞
- 通常是 application 本身在處理 JWT 時存在缺陷造成的
- 與 JWT 相關的各種規範在設計上相對靈活，允許網站開發人員自行決定許多實施細節
- 這導致他們即使使用了熱門的 library，也會意外引入漏洞

這些實作缺陷通常意味著
- JWT 的簽名沒有得到正確驗證，攻擊者就能篡改 token payload 傳遞給 application 的值
- 即使簽名得到可靠的驗證，其是否真正可信也在很大程度上取決於 server 的 secret key 是否保密
- 如果 secret key 洩露，或者被猜或暴力破解，攻擊者就可以為任意 token 產生有效的簽名，從而破壞整個機制


----------


## 利用有缺陷的 JWT 簽名驗證
從設計上講，server 通常不會存儲任何有關其簽發的 JWT 的資訊
- 相反，每個 token 都是完全獨立的實體
- 這樣做有幾個好處，但也帶來了一個根本問題: server 實際上對 token 的原始內容一無所知，甚至不知道原始簽名是什麼
- 因此，如果 server 沒有正確驗證簽名，就無法阻止攻擊者對 token 的其他部分進行任意更改

例如，一個包含以下 claims 的 JWT:

```json
{
  "username": "carlos",
  "isAdmin": false
}
```


如果 server 根據 `username` 辨識 session
- 修改其值可能會使攻擊者冒充其他登錄使用者
- 同樣，如果 `isAdmin` 被用於 access control，這就為攻擊提供了簡單的 vector


**接受任意簽名:**  
JWT library 通常提供一種驗證/解碼 token 的方法
- 如，Node.js `jsonwebtoken` library 有 `verify()` 和 `decode()`



有時，開發人員會混淆這兩個方法
- 只將傳入的 token 傳遞給 `decode()`，**實際上意味著 application 根本不驗證簽名**


**接受沒有簽名的 token:**  
- JWT header 包含 `alg` 參數
- 它告訴 server 使用哪種 algorithm 對 token 簽名，因此 server 在驗證簽名時需要使用哪種 algorithm

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```


這本身就存在缺陷
- 因為 server 別無選擇，只能隱式地信任使用者可控制的 token input 
- 而此時，token 根本沒有經過驗證。換句話說，攻擊者可以直接影響 server 檢查 token 是否可信的方式


JWTs
- 可以使用一系列不同的 algorithm 進行簽名，但也可以不簽名
- 在這種情況下，`alg` 被設為 `none`，表示 `unsecured JWT`
- 由於這樣做的危險顯而易見，server 通常會拒絕沒有簽名的 token
- 不過，由於這種**過濾依賴於字串解析**，因此**有時可以使用混淆(obfuscation)繞過過濾**
  - (如混合大寫和 unexpected encode)

注意:
- 即使 token 沒有簽名，payload 部分仍必須以 trailing dot 結束

---------


## Brute-forcing secret keys
有些 signing algorithms，如 `HS256`(`HMAC` + `SHA-256`)，使用任意獨立字串作為 secret key
- 像密碼一樣，關鍵是 secret key 不能被攻擊者輕易猜到或暴力破解
- 否則，攻擊者就能用任何 header 和 payload 建立 JWT，然後用 key 重新簽署 token
 

實作 JWT application 時，開發人員有時會犯錯
- 如忘記更改 default 或 placeholder secrets key
- 甚至他們會 copy/past 網上找到的 code，然後忘記改 example 的 key
- 在這情況，攻擊者用[眾所周知的 secrets wordlist](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list)來暴力破解 server secret 是輕而易舉的


**hashcat (Brute-forcing secret keys):**  
暴力破 secret keys 的工具
- [手動安裝 hashcat](https://hashcat.net/wiki/doku.php?id=frequently_asked_questions#how_do_i_install_hashcat)
- (mac 可以用 brew 的樣子)


只需要一個來自目標 server 的有效簽名 JWT 和 [secrets wordlist](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list)
- 將 JWT 和 wordlist 傳進去:

```
hashcat -a 0 -m 16500 <jwt> <wordlist>
```

Hashcat
- 用 wordlist 中每個 secret 對 JWT 的 header 和 payload 進行簽名
- 然後將產生的簽名與 server 上的原始簽名進行比較
- 如果匹配，就會以下列格式輸出所識別的 secret 以及其他各種詳細資訊
- 如果多次執行，則需要包含 `--show` flag 來輸出結果
- 由於 hashcat 在本地機器上執行，即使使用大的 wordlist 也非常快

```
<jwt>:<identified-secret>
```


一旦確定了 secret，就可以用它為任何 JWT header 和 payload 產生有效簽名
- 如何在 Burp 中對修改後的 JWT 重新簽名: [Editing JWTs](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts#editing-jwts)


如果 server 使用的是薄弱的 secret
- 甚至有可能逐個字地暴力破解，而不是使用 wordlist




-----------

## JWT header parameter injections
`JWS` 規範，只有 `alg` header 是強制性的
- 但實際上，JWT header (也稱為 `JOSE` header)通常包含其他幾個參數
- 攻擊者對以下參數特別感興趣:
  - `jwk` (JSON Web Key): 提供 embedded JSON object，代表 key
  - `jku` (JSON Web Key Set URL): 提供 URL，server 可從中獲取包含正確 key 的 key set
  - `kid` (Key ID): 提供 ID，在有多個 key 可選擇的情況下，server 可使用該 ID 識別正確的 key
    - 根據 key 的格式，這可能有一個匹配的 `kid` 參數


這些使用者可控參數分別告訴接收方 server 在驗證簽名時使用哪個 key，如何利用這些參數 inject 使用自己的任意 key 而非 server key 簽名的修改過的 JWT ?

**通過 `jwk` 參數 *Injecting self-signed JWTs:**  
The JSON Web Signature (JWS) specification describes an optional `jwk` header parameter, which servers can use to embed their public key directly within the token itself in JWK format.


JSON Web Signature (JWS) 規範描述了一個可選的 `jwk` header 參數
- server 可使用該參數將自己的公鑰直接嵌入 JWK 格式的 token 中

**JWK:**  
JWK (JSON Web Key) 是種將 keys 密鑰表示為 JSON object 的標準化格式  

下面的 `JWT` header 就是個例子:  

```json
{
  "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
  "typ": "JWT",
  "alg": "RS256",
  "jwk": {
    "kty": "RSA",
    "e": "AQAB",
    "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
    "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9m"
  }
}

```
 

理想情況下，server 只應使用有限的公鑰白名單來驗證 JWT 簽名
- 然而，配置錯誤的 server 有時會使用任何嵌入 `jwk` 參數的 key
- 你可以使用自己的 RSA 私鑰對修改後的 JWT 進行簽名，然後在 `jwk` header 中嵌入匹配的公鑰，從而利用這一行為


雖然可以在 Burp 中手動加或修改 `jwk` 參數，但 [JWT Editor extension](https://portswigger.net/bappstore/26aaa5ded2f74beea19e2ed8345a93dd) 提供有用的功能，可幫助測試此漏洞:
1. extension 載入後，切到 **JWT Editor Keys** tab
2. [產生一個新的 RSA key](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts#adding-a-jwt-signing-key)
3. 向 Burp Repeater 發送包含 JWT 的 request
4. 在 message editor 中，切換到 extension 產生的 **JSON Web token** tab，然後 [modify](https://portswigger.net/burp/documentation/desktop/testing-workflow/session-management/jwts#editing-jwts) token 的 payload
5. 點  **Attack**，然後選擇 **Embedded JWK**。出現提示時，選擇新產生的 RSA Key
6. 發送 request ，測試 server 的 response

也可以通過自己加 `jwk` header 來手動執行這種攻擊
- 不過，可能還需要更新 JWT 的 `kid` header 參數，以匹配 embedded key 的 `kid`
  - extension 的  built-in 攻擊功能可以幫你完成這步


**通過 `jku` 參數 Injecting self-signed JWTs:**  
有些 server 不直接使用 `jwk` header 參數來嵌入公鑰
- 而是讓你使用 `jku`(JWK Set URL) header 參數來引用包含密鑰的 JWK set
- 驗證簽名時，server 會從該 URL 獲取相關 key

**JWK Set:** 
JWK Set 是個 JSON object
- 包含代表不同 keys 的 JWK array
- 像這樣的 JWK sets 有時會通過 standard endpoint 可以公開存取 (如 `/.well-known/jwks.json`)  

例子:
```json
{
  "keys": [
    {
      "kty": "RSA",
      "e": "AQAB",
      "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
      "n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ"
    },
    {
      "kty": "RSA",
      "e": "AQAB",
      "kid": "d8fDFo-fS9-faS14a9-ASf99sa-7c1Ad5abA",
      "n": "fc3f-yy1wpYmffgXBxhAUJzHql79gNNQ_cb33HocCuJolwDqmk6GPM4Y_qTVX67WhsN3JvaFYw-dfg6DH-asAScw"
    }
  ]
}

```


更安全的網站只會從可信 domain 獲取密鑰，但有時也可以利用 parsing discrepancies 繞過此類過濾
- SSRF 主題中介紹了[範例](https://portswigger.net/web-security/ssrf#ssrf-with-whitelist-based-input-filters)


**Injecting self-signed JWTs via the kid parameter**:  
Servers may use several cryptographic keys for signing different kinds of data, not just JWTs. For this reason, the header of a JWT may contain a `kid` (Key ID) parameter, which helps the server identify which key to use when verifying the signature.


**通過 `kid` 參數 injecting self-signed JWTs:**：  
server 可能會使用多個加密密鑰來簽署不同類型的資料，而不僅僅是 JWT
- 因此，JWT header 可能包含一個 `kid` (Key ID) 參數，用於幫助 server 識別驗證簽名時要使用的密鑰


驗證密鑰通常以 JWK set 的形式存儲。在這種情況下
- server 只需查找與 token 具有相同 `kid` 的 JWK 即可
- 不過，JWS 規範並沒有為這個 ID 定義具體的結構，它只是開發者任意選擇的一個字串
  - 例如，開發者可以使用 `kid` 參數指向資料庫中的某個條目，甚至是檔案名稱

如果該參數也存在 [directory traversal](https://portswigger.net/web-security/file-path-traversal) 漏洞
- 攻擊者就有可能迫使 server 使用其文件系統中的任意文件作為驗證密鑰

```json
{
  "kid": "../../path/to/file",
  "typ": "JWT",
  "alg": "HS256",
  "k": "asGsADas3421-dfh9DGN-AFDFDbasfd8-anfjkvc"
}
```

如果 server 也支持使用 [symmetric algorithm](https://portswigger.net/web-security/jwt/algorithm-confusion#symmetric-vs-asymmetric-algorithms) 簽名的 JWT
- 這種情況就尤其危險。攻擊者可能將 `kid` 參數指向一個可預測的靜態文件，然後使用與該文件內容相匹配的密文簽署 JWT

理論上，你可以用任何文件來做這件事，但最簡單的方法是使用 `/dev/null`
- 它存在於大多數 Linux 中。這是個空文件，讀取它將返回一個空字串
- 因此，使用空字串簽署 token 將產生有效的簽名

**注意:**  
如果你用的是 JWT Editor extension，注意它不允許使用空字串簽署 token
- 不過，由於 extension 中的一個錯誤，你可以通過使用 Base64 encode 的空字節來解決這個問題

如果 server 將驗證密鑰存儲在 db 中
- `kid` header 參數也可能成為 [SQL injection attacks](https://portswigger.net/web-security/sql-injection) 的潛在載體


**其他有趣的 JWT header 參數:**  
以下 header 也可能是攻擊者感興趣的:
- `cty` (Content Type): 有時用於聲明 JWT payload 中內容的  media type
  - 這通常會從 header 中省略，但底層解析 library 可能還是會支持它
  - 如果找到繞過簽名驗證的方法，可以嘗試 inject `cty` header，將 context type 改為 `text/xml` 或 `application/x-java-serialized-object`，這樣就有可能為 [XXE](https://portswigger.net/web-security/xxe) 和 [deserialization](https://portswigger.net/web-security/deserialization) 攻擊提供新的載體
- `x5c`  (X.509 Certificate Chain): 有時用於傳遞 X.509 公鑰證書或用於對 JWT 進行數字簽名的密鑰的證書鏈
  - 該 header 可用於注入自簽名證書，類似於上文討論的 jwk header 注入攻擊
  - 由於 X.509 格式及其擴展的複雜性，解析這些證書也會帶來漏洞
  - 要了解更多攻擊細節，參考 [CVE-2017-2800](https://talosintelligence.com/vulnerability_reports/TALOS-2017-0293) 和[CVE-2018-2633](https://mbechler.github.io/2018/01/20/Java-CVE-2018-2633/)

--------

## JWT algorithm confusion
即使 server 使用的是你無法暴力破解的強大 secrets
- 你仍然可以通過使用開發人員未曾預料到的算法簽署 token 來偽造有效的 JWT
- 這就是所謂的算法混淆攻擊
- [JWT algorithm confusion attacks](https://portswigger.net/web-security/jwt/algorithm-confusion)

--------

## 如何防止 JWT 攻擊
通過採取以下措施，可以保護自己的網站
- 使用最新的 library 來處理 JWT，並確保開發人員完全了解其工作原理以及任何安全影響
  - 現代化的 library 使你更難無意中以不安全的方式實現它們
  - 但由於相關規範固有的靈活性，這並非萬無一失
- 確保對收到的任何 JWT 執行強大的簽名驗證，並考慮到 edeg 情況，如使用意外算法簽名的 JWT
- 對 `jku` header 執行**嚴格**的允許 host 白名單
- 確保不會通過 `kid` header 導致 `path traversal` 或 `SQL injection`


**處理 JWT 的其他  best practice:**  
在 application 中使用 JWT 時，建議遵守以下 best practice，儘管這並非避免引入漏洞的嚴格必要條件:
- 始終為你發出的任何 token 設置 expiration date
- 盡可能避免在 URL 參數中發送 token
- 包含 `aud` (受眾) claim (或類似聲明)，以指定 token 的預期接收方。這樣可以防止在不同網站上使用 token
- 允許發行 server 撤銷 token (例如在 logout 時註銷)