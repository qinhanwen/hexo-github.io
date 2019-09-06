---
title: npm scripts
date: 2019-07-08 22:51:51
tags: 
- npm
categories: 
- npm
---

> [npm scripts](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)
>
> https://blog.51cto.com/sunspot/1265650
>
> [Node.js process](https://www.jianshu.com/p/a8c8d435fa62)

## npm脚本

npm 允许在`package.json`文件里面，使用`scripts`字段定义脚本命令。

```javascript
... 
"scripts": {
    "build": "node build.js"
  }
...
```

通过`npm run <command> [--<args>]`执行，每当执行`npm run xxx`，就会自动新建一个 Shell，这个 Shell会将当前目录的`node_modules/.bin`子目录加入`PATH`变量，执行结束后，再将`PATH`变量恢复原样。这意味着，当前目录的`node_modules/.bin`子目录里面的所有脚本，都可以直接用脚本名调用，而不必加上路径。

为什么将当前目录的`node_modules/.bin`子目录加入`PATH`变量？

因为命令执行的时候会在PATH环境变量里包含的路径中去寻找相同名字的可执行文件。局部安装的包只在`./node_modules/.bin`目录下注册了它们的可执行文件，不会被包含在`PATH`环境变量中，例如：

```shell
$ webpack 
#zsh: command not found: webpack

$ node_modules/.bin/webpack
```

![WX20190701-125659@2x](http://www.qinhanwen.xyz/WX20190701-125659@2x.png)



两种方式运行env（查询环境变量）

```shell
$ env
#PATH=/Library/Frameworks/Python.framework/Versions/3.7/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Users/qinhanwen/.nvm/versions/node/v6.11.2/bin:/Library/Frameworks/Python.framework/Versions/3.7/bin
```

```shell
$ npm run env
#PATH=/Users/qinhanwen/.nvm/versions/node/v6.11.2/lib/node_modules/npm/bin/node-gyp-bin:/Users/qinhanwen/npm-study-qhw/node_modules/.bin:/Library/Frameworks/Python.framework/Versions/3.7/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Users/qinhanwen/.nvm/versions/node/v6.11.2/bin:/Library/Frameworks/Python.framework/Versions/3.7/bin
```

可以看到运行时的`PATH`环境变量多了两个路径：`npm`指令路径和项目中`node_modules/.bin`的绝对路径。所以，通过`npm run`可以在不添加路径前缀的情况下直接访问当前项目`node_modules/.bin`目录里面的可执行文件。



## 通配符

由于 npm 脚本就是 Shell 脚本，因为可以使用 Shell 通配符。

```json
  ...
  "scripts": {
    "lint": "jshint *.js"
  },
  ...
```

jshint：代码检查工具，`npm run lint`执行脚本，会检查根目录下的js文件 

![WX20190710-095011@2x](http://www.qinhanwen.xyz/WX20190710-095011@2x.png)



## 传参

向 npm 脚本传入参数，要使用`--`标明。

```json
	...
	"scripts": {
    "argv": "node index.js --qinhanwen"
  },
  ...
```

index.js

```javascript
 console.log(process.argv);
```

![WX20190710-110522@2x](http://www.qinhanwen.xyz/WX20190710-110522@2x.png)

**process **是一个全局变量，即 global 对象的属性。它用于描述当前 Node.js 进程状态的对象，提供了一个与操作系统的简单接口。

**process.argv**： 读取命令行参数。第一个元素是’node’，第二个元素是JavaScript文件的文件名。接下来的元素则是附加的命令行参数



## 执行顺序

如果 npm 脚本里面需要执行多个任务，那么需要明确它们的执行顺序。

如果是并行执行（即同时的平行执行），可以使用`&`符号。

> ```bash
> $ npm run script1.js & npm run script2.js
> ```

如果是继发执行（即只有前一个任务成功，才执行下一个任务），可以使用`&&`符号。

> ```bash
> $ npm run script1.js && npm run script2.js
> ```



## 钩子

npm 脚本有`pre`和`post`两个钩子。比如，`start`脚本命令的钩子就是`prestart`和`poststart`。

```json
  ...
  "scripts": {
    "prestart": "node index2.js",
    "start": "node index",
    "poststart": "node index3.js"
  },
  ...
```

执行`npm run start`，等于

```shell
$ npm run prestart && npm run start && npm run poststart
```



可以通过`process.env.npm_lifecycle_event`获取当前正在运行的脚本名称。



npm 默认提供下面这些钩子：

- prepublish，postpublish
- preinstall，postinstall
- preuninstall，postuninstall
- preversion，postversion
- pretest，posttest
- prestop，poststop
- prestart，poststart
- prerestart，postrestart



## 简写

```shell
$ npm start 
#等价于
$ npm run start
```



## 变量

通过`npm_package_`前缀，npm 脚本可以拿到`package.json`里面的字段。

```javascript
console.log(process.env.npm_package_name);//ls
console.log(process.env.npm_package_version);//1.0.0
//获取嵌套字段
console.log(process.env.npm_package_scripts_lint);//jshint *.js
```















