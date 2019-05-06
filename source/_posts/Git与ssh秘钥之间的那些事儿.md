---
title: Git与ssh秘钥之间的那些事儿
date: 
tags: 
---

 ![ssh](http://upload-images.jianshu.io/upload_images/2976869-bb9185153442b339.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果玩儿过github或者gitlab的话，大家都知道有2种克隆文件的方式，https和ssh。
https克隆写法为：`git clone https://github.com/FrankKai/machine-learning-notes.git`。
ssh克隆写法为：`git clone git@github.com:FrankKai/machine-learning-notes.git`。
写法上大同小异，无外乎是更换了一下协议名称，那么从开发效率上来说，哪种方式更方便快捷呢？
答案就是ssh。
因为https克隆时，每次都需要输入账号和密码，比较麻烦。
而ssh的话，只需要首次在本地生成一个ssh公钥，然后在远端添加，后面就不用再输账号密码了。（至于ssh与https两种协议安全性之间的对比，这种级别的问题还不是现在的我现在要去考虑的）

我们回过头来继续看ssh。ssh，是使用git进行多人合作的必会知识，最近我就因为对其掌握不精通，导致闹出了一个多个git地址需要多台计算机的笑话。

原理方面暂不深究，我只讲一下几个工作中非常需要注意的点，我将以windows系统为例。

>目录：C:\Users\frank\.ssh
私钥：id_rsa（不用理会）
公钥：id_rsa.pub（重要文件，本地和远端建立连接的凭证，包括ssh-rsa，公钥串和邮箱）
主机：known_hosts（所有已添加过SSH连接的域名，ip，ssh-rsa以及类似公钥的密码串）

如果看了我的描述理解很模糊，可以自己打开文件看下其中的内容，眼见为真，也更容易去理解和记忆。

我们知道git仓库有很多种，github，gitlab，码云，以及阿里云的云效，可以简单将其理解为一个专门用来存放代码的在线数据库。

现在我本机已经配置了github，gitlab以及码云，如果我想再配置阿里云的云效，该怎么做？

**第一步：配置git全局用户名和邮箱**
```
git config --global user.name "frank"
git config --global user.email  "12345678@qq.com"
```
**第二步：克隆项目**
```
git clone git@code.aliyun.com:schbrain/boat-app.git
```
此时会报错：pemission denied（publickey）
这时就需要生成客户端与主机之间的ssh key了。

**第三步：生成ssh-key公钥**
```
ssh-keygen -t rsa -C "12345678@qq.com"
```

**第四步：查看ssh-key公钥**
C:\Users\frank\\.ssh\id_rsa.pub
如果是linux系统，可以到指定目录使用cat命令查看，但是由于我们是windows系统，因此此处需要使用记事本，sublime text等文本编辑器去打开。

复制文件中的全部内容为第五步做准备。

**第五步：添加ssh-key公钥到目标仓库**
不同git仓库路径不一样，此处以阿里云的云效为例。
https://code.aliyun.com/
→https://code.aliyun.com/profile
→https://code.aliyun.com/profile/keys

如果对这5步操作步骤不清晰，可以再仔阅读下文件说明部分。

声明一下，这篇文章主要是以应用为主，其中的ssh key私钥，公钥以及已知主机文件背后的生成算法，以及本地和远端如何通过公钥进行认证等算法，这里不需要去管，涉及到的知识点实在太多了。

That's it !