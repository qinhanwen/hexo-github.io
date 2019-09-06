---
title: nginx入门
date: 2019-06-11 22:35:16
tags: 
- Nginx
categories: 
- Nginx
---

## Nginx

是一个高性能的HTTP和反向代理web服务器，大部分使用作为代理。



## MacOS安装

##### 1.先安装[homebrew](https://brew.sh/index_zh-cn)

```shell
/usr/bin/ruby -e "$(curl -fsSL  https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装的时候碰到这样的问题，一直卡在这里。

![WX20190611-234616@2x](http://www.qinhanwen.xyz/WX20190611-234616@2x.png)



**解决方案**：打开Xcode，然后选择「Xcode」-「Open Developer Tool」-「More Developer Tools」

登陆appleID之后下自己对应版本的Command Line Tools就好了，之后再安装一次。



##### 2.安装Nginx

```shell
$ brew install nginx
```

![WX20190611-235953@2x](http://www.qinhanwen.xyz/WX20190611-235953@2x.png)



## 运行与停止

```shell
$ brew services start nginx 
$ brew services stop nginx 
```

启动之后打开[localhost:8080](localhost:8080)

![WX20190612-000343@2x](http://www.qinhanwen.xyz/WX20190612-000343@2x.png)

##### 1.修改配置

MacOS在文件夹中 command+shift+g，打开/usr/local/etc/nginx文件夹。

修改nginx.conf文件的include，就可以在servers文件夹下添加新的配置文件

![WX20190612-124602@2x](http://www.qinhanwen.xyz/WX20190612-124602@2x.png)



/usr/local/etc/nginx目录下新建一个servers文件里，里面新建一个test.conf文件，语句结尾一定要加封号。文件内容如下

```
    server {
        listen    8899;
        server_name  test.com;

        location / {
           proxy_pass http://127.0.0.1:3000;
        }
    }
```

##### 2.配置host

```shell
$ sudo vi /etc/hosts  
```

![WX20190613-133420@2x](http://www.qinhanwen.xyz/WX20190613-133420@2x.png)

i 编辑，esc 结束，:wq 退出



##### 3.重启nginx

```shell
$ brew services restart nginx
```

koa启动个服务监听http://127.0.0.1:3000/

```javascript
const koa = require('koa');
const app = new koa();
const fs = require('fs');
const Router = require('koa-router');
const router = new Router();

