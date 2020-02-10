---





title: webpack构建
date: 2019-08-11 00:21:52
tags: 
- webpack
categories: 
- webpack
---

[Tapable](https://qinhanwen.github.io/2019/10/21/tapable/#more)

## webpack构建过程

Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：

- webpack会读取你在命令行传入的配置以及项目里的 webpack.config.js 文件，初始化本次构建的配置参数，并且执行配置文件中的插件实例化语句，生成Compiler传入plugin的apply方法，为webpack事件流挂上自定义钩子。
- 接下来到了entryOption阶段，webpack开始读取配置的Entries，递归遍历所有的入口文件
- Webpack接下来就开始了compilation过程。会依次进入其中每一个入口文件(entry)，先使用用户配置好的loader对文件内容进行编译（buildModule），我们可以从传入事件回调的compilation上拿到module的resource（资源路径）、loaders（经过的loaders）等信息；之后，再将编译好的文件内容使用acorn解析生成AST静态语法树（normalModuleLoader），分析文件的依赖关系逐个拉取依赖模块并重复上述过程，最后将所有模块中的require语法替换成\_\_webpack_require\_\_来模拟模块化操作。
- emit阶段，所有文件的编译及转化都已经完成，包含了最终输出的资源，我们可以在传入事件回调的compilation.assets 上拿到所需数据，其中包括即将输出的资源、代码块Chunk等等信息

**compilation：**一个 `Compilation` 对象包含了当前的模块资源、编译生成资源、变化的文件等。`Compilation` 对象也提供了很多事件回调供插件做扩展。

**Compiler：**`Compiler` 负责文件监听和启动编译。`Compiler` 实例中包含了完整的 `Webpack` 配置，全局只有一个 `Compiler` 实例。

![TB1GVGFNXXXXXaTapXXXXXXXXXX-4436-4244](http://114.55.30.96/TB1GVGFNXXXXXaTapXXXXXXXXXX-4436-4244.jpg)



## 开始

npm初始化项目，根目录新建文件

webpack.config.js

```javascript
const path = require('path');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
    mode: 'development',//打包模式 development和production，是否压缩打包出来的文件
    entry: "./index.js",//从哪个文件开始打包
    plugins: [
    new CleanWebpackPlugin({ cleanAfterEveryBuildPatterns: ['dist'], verbose: true })],
    output: {//输出
        filename: 'bundle.js',//最终文件命名
        path: path.resolve(__dirname, 'dist'),//输出文件夹的路径
    }
}
```

index.js

```javascript
import a from './index1.js';
console.log(a);
```

index1.js

```javascript
import b from './index2.js';
console.log(b);
const a = 1;
export default a;
```

index2.js

```javascript
const b = 2;
export default b;
```



修改package.json文件

```json
...
 "scripts": {
    "build": " node ./node_modules/webpack/bin/webpack.js --inline --progress --config ./webpack.config.js"
  },
...
```

执行sctipts命令

```shell
$ npm run build
```

看下构建完的代码

```javascript
/******/ (function(modules) { // webpackBootstrap
/******/ 	// The module cache
/******/ 	var installedModules = {};
/******/
/******/ 	// The require function
/******/ 	function __webpack_require__(moduleId) {
/******/
/******/ 		// Check if module is in cache
/******/ 		if(installedModules[moduleId]) {
/******/ 			return installedModules[moduleId].exports;
/******/ 		}
/******/ 		// Create a new module (and put it into the cache)
/******/ 		var module = installedModules[moduleId] = {
/******/ 			i: moduleId,
/******/ 			l: false,
/******/ 			exports: {}
/******/ 		};
/******/
/******/ 		// Execute the module function
/******/ 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
/******/
/******/ 		// Flag the module as loaded
/******/ 		module.l = true;
/******/
/******/ 		// Return the exports of the module
/******/ 		return module.exports;
/******/ 	}
/******/
/******/
/******/ 	// expose the modules object (__webpack_modules__)
/******/ 	__webpack_require__.m = modules;
/******/
/******/ 	// expose the module cache
/******/ 	__webpack_require__.c = installedModules;
/******/
/******/ 	// define getter function for harmony exports
/******/ 	__webpack_require__.d = function(exports, name, getter) {
/******/ 		if(!__webpack_require__.o(exports, name)) {
/******/ 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
/******/ 		}
/******/ 	};
/******/
/******/ 	// define __esModule on exports
/******/ 	__webpack_require__.r = function(exports) {
/******/ 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
/******/ 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
/******/ 		}
/******/ 		Object.defineProperty(exports, '__esModule', { value: true });
/******/ 	};
/******/
/******/ 	// create a fake namespace object
/******/ 	// mode & 1: value is a module id, require it
/******/ 	// mode & 2: merge all properties of value into the ns
/******/ 	// mode & 4: return value when already ns object
/******/ 	// mode & 8|1: behave like require
/******/ 	__webpack_require__.t = function(value, mode) {
/******/ 		if(mode & 1) value = __webpack_require__(value);
/******/ 		if(mode & 8) return value;
/******/ 		if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
/******/ 		var ns = Object.create(null);
/******/ 		__webpack_require__.r(ns);
/******/ 		Object.defineProperty(ns, 'default', { enumerable: true, value: value });
/******/ 		if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
/******/ 		return ns;
/******/ 	};
/******/
/******/ 	// getDefaultExport function for compatibility with non-harmony modules
/******/ 	__webpack_require__.n = function(module) {
/******/ 		var getter = module && module.__esModule ?
/******/ 			function getDefault() { return module['default']; } :
/******/ 			function getModuleExports() { return module; };
/******/ 		__webpack_require__.d(getter, 'a', getter);
/******/ 		return getter;
/******/ 	};
/******/
/******/ 	// Object.prototype.hasOwnProperty.call
/******/ 	__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
/******/
/******/ 	// __webpack_public_path__
/******/ 	__webpack_require__.p = "";
/******/
/******/
/******/ 	// Load entry module and return exports
/******/ 	return __webpack_require__(__webpack_require__.s = "./index.js");
/******/ })
/************************************************************************/
/******/ ({

/***/ "./index.js":
/*!******************!*\
  !*** ./index.js ***!
  \******************/
/*! no exports provided */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _index1_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./index1.js */ \"./index1.js\");\n\nconsole.log(_index1_js__WEBPACK_IMPORTED_MODULE_0__[\"default\"]);\n\n//# sourceURL=webpack:///./index.js?");

/***/ }),

/***/ "./index1.js":
/*!*******************!*\
  !*** ./index1.js ***!
  \*******************/
/*! exports provided: default */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony import */ var _index2_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./index2.js */ \"./index2.js\");\n\nconsole.log(_index2_js__WEBPACK_IMPORTED_MODULE_0__[\"default\"]);\nconst a = 1;\n/* harmony default export */ __webpack_exports__[\"default\"] = (a);\n\n//# sourceURL=webpack:///./index1.js?");

/***/ }),

/***/ "./index2.js":
/*!*******************!*\
  !*** ./index2.js ***!
  \*******************/
/*! exports provided: default */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\nconst b = 2;\n/* harmony default export */ __webpack_exports__[\"default\"] = (b);\n\n//# sourceURL=webpack:///./index2.js?");

/***/ })

/******/ });
```

webpack_require实现了自己的模块化，并且把代码都缓存在了installedModules里，代码文件以对象形式传入，key是路径，value是包裹的代码字符串。



**如何断点调试打包过程**

修改package.json文件

```json
...
 "scripts": {
    "build": " node --inspect-brk ./node_modules/webpack/bin/webpack.js --inline --progress --config ./webpack.config.js"
  },
...
```

因为可以知道构建的时候调用的是webpack.js文件，所以添加`--inspect-brk`参数，开启调试。

上面这个例子，调用webpack.js文件的时候，最后是调用到这

![微信截图_20191014215251](http://114.55.30.96/微信截图_20191014215251.png)

也就是引入这个模块

![微信截图_20191014215401](http://114.55.30.96/微信截图_20191014215401.png)

所以进入这个文件打个`debugger`

![微信截图_20191014215553](http://114.55.30.96/微信截图_20191014215553.png)



**初始化参数**

![WX20191108-102206@2x](http://114.55.30.96/WX20191108-102206@2x.png)

首先调用`yargs.parse`方法，传入命令行参数与回调函数。

```javascript
  self.parse = function parse (args, shortCircuit, _parseFn) {
    argsert('[string|array] [function|boolean|object] [function]', [args, shortCircuit, _parseFn], arguments.length)
    if (typeof args === 'undefined') {
      return self._parseArgs(processArgs)
    }

    if (typeof shortCircuit === 'object') {
      parseContext = shortCircuit
      shortCircuit = _parseFn
    }

    if (typeof shortCircuit === 'function') {
      parseFn = shortCircuit
      shortCircuit = null
    }
    if (!shortCircuit) processArgs = args

    freeze()
    if (parseFn) exitProcess = false

    const parsed = self._parseArgs(args, shortCircuit)
    if (parseFn) parseFn(exitError, parsed, output)
    unfreeze()

    return parsed
  }
```

在这里做参数合并，之后调用传入的回调函数

![微信截图_20191015083039](http://114.55.30.96/微信截图_20191015083039.png)



走进`options = require("./utils/convert-argv")(argv);`，在这里面获取配置文件位置，并且把配置项`push`到`options`数组里

![微信截图_20191015083645](http://114.55.30.96/微信截图_20191015083645.png)

`requireConfig`方法，就是引入配置文件模块

```javascript
		const requireConfig = function requireConfig(configPath) {
			let options = (function WEBPACK_OPTIONS() {
				if (argv.configRegister && argv.configRegister.length) {
					module.paths.unshift(path.resolve(process.cwd(), "node_modules"), process.cwd());
					argv.configRegister.forEach(dep => {
						require(dep);
					});
					return require(path.resolve(process.cwd(), configPath));
				} else {
					return require(path.resolve(process.cwd(), configPath));
				}
			})();
			options = prepareOptions(options, argv);
			return options;
		};
```

最后得到的配置信息

![微信截图_20191015084054](http://114.55.30.96/微信截图_20191015084054.png)

之后把配置信息传入`processOptions`函数

```javascript
		function processOptions(options) {
			...
			...
            
			const webpack = require("webpack");

			let lastHash = null;
			let compiler;
			try {
				compiler = webpack(options);
			} catch (err) {
				if (err.name === "WebpackOptionsValidationError") {
					if (argv.color) console.error(`\u001b[1m\u001b[31m${err.message}\u001b[39m\u001b[22m`);
					else console.error(err.message);
					// eslint-disable-next-line no-process-exit
					process.exit(1);
				}

				throw err;
			}

			if (argv.progress) {
				const ProgressPlugin = require("webpack").ProgressPlugin;
				new ProgressPlugin({
					profile: argv.profile
				}).apply(compiler);
			}
			if (outputOptions.infoVerbosity === "verbose") {
				if (argv.w) {
					compiler.hooks.watchRun.tap("WebpackInfo", compilation => {
						const compilationName = compilation.name ? compilation.name : "";
						console.error("\nCompilation " + compilationName + " starting…\n");
					});
				} else {
					compiler.hooks.beforeRun.tap("WebpackInfo", compilation => {
						const compilationName = compilation.name ? compilation.name : "";
						console.error("\nCompilation " + compilationName + " starting…\n");
					});
				}
				compiler.hooks.done.tap("WebpackInfo", compilation => {
					const compilationName = compilation.name ? compilation.name : "";
					console.error("\nCompilation " + compilationName + " finished\n");
				});
			}

			function compilerCallback(err, stats) {
				if (!options.watch || err) {
					// Do not keep cache anymore
					compiler.purgeInputFileSystem();
				}
				if (err) {
					lastHash = null;
					console.error(err.stack || err);
					if (err.details) console.error(err.details);
					process.exitCode = 1;
					return;
				}
				if (outputOptions.json) {
					stdout.write(JSON.stringify(stats.toJson(outputOptions), null, 2) + "\n");
				} else if (stats.hash !== lastHash) {
					lastHash = stats.hash;
					if (stats.compilation && stats.compilation.errors.length !== 0) {
						const errors = stats.compilation.errors;
						if (errors[0].name === "EntryModuleNotFoundError") {
							console.error("\n\u001b[1m\u001b[31mInsufficient number of arguments or no entry found.");
							console.error(
								"\u001b[1m\u001b[31mAlternatively, run 'webpack(-cli) --help' for usage info.\u001b[39m\u001b[22m\n"
							);
						}
					}
					const statsString = stats.toString(outputOptions);
					const delimiter = outputOptions.buildDelimiter ? `${outputOptions.buildDelimiter}\n` : "";
					if (statsString) stdout.write(`${statsString}\n${delimiter}`);
				}
				if (!options.watch && stats.hasErrors()) {
					process.exitCode = 2;
				}
			}
			if (firstOptions.watch || options.watch) {
				const watchOptions =
					firstOptions.watchOptions || options.watchOptions || firstOptions.watch || options.watch || {};
				if (watchOptions.stdin) {
					process.stdin.on("end", function(_) {
						process.exit(); // eslint-disable-line
					});
					process.stdin.resume();
				}
				compiler.watch(watchOptions, compilerCallback);
				if (outputOptions.infoVerbosity !== "none") console.error("\nwebpack is watching the files…\n");
			} else {
				compiler.run((err, stats) => {
					if (compiler.close) {
						compiler.close(err2 => {
							compilerCallback(err || err2, stats);
						});
					} else {
						compilerCallback(err, stats);
					}
				});
			}
		}
```

先引入`webpack` ，文件目录在`node_modules/webpack/lib/webpack.js`

![微信截图_20191015090614](http://114.55.30.96/微信截图_20191015090614.png)



 **开始编译**

 初始化 `Compiler` 对象 `compiler = webpack(options);`

```javascript
const webpack = (options, callback) => {
	const webpackOptionsValidationErrors = validateSchema(
		webpackOptionsSchema,
		options
	);
	if (webpackOptionsValidationErrors.length) {
		throw new WebpackOptionsValidationError(webpackOptionsValidationErrors);
	}
	let compiler;
	if (Array.isArray(options)) {
		compiler = new MultiCompiler(
			Array.from(options).map(options => webpack(options))
		);
	} else if (typeof options === "object") {
		options = new WebpackOptionsDefaulter().process(options);

		compiler = new Compiler(options.context);
		compiler.options = options;
		new NodeEnvironmentPlugin({
			infrastructureLogging: options.infrastructureLogging
		}).apply(compiler);
		if (options.plugins && Array.isArray(options.plugins)) {
			for (const plugin of options.plugins) {
				if (typeof plugin === "function") {
					plugin.call(compiler, compiler);
				} else {
					plugin.apply(compiler);
				}
			}
		}
		compiler.hooks.environment.call();
		compiler.hooks.afterEnvironment.call();
		compiler.options = new WebpackOptionsApply().process(options, compiler);
	} else {
		throw new Error("Invalid argument: options");
	}
	if (callback) {
		if (typeof callback !== "function") {
			throw new Error("Invalid argument: callback");
		}
		if (
			options.watch === true ||
			(Array.isArray(options) && options.some(o => o.watch))
		) {
			const watchOptions = Array.isArray(options)
				? options.map(o => o.watchOptions || {})
				: options.watchOptions || {};
			return compiler.watch(watchOptions, callback);
		}
		compiler.run(callback);
	}
	return compiler;
};
```

compiler是Compiler的实例

```javascript
compiler = new Compiler(options.context);
```

在创建过程中，hooks属性里创建了很多钩子实例

![WX20191108-105919@2x](http://114.55.30.96/WX20191108-105919@2x.png)

实例化compiler之后，遍历了`plugins`数组

![微信截图_20191018102343](http://114.55.30.96/微信截图_20191018102343.png)

判断是否是`function`类型，不是就调用`apply`方法（我们这边只使用了`CleanWebpackPlugin`插件，所以调用的是它实例的`apply`方法）

```javascript
    apply(compiler: Compiler) {
        if (!compiler.options.output || !compiler.options.output.path) {
            console.warn(
                'clean-webpack-plugin: options.output.path not defined. Plugin disabled...',
            );

            return;
        }

        this.outputPath = compiler.options.output.path;

        const hooks = compiler.hooks;

        if (this.cleanOnceBeforeBuildPatterns.length !== 0) {
            if (hooks) {
                hooks.emit.tap('clean-webpack-plugin', (compilation) => {
                    this.handleInitial(compilation);
                });
            } else {
                compiler.plugin('emit', (compilation, callback) => {
                    try {
                        this.handleInitial(compilation);

                        callback();
                    } catch (error) {
                        callback(error);
                    }
                });
            }
        }

        if (hooks) {
            hooks.done.tap('clean-webpack-plugin', (stats) => {
                this.handleDone(stats);
            });
        } else {
            compiler.plugin('done', (stats) => {
                this.handleDone(stats);
            });
        }
    }
```

- 先判断`compiler.options.output.path`，输出路径是否存在，不存在就报提醒
- 之后判断`cleanOnceBeforeBuildPatterns`的长度，它是在实例化的时候创建的

![微信截图_20191018104524](http://114.55.30.96/微信截图_20191018104524.png)

，再判断是否有`hooks`，之后调用下面这个方法

```javascript
hooks.emit.tap('clean-webpack-plugin', (compilation) => {
 	this.handleInitial(compilation);
});
```

把名字、类型、回调存到数组里，挂载在`hook.emit`上

![微信截图_20191018105214](http://114.55.30.96/微信截图_20191018105214.png)

- 之后又在`hooks.done`上的`taps`属性存入，这里的区别是回调函数不一样

![微信截图_20191018105417](http://114.55.30.96/微信截图_20191018105417.png)



`compiler`最后的样子

 ![微信截图_20191015153328](http://114.55.30.96/微信截图_20191015153328.png)

接着执行run方法开始执行编译

![微信截图_20191015154017](http://114.55.30.96/微信截图_20191015154017.png)

```javascript
	run(callback) {
		if (this.running) return callback(new ConcurrentCompilationError());

		const finalCallback = (err, stats) => {
			this.running = false;

			if (err) {
				this.hooks.failed.call(err);
			}

			if (callback !== undefined) return callback(err, stats);
		};

		const startTime = Date.now();

		this.running = true;

		const onCompiled = (err, compilation) => {
			if (err) return finalCallback(err);

			if (this.hooks.shouldEmit.call(compilation) === false) {
				const stats = new Stats(compilation);
				stats.startTime = startTime;
				stats.endTime = Date.now();
				this.hooks.done.callAsync(stats, err => {
					if (err) return finalCallback(err);
					return finalCallback(null, stats);
				});
				return;
			}

			this.emitAssets(compilation, err => {
				if (err) return finalCallback(err);

				if (compilation.hooks.needAdditionalPass.call()) {
					compilation.needAdditionalPass = true;

					const stats = new Stats(compilation);
					stats.startTime = startTime;
					stats.endTime = Date.now();
					this.hooks.done.callAsync(stats, err => {
						if (err) return finalCallback(err);

						this.hooks.additionalPass.callAsync(err => {
							if (err) return finalCallback(err);
							this.compile(onCompiled);
						});
					});
					return;
				}

				this.emitRecords(err => {
					if (err) return finalCallback(err);

					const stats = new Stats(compilation);
					stats.startTime = startTime;
					stats.endTime = Date.now();
					this.hooks.done.callAsync(stats, err => {
						if (err) return finalCallback(err);
						return finalCallback(null, stats);
					});
				});
			});
		};

		this.hooks.beforeRun.callAsync(this, err => {
			if (err) return finalCallback(err);

			this.hooks.run.callAsync(this, err => {
				if (err) return finalCallback(err);

				this.readRecords(err => {
					if (err) return finalCallback(err);

					this.compile(onCompiled);
				});
			});
		});
	}
```

进入`this.compile(onCompiled);`

```javascript
	compile(callback) {
		const params = this.newCompilationParams();
		this.hooks.beforeCompile.callAsync(params, err => {
			if (err) return callback(err);

			this.hooks.compile.call(params);

			const compilation = this.newCompilation(params);

			this.hooks.make.callAsync(compilation, err => {
				if (err) return callback(err);

				compilation.finish(err => {
					if (err) return callback(err);

					compilation.seal(err => {
						if (err) return callback(err);

						this.hooks.afterCompile.callAsync(compilation, err => {
							if (err) return callback(err);

							return callback(null, compilation);
						});
					});
				});
			});
		});
	}
```

在这里构建compilation对象

```javascript
const compilation = this.newCompilation(params);
```

进入newCompilation

```javascript
	newCompilation(params) {
		const compilation = this.createCompilation();
		compilation.fileTimestamps = this.fileTimestamps;
		compilation.contextTimestamps = this.contextTimestamps;
		compilation.name = this.name;
		compilation.records = this.records;
		compilation.compilationDependencies = params.compilationDependencies;
		this.hooks.thisCompilation.call(compilation, params);
		this.hooks.compilation.call(compilation, params);
		return compilation;
	}
```

进入createCompilation，实例化Compilation

```javascript
	createCompilation() {
		return new Compilation(this);
	}
```



**确认入口**

之后进入`this.hooks.make.callAsync`，目的是确认入口

![WX20191019-163429@2x](http://114.55.30.96/WX20191019-163429@2x.png)

进入`this._addModuleChain`

![WX20191020-000729@2x](http://114.55.30.96/WX20191020-000729@2x.png)

创建NormalModule实例

```javascript
(err, result) => {
  if (err) return callback(err);

  // Ignored
  if (!result) return callback();

  let createdModule = this.hooks.createModule.call(result);
  if (!createdModule) {
    if (!result.request) {
      return callback(new Error("Empty dependency (no request)"));
    }

    createdModule = new NormalModule(result);
  }

  createdModule = this.hooks.module.call(createdModule, result);

  return callback(null, createdModule);
}
```

之后对module进行build

![WX20191020-003218@2x](http://114.55.30.96/WX20191020-003218@2x.png)

进入`doBuild`方法

```javascript
this.doBuild(options, compilation, resolver, fs, err => {
			this._cachedSources.clear();

			if (err) {
				this.markModuleAsErrored(err);
				this._initBuildHash(compilation);
				return callback();
			}

			const noParseRule = options.module && options.module.noParse;
			if (this.shouldPreventParsing(noParseRule, this.request)) {
				this._initBuildHash(compilation);
				return callback();
			}

			const handleParseError = e => {
				const source = this._source.source();
				const loaders = this.loaders.map(item =>
					contextify(options.context, item.loader)
				);
				const error = new ModuleParseError(this, source, e, loaders);
				this.markModuleAsErrored(error);
				this._initBuildHash(compilation);
				return callback();
			};

			const handleParseResult = result => {
				this._lastSuccessfulBuildMeta = this.buildMeta;
				this._initBuildHash(compilation);
				return callback();
			};

			try {
				const result = this.parser.parse(
					this._ast || this._source.source(),
					{
						current: this,
						module: this,
						compilation: compilation,
						options: options
					},
					(err, result) => {
						if (err) {
							handleParseError(err);
						} else {
							handleParseResult(result);
						}
					}
				);
				if (result !== undefined) {
					// parse is sync
					handleParseResult(result);
				}
			} catch (e) {
				handleParseError(e);
			}
		});
```

loader先对文件处理转成buffer再转成字符串，挂载在_source上

![WX20191109-134648@2x](http://114.55.30.96/WX20191109-134648@2x.png)

```javascript
const asString = buf => {
	if (Buffer.isBuffer(buf)) {
		return buf.toString("utf-8");
	}
	return buf;
};
```

之后在这里解析入口文件转成ast，解析依赖关系。

![WX20191020-140848@2x](http://114.55.30.96/WX20191020-140848@2x.png)

之后进入handleParseResult

![WX20191020-004151@2x](http://114.55.30.96/WX20191020-004151@2x.png)

解析出的依赖会继续走到`parser.parse`的地方

![WX20191020-005357@2x](http://114.55.30.96/WX20191020-005357@2x.png)

![WX20191020-005557@2x](http://114.55.30.96/WX20191020-005557@2x.png)



**输出资源**

webpack 会监听 `seal` 事件调用各插件对构建后的结果进行封装，要逐次对每个 module 和 chunk 进行整理，生成编译后的源码，合并，拆分，生成 hash

进入compilation.seal

![WX20191020-104909@2x](http://114.55.30.96/WX20191020-104909@2x.png)

进入createChunkAssets方法，遍历chunks

![WX20191020-131310@2x](http://114.55.30.96/WX20191020-131310@2x.png)

先获得manifest，遍历它，之后进入fileManifest.render()

![WX20191020-132202@2x](http://114.55.30.96/WX20191020-132202@2x.png)

```javascript
	render(hash, chunk, moduleTemplate, dependencyTemplates) {
		const buf = this.renderBootstrap(
			hash,
			chunk,
			moduleTemplate,
			dependencyTemplates
		);
		let source = this.hooks.render.call(
			new OriginalSource(
				Template.prefix(buf, " \t") + "\n",
				"webpack/bootstrap"
			),
			chunk,
			hash,
			moduleTemplate,
			dependencyTemplates
		);
		if (chunk.hasEntryModule()) {
			source = this.hooks.renderWithEntry.call(source, chunk, hash);
		}
		if (!source) {
			throw new Error(
				"Compiler error: MainTemplate plugin 'render' should return something"
			);
		}
		chunk.rendered = true;
		return new ConcatSource(source, ";");
	}
```

生成source的方法在这里，所有的requirefn被替换成了\_\_webpack_require\_\_

```javascript
	renderBootstrap(hash, chunk, moduleTemplate, dependencyTemplates) {
		const buf = [];
		buf.push(
			this.hooks.bootstrap.call(
				"",
				chunk,
				hash,
				moduleTemplate,
				dependencyTemplates
			)
		);
		buf.push(this.hooks.localVars.call("", chunk, hash));
		buf.push("");
		buf.push("// The require function");
		buf.push(`function ${this.requireFn}(moduleId) {`);
		buf.push(Template.indent(this.hooks.require.call("", chunk, hash)));
		buf.push("}");
		buf.push("");
		buf.push(
			Template.asString(this.hooks.requireExtensions.call("", chunk, hash))
		);
		buf.push("");
		buf.push(Template.asString(this.hooks.beforeStartup.call("", chunk, hash)));
		const afterStartupCode = Template.asString(
			this.hooks.afterStartup.call("", chunk, hash)
		);
		if (afterStartupCode) {
			// TODO webpack 5: this is a bit hacky to avoid a breaking change
			// change it to a better way
			buf.push("var startupResult = (function() {");
		}
		buf.push(Template.asString(this.hooks.startup.call("", chunk, hash)));
		if (afterStartupCode) {
			buf.push("})();");
			buf.push(afterStartupCode);
			buf.push("return startupResult;");
		}
		return buf;
	}
```

最终得到source

![WX20191020-132551@2x](http://114.55.30.96/WX20191020-132551@2x.png)

生成好的js保存在compilation.assets里

之后走进onCompiled方法，又走进emitAssets方法，调用进入`this.hooks.emit.callAsync`中的回调方法

![WX20191021-142223@2x](http://114.55.30.96/WX20191021-142223@2x.png)

最后走到writeOut方法，先得到content

![WX20191021-142755@2x](http://114.55.30.96/WX20191021-142755@2x.png)

进入source.source方法，其实就是循环然后拼接

```javascript
	source() {
		let source = "";
		const children = this.children;
		for(let i = 0; i < children.length; i++) {
			const child = children[i];
			source += typeof child === "string" ? child : child.source();
		}
		return source;
	}
```

最后将文件写入到目标目录

![WX20191020-133753@2x](http://114.55.30.96/WX20191020-133753@2x.png)

![WX20191020-133820@2x](http://114.55.30.96/WX20191020-133820@2x.png)



**补充**

其中的广播事件，调用clean-webpack-plugin插件的时候

![WX20191019-134633@2x](http://114.55.30.96/WX20191019-134633@2x.png)

```javascript
	emitAssets(compilation, callback) {
		let outputPath;
		const emitFiles = err => {
			if (err) return callback(err);

			asyncLib.forEachLimit(
				compilation.getAssets(),
				15,
				({ name: file, source }, callback) => {
					let targetFile = file;
					const queryStringIdx = targetFile.indexOf("?");
					if (queryStringIdx >= 0) {
						targetFile = targetFile.substr(0, queryStringIdx);
					}

					const writeOut = err => {
						if (err) return callback(err);
						const targetPath = this.outputFileSystem.join(
							outputPath,
							targetFile
						);
						// TODO webpack 5 remove futureEmitAssets option and make it on by default
						if (this.options.output.futureEmitAssets) {
							// check if the target file has already been written by this Compiler
							const targetFileGeneration = this._assetEmittingWrittenFiles.get(
								targetPath
							);

							// create an cache entry for this Source if not already existing
							let cacheEntry = this._assetEmittingSourceCache.get(source);
							if (cacheEntry === undefined) {
								cacheEntry = {
									sizeOnlySource: undefined,
									writtenTo: new Map()
								};
								this._assetEmittingSourceCache.set(source, cacheEntry);
							}

							// if the target file has already been written
							if (targetFileGeneration !== undefined) {
								// check if the Source has been written to this target file
								const writtenGeneration = cacheEntry.writtenTo.get(targetPath);
								if (writtenGeneration === targetFileGeneration) {
									// if yes, we skip writing the file
									// as it's already there
									// (we assume one doesn't remove files while the Compiler is running)

									compilation.updateAsset(file, cacheEntry.sizeOnlySource, {
										size: cacheEntry.sizeOnlySource.size()
									});

									return callback();
								}
							}

							// TODO webpack 5: if info.immutable check if file already exists in output
							// skip emitting if it's already there

							// get the binary (Buffer) content from the Source
							/** @type {Buffer} */
							let content;
							if (typeof source.buffer === "function") {
								content = source.buffer();
							} else {
								const bufferOrString = source.source();
								if (Buffer.isBuffer(bufferOrString)) {
									content = bufferOrString;
								} else {
									content = Buffer.from(bufferOrString, "utf8");
								}
							}

							// Create a replacement resource which only allows to ask for size
							// This allows to GC all memory allocated by the Source
							// (expect when the Source is stored in any other cache)
							cacheEntry.sizeOnlySource = new SizeOnlySource(content.length);
							compilation.updateAsset(file, cacheEntry.sizeOnlySource, {
								size: content.length
							});

							// Write the file to output file system
							this.outputFileSystem.writeFile(targetPath, content, err => {
								if (err) return callback(err);

								// information marker that the asset has been emitted
								compilation.emittedAssets.add(file);

								// cache the information that the Source has been written to that location
								const newGeneration =
									targetFileGeneration === undefined
										? 1
										: targetFileGeneration + 1;
								cacheEntry.writtenTo.set(targetPath, newGeneration);
								this._assetEmittingWrittenFiles.set(targetPath, newGeneration);
								this.hooks.assetEmitted.callAsync(file, content, callback);
							});
						} else {
							if (source.existsAt === targetPath) {
								source.emitted = false;
								return callback();
							}
							let content = source.source();

							if (!Buffer.isBuffer(content)) {
								content = Buffer.from(content, "utf8");
							}

							source.existsAt = targetPath;
							source.emitted = true;
							this.outputFileSystem.writeFile(targetPath, content, err => {
								if (err) return callback(err);
								this.hooks.assetEmitted.callAsync(file, content, callback);
							});
						}
					};

					if (targetFile.match(/\/|\\/)) {
						const dir = path.dirname(targetFile);
						this.outputFileSystem.mkdirp(
							this.outputFileSystem.join(outputPath, dir),
							writeOut
						);
					} else {
						writeOut();
					}
				},
				err => {
					if (err) return callback(err);

					this.hooks.afterEmit.callAsync(compilation, err => {
						if (err) return callback(err);

						return callback();
					});
				}
			);
		};

		this.hooks.emit.callAsync(compilation, err => {
			if (err) return callback(err);
			outputPath = compilation.getPath(this.outputPath);
			this.outputFileSystem.mkdirp(outputPath, emitFiles);
		});
	}
```

进入`this.hooks.emit.callAsync`

![WX20191019-134936@2x](http://114.55.30.96/WX20191019-134936@2x.png)

```javascript
	tap: (context, tap) => {
					if (context) {
						context.reportProgress = (p, ...args) => {
							handler(0.95, "emitting", tap.name, ...args);
						};
					}
					handler(0.95, "emitting", tap.name);
}
```

进入`handler(0.95, "emitting", tap.name);`

```javascript
	const defaultHandler = (percentage, msg, ...args) => {
		logger.status(`${Math.floor(percentage * 100)}%`, msg, ...args);
		if (profile) {
			let state = msg;
			state = state.replace(/^\d+\/\d+\s+/, "");
			if (percentage === 0) {
				lastState = null;
				lastStateTime = Date.now();
			} else if (state !== lastState || percentage === 1) {
				const now = Date.now();
				if (lastState) {
					const diff = now - lastStateTime;
					const stateMsg = `${diff}ms ${lastState}`;
					if (diff > 1000) {
						logger.warn(stateMsg);
					} else if (diff > 10) {
						logger.info(stateMsg);
					} else if (diff > 0) {
						logger.log(stateMsg);
					} else {
						logger.debug(stateMsg);
					}
				}
				lastState = state;
				lastStateTime = now;
			}
		}
		if (percentage === 1) logger.status();
	};
```

输出进度，和插件名

![WX20191019-140006@2x](http://114.55.30.96/WX20191019-140006@2x.png)

之后调用回调

![WX20191019-140055@2x](http://114.55.30.96/WX20191019-140055@2x.png)

进入`_fn0(compilation)`

![WX20191019-140153@2x](http://114.55.30.96/WX20191019-140153@2x.png)

进入`handleInitial`方法

```javascript
    handleInitial(compilation: Compilation) {
        if (this.initialClean) {
            return;
        }

        /**
         * Do not remove files if there are compilation errors
         *
         * Handle logging inside this.handleDone
         */
        const stats = compilation.getStats();
        if (stats.hasErrors()) {
            return;
        }

        this.initialClean = true;

        this.removeFiles(this.cleanOnceBeforeBuildPatterns);
    }
```

进入`removeFiles`方法

```javascript
    removeFiles(patterns: string[]) {
        try {
            const deleted = delSync(patterns, {
                force: this.dangerouslyAllowCleanPatternsOutsideProject,
                // Change context to build directory
                cwd: this.outputPath,
                dryRun: this.dry,
                dot: true,
                ignore: this.protectWebpackAssets ? this.currentAssets : [],
            });

            /**
             * Log if verbose is enabled
             */
            if (this.verbose) {
                deleted.forEach((file) => {
                    const filename = path.relative(process.cwd(), file);

                    const message = this.dry ? 'dry' : 'removed';

                    /**
                     * Use console.warn over .log
                     * https://github.com/webpack/webpack/issues/1904
                     * https://github.com/johnagan/clean-webpack-plugin/issues/11
                     */
                    // eslint-disable-next-line no-console
                    console.warn(
                        `clean-webpack-plugin: ${message} ${filename}`,
                    );
                });
            }
        } catch (error) {
            const needsForce = /Cannot delete files\/folders outside the current working directory\./.test(
                error.message,
            );

            if (needsForce) {
                const message =
                    'clean-webpack-plugin: Cannot delete files/folders outside the current working directory. Can be overridden with the `dangerouslyAllowCleanPatternsOutsideProject` option.';

                throw new Error(message);
            }

            throw error;
        }
    }
```

进入`delSync`方法，在这里会对文件做删除处理

```javascript
module.exports.sync = (patterns, options) => {
	options = Object.assign({}, options);

	const {force, dryRun} = options;
	delete options.force;
	delete options.dryRun;

	return globby.sync(patterns, options).map(file => {
		if (!force) {
			safeCheck(file);
		}

		file = path.resolve(options.cwd || '', file);

		if (!dryRun) {
			rimraf.sync(file, {glob: false});
		}

		return file;
	});
};
```

之后打印出被删除的文件名



**hash值生成**

使用crypto.js

进入`this.createHash();`方法调用

![WX20191020-214616@2x](http://114.55.30.96/WX20191020-214616@2x.png)

hash值是fullHash截取前20位

生成逻辑在createHash方法里

```javascript
	createHash() {
		const outputOptions = this.outputOptions;
		const hashFunction = outputOptions.hashFunction;
		const hashDigest = outputOptions.hashDigest;
		const hashDigestLength = outputOptions.hashDigestLength;
		const hash = createHash(hashFunction);
		if (outputOptions.hashSalt) {
			hash.update(outputOptions.hashSalt);
		}
		this.mainTemplate.updateHash(hash);
		this.chunkTemplate.updateHash(hash);
		for (const key of Object.keys(this.moduleTemplates).sort()) {
			this.moduleTemplates[key].updateHash(hash);
		}
		for (const child of this.children) {
			hash.update(child.hash);
		}
		for (const warning of this.warnings) {
			hash.update(`${warning.message}`);
		}
		for (const error of this.errors) {
			hash.update(`${error.message}`);
		}
		const modules = this.modules;
		for (let i = 0; i < modules.length; i++) {
			const module = modules[i];
			const moduleHash = createHash(hashFunction);
			module.updateHash(moduleHash);
			module.hash = /** @type {string} */ (moduleHash.digest(hashDigest));
			module.renderedHash = module.hash.substr(0, hashDigestLength);
		}
		// clone needed as sort below is inplace mutation
		const chunks = this.chunks.slice();
		/**
		 * sort here will bring all "falsy" values to the beginning
		 * this is needed as the "hasRuntime()" chunks are dependent on the
		 * hashes of the non-runtime chunks.
		 */
		chunks.sort((a, b) => {
			const aEntry = a.hasRuntime();
			const bEntry = b.hasRuntime();
			if (aEntry && !bEntry) return 1;
			if (!aEntry && bEntry) return -1;
			return byId(a, b);
		});
		for (let i = 0; i < chunks.length; i++) {
			const chunk = chunks[i];
			const chunkHash = createHash(hashFunction);
			try {
				if (outputOptions.hashSalt) {
					chunkHash.update(outputOptions.hashSalt);
				}
				chunk.updateHash(chunkHash);
				const template = chunk.hasRuntime()
					? this.mainTemplate
					: this.chunkTemplate;
				template.updateHashForChunk(
					chunkHash,
					chunk,
					this.moduleTemplates.javascript,
					this.dependencyTemplates
				);
				this.hooks.chunkHash.call(chunk, chunkHash);
				chunk.hash = /** @type {string} */ (chunkHash.digest(hashDigest));
				hash.update(chunk.hash);
				chunk.renderedHash = chunk.hash.substr(0, hashDigestLength);
				this.hooks.contentHash.call(chunk);
			} catch (err) {
				this.errors.push(new ChunkRenderError(chunk, "", err));
			}
		}
		this.fullHash = /** @type {string} */ (hash.digest(hashDigest));
		this.hash = this.fullHash.substr(0, hashDigestLength);
	}
```



**loader**

```shell
$ npm install @babel/core @babel/preset-env babel-loader -D
```

webpack.config.js

```javascript
const path = require('path');
const {
    CleanWebpackPlugin
} = require('clean-webpack-plugin');

module.exports = {
    mode: 'development', //打包模式 development和production，是否压缩打包出来的文件
    entry: path.resolve(__dirname, "./src/index.js"), //从哪个文件开始打包
    module: {
        rules: [{
            test: /\.js$/,
            exclude: /(node_modules|bower_components)/,
            use: {
                loader: 'babel-loader',
                options: {
                    presets: ['@babel/preset-env']
                }
            }
        }]
    },
    plugins: [
        new CleanWebpackPlugin({
            cleanAfterEveryBuildPatterns: ['dist'],
            verbose: true
        })
    ],
    output: { //输出
        filename: 'bundle.js', //最终文件命名
        path: path.resolve(__dirname, 'dist'), //输出文件夹的路径
    }
}
```

目录

```
├── package-lock.json
├── package.json
├── src
│   ├── index.js
│   └── index1.js
└── webpack.config.js
```









**钩子调用顺序**

```
Compiler:beforeRun 清除缓存

Compiler:run 注册缓存数据钩子

Compiler:beforeCompile

Compiler:compile 开始编译

Compiler:make 从入口分析依赖以及间接依赖模块，创建模块对象

Compilation:buildModule 模块构建

Compiler:normalModuleFactory 构建

Compilation:seal 构建结果封装， 不可再更改

Compiler:afterCompile 完成构建，缓存数据

Compiler:emit 输出到dist目录
```



## 参考

 https://segmentfault.com/a/1190000015088834 

https://segmentfault.com/a/1190000015917768

https://juejin.im/entry/57f4ac35a0bb9f0058168491

https://juejin.im/post/5beb8875e51d455e5c4dd83f

https://juejin.im/post/5bc1a73df265da0a8d36b74f