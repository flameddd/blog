# 2024-12-21: 筆記 Ruoyu Sun 的 Game Networking Demystified.md

## Ruoyu Sun, 28 Mar 2019

### https://ruoyusun.com/2019/03/28/game-networking-1.html

---

之前有看兩篇 Ruoyu Sun performance 的文章，他有說那方面不是他的專長

- 他其實比較擅長遊戲的網路部分，所以我很好奇他這系列文章
- 隨便掃過一下，可能沒有實作的部分，都是概念解釋

---

討論 game networking 時，大家用不同的術語來描述網路架構

- 但大多數術語僅描述了架構的一部分
  - 在一個討論中，大家聊 Starcraft1 時，會說 `peer-to-peer`
  - 在另個討論中，可能會說 `lockstep`
  - 又另個討論中，會看到提 `deterministic(確定性)`

Game networking 非常複雜:

- 沒有 golden architecture
- 大多根據遊戲玩法和限制使用不同技術的組合

所以在這篇文章中，用直白的方式討論常用的術語，並討論背後的設計選擇及其影響

---

## Part I: State vs. input

`State vs. input`

- 是在設計遊戲如何通過網路執行時需要做出的**第一個重大決定**
- 要實現網路多人遊戲，在所有玩家之間傳送資料，以確保所有玩家擁有相同的 **game state**

問題是，要傳送什麼？

- 有兩種常見的方法，它們將顯著影響遊戲程式碼

---

## 方法 1: 傳 State

一個典型的例子:

1. `玩家 A` 傳 `MoveTo(1, 2)` 給`authoritative node`
2. `Authoritative` 收到請求，處理它並告訴所有玩家`玩家 A` 現在在 `(1, 2)`
3. `玩家 A` 收到回應並移到 `(1, 2)`
   - 其他玩家都收到相同的信息，他們螢幕上的`玩家 A` 也都移到 `(1, 2)`

這邊用 `authoritative node` 這個詞

- 大多人會稱為 server
- 因此，有時這被描述為 `authoritative servers` or `dumb clients` or `client-server architecture`

client 和 server 不一定是你所想像的

- 讓 client 和 server 的 code 在同一玩家上執行是很常見的
  - 通常稱為 `host`
- 當然可以將 server code 跑在真正的 server 上
  - 這個 server 與 client 是分開的
  - 在這情況下，server 基本上執行的是遊戲的 headless 版本

---

## 方法 2: 傳 Input

一個典型的例子:

1. `玩家 A` 傳 `MoveTo(1, 2)` 給所有玩家。
2. `玩家 A` 收到所有玩家的確認，並移到 `(1, 2)`。其他玩家的螢幕上的`玩家 A`也移到 `(1, 2)`

與前例子相比

- 主要區別在於不是傳送 A 的位置(game state)，而是傳送 A 的 input(移動)
- 在第一個例子中，狀態計算由 **一個** 來源，即 `authoritative node` (i.e. `host` or `server`)
  - 所有其他玩家都將其視為最終真相
- 在第二個例子中，狀態計算由所有玩家 **分別** 完成

因此，為了確保所有玩家擁有相同的 game state

- 遊戲邏輯需要是 `deterministic(確定性的)`
- 也就是說，給相同的輸入，就應該產生相同的 game state

這有時被稱為 `deterministic lockstep`

- 這令人困惑，因為即使這通常與 `lockstep` 技術一起用，但不進行 lock step 也是完全可以的
- 事實上，很多手遊因為 mobile network 的關係，而遊戲不進行 lock step

---

## 令人混淆的 `server`

在第二個例子中，我們假設了一個 peer-to-peer connection，然而

- 大多數現代遊戲使用中繼節點來轉發 input 給所有玩家，而不是用 peer-to-peer connection
- 一些人稱為 `relay server(中繼伺服器)`，但我避免使用 server 這術語

事實上，在第一個例子中，`client-server architecture`

- 我們也可以使用 peer-to-peer connection
- `client-server` 中的 `server` 是指 `authoritative node`，而不是真正的實體 server

感到困惑嗎？這有個簡單的規則:

- 忘掉所有的 client/server
- 這不是關於通訊模型(peer-to-peer, server-client or relay)
- 而是關於遊戲狀態同步

只需問一個問題:

- 為了保持遊戲狀態同步，傳的是什麼資料: 是 game state 本身還是 input ?

