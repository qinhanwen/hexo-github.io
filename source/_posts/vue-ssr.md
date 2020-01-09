---
title: vue-ssr-1
date: 2019-11-20 22:15:27
tags: 
- vue-ssr
categories: 
- vue-ssr
---

## 什么是服务器端渲染 (SSR)

Vue.js 是构建客户端应用程序的框架。默认情况下，可以在浏览器中输出 Vue 组件，进行生成 DOM 和操作 DOM。然而，也可以将同一个组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将这些静态标记"激活"为客户端上完全可交互的应用程序。

服务器渲染的 Vue.js 应用程序也可以被认为是"同构"或"通用"，因为应用程序的大部分代码都可以在**服务器**和**客户端**上运行。



## 为什么使用服务器端渲染 (SSR)

优势：

- 更好的 SEO，由于搜索引擎爬虫抓取工具可以直接查看完全渲染的页面。
- 更利于首屏渲染，首屏的渲染是node发送过来的html字符串，并不依赖于js文件了，这就会使用户更快的看到页面的内容。尤其是针对大型单页应用，打包后文件体积比较大，普通客户端渲染加载所有所需文件时间较长，首页就会有一个很长的白屏等待时间。



劣势：

- 开发条件所限。浏览器特定的代码，只能在某些生命周期钩子函数 (lifecycle hook) 中使用；一些外部扩展库 (external library) 可能需要特殊处理，才能在服务器渲染应用程序中运行。
- 涉及构建设置和部署的更多要求。与可以部署在任何静态文件服务器上的完全静态单页面应用程序 (SPA) 不同，服务器渲染应用程序，需要处于 Node.js server 运行环境。
- 更多的服务器端负载。
- 学习成本高，除了对webpack、Vue要熟悉，还需要掌握node等相关技术。相对于客户端渲染，项目构建、部署过程更加复杂。



## 服务端渲染与客户端渲染的区别

服务端渲染与客户端渲染的本质区别是谁来渲染html页面，如果html页面在服务器端那边拼接完成后，那么它就是服务器端渲染，而如果是前端做的html拼接及渲染的话，那么它就属于客户端渲染的。



## 构建

**构建图**

