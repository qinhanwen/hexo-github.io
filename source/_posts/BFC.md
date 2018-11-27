---
title: BFC
date: 2018-11-23 18:09:08
tags: 
- css
categories: 
- css
---

# BFC学习笔记

##### 1.先写几个🌰

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

效果图：

![](http://pi8irywwe.bkt.clouddn.com/WX20181126-211600@2x.png)















> 参考资料：https://www.w3cplus.com/css/understanding-css-layout-block-formatting-context.html