---
title: Flex布局
date: 2018-12-05 10:27:01
tags: 
- css
categories: 
- css
---

# Flex布局

> 参考资料：http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html



#### 1.弹性布局（flexible box）

任何一个容器都可以变成Flex布局

###### 1）块级元素

```css
.box{
    display:flex;
}
```



###### 2）行内元素也可以

```
.box{
    display:inline-flex;
}
```



强大的东西一般都有兼容性问题，Webkit 内核的浏览器，需要加上-webkit。之前碰到ios9还是10以下，没有写导致样式问题。

```css
.box{
  display: -webkit-flex; /* Safari */
  display: flex;
}
```



使用Flex布局之后，子元素`float`和`vertical-align`就不生效了。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <div style="display:flex">
     <span style="float:right">span</span><em style="vertical-align: middle;">em</em>
    </div>
</body>
</html>
```

可以看到浮动和垂直对齐方式，都失效了。



#### 2.容器的属性

##### 1）`flex-direction`属性决定项目排列顺序



属性值：

###### 1.`row`（默认值）：主轴为水平方向，起点在左端。

###### 2.`row-reverse`：主轴为水平方向，起点在右端。

###### 3.`column`：主轴为垂直方向，起点在上沿。

###### 4.`column-reverse`：主轴为垂直方向，起点在下沿。

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
    ul {
        display: flex;
        list-style: none;
    }

    li {
        width: 50px;
        height: 50px;
        line-height: 50px;
        text-align: center;
        background: red;
    }
</style>

<body>
    <ul style="flex-direction:row;">
        <!--flex-direction:row;-->
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
    <ul style="flex-direction:row-reverse;">
        <!--flex-direction:row-reverse;-->
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
    <ul style="flex-direction:column;">
        <!--flex-direction:column;-->
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
    <ul style="flex-direction:column-reverse;">
        <!--flex-direction:column-reverse;-->
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
</body>

</html>
```

如图：

