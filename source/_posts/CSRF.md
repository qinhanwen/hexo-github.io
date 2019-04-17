---
title: 了解CSRF
date: 2019-03-14 23:50:20
tags: 
- web安全
categories: 
- web安全
---

> 参考资料	<https://juejin.im/post/5bc009996fb9a05d0a055192>
>
> ​			[先了解一下Cookie相关](<https://qinhanwen.github.io/2019/03/11/cookie/#more>)



## CSRF是什么

CSRF（Cross-site request forgery）跨站请求伪造。攻击者诱导用户点击进入第三方网站，在第三方网站中，对攻击网站发送跨域请求。其中携带了用户的登录成功的信息，达到冒充用户的目的。

比如：

1）用户访问a.com网站，并且获取了登录凭证(Cookie)。

2）攻击者在a.com网站留下了b.com的地址，诱导用户点击。

3）当用户点击进入b.com的时候，b.com向a.com同域的接口发送了一个请求，这个请求将会携带a.com域的Cookie。

4）服务端验证Cookie成功，并认为是用户执行操作。



## 实战

##### 1）当访问`http://www.qinhanwen.xyz:3000/index`的时候，页面如下：

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <button id="button">登录（登录成功的话会存入cookie）</button>
    <button id="button1">已完成登录之后的其他请求（会携带cookie）</button>
</body>
<script>
    var button = document.getElementById('button');
    var button1 = document.getElementById('button1');
    var xml = new XMLHttpRequest();
    xml.onreadystatechange = function () {
        if (xml.readyState === 4) {
            if ((xml.status >= 200 && xml.status <= 300) || xml.status === 304) {
                console.log('发送成功');
            }
        }
    }
    button.addEventListener('click', function (event) { 
        xml.open('GET', 'http://www.qinhanwen.xyz:3000/setCookie');
        xml.send();
    })
    button1.addEventListener('click', function (event) {
        xml.open('GET', 'http://www.qinhanwen.xyz:3000/getCookie');
        xml.send();
    })
</script>

</html>
```



koa部分：

```javascript
const koa = require('koa');
const app = new koa();

const Router = require('koa-router');
const router = new Router();
const ejs = require('ejs');
const views = require('koa-views');
const path = require('path');

router.get('/setCookie', async (ctx, next) => {
    ctx.cookies.set(
        'name', 'qinhanwen', {
            path: '/',       // 写cookie所在的路径
            maxAge: 2 * 60 * 60 * 1000,   // cookie有效时长
            expires: new Date('2019-03-19'), // cookie失效时间
        }
    );
    ctx.body = {
        success: true
    };
})

router.get('/getCookie', async (ctx, next) => {
    let name = ctx.cookies.get('name')
    ctx.body = {
        name
    };
})


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

保证了页面与接口处于相同域。同时请求`http://www.qinhanwen.xyz:3000/setCookie`接口，会获得Cookie。请求`http://www.qinhanwen.xyz:3000/getCookie`接口，则会将传递的Cookie放在响应体内返回。



访问页面的时候，当前域下的Cookie如下图：

![WX20190317-234008@2x](http://www.qinhanwen.xyz/WX20190317-234008@2x.png)





##### 2）点击登录按钮

`http://www.qinhanwen.xyz:3000`域下多了个Cookie，这就是它的登录凭证。

![WX20190317-234118@2x](http://www.qinhanwen.xyz/WX20190317-234118@2x.png)





##### 3）点击第二个按钮，会发送另外一个请求到`http://www.qinhanwen.xyz:3000`域下，同时会自动带上这个域下的Cookie，接着请求返回，Cookie的值被放在响应体内。如图：

![WX20190317-234348@2x](http://www.qinhanwen.xyz/WX20190317-234348@2x.png)





##### 4）这时候新增一个不同域的链接，模仿第三方网站。

修改一下初始的页面，新增一个链接，跳转到新增的页面

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <button id="button">登录（登录成功的话会存入cookie）</button>
    <button id="button1">已完成登录之后的其他请求（会携带cookie）</button>
    <a href="http://192.168.0.102:8000/cookie1.html">http://192.168.0.102:8000/cookie1.html</a>
</body>
<script>
    var button = document.getElementById('button');
    var button1 = document.getElementById('button1');
    var xml = new XMLHttpRequest();
    xml.onreadystatechange = function () {
        if (xml.readyState === 4) {
            if ((xml.status >= 200 && xml.status <= 300) || xml.status === 304) {
                console.log('发送成功');
            }
        }
    }
    button.addEventListener('click', function (event) { 
        xml.open('GET', 'http://www.qinhanwen.xyz:3000/setCookie');
        xml.send();
    })
    button1.addEventListener('click', function (event) {
        xml.open('GET', 'http://www.qinhanwen.xyz:3000/getCookie');
        xml.send();
    })
</script>

</html>
```

新增的页面：

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <form id="qhw" name="qhw" action="http://www.qinhanwen.xyz:3000/csrf" method="post">
        <input type="submit" value="submit" />
    </form>
</body>
<script>
    document.qhw.submit();
</script>

</html>
```

其实就是一个表单，向`http://www.qinhanwen.xyz:3000/csrf`接口地址，发送了一个post提交请求。



修改一下koa部分，新增`http://www.qinhanwen.xyz:3000/csrf`，响应体为key为name的Cookie

```javascript
const koa = require('koa');
const app = new koa();

const Router = require('koa-router');
const router = new Router();
const ejs = require('ejs');
const views = require('koa-views');
const path = require('path');


router.get('/setCookie', async (ctx, next) => {
    ctx.cookies.set(
        'name', 'qinhanwen', {
            path: '/',       // 写cookie所在的路径
            maxAge: 2 * 60 * 60 * 1000,   // cookie有效时长
            expires: new Date('2019-03-19'), // cookie失效时间
        }
    );
    ctx.body = {
        success: true
    };
})

router.get('/getCookie', async (ctx, next) => {
    let name = ctx.cookies.get('name')
    ctx.body = {
        name
    };
})

router.post('/csrf', async (ctx, next) => {
    let name = ctx.cookies.get('name');
    ctx.body = {
        name,orgin
    };
})

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



再次访问`http://www.qinhanwen.xyz:3000/index`，Cookie中存在值，这时候点击外链，表单自动提交，并且返回响应体

![WX20190317-235320@2x](http://www.qinhanwen.xyz/WX20190317-235320@2x.png)



响应体如图：

![WX20190318-134953@2x](http://www.qinhanwen.xyz/WX20190318-134953@2x.png)

服务端处理了来自第三方站点发送的请求，同时携带了Cookie（用户凭证），并且做出响应。

以上便完成了一次CSRF攻击。



## CSRF中的攻击类型

##### 1）post请求

上面的表单提交就是一个例子



##### 2）get请求

在第三方站点内，添加一个img标签，src指向`http://www.qinhanwen.xyz:3000/getCookie`



稍微修改一下之前的代码

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <button id="button">登录（登录成功的话会存入cookie）</button>
    <button id="button1">已完成登录之后的其他请求（会携带cookie）</button>
    <a href="http://192.168.70.20:8000//cookie1.html">http://192.168.70.20:8000//cookie1.html</a>
</body>
<script>
    var button = document.getElementById('button');
    var button1 = document.getElementById('button1');
    var xml = new XMLHttpRequest();
    xml.onreadystatechange = function () {
        if (xml.readyState === 4) {
            if ((xml.status >= 200 && xml.status <= 300) || xml.status === 304) {
                console.log('发送成功');
            }
        }
    }
    button.addEventListener('click', function (event) { 
        xml.open('GET', 'http://www.qinhanwen.xyz:3000/setCookie');
        xml.send();
    })
    button1.addEventListener('click', function (event) {
        xml.open('GET', 'http://www.qinhanwen.xyz:3000/getCookie');
        xml.send();
    })
</script>

</html>
```

Koa:

```javascript
const koa = require('koa');
const app = new koa();

const Router = require('koa-router');
const router = new Router();
const ejs = require('ejs');
const views = require('koa-views');
const path = require('path');

router.get('/setCookie', async (ctx, next) => {
    ctx.cookies.set(
        'name', 'qinhanwen', {
            path: '/',       // 写cookie所在的路径
            maxAge: 2 * 60 * 60 * 1000,   // cookie有效时长
            expires: new Date('2019-03-19'), // cookie失效时间
        }
    );
    ctx.body = {
        success: true
    };
})

router.get('/getCookie', async (ctx, next) => {
    let name = ctx.cookies.get('name');
    console.log(name);
    ctx.body = {
        name
    };
})

router.post('/csrf', async (ctx, next) => {
    let name = ctx.cookies.get('name');
    ctx.body = {
        name
    };
})

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



第三方页面：

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
    <img src="http://www.qinhanwen.xyz:3000/getCookie" />
</body>



</html>
```



当点击第三方页面的时候，如图：

![WX20190318-131519@2x](http://www.qinhanwen.xyz/WX20190318-131519@2x.png)

可以看到跨域阻止了响应，但是服务端仍然收到了这个请求，并且打印出了key为name的Cookie，如图：

![WX20190318-131641@2x](http://www.qinhanwen.xyz/WX20190318-131641@2x.png)

也就是说服务端收到了携带用户凭证的请求（Cookie），并且会做出响应。





##### 3）还有一个就是在第三方页面再添加一个a标签，点击的时候会发送请求，这个就不多说了。



## 如何防御

##### 1）检测来源(origin,referer)

修改一下上面的koa代码，分别打印它们的origin和referer看一下

```javascript
const koa = require('koa');
const app = new koa();

const Router = require('koa-router');
const router = new Router();
const ejs = require('ejs');
const views = require('koa-views');
const path = require('path');



router.get('/setCookie', async (ctx, next) => {
    ctx.cookies.set(
        'name', 'qinhanwen', {
            path: '/',       // 写cookie所在的路径
            maxAge: 2 * 60 * 60 * 1000,   // cookie有效时长
            expires: new Date('2019-03-19'), // cookie失效时间
        }
    );
    ctx.body = {
        success: true
    };
})

router.get('/getCookie', async (ctx, next) => {
    let name = ctx.cookies.get('name');
    let orgin = ctx.request.header.origin;
    let referer = ctx.request.header.referer;
    console.log('name：'+name);
    console.log('orgin：'+orgin);
    console.log('referer：'+referer);
    ctx.body = {
        name,orgin,referer
    };
})

router.post('/csrf', async (ctx, next) => {
    let name = ctx.cookies.get('name');
    let orgin = ctx.request.header.origin;
    let referer = ctx.request.header.referer;
    console.log('name：'+name);
    console.log('orgin：'+orgin);
    console.log('referer：'+referer);
    ctx.body = {
        name,orgin,referer
    };
})

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



当同域下的页面访问接口`http://www.qinhanwen.xyz:3000/getCookie`的时候，打印如图：

分别是 key为name的Cookie，origin以及referer

![WX20190318-135449@2x](http://www.qinhanwen.xyz/WX20190318-135449@2x.png)



当访问第三方站点，自动提交表单访问`http://www.qinhanwen.xyz:3000/csrf`接口的时候，打印如图：

分别是 key为name的Cookie，origin以及referer

![WX20190318-135523@2x](http://www.qinhanwen.xyz/WX20190318-135523@2x.png)

#### 补充个知识点：

###### 跨域的时候：get，post都会显示origin。

###### 同域的时候：get不显示origin，post显示origin。



##### 不过origin以及referer主要依赖浏览器，不是最好的选择。



#### referer这个以后再深入了解吧，先放着



##### 2）token

比如用户登陆之后，为用户添加一个标识（随机生成的唯一标识），标识不再放在Cookie里。在请求中传递。







### 总结

CSRF攻击主要是在第三方站点发生的，它只是使用了用户的登录凭证，没能够获得用户的登录凭证。

































​	



