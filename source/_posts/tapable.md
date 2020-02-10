---
title: Tapable
date: 2019-10-21 17:39:30
tags: 
- webpack
categories: 
- webpack
---

## Tapable

`webpack`本质上是一种事件流的机制，它的工作流程就是将各个插件串联起来，而实现这一切的核心就是`Tapable`。`Tapable`暴露出挂载`plugin`的方法，使我们能将`plugin`控制在`webapack`事件流上运行， `Compiler`、`Compilation`等都是继承于`Tabable`类。

它有很多Hook（钩子）类，为插件提供挂载的钩子

```javascript
exports.__esModule = true;
exports.SyncHook = require("./SyncHook");
exports.SyncBailHook = require("./SyncBailHook");
exports.SyncWaterfallHook = require("./SyncWaterfallHook");
exports.SyncLoopHook = require("./SyncLoopHook");
exports.AsyncParallelHook = require("./AsyncParallelHook");
exports.AsyncParallelBailHook = require("./AsyncParallelBailHook");
exports.AsyncSeriesHook = require("./AsyncSeriesHook");
exports.AsyncSeriesBailHook = require("./AsyncSeriesBailHook");
exports.AsyncSeriesLoopHook = require("./AsyncSeriesLoopHook");
exports.AsyncSeriesWaterfallHook = require("./AsyncSeriesWaterfallHook");
exports.HookMap = require("./HookMap");
exports.MultiHook = require("./MultiHook");
```

