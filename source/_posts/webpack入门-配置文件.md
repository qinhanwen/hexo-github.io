---
title: webpack入门（2）-配置文件
date: 2019-06-26 08:52:14
tags: 
- webpack
categories: 
- webpack
---

## 配置文件

##### [entry](https://webpack.js.org/concepts/entry-points/)与[output](https://webpack.js.org/concepts/output/)

修改之前的文件路径，新增webpack.config.js文件（因为每个项目工程目录不同，所以可以根据需要自己的配置），业务逻辑代码放在src下面。

```shell
$ webpack --config webpack.config.js
#设置配置文件
```

文件目录

```
  |—— dist
    |—— index.html
  |—— src
    |—— a.js
    |—— b.js
    |—— c.js
    |—— index.js
  |—— webpack.config.js
```



webpack.config.js

```javascript
const path = require('path');
module.exports = {
    entry:"./src/index.js",//从哪个文件开始打包
    output:{//输出
        filename:'bundle.js',//最终文件命名
        path: path.resolve(__dirname,'dist'),//输出文件夹的路径
    }
}
```



##### 打包JS文件

```shell
$ npx webpack 
```



**也可以配置package.json文件的scripts**

```
{
  "name": "webpack-study",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "webpack"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {},
  "devDependencies": {
    "webpack": "^4.35.0",
    "webpack-cli": "^3.3.5"
  }
}
```

```shell
$ npm run build
```

**提示说明**

