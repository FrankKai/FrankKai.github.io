---
title: koa2如何设置和清除cookie
date: 
tags: 
---

![](http://upload-images.jianshu.io/upload_images/2976869-4d3c800b7fff82ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
推荐一部电影，《非常公寓》，1996年爱情悬疑片。
言归正传！
#### 一、预热
先看一个很关键的问题。
问：
　　**cookie与cookie中的session是什么？**
答：
　　**cookie是一种报文头信息，请求报文和响应报文中都可以有cookie，key-value的形式，主要用来识别用户身份。
　　cookie中的session是指会话，浏览器会话是指浏览器与服务器建立一次会话，关闭浏览器使得会话结束，此时session与cookie同时失效。与cookie有关的session，是指expires，默认为session，也就是cookie有效期，会话结束cookie失效。**
#### 二、正文
朴灵大神的《深入浅出nodejs》中对cookie进行了很精辟的讲解：
>由于HTTP是无状态协议，所以需要cookie来区分用户之间的身份。最新规范RFC6265中，定义cookie是一个由浏览器和服务器共同协作实现的规范。
Cookie的处理分为以下几步：
1.服务器向客户端发送cookie（我说怎么前端很少会用到cookie...）
2.浏览器将Cookie保存（后端设置expires或者maxAge，cookie有效期由后端来决定，默认为session）
3.之后每次浏览器都会讲Cookie发向服务器。（有效期内是这样）

下面来看几个有趣的问题。

**1.后端如何生成cookie（后端原生nodejs）**
搭建一个http服务器，并且手动生成cookie，保存到浏览器。
```
var http = require('http')
http.createServer(function(req,res){
    res.setHeader('Set-Cookie',['name=frank','age=23'])
    res.writeHead(400,{'Content-Type':'text/plain'});
    res.end('Hello World\n')
}).listen(1337,'127.0.0.1')
```
![](http://upload-images.jianshu.io/upload_images/2976869-bf5fb27cd4d58ee0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

expires/Max-Age 字段为此cookie超时时间。若设置其值为一个时间，那么当到达此时间后，此cookie失效。不设置的话默认值是Session，意思是cookie会和session一起失效。当浏览器关闭(不是浏览器标签页，而是整个浏览器) 后，此cookie失效。

**2.浏览器如何将cookie保存？（根据后端设置决定保存与否，需要在cookie有效期内）**
根据expires/MAX-AGE来保存。
现在的cookie生存周期只是一个session，也就是整个浏览器会话（单个tab标签页关闭Cookie依旧保存）
首次建立会话，只有response headers中有cookie.
![](http://upload-images.jianshu.io/upload_images/2976869-c0ff495ca4167bfb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
第二次建立会话，request headers中也有cookie

![](http://upload-images.jianshu.io/upload_images/2976869-2fb75a9bfcd07ad3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**3.如何设置expires？（后端设置，koa2，基于cookie.js模块）**
可能需要依赖redis，可能需要 koa2。
先不管redis，使用koa2来修改cookie有效时间。
~~先尝试原生方式，没有原生方式。~~
koa2方式成功。
![](http://upload-images.jianshu.io/upload_images/2976869-b318668514424bb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/2976869-f0b3314ac0576f01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
cookie有效期已经不再是session了。

代码如下：
```
const setCookie = ctx => {
    data = ctx.request.body;
    ctx.response.body = "Cookie设置成功"
    ctx.cookies.set('name', 'frank', { signed: true ,maxAge:7200000});
  	ctx.cookies.set('age', 23, { signed: true ,maxAge:7200000});
  	ctx.body = 'Hello World';
  };
app.use(route.get('/set', setCookie))
```
4.既然设置了cookie，服务器如何清除cookie呢？
我们清除的是浏览器上的cookie，让cookie过期就好。
设置cookie的值为空，并且设置cookie的maxAge为0，或者Expires设置为当前时间前的一个时间，例如当前时间1毫秒前Date.now()-1。
koa2框架清除cookie的具体操作代码为：
```
const deleteCookie = ctx =>{
    ctx.response.body = "Cookie删除成功"
    ctx.cookies.set('name','',{signed:false,maxAge:0})
    ctx.cookies.set('age','',{signed:false,maxAge:0})
    ctx.body = "Hello orld"
}
app.use(route.get('/delete',deleteCookie)
```
完整的**koa2设置cookie，清除cookie**代码示例如下：
```
const koa = require('koa');
const app = new koa();
app.keys = ['im a newer secret', 'i like turtle'];
// app.keys = new KeyGrip(['im a newer secret', 'i like turtle'], 'sha256');
const route = require('koa-route');
const setCookie = ctx => {
    data = ctx.request.body;
    ctx.response.body = "Cookie设置成功"
    ctx.cookies.set('name', 'frank', { signed: true ,maxAge:7200000});
  	ctx.cookies.set('age', 23, { signed: true ,maxAge:7200000});
  	ctx.body = 'Hello World';
  };
app.use(route.get('/set', setCookie))
const deleteCookie = ctx =>{
    ctx.response.body = "Cookie删除成功"
    ctx.cookies.set('name','',{signed:false,maxAge:0})
    ctx.cookies.set('age','',{signed:false,maxAge:0})
    ctx.body = "Hello orld"
}
app.use(route.get('/delete',deleteCookie))
app.listen(3000);
```
#### 三、思考
学习koa2的过程中，发现cookie设置有一个选项。

> ctx.cookies.set('name', 'frank', { signed: true ,maxAge:7200000});

官网给出的解释是：sign the cookie value，应该可以理解成为cookie设置签名的意思。
生成的cookie如下所示：

![](/img/bVZAHm)

生成age cookie的同时，也生成一个age.sig cookie，不是很理解，按照官网的意思，这是为cookie设置了一个签名。

产生2个疑问：
什么场景下需要为cookie设置签名？
什么场景下又不需要设置签名？

设置`signed cookie`，考虑的因素无非是**安全性问题**。

我们知道，发送 HTTP 请求，除了浏览器，还有各种代理。
即使设置了`httpOnly`，这一限制只对浏览器有效，我们同样可以把`cookie`拿过来更改，利用其他工具发送请求。

设置`signed cookie`后，`signed cookie`的生成和校验都是在服务器端处理，对客户端不可见，因此，也就达到防止篡改`cookie`的目的。

这样就安全了吗？再考虑这样一个问题，`cookie`是明文传输的，包括`signed cookie`，如果`cookie`的值是固定的，生成的`signed cookie`就是一致的，下次请求时，把这个`signed cookie`带上，就可以请求此`cookie`对应的数据了。所以，最安全的做法：还是`cookie`中不放敏感数据。

Koa2 中，`cookie`功能使用这个模块：[cookie](https://github.com/pillarjs/cookies) ，这个模块使用：[keygrip](https://github.com/crypto-utils/keygrip) 做数据的签名和验证。

至于什么时候使用`signed cookie`，什么时候不用，看你的安全性的要求了。
That it !