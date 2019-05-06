---
title: CSS3动画卡顿性能优化解决方案
date: 
tags: 
---

![CSS3 Transition](http://upload-images.jianshu.io/upload_images/2976869-aac6cda7e28a0158.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/512)

最近在开发小程序，与vue类似，它们都有生命周期这回事。
>onLoad  监听页面加载
onReady 监听页面初次渲染完成
onShow 监听页面显示

到底是什么意思？

所以这又触碰到了我的知识盲区，不过项目在磕磕绊绊中完成的差不多了，但是遇到了CSS3动画渲染的性能问题，所以我也是被逼的，再回过头来从浏览器渲染网页的流程出发，去找动画卡顿的症结。

浏览器渲染网页的流程如下：
>使用 HTML 创建文档对象模型（DOM）
使用 CSS 创建 CSS 对象模型（CSSOM）
基于 DOM 和 CSSOM 执行脚本（Scripts）
合并 DOM 和 CSSOM 形成渲染树（Render Tree）
使用渲染树布局（Layout）所有元素
渲染（Paint）所有元素


可以结合Alon的这篇[前端性能优化](http://jinlong.github.io/2017/05/08/optimising-the-front-end-for-the-browser/)和安卓开发者选项的显示页面布局。

安卓开发者选项的显示页面布局
>如何判断手机app是native，webview还是hybird?
简单说下，app中的一大块是白色的没有红线标记出来的，但是上面有按钮，图片等时，就是webview，也就是通过一个伪浏览器去请求到的数据，断网时打开app没有任何东西显示在上面

![小程序为hybird式开发](http://upload-images.jianshu.io/upload_images/2976869-99b715315895e7c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/256)


**onLoad 监听页面加载**
在渲染完界面之后，也就是通过.json中的配置项生成native界面后，开始渲染webview的部分，一个页面只会调用一次。
**onReady 监听页面初次渲染完成**
一个页面只会调用一次，代表页面已经准备妥当，可以和视图层进行交互。
**onShow 监听页面显示**
每次打开页面都会去调用其中的函数。

###我们的动画应该放在哪里？
应该放在onShow里，因为这样我每次打开都能看到动画。

###为什么会卡顿？

有一个前提必须要提，前端开发者们都知道，浏览器是单线程运行的。
但是我们要明确以下几个概念：单线程，主线程和合成线程。

虽然说浏览器执行js是单线程执行（注意，是执行，并不是说浏览器只有1个线程，而是运行时，runing），但实际上浏览器的2个重要的执行线程，这 2 个线程协同工作来渲染一个网页：主线程和合成线程。

一般情况下，主线程负责：运行 JavaScript；计算 HTML 元素的 CSS 样式；页面的布局；将元素绘制到一个或多个位图中；将这些位图交给合成线程。

相应地，合成线程负责：通过 GPU 将位图绘制到屏幕上；通知主线程更新页面中可见或即将变成可见的部分的位图；计算出页面中哪部分是可见的；计算出当你在滚动页面时哪部分是即将变成可见的；当你滚动页面时将相应位置的元素移动到可视区域。

###那么为什么会造成动画卡顿呢？
原因就是主线程和合成线程的调度不合理。

下面来详细说一下调度不合理的原因。

>在使用height，width，margin，padding作为transition的值时，会造成浏览器主线程的工作量较重，例如从margin-left：-20px渲染到margin-left:0，主线程需要计算样式margin-left:-19px,margin-left:-18px，一直到margin-left:0，而且每一次主线程计算样式后，合成进程都需要绘制到GPU然后再渲染到屏幕上，前后总共进行20次主线程渲染，20次合成线程渲染，20+20次，总计40次计算。

主线程的渲染流程，可以参考浏览器渲染网页的流程：
>使用 HTML 创建文档对象模型（DOM）
使用 CSS 创建 CSS 对象模型（CSSOM）
**基于 DOM 和 CSSOM 执行脚本（Scripts）
合并 DOM 和 CSSOM 形成渲染树（Render Tree）
使用渲染树布局（Layout）所有元素
渲染（Paint）所有元素**

也就是说，主线程每次都需要执行Scripts，Render Tree ,Layout和Paint这四个阶段的计算。

>而如果使用transform的话，例如tranform:translate(-20px,0)到transform:translate(0,0)，主线程只需要进行一次tranform:translate(-20px,0)到transform:translate(0,0)，然后合成线程去一次将-20px转换到0px，这样的话，总计1+20计算。

可能会有人说，这才提升了19次，有什么好性能提升的？

假设一次10ms。

那么就减少了约190ms的耗时。

会有人说，辣鸡，才190ms，无所谓。

那么如果margin-left是从-200px到0呢，一次10ms，10ms*199≈2s。

还会有人说，辣鸡，也就2s，无所谓。

你忘了单线程这回事了吗？

如果网页有3个动画，3*2s=6s，就是6s的性能提升。
由于数据是猜测的，所以暂时不考虑其真实性，文章后面我使用chrome devtools的performance做了一个实验。

要知道，在"客户至上"的今天，好的用户体验是所有产品的必须遵守的一条规则，无论是对于开发者还是产品经理，追求极致的性能都是我们打造一个好的产品所必备的品质。

可能看了我的略不专业的分析后，大家对主线程，合成线程以及它们在2种性能不同动画方案上的工作流程还不是很了解，可以去看一篇翻译过来的博客（英文原版链接已经失效了）：[深入浏览器理解CSS animations 和 transitions的性能问题](http://sy-tang.github.io/2014/05/14/CSS%20animations%20and%20transitions%20performance-%20looking%20inside%20the%20browser/)

这篇文章完美讲述了浏览器主线程和合成线程的区别，并且举了一个高度从100px变化到200px的2种动画方案的对比，对主线程和合成线程的整个工作流程做了很详尽的讲解，真心建议认真阅读一遍。

回过头来总结下，css3动画卡顿的解决方案：
**在使用css3 transtion做动画效果时，优先选择transform，尽量不要使用height，width，margin和padding。**

transform为我们提供了丰富的api，例如scale，translate，rotate等等，但是在使用时需要考虑兼容性。但其实对于大多数css3来说，mobile端支持性较好，desktop端支持性需要格外注意。

-----------------------------------------------------
补充：为了增强本文的说服力，特地回家做了一个实验，代码如下。
```
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>Page Title</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    .margin-transition{
      /* margin-left: 0; */
      background: rgba(0,0,255,0.3);
      transition: margin-left 1s;
    }
    .transform-transition{
      /* transform: translate(0,0); */
      background: rgba(0,255,0,0.3);
      transition: transform 1s;
    }
    .common{
      height: 300px;
      width: 300px;
    }
  </style>
</head>
<body>
  <div class="margin-transition common" id="marginTransition">
    <p>transition:margin-left 1s</p>
  </div>
  <div class="transform-transition common" id="transformTransition">
      <p>transition:tranform 1s</p>
  </div>
  <button id="control">见证奇迹</button>
  <script>
      var btn = document.getElementById('control');
      var marginTransition = document.getElementById('marginTransition');
      var transformTransition = document.getElementById('transformTransition');
      btn.addEventListener("click",function(){
        console.log(marginTransition.style,transformTransition.style)
        marginTransition.style.marginLeft = "500px";
        transformTransition.style.transform = "translate(500px,0)"
      })
  </script>  
</body>
</html>
```
我将主要借助chrome devtools的performance工具对比二者的性能差异。
先来看margin动画，动态修改DOM节点的margin-left值从0到500px;。
```
transition: margin-left 1s;
```
![margin动画实验](http://upload-images.jianshu.io/upload_images/2976869-700706c8f7fa7a63.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/768)
![margin动画总耗时](http://upload-images.jianshu.io/upload_images/2976869-5a41dae8f2dbbc65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/768)
![margin动画GPU使用率](http://upload-images.jianshu.io/upload_images/2976869-846dea4fe0ed9b36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/768)


再来看下transform动画，动态修改DOM节点的transform值从translate(0,0)到translate(500px,0)。

```
transition: transform 1s;
```
![transform动画实验](http://upload-images.jianshu.io/upload_images/2976869-417a20a6fc673650.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/768)

![transform动画总耗时](http://upload-images.jianshu.io/upload_images/2976869-3deb4c3292e97bb4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![transform动画GPU使用率](http://upload-images.jianshu.io/upload_images/2976869-a03c29bb6c2fc37c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


可能图片不是很好地能说明性能差异，那么我们来列一张耗时对比表，方便我们计算。

| 耗时        | margin           | transform  |
| ------------- |:-------------:| -----:|
|Summery| 3518ms|2286ms|
| Scripting      | 1.8ms | 2.9ms |
| Rendering     | 22.5ms    |  6.9ms |
| Painting | 9.7ms |  1.6ms |
| Other | 39.3ms|  25.2ms |
| Idle( browser is waiting on the CPU or GPU to do some processing)| 3444.4ms|   2249.8ms |
|**GPU使用率**|4.1MB|1.7MB|
通过上表我们可以计算出明margin，transform与transition组合实现CSS3动画效果时的性能差异参数。

| 关键性能参数       | margin           | transform  |
| ------------- |:-------------:| -----:|
|**实际动画耗时（总时间 减去 空闲时间）**|73.6ms|36.2ms

计算得出，transform动画耗时约等于margin动画耗时的0.49倍，性能优化50%。

由于我对Other的所做的具体事情不是很清楚，所以这里的实际动画时间也有可能还要减掉Other中的时间，下表是我们减掉后的数据。

| 关键性能参数       | margin           | transform  |
| ------------- |:-------------:| -----:|
|**实际动画耗时（总时间 减去 其他时间和空闲时间）**|34.3ms|11ms|

计算得出，transform动画耗时约等于margin动画耗时的0.32倍，性能优化接近70%。

也就是说，无论我们减去还是不减去Other的时间，我们采用transform实现动画的方式都比margin动画快。

不精确的得出一个小结论：**transform比margin性能好50%~70%**。

虽然会有50%~70%的性能提升，但是需要注意硬件差异，硬件好的情况下可能不能发现卡顿或者其他的一些性能上的问题。
例如在开发小程序的过程中，模拟器是位于desktop端的，因此它的硬件性能性能更好，例如CPU，GPU。但是一旦在mobile端运行，例如ios或者android上运行时，就可能会出现性能问题，这就是因为移动端的硬件条件逊于PC端导致的。

所以说，性能问题是一直存在的，只不过硬件差异会导致性能影响的程度不同。

所以我们再次回过头来，总结出css3动画卡顿的解决方案：
**在使用css3 transtion做动画效果时，优先选择transform，尽量不要使用height，width，margin和padding。**

That's it !

参考：
http://sy-tang.github.io/2014/05/14/CSS%20animations%20and%20transitions%20performance-%20looking%20inside%20the%20browser/
http://jinlong.github.io/2017/05/08/optimising-the-front-end-for-the-browser/
http://blog.csdn.net/yeana1/article/details/52756871
https://www.jianshu.com/p/b70b72de3c32
https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/
http://blogs.adobe.com/webplatform/2014/03/18/css-animations-and-transitions-performance/