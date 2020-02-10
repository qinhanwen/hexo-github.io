---
title: react源码过程（1）- 准备工作
date: 2019-10-01 23:50:41
tags:
  - react源码过程
categories:
  - react源码过程
---

## 准备工作

首先先到`github`下载最新的源码包

```shell
$ yarn install
```

在安装模块的时候遇到报错

![WX20191009-221629@2x](http://114.55.30.96/WX20191009-221629@2x.png)

```shell
$ yarn cache clean
```

之后再安装

## 启动

```shell
$ npm run build
```

起个服务打开`/fixtures/packaging/`，选择个调试的。建议选择`fixtures/packaging/babel-standalone/dev.html`比较好用

## 了解

- requestAnimationFrame
- requestIdleCallback
- window.postMessage

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Document</title>
  </head>

  <body>
    <div onclick="send()">clickMe</div>
    <script>
      window.addEventListener('message', idleTick, false)

      function idleTick(e) {
        console.log(e)
      }

      function send() {
        window.requestAnimationFrame(() => {
          console.log('requestAnimationFrame')
        })
        setTimeout(() => {
          console.log('timeout')
        })
        window.postMessage(123, '*')
        Promise.resolve('resolve').then(data => {
          console.log(data)
        })
      }
      //点击事件输出结果
      //resolve
      //requestAnimationFrame
      //123
      //timeout
    </script>
  </body>
</html>
```

## 参考资料

https://react.jokcy.me/

![scheduler-fiber-scheduler](http://114.55.30.96/scheduler-fiber-scheduler.png)
