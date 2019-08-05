## semantic HTML
#### source
- https://dev.to/kenbellows/stop-using-so-many-divs-an-intro-to-semantic-html-3i9i

div 是大家的最愛，但還可以更好
```html
<div></div>
```

這樣的 div 使用很常見

```html
<div class="container" id="header">
  <div class="header header-main">Super duper best blog ever</div>
  <div class="site-navigation">
    <a href="/">Home</a>
    <a href="/about">About</a>
    <a href="/archive">Archive</a>
  </div>
</div>
<div class="container" id="main">
  <div class="article-header-level-1">
    Why you should buy more cheeses than you currently do
  </div>
  <div class="article-content">
    <div class="article-section">
      <div class="article-header-level-2">
        Part 1: Variety is spicy
      </div>
      <!-- cheesy content -->
    </div>
    <div class="article-section">
      <div class="article-header-level-2">
        Part 2: Cows are great
      </div>
      <!-- more cheesy content -->
    </div>
  </div>
</div>
<div class="container" id="footer">
  Contact us!
  <div class="contact-info">
    <p class="email">
      <a href="mailto:us@example.com">us@example.com</a>
    </p>
    <div class="street-address">
      <p>123 Main St., Suite 404</p>
      <p>Yourtown, AK, 12345</p>
      <p>United States of America</p>
    </div>
  </div>
</div>
```

上面的 code 很ＯＫ，能正常 work，但還是有些問題

### SEO

### A11y Accessibility
現在很多 A11y 工具，能夠轉譯這些 page 的結構內容給 User。甚至能幫助 User 更快找到他想看的東西。
- `<div />` 這邊就沒有幫助
- 語意 tag 就很有用

### 可讀性 Readability
讀上面這 code，必須要 scan classname 才能精準知道內容是什麼  
讀 classname 對結構性了解也很差，因為只要 div 多點內容、階層拉開點，就需要前後一值對照來看

### 一致性和標準 Consistency and standards
每當換新工作 or 接收新專案時，要重新看這些結構是很沮喪的經驗
如果有能力用 standardized 的方法來  mark up common structures  
那幫助會很大

### HTML5
HTML5 引入了 `semantic elements`  
讓 document meaningful

#### HTML5 spec 
HTML5 的 `<div />` 結論這樣寫  

> NOTE:
Authors are strongly encouraged to view the div element as an element of last resort, for when no other element is suitable. Use of more appropriate elements instead of the div element leads to better accessibility for readers and easier maintainability for authors.

全部的 element 都用 div 是最後的大招，除非沒有其他 element 可以取代。
為了
- better accessibility for readers
- easier maintainability for authors

#### 分兩類
有人把 semantic block elements 歸納成兩類（非標準分類法）
- primary structure
- content indicators

#### Primary Structures
常見的區分區塊方式為，把網站分成
- header
- main
- footer

```html
<div class="container" id="header">...</div>
<div class="container" id="main">
    ...
    <div class="article-section">...</div>
    ...
</div>
<div class="container" id="footer">...</div>
```
上面這模式已經存在數十年了
- 有可讀性
- css styling 更輕鬆一點

對某些語言 solution ，header, footer 更是好處理
```php
<?php include 'header.php'; ?>

<div id="main">...</div>

<?php include 'footer.php'; ?>
```

這是公認很好 follow 的 pattern  
WHATWG and W3C 都建議這樣這 pattern，並推出新標準 elements
- `<header>`
- `<main>`
- `<footer>`
- `<section>`


### <header> and <footer>
- 這兩個基本上是一對的
- 兩者非常相似
  - headers go at the beginnings of things
  - footers go at the ends of things

用在你 document 任何一個部分
- 某一部份要呈現 a chunk of content
- 這一部分有著清楚的 beginning and end
- ex: forms, articles, sections of articles, posts on a social media site, cards


Headers and footers 語意是最接近這兩個區塊
- sectioning root
- sectioning content

有些 Assistive tool 可以利用這些 elements 來產生這份文件的 outline  
能幫助 user 更有效的 navigate 此文件  

-  每 sectioning root/content 不可以有多個 <header> or <footer>


### <header>
`<header>` 常常會有 `heading element` 在裡面  
- <h1>-<h6>

非必要，但這對 group heading 裡面的相關 element 很有幫助 ,ex
- links
- images
- subheadings

