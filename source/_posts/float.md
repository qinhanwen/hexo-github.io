---
title: float
date: 2018-11-26 17:14:12
tags: 
- css
categories: 
- css
---

# float属性学习笔记

##### 1.float语法

```css
float:none 不使用浮动
float:left 靠左浮动
float:right 靠右浮动
float:inherit 继承
```



##### 2.清除浮动的几种方法

###### 1）clear:both清除浮动

```html
	<div class="divcss5">
        <div class="divcss5_left">布局靠左浮动</div>
        <div class="divcss5_right">布局靠右浮动</div>
        <div class="clear"></div><!-- html注释：清除float产生浮动 -->
    </div>
```

```css
  .divcss5 {
        width: 400px;
        padding: 10px;
        border: 1px solid #F00
    }

    .divcss5_left {
        float: left;
        width: 150px;
        border: 1px solid #00F;
        height: 50px
    }

    .divcss5_right {
        float: right;
        width: 150px;
        border: 1px solid #00F;
        height: 50px
    }

    .clear {
        clear: both
    }
```



###### 2）父级div定义 overflow:hidden

```html
	<div class="divcss5">
        <div class="divcss5_left">布局靠左浮动</div>
        <div class="divcss5_right">布局靠右浮动</div>
    </div>
```

```css
  .divcss5 {
        width: 400px;
        padding: 10px;
        border: 1px solid #F00;
        overflow:hidden;
    }

    .divcss5_left {
        float: left;
        width: 150px;
        border: 1px solid #00F;
        height: 50px
    }

    .divcss5_right {
        float: right;
        width: 150px;
        border: 1px solid #00F;
        height: 50px
    }
```



###### 3）设置父级高度



##### 3.文字环绕问题

```css
  .outer {
        border: 5px dotted rgb(214, 129, 137);
        border-radius: 5px;
        width: 450px;
        padding: 10px;
        margin-bottom: 40px;
    }

    .float {
        padding: 10px;
        border: 5px solid rgba(214, 129, 137, .4);
        border-radius: 5px;
        background-color: rgba(233, 78, 119, .4);
        color: #fff;
        float: left;
        width: 200px;
        margin: 0 20px 0 0;
    }
```

```html
    <div class="outer">
        <div class="float">I am a floated element.</div> I am text inside the outer box.
    </div>
```



![](http://39.108.238.15:97/static/images/images/WX20181126-211600@2x.png)

顺带一提float和绝对定位的区别：



1）绝对定位是将元素彻底从文档流删除，元素原先在正常文档流中所占的空间会关闭，就好像该元素原来不存在一样，该元素再也不会影响其他元素的布局了。

```css
.a{
    width: 200px;
    height: 200px;
    background: red;
}
.b{
    width: 100px;
    height: 100px;
    background: blue;
    position: absolute;
    left: 0;
    top: 0;
}
```

```html
<div class="b">这是定位内容</div>
<div class="a">这是一大段内容这是一大段内容这是一大段内容这是一大段内容这是一大段内容这是一大段内容这是一大段内容这是一大段内容这是一大段内容这是一大段内容这是一大段内容这是一大段内容这是一大段内容这是一大段内容这是一大段内容</div>
```

效果图：

![](http://39.108.238.15:97/static/images/images/WX20181126-221025@2x.png)



2）浮动会以某种方式将元素脱离正常的文档流，但是依然占据正常文档流的文本空间。

```css
.a{
    width: 200px;
    height: 200px;
    background: red;
}
.b{
    width: 100px;
    height: 100px;
    background: blue;
    float:left;
}
```

 效果图：

![](http://39.108.238.15:97/static/images/images/WX20181126-222436@2x.png)



关于[BFC](https://qinhanwen.github.io/2018/11/23/BFC/)



