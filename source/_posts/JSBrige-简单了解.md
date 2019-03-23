---
title: JSBrige-简单了解
date: 2019-01-29 22:55:47
tags: 
- JSBridge
categories: 
- JSBridge
---

> 参考资料：[cordova.js源码分析]( https://www.jianshu.com/p/866a32ac69ea)

通过整理[JSBridge分离插件js](https://qinhanwen.github.io/2018/12/11/JSBridge%E6%8E%A5%E5%85%A5%E6%96%87%E6%A1%A3/)稍微了解了cordova.js文件，还有不同平台间js通知原生层的方式。



#### JSBridge

在Hybrid App开发中，H5应用是内嵌到原生的WebView组件中，纯H5在很多功能使用都收到限制，而且功能的使用存在很多的兼容性问题，所以使用一些受限的功能时候需要依赖原生，而JSbridge就是JS与原生之间通信的桥梁。





#### JS调用Native

##### 1）**注入 API** 

其实就是向JavaScript的Context（window）注入方法或对象，在JavaScript调用的时候，执行相关的Native方法。



##### 2）**拦截 URL SCHEME**

比如：设置iframe的src，Native拦截到，再进行相对的操作。





#### Native调用JS

在Cordova.js里看到了将成功与失败的回调都保存在了一个对象里，并且有一个单独标识callbackId，Native可以通过callbackId获取对应的成功与失败回调，回传值给JavaScript。