![WX20191126-211216@2x](http://118.24.241.76/WX20191126-211216@2x.png)



**app.js入口文件**

app.js是我们的通用entry，它的作用就是构建一个Vue的实例以供服务端和客户端使用，注意一下，在纯客户端的程序中我们的app.js将会挂载实例到dom中，而在ssr中这一部分的功能放到了Client entry中去做了。



**两个entry**

接下里我们来看Client entry和Server entry，这两者分别是客户端的入口和服务端的入口。**Client entry的功能很简单，就是挂载我们的Vue实例到指定的dom元素上**；Server entry是一个使用export导出的函数。主要负责调用组件内定义的获取数据的方法，获取到SSR渲染所需数据，并存储到上下文环境中。**这个函数会在每一次的渲染中重复的调用**。



**webpack打包构建**

然后我们的服务端代码和客户端代码通过webpack分别打包，生成Server Bundle和Client Bundle，前者会运行在服务器上通过node生成预渲染的HTML字符串，发送到我们的客户端以便完成初始化渲染；而客户端bundle就自由了，初始化渲染完全不依赖它了。客户端拿到服务端返回的HTML字符串后，会去“激活”这些静态HTML，是其变成由Vue动态管理的DOM，以便响应后续数据的变化。



##  vue-server-renderer

该软件包的作用是：vue2.0提供在node.js 服务器端呈现的。、



**API**

- createRenderer

该方法是创建一个renderer实列。如下代码：

```javascript
const renderer = require('vue-server-renderer').createRenderer();
```



- renderer.renderToString(vm, cb);

该方法的作用是：将Vue实列呈现为字符串。该方法的回调函数是一个标准的Node.js回调，它接收错误作为第一个参数。如下代码：

```javascript
// renderer.js 代码如下：
const Vue = require('vue');

// 创建渲染器
const renderer = require('vue-server-renderer').createRenderer();

const app = new Vue({
    template: `<div>Hello World</div>`
});

// 生成预渲染的HTML字符串.  如果没有传入回调函数，则会返回 promise，如下代码

renderer.renderToString(app).then(html => {
    console.log(html); // 输出：<div data-server-rendered="true">Hello World</div>
}).catch(err => {
    console.log(err);
});

// 当然我们也可以使用另外一种方式渲染，传入回调函数，
// 其实和上面的结果一样，只是两种不同的方式而已
renderer.renderToString(app, (err, html) => {
    if (err) {
        throw err;
    }
    console.log(html)
    // => <div data-server-rendered="true">Hello World</div>
})
```

![WX20191206-163234@2x](http://118.24.241.76/WX20191206-163234@2x.png)

div中的data-server-rendered属性告诉VUE这是服务器渲染的元素。并且应该以激活的模式进行挂载。



- createBundleRenderer(code, [rendererOptions])

Vue SSR依赖包 vue-server-render, 它的调用支持有2种格式，createRenderer() 和 createBundleRenderer(), 那么createRenderer()是以vue组件为入口的，而 createBundleRenderer() 以打包后的JS文件或json文件为入口的。所以createBundleRenderer()的作用和 createRenderer() 作用是一样的，无非就是支持的入口文件不一样而已。



## 与服务器集成

服务器端渲染，需要用到  vue-server-renderer 组件包。该包的基本的作用是拿到vue实列并渲染成html结构

server.js

```javascript
const Vue = require('vue');
const Koa = require('koa');
const Router = require('koa-router');
const renderer = require('vue-server-renderer').createRenderer();

// 1. 创建koa koa-router实列

const app = new Koa();
const router = new Router();

// 2. 路由中间件

router.get('*', async(ctx, next) => {
  // 创建vue实列
  const app = new Vue({
    data: {
      url: ctx.url
    },
    template: `<div>访问的URL是：{{url}}</div>`
  })
  try {
    // vue 实列转换成字符串
    const html = await renderer.renderToString(app);
    ctx.status = 200;
    ctx.body = `
      <!DOCTYPE html>
      <html>
        <head><title>vue服务器渲染组件</title></head>
        <body>${html}</body>
      </html>
    `
  } catch(e) {
    console.log(e);
    ctx.status = 500;
    ctx.body = '服务器错误';
  }
});

// 加载路由组件
app
  .use(router.routes())
  .use(router.allowedMethods());

// 启动服务
app.listen(3000, () => {
  console.log(`server started at localhost:3000`);
});
```

启动服务后访问`localhost:3000/index`

![WX20191207-091327@2x](http://118.24.241.76/WX20191207-091327@2x.png)



可以将模板页面抽出，通过fs模块读取模板页面

Index.html

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

html中必须包含 <!--vue-ssr-outlet--> ， renderer.renderToString函数 真正渲染成html后，会把内容插入到该地方来。



server.js

```javascript
const Vue = require('vue');
const Koa = require('koa');
const Router = require('koa-router');
const renderer = require('vue-server-renderer').createRenderer({
  // 读取传入的template参数
  template: require('fs').readFileSync('./index.html', 'utf-8')
});

// 1. 创建koa koa-router实列

const app = new Koa();
const router = new Router();

// 2. 路由中间件

router.get('*', async(ctx, next) => {
  // 创建vue实列
  const app = new Vue({
    data: {
      url: ctx.url
    },
    template: `<div>访问的URL是：{{url}}</div>`
  });

  const context = {
    title: 'vue服务器渲染组件',
    meta: `
      <meta charset="utf-8">
      <meta name="" content="vue服务器渲染组件">
    `
  };
  try {
    // 传入context 渲染上下文对象
    const html = await renderer.renderToString(app, context);
    ctx.status = 200;
    ctx.body = html;
  } catch (e) {
    ctx.status = 500;
    ctx.body = '服务器错误';
  }
});

// 加载路由组件
app
  .use(router.routes())
  .use(router.allowedMethods());

// 启动服务
app.listen(3000, () => {
  console.log(`server started at localhost:3000`);
});
```

启动服务后访问`localhost:3000/index`

![WX20191207-092226@2x](http://118.24.241.76/WX20191207-092226@2x.png)



## 为每个请求创建一个新的根vue实列

服务器渲染过程中，只会调用 beforeCreate 和 created两个生命周期函数。其他的生命周期函数只会在客户端调用。因此在created生命周期函数中不要使用的不能销毁的变量存在。比如常见的 setTimeout, setInterval 等这些。并且window，document这些也不能在该两个生命周期中使用，因为node中并没有这两个东西，因此如果在服务器端执行的话，也会发生报错的。但是我们可以使用 axios来发请求的。因为它在服务器端和客户端都暴露了相同的API。但是浏览器原生的XHR在node中也是不支持的。

目录结构

```javascript
├── package-lock.json
├── package.json
├── server.js
└── src
    ├── app.js
    └── index.html
```

app.js

```javascript
const Vue = require('vue');

module.exports = function createApp (ctx) {
  return new Vue({
    data: {
      url: ctx.url
    },
    template: `<div>访问的URL是：{{url}}</div>`
  })
} 
```

暴露createApp方法是为了避免状态单例。Node.js 服务器是一个长期运行的进程，当我们运行到该进程的时候，它会将进行一次取值并且留在内存当中，那么这样很容易导致每个实列中的状态值会发生混乱。因此我们这边把app.js代码抽离一份出来，就是需要为每个请求创建一个新的实列。



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



server.js

```javascript
const Vue = require('vue');
const Koa = require('koa');
const Router = require('koa-router');
const renderer = require('vue-server-renderer').createRenderer({
  // 读取传入的template参数
  template: require('fs').readFileSync('./src/index.html', 'utf-8')
});

// 1. 创建koa koa-router实列
const app = new Koa();
const router = new Router();

// 引入 app.js
const createApp = require('./src/app');

// 2. 路由中间件

router.get('*', async(ctx, next) => {
  // 创建vue实列
  const app = createApp(ctx);

  const context = {
    title: 'vue服务器渲染组件',
    meta: `
      <meta charset="utf-8">
      <meta name="" content="vue服务器渲染组件">
    `
  };
  try {
    // 传入context 渲染上下文对象
    const html = await renderer.renderToString(app, context);
    ctx.status = 200;
    ctx.body = html;
  } catch (e) {
    ctx.status = 500;
    ctx.body = '服务器错误';
  }
});

// 加载路由组件
app
  .use(router.routes())
  .use(router.allowedMethods());

// 启动服务
app.listen(3000, () => {
  console.log(`server started at localhost:3000`);
});
```



## 路由实现和代码分割

上面的demo，我们只是使用 node server.js 运行服务器端的启动程序，然后进行服务器端渲染页面。但是没有将相同的vue代码提供给客户端，因此我们要实现这一点的话，我们需要在项目中引用我们的webpack来打包我们的应用程序。

目录结构

```
├── build
│   ├── webpack.base.config.js
│   ├── webpack.client.config.js
│   └── webpack.server.config.js
├── package-lock.json
├── package.json
├── server.js
├── router.js
├── .babelrc
└── src
    ├── App.vue
    ├── app.js
    ├── components
    │   ├── home.vue
    │   └── item.vue
    ├── entry-client.js
    ├── entry-server.js
    └── index.html
```



需要安装的依赖

```json
 "dependencies": {
    "koa": "^2.11.0",
    "koa-router": "^7.4.0",
    "koa-send": "^5.0.0",
    "stylus": "^0.54.7",
    "vue": "^2.6.10",
    "vue-router": "^3.1.3"
  },
  "devDependencies": {
    "@babel/core": "^7.7.5",
    "@babel/plugin-transform-runtime": "^7.7.5",
    "babel-loader": "^8.0.6",
    "css-loader": "^3.2.1",
    "mini-css-extract-plugin": "^0.8.0",
    "stylus-loader": "^3.0.2",
    "url-loader": "^3.0.0",
    "vue-loader": "^15.7.2",
    "vue-server-renderer": "^2.6.10",
    "vue-template-compiler": "^2.6.10",
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.10",
    "webpack-merge": "^4.2.2",
    "webpack-node-externals": "^1.7.2"
  }
```



src/index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>{{ title }}</title>
</head>
<body>
  <div id="app">
    <!--vue-ssr-outlet-->
  </div>
</body>
</html>
```



src/App.vue

```vue
<style lang="stylus">
  h1
    color red
    font-size 22px
</style>

<template>
  <div id="app">
    <router-view></router-view>
    <h1>{{ msg }}</h1>
    <input type="text" v-model="msg" />
  </div>
</template>

<script type="text/javascript">
  export default {
    name: 'app',
    data() {
      return {
        msg: '欢迎光临vue.js App'
      }
    }
  }
</script>
```



src/app.js

这边只是导出一个函数，返回app实例，没有做$mount操作

```javascript
import Vue from 'vue';

import App from './App.vue';

// 导出函数，用于创建新的应用程序
export function createApp () {
  const app = new Vue({
    // 根据实列简单的渲染应用程序组件
    render: h => h(App)
  });
  return { app };
}
```



src/entry-client.js

客户端入口，做$mount挂载操作

```javascript
import { createApp } from './app';

const { app } = createApp();

// 假设 App.vue 模板中根元素 id = 'app'

app.$mount('#app');
```



src/entry-server.js

服务端入口，实例化一个vue对象，然后返回实例化对象后的对象

```javascript
import { createApp } from './app';

export default context => {
  const { app } = createApp();
  return app;
}
```



src/router.js

这边也是与app.js一样，导出一个方法，每次调用创建新的实例

```javascript
import Vue from 'vue';
import Router from 'vue-router';

Vue.use(Router);

export function createRouter () {
  return new Router({
    mode: 'history',
    routes: [
      {
        path: '/home',
        component: resolve => require(['./components/home'], resolve)
      },
      {
        path: '/item',
        component: resolve => require(['./components/item'], resolve)
      },
      {
        path: '*',
        redirect: '/home'
      }
    ]
  });
}
```



更新src/app.js文件

```javascript
import Vue from 'vue';

import App from './App.vue';

// 引入 router
import { createRouter } from './router';

// 导出函数，用于创建新的应用程序
export function createApp () {
  // 创建 router的实列 
  const router = createRouter();

  const app = new Vue({
    // 注入 router 到 根 vue实列中
    router,
    // 根实列简单的渲染应用程序组件
    render: h => h(App)
  });
  return { app, router };
}
```



更新src/entry-server.js

实现服务器端的路由逻辑

```javascript
import { createApp } from './app';

export default context => {
  /*
  const { app } = createApp();
  return app;
  */
  /*
   由于 路由钩子函数或组件 有可能是异步的，比如 同步的路由是这样引入 import Foo from './Foo.vue'
   但是异步的路由是这样引入的：
   {
      path: '/index',
      component: resolve => require(['./views/index'], resolve)
   }
   如上是 require动态加载进来的，因此我们这边需要返回一个promise对象。以便服务器能够等待所有的内容在渲染前
   就已经准备好就绪。
  */
  return new Promise((resolve, reject) => {
    const { app, router } = createApp();

    // 设置服务器端 router的位置
    router.push(context.url);

    /* 
      router.onReady()
      等到router将可能的异步组件或异步钩子函数解析完成，在执行，就好比我们js中的 
      window.onload = function(){} 这样的。
      官网的解释：该方法把一个回调排队，在路由完成初始导航时调用，这意味着它可以解析所有的异步进入钩子和
      路由初始化相关联的异步组件。
      这可以有效确保服务端渲染时服务端和客户端输出的一致。
    */
    router.onReady(() => {
      /*
       getMatchedComponents()方法的含义是：
       返回目标位置或是当前路由匹配的组件数组 (是数组的定义/构造类，不是实例)。
       通常在服务端渲染的数据预加载时使用。
       有关 Router的实列方法含义可以看官网：https://router.vuejs.org/zh/api/#router-forward
      */
      const matchedComponents = router.getMatchedComponents();

      // 如果匹配不到路由的话，执行 reject函数，并且返回404
      if (!matchedComponents.length) {
        return reject({ code: 404 });
      }
      // 正常的情况
      resolve(app);
    }, reject);
  }).catch(new Function());
}
```



更新src/entry-client.js

由于路由有可能是异步组件或路由钩子，因此在 src/entry-client.js 中挂载元素之前也需要调用router.onReady因此代码需要改成如下所示

```javascript
import { createApp } from './app';

const { app, router } = createApp();

// App.vue 模板中根元素 id = 'app'

router.onReady(() => {
  app.$mount('#app');
});
```



src/components/home.vue

```vue
<template>
  <h1>home</h1>
</template>
<script>
export default {
  name: "home",
  data(){
    return{
       
    }
  }
}
</script>
```



src/components/item.vue

```vue
<template>
  <h1>item</h1>
</template>
<script>
export default {
  name: "item",
  data(){
    return{
       
    }
  }
}
</script>
```



**webpack配置**

webpack.base.config.js 基本配置

```javascript
const path = require('path')
// vue-loader v15版本需要引入此插件
const VueLoaderPlugin = require('vue-loader/lib/plugin')

// 用于返回文件相对于根目录的绝对路径
const resolve = dir => path.posix.join(__dirname, '..', dir)

module.exports = {
  // 入口暂定客户端入口，服务端配置需要更改它
  entry: resolve('src/entry-client.js'),
  // 生成文件路径、名字、引入公共路径
  output: {
    path: resolve('dist'),
    filename: '[name].js',
    publicPath: '/'
  },
  resolve: {
    // 对于.js、.vue引入不需要写后缀
    extensions: ['.js', '.vue'],
    // 引入components、assets可以简写，可根据需要自行更改
    alias: {
      'components': resolve('src/components'),
      'assets': resolve('src/assets')
    }
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
          // 配置哪些引入路径按照模块方式查找
          transformAssetUrls: {
            video: ['src', 'poster'],
            source: 'src',
            img: 'src',
            image: 'xlink:href'
          }
        }
      },
      {
        test: /\.js$/, // 利用babel-loader编译js，使用更高的特性，排除npm下载的.vue组件
        loader: 'babel-loader',
        exclude: file => (
          /node_modules/.test(file) &&
          !/\.vue\.js/.test(file)
        )
      },
      {
        test: /\.(png|jpe?g|gif|svg)$/, // 处理图片
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 10000,
              name: 'static/img/[name].[hash:7].[ext]'
            }
          }
        ]
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/, // 处理字体
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: 'static/fonts/[name].[hash:7].[ext]'
        }
      }
    ]
  },
  plugins: [
    new VueLoaderPlugin()
  ]
}
```



 webpack.client.config.js

该配置主要对客户端代码进行打包，并且它通过 webpack-merge 插件来对 webpack.base.config.js 代码配置进行合并

```javascript
const path = require('path')
const webpack = require('webpack')
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.config.js')
// css样式提取单独文件
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
// 服务端渲染用到的插件、默认生成JSON文件(vue-ssr-client-manifest.json)
const VueSSRClientPlugin = require('vue-server-renderer/client-plugin')

