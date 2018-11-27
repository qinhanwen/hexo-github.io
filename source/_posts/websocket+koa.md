---
title: websocket+koa
date: 2018-11-05 17:54:27
tags: 
- websocket+koa
categories: 
- websocket+koa
---

# WebSocket+Koa

##### 1.TCP和HTTP的简单理解

​    Http协议建立在TCP协议基础之上，当浏览器需要从服务器获取网页数据的时候，会发一次Http请求。Http会通过TCP建立一个到服务器的连接通道，当本次请求需要的数据完毕后，Http会立即将TCP连接断开，所以Http是一种短链接。

​    当html变的复杂，嵌入了很多图片，每次访问图片都需要建立一次TCP链接就显得很低效，因此Keep-Alive被提出解决低效的问题，从HTTP1.1开始默认开启Keep-Alive，保持连接特性。总的说就是网页打开之后，客户端与服务器之间传输Http数据的TCP连接不会关闭，客户端再次访问服务器上的网页，会继续使用这条。时间是有限制范围的，在不同服务器也可以设置超时时间。



##### 2.**HTTPS与HTTP的一些区别**

​    HTTPS协议需要到CA申请证书，一般免费证书很少，需要交费。

​    HTTP协议运行在TCP之上，所有传输的内容都是明文，HTTPS运行在SSL/TLS之上，SSL/TLS运行在TCP之上，所有传输的内容都经过加密的。

​    HTTP和HTTPS使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。

​    HTTPS可以有效的防止运营商劫持，解决了防劫持的一个大问题。



##### 3.Http1.0，Http1.1和Http2.0的区别

​    HTTP/1.0 一次请求-响应，建立一个连接，用完关闭；每一个请求都要建立一个连接；

​    HTTP/1.1 解决方式为，若干个请求排队串行化单线程处理，后面的请求等待前面请求的返回才能获得执行机会，一旦有某请求超时等，后续请求只能被阻塞，毫无办法，也就是人们常说的线头阻塞；

​    HTTP/2多个请求可同时在一个连接上并行执行。某个请求任务耗时严重，不会影响到其它连接的正常执行。headers压缩。



##### 4.什么是WebSocket

​    WebSocket是HTML5的。使客户端与服务器之间数据交换变的更简单，允许服务端主动向客户端推送数据，在WebSocket API中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。

​    浏览器通过JavaScript向服务器发起建立WebSocket连接的请求，连接建立以后，客户端和服务端可以通过TCP连接直接交换数据。当你获取WebSocket连接后，可以通过send()方法向服务端发送数据，并通过onmessage事件接收服务器返回的数据。

##### 5.具体实现

###### 前端部分

```html
<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <title>websocket</title>
</head>
<body>
    <input type="text" id="pl" />
    <input type="button" value="发送" id="submit" />
    <input type="button" value="关闭" id="close" />
</body>
<script type="text/javascript">
  // 很重要 必须写，判断浏览器是否支持websocket
    var CreateWebSocket = (function () {
        return function (urlValue) {
            if (window.WebSocket) return new WebSocket(urlValue);
            if (window.MozWebSocket) return new MozWebSocket(urlValue);
            return false;
        }
    })()
   // 实例化websoscket websocket有两种协议ws(不加密)和wss(加密)
    var webSocket = CreateWebSocket("ws://127.0.0.1:3000");
    webSocket.onopen = function (evt) {
        // 一旦连接成功，就发送第一条数据
         webSocket.send("第一条数据")
    }
    webSocket.onmessage = function (evt) {
        // 这是服务端返回的数据
        console.log("服务端说" + evt.data)
    }
    // 关闭连接
    webSocket.onclose = function (evt) {
        console.log("Connection closed.")
    }
    // input事件发送数据
    document.getElementById("submit").onclick = function () {
        var str = document.getElementById("pl").value
        webSocket.send(str)
    }
</script>
</html>

```

###### koa部分

```javascript

const Koa = require('koa')
// 路由
const route = require('koa-route')
// koa封装的websocket这是官网（很简单有时间去看一下https://www.npmjs.com/package/koa-websocket）
const websockify = require('koa-websocket')
const app = websockify(new Koa());
app.ws.use(function (ctx, next) {
    return next(ctx)
})
app.ws.use(route.all('/', function (ctx) {
    ctx.websocket.on('message', function (message) {
        // 返回给前端的数据
        ctx.websocket.send(message)
    })
}))
app.listen(3000)
```



具体参考[阮大师的文章](http://www.ruanyifeng.com/blog/2017/05/websocket.html)







