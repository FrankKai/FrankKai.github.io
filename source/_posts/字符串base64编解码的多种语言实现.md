---
title: 字符串base64编解码的多种语言实现
date: 
tags: 
---

故事起源于逛v站的求职和招人贴，每次都是一长串字符，很迷惑，不知道是邮箱，还是微信号，还是钉钉，还是QQ。
感觉像是base64，又感觉不像，因为我之前只知道图片能转成base64，没想到字符串也能转，群里一问，还确实是base64。
于是有趣的事发生了，同学们纷纷用自己最擅长的语言，把base64 字符串编解码实现了一遍...
![image](https://user-images.githubusercontent.com/19262750/35339805-7ec378e4-015c-11e8-80f8-6933aa369a9a.png)
所以我把小伙伴们用各种方式实现的方法整理下，有linux shell，javascript，node，python，php，java，.net。

1.shell (author: Peng Zhao)
解码：`echo "a2FsZUBvdWNodGVhbS5jb20=" | base64 -d`
编码：`echo "kale@ouchteam.com" | base64`


2.javascript (author: Kai Gao)
```
var encodedData = window.btoa('kale@ouchteam.com'); // 编码
var decodedData = window.atob("a2FsZUBvdWNodGVhbS5jb20="); // 解码
console.log(encodedData,decodedData)
```

3.nodejs (author: Kai Gao)
```
//base64编码
var  b = new Buffer('kale@ouchteam.com');
var s = b.toString('base64')
console.log("邮箱编码:"+s)
//base64解码
var b = new Buffer('a2FsZUBvdWNodGVhbS5jb20=',"base64")
var s = b.toString();
console.log("邮箱解码:"+s)
```

4.python (author: Peng Zhao)
```
import base64
base64.b64encode("kale@ouchteam.com")
base64.b64decode("a2FsZUBvdWNodGVhbS5jb20=")
```
5.php (author: Chuang Shen)
```
<?php
$a = 'kale@ouchteam.com';
	$b = base64_encode($a);//编码
	echo $b;
	$c = base64_decode($b);//解码
	echo $c;  
?>
```
6.java (author: Chuang Shen)
```
String str = "kale@ouchteam.com";
    	String encodeStr = new String(Base64.encode(str.getBytes()));
    	System.out.println(encodeStr);
    	String decodeStr = Base64.base64Decode(encodeStr);
    	System.out.println(decodeStr);
```

7..net  (author: Peng Li)
```
static void Main(string[] args)
        {
            Console.WriteLine("输入:");
            var str = Console.ReadLine();
            //加密
            byte[] EncryptionByte = Encoding.UTF8.GetBytes(str);
            var EncryptionStr = Convert.ToBase64String(EncryptionByte);

            Console.WriteLine("加密结果：" + EncryptionStr);

            //解密
            byte[] DecryptionByte = Convert.FromBase64String(EncryptionStr);
            var DecryptionStr = Encoding.UTF8.GetString(DecryptionByte);

            Console.WriteLine("解密结果：" + DecryptionStr);

        }
```


