---
title: 移动端1px问题
date: 2019-03-04 12:40:01
tags: 
- 疑难杂症
categories: 
- 疑难杂症
---

#### 关于移动端1px在某些机型上变粗的问题



### 原因

在我们写`border-bottom:1px solid red;`的时候，在有些设备下看起来会变粗

**物理像素**：出厂就设定好的了

**逻辑像素**：就是css中写的像素

当我们在css中写了1px是逻辑像素，它与物理像素有个比例，通过`window.devicePixelRatio`获取（也就是DPR），1px逻辑像素因为DPR不同，所以会被放大。

而在我平时用的[flexible适配方案](hhttps://qinhanwen.github.io/2018/11/19/flexible%E7%A7%BB%E5%8A%A8%E7%AB%AF%E8%87%AA%E9%80%82%E5%BA%94%E6%96%B9%E6%A1%88%E5%AD%A6%E4%B9%A0/)，它会根据设备的DPR，在meta标签里设置不同的缩放比（前提是没有设置name为viewport的meta标签，才会使用flexible.js新增的meta标签）。所以在IOS上没有这个问题，而在安卓下是有的。





#### 解决方案

##### 1）用小数点写px

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="./test.css">
    <title>Document</title>
</head>
<style>
    div {
        width: 200px;
        height: 200px;
        border: 1px solid #999;
    }
    @media screen and (-webkit-min-device-pixel-ratio: 2) {
        div {
            border: 0.5px solid #999;
        }
    }

    @media screen and (-webkit-min-device-pixel-ratio: 3) {
        div {
            border: 0.333333px solid #999;
        }
    }
</style>

<body>
    <div></div>
</body>
</html>
```



DPR为2：

![WX20190304-175100@2x](http://114.55.30.96/WX20190304-175100@2x.png)



DPR为3：

![WX20190304-175222@2x](http://114.55.30.96/WX20190304-175222@2x.png)



缺点：兼容性问题





##### 2）伪元素+transform

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="./test.css">
    <title>Document</title>
</head>
<style>
    div {
        width: 200px;
        height: 200px;
        position: relative;
    }

    div::after {
        content: "";
        width: 100%;
        height: 1px;
        background: #999;
        position: absolute;
        bottom: 0;
    }

    @media screen and (-webkit-min-device-pixel-ratio: 2) {
        div::after {
            width: 200%;
            transform: scale(.5);
            -webkit-transform: scale(.5);
            transform-origin: 0 0;
            -webkit-transform-origin: 0 0;
        }
    }

    @media screen and (-webkit-min-device-pixel-ratio: 3) {
        div::after {
            width: 300%;
            transform: scale(.33333);
            -webkit-transform: scale(.33333);
            transform-origin: 0 0;
            -webkit-transform-origin: 0 0;
        }
    }
</style>

<body>
    <div></div>
</body>

</html>
```

DPR为2：

![WX20190304-233004@2x](http://114.55.30.96/WX20190304-233004@2x.png)



DPR为3：

![WX20190304-233037@2x](http://114.55.30.96/WX20190304-233037@2x.png)



缺点：占用了伪元素





##### 3）flexible适配方案

不过只处理了IOS，安卓的自定义DPR都设置为1



##### 4）线性渐变

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="./test.css">
    <title>Document</title>
</head>
<style>
    h2 {
        width: 200px;
        height: 200px;
    }
    div{
        height: 1px;
        width: 200px;
        background: -webkit-linear-gradient( bottom,#fff,#999);
    }
</style>

<body>
    <h2>h2标签</h2>
    <div></div>
    
</body>

</html>
```













