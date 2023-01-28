# 2022-12-01：The C4 model for visualising software architecture
### https://c4model.com/

---------------------------------------

如何用所謂的 4C (Context, Containers, Components, and Code) 來描繪軟體架構  

---------------------------------------

## 簡介

軟體行業，有統 `UML`、`ArchiMate` 和 `SysML`
- 但這些是否提供了溝通軟體架構的有效方式，往往是無關緊要的，因為許多團隊已經把它們扔掉了
- 轉而使用更簡單的方框和線條的圖
- 拋棄這些建模語言是一回事，但也許在追求敏捷性的比賽中，許多軟體開發團隊已經失去了視覺交流的能力

![](https://c4model.com/img/sketch-2.jpg)  

---------------------------------------

## 建構 Code map
`C4 模型`
- 是種幫助軟體開發團隊描述和交流軟體架構的方式
- 無論是在前期的設計會議還是在回顧性地記錄現有的程式庫時。它是一種創建不同層次的程式碼地圖的方法
- 就像用 google map 放大和縮小 User 感興趣的區域一樣

![](https://c4model.com/img/map-1.jpg)  


儘管 `C4 模型` 主要針對軟體架構師和開發人員
- 但它為軟體開發團隊提供了方法，在進行前期設計或回顧性地記錄現有程式碼庫時，可以在不同的細節層次上有效地溝通他們的軟體架構
- 向不同類型的受眾講述不同的故事

![](https://c4model.com/img/c4-overview.png)  

`C4 模型`是種`抽象優先`的軟體架構圖表方法
- 基於反映軟體架構師和開發人員如何思考和構建軟體的抽象
- 少量的抽象和圖表類型讓 `C4 模型`易於學習和使用
- 注意！你不需要使用所有 4 個層次的圖表；只需要使用那些增加價值的圖表
  - 對於許多軟體開發團隊來說，`System Context 圖`和 `Container 圖`已經足夠了。

---------------------------------------

## Abstractions
為了創建這些 code 的圖
- 首先需要一套通用的抽象，以創建一種無處不在的語言，可以用它來描述軟體系統的靜態結構
- `C4 模型`認為軟體系統的靜態結構是以**容器**、**組件**和**程式碼**為單位。而**人**使用我們構建的軟體系統(Software System)

![](https://c4model.com/img/abstractions.png)  

---------------------------------------

### 人
- `Person` 代表你的軟體系統的一個人類用戶（如 `actors`, `roles`, `personas` 等）

---------------------------------------

### 軟體系統 (Software System)
- 是最高 level 的抽象
- 它描述的是為 User 提供價值的東西，無論他們是否是人類
- 這包括你正在建模的軟體系統，以及你的軟體系統所依賴的其他軟體系統（反之亦然）
- 在許多情況下，一個軟體系統是由一個軟體開發團隊「擁有」的

---------------------------------------

### Container (applications 和 data stores)(這不是指 docker)  


在 `C4 模型`中
- 一個 `Container` 代表一個 `application` 或一個 `data stores`
- `Container` 是需要運行的東西，以便整個軟體系統能夠工作

實務上，一個 `Container` 可能會是一個:

- **Server-side 的 web application**:
  - Apache Tomcat 上的 Java EE web application
  - MS IIS 上的 ASP.NET MVC application
  - WEBrick 上的 Ruby on Rails
  - Node.js application
  - (這些都是)
- **Client-side 的 web application**:
  - Browser 中所執行的 JS application 如 Angular, Backbone.JS, jQuery 等等
- **Client-side 的 desktop application**:
  - WPF 寫的 Windows desktop application
  - Objective-C 寫的 OS X desktop application
  - JavaFX 寫的跨平台  desktop application 等等
- **Mobile app**: iOS app, Android app 等等
- **Server-side console application**:
  - 一個獨立的 (如 "public static void main") application
  - 一個批次處理程序等等
- **Serverless function**:
  - 一個 serverless function (如 Amazon Lambda, Azure Function)
- **Database**:
  - A schema or database in a relational database management system
  - document store
  - graph database
  - 像是 MySQL, Microsoft SQL Server, Oracle Database, MongoDB, Riak, Cassandra, Neo4j 等等
- **Blob 或 content store**:
  - blob store (如 Amazon S3, Microsoft Azure Blob Storage)
  - CDN (如 Akamai, Amazon CloudFront, etc)
- **File system**:
  - 一個完整的 local file system 或
  - 更大的網絡文件系統的一部分（如 SAN、NAS 等）
- **Shell script**:
  - 如，一個用 Bash 寫的 shell script

`Container` 本質上是一個上下文(context)或邊界(boundary)
- 一些程式碼在其中被執行或一些資料被存儲
- 每個 `container` 是一個單獨的可部署/可運行的東西或運行時環境，通常（但不總是）在自己的 process space 中運行
- 正因為如此，`container` 之間的溝通通常採用 inter-process communication 的形式


---------------------------------------


### Component

在 `C4 模型`中，`Component` 是一組被封裝在一個明確定義的 `interface` 後面的相關功能
- 像 Java 或 C# 這樣的語言，對 `component` 最簡單的理解就是 interface 後面的實作 classes 集合
- 至於這些 `component` 是如何打包的（如，每個 JAR, DLL, shared library 中的一個 component 與多個 component）是個獨立/orthogonal 的問題。


需要注意的是
- 一個 `container` 中的所有 `components` 通常在同一個 process space 中執行
- 在 `C4 模型`中，`components` **不是可被單獨部署的單元**


---------------------------------------

## Core 圖
視覺化的抽象層次(hierarchy of abstractions)
- 通過創建一個 `Context`, `Container`, `Component` 和（可選）`Code`（如UML類）圖的 collection 來完成的
- 這就是 `C4 模型`的名字由來。


---------------------------------------

## Level 1: System Context 圖
`System Context 圖`對於 `software system` 的`畫圖`和`建構文件`是很好的起始點
- 可以讓你退一步看清全局
- 畫一張圖，把你的系統作為一個盒子放在中心，周圍是它的 User 和跟它相互依賴/溝通的其他系統
- 細節在這裡並不重要，因為這是系統的 big picture
- 重點應該放在人(`actors`, `roles`, `personas`, etc) 和 `software systems` 上，而不是技術、協議和其他低級別的細節
- 這是種**向非技術人員展示的圖表**
- **範圍**: 一個單一的軟體系統(software system)
- **主要元素**: 範圍內的 software system
- **支持元素**: 與範圍內的 `software system` 直接相連的人(如 `User`,`actors`, `roles`, `personas`)和軟體系統（外部依賴關係）
  - 通常，這些其他的軟體系統位於你自己的軟體系統的範圍或邊界之外，你對它們沒有責任或所有權。
- **預期的受眾**:每個人，包括技術和非技術的人，軟內/外部體開發團隊
- **是否推薦給大多數團隊**: YES

![](https://static.structurizr.com/workspace/76748/diagrams/SystemContext.png)  
![](https://static.structurizr.com/workspace/76748/diagrams/SystemContext-key.png)  

---------------------------------------

## Level 2: Container 圖
一旦你了解了你的系統如何融入整個 IT 環境，下一步真正有用的是用 `Container 圖`來放大系統的邊界
- 一個 `Container` 是像 server-side web application, single-page application, desktop application, mobile app, database schema, file system 等等
- 從本質上講，一個 `Container` 是一個單獨的可運行/可部署的單元（如，一個單獨的 process space），它執行程式碼或存儲資料
- `Container 圖` 顯示了軟體架構的 high-level 形態，以及責任如何在其中分配
- 它還顯示了主要的技術選擇以及 `container` 之間的通信方式
- 這是一個簡單的、以 high-level 技術為重點的圖表，對軟體開發人員和 support/operations 人員都很有用
- **範圍**: 一個單一的軟體系統(software system)
- **主要元素**: 範圍內的 `software system` 的 `containers`
- **支持元素**: 與 `containers` 直接相連的 `people` 和 `software system`
- **預期的受眾**: 軟體開發團隊內/外部的技術人員；包括軟體架構師、開發人員和 support/operations 人員
- **是否推薦給大多數團隊**: YES

備註: 這張圖沒有提到部署方案、集群、複製、故障轉移等

![](https://static.structurizr.com/workspace/76748/diagrams/Containers.png)  
![](https://static.structurizr.com/workspace/76748/diagrams/Containers-key.png)  

---------------------------------------

## Level 3: Component 圖
接下來你可以放大並進一步分解每個 `container`，以確定主要的結構構件及其相互作用。

`Component 圖`:
- 展示一個 `container` 是如何由 `components` 組成的
- 這些 `component` 各自是什麼，它們的職責和技術/實作細節
- **範圍**: 單一一個 `container`
- **主要元素**: 範圍內的 `container` 內的 `components`
- **支持元素**: (在軟體系統範圍內的) `Containers` 加上與 `components` 直接相連的 `people` 和 software systems
- **預期的受眾**: 軟體架構師和開發人員
- **是否推薦給大多數團隊**: NO，只有當你覺得 `component 圖`能增加價值時，才會建立這張圖，並考慮自動創建 `component 圖`以獲得長期的文件

![](https://static.structurizr.com/workspace/76748/diagrams/Components.png)  
![](https://static.structurizr.com/workspace/76748/diagrams/Components-key.png)  

---------------------------------------

## Level 4: Code 圖
最後，你可以放大每個 `component`，以顯示它是如何用程式碼實作的
- 使用類似 UML class/entity/relationship 圖來呈現
- 這張圖是個 optional 的細節層次
- 理想情況下，這個圖是用工具(如 IDE 或 UML 工具)自動生成的
- 你應該考慮只顯示那些允許你講述你想講述的故事的屬性和方法
- 除了最重要或最複雜的 `component` 之外，其他 `component` 不建議使用這種詳細程度的圖
- **範圍**: 一個 `component`
- **主要元素**: `component` 範圍內的 code elements (如 classes, interfaces, objects, functions, database tables, 等)
- **預期的受眾**: 軟體架構師和開發人員
- **是否推薦給大多數團隊**: NO，對於長期存在的 document，大多數 IDE 可以按要求生成這種程度的細節

![](https://c4model.com/img/class-diagram.png)  

---------------------------------------

## Supplementary (補充)圖
一旦你對靜態結構有了很好的了解，你可以補充 C4 圖來顯示其他方面的內容

---------------------------------------

## System Landscape (系統景觀)圖
`C4 模型`提供了一個單一軟體系統的靜態視圖
- 但在現實世界中，系統從來不是孤立存在的
- 如果你負責一個軟體系統的集合，了解所有這些軟體系統在企業範圍內是如何配合的，往往是很有用的
- 要做到這一點，只需在 `C4` 的「上面」加另一個圖，從 IT 的角度來展示 system Landscape
  - 和系統背景圖一樣，這個圖可以顯示組織邊界、內/外部 User、內/外部系統
- 從本質上講，這是在企業層面上的軟體系統的 high level map
- 並對每個感興趣的軟體系統進行 C4 深入分析
- 從實用的角度來看，實際上只是一個系統的 context 圖，沒有具體關注某個特定的軟體系統
- **範圍**: 一個企業
- **主要元素**: 與企業相關的人員和軟體系統的範圍
- **是否推薦給大多數團隊**: 軟體開發團隊內部/外部的技術人員和非技術人員

![](https://static.structurizr.com/workspace/28201/diagrams/SystemLandscape.png)  
![](https://static.structurizr.com/workspace/28201/diagrams/SystemLandscape-key.png)  

---------------------------------------

## Dynamic 圖
當你想展示靜態模型中的元素在運行時如何協作以實現 user story, user case、功能等時，dynamic 圖會很有用
- Dynamic 圖是基於 UML `communication` 圖（`UML collaboration diagram`）
- 它類似於 UML `sequence` 圖，但它允許自由安排圖中的元素，用編號的交互來表示順序
- **範圍**: 一個企業、`software system` 或 `container`
- **主要/支持元素**: 取決於圖的範圍；企業(`System Landscape` 圖)，軟體系統(`System Context`/`Container` 圖)，`Container`(`Component` 圖)
- **預期的受眾**: 技術人員和非技術人員，軟體開發團隊內/外部

![](https://static.structurizr.com/workspace/76748/diagrams/SignIn.png)  
![](https://static.structurizr.com/workspace/76748/diagrams/SignIn-key.png)  

---------------------------------------


## Deployment (部署)圖
部署圖
- 允許你說明靜態模型中的 `software systems` 和/或 `containers` 是如何被映射到 infrastructure 的
- 部署圖是基於 UML 部署圖的，稍微簡化顯示 `containers` 和部署節點之間的映射
- 部署節點是物理基礎設施、虛擬化基礎設施(如 IaaS、PaaS)、容器化基礎設施(如 Docker)、執行環境(如 DB server, Java EE Web/application server, Microsoft IIS)等
  - 部署節點可以是嵌套的
- (你可能還想包括基礎設施節點，如 DNS services, load balancers, firewalls 等)
- **範圍**: 一或多個 `software systems`
- **主要元素**:  Deployment nodes, `software system`/`container` instances
- **支持元素**: 用於部署 `software system` 的 infrastructure nodes
- **預期的受眾**: 軟體開發團隊內/外部的技術人員；包括軟體架構師、開發人員、基礎設施架構師和 operations/support 人員

![](https://static.structurizr.com/workspace/76748/diagrams/LiveDeployment.png)  
![](https://static.structurizr.com/workspace/76748/diagrams/LiveDeployment-key.png)  

---------------------------------------

## 符號(Notation)
`C4 模型`並沒有規定任何特定的符號
- 在白板、紙、便條、索引卡和各種圖表工具上都很好用的簡單符號如下

| `Person` | `Software System` | `Container` |
| :------: | :------: | :------: |
| <img src="https://c4model.com/img/notation-person.png"> | <img src="https://c4model.com/img/notation-software-system.png"> | <img src="https://c4model.com/img/notation-container.png"> |
| `Component` | `Relationship` ||
| <img src="https://c4model.com/img/notation-component.png"> | <img src="https://c4model.com/img/notation-relationship.png"> ||


 
然後，你可以使用顏色和形狀來補充圖
- 來增加額外的資訊
- 或者讓圖更加美觀

---------------------------------------

## C4 和 UML
雖然上面的例子是用「框和線」畫的，但核心圖可以用 UML 來說明
- 然而，由此產生的 UML 圖往往缺乏同樣程度的文字描述性，因為在一些 UML 工具中不容易加這種文字說明

這裡有三個 `System Context`, `Container`, `Component` 圖的例子供比較

![](https://c4model.com/img/spring-petclinic-system-context.png)  
`System Context` 圖  

![](https://c4model.com/img/spring-petclinic-containers.png)  
`Container` 圖  

![](https://c4model.com/img/spring-petclinic-components.png)  
`Component` 圖  

![](https://c4model.com/img/spring-petclinic-system-context-plantuml.png)  
![](https://c4model.com/img/spring-petclinic-containers-plantuml.png)  
![](https://c4model.com/img/spring-petclinic-components-plantuml.png)  
![](https://c4model.com/img/spring-petclinic-system-context-staruml.png)  
![](https://c4model.com/img/spring-petclinic-containers-staruml.png)  
![](https://c4model.com/img/spring-petclinic-components-staruml.png)  

---------------------------------------

## C4 和 ArchiMate
關於如何用 ArchiMate 創建 `C4 模型`圖的細節
- 參考 [`C4 模型`、架構觀點和Archi 4.7](https://www.archimatetool.com/blog/2020/04/18/c4-model-architecture-viewpoint-and-archi-4-7/)

---------------------------------------

## 圖的 key/圖例(legend)
任何使用的符號都應該盡可能地自我描述
- 但所有的圖都應該有一個 `key/legend`，以使符號明確
  - 這也適用 UML, ArchiMate, SysML 等符號創建的圖，不是每個人都知道正在使用的符號

![](https://static.structurizr.com/workspace/76748/diagrams/Containers-key.png)  

---------------------------------------

## 符號，符號，符號
儘管 `C4 模型`是種優先抽象的方法，並且與符號無關
- 你仍然需要確保圖的符號是有意義的，並且圖是可以理解的
- 思考這個問題的好方法是問自己，每個圖是否可以獨立存在，並且在沒有敘述的情況下（大部分）可以被理解
  - 可以使用這個簡短的[軟體架構圖審查清單](https://c4model.com/review/)

![](https://c4model.com/img/software-architecture-diagram-review-checklist.png)  


還有一些與符號有關的建議:  

圖示(Diagrams):
- 每張圖都應該有一個`標題`，描述圖的類型和範圍（如，`System Context diagram for My Software System`）
- 每張圖都應該有一個 `key/legend`，解釋所使用的符號（如，形狀、顏色、邊框樣式、線條類型、箭頭等）
- 首字母縮寫詞(業務/領域或技術)應該是所有受眾都能理解的，或者在圖的 `key/legend` 中解釋

元素(Elements):
- 每個元素的類型都應該被明確定義(如 `Person`, `Software System`, `Container`, `Component`)
- 每個元素都應該有簡短的描述，讓人一目了然的了解其關鍵職責
- 每個 `container` 和 `component` 都應該有一個明確規定的技術

關聯(Relationships):
- 每一條 `line` 都應該代表一種單向的關係
- 每一條 `line` 都應該有 label，應該與關聯的方向和意圖一致
  - 如，依賴性或資料流，盡量具體化，避免使用 Uses 這種單字
- `Containers` 間的關聯(通常這些代表 inter-process communication)應該有個技術/protocol 的明確 label


------------------

## FAQ: 為什麼不直接使用UML？為什麼要重新發明UML？`C4 模型`不是一種退步嗎？
`C4 模型`是進步還是退步，取決於你的位置
- 如果你正在使用 `UML`(或 `SysML`, `ArchiMate` 等)，並且它對你有用，那就堅持下去
- 不幸的是，UML 的使用似乎在下降，許多團隊再次恢復到使用臨時性的方框和線條圖
- 鑑於許多團隊不想使用 UML(出於各種原因)，`C4 模型`有助於為軟體架構的交流方式引入一些結構和紀律
- 對於許多團隊來說，`C4 模型`已經足夠了
  - 而對於其他人來說，也許它是通往 UML 的墊腳石

------------------

## FAQ: 如何為 microservices 和 serverless 建模？
廣義上講
- 用`C4 模型`時，有兩種繪製 microservices 圖的選擇，不過這取決於你對 microservices 的理解

方法 1:
- 每個 microservices 由一個單獨的團隊擁有
- 如果你的系統依賴些不受你控制的 microservices (如，它由一個獨立的團隊擁有和/或運營)
- 那將這些 microservices 建為外部軟體系統，你無法看到其內部

方法 2:
- 一個團隊擁有多個 microservices
- 如果你有一個 API app (如 Spring Boot, ASP.NET MVC 等)，它向一個 DB 讀/寫
- 無論你認為 microservices 一詞是指 API app，還是指 API app 和 DB 的組合
  - 如果 microservices 是你正在構建的軟體系統的一部分(即你擁有它們)，將每個可部署的東西建模為一個 `container`
  - 換句話說，你會顯示兩個 `containers`:
      1. API app 和 database schema
      2. 隨意在這兩個 `containers` 周圍畫一個方框，以表示它們是相關的/分組的
- serverless/Lambdas 等也是如此: 根據所有權將它們視為 `software system` 或 `container`

------------------

## FAQ: 如何繪製大型複雜 software systems 的圖？
即使是小的 `software system`，試圖在一張圖中包含整個故事也是很誘人的
- 例如，如果一個 Web app，建立一個單一的 `component` 圖來顯示構成該 app 的所有 `components`，似乎是合理的
- 除非你的 `software system` 真的很小，否則你很容易就畫滿整張圖 or 發現很難找到一個不被無數重疊的線弄亂的佈局
- 使用更大的圖有時會有幫助，但大圖通常很難解釋和理解
  - 因為認知負荷太高。如果沒有人理解圖，就不會有人去看它

相反
- 不要害怕把單一的複雜的圖分割成更多更簡單的圖
- 每張圖都圍繞著一個業務領域、功能領域、功能分組、有邊界的 context, use case, user interaction 等具體的重點
- 關鍵是要確保每張獨立的圖在相同的抽像水平上講述同一個整體故事的不同部分

另一種方法請參考 `圖解與建模 (Diagramming vs modelling)` (在下面)  


------------------

## FAQ: 為什麼 `C4 模型`不包括業務流程、工作流、狀態機、領域模型、資料模型等？
`C4 模型`的重點是構成軟體系統的靜態結構在不同的抽象層次上
- 如果你需要描述其他方面，可以隨時用 UML, BPML, ArchiMate, entity, relationship 等圖來補充 C4

------------------

## FAQ: `C4 模型` 與 UML, ArchiMate 和 SysML 相比較？
儘管業界已經有各種圖表，如 UML, ArchiMate, SysML 存在，但許多軟體開發團隊並不使用它們
- 這通常是因為團隊對這些符號不夠了解，認為它們太複雜，認為它們與敏捷方法不兼容，或者沒有必要的工具

如果你已經成功地使用這些符號之一來溝通軟體架構，並且它是有效的
- 堅持使用它
- 如果沒有，就試試 `C4 模型`
- 如果有需要，不要害怕用 UML state/timing 圖等來補充 C4

------------------

## FAQ: C4 和 arc42 可以結合嗎？
可以，很多團隊都這樣做，`C4 模型`與 arc42 documentation template 兼容，具體如下:
- Context and Scope => System Context 圖
- Building Block View (level 1) => Container 圖
- Building Block View (level 2) => Component 圖
- Building Block View (level 3) => Class 圖

------------------

## FAQ: `C4 模型`是否意味著設計過程或團隊結構？
「團隊的設計過程應該遵循 `C4 模型`的層次結構」這是一個常見的誤解
- 也許團隊中不同的人負責不同層次的圖
- 如，business analyst 畫 `system context` 圖
- 架構師畫 `container` 圖
- 而開發人員負責其餘的細節層次

當然你可以這樣用 `C4 模型`，但這並不是預期或推薦的使用模式
- `C4 模型`只是種描述 `software system` 的方式，從不同的抽象層次來描述
- 它對軟體的交付過程沒有任何暗示作用

------------------

## FAQ: C4 能用在 libraries, frameworks and SDKs 上嗎？
`C4 模型`實際上是為 software system 建模而設計的，在不同的抽象層次上
- 要記錄 libraries, frameworks and SDKs，可能最好使用 UML 之類的東西
- 另外，可以用 `C4 模型`來描述 frameworks/SDKs 的使用實例

------------------

## FAQ: Web applications 是一個 container 還是兩個？
如果是 server-side web application(如 Spring MVC, ASP.NET, Ruby on Rails, Django等)，主要是生成 static HTML
- 那這就是一個 `container`

如果有大量的 JavaScript 被傳遞(如 SPA)
- 那就是兩個 `container`

儘管在 deployment 時，`server-side web application` 包括 server/client side 的 code
- 但將 client 和 server 作為兩個獨立的 `container`，明確了這是兩個獨立的 `process spaces`
  - 通過 inter-process/remote communication mechanism(如JSON/HTTPS)進行通信
- 它還提供了一個基礎，可以分別放大每個 `container` 來顯示裡面的 `component`


------------------

## FAQ: 「線」應該代表 dependencies 還是 data flow？
這是你的選擇
- 有時，圖表顯示依賴關係（如使用、讀取等）的效果更好
- 而有時資料流（如客戶更新事件）的效果更好
- 無論哪一種，確保線條的描述與箭頭的方向一致

另外值得記住的是
- 大多數關係可以用兩種方式表達，而且越明確越好
- 例如，將一個關係描述為「向其發送客戶更新事件」比「客戶更新事件」更具描述性

------------------

## FAQ: Is a Java JAR, C# assembly, DLL, module 等是一個 container 嗎？
一般來說不是
- `container` 是種運行時結構，就像一個 application
- 而 Java JAR, C# assembly, DLL, module 等是用來組織這些 application 中的 code

------------------

## FAQ：Java JAR, C# assembly, DLL, module, package, namespace, folder 等是一個 component 嗎？
也許是，但是，通常不是
- `C4 模型`是關於顯示運行時 `component`(`containers`)以及功能如何在它們之間劃分(`component`)
- 而不是組織單元，如 Java JAR, C# assembly, DLL, module, package, namespace, folder 等結構

當然，這些結構和 `component` 可能存在一對一的映射
- 如果你構建一個六邊形的架構，你可以為每個 `component` 建立一個 Java JAR
- 另方面，一個 `component` 可能會使用些 JAR 文件的程式碼來實現
  - 當你開始考慮第三方 frameworks/libraries，以及它們如何嵌入到你的程式碼庫中時，通常會發生這種情況

------------------

## FAQ: 應該包括 ` message buses`, `API gateways`, `service meshes` 等嗎？
如果你有兩個服務(A, B)
- 通過 `message bus`(不考慮 topics, queues, p2p, pub/sub 等）或其他中介(如 API gateway or service mesh)發送 message 進行溝通

你有幾個選擇:
1. 顯示服務 A 向中介機構發送 message，而中介機構隨後將該 message 轉發給服務 B
    - 雖然準確，但該圖的 `hub and spoke`(樞紐和輻條)性質往往掩蓋了 message 生產者和消費者(producer and consumer)之間存在耦合的概念
2. 省略中間人，而使用符號（如文字描述、顏色編碼、線條風格等）來表示 A 和 B 之間的交互是通過中間人進行的
    - 這種方法往往能使圖表講述一個更清晰的故事


------------------

## FAQ: Data storage services 應該顯示為 software systems 還是 containers ?
一個常見問題是，像 `Amazon S3`, `Amazon RDS`, `Azure SQL DB`, `CDN` 等服務是否應該被顯示為 software systems 還是 containers
- 畢竟，這些都是外部服務，大多數人並不自己擁有或運行

如果你正在構建一個使用 Amazon S3 存儲資料的軟體系統
- 你確實沒有自己運行 S3，但你確實對你使用的 buckets 有所有權和責任
- 與 `Amazon RDS` 類似，你對你創建的任何 DB (或多或少)有完全的控制權
- 出於這個原因，把它們當作 `containers？`，因為它們是你的 software architecture 的一個組成部分，儘管它們被託管在其他地方

------------------

## 圖解與建模 (Diagramming vs modelling)
我們傾向於選擇圖表(diagramming)，而不是建模(modelling)，主要是因為進入的門檻相對較低
- 無論是畫圖還是建模，都可以使用 `C4 模型`，但當你從畫圖到建模時，有一些有趣的機會

在建模時
- 你要建立一個非可視(non-visual)的模型(如軟體系統的軟體架構)
- 然後在這個模型上創建不同的圖(如圖表)，這需要更嚴格的要求，所有元素的單一定義和它們之間的關係
- 反過來又允許建模工具理解你所要做的事情的語義，並在模型之上提供額外的智能
  - 允許建模工具提供替代性的可視化，通常是自動的


其中一個經常被問到的問題是關於大型複雜軟體系統的圖示(diagramming)
- 一旦你開始在圖上有超過 20 個元素(加上它們之間的關係)，結果就很快變得雜亂無章
- 下圖是一個單一容器的組件圖

![](https://c4model.com/img/modelling-1.png)  

處理這個問題的方法是
- 不在一個圖上顯示所有的 `components`，而是創建多個圖
- 在 `container` 中每一個切片都有一個圖（如下圖2）
  - (one per "slice" through the container 回去有網路再查一下圖片網址)
  - 這種方法有幫助，但值得問的是，由此產生的圖是否有用？
- 你是否要使用它們，如果是的話，你要用它們做什麼？儘管系統 context 圖和 `container` 圖非常有用
- 但大型軟體系統的 `component` 圖往往價值不大，因為它更難保持更新
- 而且你可能會發現很少有人看它們，特別是如果它們沒有被包括在 document 或 presentation 中

(這邊要補一張 圖2 連結!!!!!!!!!!!!!!!!!!!)  

通常情況下，圖表本身並不是最終目的
- 團隊使用圖表來回答其他問題，如，component X 有依賴什麼？
- 如果是這種情況，model 能回答這些問題，而不需要花費額外的精力來創建圖表
- 換句話說，一旦有了一個 model，就可以用許多不同的方式將其可視化(上面的圖片 3 和 4)幫助回答真正問題
- 圖形當然是溝通軟體架構的一種奇妙方式，但其他的可視化方式有時也能幫助回答可能的真正的潛在問題

(要補一張 圖3,4 連結!!!!!!!!!!!!!!!!!!!)  

------------------

## 工具
For design meeting:
- 白板更適合於協作和快速迭代


對於長期存在的 document，以下工具可以幫助在 `C4 模型` 的基礎上建構軟體架構圖:
- https://c4model.com/ (最下面 Tooling 的區塊，有太多了，這邊不列了)

------------------

## Mermaid
這段不是文章中的內容，但我自己會想試試看用 `Mermaid` Markdown 來畫畫看  
找到一篇用 Mermaid 畫 C4 不錯的範例
- https://lukemerrett.com/building-c4-diagrams-in-mermaid/