module.exports = merge(baseWebpackConfig, {
  mode: 'production',
  output: {
    // chunkhash是根据内容生成的hash, 易于缓存,
    // 开发环境不需要生成hash，目前先不考虑开发环境，后面详细介绍
    filename: 'static/js/[name].[chunkhash].js',
    chunkFilename: 'static/js/[id].[chunkhash].js'
  },
  module: {
    rules: [
      {
        test: /\.styl(us)?$/,
        // 利用mini-css-extract-plugin提取css, 开发环境也不是必须
        use: [MiniCssExtractPlugin.loader, 'css-loader', 'stylus-loader']
      },
    ]
  },
  devtool: false,
  plugins: [
    // webpack4.0版本以上采用MiniCssExtractPlugin 而不使用extract-text-webpack-plugin
    new MiniCssExtractPlugin({
      filename: 'static/css/[name].[contenthash].css',
      chunkFilename: 'static/css/[name].[contenthash].css'
    }),
    //  当vendor模块不再改变时, 根据模块的相对路径生成一个四位数的hash作为模块id
    new webpack.HashedModuleIdsPlugin(),
    new VueSSRClientPlugin()
  ]
})
```



webpack.server.config.js

```javascript
const path = require('path');
const webpack = require('webpack');
const merge = require('webpack-merge');
const nodeExternals = require('webpack-node-externals');
const baseConfig = require('./webpack.base.config');

