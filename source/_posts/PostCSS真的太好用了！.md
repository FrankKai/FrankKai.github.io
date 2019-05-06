---
title: PostCSS真的太好用了！
date: 
tags: 
---

在[PostCSS官网](http://postcss.org/)有着这样的对PostCSS特性介绍，箭头后面是对应功能的插件及其github地址。

- increase code readability → [Autoprefixer](https://github.com/postcss/autoprefixer)
- Use tomorrow's CSS ,today! → [postcss-cssnext](https://github.com/MoOx/postcss-cssnext/)
- The end of global CSS → [postcss-modules](https://github.com/css-modules/postcss-modules)
- Avoid errors in your CSS → [stylelint](https://github.com/stylelint/stylelint)
- Powerful grid CSS → lost →[lost](https://github.com/peterramsing/lost)
 
PostCSS是一款使用插件去转换CSS的工具，有许多非常好用的插件，例如autoprefixer,cssnext以及CSS Modules。而上面列举出的这些特性，都是由对应的postcss插件去实现的。而使用PostCSS则需要与webpack或者parcel结合起来。
在Parcel中使用PostCSS的方法：添加配置文件.postcssrc(JSON),.postcssrc.js或者是postcss.config.js。
拿 .postcssrc 文件来举例：
```
{
  "modules": true,
  "plugins": {
    "autoprefixer": {
      "grid": true
    }
  }
}
```
Plugins 在 plugins 对象中被指定为 key，并使用对象的值定义选项。如果插件没有选项，只需将其设置为 true 即可。
下面我将对根据官方readme的演示demo，对各个插件进行一一介绍，并添加一些隐藏在插件背后的知识点的说明。

**1.什么是Autoprefixer？**
首先明确一点Autoprefixer是一个根据can i use解析css并且为其添加浏览器厂商前缀的PostCSS插件。
不加任何vender prefix的通常写法。
```
::placeholder {
    color: gray;
}
```
Autoprefixer将使用基于当前浏览器支持的特性和属性数据去为你添加前缀。你可以尝试下Autoprefixer的demo：http://autoprefixer.github.io/
![image](https://user-images.githubusercontent.com/19262750/39682933-3197f62e-51e5-11e8-87f8-e9fd5e645708.png)
由上图可以看出，像没有浏览器差异已经完全符合W3C标准的css2.1属性display，position等，Autoprefixer不会为其加前缀，而像css3属性transform就会为其加前缀，其中--webkit是chrome和safari前缀，--ms则是ie的前缀，像firefox由于已经实现了对transform的W3C标准化，因此使用transform即可。

因此Autoprefixer是一个非常有用的加速前端开发的一个工具，但是它需要基于PostCSS去发挥作用。

如果对vender prefix存疑，可以去看我的这篇博客：[rem / Vender Prefix / CSS extensions](https://www.jianshu.com/p/998f0d6bb7ac)

**2.什么是postcss-cssnext？**
postcss-cssnext语法input：
```
:root {
  --fontSize: 1rem;
  --mainColor: #12345678;
  --centered: {
      display: flex;
      align-items: center;
      justify-content: center;
  };
}
body {
    color: var(--mainColor);
    font-size: var(--fontSize);
    line-height: calc(var(--fontSize) * 1.5);
    padding: calc((var(--fontSize) / 2) + 1px);
}
.centered {
    @apply --centered;
}
```
浏览器可用语法output:
```
body {
    color: rgba(18, 52, 86, 0.47059);
    font-size: 16px;
    font-size: 1rem;
    line-height: 24px;
    line-height: 1.5rem;
    padding: calc(0.5rem + 1px);
}
.centered {
    display: -webkit-box;
    display: -ms-flexbox;
    display: flex;
    -webkit-box-align: center;
        -ms-flex-align: center;
            align-items: center;
    -webkit-box-pack: center;
        -ms-flex-pack: center;
            justify-content: center;
}
```
粗略看了一遍演示demo，http://cssnext.io/playground/，感觉既好用又不好用。
好用的地方在于通过var()和calc()进行css属性值的计算，也有@apply这样的应用大段规则的写法，也可以借此去了解一些新的css草案特性；不好用的地方在于有一定的学习成本，而且在前期与webpack，gulp以及parcel进行结合时也需要花费一定时间，并且如果有新的前端组成员加入，必须要掌握这种cssnext的语法。
这样做有些似乎在尝试将css变为一种可以进行逻辑处理的语言，但是我个人认为这对于css这样的灵活的需要具象思维并且需要大量调试的语言来说，工作中使用cssnext不是一个很好的选择，但是工作之余可以作为一个学习新的css草案特性的一个入口，待到纳入标准再去使用。

刚开始对自己的想法不确定，因此去看了下前辈们的想法，其中顾铁灵对cssnext的想法与我的想法如出一辙：
>CSS 的转译器（transpiler），根据目前仍处于草案阶段、未被浏览器实现的标准把代码转译成符合目前浏览器实现的 CSS。类似 ES6 的 Babel。
相比之下，Autoprefixer 更加具有实用价值，而 cssnext 实现的功能以后浏览器会怎么实现还存疑，感觉只能玩玩。

**3.什么是postcss-modules？**
在看postcss-modules之前，首先要明确的是CSS Modules的这个概念，关于CSS Modules，可以阅读我翻译的一篇文章：[【译】什么是CSS Modules ？我们为什么需要他们？](https://github.com/FrankKai/FrankKai.github.io/issues/45)

postcss-modules则是CSS Modules在PostCSS上的实现插件，这里有一篇插件作者本人写的介绍postcss-modules的文章：[PostCSS-modules:make CSS great again!](https://evilmartians.com/chronicles/postcss-modules-make-css-great-again)。

在我有限的开发经验中，只在react中使用过css modules，在vue和angularjs中都没用到过，而且在react中使用时，不会去用postcss-modules这个插件，而是使用[react-css-modules](https://github.com/gajus/react-css-modules)这个CSS Modules思想在react中的插件。

下面将给出最简单的入门例子：
在React上下文中，CSS Modules可能像下面这样写：
```
import React from 'react';
import styles from './table.css';

export default class Table extends React.Component {
    render () {
        return <div className={styles.table}>
            <div className={styles.row}>
                <div className={styles.cell}>A0</div>
                <div className={styles.cell}>B0</div>
            </div>
        </div>;
    }
}
```
最后渲染出的组件的标签会是如下形式：
```
<div class="table__table___32osj">
    <div class="table__row___2w27N">
        <div class="table__cell___1oVw5">A0</div>
        <div class="table__cell___1oVw5">B0</div>
    </div>
</div>
```
如果对为什么会把class名编译成table__table___32osj这样的形式存在疑惑，需要先把css modules搞清楚：[【译】什么是CSS Modules ？我们为什么需要他们？](https://github.com/FrankKai/FrankKai.github.io/issues/45)

如果需要在开发环境或者生产环境结合webpack去使用，那么可以阅读[react-css-modules](https://github.com/gajus/react-css-modules)的官方文档寻找答案。

通过这次探索我们可以发现，前端开发在不同的生态，或者说框架体系下，同一个技术，例如CSS Modules这种将思想，会有对应的实现方式，而我们要掌握的，不仅仅是在某种框架下配置使用的方法，而是这种开发思想。因为学习的核心在于学习知识，而不是无休止的学习工具。

**4.什么是stylelint？**
这是用来强制开发人员按照团队css规范写css样式的工具，类似eslint。
若想使用，只需要去启用规则即可。

节选一段stylelint作者博文中的话：
>没错，你的团队可能在某个地方的某条纯文本wiki中记录了团队的代码样式规范。但是，不容忽视的是人的因素：人总是会犯错——总是在无意之间。
而且即使你很自律地执着遵循某个规范的代码风格，但是你没办法确保你的同事或是你的开源项目的贡献者能够像你一样。没有linter的帮助，你必须人工检查代码样式和错误。而机器存在的意义就是代替人来完成能够自动化实现的任务。linter就是这样的机器，有了linter，你不需要浪费时间检查代码风格，也不需要对每一个代码错误都写一大堆的注释，因此它能够极大程度地减少你花费在代码审阅上的时间。你无须检查代码究竟做了些什么，也无需关心它看起来什么样。

事实与stylelint作者说的是一样的，即使团队有前端开发规范，也会不经意间写出不符合规范的代码，因为每次写css规则前都去规范check一遍不是谁都能做到的，如果团队再没有code review这一关的话，写出各种各样风格的css代码就是一件必然的事了，短期没有什么影响，当项目变得庞大起来，增加新功能或者重写旧功能将会是一件很痛苦的事。

我们主要去关注[Rules部分](https://github.com/stylelint/stylelint/blob/master/docs/user-guide/rules.md#possible-errors)：
sytlelint的规则主要有3类，我将从每一类规则中挑一个拿出来作为示例。
- Possible errors(常见的错误写法，强烈推荐开启)
- Limit language features(弃用一些正确的写法，中等推荐开启)
- Stylistic issues(代码风格代码统一，普通建议开启)

- Possible errors ------ color-no-invalid-hex: 禁止无效的十六进制颜色
完全形式的十六进制颜色可以是6或者8（7，8位是透明度的值）个字符。同样他们的缩写可以是3或者4个字符。
options : true
下面的代码违反规则：
```
a { color: #00; }
a { color: #fff1az; }
a { color: #12345aa; }
```
下面的代码符合规则:
```
a { color: #000; }
a { color: #000f; }
a { color: #fff1a0; }
a { color: #123450aa; }
```
- Limit language features ------ color-no-hex:禁止使用十六进制颜色
options : true
十六进制的颜色违反规则：
```
a { color: #000; }
a { color: #fff1aa; }
a { color: #123456aa; }
```
无效的十六进制色同样违规：
```
a { color: #foobar; }
a { color: #0000000000000000; }
```
下面的是符合规则的：
```
a { color: black; }
a { color: rgb(0, 0, 0); }
a { color: rgba(0, 0, 0, 1); }
```

- Stylistic issues ------ color-hex-case: 自动将十六进制色转换为大写或者小写
Options string: "lower"|"upper"
可以使用`stylelint "foo/*.css" --fix`实现同样的功能。
"小写"
下面的代码是违规的:
```
a { color: #FFF; }
```
下面的是符合规则的：
```
a { color: #000; }
a { color: #fff; }
```
"大写"
下面的代码是违规的:
```
a { color: #fff; }
```
下面的是符合规则的：
```
a { color: #000; }
a { color: #FFF; }
```
更多的stylelint的规则可以查阅：https://github.com/stylelint/stylelint/blob/master/docs/user-guide/rules.md#possible-errors

**5.什么是LostGrid？**
Lost Grid是一个强大的PostCSS网格系统，可与任何预处理器甚至是原生CSS一起使用。
在这里有非常好的demo展示：http://lostgrid.org/lostgrid-example.html

节选2个展示进行说明。
```
.ColumnSection__grid div {
    lost-column: 1/1;
}
@media (min-width: 400px) {
    .ColumnSection__grid div {
        lost-column: 1/3;
    }
}
@media (min-width: 900px) {
    .ColumnSection__grid div {
        lost-column: 1/6;
    }
}
```
大于900px时：
![image](https://user-images.githubusercontent.com/19262750/39757437-693166c2-52ff-11e8-8d3e-24ea70a90ffa.png)
小于900px时：
![image](https://user-images.githubusercontent.com/19262750/39757447-713e6770-52ff-11e8-9621-5560551ce383.png)
.NestingSection__grid {
    background: #8eb19d;
}

.NestingSection__grid div {
    background: #7ba48d;
    lost-column: 1/3;
}
```
.NestingSection__grid div div {
    background: #68977c;
    lost-column: 1/2;
}
```
![image](https://user-images.githubusercontent.com/19262750/39757518-ab15687c-52ff-11e8-918b-7cd715073849.png)

经过查看css样式我们发现，其实就是巧用了table布局，before/after伪元素，以及css选择器，以及border-box布局，但其实最最核心的地方还是在于很好的使用了css本身具有的流式布局以及BFC，针对各种情况，在插件内部使用了大量的样式进行约束。
![image](https://user-images.githubusercontent.com/19262750/39757867-b17b9a50-5300-11e8-9a95-655c05832e90.png)

在css3的flex布局和grid布局逐渐被浏览器所支持的今天，我个人建议不使用LostGrid插件。