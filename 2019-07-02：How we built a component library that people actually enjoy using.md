## How we built a component library that people actually enjoy using
### Chase McCoy 2019 Jun 27
#### https://medium.com/styled-components/how-to-build-a-great-component-library-a40d974a412d

When we launched, our system was home to four categories of guidelines and principles:
- Brand
- Visual
- Writing
- and Product

主要分為兩個部分
- 一個 npm component library
- 一份 document

沒有專門人的維護，套件就變得開發體驗差、文件也跟不上
- component documentation 跟 design system 要維護在一起

some tools used to build a component librar


### development 和 documentation 分開
- 開發完 component 後，document 常常沒跟上 
- documentation will always fall short when it’s treated as a subtask of the development process

docz
- 用來寫文件
- 用這套寫 live code
- 用好 props 來處理

### Great documentation
- should more like a workshop
  - interactivity
  - communicate more information to our consumers

When contributors author new content
- we don’t give them a structure to fill with information

Instead
- we try to let the information inform the structure itself

Here are a few cool examples

#### Interactive code blocks
Every Markdown code block in our component pages turns into a fully editable sandbox for the component

靠
- react-live


### sproutsocial 的 document
good document ref
- https://sproutsocial.com/seeds/components/alert

### Token tags
- 一種 scale 的數值設計
- 可視化
- 點擊就複製

我們可能不用做到這麼細緻

### Typography playground
- https://sproutsocial.com/seeds/visual/typography/

這文件寫得太好了吧...  
這邊可以參考來找出我們所需要的東西屬性  

### Styled System
用來把把屬性 scale 化  

Primer Components 把可以標準化的屬性整理出清楚的列表！
- https://primer.style/components/docs/system-props

### 頂級參考
- https://primer.style/components