![WX20191022-090003@2x](http://114.55.30.96/WX20191022-090003@2x.png)

- tapable 暴露出来的都是类方法，new 一个类方法获得我们需要的钩子。类方法会根据传参，接受同样数量的参数。
- 使用 tap/tapAsync/tapPromise 绑定钩子
- call/callAsync 执行绑定事件

**Hook类**

```javascript
/*
	MIT License http://www.opensource.org/licenses/mit-license.php
	Author Tobias Koppers @sokra
*/
"use strict";

const util = require("util");

const deprecateContext = util.deprecate(() => {},
"Hook.context is deprecated and will be removed");

const CALL_DELEGATE = function(...args) {
	this.call = this._createCall("sync");
	return this.call(...args);
};
const CALL_ASYNC_DELEGATE = function(...args) {
	this.callAsync = this._createCall("async");
	return this.callAsync(...args);
};
const PROMISE_DELEGATE = function(...args) {
	this.promise = this._createCall("promise");
	return this.promise(...args);
};

class Hook {
	constructor(args = [], name = undefined) {
		this._args = args;
		this.name = name;
		this.taps = [];
		this.interceptors = [];
		this._call = CALL_DELEGATE;
		this.call = CALL_DELEGATE;
		this._callAsync = CALL_ASYNC_DELEGATE;
		this.callAsync = CALL_ASYNC_DELEGATE;
		this._promise = PROMISE_DELEGATE;
		this.promise = PROMISE_DELEGATE;
		this._x = undefined;

		this.compile = this.compile;
		this.tap = this.tap;
		this.tapAsync = this.tapAsync;
		this.tapPromise = this.tapPromise;
	}

	compile(options) {
		throw new Error("Abstract: should be overridden");
	}

	_createCall(type) {
		return this.compile({
			taps: this.taps,
			interceptors: this.interceptors,
			args: this._args,
			type: type
		});
	}

	_tap(type, options, fn) {
		if (typeof options === "string") {
			options = {
				name: options
			};
		} else if (typeof options !== "object" || options === null) {
			throw new Error("Invalid tap options");
		}
		if (typeof options.name !== "string" || options.name === "") {
			throw new Error("Missing name for tap");
		}
		if (typeof options.context !== "undefined") {
			deprecateContext();
		}
		options = Object.assign({ type, fn }, options);
		options = this._runRegisterInterceptors(options);
		this._insert(options);
	}

	tap(options, fn) {
		this._tap("sync", options, fn);
	}

	tapAsync(options, fn) {
		this._tap("async", options, fn);
	}

	tapPromise(options, fn) {
		this._tap("promise", options, fn);
	}

	_runRegisterInterceptors(options) {
		for (const interceptor of this.interceptors) {
			if (interceptor.register) {
				const newOptions = interceptor.register(options);
				if (newOptions !== undefined) {
					options = newOptions;
				}
			}
		}
		return options;
	}

	withOptions(options) {
		const mergeOptions = opt =>
			Object.assign({}, options, typeof opt === "string" ? { name: opt } : opt);

		return {
			name: this.name,
			tap: (opt, fn) => this.tap(mergeOptions(opt), fn),
			tapAsync: (opt, fn) => this.tapAsync(mergeOptions(opt), fn),
			tapPromise: (opt, fn) => this.tapPromise(mergeOptions(opt), fn),
			intercept: interceptor => this.intercept(interceptor),
			isUsed: () => this.isUsed(),
			withOptions: opt => this.withOptions(mergeOptions(opt))
		};
	}

	isUsed() {
		return this.taps.length > 0 || this.interceptors.length > 0;
	}

	intercept(interceptor) {
		this._resetCompilation();
		this.interceptors.push(Object.assign({}, interceptor));
		if (interceptor.register) {
			for (let i = 0; i < this.taps.length; i++) {
				this.taps[i] = interceptor.register(this.taps[i]);
			}
		}
	}

	_resetCompilation() {
		this.call = this._call;
		this.callAsync = this._callAsync;
		this.promise = this._promise;
	}

	_insert(item) {
		this._resetCompilation();
		let before;
		if (typeof item.before === "string") {
			before = new Set([item.before]);
		} else if (Array.isArray(item.before)) {
			before = new Set(item.before);
		}
		let stage = 0;
		if (typeof item.stage === "number") {
			stage = item.stage;
		}
		let i = this.taps.length;
		while (i > 0) {
			i--;
			const x = this.taps[i];
			this.taps[i + 1] = x;
			const xStage = x.stage || 0;
			if (before) {
				if (before.has(x.name)) {
					before.delete(x.name);
					continue;
				}
				if (before.size > 0) {
					continue;
				}
			}
			if (xStage > stage) {
				continue;
			}
			i++;
			break;
		}
		this.taps[i] = item;
	}
}

Object.setPrototypeOf(Hook.prototype, null);

module.exports = Hook;
```



## 同步钩子

**SyncHook（同步钩子）**

```javascript
const { SyncHook } = require("./lib/index.js");

//实例化
let queue = new SyncHook(['name']); //所有的构造函数都接收一个可选的参数，这个参数是一个字符串的数组。

// 订阅
queue.tap('1', function (name, name2) {// tap 的第一个参数是用来标识订阅的函数的
    console.log(name, name2, 1);
    return '1'
});
queue.tap('2', function (name) {
    console.log(name, 2);
});
queue.tap('3', function (name) {
    console.log(name, 3);
});

// 发布
queue.call('webpack', 'webpack-cli');// 发布的时候触发订阅的函数 同时传入参数
// webpack undefined 1
// webpack 2
// webpack 3
```

- 通过new关键字调用SyncHook，返回Hook的实例，并且重写了 tapAsync、tapPromise、compile 三个方法

```javascript
/*
	MIT License http://www.opensource.org/licenses/mit-license.php
	Author Tobias Koppers @sokra
*/
"use strict";

const Hook = require("./Hook");
const HookCodeFactory = require("./HookCodeFactory");

class SyncHookCodeFactory extends HookCodeFactory {
	content({ onError, onDone, rethrowIfPossible }) {
		return this.callTapsSeries({
			onError: (i, err) => onError(err),
			onDone,
			rethrowIfPossible
		});
	}
}

const factory = new SyncHookCodeFactory();

const TAP_ASYNC = () => {
	throw new Error("tapAsync is not supported on a SyncHook");
};

const TAP_PROMISE = () => {
	throw new Error("tapPromise is not supported on a SyncHook");
};

const COMPILE = function(options) {
	factory.setup(this, options);
	return factory.create(options);
};

function SyncHook(args = [], name = undefined) {
	const hook = new Hook(args, name);
	hook.constructor = SyncHook;
	hook.tapAsync = TAP_ASYNC;
	hook.tapPromise = TAP_PROMISE;
	hook.compile = COMPILE;
	return hook;
}

SyncHook.prototype = null;

module.exports = SyncHook;
```

- queue.tap调用，其实就是把名称、类型与回调设置在taps数组里

![WX20191022-093020@2x](http://114.55.30.96/WX20191022-093020@2x.png)

![WX20191022-093529@2x](http://114.55.30.96/WX20191022-093529@2x.png)

- queue.call调用，这里通过new Function生成了匿名函数，之后匿名函数被调用

![WX20191022-101512@2x](http://114.55.30.96/WX20191022-101512@2x.png)

![WX20191022-101647@2x](http://114.55.30.96/WX20191022-101647@2x.png)

![WX20191022-101744@2x](http://114.55.30.96/WX20191022-101744@2x.png)

```javascript
	create(options) {
		this.init(options);
		let fn;
		switch (this.options.type) {
			case "sync":
				fn = new Function(
					this.args(),
					'"use strict";\n' +
						this.header() +
						this.content({
							onError: err => `throw ${err};\n`,
							onResult: result => `return ${result};\n`,
							resultReturns: true,
							onDone: () => "",
							rethrowIfPossible: true
						})
				);
				break;
			case "async":
				fn = new Function(
					this.args({
						after: "_callback"
					}),
					'"use strict";\n' +
						this.header() +
						this.content({
							onError: err => `_callback(${err});\n`,
							onResult: result => `_callback(null, ${result});\n`,
							onDone: () => "_callback();\n"
						})
				);
				break;
			case "promise":
				let errorHelperUsed = false;
				const content = this.content({
					onError: err => {
						errorHelperUsed = true;
						return `_error(${err});\n`;
					},
					onResult: result => `_resolve(${result});\n`,
					onDone: () => "_resolve();\n"
				});
				let code = "";
				code += '"use strict";\n';
				code += "return new Promise((_resolve, _reject) => {\n";
				if (errorHelperUsed) {
					code += "var _sync = true;\n";
					code += "function _error(_err) {\n";
					code += "if(_sync)\n";
					code += "_resolve(Promise.resolve().then(() => { throw _err; }));\n";
					code += "else\n";
					code += "_reject(_err);\n";
					code += "};\n";
				}
				code += this.header();
				code += content;
				if (errorHelperUsed) {
					code += "_sync = false;\n";
				}
				code += "});\n";
				fn = new Function(this.args(), code);
				break;
		}
		this.deinit();
		return fn;
	}
```

在这里生成fn

```javascript
anonymous(name
) {
  "use strict";
  var _context;
  var _x = this._x;
  var _fn0 = _x[0];
  _fn0(name);
  var _fn1 = _x[1];
  _fn1(name);
  var _fn2 = _x[2];
  _fn2(name);
}
```

之后调用这个函数



**SyncBailHook（同步保险钩子）**

```javascript
const {
	SyncBailHook
} = require("./lib/index.js");
let queue = new SyncBailHook(['name']); 

// 订阅
queue.tap('1', function (name, name2) {
	console.log(name, name2, 1);
	return '1'
});
queue.tap('2', function (name) {
	console.log(name, 2);
});
queue.tap('3', function (name) {
	console.log(name, 3);
});

// 发布
queue.call('webpack','webpack-cli'); 
//webpack undefined 1
```

生成的fn

```javascript
(function anonymous(name) {
  "use strict";
  var _context;
  var _x = this._x;
  var _fn0 = _x[0];
  var _result0 = _fn0(name);
  if (_result0 !== undefined) {
    return _result0;;
  } else {
    var _fn1 = _x[1];
    var _result1 = _fn1(name);
    if (_result1 !== undefined) {
      return _result1;;
    } else {
      var _fn2 = _x[2];
      var _result2 = _fn2(name);
      if (_result2 !== undefined) {
        return _result2;;
      } else {}
    }
  }
})
```

如果执行结果不等于undefined，就停止并返回执行结果，否则继续往下执行。



**SyncLoopHook（同步循环钩子）**

```javascript
const {
	SyncLoopHook
} = require("./lib/index.js");
let queue = new SyncLoopHook(['name']); 

// 订阅
queue.tap('1', function (name, name2) { 
	console.log(name, name2, 1);
	return '1'
});
queue.tap('2', function (name) {
	console.log(name, 2);
	return '2'
});
queue.tap('3', function (name) {
	console.log(name, 3);
});

// 发布
queue.call('webpack','webpack-cli'); 
```

生成的fn

```javascript
(function anonymous(name) {
  "use strict";
  var _context;
  var _x = this._x;
  var _loop;
  do {
    _loop = false;
    var _fn0 = _x[0];
    var _result0 = _fn0(name);
    if (_result0 !== undefined) {
      _loop = true;
    } else {
      var _fn1 = _x[1];
      var _result1 = _fn1(name);
      if (_result1 !== undefined) {
        _loop = true;
      } else {
        var _fn2 = _x[2];
        var _result2 = _fn2(name);
        if (_result2 !== undefined) {
          _loop = true;
        } else {
          if (!_loop) {}
        }
      }
    }
  } while (_loop);
})
```

如果执行结果不等于undefined，就会一直执行



**SyncWaterfallHook（同步瀑布钩子）**

```javascript
const {
	SyncWaterfallHook
} = require("./lib/index.js");
let queue = new SyncWaterfallHook(['name']); 

// 订阅
queue.tap('1', function (name, name2) { 
	console.log(name, name2, 1);
	return '1'
});
queue.tap('2', function (name) {
	console.log(name, 2);
	return '2'
});
queue.tap('3', function (name) {
	console.log(name, 3);
});

// 发布
queue.call('webpack','webpack-cli'); 
// webpack undefined 1
// 1 2
// 2 3
```

生成的fn

```javascript
(function anonymous(name) {
  "use strict";
  var _context;
  var _x = this._x;
  var _fn0 = _x[0];
  var _result0 = _fn0(name);
  if (_result0 !== undefined) {
    name = _result0;
  }
  var _fn1 = _x[1];
  var _result1 = _fn1(name);
  if (_result1 !== undefined) {
    name = _result1;
  }
  var _fn2 = _x[2];
  var _result2 = _fn2(name);
  if (_result2 !== undefined) {
    name = _result2;
  }
  return name;
})
```

前面的 handler 返回值作为下一个 handler 的输入值



## 异步钩子

所有的异步钩子支持 tap、tapAsync、tapPromise 方法来注册各种类型的 handler，但是不支持 call 方法来触发 handler，只支持 promise、callAsync。



**AsyncParallelBailHook（异步并行保险钩子）**

```javascript
const {
	AsyncParallelBailHook
} = require("./lib/index.js");
let queue = new AsyncParallelBailHook();

// 订阅
queue.tapAsync('1', next => {
	setTimeout(() => {
		next(2)
	}, 2000)
});
queue.tapAsync('2', next => {
	setTimeout(() => {
		next(1)
	}, 2000)
});

// 发布
queue.callAsync((result) => {
	console.log(result)
	console.log('callback 执行完成')
});
// 2
// callback 执行完成
```

生成的fn

```javascript
(function anonymous(_callback) {
  "use strict";
  var _context;
  var _x = this._x;
  var _results = new Array(2);
  var _checkDone = () => {
    for (var i = 0; i < _results.length; i++) {
      var item = _results[i];
      if (item === undefined) return false;
      if (item.result !== undefined) {
        _callback(null, item.result);
        return true;
      }
      if (item.error) {
        _callback(item.error);
        return true;
      }
    }
    return false;
  }
  do {
    var _counter = 2;
    var _done = () => {
      _callback();
    };
    if (_counter <= 0) break;
    var _fn0 = _x[0];
    _fn0((_err0, _result0) => {
      if (_err0) {
        if (_counter > 0) {
          if (0 < _results.length && ((_results.length = 1), (_results[0] = {
              error: _err0
            }), _checkDone())) {
            _counter = 0;
          } else {
            if (--_counter === 0) _done();
          }
        }
      } else {
        if (_counter > 0) {
          if (0 < _results.length && (_result0 !== undefined && (_results.length = 1), (_results[0] = {
              result: _result0
            }), _checkDone())) {
            _counter = 0;
          } else {
            if (--_counter === 0) _done();
          }
        }
      }
    });
    if (_counter <= 0) break;
    if (1 >= _results.length) {
      if (--_counter === 0) _done();
    } else {
      var _fn1 = _x[1];
      _fn1((_err1, _result1) => {
        if (_err1) {
          if (_counter > 0) {
            if (1 < _results.length && ((_results.length = 2), (_results[1] = {
                error: _err1
              }), _checkDone())) {
              _counter = 0;
            } else {
              if (--_counter === 0) _done();
            }
          }
        } else {
          if (_counter > 0) {
            if (1 < _results.length && (_result1 !== undefined && (_results.length = 2), (_results[1] = {
                result: _result1
              }), _checkDone())) {
              _counter = 0;
            } else {
              if (--_counter === 0) _done();
            }
          }
        }
      });
    }
  } while (false);
})
```

每个 handler 的最后一位形参是 next，它是一个函数，用户必须手动执行并且传参，这样 callback 会拿到该参数并且执行。从例子可以看出，callback 的执行是取决于注册的 handler 的顺序。



**AsyncParallelHook（异步并行钩子）**

```javascript
const {
	AsyncParallelHook
} = require("./lib/index.js");
let queue = new AsyncParallelHook();

// 订阅
queue.tapAsync('1', next => {
	setTimeout(() => {
		next(1)
	}, 1000)
});
queue.tapAsync('2', next => {
	setTimeout(() => {
		next(2)
	}, 2000)
});

// 发布
queue.callAsync((result) => {
	console.log(result)
	console.log('callback 执行完成')
});
// 1
// callback 执行完成
```

生成的fn

```javascript
(function anonymous(_callback) {
  "use strict";
  var _context;
  var _x = this._x;
  do {
    var _counter = 2;
    var _done = () => {
      _callback();
    };
    if (_counter <= 0) break;
    var _fn0 = _x[0];
    _fn0(_err0 => {
      if (_err0) {
        if (_counter > 0) {
          _callback(_err0);
          _counter = 0;
        }
      } else {
        if (--_counter === 0) _done();
      }
    });
    if (_counter <= 0) break;
    var _fn1 = _x[1];
    _fn1(_err1 => {
      if (_err1) {
        if (_counter > 0) {
          _callback(_err1);
          _counter = 0;
        }
      } else {
        if (--_counter === 0) _done();
      }
    });
  } while (false);
})
```

callback 的执行是取决先执行 next 函数的



## 参考

https://juejin.im/post/5beb8875e51d455e5c4dd83f

https://juejin.im/post/5c12046af265da612b1377aa#heading-7



