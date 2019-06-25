---
title: nginx入门-安装
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



