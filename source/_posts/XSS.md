---
title: XSS简单了解
date: 2019-03-18 22:37:28
tags: 
- web安全
categories: 
- web安全
---

> 参考资料 	<https://juejin.im/post/5bad9140e51d450e935c6d64>
>
> ​			<https://juejin.im/post/5b97ba596fb9a05cda774ba4>
>
>  ​		        [练习](<http://prompt.ml/0>)
>
> ​			<https://excess-xss.com/>
>
> ​			[中文](https://segmentfault.com/a/1190000006904327)

​			

​	

## XSS是什么

XSS（Cross Site Scripting）跨站脚本攻击，通过在目标网站注入恶性脚本，使之再浏览器上运行，危害数据安全。只要有输入数据的地方，就可能存在XSS漏洞。



##### 那么插入脚本可以做哪些事情：

1）获取当前域下Cookie

2）发送http请求

3）操作DOM

等等





## 漏洞原因

主要原因还是未对用户的输入进行过滤（通过构建请求可以直接越过前端的验证），后端没有对数据过滤。导致原本的HTML结构被更改，插入了恶性代码。





## 分类

#### 反射型

反射型 XSS 只是将用户输入的数据展现到浏览器上（从哪里来到哪里去），即需要一个发起人（用户）来触发黑客布下的一个陷阱（例如一个链接），才能攻击成功。

如图：

![reflected-xss](http://www.qinhanwen.xyz/reflected-xss.png)

1）攻击者构造特殊的URL，其中添加了恶意代码。

2）受害者被攻击者欺骗打开了这个URL，发送了这个带有恶意代码的请求。

3）请求得到响应，网站服务端将恶意代码从 URL 中取出，未对请求参数进行处理，拼接在 HTML 中返回给浏览器。

4）受害者遭到攻击，发送出了Cookie。



补充：使用innerHtml或outerHTML插入脚本，只是当作普通的html执行，js解析器不会执行js脚本。



##### 举个🌰

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <a href="http://localhost:3000/index?<img src='' onerror='alert(1)' />">
        a标签
    </a>
</body>

</html>
```

页面中有个构造特殊的URL，点击url跳转新页面，后端将请求参数拼接在HTML里返回。

```javascript
const koa = require('koa');
const app = new koa();

const Router = require('koa-router');
const router = new Router();
const ejs = require('ejs');
const views = require('koa-views');
const path = require('path');



router.get('/index',async (ctx) => {
    var url = decodeURIComponent(ctx.request.url);
    var str = url.substr(url.indexOf('?')+1);
    await ctx.render('index',{content: str});
})

app.use(views(path.join(__dirname, './views'), {
    extension: 'ejs'
}))
app.use(router.routes());

app.listen(3000);
console.log('koa server is listening port 3000');
```

下面是模板引擎

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
<%- content %>
</body>
</html>
```

在chrome下，自动帮助用户防御

![WX20190323-230946@2x](http://www.qinhanwen.xyz/WX20190323-230946@2x.png)



#### 存储型

存储型 XSS 会把用户输入的数据保存在服务器端。

如图：

![persistent-xss](http://www.qinhanwen.xyz/persistent-xss.png)

1）攻击者提交了带有恶意代码的数据。

2）后端未对数据处理直接存进数据库。

3）受害者请求数据，返回了带有恶意代码的html，执行了里面的恶意代码。

4）受害者遭到攻击，发送出了Cookie。



#### 那么顺便了解一下插入脚本并且会执行脚本的方法(因为使用innerHTML和outerHTML插入，会被视为一个文本，不执行里面的脚本)

##### 1）document.write

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>

</body>
<script>
  document.write('<script type="text/javascript">alert(1)<\/script>');
</script>
</html>
```



##### 2）动态插入script标签(同步异步都可以)

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <button id="button">发送请求</button>
</body>
<script>
    var button = document.getElementById('button');
    var xml = new XMLHttpRequest();
    xml.onreadystatechange = function () {
        if (xml.readyState === 4) {
            if ((xml.status >= 200 && xml.status <= 300) || xml.status === 304) {
                var myScript = document.createElement("script");
                myScript.type = "text/javascript";
                myScript.innerHTML = 'alert(1)';
                document.body.appendChild(myScript);
            }
        }
    }
    button.addEventListener('click', function (event) {
        xml.open('GET', 'http://localhost:3000/setCookie');
        xml.send();
    })

</script>

</html>
```



##### 3）动态创建script标签，src设置为外部的js文件路径，再插入body中

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <button id="button">发送请求</button>
</body>
<script>
    var button = document.getElementById('button');
    var xml = new XMLHttpRequest();
    xml.onreadystatechange = function () {
        if (xml.readyState === 4) {
            if ((xml.status >= 200 && xml.status <= 300) || xml.status === 304) {
                var myScript = document.createElement("script");
                myScript.type = "text/javascript";
                myScript.src = './test1.js';
                document.body.appendChild(myScript);
            }
        }
    }
    button.addEventListener('click', function (event) {

        xml.open('GET', 'http://localhost:3000/setCookie');
        xml.send();
    })
</script>

</html>
```



##### 4）改变已有的script标签的src

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
</body>
<script id="script"></script>
<script>
    var script = document.getElementById('script');
    script.src = './test1.js';
</script>

</html>
```



##### 5）以及服务端直接返回完整的html其中的脚本会执行



#### DOM型

1）攻击者构造出特殊的 URL，其中包含恶意代码。

2）用户打开带有恶意代码的 URL，发送请求。

3）用户浏览器接收到响应后解析执行，取出 URL 中的恶意代码并执行。

4）恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。



##### 举个🌰

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <a href="http://localhost:3000/index?<img src='' onerror='alert(1)' />">
        a标签
    </a>
</body>
<script>

</script>

</html>
```

某个页面有个别人留下的恶意链接，访问这个页面会返回一个html页面。



```javascript
const koa = require('koa');
const app = new koa();

const Router = require('koa-router');
const router = new Router();
const ejs = require('ejs');
const views = require('koa-views');
const path = require('path');

router.get('/index',async (ctx) => {
    await ctx.render('index');
})

app.use(views(path.join(__dirname, './views'), {
    extension: 'ejs'
}))
app.use(router.routes());

app.listen(3000);
console.log('koa server is listening port 3000');
```



html页面解析的时候，做的事情是从window.location中取出参数并且放到body里，就导致了一次攻击

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
</body>
<script>
    var url = decodeURIComponent(window.location);
    var str = url.substr(url.indexOf('?')+1);
    document.body.innerHTML = str;
</script>

</html>
```





DOM型与反射型比较类似

区别：

DOM型：发送接口请求，获得响应，HTML解析执行页面的脚本的时候，从URL中取出了恶意的代码并且执行。

反射型:  发送接口请求，获得响应，恶意代码附加在了HTML中，HTML解析执行页面。