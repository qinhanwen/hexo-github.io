---
title: Vue源码解析（1）- 准备工作
date: 2019-08-28 13:05:04
tags: 
- Vue源码解析
categories: 
- Vue源码解析
---

## [Flow](https://flow.org/)

Flow是JavaScript静态检查工具。

在编译阶段就发现错误，而不是运行时才发现。



## 源码目录结构

Vue.js 的源码都在 src 目录下，其目录结构如下。

```
src

├── compiler        # 编译相关 

├── core            # 核心代码 

├── platforms       # 不同平台的支持

├── server          # 服务端渲染

├── sfc             # .vue 文件解析

├── shared          # 共享代码
```

**compiler** 目录包含 Vue.js 所有编译相关的代码。它包括把模板解析成 ast 语法树，ast 语法树优化，代码生成等功能。

**core** 目录包含了 Vue.js 的核心代码，包括内置组件、全局 API 封装，Vue 实例化、观察者、虚拟 DOM、工具函数等等。

**platform** 目录，Vue.js 是一个跨平台的 MVVM 框架，它可以跑在 web 上，也可以配合 weex 跑在 natvie 客户端上。platform 是 Vue.js 的入口，2 个目录代表 2 个主要入口，分别打包成运行在 web 上和 weex 上的 Vue.js。

**server** 目录，Vue.js 2.0 支持了服务端渲染，所有服务端渲染相关的逻辑都在这个目录下。注意：这部分代码是跑在服务端的 Node.js，不要和跑在浏览器端的 Vue.js 混为一谈。

**sfc** 目录，这个目录下的代码逻辑会把 .vue 文件内容解析成一个 JavaScript 的对象。

**shared** 目录，这里定义的工具方法都是会被浏览器端的 Vue.js 和服务端的 Vue.js 所共享的。



## 代码构建

[rollup](https://rollupjs.org/guide/en/)对脚本文件打包构建，类似`webpack`。区别是`webpack`可以通过配置不同的`loader`对不同类型文件处理，

在`vue`源码目录下的`package.json`文件里`scripts`里有个`"build": "node scripts/build.js",`任务，添加`--inspect-brk`参数，然后`npm run build`做了什么自己看，大概就是将一系列的js文件打包成一个。

```json
"build": "node --inspect-brk scripts/build.js",
```



## 入口文件

文件路径`src/core/instance/index.js`

```javascript
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```

这个文件做了什么

- 声明构造函数`Vue`，并且确保只能通过`new`关键字调用
- 把`Vue`当做参数传入函数调用，基本上都是为构造函数`Vue`添加原型方法