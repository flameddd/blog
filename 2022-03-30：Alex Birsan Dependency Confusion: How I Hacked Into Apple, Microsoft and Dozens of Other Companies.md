# 2022-03-30：Alex Birsan Dependency Confusion: How I Hacked Into Apple, Microsoft and Dozens of Other Companies
## Alex Birsan Feb 10, 2021
### https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610

這篇是 Supply Chain Attack，而且主角是 npm  

作者明確指出，在這項研究中針對的每個組織都已通過公共漏洞賞金計劃或通過私人協議提供了對其安全性測試的許可。 請不要在未經授權的情況下嘗試此類測試。  
-------------------------------

python 安裝 dependencies 的指令
```
pip install package_name
```

很多語言都有這樣的設計，Node npm, python pip, Ruby's gems  
當我們這樣下載食，基本上我們都是直接相信這些 publisher，讓他們在我們的機器上跑  

任何託管服務都不能保證其 User 上傳的所有 code 都沒有惡意軟件
- typosquatting（利用 popular package 的拼寫的攻擊）就可以非常有效地訪問各地的隨機 PC
- 當然還有其他眾所周知的依賴鏈攻擊路徑包括使用各種方法破壞現有的 library

## background
@Rhynorater 分享了在 GitHub 上找到的一段有趣的 Node.js source code，這段 code 供 PayPal 內部使用，並且在其 package.json 文件中，似乎包含 public 和 private dependencies
- public 來自 npm
- private 的最有可能來自 PayPal 內部託管的 private package，這些名稱當時在 public npm 中不存在。

![](https://miro.medium.com/max/700/1*rbNPMp7FJF816Bdf_Ea-rQ.png)

那，如果把惡意的 code，用這些名稱上傳到 npm 會怎麼樣？
- 是否有可能 PayPal 的一些內部項目將開始默認使用新的 public package，而不是 private？
- developer，甚至自動化系統會開始在運行 code 嗎？
- 這種攻擊是否也適用於其他公司？

作者就打算 publish 到 public npm 中，一旦有人安裝，就會通知他  


## “It’s Always DNS”
作者在想，如何將這些數據取回給他？  
- 知道大多數可能的目標都在受到良好保護的公司網絡的深處，認為要靠滲漏 DNS 
  - 通過 DNS 將 data 發送到的服務器對於測試本身來說並不是必要的，但它確保了流量在輸出時不太可能被阻擋或檢測到
  - data 經過 hex-encoded 後，並用來當作 DNS 查詢的一部分
  - 該查詢直接或通過中間解析器到達我的 custom authoritative name server
  - server記錄每個收到的查詢，基本上記錄了下載 library 的每台機器


![](https://miro.medium.com/max/700/1*nTCUgLtONUx8EhnCTgunjQ.png)  





## 多多益善
有了基本策略後，下一步是擴大攻擊範圍
- 先是朝其他生態圈，`PyPI` and `RubyGems` 都做類似的事情

這個測試，最重要的一環是要盡可能找更多相關類似的 dependency names  
- 作者針對特定幾間公司，經過幾天的搜尋。在 github 和幾個主流的 package hosting services 找到一些被不小心發布出來的 package

作者說
- javascript 檔案 和 package.json 是最容易找到這些 private dependency name 的地方
- 裡面的 `require()` 也很常看到 private

(蘋果、Yelp 和特斯拉只是以這種方式暴露內部名稱的公司的幾個例子。)  
![](https://miro.medium.com/max/700/1*kXIpJ2pZhu40yT17cbwheg.png)  

作者找了方法，自動掃描目標公司的網域，從中找出上百個沒有被 publish 到 npm 的 javascript package names，然後就用這些 name 來上傳到 public npm

## 結果
作者的測試非常成功，從
- developer 的一次性錯誤（應該是指發布)
- 內部的 misconfigured 或 clould 設定錯誤 等等

證明這種方式很有效的能夠遠程執行 code  
，有一點很清楚：搶注有效的內部包名稱是進入一些最大的科技公司的網絡，獲得遠程代碼執行，並可能允許攻擊者在構建過程中添加後門。

作者在 Shopify 跟 apple 分別拿到 $30,000 bug bounty reward ...  


## “It’s Not a Bug, It’s a Feature”
為什麼會發生這種情況？ 此類漏洞背後的主要原因是什麼？作者說，在他的研究和與安全團隊的溝通中確實發現了一些有趣的細節
- Python 依賴混淆的罪魁禍首似乎是錯誤地使用了名為 `--extra-index-url` 的設計不安全的 cli 參數

當此參數與 `pip install library`，pip 實際上做的事情是：
1. 檢查指定（內部）library index 上是否存在此 libaray
2. 檢查公共 library index (PyPI) 上是否存在 library
3. 安裝找到的任何版本。如果包在兩者上都存在，則 default 從具有更高版本號的來源安裝。

因此，將名為 library 9000.0.0 的 library 上傳到 PyPI 會導致上例中的依賴被劫持。

儘管這種行為已廣為人知，但在 GitHub 上搜 `--extra-index-url` 就足以找到一些屬於大型組織的易受攻擊的腳本
- 包括影響微軟 .NET Core 組件的錯誤。該漏洞可能允許向 .NET Core 加後門，不幸的是，該漏洞超出了 .NET bug bounty的範圍。

