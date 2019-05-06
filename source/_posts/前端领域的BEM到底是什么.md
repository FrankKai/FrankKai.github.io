---
title: 前端领域的BEM到底是什么
date: 
tags: 
---

在这篇[【译】什么是CSS Modules ？我们为什么需要他们？](https://github.com/FrankKai/FrankKai.github.io/issues/45)的结尾处，明确指出CSS Modules不需要BEM，那么BEM到底是什么呢？

下面我将把[BEM官网](http://getbem.com/)的教程翻译出来，带领大家搞清楚前端领域的BEM到底是什么。

BEM - Block Element Modfier（块元素编辑器）是一个很有用的方法，它可以帮助你创建出可以复用的前端组件和前端代码

它有如下3个特性：
- 易用性 如果想使用BEM，你只需要采用BEM命名规范即可
- 单元性 独立的块和CSS选择器，可以使得你的代码可重用和单元化
- 灵活性 使用BEM之后，方法和工具可以按照你喜欢的方式去组织和配置

下面将从3个方面来分析BEM到底是什么？
- 介绍
BEM是一个高可用的，强大的，而且简单的命名规范，它可以使得你的前端代码更加易读和理解，容易与他人协作，容易扩展，更加强壮和明确，关键是更加严谨。
- 命名
BEM的方法，可以确保参与网站开发的每一个人，都能够使用一个代码库并且使用同一种语言。使用BEM格式的命名规范，可以从容应对需求变更。
- FAQ
 - 为什么不适用语义化的标签而是直接去使用为块元素添加CSS class呢？
 - 我可以像button.button的方式去组合标签和class到同一个选择器中吗？

还要更多的疑惑，可以查看FAQ列表！

### 介绍
在小型手册网站上，如何组织你的风格通常不是一个大问题。你进入那里，写一些CSS，或者甚至是一些SASS。用SASS的生产设置将它编译成一个样式表，然后将它聚合到一起，将模块中的所有样式表都变成一个很好的整洁包。

但是，当涉及到更大，更复杂的项目时，组织代码的方式至少有以下三种方式是提高效率的关键：它影响编写代码需要多长时间，需要多少代码写和你的浏览器将不得不做多少加载。当你与团队合作并且高性能是必不可少的时候，这变得尤为重要。

这对于使用遗留代码的长期项目也是如此（请参阅“如何使用Sass和SMACSS来扩展和维护传统CSS” - 一些不错的SMACSS和BEM混合）。

方法
目前有很多方法可以减少CSS占用空间，组织程序员之间的合作以及维护大型CSS代码库。这在Twitter，Facebook和Github等大型项目中很明显，但其他项目通常会很快成长为“巨大的CSS文件”状态。

- OOCSS  用CSS“对象”分隔容器和内容
- SMACSS 风格指南为您的CSS编写CSS规则的五个类别
- SUITCSS 结构化类名和有意义的连字符
- 原子 将样式分解为原子或不可分割的部分
为什么BEM比其他人更好？
无论您选择在项目中使用哪种方法，您都将受益于更多结构化CSS和UI的优势。一些风格不那么严格和更灵活，而另一些则更容易理解和适应团队。

>我选择BEM而不采用其他方法的原因归结为：它比其他方法（即SMACSS）更容易混淆，但仍为我们提供了我们想要的良好体系结构（即OOCSS）以及可识别的术语。
Mark McDonnell，BEM可维护的CSS

Blocks，Element和Modifier
听到BEM是方法论的关键元素 - Block，Element和Modifier的缩写，您一定不会感到惊讶。BEM严格的命名规则可以在这里找到。

- 块 
    - 独立的实体，它本身是有意义的。
    - 例子 header，container，menu，checkbox，input
- 元件
    - 块的一部分，没有独立的含义，并且在语义上与其块相关联。
    - 例子 menu item，list item，checkbox caption，header title
- 修改
    - 块或元素上的标志。用它们来改变外观或行为。
    - 例子 disabled，highlighted，checked，fixed，size big，color yellow
![image](https://user-images.githubusercontent.com/19262750/39481143-03c820e4-4d9d-11e8-85c7-f4bc755ebd7a.png)

深入理解
我们现在来看一个使用BEM的元素，是Github的button：
![image](https://user-images.githubusercontent.com/19262750/39479716-a730d6cc-4d98-11e8-9478-a924babe2f21.png)

我们可以使用一个普通按钮应对普通情况 ，再使用两种不同的应对不同情况。因为我们使用BEM的方式为Block 绑定了class，我们可以用任何想使用的标签去实现。（button，a 甚至是div）。但是命名规则告诉我们需要使用block-modifier-value的语法。
HTML:
```
<button class="button">
	Normal button
</button>
<button class="button button--state-success">
	Success button
</button>
<button class="button button--state-danger">
	Danger button
</button>
```
CSS:
```
.button {
	display: inline-block;
	border-radius: 3px;
	padding: 7px 12px;
	border: 1px solid #D5D5D5;
	background-image: linear-gradient(#EEE, #DDD);
	font: 700 13px/18px Helvetica, arial;
}
.button--state-success {
	color: #FFF;
	background: #569E3D linear-gradient(#79D858, #569E3D) repeat-x;
	border-color: #4A993E;
}
.button--state-danger {
	color: #900;
}
```
优点：
- 单元性
块的样式从来不依赖同页面其它的元素，所以你永远不会遇到级联问题。您还可以将完成的项目中的块转移到新项目中。
- 重用性
不同方式组织独立的块，并智能地重用它们，可以减少必须维护的CSS代码量。通过一系列风格指南，您可以构建一个块库，使您的CSS超级有效。
- 结构化
BEM方法可以使得你的CSS代码结构性很好，从而更加容易理解。

拓展阅读
‘Why BEM?’ in a nutshell
MindBEMding — getting your head ’round BEM syntax
CSS guidelines
BEM methodology for small projects
BEM It! for Brandwatch
Used and Abused — CSS Inheritance and Our Misuse of the Cascade.
Objects in Space — A style-guide for modular SASS development using SMACSS and BEM
How to Scale and Maintain Legacy CSS with Sass and SMACSS
Building a modular My Health Skills with BEM and Sass
Building My Health Skills — Part 3

案例分析：
我们希望尽快撰写“如何将现有项目迁移到BEM”。与此同时，您可以观看Nicole Sullivan的精彩演讲 - “[CSS预处理器性能](https://www.youtube.com/watch?v=0NDyopLKE1w)”。她很好地概述了她在大多数网站遇到的问题，并提供了跟踪和处理这些问题的方法。

### 命名
>在计算机科学领域，只有2个非常难解决的问题：一个是缓存失效，而另一个则是命名。-Phil Karlton

有一个好的样式指导手册将会显著提高开发的速度，debug的速度，以及在原有代码上完成新功能的效率。然而不幸的是，大多数的CSS 代码在开发过程中毫无任何结构和命名的规范。从长远角度来看，这会导致最后产生的CSS代码库很难维护。

BEM方法确保每一个参加了同一网站开发项目的人，基于一套代码规范去开发，这样非常有利于团队成员理解彼此的代码，而且对于后续接手项目的同学来说，也是一件好事。

**Block**
- 封装一个只对自己有意义的实体。当blocks能够被嵌套而且彼此之间可以交互的时候，语义上他们是等价的；没有层级关系。没有DOM表示的整体实体。（例如控制器和模型也可以是块）
- Naming Block的名字包含拉丁字母，数字和句号。当组合一个完整CSS class的时候，可以增加一个短的前缀：.block
- HTML 任何DOM节点可以作为一个Block，只要它接受一个class名。`<div class="block">...</div>`
- CSS 
 - 只使用class名选择器
 - 没有标签名和或者id
 - 同一页面里，不依赖其它的block或者element
 - 例如`.block {color:#042;}`

**Element**
- Block的一部分，当把它独立取出来时，没有任何实际意义。任何元素在语义上都是它自己的block紧密相连的。
- Naming Element的名字可能包含拉丁字母，数字，句号以及下划线。CSS class名由block名成加两个下划线再加element的名字，最后组织成一个块名。
- HTML 任何的在Block中的DOM节点，都是一个element。在一个已知的block中，所有的element在语义上都是相等的。
```
<div class="block">
    <span class="block__name"></span>
</div>
```
- CSS 
 - 只选择class名字作为选择器
 - 没有标签名或者id
 - 不依赖当前页面的其它block或者element
Good
```
.block__elem{color:#042};
```
Bad
```
.block .block__elem {color:#042;}
div.block__elem {color:#042;}
```

**Modifier**
- block或者element的flag。使用他们可以改变样式，行为或者是状态。
- Naming Modifier的名字可以包含拉丁字母，数字，句号以及下划线。CSS的class可以由block或者element名称后面加--组成，例如.block--mod或者.block__elem--mod，以及.block--color-black .block--color-red。复杂的modifier里由短线替代空格。
-HTML Modifier是一个额外的加在block或者element class名之后一个class 名。只为他们负责修改的blocks或者elements添加class，然后保持原有的class不变。
Good
```
<div class="block block--mod">...</div>
<div class="block block--size-big block--shadow-yes">...</div>
```
Bad
```
<div class="block--mod">...</div>
```

CSS
- 使用modifier类名作为选择器 .block--hidden { }
- 基于block级的modifier修改元素 .block--mod .block__elem { }
- 元素修改器 .block__elem--mod { }

例子：
假设你的form Block由modifier：theme:"xmas" 以及simple:true 以及element:input 和submit，submit自己的modifier是disabled:true。
HTML:
```
<form class="form form--them-xmas form--simple"
    <input class="form__input" type="text" />
    <input class="form__submit form__submit--disabled" type="submit" />
</form>
```
CSS:
```
.form {}
.form--theme-xmas {}
.form--simple {}
.form__input {}
.form__submit {}
.form__submit--disabled {}
```