---
title: 如何理解package.json中的proxy字段？
date: 
tags: 
---

这篇文章其实也可以叫做：**使用create-react-app生成的项目如何配置代理服务器？**

>入职新公司以来，第一个月接手vue项目，第二个月接手angularjs项目，第三个月加入react重构项目。心生感叹：业务驱动式学习是一种高效率的学习方式，保持好奇心，在业务中快速成长！
新项目中在package.json中有一个proxy字段，这是我从来没接触过的，因此就有了此文的诞生，我使用create-react-app 新建了一个最原始状态的项目，对proxy字段与create-react-app之间的纠葛展开了学习。

在npm-configuration中，有如下解释：
>默认值为null，类型为url，一个为了发送http请求的代理。如果HTTP__PROXY或者http_proxy环境变量已经设置好了，那么proxy设置将被底层的请求库实现。

这个proxy字段目前我只了解到可以与create-react-app的react-scripts结合使用:Proxying API Requests in Development，react-scripts应该是基于HTTP_PROXY环境变量做了一些封装。

阅读完本文，你将有一以下收获：

- **如何更优雅地为前端项目配置代理Proxy服务器**
- **复现之前啃《HTTP权威指南》代理相关的知识**
- **对easy-mock的使用限制有了新的认识**
- **对process.env可以直接在React层展示感到震惊**
- **了解到对process.env可以进行扩展的dotenv和expand-env两个库**

主要分为3部分：
- 开发过程中的Proxy API 请求设置
- 手动配置proxy
- 环境变量式配置Proxy
### 开发过程中的Proxy API 请求设置
>注意：这个特性可以在react-scripts@2.3以及更高版本中使用。

人们通常从将服务于后端实现的host和port，同样也为前端react应用提供服务。
例如，在一个应用部署后，生产配置类似下面这样：
```
/                   -静态服务器返回React应用和index.html
/todos         -静态服务器返回React应用和index.html
/api/todos   -服务器会使用后端实现去处理所有/api/*的请求
```
但其实这样的设置不是必须的。然而，如果你确实有一个这样的设置，在不考虑重定向它们到其他的host和port开发环境下，那么写出像fetch('/api/todos')这样的请求时正常的。

为了告诉开发环境的服务器去代理任何开发环境中未知的请求到我们自己的api服务器，添加一个proxy到package.json的字段，例如：
```
"proxy":"http://localhost:4000"
```
使用这种形式的话，当你在开发环境中使用fecth('api/todos')的时候，开发环境的服务器将识别出这不是一个静态资源，然后将代理转发你的请求到http://localhost:4000/api/todos 作为一个回调。生产环境服务器只能代理没有text/html在Accept头中的请求。

方便的是，这就避免了CORS问题以及类似像下面这样的错误信息。
```
Fetch API cannot load http://localhost:4000/api/todos. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:3000' is therefore not allowed access. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
```
要知道proxy只有在开发环境中会有副作用，而且类似/api/todos 这样的URL在生产环境中是否指向正确取决于我们。你不需要使用/api前缀。任何没有text／html请求头的未识别的请求将会被代理到配置的服务器。

