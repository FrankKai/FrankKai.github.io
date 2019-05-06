---
title: 从规范去看Function.prototype.apply到底是怎么工作的？
date: 
tags: 
---

![圣诞节快乐.png](http://upload-images.jianshu.io/upload_images/2976869-3af17522c415306c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

今天看element-react源码的时候，又看到了这张似曾相识却又异常陌生的老面孔，那就是Function.prototype.apply()...
```
import React from 'react';
import PropTypes from 'prop-types';
import classnames from 'classnames';

export default class Component extends React.Component {
  classNames(...args) {
         return classnames(args);
  }

  className(...args) {
         return this.classNames.apply(this, args.concat([this.props.className]));
  }

  style(args) {
         return Object.assign({}, args, this.props.style)
  }
}

Component.propTypes = {
  className: PropTypes.string,
  style: PropTypes.object
};
```
虽然高设，犀牛书以及你不知道的Javascrit都看过apply的原理，但依然没什么卵用，可能是缺少实践的原因，看完就忘。

Q：
其中这句`this.classNames.apply(this, args.concat([this.props.className]))`，又把我搞得晕头转向，this到底是谁？没猜错的话应该是组件实例吧？
那为啥不直接 `this.classNames(args.concat([this.props.className]))`,直接传一个数组不就行了吗？
为什么一定要用`this.classNames.apply(this, args.concat([this.props.className]))`？
同理，印象中apply用来取数组最大最小值很方便，`Math.max.apply(null,[1,3,8,2])`,但是`Math.apply( [1,3,8,2] )`却返回传参类型错误？这又是为什么？

带着这些问题，我又一次开始了啃规范之旅。
>Function.prototype.apply (thisArg, argArray)
The apply method takes two arguments, thisArg and argArray, and performs a function call using the [[Call]] property of the object. If the object does not have a [[Call]] property, a TypeError exception is thrown. If thisArg is null or undefined, the called function is passed the global object as the this value. Otherwise, the called function is passed ToObject(thisArg) as the this value. If argArray is null or undefined, the called function is passed no arguments. Otherwise, if argArray is neither an array nor an arguments object (see 10.1.8), a TypeError exception is thrown. If argArray is either an array or an arguments object, the function is passed the (ToUint32(argArray.length)) arguments argArray[0], argArray[1], …, argArray[ToUint32(argArray.length)–1].

apply有2个方法，一个是thisArg，一个是argArray，然后再执行对象的[[call]]属性。如果对象没有call属性，一个TypeError会被扔出来。
如果thisArg是null或者undefined，会将global作为this值。否则，被调用的函数，进行ToObject(thisArg)转换后，作为this值。
如果argArray是null或者undefined，被调用函数不传入任何参数。否则，argArray如果既不是数组也不是arguments object（类数组对象），会报错TypeError.
如果 argArray是数组或者类数组对象，被调用函数传入ToUnit32(argArray.length) arguments argArray[0]，argArray[1]一直到argArray[ToUint32(argArray.length)–1].

**ToObject内部怎么操作？这个不用管。**
**ToUnit32又怎么操作？这个很神奇。**

> ToUint32: (Unsigned 32 Bit Integer)
>The operator ToUint32 converts its argument to one of 232 integer values in the range 0 through 232−1,
inclusive. This operator functions as follows:
>1. Call ToNumber on the input argument.
>2. If Result(1) is NaN, +0, −0, +∞, or −∞, return +0.
>3. Compute sign(Result(1)) * floor(abs(Result(1))).
>4. Compute Result(3) modulo 232; that is, a finite integer value k of Number type with positive sign and
less than 232 in magnitude such the mathematical difference of Result(3) and k is mathematically an
integer multiple of 232.
>5. Return Result(4).

ToUnit32能转换它的参数为0到231总共232个整数中的一个，这个函数遵循以下规则。
1.对输入参数调用ToNumber()函数
2.如果Result(1)是 NaN, +0, −0, +∞, 或者 −∞, return +0
3.计算sign(Result(1)*floor(abs(Result(1))))
4.计算Result(3) modulo 232；就是说 一个无穷的正整数值k，大于0小于232，Result(3)与k的数学差异是232的整数倍。
5.返回Result(4)

A1:
多说无益，举个例子实在：
**Math.max.apply(null,[1,3,2,""])**

第一步：global值替换null，浏览器环境为window
Math.max.apply(window,[1,3,2,""])

第二步：[1,3,2,""].length传入到ToUnit32()中
ToUnit32(4)

第三步：计算ToUnit32(4)， 作为数组转换成arguments类数组对象的编号
1.ToNumber(4)→Result(1)=4
2.Not pass the condition
3.Math.sign(4*Math.floor(Math.abs(4))) →Result(3)=1
4.1 Mod 232 → Result(4) =1
5.return 1

第四步：传入arguments到Math.max中
第三步会生成(1)arguments 类数组对象（是否为纯函数方式生成暂时未知），将这个arguments传入到Math.max中，与直接这样写Math.max(1,3,2,"")的效果一样，此时就会返回最大值3。

这里有一个很重要的坑，为什么arguments传入，也可以像正常Math.max(1,3,2,"")一样？
其实这个坑是因为Math.max(1,3,2,"")这个例子中的实参，在函数内部的实参就是arguments，且这个arguments的长度为4，这和绝大多数的函数一样，除箭头函数外的所有函数，都有这个arguemnts实参array-like object。

看完这个， Math.max.apply(null,[1,3,8,2])的内部原理应该也就清楚了吧。

A2:
回到源代码中`this.classNames.apply(this, args.concat([this.props.className]))`。
**1.为什么直接传入["foo","bar"]数组不行？**
因为当我们传入["foo","bar"]到classNames时，此时return classnames(args);中的args会变成一个二维数组[["foo","bar"]],与classnames模块的预期值类型一维数组相悖。
**2.为什么调用Function.prototype.apply后就可以？**
因为如规范中的第三步中所示， this.classNames.apply(this, args.concat([this.props.className]))中，在apply的内部算法中，会将args.concat([this.props.className])这个数组，转换为一个实参类数组对象arguments，跳过形参赋值步骤，直接深入到 this.classNames()内部，这样一来，就实现了一个一个传值的效果，其中的...args就会成为一个一个独立的参数，而args会作为实参数组传给classnames函数。

说了这么多，总结起来其实就一句话：
###*callObject.method.apply(thisArg,thisArray)，可以将thisArray转化为arguments，传入到callObject.method内部。*

加粗加斜的这句真的很重要！
加粗加斜的这句真的很重要！
加粗加斜的这句真的很重要！

参考：
1.The mathematical function sign(x) yields 1 if x is positive and −1 if x is negative. The sign function is not
used in this standard for cases when x is zero
2.求模
对于整数a，b来说，取模运算或者求余运算的方法要分如下两步：
1.求整数商：c=a/b
2.计算模或者余数：r=a-(c*b)
求模运算和求余运算在第一步不同
取余运算在计算商值向正无穷方向舍弃小数位
取模运算在计算商值向负无穷方向舍弃小数位
例如：4/(-3)约等于-1.3
在取余运算时候商值向0方向舍弃小数位为-1
在取模运算时商值向负无穷方向舍弃小数位为-2
所以
4rem(-3)=1
4mod(-3)=-2
3.C和JS中%表示取余，python中表示取模
python中%表示取模，have a try 1 Mod 232 →1

Merry Christmas ~
That's it !
