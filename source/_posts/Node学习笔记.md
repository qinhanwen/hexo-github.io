---
title: Node学习笔记-准备工作
date: 2019-05-08 21:14:23
tags: 
- Node
categories: 
- Node
---

一些实用工具，安装过程就不写了



## 3m

**nvm（node version manager）：**Node.js版本管理工具，可以通过它安装多版本的Node，以及切换项目Node版本，常用命令：

```shell
$ nvm ls
$ nvm install v10.15.0
$ nvm uninstall v10.15.0
$ nvm use 10.15.0
```

**npm（node package manager）：**通过nvm安装高版本Node的时候，会同时安装npm，它是用来管理Node.js包的，[常用命令](https://qinhanwen.github.io/2019/06/29/npm/)

**nrm（node registry manager）：**解决npm镜像访问的问题，常用命令：

```shell
$ nrm ls
$ nrm test
$ nrm add xxx http:xxxx
$ nrm del xxx
$ nrm use taobao
```

![WX20191008-234629@2x](http://118.24.241.76/WX20191008-234629@2x.png)



## 多模块管理器lerna

专门用于管理Node.js多模块的工具