proxy选项支持HTTP，HTTPS以及WebSocket连接。
如果proxy选项还不够灵活的话，你可以去做自定义：
- [自己配置代理](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#configuring-the-proxy-manually)（未实验）
- 服务器端开启CORS（亲测，[express](https://enable-cors.org/server_expressjs.html)和koa均可实现，koa可以直接使用koa-cors）
- [使用环境变量注入正确的服务器以及端口到应用](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#adding-custom-environment-variables)（未实验）

工科男的执着：）
为了更好的说明问题，我们来做一次本地实验：
- 启动服务
```
npx creat-react-app my-app
cd my-app
npm run start
```
- 引入axios并发送请求
```
npm i axios --save
```
```
componentDidMount(){
    axios.get('/foo')
        .then(function (response) {
          console.log(response);
        })
        .catch(function (error) {
          console.log(error);
        });
}
```
请求发送："http://localhost:3000/foo"
错误信息：404

我们为package.json新增proxy服务器：
```
"proxy":"http://0.0.0.89:7300"
```

ctrl + s 热更新react代码后，没有生效，依旧报404的错误。

npm run start 重启本地服务后，代理服务器生效，返回正常的数据。

实现了自动将"http://localhost:3000" 请求转发到"http://0.0.0.89:7300" 的服务器。

不知道聪明的你们发现没有，我们并没有遇到CORS问题，因为在浏览器眼里，我们还是将请求发送到"http://localhost:3000" 中的，它并不知道creat-react-app已经将请求转发到了"http://0.0.0.89:7300" 这个所谓的会触发浏览器CORS安全策略的其他Origin。

天真的浏览器：
![image](https://user-images.githubusercontent.com/19262750/40101099-ea4b20c6-5918-11e8-9090-3996f3f255fa.png)

请求发送路径：
"http://localhost:3000" →"http://0.0.0.89:7300/foo"

响应返回路径：
"http://0.0.0.89:7300/foo" →"http://localhost:3000"
备注：
1.此处需要重新运行npm run start 重启本地服务，否则在package.json中设置的proxy不会被检测到并生效。
2.此处的服务器可以是公司内网某台虚拟机上的启动的node服务，也可以是easy-mock等mock服务器（仅支持公司内网部署版，大搜车公网线上服务器不支持）。

因此我们得出一个结论：
>creat-react-app脚手架可以结合package.json中的proxy实现请求转发。

实验成功！

### 手动配置proxy
>注意：这个特性可以在react-scripts@1.0.0以及更高版本中使用。

如果proxy的默认配置不够灵活，可以在package.json自定义一个像下面这样形式的对象。
你也可以http-proxy-middleware或者http-proxy去实现。
```
{
    “proxy”:{
        "/api":{
            "target":"<url>",
            "ws":true 
        }
    }
}
```
所有与这个路径相互匹配的请求将被代理转发。这包括了text/html类型的请求，这种类型是标准proxy选项不支持的。

如果你需要配置多个代理，你需要在定义几个入口。匹配规则还是那样，这样你才能使用正则匹配多个路径。
```
{
  // ...
  "proxy": {
    // Matches any request starting with /api
    "/api": {
      "target": "<url_1>",
      "ws": true
      // ...
    },
    // Matches any request starting with /foo
    "/foo": {
      "target": "<url_2>",
      "ssl": true,
      "pathRewrite": {
        "^/foo": "/foo/beta"
      }
      // ...
    },
    // Matches /bar/abc.html but not /bar/sub/def.html
    "/bar/[^/]*[.]html": {
      "target": "<url_3>",
      // ...
    },
    // Matches /baz/abc.html and /baz/sub/def.html
    "/baz/.*/.*[.]html": {
      "target": "<url_4>"
      // ...
    }
  }
  // ...
}
```

工科男的执着，继续来做一个实验：

依然使用上面的my-app项目，proxy配置如下：
```
"proxy":{
    "/api": {
      "target": "http://0.0.0.89:7300",
      "ws": true
    },
    "/foo": {
      "target": "http://0.0.11.22:8848",
      "ws": true,
      "pathRewrite": {
        "^/foo": "/foo/beta"
      }
    }
}
```
代码如下：
```
    axios.get('/api')
    .then(function (response) {
      console.log(response);
    })
    .catch(function (error) {
      console.log(error);
    });
    axios.get('/foo')
    .then(function (response) {
      console.log(response);
    })
    .catch(function (error) {
      console.log(error);
    });
```
执行结果：
api接口和之前一致，我们这里主要看重定向的foo接口。

请求发送路径：
"http://localhost:3000" →"http://0.0.11.22:8848/foo" →"http://0.0.11.22:8848/foo/beta"

响应返回路径：
"http://0.0.11.22:8848/foo/beta" →"http://localhost:3000"

可以配置对个代理，我们此处使用的是"http://0.0.0.89:7300" 和"http://0.0.11.22:8848" 这个两台代理服务器，其中
"http://0.0.0.89:7300" 提供了api接口，"http://0.0.11.22:8848" 提供了foo接口。而且我们可以在代理服务器上重定向接口。

因此我们得出一个结论：
>creat-react-app脚手架可以结合package.json中的proxy，可以配置对个代理，而且我们可以在代理服务器上重定向接口。

实验成功！

### 环境变量式配置proxy
>这个功能在react-scripts@0.2.3及更高本版中适用。

react的项目可以使用已经声明好的环境变量，这些变量就像是在你的js文件中定义的本地变量一样。默认情况下，已经有NODE_ENV默认环境变量，以及其他的以REACT_APP_为前缀的环境变量。

**环境变量在构建期间是被嵌入进去的。**因为Create React App提供了静态的HTML/CSS/JS打包，不能在runtime时被读取到。为了在runtime期间读取到环境变量，你需要还在HTML到服务器的内存，并且在运行时替换占位符，就像这里描述的这样：[Injecting Data from the Server into the Page](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#injecting-data-from-the-server-into-the-page)。另外你可以在任何你更改他们的时间里重新构建应用。
>你需要使用REACT_APP_创建通用的环境变量。除了NODE_ENV之外的任何其他的变量将被忽略，这是为了避免[exposing a private key on the machine that could have the same name](https://github.com/facebook/create-react-app/issues/865#issuecomment-252199527)。运行期间，只要你修改了环境变量，就需要重启开发服务器。

这些环境变量将被定义在process.env。例如，有一个名叫REACT_APP_SECRET_CODE的环境变量，它可以通过process.env.REACT_APP_SECRET_CODE暴露在我们的javascript文件中。

我们这里同样也有一个内建的叫做NODE_ENV的环境变量。你可以通过process.env.NODE_ENV去读取它。当你运行npm start时，NODE_ENV的值是development，当你运行npm test时，NODE_ENV的值是test，而且当你运行npm run build构建生产环境的包的时候，它通常是production。**你不能的手动覆盖NODE_ENV。**这样可以预防开发者错把开发环境的代码部署到生产环境。

这些环境变量可以用于根据项目的部署位置或使用超出版本控制的敏感数据来有条件地显示信息。

首先，你需要一个已经定义的环境变量。例如，你想在form表单中控制一个secret变量。
```
render(){
    return (
        <div>
            <small>你的应用运行在<b>{process.env.NODE_ENV}</b>模式。</small>
            <form>
               <input type="hidden" defaultValue={process.env.REACT_APP_SECRET_CODE} />
            </form>
        </div>
    );
}
```
在构建期间，process.env.REACT_APP_SECRET_CODE将会被环境变量中的当前值替代。谨记NODE_ENV是自动设置的变量。

当你在浏览器查看input时，它已经被设置成了abcde（或者是空）。
上面的表单从环境变量中搜索一个名叫REACT_APP_SECRET_CODE的变量。为了使用这个值，我们需要将其定义在环境中。使用两种方式可以做到，一种是在shell中定义，一种是.env文件中。

可以通过NODE_ENV去对一些操作进行控制：
```
if(process.env.NODE_ENV !== 'production'){
   analytics.disable();
}
```
当你使用npm run build编译app时，将会使文件变得更小。

在HTML中引用环境变量：
>注意：这个特性在react-scripts@0.9.0以及更高版本中使用。

你可以在public/index.html中获取到以REACT_APP_为前缀的环境变量。例如：
<title>%REACT_APP_WEBSITE_NAME%</title>

注意事项：
- 除了内建变量(NODE_ENV和PUBLIC_URL)，变量名必须以REACT_APP_开头才能正常工作。
- 构建期间环境变量可以被注入进去。如果你想在运行期间注入它们，采用这个方法：[Generating Dynamic <meta> Tags on the Server](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#generating-dynamic-meta-tags-on-the-server)

在shell中添加临时的环境变量
对于不同的操作系统，环境变量的设置是不同的。但是更加需要注意的是，这是创建变量的方式仅仅是当前shell session窗口有效。

Linux和macOS(Bash)
```
REACT_APP_SECRECT_CODE=abcdef npm start
```
还有一种创建.env文件定义环境变量的方式。

.env文件将被检如源代码控制。

其他.env文件将怎么使用？
>这个特性仅在react-scripts@1.0.0及更高中使用
使用dotenv可以将.env中的值注入到process.env中。例如：
```
require('dotenv').config()
```
**在项目的根目录定义一个.env文件并键入如下内容**：
```
DB_HOST=localhost
DB_USER=root
DB_PASS=s1mpl3
```
然后就可以在process.env中访问到了：
```
const db = require('db')
db.connect({
  host: process.env.DB_HOST,
  username: process.env.DB_USER,
  password: process.env.DB_PASS
})
```
```
.env: Default.
.env.local: Local overrides. 加载除了test之外的环境变量。
.env.development, .env.test, .env.production: 公用的环境变量。
.env.development.local, .env.test.local, .env.production.local:本地的环境变量。
```
左边的比右边的优先级高：
```
npm start: .env.development.local, .env.development, .env.local, .env
npm run build: .env.production.local, .env.production, .env.local, .env
npm test: .env.test.local, .env.test, .env (note .env.local is missing)
```
如何将系统环境变量扩展到我们项目下的.env文件使用：

使用dotenv-expand
```
REACT_APP_VERSION=$npm_package_version
# also works:
# REACT_APP_VERSION=${npm_package_version}
```
在.env文件内部也可以使用变量：
```
DOMAIN=www.example.com
REACT_APP_FOO=$DOMAIN/foo
REACT_APP_BAR=$DOMAIN/bar
```

工科男的执着：）
简单做个实验：
```
touch .env
code .env
```
键入：
```
DB_HOST=localhost
DB_USER=root
DB_PASS=s1mpl3
```
代码：
```
console.log("process.env.DB_HOST-->%s,process.env.DB_USER-->%s,process.env.DB_PASS-->%s",process.env.DB_HOST,process.env.DB_USER,process.env.DB_PASS)
```
实验结果：
```
process.env.DB_HOST-->undefined,process.env.DB_USER-->undefined,process.env.DB_PASS-->undefined
```
实验结果并不总是令人满意，问题在于不知道在何处require('dotenv').config()，可能需要在node层引入，也可能需要借助webpack之类的工具，使得view层能访问到。

实验失败。
做一下总结：
- 开发过程中的Proxy API 请求设置(默认选型，满足大多数情况下需求)
- 手动配置Proxy （可实现多代理，重定向）
- 环境变量式配置Proxy (临时变量方式简单易用，.env方式较为复杂，可以使用配置文件代替)

充实的一天！