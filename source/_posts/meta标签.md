---
title: meta标签
date: 2018-11-24 13:15:22
tags: 
- HTML
categories: 
- HTML
---

# meta标签学习笔记

##### 1.meta标签定义和用法

META标签用来描述一个HTML网页文档的属性，例如作者、日期和时间、网页描述、关键词、页面刷新等。它提供的信息虽然用户不可见，但却是文档的最基本的`元数据`。

标签位于文档的头部，永远位于 head 元素内部，不包含任何内容，并且没有结束标签。

> 云数据：用来概括描述数据的一些基本数据。也就是描述数据的数据



##### 2.name属性

###### name属性主要用于描述网页，比如网页的关键词，叙述等。与之对应的属性值为content，content中的内容是对name填入类型的具体描述，便于搜索引擎抓取。meta标签中name属性语法格式是：

```html
<meta name="参数" content="具体的描述">。
```



1）keywords(关键字)

说明：用于告诉搜索引擎，你网页的关键字。

```html
<meta name="keywords" content="Hexo, NexT">
```



2）description(网站内容的描述)

```html
<meta name="description" content="文科生，热爱前端与编程。目前大二，这是我的前端博客">
```



3）viewport(移动端的窗口)

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

width=device-width：不会出现滚动条（如果不设置width就会取设备的`layout viewport`的值）

initial-scale ：页面初始缩放值

user-scalable=no：不允许手动缩放

maximum-scale=1：允许用户缩放到的最大比例

minimum-scale=1： 允许用户缩放到的最小比例



##### 顺带一提移动端设备300ms点击延迟的问题

> `双击缩放(double tap to zoom)`，这也是会有上述 300 毫秒延迟的主要原因。
>
> 双击缩放，顾名思义，即用手指在屏幕上快速点击两次，iOS 自带的 Safari 浏览器会将网页缩放至原始比例。 那么这和 300 毫秒延迟有什么联系呢？ 假定这么一个场景。用户在 iOS Safari 里边点击了一个链接。由于用户可以进行双击缩放或者双击滚动的操作，当用户一次点击屏幕之后，浏览器并不能立刻判断用户是确实要打开这个链接，还是想要进行双击操作。因此，iOS Safari 就等待 300 毫秒，以判断用户是否再次点击了屏幕。 鉴于iPhone的成功，其他移动浏览器都复制了 iPhone Safari 浏览器的多数约定，包括双击缩放，几乎现在所有的移动端浏览器都有这个功能。之前人们刚刚接触移动端的页面，在欣喜的时候往往不会care这个300ms的延时问题，可是如今touch端界面如雨后春笋，用户对体验的要求也更高，这300ms带来的卡顿慢慢变得让人难以接受。
>
> 转：https://www.jianshu.com/p/6e2b68a93c88



解决方案：

**1.禁用缩放**

```html
<meta name="viewport" content="user-scalable=no">
<meta name="viewport" content="initial-scale=1,maximum-scale=1">
```

表明页面不可缩放，双击缩放功能失去意义

缺点：完全禁止缩放达到目标，通常情况下，还是想要使用缩放功能来查看一些看不清的内容。



**2.更改默认的视口宽度**

```html
<meta name="viewport" content="width=device-width">
<!--我的理解就是让layout viewport宽度等于visual viewport宽度-->
```

大部分移动设备上的应用，都会做适配处理，也就是不需要双击缩放。

优点：只是禁用了浏览器默认的双击缩放行为，用户仍然可以通过双指缩放操作来缩放页面



**在ios10以后上面这两个方法失去了效果**

我所了解的解决方案：手势（gesture）事件

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>

<body>
    sadsadk;sak;d
</body>
<script>
window.onload = function(){
    document.addEventListener('gesturestart',function(event){
        event.preventDefault();
    })
}
</script>
</html>
```

gesturestart：当一个手指已经按在屏幕上，另一个手指又触摸屏幕的时候触发



3.CSS touch-action**

emm。。这个先不看







##### 又顺带一提点击穿透问题



> **点击穿透**：假如页面上有两个元素A和B。B元素在A元素之上。我们在B元素的touchstart事件上注册了一个回调函数，该回调函数的作用是隐藏B元素。我们发现，当我们点击B元素，B元素被隐藏了，随后，A元素触发了click事件。
>
> 因为在移动端浏览器，事件执行的顺序是touchstart > touchend > click。而click事件有300ms的延迟，当touchstart事件把B元素隐藏之后，隔了300ms，浏览器触发了click事件，但是此时B元素不见了，所以该事件被派发到了A元素身上。如果A元素是一个链接，那此时页面就会意外地跳转。



4）robots(定义搜索引擎爬虫的索引方式)

robots用来告诉爬虫哪些页面需要索引，哪些页面不需要索引。content的参数有all,none,index,noindex,follow,nofollow。默认是all。

```html
<meta name="robots" content="none">
```



5）author(作者)

```html
<meta name="author" content="Lxxyx,841380530@qq.com">
```



##### 3.http-equiv属性

相当于http的文件头作用，它可以向浏览器传回一些有用的信息。

```html
<meta http-equiv="参数" content="参数变量值">
```



1）Expires(期限) 

可以用于设定网页的到期时间。一旦网页过期，必须到服务器上重新传输。 

```html
<meta http-equiv="expires" content="Fri,12Jan200118:18:18GMT"> 
```



2）Pragma(cache模式) 

禁止浏览器从本地计算机的缓存中访问页面内容

```html
<meta http-equiv="Pragma" content="no-cache">
```

似乎只有ie才可以识别这个，而其他的浏览器是Cache-Control: no-store的标签（不过现在浏览器好像放弃了使用meta控制浏览器缓存）

3）Refresh(刷新) 

```html
<meta http-equiv="Refresh" content="秒数; url=跳转的文件或地址" > 
<!--或者-->
<meta http-equiv="Refresh" content="秒数" url="跳转的文件或地址" > 
```



> 参考资料：https://segmentfault.com/a/1190000004279791