---
title: vue-ssr之vue-server-renderer插件
date: 2019-12-15 20:18:47
tags:
  - vue-ssr
categories:
  - vue-ssr
---

## vue-server-renderer 简单了解

该软件包的作用是：vue2.0 提供在 node.js 服务器端呈现的。

**API**

- createRenderer

该方法是创建一个 renderer 实列。如下代码：

```javascript
const renderer = require('vue-server-renderer').createRenderer()
```

- renderer.renderToString(vm, cb);

该方法的作用是：将 Vue 实列呈现为字符串。该方法的回调函数是一个标准的 Node.js 回调，它接收错误作为第一个参数。如下代码：

```javascript
// renderer.js 代码如下：
const Vue = require('vue')

// 创建渲染器
const renderer = require('vue-server-renderer').createRenderer()

const app = new Vue({
  template: `<div>Hello World</div>`
})

// 生成预渲染的HTML字符串.  如果没有传入回调函数，则会返回 promise，如下代码

renderer
  .renderToString(app)
  .then(html => {
    console.log(html) // 输出：<div data-server-rendered="true">Hello World</div>
  })
  .catch(err => {
    console.log(err)
  })

// 当然我们也可以使用另外一种方式渲染，传入回调函数，
// 其实和上面的结果一样，只是两种不同的方式而已
renderer.renderToString(app, (err, html) => {
  if (err) {
    throw err
  }
  console.log(html)
  // => <div data-server-rendered="true">Hello World</div>
})
```