---

## 該用哪一個？

視情況而定。以下是簡要比較:

|                    | 傳送 State                                                    | 傳送 input                                                                                                       |
| :----------------- | :------------------------------------------------------------ | :--------------------------------------------------------------------------------------------------------------- |
| 網路流量           | 通常 **高**                                                   | 通常 **低**                                                                                                      |
| 重新連接           | 相對 **容易**，玩家只需從 `authoritative node` 取最新遊戲狀態 | 相對 **困難**。需要有個 `relay node` 來存所有 input。當重連時，所有 input 將被重新傳送並在 client 端重新跑過一遍 |
| Replay 的檔案大小  | 大                                                            | 小                                                                                                               |
| 防作弊             | 相對 **困難**，尤其是 `authoritative node` 在實體 server 上   | 相對 **容易**                                                                                                    |
| Deterministic 邏輯 | 不必要                                                        | 必須                                                                                                             |
| 常見遊戲類型       | MMO                                                           | RTS                                                                                                              |

---

## 作弊和防作弊

在任一種方法下，玩家都可以破解、作弊。下面討論三種常見的作弊方式以及防範

**傳 state**:

- 如果玩家不是 `authoritative node`，這通常不是問題，因為下次 state 同步時，所有被破解的數值都會被丟棄
- 如果玩家是 host，這將成為一個問題
- 常見的解決方案是將的 `authoritative node` 跑在實體 server 上，只有你能訪問

**傳 input**:

- 在這種情況下，玩家將會有不同步的遊戲體驗
  - 如作弊玩家在 local 遊戲中有 200 HP，但所有其他玩家將看到 100 HP
- 解決這問題有兩種常見方法:
  1. Sample 遊戲 state checksums 來偵測 `desync` 並踢出 desync player


      - 然而，你可能不想立即踢出玩家，因為如果你的遊戲邏輯是非確定性的，這很可能會發生(因為實現確定性很難，稍後會討論)
      - 一個實際的方法是只有當不同步超過閾值時才踢出玩家
  2. 重新演繹(Re-enact)和驗證 play session
     - 為此需要 relay server 來記錄所有 input 並 Re-enact play session
     - 這樣將得到一個沒有不同步和破解的結果
     - Re-enact 時，作弊者的遊戲體驗會非常混亂，因為 input 將不再有意義。但我們不要關心作弊的遊戲體驗
- 這兩種方法並不相互排斥。事實上，大多數遊戲用的是兩者的組合

---

## 玩家破解/移除戰爭迷霧

**傳 state**:

- 再次強調，首先需要將 `authoritative node` 放在實體 server 上
- 然後 `authoritative node` 需要變得更`聰明`
- 只傳送玩家應該知道的 game state
- 如果這樣做了，即使作弊者解除戰爭迷霧，狀態也不會被洩露

**傳 input**:

- 這時事情變得棘手了。因為 state 需要在本地計算
- 玩家知道所有的 game state
- 沒有好的方法可以防止作弊者揭露整個遊戲狀態
- 當然可以加一些 memory obfuscation 或 game binary protection 來增加難度，但這只是時間和努力的問題

---

## 玩家破解/用自動瞄準

在這種情況下

- 兩種模型都沒有防作弊的萬無一失的方法
- 因為作弊者利用的是已知的 local game state
- 唯一的希望是 binary protection, obfuscation, 檢測常用 plugin (有點像防病毒)和行為檢測

下面將討論 `deterministic` 和 `lockstep`

---

## Part II: Deterministic (確定性)

`Deterministic` 通常與 `lockstep` 一起出現

- 但實際上它們是兩個獨立的概念

在上面，我們指出，如果你想 `send only input`

- 那遊戲的邏輯需要是 `deterministic` 的

然而，反之則不然: 即使遊戲邏輯是 `deterministic` 的

- 你也不僅限於 `send input` 同步模型
- 許多用 `send state` 模型的遊戲，大量使用了 `deterministic`

---

## 什麼是 `Deterministic` (確定性)

- 當遊戲邏輯是 `deterministic` 時，意味著在給定相同的 input sequence 時，會在所有機器上產生相同的 game state
- 如果再次以完全相同的 input 玩遊戲，會得到完全相同的 game state

`Deterministic` 通常是件好事:

- 它啟用了許多無法通過非確定性邏輯實現的可能性
- 那為什麼不是所有的遊戲邏輯都是 `deterministic` 的呢？因為**實現 `deterministic` 很難**

---

## Deterministic is Hard

實作 `deterministic` 遊戲邏輯時的一些常見挑戰:

- **浮點數**:
  - 在不同機器上進行相同的浮點數計算會得到不同的結果
  - 因此，為了確保 `deterministic`，你必須避免使用浮點數。但這並不容易
  - 浮點數是引擎、 middleware, library 中無處不在的基本數字類型
  - 對於邏輯中的計算，通常可以用 fixed point library
- **確定性的隨機數**:
  - 這相對容易。當 game session 開始時，每個玩家得到相同的 random seed
  - 這確保他們所有的隨機性是確定性的
- **有序資料結構**:
  - 在大多數語言中，hash(or map or dictionary)中的 order 不是保證的
  - 因此，掃描 hash 應該仔細檢查
  - 並且應該偏好像 `SortedDictionary` 這樣的結構
  - 這相當微妙且難以追蹤
- **執行順序**:
  - 對於許多遊戲引擎，更新 callback order 是 `non-deterministic` 的
  - 例如，在 Unity，你應該將遊戲邏輯分散在不同的 `Monobehaviors` 中
    - 因為它們的更新順序不是保證的
      另外，通常應避免使用 Unity `coroutine`
- **物理引擎**: -大多數物理引擎是 `non-deterministic` 的(因為浮點數或其內部架構)
  - 這並不意味著完全不能使用它們
  - 你只能將 `non-deterministic` 的物理引擎用於 `display`
  - 例如，一款 3D 即時戰略遊戲，你的 deterministic 碰撞檢測僅基於 `x`, `y` 軸(例如，沒有飛行單位)。那麼你可以自由地在 `z` 軸上使用物理引擎(例如，爬坡)
- **分離 logic 和 view**:
  - logic 一般避免依賴 view(例如動畫)
  - 例如
    - 當開發玩家技能系統時，一種方法是當技能被使用時將 state 改為 `USING_ABILITY`，並播放動畫
    - 當動畫結束時，callback 將 state 改為 `IDLE`
  - 然而，這意味著 logic 依賴於 view，這通常是 indeterministic
  - 因此，必須這樣實作:
    - 當玩家用技能時，state 改為 `USING_ABILITY`，並告訴動畫系統播放動畫
    - 經過 10 個邏輯滴答後，將 state 改為 `IDLE`，不管動畫系統的狀態
    - 另外，logic 以不同於 rendering frames 的頻率執行是非常常見的
    - 許多遊戲以 `10Hz`(或 `10FPS`)的頻率執行 logic tick，而以 60Hz 的頻率 rendering tick

---

## 那為什麼要確定性？

如之前所提，如果想用 `send input only` 模型進行同步

- 那遊戲邏輯需要是 `deterministic`，否則會導致不同步
- 即使用 `send state` 模型，如果能使遊戲邏輯 `deterministic`，仍然有好處
  - 如，用於處理網路延遲的 client 預測和 rollback 變得更好，遊戲玩法也變得不那麼容易出現故障

---

## 追蹤和除錯

追蹤不確定性(不同步)就像追蹤 memory leaks，甚至更糟

- 至少 memory leaks，還有 memory profilers 和其他工具可以幫助
- 要追蹤你的 deterministic 邏輯，幾乎需要自己開發工具

一個值得投資的特別有用的工具是 logic frame dump

- 也就是，每一 frame 轉存成 game state
- 然後你可以比較不同玩家的 dumps
- 甚至重新執行你的邏輯以產生 dumps 作為基準
- 甚至可以進行一些自動化測試，多次重新執行相同的 game input，並 assert 所有 frame dumps 應該完全相同

由於 desync 很難重現

- 因此知道它在實際 game session 中發生的頻率也是有用的
- 你可以為每次遊戲產生 logic frame dump 並發到 server，但玩家可能會抱怨網路用量
- 相反，考慮產生 checksum
  - 如果每一 frame checksum CPU 而言太昂貴
  - 可以對 frame checksum 做 sample
  - 這種方式，可以了解 desync 在野外的嚴重程度，並確定修復它們的優先級

---

## Part III: Lockstep(同步步驟)

