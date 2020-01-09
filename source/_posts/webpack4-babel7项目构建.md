---
title: webpack4 + babel7 + Vue2.x
date: 2019-11-10 18:38:31
tags: 
- webpack
categories: 
- webpack
---

## 新建项目

**目录结构**

```
.
├── README.en.md
├── README.md
├── config
│   ├── utils.js //工具类
│   ├── webpack.base.js //webpack通用配置
│   ├── webpack.dev.js //webpack dev环境配置
│   └── webpack.prod.js// webpack prod环境配置
├── package-lock.json
├── package.json
├── postcss.config.js //postcss配置文件
├── .babelrc	//babel配置
├── .gitignore //不添加到版本控制文件
└── src
    ├── app.vue 
    ├── assets // 图片、依赖、字体等资源目录
    │   └── images
    ├── common //通用样式配置文件
    │   ├── common.scss
    │   └── variable.scss
    ├── components //组件
    │   └── index.js
    ├── directives //指令
    │   └── index.js
    ├── filters //管道
    │   └── index.js
    ├── index.html //应用入口
    ├── index.js //入口文件
    ├── mixins // mixins
    │   └── index.js
    ├── router //路由配置
    │   └── index.js
    ├── services //服务
    │   └── index.js
    └── views //视图
        └── home
            ├── home.scss
            └── home.vue
```

**安装webpack和webpack-cli**

```shell
$ npm install webpack webpack-cli -D
```

**编辑package.json文件**

```javascript
  ...
	"scripts": {
    	"start": "webpack --config ./config/webpack.base.js"
  },
  ...
```



## Babel

