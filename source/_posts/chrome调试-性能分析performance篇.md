---
title: chrome调试-性能分析performance篇
date: 2019-11-21 22:44:50
tags: 
- 性能监控
categories: 
- 性能监控
---

## Performance

Performance 工具的侧重点则在于前端渲染过程，它拥有帧率条形图、CPU 使用率面积图、资源瀑布图、主线程火焰图、事件总揽等模块，它们和渲染息息相关，善用它们可以清晰地观察整个渲染阶段。

首先我们打开 Chrome 匿名窗口，在匿名环境下，浏览器不会有额外的插件、用户特性、缓存等影响实验可重复性的因素。

![WX20191121-231138@2x](http://www.qinhanwen.xyz/WX20191121-231138@2x.png)

打开网页，打开Performance，有两个选项，立即记录和重载页面并记录

![WX20191121-231857@2x](http://www.qinhanwen.xyz/WX20191121-231857@2x.png)

点击掘金页面的reload收集渲染数据

![WX20191122-094818@2x](http://www.qinhanwen.xyz/WX20191122-094818@2x.png)

关于图中内容介绍：



![WX20191122-095138@2x](http://www.qinhanwen.xyz/WX20191122-095138@2x.png)

- `juejin.im #3 `这边是切换每一次的检测报告；
- `Screenshots `是用来查看在每个时间段界面的变化；
- `Memory` 存储调用栈的大小，在不同时间段的不同大小；



![WX20191121-235755@2x](http://www.qinhanwen.xyz/WX20191121-235755@2x.png)

- `Disable Javascript samples` 禁用 `javascript` 调用栈；
- `Enable advanced paint instrumentation (slow)` 记录渲染事件的细节；

- `Network` 用来修改检测在不同的网络环境下，界面的渲染；

- `CPU` 用来查看电脑的性能问题；



![WX20191122-110606@2x](http://www.qinhanwen.xyz/WX20191122-110606@2x.png)

- Network：资源加载的一个顺序情况，什么时间加载了什么资源



![WX20191122-111135@2x](http://www.qinhanwen.xyz/WX20191122-111135@2x.png)

- Frames：查看我们在什么时间，界面发生了改变，它和第一部分的区别就是在界面没有改变的时候，它是不做记录的，但是概览部分是会做记录的



- Timings



![WX20191122-111442@2x](http://www.qinhanwen.xyz/WX20191122-111442@2x.png)

- Main：俗称 **火焰图** , 这里是一个由下而上的事件执行图



![WX20191122-111725@2x](http://www.qinhanwen.xyz/WX20191122-111725@2x.png)

- Raster：在前端部分一共使用了几条浏览器渲染线程



![WX20191122-111826@2x](http://www.qinhanwen.xyz/WX20191122-111826@2x.png)

- GPU：网站在什么时间有 `GPU` 加速



![WX20191122-112346@2x](http://www.qinhanwen.xyz/WX20191122-112346@2x.png)

- 性能检测详情
  -  `Summary（性能摘要）` 
    - `Loading` ：加载时间
    - `Scripting` ：js计算时间
    - `Rendering` ：渲染时间
    - `Painting` ：绘制时间
    - `Other` ：其他时间
    - `Idle` ：浏览器闲置时间
  -  `Bottom-Up（事件列表，由下至上，对应 Main 火焰图）` 
    -  `Self Time（自己调用的时间）` 
    -  `Total Time（总调用时间，包括子项调用时间）` 
    -  `Activity（行为，包括调用该事件的位置）`
  -  `Call Tree（每个事件的子项信息）` 
    -  `Self Time（自己调用的时间）` 
    -  `Total Time（总调用时间，包括子项调用时间）` 
    -  `Activity（行为，包括调用该事件的位置）`
  -  `Event log（事件日志）`
    -  `Self Time（自己调用的时间）` 
    -  `Total Time（总调用时间，包括子项调用时间）` 
    -  `Activity（行为，包括调用该事件的位置）`
    -  `Start Time` 属性，这个属性其实就是开始的时间，从什么时间开始执行的什么事件



## 参考资料

https://juejin.im/post/5c8fa71d5188252d785f0ea3#heading-67