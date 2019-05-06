---
title: API网关express-gateway初体验
date: 
tags: 
---

昨天朋友发来一个消息，问我express源码中的一个问题，我去看了一番源码以后，发现自己并不是很懂，只能看懂一些字面意思...
![](https://upload-images.jianshu.io/upload_images/2976869-364a7c2ec54f3d15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/256)
不过在聊天过程中，了解到他们公司在用express做api网关，什么是api网关呢？
![](https://upload-images.jianshu.io/upload_images/2976869-0eb969259d431a1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/256)

我将以express-gateway为例，学习[get started教程](https://www.express-gateway.io/getting-started/)，对api网关进行探索。

安装
①npm i -g express-gateway
②创建一个express网关：eg gateway create
③根据提示选择server模板
④运行express网关：npm start

5分钟入门教程
目标：
1.选择一个微服务并且作为一个api暴露出去
2.定义一个api的消费者
3.使用key认证保证api的安全性

1.选择一个微服务并且作为一个api暴露出去
①直接访问微服务
`curl http://httpbin.org/ip`
```
{
   "origin": "73.92.47.31" # this will be your own IP address
 }
```
②指定微服务
![](https://upload-images.jianshu.io/upload_images/2976869-82d680270dfe4574.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在express gateway的一个默认的管道中，服务将被分配到一个端口。一个管道指的是一个策略集合。Express Gateway有一个代理策略。在默认的管道中，会使用这个代理策略，网关现在挡在了https://httpbin.org/ip服务前面，并且并且路由其它请求到网关的端口。

在config/gateway.config.yml这个文件中，可以找到一个serviceEndpoints选项，这里定义了httpbin这个服务。
```
serviceEndpoints:
   httpbin:
     url: 'https://httpbin.org'
```
在默认管道的proxy选项的action中，可以找到serviceEndpoint: httpbin

③以api的方式公开微服务
![](https://upload-images.jianshu.io/upload_images/2976869-65f646655302ea62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 通过Express Gateway，我们将暴露httpbin 服务到api端口。当通过api端口公开api后，外部可以访问到api。

在config/gateway.config.yml这个文件中，可以找到apiEndpoints选项，这里定义了api。
```
apiEndpoints:
    api:
      host: 'localhost'
      paths: '/ip'
```
![](https://upload-images.jianshu.io/upload_images/2976869-9aba088dbc885ec2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在我们有一个公共api浮出水面了，我们应该确保自己可以穿过express网关获取到service的权限。

2.定义一个api网关的消费者
管理我们api的人，在这里我们称之为“Consumer”。
![](https://upload-images.jianshu.io/upload_images/2976869-1e204f61e2c80052.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`eg users create`

3.使用key认证方式确保安全
①现在api暴露了出去，而且也获得了访问权限。我们现在将为其加上key安全认证。
在config/gateway.config.yml这个文件中，可以找到pipelines选项，这里定义了key-auth。
```
 pipelines:
    - name: getting-started
      apiEndpoints:
        - api
      policies:
        - key-auth:
        - proxy:
            - action:
                serviceEndpoint: httpbin
                changeOrigin: true
```
②为Consumer Bob分配key。

`eg credentials create -c bob -t key-auth -q`
![](https://upload-images.jianshu.io/upload_images/2976869-fe7ffed41afa9719.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



③Bob没有加key直接访问。
```
curl http://localhost:8080/ip
    //Unauthorized
```
![](https://upload-images.jianshu.io/upload_images/2976869-b061cc3368634f83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

④Bob加了key进行访问。
```
$ curl -H "Authorization: apiKey 0Er0Ldv5EHSUE364Dj9Gv:2Yzq1Pngs1JYaB2my9Ge4u" http://localhost:8080/ip
  {
    "origin": "73.92.47.31"
  }
```
![](https://upload-images.jianshu.io/upload_images/2976869-b9ca7b49de4a783f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

That's it!