[Babel](https://babeljs.io/docs/en/) 是一个让我们能够使用 ES 新特性的 JS 编译工具，我们可以在 `webpack` 中配置 `Babel`，以便使用 ES6、ES7 标准来编写 JS 代码。



##### babel-loader 和 @babel/core

- [babel-loader](https://www.npmjs.com/package/babel-loader): 转义 js 文件代码的 loader
- [@babel/core](https://www.npmjs.com/package/@babel/core)：babel 核心库

**安装**

```shell
$ npm install babel-loader @babel/core -D
```

**编辑webpack.base.js**

```javascript
const {
    resolve
} = require('./utils');

module.exports = {
  	mode:'development',
    entry: resolve('./src/index.js'),//入口文件
    output: {//输出文件
        filename: "bundle.[hash:8].js",//文件名
        path: resolve('./dist')//路径
    },
    module: {
        rules: [{
            test: /\.js$/,
            exclude: /(node_modules)/,
            use: {
                loader: 'babel-loader'
            } // options 在 .babelrc 定义
        }]
    }
}
```

**引入了一个utils.js工具js**

```javascript
const path = require('path');

// 减少路径书写
function resolve(dir) {
    return path.join(__dirname,'..', dir)
}
module.exports = {
    resolve
}
```

**编辑index.js**

```javascript
const func = () => {
  console.log('hello webpack')
}
func()

class User {
  constructor() {
    console.log('new User')
  }
}

const user = new User();
```

**构建**

```shell
$ npm run start
# 如果有创建.babelrc文件的话可以限制为空对象，不然空文件会报错
```

关于mode，它有三个参数`production`，`development`，`none`，前两个是有预设的插件，而最后一个则是什么都没有，也就是说设置为`none`的话，webpack就是最初的样子，无任何预设，需要从无到有开始配置。

- development：是告诉程序，我现在是开发状态，也就是打包出来的内容要对开发友好。在此mode下，就做了以下插件的事，其他都没做，所以这些插件可以省略。

```javascript
module.exports = {
+ mode: 'development'
- devtool: 'eval',
- plugins: [
-   new webpack.NamedModulesPlugin(),
-   new webpack.NamedChunksPlugin(),
-   new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("development") }),
- ]
}
```

NamedModulesPlugin：没有`NamedModulesPlugin`，模块就是一个数组，引用也是按照在数组中的顺序引用，新增减模块都会导致序号的变化。有了`NamedModulesPlugin`，模块都拥有了姓名，而且都是独一无二的key，不管新增减多少模块，模块的key都是固定的。

development:

```javascript
 (function(modules) { 
   ...
 })

 ({

 "./node_modules/@babel/runtime-corejs2/helpers/classCallCheck.js":
 (function(module, exports) {

eval("function _classCallCheck(instance, Constructor) {\n  if (!(instance instanceof Constructor)) {\n    throw new TypeError(\"Cannot call a class as a function\");\n  }\n}\n\nmodule.exports = _classCallCheck;\n\n//# sourceURL=webpack:///./node_modules/@babel/runtime-corejs2/helpers/classCallCheck.js?");

 }),

 "./src/index.js":
 (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _babel_runtime_corejs2_helpers_classCallCheck__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! @babel/runtime-corejs2/helpers/classCallCheck */ \"./node_modules/@babel/runtime-corejs2/helpers/classCallCheck.js\");\n/* harmony import */ var _babel_runtime_corejs2_helpers_classCallCheck__WEBPACK_IMPORTED_MODULE_0___default = /*#__PURE__*/__webpack_require__.n(_babel_runtime_corejs2_helpers_classCallCheck__WEBPACK_IMPORTED_MODULE_0__);\n\n\nvar func = function func() {\n  console.log('hello webpack');\n};\n\nfunc();\n\nvar User = function User() {\n  _babel_runtime_corejs2_helpers_classCallCheck__WEBPACK_IMPORTED_MODULE_0___default()(this, User);\n\n  console.log('new User');\n};\n\nvar user = new User();\n\n//# sourceURL=webpack:///./src/index.js?");

 })

 });
```

production:

```javascript
! function (e) {
    
  ...
  
}([function (e, n) {
    e.exports = function (e, n) {
        if (!(e instanceof n)) throw new TypeError("Cannot call a class as a function")
    }
}, function (e, n, t) {
    "use strict";
    t.r(n);
    var r = t(0),
        o = t.n(r);
    console.log("hello webpack");
    new function e() {
        o()(this, e), console.log("new User")
    }
}]);
```

NamedChunksPlugin：还有一个`NamedChunksPlugin`，这个是给配置的每个chunks命名，原本的chunks也是数组，没有姓名。

- 在正式版本中，所省略的插件们，如下所示

```javascript
module.exports = {
+  mode: 'production',
-  plugins: [
-    new UglifyJsPlugin(/* ... */),
-    new webpack.DefinePlugin({ "process.env.NODE_ENV": JSON.stringify("production") }),
-    new webpack.optimize.ModuleConcatenationPlugin(),
-    new webpack.NoEmitOnErrorsPlugin()
-  ]
}
```

UglifyJsPlugin：混淆&压缩JS

ModuleConcatenationPlugin：此插件仅适用于由 webpack 直接处理的 ES6 模块。在使用转译器(transpiler)时，你需要禁用对模块的处理（例如 Babel 中的 modules 选项）。

NoEmitOnErrorsPlugin：这个就是用于防止程序报错，就算有错误也给我继续编译，很暴力的做法呢。

**构建之后的文件**

```javascript
(function (modules) { 

  ...
  
})
({
  "./src/index.js":
  (function (module, exports) {
      eval("const func = () => {\n  console.log('hello webpack');\n};\n\nfunc();\n\nclass User {\n  constructor() {\n    console.log('new User');\n  }\n\n}\n\nconst user = new User();\n\n//# sourceURL=webpack:///./src/index.js?");
    })
});
```

实现它的模块化的地方不看，主要看编译出来的文件，es6语法没有被转成es5语法



##### @babel-preset-env

如果要转义箭头函数，需要使用到 `@babel/plugin-transform-arrow-functions` 这个插件 同理转义 `class` 需要使用 `@babel/plugin-transform-classes`，但是如果还需要转义其他的（比如：async await），一个一个添加不合理，所以`@babel-preset-env`整合了它们

**安装**

```shell
$ npm install @babel/preset-env -D
```

**编辑.babelrc文件**

```javascript
{
  "presets": ["@babel/preset-env"]
}
```

这时候打包的就有转义语法了

```javascript
 (function(modules) { 
  
   ...

 })
 ({

 "./src/index.js":
 (function(module, exports) {
eval("function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError(\"Cannot call a class as a function\"); } }\n\nvar func = function func() {\n  console.log('hello webpack');\n};\n\nfunc();\n\nvar User = function User() {\n  _classCallCheck(this, User);\n\n  console.log('new User');\n};\n\nvar user = new User();\n\n//# sourceURL=webpack:///./src/index.js?");
 })
 });
```

但是这样也只是转换了语法，而没有转换API，比如 Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，这样就导致了一些新的 API 老版浏览器不兼容。



##### @babel/polyfill

使用它来兼容，这边只说按需编译。

**安装**

```shell
$ npm install @babel/polyfill -D
```

**编辑.babelrc文件**

```json
{
    "presets": [
        ["@babel/preset-env", {
            "useBuiltIns": "usage"
        }]
    ]
}
```

**编辑index.js文件**

```javascript
import '@babel/polyfill' // 引入

const func = () => {
  console.log('hello webpack')
}
func()

class User {
  constructor() {
    console.log('new User')
  }
}

const user = new User()

new Promise(resolve => console.log('promise'))

Array.from('foo');
```

**构建**

```shell
$ npm start
```

编译出来的文件大了挺多的



##### @babel/runtime 和 @babel/plugin-transform-runtime

`babel-polyfill` 会污染全局作用域, 如引入 `Array.prototype.includes` 修改了 Array 的原型，除此外还有 String等。也会引入新的对象： `Promise`、`WeakMap` 等。

`@babel/runtime` 的作用： 

- 提取辅助函数。ES6 转码时，babel 会需要一些辅助函数，例如 _extend。babel 默认会将这些辅助函数内联到每一个 js 文件里， babel 提供了 transform-runtime 来将这些辅助函数“搬”到一个单独的模块 `babel-runtime` 中，这样做能减小项目文件的大小。

- 提供 `polyfill`：不会污染全局作用域，但是不支持实例方法如 Array.includes

`@transform-runtime` 的作用： 

- `babel-runtime` 更像是分散的 `polyfill` 模块，需要在各自的模块里单独引入，借助 transform-runtime 插件来自动化处理这一切，也就是说你不要在文件开头 import 相关的 `polyfill`，你只需使用，`transform-runtime` 会帮你引入

**安装**

```shell
$ npm install @babel/runtime-corejs2 @babel/plugin-transform-runtime -D
```

**编辑.babelrc文件**

```json
{
    "presets": ["@babel/preset-env"],
    "plugins": [
        ["@babel/plugin-transform-runtime", {
            "corejs": 2
        }]
    ]
}
```

**编辑index.js文件**

```javascript
const func = () => {
  console.log('hello webpack')
}
func()

class User {
  constructor() {
    console.log('new User')
  }
}

const user = new User()

new Promise(resolve => console.log('promise'))

Array.from('foo');
```

**构建**

```shell
$ npm start
```



##### @babel/plugin-proposal-decorators

添加装饰器支持

**安装**

```shell
$ npm install @babel/plugin-proposal-decorators -D
```

**编辑.babelrc文件**

```json
{
    "presets": ["@babel/preset-env"],
    "plugins": [
        ["@babel/plugin-proposal-decorators", {
            "decoratorsBeforeExport": true
        }],
        ["@babel/plugin-transform-runtime", {
            "corejs": 2
        }]
    ]
}
```

因为我没使用到，所以忽略装饰器支持。



##### 引入tree shaking

它可以剔除JavaScript中用不上的死代码。它依赖静态的ES6模块化语法，需要配置Babel让它保留ES6模块化代码语句交给webpack。



**编辑.babelrc.json文件**

```javascript
{
    "presets": [[
        "@babel/preset-env",
        {
            "modules": false
        }
    ]],
    "plugins": [
        ["@babel/plugin-transform-runtime", {
            "corejs": 2
        }]
    ]
}
```



**测试**

```javascript
//index.js
import {
  A,
} from './index1'
A();

//index1.js
function A() {
}
function B(c) {
}
export {
    A,
    B
};
```



**构建**

需要Uglifyjs之类的配合（webpack4，设置mode为production就可以）。可以在构建的shell后面添加`--display-used-exports`参数

![WX20191113-230408@2x](http://118.24.241.76/WX20191113-230408@2x.png)





## Webpack

##### 几种 hash 的区别

- **hash**
  - `hash` 和每次 `build`有关，没有任何改变的情况下，每次编译出来的 `hash`都是一样的，但当你改变了任何一点东西，它的`hash`就会发生改变。
  
    简单理解，你改了任何东西，`hash` 就会和上次不一样了。
- **chunkhash**
  
  - `chunkhash`是根据具体每一个模块文件自己的的内容包括它的依赖计算所得的`hash`，所以某个文件的改动只会影响它本身的`hash`，不会影响其它文件（但是会影响引入的css文件）。
  
    简单理解，改变了js文件不会影响其他js文件的chunkhash，但会影响到被引入的css的chunkhash
- **contenthash**
  - 它的出现主要是为了解决，让`css`文件不受`js`文件的影响。比如`foo.css`被`foo.js`引用了，所以它们共用相同的`chunkhash`值。但这样子是有问题的，如果`foo.js`修改了代码，`css`文件就算内容没有任何改变，由于是该模块的 `hash` 发生了改变，其`css`文件的`hash`也会随之改变。这个时候我们就可以使用`contenthash`了，保证即使`css`文件所处的模块里有任何内容的改变，只要 css 文件内容不变，那么它的`hash`就不会发生变化。
  
    简单理解，自己变化才变化，只影响自己。



##### 安装插件

```shell
$ npm install html-webpack-plugin -D
# 创建html引用打包文件，并且对html压缩

$ npm install clean-webpack-plugin -D
# 清理文件

$ npm install style-loader css-loader mini-css-extract-plugin postcss-loader autoprefixer -D
# 打包css文件并且抽离出来，以及自动添加前缀

$ npm install optimize-css-assets-webpack-plugin -D
#压缩优化css

$ npm install node-sass sass-loader -D 
# 处理sass为css

$ npm install file-loader url-loader -D
# 处理资源文件

$ npm install image-webpack-loader -D
#图片资源压缩

$ npm install webpack-merge -D
# 合并配置

$ npm install webpack-dev-server -D
# 本地开启一个简单的静态服务来进行开发

$ npm install vue-loader vue-template-compiler vue-style-loader -D
# vue-loader :处理.vue文件 vue-style-loader:处理.vue里面的样式 vue-template-compiler:编译.vue中template里面的内容

$ npm install @babel/core @babel/plugin-transform-runtime @babel/preset-env @babel/runtime-corejs2 -D
#babel部分

$ npm install cross-env -D
# 运行跨平台设置和使用环境变量的脚本

$ npm install sass-resources-loader -D
# 处理通用sass引入(变量之类的)

$ npm install terser-webpack-plugin -D
# 压缩js

$ npm install hard-source-webpack-plugin -D
# 缓存 动态链接库插件

$ npm install happypack -D
# 多进程使用，分解成多个子任务
```

##### 补充

**[mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin#minimal-example)**

该插件将CSS提取到单独的文件中。它为每个包含CSS的JS文件创建一个CSS文件。它自动会配合`optimization.splitChunks`的配置。假如单独配置了`element-ui`作为单独一个`bundle`，它会自动也将它的样式单独打包成一个 css 文件，不会像以前默认将第三方的 css 全部打包成一个几十甚至上百 KB 的`app.xxx.css`文件了。

```json
//打包 css 之后查看源码，我们发现它并没有帮我们做代码压缩
//使用 optimize-css-assets-webpack-plugin 压缩与优化

//如何配置
optimization: {
  minimizer: [new OptimizeCSSAssetsPlugin()];
}

//比如padding:6px 6px 6px 6px;会优化为padding:6px; 
```

建议输出文件名需要使用hash的时候使用`contenthash`而不是`hash`。否则每次改动一个`xx.js`文件，它对应的 css 虽然没做任何改动，但它的 文件 hash 还是会发生变化。



**optimization.splitChunks**

代码分包策略：按照体积大小、共用率、更新频率重新划分包，使其尽可能的利用浏览器缓存。（不过需要注意的是不需要细分太多包，解决了一部分问题，同时也带来了页面请求时候加载太多会阻塞的问题）。

我们希望更新频率低的抽出成单个文件，并且一直走缓存（它们的js与css）。

| 类型     | 共用率 | 使用频率 | 更新频率 | 例子             |      |
| -------- | ------ | -------- | -------- | ---------------- | ---- |
| 基础类库 | 高     | 高       | 低       | vue/vue-router等 |      |
| UI库     | 高     | 高       | 中       | Element-UI等     |      |
| 组件     | 高     | 高       | 中       | 自定义组件等     |      |
| 业务代码 | 低     | 高       | 高       | 业务逻辑等       |      |

**补充一下mainfest与vendor.js**

- vendor.js: 第三方库，一般是 node_modules里面的依赖进行打包
- manifest: runtime代码。浏览器运行时，webpack用来连接模块化的应用程序的所有代码（我的理解就是它就是实现模块化的代码）

`manifest.js`长这样，这部分其实不怎么会变化，应该使用缓存

```javascript
 (function(modules) { // webpackBootstrap
 	// install a JSONP callback for chunk loading
 	function webpackJsonpCallback(data) {
 		var chunkIds = data[0];
 		var moreModules = data[1];
 		var executeModules = data[2];

 		// add "moreModules" to the modules object,
 		// then flag all "chunkIds" as loaded and fire callback
 		var moduleId, chunkId, i = 0, resolves = [];
 		for(;i < chunkIds.length; i++) {
 			chunkId = chunkIds[i];
 			if(Object.prototype.hasOwnProperty.call(installedChunks, chunkId) && installedChunks[chunkId]) {
 				resolves.push(installedChunks[chunkId][0]);
 			}
 			installedChunks[chunkId] = 0;
 		}
 		for(moduleId in moreModules) {
 			if(Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
 				modules[moduleId] = moreModules[moduleId];
 			}
 		}
 		if(parentJsonpFunction) parentJsonpFunction(data);

 		while(resolves.length) {
 			resolves.shift()();
 		}

 		// add entry modules from loaded chunk to deferred list
 		deferredModules.push.apply(deferredModules, executeModules || []);

 		// run deferred modules when all chunks ready
 		return checkDeferredModules();
 	};
 	function checkDeferredModules() {
 		var result;
 		for(var i = 0; i < deferredModules.length; i++) {
 			var deferredModule = deferredModules[i];
 			var fulfilled = true;
 			for(var j = 1; j < deferredModule.length; j++) {
 				var depId = deferredModule[j];
 				if(installedChunks[depId] !== 0) fulfilled = false;
 			}
 			if(fulfilled) {
 				deferredModules.splice(i--, 1);
 				result = __webpack_require__(__webpack_require__.s = deferredModule[0]);
 			}
 		}

 		return result;
 	}

 	// The module cache
 	var installedModules = {};

 	// object to store loaded and loading chunks
 	// undefined = chunk not loaded, null = chunk preloaded/prefetched
 	// Promise = chunk loading, 0 = chunk loaded
 	var installedChunks = {
 		"manifest": 0
 	};

 	var deferredModules = [];

 	// The require function
 	function __webpack_require__(moduleId) {

 		// Check if module is in cache
 		if(installedModules[moduleId]) {
 			return installedModules[moduleId].exports;
 		}
 		// Create a new module (and put it into the cache)
 		var module = installedModules[moduleId] = {
 			i: moduleId,
 			l: false,
 			exports: {}
 		};

 		// Execute the module function
 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

 		// Flag the module as loaded
 		module.l = true;

 		// Return the exports of the module
 		return module.exports;
 	}


 	// expose the modules object (__webpack_modules__)
 	__webpack_require__.m = modules;

 	// expose the module cache
 	__webpack_require__.c = installedModules;

 	// define getter function for harmony exports
 	__webpack_require__.d = function(exports, name, getter) {
 		if(!__webpack_require__.o(exports, name)) {
 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
 		}
 	};

 	// define __esModule on exports
 	__webpack_require__.r = function(exports) {
 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
 		}
 		Object.defineProperty(exports, '__esModule', { value: true });
 	};

 	// create a fake namespace object
 	// mode & 1: value is a module id, require it
 	// mode & 2: merge all properties of value into the ns
 	// mode & 4: return value when already ns object
 	// mode & 8|1: behave like require
 	__webpack_require__.t = function(value, mode) {
 		if(mode & 1) value = __webpack_require__(value);
 		if(mode & 8) return value;
 		if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
 		var ns = Object.create(null);
 		__webpack_require__.r(ns);
 		Object.defineProperty(ns, 'default', { enumerable: true, value: value });
 		if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
 		return ns;
 	};

 	// getDefaultExport function for compatibility with non-harmony modules
 	__webpack_require__.n = function(module) {
 		var getter = module && module.__esModule ?
 			function getDefault() { return module['default']; } :
 			function getModuleExports() { return module; };
 		__webpack_require__.d(getter, 'a', getter);
 		return getter;
 	};

 	// Object.prototype.hasOwnProperty.call
 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };

 	// __webpack_public_path__
 	__webpack_require__.p = "";

 	var jsonpArray = window["webpackJsonp"] = window["webpackJsonp"] || [];
 	var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
 	jsonpArray.push = webpackJsonpCallback;
 	jsonpArray = jsonpArray.slice();
 	for(var i = 0; i < jsonpArray.length; i++) webpackJsonpCallback(jsonpArray[i]);
 	var parentJsonpFunction = oldJsonpFunction;


 	// run deferred modules from other chunks
 	checkDeferredModules();
 })

 ([]);
```



optimization配置

```javascript
        optimization: { // 抽离共用部分
            minimizer: [new OptimizeCSSAssetsPlugin()],
            namedChunks: true, //除了 moduleId，我们知道分离出的 chunk 也有其 chunkId。同样的，chunkId 也有因其 chunkId 发生变化而导致缓存失效的问题。由于manifest与打出的 chunk 包中有chunkId相关数据，所以一旦如“增删页面”这样的操作导致 chunkId 发生变化，可能会影响很多的 chunk 缓存失效。
            runtimeChunk: { //在 webpack4 之前，抽离 manifest，需要使用 CommonsChunkPlugin，配置一个指定 name 属性为'manifest'的 chunk。在 webpack4 中，无需手动引入插件，配置 runtimeChunk 即可。
                name: 'manifest'
            },
            moduleIds: 'hashed', //项目工程中加载的 module，webpack 会为其分配一个 moduleId，映射对应的模块。这样产生的问题是一旦工程中模块有增删或者顺序变化，moduleId 就会发生变化，进而可能影响所有 chunk 的 content hash 值。只是因为 moduleId 变化就导致缓存失效，这肯定不是我们想要的结果，设置这个可以让 hash 值基本不变。
            splitChunks: {
                chunks: "all",
                cacheGroups: {
                    verdor: {
                        name: "vendor", // 打包后的文件名，任意命名
                        test: /node_modules/, // 匹配路径
                        priority: 10, // 权重
                        chunks: "initial" // 只打包初始时依赖的第三方
                        // all 把动态和非动态模块同时进行优化打包；所有模块都扔到 vendors.bundle.js 里面。 
                        // initial 把非动态模块打包进 vendor，动态模块优化打包。
                        // async 把动态模块打包进 vendor，非动态模块保持原样（不优化）
                    },
                    elementUI: {
                        name: "elementUI", // 单独将 elementUI 拆包
                        priority: 20, // 权重要大，不然会被打包进其他的
                        test: /node_modules\/element-ui/
                    },
                    commons: {
                        name: "commons",
                        test: resolve("src/components"), // 可自定义拓展你的规则
                        minChunks: 2, // 最小共用次数
                        priority: 5,
                        reuseExistingChunk: true
                    }
                }
            }
        }
```

配置了分包策略后，tree shaking失效了，需要使用一个支持删除死代码的压缩器。使用`install terser-webpack-plugin`插件

**编辑webpack.prod.js文件**

```javascript
...
const TerserPlugin = require('terser-webpack-plugin');
...
  optimization: { // 抽离共用部分
            minimizer: [new TerserPlugin(), new OptimizeCSSAssetsPlugin()],
  }
...
```



**最后的package.json文件**

```json
{
  "name": "webpack4_babel7_vue2.x",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "dev": "webpack-dev-server --config ./config/webpack.dev.js",
    "build": "cross-env NODE_ENV=production webpack --config  ./config/webpack.prod.js"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/qinhanwen/webpack4-babel7-vue2.git"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "axios": "^0.19.0",
    "element-ui": "^2.12.0",
    "qs": "^6.9.1",
    "vue": "^2.6.10",
    "vue-router": "^3.1.3"
  },
  "devDependencies": {
    "@babel/core": "^7.7.2",
    "@babel/plugin-transform-runtime": "^7.6.2",
    "@babel/preset-env": "^7.7.1",
    "@babel/runtime-corejs2": "^7.7.2",
    "autoprefixer": "^9.7.1",
    "babel-loader": "^8.0.6",
    "clean-webpack-plugin": "^3.0.0",
    "cross-env": "^6.0.3",
    "css-loader": "^3.2.0",
    "file-loader": "^4.2.0",
    "hard-source-webpack-plugin": "^0.13.1",
    "html-webpack-plugin": "^3.2.0",
    "image-webpack-loader": "^6.0.0",
    "mini-css-extract-plugin": "^0.8.0",
    "node-sass": "^4.13.0",
    "optimize-css-assets-webpack-plugin": "^5.0.3",
    "postcss-loader": "^3.0.0",
    "sass-loader": "^8.0.0",
    "sass-resources-loader": "^2.0.1",
    "style-loader": "^1.0.0",
    "terser-webpack-plugin": "^2.2.1",
    "url-loader": "^2.2.0",
    "vue-loader": "^15.7.2",
    "vue-style-loader": "^4.1.2",
    "vue-template-compiler": "^2.6.10",
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.10",
    "webpack-dev-server": "^3.9.0",
    "webpack-merge": "^4.2.2"
  }
}
```

**编辑webpack.base.js文件**

```javascript
const VueLoaderPlugin = require('vue-loader/lib/plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const HardSourceWebpackPlugin = require('hard-source-webpack-plugin');
const {
    resolve,
    isDevMode
} = require('./utils');

const devMode = isDevMode();

module.exports = {
    entry: resolve('./src/index.js'),
    resolve: { // 配置别名
        // 扩展名，比如import 'app.vue'，扩展后只需要写成import 'app'就可以了
        extensions: ['.js', '.vue', '.scss'],
        // 取路径别名，方便在业务代码中import
        alias: {
            views: resolve('src/views/'),
            components: resolve('src/components/'),
            directives: resolve('src/directives/'),
            filters: resolve('src/filters/'),
            mixins: resolve('src/mixins/'),
            services: resolve('src/services/'),
            assets: resolve('src/assets/'),
        }
    },
    module: {
        rules: [{
                test: /\.vue$/,
                exclude: /node_modules/,
                loader: 'vue-loader',
                options: {
                    cacheBusting: true
                }
            },
            {
                test: /\.(css|scss)$/,
                use: [{
                        loader: devMode ? 'vue-style-loader' : MiniCssExtractPlugin.loader,
                        options: devMode ? {} : {//解决css图片路径问题
                            publicPath: '../../',
                        }
                    },
                    'css-loader',
                    'postcss-loader',
                    'sass-loader',
                    {
                        loader: 'sass-resources-loader',
                        options: {
                            sourceMap: true,
                            resources: [resolve('src/common/index.scss')]
                        }
                    }
                ],
            },
            { // 图片资源太小转成内联，减少http请求
                test: /\.(jpg|jpeg|png)$/,
                use: [{
                        loader: 'url-loader', // 是对file-loader的封装
                        options: {
                            limit: 1024, // 如果小于10k转成base64
                            name: '[name].[ext]',
                            outputPath: "static/images", //webpack打包后文件的输出目录
                        }
                    },
                    {
                        loader: 'image-webpack-loader', // 压缩图片
                        options: {
                            mozjpeg: {
                                progressive: true,
                                quality: 65
                            },
                            // optipng.enabled: false will disable optipng
                            optipng: {
                                enabled: false,
                            },
                            pngquant: {
                                quality: [0.65, 0.90],
                                speed: 4
                            },
                            gifsicle: {
                                interlaced: false,
                            },
                            // the webp option will enable WEBP
                            webp: {
                                quality: 75
                            }
                        }
                    }
                ]
            },
            {
                test: /\.js$/,
                use: 'babel-loader',
                include: /src/, // 只转化src目录下的js
                exclude: /node_modules/ // 排除掉node_modules，优化打包速度
            },
            {
                test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
                loader: 'url-loader',
                options: {
                    limit: 10000,
                    outputPath: 'static/fonts/', //输出到images文件夹
                }
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            filename: 'index.html', // 配置输出文件名和路径
            template: resolve('./src/index.html'), // 配置要被编译的html文件
            minify: { // 压缩
                removeAttributeQuotes: true, //删除双引号
                collapseWhitespace: true //折叠 html 为一行
            }
        }),
        new VueLoaderPlugin(),
        new HardSourceWebpackPlugin()
    ],
}
```



**编辑webpack.dev.js文件**

```javascript
const webpackMerge = require("webpack-merge");
const webpackBaseConfig = require('./webpack.base');

const {
    resolve
} = require('./utils');

const config = require('./config');

module.exports = () => {
    const devConfig = {
        mode: 'development',
        output: { //输出文件
            filename: "static/js/[name].js", //文件名
            path: resolve('./dist') //路径
        },
        devtool: 'eval-source-map', //生成用于开发环境的最佳品质的 source map
        devServer: { // 开发服务器配置
            port: config.PORT, // 端口号
            host: config.HOST, //域名
            progress: config.PROCESS, // 进度条
            // contentBase: config.CONTENT_BASE, // 服务默认指向文件夹
            inline: config.INLINE, // 设置为true，当源文件改变的时候会自动刷新
            historyApiFallback: config.HISTORY_API_FALLBACK, // 在开发单页应用时非常有用，它依赖于HTML5 history API，如果设置为true，所有的跳转将指向index.html
            hot: config.AUTO_OPEN_BROWER, // 允许热加载
            open: config.AUTO_OPEN_BROWER // 自动打开浏览器
        },
        watch: true,
        watchOptions: {
            ignored: config.IGNORED, //正则匹配不监听的文件夹，默认为空
            aggregateTimeout: config.AGGREGATE_TIMEOUT, //监听到文件的最后编辑时间变化等待300ms后去执行，默认300ms
            // poll: 1000, //判断文件是否发生变化是通过不停的轮询系统指定的文件有没有变化实现的，默认1000ms轮询一次
        }
    }
    return webpackMerge(webpackBaseConfig, devConfig);
}
```

**编辑webpack.prod.js文件**

```javascript
const webpackMerge = require("webpack-merge");
const webpackBaseConfig = require('./webpack.base');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');

const {
    resolve
} = require('./utils');

const {
    CleanWebpackPlugin
} = require('clean-webpack-plugin') // 自动清除


module.exports = () => {

    const prodConfig = {
        mode: 'production',
        output: { //输出文件
            filename: "static/js/[name].[chunkhash:8].js", //文件名
            path: resolve('./dist') //路径
        },
        performance: {
            hints: 'warning',
            //入口起点的最大体积
            maxEntrypointSize: 50000000,
            //生成文件的最大体积
            maxAssetSize: 30000000,
            //只给出 js 文件的性能提示
            assetFilter: function (assetFilename) {
                return assetFilename.endsWith('.js');
            }
        },
        plugins: [
            new MiniCssExtractPlugin({
                filename: 'static/css/base.[contenthash:8].css',
            }),
            new CleanWebpackPlugin({
                cleanAfterEveryBuildPatterns: ['dist'],
                verbose: true
            })
        ],
        optimization: { // 抽离共用部分
            minimizer: [new TerserPlugin(), new OptimizeCSSAssetsPlugin()],
            namedChunks: true, //除了 moduleId，我们知道分离出的 chunk 也有其 chunkId。同样的，chunkId 也有因其 chunkId 发生变化而导致缓存失效的问题。由于manifest与打出的 chunk 包中有chunkId相关数据，所以一旦如“增删页面”这样的操作导致 chunkId 发生变化，可能会影响很多的 chunk 缓存失效。
            runtimeChunk: { //在 webpack4 之前，抽离 manifest，需要使用 CommonsChunkPlugin，配置一个指定 name 属性为'manifest'的 chunk。在 webpack4 中，无需手动引入插件，配置 runtimeChunk 即可。
                name: 'manifest'
            },
            moduleIds: 'hashed', //项目工程中加载的 module，webpack 会为其分配一个 moduleId，映射对应的模块。这样产生的问题是一旦工程中模块有增删或者顺序变化，moduleId 就会发生变化，进而可能影响所有 chunk 的 content hash 值。只是因为 moduleId 变化就导致缓存失效，这肯定不是我们想要的结果，设置这个可以让 hash 值基本不变。
            splitChunks: {
                chunks: "all",
                cacheGroups: {
                    verdor: {
                        name: "vendor", // 打包后的文件名，任意命名
                        test: /node_modules/, // 匹配路径
                        priority: 10, // 权重
                        chunks: "initial" // 只打包初始时依赖的第三方
                        // all 把动态和非动态模块同时进行优化打包；所有模块都扔到 vendors.bundle.js 里面。 
                        // initial 把非动态模块打包进 vendor，动态模块优化打包。
                        // async 把动态模块打包进 vendor，非动态模块保持原样（不优化）
                    },
                    elementUI: {
                        name: "elementUI", // 单独将 elementUI 拆包
                        priority: 20, // 权重要大，不然会被打包进其他的
                        test: /node_modules\/element-ui/
                    },
                    commons: {
                        name: "commons",
                        test: resolve("src/components"), // 可自定义拓展你的规则
                        minChunks: 2, // 最小共用次数
                        priority: 5,
                        reuseExistingChunk: true
                    }
                }
            }
        }
    }
    return webpackMerge(webpackBaseConfig, prodConfig);
}
```

**编辑config.js文件**

```javascript
module.exports = {
    //devserver
    PORT: 3000,
    HOST: 'localhost',
    AUTO_OPEN_BROWER: true,
    HOT: true,
    PROCESS: true,
    INLINE:true,
    HISTORY_API_FALLBACK:true,
    CONTENT_BASE:'./static',

    //watch
    WATCH: true,

    //watchoptions
    IGNORED:'/node_modules/',
    AGGREGATE_TIMEOUT:300
}
```

**编辑.babelrc文件**

```json
{
    "presets": [
        [
            "@babel/preset-env",
            {
                "modules": false
            }
        ]
    ],
    "plugins": [
        ["@babel/plugin-transform-runtime", {
            "corejs": 2
        }]
    ]
}
```

**编辑postcss.config.js文件**

```javascript
module.exports = {
    plugins: [
      require('autoprefixer')
    ],
    // 配置autoprefix
    browsers: [
      "> 1%",
      "last 2 versions",
      "ie >= 10"
    ]
  }
```

**编辑utils.js**

```javascript
const path = require('path');

// 减少路径书写
function resolve(dir) {
    return path.join(__dirname,'..', dir)
}
function isDevMode(){
    return process.env.NODE_ENV !== 'production';
}
module.exports = {
    resolve,
    isDevMode
}
```



##### DLLPlugin

DLL：动态链接库

- 将网页依赖的基础模块抽离出来，打包到一个个单独的动态链接库中，一个动态链接库中可以包含多个模块
- 当需要导入的模块存在于某个动态链接库中时，这个模块不能再次被打包，而是去动态链接库中获取
- 页面依赖的所有动态链接库都需要被加载

也就是说webpack打包时，有一些框架代码是基本不变的，比如说vue、vue-router、vuex、axios、element-ui 等，这些模块也有不小的 size，每次编译都要加载一遍，比较费时费力。



没有选择使用[DLLPlugin](https://www.webpackjs.com/plugins/dll-plugin/)

也没有选择[AutoDllPlugin](https://github.com/asfktz/autodll-webpack-plugin)插件，虽然减少了大量的配置文件，但是路径容易出问题，加速也不明显。

最后选择[HardSourceWebpackPlugin](https://github.com/mzgoddard/hard-source-webpack-plugin)插件：

配置之前打包时间大概  **4916ms**

![WX20191121-104615@2x](http://118.24.241.76/WX20191121-104615@2x.png)



配置文件，很简单的配置

```javascript
const HardSourceWebpackPlugin = require('hard-source-webpack-plugin');

module.exports = {
  // ......
  plugins: [
    new HardSourceWebpackPlugin()
  ]
}
```

![WX20191121-105742@2x](http://118.24.241.76/WX20191121-105742@2x.png)

文档说需要打包两次，才能看出来效果，那就看第二次构建的效果，打包时间大概 **2577ms**

![WX20191121-105921@2x](http://118.24.241.76/WX20191121-105921@2x.png)



**HappyPack**

由于大量文件需要解析和处理，当文件数量多的时候，webpack构建慢的问题会显得更为严重。运行在Node环境的webpack是单线程模型，不能同时处理多任务。而HappyPack能让webpack做到这一点。它将任务分解给多个子进程并发执行，子进程处理完后再将结果送给主进程。

然而项目较小时，多线程打包反而会使打包速度变慢，所以这边**只是补充代码，项目没有使用**。

配置文件（这种使用方法下一定要在根目录下加`.babelrc`文件来设置`babel`的打包配置）

```javascript
const HappyPack = require('happypack');
const os = require('os'); //获取电脑的处理器有几个核心，作为配置传入
const happyThreadPool = HappyPack.ThreadPool({
    size: os.cpus().length
});
...

          {
                test: /\.js$/,
                use: ['happypack/loader?id=happy-babel-js'],
                include: /src/, // 只转化src目录下的js
                exclude: /node_modules/ // 排除掉node_modules，优化打包速度
          },
...

		    new HappyPack({ //开启多线程打包
            id: 'happy-babel-js',
            loaders: ['babel-loader?cacheDirectory=true'],
            threadPool: happyThreadPool
        }),
...
```



**打包分析优化报表**

```shell
$ npm install webpack-bundle-analyzer -D
```

配置

```javascript
...
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
 ...

  plugins: [
    ...
    new BundleAnalyzerPlugin()
  ]
...
```

可以从图中分析哪些模块太大，和如何优化。

![WX20191123-200013@2x](http://118.24.241.76/WX20191123-200013@2x.png)



比如这边lodash，我只使用了一个方法，有60多kb，因为在引入的时候只是引入了lodash的某个方法

```javascript
import cloneDeepWith from 'lodash/cloneDeepWith';
```

如果变成这样，会导入整个lodash，有500多kb

```javascript
import {cloneDeepWith} from 'lodash';
```

![WX20191123-200351@2x](http://118.24.241.76/WX20191123-200351@2x.png)



**路由懒加载**

构建配置

```javascript
...
  output: { //输出文件
    chunkFilename:"static/js/[name].[chunkhash:8].js",//配置懒加载需要
    filename: "static/js/[name].[chunkhash:8].js", //文件名
    path: resolve('./dist') //路径
  },
...
```

路由文件配置

```javascript
import Vue from 'vue';
import VueRouter from 'vue-router';

//方式一
const Home = r => require.ensure([], () => r(require('views/home/home')), 'home');
const Login = r => require.ensure([], () => r(require('views/login/login')), 'login');

//方式二
//const Home = () => import('views/home/home.vue')
//const Login = () => import('views/login/login.vue')

Vue.use(VueRouter);

export default new VueRouter({
  mode: 'hash',
  routes: [{
      path: '/',
      component: Login
    },
    {
      path: '/home',
      component: Home
    },
  ]
})
```

其中有遇到个报错，关于`hard-source-webpack-plugin`插件的

![WX20191124-234008@2x](http://118.24.241.76/WX20191124-234008@2x.png)



解决方案：删除`node_modules/.cache`文件



## Vue

**先安装vue与vue-router**

```shell
$ npm install vue vue-router -S
```

略



## 接入[ELementUI](https://element.eleme.cn/#/zh-CN/guide/design)

**安装**

```shell
$ npm i element-ui -S
```



## 接入axios

**安装**

```shell
$ npm install axios -S
$ npm install qs -S
```

封装一下

```javascript
import axios from 'axios';
import qs from 'qs';

const config = {
    prod: '',
    dev: ''
}

const showStatus = (status) => {
    let message = ''
    switch (status) {
        case 400:
            message = '请求错误(400)'
            break
        case 401:
            message = '未授权，请重新登录(401)'
            break
        case 403:
            message = '拒绝访问(403)'
            break
        case 404:
            message = '请求出错(404)'
            break
        case 408:
            message = '请求超时(408)'
            break
        case 500:
            message = '服务器错误(500)'
            break
        case 501:
            message = '服务未实现(501)'
            break
        case 502:
            message = '网络错误(502)'
            break
        case 503:
            message = '服务不可用(503)'
            break
        case 504:
            message = '网络超时(504)'
            break
        case 505:
            message = 'HTTP版本不受支持(505)'
            break
        default:
            message = `连接出错(${status})!`
    }
    return `${message}，请检查网络或联系管理员！`
}

const service = axios.create({
    headers: {
        get: {
            'Content-Type': 'application/x-www-form-urlencoded;charset=utf-8'
        },
        post: {
            'Content-Type': 'application/json;charset=utf-8'
        }
    },
    timeout: 20000,
    transformRequest: [(data) => {
        data = JSON.stringify(data)
        return data
    }],
    validateStatus() {
        // 使用async-await，处理reject情况较为繁琐，所以全部返回resolve，在业务代码中处理异常
        return true
    },
    transformResponse: [(data) => {
        if (typeof data === 'string' && data.startsWith('{')) {
            data = JSON.parse(data)
        }
        return data
    }]
})

// 请求拦截器
service.interceptors.request.use((config) => {
    return config
}, (error) => {
    // 错误抛到业务代码
    error.data = {}
    error.data.msg = '服务器异常，请联系管理员！'
    return Promise.resolve(error)
})

// 响应拦截器
service.interceptors.response.use((response) => {
    const status = response.status
    let msg = ''
    if (status < 200 || status >= 300) {
        // 处理http错误，抛到业务代码
        msg = showStatus(status)
        if (typeof response.data === 'string') {
            response.data = {
                msg
            }
        } else {
            response.data.msg = msg
        }
    }
    return response
}, (error) => {
    // 错误抛到业务代码
    error.data = {}
    error.data.msg = '请求超时或服务器异常，请检查网络或联系管理员！'
    return Promise.resolve(error)
})

const get = (url, params) => {
    const getUrl = `${config.dev}${url}${qs.stringify(params)}`;
    return service.get(getUrl);
}

const post = (url, params) => {
    const postUrl = `${config.dev}${url}`;
    return service.post(postUrl, params)
}

export default {
    get,
    post
}
```



## [devServer.proxy](https://webpack.js.org/configuration/dev-server/#devserverproxy)

当您拥有单独的API后端开发服务器，并且希望在同一域上发送API请求时，代理某些URL可能会很有用。也就是解决跨域问题。



deveServer配置

```javascript
...
   devServer: {
      proxy: {
        '/api': {
          target: 'http://localhost:3001',
          pathRewrite: { '^/api': '' }, // 重写路径不需要 api
          changeOrigin: true, //本地起虚拟服务器接收请求并转发
        }
      },
   }
...
```



后端服务

```javascript
const koa = require('koa')
const app = new koa()

const Router = require('koa-router')
const router = new Router()

router.get('/cors', async (ctx, next) => {
  ctx.body = {
    success: true
  }
})

app.use(router.routes())

app.listen(3001)
console.log('koa server is listening port 3001')

```



## 项目地址

https://github.com/qinhanwen/webpack4-babel7-vue2



## 续集

[ssr配置](https://qinhanwen.github.io/2019/11/20/vue-ssr/#more)



## 参考

[手摸手，带你用合理的姿势使用webpack4（上）](https://juejin.im/post/5b56909a518825195f499806)

[webpack4.0 入门篇 - 构建前端开发的基本环境](https://juejin.im/post/5bb089e86fb9a05cd84935d0)

[webpack - babel 篇](https://juejin.im/post/5bfe541bf265da6179748834)

[mode的production与development区别](https://juejin.im/post/5bc80e09f265da0ac07c84ed)

[webpack 、manifest 、runtime 、缓存与CommonsChunkPlugin](https://www.jianshu.com/p/95752b101582)

[webapck4 玄妙的 SplitChunks Plugin](https://juejin.im/post/5c08fe7d6fb9a04a0d56a702)

[webpack4提升180%编译速度](https://www.cnblogs.com/yangsg/p/10601604.html)

[webpack可视化打包性能分析插件](https://www.jianshu.com/p/5381df3c2afa)

[深入浅出 Babel 上篇：架构和原理 + 实战](https://juejin.im/post/5d94bfbf5188256db95589be)