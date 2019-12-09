---
title: Vue源码过程（8）- Vuex
date: 2019-10-02 23:03:34
tags: 
- Vue源码过程
categories: 
- Vue源码过程
---

## Vuex

main.js

```javascript
import Vue from 'vue'
import store from './store/index'

new Vue({
  el: '#app',
  computed: {
    count(){
      return store.state.count;
    }
  },
  store,
  methods: {
    increment() {
      // store.commit('increment')
      console.log(this.$store.state.count);
      console.log(this.$store.getters.getCount);
      this.$store.dispatch('incrementAction');
    }
  },
  template: `
  <div @click="increment()">
    count:{{count}}
  </div>
  `
})
```



store/index.js

```javascript
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const state = { //要设置的全局访问的state对象
  count: 0
};

const getters = { //实时监听state值的变化(最新状态)
  getCount() { //方法名随意,主要是用来承载变化的changableNum的值
    return state.count
  }
};

const mutations = { //改变state的初始值的方法
  increment(state) {
    state.count++
  }
};

const actions = {
  incrementAction(context) { //自定义触发mutations里函数的方法，context与store 实例具有相同方法和属性
    context.commit('increment');
  }
}

const store = new Vuex.Store({
  state,
  getters,
  mutations,
  actions
})

export default store;
```

**安装插件**

从`Vue.use(Vuex)`开始，首先`Vuex`是一个对象

