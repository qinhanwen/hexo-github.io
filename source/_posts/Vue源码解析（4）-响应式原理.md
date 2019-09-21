---
title: Vue源码解析（4）- 响应式原理
date: 2019-09-01 20:25:21
tags: 
- Vue源码解析
categories: 
- Vue源码解析
---

## 响应式

数据变化的时候触发DOM的变化，数据变化对DOM的处理有几个问题：

- 修改哪块的 DOM？
- 效率和性能是不是最优的？
- 是每次数据变化都去修改DOM吗



## 响应式对象

[`Object.defineProperty(obj, prop, descriptor)`方法]([https://qinhanwen.github.io/2019/02/18/%E5%AF%B9%E8%B1%A1/](https://qinhanwen.github.io/2019/02/18/对象/))

- 响应式对象核心就是使用这个方法为对象属性添加`getter`与`setter`

- `Vue`会把props、data等变成响应式对象，在创建过程中，发现子属性也是对象，就递归把对象变成响应式

**模板代码：**

main.js

```javascript
import Vue from 'vue'

Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data: function () {
    return {
      message: 'hello',
      personInfo:{
        name:'qinhanwen'
      }
    }
  },
  methods: {
    changeMsg() {
      this.message = 'Hello World!'
      this.personInfo.name = 'qhw';
    }
  },
  template: `<div @click="changeMsg">
  {{ message }}
  {{personInfo.name}}
  </div>`
})
```

**从前两章说过`_init`中的的`initState(vm)`方法开始，这个方法对数据做了一些初始化操作**

```javascript
function initState (vm) {
  vm._watchers = [];
  var opts = vm.$options;
  if (opts.props) { initProps(vm, opts.props); }
  if (opts.methods) { initMethods(vm, opts.methods); }
  if (opts.data) {
    initData(vm);
  } else {
    observe(vm._data = {}, true /* asRootData */);
  }
  if (opts.computed) { initComputed(vm, opts.computed); }
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch);
  }
}
```

**主要看`initData(vm);`方法**

```javascript
function initData (vm) {
  var data = vm.$options.data;
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {};
  if (!isPlainObject(data)) {
    data = {};
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object:\n' +
      'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
      vm
    );
  }
  // proxy data on instance
  var keys = Object.keys(data);
  var props = vm.$options.props;
  var methods = vm.$options.methods;
  var i = keys.length;
  while (i--) {
    var key = keys[i];
    if (process.env.NODE_ENV !== 'production') {
      if (methods && hasOwn(methods, key)) {
        warn(
          ("Method \"" + key + "\" has already been defined as a data property."),
          vm
        );
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== 'production' && warn(
        "The data property \"" + key + "\" is already declared as a prop. " +
        "Use prop default value instead.",
        vm
      );
    } else if (!isReserved(key)) {
      proxy(vm, "_data", key);
    }
  }
  // observe data
  observe(data, true /* asRootData */);
}
```

这个方法做了几件事

1）判断了`data`是否类型是`function`，是的话就调用`getData(data, vm)`，否则直接返回`data`或者一个空对象

2）判断是否有重名的`method`、`prop`与`data`，之后通过`proxy`把每一个值 `vm._data.xxx` 都代理到 `vm.xxx` 上

3）最后调用`observe(data, true /* asRootData */);`，监测数据的变化

**进入`observe`方法：**

```javascript
function observe (value, asRootData) {
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  var ob;
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__;
  } else if (
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    ob = new Observer(value);
  }
  if (asRootData && ob) {
    ob.vmCount++;
  }
  return ob
}
```

这个方法做的事情：

1）判断`value`是不是对象，或者是`VNode`的实例就返回

2）如果有`__ob__`属性并且是`Observer`的实例，说明给对象类型数据添加过`Observer`，就直接返回

3）实例一个`Observer`

**进入实例化`Observer`的地方`new Observer(value)`**

```javascript
var Observer = function Observer (value) {
  this.value = value;
  this.dep = new Dep();
  this.vmCount = 0;
  def(value, '__ob__', this);
  if (Array.isArray(value)) {
    if (hasProto) {
      protoAugment(value, arrayMethods);
    } else {
      copyAugment(value, arrayMethods, arrayKeys);
    }
    this.observeArray(value);
  } else {
    this.walk(value);
  }
};
```

这个方法做的事情：

1）实例化了`Dep`，那么`Dep`是什么，就是个[观察者模式]([https://qinhanwen.github.io/2019/08/25/JavaScript%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F-%E8%A7%82%E5%AF%9F%E8%80%85%E6%A8%A1%E5%BC%8F/#more](https://qinhanwen.github.io/2019/08/25/JavaScript设计模式-观察者模式/#more))嘛

```javascript
var Dep = function Dep () {
  this.id = uid++;
  this.subs = [];
};

Dep.prototype.addSub = function addSub (sub) {
  this.subs.push(sub);
};

Dep.prototype.removeSub = function removeSub (sub) {
  remove(this.subs, sub);
};

Dep.prototype.depend = function depend () {
  if (Dep.target) {
    Dep.target.addDep(this);
  }
};

Dep.prototype.notify = function notify () {
  // stabilize the subscriber list first
  var subs = this.subs.slice();
  if (process.env.NODE_ENV !== 'production' && !config.async) {
    // subs aren't sorted in scheduler if not running async
    // we need to sort them now to make sure they fire in correct
    // order
    subs.sort(function (a, b) { return a.id - b.id; });
  }
  for (var i = 0, l = subs.length; i < l; i++) {
    subs[i].update();
  }
};

Dep.target = null;
var targetStack = [];

function pushTarget (target) {
  targetStack.push(target);
  Dep.target = target;
}

function popTarget () {
  targetStack.pop();
  Dep.target = targetStack[targetStack.length - 1];
}
```

2）执行 `def` 函数把自身实例添加到数据对象 `value` 的 `__ob__` 属性上

```javascript
function def (obj, key, val, enumerable) {
  Object.defineProperty(obj, key, {
    value: val,
    enumerable: !!enumerable,
    writable: true,
    configurable: true
  });
}
```

3）接下来会对 `value` 做判断，对于数组会调用 `observeArray` 方法，否则对纯对象调用 `walk` 方法，进入`walk`方法

```javascript
Observer.prototype.walk = function walk (obj) {
  var keys = Object.keys(obj);
  for (var i = 0; i < keys.length; i++) {
    defineReactive$$1(obj, keys[i]);
  }
};
```

进入`defineReactive$$1`方法

```javascript
function defineReactive$$1 (
  obj,
  key,
  val,
  customSetter,
  shallow
) {
  var dep = new Dep();

  var property = Object.getOwnPropertyDescriptor(obj, key);
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  var getter = property && property.get;
  var setter = property && property.set;
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key];
  }

  var childOb = !shallow && observe(val);
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      var value = getter ? getter.call(obj) : val;
      if (Dep.target) {
        dep.depend();
        if (childOb) {
          childOb.dep.depend();
          if (Array.isArray(value)) {
            dependArray(value);
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      var value = getter ? getter.call(obj) : val;
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter();
      }
      // #7981: for accessor properties without setter
      if (getter && !setter) { return }
      if (setter) {
        setter.call(obj, newVal);
      } else {
        val = newVal;
      }
      childOb = !shallow && observe(newVal);
      dep.notify();
    }
  });
}
```

这个方法做的事情：

1）`  var childOb = !shallow && observe(val);`这里的意思就是如果`val`是对象，会递归调用`observe`

2）然后使用`Object.defineProperty`方法为对象属性添加`getter`与`setter`

![WX20190902-124407@2x](http://www.qinhanwen.xyz/WX20190902-124407@2x.png)

![WX20190902-124858@2x](http://www.qinhanwen.xyz/WX20190902-124858@2x.png)



**所以响应式对象，就是通过`observe`方法，对`data`对象每一项添加属性描述符`getter`和`setter`得来的（这里只看了`data`的）**

这是`data`最后的样子

![WX20190902-125823@2x](http://www.qinhanwen.xyz/WX20190902-125823@2x.png)



## 依赖收集

- 依赖收集就是订阅数据变化的`watcher`的收集，收集的目的是为了当响应式数据发生变化，触发`setter`的时候，通知订阅者去做逻辑处理

- 响应式对象 `getter` 相关的逻辑就是做依赖收集

**依赖收集在`mount`阶段的 `mountComponent` 方法里**

```javascript
function mountComponent (
  vm,
  el,
  hydrating
) {
  ...
  
  var updateComponent;
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = function () {
      var name = vm._name;
      var id = vm._uid;
      var startTag = "vue-perf-start:" + id;
      var endTag = "vue-perf-end:" + id;

      mark(startTag);
      var vnode = vm._render();
      mark(endTag);
      measure(("vue " + name + " render"), startTag, endTag);

      mark(startTag);
      vm._update(vnode, hydrating);
      mark(endTag);
      measure(("vue " + name + " patch"), startTag, endTag);
    };
  } else {
    updateComponent = function () {
      vm._update(vm._render(), hydrating);
    };
  }

  new Watcher(vm, updateComponent, noop, {
    before: function before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate');
      }
    }
  }, true /* isRenderWatcher */);

  ...
}
```

**从`new Watcher`实例化的地方进入**

```javascript
var Watcher = function Watcher (
  vm,
  expOrFn,
  cb,
  options,
  isRenderWatcher
) {
  this.vm = vm;
  if (isRenderWatcher) {
    vm._watcher = this;
  }
  vm._watchers.push(this);
  // options
  if (options) {
    this.deep = !!options.deep;
    this.user = !!options.user;
    this.lazy = !!options.lazy;
    this.sync = !!options.sync;
    this.before = options.before;
  } else {
    this.deep = this.user = this.lazy = this.sync = false;
  }
  this.cb = cb;
  this.id = ++uid$2; // uid for batching
  this.active = true;
  this.dirty = this.lazy; // for lazy watchers
  this.deps = [];
  this.newDeps = [];
  this.depIds = new _Set();
  this.newDepIds = new _Set();
  this.expression = process.env.NODE_ENV !== 'production'
    ? expOrFn.toString()
    : '';
  // parse expression for getter
  if (typeof expOrFn === 'function') {
    this.getter = expOrFn;
  } else {
    this.getter = parsePath(expOrFn);
    if (!this.getter) {
      this.getter = noop;
      process.env.NODE_ENV !== 'production' && warn(
        "Failed watching path: \"" + expOrFn + "\" " +
        'Watcher only accepts simple dot-delimited paths. ' +
        'For full control, use a function instead.',
        vm
      );
    }
  }
  this.value = this.lazy
    ? undefined
    : this.get();
};
```

**走进`this.get()`**

```javascript
Watcher.prototype.get = function get () {
  pushTarget(this);
  var value;
  var vm = this.vm;
  try {
    value = this.getter.call(vm, vm);
  } catch (e) {
    if (this.user) {
      handleError(e, vm, ("getter for watcher \"" + (this.expression) + "\""));
    } else {
      throw e
    }
  } finally {
    // "touch" every property so they are all tracked as
    // dependencies for deep watching
    if (this.deep) {
      traverse(value);
    }
    popTarget();
    this.cleanupDeps();
  }
  return value
};
```

**先执行`pushTarget(this);`，其实就是把`watcher`实例存起来**

![WX20190903-001044@2x](http://www.qinhanwen.xyz/WX20190903-001044@2x.png)

**`value = this.getter.call(vm, vm)`其实就是执行`vm._update(vm._render(), hydrating)`，在` vm._render()` 过程中这个阶段会触发了所有数据的 `getter`。**

**走进`_render`方法**

```javascript
  Vue.prototype._render = function () {
   	...
    ...
    try {
      currentRenderingInstance = vm;
      vnode = render.call(vm._renderProxy, vm.$createElement);
    } catch (e) {
      handleError(e, vm, "render");
      if (process.env.NODE_ENV !== 'production' && vm.$options.renderError) {
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
    ...
  };
```

**走进`render.call(vm._renderProxy, vm.$createElement);`（`render`方法是在说模板编译的时候产生的，后面会说），看一下这里的模板生成的`render`方法**

```javascript
(function anonymous(
) {
with(this){return _c('div',{on:{"click":changeMsg}},[_v("\n  "+_s(message)+"\n  "+_s(personInfo.name)+"\n  ")])}
})
```

顺便说一下[`with`语句](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/with)，语法：

```javascript
with (expression) {
    statement
}
```

`expression`:将给定的表达式添加到在评估语句时使用的作用域链上。表达式周围的括号是必需的。

`statement`:任何语句。要执行多个语句，请使用一个[块](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/block)语句 ({ ... })对这些语句进行分组。

举个例子：

```javascript
function test(obj){
    with(obj){
        return a()
    }    
}
var obj = {
    a:function(params){
        console.log(1213);//1213
    }
}
test(obj);//相当于调用obj上的a
```

**也就是在调用`render`方法的时候触发所有数据了`getter`，完成了依赖收集，那么怎么触发`getter`的呢**

(之前初始化数据的时候的`initData`方法里面有这个一步，所以在访问`message`的时候，会调用这里的`proxyGetter`方法)

![WX20190902-215945@2x](http://www.qinhanwen.xyz/WX20190902-215945@2x.png)

就触发了`this[sourceKey][key]`，意思也就是访问`vm._data.message`

![WX20190902-220304@2x](http://www.qinhanwen.xyz/WX20190902-220304@2x.png)

紧接着又触发了这个`message`的`get`方法

![WX20190902-221723@2x](http://www.qinhanwen.xyz/WX20190902-221723@2x.png)

这个`get`方法做了什么：

1）先判断是否有`getter`，这个是在上面闭包，从这个属性的描述符里拿的

![WX20190903-002017@2x](http://www.qinhanwen.xyz/WX20190903-002017@2x.png)

2）判断是否有`Dep.target`，就是`watcher`的实例，有就调用`dep.depend();`（`dep`是之前实例化的，每个属性都有一个）

```javascript
Dep.prototype.depend = function depend () {
  if (Dep.target) {
    Dep.target.addDep(this);
  }
};
```

进入`addDep`方法，方法里的`this`指向`watcher`实例

```javascript
Watcher.prototype.addDep = function addDep (dep) {
  var id = dep.id;
  if (!this.newDepIds.has(id)) {
    this.newDepIds.add(id);
    this.newDeps.push(dep);
    if (!this.depIds.has(id)) {
      dep.addSub(this);
    }
  }
};
```

`dep.addSub(this);`是把当前`watcher`实例订阅到这个数据持有的 `dep` 的 `subs` 中，这个目的是为后续数据变化时候能通知到哪些 `subs` 做准备。

**之后取`personInfo`属性的时候，又触发了`getter`，并且这个属性是个对象所以进入到了`childOb.dep.depend()`，跟`dep.depend();`一个意思的**

![WX20190903-003637@2x](http://www.qinhanwen.xyz/WX20190903-003637@2x.png)

**取`name`属性时候又重复了一次，这里就不写了，于是完成了全部的依赖收集之后，回到`get`方法里，执行`popTarget()`**

![WX20190903-004631@2x](http://www.qinhanwen.xyz/WX20190903-004631@2x.png)

```javascript
function popTarget () {
  targetStack.pop();
  Dep.target = targetStack[targetStack.length - 1];
}
```

这里把`targetStack`和`Dep.target`恢复成了上一次的状态

**然后执行`this.cleanupDeps()`**

```javascript
Watcher.prototype.cleanupDeps = function cleanupDeps () {
  var i = this.deps.length;
  while (i--) {
    var dep = this.deps[i];
    if (!this.newDepIds.has(dep.id)) {
      dep.removeSub(this);
    }
  }
  var tmp = this.depIds;
  this.depIds = this.newDepIds;
  this.newDepIds = tmp;
  this.newDepIds.clear();
  tmp = this.deps;
  this.deps = this.newDeps;
  this.newDeps = tmp;
  this.newDeps.length = 0;
};
```

Vue 是数据驱动的，所以每次数据变化都会重新`render`，那么 `vm._render()` 方法又会再次执行，并再次触发数据的 getters，所以 `Wathcer` 在构造函数中会初始化 2 个 `Dep` 实例数组，`newDeps`表示新添加的 `Dep` 实例数组，而 `deps` 表示上一次添加的 `Dep` 实例数组。

在执行 `cleanupDeps` 函数的时候，会首先遍历 `deps`，移除对 `dep.subs` 数组中 `Wathcer` 的订阅，然后把 `newDepIds` 和 `depIds` 交换，`newDeps` 和 `deps` 交换，并把 `newDepIds` 和 `newDeps` 清空。



## nextTick

源码目录`src/core/util/next-tick.js`

```javascript

var isUsingMicroTask = false;

var callbacks = [];
var pending = false;

function flushCallbacks () {
  pending = false;
  var copies = callbacks.slice(0);
  callbacks.length = 0;
  for (var i = 0; i < copies.length; i++) {
    copies[i]();
  }
}

// Here we have async deferring wrappers using microtasks.
// In 2.5 we used (macro) tasks (in combination with microtasks).
// However, it has subtle problems when state is changed right before repaint
// (e.g. #6813, out-in transitions).
// Also, using (macro) tasks in event handler would cause some weird behaviors
// that cannot be circumvented (e.g. #7109, #7153, #7546, #7834, #8109).
// So we now use microtasks everywhere, again.
// A major drawback of this tradeoff is that there are some scenarios
// where microtasks have too high a priority and fire in between supposedly
// sequential events (e.g. #4521, #6690, which have workarounds)
// or even between bubbling of the same event (#6566).
var timerFunc;

// The nextTick behavior leverages the microtask queue, which can be accessed
// via either native Promise.then or MutationObserver.
// MutationObserver has wider support, however it is seriously bugged in
// UIWebView in iOS >= 9.3.3 when triggered in touch event handlers. It
// completely stops working after triggering a few times... so, if native
// Promise is available, we will use it:
/* istanbul ignore next, $flow-disable-line */
if (typeof Promise !== 'undefined' && isNative(Promise)) {
  var p = Promise.resolve();
  timerFunc = function () {
    p.then(flushCallbacks);
    // In problematic UIWebViews, Promise.then doesn't completely break, but
    // it can get stuck in a weird state where callbacks are pushed into the
    // microtask queue but the queue isn't being flushed, until the browser
    // needs to do some other work, e.g. handle a timer. Therefore we can
    // "force" the microtask queue to be flushed by adding an empty timer.
    if (isIOS) { setTimeout(noop); }
  };
  isUsingMicroTask = true;
} else if (!isIE && typeof MutationObserver !== 'undefined' && (
  isNative(MutationObserver) ||
  // PhantomJS and iOS 7.x
  MutationObserver.toString() === '[object MutationObserverConstructor]'
)) {
  // Use MutationObserver where native Promise is not available,
  // e.g. PhantomJS, iOS7, Android 4.4
  // (#6466 MutationObserver is unreliable in IE11)
  var counter = 1;
  var observer = new MutationObserver(flushCallbacks);
  var textNode = document.createTextNode(String(counter));
  observer.observe(textNode, {
    characterData: true
  });
  timerFunc = function () {
    counter = (counter + 1) % 2;
    textNode.data = String(counter);
  };
  isUsingMicroTask = true;
} else if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
  // Fallback to setImmediate.
  // Techinically it leverages the (macro) task queue,
  // but it is still a better choice than setTimeout.
  timerFunc = function () {
    setImmediate(flushCallbacks);
  };
} else {
  // Fallback to setTimeout.
  timerFunc = function () {
    setTimeout(flushCallbacks, 0);
  };
}

function nextTick (cb, ctx) {
  var _resolve;
  callbacks.push(function () {
    if (cb) {
      try {
        cb.call(ctx);
      } catch (e) {
        handleError(e, ctx, 'nextTick');
      }
    } else if (_resolve) {
      _resolve(ctx);
    }
  });
  if (!pending) {
    pending = true;
    timerFunc();
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(function (resolve) {
      _resolve = resolve;
    })
  }
}
```

说一下这里做了什么吧

1）首先声明了`timerFunc`变量

2）为`timerFunc`赋值，它会判断是否支持原生的方法，然后赋值为（优先级从上到下），只要满足其中一个就不再做赋值操作

- 等于

```javascript
  timerFunc = function () {
    p.then(flushCallbacks);
    // In problematic UIWebViews, Promise.then doesn't completely break, but
    // it can get stuck in a weird state where callbacks are pushed into the
    // microtask queue but the queue isn't being flushed, until the browser
    // needs to do some other work, e.g. handle a timer. Therefore we can
    // "force" the microtask queue to be flushed by adding an empty timer.
    if (isIOS) {
      setTimeout(noop);
    }
  };
```

- 或者

```javascript
  timerFunc = function () {
    counter = (counter + 1) % 2;
    textNode.data = String(counter);
  };
```

- 或者

```javascript
  timerFunc = function () {
    setImmediate(flushCallbacks);
  };
```

- 或者

```javascript
  timerFunc = function () {
    setTimeout(flushCallbacks, 0);
  };
```



## 派发更新(setter)

当数据发生变化的时候，触发 `setter` 逻辑，把在依赖过程中订阅的的所有观察者，也就是 `watcher`，都触发它们的 `update`过程，在 `nextTick` 后执行所有 `watcher` 的 `run`，最后执行它们的回调函数



```javascript
function defineReactive$$1 (
  obj,
  key,
  val,
  customSetter,
  shallow
) {
  var dep = new Dep();

  var property = Object.getOwnPropertyDescriptor(obj, key);
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  var getter = property && property.get;
  var setter = property && property.set;
  if ((!getter || setter) && arguments.length === 2) {
    val = obj[key];
  }

  var childOb = !shallow && observe(val);
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      var value = getter ? getter.call(obj) : val;
      if (Dep.target) {
        dep.depend();
        if (childOb) {
          childOb.dep.depend();
          if (Array.isArray(value)) {
            dependArray(value);
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      var value = getter ? getter.call(obj) : val;
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter();
      }
      // #7981: for accessor properties without setter
      if (getter && !setter) { return }
      if (setter) {
        setter.call(obj, newVal);
      } else {
        val = newVal;
      }
      childOb = !shallow && observe(newVal);
      dep.notify();
    }
  });
}
```

触发`setter`

1）`childOb = !shallow && observe(newVal);`，进入`observe`如果`typeof`类型判断是`object`，又会变成响应式对象

2）调用`dep.notify();`

```javascript
Dep.prototype.notify = function notify () {
  var subs = this.subs.slice();
  if (process.env.NODE_ENV !== 'production' && !config.async) {
    subs.sort(function (a, b) { return a.id - b.id; });
  }
  for (var i = 0, l = subs.length; i < l; i++) {
    subs[i].update();
  }
};
```

遍历`subs`，调用`update`方法

```javascript
Watcher.prototype.update = function update () {
  /* istanbul ignore else */
  if (this.lazy) {
    this.dirty = true;
  } else if (this.sync) {
    this.run();
  } else {
    queueWatcher(this);
  }
};
```

最后走进`queueWatcher`，`Vue` 在做派发更新的时候不会每次数据改变都触发 `watcher` 的回调，而是把这些 `watcher` 先添加到一个队列里（这里会过滤有重复`id`的`watcher`），然后在 `nextTick` 后执行 `flushSchedulerQueue`。

```javascript
var queue = [];
var has = {};
var waiting = false;
var flushing = false;
var index = 0;
...
...
function queueWatcher (watcher) {
  var id = watcher.id;
  if (has[id] == null) {
    has[id] = true;
    if (!flushing) {
      queue.push(watcher);
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      var i = queue.length - 1;
      while (i > index && queue[i].id > watcher.id) {
        i--;
      }
      queue.splice(i + 1, 0, watcher);
    }
    // queue the flush
    if (!waiting) {
      waiting = true;

      if (process.env.NODE_ENV !== 'production' && !config.async) {
        flushSchedulerQueue();
        return
      }
      nextTick(flushSchedulerQueue);
    }
  }
}
```

`nextTick`方法做的事情就是往`callbacks`数组`push`进去一个匿名函数，之后调用`timerFunc`方法

```javascript
function nextTick (cb, ctx) {
  var _resolve;
  callbacks.push(function () {
    if (cb) {
      try {
        cb.call(ctx);
      } catch (e) {
        handleError(e, ctx, 'nextTick');
      }
    } else if (_resolve) {
      _resolve(ctx);
    }
  });
  if (!pending) {
    pending = true;
    timerFunc();
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(function (resolve) {
      _resolve = resolve;
    })
  }
}
```

进入`timeFunc`

```javascript
timerFunc = function () {   
  p.then(flushCallbacks);
  if (isIOS) { setTimeout(noop); }
};
```

![WX20190903-123424@2x](http://www.qinhanwen.xyz/WX20190903-123424@2x.png)

这里做的事情是把`flushCallbacks`任务放到`微任务队列`，而这个方法做的事情就是遍历`callbacks`然后执行之前`push`进去的每一个匿名函数

```javascript
function flushCallbacks () {
  pending = false;
  var copies = callbacks.slice(0);
  callbacks.length = 0;
  for (var i = 0; i < copies.length; i++) {
    copies[i]();
  }
}
```



**当`setter`操作完成之后开始执行上面的`flushCallbacks`方法**

![WX20190903-132043@2x](http://www.qinhanwen.xyz/WX20190903-132043@2x.png)

走进`copies[i]()`调用

![WX20190903-132122@2x](http://www.qinhanwen.xyz/WX20190903-132122@2x.png)

`cb.call(ctx)`调用到`flushSchedulerQueue`方法

```javascript
function flushSchedulerQueue () {
  currentFlushTimestamp = getNow();
  flushing = true;
  var watcher, id;
  
  queue.sort(function (a, b) { return a.id - b.id; });

  // do not cache length because more watchers might be pushed
  // as we run existing watchers
  for (index = 0; index < queue.length; index++) {
    watcher = queue[index];
    if (watcher.before) {
      watcher.before();
    }
    id = watcher.id;
    has[id] = null;
    watcher.run();
    // in dev build, check and stop circular updates.
    if (process.env.NODE_ENV !== 'production' && has[id] != null) {
      circular[id] = (circular[id] || 0) + 1;
      if (circular[id] > MAX_UPDATE_COUNT) {
        warn(
          'You may have an infinite update loop ' + (
            watcher.user
              ? ("in watcher with expression \"" + (watcher.expression) + "\"")
              : "in a component render function."
          ),
          watcher.vm
        );
        break
      }
    }
  }

  // keep copies of post queues before resetting state
  var activatedQueue = activatedChildren.slice();
  var updatedQueue = queue.slice();

  resetSchedulerState();

  // call component updated and activated hooks
  callActivatedHooks(activatedQueue);
  callUpdatedHooks(updatedQueue);

  // devtool hook
  /* istanbul ignore if */
  if (devtools && config.devtools) {
    devtools.emit('flush');
  }
}
```

这个方法把`queue`数组排序，之后遍历（`queue`数组`push` 进`watcher`的地方在这里）

![WX20190903-132532@2x](http://www.qinhanwen.xyz/WX20190903-132532@2x.png)

遍历每一项的时候，如果有`watcher.before`方法就执行，然后执行`watcher.run`方法。

先看`before`方法，调用`befoerUpdate`钩子函数

```javascript
  before: function before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate');
      }
	}
```

看`run`方法，其实就是再执行

```javascript
 vm._update(vm._render(), hydrating);
```

之后返回，执行到`resetSchedulerState`方法，这个方法做一些初始化操作，把使用过的变量还原。

```javascript
function resetSchedulerState () {
  index = queue.length = activatedChildren.length = 0;
  has = {};
  if (process.env.NODE_ENV !== 'production') {
    circular = {};
  }
  waiting = flushing = false;
}
```





## 补充

**一、数组操作**

Vue 不能检测到以下变动的数组

- 直接使用索引设置某项
- 通过`length`改变数组长度

修改一下示例：

main.js

```javascript
import Vue from 'vue'

Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data: function () {
    return {
      arr: [1, 2, 3, 4, 5],
    }
  },
  methods: {
    changeArr() {
      console.log('change');
      this.arr.push(6);
    },
  },
  template: `<div>
  <p v-for="item in arr" @click="changeArr">
  {{item}}
  </p>
  </div>`
})
```

**在`initData`里生成`响应式对象`的时候，有对数组对象做处理。**

```javascript
var Observer = function Observer (value) {
  this.value = value;
  this.dep = new Dep();
  this.vmCount = 0;
  def(value, '__ob__', this);
  if (Array.isArray(value)) {
    if (hasProto) {
      protoAugment(value, arrayMethods);
    } else {
      copyAugment(value, arrayMethods, arrayKeys);
    }
    this.observeArray(value);
  } else {
    this.walk(value);
  }
};
```

走进`protoAugment`方法

```javascript
function protoAugment (target, src) {
  /* eslint-disable no-proto */
  target.__proto__ = src;
  /* eslint-enable no-proto */
}
```

这个方法把目标元素的原型指向了`src`，也就是指向了`arrayMethods`，看一下`arrayMethods`是什么

```javascript
var arrayProto = Array.prototype;
var arrayMethods = Object.create(arrayProto);

var methodsToPatch = [
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
];

/**
 * Intercept mutating methods and emit events
 */
methodsToPatch.forEach(function (method) {
  // cache original method
  var original = arrayProto[method];
  def(arrayMethods, method, function mutator () {
    var args = [], len = arguments.length;
    while ( len-- ) args[ len ] = arguments[ len ];

    var result = original.apply(this, args);
    var ob = this.__ob__;
    var inserted;
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args;
        break
      case 'splice':
        inserted = args.slice(2);
        break
    }
    if (inserted) { ob.observeArray(inserted); }
    // notify change
    ob.dep.notify();
    return result
  });
});
```

这个方法做了什么：

1）首先，`arrayProto`指向`Array.prototype`

2）然后，`arrayMethods.__proto__= Array.prototype`

3）接着，遍历`methodsToPatch`

![WX20190903-174019@2x](http://www.qinhanwen.xyz/WX20190903-174019@2x.png)

进去`def`

![WX20190903-174209@2x](http://www.qinhanwen.xyz/WX20190903-174209@2x.png)

就是为`arrayMethods`添加下面几个属性

```
  'push',
  'pop',
  'shift',
  'unshift',
  'splice',
  'sort',
  'reverse'
```

访问这几个属性的时候，调用的方法是`mutator`

```javascript
function mutator () {
    var args = [], len = arguments.length;
    while ( len-- ) args[ len ] = arguments[ len ];

    var result = original.apply(this, args);
    var ob = this.__ob__;
    var inserted;
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args;
        break
      case 'splice':
        inserted = args.slice(2);
        break
    }
    if (inserted) { ob.observeArray(inserted); }
    // notify change
    ob.dep.notify();
    return result
  }
```

返回到外面`Observer`方法调用里，进去`this.observeArray`方法

![WX20190903-174959@2x](http://www.qinhanwen.xyz/WX20190903-174959@2x.png)

```javascript
Observer.prototype.observeArray = function observeArray (items) {
  for (var i = 0, l = items.length; i < l; i++) {
    observe(items[i]);
  }
};
```

又进去了`observe`方法，然而我们的数据每一项都是基础类型，就直接`return`了

```javascript
function observe (value, asRootData) {
  if (!isObject(value) || value instanceof VNode) {
    return
  }
  var ob;
  if (hasOwn(value, '__ob__') && value.__ob__ instanceof Observer) {
    ob = value.__ob__;
  } else if (
    shouldObserve &&
    !isServerRendering() &&
    (Array.isArray(value) || isPlainObject(value)) &&
    Object.isExtensible(value) &&
    !value._isVue
  ) {
    ob = new Observer(value);
  }
  if (asRootData && ob) {
    ob.vmCount++;
  }
  return ob
}
```

DOM触发点击事件，进入`mutator`方法，这个方法对于`push`与`unshift`的都是`inserted = args;`

```javascript
function mutator () {
    var args = [], len = arguments.length;
    while ( len-- ) args[ len ] = arguments[ len ];

    var result = original.apply(this, args);
    var ob = this.__ob__;
    var inserted;
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args;
        break
      case 'splice':
        inserted = args.slice(2);
        break
    }
    if (inserted) { ob.observeArray(inserted); }
    // notify change
    ob.dep.notify();
    return result
  }
```

先进入`ob.observeArray(inserted)`发现是个基本类型数据，直接`return`了

然后调用`ob.dep.notify();`

```javascript
Dep.prototype.notify = function notify () {
  // stabilize the subscriber list first
  var subs = this.subs.slice();
  if (process.env.NODE_ENV !== 'production' && !config.async) {
    // subs aren't sorted in scheduler if not running async
    // we need to sort them now to make sure they fire in correct
    // order
    subs.sort(function (a, b) { return a.id - b.id; });
  }
  for (var i = 0, l = subs.length; i < l; i++) {
    subs[i].update();
  }
};
```

断点往下走最后调用到

```javascript
vm._update(vm._render(), hydrating);
```

到这里，后面就不说了



**调整一下例子：**

main.js

```javascript
import Vue from 'vue'

Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data: function () {
    return {
      arr: [1, 2, 3, 4, 5],
    }
  },
  methods: {
    changeArr() {
      console.log('change');
      this.arr[0] = 2;
      this.arr.length = 2;
    },
  },
  template: `<div>
  <p v-for="item in arr" @click="changeArr">
  {{item}}
  </p>
  </div>`
})
```

**直接使用索引设置某项，或者通过`length`改变数组长度无法触发派发更新**

解决方案：

第一个：`Vue.set(example1.items, indexOfItem, newValue)`直接修改某项的值

第二个：`vm.items.splice(newLength)`

```javascript
...
  methods: {
    changeArr() {
      console.log('change');
      Vue.set(this.arr, 0, 2);
      this.arr.splice(2);
    },
  },
...
```

那看下`Vue.set`方法吧

```javascript
function set (target, key, val) {
  if (process.env.NODE_ENV !== 'production' &&
    (isUndef(target) || isPrimitive(target))
  ) {
    warn(("Cannot set reactive property on undefined, null, or primitive value: " + ((target))));
  }
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    target.length = Math.max(target.length, key);
    target.splice(key, 1, val);
    return val
  }
  if (key in target && !(key in Object.prototype)) {
    target[key] = val;
    return val
  }
  var ob = (target).__ob__;
  if (target._isVue || (ob && ob.vmCount)) {
    process.env.NODE_ENV !== 'production' && warn(
      'Avoid adding reactive properties to a Vue instance or its root $data ' +
      'at runtime - declare it upfront in the data option.'
    );
    return val
  }
  if (!ob) {
    target[key] = val;
    return val
  }
  defineReactive$$1(ob.value, key, val);
  ob.dep.notify();
  return val
}
```

其实是调用`splice`方法

![WX20190903-185940@2x](http://www.qinhanwen.xyz/WX20190903-185940@2x.png)



**二、计算属性与监听属性**

**computed 计算属性**

先写例子

main.js

```javascript
import Vue from 'vue'

Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data: function () {
    return {
      firstName: 'qin',
      lastName: 'hanwen'
    }
  },
  computed: {
    totalName: function () {
      return this.firstName + this.lastName
    }
  },
  template: `<div>
  {{totalName}}
  </div>`
})
```

从`initState`方法开始

```javascript
function initState (vm) {
  vm._watchers = [];
  var opts = vm.$options;
  if (opts.props) { initProps(vm, opts.props); }
  if (opts.methods) { initMethods(vm, opts.methods); }
  if (opts.data) {
    initData(vm);
  } else {
    observe(vm._data = {}, true /* asRootData */);
  }
  if (opts.computed) { initComputed(vm, opts.computed); }
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch);
  }
}
```

进入`initComputed`方法

```javascript
function initComputed (vm, computed) {
  // $flow-disable-line
  var watchers = vm._computedWatchers = Object.create(null);
  // computed properties are just getters during SSR
  var isSSR = isServerRendering();

  for (var key in computed) {
    var userDef = computed[key];
    var getter = typeof userDef === 'function' ? userDef : userDef.get;
    if (process.env.NODE_ENV !== 'production' && getter == null) {
      warn(
        ("Getter is missing for computed property \"" + key + "\"."),
        vm
      );
    }

    if (!isSSR) {
      // create internal watcher for the computed property.
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions
      );
    }

    // component-defined computed properties are already defined on the
    // component prototype. We only need to define computed properties defined
    // at instantiation here.
    if (!(key in vm)) {
      defineComputed(vm, key, userDef);
    } else if (process.env.NODE_ENV !== 'production') {
      if (key in vm.$data) {
        warn(("The computed property \"" + key + "\" is already defined in data."), vm);
      } else if (vm.$options.props && key in vm.$options.props) {
        warn(("The computed property \"" + key + "\" is already defined as a prop."), vm);
      }
    }
  }
}
```

1）创建空对象`watchers`和`vm._computedWatchers`

2）遍历计算属性`computed`，拿到计算属性的每一个 `userDef`，然后判断 `userDef` 的类型是否是`function`，否则获取它的 `getter` 函数赋值给`getter`变量

3）创建`watcher`实例并且存在`watchers[key]`中，`key`是计算属性名，`watcher`实例的`getter`就是上面`userDef`的`getter`

4）判断`key`是否在`vm`上，存在的话说明已有重复的声明键名，否则走进`defineComputed`方法

```javascript
function defineComputed (
  target,
  key,
  userDef
) {
  var shouldCache = !isServerRendering();
  if (typeof userDef === 'function') {
    sharedPropertyDefinition.get = shouldCache
      ? createComputedGetter(key)
      : createGetterInvoker(userDef);
    sharedPropertyDefinition.set = noop;
  } else {
    sharedPropertyDefinition.get = userDef.get
      ? shouldCache && userDef.cache !== false
        ? createComputedGetter(key)
        : createGetterInvoker(userDef.get)
      : noop;
    sharedPropertyDefinition.set = userDef.set || noop;
  }
  if (process.env.NODE_ENV !== 'production' &&
      sharedPropertyDefinition.set === noop) {
    sharedPropertyDefinition.set = function () {
      warn(
        ("Computed property \"" + key + "\" was assigned to but it has no setter."),
        this
      );
    };
  }
  Object.defineProperty(target, key, sharedPropertyDefinition);
}
```

这个方法其实就是为计算属性，添加`getter`与`setter`，这里的`getter`方法是这样的

```javascript
function computedGetter () {
    var watcher = this._computedWatchers && this._computedWatchers[key];
    if (watcher) {
      if (watcher.dirty) {
        watcher.evaluate();
      }
      if (Dep.target) {
        watcher.depend();
      }
      return watcher.value
    }
  }
```

然后在`render`的时候触发计算属性的`getter`，拿到计算属性的`watcher`，因为`watcher.dirty`的值为`true`，所以执行`watcher.evaluate`

![WX20190903-233522@2x](http://www.qinhanwen.xyz/WX20190903-233522@2x.png)

断点往下看，发现最终执行了`watcher.getter`（是上面实例化`watcher`的时候存的）

![WX20190903-233741@2x](http://www.qinhanwen.xyz/WX20190903-233741@2x.png)

调用完成后返回，接着执行`watcher.depend();`，断点进去，发现是渲染`watcher`对`计算属性watcher`的订阅

![WX20190903-235049@2x](http://www.qinhanwen.xyz/WX20190903-235049@2x.png)



**watcher 监听属性**

示例

main.js

```javascript
import Vue from 'vue'

Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data: function () {
    return {
      firstName: 'hanwen',
      totalName: '',
    }
  },
  watch: {
    firstName: function (val,oVal) {
      this.totalName = val + oVal;
    },
  },
  methods:{
    changeFirstName(){
      this.firstName = 'qin';
    }
  },
  template: `<div @click="changeFirstName()">
  my name is {{totalName}}
  </div>`
})
```

也从`initState`开始

```javascript
function initState (vm) {
  vm._watchers = [];
  var opts = vm.$options;
  if (opts.props) { initProps(vm, opts.props); }
  if (opts.methods) { initMethods(vm, opts.methods); }
  if (opts.data) {
    initData(vm);
  } else {
    observe(vm._data = {}, true /* asRootData */);
  }
  if (opts.computed) { initComputed(vm, opts.computed); }
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch);
  }
}
```

进入到`initWatch`方法

```javascript
function initWatch (vm, watch) {
  for (var key in watch) {
    var handler = watch[key];
    if (Array.isArray(handler)) {
      for (var i = 0; i < handler.length; i++) {
        createWatcher(vm, key, handler[i]);
      }
    } else {
      createWatcher(vm, key, handler);
    }
  }
}
```

遍历`watch`，进入`createWatcher`方法（如果`watch`的某一项是数组，又会遍历这一项调用`createWatcher`）

```javascript
function createWatcher (
  vm,
  expOrFn,
  handler,
  options
) {
  if (isPlainObject(handler)) {
    options = handler;
    handler = handler.handler;
  }
  if (typeof handler === 'string') {
    handler = vm[handler];
  }
  return vm.$watch(expOrFn, handler, options)
}
```

对`handler`做类型判断，想要拿到回调函数，然后进入`vm.$watch`方法

```javascript
  Vue.prototype.$watch = function (
    expOrFn,
    cb,
    options
  ) {
    var vm = this;
    if (isPlainObject(cb)) {
      return createWatcher(vm, expOrFn, cb, options)
    }
    options = options || {};
    options.user = true;
    var watcher = new Watcher(vm, expOrFn, cb, options);
    if (options.immediate) {
      try {
        cb.call(vm, watcher.value);
      } catch (error) {
        handleError(error, vm, ("callback for immediate watcher \"" + (watcher.expression) + "\""));
      }
    }
    return function unwatchFn () {
      watcher.teardown();
    }
  };
```

1）如果`cb`是对象就继续调用`createWatcher`，因为`watch`可以传入一个对象

2）实例化一个`watcher`，是一个`user wathcer`，这个实例`cb`属性就是`handler`，然后`getter`属性是这个

![WX20190904-130806@2x](http://www.qinhanwen.xyz/WX20190904-130806@2x.png)

```javascript
function parsePath (path) {
  if (bailRE.test(path)) {
    return
  }
  var segments = path.split('.');
  return function (obj) {
    for (var i = 0; i < segments.length; i++) {
      if (!obj) { return }
      obj = obj[segments[i]];
    }
    return obj
  }
}
```

这个方法返回的值就赋值给`watcher.value`，同时这里获取`obj[segments[i]]`触发了`getter`，也是在这里完成了依赖收集

![WX20190904-134857@2x](http://www.qinhanwen.xyz/WX20190904-134857@2x.png)

![WX20190904-135020@2x](http://www.qinhanwen.xyz/WX20190904-135020@2x.png)



3）如果`immediate`为`true`，就立刻调用回调。就是下面这样的输入参数，就会立刻调用`handler`

```javascript
...
watch: {
  firstName: {
    immediate:true,
      handler:function (val,oVal) {
        this.totalName = val + oVal;
      },
  } 
},
...
```



**触发一下点击事件看一下**

![WX20190904-125055@2x](http://www.qinhanwen.xyz/WX20190904-125055@2x.png)

`firstName`的值变化，先进`setter`

![WX20190904-125118@2x](http://www.qinhanwen.xyz/WX20190904-125118@2x.png)

然后进入`dep.notify()`派发更新

![WX20190904-135236@2x](http://www.qinhanwen.xyz/WX20190904-135236@2x.png)

这里的`subs[i]`就是`user watcher`(在上面实例化`user watcher`，调用`parsePath`返回的匿名函数的地方完成的依赖收集)

![WX20190904-185715@2x](http://www.qinhanwen.xyz/WX20190904-185715@2x.png)

走进`queueWatcher`方法

![WX20190904-185856@2x](http://www.qinhanwen.xyz/WX20190904-185856@2x.png)

走到`nextTick`这个方法上面有说，这里就忽略

一直往下走到`watcher.run();`

![WX20190904-233441@2x](http://www.qinhanwen.xyz/WX20190904-233441@2x.png)

这里拿到`firstName`新值和旧值，调用回调函数，就是这个方法

![WX20190904-233642@2x](http://www.qinhanwen.xyz/WX20190904-233642@2x.png)

然后触发`totalName`的`setter`

![WX20190905-000215@2x](http://www.qinhanwen.xyz/WX20190905-000215@2x.png)

关于这个`setter`一开始有点搞不懂为什么执行完`val = newVal`以后，`totalName`的值就变成了新值？后面发现得从`getter`的角度来看，当执行完这句之后，`val`变为新值，所以`getter`取到的就是这个值了

![WX20190905-000445@2x](http://www.qinhanwen.xyz/WX20190905-000445@2x.png)

之后又走`dep.notify()`函数调用，最后调用的是

```javascript
 vm._update(vm._render(), hydrating);
```

重复的步骤就不多说了



**对于`watcher options`**，也就不多说了。

options:

- deep	
- user
- computed
- Sync



**计算属性和监听属性的应用场景：**

计算属性本质上是 `computed watcher`，而侦听属性本质上是 `user watcher`。就应用场景而言，计算属性适合用在模板渲染中，某个值是依赖了其它的响应式对象甚至是计算属性计算而来；而监听属性适用于观测某个值的变化去完成一段复杂的业务逻辑。





## 组件更新

上面的例子都只讲到`dep.notify`后，最后调用` vm._update(vm._render(), hydrating);`，没有讲组件更新的逻辑

改一下例子

main.js

```javascript
import Vue from 'vue'

Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data: function () {
    return {
      firstName: 'qinhanwen',
    }
  },
  methods:{
    changeFirstName(){
      this.firstName = 'qhw';
    }
  },
  template: `<div @click="changeFirstName()">
  my name is {{firstName}}
  </div>`
})

```

触发点击事件之后，前面流程略过，直接进入到`vm._update`方法，因为存在`preVnode`所以走了`else`的逻辑

![WX20190905-123924@2x](http://www.qinhanwen.xyz/WX20190905-123924@2x.png)

然后走进`sameVnode`，判断它们是否是相同的`vnode`，来执行不同的逻辑

![WX20190905-124120@2x](http://www.qinhanwen.xyz/WX20190905-124120@2x.png)

`sameVnode`方法，首先直接判断`key`是否相同，如果相同再判断其他的一些属性是否相同

```javascript
function sameVnode (a, b) {
  return (
    a.key === b.key && (
      (
        a.tag === b.tag &&
        a.isComment === b.isComment &&
        isDef(a.data) === isDef(b.data) &&
        sameInputType(a, b)
      ) || (
        isTrue(a.isAsyncPlaceholder) &&
        a.asyncFactory === b.asyncFactory &&
        isUndef(b.asyncFactory.error)
      )
    )
  )
}
```

因为这里是相同的，走进`patchVnode`方法

```javascript
  function patchVnode (
    oldVnode,
    vnode,
    insertedVnodeQueue,
    ownerArray,
    index,
    removeOnly
  ) {
    if (oldVnode === vnode) {
      return
    }

    if (isDef(vnode.elm) && isDef(ownerArray)) {
      // clone reused vnode
      vnode = ownerArray[index] = cloneVNode(vnode);
    }

    var elm = vnode.elm = oldVnode.elm;

    if (isTrue(oldVnode.isAsyncPlaceholder)) {
      if (isDef(vnode.asyncFactory.resolved)) {
        hydrate(oldVnode.elm, vnode, insertedVnodeQueue);
      } else {
        vnode.isAsyncPlaceholder = true;
      }
      return
    }

    // reuse element for static trees.
    // note we only do this if the vnode is cloned -
    // if the new node is not cloned it means the render functions have been
    // reset by the hot-reload-api and we need to do a proper re-render.
    if (isTrue(vnode.isStatic) &&
      isTrue(oldVnode.isStatic) &&
      vnode.key === oldVnode.key &&
      (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
    ) {
      vnode.componentInstance = oldVnode.componentInstance;
      return
    }

    var i;
    var data = vnode.data;
    if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
      i(oldVnode, vnode);
    }

    var oldCh = oldVnode.children;
    var ch = vnode.children;
    if (isDef(data) && isPatchable(vnode)) {
      for (i = 0; i < cbs.update.length; ++i) { cbs.update[i](oldVnode, vnode); }
      if (isDef(i = data.hook) && isDef(i = i.update)) { i(oldVnode, vnode); }
    }
    if (isUndef(vnode.text)) {
      if (isDef(oldCh) && isDef(ch)) {
        if (oldCh !== ch) { updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly); }
      } else if (isDef(ch)) {
        if (process.env.NODE_ENV !== 'production') {
          checkDuplicateKeys(ch);
        }
        if (isDef(oldVnode.text)) { nodeOps.setTextContent(elm, ''); }
        addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue);
      } else if (isDef(oldCh)) {
        removeVnodes(elm, oldCh, 0, oldCh.length - 1);
      } else if (isDef(oldVnode.text)) {
        nodeOps.setTextContent(elm, '');
      }
    } else if (oldVnode.text !== vnode.text) {
      nodeOps.setTextContent(elm, vnode.text);
    }
    if (isDef(data)) {
      if (isDef(i = data.hook) && isDef(i = i.postpatch)) { i(oldVnode, vnode); }
    }
  }
```

1）执行`update`钩子函数

```javascript
    if (isDef(data) && isPatchable(vnode)) {
      for (i = 0; i < cbs.update.length; ++i) { cbs.update[i](oldVnode, vnode); }
      if (isDef(i = data.hook) && isDef(i = i.update)) { i(oldVnode, vnode); }
    }
```

2）执行`updateChildren`

![WX20190905-130348@2x](http://www.qinhanwen.xyz/WX20190905-130348@2x.png)

```javascript
  function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    var oldStartIdx = 0;
    var newStartIdx = 0;
    var oldEndIdx = oldCh.length - 1;
    var oldStartVnode = oldCh[0];
    var oldEndVnode = oldCh[oldEndIdx];
    var newEndIdx = newCh.length - 1;
    var newStartVnode = newCh[0];
    var newEndVnode = newCh[newEndIdx];
    var oldKeyToIdx, idxInOld, vnodeToMove, refElm;

    // removeOnly is a special flag used only by <transition-group>
    // to ensure removed elements stay in correct relative positions
    // during leaving transitions
    var canMove = !removeOnly;

    if (process.env.NODE_ENV !== 'production') {
      checkDuplicateKeys(newCh);
    }

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      if (isUndef(oldStartVnode)) {
        oldStartVnode = oldCh[++oldStartIdx]; // Vnode has been moved left
      } else if (isUndef(oldEndVnode)) {
        oldEndVnode = oldCh[--oldEndIdx];
      } else if (sameVnode(oldStartVnode, newStartVnode)) {
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
        oldStartVnode = oldCh[++oldStartIdx];
        newStartVnode = newCh[++newStartIdx];
      } else if (sameVnode(oldEndVnode, newEndVnode)) {
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx);
        oldEndVnode = oldCh[--oldEndIdx];
        newEndVnode = newCh[--newEndIdx];
      } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx);
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm));
        oldStartVnode = oldCh[++oldStartIdx];
        newEndVnode = newCh[--newEndIdx];
      } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm);
        oldEndVnode = oldCh[--oldEndIdx];
        newStartVnode = newCh[++newStartIdx];
      } else {
        if (isUndef(oldKeyToIdx)) { oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx); }
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx);
        if (isUndef(idxInOld)) { // New element
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx);
        } else {
          vnodeToMove = oldCh[idxInOld];
          if (sameVnode(vnodeToMove, newStartVnode)) {
            patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
            oldCh[idxInOld] = undefined;
            canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm);
          } else {
            // same key but different element. treat as new element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx);
          }
        }
        newStartVnode = newCh[++newStartIdx];
      }
    }
    if (oldStartIdx > oldEndIdx) {
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm;
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue);
    } else if (newStartIdx > newEndIdx) {
      removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx);
    }
  }
```

因为我们这边的两个节点是相同的，所以又进入`patchVode`方法

![WX20190905-130922@2x](http://www.qinhanwen.xyz/WX20190905-130922@2x.png)

因为节点的`text`不是`undefined`，并且新`vnode.text`不等于旧的`vnode.text`，执行`nodeOps.setTextContent`方法

![WX20190905-131039@2x](http://www.qinhanwen.xyz/WX20190905-131039@2x.png)

这个方法只是把节点的`textContent`直接替换成新的文本

```javascript
function setTextContent (node, text) {
  node.textContent = text;
}
```



**修改一下例子**

main.js

```javascript
import Vue from 'vue'
Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data: function () {
    return {
      userList: [{
          name: '1',
          id: 1
        },
        {
          name: '2',
          id: 2
        },
      ],
    }
  },
  methods: {
    changeUserList() {
      this.userList.reverse().push({
        name: '3',
        id: 3
      });
    }
  },
  template: `<ul @click="changeUserList()">
    <li v-for="item in userList" :key="item.id">{{item.name}}</li>
  </ul>`
})
```

触发点击事件，前面就不说了，从进入到`patchVnode`方法调用开始

```javascript
  function patchVnode (
    oldVnode,
    vnode,
    insertedVnodeQueue,
    ownerArray,
    index,
    removeOnly
  ) {
    if (oldVnode === vnode) {
      return
    }

    if (isDef(vnode.elm) && isDef(ownerArray)) {
      // clone reused vnode
      vnode = ownerArray[index] = cloneVNode(vnode);
    }

    var elm = vnode.elm = oldVnode.elm;

    if (isTrue(oldVnode.isAsyncPlaceholder)) {
      if (isDef(vnode.asyncFactory.resolved)) {
        hydrate(oldVnode.elm, vnode, insertedVnodeQueue);
      } else {
        vnode.isAsyncPlaceholder = true;
      }
      return
    }

    // reuse element for static trees.
    // note we only do this if the vnode is cloned -
    // if the new node is not cloned it means the render functions have been
    // reset by the hot-reload-api and we need to do a proper re-render.
    if (isTrue(vnode.isStatic) &&
      isTrue(oldVnode.isStatic) &&
      vnode.key === oldVnode.key &&
      (isTrue(vnode.isCloned) || isTrue(vnode.isOnce))
    ) {
      vnode.componentInstance = oldVnode.componentInstance;
      return
    }

    var i;
    var data = vnode.data;
    if (isDef(data) && isDef(i = data.hook) && isDef(i = i.prepatch)) {
      i(oldVnode, vnode);
    }

    var oldCh = oldVnode.children;
    var ch = vnode.children;
    if (isDef(data) && isPatchable(vnode)) {
      for (i = 0; i < cbs.update.length; ++i) { cbs.update[i](oldVnode, vnode); }
      if (isDef(i = data.hook) && isDef(i = i.update)) { i(oldVnode, vnode); }
    }
    if (isUndef(vnode.text)) {
      if (isDef(oldCh) && isDef(ch)) {
        if (oldCh !== ch) { updateChildren(elm, oldCh, ch, insertedVnodeQueue, removeOnly); }
      } else if (isDef(ch)) {
        if (process.env.NODE_ENV !== 'production') {
          checkDuplicateKeys(ch);
        }
        if (isDef(oldVnode.text)) { nodeOps.setTextContent(elm, ''); }
        addVnodes(elm, null, ch, 0, ch.length - 1, insertedVnodeQueue);
      } else if (isDef(oldCh)) {
        removeVnodes(elm, oldCh, 0, oldCh.length - 1);
      } else if (isDef(oldVnode.text)) {
        nodeOps.setTextContent(elm, '');
      }
    } else if (oldVnode.text !== vnode.text) {
      nodeOps.setTextContent(elm, vnode.text);
    }
    if (isDef(data)) {
      if (isDef(i = data.hook) && isDef(i = i.postpatch)) { i(oldVnode, vnode); }
    }
  }
```

存了`oldVnode.children`，也存了`vnode.children`

![WX20190905-190113@2x](http://www.qinhanwen.xyz/WX20190905-190113@2x.png)

然后进入`updateChildren`

![WX20190905-191339@2x](http://www.qinhanwen.xyz/WX20190905-191339@2x.png)

```javascript

  function updateChildren (parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
    var oldStartIdx = 0;
    var newStartIdx = 0;
    var oldEndIdx = oldCh.length - 1;
    var oldStartVnode = oldCh[0];
    var oldEndVnode = oldCh[oldEndIdx];
    var newEndIdx = newCh.length - 1;
    var newStartVnode = newCh[0];
    var newEndVnode = newCh[newEndIdx];
    var oldKeyToIdx, idxInOld, vnodeToMove, refElm;

    // removeOnly is a special flag used only by <transition-group>
    // to ensure removed elements stay in correct relative positions
    // during leaving transitions
    var canMove = !removeOnly;

    if (process.env.NODE_ENV !== 'production') {
      checkDuplicateKeys(newCh);
    }

    while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
      if (isUndef(oldStartVnode)) {
        oldStartVnode = oldCh[++oldStartIdx]; // Vnode has been moved left
      } else if (isUndef(oldEndVnode)) {
        oldEndVnode = oldCh[--oldEndIdx];
      } else if (sameVnode(oldStartVnode, newStartVnode)) {
        patchVnode(oldStartVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
        oldStartVnode = oldCh[++oldStartIdx];
        newStartVnode = newCh[++newStartIdx];
      } else if (sameVnode(oldEndVnode, newEndVnode)) {
        patchVnode(oldEndVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx);
        oldEndVnode = oldCh[--oldEndIdx];
        newEndVnode = newCh[--newEndIdx];
      } else if (sameVnode(oldStartVnode, newEndVnode)) { // Vnode moved right
        patchVnode(oldStartVnode, newEndVnode, insertedVnodeQueue, newCh, newEndIdx);
        canMove && nodeOps.insertBefore(parentElm, oldStartVnode.elm, nodeOps.nextSibling(oldEndVnode.elm));
        oldStartVnode = oldCh[++oldStartIdx];
        newEndVnode = newCh[--newEndIdx];
      } else if (sameVnode(oldEndVnode, newStartVnode)) { // Vnode moved left
        patchVnode(oldEndVnode, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
        canMove && nodeOps.insertBefore(parentElm, oldEndVnode.elm, oldStartVnode.elm);
        oldEndVnode = oldCh[--oldEndIdx];
        newStartVnode = newCh[++newStartIdx];
      } else {
        if (isUndef(oldKeyToIdx)) { oldKeyToIdx = createKeyToOldIdx(oldCh, oldStartIdx, oldEndIdx); }
        idxInOld = isDef(newStartVnode.key)
          ? oldKeyToIdx[newStartVnode.key]
          : findIdxInOld(newStartVnode, oldCh, oldStartIdx, oldEndIdx);
        if (isUndef(idxInOld)) { // New element
          createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx);
        } else {
          vnodeToMove = oldCh[idxInOld];
          if (sameVnode(vnodeToMove, newStartVnode)) {
            patchVnode(vnodeToMove, newStartVnode, insertedVnodeQueue, newCh, newStartIdx);
            oldCh[idxInOld] = undefined;
            canMove && nodeOps.insertBefore(parentElm, vnodeToMove.elm, oldStartVnode.elm);
          } else {
            // same key but different element. treat as new element
            createElm(newStartVnode, insertedVnodeQueue, parentElm, oldStartVnode.elm, false, newCh, newStartIdx);
          }
        }
        newStartVnode = newCh[++newStartIdx];
      }
    }
    if (oldStartIdx > oldEndIdx) {
      refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm;
      addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue);
    } else if (newStartIdx > newEndIdx) {
      removeVnodes(parentElm, oldCh, oldStartIdx, oldEndIdx);
    }
  }
```

这里发生的事情大概是这个样子的：

1）

![WX20190905-214901@2x](http://www.qinhanwen.xyz/WX20190905-214901@2x.png)

2）

![WX20190905-215956@2x](http://www.qinhanwen.xyz/WX20190905-215956@2x.png)

3）

![WX20190905-230734@2x](http://www.qinhanwen.xyz/WX20190905-230734@2x.png)

4）最后一个比较特别，`createElm`创建了一个`text:3`，然后`insert`进去

![WX20190905-231255@2x](http://www.qinhanwen.xyz/WX20190905-231255@2x.png)

**总结**

新旧节点相同的更新流程是去获取它们的 `children`，根据不同情况做不同的更新逻辑。

新旧节点不同的更新流程是创建新节点->更新父占位符节点->删除旧节点。

