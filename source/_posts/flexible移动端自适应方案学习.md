---
title: flexible移动端自适应方案学习
date: 2018-11-19 10:12:21
tags: 
- 深入学习
categories: 
- 深入学习
---

# flexible移动端适配学习笔记

> 参考资料
>
> https://www.w3cplus.com/css/viewports.html
>
> https://www.w3cplus.com/mobile/lib-flexible-for-html5-layout.html
>
> https://www.w3cplus.com/css/towards-retina-web.html



##### 1.需求背景

其实就是为了各种机型适配。

选择iphone6设计稿为基准，来适配其他机型



##### 2.使用之前需要先了解的基础（暂时忽略兼容性问题和异常情况）

###### 桌面设备浏览器

------

 1) 屏幕尺寸（设备的pixels）

​       从`screen.width/screen.height`获取。(这个是显示器的特征，与浏览无关，不会因为缩放而改变)。如图：

![](http://39.105.62.145/assets/images/1542689394373.jpg) 



2) 浏览器尺寸（CSS的pixels）

​       从`window.innerWidth/window.innerHeight`获取，浏览器窗口的内部宽度使用CSS的pixels（意思就是我打开浏览器，能看到里面的`元素`会因为我`放大或者缩小`浏览器而`变少或者变多`），而缩放比例就是如图：

![](http://39.105.62.145/assets/images/1542690089176.jpg)



3) 视窗 viewport

​    viewport的功能在于控制`<html>`元素，`viewport是严格的等于浏览器的窗口`。

​    举个例子：

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<style>
    .navbar {
        height: 200px;
        background: #000;
        width:100%;
    }

    .navbar span {
        font-size: 40px;
        color: #16b4bd;
        white-space:nowrap;
    }

    .left {
        position: absolute;
        left: 0;
    }

    .right {
        position: absolute;
        left: 800px;
    }
</style>

<body>
    <div class="navbar">
        <span class="left">左边定位</span>
        <span class="right">右边定位</span>
    </div>
</body>

</html>
```



运行代码可以看到,`div宽度是100%`,也就是`viewport`的宽度。



![](http://39.105.62.145/assets/images/WX20181121-125957@2x.png)



当我们缩小宽度的时候，并且向左边滚动一点，可以看到`div宽度100%`变成了`viewport`的宽度。而页面里的元素溢出了，溢出的元素仍然会显示。



![](http://39.105.62.145/assets/images/WX20181121-130455@2x.png)



总而言之：元素`width:100%;`与页面的宽度有关，与元素无关。



如何获取`viewport`的尺寸

```javascript
document.documentElement.clientWidth
document.documentElement.clientHeight
//document.documentElement实际上就是<html>元素
```



`document.documentElement.clientWidth`与`window.innerWidth`的区别

```javascript
window.innerWidth/Height包含滚动条

document. documentElement. clientWidth/Height不包含
```







###### 移动设备浏览器

------

1) visualviewport是当前显示在`屏幕上的部分页面`。用户会滚动页面来改变可见部分，或者缩放浏览器来改变visualviewport的尺寸。如图：

![](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2014/1404/viewport-27.jpg)





2) layoutviewport，CSS 布局通常是按照layoutviewport来定义，而比visualviewport宽很多，它是<html>元素的父容器。只要你不在css中修改`<html>`元素的宽度，`<html>`元素的宽度就会撑满layout viewport的宽度。（一句话来说就是：浏览器加载出了整个页面，出现了滚动条，滚动可以浏览页面其他部分，缩放和浏览器窗口改变都不影响）



![](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2014/1404/viewport-25.jpg)



3）Media查询

width/height 反映`document.documentElement.clientWidth/document.documentElement.Height`的值, device-width/height 反映`screen.width/screen.height`

![](http://www.w3cplus.com/sites/default/files/styles/print_image/public/blogs/2014/1404/viewport-31.jpg)



4) viewport的meta标签（在之后的内容详细说）

用于设置layoutviewport的宽度。





##### 3.设备像素比(设备像素比 ＝ 物理像素 / 设备独立像素)

**物理像素（设备像素）：平时所说的分辨率，` iphone6s 的分辨率是 750*1334(表示横向右750个像素点，纵向有1334个像素点)`**



![](http://39.105.62.145/assets/images/WX20181123-155710@2x.png)

**设备独立像素：一台设备上程序用来描绘数据的一个个的“点”，也就是虚拟的像素。以前设备像素对应一个设备独立像素，后来出现了高分辨率的手机，不可能再一个设备像素对应一个设备独立像素了，因为这样东西会被缩小，看不清。`iPhone6s的设备宽度和高度为375 * 667,可以理解为设备的独立像素。`**



举个🌰

```css
width:2px;
height:2px;
```

![](http://39.105.62.145/assets/images/WX20181123-162010@2x.png)

1倍屏上，1个css像素对应1个物理像素，在2倍屏上，1个css像素对应4个物理像素。





##### 4.CSS单位rem

`rem`就是相对于根元素`<html>`的`font-size`来做计算。



##### 5.meta标签

```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
```

```html
<meta name="flexible" content="initial-dpr=2" />
```

其中`initial-dpr`会把`dpr`强制设置为给定的值。如果手动设置了`dpr`之后，不管设备是多少的`dpr`，都会强制认为其`dpr`是你设置的值。在此不建议手动强制设置`dpr`。



##### 6.使用方法

 1) 引入cdn的库或者[下载](https://github.com/amfe/lib-flexible/archive/master.zip)

```html
<script src="http://g.tbcdn.cn/mtb/lib-flexible/0.3.4/??flexible_css.js,flexible.js"></script>
```



2）实质是通过JS动态改写meta标签

- 动态改写`<meta>`标签
- 给`<html>`元素添加`data-dpr`属性，并且动态改写`data-dpr`的值
- 给`<html>`元素添加`font-size`属性，并且动态改写`font-size`的值



3) 如何把设计稿的px转换为rem(使用css处理器)

  使用SCSS函数：

```scss
@function fn-pxtorem($px){
  @return $px/ 75 * 1rem;
}
```









##### 







































