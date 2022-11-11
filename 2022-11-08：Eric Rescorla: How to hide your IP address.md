# 2022-11-08：Eric Rescorla: How to hide your IP address
## Eric Rescorla, Firefox 的 CTO, 17 Oct 2022
### https://educatedguesswork.org/posts/traffic-relaying/

來看看 Firefox 的 CTO, Eric Rescorla 如何談 How to hide your IP address  

----------------------------------------

Eric 說在前兩篇文章有提到，如果真的想要保持你在 internet 上的隱私的話，你需要
1. 保護你自己的 IP
2. 保護你要去的 server 的 IP

這篇文章就是在談這些事情

前兩篇文章:
1. [private browsing](https://educatedguesswork.org/posts/private-browsing/)
2. [public WiFi](https://educatedguesswork.org/posts/public-wifi/)


--------------------

## 1. 基礎知識
任何安全問題，我們需要從威脅模型開始。我們關注的是兩種主要的攻擊模式:
1. Server 學習 User 的 IP 並利用它來識別他們或關聯他們的活動。
2. 本地網路學習 User 要去哪些 server 
3. Server 利用你的 IP 確定的明顯的地理位置來限制對某些類型的內容（足球、BBC，等等）的訪問
    - (有些人可能會認為最點不算是一種攻擊，這取決於你的觀點)

防禦 1 和 3 的基本技術是通過某種匿名中繼(anonymizing relay)來推動流量

![relay-basic](https://educatedguesswork.org/img/relay-basic.png)  

上圖
- client 連接到 Relay 並告訴它在哪裡連接
- 然後，client 將流量發送到 Relay，由 Relay 轉發到 server
- Relay 用自己的 IP 替換了 User 的 IP，所以 server 只看到 Relay 的地址

一般來說，Relay 將為相當多的 User 提供服務
- 所以 server 將發現很難區分哪一個是哪一個
- 這個簡單的版本清楚地解決了`威脅 1`
- 而且，如果 Relay 營商讓你選擇一個在你自己的地理區域之外的 IP，`威脅 3` 也能解決
- 為了防禦`威脅 2`，你還需要加密到 Relay 的流量，這樣網路上的攻擊者就不能看到你連接到哪個 server 以及你向它發送的流量
  - （關於這種形式的 data leak，請看[這裡](https://educatedguesswork.org/posts/public-wifi/#routes-for-browsing-behavior-leakage)）
- 理想情況下，你也會對到 server 的流量進行 E2E 加密(用 TLS 或 QUIC)，但這只是一般的良好做法，對 Relay 提供的隱私沒有要求

--------------------

## 2. Relaying 的選擇
這種基本設計是每個 relaying 系統的核心，但細節上有重要的不同。有三個主要的變化軸
- 發生 relaying 的網路層
- 網路中的 hops (跳躍次數)
- 商業模式

下文中分別介紹

--------------------

## 2.1 網路層
第一個主要的變化點是發生 relaying 的層
- 這一點需要對 Internet networking protocols 有一定的背景了解


--------------------

## 2.2 IP
Internet 上
- 最基本的協議就是所謂的 Internet Protocol（IP）
- IP是所謂的 `packet switching` (包交換) 協議
- 意味著其基本單位是一個獨立的 message，稱為 packet
- 一個 packet 就像一封信，它有一個來源地址和一個目的地址
- 你在網路上發送一個 IP packet 時，Internet 可以通過查看 packet 自動傳送到目標地址，而不需要關於任何一台 computer 的其他狀態

一個簡化的 IP packet 看起來像這樣:

![](https://educatedguesswork.org/img/IP-packet.png)

packet 中的主要內容是要從源頭傳遞到目的地的實際資料
- 也稱為 payload。payload 的長度是可變的，最大通常為 1500 bytes 左右
- packet 還有一個下一個協議字段，它告訴接收者如何解釋 payload（後面會介紹）
- packet 還有一個長度字段，這樣就可以知道整個 packet 有多長，包括可變長度的 payload

使用IP非常簡單:
- 你的 computer 在電線上傳輸一個 IP packet
- Internet 使用目標地址來計算出路由的位置。當有人想向你傳輸時，他們也做同樣的事情

--------------------

## 2.3 TCP
想要發送一個比 1500 bytes 長得多的 IP packet（例如，一個文件）是非常常見的
- 在 high level 上，可以將資料分解成一系列 smaller chunks 並在單個 packet 中發送來實現

但生活並不那麼簡單。比如說:
- packet 可能會丟失，必須重新傳輸，以便接收者得到它們
- packet 可能會被重新排序，而接收方必須知道該把它們放在哪個順序
- 一般來說，網路將無法一次處理整個大文件，所以資料必須在一段時間內逐步傳輸。發送方必須有某種方法來確定適當的發送速率


[`Transmission Control Protocol (TCP)` (傳輸控制協議)](https://en.wikipedia.org/w/index.php?title=Transmission_Control_Protocol&oldid=1114104762)負責處理這問題:
- TCP 的細節太複雜，無法在這文章中體現出來
- 但在高層次上，data stream 被分成若干段，每段都有長度和序列號，它告訴你它在 data stream 中的位置
- 每個段都在一個 IP packet 中發送。當接收器得到一個段時，它可以查看序列號來重建 data stream，並能夠檢測到 packet lose
- TCP 還包括一個確認機制，接收方告訴發送方它收到了哪些段；這允許發送方重新傳輸丟失的 packet，並適當調整其發送速率

當然，除了 TCP，還有其他協議可以在 IP 上運行（例如，後面提到的 UDP）
- 這就是為什麼你需要 IP 中的  `next protocol` 字段告訴接收者 IP payload 中的協議是什麼

--------------------

## 2.4 TLS
TCP 是一個非常古老的協議
- 和大多數老的 Internet protocols 一樣，它是在廣泛使用加密技術之前設計的
- 從安全角度來看，這顯然是個壞消息，最終人們開始著手解決這個問題
- 標準的解決方案是通過[Transport Layer Security(TLS)](https://en.wikipedia.org/w/index.php?title=Transport_Layer_Security&oldid=1110721112)(傳輸層安全)來傳輸資料
- TLS 在 TCP connection 的基礎上提供了一個加密和認證的 data stream 的抽象概念
- 與 TCP 一樣，你需要設置一些狀態來使用 TLS，這被稱為 `TLS connection`


--------------------

## 2.5 UDP 和 QUIC

Applications 本身並沒有實現 TCP
- 相反，它是內建在 OS 中的，特別是在所謂的 OS kernel 中
- Client side application 告訴 OS 建立一個與 server 的 TCP connection
  - 從而在 client side 建立所謂的 `socket`
- Client 將 data 寫入 socket，kernel 自動打包成 TCP segments，並將其傳輸給另一方，並負責重傳、速率控制等
- kernel 還從另一側讀取  TCP segments，並將其提供給 application 讀取
- 通常情況下，application 自己實現 TLS，或者更有可能的是使用一些現有的 TLS library

為什麼不能寫自己的 TCP stack ?
- 你當然可以寫自己的 TCP stack（畢竟它只是 software）
- 但問題是你不能安裝它，因為在大多數 OS 上，普通 application 不允許寫入或接收原始 IP datagrams
- 這是對網路行為的一些限制，在前加密時代，這些限制曾經被用於 security enforcement

例如
- 以前某段時期，人們認為如果一個 packet 來自一個特定的機器地址和一個特定的 `port number`，那它就是來自一個有特權的 process（一個 OS privileged process）
  - 甚至有整個 [system for remote login](https://en.wikipedia.org/w/index.php?title=Remote_Shell&oldid=1070274903) 基於此，你可以在 A 機器上執行 B 機器上的命令，而不需要驗證
- 現在聽起來很荒謬，但這是 80 年代初到 90 年代末的情況，那時我們終於有了適當的加密認證（至少在某些時候是這樣）

這很方便
- 因為 application 不需要隨身攜帶自己的 TCP implementation
- 但不方便的是它不靈活：假設 application 想對 TCP 做些改變以使其更有效？
- 如果不改變 OS，就沒有辦法做到這一點。相比之下，改變 TLS 的行為很容易，只需提供新版本的 application
- 這點在 2010 年代末變得尤為突出，當時人們想對 TCP 進行性能提升，但由於 OS 的動作不夠快而無法實現
- 解決方案是發明一個新的協議，可以完全在 application 中實現: [QUIC](https://en.wikipedia.org/w/index.php?title=QUIC&oldid=1114290192)。

QUIC
- 有點像 TCP 的高級版本和 TLS 的加密技術的結合（它在內部使用許多 TLS 的部分）
- 由於它可以完全在 appliction 中實現，它可以非常迅速地被改變
- 不幸的是，在大多 OS 中，application 不允許直接寫入 IP packets
- 因此 QUIC 通過一個稱為 [User Datagram Protocol (UDP)](https://en.wikipedia.org/w/index.php?title=User_Datagram_Protocol&oldid=1112673995) (用戶數據報協議)的協議運行
- UDP 是非常簡單的協議，它只允許 application 過 IP 發送單一的 data (datagrams)
  - 因此，QUIC 通過 UDP 運行，UDP 通過 IP 運行

--------------------

## 2.6 The protocol stack
傳統的做法是將其作為協議的 `stack` 來談論
- 並在一張被稱為 `layer diagram` 的圖片中將其可視化，就像這樣

![](https://educatedguesswork.org/img/TCPIP-layer.png)   

當 application 想寫資料時，它從 stack 的頂部開始，data 向下移動到網路。當 data 從網路進來時，它在 stack 上向 application 移動  


就 data 在網路上出現的方式而言
- 每一層都增加了自己的封裝，通常是在 data 之前或之後
- 下圖顯示了兩個例子
    1. 通過 TCP 發送的 data，在這個例子中是字符串 `Four score and seven years ago`
        - TCP 加了自己的 header，包括序列號等，然後將其傳遞給 IP 層，後者添加了帶有來源地址和目標地址的 IP header
    2. 通過 TLS 發送相同的 data
        - TLS 層對 data 進行加密（如劃線所示）
        -並加自己的 header。然後，它將 data 傳給 TCP，TCP 再加上自己的 header，等等
    3. 接收過程將這些操作反過來


![](https://educatedguesswork.org/img/tcp-tls-packets.png)    

命名 chunks of data:
- 你可能會注意到，文章一直用 `packet`, `record` 等術語。這些都是不能互換的
- 網路中最惱人的問題之一是如何命名一個單一的 data 單元，如 packet (有時泛稱 protocol data unit(PDU))
- 每個協議往往都有自己的術語，部分原因是由不同的人定義，部分原因是當你在 protocol stack 的多個層工作時，談論 `IP datagrams`, `UDP datagrams` 等是很麻煩的

下面是 Eric 對不同協議中 PDU 名稱的不完整表格:  
|協議|名稱|
| :---: | :--- |
|Ethernet|Frame|
|IP|Packet (datagram)|
|UDP|Datagram|
|TCP|Segment|
|TLS|Record|
|QUIC|Packet (but it has things inside it called frames)|
|HTTP|Message|
|RTP|Packet (but they carry media frames)|
|OpenPGP|Packet|
|XMPP|Stanza|

有一點很重要
- 就是 TCP 和 TLS 提供的是 stream of data 的 abstraction，而不是一組 records 的 abstraction
- 這意味著 application 只是寫 data，而 TLS 或 TCP 將這些 data chunks 凝聚成一個 record (packet)，或在其方便時將其分割開來
- TCP 甚至可能用兩個不同的框架將相同的 data 發送兩次
  - 例如，假設 application 寫了 "Hello"，然後 kernel 在一個 packet 包中發送。當 packet 在飛行時，application 寫下 "again"
  - 如果這兩個 packet 包都丟失了，kernel 不得不重新傳輸，它可能會把它們寫成一個 TCP segment（"HelloAgain"）

--------------------

## 3.  哪個 Layer ?
有了這些基礎背景，就可以討論「要在哪一層轉發流量？」，有兩個主要的選擇，至少對於 relaying 加密流量而言
1. 中繼 IP 層的流量 (Relay the IP-layer traffic)
2. 中繼應用層流量（即通過UDP或TCP的數據）(Relay the application layer traffic)



--------------------

## 3.1 中繼 IP 流量 (Relaying IP Traffic)
在網路層（IP）對流量進行加密是解決網路安全問題的明顯方法之一
- 因為它有一個重要的優勢，一旦設置了它，它就能保證兩個端點之間的所有通信安全
- IETF 在1992 年開始以 IPsec 的名稱進行技術標準化
  - 最初的想法實際上並不是上面討論的那種中繼系統，而是在兩台相互通信的機器之間加密通信
  - 舉例，如果我的客戶想要與你的 server 進行通信，我們將採取我們想要發送的 IP packets，對它們進行加密，然後直接發送

像我們上面討論的協議一樣，IPsec 是一個封裝協議
- 意味著為了加密一個從 A 到 B 的 IP packet，我們將整個 original packet 進行加密，然後將其塞入另一個 IP packet


![](https://educatedguesswork.org/img/ipsec1.png)  


在上面討論的方案中
- 內部（加密）的 IP header 和外部（明文）的 IP headeer 將有相同的 address information
- 也可以讓它們有不同的 address information，這對於創建所謂的虛擬私人網路（VPN）很有用

這裡的動機是，你有兩個網路（比如同一公司的兩個辦公室），你想把它們連接起來，就像它們在同一個地方一樣
- 在辦公室內，你相信電線沒有被篡改（這是在 WiFi 之前），所以你不對所有的資料進行加密（這現在聽起來很天真）
- 所以你真正想要的只是一條連接`辦公室1` 和`辦公室2` 的電線。這種私人連接過去被稱為「租賃線路」
- 而你實際上擁有的是一個 Internet connection，它讓你與所有人連接
- 但如果對`辦公室1` 和`辦公室2` 之間的通信進行加密，那麼就可以模擬擁有自己的私人線路

因此，虛擬專用網路。典型的拓撲結構看起來像這樣

![](https://educatedguesswork.org/img/enterprise-vpn.png)  

在這種情況下，你有兩個辦公室
- 每個辦公室都有一個 `VPN gateway`，檢測從`辦公室1`到`辦公室2`的流量，並在發送前對其進行加密
- 其他流量，比如說到 Facebook，則不被觸及
- 當遠端 `VPN gateway` 收到 packets 時，它只是刪除了封裝並將其丟棄在網路上。其效果就像有一個單一的網路而不是兩個網路

也可以在更簡單的情況下部署這種東西，即單個用戶 VPN 進入他們的辦公室網路
- 例如，如果你在酒店遠程工作，如下圖所示

![](https://educatedguesswork.org/img/remote-access-vpn.png)  

就像你在辦公室裡一樣，但你實際上不在
- 但這帶來了一個真正的問題，遠程用戶的機器沒有正確的 IP
- 它有一個與用戶的家或辦公室相關的 IP （上圖中的 192.0.2.1）
- 但你希望它看起來是在辦公室，這意味著它必須有一個辦公室的 IP (以 203.0.112 開頭的東西)

有兩種主要方式
1. `VPN gateway` 告訴 User 的設備它希望有什麼 IP，然後用戶的設備把它放在內部 IP header 中，而外部 IP header 有實際地址
    - 例如，內部（加密）IP header 將有 `203.0.11.50`，外部（明文）IP header 將有 `192.0.2.1`
2. 讓兩個 header 都有 User 的實際 IP，並讓 `VPN gateway` 將該地址翻譯成適合辦公室網路的本地地址（並在回程中以另一種方式翻譯）
    
在這兩種情況下，`VPN gateway` 都需要做一些工作
- 第一種情況是跟踪分配的地址並強制客戶使用正確的地址
- 第二種情況是進行翻譯


有了這些背景，我們可以開始討論問題陳述，那就是隱藏用戶行為
- 你可以使用與遠程訪問相同的技術
- 不同的是，`VPN gateway` 直接在 Internet 上，而不是在某個企業網路上

如下圖所示:  
![](https://educatedguesswork.org/img/consumer-vpn.png)  

對 server 來說
- 這看起來就像 User 從 `VPN gateway` 連接，不管 `VPN gateway` 的 IP 是什麼
- 客戶端的本地網路只是看到了與 VPN server 的連接，但不知道最終會去哪裡

在這裡，我把重點放在 `IPsec` 上，但使用哪種加密層協議來傳輸 IP packets 包並不重要：它們只是被封裝並進行端到端的傳輸  
在實踐中，人們看到 VPN 部署了各種傳輸協議，包括
- [DTLS]（https://en.wikipedia.org/w/index.php?title=Datagram_Transport_Layer_Security&oldid=1115742845）
- [OpenVPN]（https://en.wikipedia.org/w/index.php?title=OpenVPN&oldid=1107224624）
- [WireGuard]（https://en.wikipedia.org/w/index.php?title=WireGuard&oldid=1113944737）
- QUIC

從用戶的角度來看
- 這些協議的屬性基本相同。大多數標有 `VPN` 字樣的產品在 `IP layer` 使用一種或多種協議保護通信


--------------------

## 3.1 中繼應用層流量 (Relaying Application Layer Traffic)
如上所述，在 `IP layer` 保護流量的好處是，它可以保護系統中的所有流量。然而，不好的是，保護 `IP layer` 流量需要 OS 的配合。有幾個不理想的後果:
1. Code 在 OS 之間不能移植
2. 許多 OS 需要某種 admin 權限，以便安裝或配置在 `IP layer` 發揮作用的東西
3. 你經常被限制在 OS 供給你的任何功能上。例如，你可能不容易保護某些流量而不保護其他類型的流量

這些問題可以通過在 `application layer` 而不是在 `IP layer` 進行 layer 來解決
- 這可以完全在 application 中實現，而不接觸 OS，application 只是連接到中繼（例如，通過 TCP），並將流量發送到中繼（希望是加密的 server）
- 中轉站建立自己的傳輸級連接到 serve ，並將應用級流量發送到 server，如下圖所示。


![](https://educatedguesswork.org/img/application-relay.png)  


注意，圖中有兩個 TCP connections
- 一個在 User 和中繼之間
- 一個在中繼和 server 之間
- client 通過 TLS 連接到中轉站
- 在此基礎上建立一個 E2E 的 TLS 連接到 server（當然你可以不對 server 的數據進行加密，但不要這樣做）


這種設計的一大優勢是
- 它可以很容易地轉發某些類型的流量，而不是其他的流量

一個具體的例子是 [Google Safe Browsing](https://safebrowsing.google.com/)
- 它將用戶的瀏覽歷史信息洩露給 `Safe Browsing server`。你可能想 `proxy Safe Browsing` 檢查（這可以非常便宜地完成，因為沒有多少流量），但不代理一般的瀏覽流量（這是更高的流量，因此更昂貴）
- 這對瀏覽器來說很容易做到，因為它知道哪些流量是什麼，但對 `IP layer` 系統來說就比較困難了，它必須以某種方式區分不同類型的流量
- 這不一定是不可能的，但它的工作量明顯更大
- 例如，如果 Safe Browsing 使用一個獨立於 Google 其他部門的 IP，那麼你可以直接轉發該流量，但如果它共享同一個 IP，那麼你也會對人們的搜索流量進行加密

一些 IP 隱蔽系統在 `application layer` 進行中繼，包括
- `Tor`
- Apple 的[iCloud Private Relay](https://support.apple.com/en-us/HT212614)
- [Firefox Private Network](https://fpn.firefox.com/)

通常情況下，像這樣的系統被稱為 `proxies`
- Apple 的系統很有趣，因為它在 OS 中主要是通過 hooking 住 Apple 的  higher level networking APIs 來實現的
- 即便如此，它也只適用於 Safari，而不是其他 application

--------------------

## 4. 有多少 hops ?

無論採用何種中繼技術，到最後，中繼都需要向 server 發送流量
- 這意味著它必須知道你連接的是什麼 server
- 但這又產生了一個新的隱私問題: 你連接到中轉站，然後告訴它要連接到哪個 server
- 這意味著，雖然你已經阻止了 server 了解你的身份，但對於中轉站本身，你仍然有一個隱私問題
- 中轉站會有一些關於它如何處理這些信息的隱私政策（最好是完全不保留日誌）
- 但這只是你必須信任他們的東西。甚至更好的是有某種形式的技術保護

在這裡提供技術保護的標準方法是有多層中繼(multiple layers of relaying)，如下圖所示:
![](https://educatedguesswork.org/img/multi-hop-relay.png)  

這種工作方式是，client 連接到 `Relay 1`。然後 `Relay 1` 將其連接到 `Relay 2`
- 與我們的 `single-hop`(單跳)系統一樣，該 data 通過加密信道被發送到 `Relay 1`，並且本身也被加密到 `Relay 2`
- 然後，client 告訴 `Relay 2` 將其連接到 server
- 因此，到 server 的 data 被 client 以嵌套的方式加密了三次:
  - 一次到 server，然後到 `Relay 2`，然後到 `Relay 1`，每一跳都剝去一層加密，並將其傳遞給下一跳

其結果是，沒有任何一個實體（除了客戶端）可以同時看到用戶的身份和它所連接的 server 的身份。下面是每個人看到的內容:
|Entity|Knowledge|
| :---: | :--- |
|Relay 1|Client address, Relay 2 address|
|Relay 2|Relay 1 address, Server address|
|Server|Relay 2 address, Server address|
 
如果兩個中繼串通
- 他們可以一起發現 User 的地址和 server 的地址
- 如果任何一個都是誠實的，那麼客戶的隱私應該得到保護，因為這兩個都不能輕易地與 server 串通來了解這些信息
- `Relay 2` 因為它不知道 User 的地址，`Relay 2` 因為(希望) User 與 `Relay 2` 的連接是它在這段時間內與 `Relay 2` 的許多連接之一
- 最後這部分工作的好壞取決於系統的運行規模，client 將連接留到多長時間，它是否重複使用與 `Relay 2` 的連接來連接多個 server

為了使其發揮作用， Relay 需要由不同的實體操作。否則就無法保證不發生衝突
- 這包括不在同一個雲服務提供商（例如 AWS）上運行
- 有時你會聽到 [multi-hop VPNs](https://www.comparitech.com/blog/vpn-privacy/multi-hop-vpn/)
- 但如果是同一家公司提供兩個 VPN server，那麼這並沒有真正的幫助
- [iCloud Private Relay](https://support.apple.com/en-us/HT212614) 的一個很好的特點是，你的賬戶在 Apple 公司，但他們安排了不同供應商的多跳，所以你不需要擔心細節問題

Multiple hops 的一個重要限制是
- 對性能產生負面影響

一般來說，Internet 的路由算法試圖在兩個地點之間找到一條合理有效的路線
- 如果你不是在 A 點和 B 點之間路由，而是從 A 到 C 到 B，這將會有點慢
- 做的跳數越多，越有可能性能影響
  - 這不是一個精確的效果，但一般來說，應該預期會有影響

[iCloud Private Relay](https://support.apple.com/en-us/HT212614) 是一個[two hop network]（https://support.apple.com/en-us/HT212614）
- 第一跳由 Apple 公司運營
- 第二跳是 Apple 與之簽約的大型供應商（主要是 CDN，如 Cloudflare 或 Akamai）
- Apple 和這些 CD N都有快速的連接和良好的地理分佈，這是為了確保高性能

`Tor` 使用 three hops
- 一個 `guard node` (防護節點)
- 一個 `middle relay` (中間中繼)
- 一個 `exit node` (出口節點)
- Tor relays 實際上是志願者服務，因此在實際中 performance 各不相同

--------------------

## 5. 商業模式
典型的 VPN 有一個簡單的商業模式
- 你向 VPN 供應商付款，然後在連接時向他們認證（例如，用密碼）
- 這對隱私來說並不理想，因為他們知道你的姓名、聯繫資訊和信用卡號碼
- 另一方面，如上所述，他們已經知道你的 IP 和你要去的網站，所以不清楚這讓事情變得有多糟糕

然而，對於 `Private Relay`，這將造成一個真正的問題
- 第一跳中繼並不那麼糟糕，因為它無論如何都會得到你的 IP
- 但如果你用你的身份去驗證第二個中繼，那麼你就毀了一切，你還不如回到一個單跳系統
- 為了解決這個問題，Apple 使用 anonymous credentials (匿名證書)生成的 [blind signatures](https://en.wikipedia.org/w/index.php?title=Blind_signature&oldid=1088007463) 來驗證代理，如下所示


![](https://educatedguesswork.org/img/anonymous-proxy.png)  


簡而言之，其工作方式是
- Client 連接到 Apple，並使用其 iCloud 賬戶對其進行認證
- 然後 Apple 發出一個不包含用戶身份的匿名憑證。這個憑證可以提供給中轉站，以授權使用該服務
- 為了防止 Apple 將這兩種活動聯繫起來，生成憑證時，該憑證是保密的(基本上是加密的)
- 然後客戶端在將其發送給中繼站之前解除保密（關於這種憑證如何工作的更多細節，見 [here](https://educatedguesswork.org/posts/vaccine-passport-anon/#digression%3A-anonymous-credentials)）
- 這種設計允許代理知道你被授權使用服務，但不知道你是誰


Tor 與上述任何一種都不同
- 因為它是免費服務，由社區成員運營（你可以[donate](https://support.torproject.org/faq/relay-donations/)給運行中繼器的人）
- 這就造成了一些不可預測的 performance 後果，因為真的沒有什麼 `Service Level Agreement(SLA)`(服務水平協議)的方式
- 這也使得評估實際的隱私保證有些困難，因為一些 Tor 節點可能是由你不信任的人或積極惡意的人運行
- 顯然，對於 `iCloud Private Relay`，你必須自己判斷你對 Apple 及其合作夥伴的信任程度，但至少你對他們是誰有一些了解

--------------------

## 6. 總結
IP 是一個重要的、非常有效的跟踪載體
- 如果你想私下瀏覽，你需要做一些事情來掩蓋 IP
- 這主要意味著中繼。只要你的供應商不與 server 串通，任何中繼系統都會將你的身份從 server 那裡隱藏起來
- 任何一 hop system 必然意味著你相信供應商不會跟踪你的行為，也不會與 server 串通
- 根據你對你的本地網路及其隱私政策的看法，single hop system 可能是也可能不是一種改進
  - （參閱 Yael Grauer 在[消費者報告](https://www.consumerreports.org/vpn-services/mullvad-ivpn-mozilla-vpn-top-consumer-reports-vpn-testing-a9588707317/) 的文章）
- Multi-hop system 有更好的隱私，因為 single relay 的不當行為不足以損害你的隱私

系統工作方式的技術細節（主要是 IP 與 application layer）對隱私並不重要，但對功能卻很重要
- application layer 系統更靈活，但對你設備上的其他 application 提供的覆蓋面卻不完整
- 此外，所有 multi-hop 系統都在 `application layer`，所以作為一個實際問題，如果你想要一個 multi-hop 系統，你可能會使用 application layer system

最後，重要的是要知道
- 即使是最好的系統也只能提供有限的保護
- 一個對網路有完整看法的攻擊者通常可以做足夠的流量分析，以確定誰在流量的每一端
- 幸運的是，我們大多數人不需要擔心這種強大的攻擊者