![WX20191206-163234@2x](http://114.55.30.96/WX20191206-163234@2x.png)

div 中的 data-server-rendered 属性告诉 VUE 这是服务器渲染的元素。并且应该以激活的模式进行挂载。

- createBundleRenderer(code, [rendererOptions])

Vue SSR 依赖包 vue-server-render, 它的调用支持有 2 种格式，createRenderer() 和 createBundleRenderer(), 那么 createRenderer()是以 vue 组件为入口的，而 createBundleRenderer() 以打包后的 JS 文件或 json 文件为入口的。所以 createBundleRenderer()的作用和 createRenderer() 作用是一样的，无非就是支持的入口文件不一样而已。

## 这个插件做了什么

**[从一个简单的例子看一下这个插件做了什么](https://github.com/qinhanwen/vue-ssr-demo/tree/master/demo1)**

server.js

```javascript
const Vue = require('vue')
const Koa = require('koa')
const Router = require('koa-router')
const renderer = require('vue-server-renderer').createRenderer({
  // 读取传入的template参数
  template: require('fs').readFileSync('./src/index.html', 'utf-8')
})

// 1. 创建koa koa-router实列
const app = new Koa()
const router = new Router()

// 引入 app.js
const createApp = require('./src/app')

// 2. 路由中间件

router.get('*', async (ctx, next) => {
  // 创建vue实列
  const app = createApp(ctx)

  const context = {
    title: 'vue服务器渲染组件',
    meta: `
      <meta charset="utf-8">
      <meta name="" content="vue服务器渲染组件">
    `
  }
  try {
    // 传入context 渲染上下文对象
    const html = await renderer.renderToString(app, context)
    ctx.status = 200
    ctx.body = html
  } catch (e) {
    ctx.status = 500
    ctx.body = '服务器错误'
  }
})

// 加载路由组件
app.use(router.routes()).use(router.allowedMethods())

// 启动服务
app.listen(3000, () => {
  console.log(`server started at localhost:3000`)
})
```

app.js

```javascript
const Vue = require('vue')

// 这里导出了一个 createApp方法，没有做 $mount 挂载操作，只是返回 Vue实例
module.exports = function createApp(ctx) {
  return new Vue({
    data: {
      url: ctx.url
    },
    template: `<div>访问的URL是：{{url}}</div>`
  })
}
```

index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- 三花括号不会进行html转义 -->
    {{{ meta }}}
    <title>{{title}}</title>
  </head>
  <body>
    <!--vue-ssr-outlet-->
  </body>
</html>
```

访问一下`localhost:3000`

![WX20191215-213532@2x](http://114.55.30.96/WX20191215-213532@2x.png)

1. 进入`renderer.renderToString`调用

```javascript
    renderToString: function renderToString (
      component, // vm 实例
      context, // 传入的上下文属性
      cb //
    ) {

      ...

      try {
        render(component, write, context, function (err) {
          if (err) {
            return cb(err)
          }
          if (context && context.rendered) {
            context.rendered(context);
          }
          if (template) {
            try {
              var res = templateRenderer.render(result, context);
              if (typeof res !== 'string') {
                res
                  .then(function (html) { return cb(null, html); })
                  .catch(cb);
              } else {
                cb(null, res);
              }
            } catch (e) {
              cb(e);
            }
          } else {
            cb(null, result);
          }
        });
      } catch (e) {
        cb(e);
      }

      return promise
    },
```

2. 走进 render 方法后断点往下走，在 renderNode 方法中，第一个参数是生成的 Vnode





看一下 Vue.prototype.\_render 方法

```javascript
  Vue.prototype._render = function () {
    var vm = this;
    var ref = vm.$options;
    var render = ref.render;
    var _parentVnode = ref._parentVnode;

    ...

    var vnode;
    try {
      currentRenderingInstance = vm;
      vnode = render.call(vm._renderProxy, vm.$createElement);

      /**
       * render 函数长这样
       * ƒ anonymous(
       * ) {
       * with(this){return _c('div',[_ssrNode(_ssrEscape("访问的URL是："+_s(url)))])}
       * }
       */

    } catch (e) {
      handleError(e, vm, "render");
      if (vm.$options.renderError) {
        try {
          vnode = vm.$options.renderError.call(vm._renderProxy, vm.$createElement, e);
        } catch (e) {
          handleError(e, vm, "renderError");
          vnode = vm._vnode;
        }
      } else {
        vnode = vm._vnode;
      }
    } finally {
      currentRenderingInstance = null;
    }

    ...

    return vnode
  };
```

最后生成的 Vnode

![WX20191215-222700@2x](http://114.55.30.96/WX20191215-222700@2x.png)

3. 进入 templateRenderer.render 方法调用

![WX20191215-223154@2x](http://114.55.30.96/WX20191215-223154@2x.png)

```javascript
TemplateRenderer.prototype.render = function render (content, context) {
  // result 的值是 <div data-server-rendered="true">访问的URL是：/</div>

  ...

  if (this.inject) {
    return ( // 这边生成 模板 文件
      template.head(context) +
      (context.head || '') +
      this.renderResourceHints(context) +
      this.renderStyles(context) +
      template.neck(context) +
      content +
      this.renderState(context) +
      this.renderScripts(context) +
      template.tail(context)
    )
  } else {

    ...

  }
};
```

得到的模板

![WX20191215-225232@2x](http://114.55.30.96/WX20191215-225232@2x.png)

4. 模板作为结果返回

![WX20191215-225835@2x](http://114.55.30.96/WX20191215-225835@2x.png)





**[第二个 🌰](https://github.com/qinhanwen/vue-ssr-demo/tree/master/demo2)**

创建 renderer 传入参数（ utf-8 编码格式的入口文件，客户端构建出来的 json 文件内容）

![WX20191216-193856](http://114.55.30.96/WX20191216-193856.png)

生成的 renderer

![WX20191216-200301](http://114.55.30.96/WX20191216-200301.png)

访问 `http://localhost:3000/home`

生成模板文件返回

![WX20191216-200659](http://114.55.30.96/WX20191216-200659.png)

是没有事件的，看一下前端激活

通过模板加载 js 文件，前端入口文件调用 createApp 方法

![WX20191216-201316](http://114.55.30.96/WX20191216-201316.png)

获取到 vm 实例与 router 实例

![WX20191216-201430](http://114.55.30.96/WX20191216-201430.png)

往 readyCbs 数组里添加回调

![WX20191216-204228](http://114.55.30.96/WX20191216-204228.png)

当路由准备好的时候遍历调用数组回调

![WX20191216-204417](http://114.55.30.96/WX20191216-204417.png)

进入 vm.\$mount 方法挂载实例

![WX20191216-204838](http://114.55.30.96/WX20191216-204838.png)

后面就不说了
