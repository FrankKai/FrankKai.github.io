---
title: create-react-app如何使用less，sass和react-css-modules?
date: 
tags: 
---

- create-react-app如何使用less？
- create-react-app如何使用sass？
- create-react-app如何使用react-css-modules？

Thanks @[pengzeya](https://github.com/zeyap)

### 1.create-react-app如何使用less？
安装less-watch-compiler到开发依赖。
```
npm i less-watch-compiler --save-dev
```
添加scripts
```
  "scripts": {
    "build-css": "less-watch-compiler --run-once --main-file=index.less src/less src/css",
    "watch-css": "npm run build-css && less-watch-compiler --main-file=index.less src/less src/css"
  }
```
运行命令
```
npm run watch-css
```
运行结果
创建css目录并生成编译后的index.css文件
```
src/less/index.less    →    src/css/index.css
```
程序运行起来后，会监控对应目录下的.less文件，热更新到css目录下。

参考：https://github.com/facebook/create-react-app/issues/3457

### 2.create-react-app如何使用sass？
安装node-sass-chokidar到依赖
```
npm install --save node-sass-chokidar
```
添加scripts
```
"build-css": "node-sass-chokidar src/sass -o src/css",
"watch-css": "npm run build-css && node-sass-chokidar src/sass -o src/css --watch --recursive",
```
运行命令
```
npm run watch-css
```
运行结果
创建css目录并生成编译后的index.css文件
```
src/sass/index.scss    →    src/css/index.css
```

程序运行起来后，会监控对应目录下的.scss文件，热更新到css目录下。

参考：https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#adding-a-css-preprocessor-sass-less-etc

### 3.create-react-app如何使用react-css-modules？
暴露出配置文件：
```
npm run eject
```
运行eject命令后，CRA会自动把wepack中与css-modules相关的依赖为我们准备好，无需新增多余的依赖。

修改config/webpack.config.dev.js：
```
// 修改前
{
  loader: require.resolve('css-loader'),
  options: {
    importLoaders: 1,
  },
},
// 修改后
{
  loader: require.resolve('css-loader'),
  options: {
    importLoaders: 1,
    modules: true,
    localIdentName: "[name]__[local]___[hash:base64:5]"  
  },
},
```

修改config/webpack.config.prod.js：
```
// 修改前
{
  loader: require.resolve('css-loader'),
  options: {
    importLoaders: 1,
    minimize: true,
    sourceMap: true,
   },
},
// 修改后
{
  loader: require.resolve('css-loader'),
  options: {
    importLoaders: 1,
    modules: true,
    minimize: true,
    sourceMap: true,
   },
},
```
代码变化：
引入CSS Modules之前:
```
import React, { Component } from 'react';
import logo from './logo.svg';
import './foo.css';
class App extends Component {
  render() {
    return (
      <div className="bar">

      </div>
    );
  }
}
export default App;
```
引入CSS Modules之后：
```
import React, { Component } from 'react';
import logo from './logo.svg';
import foo from './foo.css';
class App extends Component {
  render() {
    return (
      <div className={foo.bar}>

      </div>
    );
  }
}
export default App;
```
添加css-modules之前：
```
.bar{
    display: block;
    background: yellow;
}
```
添加css-modules之后：
```
.foo__bar__1t6eA{
    display: block;
    background: yellow;
}
```
运行结果
```
.bar   →    foo__bar__1t6eA
```
注意事项：
- 将import进来的'./SomeComponent.css'分配给一个本地常量，例如，import foo from './foo.css';
- 在JSX中将className替换成的形式{styles.myClass}，例如，className={styles.myClass}.
- 需要修改CSS文件和JSX中的class名。因为CSS Modules不允许"-"出现在类名中，有过开发经验的朋友都知道，正则中有"word"与"none-word"之分，word包括0~9,a~z以及下划线。而"-"不属于单词，因此CSS Modules不支持。例如我们需要将.Foo.bar修改为.bar，某些方面也简化了我们的代码。使用CSS Modules之后，我们就不需要再使用BEM之类的CSS规范。

参考：https://medium.com/nulogy/how-to-use-css-modules-with-create-react-app-9e44bec2b5c2