---
title: npm install后发生了什么
date: 2019-07-11 18:54:12
tags: 
- npm
categories: 
- npm
---

> [npm install 之后发生了什么？](https://zhuanlan.zhihu.com/p/37285173)
>
> [记一次排错经历——npm缓存浅析](https://juejin.im/post/5c77e05e518825407a32b94b)
>
> [npm 模块安装机制简介](http://www.ruanyifeng.com/blog/2016/01/npm-install.html)
>
> [介绍下 npm 模块安装机制，为什么输入 npm install 就可以自动安装对应的模块？](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/22)



## npm install是干嘛的

`npm install`命令用来安装模块到`node_modules`目录。

安装之前，`npm install`会先检查`node_modules`目录之中是否已经存在指定模块。如果存在，就不再重新安装了，即使远程仓库已经有了一个新版本，也不会安装。

如果希望一个模块不管是否安装过，npm 都要强制重新安装，可以使用`-f`或`--force`参数。

```shell
$ npm install <packageName> --force
```



```shell
$ npm install --verbose
```

运行命令，会把安装过程的log输出。	

![WX20190808-134108@2x](http://114.55.30.96/WX20190808-134108@2x.png)



## registry

npm 模块仓库提供了一个查询服务，叫做 `registry`，npmjs.org 的查询服务网址是[https://registry.npmjs.org/](https://registry.npmjs.org/)

后面可以加上`npm包名`，比如：[https://registry.npmjs.org/npm-study4](https://registry.npmjs.org/npm-study4)，打开地址是一个JSON 对象，里面是该模块所有版本的信息。`dist.tarball`属性是该版本压缩包的网址，打开这个网址就可以下载npm压缩包，在本地解压，就可以获得源码，`npm install`和`npm update`命令，都是通过这种方式安装模块。

![WX20190709-102920@2x](http://114.55.30.96/WX20190709-102920@2x.png)



## npm update

```shell
$ npm update <packageName>
```

它会先到远程仓库查询最新版本，然后查询本地版本。如果本地版本不存在，或者远程版本较新，就会安装。



## 缓存

```shell
#获取目录的具体位置
$ npm config get cache
#/Users/qinhanwen/.npm

#清除缓存目录文件
$ npm cache clean -f
```

要注意在 **npm@5** 之后

- 缓存数据放在 `.npm/_cacache` 文件夹中
- 清除命令清除的是` .npm/_cacache` 目录下文件

那么看一下`.npm/_cacache`目录下

![WX20190711-192302@2x](http://114.55.30.96/WX20190711-192302@2x.png)

在`content-v2` 里面基本都是一些二进制文件，把二进制文件的扩展名改为 `.tgz` 再解压之后，会发现就是在我们熟知的npm包。 

![WX20190711-212439@2x](http://114.55.30.96/WX20190711-212439@2x.png)

而`index-v5` 里面是一些描述性的文件，也是 `content-v2` 里文件的索引，有点像HTTP的响应头，而且还有缓存相关的值

![WX20190711-213336@2x](http://114.55.30.96/WX20190711-213336@2x.png)



## 所以`npm install`之后发生了什么

**先了解一下shell的`解释行（Shebang）`是什么**

**Shebang**（也称为**Hashbang**）是一个由井号和叹号构成的字符串行（*#!*），其出现在文本文件的第一行的前两个字符。

如下列出了一些典型的shebang解释器指令：

```
#!/bin/sh—使用sh，即Bourne shell或其它兼容shell执行脚本
#!/bin/csh—使用csh，即C shell执行
#!/usr/bin/perl -w—使用带警告的Perl执行
#!/usr/bin/python -O—使用具有代码优化的Python执行
#!/usr/bin/php—使用PHP的命令行解释器执行
```

通常出现在linux的shell脚本第一行，作为解释行，告诉解释器shell的执行方式。



在`npm`入口文件后面添加`--inspect-brk`就可以断点调试

```javascript
#!/usr/bin/env node  --inspect-brk
```

入口文件在：

![WX20190808-134156@2x](http://114.55.30.96/WX20190808-134156@2x.png)



 前置条件：**npm版本6.4.1**，**不存在package-lock.json**，以及**没有缓存文件存在**的情况。

1.入口文件进入

2.加载install.js

![WX20190805-224325@2x](http://114.55.30.96/WX20190805-224325@2x.png)

![WX20190712-094454@2x](http://114.55.30.96/WX20190712-094454@2x.png)

```javascript
function Installer (where, dryrun, args, opts) {
  validate('SBA|SBAO', arguments)
  if (!opts) opts = {}
  this.where = where
  this.dryrun = dryrun
  this.args = args
  // fakechildren are children created from the lockfile and lack relationship data
  // the only exist when the tree does not match the lockfile
  // this is fine when doing full tree installs/updates but not ok when modifying only
  // a few deps via `npm install` or `npm uninstall`.
  this.currentTree = null
  this.idealTree = null
  this.differences = []
  this.todo = []
  this.progress = {}
  this.noPackageJsonOk = !!args.length
  this.topLevelLifecycles = !args.length

  this.autoPrune = npm.config.get('package-lock')

  const dev = npm.config.get('dev')
  const only = npm.config.get('only')
  const onlyProd = /^prod(uction)?$/.test(only)
  const onlyDev = /^dev(elopment)?$/.test(only)
  const prod = npm.config.get('production')
  this.dev = opts.dev != null ? opts.dev : dev || (!onlyProd && !prod) || onlyDev
  this.prod = opts.prod != null ? opts.prod : !onlyDev

  this.packageLockOnly = opts.packageLockOnly != null
    ? opts.packageLockOnly : npm.config.get('package-lock-only')
  this.rollback = opts.rollback != null ? opts.rollback : npm.config.get('rollback')
  this.link = opts.link != null ? opts.link : npm.config.get('link')
  this.saveOnlyLock = opts.saveOnlyLock
  this.global = opts.global != null ? opts.global : this.where === path.resolve(npm.globalDir, '..')
  this.audit = npm.config.get('audit') && !this.global
  this.started = Date.now()
}
```

```javascript
Installer.prototype.run = function (_cb) {
  //进行一些校验
  ...
  //将安装的步骤放入队列里面
  var installSteps = []
  var postInstallSteps = []
  if (!this.dryrun) {
    installSteps.push(
      [this.newTracker(log, 'runTopLevelLifecycles', 2)],
      [this, this.runPreinstallTopLevelLifecycles])
  }
  installSteps.push(
    [this.newTracker(log, 'loadCurrentTree', 4)],
    [this, this.loadCurrentTree],
    [this, this.finishTracker, 'loadCurrentTree'],

    [this.newTracker(log, 'loadIdealTree', 12)],
    [this, this.loadIdealTree],
    [this, this.finishTracker, 'loadIdealTree'],

    [this, this.debugTree, 'currentTree', 'currentTree'],
    [this, this.debugTree, 'idealTree', 'idealTree'],

    [this.newTracker(log, 'generateActionsToTake')],
    [this, this.generateActionsToTake],
    [this, this.finishTracker, 'generateActionsToTake'],

    [this, this.debugActions, 'diffTrees', 'differences'],
    [this, this.debugActions, 'decomposeActions', 'todo'],
    [this, this.startAudit]
  )

  if (this.packageLockOnly) {
    postInstallSteps.push(
      [this, this.saveToDependencies])
  } else if (!this.dryrun) {
    installSteps.push(
      [this.newTracker(log, 'executeActions', 8)],
      [this, this.executeActions],
      [this, this.finishTracker, 'executeActions'])
    var node_modules = path.resolve(this.where, 'node_modules')
    var staging = path.resolve(node_modules, '.staging')
    postInstallSteps.push(
      [this.newTracker(log, 'rollbackFailedOptional', 1)],
      [this, this.rollbackFailedOptional, staging, this.todo],
      [this, this.finishTracker, 'rollbackFailedOptional'],
      [this, this.commit, staging, this.todo],

      [this, this.runPostinstallTopLevelLifecycles],
      [this, this.finishTracker, 'runTopLevelLifecycles']
    )
    if (getSaveType()) {
      postInstallSteps.push(
        [this, (next) => { computeMetadata(this.idealTree); next() }],
        [this, this.pruneIdealTree],
        [this, this.debugLogicalTree, 'saveTree', 'idealTree'],
        [this, this.saveToDependencies])
    }
  }
  postInstallSteps.push(
    [this, this.printWarnings],
    [this, this.printInstalled])

  var self = this
  //到这里才真正开始执行
  chain(installSteps, function (installEr) {
    if (installEr) self.failing = true
    chain(postInstallSteps, function (postInstallEr) {
      if (installEr && postInstallEr) {
        var msg = errorMessage(postInstallEr)
        msg.summary.forEach(function (logline) {
          log.warn.apply(log, logline)
        })
        msg.detail.forEach(function (logline) {
          log.verbose.apply(log, logline)
        })
      }
      cb(installEr || postInstallEr, self.getInstalledModules(), self.idealTree)
    })
  })
  return result
}
```

3.经历install的生命周期

在各种生命周期中做了的一些事情：

读根目录下`package.json`文件，确定依赖树

![WX20190712-105741@2x](http://114.55.30.96/WX20190712-105741@2x.png)

获取元信息

![WX20190712-140123@2x](http://114.55.30.96/WX20190712-140123@2x.png)

获取包信息，就是要安装的包的数据

![WX20190712-163503@2x](http://114.55.30.96/WX20190712-163503@2x.png)

内容：

![WX20190712-164143@2x](http://114.55.30.96/WX20190712-164143@2x.png)

下载的`.tgz`后缀的压缩包，比如`http://registry.npmjs.org/npm-study1/-/npm-study1-2.0.0.tgz`，保存到`.npm/_cacache`文件夹，解压到`node_modules`目录下，创建`package-lock.json`文件。



**补充：**

当运行`npm install`的时候，会检查`node_modules`目录下，并不是`node_modules`目录下有依赖包，就不安装了（情况比较多）。

如果`node_modules`目录下没有这个文件，缓存里有的话，据我对打印出log的观察，应该是从缓存里解压到`node_modules`目录下。



那么哪些操作会更新`package.json`或更新`package-lock.json`，同时更新`node_modules`下的依赖包？

当`node_modules`下不存在某依赖包:

`npm init`初始化后，`npm install xxx`（高版本的npm）会自动更新`package.json`文件，并且在最后创建`package-lock.json`文件，同时更新`node_modules`下的依赖包。



当`node_modules`下存在某依赖包:

1）`^`版本号情况下， `package.json`中某个依赖包的小版本要**大于**`package-lock.json`中的同个依赖包小版本，

`npm install`的时候，会修改`package-lock.json`文件，同时更新`node_modules`下的依赖包（安装某依赖包为当前大版本下的最新版本）。 **小于**的话，更新`node_modules`下的某依赖包，版本号与`package-lock.json`版本号一致。

2）版本号具体到某个版本的话，只要和`node_modules`下的依赖包版本不同，就会安装。

2）`npm update packageName`，更新`package.json`文件，同时更新`node_modules`下的某依赖包，版本号为当前依赖包大版本号下最新的版本（比如2.1.0就升到2.9.0，不会超过3.0.0）。

4）或者`npm install xxx@x.x.x`，更新`package.json`文件，更新`package-lock.json`文件，更新`node_modules`

5）`npm uninstall packageName <args>`，更新`package.json`文件，更新`package-lock.json`文件，更新`node_modules`







