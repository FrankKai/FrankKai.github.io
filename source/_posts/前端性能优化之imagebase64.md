---
title: 前端性能优化之imagebase64
date: 
tags: 
---

先来看一段将图片转换为base64字符的nodejs代码：
```
const fs = require('fs');
let files = fs.readdirSync('.');
let dist = fs.openSync('./base.txt', 'a');
let buf = '';
for (let i =0; i<files.length; i++) {
    let content =  fs.readFileSync(files[i]);
    
    buf += files[i] + '-->: data:image/' + files[i].split('.').slice(-1) + ';base64,'+ content.toString('base64') + '\r\n';
}
fs.writeSync(dist, buf);
fs.close(dist)
```
用法：
```
把这个文件移动到图片文件夹里面
打开控制台 cmd
cd 到放文件的目录
node image2base64.js
生成一个txt文件
打开文件选择你要的内容
```
___________________________________

解析：
>1.获取fs模块并且赋值给声明的常量变量fs
2.读取目录，返回一个所包含所有文件的数组，赋值给块作用域本地变量files
3.创建base.txt文件。以追加模式打开base.txt文件。如果文件不存在，则会被创建
4.定义空buffer字符串
5.遍历读取所有数组中所有文件
6.读取结果保存在content变量中
7.判断图片格式类型，并且将读取结果转换成base64格式，回车换行
buf=buf+tia.png+'-->:data:image/'+'tia.png'.split('.').slice(-1)+';base64,'+content.toString('base64')+'\r\n'
8.异步的 close(2)。 完成回调只有一个可能的异常参数

有几个问题值得思考？
>1.base64编码的原理是什么？
2.base64对图片编码的原理是什么，为什么不采用原先的256位编码，而采用64位编码？
3.base64一定比直接加载图片快吗？
4.base64编码的应用场景是什么？
5.除了base64编码，还有没有别的编码提速方案？

先来看第一个问题
**1.base64对字符进行编码**
wiki的［例子］（https://zh.wikipedia.org/wiki/Base64）说明了的base64的编码原理
　　这和通信原理中的编码非常相近，所以理解起来也相对容易。
　　说白了就是把256位的字符，采用base64算法，编码成64位算法，前者是2的8次方，后者是2的6次方，因此前者的一个字符是8位二进制码，后者是6位二进制码，前者从ASCII码表中匹配，后者从base64位表中匹配。

![](http://upload-images.jianshu.io/upload_images/2976869-1e7e593001df5535.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**2.base64压缩了文件大小**　
 　　一般应用比较广泛的是png8图，每一张“png8”图像，都最多只能展示256（2的8次方）种颜色，8位的png其实支持不透明、索引透明、alpha透明。所以“png8”格式更适合那些颜色比较单一的图像，例如纯色、logo、图标等；因为颜色数量少，所以图片的体积也会更小。
举个例子来说：
　　比如love.png，文件大小是2KB，其实是2×1024byte，也就是2048字节，也就是2048个8位二进制序列，按照ASCII转换的话，就是2048个字符，也就是类似SaqeqWEQwQ871...这个的2048个字符。
　　采用base64算法将其转换成base64格式后，这个图片会转换成2048×8/6==2734，即2734个6位二进制序列，按照Base64表转换的话，就是2048个字符，也就是类psadSaqeqWEQ1231171...这个的2734个字符。
　　聪明的你也许会问？2734个字符比2048个字符多，为什么反而更加快速了呢？
　　抛开别的不谈，我们只谈当前问题本身，从数据库思维去思考，从一张256索引表中查找2048个字符串，与从64索引表中查找2734个字符串，哪个成本更低一点，显然256×2048远大于64×2734。
　　也就是说，在一定范围内，base64解析速度更快。
**3.不一定**
　　图片的 base64 编码就是可以将一副图片数据编码成一串字符串，使用该字符串代替图像地址。
　　这样做有什么意义呢？我们知道，我们所看到的网页上的每一个图片，都是需要消耗一个 http 请求下载而来的（所有才有了 csssprites 技术的应运而生，但是 csssprites 有自身的局限性，下文会提到）。
　　没错，不管如何，图片的下载始终都要向服务器发出请求，要是图片的下载不用向服务器发出请求，而可以随着 HTML 的下载同时下载到本地那就太好了，而 base64 正好能解决这个问题。
　　所以，在这里要明确使用 base64 的一个前提，那就是被 base64 编码的图片足够尺寸小
　　使用 Base64 的好处是能够减少一个图片的 HTTP 请求，然而，与之同时付出的代价则是 CSS 文件体积的增大。
　　Base64 跟 CSS 混在一起，大大增加了浏览器需要解析CSS树的耗时。其实解析CSS树的过程是很快的，一般在几十微妙到几毫秒之间。
**4.base64的应用场景**
无额外请求
对于极小或者极简单图片
可像单独图片一样使用，比如背景图片重复使用等
没有跨域问题，无需考虑缓存、文件头或者cookies问题  
**5.其他编码提速方案**
为什么不是base128？base256？base32？base16？
　　因为这128个字符中的**一些是不可打印的**（主要是那些位于代码点0x20之下的字符）。因此，它们不能被可靠地以线的方式传输。而且，如果你去上面的代码点128，你可能会遇到编码问题，因为在**系统**之间使用不同的编码。
　　因为Base64编码 比十六进制编码更节省磁盘容量，所以一般较大的数据需要进行 ASCII 编码多采用 Base64；而较小的数据，则使用易于人工识别十六进制（用纸笔就能解码出来）。
参考：
https://zh.wikipedia.org/wiki/Base64
http://celeron633.blog.sohu.com/263502449.html
http://www.cnblogs.com/coco1s/p/4375774.html
https://stackoverflow.com/questions/6008047/why-dont-people-use-base128
https://segmentfault.com/q/1010000000801988