---
title: 从规范去看Function.prototype.call到底是怎么工作的？
date: 
tags: 
---

![image.png](https://upload-images.jianshu.io/upload_images/2976869-1cd1f00d91b44363.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

今天在学习前端工程化的过程中，遇到一个是实验中的css属性`:fullscreen`，有这样一个例子：[fullscreen伪元素官方demo](https://mdn.mozillademos.org/en-US/docs/Web/CSS/:fullscreen$samples/Example?revision=1342881)
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
其中有一段代码：
```
function enterFullscreen() {
    fullscreenFunc.call(fullscreenDiv);
} 
```
虽然结合上下文能看出来是为了兼容浏览器的fullscreen API，但是其中的Function.prototype.call()我自己其实没有特别深究过。

**为什么不直接fullscreenFunc()，这样不能使得fullscreenDiv全屏吗?**

大家都说call与apply都是为了动态改变this的，仅仅是传入参数的方式不同，call传入(this,foo,bar,baz)，而apply传入(this,[foo,bar,baz])那么事实真如大家所说的那样吗？既然apply能动态改变this，那么为什么还要多此一举开放一个call？
这其中肯定隐藏着一些秘密，那就是有些事情是apply做不到，而call可以胜任的。
继续我们的啃规范之旅，去深入到Function.prototype.call()的内部，彻底把它搞清楚。

### [19.2.3.4](https://www.ecma-international.org/ecma-262/6.0/#sec-function.prototype.constructor "link to this section") Function.prototype.call (thisArg , ...args)
>When the `call` method is called on an object func with argument, thisArg and zero or more args, the following steps are taken:
1.  If [IsCallable](https://www.ecma-international.org/ecma-262/6.0/#sec-iscallable)(*func*) is **false**, throw a **TypeError** exception.
2.  Let *argList* be an empty [List](https://www.ecma-international.org/ecma-262/6.0/#sec-list-and-record-specification-type).
3.  If this method was called with more than one argument then in left to right order, starting with the second argument, append each argument as the last element of *argList*.
4.  Perform [PrepareForTailCall](https://www.ecma-international.org/ecma-262/6.0/#sec-preparefortailcall)().
5.  Return [Call](https://www.ecma-international.org/ecma-262/6.0/#sec-call)(*func*, *thisArg*, *argList*).

The `length` property of the `call` method is **1**.

当call方法在带参数的对象的方法上调用时，thisArg和零个或者对个参数，会进行如下的步骤：
1. 如果IsCallable(func)返回false，抛出TypeError异常。
2. 定义argList为一个空的列表。
3. 如果方法按照从左到右传入的参数个数不止一个，从第二个参数开始，依次将每个参数从尾部添加到argList数组。
4. 执行PrepareForTailCall()
5. 返回Call(func,thisArg,argList)

有3个点看不懂：
- IsCallable(func)
- PrepareForTailCall()
- Call(func,thisArg,argList)

这些同样在规范中有对应描述：
### [7.2.3](https://www.ecma-international.org/ecma-262/6.0/#sec-iscallable "link to this section")IsCallable ( argument )

The abstract operation IsCallable determines if argument, which must be an [ECMAScript language value](https://www.ecma-international.org/ecma-262/6.0/#sec-ecmascript-language-types)or a [Completion Record](https://www.ecma-international.org/ecma-262/6.0/#sec-completion-record-specification-type), is a callable function with a [[Call]] internal method.

重点在于is a callable function with a [[Call]] internal method.，也就是说执行isCallable(func)运算的func，如果函数内部有一个内在的[[Call]]方法，那么运算结果为true，也就是说这个函数是可调用的的。(callable)
### [14.6.3](https://www.ecma-international.org/ecma-262/6.0/#sec-preparefortailcall "link to this section")Runtime Semantics: PrepareForTailCall ( )

The abstract operation PrepareForTailCall performs the following steps:

1.  Let *leafContext* be [the running execution context](https://www.ecma-international.org/ecma-262/6.0/#sec-execution-contexts).
2.  [Suspend](https://www.ecma-international.org/ecma-262/6.0/#sec-execution-contexts) *leafContext*.
3.  Pop *leafContext* from [the execution context stack](https://www.ecma-international.org/ecma-262/6.0/#sec-execution-contexts). The [execution context](https://www.ecma-international.org/ecma-262/6.0/#sec-execution-contexts) now on the top of the stack becomes [the running execution context](https://www.ecma-international.org/ecma-262/6.0/#sec-execution-contexts).
4.  [Assert](https://www.ecma-international.org/ecma-262/6.0/#sec-algorithm-conventions): *leafContext* has no further use. It will never be activated as [the running execution context](https://www.ecma-international.org/ecma-262/6.0/#sec-execution-contexts).

A tail position call must either release any transient internal resources associated with the currently executing function [execution context](https://www.ecma-international.org/ecma-262/6.0/#sec-execution-contexts) before invoking the target function or reuse those resources in support of the target function.

1.  [ReturnIfAbrupt](https://www.ecma-international.org/ecma-262/6.0/#sec-returnifabrupt)(*argument*).
2.  If [Type](https://www.ecma-international.org/ecma-262/6.0/#sec-ecmascript-data-types-and-values)(*argument*) is not Object, return **false**.
3.  If *argument* has a [[Call]] internal method, return **true**.
4.  Return **false**.

虽然看不懂，但还是得硬着头皮学习一波。
抽象操作PrepareForTailCall执行以下几个步骤：
1. 让叶子上下文成为运行中的执行上下文
2. 暂停叶子上下文
3. 顶叶子上下文来自执行上下文的堆。当前的在堆顶部的执行上下文成为运行中的执行上下文
4. 断言：叶子上下文没有其他作用。它再也不会作为运行中执行上下文被激活。

在调用目标函数或者重用这些资源去支持目标函数之前，尾部位置调用必须释放与当前执行函数上下文相关的瞬态内部资源。

1. ReturnIfAbrupt(argument).
2. 如果Type(argument)不是对象，返回false。
3. 如果argument含有[[call]]内部方法，返回true。
4. 返回 false

看懂一个大概，是为了在函数调用栈的尾部调用当前函数做准备，其中的运行中执行上下文，正是我们所说的this动态改变的原因，因为本质上this改变并不仅仅是指向的对象发生变化，而是连带着与其相关的上下文都发生了变化。

所以说，这一步是this动态改变的真正原因。

### [7.3.12](https://www.ecma-international.org/ecma-262/6.0/#sec-call "link to this section")Call(F, V, [argumentsList])

The abstract operation Call is used to call the [[Call]] internal method of a function object. The operation is called with arguments F, V , and optionally argumentsList where F is the function object, V is an [ECMAScript language value](https://www.ecma-international.org/ecma-262/6.0/#sec-ecmascript-language-types) that is the **this** value of the [[Call]], and argumentsList is the value passed to the corresponding argument of the internal method. If argumentsList is not present, an empty [List](https://www.ecma-international.org/ecma-262/6.0/#sec-list-and-record-specification-type) is used as its value. This abstract operation performs the following steps:

1.  [ReturnIfAbrupt](https://www.ecma-international.org/ecma-262/6.0/#sec-returnifabrupt)(*F*).
2.  If *argumentsList* was not passed, let *argumentsList* be a new empty [List](https://www.ecma-international.org/ecma-262/6.0/#sec-list-and-record-specification-type).
3.  If [IsCallable](https://www.ecma-international.org/ecma-262/6.0/#sec-iscallable)(*F*) is **false**, throw a **TypeError** exception.
4.  Return *F*.[[Call]](*V*, *argumentsList*).

Call抽象操作是在调用函数对象的内部的[[Call]]方法。这个操作参数类型包括F,V以及可选的argumentList。F指的是调用函数，V指的是[[Call]]的this值，然后argumentsList是传入到[[Call]]内部方法相应参数的值。如果argumentList不存在，那么argumentList将被置为一个空数组。这个方法按照下列几步执行：
1. ReturnIfAbrupt(F)
2. 如果没传入argumentList,那么argumentList将会被置为一个空数组。
3. 如果IsCallable(F)是false，返回TypeError异常。
4. 返回 `F.[[call]](V,argumentsList)`.

所以Function.prototype.call(this,...args)执行过程现在很明了：
1. 判断传入的func是否有[[call]]属性，有[[call]]才意味着函数能被调用，否则抛出TypeError异常。
2. 定义argList为一个空的列表。
3. **传参：如果方法按照从左到右传入的参数个数不止一个，从第二个参数开始，依次将每个参数从尾部添加到argList数组。**
4. **切换this上下文：执行PrepareForTailCall()，为函数调用栈在尾部调用函数做准备，切换运行中执行上下文，实现this上下文的动态改变。**
5. 万事具备，执行Call(func,thisArg,argList)，调用函数即可。

回到我们的例子：
```
fullscreenFunc.call(fullscreenDiv);
```
1. func为fullscreenDiv DOM 节点的方法：'requestFullscreen' || 'mozRequestFullScreen' || 'msRequestFullscreen' 
|| 'webkitRequestFullScreen'，由于是fullscreen API，所以isCallable(func)返回true。
2. 定义一个argList空数组用来传参。
3. **传参：由于fullscreenFunc.call(fullscreenDiv);只有一个参数，所以直接传入argList空数组。**
4. **切换this上下文：停止当前的this叶子上下文，也就是window，切换到fullscreenDiv的执行上下文。**
5. 由于当前浏览器为chrome，因此执行 `fullscreenDiv.webkitRequestFullscreen.[[call]](this,[])`。

因此我们之前提的那个**为什么不直接fullscreenFunc()，这样不能使得fullscreenDiv全屏吗?**，答案就很清楚了？不能。
为什么呢？
```
var fullscreenFunc   = fullscreenDiv.requestFullscreen;
if (!fullscreenFunc) {
     ['mozRequestFullScreen', 'msRequestFullscreen','webkitRequestFullScreen'].forEach(function (req) {
        fullscreenFunc = fullscreenFunc || fullscreenDiv[req];
     });
}
```
下面的代码，仅仅是获得了fullscreenDiv对象的fullscreen request API的引用，而fullscreenFunc的作用域是全局的window对象，也就是this的当前指向为window。
![image.png](https://upload-images.jianshu.io/upload_images/2976869-d5121abf1a88ac85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**而我们是想触发window的子对象fullscreenDiv的全屏方法，所以需要将this上下文切换为fullscreenDiv，这就是不直接调用fullscreenFunc()，需要fullscreenFunc.call(fullscreenDiv)的原因**。

最近在看龙书，第一章讲到动态语言与静态语言的区别，龙书中讲到"运行时决定作用域的语言是动态语言，在编译时指定作用域的预言是静态语言"。例子中的以function关键字定义的类，this运行中执行上下文的切换，恰恰证明了javascript是一门动态语言；再举个形象的静态语言的例子，java会使用class关键字构建类，在类内部使用private，public等关键字去指定作用域，编译时就会去约束其作用域，具有非常强的约束性，this始终指向当前类。

刚才和一个java后端同事确认，java也有this关键字，但是仅能使用当前类中的方法，B类不能调用A类中的方法。

**js中的this，是可以通过call或者apply进行动态切换从而去调用其他类中的方法的，B类不能调用A类中的方法。（注意：我们这里的类指的是以function关键字进行定义的类，暂时不考虑es6的class关键字构造类的方式）**

说了这么多，我们再来强调下重点：

加粗的部分是重点！
加粗的部分是重点！
加粗的部分是重点！

抛开V8引擎内部执行call和apply的原理不说，二者最终实现的都是this上下文的动态切换，所以就像大家所说的那样，都是动态改变this。我们只要心里知道，其实二者在背后实现动态切换this的操作部分有很大的不同就可以了，当出现由于内部实现细节引起的问题时，我们可以快速定位。

That's it !

>期待和大家交流，共同进步，欢迎大家加入我创建的与前端开发密切相关的技术讨论小组：
> - SegmentFault技术圈:[ES新规范语法糖](https://segmentfault.com/g/1570000010695363)
> - SegmentFault专栏：[趁你还年轻，做个优秀的前端工程师](https://segmentfault.com/blog/chennihainianqing)
>- 知乎专栏：[趁你还年轻，做个优秀的前端工程师](https://zhuanlan.zhihu.com/wyasy)
>- Github博客: [趁你还年轻233的个人博客
](https://github.com/FrankKai/FrankKai.github.io)
>- 前端开发交流群：660634678

>努力成为优秀前端工程师！