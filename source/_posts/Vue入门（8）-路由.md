---
title: Vue入门（8）- 路由
date: 2019-08-28 13:02:43
tags: 
- Vue入门
categories: 
- Vue入门
---

## History api

```
方法：
back() 
forward()
go()

属性：
length
```

```javascript
//返回
window.history.back()
window.history.go(-1)

//向前跳转
window.history.forward()
window.history.go(1)

//历史记录中页面总数
history.length
```



**HTML5 新方法：添加和替换历史记录的条目**

**pushState**

```javascript
history.pushState(state, title, url); //添加一条历史记录，不刷新页面
```

`state`:一个对象或者字符串，用于描述新记录的一些特性。这个参数会被一并添加到历史记录中，以供以后使用。这个参数是开发者根据自己的需要自由给出的。

`title`:一个字符串，代表新页面的标题。当前基本上所有浏览器都会忽略这个参数。

`url`:一个字符串，代表新页面的相对地址。



假设当前页面为`/index.html`，那么执行下面的 JavaScript 语句：

```javascript
window.history.pushState({name:"qinhanwen"}, "My Profile", "/profile/");
```

之后，地址栏的地址就会变成`/profile/`，但同时浏览器不会刷新页面，甚至不会检测目标页面是否存在。



**replaceState**

```javascript
history.replaceState({name:"qinhanwen"}, "My Profile", "/profile/");
```

与`pushState`一致，不同的是不会往历史栈添加新记录，而是替换当前的浏览记录。



**popState事件**

当用户点击浏览器的「前进」、「后退」按钮时，就会触发`popstate`事件。你可以监听这一事件，从而作出反应。

```javascript
window.addEventListener("popstate", function (e) {
	console.log(e.state);
});
```

`e.state`里有刚才的参数



## Location

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <a href="#/index">index</a>
    <a href="#/about">about</a>
</body>
<script>
    window.addEventListener("hashchange", function (e) {
        console.log(location.hash);
    });
</script>

</html>
```



## Vue-router

引入插件js

```html
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>
```

