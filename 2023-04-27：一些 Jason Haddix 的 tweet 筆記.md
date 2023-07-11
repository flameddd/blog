# 2023-04-27：一些 Jason Haddix 的 tweet 筆記

## Ref: https://youtu.be/IN-TElTkVmU?t=212

---------------------------

Jason Haddix 是為 web hacker，樂於分享。\
看了幾個 他的 tweet thread (and 他推薦的 thread)，這邊想簡單記錄一下。

---------------------------

Ref Link: https://twitter.com/Jhaddix/status/1649502078610427921 

Jason Haddix: My thoughts onrecon today:
- Sometimes 10% is the winning difference in red teaming and bug bounty.
  - permutations, bruteforce, cloud asset recon, IPv6, +++ Passive is fast. 
- BUT !! active is best imo.

有時，就是多最後 10% recon 的成果讓我們能成功

---------------------------

Ref: https://twitter.com/Jhaddix/status/1648355022697033730

Jason Haddix: This is an absolutely dope mindmap for attacking AD. 
- https://orange-cyberdefense.github.io/ocd-mindmaps/img/pentest_ad_dark_2023_02.svg
- Source: https://github.com/Orange-Cyberdefense/ocd-mindmaps

---------------------------

Ref: https://twitter.com/Jhaddix/status/1524817015961047053

Jason Haddix: 向學生推薦的練習目標的主題。它包括包括 online/offline 資源，以指導你的工作  

