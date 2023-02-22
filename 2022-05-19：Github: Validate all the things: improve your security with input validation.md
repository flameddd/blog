# Github: Validate all the things: improve your security with input validation!
## Github, Jaroslav Lobacevski March 21, 2022
### https://github.blog/2022-03-21-validate-all-things-input-validation/


(很簡略的一篇，簡單分類、歸納 input validation 的手段、時機。沒有太多技術內容)  

------------------------------------------

Input validation 可以實作為
- 採用 `allow list` or `deny list`
- 用驗證的方法 (Validation) or 消毒的方法 (sanitization)
- 在 server-side or client-side 執行

### `allow list` 與 `deny list`

每次都問自己：
- 是否需要支持**無限長度**的輸入字段和**所有可能的字符**？

理想情況下
- 限制一組有限的最大長度的僅允許字符

下面 Java 範例，電話號碼僅限於加號和 9 到 12 位數字：
```
@Pattern(regexp="\\+\\d{9,12}")
String phoneNumber;
```


以下 ASP.NET Core 範例，設置預期值範圍和必填字段：
```aspnet
public class Movie
{
    [Required]
    [StringLength(100)]
    public string Title { get; set; }

    [Required]
    [StringLength(1000)]
    public string Description { get; set; }

    [Range(0, 999.99)]
    public decimal Price { get; set; }
}
```


但
- 想出一組有限的允許字符並不容易
- 例如，[[Wiki 中所述](https://en.wikipedia.org/wiki/Email_address)，email address 可以非常靈活
  - 根據 IETF 標準和後續 RFC（5322、6854），`" "@example.org` 地址是有效的，但可用於注入惡意 payload

雖然 `disallowed characters list (deny list)` 是比 `allow list` 更弱的防禦措施
- 但仍然可以防禦許多不同的攻擊


### 驗證與消毒
sanitization (消毒)
- 是最不有效的技術，但總比沒有好

採用 sanitization 時，有些解決方案不會直接拒絕這類 input，而是修補 input
- 例如刪除某些字符或替換它們
- 攻擊者通常會找到繞過它的方法

假設為了防止路徑遍歷攻擊 (path traversal attacks)
- 對 filename 進行了清理，並從 input 中刪除了所有出現的 `../`
  - （這不是最推薦防禦 path traversal attacks 的方法）
- 攻擊者可以通過輸入繞過保護，如 `abc/....//xyz.`
  - 清理檢查後，輸入將變為 `abc/../xyz.`

### Server-side vs. client-side
輸入驗證的主要工作是改善用戶體驗
- client-side 驗證通常用於改善 UX

但是，攻擊者（或是與預期使用方式不同的 User）始終可以繞過 client-side
- 因此，Server-side 驗證是它扮演安全角色的地方
- 換句話說，“永遠不要相信 client-side”

除非你有專門的 unit test 來針對 validation routines
- （其實，專門的 unit test 是推薦的方案）
- 否則它可能永遠不會在測試環境中進行驗證出
- 但是，這些輸入驗證應該保留。它們的目的不是捕獲常規錯誤（不是主要目的），而是檢測和防止對安全漏洞的利用


### 自動化
使用任何輕量級靜態分析來強制執行驗證規則並尋找 anti-patterns
- semantic CodeQL 查詢可以根據使用模式進行調整，並放到 CI/CD 管道
  - （如 [GitHub code scanning](https://docs.github.com/en/code-security/code-scanning/automatically-scanning-your-code-for-vulnerabilities-and-errors/about-code-scanning)）

The example below flags any potentially untrusted Spring Controller input parameter that doesn’t have validation annotation.  
```
import semmle.code.java.dataflow.FlowSources

class SpringServletInputParameterSource extends RemoteFlowSource {
    SpringServletInputParameterSource() {
      this.asParameter() = any(SpringRequestMappingParameter srmp | 
                                                    srmp.isTaintedInput())
    }

    override string getSourceType() {
        result = "Spring servlet input parameter" }
  }

from SpringServletInputParameterSource c
where not c.asParameter()
                   .getAnAnnotation()
                   .getType()
                   .hasQualifiedName("javax.validation", "Valid")
select c
```


### Wrap up
限縮 attacker 的 input data 是很有效的技術
- 可以減少 app 的 [attack surface](https://en.wikipedia.org/wiki/Attack_surface)
- 但不應用作防御 injection attacks 的主要方法