`Lockstep` 本質上是

- 遊戲邏輯只有在收集到所有 client 的 input 後才會前進
- 從而保證所有 client 看到相同的 game state

這使得遊戲邏輯類似於回合制遊戲

- 想像回合制遊戲，每個玩家必須按下結束回合按鈕，遊戲才能進入下一回合
- 現在，想像每個回合持續 1/60 秒(或實際一點，1/10 秒)，這就是 `lockstep`

---

## 區網的多人遊戲(LAN)

早期多人遊戲，大部分是局網(LAN)的

- 這有相對較低且穩定的延遲(即沒有峰值)
- 這適合 `lockstep` 模型
- 我們可以安排在 N 回合收到的所有 input 在 N+2 回合執行，並期望網路往返在 N+1 時完成
- 如果遊戲邏輯每 100ms 執行一次，那在 LAN 中這是一個非常合理的期望

缺點是，如果**一個玩家**的網路無法在截止時間內完成

- 那**所有玩家**的遊戲都會暫停，並等待那個慢的玩家
- 所以每個人的體驗很大程度上取決於最慢的玩家

---

## `Lockstep` 和 `State vs. Input`

`Sending input` + `lockstep` + `deterministic logic` 是個非常流行的組合

- 然而，這並不意味著不能在 `sending state` 模型中用 `lockstep`
  - (通常稱為 client/server model，但這邊避免用這個術語)
- 事實上，一些遊戲這樣做:
  - `authoritative node` 只會在收到並處理所有 input 後才傳送 game state
  - 這樣的好處之一是**公平**，所有玩家都處於平等的地位

---

## 延遲

在網路上用 `lockstep` 時

- 最簡單的方法是增加 input processing delay
  - 在 N 回合收到的 input 將在 N+6 執行
  - 然而，這會讓遊戲感覺`延遲`
- 有些 presentation layer tricks 可以隱藏
  - 例如，當玩家放技能時，在施放前加一個施放動畫
  - 動畫會立即播放，但實際傷害(即 game state change)會在稍後的 tick 中執行
  - 甚至可以根據網路延遲動態調整延遲
  - 所以網路較差的玩家會提前發送訊息來彌補這點，所有玩家都得到相同的延遲。

這個想法是用 presentation layer (visual effects, particles, animations) 為玩家動作提供直接的**反饋**

- 而真正的 game state 會在稍後變更
- 然而，這招不一定能針對所有情境，取決於遊戲類型
- 而且，如果網路有隨機 spikes，這方法效果不佳，這在 mobile network 上很常見

---

## 峰值

Visual tricks 無法幫到應對 Spikes

- 因為 `lockstep` 會將一個玩家的 Spikes 傳給所有玩家
- 這對於 mobile game 尤其麻煩

一種方案是設置 timeout

- `authoritative node`(或 authoritative tick coordinator)將在 timeout value(例如 100ms)內前進
- 而不是等待所有玩家的 input
- 如果 miis 一位玩家的 input，遊戲邏輯基本上會將其視為`此回合無輸入`
- 在這情況下，延遲的玩家將會有延遲的體驗，而其他人不受影響

對於`傳送輸入`模型，滴答協調器通常是中繼伺服器。對於`傳送狀態`模式，它是權威節點。

---

## Part IV: Server 和 Network Topology (網路拓撲)

實體伺服器

- 首先弄清楚術語
- 這邊提到的 server 是指`實體伺服器`。不要與 `client/server` 術語混淆
  - `client/server` 是遊戲同步模型，`client/server` 的 `server` 實際上可以是其中一位 client(即 host)。所以前面一直用 `authoritative node` 這名詞

---

## `Peer to Peer` vs. `Central Server`

Whichever synchronization model is chosen (sending state vs. sending input), we can use either network topology. In fact, we can even allow both. Normally, games will only choose one and here’re some considerations.

## 點對點 vs. 中央伺服器

無論選擇哪種同步模型(`sending state` vs. `sending input`)，都可以使用任何一種網路拓撲

- 事實上，甚至可以允許兩者並存
- 通常遊戲會選擇其中一種，以下是一些考量。

