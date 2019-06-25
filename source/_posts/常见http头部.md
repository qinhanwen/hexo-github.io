---
title: 常见HTTP头部
date: 2019-02-27 11:02:55
tags: 
- http
categories: 
- http
---

> 参考资料    图解HTTP

只写了一些常见的（我常见），没有区分HTTP不同版本的头部



## 为什么需要头部

http头部字段主要是给客户端还有服务器端提供报文主体大小，所用语言，以及认证信息等。



## 头部字段结构

```javascript
头部字段名 : 字段值
```



## 头部字段类型

##### 1）通用头部字段：请求报文和响应报文都会使用的字段

##### 2）请求头部字段：客户端向服务端发送请求报文时使用的字段

##### 3）响应头部字段：服务端向客户端返回响应报文时使用的字段

##### 4）实体头部字段：针对请求报文与响应报文的实体部分使用的字段



## 通用头部字段

#### Cache-Control    

[与浏览器缓存策略有关](https://qinhanwen.github.io/2018/11/26/%E6%B5%8F%E8%A7%88%E5%99%A8%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6/)

###### 属性值：

**public**：表明其他用户也可以利用缓存

**private**：缓存只能提供给某个用户利用

[**no-cache**](https://wpcertification.blogspot.com/2010/07/difference-between-no-cache-and-no.html)：

​	1）在请求报文中：不接收缓存过的响应，`缓存服务器`会将请求发送给`源服务器`获取资源（就是每次请求都需要服务器验证）。

​	2）在响应报文中：不是不缓存，告诉客户端下次请求资源的时候，会发送请求给`源服务器`，判断文件是否修改。

**no-store**：不能缓存请求或响应的内容。

**max-age=6000**（单位:秒）：

1）客户端发送的的请求中如果带有max-age字段，缓存指定的时间比max-age字段所带的数值小的话，就会从缓存服务器返回。如果`max-age=0`，就需要将请求发送给`源服务器`。HTTP1.1版本缓存服务器中，max-age优先级大于Expires，HTTP1.0相反。

2）在响应报文中的时候，表示不超过这段时间内，`缓存服务器`就不会发送请求给`源服务器`。





#### Connection

##### 1）控制不再转发给代理服务器的头部字段

##### 2）管理持久链接

属性值：

**不再转发的属性值**

![WX20190227-184022@2x](http://www.qinhanwen.xyz/WX20190227-184022@2x.png)

**Close**：当服务端想断开链接的时候的值。

**Keep-Alive**：HTTP1.1默认支持长链接，之前的版本需要加上才支持长链接。





#### Date

表示报文创建的时间





#### Pragma

HTTP1.1之前版本遗留的字段

属性值：

**no-cache**：不接受缓存的资源





#### Transfer-Encoding

传输报文时使用的编码方式





#### Upgrade

用于检测是否可以使用更高版本的HTTP协议或其他协议

属性值：

可以设置为一个完全不同的协议

![WX20190227-222137@2x](http://www.qinhanwen.xyz/WX20190227-222137@2x.png)











## 请求头部字段

下面所有的`q=`后面跟随一个数值从0到1，表示优先级。

#### Accept

告知服务器，客户端能够接受的文件类型。

属性值之间` ;`隔开，`q=`表示优先级。





#### Accept-Encoding

告知服务器，客户端所支持的内容编码，以及内容编码优先级。

![WX20190227-223306@2x](http://www.qinhanwen.xyz/WX20190227-223306@2x.png)





#### Accept-Language

告知服务器，客户端能够处理的自然语言集，以及优先级`q=`表示。





#### Authorization

一般用来放一些认证信息，使用JWT的时候，也是放在这个字段传递。





#### Host

告诉服务器请求的资源的主机名



#### 

#### If-None-Match

其实是上一次请求返回的ETag字段（文件在服务器上的唯一标识）的值。(可以根据文件内容hash运算生成)

![WX20190227-235752@2x](http://www.qinhanwen.xyz/WX20190227-235752@2x.png)



#### If-Modified-Since

服务器上文件的最后修改时间，其实就是上一次请求返回的Last-Modified的值。优先级小于`If-None-Match`。

![WX20190227-235514@2x](http://www.qinhanwen.xyz/WX20190227-235514@2x.png)



#### If-Match

与`If-None-Match`相反，如果一样才返回200。

![WX20190301-105530@2x](http://www.qinhanwen.xyz/WX20190301-105530@2x.png)



#### If-Unmodified-Since

与`If-Modified-Since`相反，如果一样的话返回200。不一样的话返回412。





#### Referer

请求发出的URI





#### User-Agent

浏览器以及用户代理等信息







## 响应头部字段

由服务端向客服端返回的响应报文中使用的字段。



#### ETag

资源的唯一标识，资源被更新，ETag也会改变。







## 实体头部字段

请求报文与响应报文中对实体的描述信息的字段

#### Allow

告诉客户端所支持的HTTP请求方法，如果接受到不支持的方法请求，返回405 Method Not Allowed状态码





#### Content-Encoding

告诉客户端对实体的主题内容部分使用的内容编码格式





#### Content-Length

实体主体部分的大小(单位：字节)





#### Content-Type

主体内传输的媒体类型

属性值：

**application/x-www-form-urlencoded** 

 x-www-form-urlencoded：只能上传键值对，并且键值对都是间隔分开的，只是最后会转化为一条信息。
 ```
POST  HTTP/1.1
Host: test.app.com
Content-Type: application/x-www-form-urlencoded
Cache-Control: no-cache
Postman-Token: e00dbaf5-15e8-3667-6fc5-48ee3cc89758

key1=value1&key2=value2
 ```



**multipart/form-data** 

multipart/form-data：既可以上传文件等二进制数据，也可以上传表单键值对。

```
POST  HTTP/1.1
Host: test.app.com
Cache-Control: no-cache
Postman-Token: 59227787-c438-361d-fbe1-75feeb78047e
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="filekey"; filename=""
Content-Type: 


------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="textkey"

tttttt
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```



**application/json** 

比如在数据提交或者表单提交的时候，Content-Type字段会说明提交的数据的类型，上面这几种对应的数据类型格式也不一样。之后学习[表单提交与AJAX提交的区别]()的时候，会再深入了解。





#### Expires

缓存资源失效时间，优先级比`Cache-Control`的`max-age`低，HTTP1.1之前的字段。





#### Last-Modified

资源最终修改时间

























