---
title: Vue源码过程（3）- 组件
date: 2019-08-30 12:55:52
tags: 
- Vue源码过程
categories: 
- Vue源码过程
---

## 组件

每个组件依赖的 CSS、JavaScript、模板、图片等资源放在一起开发和维护。页面可以有多个组件拼接起来。

模板

main.js

```javascript
import Vue from 'vue'
import App from './App'
Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
  el: '#app',
  data: function () {
    return {
      name: 'qinhanwen'
    }
  },
  components: {
    App
  },
  mounted () {
    console.log(this.name)
  },
  template: '<App :name="name" />'
})

```

App.vue

```vue
<template>
  <div>
    {{name}}
  </div>
</template>

<script>
export default {
  name: 'App',
  props:{
    name:String
  }
}
</script>

```

上一章提到的`createElement` 方法，它会创建 VNode

```javascript
function createElement (
  context,
  tag,
  data,
  children,
  normalizationType,
  alwaysNormalize
) {
  if (Array.isArray(data) || isPrimitive(data)) {
    normalizationType = children;
    children = data;
    data = undefined;
  }
  if (isTrue(alwaysNormalize)) {
    normalizationType = ALWAYS_NORMALIZE;
  }
  return _createElement(context, tag, data, children, normalizationType)
}
```

它最后调用的是`_createElement`方法

```javascript
function _createElement (
  context,
  tag,
  data,
  children,
  normalizationType
) {
  if (isDef(data) && isDef((data).__ob__)) {
    process.env.NODE_ENV !== 'production' && warn(
      "Avoid using observed data object as vnode data: " + (JSON.stringify(data)) + "\n" +
      'Always create fresh vnode data objects in each render!',
      context
    );
    return createEmptyVNode()
  }
  // object syntax in v-bind
  if (isDef(data) && isDef(data.is)) {
    tag = data.is;
  }
  if (!tag) {
    // in case of component :is set to falsy value
    return createEmptyVNode()
  }
  // warn against non-primitive key
  if (process.env.NODE_ENV !== 'production' &&
    isDef(data) && isDef(data.key) && !isPrimitive(data.key)
  ) {
    {
      warn(
        'Avoid using non-primitive value as key, ' +
        'use string/number value instead.',
        context
      );
    }
  }
  // support single function children as default scoped slot
  if (Array.isArray(children) &&
    typeof children[0] === 'function'
  ) {
    data = data || {};
    data.scopedSlots = { default: children[0] };
    children.length = 0;
  }
  if (normalizationType === ALWAYS_NORMALIZE) {
    children = normalizeChildren(children);
  } else if (normalizationType === SIMPLE_NORMALIZE) {
    children = simpleNormalizeChildren(children);
  }
  var vnode, ns;
  if (typeof tag === 'string') {
    var Ctor;
    ns = (context.$vnode && context.$vnode.ns) || config.getTagNamespace(tag);
    if (config.isReservedTag(tag)) {
      // platform built-in elements
      vnode = new VNode(
        config.parsePlatformTagName(tag), data, children,
        undefined, undefined, context
      );
    } else if ((!data || !data.pre) && isDef(Ctor = resolveAsset(context.$options, 'components', tag))) {
      // component
      vnode = createComponent(Ctor, data, context, children, tag);
    } else {
      // unknown or unlisted namespaced elements
      // check at runtime because it may get assigned a namespace when its
      // parent normalizes children
      vnode = new VNode(
        tag, data, children,
        undefined, undefined, context
      );
    }
  } else {
    // direct component options / constructor
    vnode = createComponent(tag, data, context, children);
  }
  if (Array.isArray(vnode)) {
    return vnode
  } else if (isDef(vnode)) {
    if (isDef(ns)) { applyNS(vnode, ns); }
    if (isDef(data)) { registerDeepBindings(data); }
    return vnode
  } else {
    return createEmptyVNode()
  }
}
```

因为上一章是一个普通`p`标签，所以实例化了一个普通的VNode