|                        | P2P                                                            | Server                                                             |
| :--------------------- | :------------------------------------------------------------- | :----------------------------------------------------------------- |
| `Sending Input` 的成本 | 幾乎沒有成本。然而，可能需要一個伺服器來進行 NAT punch-through | 相對便宜。server 主要轉發網路封包                                  |
| `Sending state` 的成本 | 同上                                                           | 相對昂貴。server 需要執行遊戲邏輯                                  |
| 防作弊                 | 無防範措施                                                     | 很好的防護                                                         |
| 網路延遲               | 依賴於玩家間的連接，我們無法控制                               | 依賴於到伺服器的連接。可以設立區域伺服器並將玩家路由到最近的伺服器 |

---

## NAT Punch-through Server (穿透伺服器)

即使選擇 `peer-to-peer` 網路拓撲

- 可能仍需要一個最小的 server 來進行 NAT Punch-through
- `NAT Punch-through` 基本上是個允許處於路由器或防火牆後面的玩家直接互相通訊的技巧
- 運行這個 server 的成本非常低: 玩家建立直接連接後，便不再需要 server
  - https://en.wikipedia.org/wiki/Hole_punching_(networking)

---

## Relay Server (中繼伺服器)

`Relay server`:

- 部署在多個區域的 relay server 可以降低網路延遲
- 對於 `sending input` 模型，唯一需要的 server 是 relay server
  - 因為遊戲邏輯是在每個玩家的 client 計算的
- Relay server 也可以作為 tick coordinator

---

## 遊戲邏輯 server

只有 `sending state` model 需要 Game logic server

- 它是計算 game state 並傳送給所有玩家的 `authoritative node`
- 開發 game logic server 的最簡單方法是做遊戲的 headless 版本
  - 但這樣執行成本可能非常高
- 另種方法是將 game logic 寫為可以獨立執行的模式，這樣 scale 起來可能更便宜

---

## 其他類型 server

除了上述的三種類型的多人遊戲 server 外，還可能需要其他幾種類型:

- **Matchmaking server(配對伺服器)**:
  - 用於配對玩家並組成 game session。配對完成後，玩家可以通過 server 或用 `peer-to-peer` 通訊
  - 在 `peer-to-peer` 的情況下，`matchmaking server` 可以作為 NAT punch-through server 的擴展
- **\*Player identity server(玩家身份伺服器)**:
  - 有時我們可能希望在 server 上存 User 資料
    - UserID, level, 擁有的遊戲內物品等
    - 這通常與遊戲類型無關，通過 HTTP API 即可

---

## 第五部分: 內插(Interpolation)和 Rollback

隱藏網路延遲

- 儘管近年來網路速度大幅提高，但延遲仍然是一個問題
- 就像即時渲染在於`儘可能地作弊`一樣，netcode latency 的緩解遵循相同的原則
- 最簡單的方法是忽略延遲，專注於用`視覺技巧`來隱藏它

前面有提到用動畫來隱藏延遲

- 然而，最常感知到的延遲是玩家移動
- 最簡單的技巧是允許玩家的視覺位置與邏輯位置略有偏差
- 當接收到移動 input 時，玩家的視覺位置會立即移動，而邏輯位置在 server 確認時移動
- 對於移動速度相對較慢且不需要非常精確和公平的碰撞檢測(即非電競)的遊戲，這通常已經足夠

這個技巧的美妙之處在於它非常簡單:

- 增加的複雜性很小，不會增加 server 計算量。這是減輕網路延遲的 low-hanging fruit

---

## Client 的內插(Interpolation)

對快節奏的電競遊戲，上面的技巧效果不佳:

1. 玩家移動越快，視覺和邏輯位置的偏差越大
2. 碰撞檢測不準確(這對射擊遊戲是致命缺陷)

Client interpolation 是種解決這個問題的技術

- 核心思想是顯示「當下的 local 玩家」和「過去的 remote 玩家」
- 當接收到 local 玩家的 input 時，立即反映
- 而 remote 玩家的位置在從 server 接收時更新(因此是在 past)

缺點是

- 碰撞檢測(命中掃描)變得非常複雜
- 因為每個玩家對遊戲的視角不同，server 在檢查碰撞時需要重建每個玩家的視角
- 例如，當判斷 `player A` 是否射到 `player B` 時，server 需要回溯到 `player A` 的遊戲視角
  - 即 `player A` 在 T 時間點，`player B` 在 T-3(假設延遲為 3)

這也引入了一個潛在問題

