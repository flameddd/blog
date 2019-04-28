## Launching the Samsung S10 to 17 different markets in three weeks — a UX case study
### Jon Crabb 2019 Apr 12
#### https://uxdesign.cc/launching-the-samsung-s10-to-17-different-markets-in-three-weeks-a-ux-case-study-25c78ef2ce9b


### how Samsung configured their `.com` for the launch of the S10. 

這邊會討論到
-UX process
- user flows
- wireframes

想了解 giant product 的 launches work  
這篇應該很有趣

這個大專案已經 run 很多年了
- final web 的 launch 只用了一個 sprit (三週)

 S10 launch date 的前三週 
 - 招集世界各地的同事到一起，並 set up a clean room

在這邊，我們
- 能夠嚴謹的保證數位安全
- 最重要的是，能夠當面溝通、討論

#### email and Slack 禁用
禁用這些
- 提升做決策的效率
- 減少 security leak 機會

每一個市場地區的網頁
- 設計
- coded
- 發佈

都是藉由相對 小的group 在很短的時間內完成！　　
作者加入 Cheil (新加坡商第一企劃行銷股份有限公司、韓國第一大的廣告公司)  
- 還不知道如何 product launch
- 從沒進去過 `clean room`

NDAs 過期了，所以他來寫下這段心得

### The Challenge 挑戰

Samsung Europe 
- 有 17 個市場
- 遍佈 31 個網站
- 需要 27 種語言
- 每個網站有不同的需求


#### 首先
- 送貨員能穿越的 international borders 都有所不一樣
- Tariff(進口貨物稅) 有的有選擇、有的無法選擇
- 有些市場允許 trade-ins (折價貼換交易、例如舊機換新機)，有些市場沒這個
- Some allow the user to lease or purchase outright

這些都是各自市場的特別 offers

#### 舉例
- 客戶在 UK 點擊 `buy now`
- 並且設定她偏好的 options
  - 她要 512GB RAM
  - Prism Black
  - 2年保固

同時，她朋友在法國決定要買同一台手機，但
- 不同顏色
- 不同容量
- 特別方案，所以沒有提供延長保固
- 有提供不同配件

她們可能買
- 同樣的手機
- 但是在不同的 website

這其中就會**有數百種不同的流程**  
- 結合不同的選擇
- 針對不同的市場

我們面前有上千種路

