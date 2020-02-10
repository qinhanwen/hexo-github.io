---
title: webpack构建过程-补充
date: 2020-01-17 12:41:30
tags: 
- webpack
categories: 
- webpack
---

## 补充

添加了一个简单的 loader，替换 const 为 var

```javascript
module.exports = function loader(source) {
    return source.replace('const','var');
};
```

```javascript
const path = require('path')
const MyPlugin = require('./my-webpack-plugin')

const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  mode: 'development', //打包模式 development和production，是否压缩打包出来的文件
  entry: './index.js', //从哪个文件开始打包
  module: {
    rules: [{
      test: /\.js$/,
      loader: path.resolve(__dirname, './my-loader.js'),
    }]
  },
  plugins: [
    new CleanWebpackPlugin({
      cleanAfterEveryBuildPatterns: ['dist'],
      verbose: true
    }),
    new MyPlugin()
  ],
  output: {
    //输出
    filename: '[hash].js', //最终文件命名
    path: path.resolve(__dirname, 'dist') //输出文件夹的路径
  }
}
```



- 执行 webpack 执行文件
- 解析 shell 参数，如果有 config 配置，取得配置并初始化。生成 compiler 实例，遍历 plugins 数组如果是对象就调用 apply 方法
- 进入 compiler.run 开始执行编译
- 拿到 loaders 数组，进入 runloader 方法，之后 loadLoader，也就是用 require 加载 loader 的文件。之后读入口文件，转成 Buffer。

![WX20200117-141124@2x](http://114.55.30.96/WX20200117-141124@2x.png)

在调用loader处理的时候，又调用 utf8BufferToString，把 Buffer 转成字符串

![WX20200117-142045@2x](http://114.55.30.96/WX20200117-142045@2x.png)

进入 loader 的调用

![WX20200117-143820@2x](http://114.55.30.96/WX20200117-143820@2x.png)



- 进入解析阶段，对 loader 处理完的数据进行解析，解析成 ast

![WX20200117-144310@2x](http://114.55.30.96/WX20200117-144310@2x.png)

如果例子改成这样，也就是依赖其他模块的话

```javascript
import a from './index1';
console.log(a)
```

在解析的时候是这样的过程

![WX20200117-145652@2x](http://114.55.30.96/WX20200117-145652@2x.png)

添加依赖模块，之后再解析到这个依赖的模块里

![WX20200117-145844@2x](http://114.55.30.96/WX20200117-145844@2x.png)



- 进入 seal 阶段， 遍历 entrys，生成对应的 chunk

![WX20200117-165529@2x](http://114.55.30.96/WX20200117-165529@2x.png)

之后在 createChunkAssets 方法里遍历 chunk 数组。在配置文件中输出的名字使用的是  **[hash].js**，它在这生成的。就是 replace 正则匹配替换

![WX20200117-165942@2x](http://114.55.30.96/WX20200117-165942@2x.png)

![WX20200117-165847@2x](http://114.55.30.96/WX20200117-165847@2x.png)

```javascript
const replacePathVariables = (path, data, assetInfo) => {
	const chunk = data.chunk;
	const chunkId = chunk && chunk.id;
	const chunkName = chunk && (chunk.name || chunk.id);
	const chunkHash = chunk && (chunk.renderedHash || chunk.hash);
	const chunkHashWithLength = chunk && chunk.hashWithLength;
	const contentHashType = data.contentHashType;
	const contentHash =
		(chunk && chunk.contentHash && chunk.contentHash[contentHashType]) ||
		data.contentHash;
	const contentHashWithLength =
		(chunk &&
			chunk.contentHashWithLength &&
			chunk.contentHashWithLength[contentHashType]) ||
		data.contentHashWithLength;
	const module = data.module;
	const moduleId = module && module.id;
	const moduleHash = module && (module.renderedHash || module.hash);
	const moduleHashWithLength = module && module.hashWithLength;

	if (typeof path === "function") {
		path = path(data);
	}

	if (
		data.noChunkHash &&
		(REGEXP_CHUNKHASH_FOR_TEST.test(path) ||
			REGEXP_CONTENTHASH_FOR_TEST.test(path))
	) {
		throw new Error(
			`Cannot use [chunkhash] or [contenthash] for chunk in '${path}' (use [hash] instead)`
		);
	}

	return (
		path
			.replace(
				REGEXP_HASH,
				withHashLength(getReplacer(data.hash), data.hashWithLength, assetInfo)
			)
			.replace(
				REGEXP_CHUNKHASH,
				withHashLength(getReplacer(chunkHash), chunkHashWithLength, assetInfo)
			)
			.replace(
				REGEXP_CONTENTHASH,
				withHashLength(
					getReplacer(contentHash),
					contentHashWithLength,
					assetInfo
				)
			)
			.replace(
				REGEXP_MODULEHASH,
				withHashLength(getReplacer(moduleHash), moduleHashWithLength, assetInfo)
			)
			.replace(REGEXP_ID, getReplacer(chunkId))
			.replace(REGEXP_MODULEID, getReplacer(moduleId))
			.replace(REGEXP_NAME, getReplacer(chunkName))
			.replace(REGEXP_FILE, getReplacer(data.filename))
			.replace(REGEXP_FILEBASE, getReplacer(data.basename))
			// query is optional, it's OK if it's in a path but there's nothing to replace it with
			.replace(REGEXP_QUERY, getReplacer(data.query, true))
			// only available in sourceMappingURLComment
			.replace(REGEXP_URL, getReplacer(data.url))
			.replace(/\[\\(\\*[\w:]+\\*)\\\]/gi, "[$1]")
	);
};
```



之后进入 manifest 模板的 render，一个数组用来存一些固定字符串

![WX20200117-172244@2x](http://114.55.30.96/WX20200117-172244@2x.png)





最后将所有模块中的`require`语法替换成`__webpack_require__`来模拟模块化操作。

![WX20200117-175056@2x](http://114.55.30.96/WX20200117-175056@2x.png)

![WX20200117-174845@2x](http://114.55.30.96/WX20200117-174845@2x.png)



- 文件输出阶段，根据模板，往数组里 push 字符串，最后遍历一下拼接起来，通过 fs 文件系统写文件