![WX20190830-152243@2x](http://www.qinhanwen.xyz/WX20190830-152243@2x.png)

而这里这里通过`createComponent`实例化一个VNode，

`vnode = createComponent(Ctor, data, context, children, tag);`

```javascript
function createComponent (
  Ctor,
  data,
  context,
  children,
  tag
) {
  if (isUndef(Ctor)) {
    return
  }

  var baseCtor = context.$options._base;

  // plain options object: turn it into a constructor
  if (isObject(Ctor)) {
    Ctor = baseCtor.extend(Ctor);
  }

  // if at this stage it's not a constructor or an async component factory,
  // reject.
  if (typeof Ctor !== 'function') {
    if (process.env.NODE_ENV !== 'production') {
      warn(("Invalid Component definition: " + (String(Ctor))), context);
    }
    return
  }

  // async component
  var asyncFactory;
  if (isUndef(Ctor.cid)) {
    asyncFactory = Ctor;
    Ctor = resolveAsyncComponent(asyncFactory, baseCtor);
    if (Ctor === undefined) {
      // return a placeholder node for async component, which is rendered
      // as a comment node but preserves all the raw information for the node.
      // the information will be used for async server-rendering and hydration.
      return createAsyncPlaceholder(
        asyncFactory,
        data,
        context,
        children,
        tag
      )
    }
  }

  data = data || {};

  // resolve constructor options in case global mixins are applied after
  // component constructor creation
  resolveConstructorOptions(Ctor);

  // transform component v-model data into props & events
  if (isDef(data.model)) {
    transformModel(Ctor.options, data);
  }

  // extract props
  var propsData = extractPropsFromVNodeData(data, Ctor, tag);

  // functional component
  if (isTrue(Ctor.options.functional)) {
    return createFunctionalComponent(Ctor, propsData, data, context, children)
  }

  // extract listeners, since these needs to be treated as
  // child component listeners instead of DOM listeners
  var listeners = data.on;
  // replace with listeners with .native modifier
  // so it gets processed during parent component patch.
  data.on = data.nativeOn;

  if (isTrue(Ctor.options.abstract)) {
    // abstract components do not keep anything
    // other than props & listeners & slot

    // work around flow
    var slot = data.slot;
    data = {};
    if (slot) {
      data.slot = slot;
    }
  }

  // install component management hooks onto the placeholder node
  installComponentHooks(data);

  // return a placeholder vnode
  var name = Ctor.options.name || tag;
  var vnode = new VNode(
    ("vue-component-" + (Ctor.cid) + (name ? ("-" + name) : '')),
    data, undefined, undefined, undefined, context,
    { Ctor: Ctor, propsData: propsData, listeners: listeners, tag: tag, children: children },
    asyncFactory
  );

  return vnode
}
```

在这个过程中关注两个地方

**一、安装组件钩子函数**

```javascript
function installComponentHooks (data) {
  var hooks = data.hook || (data.hook = {});
  for (var i = 0; i < hooksToMerge.length; i++) {
    var key = hooksToMerge[i];
    var existing = hooks[key];
    var toMerge = componentVNodeHooks[key];
    if (existing !== toMerge && !(existing && existing._merged)) {
      hooks[key] = existing ? mergeHook$1(toMerge, existing) : toMerge;
    }
  }
}
```

定义钩子函数的地方:

```javascript
var componentVNodeHooks = {
  init: function init (vnode, hydrating) {
    if (
      vnode.componentInstance &&
      !vnode.componentInstance._isDestroyed &&
      vnode.data.keepAlive
    ) {
      // kept-alive components, treat as a patch
      var mountedNode = vnode; // work around flow
      componentVNodeHooks.prepatch(mountedNode, mountedNode);
    } else {
      var child = vnode.componentInstance = createComponentInstanceForVnode(
        vnode,
        activeInstance
      );
      child.$mount(hydrating ? vnode.elm : undefined, hydrating);
    }
  },

  prepatch: function prepatch (oldVnode, vnode) {
    var options = vnode.componentOptions;
    var child = vnode.componentInstance = oldVnode.componentInstance;
    updateChildComponent(
      child,
      options.propsData, // updated props
      options.listeners, // updated listeners
      vnode, // new parent vnode
      options.children // new children
    );
  },

  insert: function insert (vnode) {
    var context = vnode.context;
    var componentInstance = vnode.componentInstance;
    if (!componentInstance._isMounted) {
      componentInstance._isMounted = true;
      callHook(componentInstance, 'mounted');
    }
    if (vnode.data.keepAlive) {
      if (context._isMounted) {
        // vue-router#1212
        // During updates, a kept-alive component's child components may
        // change, so directly walking the tree here may call activated hooks
        // on incorrect children. Instead we push them into a queue which will
        // be processed after the whole patch process ended.
        queueActivatedComponent(componentInstance);
      } else {
        activateChildComponent(componentInstance, true /* direct */);
      }
    }
  },

  destroy: function destroy (vnode) {
    var componentInstance = vnode.componentInstance;
    if (!componentInstance._isDestroyed) {
      if (!vnode.data.keepAlive) {
        componentInstance.$destroy();
      } else {
        deactivateChildComponent(componentInstance, true /* direct */);
      }
    }
  }
};
```

之后在 VNode 执行 `patch` 的过程中执行相关的钩子函数

**二、实例化VNode**

```javascript
  var vnode = new VNode(
    ("vue-component-" + (Ctor.cid) + (name ? ("-" + name) : '')),
    data, undefined, undefined, undefined, context,
    { Ctor: Ctor, propsData: propsData, listeners: listeners, tag: tag, children: children },
    asyncFactory
  );
```



获得`vnode`之后再调用`vm._update`，把返回的`vnode`转换成DOM

```javascript
  Vue.prototype._update = function (vnode, hydrating) {
    var vm = this;
    var prevEl = vm.$el;
    var prevVnode = vm._vnode;
    var restoreActiveInstance = setActiveInstance(vm);
    vm._vnode = vnode;
    // Vue.prototype.__patch__ is injected in entry points
    // based on the rendering backend used.
    if (!prevVnode) {
      // initial render
      vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */);
    } else {
      // updates
      vm.$el = vm.__patch__(prevVnode, vnode);
    }
    restoreActiveInstance();
    // update __vue__ reference
    if (prevEl) {
      prevEl.__vue__ = null;
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm;
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el;
    }
    // updated hook is called by the scheduler to ensure that children are
    // updated in a parent's updated hook.
  };
```

上一章有说过走进`vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */);`方法，这里有点稍微不同的地方，从`patch`方法里调用进`createElm`方法之后

```javascript
  function createElm (
    vnode,
    insertedVnodeQueue,
    parentElm,
    refElm,
    nested,
    ownerArray,
    index
  ) {
    if (isDef(vnode.elm) && isDef(ownerArray)) {
      // This vnode was used in a previous render!
      // now it's used as a new node, overwriting its elm would cause
      // potential patch errors down the road when it's used as an insertion
      // reference node. Instead, we clone the node on-demand before creating
      // associated DOM element for it.
      vnode = ownerArray[index] = cloneVNode(vnode);
    }

    vnode.isRootInsert = !nested; // for transition enter check
    if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
      return
    }

    var data = vnode.data;
    var children = vnode.children;
    var tag = vnode.tag;
    if (isDef(tag)) {
      if (process.env.NODE_ENV !== 'production') {
        if (data && data.pre) {
          creatingElmInVPre++;
        }
        if (isUnknownElement$$1(vnode, creatingElmInVPre)) {
          warn(
            'Unknown custom element: <' + tag + '> - did you ' +
            'register the component correctly? For recursive components, ' +
            'make sure to provide the "name" option.',
            vnode.context
          );
        }
      }

      vnode.elm = vnode.ns
        ? nodeOps.createElementNS(vnode.ns, tag)
        : nodeOps.createElement(tag, vnode);
      setScope(vnode);

      /* istanbul ignore if */
      {
        createChildren(vnode, children, insertedVnodeQueue);
        if (isDef(data)) {
          invokeCreateHooks(vnode, insertedVnodeQueue);
        }
        insert(parentElm, vnode.elm, refElm);
      }

      if (process.env.NODE_ENV !== 'production' && data && data.pre) {
        creatingElmInVPre--;
      }
    } else if (isTrue(vnode.isComment)) {
      vnode.elm = nodeOps.createComment(vnode.text);
      insert(parentElm, vnode.elm, refElm);
    } else {
      vnode.elm = nodeOps.createTextNode(vnode.text);
      insert(parentElm, vnode.elm, refElm);
    }
  }
```

![WX20190830-180321@2x](http://www.qinhanwen.xyz/WX20190830-180321@2x.png)

因为是个组件，所以看下`createComponent`

```javascript
  function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {
    var i = vnode.data;
    if (isDef(i)) {
      var isReactivated = isDef(vnode.componentInstance) && i.keepAlive;
      if (isDef(i = i.hook) && isDef(i = i.init)) {
        i(vnode, false /* hydrating */);
      }
      // after calling the init hook, if the vnode is a child component
      // it should've created a child instance and mounted it. the child
      // component also has set the placeholder vnode's elm.
      // in that case we can just return the element and be done.
      if (isDef(vnode.componentInstance)) {
        initComponent(vnode, insertedVnodeQueue);
        insert(parentElm, vnode.elm, refElm);
        if (isTrue(isReactivated)) {
          reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm);
        }
        return true
      }
    }
  }
```

![WX20190830-181035@2x](http://www.qinhanwen.xyz/WX20190830-181035@2x.png)

这里的`i`就是上面定义钩子函数中的`init`

```javascript
  init: function init (vnode, hydrating) {
    if (
      vnode.componentInstance &&
      !vnode.componentInstance._isDestroyed &&
      vnode.data.keepAlive
    ) {
      // kept-alive components, treat as a patch
      var mountedNode = vnode; // work around flow
      componentVNodeHooks.prepatch(mountedNode, mountedNode);
    } else {
      var child = vnode.componentInstance = createComponentInstanceForVnode(
        vnode,
        activeInstance
      );
      child.$mount(hydrating ? vnode.elm : undefined, hydrating);
    }
  },
```

**挂载子组件过程**

通过 `createComponentInstanceForVnode` 创建一个 Vue 的实例，然后调用 `child.$mount(hydrating ? vnode.elm : undefined, hydrating); `方法挂载子组件

```javascript
function createComponentInstanceForVnode (
  vnode, // we know it's MountedComponentVNode but flow doesn't
  parent // activeInstance in lifecycle state
) {
  var options = {
    _isComponent: true,
    _parentVnode: vnode,
    parent: parent
  };
  // check inline-template render functions
  var inlineTemplate = vnode.data.inlineTemplate;
  if (isDef(inlineTemplate)) {
    options.render = inlineTemplate.render;
    options.staticRenderFns = inlineTemplate.staticRenderFns;
  }
  return new vnode.componentOptions.Ctor(options)
}
```

![WX20190830-181525@2x](http://www.qinhanwen.xyz/WX20190830-181525@2x.png)

断点继续往下走进`mountComponent`方法

```javascript
function mountComponent (
  vm,
  el,
  hydrating
) {
  vm.$el = el;
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode;
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el || el) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'compiler is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        );
      } else {
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        );
      }
    }
  }
  callHook(vm, 'beforeMount');

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

  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
  new Watcher(vm, updateComponent, noop, {
    before: function before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate');
      }
    }
  }, true /* isRenderWatcher */);
  hydrating = false;

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true;
    callHook(vm, 'mounted');
  }
  return vm
}
```

断点往下到`createElm`方法里创建了一个占位符，并且通过`createChildren`方法，创建了一个文本插入到占位符里，之后执行`insert`方法，但是因为传入的`parentElm`是`undefined`，所以直接`return`了

![WX20190830-202356@2x](http://www.qinhanwen.xyz/WX20190830-202356@2x.png)



**挂载到根组件**

上面的描述的是`i(vnode, false /* hydrating */);`方法的调用，结束以后

```javascript
 function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {
    var i = vnode.data;
    if (isDef(i)) {
      var isReactivated = isDef(vnode.componentInstance) && i.keepAlive;
      if (isDef(i = i.hook) && isDef(i = i.init)) {
        i(vnode, false /* hydrating */);
      }
      // after calling the init hook, if the vnode is a child component
      // it should've created a child instance and mounted it. the child
      // component also has set the placeholder vnode's elm.
      // in that case we can just return the element and be done.
      if (isDef(vnode.componentInstance)) {
        initComponent(vnode, insertedVnodeQueue);
        insert(parentElm, vnode.elm, refElm);
        if (isTrue(isReactivated)) {
          reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm);
        }
        return true
      }
    }
  }
```

走进去` initComponent(vnode, insertedVnodeQueue);`方法，这个方法里把子组件实例的`$el`赋值给了`vnode.elm`，调用结束之后，调用上面代码中的` insert(parentElm, vnode.elm, refElm);`，在这个方法里插入到DOM里

```javascript
  function initComponent (vnode, insertedVnodeQueue) {
    if (isDef(vnode.data.pendingInsert)) {
      insertedVnodeQueue.push.apply(insertedVnodeQueue, vnode.data.pendingInsert);
      vnode.data.pendingInsert = null;
    }
    vnode.elm = vnode.componentInstance.$el;
    if (isPatchable(vnode)) {
      invokeCreateHooks(vnode, insertedVnodeQueue);
      setScope(vnode);
    } else {
      // empty component root.
      // skip all element-related modules except for ref (#3455)
      registerRef(vnode);
      // make sure to invoke the insert hook
      insertedVnodeQueue.push(vnode);
    }
  }
```

![WX20190830-203942@2x](http://www.qinhanwen.xyz/WX20190830-203942@2x.png)



- 组件patch过程：createComponent  ->  子组件初始化  -> 子组件render -> 子组件patch（如果子组件里还有子组件，就重复这个流程）
- 嵌套组件插入顺序，先子后父，最后挂载到body



## 配置合并

模板文件

main.js

```javascript
import Vue from 'vue'
Vue.config.productionTip = false

/* eslint-disable no-new */
Vue.mixin({
  created() {
    console.log('parent created')
  }
})

new Vue({
  el: '#app',
  data: function () {
    return {
      name: 'qinhanwen'
    }
  },
  mounted(){
    console.log('mount');
  },
  created(){
    console.log('child created');
  },
  template: '<p>{{name}}</p>'
})
```



**从初始化开始**

```javascript
  Vue.prototype._init = function (options) {
		...
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options);
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      );
    }
    ...
  };
```

这里的`mergeOptions`做数据合并操作，它合并了`resolveConstructorOptions(vm.constructor)`和`options`，

`resolveConstructorOptions(vm.constructor)`的返回值其实是`vm.constructor.options`

```javascript
function resolveConstructorOptions (Ctor) {
  var options = Ctor.options;
  if (Ctor.super) {
    var superOptions = resolveConstructorOptions(Ctor.super);
    var cachedSuperOptions = Ctor.superOptions;
    if (superOptions !== cachedSuperOptions) {
      // super option changed,
      // need to resolve new options.
      Ctor.superOptions = superOptions;
      // check if there are any late-modified/attached options (#4976)
      var modifiedOptions = resolveModifiedOptions(Ctor);
      // update base extend options
      if (modifiedOptions) {
        extend(Ctor.extendOptions, modifiedOptions);
      }
      options = Ctor.options = mergeOptions(superOptions, Ctor.extendOptions);
      if (options.name) {
        options.components[options.name] = Ctor;
      }
    }
  }
  return options
}
```

那`vm.constructor.options`是什么，我们知道实例的`constructor`等于构造函数，也就是`vm.constructor.options`等于`Vue.options`

```javascript
function initGlobalAPI (Vue) {
  // config
  var configDef = {};
  configDef.get = function () { return config; };
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = function () {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      );
    };
  }
  Object.defineProperty(Vue, 'config', configDef);

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  Vue.util = {
    warn: warn,
    extend: extend,
    mergeOptions: mergeOptions,
    defineReactive: defineReactive$$1
  };

  Vue.set = set;
  Vue.delete = del;
  Vue.nextTick = nextTick;

  // 2.6 explicit observable API
  Vue.observable = function (obj) {
    observe(obj);
    return obj
  };

  Vue.options = Object.create(null);
  ASSET_TYPES.forEach(function (type) {
    Vue.options[type + 's'] = Object.create(null);
  });

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue;

  extend(Vue.options.components, builtInComponents);

  initUse(Vue);
  initMixin$1(Vue);
  initExtend(Vue);
  initAssetRegisters(Vue);
}
```

上面代码中执行了

```javascript
  Vue.options = Object.create(null);
  ASSET_TYPES.forEach(function (type) {
    Vue.options[type + 's'] = Object.create(null);
  });
```

`Vue.options`初始化为一个对象，并且它的`__proto__`是`undefined`，之后遍历`ASSET_TYPES`，为`Vue.options`添加属性，加完以后是这样的。

![WX20190831-155051@2x](http://www.qinhanwen.xyz/WX20190831-155051@2x.png)

后面的`  extend(Vue.options.components, builtInComponents);`，把内置组件添加到了这个属性上。

回到`mergeOptions`，发现第一个参数上面还多了`created`方法，这是哪来的（`_base`就不说了`init`的时候加的）？

![WX20190831-170122@2x](http://www.qinhanwen.xyz/WX20190831-170122@2x.png)

在`initGlobalAPI`中` initMixin$1(Vue);`方法，它为`Vue`构造函数添加了个`mixin`方法

```javascript
function initMixin$1 (Vue) {
  Vue.mixin = function (mixin) {
    this.options = mergeOptions(this.options, mixin);
    return this
  };
}
```

也就是说`created`是我们业务代码中调用`Vue.mixin`的时候添加的

```javascript
Vue.mixin({
  created() {
    console.log('parent created')
  }
})
```

继续看`mergeOptions`方法

```javascript
function mergeOptions (
  parent,
  child,
  vm
) {
  if (process.env.NODE_ENV !== 'production') {
    checkComponents(child);
  }

  if (typeof child === 'function') {
    child = child.options;
  }

  normalizeProps(child, vm);
  normalizeInject(child, vm);
  normalizeDirectives(child);

  // Apply extends and mixins on the child options,
  // but only if it is a raw options object that isn't
  // the result of another mergeOptions call.
  // Only merged options has the _base property.
  if (!child._base) {
    if (child.extends) {
      parent = mergeOptions(parent, child.extends, vm);
    }
    if (child.mixins) {
      for (var i = 0, l = child.mixins.length; i < l; i++) {
        parent = mergeOptions(parent, child.mixins[i], vm);
      }
    }
  }

  var options = {};
  var key;
  for (key in parent) {
    mergeField(key);
  }
  for (key in child) {
    if (!hasOwn(parent, key)) {
      mergeField(key);
    }
  }
  function mergeField (key) {
    var strat = strats[key] || defaultStrat;
    options[key] = strat(parent[key], child[key], vm, key);
  }
  return options
}
```

`mergeOptions` 主要功能就是把 `parent` 和 `child` 这两个对象根据一些合并策略，合并成一个新对象并返回。比较核心的几步，先递归把 `extends` 和 `mixins` 合并到 `parent` 上，然后遍历 `parent`，调用 `mergeField`，然后再遍历 `child`，如果 `key` 不在 `parent` 的自身属性上，则调用 `mergeField`。`mergeField`方法对不同的`key`有不同的处理方式。对于生命周期钩子的处理都是`mergeHook`方法

```javascript
var LIFECYCLE_HOOKS = [
  'beforeCreate',
  'created',
  'beforeMount',
  'mounted',
  'beforeUpdate',
  'updated',
  'beforeDestroy',
  'destroyed',
  'activated',
  'deactivated',
  'errorCaptured',
  'serverPrefetch'
];
...
LIFECYCLE_HOOKS.forEach(function (hook) {
  strats[hook] = mergeHook;
});
...
```

```javascript
function mergeHook (
  parentVal,
  childVal
) {
  var res = childVal
    ? parentVal
      ? parentVal.concat(childVal)
      : Array.isArray(childVal)
        ? childVal
        : [childVal]
    : parentVal;
  return res
    ? dedupeHooks(res)
    : res
}
```

如果不存在 `childVal` ，就返回 `parentVal`；否则再判断是否存在 `parentVal`，如果存在就把 `childVal` 添加到 `parentVal` 后返回新数组；否则返回 `childVal` 的数组。所以回到 `mergeOptions` 函数，一旦 `parent` 和 `child` 都定义了相同的钩子函数，那么它们会把 2 个钩子函数合并成一个数组（父的在栈底，子的在栈顶）。

合并完之后返回的对象是这样的：

![WX20190831-200159@2x](http://www.qinhanwen.xyz/WX20190831-200159@2x.png)



## 生命周期

从`new Vue`开始，到挂载到DOM上，过程中也会运行一些叫做生命周期钩子的函数，给予用户机会在一些特定的场景下添加他们自己的代码。

![](https://cn.vuejs.org/images/lifecycle.png)

源码中执行生命周期方法的是通过`callHook`方法，它在执行的时候，根据传入的字符串 `hook`，去拿到 `vm.$options[hook]` 对应的回调函数数组，然后遍历执行，执行的时候把 `vm` 作为函数执行的上下文。

```javascript
function callHook (vm, hook) {
  // #7573 disable dep collection when invoking lifecycle hooks
  pushTarget();
  var handlers = vm.$options[hook];
  var info = hook + " hook";
  if (handlers) {
    for (var i = 0, j = handlers.length; i < j; i++) {
      invokeWithErrorHandling(handlers[i], vm, null, vm, info);
    }
  }
  if (vm._hasHookEvent) {
    vm.$emit('hook:' + hook);
  }
  popTarget();
}
```



**beforeCreated & created**

```javascript
Vue.prototype._init = function (options?: Object) {
  // ...
  initLifecycle(vm)
  initEvents(vm)
  initRender(vm)
  callHook(vm, 'beforeCreate')
  initInjections(vm) // resolve injections before data/props
  initState(vm)
  initProvide(vm) // resolve provide after data/props
  callHook(vm, 'created')
  // ...
}
```

`initState` 的作用是初始化 `props`、`data`、`methods`、`watch`、`computed` 等属性。

**beforeCreate：**不能获取到 `props`、`data` 中定义的值，也不能调用 `methods` 中定义的函数。

**created：**如果是需要访问 `props`、`data` 等数据的话，可以在这个钩子。



 **beforeMount & mounted**

```javascript
function mountComponent (
  vm,
  el,
  hydrating
) {
  vm.$el = el;
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode;
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el || el) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'compiler is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        );
      } else {
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        );
      }
    }
  }
  callHook(vm, 'beforeMount');

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

  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
  new Watcher(vm, updateComponent, noop, {
    before: function before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate');
      }
    }
  }, true /* isRenderWatcher */);
  hydrating = false;

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true;
    callHook(vm, 'mounted');
  }
  return vm
}
```

在`mountComponent`的时候调用的声明周期

**beforeMount：**在执行`vm._render()`与`vm._update()`之前

**mounted：**在执行`vm._render()`与`vm._update()`之后，实例插入DOM后，执行这个钩子函数



那么子组件的`mounted`钩子是在哪里调用

```javascript
  return function patch (oldVnode, vnode, hydrating, removeOnly) {
  	...
    invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch);
    return vnode.elm
  }
```

从` invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch);`，走进去`invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch);`

```javascript
  function invokeInsertHook (vnode, queue, initial) {
    // delay insert hooks for component root nodes, invoke them after the
    // element is really inserted
    if (isTrue(initial) && isDef(vnode.parent)) {
      vnode.parent.data.pendingInsert = queue;
    } else {
      for (var i = 0; i < queue.length; ++i) {
        queue[i].data.hook.insert(queue[i]);
      }
    }
  }
```

调用走进到`queue[i].data.hook.insert(queue[i]);`

```javascript
  insert: function insert (vnode) {
    var context = vnode.context;
    var componentInstance = vnode.componentInstance;
    if (!componentInstance._isMounted) {
      componentInstance._isMounted = true;
      callHook(componentInstance, 'mounted');
    }
    if (vnode.data.keepAlive) {
      if (context._isMounted) {
        // vue-router#1212
        // During updates, a kept-alive component's child components may
        // change, so directly walking the tree here may call activated hooks
        // on incorrect children. Instead we push them into a queue which will
        // be processed after the whole patch process ended.
        queueActivatedComponent(componentInstance);
      } else {
        activateChildComponent(componentInstance, true /* direct */);
      }
    }
  },
```

同步渲染的子组件而言，`mounted` 钩子函数的执行顺序也是先子后父（都是在实例插入到DOM之后调用的）。



**beforeUpdate & updated**

**beforeDestroy & destroyed**

**activated & deactivated**



## 组件注册

上面说了`Vue.mixin`是在哪定义的，`Vue.component`的定义跟`Vue.mixin`都在`initGlobalAPI`方法里

```javascript
function initGlobalAPI (Vue) {
	...
  initAssetRegisters(Vue);
}
```

`initAssetRegisters`方法

```javascript
function initAssetRegisters (Vue) {
  /**
   * Create asset registration methods.
   */
  ASSET_TYPES.forEach(function (type) {
    Vue[type] = function (
      id,
      definition
    ) {
      if (!definition) {
        return this.options[type + 's'][id]
      } else {
        /* istanbul ignore if */
        if (process.env.NODE_ENV !== 'production' && type === 'component') {
          validateComponentName(id);
        }
        if (type === 'component' && isPlainObject(definition)) {
          definition.name = definition.name || id;
          definition = this.options._base.extend(definition);
        }
        if (type === 'directive' && typeof definition === 'function') {
          definition = { bind: definition, update: definition };
        }
        this.options[type + 's'][id] = definition;
        return definition
      }
    };
  });
}
```

所以`Vue`构造函数一下多了3个静态方法`component`、`directive`和`filter`。

上面有说因为组件不是普通的HTML标签，所以在`_createElement`方法里走进了`createComponent`方法

```javascript
export function _createElement (
  context: Component,
  tag?: string | Class<Component> | Function | Object,
  data?: VNodeData,
  children?: any,
  normalizationType?: number
): VNode | Array<VNode> {
  // ...
  let vnode, ns
  if (typeof tag === 'string') {
    let Ctor
    ns = (context.$vnode && context.$vnode.ns) || config.getTagNamespace(tag)
    if (config.isReservedTag(tag)) {
      // platform built-in elements
      vnode = new VNode(
        config.parsePlatformTagName(tag), data, children,
        undefined, undefined, context
      )
    } else if (isDef(Ctor = resolveAsset(context.$options, 'components', tag))) {
      // component
      vnode = createComponent(Ctor, data, context, children, tag)
    } else {
      // unknown or unlisted namespaced elements
      // check at runtime because it may get assigned a namespace when its
      // parent normalizes children
      vnode = new VNode(
        tag, data, children,
        undefined, undefined, context
      )
    }
  } else {
    // direct component options / constructor
    vnode = createComponent(tag, data, context, children)
  }
  // ...
}
```

走进去之前会先判断组件是否存在，通过`resolveAsset`方法

```javascript
function resolveAsset (
  options,
  type,
  id,
  warnMissing
) {
  /* istanbul ignore if */
  if (typeof id !== 'string') {
    return
  }
  var assets = options[type];
  // check local registration variations first
  if (hasOwn(assets, id)) { return assets[id] }
  var camelizedId = camelize(id);
  if (hasOwn(assets, camelizedId)) { return assets[camelizedId] }
  var PascalCaseId = capitalize(camelizedId);
  if (hasOwn(assets, PascalCaseId)) { return assets[PascalCaseId] }
  // fallback to prototype chain
  var res = assets[id] || assets[camelizedId] || assets[PascalCaseId];
  if (process.env.NODE_ENV !== 'production' && warnMissing && !res) {
    warn(
      'Failed to resolve ' + type.slice(0, -1) + ': ' + id,
      options
    );
  }
  return res
}
```

先通过 `const assets = options[type]` 拿到 `assets`，然后再尝试拿 `assets[id]`，这里有个顺序，先直接使用 `id` 拿，如果不存在，则把 `id` 变成驼峰的形式再拿，如果仍然不存在则在驼峰的基础上把首字母再变成大写的形式再拿，如果仍然拿不到则报错。这样说明了我们在使用 `Vue.component(id, definition)` 全局注册组件的时候，id 可以是连字符、驼峰或首字母大写的形式。



## 总结

举个例子吧，看在例子被插入到DOM上，之间发生了什么（目前已知的事情）

```javascript
import Vue from 'vue'
import App from './App';

Vue.config.productionTip = false

/* eslint-disable no-new */
Vue.mixin({
  created() {
    console.log('parent created')
  }
})
Vue.component('name-box',{
  template:'<p>{{name}}{{age}}</p>',
  props:{
    name:String
  },
  data:function(){
    return {
      age:25
    }
  }
})

new Vue({
  el: '#app',
  data: function () {
    return {
      name: 'qinhanwen'
    }
  },
  components:{
    App
  },
  mounted(){
    console.log('parent mount');
  },
  created(){
    console.log('child created');
  },
  template: '<App :name="name"></App>'
})
```

App.vue

```vue
<template>
  <name-box :name="name"></name-box>
</template>

<script>
export default {
  name: 'App',
  props:{
    name:String
  },
  mounted(){
    console.log('child mount');
  },
  beforeMount(){
    console.log('child beforemount')
  }
}
</script>
```



1. 在`initGlobalAPI`阶段先在`Vue`上，初始化`Vue.option`并且为它添加了初始的字段`components`、`directives`和`filters`，之后` Vue.options._base = Vue;`，接着为它增加了3个静态方法`component`、`directive`和`filter`（当然还有其他的方法，这里不说了）

2. 在业务逻辑中调用`Vue.mixin`，做了个`mergeOptions`操作，之后`Vue.options`就多了`created`字段，是个函数数组。

![WX20190901-115034@2x](http://www.qinhanwen.xyz/WX20190901-115034@2x.png)

3. 之后业务逻辑中调用`Vue.component`，这里的`this.options`就是`Vue.options`。这里的 `this.opitons._base.extend`做的操作其实就是把这个对象转换成一个继承于 `Vue`的构造函数，最后通过 `this.options[type + 's'][id] = definition` 把它挂载到 `Vue.options.components` 上

![WX20190901-115517@2x](http://www.qinhanwen.xyz/WX20190901-115517@2x.png)

4. 然后进入业务代码的`new Vue`调用，先执行初始化`_init`函数，这一步做合并参数（也就是在这个阶段，生命周期钩子函数合并），初始化参数，并且调用`beforeCreate`和`created`钩子函数

5. 之后进入` $mount`方法

6. 进入`mountComponent`，调用父组件的`beforeMount`声明周期钩子

7. 进入`vm._render`方法（这个方法用于生成`vnode`）

   1）走进`createElement`方法，发现`App`不是普通的HTML标签，走进到`resolveAsset`方法判断

![WX20190901-135155@2x](http://www.qinhanwen.xyz/WX20190901-135155@2x.png)



​	2）发现`App`是个组件存在，走进`createComponent`方法（那这个`App`对象是什么时候被加到`context.$options`的呢），在第4步，`_init`函数初始化时候的合并参数`mergeOptions`方法调用这里

![WX20190901-140340@2x](http://www.qinhanwen.xyz/WX20190901-140340@2x.png)

​	3）` installComponentHooks(data);`安装组件钩子函数

​	4）`createComponent`方法返回值是组件`vnode`

8. 生成组件`vnode`之后调用`vm._update`方法

9. 进入到`patch`方法，这个方法做了一些简单的判断，创建了一个空的节点替代了`oldVnode`的值，然后又拿到了`oldVnode.elm`的父节点，之后调用进`createElm`方法

10. 进入`createComponent(vnode, insertedVnodeQueue, parentElm, refElm)`方法

    ```javascript
      function createComponent (vnode, insertedVnodeQueue, parentElm, refElm) {
        var i = vnode.data;
        if (isDef(i)) {
          var isReactivated = isDef(vnode.componentInstance) && i.keepAlive;
          if (isDef(i = i.hook) && isDef(i = i.init)) {
            i(vnode, false /* hydrating */);
          }
          // after calling the init hook, if the vnode is a child component
          // it should've created a child instance and mounted it. the child
          // component also has set the placeholder vnode's elm.
          // in that case we can just return the element and be done.
          if (isDef(vnode.componentInstance)) {
            initComponent(vnode, insertedVnodeQueue);
            insert(parentElm, vnode.elm, refElm);
            if (isTrue(isReactivated)) {
              reactivateComponent(vnode, insertedVnodeQueue, parentElm, refElm);
            }
            return true
          }
        }
      }
    ```

1）` i(vnode, false /* hydrating */);`就是调用了组件的`init`钩子函数

![WX20190901-150842@2x](http://www.qinhanwen.xyz/WX20190901-150842@2x.png)

- 这里的`createComponentInstanceForVnode`方法，这里其实是创建了一个`Vue`的实例

![WX20190901-150751@2x](http://www.qinhanwen.xyz/WX20190901-150751@2x.png)

- 然后调用`$mount`挂载子组件，重复进入第5步的步骤（入参会不一样），直到`tag`是一个普通的HTML标签时，到第10步的`createElm`里的`createComponent(vnode, insertedVnodeQueue, parentElm, refElm)`返回值为`undefined`

  ![WX20190901-153926@2x](http://www.qinhanwen.xyz/WX20190901-153926@2x.png)

- 指针往下走创建占位符`vnode.elm`，就是业务代码中的`name-box`组件的`p`标签

  ![WX20190901-154959@2x](http://www.qinhanwen.xyz/WX20190901-154959@2x.png)

- 接着进入`createChildren`方法，遍历子元素，插入到占位符中，就是上面`p`标签

- 再进入`insert`方法，没有`parentElm`所以这步忽略

- 调用指针返回到`update`方法， 里面有个判断如果父级也是组件，就是`vm.$parent.$el = vm.$el;`

  ![WX20190901-155701@2x](http://www.qinhanwen.xyz/WX20190901-155701@2x.png)

- ` init`执行完了,回到这

![WX20190901-162031@2x](http://www.qinhanwen.xyz/WX20190901-162031@2x.png)

- `initComponent`方法，`vnode.elm=vnode.componentInstance.$el`，这个就是组件实例了，等下要插入到DOM里

![WX20190901-162253@2x](http://www.qinhanwen.xyz/WX20190901-162253@2x.png)

- `insert`方法，这里没有`parentElm`，所以直接跳过了

2）返回到最外层的`createComponent`方法，这是第一次调用`createComponent`的时候，在这里的`insert`方法将占位符插入到了DOM中

![WX20190901-163107@2x](http://www.qinhanwen.xyz/WX20190901-163107@2x.png)



11. 插入到DOM中后，还有一个`invokeInsertHook(vnode, insertedVnodeQueue, isInitialPatch);`，就是执行各组件的`mounted`钩子函数

    ![WX20190901-163511@2x](http://www.qinhanwen.xyz/WX20190901-163511@2x.png)

12. 之后再执行父组件的`mounted`钩子

![WX20190901-163629@2x](http://www.qinhanwen.xyz/WX20190901-163629@2x.png)





过程大概就是

- new Vue   

- init初始化  

- mount 

-  render （这个阶段判断是普通标签还是组件标签，是组件标签就初始化，安装组件钩子函数，最后返回
  vnode） 
- update（进去到patch阶段，这个阶段从最深的地方开始创建占位符，然后把子元素
  加进去，一直到整个DOM结构加入`#app`的父亲下面）

其中各种生命周期先忽略。。



## 异步组件

改个例子

main.js

```javascript
import Vue from 'vue'
Vue.config.productionTip = false
Vue.component('async-example', function (resolve, reject) {
  // 这个特殊的 require 语法告诉 webpack
  // 自动将编译后的代码分割成不同的块，
  // 这些块将通过 Ajax 请求自动下载。
  setTimeout(() => require(['./App'], resolve), 2000)
})

new Vue({
  el: '#app',
  data: function () {
    return {}
  },

  template: `<div>
      <async-example></async-example>
  </div>`
})
```

**创建异步组件的时候，进入`resolveAsyncComponent`方法**

```javascript
function resolveAsyncComponent (
  factory,
  baseCtor
) {
  if (isTrue(factory.error) && isDef(factory.errorComp)) {
    return factory.errorComp
  }

  if (isDef(factory.resolved)) {
    return factory.resolved
  }

  var owner = currentRenderingInstance;
  if (owner && isDef(factory.owners) && factory.owners.indexOf(owner) === -1) {
    // already pending
    factory.owners.push(owner);
  }

  if (isTrue(factory.loading) && isDef(factory.loadingComp)) {
    return factory.loadingComp
  }

  if (owner && !isDef(factory.owners)) {
    var owners = factory.owners = [owner];
    var sync = true;
    var timerLoading = null;
    var timerTimeout = null

    ;(owner).$on('hook:destroyed', function () { return remove(owners, owner); });

    var forceRender = function (renderCompleted) {
      for (var i = 0, l = owners.length; i < l; i++) {
        (owners[i]).$forceUpdate();
      }

      if (renderCompleted) {
        owners.length = 0;
        if (timerLoading !== null) {
          clearTimeout(timerLoading);
          timerLoading = null;
        }
        if (timerTimeout !== null) {
          clearTimeout(timerTimeout);
          timerTimeout = null;
        }
      }
    };

    var resolve = once(function (res) {
      // cache resolved
      factory.resolved = ensureCtor(res, baseCtor);
      // invoke callbacks only if this is not a synchronous resolve
      // (async resolves are shimmed as synchronous during SSR)
      if (!sync) {
        forceRender(true);
      } else {
        owners.length = 0;
      }
    });

    var reject = once(function (reason) {
      process.env.NODE_ENV !== 'production' && warn(
        "Failed to resolve async component: " + (String(factory)) +
        (reason ? ("\nReason: " + reason) : '')
      );
      if (isDef(factory.errorComp)) {
        factory.error = true;
        forceRender(true);
      }
    });

    var res = factory(resolve, reject);

    if (isObject(res)) {
      if (isPromise(res)) {
        // () => Promise
        if (isUndef(factory.resolved)) {
          res.then(resolve, reject);
        }
      } else if (isPromise(res.component)) {
        res.component.then(resolve, reject);

        if (isDef(res.error)) {
          factory.errorComp = ensureCtor(res.error, baseCtor);
        }

        if (isDef(res.loading)) {
          factory.loadingComp = ensureCtor(res.loading, baseCtor);
          if (res.delay === 0) {
            factory.loading = true;
          } else {
            timerLoading = setTimeout(function () {
              timerLoading = null;
              if (isUndef(factory.resolved) && isUndef(factory.error)) {
                factory.loading = true;
                forceRender(false);
              }
            }, res.delay || 200);
          }
        }

        if (isDef(res.timeout)) {
          timerTimeout = setTimeout(function () {
            timerTimeout = null;
            if (isUndef(factory.resolved)) {
              reject(
                process.env.NODE_ENV !== 'production'
                  ? ("timeout (" + (res.timeout) + "ms)")
                  : null
              );
            }
          }, res.timeout);
        }
      }
    }

    sync = false;
    // return in case resolved synchronously
    return factory.loading
      ? factory.loadingComp
      : factory.resolved
  }
}
```

**声明`resolve`**

![WX20190919-222351@2x](http://www.qinhanwen.xyz/WX20190919-222351@2x.png)

进入`once`方法，闭包存了一个`called`的值，只允许传入的`fn`被调用一次

```javascript
function once (fn) {
  var called = false;
  return function () {
    if (!called) {
      called = true;
      fn.apply(this, arguments);
    }
  }
}
```

**之后走到` var res = factory(resolve, reject);`，这个`factory`方法其实就是，声明组件的第二个参数**

![WX20190919-223256@2x](http://www.qinhanwen.xyz/WX20190919-223256@2x.png)



**之后创建一个异步组件占位符**

![WX20190919-224423@2x](http://www.qinhanwen.xyz/WX20190919-224423@2x.png)

进入`createAsyncPlaceholder`方法

```javascript
function createAsyncPlaceholder (
  factory,
  data,
  context,
  children,
  tag
) {
  var node = createEmptyVNode();
  node.asyncFactory = factory;
  node.asyncMeta = { data: data, context: context, children: children, tag: tag };
  return node
}
```

其实就是一个空节点，多了个`asyncFactory`和`asyncMeta`

![WX20190919-224709@2x](http://www.qinhanwen.xyz/WX20190919-224709@2x.png)



**在组件加载的时候，调用到这里**

![WX20190919-223538@2x](http://www.qinhanwen.xyz/WX20190919-223538@2x.png)

之后进入`forceRender`方法

```javascript
function (renderCompleted) {
      for (var i = 0, l = owners.length; i < l; i++) {
        (owners[i]).$forceUpdate();
      }

      if (renderCompleted) {
        owners.length = 0;
        if (timerLoading !== null) {
          clearTimeout(timerLoading);
          timerLoading = null;
        }
        if (timerTimeout !== null) {
          clearTimeout(timerTimeout);
          timerTimeout = null;
        }
      }
    };
```

进入`(owners[i]).$forceUpdate();`方法

```javascript
  Vue.prototype.$forceUpdate = function () {
    var vm = this;
    if (vm._watcher) {
      vm._watcher.update();
    }
  };
```

最后调用的是` vm._watcher.update();`，之后更新视图

