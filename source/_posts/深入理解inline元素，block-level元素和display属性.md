---
title: 深入理解inline元素，block-level元素和display属性
date: 
tags: 
---

>行内元素和块级元素有什么异同？

*可能你觉得这是一个老掉牙的问题，但其实并没有那么简单。*

对于初学者来说，当遇到面试官问到这个问题的时候，脑海里的第一印象是css的display属性，值为inline时为行内元素，值为block时为块级元素，那值为inline-block时呢，元素是块级行内元素吗？

其实我们看问题的角度从一开始就错了，错误地站在了css的角度，而真正正确的是站在html的角度。因为这里的元素，本质上是指html的DOM元素。

下面我们就3个原理展开深入：
- 深入理解inline元素
- 深入理解block-level元素
- 深入理解display属性

并且提出两个问题：
 - 行内元素和块级元素有什么异同？
 - css属性display如何影响元素？

 ### 深入理解inline 元素
在HTML中，行内元素是那些仅仅占据定义元素边界的标签空间的元素，而不是去破坏内容流。在这篇文章中，我们将检查HTML行内元素并且与block做对比。
> 一个行内元素不会在a new line开始，而且仅仅占据必要的元素宽度空间，不会多占空间。

### inline vs block-level 元素：示范
用一个简单例子来示范就好。
```
.highlight {
  font-weight:bold;
}
```
**Inline**
首先，让我们看一下下面的inline元素的示范：
```
<p>The following span is an <span class="highlight">inline element</span>;
its background has been colored to display both the beginning and end of
the inline element's influence.</p>
```
在这个例子中，<p>(段落)块级元素包含一些文本。在text文本内部有一个<span>行内元素。因为<span>标签是行内的，所以段落正确地绘制出一个单个的，没有断开的文本流，就系那个下面这样。

>The following span is an **inline element**; its background has been colored to display both the beginning and end of the inline element's influence.

**block-level**
现在把`<span>`替换成`<div>`：
```
<p>The following div is an <div class="highlight">block-level element;</div>
its background has been colored to display both the beginning and end of
the block-level element's influence.</p>
```
绘制完之后是这样的：
>The following div is an
block-level element;
its background has been colored to display both the beginning and end of the block-level element's influence.
看到差异了吗？<div>元素改变了text的layout，将文本切分成了3个片段：在<div>之前的text，<div>的text，在<div>之后的text。

#### 改变元素的level
你可以使用CSS的display属性改变元素的level。
inline blocks 通过将display的值从inline改为block，你可以告诉浏览器将一个inline元素，渲染成block box而不是inline box。改了属性值以后，不一定会完全呈现，它可能还是表现成行内元素，所以一定要看看结果。

### 概念上的区别
简而言之，在概念上inline和block-level元素有以下区别：
- **Content model**
总体来说，行内元素也许仅仅包含数据和其他的行内元素。**你不能在inline elements内部添加block elements**。
- **Formatting**
默认情况下，行内元素不会在文档流的开始位置强制生成一个新行。Block元素的话，正好与之相反，会很显著地break一行(但是，这可以使用CSS去改变。)

### 块级元素列表
下面的元素默认是inline的。
```
<a>
<abbr>
<acronym>
<b>
<bdo>
<big>
<br>
<button>
<cite>
<code>
<dfn>
<em>
<i>
<img>
<input>
<kbd>
<label>
<map>
<object>
<q>
<samp>
<script>
<select>
<small>
<span>
<strong>
<sub>
<sup>
<textarea>
<time>
<tt>
<var>
```
https://developer.mozilla.org/en-US/docs/Web/HTML/Inline_elements

### 深入理解block-level元素

HTML元素通常要么是"block-level"元素，要么是"inline elements"。
- 一个block-level元素，会占据它的父元素，也就是容器的全部空间，从而新建了一个"block"。这篇文章帮助我们去解释这意味着什么。

浏览器一般会在block-level元素前后展示一个新行。我们可以把它想象成一个box stack(盒子栈)。

