---
title: React结合Webpack使用
date: 2016-08-25 09:59:51
tags: [React,Webpack]
---

## 准备工作

因为要使用webpack，需要先在电脑上安装nodeJs软件，并配置好电脑环境变量，即path中增加一条nodejs的安装路径（windows 7系统是自动添加的）。
本文操作均按照本人已经在实际项目中编写的路径中填写，react相关根目录为viewer，即react相关代码均在该目录下编写。

### 安装webpack

``` bash
$ npm install webpack -g
```

参数-g表示我们将全局(global)安装 webpack, 这样你就能使用 webpack 命令了.
webpack 也有一个 web 服务器 webpack-dev-server, 我们也安装上

``` bash
$ npm install webpack-dev-server -g
```

方便我们修改后界面可以及时自动刷新。
全局安装：就是再电脑的任何一个文件夹下打开命令窗口都可以使用该命令。
实际安装路径：C:\Users\帐户名\AppData\Roaming\npm\node_modules

### 配置package.json

如果是第一次运行，则直接在相应目录（viewer）下执行npm init 会自动生成该文件，生成后需要修改该文件的script的键值如下：

``` bash
"scripts": {
    "start": "webpack-dev-server --hot --colors"
  },
```
现在在命令窗口下执行 npm (run) start 相当于 webpack-dev-server.当项目变得相当复杂时, 你可以使用这种技巧隐藏其中的细节.

另外，如果需要在该文件中添加注释，则要在description中添加，如：

``` bash
"description": {
    "start": "开启小型服务器使用热加载",
  },
```

### 编写webpack配置文件

webpack 使用一个名为 webpack.config.js 的配置文件(默认), 在相应目录下创建该文件. 配置如下:

``` bash
//用于提取多个入口文件的公共脚本部分，然后生成一个 xxx.js 来方便多页面之间进行复用。
module.exports = {
    entry:{//指定打包的入口文件，每有一个键值对，就是一个入口文件
    },
    output: { //配置打包结果，path定义了输出的文件夹，filename则定义了打包结果文件的名称
    },
    devServer: {//热加载服务器的配置
        inline: true,
        port: 7777
    },
    resolve: {//定义了解析模块路径时的配置，常用的就是extensions，可以用来指定模块的后缀，这样在引入模块时就不需要写后缀了，会自动补全
          extensions: ['','.js','.jsx']
      },
    module: {//定义了对模块的处理逻辑，这里可以用
        loaders: [//loaders定义了一系列的加载器，以及一些正则。当需要加载的文件匹配test的正则时，就会调用后面的loader对文件进行处理，这正是webpack强大的原因。
    },
};
```

指定相应的入口文件和出口文件以及对相应文件的加载器，比如我们写的js文件是jsx语法需要通过jsx-loader来加载的。
一开始说了这个文件是默认的webpack配置文件名称，如果想要名称自定义也可以通过如下命令处理：

``` bash
$ webpack --config library.config.js
```

一般来说，是不会修改默认的文件的，但是做到后面会发现，有些事件等情况可以完全打包成一个独立的库处理，那么就需要经常使用这种类似的命令，每次输入这样一行的命令难免会觉得繁琐，这个时候就可以在package.json配置文件中添加script键值以及相应注释以便其他开发人员了解。

``` bash
"scripts": {
    "wlib": "webpack --config library.config.js"
  },
  "description": {
    "wlib":"按照library.config.js配置打包文件,启动方法： npm run wlib (只会运行一次 不改变默认配置文件)",
  },
```

### 安装依赖
以后要是多个项目需要用到的话，建议直接全局安装，需要引用哪个直接使用 npm link加载进来，避免到时候另一个项目需要用到又要重新下载安装一遍，费时。

同时注意：如果想要删除其中一个可以使用npm unlink 但不要使用右键删除这个方式，这个方式会导致全局的这个资源包也会被删除掉，之后又得重新下载。

#### 安装React

``` bash
$ npm install react react-dom --save
```

–save和–save-dev可以省掉你手动修改package.json文件的步骤。
npm install module-name –save 自动把模块和版本号添加到dependencies部分
npm install module-name –save-dev 自动把模块和版本号添加到devdependencies部分

#### 安装jsx,css的loader

``` bash
$ npm install webpack jsx-loader css-loader style-loader --save
```

然后配置 webpack.config.js 来使用安装的 loader.

```bash
module: {//定义了对模块的处理逻辑，这里可以用
    loaders: [//loaders定义了一系列的加载器，以及一些正则。当需要加载的文件匹配test的正则时，就会调用后面的loader对文件进行处理，这正是webpack强大的原因。
         { test: /\.js$/, loader: 'jsx-loader' },
        { test: /\.css$/, loader: "style!css" }
    ]
},
```

介绍到这里，那么可以看下配置文件还剩哪里没有填写，就是入口文件和出口文件，入口文件就是我们自己编写的jsx文件，这里直接以实际项目中为例，先看下项目文件结构图：
viewer文件结构图
入口文件为 main.js 出口文件可以自定义，这样的写法是main里面所有引用的其他文件（js、css等）打包成一个文件。

``` bash
entry:{//指定打包的入口文件，每有一个键值对，就是一个入口文件
        main:["./component/main.js"]
    },
    output: { //配置打包结果，path定义了输出的文件夹，filename则定义了打包结果文件的名称
        filename: "export/[name].js",
    },
```

入口文件写法格式为 key:value 形式，出口文件可以直接通过[name]的写法直接获取到key。

综上已经可以打包一个简单的项目js出来了，然后 直接在 html 页面引入这个js标签即可。

> 本篇文章参考其他blog如下：

[WebPack实例与前端性能优化](http://www.cnblogs.com/giveiris/p/5237080.html)
[如何使用 Webpack 模組整合工具](https://rhadow.github.io/2015/03/23/webpackIntro/)