---
title: webpack入门（1）-介绍
date: 2019-06-25 09:24:48
tags: 
- webpack
categories: 
- webpack
---

## 介绍

webpack 是一个前端`模块`打包工具。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。



## 安装

先初始化一个npm项目

```shell
$ npm init
```



4.X之后需要安装webpack和webpack-cli两个模块

1）项目里安装

```shell
$ npm install webpack webpack-cli  --save-dev
```



2）全局安装

```shell
$ npm install webpack webpack-cli  -g
```



## 模块打包

新建文件夹，文件目录

```文件目录
  |—— a.js
  |—— b.js
  |—— c.js
  |—— index.html
  |—— index.js
  |—— package-lock.json
  |—— package.json
```

index.js

```javascript
import A from './a';
import B from './b';
import C from './c';

new A();
new B();
new C();
```

a.js

```javascript
export default function A(){
    console.log("module A");
}
```

b.js

```javascript
export default function B(){
    console.log("module B");
}
```

c.js

```javascript
export default function C(){
    console.log("module C");
}
```

index.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    index.html
</body>
<script src="./index.js">

</script>

</html>
```

打开页面的时候会看到报错，（uncaught SyntaxError: Unexpected identifier）意外的标识符，浏览器不认识ES模块引入和导出的方式。

![WX20190625-200220@2x](http://118.24.241.76/WX20190625-200220@2x.png)





使用webpack打包

```shell
$ npx webpack index.js
```

[npx](http://www.ruanyifeng.com/blog/2019/02/npx.html)运行的时候会自动查找当前依赖包中的可执行文件，会到`node_modules/.bin`路径和环境变量`$PATH`里面，检查命令是否存在。由于 npx 会检查环境变量`$PATH`，所以系统命令也可以调用。



打包后的文件在dist/main.js里

```
  |—— a.js
  |—— b.js
  |—— c.js
  |—— dist
    |—— main.js
  |—— index.html
  |—— index.js
```

在index.html页面中引入dist/main.js

![WX20190625-220157@2x](http://118.24.241.76/WX20190625-220157@2x.png)



webpack也可以打包使用其他模块的规范的文件，比如commonJs，AMD，CMD。webpack它将根据模块的依赖关系进行静态分析，最后生成一个最终的js文件。（它也可以处理其他类型文件）

修改a.js，b.js，c.jx和index.js

```javascript
module.exports = function A(){
    console.log("module A");
}
```

```javascript
module.exports = function B() {
    console.log("module B");
}
```

```javascript
module.exports = function C() {
    console.log("module C");
}
```

```javascript
const A = require('./a');
const B = require('./b');
const C = require('./c');

new A();
new B();
new C();
```

重新打包

```shell
$ npm webpack index.js
```

打开页面

![WX20190625-222729@2x](http://118.24.241.76/WX20190625-222729@2x.png)





