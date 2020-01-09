---
title: box-sizing属性
date: 2018-11-23 19:06:47
tags: 
- css
categories: 
- css
---

> 参考资料：https://www.cnblogs.com/ylliap/p/6119740.html



#### CSS3 box-sizing属性

##### 1.盒子模型

现在所有浏览器都遵循W3C盒子模型，IE6以下遵循IE盒子模型。

###### 1）IE盒子模型

![](http://118.24.241.76/869623-20161130211301599-865892975.png)

`width = border + padding + content`



###### 2）W3C盒子模型

![](http://118.24.241.76/869623-20161130211303568-1731032408.png)

`width = content`



##### 2.box-sizing的三个属性值

###### 1）content-box

这个属性值，是box-sizing的默认值，遵循`W3C盒子模型`，看看他的表现

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
    <div style="width: 200px;height: 200px;padding: 10px;border:10px solid blue;background: red">

    </div>
</body>

</html>
```

![](http://118.24.241.76/WX20181210-212255@2x.png)

content是200



###### 2）border-box

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
    <div style="width: 200px;height: 200px;padding: 10px;border:10px solid blue;background: red;box-sizing: border-box">

    </div>
</body>

</html>
```

![](http://118.24.241.76/WX20181210-223259@2x.png)

content变成了160



###### 3）inherit

inherit属性值的时候，值从父级继承。





