# 2022-11-07：Jess Sattell: Naming colors in design systems
## Jess Sattell (Staff Content Designer, Spectrum Design System), Nate Baldwin, Shawn Cheris, April 12, 2022
### https://adobe.design/stories/design-for-scale/naming-colors-in-design-systems

對於 design system 裡面的命名一直不太清楚，來看看 Adobe Spectrum 的 content designer leader 怎麼談  

----------------------------------------

Adobe 擁有
- 複雜的品牌組合，包含 100 多種不同的產品
- 使用各種前端框架，涵蓋 Web、mobile 和 desktop
- 人們用來交流 UI 概念的語言中有一層又一層的微妙抽象
- 這需要可靠的 `source of truth` 來概述設計和實現細節
- 在這樣規模的情況下，design system 非常複雜

Spectrum 一直在思考文字。在 logically 和 strategically 之間來命名事物、以簡單明了的方式交流抽象和複雜概念以及建立詳細文件
- 語言是團隊對話的核心。 我們製定一種 identify patterns，與之進行溝通

將概念性和模棱兩可的事物，構建出 design system 的標誌，並將抽象的想法轉化為有形的想法  
這是個 skillset，對於高度技術化和 systems-oriented 的團隊來說是個很好的補充，任何從事 design system 工作的人都知道
- 它需要協調兩個系統：設計和 code
- 每個人都使用特定的語言、獨特的邏輯和語法來理解抽象和復雜的概念
- 組件、樣式和行為的想法和溝通方式不同，具體取決於詢問的對象

Color naming 是很好的例子，來展示如何將 content design 結合到 design system 中，以及語言如何將邏輯帶入一個微妙且通常是主觀的主題

----------------------------------------

## 1. 在 Design system 中命名 color

Color 是 design system 的基礎
- 建立 color system 時，需要為其所有不同方面和用途命名


許多 design system
- 都有 UI `color palettes`，以及用於 illustration 和 data visualization 之類的附加 `color palettes`
- `color palettes` 通常具有非常常見的 common names
- 而其他調色板更具描述性或使用不太常見（但仍可識別）的顏色名稱
- 例如，可以為 UI 或核心顏色指定其顏色系列的名稱，如 `red`, `blue` or `Purple`
  -  在增量狀態下以 `100` 的對比度分配一個值，同時為其他調色板命名具有特定意圖或用途的顏色
    - (with prefixes like) `vivid blue` (鮮豔的藍色) 或 `subdued green` (柔和的綠色)


最成功的顏色命名法堅持 common language。當名稱古怪、參考文化趨勢或過度品牌化時，就很難理解顏色如何作為一個系統協同工作  
一些例子:
- 如果公司為 Awesome Company，並且你將顏色命為 `Awesome Company Purple`，那大部分的人會搞不懂這是哪種紫色
- 如果命名一種顏色為 `Raichu Orange` （`Raichu` 應該是神奇寶貝的`雷丘`），則必須要假設 User 知道這個流行文化。User 不應該去尋找顏色名稱背後的含義，這會給 design system 增加認知負擔
- 如果命名一個顏色為 `Scary Monster`，可能會引發各種各樣的想像，因為大家對什麼是 `Scary Monster` 都有自己的想法
- 即使是為了通過使用多個形容詞來表達特殊性而命名的顏色，如 `dark blueish green`(深藍綠色) 或 `teal blue`(藍綠色)，也可能不清楚。顏色是藍色嗎？還是綠色？還是青色？


所以說 `Color naming` 是很好的例子，說明如何將 content design 的實作應用於 design system，以及語言如何將邏輯帶入一個微妙且通常是主觀的主題之中

----------------------------------------