const VueSSRServerPlugin = require('vue-server-renderer/server-plugin');

module.exports = merge(baseConfig, {
  entry: path.resolve(__dirname, '../src/entry-server.js'),
  /*
   允许webpack以Node适用方式(Node-appropriate fashion)处理动态导入(dynamic import)，
   编译vue组件时，告知 vue-loader 输送面向服务器代码
  */
  target: 'node',
  devtool: 'source-map',
  // 此处告知 server bundle 使用 Node 风格导出模块(Node-style exports)
  output: {
    libraryTarget: 'commonjs2',
    filename: '[name].server.js'
  },
  /*
   服务器端也需要编译样式，不能使用 mini-css-extract-plugin 插件
   ，因为该插件会使用document，但是服务器端并没有document, 因此会导致打包报错，我们可以如下的issues:
   https://github.com/webpack-contrib/mini-css-extract-plugin/issues/48#issuecomment-375288454
  */
  module: {
    rules: [
      {
        test: /\.styl(us)?$/,
        use: ['css-loader/locals', 'stylus-loader']
      }
    ]
  },
  // https://webpack.js.org/configuration/externals/#function
  // https://github.com/liady/webpack-node-externals
  // 外置化应用程序依赖模块。可以使服务器构建速度更快，
  // 并生成较小的 bundle 文件。
  externals: nodeExternals({
    // 不要外置化 webpack 需要处理的依赖模块。
    // 你可以在这里添加更多的文件类型。例如，未处理 *.vue 原始文件，
    // 你还应该将修改 `global`（例如 polyfill）的依赖模块列入白名单
    whitelist: /\.css$/
  }),

  // 这是将服务器的整个输出
  // 构建为单个 JSON 文件的插件。
  // 默认文件名为 `vue-ssr-server-bundle.json`
  plugins: [
    new webpack.DefinePlugin({
      'process.env.VUE_ENV': '"server"'
    }),
    new VueSSRServerPlugin()
  ]
});
```



编辑package.json文件

```json
...
"scripts": {
  "build:server": "webpack --config ./build/webpack.server.config.js",
  "build:client": "webpack --config ./build/webpack.client.config.js",
	"build:all":"npm run build:server && npm run build:client"
},
...
```



构建

```shell
$ npm run build:all
```



打包出来的文件

![WX20191207-162233@2x](http://118.24.241.76/WX20191207-162233@2x.png)



**服务**

server.js

我们在server.js 中需要引入我们刚刚打包完的客户端的 vue-ssr-client-manifest.json 文件 和 服务器端渲染的vue-ssr-server-bundle.json 文件，及 html模板 作为参数传入 到 createBundleRenderer 函数中

```javascript
const Vue = require('vue');
const Koa = require('koa');
const Router = require('koa-router');
const send = require('koa-send');