![](http://39.108.238.15:97/static/images/images/WX20181211-192019@2x.png)

###### 5.`initial`: 设置为默认值

###### 6.`inherit`: 从父级继承该属性

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
    ul {
        display: flex;
        list-style: none;
        padding: 0;
    }

    li {
        width: 200px;
        height: 50px;
        line-height: 50px;
        text-align: center;
        background: red;
    }

    span {
        color: blue;
    }
</style>

<body>
    <div>
        <ul style="flex-direction:initial;">
            <!--flex-direction:initial;-->
            <li>1</li>
            <li>2</li>
            <li>3</li>
        </ul>
    </div>

    <div>
        <ul style="flex-direction:row-reverse;">
            <li style="display:flex;flex-direction: inherit">
                <!--flex-direction:inherit;-->
                <span>span1</span>
                <span>span2</span>
                <span>span3</span>
            </li>
            <li>2</li>
            <li>3</li>
        </ul>
    </div>
</body>

</html>
```

如图：

![](http://39.108.238.15:97/static/images/images/WX20181211-193656@2x.png)

##### 2）`flex-wrap`属性决定项目排不下如何换行



属性值：

###### 1.`nowrap`（默认值）：不换行。

###### 2.`wrap`   :  换行。

###### 3.`wrap-reverse`  :   第一行在下方。

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
    ul {
        display: flex;
        list-style: none;
        padding: 0;
    }

    li {
        width: 100px;
        height: 100px;
        line-height: 100px;
        text-align: center;
        background: red;
        border: 1px solid #ddd;
    }
</style>

<body>
    <ul style="flex-wrap: nowrap">
        <!-- flex-wrap: nowrap -->
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>

    <ul style="flex-wrap: wrap">
        <!-- flex-wrap: wrap -->
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>

    <ul style="flex-wrap:wrap-reverse">
        <!-- flex-wrap: wrap-reverse -->
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>
</body>

</html>
```

如图：

![](http://39.108.238.15:97/static/images/images/WX20181211-200324@2x.png)

###### 4.`initial`: 设置为默认值

###### 5.`inherit`: 从父级继承该属性

##### 



##### 3） `flex-flow`是`flex-direction`属性和`flex-wrap`属性的简写形式，默认值为`row nowrap`

```html
.box {
  flex-flow: <flex-direction> || <flex-wrap>;
}
```

`flex-direction`与`flex-wrap`先后顺序不重要。



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
    ul {
        display: flex;
        list-style: none;
        padding: 0;
    }

    li {
        width: 100px;
        height: 100px;
        line-height: 100px;
        text-align: center;
        background: red;
        border: 1px solid #ddd;
    }
</style>

<body>
    <ul style="flex-flow:row-reverse wrap-reverse">
        <!-- flex-flow:row-reverse wrap-reverse -->
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>
</body>

</html>
```
如图：

![](http://39.108.238.15:97/static/images/images/WX20181211-223712@2x.png)



##### 4）`justify-content`属性是水平的主轴上项目的对齐方式

属性值：

###### 1）`flex-start `  (默认值)  ：左对齐

###### 2）`flex-end`   :   右对齐

###### 3）`center` 	 :  居中

###### 4）`space-between`   :  两端对齐，项目之间的间隔都相等

###### 5）`space-around`   :   每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。



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
    body{
        margin: 0;
        padding: 0;
    }
    ul {
        display: flex;
        list-style: none;
        padding: 0;
    }

    li {
        width: 100px;
        height: 100px;
        line-height: 100px;
        text-align: center;
        background: red;
        border: 1px solid #ddd;
    }
</style>

<body>
    <ul style="justify-content: flex-start;">
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>

    <ul style="justify-content: flex-end;">
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>


    <ul style="justify-content:center;">
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>

    <ul style="justify-content:space-between;">
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>

    <ul style="justify-content:space-around;">
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>
</body>

</html>
```



![](http://39.108.238.15:97/static/images/images/WX20181211-235114@2x.png)





##### 5）`align-items`属性是垂直的交叉轴上项目的对齐方式

属性值：

###### 1.`flex-start`：交叉轴的起点对齐。

###### 2.`flex-end`：交叉轴的终点对齐。

###### 3.`center`：交叉轴的中点对齐。

###### 4.`baseline`: 项目的第一行文字的基线对齐。

###### 5.`stretch`（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。



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
    body {
        margin: 0;
        padding: 0;
    }

    ul {
        display: flex;
        list-style: none;
        padding: 0;
        width: 150px;
        height: 150px;
        border:1px solid black;
    }

    li {
        width: 30px;
        line-height: 30px;
        text-align: center;
        background: red;
        border: 1px solid #ddd;
    }
    /*不设置li的高度*/
</style>

<body>
    <ul style="align-items: flex-start">
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>

    <ul style="align-items: flex-end">
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>

    <ul style="align-items: center">
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>

    <ul style="align-items: baseline">
        <li>li first</li>
        <li  style="margin-top:20px">li</li>
        <li>li</li>
        <li>li last</li>
    </ul>

    <ul >
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>
</body>

</html>
```



![](http://39.108.238.15:97/static/images/images/WX20181212-125359@2x.png)

##### 6）`align-content`属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用（就是项目有多行的时候起作用）。

举个🌰，比较一下`align-content`和`align-items`的区别：

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
    * {
        margin: 0;
        padding: 0;
    }

    ul {
        display: flex;
        list-style: none;
        padding: 0;
        width: 150px;
        height: 150px;
        border:1px solid black;
    }

    li {
        width: 30px;
        height: 30px;
        line-height: 30px;
        text-align: center;
        background: red;
        border: 1px solid #ddd;
    }
</style>

<body>
    <ul style="align-items: center">
        <li>1</li>
        <li>2</li>
    </ul>

    <ul style="align-content: center">
        <li>1</li>
        <li>2</li>
    </ul>
</body>

</html>
```

第一个ul使用的`align-items: center`,可以看到它生效了。

第二个ul使用的`align-content: center`，未生效。



![ ](http://39.108.238.15:97/static/images/images/WX20181212-235018@2x.png)

使第二个ul生效的方法：使 li 换行，变成多行。

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
    * {
        margin: 0;
        padding: 0;
    }

    ul {
        display: flex;
        list-style: none;
        padding: 0;
        width: 150px;
        height: 150px;
        border:1px solid black;
    }

    li {
        width: 30px;
        height: 30px;
        line-height: 30px;
        text-align: center;
        background: red;
        border: 1px solid #ddd;
    }
</style>

<body>
    <ul style="align-items: center">
        <li>1</li>
        <li>2</li>
    </ul>

    <ul style="align-content: center;flex-wrap: wrap">
        <!--设置右外边距，超出换行-->
        <li style="margin-right: 120px">1</li>
        <li>2</li>
    </ul>
</body>

</html>
```

可以看到换行以后，居中了。

![](http://39.108.238.15:97/static/images/images/WX20181212-235458@2x.png)

属性值：

###### 1.`flex-start`：与交叉轴的起点对齐。

###### 2.`flex-end`：与交叉轴的终点对齐。

###### 3.`center`：与交叉轴的中点对齐。

###### 4.`space-between`：与交叉轴两端对齐，轴线之间的间隔平均分布。

###### 5.`space-around`：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。

###### 6.`stretch`（默认值）：轴线占满整个交叉轴。



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
    body {
        margin: 0;
        padding: 0;
    }

    ul {
        display: flex;
        list-style: none;
        padding: 0;
        width: 150px;
        height: 150px;
        border: 1px solid black;
    }

    li {
        width: 30px;
        line-height: 30px;
        text-align: center;
        background: red;
        border: 1px solid #ddd;
    }
</style>

<body>
    <ul style="flex-wrap: wrap;align-content: flex-start">
        <!-- align-content: flex-start -->
        <li>first</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>last</li>
    </ul>

    <ul style="flex-wrap: wrap;align-content: flex-end">
        <!-- align-content: flex-end -->
        <li>first</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>last</li>
    </ul>

    <ul style="flex-wrap: wrap;align-content: center">
        <!-- align-content: center -->
        <li>first</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>last</li>
    </ul>

    <ul style="flex-wrap: wrap;align-content: space-between">
        <!-- align-content: space-between -->
        <li>first</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>last</li>
    </ul>

    <ul style="flex-wrap: wrap;align-content: space-around">
        <!-- align-content: space-around -->
        <li>first</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>last</li>
    </ul>

    <ul style="flex-wrap: wrap;align-content: stretch">
        <!-- align-content: stretch -->
        <li>first</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>last</li>
    </ul>
</body>

</html>
```



![](http://39.108.238.15:97/static/images/images/WX20181213-234041@2x.png)

#### 3.项目的属性

##### 1）`order`属性是项目排列的顺序，数值越小越靠前（默认0）

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
    ul {
        display: flex;
        list-style: none;
        padding: 0;
        width: 300px;
        height: 300px;
        border:1px solid black;
    }

    li {
        width: 90px;
        height: 90px;
        line-height: 90px;
        text-align: center;
        background: red;
    }
</style>

<body>
    <ul>
        <li style="order: -1">order -1</li>
        <li style="order: 0">order 0</li>
        <li style="order: 1">order 1</li>
    </ul>
</body>

</html>
```



![](http://39.108.238.15:97/static/images/images/WX20181214-002033@2x.png)



##### 2）flex-grow属性是项目的放大比例（默认是0）

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
    ul {
        display: flex;
        list-style: none;
        padding: 0;
        width: 400px;
        height: 200px;
        border: 1px solid black;
    }

    li {
        width: 50px;
        height: 50px;
        line-height: 50px;
        text-align: center;
        background: red;
        border: 1px solid #eee;
    }
</style>

<body>
    <ul>
        <li>1</li>
        <li>1</li>
        <li>1</li>
    </ul>
    <ul>
        <li style="flex-grow: 1">flex-grow: 1</li>
        <li style="flex-grow: 1">flex-grow: 1</li>
        <li style="flex-grow: 1">flex-grow: 1</li>
    </ul>

    <ul>
        <li style="flex-grow: 1">flex-grow: 1</li>
        <li style="flex-grow: 2">flex-grow: 2</li>
        <li style="flex-grow: 1">flex-grow: 1</li>
    </ul>

</body>

</html>
```



![](http://39.108.238.15:97/static/images/images/WX20181214-233620@2x.png)

`li`元素的宽为50px，当`li`设置`flex-grow`属性值为1的时候，他们会平分并且占满剩下的空间。

当某个`li`设置`flex-grow`属性值为 2，其他为 1 的时候，那这个设置为 2 的项目占有剩下空间会是其他为 1 的两倍。

默认为0，就是说不放大。





##### 3）`flex-shrink`属性是项目的缩小比例(默认1)

空间不足又不换行的时候，项目会被挤压

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
    ul {
        display: flex;
        list-style: none;
        padding: 0;
    }

    li {
        width: 200px;
        height: 100px;
        line-height: 100px;
        text-align: center;
        background: red;
        border: 1px solid #ddd;
    }
</style>

<body>
    <ul>
        <li>li first</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>

    <ul>
        <li>li first</li>
        <li style="flex-shrink:0">li flex-shrink:0</li>
        <li>li</li>
        <li>li</li>
        <li>li</li>
        <li>li last</li>
    </ul>
</body>

</html>
```



![](http://39.108.238.15:97/static/images/images/WX20181214-235104@2x.png)

当属性值设置为 0 的时候，就不会被挤压。



##### 4）`flex-basis`属性是设置宽高，类似width和height

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
    ul {
        display: flex;
        width: 1000px;
        list-style: none;
        padding: 0;
        border: 1px solid black;
    }

    li {
        width: 200px;
        height: 100px;
        line-height: 100px;
        text-align: center;
        background: red;
        border: 1px solid #ddd;
    }
</style>

<body>
    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
        <li>3</li>
    </ul>

    <ul>
        <li style="flex-basis:250px">flex-basis:250px</li>
        <li style="flex-basis:250px">flex-basis:250px</li>
        <li style="flex-basis:250px">flex-basis:250px</li>
        <li style="flex-basis:250px">flex-basis:250px</li>
    </ul>

    <ul>
        <li style="flex-basis:250px">flex-basis:250px</li>
        <li style="flex-basis:250px">flex-basis:250px</li>
        <li style="flex-basis:250px">flex-basis:250px</li>
        <li style="flex-basis:250px">flex-basis:250px</li>
        <li style="flex-basis:250px">flex-basis:250px</li>
        <li style="flex-basis:250px">flex-basis:250px</li>
    </ul>

    <ul>
        <li style="flex-basis:250px;max-width: 180px;">max-width: 180px</li>
        <li style="flex-basis:250px;max-width: 180px;">max-width: 180px</li>
        <li style="flex-basis:250px;max-width: 180px;">max-width: 180px</li>
        <li style="flex-basis:250px;max-width: 180px;">max-width: 180px</li>
    </ul>
</body>

</html>
```



![](http://39.108.238.15:97/static/images/images/WX20181218-102851@2x.png)

可以看到优先级 max-width > flex-basis > width

并且项目太多的时候会被压缩。



##### 5）`flex`属性是`flex-grow`, `flex-shrink` 和 `flex-basis`的简写，默认值为`0 1 auto`

##### 6）align-self属性允许项目与其他有不同的对齐方式，会覆盖`align-item`属性

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
    ul {
        display: flex;
        width: 1000px;
        height: 500px;
        list-style: none;
        padding: 0;
        border: 1px solid black;
    }

    li {
        width: 300px;
        height: 300px;
        line-height: 300px;
        text-align: center;
        background: red;
        border: 1px solid #ddd;
    }
</style>
<body>
    <ul>
        <li>1</li>
        <li style="align-self: flex-end">2 lign-self: flex-end</li>
        <li>3</li>
    </ul>
</body>
</html>
```



![](http://39.108.238.15:97/static/images/images/WX20181218-110113@2x.png)