---
title: nginx入门-缓存
date: 2019-06-19 16:23:16
tags: 
- Nginx
categories: 
- Nginx
---
## 缓存配置

##### 1.修改一下之前新建的test.conf文件

```
proxy_cache_path /usr⁩/local⁩/etc⁩/nginx⁩ levels=1:2 keys_zone=my_cache:10m;

server {
    listen    8899;
    server_name  test.com;

    location / {
        proxy_cache my_cache;
        proxy_pass http://127.0.0.1:3000;
    }
}

```

说明：

/usr⁩/local⁩/etc⁩/nginx⁩   目录      

levels=1:2 	 是否创建二级文件夹，如果没创建会随着缓存变多越来越慢

 keys_zone=my_cache:10m 	声明内存大小来缓存，以及之后可以通过名字来确定使用哪个缓存



##### 2.重启nginx的时候，打不开[test.com:8899]()

```shell
$ nginx -T
#检查配置
```



**报错信息：**

![WX20190614-144633@2x](http://www.qinhanwen.xyz/WX20190614-144633@2x.png)

原因是在nginx.conf文件里，两次 include server/* 导致的，删掉一个就好了



## 缓存功能

#### 第一个例子，设置 Cache-Control:max-age=20,s-maxage=0

##### 1.创建服务

```javascript
const koa = require('koa');
const app = new koa();
const fs = require('fs');
const Router = require('koa-router');
const router = new Router();

timeout = () => {
    return new Promise(resolve => {
        setTimeout(resolve, 2000);
    })
}

router.get('/', async (ctx, next) => {
    const html = fs.readFileSync('./nginx.html', 'utf-8');
    ctx.body = html;
})

router.get('/getData', async (ctx, next) => {
    ctx.set('Cache-Control', 'max-age=20,s-maxage=0');
    await timeout();
    ctx.body = 'success';
})

app.use(router.routes());

app.listen(3000);
console.log('koa server is listening port 3000');
```

服务监听127.0.0.1:3000，访问 / 的时候返回html页面，访问 /getData 的时候设置响应头，延迟2秒返回响应体



##### 2.添加页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    <div>nginx</div>
</body>
<script>
    fetch('/getData').then(result=>{
        return result.text()
    }).then((text)=>{
        document.body.innerHTML = text;
    })
</script>
</html>
```

页面加载的时候发送请求，并且改变body的innerHTML



##### 3.打开页面

上面配置了test.com:8899映射到127.0.0.1:3000，所以打开[test.com:8899](test.com:8899)，两秒后响应，body内容变成success

##### ![WX20190614-235629@2x](http://www.qinhanwen.xyz/WX20190614-235629@2x.png)

然后快速刷新页面，发现页面显示success，并且状态码旁边显示 from disk cache

![WX20190615-000334@2x](http://www.qinhanwen.xyz/WX20190615-000334@2x.png)

然后快速打开火狐浏览器，再缓存失效时间之前，访问[test.com:8899](test.com:8899)，页面2秒后，从body内容变成了success。

![WX20190616-221144@2x](http://www.qinhanwen.xyz/WX20190616-221144@2x.png)

**以上是浏览器缓存**





#### 第二个例子，设置Cache-Control:max-age=0,s-maxage=20

```javascript
const koa = require('koa');
const app = new koa();
const fs = require('fs');
const Router = require('koa-router');
const router = new Router();

timeout = () => {
    return new Promise(resolve => {
        setTimeout(resolve, 2000);
    })
}

router.get('/', async (ctx, next) => {
    const html = fs.readFileSync('./nginx.html', 'utf-8');
    ctx.body = html;
})

router.get('/getData', async (ctx, next) => {
    ctx.set('Cache-Control', 'max-age=0,s-maxage=20');
    await timeout();
    ctx.body = 'success';
})

app.use(router.routes());

app.listen(3000);
console.log('koa server is listening port 3000');
```



先打开chrome，访问[test.com:8899](test.com:8899)，页面2秒后，body内容变成了success。

在缓存失效之前，打开火狐，访问[test.com:8899](test.com:8899)，页面一闪而过显示success



**以上是代理缓存**



#### 第三个例子

###### 1）先设置Cache-Control:max-age=0,s-maxage=20,private

刷新页面，每次都要等待2秒才会将body内容变为success。



###### 2）再设置max-age=5,s-maxage=20,private

第一次刷新页面等待2秒后，body内容变为success，只要缓存5秒失效时间内刷新，都是显示success，并且是来自磁盘的缓存

![WX20190616-235017@2x](http://www.qinhanwen.xyz/WX20190616-235017@2x.png)





**private这个属性是代理服务器不允许缓存**





#### 第四个例子，设置Cache-Control:max-age=20,s-maxage=20,no-store

这个是代理服务器和浏览器都不缓存



#### 第五个例子，Vary头部

```javascript
xconst koa = require('koa');
const app = new koa();
const fs = require('fs');
const Router = require('koa-router');
const router = new Router();

timeout = () => {
    return new Promise(resolve => {
        setTimeout(resolve, 2000);
    })
}

router.get('/', async (ctx, next) => {
    const html = fs.readFileSync('./nginx.html', 'utf-8');
    ctx.body = html;
})

router.get('/getData', async (ctx, next) => {
  	//这里设置浏览器缓存200秒
    ctx.set('Cache-Control', 'max-age=200');
  	//并且设置Vary头部，只有Test-Cache值一样的时候才使用缓存
    ctx.set('Vary', 'Test-Cache');
    await timeout();
    ctx.body = 'success';
})

app.use(router.routes());

app.listen(3000);
console.log('koa server is listening port 3000');
```

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <div>nginx</div>
    <button onclick="send()">send</button>
</body>
<script>
    var index = 0;
    function send() {
        fetch('/getData',{
            headers:{
                'Test-Cache':index++
            }
        }).then(result => {
            return result.text()
        }).then((text) => {
            document.getElementsByTagName('div')[0].innerHTML = index;
        })
    }
</script>

</html>
```

点击到页面变成5之前，都是缓存的数据

![nginx-cache](http://www.qinhanwen.xyz/nginx-cache.gif)



因为这个头部

![WX20190617-143618@2x](http://www.qinhanwen.xyz/WX20190617-143618@2x.png)

这个头部的意思就是接受某些指定请求头，请求头的值如果被缓存过，未失效，就使用缓存。一般使用这个头部用于区分平台，是否使用缓存。











































