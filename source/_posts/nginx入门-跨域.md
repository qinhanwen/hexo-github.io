---
title: nginx入门-跨域
date: 2019-06-20 23:59:10
tags: 
- Nginx
categories: 
- Nginx
---

## Nginx跨域

页面在[http://192.168.0.103:8000/](http://192.168.0.103:8000/)域下

前端接口请求[http://test.com:8899](http://test.com:8899)

服务起在[http://127.0.0.1/3000](http://127.0.0.1/3000)域下



koa

```javascript
const koa = require('koa');
const app = new koa();
const fs = require('fs');
const Router = require('koa-router');
const router = new Router();

app.use((ctx, next) => {
    if (ctx.method === 'OPTIONS') {
        ctx.body = '';
    }
    next();
});

router.get('/', async (ctx, next) => {
    const html = fs.readFileSync('./nginx.html', 'utf-8');
    ctx.body = html;
})

router.put('/getData', async (ctx, next) => {
    ctx.body = 'success';
})

app.use(router.routes());

app.listen(3000);
console.log('koa server is listening port 3000');
```



nginx.html

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
    function send() {
        fetch('http://test.com:8899/getData',{
            method:'PUT',
            headers:{
                name:'qinhanwen'
            }
        }).then(result => {
            return result.text()
        }).then((text) => {
            document.getElementsByTagName('div')[0].innerHTML = 'success';
        })
    }
</script>

</html>
```



Nginx配置文件

```shell
server {
    listen    8899;
    server_name  test.com;

    location / {
        add_header 'Access-Control-Allow-Origin' '*';
	    	add_header 'Access-Control-Allow-Methods' 'PUT';
	   	 	add_header 'Access-Control-Allow-Headers' 'name';
	   	 	add_header 'Access-Control-Max-Age' '6';

        proxy_pass http://127.0.0.1:3000;
    }  
}   
```

[跨域，以及关于上面的头部介绍]([https://qinhanwen.github.io/2018/11/27/%E8%B7%A8%E5%9F%9F%E7%9A%84%E5%87%A0%E7%A7%8D%E6%96%B9%E5%BC%8F/](https://qinhanwen.github.io/2018/11/27/跨域的几种方式/))