---
title: 我不知道的CSS，你知道吗？
date: 
tags: 
---

经历过几次面试后，才深知精通css对于前端开发的重要性，这个issue将集中针对css展开进攻，提升自己的薄弱的css基础。
参考资料：《CSS世界》[MDN](https://developer.mozilla.org/en-US/docs/Web/CSS) [CSS规范](https://www.w3.org/standards/techs/css#w3c_all)
验证工具：Chrome 本地HTML

![image](https://user-images.githubusercontent.com/19262750/36015989-ebcb0488-0dab-11e8-8f5e-7f34f352dc2a.png)
**1.ex是什么？**
这是一个相对长度单位，相对谁呢？相对的是"x"，x指的是英文小写字母的"爱克斯"，而不是拼音里的"西"。就是说1ex就等于一个英文字母"x"的长度，那么一个"x"的长度，也就是1ex等于多少呢？是个确切的数字吗？是因字体而异的，在许多字体中1ex ≈ 0.5em。

> chrome64: 8.64px
firefox57:   8.6333px 
ie11:          7.13px
edge41:     7.13px

下面是默认字体的1ex的高度，如果把浏览器字体都改为“Microsoft YaHei”，那么1ex就约等于8.6px。
思考：
暂时没有想到好的使用场景，如果是需要做一些相对当前使用字体的属性值的计算，可以去使用ex。
代码：
```
<!DOCTYPE html>
<html lang="zh-cn">
    <head>
        <meta charset="utf-8" />
        <title>ex是什么？</title>
        <style>
            html{font-family:"Microsoft YaHei"}
            h1{margin:10px 0;font-size:16px;}
            .x{overflow:hidden;height:1ex;background:#aaa;}
        </style>
    </head>
    <body>
        <h1>定义一条与字母x高度相同的线：</h1>
        <div>xxxx</div>
        <div class="x"></div>
    </body>
</html>
```

**2.ch是什么?**
现在已经有了垂直方向的相对长度单位ex了，那么水平方向有没有呢？当然是有的，ch就是这样一个水平方向的相对长度单位，它的大小等于当前字体的"0"（Unicode：U+0030）的宽度，微软雅黑字体的“0”的宽度为9.375px，也就是1ch ≈9.375px。
思考：
暂时没有想到好的使用场景，如果是需要做一些水平方向的相对当前使用字体的属性值的计算，可以去使用ch。
代码：
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>ch是什么？</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <style>
        html{
            /* font-size: 12px; */
            font-family: "Microsoft YaHei"
        }
        .traffic-light{
            height: 1rem;
            width: 1ch;
            background: rgba(255,0,0,1);
        }
    </style>
</head>
<body>
    <div class="traffic-light">
        0
    </div>
</body>
</html>
```

**3.resolution是什么？**
这是css中的数据类型，在媒体查询时用来描述一个分辨率。在屏幕上，与CSS中的英尺，厘米，或者像素有关，而与物理单位无关。
常见单位：dpi（dots per inch） dpcm（dots per cm） dppx（dots per px）。
转换关系：
1英尺等于2.53厘米，所以1dpcm ≈ 2.54dpi。
css中1in:1px = 1:96，所以1dppx = 96dpi。
例子：各种分辨率下的1像素的直线。
```
<div class="hairline-border">text</div>
.hairline-border {
  box-shadow: 0 0 0 1px;
}
@media (min-resolution: 2dppx) {
  .hairline-border {
    box-shadow: 0 0 0 0.5px;
  }
}
@media (min-resolution: 3dppx) {
  .hairline-border {
    box-shadow: 0 0 0 0.33333333px;
  }
}
@media (min-resolution: 4dppx) {
  .hairline-border {
    box-shadow: 0 0 0 0.25px;
  }
}
```

**4.不要用!important**
demo地址：http://jsbin.com/wakequje/1/edit?html,css,output
```
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>!important</title>
</head>
<body>
<div class="foo">
  Lorem <span class="bar">ipsum</span> dolor sit amet…
</div>
</body>
</html>
```
```
html {
  font: 2em/1.5 sans-serif;
}

.foo .bar {
  color: red;
}

.bar.bar.bar.bar.bar.bar.bar.bar{
  color: green; 
}
```
不提倡的写法：
```
.bar{
  color: green !important;
}
```
MDN优质资料：[优先级](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity)
> 使用 !important 是一个坏习惯，应该尽量避免，因为这破坏了样式表中的固有的级联规则，使得调试找bug变得更加困难了。

一些比!important更好的方法：
1.更好地利用css级联关系
2.增加元素权重，增加更详细的父级选择器
**3.当没有其他可以选择的父选择器时，重复多次选择自身增加权重（此comment的原因所在）**

其实只要写成
```
.bar.bar{
  color: green; 
}
```
就好了，多写几个.bar是为了提升代码可读性。

**5.子元素浮动导致父元素塌陷怎么处理？**
```
overflow:hidden;
```
or
```
display:inline-block;
```
为父元素添加`overflow:hidden`(常用)或者是`display:inline-block`（常用）。
除此之外，也可以是auto、scroll; table-cell、table-caption、inline-block。
或者position的值不为relative或static，或者float的值不为none，或者是<html>根元素。

本质上，这是激活了元素的BFC，所谓BFC，其实就是Block Format Context，如果一个元素具有BFC，那么内部子元素无论有什么骚操作，无论是float,absolute,fixed,z-index等等等等，都不会对外部的元素有任何影响。

本质上大家最常用的clear:both，也是为当前元素激活了BFC。

除此之外，BFC还可以去除margin重叠。

题外话：随着浏览器厂商对flex，grid兼容性的进一步支持，BFC依旧很重要，哈哈。

**6.::placeholder是什么意思？**
这是一个实验性的css技术，这个css伪元素代表这form元素的placeholder的文字内容。
只有适用于:: first-line伪元素的CSS属性的子集可以在其选择器中使用:: placeholder的规则中使用。
```
<input placeholder="Type something here!">
```
```
input::placeholder {
    color: red;
    font-size: 1.2em;
    font-style: italic;
}
```
![image](https://user-images.githubusercontent.com/19262750/39682818-b22b31f8-51e4-11e8-883a-c04e1e82fac9.png)

**7.:fullscreen是什么意思？**
这是一个实验性的技术。这个:fullscreen伪元素会使得元素在浏览器的全屏模式下去显示。

那么如何触发浏览器的全屏模式呢？

我们来看一个<video>标签的例子：
```
<video controls id="myvideo">
    <source src="somevideo.webm"></source>
    <source src="somevideo.mp4"></source>
</video>
```
我们可以使得视频通过如下方式进入全屏模式：
```
var elem = document.getElementById("myvideo");
if (elem.requestFullscreen) {
  elem.requestFullscreen();
}
```
再来看个普通标签的例子：
```
<div id="fullscreen">
    <h1>:fullscreen Demo</h1>
    <p>This text will become big and red when the browser is in fullscreen mode.</p>
    <button id="fullscreen-button">Enter Fullscreen</button>
</div>
```
```
<script>
var fullscreenButton = document.getElementById("fullscreen-button");
var fullscreenDiv    = document.getElementById("fullscreen");
var fullscreenFunc   = fullscreenDiv.requestFullscreen;
if (!fullscreenFunc) {
     ['mozRequestFullScreen', 'msRequestFullscreen','webkitRequestFullScreen'].forEach(function (req) {
        fullscreenFunc = fullscreenFunc || fullscreenDiv[req];
     });
}  
function enterFullscreen() {
    fullscreenFunc.call(fullscreenDiv);
} 
fullscreenButton.addEventListener('click', enterFullscreen);
</script>
```
普通模式下是这样：很普通，:fullscreen伪元素未激活。
![image](https://user-images.githubusercontent.com/19262750/39688713-96c12ae0-5206-11e8-9ff2-06926e3dab33.png)
全屏模式下是这样：很丰满，fullscreen伪元素的被激活。
![image](https://user-images.githubusercontent.com/19262750/39688677-66d2cd16-5206-11e8-8f1b-13e87b27bba5.png)

### 8.说一下position的区别
值集合：**static relative absolute fixed sticky**
关键词：**正常文档流 占用页面空间 参照元素偏移 副作用 z-index堆叠上下文 margin边缘重叠** 

- **static**
1. 在正常文档流中
2. 占用页面layout空间
3. top right bottom left不生效
4. 无副作用
5. z-index属性不生效

- **relative**
1. 在正常文档流中
2. 页面上的空间与static一致
3. 根据top right bottom left 参考自身进行偏移
4. 偏移不影响其他元素的位置
5. 当z-index值不为auto时，这个值创建了一个新的堆叠上下文
6. 对table-*-group，table-row，table-column，table-cell和table-caption的影响未定义。

- **absolute**
1. 从正常文档流移除
2. 不占用当前页面布局空间
3. 根据top right bottom left，参考最近的已定位的上级元素偏移，如果没有，则向上查找到最高层级的初始化包含块
4. 偏移不影响其他元素的位置
5. 当z-index不为auto时，创建新的堆叠上下文
6. 绝对定位的盒子不会与其他元素发生边缘重叠

- **fixed**
1. 从正常文档流移除
2. 不占用当前页面布局空间
3. 根据top right bottom left，参考viewport偏移；当它的上级元素中有transform，perspective，filter属性，这个元素被作为参考的对象。
4. 偏移不影响其他元素的位置
5. 直接创建一个新的堆叠上下文
6.在打印文档中，元素在每一页的相同位置

- **sticky (实验性)**
1. 在正常文档流中
2. 不占用当前页面布局空间
3. 根据top right bottom left ，参考最近的scrolling祖先，包含块以及table类元素定位，或者是最近的粘性机制元素。注意粘性元素与最近的有滚动机制的元素粘在一起。(当overflow值为hidden,scroll,auto或者是overlay)，即使在上级元素不是最近的祖先元素。
4. 偏移不影响其他元素位置
5. 直接创建新的堆叠上下文

通过**关键词记忆**的方式，对position区别记忆和理解的方式更佳。

参考：https://developer.mozilla.org/en-US/docs/Web/CSS/position