// 引入客户端，服务端生成的json文件, html 模板文件
const serverBundle = require('./dist/vue-ssr-server-bundle.json');
const clientManifest = require('./dist/vue-ssr-client-manifest.json');

let renderer = require('vue-server-renderer').createBundleRenderer(serverBundle, {
  runInNewContext: false, // 推荐
  template: require('fs').readFileSync('./src/index.html', 'utf-8'), // 页面模板
  clientManifest // 客户端构建 manifest
});

// 1. 创建koa koa-router实列
const app = new Koa();
const router = new Router();

const render = async (ctx, next) => {
  ctx.set('Content-Type', 'text/html')

  const handleError = err => {
    if (err.code === 404) {
      ctx.status = 404
      ctx.body = '404 Page Not Found'
    } else {
      ctx.status = 500
      ctx.body = '500 Internal Server Error'
      console.error(`error during render : ${ctx.url}`)
      console.error(err.stack)
    }
  }
  const context = {
    url: ctx.url,
    title: 'vue服务器渲染组件',
  }
  try {
    const html = await renderer.renderToString(context);
    ctx.status = 200
    ctx.body = html;
  } catch(err) {
    handleError(err);
  }
  next();
}
// 设置静态资源文件
router.get('/static/*', async(ctx, next) => {
  await send(ctx, ctx.path, { root: __dirname + '/./dist' });
});
router.get('*', render);

// 加载路由组件
app
  .use(router.routes())
  .use(router.allowedMethods());

// 启动服务
app.listen(3000, () => {
  console.log(`server started at localhost:3000`);
});
```



编辑package.json

添加一个启动server的命令

```json
"server":"node server.js"
```



启动服务并且访问`localhost:3000/home`

![WX20191207-231146@2x](http://118.24.241.76/WX20191207-231146@2x.png)



## 数据预获取和状态

在服务器端渲染(SSR)期间，比如说我们的应用程序有异步请求，在服务器端渲染之前，我们希望先返回异步数据后，我们再进行SSR渲染，因此我们需要的是先预取和解析好这些数据。

并且在客户端，在挂载(mount)到客户端应用程序之前，需要获取到与服务器端应用程序完全相同的数据。否则的话，客户端应用程序会因为使用与服务器端应用程序不同的状态。会导致混合失败。

因此为了解决上面的两个问题，我们需要把专门的数据放置到预取存储容器或状态容器中，因此store就这样产生了。我们可以把数据放在全局变量state中。并且，我们将在html中序列化和内联预置状态，这样，在挂载到客户端应用程序之前，可以直接从store获取到内联预置状态。



目录结构：

```
├── build
│   ├── webpack.base.config.js
│   ├── webpack.client.config.js
│   └── webpack.server.config.js
├── package-lock.json
├── package.json
├── server.js
├── router.js
├── .babelrc
├── store
│       └── index.js
├── api
│       └── index.js
└── src
    ├── App.vue
    ├── app.js
    ├── components
    │   ├── home.vue
    │   └── item.vue
    ├── entry-client.js
    ├── entry-server.js
    └── index.html
```



src/store/index.js

```javascript
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

// 假定我们有一个可以返回 Promise 的
import { fetchItem } from '../api/index';

export function createStore() {
  return new Vuex.Store({
    state: {
      items: {}
    },
    actions: {
      fetchItem({ commit }, id) {
        // `store.dispatch()` 会返回 Promise，
        // 以便我们能够知道数据在何时更新
        return fetchItem(id).then(item => {
          commit('setItem', { id, item });
        });
      }
    },
    mutations: {
      setItem(state, { id, item }) {
        Vue.set(state.items, id, item);
      }
    }
  });
}
```



src/api/index.js 

```javascript
export function fetchItem(id) {
  return Promise.resolve({
    text: 'kongzhi'
  })
}
```



src/app.js

```javascript
import Vue from 'vue';

