---
title: 为什么typeof null→object ?
date: 
tags: 
---

typeof null→"object" ?

《你不知道的Javascript》译者：
>原理是这样的，不同的对象在底层都表示为二进制，在Javascript中二进制前三位都为0的话会被判断为Object类型，null的二进制表示全为0，自然前三位也是0，所以执行typeof时会返回"object"。


不知道大家有没有理解，我举个非常不恰当的例子。

**假设**所有的Javascript对象都是16位的，也就是有16个0或1组成的序列。

基于上面的假设，我们猜想：

```
Array: 1000100010001000
null:  0000000000000000
```
使用typeof检测[]和null的结果为：
```
typeof [] →"object"
typeof null→"object"
```
因为Array和null的前三位都是000。

注意：
为什么Array的前三位不是100？
这是因为二进制中的"前"一般来说代表的是低位，比如二进制00000011对应十进制数是3，它的前三位是011。

最后啰嗦一句，《通信原理》是门好课，编码方面的东西讲得一应俱全，通信计算机一家亲~

努力成为优秀的前端工程师！

福利：字面量和对象类型常用形式表

| 字面量       | 对象类型           |常用形式  |
| :-------------: |:-------------:|:-----:|
|string|String| 'hello'|
|number|Number|   1024 |
|boolean|Boolean|   true  |
|null|/|null|
|undefined|/|undefined|
|object|Object|{}|
|/|Array|[]|
|/|Date|new Date()|
|/|RegExp|/[a-zA-Z]/|
|/|Error|/|