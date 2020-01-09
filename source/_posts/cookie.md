---
title: cookie
date: 2019-03-11 19:09:54
tags: 
- 优化
categories: 
- 优化
---

> 参考资料：    	https://juejin.im/post/5aede266f265da0ba266e0ef
                  https://www.jianshu.com/p/0fff1ba9fba5  

## cookie是什么

曲奇饼，哈哈哈哈哈。

首先，http是无状态（就是上一次发请求和下一次发请求无关联，据说是因为http起初只是使用来浏览静态文件的，所以无状态协议就足够了），所以无法知道上一次是谁访问了它，那么cookie其实就是用来记录状态的一种东西。

关于cookie：

1）第一次请求的时候，服务器响应，并把cookie带在响应报文中（从服务端返回的响应报文内有个Set-Cookie的头部字段，告诉客户端保存cookie）

2）在后续的请求中，会将保存的cookie传递，服务器根据cookie判断用户

3）服务端也可以修改cookie内容

![WX20190311-221121@2x](http://118.24.241.76/WX20190311-221121@2x.png)





## cookie属性

**name：**cookie的名字



**value：**cookie的值



**comment：**cookie用处说明



**domain：**可以访问该cookie的`域`，如果设置为“.baidu.com”，则所有以“baidu.com”结尾的域名都可以访问该Cookie；第一个字符必须为“.”



**maxAge：**cookie失效时间（单位：秒）

- 正数，则超过maxAge秒之后失效。
- 负数，该Cookie为临时Cookie，关闭浏览器即失效，浏览器也不会以任何形式保存该Cookie。
- 为0，表示删除该Cookie。



**path：**该cookie的`使用路径`

- path=/，说明本域名下都可以访问该Cookie。
- path=/app/，则只有为“/app”的程序可以访问该Cookie

path设置时，其以“/”结尾.



**secure：**是否使用安全协议，默认false



**version：**该cookie使用的版本号



**isHttpOnly**

HttpOnly属性是用来限制非HTTP协议程序接口对客户端Cookie进行访问；也就是说如果想要在客户端取到httponly的Cookie的唯一方法就是使用AJAX，将取Cookie的操作放到服务端，接收客户端发送的AJAX请求后将取值结果通过HTTP返回客户端。这样能有效的防止XSS攻击。



上述的这些属性，除了name与value属性会被提交外，其他的属性对于客户端来说都是不可读的，也是不可被提交的。



#### 先了解一下koa中cookie的使用，以及cookie与跨域

##### 1）第一步，设置cookie

koa：

```javascript
const koa = require('koa');
const app = new koa();

const Router = require('koa-router');
const router = new Router();

router.get('/setCookie', async (ctx, next) => {
    ctx.cookies.set(
        'name', 'qinhanwen', {
            domain: 'localhost', // 写cookie所在的域名
            path: '/',       // 写cookie所在的路径
            maxAge: 2 * 60 * 60 * 1000,   // cookie有效时长
            expires: new Date('2019-03-15'), // cookie失效时间
            httpOnly: false,  // 是否只用于http请求中获取
            overwrite: false  // 是否允许重写
        }
    );
    ctx.body = {
        success: true
    };
})

app.use(router.routes());

app.listen(3000);
console.log('koa server is listening port 3000');
```

访问`http://localhost:3000/setCookie`



![WX20190312-224503@2x](http://118.24.241.76/WX20190312-224503@2x.png)



##### 2）获取cookie

```javascript
const koa = require('koa');
const app = new koa();

const Router = require('koa-router');
const router = new Router();

router.get('/setCookie', async (ctx, next) => {
    ctx.cookies.set(
        'name', 'qinhanwen', {
            domain: 'localhost', // 写cookie所在的域名
            path: '/',       // 写cookie所在的路径
            maxAge: 2 * 60 * 60 * 1000,   // cookie有效时长
            expires: new Date('2019-03-15'), // cookie失效时间
            httpOnly: false,  // 是否只用于http请求中获取
            overwrite: false  // 是否允许重写
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

app.use(router.routes());

app.listen(3000);
console.log('koa server is listening port 3000');
```

访问`http://localhost:3000/getCookie`

![WX20190312-225045@2x](http://118.24.241.76/WX20190312-225045@2x.png)





#### cookie与跨域

##### 1）基于上面的代码稍作修改，新增一个`cookie.html`前端页面(域名为：http://192.168.70.20:8001)，请求地址（域名为：http://localhost:3000）它们不同域，使用AJAX的方式发送请求。

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
        xml.open('GET', 'http://localhost:3001/setCookie');
        xml.send();
    })
    button1.addEventListener('click', function (event) {
        xml.open('GET', 'http://localhost:3001/getCookie');
        xml.send();
    })
</script>

</html>
```



##### 2）koa部分增加个白名单，设置响应头，允许跨域来源访问。同时将当前目录下的`index`目录设置为静态资源的目录（域名为：http://localhost:3000）与请求地址同域。

```javascript
const koa = require('koa');
const app = new koa();

const Router = require('koa-router');
const router = new Router();
const serveStatic = require('koa-serve-static');


const whiteList = ['http://192.168.70.20:8001'];

app.use((ctx, next) => {
    if (ctx.method === 'OPTIONS') {
        ctx.body = '';
    }
    next();
});

app.use((ctx, next) => {
    let origin = ctx.headers.origin;
    if (whiteList.includes(origin)) {
        ctx.set('Access-Control-Allow-Origin', origin);
    }
    next();
})

router.get('/setCookie', async (ctx, next) => {
    ctx.cookies.set(
        'name', 'qinhanwen', {
            path: '/',       // 写cookie所在的路径
            maxAge: 2 * 60 * 60 * 1000,   // cookie有效时长
            expires: new Date('2019-03-15'), // cookie失效时间
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

app.use(router.routes());
app.use(serveStatic(__dirname + '/index'));

app.listen(3000);
console.log('koa server is listening port 3000');
```



##### 3）先打开刚才这个静态资源的页面`http://localhost:3000/test.html`

![image-20190315183949714](http://118.24.241.76/WX20190315-183943@2x.png)

这个域下的cookies没有数据。



##### 4）打开`http://192.168.70.20:8001/test.html`，点击登录按钮

![WX20190315-184516@2x](http://118.24.241.76/WX20190315-184516@2x.png)

请求发送成功，没有在当前的域写入cookie。

点击第二个按钮，请求发送成功，没有取得cookie

![WX20190315-184741@2x](http://118.24.241.76/WX20190315-184741@2x.png)



##### 以上与cookie没有写入，以及没能取得的相关知识：

- CORS默认不会发送cookie，如果需要发送cookie，AJAX需要打开`withCredentials`属性。
- 服务器也需要同意，设置`Access-Control-Allow-Credentials: true`。

如果没有同时设置，那么浏览器不会发送cookie，服务器要求设置cookie，也会被浏览器无视。



##### 5）修改一下代码

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
        xml.open('GET', 'http://localhost:3000/setCookie');
        //新增
        xml.withCredentials = true;
        //新增
        xml.send();
    })
    button1.addEventListener('click', function (event) {
        xml.open('GET', 'http://localhost:3000/getCookie');
        //新增
        xml.withCredentials = true;
        //新增
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
const serveStatic = require('koa-serve-static');


const whiteList = ['http://192.168.70.20:8001'];

app.use((ctx, next) => {
    if (ctx.method === 'OPTIONS') {
        ctx.body = '';
    }
    next();
});

app.use((ctx, next) => {
    let origin = ctx.headers.origin;
    if (whiteList.includes(origin)) {
        ctx.set('Access-Control-Allow-Origin', origin);
        //新增
        ctx.set('Access-Control-Allow-Credentials', true);
        //新增
    }
    next();
})

router.get('/setCookie', async (ctx, next) => {
    ctx.cookies.set(
        'name', 'qinhanwen', {
            path: '/',       // 写cookie所在的路径
            maxAge: 2 * 60 * 60 * 1000,   // cookie有效时长
            expires: new Date('2019-03-15'), // cookie失效时间
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

app.use(router.routes());
app.use(serveStatic(__dirname + '/index'));

app.listen(3000);
console.log('koa server is listening port 3000');
```

重新点击登录发送请求，cookies仍然没有存入当前域。

![WX20190315-191006@2x](http://118.24.241.76/WX20190315-191006@2x.png)

点击第二个按钮，请求发送成功，并且取得了cookie的值。

![WX20190315-191128@2x](http://118.24.241.76/WX20190315-191128@2x.png)



##### 6）那么cookie的值是哪来的？

打开之前的`http://localhost:3000/test.html`页面，在`http://localhost:3000`的域下发现了cookie。

![WX20190315-191314@2x](http://118.24.241.76/WX20190315-191314@2x.png)

因为后端项目运行在`http://localhost:3000`域上，所以发回的cookie对应的域名也就是`http://localhost:3000`



##### 

#### 接着将之前允许跨域来源访问的字段设置为`*`号

```javascript
const koa = require('koa');
const app = new koa();

const Router = require('koa-router');
const router = new Router();
const serveStatic = require('koa-serve-static');


const whiteList = ['http://192.168.70.20:8001'];

app.use((ctx, next) => {
    if (ctx.method === 'OPTIONS') {
        ctx.body = '';
    }
    next();
});

app.use((ctx, next) => {
    let origin = ctx.headers.origin;
    if (whiteList.includes(origin)) {
        ctx.set('Access-Control-Allow-Origin', '*');
        ctx.set('Access-Control-Allow-Credentials', true);
    }
    next();
})

router.get('/setCookie', async (ctx, next) => {
    ctx.cookies.set(
        'name', 'qinhanwen', {
            path: '/',       // 写cookie所在的路径
            maxAge: 2 * 60 * 60 * 1000,   // cookie有效时长
            expires: new Date('2019-03-15'), // cookie失效时间
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

app.use(router.routes());
app.use(serveStatic(__dirname + '/index'));

app.listen(3000);
console.log('koa server is listening port 3000');
```

页面再次发送请求的时候，看到如图报错：

![WX20190315-192336@2x](http://118.24.241.76/WX20190315-192336@2x.png)



大概就是说`http//192.168.70.20:8001`访问`http://localhost:3000/setCookie`被CORS拦截，当XMLHttpRequest对象的控制的`withCredentials`是开启的时候，`Access-Control-Allow-Origin`不能是`*`号。



​	如果要发送Cookie，`Access-Control-Allow-Origin`就不能设为星号，必须指定明确的、与请求网页一致的域名。同时，Cookie依然遵循同源政策，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨源）原网页代码中的`document.cookie`也无法读取服务器域名下的Cookie。



#### cookie更新

一般使用相同`name`的`cookie`覆盖之前的。



#### cookie删除

分别测试`maxAge`不同值时，浏览器表现：

1）为正值：存入cookie

2）为负值：没有存入





#### JavaScript与cookie

##### 1）读cookie

```javascript
document.cookie
```



##### 2）写cookie

```javascript
document.cookie = "age=25";
```

![WX20190314-225013@2x](http://118.24.241.76/WX20190314-225013@2x.png)



## cookie限制

![WX20190625-234902@2x](http://118.24.241.76/WX20190625-234902@2x.png)



#### **总结**

1）客户端与服务端都可以设置cookie，不同域只能看到自己域下的cookie。

2）cookie默认不会跨域，发送请求的时候，机器会查看与域名有关的cookie，如果存在就一起发送到服务器，不存在就不发送。

3）cookie需要跨域的话，AJAX的withCredentials设置为开启，并且服务器设置`Access-Control-Allow-Credentials: true`。客户端才可以发送cookie，服务端设置cookie才会被客户端接受。

4）跨域的时候cookie被存储的地方是与请求地址同源的地方。

5）跨域的时候传递cookie，`Access-Control-Allow-Origin`不能是`*`号。

6）Cookie是否同源与domain和path有关。 



























