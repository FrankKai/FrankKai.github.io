---
title: Javascript对象的深浅拷贝
date: 
tags: 
---

![](https://user-images.githubusercontent.com/19262750/38594811-bf9e7ad0-3d7a-11e8-886c-9d803064536f.png)



开门见山，有人叫对象的复制为深复制浅复制，也有人叫深拷贝浅拷贝。
其实都是copy。

**深拷贝(递归复制，复制所有层级，独立副本，一个完全和原来对象属性无关的副本)**
返回对象：一个。
传入对象：一个。
条件：JSON安全的对象，可以序列化为JSON字符串，并且可以解析为新的字符串。
深拷贝算法：
```
function deepCopy(data){
    let memory = null;
    const type = Object.prototype.toString.call(data);
    if (type === "[object Array]"){
        memory = []
        for (let i=0 ;i<data.length;i++){
            memory.push(deepCopy(data[i]))
        }
    }else if (type === "[object Object]"){
        memory = {}
        Object.keys[data].forEach((key)=>{
            memory[key] = data[key]
        })
    }else{
        return data;
    }
    return memory;
}
```

jQuery深拷贝:
`var copiedObject = jQuery.extend(true, {}, originalObject)`

es6深拷贝：
`var copiedObject= JSON.parse(JSON.stringify(originalObject));`

深拷贝是递归复制，新复制的对象与原对象是完全独立的两个对象，它们指向不同的内存地址，做set不会影响到对方。

![](https://upload-images.jianshu.io/upload_images/2976869-a365a396abb3cfaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/512)

**浅拷贝（单次复制，复制最高层级，引用副本，一个基于对原对象属性引用的副本）**
返回对象：一个。
传入对象：一个或多个。
条件：无。

jQuery浅拷贝：
`var copiedObject = jQuery.extend({}, originalObject)`

es6浅拷贝：
`var copiedObject = Object.assign({},originalObject)`

es7浅拷贝：
`var copiedObject = {...originalObject}`

浅拷贝算法：
```
  function shallowCopyObj(original){
        let copy = {}
        Object.keys(original).forEach(key=>{
            copy[key] = original[key]
        })
        return copy
	}
```
由于javascript的对象是存地址的，所以浅复制的对象与原对象，都指向同一个内存地址，属于引用复制，做set会影响到对方。
![](https://upload-images.jianshu.io/upload_images/2976869-973aa14f7f608a13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/512)



实验：
**①普通属性修改：深拷贝和浅拷贝都可以满足对象的复制。**
普通属性是指value值为非Array，Object类型的数据类型，也就是Number，String，Boolean等基本数据类型。
原因：基本数据类型属于值传递。
```
var obj = {foo:1};
var deepCopyObj = JSON.parse(JSON.stringify(obj));
deepCopyObj.foo = 2;
console.log("obj.foo:",obj.foo);//1
console.log("deepCopyObj.foo:",deepCopyObj.foo);//2
```
```
var obj = {foo:1};
var shallowCopyObj = Object.assign({},obj);
shallowCopyObj.foo = 2;
console.log("obj.foo:",obj.foo);//1
console.log("shallowCopyObj.foo:",shallowCopyObj.foo);//2
```
**②高级属性修改：深拷贝满足对象的复制，浅拷贝影响原数组。**
高级属性是指Array，Object数据类型。
原因：基本数据类型属于引用传递。
```
var obj = {foo:1,bar:{baz:1}};
var deepCopyObj = JSON.parse(JSON.stringify(obj));
deepCopyObj.bar.baz = 2;
console.log("obj.bar.baz:",obj.bar.baz);//1
console.log("deepCopyObj.bar.baz:",deepCopyObj.bar.baz);//2
```

```
var obj = {foo:1,bar:{baz:1}};
var shallowCopyObj = Object.assign({},obj);
shallowCopyObj.bar.baz = 2;
console.log("obj.bar.baz:",obj.bar.baz);//2 Attention！
console.log("shallowCopyObj.bar.baz:",shallowCopyObj.bar.baz);//2
```

印象中const也是保持变量地址不变的操作，那么es6中的let和const对于对象的深浅拷贝有影响吗？

也就是将上面代码中的var替换为let和const。

实验结果是let和const不会影响深浅拷贝的结果，因为let强调块作用域，而const强调变量整体地址空间的不变性。

关于对象的深浅拷贝，暂且探索到这里，后续有新发现再进行补充。

谢谢您的阅读～