有這些能保持很好的結構，比起單單一個 <header> 來說更好

### <main> 好東西啊
- 第三個 primary region element
- spec 指出兩件重要的事情

>The main content area of a document includes content that is unique to that document and excludes content that is repeated across a set of documents such as site navigation links, copyright information, site logos and banners and search forms (unless the document or application’s main function is that of a search form).

- document 的 main content
- 排除 document 中重複出現的東西
  - site navigation links, copyright information, site logos and banners
  - search forms (unless the document or application’s main function is that of a search form)

所以 `<main>` 用來
- 放好東西
- 放重要的部分
- user 來這網站，就是看這塊
- main content 就對了

logos and search forms and navigation 這些可以放到
- <header> or <footer> within the <body>
- but outside of <main>

> There must not be more than one visible main element in a document. If more than one main element is present in a document, all other instances must be hidden using `the hidden attribute`

- 跟 <header> and <footer> 還有其他常見的 element 不一樣
  - <main> 不能過度使用
  - it should be used once and only once. Or rather
  - it can be used multiple times in a document, but only one <main> element should be visible at a time
    - all others must be hidden with the hidden attribute
    - like display: none; in CSS
    
case study
```html
<header>
    <h1>Super duper best blog ever</h1>
    ...
</header>
<main>
    <h2>Why you should buy more cheeses than you currently do</h2>
    ...
</main>
<footer>
    Contact us!
    <div class="contact-info">this.is.us@example.com</div>
</footer>
```

### Break it down: <section>
上面的例子有基本的 outline 了
- a header, a footer, and a main content region

可以在做點什麼  
基本上，你會想要把大量的內容拆分到 `sections` 中，像是 article  
沒人想要讀 impenetrable walls of text  

這時候就靠 `<section>`
- <section> 基本上就是「帶有特定 semantic meaning 的 <div>」
- <section> 是指一個新的 sectioning content 要開始了的區塊
  - 所以它可以有它自己的 <header> and/or <footer>

比如，內容中的一個專題組，一般來說會有包含一個標題（heading）  
一般通過是否包含一個標題 (<h1>-<h6> element) 作為子節點 來 辨識每一個<section>

#### <section> 跟 <div> 的差別
> NOTE:
The <section> element is not a generic container element. When an element is needed only for styling purposes or as a convenience for scripting, authors are encouraged to use the <div> element instead. A general rule is that the <section> element is appropriate only if the element’s contents would be listed explicitly in the document’s outline.

- <section> 不是 generic container element
- 當一個 element 只是需要 for styling purposes 或者為了方便 scripting
  - 那用 <div> 就好
- <section> 只適合在 element 的 content 會被精確列出在 document 的 outline 當中的內容

HTML5 spec 是非常可讀性的 spec 文件，每次看都能學到些東西

### Content Indicators
上面的內容，讓網站有了主要的結構  
HTML5 還有新增很多 content 的 semantics 而不是 for structure 的

### <article>
- 用在呈現 a fully complete self-contained region of content
- 可能是
  - literal article
  - blog post
  - social media post
    - a tweet
    - a Facebook wall post
- 推薦用 heading element (<h1>-<h6>)
- 可以有 <header>, <footer>, and <section>
- 所以真的可以用來 embed a full document fragment
- Each article should be identified
  - typically by including a heading(h1-h6 element) as a child of the article element.
- Assistive tool 能夠辨識 article

範例
```html
<article>
 <header>
  <h2><a href="https://herbert.io">Short note on wearing shorts</a></h2>
   <p>Posted on Wednesday, 10 February 2016 by Patrick Lauke.
   <a href="https://herbert.io/short-note/#comments">6 comments</a></p>
 </header>
 <p>A fellow traveller posed an interesting question: Why do you wear shorts rather than
 longs? The person was wearing culottes as the time, so I considered the question equivocal in nature,
 but I attempted to provide an honest answer despite the dubiousness of the questioner’s dress.</p>
 <p>The short answer is that I enjoy wearing shorts, the long answer is...</p>
 <p><a href="https://herbert.io/short-note/">Continue reading: Short note on
 wearing shorts</a></p>
</article>
```

在上面提到的範例可以這樣改
```html
<article>
  <header>
    <h1>Why you should buy more cheeses than you currently do</h1>
  </header>
  <section>
    <header>
      <h2>Part 1: Variety is spicy</h2>
    </header>
    <!-- cheesy content -->
  </section>
  <section>
    <header>
      <h2>Part 2: Cows are great</h2>
    </header>
    <!-- more cheesy content -->
  </section>
</article>
```

