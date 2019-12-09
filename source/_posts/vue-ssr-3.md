---
title: vue-ssr-3
date: 2019-12-06 15:57:34
tags: 
- vue-ssr
categories: 
- vue-ssr
---

基于之前构建的[webpack4+babel7+vue2](https://qinhanwen.github.io/2019/11/10/webpack4-babel7项目构建/#more)项目









**遇到的报错**

![WX20191205-213854@2x](http://www.qinhanwen.xyz/WX20191205-213854@2x.png)

问题原因：引入的flexible使用了window对象

解决方案：把引入flexible的地方放到client-entry.js文件

