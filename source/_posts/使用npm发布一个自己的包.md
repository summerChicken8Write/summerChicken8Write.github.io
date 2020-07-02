---
title: 使用npm发布一个自己的包
date: 2020-06-19 14:56:24
categories: 
- web前端
tags: 
  - npm
  - webpack
---

> 记录一下npm发布包的过程，方便日后复习。

## 1、新建一个项目

`mkdir npm-publish-example`

### 进入目录

`cd npm-publish-example`

### 初始化

`yarn init`

执行完这个命令后会让你在命令行中填入一些项目基本信息（如：项目名称、版本号、作者等）,如果想用默认值的话一直按回车就行，这个以后可以修改，所以不用担心。

然后我们的项目目录中会生成一个`package.json`文件，打开后可以看到我们刚才输入的信息。

## 2、安装依赖

`yarn add -D webpack webpack-cli clean-webpack-plugin`

`yarn add`表示安装哪些包，`-D`参数表示这些包只在生产环境中使用。安装完成后`package.json`会新增如下代码

```
  "devDependencies": {
    "clean-webpack-plugin": "^3.0.0",
    "webpack": "^4.43.0",
    "webpack-cli": "^3.3.12"
  }
```

## 3、逻辑代码

我们在项目跟目录下新建一个`src`目录，并在该目录下新建一个`index.js`文件，作为打包入口，在`index.js`中我们写一些逻辑代码。

```
  function double (num) {
    if (typeof num === 'number') {
      return num * 2
    } else {
      return 'I need number~'
    }
  }

  module.exports = { double }
```
## 4、配置webpack

首先我们在项目根目录下新建一个`webpack.config.js`文件，里边写一些打包需要的配置。webpack4已经支持了无配置直接打包，我们为了更灵活的配置选择了用配置文件的方式。配置如下。

```
const path = require('path')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  mode: 'none',
  entry: './src/index.js',
  output: {
    filename: 'tool.js',
    path: path.resolve(__dirname, 'dist'),
    libraryTarget: 'umd',
    globalObject: 'this'
  },
  plugins: [
    new CleanWebpackPlugin()
  ]
}
```

`module.exports`中导出的对象就是webpack的配置，`entry`代表打包时的入口，`output`代表打包出口相关配置，`filename`和`path`分别是打包结果的文件名和路径，`libraryTarget`表示打包后的内容使用那种模块化规范暴露出来，其中`umd`兼容性最好，`globalObject`表示打包出来的内容挂在在哪个全局对象中，浏览器环境就是`window`，node环境就是`global`，由于上边`libraryTarget: 'umd'`兼容浏览器环境和node环境的模块化规范，所以将`globalObject`的值设为`this`。

## 5、配置package.json

目前为止我们完成了大部分工作，下面我们对`package.json`中的配置进行一下调整。
```
{
  "name": "npm-publish-example",
  "version": "1.0.0",
  "main": "dist/tool.js",
  "license": "ISC",
  "scripts": {
    "build": "webpack --config webpack.config.js"
  },
  "devDependencies": {
    "clean-webpack-plugin": "^3.0.0",
    "webpack": "^4.43.0",
    "webpack-cli": "^3.3.12"
  }
}
```
我们调整了`main`字段的内容，他代表项目的默认导出路径。并且新增了`script`字段，这样我们就可以通过执行`yarn build`命令来打包了，可以不用每次都输入一大段字符。

## 6、打包、测试

我们来执行一下`yarn build`命令，在根目录下出现了`dist/tool.js`。

然后执行`yarn link`命令进行本地测试，我们在跟目录下新建一个`test.js`文件输入：
```
const { double } = require('npm-publish-example')
console.log(double(2));
console.log(double('hello'));
```
执行`node test.js`，控制台打印出了：
```
$ node test.js
4
I need number~
```
说明代码正常工作了

## 7、发布

首先我们去npmjs.com注册一个账号，然后通过`npm adduser`配置账号密码信息，最后执行`npm publish`发布就可以啦。

如果报错了有可能是包的名字已存在了，所以事先先到官网里收一下名字是不是重复，或者使用`npm view <pkg>`命令来搜索名字是否存在

发布成功之后就可以像test.js中那样在别的项目中引用啦

[项目地址](https://github.com/summerChicken8Write/npm-publish-example)