router.get('/', async (ctx, next) => {
    const html = fs.readFileSync('./nginx.html','utf8');
    ctx.body = html;
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
</body>
</html>
```



##### 4.打开[test.com:8899](test.com:8899)

![WX20190613-125410@2x](http://www.qinhanwen.xyz/WX20190613-125410@2x.png)







## Nginx配置文件

```shell
#user  nobody;
#Nginx进程，一般设置为和CPU核数一样
worker_processes  1;
#错误日志存放目录
error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#进程pid存放位置
#pid        logs/nginx.pid;


events {
    worker_connections  1024;# 单个后台进程的最大并发数
}


http {
    include       mime.types;#文件扩展名与类型映射表
    default_type  application/octet-stream;#默认文件类型

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;#开启高效传输模式
    #tcp_nopush     on;#减少网络报文段的数量

    #keepalive_timeout  0;
    keepalive_timeout  65;#保持连接的时间，也叫超时时间

    #gzip  on;#开启gzip压缩
    include servers/*.conf;#包含的子配置项位置和文件

    server {
        listen       8080;#配置监听端口
        server_name  localhost;//配置域名

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;#服务默认启动目录
            index  index.html index.htm;#默认访问文件
        }

        #error_page  404              /404.html;# 配置404页面

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;#错误状态码的显示页面，配置后需要重启
        location = /50x.html {
            root   html;
        }
    }
}
```



##### 多错误指向同个页面

```shell
error_page   500 502 503 504  /50x.html;#错误状态码的显示页面，配置后需要重启
```



##### 单独错误指向页面

```shell
error_page  404              /404.html;# 配置404页面
```



##### 页面可以使用服务器资源

```shell
error_page  404  https://qinhanwen.github.io/;
```

如果找不到页面就会跳转到 [https://qinhanwen.github.io/](https://qinhanwen.github.io/)



##### 控制一些IP访问，我们可以直接在`location`里进行配置



##### 就是在同一个块下的两个权限指令，先出现的设置会覆盖后出现的设置（也就是谁先触发，谁起作用）

```shell
location / {
        allow  45.76.202.231;
        deny   all;
}
```

 deny   all;在前面，就所有ip都不允许访问



##### 所有img目录都允许访问，admin目录不允许访问，allow允许，deny不允许

```shell
 location =/img{
        allow all;
 }
 location =/admin{
   	    deny all;
 }
```

= 号是精确匹配。



##### 修改server选项

```shell
server {
    listen    8899;
    server_name  test.com;
    root /usr/local/var/www;
    index test.html;
} 
#这两个配置结果是一样的，location是做代理，它后面的目录映射到 root 的目录
server {
    listen    8899;
    server_name  test.com;
    location / {
        root /usr/local/var/www;
        index test.html;
    }
}   
```

在[user/local/var/www]()下面新增了一个test.html，重启nginx打开[test.com:8899](test.com:8899)

#####  

##### 反向代理

```shell
server {
    listen    8899;
    server_name  test.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
    }  
}   
```

访问[test.com:8899](test.com:8899)反向代理到[http://127.0.0.1:3000](http://127.0.0.1:3000)



##### 其他反向代理指令

- proxy_set_header :在将客户端请求发送给后端服务器之前，更改来自客户端的请求头信息。
- proxy_connect_timeout:配置Nginx与后端代理服务器尝试建立连接的超时时间。
- proxy_read_timeout : 配置Nginx向后端服务器组发出read请求后，等待相应的超时时间。
- proxy_send_timeout：配置Nginx向后端服务器组发出write请求后，等待相应的超时时间。
- proxy_redirect :用于修改后端服务器返回的响应头中的Location和Refresh。



##### 根据userAgent匹配，加载不同资源

```shell
server {
    listen    8899;
    server_name  test.com;

    location / {
         root /usr/local/var/www/chrome;
         if ($http_user_agent ~* 'Firefox'){
            root /usr/local/var/www/firfox;
         }
         index test.html;   
    }  
}   
```

在[/usr/local/var/www/]()下新建两个文件夹，里面分别有一个test.html

![WX20190620-234420@2x](http://www.qinhanwen.xyz/WX20190620-234420@2x.png)

在chrome访问[test.com:8899](test.com:8899)

![WX20190620-234330@2x](http://www.qinhanwen.xyz/WX20190620-234330@2x.png)

在firfox访问[test.com:8899](test.com:8899)

![WX20190620-234252@2x](http://www.qinhanwen.xyz/WX20190620-234252@2x.png)



可以通过userAgent判断，区别不同平台，来加载不同的资源。



##### Gzip，Gzip是需要服务器和浏览器同时支持的

修改一下配置文件

```shell
server {
    listen    8899;
    server_name  test.com;

    location / {
        proxy_cache my_cache;
        proxy_pass http://127.0.0.1:3000;
    }  
}   

```

访问test.com的时候返回html，size是692B

![WX20190620-235326@2x](http://www.qinhanwen.xyz/WX20190620-235326@2x.png)



开启Gzip，重启nginx

```shell
...
gzip  on;#开启gzip压缩
...

```

![WX20190620-235540@2x](http://www.qinhanwen.xyz/WX20190620-235540@2x.png)

size为546B



Nginx提供了专门的gzip模块，指令：

- gzip : 该指令用于开启或 关闭gzip模块。
- gzip_buffers : 设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。
- gzip_comp_level : gzip压缩比，压缩级别是1-9，1的压缩级别最低，9的压缩级别最高。压缩级别越高压缩率越大，压缩时间越长。
- gzip_disable : 可以通过该指令对一些特定的User-Agent不使用压缩功能。
- gzip_min_length:设置允许压缩的页面最小字节数，页面字节数从相应消息头的Content-length中进行获取。
- gzip_http_version：识别HTTP协议版本，其值可以是1.1.或1.0.
- gzip_proxied : 用于设置启用或禁用从代理服务器上收到相应内容gzip压缩。
- gzip_vary : 用于在响应消息头中添加Vary：Accept-Encoding,使代理服务器根据请求头中的Accept-Encoding识别是否启用gzip压缩。



## 内置全局变量

$host：请求信息中的`Host`，如果请求中没有`Host`行，则等于设置的服务器名

$request_method：客户端请求类型，如`GET`、`POST`

$remote_addr：客户端的`IP`地址

$args：请求中的参数

$content_length：请求头中的`Content-length`字段

$http_user_agent：客户端agent信息

$http_cookie：客户端cookie信息

$remote_port：客户端的端口

$server_protocol：请求使用的协议，如`HTTP/1.0`、·HTTP/1.1`

$server_addr：服务器地址

$server_name：服务器名称

$server_port：服务器的端口号





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

