## 2. Spectrum 如何命名 colors
Spectrum 的 [colors](https://spectrum.adobe.com/page/color-fundamentals/) 會 mapping 到我們的 [design tokens](https://spectrum.adobe.com/page/design-tokens/)
- 這些 tokens 是構建和維護 design system 所需的所有值
  - 如 spacing, color, typography, object styles, and animation—represented as data
- Spectrum 對命名決定盡可能利用行業術語和眾所周知的術語
  - 避免選擇在設計和開發社群之外無法識別的顏色名稱
  - 還避免使用流行的、主觀的或 U.S. English 以外的語言（Adobe’s source locale for in-product content）的名稱。

考慮廣泛而多樣化的 User 需求
- 顏色以非常常見的詞(`blue`，而不是 `oceanic`)命名
- 增量亮度值比例 (50–900) 配對
  - 如，`gray-50` 比 `gray-100` 亮
- 隨著 Spectrum 多年來不斷發展以滿足 Adob​​e 眾多產品的需求，**這種方法幫助我們進行了穩定的更改，而不會產生重大影響**

這不是我們馬上想出來的。design system 中的所有事物都會改變、顏色會逐漸變化
- 我們不得不偶爾改變方向
- 我們還進行了幾次迭代來完善我們的顏色 numeric system
- 這直接影響我們如何思考和談論它們
- 但我們已經了解到，與通用語言顏色名稱一起分配的數值對於傳達意義同樣重要

----------------------------------------

## 3. 提出顏色命名系統 (color nomenclature system)

Color system 需要邏輯、重點和大量的分類工作  

謹慎使用特殊名稱
- 人們犯的最大錯誤是用一個不適合其他顏色的一次性名稱來命名每種顏色
- 如果顏色系統中的每種顏色都被命名為非常獨特的東西，很快就會導致理解和可用性方面的大問題
- 為重要的品牌資產或具有特定用途的 color palettes 保留特殊名稱


了解描述性(descriptive)命名的位置
- 沒有映射到特定 design component 的顏色，或者將用於不同視覺色調的比例以建立特定效果或情感反應（例如產品內插圖）的顏色可能具有超出標準 `ROYGBIV` 顏色的更具描述性的名稱
- 這通常是 brand voice 或 brand principles 可以發揮作用的地方
- 例如，如果正在為化妝品製造商命名產品，可以將 `light-pink`(淺粉色)指甲油稱為「泡泡糖」，來喚起一種情緒。將 `cherry blossom`(櫻花)為另一種情緒

用詞語加上 values 的組合來工作
- 盡量保持「名稱 + value」的配對系統（如 `gray-100`）
- 如果增加了更多的色調，就會有更多的空間來容納它們
- 使用數字標尺也有一個可預測的因素
  - 如 Spectrum 的系統是相對於對比度的，而不是亮度的
    - 淺色主題的命名方式讓人很容易理解，`blue-100` 比 `blue-400` 淺
    - 深色主題中，`blue-100 `比 `blue-400` 更深
    - 在所有主題中，`blue-100` 與背景的對比度(contrast)都比 `blue-400` 低


熟悉術語
- 就像每個人對顏色的看法不同一樣，更大的設計和開發社群如何就顏色進行交流也存在差異
- 確保你的團隊在如何稱呼顏色的不同特徵方面保持一致，並清楚地定義每個單詞和概念的相關性
- 更好的是，將它們與你的 color charts 或 design tokens 列表一起記錄
  
例如，你可以決定使用術語 `color family` 和 `color group` 來描述系統中的顏色分類。他們的定義可能是:
- 一個 `color family` 是根據色彩理論和/或更大的產品設計的原則對一種顏色進行分類
  - 例如，可以說 `cyan`(青色)屬於藍色系列(blue color family)
  - 通過擴展，它還指構成特定命名顏色的 value scale 的一組顏色。
- 一個 `color group` 是指一個主觀實體指定的一組顏色，應該根據特定的使用屬性歸為一組
  - 例如，顏色組可以被命名為 `Brand Gray`, Illustration, or Data Visualization

最成功的顏色命名法堅持 common language
- 當顏色的名稱古怪、參考文化趨勢或過度品牌化時，就很難理解顏色如何作為系統協同工作

----------------------------------------

## 4. 命名顏色時要嘗試的事情
盡可能地使用 `name + value` 配對模型:
- 它的可擴展性更強，可以在最小的干擾下實現未來的變化
- 一個標準的方法是使用色系分類器（如 `green` 或 `gray`）與亮度或對比度值表（`10` 為淺色，`100` 為深色）配對的組合


將多個屬性 map 到每種顏色
- 例如，如你對一種較淺的藍色使用 `blue-10`，它標誌著色系（藍色）和值（10）
- 所以 `blue-10` 是它的 `UI color name`
- 別名可以是 `primary call to action button` 這樣的東西
  - 以描述它的用途或目的
- 這就創造了一個 metadata 或與顏色有關的同義詞層

在 documentation 中
- 將 `color codes` 與 `color names` 一起傳達是很重要的
- 可以幫助 User 理解更廣泛的系統
- 但不要使用 hex codes，或在顏色名稱中引用任何其他顏色代碼值，如 `RGB` 或 `HSL`
  - 你會經常改變飽和度和其他屬性
  - 而且由於顏色代碼也會隨之改變，這就需要大量的重命名

策略性命名:
- **少量的**使用描述性的，但不是過於感性或具體的顏色名稱可以非常有效
  - 例如，`Mint` (薄荷色)（但不是 `Wintry Green`(冬日綠)）或 `Coral` (珊瑚色)（但不是 `Tropical Vacation`(熱帶假期)）
- 對 illustration and data visualization 的 `color groups` 使用更具描述性的命名是可以的
  - 因為這些顏色在 UI 中要達到不同的目的。就像書面語言中的語調一樣，它們經常被用於視覺語調的不同方法

----------------------------------------

## 5. 命名顏色時要避免的事情
一次性的、抽象的、或過於隱喻的名字:
- 例如，`Odysseus Blue`(奧德修斯藍)是什麼意思？
- 像這樣的名字的解釋是開放的，創造了不一致
- Design system 中的顏色來說，精確性是非常重要的
- 對於系統的 User 和 UI 的 User 來說都是如此，尤其是在設計無障礙界面時

用 project names 或 code names:
- 假設你正參與 `Ice Cream` project，所以你決定將一個重點顏色命名為 `Ice Cream`，希望它能在公司 brand palette 中與其他顏色的區別
- Code name 是晦澀難懂的，在一個以清晰和易於理解為目標的 design system 中，它們並沒有什麼地位
  - `Ice Cream` 有可能是任何一種顏色
  - 而且，所有的 code name 都有一個最終的意義到期日，這意味著無論如何，你遲早要改變這種顏色的名稱

時間標記或相對時間:
- 把一種顏色的更新版本稱為 "新"，然後就此作罷，往往是很誘人的
- `New Blue` 只是現在為 new，如果幾年後（甚至幾個月後）出現了更新的藍色，會發生什麼？
- 像 `Heritage Blue` 這樣意味著「一個特定藍色的廢棄版本」也導致同樣的問題
- 這種方式命名，會使你陷入破壞性的變化，不得不花時間重新思考整個系統

沿著設計語言的更新來命名
- 比如根據相對的時間來命名事物。如果給一種顏色命名為 `Update 5 Blue`，這聽起來像是一種顏色的版本，而不是一種設計語言的版本

----------------------------------------

## 6. 命名 design system 中其他事物的時候，採用命名 colors 同樣的方法

總而言之，命名顏色的 best practices 與 design system 中任何東西的命名原則相同  

依靠現有的範式:
- 在命名和分類法方面，向 established leaders 尋求靈感
- [pantone](https://www.pantone.com/)， 以其廣泛的色彩選擇和管理良好的分類法引領了潮流
- Operating-level design systems，像是 Google 的 `Material Design` 和 Apple 的 `Human Interface Guidelines` 都有深入研究和測試的支持，以支持他們的決策，包括命名
- Spectrum: https://spectrum.adobe.com/

Don’t be cute:
- Design system最終要歸結為 utility，如果它容易理解，人們就更有可能使用它
- 第一次遇到你的顏色的人，不管是公司新人還是第三方用戶，都不會熟悉內部品牌的行話或你團隊的內部笑話
- 而且，許多人不認識那些依賴於特定文化參考的名稱，或相關的潛台詞


有策略地設計:
- 如果你使 design system 中的每個名稱都特別或過於情緒化，它會減弱重要的品牌資產，而這些資產由於商業原因是需要獨一無二的

傳達意義(Communicate meaning)
- Design systems 是關於 utility 和 scalability
- 使用像 numeric scale 這樣的方法來傳達意義，會帶來很大的靈活性
- 因為你在成長過程中不可避免地會一次又一次地改變你的 color system

最後記住
- 名稱可以而且將會逐漸改變，就像語言和 design system 的所有其他部分一樣
- 因此，請堅持並 `evolve complex systems incrementally` (逐步發展複雜的系統)