>一个block-level元素通常从一个新的行开始，占据可用空间的全部宽度。(从左到右尽可能的拉伸)。

下面的例子示范了block-level元素的影响：

**HTML**
`<p>This paragraph is a block-level element; its background has been colored to display the paragraph's parent element.</p>`

**CSS**
`p { background-color: #8ABB55; }`

![image](https://user-images.githubusercontent.com/19262750/40956726-5a3be70c-68c4-11e8-8e39-da0dde23a651.png)

### 用法
- Block-Level 元素只能在<body>元素内部生效。

### Block-level vs inline

这里有一对行内元素和块级元素之间的区别。

- Formatting
默认情况下，block-level元素从新的一行开始，但是inline元素可以从一行内的任何位置开始。
- Content model
总体而言，**block-level元素也许包含inline元素和其他的block-level元素。**从这个角度去看，block-level元素要比inline 元素"larger"。

block-level 与inline 元素之间的区别用于HTML4.01以下的规范。

在HTML5中，这个两者之间的区别，由于content categories所以变得更为复杂了。
- "block-level"目录大致对应于HTML5中的flow content
- "inline"目录大致对应于HTML5中的phrasing content等

### 块级元素
下面的列表是一个复杂的HTML块级元素列表（尽管"block-level"在HTML5中没有明确定义给任何元素。）
```
<address>
Contact information.//联系信息

<article>
Article content.//文章内容

<aside>
Aside content.//旁边内容

<blockquote>
Long ("block") quotation.//长块引用

<canvas>
Drawing canvas.//绘制canvas

<dd>
Describes a term in a description list.//在说明列表中介绍单个term

<div>
Document division.//文档格

<dl>
Description list.//说明列表

<dt>
Description list term.//说明列表某一项

<fieldset>
Field set label.//设置label的field

<figcaption>
Figure caption.//图标题

<figure>
Groups media content with a caption (see <figcaption>).//媒体内容与标题分开

<footer>
Section or page footer.//部分或者是page的页脚

<form>
Input form.//输入表单

<h1>, <h2>, <h3>, <h4>, <h5>, <h6>
Heading levels 1-6.//标题

<header>
Section or page header.//章节或者page的页头

<hgroup>
Groups header information.//Group的头部信息

<hr>
Horizontal rule (dividing line).//横向规则

<li>
List item.//列表条目

<main>
Contains the central content unique to this document.//包含当前文档的中心唯一内容

<nav>
Contains navigation links.//包含导航链接

<noscript>
Content to use if scripting is not supported or turned off.//如果scripting不被支持或者关闭时使用

<ol>
Ordered list.//有序列表

<output>
Form output.//表单输出

<p>
Paragraph.//段落

<pre>
Preformatted text.//预先格式化的文本

<section>
Section of a web page.//一个web page的章节

<table>
Table.//表格

<tfoot>
Table footer//表格脚

<ul>
Unordered list.//无序列表

<video>
Video player.//视频播放器
```

https://developer.mozilla.org/en-US/docs/Web/HTML/Block-level_elements

### 深入理解display属性
display CSS属性指明了元素渲染盒子的类型。在HTML中，默认display属性值取自HTML规范或浏览器/用户默认样式表中描述的行为。XML默认值是inline，包括SVG元素也是。

除了各种各样的display盒类型之外，none 值使得你关闭元素的展示。当你使用none，所有的后裔元素也都关闭了展示。**尽管element不能在document tree中展示，但是它依然会被渲染。**

关于display值，有以下几种分类：
- `<display-outside>`
指明了元素外部的展现类型，这基本是它在流式布局中的作用。例如inline，block
- `<display-inside>`
指明了元素内部的展现类型，定义了布局formatting context的类型（假设它是一个不可替换的元素）。例如table，flex，grid
- `<display-listitem>`
元素生成了用来存放内容的块盒子以及一个分离的list-item行内盒子。例如list-item
- `<display-internal>`
一些布局模型，例如table和ruby，有复杂的内部结构，有几个不同的角色，他们的孩子和后代可以填补。这一步跟定义了那些"internal"display值，它只在特定的layout mode下才生效。
除非有特殊声明，否则inner和outer display类型都可以设置为下面的关键字。
例如：table-row-group，table-header-group等等
- `<display-box>`
是否展示box。例如none，contents（实验中）
- `<display-legacy>`
CSS2 使用单关键字的语法为display属性设置值，在同一个layout模型中，为块级元素和行内元素引用分离的关键词。例如inline-block，inline-table等等。
- Global

最好只为display设置一个值，因为组合的方式还不被一些浏览器支持。

关于这部分，我将挑选常见的inline，block，table，inline-block，flex以及grid进行举例。
- `<display-outside>`
  - inline 这个元素生成一个或者多个行内盒子
  - block 这个元素生成一个块元素盒子
- `<display-inside>`
  - table 元素就像`<table>` HTML标签一样表现。它定义了一个block-level盒子。
  - **flex 元素表现形式是块级元素，根据flexbox model绘制自己的内容**。
  - grid 元素表现得像块级元素，根据grid model绘制自己的内容。
- `<display-legacy>`
  -  **inline-block 元素生成一个块元素盒子，它会被周围的内容包裹，就仿佛它是一个单inline box一样。等价于inline flow-root**。

由于display-legacy类型较为有趣，因此将其全部进行翻译。

- inline-table inline-table在HTML中没有直接的mapping。它表现地就像<table> HTML标签一样，但是是一个inline box，而不是block-level盒子。在table盒子内部是块级内容。等价于inline table。
- inline-flex 行内元素表现，布局内容时根据flexbox模型布局。等价于inline flex。
- inline-grid 行内元素展现，布局内容时根据grid模型。

由于display-inside部分的flow-root于inline-block有关，因此也添加进来。
- flow-root 元素生成一个block元素，以建立一个新的块格式上下文(Block Formatting Context)。

### 无障碍考虑
- display：none
使用无显示值或元素上的内容将从可访问性树中删除它。这将导致元素及其所有后代元素不再通过屏幕阅读技术进行通告。 如果你想直观地隐藏元素，更容易获得的替代方法是使用一组属性将它从屏幕上直观地移除，但通过辅助技术（如屏幕阅读器）保持它可解析。
- Tables
将表格元素的显示值更改为块，网格或弹性将改变其在辅助功能树中的表示。这将导致表格不再被屏幕阅读技术正确地宣布。

https://developer.mozilla.org/en-US/docs/Web/CSS/display

### Summary
- **行内元素和块级元素有什么异同？**
Content Model和Formatting上的区别，
  - 块级元素的后代元素既可以是行内元素也可以是块级元素，行内元素的后代只能是行内元素。
  - 块级元素会格式化成3行，before，current，after，行内元素仅在当前行显示，不会破坏格式。
- **css属性display如何影响元素？**
大多数情况是通过修改`<display-outside>`，`<display-inside>`这两个属性值去改变元素外部的**流式布局**和元素内部的**formatting context**，其中formatting context会根据model类型进行渲染。

因此文章开头的问题就迎刃而解了。
- **display值从inline更改为block的原理？**
本质上是改变了`<display-outside>`，改变了元素的流式布局，也就是从生成1个inline盒子，变为生成一个block盒子。反之同理。
- **display值从block更改为inline-block的原理？**
本质上是改变了`<display-outside>`，`<display-inside>`。修改前为block null，修改后为inline flow-root，从生成一个block盒子，变为生成一个inline盒子，同时这个盒子按照flow-root的方式渲染(创建一个新的block formatting context)。

### Q&A
上文提到了model，display值为flexbox时为flexbox model，display值为grid时为grid model。由此引申出2个问题：
- 什么是flexbox model和grid model？
CSS3的flexbox model，和grid model。
- display值为inline，block以及inline-block时根据什么model进行渲染？
inline box model，block box model，inline block(flow-root BFC) model。
- 总计有多少种model，有何异同？
粗略理解为inline，block，flexbox，girdbox等等。实际上是由`<display-outside>`，`<display-inside>`共同决定。