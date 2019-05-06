---
title: 饿了么大前端题目解析：JS基础问题
date: 
tags: 
---

问题链接：[JavaScript 基础问题](https://github.com/ElemeFE/node-interview/blob/master/sections/zh-cn/common.md)
- js 中什么类型是引用传递, 什么类型是值传递? 如何将值类型的变量以引用的方式传递? 
- js 中， 0.1 + 0.2 === 0.3 是否为 true ? 在不知道浮点数位数时应该怎样判断两个浮点数之和与第三数是否相等？
- const 定义的 Array 中间元素能否被修改? 如果可以, 那 const 修饰对象的意义是? 
- JavaScript 中不同类型以及不同环境下变量的内存都是何时释放? 

**问：js中什么是引用传递，什么是值传递？如何将值类型的变量以引用的方式传递？**

答：简单点说，对象是引用传递，基础类型是值传递，通过将基础类型boxing(包装)可以以引用方式传递。

例1：对象数据属性

```
var str = 'a,b,c'
var obj = {}
obj.key = str
var ref = obj
ref.key = "a"
console.log(obj)
```
注意：引用传递传递的是整个对象，而不是单一的对象属性。

例2：函数闭包引用

```
function Persion(name,age){
    var n = name
    var a = age
    return {
        getName:function(){
            return n;

        },

        addAge:function(){

            return a++;

        },

        getAge:function(){

            return a

        }
    }
}
var Foo = new Persion("foo",23)
Foo.addAge()
Foo.getAge()
```

**问：js中，0.1+0.2 ===0.3是否为true？在不知道浮点数位数时应该怎样判断两个浮点数之和与第三数是否相等？**

0.1+0.2 ===0.30000000000000004(17位小数)，返回为false。

为什么0.1+0.2 ===0.30000000000000004？

由此派生出两篇原创博客。

[深度剖析0.1 +0.2===0.30000000000000004的原因](https://github.com/FrankKai/FrankKai.github.io/issues/7)

[如何解决0.1 +0.2===0.30000000000000004类问题](https://github.com/FrankKai/FrankKai.github.io/issues/11)

**问：const 定义的 Array 中间元素能否被修改? 如果可以, 那 const 修饰对象的意义是?**

可以被修改。

const arr = [1,2,3] arr[0] = "hello" → ['hello']

const的意义：保证变量指向的内存地址不发生变化。

对于值是基本类型的变量，具有完全不可变性，地址和数据均不可变；对于值是引用类型的变量，数据具有部分不可变性，内存地址不可变，数据可变。

也就是说，使用const的定义的变量，牢牢地与value绑定在一起，而且这种绑定是不可解除的，但是这种绑定，指的是整体的绑定，也就是内存地址上的绑定。举例来说：const str = 'foo' ,‘str永远指向'foo'的内存地址；对于const obj ={name:'foo'}，obj永远指向{}的内存地址，指向内容不包含对象数据。

做一个很形象的类比小故事：你的0岁到23岁

```
//刚出生的你，父母给予了生命的同时，给予了你一个名字 Jack，落地1岁
const you = {}
you.name="Jack",
you.age=1
//7岁，你上了小学
you.age = 7
//13岁，小学毕业，开始读初中
you.age  = 13 , you.primary = true
//16岁，初中毕业。开始读高中
you.age = 16 , you.junior = true
//19岁，高中毕业，开始读大学
you.age = 19 , you.high = true
//23岁，大学毕业，开始工作
you.age = 23 , you.university = true , you.salary = "RMB 100 yuan"
```
......故事还在继续
但是，你始终是你，你的父母始终是你的父母，即使到了66世纪，这是亘古不变的。但是在这一路上，你的age属性在发生变化，你解锁了小学，初中，高中，大学毕业证等一系列证书，工作后也新增了薪酬这个新属性，虽然只有可怜的100块。
或许有一天，you.girlfriend = 'Rose'
又或许有一天，you.baby = little Jack"
回过头来分析下刚出生的你const you = {name:"Jack",age:1}
const不能重新定义：试问一下，在这个世界，你还能重新出生一次吗？
const必须赋初始值: const you = {}是一个整体，谁都不能少，因为你本身是一个完整的整体，在{}.prototype你遗传了父母的基因。
const不能重新赋值：you = {name:'Tom',age:18}，据我所致，目前还没有什么技术可以实现克隆人的复制粘贴。
const复杂类型可变：name和age这些属性，则是后期为你新增的新属性，这些数据是有限且可变的，例如age，salary等等。
但是在另外一个世界，也就是另外一个作用域（块作用域或函数局部作用域），you can reborn
```
const a = 1
if(a === 1){
 const a = 2
}
//all is well
const b = 1
function j(){
 const b = 1
}
//all is well
```
因此，当有那么一个object或者array时，就像故事中的you一样，无论风吹雨打，本质（内存地址不变）亘古不变时，极大程度上确保变量不被完全重新赋值时，就可以用const来定义。

**问：JavaScript中不同类型以及不同环境下变量的内存都是何时释放？**

Javascritp中类型：值类型，引用类型

环境：Browser Nodejs

引用类型是在没有引用之后，通过V8的GC自动回收。值类型如果处于闭包的情况下，要等闭包没有引用才会被GC回收，非闭包的情况下，等待V8的新生代(new space)切换的时候回收。

你需要了解哪些操作一定会导致内存泄露，或者可以崩掉内存。比如以下代码是否能爆掉V8的内存？

```
let arr = [];
while(true) {
    arr.push\(1\);
}
```
都会爆掉，因为超出了数组的最大长度，会自动结束异常。

浏览器V8引擎崩溃：
![image](https://user-images.githubusercontent.com/19262750/39180472-892228b6-47e9-11e8-8662-51fa558439ec.png)

NodeV8引擎崩溃：
![image](https://user-images.githubusercontent.com/19262750/39180482-90210628-47e9-11e8-830d-00dc3f5308ff.png)



然后上述代码与下方的情况有什么区别?

```
let arr = [];
while(true){
    arr.push\(\);
}
```
死循环，不会爆掉V8内存，但是导致进程阻塞。

有什么办法终止死循环？在任务管理器中强制结束进程，会出现上面的浏览器崩溃页面，或者Node环境下一直处于计算状态，ctrl+c强制终止。

如果 push 的是 Buffer 情况又会有什么区别?

```
let arr = [];
while(true)
arr.push(new Buffer(1000));
```
区别很大，内存泄露的速度比直接push Number类型的慢很多，因为ES定义的Number类型，实质上是遵循IEEE-2008的64位存储，也就是说，Number类型的1与buffer类型的1，完全不一样，前者在编译器中是00000000000000000000000000（63个0）1，占了64位；但是后者就只是1，只占了1位。

当push buffer崩溃时...不存在的，push buffer不会崩溃，当内存顶到爆，也就是即将到达100%时，会自动垃圾回收，直观地讲，就是会瞬间降低内存，push工作继续，很是神奇。