### <nav>
- 用來定義 page 主要的 navigation blocks
- 給 user 清楚整個 site 的分佈、他們自己現在在哪裡

```html
<nav>
  <a href="/">Home</a>
  <a href="/about">About</a>
  <a href="/archive">Archive</a>
</nav>
```

### <address>
- 用來招集 contact info
- 常用在 <footer> 中 for
  - mailing address
  - phone number
  - customer service email address etc. for a business
- 是一種 left open element (應該是指裡面的設計可以很自由)

```html
<address>      
UNIVERSITY INTERSCHOLASTIC LEAGUE<br>
1701 Manor Road, Austin, TX 78722<br>
Tel: (512) 471-5883 | Fax: (512) 471-5908
</address>

<label for="name">Name:</label> <input type="text" id="name">
<label for="hn">House number:</label> <input type="text" id="hn">
<label for="street">Street:</label> <input type="text" id="street">
...
<address>
<p>Name: Hament Dhanji
<p>House number: 1976
<p>Street: Meadowband Road
...
</address>
```

要進一步的話，用 `RDFa` 來強化

```html
<footer>
  <section class="contact" vocab="http://schema.org/" typeof="LocalBusiness">
    <h2>Contact us!</h2>
    <address property="email">
      <a href="mailto:us@example.com">us@example.com</a>
    </address>
    <address property="address" typeof="PostalAddress">
      <p property="streetAddress">123 Main St., Suite 404</p>
      <p>
        <span property="addressLocality">Yourtown</span>,
        <span property="addressRegion">AK</span>,
        <span property="postalCode">12345</span>   
      </p>
      <p property="addressCountry">United States of America</p>
    </address>
  </section>
</footer>
```

```html
<div vocab="http://schema.org/" typeof="LocalBusiness">
<h1><span property="name">Beachwalk Beachwear & Giftware</span></h1>
<span property="description"> A superb collection of fine gifts and clothing
to accent your stay in Mexico Beach.</span>

<address property="address" typeof="PostalAddress">
<span property="streetAddress">3102 Highway 98</span>
<span property="addressLocality">Mexico Beach</span>,
<span property="addressRegion">FL</span>
</address>

Phone: <span property="telephone">850-648-4200</span>
</div>
```

最後的調整為
```html
<header>
  <h1>Super duper best blog ever</h1>
  <nav>
    <a href="/">Home</a>
    <a href="/about">About</a>
    <a href="/archive">Archive</a>
  </nav>
</header>
<main>
  <article>
  <header>
    <h1>Why you should buy more cheeses than you currently do</h1>
  </header>
  <section>
    <header>
      <h2>Part 1: Variety is spicy</h2>
    </header>
    <!-- cheesy content -->
  </section>
  <section>
    <header>
      <h2>Part 2: Cows are great</h2>
    </header>
    <!-- more cheesy content -->
  </section>
</article>
</main>
<footer>
  <section class="contact" vocab="http://schema.org/" typeof="LocalBusiness">
    <h2>Contact us!</h2>
    <address property="email">
      <a href="mailto:us@example.com">us@example.com</a>
    </address>
    <address property="address" typeof="PostalAddress">
      <p property="streetAddress">123 Main St., Suite 404</p>
      <p>
        <span property="addressLocality">Yourtown</span>,
        <span property="addressRegion">AK</span>,
        <span property="postalCode">12345</span>   
      </p>
      <p property="addressCountry">United States of America</p>
    </address>
  </section>
</footer>
```

#### sectioning root
- https://www.w3.org/TR/html5/sections.html#sectioning-roots

Certain elements are said to be sectioning roots, including blockquote and td elements. These elements can have their own outlines, but the sections and headings inside these elements do not contribute to the outlines of their ancestors.
- <blockquote>
- <body>
- <details>
- <dialog>
- <fieldset>
- <figure>
- <td>

#### sectioning content
https://www.w3.org/TR/html5/dom.html#sectioning-content-2

Sectioning content is content that defines the scope of headings and footers.

- <article>
- <aside>
- <nav>
- <section>

Each sectioning content element potentially has a heading and an outline. See the section on headings and sections for further details.

