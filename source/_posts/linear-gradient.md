---
title: linear-gradient
date: 2019-01-13 11:40:43
tags: 
- css
categories: 
- css
---

> 参考资料 https://www.w3cplus.com/css3/do-you-really-understand-css-linear-gradients.html

### 渐变背景，都是在chrome下调试，这个样式有兼容性问题，在不同浏览器有差异

#### 语法:

```css
<linear-gradient> = linear-gradient([ [ <angle> | to <side-or-corner> ] ,]? <color-stop>[, <color-stop>]+)

<side-or-corner> = [left | right] || [top | bottom]

<color-stop> = <color> [ <length> | <percentage> ]?
```

#### 值:

<angle>：用角度值指定渐变的方向

to left：设置渐变为从右到左。相当于: 270deg

to right：设置渐变从左到右。相当于: 90deg

to top：设置渐变从下到上。相当于: 0deg

to bottom：设置渐变从上到下。相当于: 180deg。这是`默认值`，等同于留空不写。

<color-stop>： 用于指定渐变的起止颜色：

<color>：指定颜色。

<length>：用长度值指定起止色位置。不允许负值

<percentage>：用百分比指定起止色位置。



#### 1）不同方向的渐变（`top`、`right`、`bottom`、`left`、`left top`、`top right`、`bottom right`或者`left bottom`）

```css
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<style>
    body {
        display: flex;
        flex-wrap: wrap;
    }

    div {
        width: 100px;
        height: 100px;
        margin:10px 10px 0 0;
    }

    .div1 {
        background-image: -webkit-linear-gradient(top, white, red);
    }

    .div2 {
        background-image: -webkit-linear-gradient(left, white, red);
    }

    .div3 {
        background-image: -webkit-linear-gradient(right, white, red);
    }

    .div4 {
        background-image: -webkit-linear-gradient(bottom, white, red);
    }

    .div5 {
        background-image: -webkit-linear-gradient(top right, white, red);
    }

    .div6 {
        background-image: -webkit-linear-gradient(left top, white, red);
    }

    .div7 {
        background-image: -webkit-linear-gradient(bottom right, white, red);
    }

    .div8 {
        background-image: -webkit-linear-gradient(left bottom, white, red);
    }
</style>

<body>
    <div class="div1"></div>
    <div class="div2"></div>
    <div class="div3"></div>
    <div class="div4"></div>
    <div class="div5"></div>
    <div class="div6"></div>
    <div class="div7"></div>
    <div class="div8"></div>
</body>

</html>
```

![WX20190113-124601@2x](http://118.24.241.76/WX20190113-124601@2x.png)



#### 2）起始颜色到结束颜色

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<style>
    body {
        display: flex;
        flex-wrap: wrap;
    }

    div {
        width: 100px;
        height: 100px;
        margin:10px 10px 0 0;
    }

    .div1 {
        background-image: -webkit-linear-gradient(top, white 50%, red 70%);
    }

    .div2 {
        background-image: -webkit-linear-gradient(left, white 50%, red 70%);
    }

    .div3 {
        background-image: -webkit-linear-gradient(right, white 50%, red 70%);
    }

    .div4 {
        background-image: -webkit-linear-gradient(bottom, white 50%, red 70%);
    }

    .div5 {
        background-image: -webkit-linear-gradient(top right, white 50%, red 70%);
    }

    .div6 {
        background-image: -webkit-linear-gradient(left top, white 50%, red 70%);
    }

    .div7 {
        background-image: -webkit-linear-gradient(bottom right, white 50%, red 70%);
    }

    .div8 {
        background-image: -webkit-linear-gradient(left bottom, white 50%, red 70%);
    }
</style>

<body>
    <div class="div1"></div>
    <div class="div2"></div>
    <div class="div3"></div>
    <div class="div4"></div>
    <div class="div5"></div>
    <div class="div6"></div>
    <div class="div7"></div>
    <div class="div8"></div>
</body>

</html>
```

![WX20190113-163003@2x](http://118.24.241.76/WX20190113-163003@2x.png)

起始位置到50%之间白色，50%到70%之间渐变，70%之后红色。



#### 3）实现时间轴效果

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<style>
    div {
        width: 300px;
        height: 400px;
        margin:10px 10px 0 0;
        border:1px solid red;
    }
    div{
        background-image:-webkit-linear-gradient(left,white 20%,#666 20%,#666 22%,white 22%);
    }
</style>
<body>
    <div>

    </div>
</body>
</html>
```

记得刚入职不久，为了写这条线，延后沿着它展开布局，费了不少劲。

![WX20190113-163610@2x](http://118.24.241.76/WX20190113-163610@2x.png)



#### 总结

考虑到浏览器兼容性，写下区别

```css
  /* Safari 5.1 - 6.0 */
  background: -webkit-linear-gradient(left,red,orange,yellow,green,blue,indigo,violet);
  /* Opera 11.1 - 12.0 */
  background: -o-linear-gradient(left,red,orange,yellow,green,blue,indigo,violet);
  /* Firefox 3.6 - 15 */
  background: -moz-linear-gradient(left,red,orange,yellow,green,blue,indigo,violet);
  /* 标准的语法 */
  background: linear-gradient(to right, red,orange,yellow,green,blue,indigo,violet); 
```

