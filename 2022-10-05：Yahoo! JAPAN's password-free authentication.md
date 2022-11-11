# 2022-10-05：`Yahoo! JAPAN`'s password-free authentication
## May 10, 2022 Yuya Ito, Eiji Kitamura, Alexandra White
### https://web.dev/yahoo-japan-identity-case-study/


`Yahoo! JAPAN`
- 日本最大的媒體公司之一
- 提供 search、新聞、電子商務和電子郵件等服務
- 每月有超過 5000 萬 User 登入服務

多年來
- 有許多針對 User 的攻擊導致 account access 權限失去的問題
- 這些問題大多與使用密碼進行認證有關
- 隨著最近 authentication 技術的進步，`Yahoo! JAPAN` 決定從基於密碼的認證轉向 passwordless 認證。

## 為什麼採用 passwordless ?
為什麼無密碼？
由於 `Yahoo! JAPAN` 提供電子商務和其他與金錢有關的服務
- 在 unauthorized access 或 account list 的情況下，有可能給 User 帶來重大損失
- 與密碼有關的最常見的攻擊是 `password list attacks` 和網路釣魚詐騙
  - `password list attacks` 常見且有效的原因是許多人習慣於在多個 App 和 Web 上使用相同的密碼

以下是 `Yahoo! JAPAN` 進行的調查的結果:
- `50%` 的人在六個或更多的網站上使用相同的 ID 和密碼
- `60%` 的人在多個網站上使用相同的密碼
- `70%` 的人使用密碼作為登入的主要方式

User 經常忘記密碼
- 這佔了大多數的密碼查詢
- 也有 User 在忘記密碼的同時，還忘記了自己的 ID
- 高峰時期，這些查詢佔所有帳號相關查詢的三分之一以上

通過取 passwordless，`Yahoo! JAPAN`
- 不僅是為了提高安全性，而且是為了提高可用性，而不給 User 帶來任何額外的負擔
- 從安全的角度來看，從 User 認證過程中消除密碼可以減少 `list-based attacks` 所帶來的損害
- 從可用性的角度來看，不依賴記憶密碼的認證方法可以防止 User 因為忘記密碼而無法登入的情況

## `Yahoo! JAPAN` 的 passwordless 行動
`Yahoo! JAPAN` 正在採取一些措施來 promote passwordless authentication，大致可分為三類:
1. 提供替代 password 的認證方式
2. 停用 password。
3. Passwordless 帳號註冊

前兩項針對的是現有 User，passwordless 註冊是針對新 User