import App from './App.vue';

// 引入 router
import { createRouter } from './router';
// 引入store
import { createStore } from './store/index';

import { sync } from 'vuex-router-sync';

// 导出函数，用于创建新的应用程序
export function createApp () {

  // 创建 router的实列 
  const router = createRouter();

  // 创建 store 的实列
  const store = createStore();

  // 同步路由状态 (route state) 到 store
  sync(store, router);

  const app = new Vue({
    // 注入 router 到 根 vue实列中
    router,
    store,
    // 根实列简单的渲染应用程序组件
    render: h => h(App)
  });
  // 暴露 app, router, store
  return { app, router, store };
}
```

我们需要在什么地方使用 dispatch来触发action代码呢？

按照官网说的，我们需要通过访问路由，来决定获取哪部分数据，这也决定了哪些组件需要被渲染。因此我们在组件 Item.vue 路由组件上暴露了一个自定义静态函数 asyncData。

**注意**：asyncData函数会在组件实例化之前被调用。因此不能使用this，需要将store和路由信息作为参数传递进去。



 src/components/item.vue

```vue
<template>
  <div>item页 请求数据结果：{{ item.name.text }}</div>
</template>
<script>
export default {
  name: "item",
  asyncData ({ store, route }) {
    // 触发action代码，会返回 Promise
    return store.dispatch('fetchItem', 'name');
  },
  computed: {
    // 从 store 的 state 对象中的获取 item。
    item () {
      console.log(this.$store.state);
      return this.$store.state.items;
    }
  }
}
</script>

<style scoped>

</style>
```



**服务器端数据预取**

服务器端预取的原理是：在 entry-server.js中，我们可以通过路由获得与 router.getMatchedComponents() 相匹配的组件，该方法是获取到所有的组件，然后我们遍历该所有匹配到的组件。如果组件暴露出 asyncData 的话，我们就调用该方法。并将我们的state挂载到context上下文中。vue-server-renderer 会将state序列化 window.\_\_INITAL_STATE\_\_. 这样，entry-client.js客户端就可以替换state，实现同步。

src/entry-server.js

```javascript
import { createApp } from './app';
export default context => {
  /*
  const { app } = createApp();
  return app;
  */
  /*
   由于 路由钩子函数或组件 有可能是异步的，比如 同步的路由是这样引入 import Foo from './Foo.vue'
   但是异步的路由是这样引入的：
   {
      path: '/index',
      component: resolve => require(['./views/index'], resolve)
   }
   如上是 require动态加载进来的，因此我们这边需要返回一个promise对象。以便服务器能够等待所有的内容在渲染前
   就已经准备好就绪。
  */
  return new Promise((resolve, reject) => {
    const { app, router, store } = createApp();

    // 设置服务器端 router的位置
    router.push(context.url);

    /* 
      router.onReady()
      等到router将可能的异步组件或异步钩子函数解析完成，在执行，就好比我们js中的 
      window.onload = function(){} 这样的。
      官网的解释：该方法把一个回调排队，在路由完成初始导航时调用，这意味着它可以解析所有的异步进入钩子和
      路由初始化相关联的异步组件。
      这可以有效确保服务端渲染时服务端和客户端输出的一致。
    */
    router.onReady(() => {
      /*
       getMatchedComponents()方法的含义是：
       返回目标位置或是当前路由匹配的组件数组 (是数组的定义/构造类，不是实例)。
       通常在服务端渲染的数据预加载时使用。
       有关 Router的实列方法含义可以看官网：https://router.vuejs.org/zh/api/#router-forward
      */
      const matchedComponents = router.getMatchedComponents();

      // 如果匹配不到路由的话，执行 reject函数，并且返回404
      if (!matchedComponents.length) {
        return reject({ code: 404 });
      }
      // 对所有匹配的路由组件 调用  'asyncData()'
      Promise.all(matchedComponents.map(Component => {
        if (Component.asyncData) {
          return Component.asyncData({
            store,
            route: router.currentRoute
          });
        }
      })).then(() => {
        // 在所有预取钩子(preFetch hook) resolve 后，
        // 我们的 store 现在已经填充入渲染应用程序所需的状态。
        // 当我们将状态附加到上下文，
        // 并且 `template` 选项用于 renderer 时，
        // 状态将自动序列化为 `window.__INITIAL_STATE__`，并注入 HTML。
        context.state = store.state
        resolve(app);
      }).catch(reject)
      // 正常的情况
      // resolve(app);
    }, reject);
  }).catch(new Function());
}

```

如上官网代码，当我们使用 template 时，context.state 将作为 window.\_\_INITIAL_STATE\_\_ 状态，自动嵌入到最终的 HTML 中。而在客户端，在挂载到应用程序之前，store 就应该获取到状态



entry-client.js

```javascript
import { createApp } from './app';

const { app, router, store } = createApp();

if (window.__INITIAL_STATE__) {
  store.replaceState(window.__INITIAL_STATE__);
}