- 即在 `player B` 的視角中(`player B` 在 T 時間點，`player A` 在 T-3)，玩家可能會覺得沒射中
- 在實際操作中，這問題較小，因為玩家更容易判斷自己是否射中目標，而不是判斷自己是否被射中

Client interpolation 的另個缺點是近戰碰撞檢測(相對於命中掃描)變得非常棘手

- 這就是為什麼不會看到格鬥遊戲用這方法
- 這技術最常見於競技射擊遊戲

---

## Rollback

核心思想是:

- 我們簡單地預測 remote 玩家的 input，並用它來推進 game state，而不用等待 server 確認
- 如果預測錯誤，rollback 到最後確認的 game state，並從那裡進行新的預測

Rollback networking 最困難的部分不是 netcode

- 事實上，netcode 的實作非常簡單
  1. 保存每個預測 frame 的副本
  2. 當 server frame 到達時(`T-3`，假設延遲為 3)
  3. 我們將其與我們的預測(`T'-3`)進行比較，通常用 checksum
  4. 如果不同，我們採用 server state(`T-3`)，替換我們的舊預測(`T'-3`)
  5. 並基於 `T-3` 進行新的預測(`T-2`, `T-1`, `T`)
- 最棘手的部分是視覺:
  - rollback netcode 要求視覺能夠比較無縫地從一個狀態過渡到另一個狀態
  - 即 `View = f(state)`
  - 在實際操作中，這非常困難
  - 考慮遊戲中的所有 view states: 動畫、燈光、物理、UI
    - 要求所有這些子系統回溯(rewind)到新狀態並不容易，特別是如果遊戲引擎不是為此設計的(大多遊戲引擎不是)

即使遊戲的畫面能夠從 A 過渡到 B，也不可避免會有視覺故障

- 例如，如果預測 Object A 已經被摧毀但實際上沒有，那玩家會看到`死去的對象復活`
- 有些技術可以減輕這種情況(如，在收到 server 確認後才摧毀對象)，但很難做到完美

另個明顯的問題是預測 remote input

- 隨玩家數量的增加，預測準確性會下降，最終預測很少正確，rollback 幾乎總是會發生
- 這可能沒有你想的那麼糟。如果 prediction window 很小，誤預測只會導致小的差異，那麼修正(即 rollback 和重新預測)幾乎是不可察覺的
- 實際上，誤預測的頻率比 prediction window 的影響要小
- 即增加 prediction window 會導致更明顯的視覺故障(這很大程度上取決於遊戲)
- 解決這個問題的方法是增加 input delay
  - 這減少了 prediction window，並有助於防止一些最嚴重的視覺故障
  - 但代價是減少遊戲的 responsiveness

---

## Recommended Readings

- Gabriel Gambetta has a series of articles on client prediction for Client/Server architecture.
  - https://www.gabrielgambetta.com/client-server-game-architecture.html
- Valve’s Source Multiplayer Networking is also a good read.
  - https://developer.valvesoftware.com/wiki/Source_Multiplayer_Networking

另外從文章連結的相關連結一路找到這

- https://github.com/0xFA11/GameNetworkingResources

---

## Part VI: 遊戲類型和常見問題

Game networking 領域沒有銀彈

- 解決方案很大程度上取決於遊戲類型和遊戲設計

---

## 即時戰略(RTS)

在 RTS 中

- 玩家通常會控制大量單位。因此，`sending state` 幾乎是不可能的
  - 因為 game state 包含所有單位，實在太大了
- 幾乎所有老派的 RTS 都採用 `deterministic lockstep` 模型
- 然而，`lockstep` 在 mobile network 上可能會出問題，因為任何 spike 都會導致延遲
- 幸運的是，老派的 RTS 並不適合 mobile
  - 在小型觸碰螢幕上控制大量單位並不現實

大多數現代的 `lockstep` 模型，尤其在 mobile 上，不再是嚴格的 lockstep

- 有些用 `client prediction and rollback` 模型
- 有些則用 `server coordinated timed step` 而不是 `lockstep`

---

## MOBA

這裡變得有趣了。無論是 `sending input` 還是 `sending state` 都可以。這兩種模型都有的成功遊戲:

- `Dota 1` 是魔獸爭霸 3 的自定地圖，`sending input` 模型
  - 而魔獸 3，為 RTS，用 `deterministic lockstep`
