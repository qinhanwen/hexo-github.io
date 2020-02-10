---
title: cli学习
date: 2020-01-14 22:11:15
tags: 
- cli
categories: 
- cli
---

## 说明

项目想要根据选择来实现初始化不同的模板，之前的模板太重了。



## 如何实现 cli 插件

可能使用的依赖介绍：

- commander：node.js 命令行接口的完整解决方案，自定义命令
- chalk：修改控制台中字符串的样式（字体样式加粗等／字体颜色／背景颜色）
- cross-spawn：Node.js 的子进程 child_process 模块下有一 spawn 函数，可以用于调用系统上的命令，如在 Linux, macOS 等系统上。这个库是用来兼容多平台
- fs-extra：文件操作
- path：这个模块提供用于处理文件路径和目录路径的实用工具
- os：该模块提供了与操作系统相关的实用方法和属性
- ora：爱的魔力转圈圈
- download-git-repo： 一个用于下载git仓库的项目的模块
- inquirer：与用户交互



## 开始

**新建目录 my-cli** 

```shell
$ mkdir my-cli && cd my-cli
```



**新建 index.js**

```javascript
//index.js
console.log('index.js');
```

执行 node index ，来运行这个 js 文件



**使用自定义命令执行这个文件**

使用`npm init`创建`package.json`，可以选择添加配置信息，也可以选择跳过。并在文件里添加 bin 字段，用来存放一个可执行的文件。

```json
{
  "name": "my-cli",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "bin":{
    "my-cli":"./index.js"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```



**在 index.js 文件首部加上shebang（解释行）**

```javascript
#!/usr/bin/env node
console.log('index.js');
```

本地调试可以把当前项目通过 npm link 创建软链，添加到全局环境，之后就可以通过 shell 来运行这个文件

![WX20200114-232953@2x](http://114.55.30.96/WX20200114-232953@2x.png)



## 了解一下 vue-cli 怎么做的

在 vue 的可执行文件的 shebang 后面添加 --inspect-brk

![WX20200115-123619@2x](http://114.55.30.96/WX20200115-123619@2x.png)

之后执行

```shell
$ vue create hello-world
```



[Commander 介绍](https://www.cnblogs.com/mirandachen/p/9826886.html)

- command：自定义执行的命令
- option：可选参数
- alias：用于 执行命令的别名
- description：命令描述
- action：执行命令后所执行的方法
- usage：用户使用提示
- parse：解析命令行参数，**注意这个方法一定要放到最后调用**



这里定义了很多的自定义执行命令

![WX20200115-125607](http://114.55.30.96/WX20200115-125607.png)



之后解析命令行参数

![WX20200115-130722](http://114.55.30.96/WX20200115-130722.png)



进入 create 的 action ，加载 create 并且执行

![WX20200115-162601](http://114.55.30.96/WX20200115-162601.png)



实例化 Creator，并且调用 creator.create

![WX20200115-162911](http://114.55.30.96/WX20200115-162911.png)



之后调用 inquirer.prompt(this.resolveFinalPrompts())，出选项，与用户交互。返回的是一个 promise 对象，用户选择后得到结果，继续往下执行（我选了 default）

![WX20200115-163310](http://114.55.30.96/WX20200115-163310.png)



准备写 package.json 文件

![WX20200115-164517](http://114.55.30.96/WX20200115-164517.png)



通过 fs 文件系统写

![WX20200115-164620](http://114.55.30.96/WX20200115-164620.png)









## 参考资料

https://juejin.im/post/5cc160b2f265da03452bdf5b

https://juejin.im/post/5b592db551882536e5178ce6

https://www.cnblogs.com/mirandachen/p/9826886.html

https://www.jianshu.com/p/9ff49cb88204