首先 [@hackthebox_eu](https://twitter.com/hackthebox_eu)  
- HTB 是個史詩般的資源，有許多免費的 "盒子 "可以 hack
- 付費級別也提供了大量的退役盒子。一些有演練的，現在甚至還有一個 in-browser hacking box，如果你不想通過 VPN 進入目標網絡
  - HTB 也有 certificates 和 learning tracks。netpen 和 web 挑戰都有。


[@PentesterLab](https://twitter.com/PentesterLab)
- 多年來，PentesterLab 一直是我的最愛之一。
- Louis 不遺餘力地支持 offensive security community，他的實驗室是一流的
- 當一個新的命名的漏洞出現時，PentesterLab 通常是第一個有一個練習箱的平台。
- 大部分是 web 挑戰


[@WebSecAcademy](https://twitter.com/WebSecAcademy)
- 由 @PortSwigger（Burp_Suite 的公司）創建的，是一個非常全面的實驗室環境
- For web testing
- 超過 200 個練習
- 是一種很好的方式學習使用攔截代理 (interception proxy)
- It is 100% free !


[@VulnHub](https://twitter.com/VulnHub)
- 最好的 crackmes 和 practice boxes 目錄之一
- 你可以自己測試所有的題目
- 大部分是你可以下載並嘗試闖入的 VM
- 100% free
- 超過 200 個 network and web 挑戰。

[@owasp](https://twitter.com/owasp) VWAD
- `OWASP Vulnerable Web Applications Directory` 是個受歡迎的有意破壞的 app 的目錄
  - https://owasp.org/www-project-vulnerable-web-applications-directory/
- 可供練習。這些項目大部分是你必須自己 host
- VWAD 的範圍有些 edge 的東西是很有趣的，比如 cloud apps, mobile
- 100% free

[@RealTryHackMe](https://twitter.com/RealTryHackMe)
- TryHackMe 是對初學者非常友好的平台，有各種挑戰，包括 web, and network
- 有學習路徑。通常被認為是最好開始的平台


Github
- 許多新的 hack challenges 在 conferences 上推出後，就會出現在 GitHub 上或從 CTF 中開源
- 為了保持你的技能，可以搜尋
  - https://github.com/search?o=desc&q=damn+vulnerable&s=updated&type=Repositories


[@CTFtime](https://twitter.com/CTFtime)
- CTFTime 跟踪世界上所有 CTF 的目錄
- 你可以使用這個目錄來查看已經完成的 CTF
- 如果挑戰還在進行、自己嘗試一下。如果需要幫助，CTF 頁面也會收集寫的內容
- 如果挑戰不是仍然在進行，看 CTF 的網站，看看他們是否開源他們的挑戰!
- "ctf" "hacking practice" 搜尋關鍵字，也能找到 ctf 的相關資源

[@zseano](https://twitter.com/zseano?s=20) 的 `Barker ++`
- Zseano 提供了一個免費的方法和一些練習目標來鍛煉
- https://bugbountyhunter.com
- 在專業版中，Barker 提供了一個經過深思熟慮的練習目標
  - 它還教給其他相關內容，如報告和鍊式漏洞
  - 專業版是終身訂閱
  
[@AmanHardikar](https://twitter.com/AmanHardikar) 的 Targets mindmaps
- 雖然有點過時了，但這 mindmaps 有些按技術類型劃分的 self-hosted 或 free-hosted 的挑戰按技術類型進行了細分
- 特別注意 cloud and mobile 部分 
- https://amanhardikar.com/mindmaps/Practice.html

[@Hacker0x01](https://twitter.com/Hacker0x01) CTF
- 免費 26 項挑戰，與 web and mobile security 相關
- 完成後還可以得到在 HackerOne 平台上得一些邀請

自我託管VS虛擬機VS平台：
- 我喜歡提醒人們關於自我託管的免費資源（與平台）的一件事。
- 你可以通過自己安裝挑戰，然後黑掉它們。
- 你會學到、你可以學到虛擬機技能 webserver, nix, networking and framework skills.
- 所以，不要略過 self-hosted 的選項

---------------------------

Ref: https://twitter.com/Jhaddix/status/1612480592909852672

Jason Haddix: 引用的推文是 2022 年的 high profile breaches thread (https://twitter.com/tayvano_/status/1611131230074073088)。我們能學到什麼來指導 2023 的安全計劃？
- `Two-factor auth`，但更好的是，FIDO 必須成為你 security 的基石。
- 如果你有幸擁有優秀的 `IAM`，那麼這裡的最低要求應該是部署給技術人員、開發人員和管理員
- Repo 和 cloud security，尤其是與 `IAM` 和**賬戶的離職**相關的安全，是非常重要的
  - **真的很重要。不要吝嗇在 logging in cloud，你會為之付出代價的。**
- 使用者意識訓練是需要的，即使是 IT help desk 員工
- 雙重驗證流程必須建立起來，包括`刪除2FA`
- Threat 情報是很重要的，無論你用什麼方式，內部還是外包
- 應該在暗網市場上搜尋你員工被盜的 cookies 和 credentials
- 供應鏈安全風險是非常真實存在的
- 審計你的供應商和 SaaS。要求他們審計自己。盡可能限制依賴和訪問
- 與那些真正有租戶分離 data 的供應商合作
  - 在你的安全計劃中，以同樣的標準要求他們
- 你必須有個面向互聯網的 assets 的視圖和一個內部註冊表來跟踪上述 assets 的所有權
- 假設出現漏洞，必須做一個完整的內部回購憑證輪換 (complete internal repo credential
rotation)
- 即使是最好的安全策略也會失敗，這個過程需要被定義和實踐
- 在可能的情況下，`Strong authentication`, `double verification` 和 `segmentation` 需要被應用於關鍵任務的內部服務，特別是大多數 IT 和 security management web portals。可能還有 financial apps。
- App 安全測試作為你的安全計劃的核心部分仍然是是必要的。特別是對於 internet accessible apps and mobile
- Common stack 的公司有時可以將此承包出去。More modern stack 的公司通常需要在內部建立這個團隊
- 對於我們看到的每一個漏洞，每年有 3-7 個可能通過 bug bounty 防範，漏洞賞金計劃可以節省數百萬美元。


 
---------------------------


Ref: https://twitter.com/Jhaddix/status/1513627638144790528  
Jason Haddix: 為 hackers/defenders/組織 提供的提示  
- 對組織來說，一個常見的 credentials leaked 會在 Github
  - 有時來自組織在 GitHub 上的 repo
- 但，最常見的是，開發人員不小心將公司的 code 或機密 clone 到他們的個人和公共 repo

常見的錯誤：
- 洩露 API keys
- 服務 username 和 pass（SSH、FTP、LDAP）
- DB 連接 username 和 pass
- 要更深入地了解賞金獵人是如何發現這些問題的？
  - 查看: https://youtube.com/watch?v=l0YsEk_59fQ by @Th3G3nt3lman
  - (聲音品質不是非常好，但也沒辦法)

全面掃描你的 repo 中的秘密，可以使用以下工具
- TruffleHog 和 Git-secrets
- https://github.com/trufflesecurity/trufflehog
- https://github.com/awslabs/git-secrets


但如果員工洩露了 secrets，但他已經不在公司了呢？
- 可以聯絡 GitHub
  - https://support.github.com/contact/private-information
- 相關 Github 文件: https://docs.github.com/en/site-policy/content-removal-policies/submitting-content-removal-requests

一個全面的 secrets management 計劃有三個租戶(tenants):
- 預防（pre/post-commit hooks，以及 a responsible alternative to hardcoding
  secrets）
- 檢測（pipeline + repo scanning）
- Response（由 `Tiger-team` 定期發現異常）。

這是一個值得討論的大話題，每個組織至少都會發生一次。

 
---------------------------


Ref: https://twitter.com/Jhaddix/status/1542192205166612480  
Jason Haddix: bug bounty automation-kings 的秘密  
- 找到一天（或一個月）內還沒有進入掃描器的 web exploits，可以讓你賺大錢。
- bug bounty 的一個競爭優勢是能夠寫自己的漏洞檢查。
- 有數以百計的 COTS 和 OSS 軟體的漏洞從來沒有在掃描器中，因為各種原因而最終被發現
  - 也許是因為該軟體不是像微軟或JIRA那樣的大公司
  - 也許是廠商和報告者沒有對這個漏洞進行任何宣傳
- 有了 @pdnuclei 和 Jaeles 這樣的工具，你可以自己的檢查來獲利
- 你可以使用 YAML 和 Nuclei 模板
  - https://nuclei.projectdiscovery.io/templating-guide/
  - @jtsec 提供了一個指南，他創建了自定義的 CORS 檢查
  - https://youtube.com/watch?v=bHXkQjtBOLo
- 對於 @j3ssiejjj 的 Jaeles，可以按照以下來做：
  - https://jaeles-project.github.io/signatures/examples/
  - 如果對 Jaeles 不熟，這有個 demo
    - https://youtube.com/watch?v=atdnd_WQAlY
- 我用 nuclei 進行大量的 misconfiguration 和 CVE 檢查
- 我用 Jaeles 更多用於自定義的 web fuzzing

那麼現在你知道如何進行檢查了，但你怎麼知道要做什麼？
- 令人驚訝的是，Twitter 為這方面最好的來源之一
- 我和其他一些人是如何做到這一點的？
- 首先，你需要 Twitter account
- 然後到 https://tweetdeck.twitter.com
- 使用 Twitter 的 live search search 來找 CVE 和 exploits，並將其製作成 templates

我在 TweetDeck 裡有一欄，代表了對每個漏洞類型。例如，我的其中一欄是 live search:
- "local file include" OR "path traversal" OR "directory traversal" OR "arbitrary
file read"

另個是：
- "Broken Authentication" OR "Authentication Bypass" OR "account takeover" OR
"Sensitive Data Exposure"


![](https://pbs.twimg.com/media/FWb3GE0UIAEu5lD?format=jpg&name=large)  
(我在 TweetDeck 中運行了 30 多個 live search 實時搜索，來 update 我對新 CVE 的看法、漏洞類別和寫法，然後我可以將其移植到漏洞檢查中)  


另個情報來源和是 Hackerone 的 hacktivity 頁面
- https://hackerone.com/hacktivity
- https://bugcrowd.com/crowdstream
- 閱讀這些文章，如果其中有一個看起來是很好的檢查或新奇的模糊字符串，就加到你的武器庫中

這方面的 UBER level 是什麼？
- 我認識兩個 hunters，他們支付訂閱 Threat 情報的費用，作為前期費用。
- 這些資訊通常有關於 CVE endpoints 的內部資訊，而這些資訊尚未公開。
- 威脅情報的內部資訊，包括 PoC 的模糊字符串。他們利用這些資訊製作模板並獲利

有了持續的、自動化的掃描程序，你就可以建立一個巨大的漏洞掃描機器!
- 你應該研究一下 `@pry0cc` 的 Axiom
  - https://youtube.com/watch?v=t-FCvQK2Y88

 
---------------------------

Ref: https://twitter.com/Jhaddix/status/1618322453226459137
Jason Haddix: My ultimate workflow for simple and easy JavaScript Analysis  
(這 thread 針對分析 JS 來做 recon)

應用安全測試和紅色團隊中的綜合 JavaScript 分析:
- 通常，你可以 JS 中找到更的隱藏 endpoints、參數和 domain
- 通常情況下，動態工具無法解析、訪問和理解複雜的 JS。因為 JS 中有各種表達方式


![](https://pbs.twimg.com/media/FnVnQp9akAAwZq_?format=png&name=small)  
(JS 中各種表達 URL path 的方法，這才只是其中幾種)

在 cli 中，許多獵手使用 @gerben_javado 的 LinkFinder 來查隱藏的 endpoints
- https://github.com/GerbenJavado/LinkFinder
  - 有些人將其寫成 chrome ext
  - https://github.com/GerbenJavado/LinkFinder/tree/chrome_extension

還有更簡單的方法，`GAP` by `@xnl_h4ck3r` 是 Burp extension，利用了同樣的技術，但有些額外的優勢
- https://github.com/xnl-h4ck3r/GAP-Burp-Extension
- GAP增加了：
  - 右鍵單擊並在 Burp 中的整個範圍內運行
  - 在所有頁面上運行 regex 
  - 而不僅僅是 JS 文件（capturing inline JS）
  - 解析 links/endpoints 和參數進行分析

如果遇到嚴重包裝或混淆的 JS，試試
- https://deobfuscate.io 
- 解壓縮 arrays、簡化表達式、Beatifies the code 等等
- 你應該在 JS 中尋找逐字記錄、hard-coded secrets 的秘密

JS Miner（Burp extension）增加了被動掃描檢查，提醒你注意這些
- https://portswigger.net/bappstore/0ab7a94d8e11449daaf0fb387431225b
  - https://github.com/minamo7sen/burp-JS-Miner
- 不僅僅是 regex，它使用一個(Shannon)熵函數來尋找可能有趣的東西


If you are looking to include discovering
如果你想在目標（頂層和底層）的 JS 文件中發現 related domains，我使用這個簡單但貧乏的方法:
- https://twitter.com/jhaddix/status/972926512595746816?lang=en
  - The lost art of LINKED target discovery w/ Burp Suite: 
      1.  Turn off passive scanning 
      2. Set forms auto to submit 
      3. Set scope to advanced control and use string of target name (not a normal FQDN) 
      4. Walk+browse, then spider all hosts recursively! 
      5. Profit (more targets)!
      6. (一小段操作影片: https://twitter.com/Jhaddix/status/972926512595746816 )
- 其他 cli spiders 包括 Hakrawler 和 GoSpider:
  - https://github.com/hakluke/hakrawler
  - https://github.com/jaeles-project/gospider


@LewisArdern 不久前製作了 `Metasec.js`，它用了以下的靜態 source 安全分析引擎:
- npm-audit, yarn-audit, semgrep 用於 secrete 和JS 安全問題
- https://github.com/LewisArdern/metasecjs
- https://youtube.com/watch?v=djCNhYMo3eE


另個用於解壓 Webpacked JS 的資源是 `Webpack Exploder`，作者是 @spaceraccoonsec: 
- https://spaceraccoon.github.io/webpack-exploder/

此外，閱讀 @infosec_au 的關於在 React Native 在 Android 的這個的優秀 blog
- https://blog.assetnote.io/bug-bounty/2020/02/02/expanding-attack-surface-react-native/

另外 sourcemapper
- https://github.com/denandz/sourcemapper
- 將吐出原始的 JavaScrip t文件，根據源碼表中的文件路徑重新創建源碼樹


 
---------------------------



Ref: https://twitter.com/pdp/status/1147928550307258368  
這篇不是 Jason 的，而是他 retweet 的  

Petko D. Petkov @pdp: 你沒有發現漏洞/缺陷的原因也許是環境設置不正確或者你的方法需要改進。  
這裡有些提示

0. 在多個 nodes 出口之間進行透視
    - 有些服務用了 ACL 規則，所以你將永遠無法從你的家到達它們
    - 使用 `AWS C9`
1. 如果你為 Chrome 瀏覽器使用單獨的 profiles，你就做錯了
    - 你將會瀏覽器和目標 app 同時進行搏鬥。不要這樣做
    - 使用 `pown cdb launch -t` 或 `pown cdb launch -t -P auto` 來啟動 chrome 的配置對 pentest有利
2. 不要拘泥於 scope。你需要有一個鳥瞰圖來了解事情是如何運作的
    - 不要 hack out of scope，但要把範圍外的目標考慮進去。
3. 你將和你的工具一樣優秀
    - 如果你所做的只是 Burp、ZAP 和類似的東西，你就會發現和同行發現一樣的錯誤
    - 你要明白，所有的工具都有自己的複雜性，如果你堅持使用一種方法，你會錯過一些東西。
    - Do Diversify
4. 盡可能多地實現自動化。有時你很幸運，你會得到小小的機會的窗口
    - 我會讓你知道一個這樣的錯誤，一旦它得到，我們可以說，我似乎只有幾個小時的時間來發現它
    - (這條的大意應該是，自動化能幫忙發現機會，但這個機會可能一下就過去了。)
5. 你可以做 surface 掃描，也可以 go deep，但不要同時做這兩件事
    - 你會迷路，你會錯過一些東西。我自己也多次犯過這個錯誤
6. 讀 old reports，越舊越好
    - 大家都在看最近的報告，其實都是遵循相同的軌跡
    - 一些你在今年 BH 會議上聽到的最酷的研究其實是基於 2006 年的論文
7. 擁有一個方法論。當我為倫敦一家精品諮詢公司進行 pentesting 時，我學會了使用一種成熟的方法
    - 但我最初並不喜歡這種方法論。
    - 我仍然認為它是我所遇到的最好的方法論之一。
    - (好像可以 google `pentesting methodology` 來查點資料)
8. 慢慢來
    - 明天你會有更好的想法。當你沒有找到任何東西時，你會感到很痛苦。
    - 這是創作過程的一部分
    - 不把它稱為失敗，我稱它為迭代

9. 不要執著於製造完美的系統。太多的人，包括我自己，都試圖做出完美的重建或完美的系統
    - 包括我自己，試圖製造完美的偵察或完美的自動掃描/基礎設施等等
    - 如果其他人已經建立了它，就利用他們的工作
    - 解決那些尚未解決的問題。


---------------------------

這篇不是 tweet，而是 Jason 的一篇 blog  
順勢讀了，在這邊做筆記  

The Anti-Recon Recon Club (using ReconFTW)  
Ref: https://www.jhaddix.com/post/the-anti-recon-recon-club-using-reconftw  


一些 Jason 前言  

Recon 很重要，但有些人討厭它。我明白。
- 當你進入狀態並準備撲向一個目標時，你只想開始砍人。
- 想要兩全其美嗎？快速/完整的偵察？不犧牲覆蓋範圍？
- 作為一個進攻性的安全和測試行家，我喜歡偵察
- 但在與許多其他黑客談論他們的流程後，它總是有分歧。其他人絕對不喜歡它，他們更喜歡以最快的速度進入一個目標。

我有什麼建議可以讓你們獲得最大的 recon 好處？怎樣才算是大覆蓋？
- 我稱它為 **recon++**
- 它是個尋找 subdomain 和相關偵察的 package

任何能做到這些的設置都是非常好的：
- Passive scraping (被動搜刮)
- Bruteforce (暴力)
- Permutations (排列組合)
- Certificate transparency (證書透明度)
- Github source code scraping (刮取 Github source code)
- Analytics analysis
- DNS 記錄分析
- Screenshotting
- 過濾重複的和 Live 的主機（web probing）
- Port Scanning
- Introductory content discovery

這使你對一個組織的 subdomains 有一個完整的了解，並開始對它們進行粗略的分析  


單獨做這些是很耗時的
- 因此，對於初始設置時間的 low-low cost，你就可以得到非常好的 `Recon++`
- 我的首選是 @six2dez1 的 [ReconFTW](https://github.com/six2dez/reconftw) framework，它將許多業界喜愛的工具打包成易於使用的 framework

ReconFTW 為你自動完成 recon 的整個過程
  - 也可用於運行一些粗略的自動漏洞檢查，如 XSS, Open Redirects, SSRF, CRLF, LFI, SQLi, SSL tests, SSTI, DNS zone transfers, and more.

這裡可以看到它能做的全部範圍和它使用的東西：

![](https://static.wixstatic.com/media/abefef_c915a29ac7af4cee8cc1bb765a64dc9e~mv2.png/v1/fill/w_2066,h_2912,al_c,q_90/abefef_c915a29ac7af4cee8cc1bb765a64dc9e~mv2.webp)  



## 它是做什麼的？它需要多長時間？

subdomain enumeration (using multiple tools)、截圖、buckets 和 zone transfers
- `./reconftw.sh -d target.com -s`
- 它可以在 10 ~ 60 mins 內提供這些所有，取決於目標的大小

這是 **recon++**，它加入了 port scane, content discovery 和 nuclei scanning
- `./reconftw.sh -d target.com -r`
- 可能需要幾個小時，取決於目標的大小


深度模式，增加了 cli spidering，並使用所有的工具進行掃描
- `./reconftw.sh -d target.com -a --deep`
- 這**可能需要一夜或幾天的時間**，取決於目標的大小


它看起來像這樣：  
![](https://static.wixstatic.com/media/abefef_68e05607d55f441985dbd8cc05bbacf8~mv2.gif)  

## 那如何從 ReconFTW 獲得最大利益？

ReconFTW 把不同的工具粘在一起，如 `Amass`, `subfinder`, `nmap`
- 為了得到最好的輸出，需要確保你有這些工具拉動 data 的服務的 **API key**

ReconFTW config 是你可以調整的地方
- 在全球範圍內
- 什麼樣的枚舉和掃描的工具做
- 一切從它的掃描速度，代理設置，哪些漏洞檢查，哪些 wordlist，哪些 proxy settings，timeout 選項，等等。
- Doc: [https://github.com/six2dez/reconftw/wiki/3.-Configuration-file](https://github.com/six2dez/reconftw/wiki/3.-Configuration-file)  


## 1. 設置底層工具之一的 Amass，讓你擁有的所有免費和付費 API Key: 

Amass 的 API key 定義在
- `~/.config/amass/config.ini`

Hahwul 有篇關於獲取 Amass 的 API key 的優秀 blog
- [https://www.hahwul.com/2020/09/23/amass-go-deep-in-the-sea-with-free-apis/](https://www.hahwul.com/2020/09/23/amass-go-deep-in-the-sea-with-free-apis/)


## 2. 確保你至少有 5 個 GitHub token 

定義在一個叫做: `${tools}/.github_tokens` 的文件  
這樣 GitHub 的搜刮有可能返回更正確的結果  
- https://github.com/six2dez/reconftw/wiki/3.-Configuration-file#3-tools-config-files

## 3. 如果想 ReconFTW 在 slack 中提醒你，或者有些 intel 平台的 PAID API key

可以在 ReconFTW config 中定義  

![](https://static.wixstatic.com/media/abefef_e838b15ce1ff4d88a84036e79b3ed50f~mv2.png/v1/fill/w_1215,h_703,al_c,q_90,enc_auto/abefef_e838b15ce1ff4d88a84036e79b3ed50f~mv2.png)  

## 這是我的 config（沒有 API key）

它刪除了一些 OSINT/Googling，和一些 js 分析，以及其他一些小調整。它允許快速`recon++`  
- [https://gist.github.com/jhaddix/141d9cb07ca0590dbc43389e0e4af98f](https://gist.github.com/jhaddix/141d9cb07ca0590dbc43389e0e4af98f)


到這邊，對於一次性的設置 cost，你可以得到偉大的 **recon++**，而且沒有什麼麻煩，讓我們回顧一下：
1. 安裝 ReconFTW
2. 註冊 API 和服務
3. 複製我的配置文件
4. 向 Amass 和 ReconFTW 加 API Key
5. 獲利

## ReconFTW and Amass pro Tips：

有新的工具出現時？    

現在你已經完成 Amass 和 ReconFTW config
- 如果另個工具出來，在尋找 subdomains 方面是新穎的呢？
- 看看 Patrik 關於將新工具與 Amass scripting engine 的 talk
  - 這 talk 非常熱門
- Youtube: [NahamCon2021 - Amassive Leap in Host Discovery - @ITSecurityGuard](https://www.youtube.com/watch?v=yCZqgg-GNx8)


處理 hanging 的掃描的問題
- ReconFTW 包裝很多行業標準工具包，ReconFTW 的每個階段都在運行這些工具中的一個
- 如果出於任何原因，你想跳過這個階段，你可以 spin up 

`htop`  

並找到`reconftw.sh`，在它下面的工具正在運行，在它下面的工具的 thead。  
只需 F9（殺死）`reconftw.sh` 下的工具就可以跳過它。  

![](https://static.wixstatic.com/media/abefef_6f5093e6cf004d18b444acba94b32c84~mv2.png/v1/fill/w_3135,h_1235,al_c,q_90/abefef_6f5093e6cf004d18b444acba94b32c84~mv2.webp)  


如果該階段停滯不前或耗時過長，或者你在 hack 時忘記從配置文件中刪除一個你不關心的階段，這就很有用
- ReconFTW 的偉大之處在於它在平面文件中保存了掃描進度。所以重新啟動它並不痛苦


設定 `Nuclei` Checks
- 當做 `-r` 或以上 `ReconFTW` 增加了 nuclei scanning
- 來自 `nuclei` 的 info 和 low check 可能相當繁忙，你可以調整 ReconFTW config 排除哪些檢查
- 打開 config 文件並搜尋 `"NUCLEI_FLAGS="`


在這裡可以指定想排除的模板，用 `-eid` flag。編輯會看起來像
- `NUCLEI_FLAGS=" -silent -t $HOME/nuclei-templates/ -retries 2 -eid addeventlistener-detect,tech-detect,ssl-issuer,ssl-dns-names"`


有時，我只將我的 Nuclei config 設置為:
- `NUCLEI_SEVERITY="medium,high,critical"`。
- 並完全省略 info 和 low

對於藍隊隊員的 ReconFTW
- 用 ReconFTW 的另個好方法是從藍隊的角度
- 你可以用它來為你的組織建立一個外部資產清單/攻擊面程序
- 它比外面 80% 的付費產品更便宜、更有效
- 來自黑客領域的偵察工具在不斷發展，許多資產管理/攻擊面管理公司都沒有跟上。為你的組織節省20-40萬，並在電子表格中跟踪它。


## 這些工具/框架不會為你做什麼？

當使用 `-a或/和-deep` 進行漏洞檢查類型的掃描時
- 大多數漏洞檢查工具都是 command line tools、檢查 URL 的 config 問題(nuclei)，並對每個掃描的 "gf files" 中的參數進行模糊處理

`gf files` 只是 cli scanners 從歷史來源（wayback machine ++）中發現的 URL 和 parameter
- gf 從輸出中解析出 "有趣的" 參數名稱

因此，這些工具
- 不是對所有的東西進行模糊處理，也不是對基於 REST 的端點進行模糊處理
- 如果你加 `--deep`，`gospider` 會做一些 cli spidering，但這也有其局限性（沉重的 js 和其他技術）
- 另外，這 option 使得掃描需要幾天時間，而不是幾個小時

## 這是我的看法：


我重視 ReconFTW，因為它的 subdomain discovery 和相關偵察
- 我把其他的東西都當作一個快樂的獎勵。我並不指望這些漏洞工具能給我帶來多少東西
- 任何 `on application` 的測試都應該用 Burp 手動完成
- 大多數時候，我甚至會重新進行 content discovery，以調整特定網站的過濾器和規則
- 總而言之，這些注意事項適用於任何將這樣的工具整合在一起的 framework


更多關於作為 ReconFTW 一部分的 Nuclei 的資訊
- 找到你想排除的 template ID的最快方法（因為它並不總是與 cli 輸出相匹配）是從你想排除的命令行中取出模板並運行

這應該會返回 yaml template 的路徑，裡面有 ID
- `nuclei -tl | grep ssl-dns-names`


不過，有些 template 是巨大的集合
- 以 `tech-detect` 為例。它容納了許多技術檢測規則
- 但如果只想在那個 multi-process template 中排除其中一個規則呢？

My command line says it’s alerting on:
- `[tech-detect:cloudflare] [http] [info]`
- 我並不關心是否知道這些，而且這也堵塞了我的掃描

我首先要用上述命令找到 `tech-detect` 的 YAML
- 並通過排除其 ID 來排除其所有檢測
- 要單獨排除 Cloudflare 檢測部分，唯一的辦法是實際編輯 YAML 文件，刪除那個檢查


Bonus: @six2dez1 關於他的經驗和創建該工具的 interview
- Youtube: [Bounty Talks v1.9 - Alexis (six2dez)](https://www.youtube.com/watch?v=RSj09I3wRFM)