// App.vue 模板中根元素 id = 'app'
router.onReady(() => {
  app.$mount('#app');
});
```



**客户端数据预取**

在客户端，处理数据预取有 **2种方式** ：分别是：在**路由导航之前解析数据** 和 **匹配要渲染的视图后**，再获取数据。

- 在路由导航之前解析数据

在这种方式下，应用程序会在所需要的数据全部解析完成后，再传入数据并处理当前的视图。它的优点是：可以直接在数据准备就绪时，传入数据到视图渲染完整的内容。但是如果数据预取需要很长时间的话，那么用户在当前视图会感受到 "明显卡顿"。因此，如果我们使用这种方式预取数据的话，我们可以使用一个菊花加载icon，等所有数据预取完成后，再把该菊花消失掉。

src/entry-client.js

```javascript
import { createApp } from './app';

const { app, router, store } = createApp();

if (window.__INITIAL_STATE__) {
  store.replaceState(window.__INITIAL_STATE__);
}

router.onReady(() => {
  // 添加路由钩子，用于处理 asyncData
  // 在初始路由 resolve 后执行
  // 以便我们不会二次预取已有的数据
  // 使用 router.beforeResolve(), 确保所有的异步组件都 resolve
  router.beforeResolve((to, from, next) => {
    const matched = router.getMatchedComponents(to);
    const prevMatched = router.getMatchedComponents(from);

    // 我们只关心非预渲染的组件
    // 所有我们需要对比他们，找出两个品牌列表的差异组件
    let diffed = false
    const activated = matched.filter((c, i) => {
      return diffed || (diffed = (prevMatched[i] !== c))
    })

    if (!activated.length) {
      return next()
    }
    // 这里如果有加载指示器 (loading indicator)，就触发
    Promise.all(activated.map(c => {
      if (c.asyncData) {
        return c.asyncData({ store, route: to })
      }
    })).then(() => {
        // 停止加载指示器(loading indicator)
        next()
    }).catch(next)
  });
  app.$mount('#app')
});
```



- 匹配渲染的视图后，再获取数据

根据官网介绍：该方式是将客户端数据预取，放在视图组件的 beforeMount 函数中。当路由导航被触发时，我们可以立即切换视图，因此应用程序具有更快的响应速度。但是，传入视图在渲染时不会有完整的可用数据。因此，对于使用此策略的每个视图组件，都需要具有条件的加载状态。因此这可以通过纯客户端的全局mixin来实现

src/entry-client.js

```javascript
import { createApp } from './app';
import Vue from 'vue';

Vue.mixin({
  beforeRouteUpdate (to, from, next) {
    const { asyncData } = this.$options;
    if (asyncData) {
      asyncData({
        store: this.$store,
        route: to
      }).then(next).catch(next)
    } else {
      next();
    }
  }
})


const { app, router, store } = createApp();

if (window.__INITIAL_STATE__) {
  store.replaceState(window.__INITIAL_STATE__);
}

router.onReady(() => {
  // 添加路由钩子，用于处理 asyncData
  // 在初始路由 resolve 后执行
  // 以便我们不会二次预取已有的数据
  // 使用 router.beforeResolve(), 确保所有的异步组件都 resolve
  router.beforeResolve((to, from, next) => {
    const matched = router.getMatchedComponents(to);
    const prevMatched = router.getMatchedComponents(from);

    // 我们只关心非预渲染的组件
    // 所有我们需要对比他们，找出两个品牌列表的差异组件
    let diffed = false
    const activated = matched.filter((c, i) => {
      return diffed || (diffed = (prevMatched[i] !== c))
    })

    if (!activated.length) {
      return next()
    }
    // 这里如果有加载指示器 (loading indicator)，就触发
    Promise.all(activated.map(c => {
      if (c.asyncData) {
        return c.asyncData({ store, route: to })
      }
    })).then(() => {
        // 停止加载指示器(loading indicator)
        next()
    }).catch(next)
  });
  app.$mount('#app')
});
```



构建并且访问`localhost:3000/item`

![WX20191208-150054@2x](http://118.24.241.76/WX20191208-150054@2x.png)





## 页面注入不同的Head

在如上服务器端渲染的时候，我们会根据不同的页面会有不同的meta或title。因此我们需要注入不同的Head内容, 我们按照官方
文档来实现一个简单的title注入。如何做呢？



我们需要在我们的`index.html`模块中定义 `<title>{{ title }}</title>`, 它的基本原理和数据预取是类似的。

index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>{{ title }}</title>
</head>
<body>
  <div id="app">
    <!--vue-ssr-outlet-->
  </div>
</body>
</html>
```

**注意：**
1. 使用双花括号(double-mustache)进行 HTML 转义插值(HTML-escaped interpolation)，以避免 XSS 攻击。
2. 应该在创建 context 对象时提供一个默认标题，以防在渲染过程中组件没有设置标题。



目录结构：

```
├── build
│   ├── webpack.base.config.js
│   ├── webpack.client.config.js
│   └── webpack.server.config.js
├── package-lock.json
├── package.json
├── server.js
├── router.js
├── .babelrc
├── store
│       └── index.js
├── api
│       └── index.js
├── mixins
│       └── title-mixin.js
└── src
    ├── App.vue
    ├── app.js
    ├── components
    │   ├── home.vue
    │   └── item.vue
    ├── entry-client.js
    ├── entry-server.js
    └── index.html
```



src/mixins/title-mixin.js

