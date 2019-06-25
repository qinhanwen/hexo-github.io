---
title: nginx入门-配置
date: 2019-06-19 16:18:07
tags: 
- Nginx
categories: 
- Nginx
---

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



























