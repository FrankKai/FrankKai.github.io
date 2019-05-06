---
title: 前端开发神器之ngrok
date: 
tags: 
---

ngrok能做什么，为什么是前端开发神器？

**内网穿透，映射本地开发环境到公网，模拟多终端线上环境。**

结合一个很简单的[PWA demo](https://github.com/minimal-xyz/minimal-pwa)，举个简单的例子
####1.克隆demo到本地
```git clone https://github.com/minimal-xyz/minimal-pwa.git```

![](http://upload-images.jianshu.io/upload_images/2976869-e3064819f017b349.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/762)

####2.本地8080端口运行demo
```
cnpm i http-server -g
http-server -c-1
chrome http://localhost:8080
```

![](http://upload-images.jianshu.io/upload_images/2976869-06cf0595adc86695.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/762)


####3.ngrok内网穿透到公网
```
ngrok http 8080
```
####4.查看公网地址
```
chrome http://127.0.0.1:4040/inspect/http
```
![](http://upload-images.jianshu.io/upload_images/2976869-2f3174848bb56026.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/762)
####5.公网多终端访问
①使用另外一台或本机desktop
```
chrome http://733a1ad2.ngrok.io
```

![](http://upload-images.jianshu.io/upload_images/2976869-231baadfce53e28e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/762)


②使用android，safari mobile，ff mobile等mobile phones
地址栏手动输入http://733a1ad2.ngrok.io

![](http://upload-images.jianshu.io/upload_images/2976869-bcf9c1879a7bc220.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/512)

####优点：
1.一条url甩产品脸上
2.内网穿透多终端测试
3.监控请求和响应

####缺点：
1.响应时间较长
2.安全性不保证

####tips:
1.ngrok下载地址：https://ngrok.com/download
2.环境:git，node，cnpm，ngrok，Chrome
3.chrome和ngrok命令需设置环境变量
4.以上命令在终端中运行即可
5.协议选择https和http都行

题外话：
　　今天MDN官方宣布say goodbye to firebug，**say hello to pwa and view sources**，最后说了很多煽情的话。我只在很久以前用过一次firebug改dom装逼，所以对这些煽情的话无感，倒是对pwa和view source有了很深的兴趣。
　　于是借这个大好的加班前夜，学了下PWA，学习链接附上：**https://zhuanlan.zhihu.com/p/25459319**，他们家的各种教程不能太好，无论前端还是nodejs，真心不错，以后外卖我只吃饿了么。
　　无意中发现ngrok这个神器，一开始以为这家伙只能做内网穿透，后来发现既然公网地址提供出来了，手机上也可以访问啊，一下子有些兴奋，因为第一家实习时学到的移动端前端测试，需要手动配置本地localhost开发环境，而且需要连接到360免费wifi，较为麻烦。
　　其实ngrok还有很多其他的功能，有兴趣的同学可以到https://ngrok.com/product查看。

努力成为优秀的前端开发工程师！