```javascript
function getTitle (vm) {
  // 组件可以提供一个 `title` 选项
  // 此选项可以是一个字符串或函数
  const { title } = vm.$options;
  if (title) {
    return typeof title === 'function' ? title.call(vm) : title;
  } else {
    return 'Vue SSR Demo';
  }
}

const serverTitleMixin = {
  created () {
    const title = getTitle(this);
    if (title && this.$ssrContext) {
      this.$ssrContext.title = title;
    }
  }
};

const clientTitleMixin = {
  mounted () {
    const title = getTitle(this);
    if (title) {
      document.title = title;
    }
  }
};

// 我们可以通过 'webpack.DefinePlugin' 注入 'VUE_ENV'

export default process.env.VUE_ENV === 'server' ? serverTitleMixin : clientTitleMixin;
```

在webpack配置里设置的全局变量

![WX20191208-162840@2x](http://118.24.241.76/WX20191208-162840@2x.png)



src/components/item.vue

```vue
<template>
  <div>item页 请求数据结果：{{ item.name.text }}</div>
</template>
<script>
  import titleMixin from '../mixins/title-mixin.js';
  export default {
    name: "item",
    mixins: [titleMixin],
    title() {
      return 'item页面';
    },
    asyncData ({ store, route }) {
      // 触发action代码，会返回 Promise
      return store.dispatch('fetchItem', 'name');
    },
    computed: {
      // 从 store 的 state 对象中的获取 item。
      item () {
        console.log(this.$store.state);
        return this.$store.state.items;
      }
    }
  }
</script>

<style scoped>

</style>
```



src/components/home.vue

```vue
<template>
  <h1>home222</h1>
</template>
<script>
  import titleMixin from '../mixins/title-mixin.js';
  export default {
    name: "home",
    mixins: [titleMixin],
    title() {
      return 'Home页面';
    },
    data(){
      return{
         
      }
    }
  }
</script>
<style scoped>

</style>
```



构建并访问`localhost:3000/home`和`localhost:3000/item`

![WX20191208-163706@2x](http://118.24.241.76/WX20191208-163706@2x.png)

![WX20191208-163724@2x](http://118.24.241.76/WX20191208-163724@2x.png)



title发生变化



## 页面级别的缓存

缓存(官网介绍)：虽然vue的服务器端渲染非常快，但是由于创建组件实列和虚拟DOM节点的开销，无法与纯基于字符串拼接的模板性能相当。因此我们需要使用缓存策略，可以极大的提高响应时间且能减少服务器的负载。



**页面级别缓存**

server.js

```javascript
const Vue = require('vue');
const Koa = require('koa');
const Router = require('koa-router');
const send = require('koa-send');

// 引入缓存相关的模块
const LRU = require('lru-cache');

// 缓存
const microCache = new LRU({
  max: 100,
  maxAge: 1000 * 60 // 在1分钟后过期
});

const isCacheable = ctx => {
  // 假如 item 页面进行缓存
  if (ctx.url === '/item') {
    return true;
  }
  return false;
};

// 引入客户端，服务端生成的json文件, html 模板文件
const serverBundle = require('./dist/vue-ssr-server-bundle.json');
const clientManifest = require('./dist/vue-ssr-client-manifest.json');

let renderer = require('vue-server-renderer').createBundleRenderer(serverBundle, {
  runInNewContext: false, // 推荐
  template: require('fs').readFileSync('./src/index.html', 'utf-8'), // 页面模板
  clientManifest // 客户端构建 manifest
});

// 1. 创建koa koa-router实列
const app = new Koa();
const router = new Router();

const render = async (ctx, next) => {
  ctx.set('Content-Type', 'text/html')

  const handleError = err => {
    if (err.code === 404) {
      ctx.status = 404
      ctx.body = '404 Page Not Found'
    } else {
      ctx.status = 500
      ctx.body = '500 Internal Server Error'
      console.error(`error during render : ${ctx.url}`)
      console.error(err.stack)
    }
  }
  const context = {
    url: ctx.url,
    title: 'vue服务器渲染组件',
  }
  // 判断是否可缓存，可缓存，且缓存中有的话，直接把缓存中返回
  const cacheable = isCacheable(ctx);
  if (cacheable) {
    const hit = microCache.get(ctx.url);
    if (hit) {
      console.log('从缓存中取', hit);
      return ctx.body = hit;
    }
  }

  try {
    const html = await renderer.renderToString(context);
    ctx.body = html;
    if (cacheable) {
      console.log('设置缓存：', ctx.url);
      microCache.set(ctx.url, html);
    }
  } catch (err) {
    console.log(err);
    handleError(err);
  }
  next();
}
// 设置静态资源文件
router.get('/static/*', async (ctx, next) => {
  await send(ctx, ctx.path, {
    root: __dirname + '/./dist'
  });
});

router.get('*', render);

// 加载路由组件
app
  .use(router.routes())
  .use(router.allowedMethods());

// 启动服务
app.listen(3000, () => {
  console.log(`server started at localhost:3000`);
});
```



构建项目并且访问`localhost:3000/item`

![WX20191208-224803@2x](http://118.24.241.76/WX20191208-224803@2x.png)





## 参考资料

[webpack4+koa2+vue 实现服务器端渲染(详解)](https://www.cnblogs.com/tugenhua0707/p/11048465.html)

[Vue SSR 指南](https://ssr.vuejs.org/zh/)

[解密Vue SSR](https://juejin.im/post/5b063962f265da0ddb63dac3)

[理解webpack之process.env.NODE_ENV详解(十八)](https://www.cnblogs.com/tugenhua0707/p/9780621.html)

[demo地址]()