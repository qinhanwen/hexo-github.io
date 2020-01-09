---
title: BFC
date: 2018-11-23 18:09:08
tags: 
- css
categories: 
- css
---

# BFC学习笔记

#### 1.Block Formatting Contexts(格式化上下文)

它是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系和相互作用。



#### 2.触发 BFC

- 浮动元素：float 除 none 以外的值

- 绝对定位元素：position (absolute、fixed)

- display 为 inline-block、table-cells、flex

- overflow 除了 visible 以外的值 (hidden、auto、scroll)


#### 3.BFC解决的问题

##### 1) 同一个 BFC 下外边距会发生折叠

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
    div {
        width: 100px;
        height: 100px;
        background: lightblue;
        margin: 100px;
    }
</style>

<body>
    <div></div>
    <div></div>
</body>

</html>
```

![](http://118.24.241.76/WX20181209-233016@2x.png)



**解决方案**：避免边框重叠，将元素放置在新的BFC容器内

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
    div {
        width: 100px;
        height: 100px;
        background: lightblue;
        margin: 100px;
    }
    section{
        overflow: hidden;
    }
</style>

<body>
    <section>
            <div></div>
    </section>
    <div></div>
</body>

</html>
```

一般使用`overflow: hidden`因为这个属性对布局的影响比较小。

![](http://118.24.241.76/WX20181209-233154@2x.png)



##### 2) 高度塌陷

float导致元素脱离了普通文档流

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
    section {
      background: red;
      padding: 20px;
    }
    div{
        float: left;
        width: 200px;
        height: 200px;
        background: blue;
    }
</style>

<body>
   <section>
       <div></div>
   </section>
</body>

</html>
```

![](http://118.24.241.76/WX20181209-233402@2x.png)

**解决方案**：触发容器的BFC，就会让容器包含浮动元素

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
    section {
      background: red;
      padding: 20px;
      overflow: hidden;
    }
    div{
        float: left;
        width: 200px;
        height: 200px;
        background: blue;
    }
</style>

<body>
   <section>
       <div></div>
   </section>
</body>

</html>
```



##### 3)  元素被浮动元素覆盖

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
    div:first-child{
        width: 200px;
        height: 200px;
        background: blue;
        float: left;
    }
    div:last-child{
        width: 400px;
        height: 400px;
        background: red;
    }
</style>

<body>
 <section>
     <div>

     </div>
     <div>

     </div>
 </section>
</body>

</html>
```



![](http://118.24.241.76/WX20181209-235322@2x.png)

**解决方案**：触发元素的BFC

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
    div:first-child{
        width: 200px;
        height: 200px;
        background: blue;
        float: left;
    }
    div:last-child{
        width: 400px;
        height: 400px;
        background: red;
        overflow: hidden;
    }
</style>

<body>
 <section>
     <div>

     </div>
     <div>

     </div>
 </section>
</body>

</html>
```



![](http://118.24.241.76/WX20181209-235310@2x.png)

#### 4.两列布局与三列布局的应用

##### 1）两列布局（左边部分宽度限定，右边自适应）

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
div:first-child{
 width: 200px;
 height: 200px;
 float: left;
 background: red;
}
div:last-child{
height: 200px;
overflow: hidden;
background: blue;
}
</style>
<body>
    <section>
            <div></div>
            <div></div>
    </section>
</body>
</html>
```



![](http://118.24.241.76/WX20181209-235810@2x.png)

##### 2) 三列布局（左和右列宽度固定，中间自适应）

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
    .first {
        width: 200px;
        height: 200px;
        float: left;
        background: blue;
    }

    .second {
        width: 200px;
        height: 200px;
        float: right;
        background: blue;
    }

    .third {
        height: 300px;
        overflow: hidden;
        background: red;
    }
</style>

<body>
    <section>
        <div class="first"></div>
        <div class="second"></div>
        <div class="third"></div>
    </section>
</body>

</html>
```



![](http://118.24.241.76/WX20181210-001041@2x.png)

需要注意的地方就是，浮动的元素要写在前面。







> 参考资料：https://blog.csdn.net/jiaojsun/article/details/76408215