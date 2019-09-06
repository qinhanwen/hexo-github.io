---
title: 大前端网络专题篇之HTTP(3)
date: 2019-06-09 21:34:55
tags: 
- http
categories: 
- http
---

# 大前端网络专题篇之HTTP

1️⃣、什么是跨域，如何解决跨域

什么是跨域：跨域是由浏览器的同源策略造成的，是浏览器对JavaScript实施的安全限制。

同源策略限制了以下行为：

- Cookie、LocalStorage 和 IndexDB 无法读取
- DOM 和 JS 对象无法获取
- Ajax请求发送不出去



跨域通信方式：

​	1.JSONP

​	2.CORS

​	3.NGINX

​	4.WebSocket

​	5.Postmessage

​	6.window.name

​	7.location.hash

​	8.document.domain



2️⃣、HTTP 中与缓存相关的头部有哪些，它们有什么区别

​		

​		Expires：缓存资源失效时间，优先级比`Cache-Control`的`max-age`低，HTTP1.1之前的字段。

​	

​		ETag：资源的唯一标识，资源被更新，ETag也会改变。Etag可以根据文件内容实体的hash运算生成。

​		If-None-Match：上一次请求返回的ETag字段（文件在服务器上的唯一标识）的值，如果请求资源被修改返回状态码200，否则返回304

​		If-Match：与`If-None-Match`相反。上一次请求返回的ETag字段（文件在服务器上的唯一标识）的值，如果一样才返回200。不一样的话返回412 Precondition Failed（先决条件失败）意味着对于目标资源的访问请求被拒绝（就是说字段一致，服务器才接受请求）。

​		

​		Last-Modified：资源最终修改时间

​		If-Modified-Since：服务器上文件的最后修改时间，其实就是上一次请求返回的Last-Modified的值。优先级小于`If-None-Match`。如果请求资源被修改返回状态码200，否则返回304。

​		If-Unmodified-Since：与`If-Modified-Since`相反，如果一样的话返回200。不一样的话返回412 Precondition Failed（先决条件失败）意味着对于目标资源的访问请求被拒绝（就是说字段一致，服务器才接受请求）。

​		

​		Cache-Control：

​		**public**：表明其他用户也可以利用缓存

​		**private**：缓存只能提供给某个用户利用

​		**no-cache**：

​		 1）在请求报文中：不接收缓存过的响应，`缓存服务器`会将请求发送给`源服务器`获取资源（就是每次请求都需要服务器验证）。

​		 2）在响应报文中：不是不缓存，告诉客户端下次请求资源的时候，会发送请求给`源服务器`，判断文件是否修改。

​		**max-age=6000**（单位:秒）：

​		1）客户端发送的的请求中如果带有max-age字段，缓存指定的时间比max-age字段所带的数值小的话，就会从缓存服务器返回。如果`max-age=0`，就需要将请求发送给`源服务器`。HTTP1.1版本缓存服务器中，max-age优先级大于Expires，HTTP1.0相反。

​		2）在响应报文中的时候，表示不超过这段时间内，`缓存服务器`就不会发送请求给`源服务器`。

​		**s-maxage**：代理缓存相关



3️⃣、什么是强缓存

​	不会向服务器发送请求，直接从缓存中读取资源，在chrome控制台的network选项中可以看到该请求返回200的状态码。分为 from disk cache 和 from memory cache 

**from disk cache ：一般非脚本会存在内存当中，如css，html等**

**from memory cache ：一般脚本、字体、图片会存在内存中**



有缓存命中和缓存未命中状态

![http://www.qinhanwen.xyz/WX20181218-182944@2x.png](http://www.qinhanwen.xyz/WX20181218-182944@2x.png)

![](http://www.qinhanwen.xyz/WX20181218-185000@2x.png)



4️⃣、什么是协商缓存

​	向服务器发送请求，服务器会根据这个请求的request header的一些参数来判断是否命中协商缓存。

​	有缓存命中和缓存未命中状态

![](http://www.qinhanwen.xyz/WX20181218-232303@2x.png)



​		![](http://www.qinhanwen.xyz/WX20181219-123144@2x.png)



![WX20190616-213040@2x](http://www.qinhanwen.xyz/WX20190616-213040@2x.png)

5️⃣、请简单介绍一下 LRU （Least recently used）算法

LRU算法的设计原则是：**如果一个数据在最近一段时间没有被访问到，那么在将来它被访问的可能性也很小**。也就是说，当限定的空间已存满数据时，应当把最久没有被访问到的数据淘汰。

实现方式：

1）最常见的实现是使用一个链表保存缓存数据，如下图

![](http://www.qinhanwen.xyz/WX20190615-144815@2x.png)



2）新加入数据插入表头

3）当数据被访问的时候，将数据移动到表头

4）当链表满了的时候，将链表尾部数据丢弃

























