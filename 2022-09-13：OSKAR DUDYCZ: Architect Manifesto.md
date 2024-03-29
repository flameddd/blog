# 2022-09-13：OSKAR DUDYCZ: Architect Manifesto
## OSKAR DUDYCZ, 2020-11-29
### https://event-driven.io/en/architect_manifesto/

多年來，已經創造了一套龐大的架構師亞類型--軟件架構師、企業架構師、解決方案架構師、首席架構師、DevOps架構師、初級架構師
- 架構師這個頭銜最近被濫用了

世界很少是完美的，團隊正在與[康威定律](https://zh.wikipedia.org/zh-tw/%E5%BA%B7%E5%A8%81%E5%AE%9A%E5%BE%8B)鬥爭
- 知識共享很少發生
- 保持一致性是個挑戰
- 設計會偏離切線
- 團隊成為自己的孤島
- DevOps 經常出現在只是改頭換面的運營團隊或持有 CI/CD 工具的人群中


我們中的一些人，包括企業，已經發現將我們的開發分成不同的工作
- 自主地工作，並不總是最好的方法
- Uber 從 `rewrite everything` 模式發展到基於「[面向領域的微服務架構](https://eng.uber.com/microservice-architecture/) 」的模式
- Matthew Skelton 和 Manuel Pais 提供了[團隊拓撲結構](https://teamtopologies.com/)的概念
- 以[減少認知負荷](https://www.youtube.com/watch?v=haejb5rzKsM)和康威定律的影響。我們發現，自主權可能並不像我們曾經認為的那樣是種好處


說了這麼多
- 我仍相信擁有架構師是有意義的
- 即使是在 DevOps 和 microservice development 文化中，我們可能不稱他們為架構師，但他們就是架構師
- 從我的角度來看，根據專案的規模和專案中的不同需求，應該由一群人組成，而不是依賴一個人
- 一個小組有能力解決許多相關的問題和領域，而不會在專案中造成瓶頸
- 潛在地，真正允許開發團隊在不對專案產生不利影響的情況下以更大的自主權工作。防止或至少減少康威定律的影響

我決定寫下我對這樣一個架構師團隊的預見
- 以我參與的一個專案為基礎的
- 它的規模和範圍似乎證明了這種需要
- 儘管它們可能看起來有點理想化，甚至有點天真，但在我看來，它們是一個值得進行的努力，甚至可能是我們應該爭取的東西

我把這篇寫作稱為「架構師宣言」:


------------------------------------


## 1. 架構團隊應該是一個對系統架構的形狀負責的小組，或者是一個對專案的成功負有直接責任的諮詢委員會
- 第一種選擇是最好的
- 因為諮詢委員會可能會導致更大的專案混亂和客觀模糊
  - 如，團隊是否必須或應該採用諮詢委員會的建議

## 2. 架構團隊應該提供明確的指導和文件
- 架構團隊應提供清晰的指導和文件（書面形式、圖表等）
- 說明整體的架構願景和影響最終確定上述解決方案的決定的假設
- 架構解決方案和與之相關的所有關鍵決定應該是透明的，並公開提供給所有開發者
- 架構團隊應與開發人員一起審查架構解決方案，以確保理解的清晰性
  - 如，使用哪種 DB，為什麼以及何時使用 event queue，unit test 以及如何寫 acceptance tests 等

## 3. 架構團隊應該有來自管理層的權力和授權。
- 由於公司可以有其他強大的、有經驗的開發人員，那麼它應該由管理層明確表達，並向開發人員解釋什麼是架構團隊的責任
- 互動和規則應該被清楚地定義
  - 如，「說開發團隊有義務在做新的設計時聯繫架構師團隊」，這將減少出現混亂和分歧的機會。


## 4. 架構團隊不應該阻礙團隊的創造力，也不應該成為設計過程中的瓶頸。
- 開發團隊應該對功能的設計負責
- 然而，他們應該在設計的早期階段與架構團隊協商，以確保符合架構，並及早發現設計中的任何明顯缺陷
- 如果開發團隊把架構團隊的設計審查推遲到最後的功能設計，那麼有時拒絕設計可能就太晚了
  - 因為已經在設計上投入了太多的精力。可能會造成功能設計上的混亂，並使人們處於防御狀態
- 在功能的設計過程中進行早期審查，可以使人更清楚地了解該功能在整個架構中的意圖

有了明確的建議和組織定義的最佳實踐，將有助於開發團隊提供一個功能設計，並正確地融入到架構的邊界
- 如果需要設計的功能過於復雜，那麼架構師可以作為團隊的一員加入開發團隊，而不是擔任領導角色
- 在設計工作中提供一個實踐性的工作關係


## 5. 架構團隊應該有一個正式的時間來進行架構工作
- 設計需要作為一項重點工作來構建，並且要有耐心地完成
- 應指派一名首席/領導架構師負責建築設計工作，並且是全職的、專門的貢獻者
- 被分配到設計的建築團隊的其他成員可以是兼職的，根據工作的需要來確定，並承諾一定的時間來做這項工作
- 時間分配應該包括與開發團隊在指定的功能工作上的直接合作，作為顧問以確保從架構計劃到實際功能開發的轉移

所有架構團隊成員的工作對所有的開發團隊都應該是透明的
- 架構團隊應該被分配一些專門的時間，每周至少一天，用於諮詢來自開發團隊或管理層的持續問題
- 架構團隊應該作為顧問，參與到每個待開發功能的計劃、安排和資源分配中。他們應該應該審查設計的當前狀態，並優先考慮需要的改變


## 6. 架構團隊應該與管理層、PM、PO 緊密合作，了解產品的優先級。
- 架構，或至少是一個工作的架構，不能在真空中創建，如果沒有設計的背景，要建立高品質的軟體是不現實的
- 架構，為了適用，需要有前瞻性，如果不知道未來會發生什麼，就不可能提供/提供持久的架構


## 7. 架構團隊應該積極主動地與開發人員合作，了解他們的痛點，解釋和介紹架構決策
- 例如，通過會議
- 他們還應該積極徵求開發人員的建設性批評意見

## 8. 架構團隊應該負責不斷審查所設計模式的應用
- 作為架構師和開發團隊的顧問，架構團隊應該審查設計，驗證 PR，確保單元測試的實施和完成，並對 codebase 進行隨機檢查
- 架構團隊還應該驗證功能是否被正確地記錄下來
- 在審查過程中，負責特定功能實現的架構師和架構負責人應該與開發團隊負責人一起討論架構中的變體和代碼庫的重複性和/或潛在的誤導性
- 架構團隊不應該扮演 code 警察的角色，也不應該成為軟件開發的阻力


## 9. 架構團隊不應假定一旦設計好的架構將永遠是最新的
- 架構團隊應該重新評估設計假設是否仍然適用
- 他們應該確保架構是與業務需求一起發展的

## 10. 架構團隊也應該在業務功能上下功夫
- 至少每隔一段時間進行一次
- 架構師如果不對他們設計的架構做任何定期的工作，往往不理解開發團隊的掙扎，也不能抓住痛點

## 11. 架構師團隊應該確保（與 DevOps 團隊一起）CI/CD 過程的流暢性
- 幫助有效地交付新功能（所以如果它是可靠的，足夠快的，安全的，等等）
- 還應該與系統團隊合作，了解 production 的 configurations，以便能夠提出建議並確保對 production debug 是有效的