![img](https://cdn-images-1.medium.com/max/1000/1*3_YTmESnZ8yge1ustTyFvQ.gif)

### UX team
UX 的工作是
- 把 configurators 的 wireframe 弄出來
- stay on top of all the updates
- 盡可能清楚的跟 developers 溝通

UX had the unenviable job of writing raw code over some very long nights.

### Clean Room
在 Clean Room 工作  
相關的規則  

Clean Room Security Governance
- A NDA must be signed and approved prior to any employee accessing the clean room
- All employees must use their electronic ID cards every time they enter the clean room
- External storage devices must not be used. Where possible, disable USB ports on PCs
- Internet restrictions must be applied to allow only protocols and services as approved and required
- VPNis notallowed
' A printer, shredder, and landline must be provided, and CCTV implemented
- Daily checks are required to ensure confidential data is properly stored/disposed of
- Any security breach must be reported to ___ and ____ immediately, who will report to Samsung as required
- Web development must be on an offline server with access to the internet blocked until Unpacked
- Server configuration must be checked regularly to ensure vulnerabilities such as "Directory listing” (目錄瀏覽弱點 the biggest cause of data leaks)
- ___________ must be protected with a strong password and only shared with those
that need access, The ability to share/forward/copy/print/download must be disabled
- Any content shared via email must
  - omit any reference to the launch product
  - be zipped and password protected (the password must be sent separately to the content)
  - have a watermark
- It is prohibited to;
  - take/capture any photographs, video, audio
  - or any digital image using any personal device
  - use any external file transfer and storage that isn't approved by the business

![img](https://cdn-images-1.medium.com/max/1000/1*781wWh-Wv0vrXlPxsvdwsg.jpeg)

所有的照片、影片拍攝都是透過供公司器材，這些器材都被鎖在 cleanroom 值到 unpack day

### 第一天
- 我們搬進空的 office
- 一排 monitors、pristine desks
- 一整面牆的磁力白板
- 另邊有好幾個時鐘，顯示各國際時間
- 幾間會議室
- a well-appointed kitchen

這就是未來三週的 command centre  
每人一定要簽 NDAs  

- 禁用 USB 
- 個人裝置去 Photography 也禁止
- 有保護的 wifi
- 所有時間，我們都關起百葉窗，避免偷拍、特別強的鏡頭偷拍

雖然 remote 越來越熱門，但這種集中式的 clean room 也有其優勢  
- UXing 大概是我學到最多的部分，２週、大家能坐在一起工作的成效真的很驚人  
- `Lean` 是有名的難以 scale 的
  - 但對問題來說，這確實是個 elegant solution。
- 3週，核心成員 40 在  clean room
  - 最忙的時候，擴到 108 人
- 一想到如果大家在不同地方、用 email 之類的溝通...  效率...

![img](https://cdn-images-1.medium.com/max/1000/1*0DtAjeVBxFUJLMp-pXU-ug.gif)

### The Process
我們用 `Axure` 做 `wireframes`  
（作者說另外還有兩套更好 Figma and XD）  
這些 app 都能
- 產生高真度的設計
- 但這些也 encourage time spent labouring over pixels
  - 有時候這會是阻礙

我們的目標是
- 需要 simple wireframes
  - 很多很多的 simple wireframes
  - 要快！

Axure 是很好的工具來
- 產生網站的 broad strokes 

大致上來說，`每一個 configurator` 所包含的
- 不同的 page elements，例如
  - 顏色
  - 容量
  - OFFERS
  
我們用 `atomic design` 的方式，這樣來讓每個元素的行為能夠單獨  
設計這些元素，把它存在 Axure 的 `Masters` （應該是類似版本的控管 or 分支  

- UX team 跟 PM 一起
- 把 31 個網站建立了一張巨大的 spreadsheet
- 每個單獨市場的 account manager 跟 PM 溝通
  - 需求有變的時候，PM 更新 spreadsheet

This listed whether a component was needed, and where it should be located.

我們
- 可以用 spreadsheet 來確認
- 對 pages 上的元件正確、依序的做 wireframe
- 然後用 密碼加密的 pdf 給 PM review

當
- page elements 存到 masters 後
- 我們就能 drag and drop elements 到 wireframes
- 有問題的話，就 update 到 master，然後看 all pages 的 flow 改變

![img](https://cdn-images-1.medium.com/max/1000/1*zSxl-C_oPvNRP11r-zjh2A.gif)


但
- spreadsheets 還是太抽象了
- 人是視覺動物
- 為了安全性，email 前，pdfs 必須要加密 (長密碼)

我們的 lean process 慢下來了  
這時候還是要靠老方法！紙張

### Print-outs

每一個 wireframe 我們
- 加入了一張 specs 清單在一旁
- 印在 A3 上

這份清單包含
- full page 的 wireframes
- 元件的 reference checklist，附註 Keep/Remove 在旁邊

這些圖是個範例、也顯示出每一個區域市場的不同  
![img](https://cdn-images-1.medium.com/max/500/1*Dhu81w2kCvtIpxXBOKGQag.jpeg)
![img](https://cdn-images-1.medium.com/max/500/1*spYTYf4qb52gMG3Y-D9GMQ.jpeg)  
![img](https://cdn-images-1.medium.com/max/500/1*JakYr2a-Pl-EA0TVpV2kjA.jpeg)  

這些都黏在白板上  
這些方法看似簡單，但提供 
- ncredibly clear guidelines to work with
- 也能夠清楚的 at-a-glance view，在 clean room 每一個人都能一眼就能到整個專案

![img](https://cdn-images-1.medium.com/max/1000/1*Xo9rsfHAANHn5nayRl64nw.gif)  

任何人
- 有需要的話都走過能來 check a page
- 這是 Single Source of Truth
- 當單一市場改變、改動 spreadsheet，PM 會 request 這些改動給 UX
  - UX 會調整，然後再給 PM review
- 有時我們還是用 pdfs，但印出來的還是 main reference

#### 別低估
- 別低估這些有形、實體的物件，在數位、抽像的世界裡

### Signposting 指示牌
我們主要工作是
- `configurator page`
- 另外還負責整個 .com 的 `user flow`

主要的日期是 `UNPACKED event`，Samsung 的 live conference 來宣布產品，啟動 S10  

1. pre unpack
2. unpacked
3. post unpacked
4. launch

![img](https://cdn-images-1.medium.com/max/1500/1*NagpZLdiBXx8Feb4cbHBbg.png)  
 
查看`購買 S10 的潛在 user journeys`， key pages 有
- /mobile
- /smartphones
- /galaxy-s
頁面

這些分類全部對 featured products 提供某些  digital real estate  
我們的 wireframes 清楚指出　the zones that would be populated with BEYOND assets  
這些也印出來貼在牆上給所有人看  

![img](https://cdn-images-1.medium.com/max/500/1*oToA3WVGdkaXkVcaS2Gxhw.jpeg)
![img](https://cdn-images-1.medium.com/max/500/1*Yn_6KHHd4Nd3lnws5tuKzA.jpeg)
![img](https://cdn-images-1.medium.com/max/500/1*8swKd_x3kh58ncSPbVDghw.jpeg)  


### Connecting it all together
為了要展示每一個畫面的 flow ，我們用 `string`
- Again 別低估 physical  
- 數位流程圖雖然清楚，但 sterile

Following bits of string to see where the link will take you is an engaging action  
You’re on
- your feet
- you move
- you physically interact with the process
- and you actually understand the connections.

Samsung 像其他大廠一樣，用不同的縮寫代表網站的不同區塊 
- PCD (Product Category Display), 
- PFS (Product Family Showcase), 
- PF (Product Finder), 

這些定義也有助於連結一些不不透明的區塊  
一大張　spider diagram 印出來貼在牆上，幫助 visualize that network  

### Conclusion 結論
I’ve focused on the UX process, but it’s important to state we were a small cog in a big machine. So I have to conclude this by giving huge props to every member of the BEYOND team. The fact that everything worked so well is testament to their talent and effort.　　

The team included
- various levels of Samsung Europe

supported by Cheil’s
- teams of project
- and account management
- user experience
- content & publishing
- developers
- designers
- operations
- QA
- SEO
- management
- and of course IT