### 1. 提供一個替代 password 的認證方式
`Yahoo! JAPAN` 提供以下替代密碼的方式
1. `SMS 認證` ([SMS OTP form best practices](https://web.dev/i18n/zh/sms-otp-form/))
2. [FIDO with WebAuthn](https://developers.google.com/identity/fido)

另外，還提供 email 認證、password 與 SMS OTP（一次性密碼）相結合、password 與 email OTP相結合等認證方式
- (`Yahoo! JAPAN` 將其服務限制在日本境內的電話運營商，並禁止VoIP SMS)

#### 關於 SMS 認證
SMS 認證是允許 User 通過 SMS 接收六位數認證碼的一個註冊系統
- 一旦 User 收到 SMS，就可以在 App 或 Web 上輸入認證碼

![](https://web-dev.imgix.net/image/VbsHyyQopiec0718rMq2kTE1hke2/h08m9uzVJg9uNzM3LE5k.jpg?auto=format&w=600)  


長期以來，`Apple` 允許 iOS 讀 SMS messages，並從 `text body` 中提示 authentication codes
- 最近，通過在 `<input />` 的 `autocomplete` 屬性中指定 `"one-time-code"`，就可以使用建議
- `Android`、`Windows` 和 `Mac` 上的 `Chrome` 可以使用 [WebOTP API](https://developer.mozilla.org/en-US/docs/Web/API/WebOTP_API) 提供同樣的體驗
- (更多資訊可以參考上面的連結: [SMS OTP form best practices](https://web.dev/i18n/zh/sms-otp-form/))


For example:

```html
<form>
  <input type="text" id="code" autocomplete="one-time-code"/>
  <button type="submit">sign in</button>
</form>
```

```js
if ('OTPCredential' in window) {
  const input = document.getElementById('code');
  if (!input) return;
  const ac = new AbortController();
  const form = input.closest('form');
  if (form) {
    form.addEventListener('submit', e => {
      ac.abort();
    });
  }
  navigator.credentials.get({
    otp: { transport:['sms'] },
    signal: ac.signal
  }).then(otp => {
    input.value = otp.code;
  }).catch(err => {
    console.log(err);
  });
}
```

這兩種方法都是為了防止網路釣魚
- 在 SMS body  包括 `domain`，並只為指定的 `domain` 提供建議

![](https://web-dev.imgix.net/image/VbsHyyQopiec0718rMq2kTE1hke2/Szaf3C0hfjLNkTWAVf9B.png?auto=format&w=400)  

#### 關於 FIDO with WebAuthn
`FIDO with WebAuthn` 是使用硬體 authenticator 來產生 `public key cipher pair` 並證明其擁有權
- 當 smartphone 被用作 authenticator 時，它可以與生物識別認證（如指紋傳感器或臉部識別）相結合，進行 `one-step two-factor authentication`
- 在這情況，只有 `signature` 和生物識別認證的成功指示(`success indication`)被發送到 server，不存在生物識別 data 被盜的風險

下圖顯示 FIDO 的 server-client configuration
- client 的 authenticator 通過生物識別技術對 User 進行認證，並使用 public key cryptography 技術對結果進行簽名
- 用於建立 `signature` 的 `private key` 被安全地存儲在 [TEE（`Trusted Execution Environment`）](https://en.wikipedia.org/wiki/Trusted_execution_environment)或類似的地方
- 使用 FIDO 的服務提供者被稱為 RP（relying party）

![](https://web-dev.imgix.net/image/VbsHyyQopiec0718rMq2kTE1hke2/PkFYWnOZABjPu7Zc7rXN.png?auto=format&w=1600)

一旦 User 進行了認證（通常使用生物識別掃描或 PIN）
- authenticator 就會使用 `private key` 向 browser 發送一個簽名的驗證信號。然後，browser 與 RP 網站共享該信號
- RP 網站然後將簽名的驗證信號發送到 RP 的 server，server 根據 public key 驗證簽名以完成認證
  - (更多資訊: [authentication guidelines from the FIDO Alliance](https://fidoalliance.org/fido-authentication/))

`Yahoo! JAPAN`
- 在 Android（mobile app 和 web）、iOS（mobile app 和 web）、Windows（Edge、Chrome、Firefox）和m acOS（Safari、Chrome）上支持 FIDO
- 作為消費者服務，FIDO 幾乎可以在任何設備上使用，這使它成為推廣 passwordless authentication 的好選擇

FIDO 的 OS 支持
- Android	Apps, Browser (Chrome)
- iOS Apps（iOS14 或更高），Browser（Safari 14 或更高）
- Windows Browser（Edge、Chrome、Firefox）
- Mac（Big Sur 或更高）Browser（Safari、Chrome）

![](https://web-dev.imgix.net/image/VbsHyyQopiec0718rMq2kTE1hke2/iq0dODcbp3WcPUGTwHkl.png?auto=format&w=400)  
(Sample `Yahoo! JAPAN` prompt to authenticate with FIDO.)

`Yahoo! JAPAN` 建議
- 如果 User 還沒有通過其他方式進行認證，就用 `WebAuthn` 註冊 `FIDO`
- 當 User 需要用同一台設備登入時，可以使用生物識別傳感器快速進行認證

User **必須**在他們用來登入 `Yahoo! JAPAN` 的所有設備上設置 FIDO 認證  

為了促進 passwordless authentication，並考慮到正在向 passwordless 過渡的 User
- 我們提供多種認證方式。這意味著不同的 User 可以有不同的認證方式設置
- 他們可以使用的認證方式可能因 browser 而異
- 我們相信，如果 User 每次都使用相同的認證方法來登入，會有更好的體驗

為了滿足這些要求
- 我們有必要跟踪以前的認證方法，並通過存儲 cookies 等形式將這些資訊與 client 聯繫起來
- 然後可以分析不同的 browser 和 App 是如何進行認證的
- 根據 User 的 settings、以前使用的認證方法以及所需的最低認證級別，要求 User 提供適當的認證

### 2. 停用 Password

`Yahoo! JAPAN`
- 要求 User 設置一個替代的認證方法，然後禁用密碼，使其無法被使用
- 除了設置替代認證外，停用密碼認證（因此使 User 無法僅憑密碼登入）有助於保護 User 免受 `list-based attacks`

我們已經採取了以下措施來鼓勵 User 禁用密碼
- 在 User 重新設置密碼時，推廣替代的認證方法
- 鼓勵 User 設置易於使用的認證方法（如 FIDO），在需要頻繁認證的情況下禁用密碼
- 敦促 User 在使用電子商務支付等高風險服務前禁用其密碼

如果 User 忘記 password，可以執行 account recovery
- 以前這涉及到 `password reset`
- 現在 User 可以選擇設置一個不同的認證方法，我們鼓勵他們這樣做

### 3. Passwordless 帳號註冊
新 User 可以建立 password-free 的 `Yahoo! JAPAN` 帳號
- User 首先需要用 SMS 認證來註冊。一旦登入了，我們鼓勵 User 設置 FIDO 認證
- 由於 FIDO 是按設備設置的，如果設備無法使用，recover account 很困難
  - 因此要求 User 保持他們的手機號碼註冊，即使他們已經設置了額外的認證

### passwordless authentication 的關鍵挑戰

密碼依賴於人的記憶，並且是與設備無關的
- 另方面，到目前為止所介紹的認證方法是依賴於設備的，這帶來了幾個挑戰

當使用多種設備時，有一些與可用性有關的問題:
- 當使用 SMS 認證從 PC 上登入時， User 必須檢查手機是否有 SMS 資訊。這很不方便
  - 因為它要求 User 的手機隨時都可以使用，而且很容易進入
- 使用 FIDO，特別是使用平台認證器，擁有多台設備的 User 將無法在未註冊的設備上進行認證
  - 他們必須為打算使用的每台設備完成註冊
  
FIDO 認證與特定的設備相聯繫，這就要求這些設備一直在 User 手中並處於 active 狀態
- 如果取消了服務合約，就不能再向註冊的手機號碼發送 SMS
- FIDO 在特定的設備上存儲 private keys，如果該設備遺失，這些 private keys 就無法使用

最重要的解決方案是
- 鼓勵 User 設置多種認證方法
- 這在設備丟失時提供了替代的帳號訪問
- 由於 FIDO keys 與設備有關，在多台設備上註冊 FIDO private keys 也是個好的做法

另外，User 還可以使用 WebOTP API 將 SMS verification codes 從 Android phone 傳到 PC 的 Chrome 上
- ([SMS OTP form best practices](https://web.dev/i18n/zh/sms-otp-form/))


`Apple` 最近宣布了 [passkeys](https://developer.apple.com/documentation/authenticationservices/public-private_key_authentication/supporting_passkeys) 功能
- Apple 利用 `iCloud Keychain` 在用同一 Apple ID 登入的設備之間共享 private key（存在設備上）
  - 這樣就不需要為每台設備進行註冊
- FIDO Alliance 意識到帳號恢復問題的重要性，發表了一份[白皮書](https://fidoalliance.org/white-paper-multiple-authenticators-for-reducing-account-recovery-needs-for-fido-enabled-consumer-accounts/)

隨著 passwordless authentication 普及，解決這些問題將變得更加重要

### 推廣 passwordless authentication
`Yahoo! JAPAN`
- 從 2015 年以來一直在致力於這些 passwordless 舉措
- 從 2015 年 5 月獲得 FIDO server 認證開始的
- 隨後又推出了 SMS 認證、密碼停用功能，以及每個設備對 FIDO 的支持
- 如今，超過 3000 萬月度活躍 User 已經禁用密碼，並使用非密碼認證方法
- `Yahoo! JAPAN` 對 FIDO 的支持從 Android 上的 Chrome 開始，已經有超過 1000 萬 User 設置 FIDO

結果為
- 涉及遺忘登入 ID 或密碼的諮詢比例與此類諮詢數量最高的時期相比下降了 25%
- 我們還能夠確認，由於 passwordless 帳號數量的增加，未經授權的訪問也有所下降
- 由於 FIDO 的設置非常簡單，它的轉換率特別高
  - FIDO 的 CVR 比 SMS 認證要高
- 遺忘憑證的請求減少了 `25%`
- `74%` 的 User 成功使用 FIDO 認證
- `65%` 的 User 使用 SMS 驗證成功

FIDO 的成功率比 SMS 認證高，平均和中位認證時間也更快
- 至於密碼，有些 groups 的認證時間很短，我們懷疑這是由於 browser 的 `autocomplete="current-password"` 造成的

![](https://web-dev.imgix.net/image/VbsHyyQopiec0718rMq2kTE1hke2/KhkGTZgYgmUM9pJGrg5h.png?auto=format&w=800)  
(平均，FIDO 的認證時間為 8 秒，密碼需要 21 秒，SMS 驗證 27 秒)  

提供 passwordless accounts 的最大困難不是增加認證方法
- 而是普及 authenticators 的使用
- 如果使用 passwordless service 的 UX 不友好，過渡就不容易
- 我們認為，要實現提高安全性，必須首先提高可用性，這將需要對每項服務進行獨特的創新

### 結論
就安全性而言，Password authentication  是有風險的
- 而且在可用性方面也構成了挑戰
- 現在支持 non-password authentication 的技術，如 `WebOTP API` 和 `FIDO`，已經越來越廣泛，是時候開始努力實現 passwordless authentication

在 `Yahoo! JAPAN`
- 採取這種方法對可用性和安全性都產生了一定的影響
- 然而，許多 User 仍然在使用密碼，我們將繼續鼓勵更多的 User 轉向無密碼認證方式