![WX20190627-205301@2x](http://114.55.30.96/WX20190627-205301@2x.png)

Hash：本次打包对应的唯一hash值

Version：使用的webpack版本

Time：打包耗时

Build at：打包时间

Asset：打包出来的文件 

Size：文件的大小

Chunks：每个打包出来文件对应的id

Chunk Names：每个打包出来文件对应的名字

Entrypoint：入口文件

```javascript
entry:"./src/index.js",//从哪个文件开始打包
//等价于
entry:{
  main:"./src/index.js"
}
```

如果`output`没设置文件名，则打包出来文件就是main.js (打包出来的文件名称就是entry设置的**对象的键名.js** )，在打包多个文件的时候可以设置

```js
...
entry:{//入口
  main:'./src/index.js',
  sub:'./src/index.js'
}
...
 output: {//输出
   filename: '[name].js',//最终文件命名
   path: path.resolve(__dirname, 'dist'),//输出文件夹的路径
 }
...
```

打包出来的文件是`main.js`和`sub.js`，在`index.html`文件里对这两个js文件的引用，都是在根目录下。



**publicPath**

如果之后想把js文件布置到`cdn`或者`服务器`上，可以通过设置`publicPath`输出属性，为引用js文件自动添加前缀。

```json
...
output: {//输出
  publicPath: 'http://www.qinhanwen.xyz',
  filename: '[name].js',//最终文件命名
  path: path.resolve(__dirname, 'dist'),//输出文件夹的路径
}
...
```



**`library`和`libraryTaget`**

```javascript
...
    output: {//输出
        // publicPath: 'http://www.qinhanwen.xyz',
        filename: '[name].js',//最终文件命名
        path: path.resolve(__dirname, 'dist'),//输出文件夹的路径
        library:'eft',
        libraryTarget:'umd'
    }
...
```

加了这两个之后编译成这样，`libraryTarget:'umd'`是兼容各种模块化规范，比如AMD，CMD，CommonJS，还有`library:'eft'`是暴露在某个命名空间下（使用script标签引入的方式）。

![WX20190728-140741@2x](http://114.55.30.96/WX20190728-140741@2x.png)



##### mode

```javascript
const path = require('path');
module.exports = {
  	mode:'development',//打包模式 development和production（默认模式），是否压缩打包出来的文件
    entry:"./src/index.js",//从哪个文件开始打包
    output:{//输出
        filename:'bundle.js',//最终文件命名
        path: path.resolve(__dirname,'dist'),//输出文件夹的路径
    }
}
```



##### loader

用于告诉webpack，对于**特定的文件（非js文件）**使用怎样的处理方式

举个例子，打包`图片类型`和`scss类型`文件

1）新建文件，目录如下

```
  |—— assets
    |—— images
      |—— WX20190627-205301@2x.png
  |—— dist
    |—— index.html
  |—— src
    |—— a.js
    |—— index.js
    |—— index.scss
  |—— webpack.config.js
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
    <img id="img" />
</body>
<script src="./bundle.js">

</script>

</html>
```

index.js

```javascript
import A from './a';
import img from '../assets/images/WX20190627-205301@2x.png';
import './index.scss';

document.getElementById('img').src = img;
new A();
```

a.js

```javascript
export default function A(){
    console.log("module A");
}
```

index.scss

```scss
body{
    img{
        height: 200px;
        width: 200px;
    }
}
```

以及一张 WX20190627-205301@2x.png 图片



2）安装`loader`

```shell
$ npm install file-loader --save-dev
$ npm install css-loader --save-dev
$ npm install style-loader --save-dev

#编译scss
$ npm install sass-loader node-sass --save-dev
```

 备注：`postcss-loader` 可以在打包的时候为各种存在兼容性问题的css新特性增加前缀（-webkit-，-mozila-）等等。



3）修改webpack.config.js文件

```javascript
const path = require('path');
module.exports = {
    mode: 'development',//打包模式 development和production，是否压缩打包出来的文件
    entry: "./src/index.js",//从哪个文件开始打包
    module: {
        rules: [{
            test: /\.(jpg|png)$/,//匹配规则
            use: {
                loader: 'file-loader',//使用的loader
                options: {
                    name: '[name]_[hash].[ext]',//文件名称
                    outputPath: 'image/'//打包到哪个路径下
                }
            }
        }, {
            test: /\.scss$/,//匹配规则
            use: ['style-loader','css-loader','sass-loader'],//使用的loader
        }]
    },
    output: {//输出
        filename: 'bundle.js',//最终文件命名
        path: path.resolve(__dirname, 'dist'),//输出文件夹的路径
    }
}
```

loader有引入顺序，从右到左，从下到上。

[name]，[ext]，[hash]这类是占位符，具体可以看[文档](https://www.webpackjs.com/loaders/css-loader/)。



4）打包

```shell
$ npm run build
```

新的目录结构，dist目录下文件发生变化

```
  |—— assets
    |—— images
      |—— WX20190627-205301@2x.png
  |—— dist
    |—— bundle.js
    |—— image
      |—— WX20190627-205301@2x_25b6cf8a3379db4d1fc7bc8c2a3cc9bd.png
    |—— index.html
  |—— src
    |—— a.js
    |—— index.js
    |—— index.scss
  |—— webpack.config.js
```

打开index.html页面，看到head下插入了一个style标签。

![WX20190630-162033@2x](http://114.55.30.96/WX20190630-162033@2x.png)





##### plugins

作用：在打包运行到某些时刻的时候，可以做某些事情。



1）举个例子，先新建文件


public/index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    <img id="img" />
</body>
</html>
```

目录结构

```
  |—— assets
    |—— images
      |—— WX20190627-205301@2x.png
  |—— dist
  |—— public
    |—— index.html
  |—— src
    |—— a.js
    |—— index.js
    |—— index.scss
```



2）安装两个plugin

```shell
$ npm install html-webpack-plugin -D
$ npm install clean-webpack-plugin -D
```

这两个插件其实就是在打包的某个时刻，做某些事情，下面说这两个插件做了些什么。



3）修改webpack.config.js文件

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    mode: 'development',//打包模式 development和production，是否压缩打包出来的文件
    entry: "./src/index.js",//从哪个文件开始打包
    module: {
        rules: [{
            test: /\.(jpg|png)$/,//匹配规则
            use: {
                loader: 'file-loader',//使用的loader
                options: {
                    name: '[name]_[hash].[ext]',//文件名称
                    outputPath: 'image/'//打包到哪个路径下
                }
            }
        }, {
            test: /\.scss$/,//匹配规则
            use: ['style-loader', 'css-loader', 'sass-loader'],//使用的loader
        }]
    },
    plugins: [new HtmlWebpackPlugin({
        template: 'public/index.html'
    }), new CleanWebpackPlugin({ cleanAfterEveryBuildPatterns: ['dist'], verbose: true })],
    output: {//输出
        filename: 'bundle.js',//最终文件命名
        path: path.resolve(__dirname, 'dist'),//输出文件夹的路径
    }
}
```

html-webpack-plugin，设置了默认模板`public/index.html`（如果未设置template，会自动生成新的`index.html`）到dist目录下，打包完成之后会在生成的dist目录下的`index.html`中引入依赖文件。

clean-webpack-plugin，options设置了cleanAfterEveryBuildPatterns和verbose，清除了dist目录下的文件。



4）打包

```shell
$ npm run build
```

![WX20190630-231105@2x](http://114.55.30.96/WX20190630-231105@2x.png)



5）打开 `dist/index.html`

![WX20190630-231534@2x](http://114.55.30.96/WX20190630-231534@2x.png)

## 编译

##### 怎样在文件变化时，自动编译而不用手动执行编译。

1.添加`—watch`参数，可以在文件发生改变的时候时时编译，在`package.json`文件添加配置

```json
...
"scripts": {
    "build": "webpack --watch"
},
...
```



2.`webpackDevServer`监听文件变化，并且自动更新浏览器

安装`webpack-dev-server`，修改`package.json`和`webpack.config.js`

```shell
$ npm install webpack-dev-server -D
```

```json
  ...
  "scripts": {
    "watch":"webpack-dev-server"
  },
  ...
```

```javascript
...
devServer: {
    contentBase: "./dist",//服务器启在哪个目录下
    open: true,//自动打开浏览器
    port: 8000,//端口号
}
...
```

执行`npm run watch`的时候会自动打开浏览器`localhost:8000`，并且时时监听文件变，并且自动刷新浏览器化，打包的内容在内存中，没有在dist目录下。





##### babel

安装依赖包

```shell
#@babel/preset-env包含es6转es5的规则
$ npm install @babel/preset-env -D
$ npm install babel-loader -D
$ npm install @babel/core -D 

#安装@babel/polyfill，用于兼容低版本
$ npm install @babel/polyfill -D
```

修改`webpack.config.json`

```javascript
...   
   module: {
        rules: [{
            test: /\.js$/,
            exclude: '/node_modules/',//不包含的
            loader: 'babel-loader',//使用的loader
            options: {
                presets: [["@babel/preset-env",{
                	useBuiltIns:"usage"//当兼容低版本浏览器的时候，根据业务代码用到的特性，加入对应的方法，比如使用了promise，就添加低版本的promise实现的方式，这样子打包出来文件就不会太大
                }]]
            }
        }]
    },
...
```

[babel-preset-env其他配置](https://babeljs.io/docs/en/babel-preset-env)

遇到报错：[__webpack_require__(...) is not a function with babel-loader](https://github.com/jantimon/html-webpack-plugin/issues/1223)



**完整配置文件：**

```javascript
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    mode: 'development',//打包模式 development和production，是否压缩打包出来的文件
    entry: {//入口
        main: './index.js',
    },
    module: {
        rules: [{
            test: /\.(jpg|png)$/,//匹配规则
            use: {
                loader: 'file-loader',//使用的loader
                options: {
                    name: '[name]_[hash].[ext]',//文件名称
                    outputPath: 'image/'//打包到哪个路径下
                }
            }
        }, {
            test: /\.scss$/,//匹配规则
            use: ['style-loader', 'css-loader', 'sass-loader'],//使用的loader
        }, {
            test: /\.js$/,
            exclude: '/node_modules/',//不包含的
            loader: 'babel-loader',//使用的loader
            options: {
                presets: [["@babel/preset-env",{
                	useBuiltIns:"usage"//当兼容低版本浏览器的时候，根据业务代码用到的特性，加入对应的方法，比如使用了promise，就添加低版本的promise实现的方式
                }]]
            }
        }]
    },
    plugins: [
    //     new HtmlWebpackPlugin({
    //     template: 'public/index.html'
    // }), 
    new CleanWebpackPlugin({ cleanAfterEveryBuildPatterns: ['dist'], verbose: true })],
    output: {//输出
        // publicPath: 'http://www.qinhanwen.xyz',
        filename: '[name].js',//最终文件命名
        path: path.resolve(__dirname, 'dist'),//输出文件夹的路径
    },
    devServer: {
        contentBase: "./dist",//服务器启在哪个目录下
        open: true,//自动打开浏览器
        port: 8000,//端口号
    }
}
```

