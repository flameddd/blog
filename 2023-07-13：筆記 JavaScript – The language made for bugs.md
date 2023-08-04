# 2023-07-13：筆記 JavaScript – The language made for bugs.md
## YesWeRHackers, 08/12/2022
### Ref: https://blog.yeswehack.com/yeswerhackers/javascript-language-made-for-bugs/

---

針對 Javascript 語言 security 相關的文章
- JavaScript 的奇怪有趣的行為，如何被用來攻擊


---

## Function calls
JavaScript 中調呼叫 function 有多種不同的方式  
- 下面的 payload 中，可以看到 alert() 如何被呼叫的一些範例。可以用這些 payload 來 poc XSS 的存在

```js
window.alert()
window['alert']()
alert()
alert``
onload=alert
\u0061lert()
globalThis.alert()
window?.alert()
eval?.('ale'+'rt('+')')
[1].find(alert)

好=alert,好()
var x=x||alert;x()
alert(1),ErrExit
[][0]??alert()
null??alert()
undefined??alert()
x??alert();var x;
```

------

## Newlines
JavaScript supports newlines that are not valid ASCII characters (`\u2028` and `\u2029`)
- 這兩個 chars 可以像其他 newline char 一樣使用
- (換句話說，Javascript 的換行可以用這兩個 ASCII chars，而這兩個 chars 實際上並不是「字」，而是 newline，是用 ASCII code 來表達的 newline。這算是在 ASCII 中的例外之一，所以原文才寫 "not valid ASCII characters")

要測試這個功能，可以這樣執行:

```js
console.log('alert//\u2028(1)')
console.log('alert//\u2029(1)')
```

作為 poc，您可以看到執行的 code 是在單一一行上顯示的，並且有註釋（`//`）
- JavaScript code 仍然可以正常工作，這是因為 `\u2028` 和 `\u2029` 被看作是新的一行

------

## Comments
Javascript 的 comment (不過下面的舉例感覺不出來有混淆攻擊 or xss 跡象，例子不夠具體)  

![](https://blog.yeswehack.com/wp-content/uploads/JSComments-2.png.webp)  

-----

## Hoisting
JavaScript 的 hoisting 行為  

這邊會 print 出 `Oh snap`，因為 function 定義被 hoisting 了
![](https://blog.yeswehack.com/wp-content/uploads/JSHoisting.png.webp)  


如果在 function 中直接定義變數 `x`，`x` 將在 function 內部和外部設置其值 `20`


![](https://blog.yeswehack.com/wp-content/uploads/HoistingFunc.png.webp)  


從攻擊者的角度來看
- 當攻擊者可以控制一個 function 中設置的 value 時，**他就可以感染其他使用該變數的 code**
- 開發人員宣告兩個同名變數，一個在 function 內部，一個在 function 外部
  - 開發人員沒有意識到它們是同一個變數，因此實際上他創建了一個變數，在呼叫 function 時被重複使用
  - 因為他忘記了用 `var`, `let` or `const` 來宣告



hoisting bug example
- 下面 `str` 先被設為 `location.search`
- `name` 被設置為 `ParamNames` function return value (`infected`)
- 當呼叫 `ParamNames` 時，它使用與 function 外相同的變數(`str`)
- 這使 `str` 的 global value 被更新，因為 `str` 沒有在 function 內部正確宣告


![](https://blog.yeswehack.com/wp-content/uploads/HoistEx.png.webp)  


------

## Error behaviour
只要不是 syntax error，JavaScript 就會一直運行直到發生錯誤。

假設發現一個 XSS 注入成功了，但你想阻止之後的 code 運行
- 可以觸發錯誤，作為「退出命令」
- 這將導致 JavaScript 執行 code (包括你的 payload )，並在遇到錯誤時停止

![](https://blog.yeswehack.com/wp-content/uploads/JSErrExit-1.png.webp)  


------

## DOM XSS
易受 DOM XSS 攻擊的 input 在攻擊者利用 XSS 時具有很大的優勢
- 在 DOM XSS 中，攻擊者有時只需要一個反斜杠 `\` 就可以利用漏洞
- `\` 可用於 Hex, Unicode 和 Octal encode 形式

在這種情況下，payload `<h1>Hello</h1>` 可以用下面所有不同方式表達：

```
<h1>Hello</h1>
\x3Ch1\x3EHello\x3C\x2Fh1\x3E
\u003Ch1\u003EHello\u003C\u002Fh1\u003E
\u{3C}h1\u{3E}Hello\u{3C}\u{2F}h1\u{3E}
\74\150\61\76Hello\74\57\150\61\76
```


---- 


## Unicode
可以將字 `[a-zA-Z]` 以 Unicode encode 的形式寫，而不是按照 JavaScript 的原意寫

這樣可以順利運行 code
```
var a; a == \u0061 // true
alert() == \u0061\u006c\u0065\u0072\u0074() // true
```

-----

### JSFuck
JavaScript 可以寫成只有六個字符（`[]()+!`）
- 這種方法被稱為 [JsFuck](https://jsfuck.com/)
- 可用於繞過受限制的 filter

```
(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]] === "ert" // true
```

當 JsFuck 與常規的 JavaScript 混合時，它就會發揮作用  

下面是一個呼叫 object `top` 的 alert() function 的 payload:
```
top['al'+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]+(!![]+[])[+[]]](1)
```

該有 payload 使用 JsFuck 將字符 `"ert"`（包含引號）放入到 `top` 中，產生以下 payload
- `top['al'+"ert"](1)`

----------

其他一些透過 Javascript 來達成的攻擊
- XSS
- Session hijack
- Iframe exploit
  - 當在 cookie 中使用 session 並且安全性很強時，這時利用 iframe 是種很好的技術
  - 如果 iframe 安全策略較弱，可以使用 XSS 嵌入 iframe 並劫持嵌入 iframe 中的資料
- Keylogger
  - JavaScript 可以做為 Keylogger，劫持受害者的鍵盤輸入
  - Keylogger 可與其他利用技術相結合，如 iframe。結合這兩種技術，可以跟踪受害者並劫持幾乎任何 endpoint 的所有按鍵
    - (不太懂，有機會可以再找看看 real case)

另外提到一個攻擊工具
- [Metasploit](https://www.metasploit.com/) offers JavaScript exploitation modules, e.g. the JavaScript keylogger. This module is good to include in your report as a Proof of Concept when an XSS vulnerability is detected in an application.  
