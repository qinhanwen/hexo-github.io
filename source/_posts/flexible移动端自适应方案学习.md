---
title: flexible移动端自适应方案学习
date: 2018-11-19 10:12:21
tags: 
- 优化
categories: 
- 优化
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

![](http://114.55.30.96/1542689394373.jpg) 



2) 浏览器尺寸（CSS的pixels）

​       从`window.innerWidth/window.innerHeight`获取，浏览器窗口的内部宽度使用CSS的pixels（意思就是我打开浏览器，能看到里面的`元素`会因为我`放大或者缩小`浏览器而`变少或者变多`），而缩放比例就是如图：

![](http://114.55.30.96/1542690089176.jpg)



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



![](http://114.55.30.96/WX20181121-125957@2x.png)



当我们缩小宽度的时候，并且向左边滚动一点，可以看到`div宽度100%`变成了`viewport`的宽度。而页面里的元素溢出了，溢出的元素仍然会显示。



![](http://114.55.30.96/WX20181121-130455@2x.png)



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



4) viewport的meta标签

可以用于设置layoutviewport的宽度。





##### 3.设备像素比(设备像素比 ＝ 物理像素 / 设备独立像素)

**物理像素（设备像素）：平时所说的分辨率，` iphone6s 的分辨率是 750*1334`**



![](http://114.55.30.96/WX20181123-155710@2x.png)

**设备独立像素：`iPhone6s的设备宽度和高度为375 * 667,可以理解为设备的独立像素`**

**一台设备上程序用来描绘数据的一个个的“点”，这个点代表一个可以由程序使用的虚拟像素(比如说CSS像素)。以前设备像素对应一个设备独立像素，后来出现了高分辨率的手机，不可能再一个设备像素对应一个设备独立像素了，因为这样东西会被缩小，看不清。这个点代表一个可以由程序使用的虚拟像素(比如说CSS像素)**



举个🌰

1倍屏：高宽1px的图片，是1个设备像素

 ![](http://114.55.30.96/WX20181228-220032@2x.png)

2倍屏：高宽1px的方格，是4个设备像素，因为如果还是用一个设备像素就太小了

​	![](http://114.55.30.96/WX20181228-230615@2x.png)





##### 4.CSS单位rem

`rem`就是相对于根元素`<html>`的`font-size`来做计算。

而`em`是根据父元素的`fon-size`计算



Flexible会将视觉稿分成**100份**，而每一份被称为一个单位`a`。同时`1rem`单位被认定为`10a`，我就直接理解为设计稿由`10rem`组成

![WX20181229-012813@2x](http://114.55.30.96/WX20181229-012813@2x.png)

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





##### 7.实践出真知，来吧

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <script src="http://g.tbcdn.cn/mtb/lib-flexible/0.3.4/??flexible_css.js,flexible.js"></script>
</head>
<style>
    * {
        margin: 0;
        padding: 0;
    }
    p{
        width:10rem;
        background: red;
        text-align: center;
    }
</style>

<body>
  <p>
      flexible学习笔记
  </p>
</body>

</html>
```



###### 1）安卓设备：动态添加了meta标签，初始缩放比为1，根元素添加data-dpr为1，font-size:36px，

![](http://114.55.30.96/WX20181229-011051@2x.png)



###### 2）2倍屏：动态添加meta，设置缩放比为0.5，设置根元素的data-dpr=2，font-size:75px;

![](http://114.55.30.96/WX20181229-011248@2x.png)



###### 3）3倍屏幕：动态添加meta，设置缩放比为0.333(很多个3)，设置根元素的data-dpr=3，font-size:112.5px;

![WX20181229-011651@2x](http://114.55.30.96/WX20181229-011651@2x.png)





这里有个问题，如果我在头部加了这句

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

那么，就不会动态的添加meta，并且提示

![WX20181229-012037@2x](http://114.55.30.96/WX20181229-012037@2x.png)



##### 8.具体实现（不要说， 来看[源码](https://github.com/amfe/lib-flexible/blob/master/src/flexible.js)吧）

```javascript
(function(win, lib) {
    var doc = win.document;
    var docEl = doc.documentElement;//获取HTML根元素
    var metaEl = doc.querySelector('meta[name="viewport"]');//选中meta，属性是name="viewport"
    var flexibleEl = doc.querySelector('meta[name="flexible"]');//选中meta，属性是name="flexible"
    var dpr = 0;//设置dpr初始值
    var scale = 0;//设置scale初始值
    var tid;
    var flexible = lib.flexible || (lib.flexible = {});
    
    if (metaEl) {//如果存在就是用已有的meta头部
        console.warn('将根据已有的meta标签来设置缩放比例');
        var match = metaEl.getAttribute('content').match(/initial\-scale=([\d\.]+)/);//获取meta的content属性，是否有设置初始缩放比
        if (match) {//如果设置了，就取初始缩放比和dpr，
            scale = parseFloat(match[1]);
            dpr = parseInt(1 / scale);
        }
    } else if (flexibleEl) {//如果有设置flexibleEl，而且有设置缩放比，就是用设置的缩放比
        var content = flexibleEl.getAttribute('content');
        if (content) {
            var initialDpr = content.match(/initial\-dpr=([\d\.]+)/);
            var maximumDpr = content.match(/maximum\-dpr=([\d\.]+)/);
            if (initialDpr) {
                dpr = parseFloat(initialDpr[1]);
                scale = parseFloat((1 / dpr).toFixed(2));    
            }
            if (maximumDpr) {
                dpr = parseFloat(maximumDpr[1]);
                scale = parseFloat((1 / dpr).toFixed(2));    
            }
        }
    }

    if (!dpr && !scale) {//如果上面两个都没有的话，就是没有设置属性name=viewport，和name=flexible的meta标签的话，dpr和scale就是0，
        //判断平台，取得设备像素比，如果是iphone，就是根据设备像素比设置dpr，否则一律设置为1，缩放比为1除dpr。
        var isAndroid = win.navigator.appVersion.match(/android/gi);
        var isIPhone = win.navigator.appVersion.match(/iphone/gi);
        var devicePixelRatio = win.devicePixelRatio;
        if (isIPhone) {
            if (devicePixelRatio >= 3 && (!dpr || dpr >= 3)) {                
                dpr = 3;
            } else if (devicePixelRatio >= 2 && (!dpr || dpr >= 2)){
                dpr = 2;
            } else {
                dpr = 1;
            }
        } else {
            dpr = 1;
        }
        scale = 1 / dpr;
    }

    docEl.setAttribute('data-dpr', dpr);//给根元素动态添加data-dpr属性，值为dpr
    if (!metaEl) {//如果没有已有的meta头部，创建meta标签，name属性是viewport，content里面的初始缩放比，最大、最小缩放比为scale，不允许缩放
        metaEl = doc.createElement('meta');
        metaEl.setAttribute('name', 'viewport');
        metaEl.setAttribute('content', 'initial-scale=' + scale + ', maximum-scale=' + scale + ', minimum-scale=' + scale + ', user-scalable=no');
        if (docEl.firstElementChild) {//如果有第一个子元素，就插入meta标签
            docEl.firstElementChild.appendChild(metaEl);
        } else {//否则创建一个div，在div内插入meta标签，然后在文档流中写入div的内容
            var wrap = doc.createElement('div');
            wrap.appendChild(metaEl);
            doc.write(wrap.innerHTML);
        }
    }

    function refreshRem(){
        var width = docEl.getBoundingClientRect().width;//获取根元素的宽
        if (width / dpr > 540) {//如果逻辑像素大于540，就重写width
            width = 540 * dpr;
        }
        var rem = width / 10;//声明变量值为width/10，这里的值就是等下设置根元素font-size的值
        docEl.style.fontSize = rem + 'px';//设置根元素的font-size为多少px
        flexible.rem = win.rem = rem;//设置flexible对象的rem属性值为多少，window对象的rem属性值为多少
    }

    win.addEventListener('resize', function() {//监听页面resize事件，窗体的状态改变时将触发Resize事件
        clearTimeout(tid);
        tid = setTimeout(refreshRem, 300);//设置定时器
    }, false);
    win.addEventListener('pageshow', function(e) {//在chrome下事件触发顺序ready→load→pageshow，在页面加载，或者后退前进的时候触发。
        if (e.persisted) {//是否引用缓存
            clearTimeout(tid);
            tid = setTimeout(refreshRem, 300);
        }
    }, false);

    if (doc.readyState === 'complete') {//文档载入完成
        doc.body.style.fontSize = 12 * dpr + 'px';//设置body的font-size
    } else {
        doc.addEventListener('DOMContentLoaded', function(e) {//监听DOM内容加载完毕
            doc.body.style.fontSize = 12 * dpr + 'px';//设置body的font-size
        }, false);
    }
    
    refreshRem();

    flexible.dpr = win.dpr = dpr;
    flexible.refreshRem = refreshRem;
    flexible.rem2px = function(d) {
        var val = parseFloat(d) * this.rem;
        if (typeof d === 'string' && d.match(/rem$/)) {
            val += 'px';
        }
        return val;
    }
    flexible.px2rem = function(d) {
        var val = parseFloat(d) / this.rem;
        if (typeof d === 'string' && d.match(/px$/)) {
            val += 'rem';
        }
        return val;
    }

})(window, window['lib'] || (window['lib'] = {}));
```



###### 













































