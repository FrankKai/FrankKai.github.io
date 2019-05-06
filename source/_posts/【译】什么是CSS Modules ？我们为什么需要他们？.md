---
title: 【译】什么是CSS Modules ？我们为什么需要他们？
date: 
tags: 
---

原文地址：https://css-tricks.com/css-modules-part-1-need/

最近我对CSS Modules比较好奇。如果你曾经听说过他们，那么这篇博客正适合你。我们将去探索它的目的和主旨。如果你同样很好奇，敬请关注，在下一篇博文中我们将介绍如何使用CSS Modules。如果你想亲自尝试并且掌握如何使用，第三部分将会很适合你，这一部分剖析了如何在React环境中使用。

# 什么是CSS Modules？
根据官方的repository介绍，CSS Modules是：
> 所有的class的名称和动画的名称默认属于本地作用域的CSS文件。
所以CSS Modules不是一个官方的规范，也不是浏览器的一种机制，它是一种构建步骤中的一个进程。（构建通常需要webpack或者browserify的帮助）。通过构建工具的帮助，可以将class的名字或者选择器的名字作用域化。（类似命名空间化。）

这到底是什么呢？我们为什么要这么做呢？我们很快就进行介绍。首先，不要忘记HTML和CSS的工作原理。在HTML中一个类添加：
```
<h1 class="title">An example heading</h1>
```
在CSS中这个class的定义如下：
```
.title {
    background-color: red;
}
```
只要CSS被添加到HTML文档上，那个`<h1>`的背景色就是红色。我们不需要人为处理CSS和HTML文件。浏览器本身自己就理解这些文件的格式。

CSS Modules 和上面的方法不一样。我们不写纯HTML，我们需要在一个类似index.js这样的Javascript 文件中取写我们所有的标签。这里有一个例子来说明这是怎么回事（我们之后将会去看更多真实的实例）：
```
import styles from "./styles.css";

element.innerHTML = 
    `<h1 class="${styles.title}">
        An example heading
      </h1>`;
```
在我们构建的步骤中，编译器将会搜索我们导入的styles.css文件，然后到我们刚刚写的js文件中，通过`styles.title`使得.title class可用。我们的构建步骤将会同时处理这些东西成为新的，分离的HTML和CSS文件，并且用一个新的字符串去替换HTML和CSS选择器的class。

通过构建工具生成的HTML也许像下面这样：
```
<h1 class="_styles__title_309571057">
    An example heading
</h1>
```
通过构建工具生成的CSS也许像下面这样：
```
._styles__title_309571057{
    background-color: red;
}
```
![image](https://user-images.githubusercontent.com/19262750/39440393-57da9b50-4cdd-11e8-86fc-ad5471984794.png)
class属性和.title选择器已经完全不见了，取而代之的是这个全新的字符串；我们的源CSS文件也没有为浏览器提供服务。
就像Hugo Griaudel在他的这一模块的教程中所说：
> [the classes]是动态生成的，唯一的，而且和当前的样式有映射关系的。

这就是样式也有作用域的原因。它们的作用域是特定的模板。如果我们有一个buttons.css文件我们将在buttons.js模板中导入它，并且在css文件内的.btn class相对于其他模板(例如forms.js)也是不可用的，除非我们在这个文件中同样导入了进来。

为什么我们想要把CSS和HTML搞成这样？我们这样做的真真正正的原因是什么？

# 为什么我们要用CSS Modules?
有了CSS Modules，就可以确保所有的样式能够服务于单个组件：
1.  集中在一个地方
2. 只应用于那个组件，其他组件不适用
除此之外，任何组件都能拥有真正的依赖，就像下面这样：

```
import buttons from "./buttons.css";
import padding from "./padding.css";

element.innerHTML = `<div class="${buttons.red}${padding.large}">`;
```
**这样设计的目的在于解决CSS中的*全局作用域*问题。**

你曾有过为了提升效率，节省时间去简略的写css吗？而且是在完全不考虑你会不会影响其他代码的情况下？

你曾有过在样式表的底部随机打了一些的比特和垃圾，然后尝试回过头来重新去组织但是从来没这么做吗？

你曾有过看到样式却不完全知道它的意义的时候吗？即使它们被用在了当前的标签上？

你曾有过思考如何去不破坏任何东西的情况下，去弃用一些现有的样式吗？考虑过这些样式是仅仅作用于自己还是依赖其它样式呢？或者是在哪里重新覆盖了样式了？

这些问题会让人很头痛，项目时间紧张，而你的心思又在窗外的花花世界。

但是当你有了CSS Modules之后，关键是这种默认本地作用域的概念，这个问题将会被避免。你必须去思考写样式的方便性。

例如，如果你在不应用CSS module-style class去做转换的情况下，在HTML中使用random-gross-class，这个样式将不会被应用，因为这个选择器将会被转换为._style_random-gross-class_0038089.

# composes 关键词
我们现在拥有一个叫做type.css的模块去渲染text样式。在那个文件中，我们也许会有如下代码：
```
.serif-font {
    font-family: Georgia,serif;
}
.display {
   composes: serif-font;
    font-size: 30px;
    line-height: 35px;
}
```
我们将在我们的模板中声明class：
```
import type from "./type.css";
element.innerHTML = 
`
    <h1 class="${type.display}">
        This is a heading
    </h1>;
`
```
编译后的模板上的标签会是下面这样：
```
<h1 class="_type__display_0980340 _type_serif_404840">
    Heading title
</h1>
```
使用composes关键词汇将2个class都绑定到元素上，从而避免了类似解决方案的一些问题，类似Sass中的@extend。

我们甚至可以在一个分离的CSS文件中去compose，从而实现继承。
```
.element{
    compose: dark-red from "./colors.css";
    font-size: 30px;
    line-height: 1.2;
}
```

# 不需要BEM
在构建CSS module的过程中，不需要BEM。有2个原因：
1. 易解析- 类似type.display这样的代码，对于开发者来说就像BEM-y的.font-size__serif--large。当BEM选择器变长时，可能更容易被理解。
2. 本地作用域- 比如我们在模块中有一个类似.big去修改font-size属性的class。同样我们可能还会用.big去同时增大padding和font-size。这没关系！他们不会冲突，因为作用域中的样式是有各自意义的。甚至一个module引入2个样式表，然后它有通常的名字我们的构建工具会为那个class加上前缀作区分。换句话说：CSS modules消除了特殊性问题。

很酷，难道不是吗？

这些仅仅是CSS Modules的部分优点。

如果你想学习更多，Glen Madden写了更多的这样做的好处。

这个系列的下一篇文章将会去探索如何在项目中使用Webpack和CSS Modules。我们将使用最新的ES2015的特性去实现，也会给出一些代码例子去引导大家去理解。