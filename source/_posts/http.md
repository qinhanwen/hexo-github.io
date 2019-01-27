---
title: http
date: 2018-12-27 17:48:31
tags: 
- 深入学习
categories: 
- 深入学习
---

# HTTP（HyperText Transfer Protocol）

> 参考资料		图解HTTP

#### 网络基础TCP/IP简单了解

##### 1）协议：计算机与网络设备之间的通信，双方要基于相同的方法，比如谁发起，用哪种语言，如何结束通信（协议中内容很多不止这些）。而TCP/IP是各种协议的总称。



##### 2）TCP/IP分层

应用层，传输层，网络层，链路层

![WX20190113-233417@2x](http://www.qinhanwen.xyz/images/WX20190113-233417@2x.png)

我对发送端的理解：

`应用层`生成请求报文。

`传输层`分割请求报文，传递数据。

`网络层`做数据的转发，寻址。

`链路层`真正传输数据



##### 3）与HTTP相关的协议：IP、TCP、DNS

###### 1. IP协议：位于`网络层`作用是将数据包传给对方，一般会经过多台设备中转，才连接到对方。

![WX20190113-234540@2x](http://www.qinhanwen.xyz/images/WX20190113-234540@2x.png)





###### 2. TCP协议：位于`传输层`TCP可以把数据分割，方便传输，并且能准确判断对方是否收到。

三次握手：为了保证数据准确的送达

![WX20190114-000520@2x](http://www.qinhanwen.xyz/images/WX20190114-000520@2x.png)



当然还要四次挥手，后面再说。



###### 3.DNS协议：位于`应用层`，主要做域名解析成IP地址或者IP地址查域名。

![WX20190114-004038@2x](http://www.qinhanwen.xyz/images/WX20190114-004038@2x.png)

#### HTTP基于请求与响应

![](http://www.qinhanwen.xyz/images/WX20181227-191704@2x.png)



#### HTTP请求报文

![](http://www.qinhanwen.xyz/images/8BB78A283B1A235F90EA35F889E20097.png)



##### 1）请求行

![WX20190114-011840@2x](http://www.qinhanwen.xyz/images/WX20190114-011840@2x.png)



##### 2）请求头

key—value的形式，换行隔开每一个key—value



##### 3）空行

标识请求头的结束



##### 4）请求体





#### HTTP响应报文

![](http://www.qinhanwen.xyz/images/5B4009A4A14D20AF74997DEDE2F1A0EA.png)



##### 1）响应行

![WX20190114-012302@2x](http://www.qinhanwen.xyz/images/WX20190114-012302@2x.png)



##### 2）响应头

key—value的形式，换行隔开每一个key—value

```
Content-Type:text/html;charset=UTF-8
```

服务器告诉浏览器响应内容是什么类型，以及采用的是什么字符编码



##### 3）空行

标识响应头结束



##### 4）响应体

这里的响应体是一个html，浏览器可以直接识别



#### 3.请求方法（HTTP1.1支持）

1）常见请求方法

GET，POST，OPTIONS，HEAD，PUT，DELETE

OPTIONS方法有关于CORS，[跨域的几种方法](https://qinhanwen.github.io/2018/11/27/%E8%B7%A8%E5%9F%9F%E7%9A%84%E5%87%A0%E7%A7%8D%E6%96%B9%E5%BC%8F/)



2）不常见（我不常见哈哈哈哈）

TRACE，CONNECT



#### 常见状态码（我常见的😂）

200：成功

304：文档内容没有修改，[了解更多](https://qinhanwen.github.io/2018/11/26/%E6%B5%8F%E8%A7%88%E5%99%A8%E7%BC%93%E5%AD%98%E6%9C%BA%E5%88%B6/)

401：没有令牌不允许访问

403：禁止访问

404：找不到

405：方法不被允许

500：服务器内部错误

[其他的状态码](https://blog.csdn.net/qq_31796711/article/details/78924366)



#### HTTP1.0 、HTTP 1.1、 HTTP 2.0主要区别

###### 1）长链接

初始版本每次链接之后就会断开

HTTP1.0需要带上keep-alive标识告诉服务器建立长链接，而HTTP1.1默认支持，链接的时候可以发送多次请求，收到多次响应。

HTTP基于TCP/IP，需要经历三次握手，四次挥手，长链接可以节省开销，和开销的时间。这样子数据可以更快返回，页面也可以更快的展示。



###### 2）多路复用

HTTP1.0：一次连接，一个request，一个response，连接断开。

HTTP1.1：一次连接，多个request，多个response（后面的请求等待前面请求的返回才能获得执行机会，一旦有某请求超时等，后续请求只能被阻塞），连接超时时间可以在服务器设置。

HTTP2.0：一次连接，多个request，多个response，并不会被阻塞（一个请求耗时很多，不会影响其他请求）。



###### 3）数据压缩

HTTP2.0支持header压缩，头部大多数时候，并不是都是有用的字段，压缩可以节省开销，也让传输变的更快。



###### 4）还有一些请求头与响应头，在不同版本也不再有效。



