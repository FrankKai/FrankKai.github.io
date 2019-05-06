---
title: IIFE(立即执行函数)那些事儿
date: 
tags: 
---

IIFE(Immediately Invoked Function Expression)是一个定义即运行的JavaScript函数。

它是一种设计模式，是为了做**Self-Executing Anonymous Function** （自执行匿名函数），并且包含2个主要的部分。
第一个是与特定词法作用域相关的在**Grouping Operator**()中的匿名函数。这会不能访问IIFE惯用语中的变量，不会污染全局作用域。
第二个部分是创建立即执行函数的表达式（），js引擎通过它直接解析函数。

例子
函数变成一个立即执行的函数表达式。表达式内的变量不能外部获取不到。
```
(function(){
    var aName = "Barry";
})();
aName;//抛出"Uncaght ReferenceError:aName is not defined"
```

将IIFE返回值分配给一个变量
```
var result = (function(){
    var name = "Barry";
    return name;
})();
result;//"Barry"
```

QA环节：
- IIFE的两个不同的圆括号中哪一个是Grouping Operator，它的作用是什么？
```
(function(){
    var aName = "Barry";
})();
aName;//抛出"Uncaght ReferenceError:aName is not defined"
```
分组操作符（）在表达式中控制评估的优先级。
比如console.log((1 + 2) * 3)；将(1+2)包裹起来，保证输出的不是7，而是9

拿上面的例子来说，将anonymous函数包裹起来的是分组Group Opetator。它可以保证变量只声明在匿名函数内部，不会污染全局的命名空间。

下面代码中function(){}外部的圆括号指的就是Group Operator。
```
(function(){
    var aName = "Barry";
})
```
- IIFE的两个不同的圆括号中不是Grouping Operator的有什么用？
```
(function(){
    var aName = "Barry";
})();
```
非分组操作符的圆括号，作用是告知浏览器正确解析函数。
说得通俗点，其实就是执行这个函数，就像我们声明了function foo(){}，然后foo()这样。

总结一下：
- 声明函数
    - 匿名函数function(){}创建函数
    - Group Operator保证了全局命名空间不受污染
- 执行函数
    - ()保证了引擎正确运行函数

引申环节:
- IIFE中的this指向谁？
this指向Window对象。
```
(function(){
    var aName = "Barry";
    console.log(this);//Window对象
})();
```
- function可以用箭头函数替代吗？
亲测可以，不会污染全局命名空间。
```
(()=>{
    var aName = "Barry";
})();
aName;//抛出"Uncaght ReferenceError:aName is not defined"
```