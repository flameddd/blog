# Working Backwards
## Werner Vogels (CTO - Amazon.com) 01 November 2006
### https://www.allthingsdistributed.com/2006/11/working_backwards.html

Amazon 的 services
- 不只代表軟件 **software structure**
- 也代表 **organizational structure**

這些 services
- 都有 strong ownership model
- 跟「小的團隊規模」

這是為了 -> **to make it very easy to innovate**

某種程度來說，這些 services 可以視為
- 大公司內部的小型創業公司


每一個服務
- 都需要特別關注其客戶是誰，無論他們是外部還是內部

為確保服務滿足客戶需求（**且不超過此需求**），我們使用 **“Working Backwards”** 流程
- you start with your custome
- and work your way backwards until you get to the minimum set of technology requirements to satisfy what you try to achieve.

從客戶開始，然後**向後**工作，直到達到最低的技術要求。滿足您嘗試要實現的目標。
- 目標是透過 simplicity 來達到一個持續的流程，逐漸釐清客戶的關注點
  - (The goal is to drive simplicity through a continuous, explicit customer focus.)

Working Backwards
- 首先寫在發佈時需要的 documents（「新聞稿」和「常見問題解答」）
- 然後朝這些 documents 來 implement

Working Backwards 的產品定義過程都是為了
- 充實 concept 的細節
- 對於我們最後要推出的東西，建構出這產品明確的想法

有四個步驟

## step. 1
- 首先，從「新聞稿」開始寫，以簡單的方式描述了該產品
  - 「產品的功能」和「其存在的原因」
  - 功能和優點是什麼

「新聞稿」必須非常明確並且要切合實際  
預先撰寫「新聞稿」說明了
- 世界將如何看待該產品
- 而不僅僅是 Amazon 在內部如何看待它
  
## step. 2
寫 FAQ 文件
- 「新聞稿」提供了一個骨架，FAQ 就是肉
- FAQ 包含了我們寫「新聞稿」時所出現的問題
- FAQ 包含當發布「新聞稿」時，別人所問的問題
- FAQ 包括定義產品優勢的問題
- FAQ 包括定義產品優勢的問題

站在客戶的角度思考，想想還有哪些問題

## step. 3
定義客戶體驗
- 針對「客戶會對產品有不同的使用方法」，去詳細描述出客戶體驗（說明）
 
對於有 user interface 的產品
- 我們會為客戶使用的每個 screen 去建立 mock ups

對於 web services
- 我們提供 **use cases** 和 code snippets

這些「描述方式」
- 讓你可以想像 user 使用該產品的方式

此處的目的是
- 講述有關客戶「如何使用產品」來「解決問題」的故事。

## step. 4
寫用戶手冊 (User Manual)
- 用戶手冊是客戶用來真正了解產品是什麼以及如何使用的手冊

通常包含三個部分
- 概念
- 操作方法
- 參考

這三個部分可以告訴客戶使用該產品所需的一切知識。  
甚至對於具有多種用戶 (more than one kind of user) 的產品，我們寫了不止一本用戶手冊。  


## finally
當完成了 step 1 ~ 4 的過程
- it is amazing how much clearer it is what you are planning to build.


我們將提供一套 documents，可用於向 Amazon 內部的其他團隊解釋新產品  
到那時，我們知道整個團隊對我們要生產的產品有共同的願景


## others
我覺得這篇可以搭配 Uber 的一篇文章一起看
- https://github.com/flameddd/blog/blob/master/2019-06-15%EF%BC%9AUber%20%E6%97%A9%E6%9C%9F%E5%A6%82%E4%BD%95%20scaling%20engineering%20team.md