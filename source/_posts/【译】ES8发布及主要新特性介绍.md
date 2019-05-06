---
title: 【译】ES8发布及主要新特性介绍
date: 
tags: 
---

原文链接：[ES8 was Released and here are its Main New Features ](https://hackernoon.com/es8-was-released-and-here-are-its-main-new-features-ee9c394adf66)
第8版EcmaScript规范新特性介绍

![](http://upload-images.jianshu.io/upload_images/2976869-86e9b0ba6ae452b5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2017年6月末，TC39官方发布了EcmaScript8，也叫EcmaScript2017。去年似乎就有很多关于EcmaScript的讨论。这没什么好奇怪的，因为现在每年都会有新的ES标准发布出来。2015年发布了ES6，2016年发布了ES7，但是还人记得ES5是什么时候发布的吗？答案是：2009年。而这一年，JavaScript还没有开始兴风作浪。
所以我们必须紧跟Javascript的发展脚步。在我们的日常开发工作中，学习新特性，运用新特性。
如果你足够厉害，深呼吸，然后去看新标准的web版或者PDF版本。如果还没那么厉害的话，这篇文章就很适合你，我们将通过代码示例，详细得阐述和介绍主要的ES8新特性。
这篇文章已经被Jason Cheng翻译成了中文版本。（我不管，我还要翻译一遍）

###String padding
这一部分将2个新的函数添加到了String类：padStart和padEnd。
就像字面意思一样，这些函数的目的是为了填补字符串的前面或者后面，目的是让结果字符串可以达到规定的长度。你可以使用特殊的字符或者字符串填补，也可以用默认的空格去填补。这里给出了函数的语法声明：
```
str.padStart(targetLength [, padString])
str.padEnd(targetLength [, padString])
```
如你所见，第一个参数是`targetLength`，是结果字母串的目标长度。第二个参数是可选的`padString`，决定了用什么字符或字符串源去填充字符串。默认使用空格填充。
```
'es8'.padStart(2);          // 'es8'
'es8'.padStart(5);          // '  es8'
'es8'.padStart(6, 'woof');  // 'wooes8'
'es8'.padStart(14, 'wow');  // 'wowwowwowwoes8'
'es8'.padStart(7, '0');     // '0000es8'
'es8'.padEnd(2);          // 'es8'
'es8'.padEnd(5);          // 'es8  '
'es8'.padEnd(6, 'woof');  // 'es8woo'
'es8'.padEnd(14, 'wow');  // 'es8wowwowwowwo'
'es8'.padEnd(7, '6');     // 'es86666'
```

![浏览器对String Padding的兼容性](http://upload-images.jianshu.io/upload_images/2976869-f29386b0e9aa0fe4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###Object.values 和 Object.entries
`Object.values`方法可以返回由给定对象自身的可枚举的属性值组成的数组，枚举顺序和`for in`循环的一样。函数声明很简单：
```
Object.values(obj)
```
`obj`参数是即将处理的对象。可以是对象，也可以是数组（数组其实可以转换成带索引值的对象，例如[10, 20, 30] -> { 0: 10, 1: 20, 2: 30 }）。
```
const obj = { x: 'xxx', y: 1 };
Object.values(obj); // ['xxx', 1]

const obj = ['e', 's', '8']; // same as { 0: 'e', 1: 's', 2: '8' };
Object.values(obj); // ['e', 's', '8']

// when we use numeric keys, the values returned in a numerical 
// order according to the keys
const obj = { 10: 'xxx', 1: 'yyy', 3: 'zzz' };
Object.values(obj); // ['yyy', 'zzz', 'xxx']
Object.values('es8'); // ['e', 's', '8']
```

![浏览器对Object.values的兼容性](http://upload-images.jianshu.io/upload_images/2976869-447c006a3f1b2d72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`Object.entries`方法同样也返回对象的可枚举属性，但时会包裹在`[key,value]`这样的名值对里，元素顺序和Object.values的顺序是一样的。函数声明很简单：
```
const obj = { x: 'xxx’, y: 1 };
Object.entries(obj); // [[’x’, 'xxx’], [’y’, 1]]

const obj = [’e’, 's’, '8’];
Object.entries(obj); // [[’0’, 'e’], [’1’, 's’], [’2’, '8’]]

const obj = { 10: 'xxx’, 1: 'yyy’, 3: 'zzz' };
Object.entries(obj); // [[’1’, 'yyy’], [’3’, 'zzz’], [’10’, 'xxx’]]
Object.entries('es8'); // [['0', 'e'], ['1', 's'], ['2', '8']]
```

![浏览器对Object.entries的兼容性](http://upload-images.jianshu.io/upload_images/2976869-91afe95cec5bdfa0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###Object.getOwnPropertyDescriptors

`getOwnPropertyDescriptors`方法返回自身的属性描述符集合。一个自身的属性描述符，由对象直接定义，而且继承自对象的原型。函数声明如下：
```
Object.getOwnPropertyDescriptors(obj)
```
`obj`是源对象。描述对象返回结果中，主要包括键值：configurable, enumerable, writable, get, set and value。
```
const obj = { 
  get es7() { return 777; },
  get es8() { return 888; }
};
Object.getOwnPropertyDescriptors(obj);
// {
//   es7: {
//     configurable: true,
//     enumerable: true,
//     get: function es7(){}, //the getter function
//     set: undefined
//   },
//   es8: {
//     configurable: true,
//     enumerable: true,
//     get: function es8(){}, //the getter function
//     set: undefined
//   }
// }
```
描述对象返回数据对于高级功能修饰符来说很重要。

![浏览器对Object.getOwnPropertyDescriptors(obj)的兼容性](http://upload-images.jianshu.io/upload_images/2976869-34c8bb772407af7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###Trailing commas函数参数列表后的逗号
当我们在参数列表后面添加一个逗号的时候，编译器会报语法错误（SyntaxError），但是Trailing commas的出现，可以使得不报错。
```
function es8(var1, var2, var3,) {
  // ...
}
```
像函数声明那样，加了多余逗号的函数同样可以进行函数调用：
```
es8(10, 20, 30,);
```
这个特性是受到了对象字面量和数组字面量的启发，例如`[10,20,30,]`和`{x:1,}`。
###Async functions
`async function`声明定义了异步函数，它可以返回一个异步的对象。从原理上来说，异步函数的工作原理更像构造函数，但是它们不能被翻译成构造函数。
```
function fetchTextByPromise() {
  return new Promise(resolve => { 
    setTimeout(() => { 
      resolve("es8");
    }, 2000);
  });
}
async function sayHello() { 
  const externalFetchedText = await fetchTextByPromise();
  console.log(`Hello, ${externalFetchedText}`); // Hello, es8
}
sayHello();
```
调用`sayHello`函数后，将在2秒后打印出`Hello,es8`。
```
console.log(1);
sayHello();
console.log(2);
```
1,2会立刻打印，但是Hello,es8会在2秒后打印。
```
1 // immediately
2 // immediately
Hello, es8 // after 2 seconds
```
这是因为函数调用不会阻塞流。
注意`async function`总是返回一个promise对象，而且`await`关键字只能在用`async`关键字声明的函数中使用。

![浏览器对Async functions的兼容性](http://upload-images.jianshu.io/upload_images/2976869-7f25ac5340008120.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###Shared meomory and atomics
当内存被共享时，多个线程可以在内存中读写相同的数据。原子操作确保预测值的可读和可写，在下一个线程开启前操作未结束，不会导致开启新的线程发生中断。这一部分介绍了一个新的构造器`ShareArrayBuffer`和带有静态方法的名称空间对象`Atomics`。
`Atomic`对象是一个类似于有很多方法的`Math`对象，可以向一个构造器一样去使用。这个对象的静态方法如下：
```
add / sub---add / 在一个特定位置上减去一个值
and/or/xor---与，或，异或
load---获取特定位置的值
```
![浏览器兼容性](http://upload-images.jianshu.io/upload_images/2976869-cd8524d10beaabd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###取消模板文字限制
使用带标记的模板文字(ES6)，我们可以做一些事情，比如声明一个模板解析函数，并根据我们的逻辑返回一个值:
```
const esth = 8;
helper`ES ${esth} is `;
function helper(strs, ...keys) {
  const str1 = strs[0]; // ES
  const str2 = strs[1]; // is
  let additionalPart = '';
  if (keys[0] == 8) { // 8
    additionalPart = 'awesome';
  }
  else {
    additionalPart = 'good';
  }
  
  return `${str1} ${keys[0]} ${str2} ${additionalPart}.`;
}
```
返回的值将是→ ES 8 is awesome.
对于esth中的7的值，返回的值将是→ ES 7 is good.
但是对于包含\u或\b子字符串的模板有一个限制。ES9将处理这个逃逸问题。在MDN网站或TC39文档中阅读更多相关信息。

![浏览器兼容性](http://upload-images.jianshu.io/upload_images/2976869-26351740d17b402d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###总结
JavaScript在生产中，但它总是不断更新。在规范中关于是否采用新特性的讨论过程是非常缜密的。在最后阶段，TC39委员会确认了这些特性，并由核心开发人员实现。它们中的大多数已经是Typescript、浏览器或其他多填充部分的一部分，所以现在就可以尝试它们了。

其他链接：
ES8web版：https://www.ecma-international.org/ecma-262/8.0/index.html
ES8PDF版：https://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf
JsonCheng中文版：https://github.com/xitu/gold-miner/blob/master/TODO/es8-was-released-and-here-are-its-main-new-features.md
高级功能修饰符：https://hackernoon.com/all-you-need-to-know-about-decorators-a-case-study-4a7e776b22a6
MDN官网：https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals
TC39文档：https://tc39.github.io/proposal-template-literal-revision/