![WX20191003-191819@2x](http://www.qinhanwen.xyz/WX20191003-191819@2x.png)

与路由插件一样，都会先执行插件的`install`方法

```javascript
function install (_Vue) {
  if (Vue && _Vue === Vue) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(
        '[vuex] already installed. Vue.use(Vuex) should be called only once.'
      );
    }
    return
  }
  Vue = _Vue;
  applyMixin(Vue);
}

```

走进`applyMixin`方法

```javascript
function applyMixin (Vue) {
  var version = Number(Vue.version.split('.')[0]);

  if (version >= 2) {
    Vue.mixin({ beforeCreate: vuexInit });
  } else {
    var _init = Vue.prototype._init;
    Vue.prototype._init = function (options) {
      if ( options === void 0 ) options = {};

      options.init = options.init
        ? [vuexInit].concat(options.init)
        : vuexInit;
      _init.call(this, options);
    };
  }

  function vuexInit () {
    var options = this.$options;
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store;
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store;
    }
  }
}
```

`versions >= 2`所以调用` Vue.mixin({ beforeCreate: vuexInit });`，也就是为所有组件安装`beforeCreate`声明周期钩子函数



**实例化store**

进入`new Vuex.Store`

```javascript
...
const store = new Vuex.Store({
  state,
  getters,
  mutations,
  actions
})
```



```javascript
var Store = function Store (options) {
  var this$1 = this;
  if ( options === void 0 ) options = {};

  // Auto install if it is not done yet and `window` has `Vue`.
  // To allow users to avoid auto-installation in some cases,
  // this code should be placed here. See #731
  if (!Vue && typeof window !== 'undefined' && window.Vue) {
    install(window.Vue);
  }

  if (process.env.NODE_ENV !== 'production') {
    assert(Vue, "must call Vue.use(Vuex) before creating a store instance.");
    assert(typeof Promise !== 'undefined', "vuex requires a Promise polyfill in this browser.");
    assert(this instanceof Store, "store must be called with the new operator.");
  }

  var plugins = options.plugins; if ( plugins === void 0 ) plugins = [];
  var strict = options.strict; if ( strict === void 0 ) strict = false;

  // store internal state
  this._committing = false;
  this._actions = Object.create(null);
  this._actionSubscribers = [];
  this._mutations = Object.create(null);
  this._wrappedGetters = Object.create(null);
  this._modules = new ModuleCollection(options);
  this._modulesNamespaceMap = Object.create(null);
  this._subscribers = [];
  this._watcherVM = new Vue();

  // bind commit and dispatch to self
  var store = this;
  var ref = this;
  var dispatch = ref.dispatch;
  var commit = ref.commit;
  this.dispatch = function boundDispatch (type, payload) {
    return dispatch.call(store, type, payload)
  };
  this.commit = function boundCommit (type, payload, options) {
    return commit.call(store, type, payload, options)
  };

  // strict mode
  this.strict = strict;

  var state = this._modules.root.state;

  // init root module.
  // this also recursively registers all sub-modules
  // and collects all module getters inside this._wrappedGetters
  installModule(this, state, [], this._modules.root);

  // initialize the store vm, which is responsible for the reactivity
  // (also registers _wrappedGetters as computed properties)
  resetStoreVM(this, state);

  // apply plugins
  plugins.forEach(function (plugin) { return plugin(this$1); });

  var useDevtools = options.devtools !== undefined ? options.devtools : Vue.config.devtools;
  if (useDevtools) {
    devtoolPlugin(this);
  }
};
```



**1）初始化模块**：Vuex 允许我们将 `store` 分割成模块（module）。每个模块拥有自己的 `state`、`mutation`、`action`、`getter`

`store` 本身可以理解为一个 `root module`，它下面的 `modules` 就是子模块，看下创建过程

```javascript
 this._modules = new ModuleCollection(options);
```

![WX20191003-194046@2x](http://www.qinhanwen.xyz/WX20191003-194046@2x.png)

看一下`ModuleCollection`构造函数

```javascript
var ModuleCollection = function ModuleCollection (rawRootModule) {
  // register root module (Vuex.Store options)
  this.register([], rawRootModule, false);
};
ModuleCollection.prototype.get = function get (path) {
  return path.reduce(function (module, key) {
    return module.getChild(key)
  }, this.root)
};

ModuleCollection.prototype.getNamespace = function getNamespace (path) {
  var module = this.root;
  return path.reduce(function (namespace, key) {
    module = module.getChild(key);
    return namespace + (module.namespaced ? key + '/' : '')
  }, '')
};

ModuleCollection.prototype.update = function update$1 (rawRootModule) {
  update([], this.root, rawRootModule);
};

ModuleCollection.prototype.register = function register (path, rawModule, runtime) {
    var this$1 = this;
    if ( runtime === void 0 ) runtime = true;

  if (process.env.NODE_ENV !== 'production') {
    assertRawModule(path, rawModule);
  }

  var newModule = new Module(rawModule, runtime);
  if (path.length === 0) {
    this.root = newModule;
  } else {
    var parent = this.get(path.slice(0, -1));
    parent.addChild(path[path.length - 1], newModule);
  }

  // register nested modules
  if (rawModule.modules) {
    forEachValue(rawModule.modules, function (rawChildModule, key) {
      this$1.register(path.concat(key), rawChildModule, runtime);
    });
  }
};
```

这里调用`this.register([], rawRootModule, false);`，之后通过`new Module(rawModule, runtime)`创建实例

![WX20191003-194829@2x](http://www.qinhanwen.xyz/WX20191003-194829@2x.png)

看一下`Module`构造函数

```javascript
var Module = function Module (rawModule, runtime) {
  this.runtime = runtime;
  // Store some children item
  this._children = Object.create(null);
  // Store the origin module object which passed by programmer
  this._rawModule = rawModule;
  var rawState = rawModule.state;

  // Store the origin module's state
  this.state = (typeof rawState === 'function' ? rawState() : rawState) || {};
};
```

![WX20191003-195156@2x](http://www.qinhanwen.xyz/WX20191003-195156@2x.png)

之后把`newModule` 赋值给了 `this.root`

![WX20191003-195627@2x](http://www.qinhanwen.xyz/WX20191003-195627@2x.png)



**2）安装模块**

![WX20191003-200357@2x](http://www.qinhanwen.xyz/WX20191003-200357@2x.png)

看下`installModule`方法

```javascript
function installModule (store, rootState, path, module, hot) {
  var isRoot = !path.length;
  var namespace = store._modules.getNamespace(path);

  // register in namespace map
  if (module.namespaced) {
    store._modulesNamespaceMap[namespace] = module;
  }

  // set state
  if (!isRoot && !hot) {
    var parentState = getNestedState(rootState, path.slice(0, -1));
    var moduleName = path[path.length - 1];
    store._withCommit(function () {
      Vue.set(parentState, moduleName, module.state);
    });
  }

  var local = module.context = makeLocalContext(store, namespace, path);

  module.forEachMutation(function (mutation, key) {
    var namespacedType = namespace + key;
    registerMutation(store, namespacedType, mutation, local);
  });

  module.forEachAction(function (action, key) {
    var type = action.root ? key : namespace + key;
    var handler = action.handler || action;
    registerAction(store, type, handler, local);
  });

  module.forEachGetter(function (getter, key) {
    var namespacedType = namespace + key;
    registerGetter(store, namespacedType, getter, local);
  });

  module.forEachChild(function (child, key) {
    installModule(store, rootState, path.concat(key), child, hot);
  });
}
```

在这里遍历模块中定义的 `mutations`、`actions`、`getters`分别执行它们的注册工作，实现方式差不多，看一下` module.forEachGetter`

```javascript
Module.prototype.forEachGetter = function forEachGetter (fn) {
  if (this._rawModule.getters) {
    forEachValue(this._rawModule.getters, fn);
  }
};
```

进入`forEachValue(this._rawModule.getters, fn);`

```javascript
function forEachValue (obj, fn) {
  Object.keys(obj).forEach(function (key) { return fn(obj[key], key); });
}
```

其实就是获取`this._rawModule.getters`的键名数组，遍历它，最后调用`registerGetter(store, namespacedType, getter, local);`

```javascript
function registerGetter (store, type, rawGetter, local) {
  if (store._wrappedGetters[type]) {
    if (process.env.NODE_ENV !== 'production') {
      console.error(("[vuex] duplicate getter key: " + type));
    }
    return
  }
  store._wrappedGetters[type] = function wrappedGetter (store) {
    return rawGetter(
      local.state, // local state
      local.getters, // local getters
      store.state, // root state
      store.getters // root getters
    )
  };
}
```

最后添加到`store._wrappedGetters`上，也就是`root store`上

![WX20191003-201815@2x](http://www.qinhanwen.xyz/WX20191003-201815@2x.png)



**3）初始化 `store._vm`**

![WX20191004-163503@2x](http://www.qinhanwen.xyz/WX20191004-163503@2x.png)

看一下`resetStoreVM`

```javascript
function resetStoreVM (store, state, hot) {
  var oldVm = store._vm;

  // bind store public getters
  store.getters = {};
  var wrappedGetters = store._wrappedGetters;
  var computed = {};
  forEachValue(wrappedGetters, function (fn, key) {
    // use computed to leverage its lazy-caching mechanism
    // direct inline function use will lead to closure preserving oldVm.
    // using partial to return function with only arguments preserved in closure enviroment.
    computed[key] = partial(fn, store);
    Object.defineProperty(store.getters, key, {
      get: function () { return store._vm[key]; },
      enumerable: true // for local getters
    });
  });

  // use a Vue instance to store the state tree
  // suppress warnings just in case the user has added
  // some funky global mixins
  var silent = Vue.config.silent;
  Vue.config.silent = true;
  store._vm = new Vue({
    data: {
      $$state: state
    },
    computed: computed
  });
  Vue.config.silent = silent;

  // enable strict mode for new vm
  if (store.strict) {
    enableStrictMode(store);
  }

  if (oldVm) {
    if (hot) {
      // dispatch changes in all subscribed watchers
      // to force getter re-evaluation for hot reloading.
      store._withCommit(function () {
        oldVm._data.$$state = null;
      });
    }
    Vue.nextTick(function () { return oldVm.$destroy(); });
  }
}
```

在这里遍历`store._wrappedGetters`，之后为`store.getters`添加属性描述符

![WX20191004-164945@2x](http://www.qinhanwen.xyz/WX20191004-164945@2x.png)

也就是访问`store.getters`上的属性的时候，其实访问的是`store._vm`上的属性

在下面实例化的`store._vm`属性

![WX20191004-165500@2x](http://www.qinhanwen.xyz/WX20191004-165500@2x.png)

这边调用`new Vue`，传入`computed`属性，走初始化计算属性`watcher`的流程，也传入了`data`属性



看一下初始化`vm`的时候传入的`store`

```javascript
new Vue({
  el: '#app',
  computed: {
    count(){
      return store.state.count;
    }
  },
  store,
  methods: {
    increment() {
      // store.commit('increment')
      console.log(this.$store.state.count);
      console.log(this.$store.getters.getCount);
      this.$store.dispatch('incrementAction');
    }
  },
  template: `
  <div @click="increment()">
    count:{{count}}
  </div>
  `
})
```





**上面是初始化的过程，看下`$store`是在上面时候挂载到`vm`实例上的**

之前`Vuex`做`install`的时候为组件添加的`beforeCreate`钩子被调用的时候添加的

```javascript
  function vuexInit () {
    var options = this.$options;
    // store injection
    if (options.store) {
      this.$store = typeof options.store === 'function'
        ? options.store()
        : options.store;
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store;
    }
  }
```

![WX20191004-195758@2x](http://www.qinhanwen.xyz/WX20191004-195758@2x.png)

前提是创建`vm`实例的时候，参数传入了`store`



**回到上面的例子**

执行到`render`生成`vnode`的时候，会调用计算属性的回调获取`count`

![WX20191004-200604@2x](http://www.qinhanwen.xyz/WX20191004-200604@2x.png)

触发`state`的`get`

![WX20191004-200642@2x](http://www.qinhanwen.xyz/WX20191004-200642@2x.png)

又触发`count`的`get`，拿到`count`的值

![WX20191004-201404@2x](http://www.qinhanwen.xyz/WX20191004-201404@2x.png)

也就是在这个阶段完成依赖收集

**触发点击事件看一下**

```javascript
 increment() {
      console.log(this.$store.state.count);
      console.log(this.$store.getters.getCount);
      this.$store.dispatch('incrementAction');
    }
```

1）打印`this.$store.state.count`，这个没什么好说的

2）第二个获取`getters.getCount`，在前面`resetStoreVM`的时候，为属性添加的`get`描述符被触发

![WX20191004-201820@2x](http://www.qinhanwen.xyz/WX20191004-201820@2x.png)

其实就是触发计算属性`watcher`的`get`方法（这个计算属性`watcher`是在上面创建`store._vm`实例时候创建的），最后调用到`getCount`方法，右边是调用栈

![WX20191004-202730@2x](http://www.qinhanwen.xyz/WX20191004-202730@2x.png)

3）`this.$store.dispatch('incrementAction');`

![WX20191004-205047@2x](http://www.qinhanwen.xyz/WX20191004-205047@2x.png)



![WX20191004-205159@2x](http://www.qinhanwen.xyz/WX20191004-205159@2x.png)



![WX20191004-205318@2x](http://www.qinhanwen.xyz/WX20191004-205318@2x.png)



![WX20191004-205428@2x](http://www.qinhanwen.xyz/WX20191004-205428@2x.png)



![WX20191004-205446@2x](http://www.qinhanwen.xyz/WX20191004-205446@2x.png)



![WX20191004-205531@2x](http://www.qinhanwen.xyz/WX20191004-205531@2x.png)



![WX20191004-205543@2x](http://www.qinhanwen.xyz/WX20191004-205543@2x.png)



![WX20191004-205558@2x](http://www.qinhanwen.xyz/WX20191004-205558@2x.png)

在这里触发`set`

![WX20191004-205650@2x](http://www.qinhanwen.xyz/WX20191004-205650@2x.png)

之前做过依赖收集，所以这里做派发更新

触发视图更新就是` vm._update(vm._render(), hydrating);`



## 小结

也就是在获取`state`上的属性的时候会做依赖收集，`getter`里的属性在创建`store._vm`实例的时候被当做计算属性，会创建计算属性`watcher`，在获取的时候会调用回调，`dispatch`或者`commit`的时候，最后触发视图更新。



## 资料

https://vuex.vuejs.org/zh/guide/modules.html