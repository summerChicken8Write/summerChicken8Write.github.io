---
title: 使用express和webhook搭建一个前端自动化部署服务
date: 2020-07-02 13:54:06
categories: 
- web前端
tags: 
- express
- node
- 部署
---

## 前言

在传统的前端开发流程中，我们完成本地的开发工作后需要将代码部署到服务器上。拿react为例，如果你使用的脚手架是create-react-app，在执行`npm run build`后，webpack会将代码打包在`/build`文件夹中，然后可以用Xftp等工具将文件夹中的代码上传到服务器中相应目录中（也可以在服务器中将代码pull下来，在服务器打包），然后用户就可以访问我们的网页。

但是，这个过程比较繁琐，需要登陆服务器、上传代码、打包等一系列手动操作，如果我们想自动完成这个过程可以选择jenkins等现有方案，也可以自己手动实现一个，其实原理非常简单~~

这里介绍一下webhook，通俗的讲它就是一个钩子，在你向远端仓库push代码的时候会触发这个钩子，并向你指定的地址发送一个POST请求。然后我们可以部署一个服务，在接收到这个请求时执行一系列操作，例如从远端拉取代码、构建、移动静态资源到指定路径或者打包docker镜像等一系列操作。接下来我们动手实现一下这个功能。

## 配置webhook

首先我们在github上新建一个仓库
然后点击设置
![pic1.png](https://i.loli.net/2020/07/02/C9EtdKJ8XQSNjMl.png)
在左边找到webhook
![pic2.png](https://i.loli.net/2020/07/02/zPCacbTqZNYenEg.png)
添加一条新规则
![pic3.png](https://i.loli.net/2020/07/02/k7vuzBZ4tsgXQJc.png)
填写这个表单，分别填写你的服务器地址、发送数据类型、密码和触发钩子的类型
![pic4.png](https://i.loli.net/2020/07/02/EAPyisqHhVvLYkn.png)

## 写测试代码

现在我们先写一段测试代码验证是否能正常接收到webhook请求

`yarn init`新建一个项目，`yarn add express`安装express，然后新建一个`server.js`作为入口文件，在里边写入如下内容：

```
const express = require('express')

const app = express()

const PORT = 2333
const HOST = '0.0.0.0'

app.post('/', (req, res) => {
  console.log(req)

  res.status(200)
  res.send('It is ok')
})

app.listen(PORT, HOST)

console.log(`webhook serve is runing on http://${HOST}:${PORT}`)
```

然后我们把这段代码丢到服务器上，用`node server.js`来运行它
ps: 别忘了在你的服务器上进行配置，开放对应端口的访问权限

## 测试

回到刚才的github中的webhook选项中，会看到刚才设置的规则列表，点进去，点击按钮，发送一次测试请求，然后在服务器上可以看到请求的信息打印了出来
![pic5.png](https://i.loli.net/2020/07/02/ekalfWiQBnUhsRH.png)

## 接下来

然后我们就可以在刚才的回调函数中写一些业务逻辑，利用node提供的child_process.exec就可以操作命令行来做原来我们手动处理的事情。

甚至你可以随意发挥加入请求校验、任务队列或者日志功能等任何你想要的东西

[项目地址](https://github.com/summerChicken8Write/webhook-watcher)