- `Dota 2` 用 Valve 自家的 `Source 2` 引擎，`sending state` 模型(也稱為 Client/Server)
- `Heroes of the Storm` 用與星海 2 相同的引擎，`sending input` 模型
- 騰訊的 mobile MOBA 傳說對決 用 `Unity` 引擎並採用 `sending input` 模型
  - 即使用 `sending state` 也是完全可行的
  - 我猜 `sending input` 需要較低的頻寬，這在 mobile 上效果更好

判斷遊戲是否使用 `sending input` 或 `sending state` 的方法是通過 reconnection

- 如果幾乎可以立即重新連接到遊戲，則很可能用的是 `sending state`
  - 因為 client 只需從 server 取最新的 game state snapshot
- `sending input`，server 需要將所有 miss 的 input 發到 client
  - client 需要重新模擬遊戲，這需要一段時間

---

## FPS

FPS 通常用 `sending state` 模型，因為:

- game state 通常足夠小，可以傳送
- 在 server 執行重要的遊戲邏輯(如子彈命中)可以減少作弊
- 大多數 FPS 具有重度物理互動，通常很難使物理行為確定性

快節奏的多人 FPS 遊戲難以實現

- 為了保證流暢和公平的體驗，需要許多技術和權衡
- 如果地圖很大且有很多玩家，會更加棘手
  - 在這情況下，需要在 server 上實現剔除算法(例如基於距離)，以便 client 只取它需要知道的內容

---

## MMO

MMO 遊戲幾乎都是 `sending state`

- 因為 game state 必須在 server 上 persisted
- 而且，由於每個玩家通常控制一個角色，因此與 `sending state` 相比，`sending input` 不會節省多少網路

最大挑戰是 game state 所需的大量資料

- 顯然，我們需要一些興趣管理(即 `culling(剔除)`)
- 例如，我們通常不關心離我們很遠的玩家，因此 server 可以從 state update 中排除這些玩家

另個挑戰是世界的大小。如果整個地圖太大，一個 server 無法處理

- 需要將地圖劃分成 `tiles(瓦片)` 或 `areas(區域)`
  - 每個 server 處理一個區塊的遊戲邏輯
- 還需要另個 server，根據玩家位置將玩家轉到相應的 `tile server`，並在玩家從一個 `tile server` 移動到另個時可能傳送玩家資訊

MMO 的玩法非常多樣化:

- 可能需要根據不同需求混合用 server
- 如，專門聊天的 chat server
- 有些還有專門的 login server for authentication

---

## TCP vs UDP ?

一般來說，`UDP` 比 `TCP` 更受青睞

- 一個重要的原因是 `TCP` 的 packet loss recovery 並不是為遊戲設計的
- 大多遊戲可以處理一定程度的 packet loss
- 然而，TCP 為 reliable protocol，會嘗試重新傳送任何遺失的 packet，因此減少網路的吞吐量

對於快節奏的遊戲，與預設的 TCP 實作相比，自定義的 packet loss recovery 會更好

- 有幾種協定在 `UDP` 之上提供可靠性層:
  - ENet: http://enet.bespin.org/
  - RakNet: https://github.com/facebookarchive/RakNet
  - KCP: https://github.com/skywind3000/kcp/blob/master/README.en.md

然而，如果遊戲不是那麼快節奏玩法，可以接受 TCP，也可以考慮用它

- 因為 TCP 非常廣泛，整個生態系統非常成熟，節省大量開發時間

---

## Server 的程式語言？

Server 可以用不同的語言完成。然而，用跟 client 相同的有些好處:

- 如果 server 有大量的遊戲邏輯(例如，`sending state `模型)，我們可以 re-use code
  - 在某些情況下，game server 可以是 client 的 `headless` 版本
  - 對於 client 和 server 的工程師來說，用相同的語言更容易協作
- `C++` 作為傳統的遊戲 client programming language，也可用於 server
  - 然而，沒有 GC 的低階語言，從 server 來看，它更具挑戰性
  - `C#` 為 `Unity` 的 script language，擁有更大且更成熟的生態系統來開發可擴展的 server
    - 特別是用 `.Net` Core

然而，使用不同的語言來開發 game server 完全沒問題

- 特別是當你不需要 re-use client code 或你的 client 語言沒有成熟的 server 生態系統
- 在這情況下，成熟的生態圈會節省更多時間
