# 2023-05-05：筆記 samcurry 的 How I could’ve taken over the production server of a Yahoo acquisition through command injection.md
## samcurry (samwcyo), June 4, 2017
### Ref: https://samcurry.net/how-i-couldve-taken-over-the-production-server-of-a-yahoo-acquisition-through-command-injection/


----------------

一篇 samcurry 的舊 blog

----------------

5 月 20日晚上，我在花了幾天時間看 Yahoo 的 messenger app 後，開始出現了小的頭痛和頸部疼痛
- 我無法掌握它的運作方式
- 所以我走到外面，決定尋找一個新的目標
- 讓我感興趣的是，一位叫做 `meals` 的人在參加 Yahoo 的錯誤賞金計劃後被 ban 了
  - [‘overstepping his boundaries’ while participating in Yahoo’s bug bounty program](https://seanmelia.files.wordpress.com/2016/02/yahoo-remote-code-execution-cms1.pdf)

走回屋裡，和我朋友 Thomas (dawgyg) 交談後，我們決定 it’d be a good idea to look at the program Sean was looking at right before he got banned.

## Step 1: Reconnaissance

在 [Sean's whiteepaper](https://seanmelia.files.wordpress.com/2016/02/yahoo-remote-code-execution-cms1.pdf) 中詳細說明收購的範圍：

```
*.mediagroupone.de
*.snacktv.de
*.vertical-network.de
*.vertical-n.de
*.fabalista.com
```

雖然列出了許多 domain
- 但 Sean 在他的報告中似乎主要針對 `SnackTV 的內容管理系統(content management system)`
- 我和 Thomas 決定調整方法，以 SnackTV 的 `www` 部分為目標
- 因為 Thomas 以前曾在該平台上花過時間，發現了一些 blind XSS 漏洞
- 這個平台似乎與大多數平台不同，因為它是
    1. 一家德國公司和
    2. 一個為影片製作者提供的開發者 (social) network，而不是標準的 Yahoo 使用者

![](https://samcurry.net/wp-content/uploads/2017/06/snacktv-1.png)  
(這是 SnackTV 的 search page。很明顯，這是 video network，但註冊必須經過人工審核，因此我們無法直接進入網站的上傳面板)  


當 Thomas 忙於將我們項目的掃描部分自動化時
- 我花了一些時間來感受這個 app
  - (了解一個東西應該如何操作，通常是了解它不應該如何操作的前提)

## Step 2: Scanning
我和 Thomas 在查看新的 scope 時都會做的事情是
- run background tasks specific to the application
  - 工具 [subbrute](https://github.com/TheRook/subbrute) 和 [dirsearch](https://github.com/maurosoria/dirsearch) 是我常用的腳本
  - 用於被動識別
      1. 直接漏洞和
      2. 潛在的漏洞內容
  - 了解如何使用這樣的工具將有助於尋找漏洞時

在運行這些工具**很長時間後**
- 我們收到了很多沒有什麼幫助的輸出
- 大多數資訊都是非常標準的，比如 `.htpasswd` 被封鎖在一個標準的 HTTP 403 錯誤後面
- `admin` 被封鎖在一個 redirect 到一個 login 頁面，等等

然而，最終，我們用一個**大得離譜的 wordlist** 與 `dirsearch`配對
- **確實得到了一個結果**


有問題的文件名是 `getImg.php`，藏在 `imged` 目錄後面
- `http://snacktv.de/imged/getImg.php`
- 經過一番觀察，我們發現這個文件可以通過 Google dork `site:snacktv.de filetype:php` 公開訪問
- **這一步非常重要**
  - 因為這個有漏洞的文件需要 GET 參數才能返回內容
  - 被發現的 **GET 參數需要花費數週的時間來破解或猜測，沒有人願意這樣做**
  - 因為它們通常與額外的輔助參數配對，這些參數必須存在才能執行查詢。


**關於所需 GET 參數的 example logic**  

`http://example.com/supersecretdevblog.php`:
```
500 Internal Server error… you must have parameters to see this content!
```

`http://example.com/supersecretdevblog.php?page=index&post=1`:  
```
200 OK

<p>how I hax0r’d the president and stole his DOB – <b>CONFIDENTIAL</b></p>
```


### 到此為止，我們所知道的：
- 文件 `getImg.php` 接受多個 HTTP GET 參數
- 如果通過 `imgurl` 參數提供圖片連結，就會自動下載一個修改過的圖片文件
- 在 Google search 中出現的參數充滿了 ImageMagick 的裁剪功能的味道

## Step 3: 訪問和升級 (Access and escalation)
The first thing we thought of when we saw this was [ImageTragick (CVE-2016-3714)](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-3714) and decided to send it a few test payloads.

Thomas and I spent hours crafting image files that contained payloads specific to the vulnerability. The way it worked was that a hot image file (an image file that contained a payload) would be processed by the command line tool `ImageMagick` then accidentally execute arbitrary commands because it lacked sanitation. None of our payloads worked and eventually we both got kind of tired of looking at it. Is it possible that they could be running the post-fix version on such a strange file?

當我們看到這一點時，我們首先想到的是
- [ImageTragick (CVE-2016-3714)](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-3714)
- 並決定向它發送些測試 payloads

Thomas 和我花了幾個小時製作 image files
- 其中包含針對該漏洞的 payloads
- 它的工作方式是，一個 hot image file (包含 payload 的圖)將被 cli `ImageMagick` 處理，然後意外地執行任意命令
  - 因為它缺乏淨化(sanitation, 消毒)功能
- 我們的 payload 都沒有效果，最後我們都有點看不下去了
  - 他們有可能在這樣一個奇怪的文件上運行修復後的版本嗎？

```
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd" ;>
<svg width="640px" height="480px" version="1.1" xmlns="http://www.w3.org/2000/svg" ; xmlns:xlink="http://www.w3.org/1999/xlink" ;>
  <image xlink:href="https://example.com/image.jpg&quot;|ls &quot;-la" x="0" y="0" height="640px" width="480px" />
</svg>
```

一個發送給 server 的 payload 的例子
- 通過 `imageurl` 參數獲取它。目標是讓它執行一個任意的命令
- 注意 `<image />` 的 `xlink:href` URL

除了 server 處理文件目標 URL 的方式，沒有任何不尋常的地方
- 我們向它發送隨機 text file，它仍然會返回上次調用的相同 data
- 根據漏洞披露的細節和我們讀到的關於 `ImageMagick` 的所有內容
- 這似乎不是漏洞，也不是 ImageMagick。我們從攻擊這個特定的文件中抽身出來，決定到其他地方去看看。


當時大概是凌晨 3 點半
- 我們發現了一些存儲的跨網站腳本漏洞(cross site scripting vulnerabilities)
- HTTP 401 response injection 缺陷，以及
- 一般的管理不善問題，但沒有什麼關鍵的

當你在做 bug bounty 並盯著(被)收購(的公司，業界也稱呼為 acquisition)的時候，這有點糟糕
- 因為大多數賞金都會被大幅縮減，因為影響太小了
- 在一些人眼裡，收到減少的賞金仍然是富有成效的，但在另一些人眼裡，與他們所能接觸到的私人項目清單相比，這可能只是一種時間的浪費
- 針對 acquisition 的唯一好處是，許多人完全將其排除在範圍之外


回到網址後，我有點厭煩，並對實施提出質疑
- 如果 Yahoo 不是把圖片作為一個整體來處理，而是以某種方式把 URL 注入到一個XML `image xlink:href` 中，就像 proof of concept content 一樣，會怎麼樣？
- 我應該嘗試什麼 payload 來驗證這一點？



在我的瀏覽器中，我附加了一個單一的雙引號字符(single double quote character)，看到一些有趣的輸出

```
REQUEST

GET /imged/getImg.php?imageurl=" HTTP/1.1
Host: snacktv.de
Connection: close
Upgrade-Insecure-Requests: 1

RESPONSE

By default, the image format is determined by its magic number.
To specify a particular image format, precede the filename with an image format name and a colon (i.e. ps:image)...
... or specify the image type as the filename suffix (i.e. image.ps). Specify file as - for standard input or output.

默認情況下，圖像格式由其 magic number。
要指定特定的圖像格式，請在文件名前加上圖像格式名稱和冒號（即 ps:image）... 或將圖像類型指定為文件名後綴（即 image.ps）。 將文件指定為 - 用於標準輸入或輸出。
```



我發出這個請求的原因是
- 在最初的 proof of concept XML文 件中，我們通過雙引號（或可能是單引號）來限制URL 實體(entity)
- 如果發送一個雙引號，我們可以迫使 server 擺脫這個範圍，並對命令的寫入位置有寫入權限（見上面的概念證明內容）

哼! 沒有辦法! 這是在運行 ImageMagick! 我是不是以某種方式破壞了執行？這是命令行內容嗎？我應該發送額外的內容:  
```
REQUEST

GET /imged/getImg.php?imageurl=";ls HTTP/1.1
Host: snacktv.de
Connection: close
Upgrade-Insecure-Requests: 1

RESPONSE

By default, the image format is determined by its magic number.
To specify a particular image format, precede the filename with an image format name and a colon (i.e. ps:image)...
... or specify the image type as the filename suffix (i.e. image.ps). Specify file as - for standard input or output.
[redacted]
[redacted]
index.php
getImage.php
[redacted]
[redacted]

```

我之所以發送上面列出的字符串是**為了逃避第一個命令的範圍**
- **在 Linux 環境下，你可以將分號(;)附加到你的初始命令上，然後開始編寫第二個命令**
- 這對攻擊者很有用，因為它允許在最初的預定義內容之外執行。


我很興奮。在我的狩獵生涯中，我第一次實現了 command injection
- 在這之前的測試過程中，我總是覺得發送引號和分號來實現 command injection 的做法很傻，但這完全改變了我的觀點。


在發現後不久，我通過 HackerOne 的 Yahoo 報告了這個問題
- 在 24 小時內收到了回复，並在另外 6 小時內修復了這個錯誤

## Overview
你以為會有第四步？沒有，沒有。我們是有道德的黑客，記得嗎？  

在攻擊了 SnackTV 之後，它讓我意識到錯誤的 implementations 可能是多麼嚴重
- 該 server 沒有通過通用的方法受到 ImageTragick 的攻擊
- 因為它沒有遵循標準的格式，而是非常類似和自定義的東西
- 如果你不能弄清楚你確定是易受攻擊的東西，有時重新設定你的想法並從根本上攻擊它是好的。它允許哪些字符，哪些會被拒絕？你的輸入可以有多長？在反應上有什麼不同？

在 Yahoo 的大型 app 中呆了好幾天之後，攻擊這樣一個多汁的平台，讓人感到非常振奮和新鮮

獎勵：3,000 美元