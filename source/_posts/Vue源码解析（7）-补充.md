---
title: Vue源码过程（7）- 补充
date: 2019-09-08 14:23:31
tags: 
- Vue源码过程
categories: 
- Vue源码过程
---

## 补充

### 事件

改一下例子

main.js

```javascript
import Vue from 'vue'
import App from './App';
Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data: function () {
    return {
      message: "123"
    }
  },
  components: {
    App
  },
  methods: {
    clickHandler() {
      console.log('Child clicked!')
    },
    selectHandler() {
      console.log('Child select!')
    }
  },
  template: `<div> 
  <App @select="selectHandler" @click.native.prevent="clickHandler" />
  </div>`
})
```

App.vue

```vue
<template>
  <button @click="clickHandler($event)">click me</button>
</template>

<script>
export default {
  name: "App",
  props: {
    firstName: String
  },
  methods: {
    clickHandler(e) {
      console.log("Button clicked!", e);
      this.$emit("select");
    }
  }
};
</script>
```

从` var code = ast ? genElement(ast, state) : '_c("div")';`开始，先看生成的`ast`，进入`genElement`方法

![WX20190906-230735@2x](http://www.qinhanwen.xyz/WX20190906-230735@2x.png)

```javascript
function generate (
  ast,
  options
) {
  var state = new CodegenState(options);
  var code = ast ? genElement(ast, state) : '_c("div")';
  return {
    render: ("with(this){return " + code + "}"),
    staticRenderFns: state.staticRenderFns
  }
}
```

进入`genElement`

```javascript
function genElement (el, state) {
  if (el.parent) {
    el.pre = el.pre || el.parent.pre;
  }

  if (el.staticRoot && !el.staticProcessed) {
    return genStatic(el, state)
  } else if (el.once && !el.onceProcessed) {
    return genOnce(el, state)
  } else if (el.for && !el.forProcessed) {
    return genFor(el, state)
  } else if (el.if && !el.ifProcessed) {
    return genIf(el, state)
  } else if (el.tag === 'template' && !el.slotTarget && !state.pre) {
    return genChildren(el, state) || 'void 0'
  } else if (el.tag === 'slot') {
    return genSlot(el, state)
  } else {
    // component or element
    var code;
    if (el.component) {
      code = genComponent(el.component, el, state);
    } else {
      var data;
      if (!el.plain || (el.pre && state.maybeComponent(el))) {
        data = genData$2(el, state);
      }

      var children = el.inlineTemplate ? null : genChildren(el, state, true);
      code = "_c('" + (el.tag) + "'" + (data ? ("," + data) : '') + (children ? ("," + children) : '') + ")";
    }
    // module transforms
    for (var i = 0; i < state.transforms.length; i++) {
      code = state.transforms[i](el, code);
    }
    return code
  }
}
```

进入`genChildren`

```javascript
function genChildren (
  el,
  state,
  checkSkip,
  altGenElement,
  altGenNode
) {
  var children = el.children;
  if (children.length) {
    var el$1 = children[0];
    // optimize single v-for
    if (children.length === 1 &&
      el$1.for &&
      el$1.tag !== 'template' &&
      el$1.tag !== 'slot'
    ) {
      var normalizationType = checkSkip
        ? state.maybeComponent(el$1) ? ",1" : ",0"
        : "";
      return ("" + ((altGenElement || genElement)(el$1, state)) + normalizationType)
    }
    var normalizationType$1 = checkSkip
      ? getNormalizationType(children, state.maybeComponent)
      : 0;
    var gen = altGenNode || genNode;
    return ("[" + (children.map(function (c) { return gen(c, state); }).join(',')) + "]" + (normalizationType$1 ? ("," + normalizationType$1) : ''))
  }
}
```

这里遍历子节点数组，进入`gen`方法

```javascript
function genNode (node, state) {
  if (node.type === 1) {
    return genElement(node, state)
  } else if (node.type === 3 && node.isComment) {
    return genComment(node)
  } else {
    return genText(node)
  }
}
```

又进到`genElement`

![WX20190906-232213@2x](http://www.qinhanwen.xyz/WX20190906-232213@2x.png)

这次走到了`genData$2`

```javascript
function genData$2 (el, state) {
  var data = '{';
  var dirs = genDirectives(el, state);
  if (dirs) { data += dirs + ','; }
  if (el.key) {
    data += "key:" + (el.key) + ",";
  }
  if (el.ref) {
    data += "ref:" + (el.ref) + ",";
  }
  if (el.refInFor) {
    data += "refInFor:true,";
  }
  if (el.pre) {
    data += "pre:true,";
  }
  if (el.component) {
    data += "tag:\"" + (el.tag) + "\",";
  }
  for (var i = 0; i < state.dataGenFns.length; i++) {
    data += state.dataGenFns[i](el);
  }
  if (el.attrs) {
    data += "attrs:" + (genProps(el.attrs)) + ",";
  }
  if (el.props) {
    data += "domProps:" + (genProps(el.props)) + ",";
  }
  if (el.events) {
    data += (genHandlers(el.events, false)) + ",";
  }
  if (el.nativeEvents) {
    data += (genHandlers(el.nativeEvents, true)) + ",";
  }
  if (el.slotTarget && !el.slotScope) {
    data += "slot:" + (el.slotTarget) + ",";
  }
  if (el.scopedSlots) {
    data += (genScopedSlots(el, el.scopedSlots, state)) + ",";
  }
  if (el.model) {
    data += "model:{value:" + (el.model.value) + ",callback:" + (el.model.callback) + ",expression:" + (el.model.expression) + "},";
  }
  if (el.inlineTemplate) {
    var inlineTemplate = genInlineTemplate(el, state);
    if (inlineTemplate) {
      data += inlineTemplate + ",";
    }
  }
  data = data.replace(/,$/, '') + '}';
  if (el.dynamicAttrs) {
    data = "_b(" + data + ",\"" + (el.tag) + "\"," + (genProps(el.dynamicAttrs)) + ")";
  }
  if (el.wrapData) {
    data = el.wrapData(data);
  }
  if (el.wrapListeners) {
    data = el.wrapListeners(data);
  }
  return data
}
```

1）声明`data`初始值为`{`

2）这两个逻辑中，为`data`拼接事件

```javascript
  if (el.events) {
    data += (genHandlers(el.events, false)) + ",";
  }
  if (el.nativeEvents) {
    data += (genHandlers(el.nativeEvents, true)) + ",";
  }
```

```javascript
"{on:{"select":selectHandler},nativeOn:{"click":function($event){return clickHandler($event)}},"
```

3）最后去掉尾巴的逗号



所以最后得到的`render`方法是这样的

```javascript
"with(this){return _c('div',[_c('App',{on:{"select":selectHandler},nativeOn:{"click":function($event){return clickHandler($event)}}})],1)}"
```



看一下生成`vnode`调用`render`方法的时候，发现`App`是个`component`，不是普通的HTML标签，所以创建组件`vnode`，下面是创建组件`vnode`的过程：

![WX20190907-125550@2x](http://www.qinhanwen.xyz/WX20190907-125550@2x.png)

![WX20190907-125619@2x](http://www.qinhanwen.xyz/WX20190907-125619@2x.png)

变量`lsteners`保存了`data.on`，并将`data.on`指向`data.nativeOn`，然后为组件安装声明周期钩子函数

![WX20190907-133025@2x](http://www.qinhanwen.xyz/WX20190907-133025@2x.png)

之后创建组件`vnode`，这就是生成的组件`vnode`

![WX20190907-133856@2x](http://www.qinhanwen.xyz/WX20190907-133856@2x.png)



进入`update`方法，首先是创建`div`占位符，然后遍历`children`，创建子节点，就看一下创建组件`App`的过程，在`createComponent`的时候，会调用子组件的`init`钩子方法

![WX20190907-145809@2x](http://www.qinhanwen.xyz/WX20190907-145809@2x.png)

最后调用`child.$mount`挂载子组件，在挂载子组件过程中，会先生成子组件`vnode`，之后再挂载。

挂载之前会为元素添加事件了

![WX20190907-151133@2x](http://www.qinhanwen.xyz/WX20190907-151133@2x.png)

进入`invokeCreateHooks`方法

```javascript
  function invokeCreateHooks (vnode, insertedVnodeQueue) {
    for (var i$1 = 0; i$1 < cbs.create.length; ++i$1) {
      cbs.create[i$1](emptyNode, vnode);
    }
    i = vnode.data.hook; // Reuse variable
    if (isDef(i)) {
      if (isDef(i.create)) { i.create(emptyNode, vnode); }
      if (isDef(i.insert)) { insertedVnodeQueue.push(vnode); }
    }
  }
```

![WX20190907-151217@2x](http://www.qinhanwen.xyz/WX20190907-151217@2x.png)

进入`updateDOMListeners`方法

```javascript
function updateDOMListeners (oldVnode, vnode) {
  if (isUndef(oldVnode.data.on) && isUndef(vnode.data.on)) {
    return
  }
  var on = vnode.data.on || {};
  var oldOn = oldVnode.data.on || {};
  target$1 = vnode.elm;
  normalizeEvents(on);
  updateListeners(on, oldOn, add$1, remove$2, createOnceHandler$1, vnode.context);
  target$1 = undefined;
}
```

![WX20190907-151354@2x](http://www.qinhanwen.xyz/WX20190907-151354@2x.png)

进入`updateListeners`方法

```javascript
function updateListeners (
  on,
  oldOn,
  add,
  remove$$1,
  createOnceHandler,
  vm
) {
  var name, def$$1, cur, old, event;
  for (name in on) {
    def$$1 = cur = on[name];
    old = oldOn[name];
    event = normalizeEvent(name);
    if (isUndef(cur)) {
      process.env.NODE_ENV !== 'production' && warn(
        "Invalid handler for event \"" + (event.name) + "\": got " + String(cur),
        vm
      );
    } else if (isUndef(old)) {
      if (isUndef(cur.fns)) {
        cur = on[name] = createFnInvoker(cur, vm);
      }
      if (isTrue(event.once)) {
        cur = on[name] = createOnceHandler(event.name, cur, event.capture);
      }
      add(event.name, cur, event.capture, event.passive, event.params);
    } else if (cur !== old) {
      old.fns = cur;
      on[name] = old;
    }
  }
  for (name in oldOn) {
    if (isUndef(on[name])) {
      event = normalizeEvent(name);
      remove$$1(event.name, oldOn[name], event.capture);
    }
  }
}
```

![WX20190907-151541@2x](http://www.qinhanwen.xyz/WX20190907-151541@2x.png)

进入`add`方法

```javascript
function add$1 (
  name,
  handler,
  capture,
  passive
) {
  // async edge case #6566: inner click event triggers patch, event handler
  // attached to outer element during patch, and triggered again. This
  // happens because browsers fire microtask ticks between event propagation.
  // the solution is simple: we save the timestamp when a handler is attached,
  // and the handler would only fire if the event passed to it was fired
  // AFTER it was attached.
  if (useMicrotaskFix) {
    var attachedTimestamp = currentFlushTimestamp;
    var original = handler;
    handler = original._wrapper = function (e) {
      if (
        // no bubbling, should always fire.
        // this is just a safety net in case event.timeStamp is unreliable in
        // certain weird environments...
        e.target === e.currentTarget ||
        // event is fired after handler attachment
        e.timeStamp >= attachedTimestamp ||
        // bail for environments that have buggy event.timeStamp implementations
        // #9462 iOS 9 bug: event.timeStamp is 0 after history.pushState
        // #9681 QtWebEngine event.timeStamp is negative value
        e.timeStamp <= 0 ||
        // #9448 bail if event is fired in another document in a multi-page
        // electron/nw.js app, since event.timeStamp will be using a different
        // starting reference
        e.target.ownerDocument !== document
      ) {
        return original.apply(this, arguments)
      }
    };
  }
  target$1.addEventListener(
    name,
    handler,
    supportsPassive
      ? { capture: capture, passive: passive }
      : capture
  );
}
```

在这里为元素添加事件



点击一下触发点击事件

![WX20190907-165247@2x](http://www.qinhanwen.xyz/WX20190907-165247@2x.png)

走进`original.apply(this,arguments)`

```javascript
  function invoker () {
    var arguments$1 = arguments;

    var fns = invoker.fns;
    if (Array.isArray(fns)) {
      var cloned = fns.slice();
      for (var i = 0; i < cloned.length; i++) {
        invokeWithErrorHandling(cloned[i], null, arguments$1, vm, "v-on handler");
      }
    } else {
      // return handler return value for single handlers
      return invokeWithErrorHandling(fns, null, arguments, vm, "v-on handler")
    }
  }
```

走进`invokeWithErrorHandling(fns, null, arguments, vm, "v-on handler")`

```javascript
function invokeWithErrorHandling (
  handler,
  context,
  args,
  vm,
  info
) {
  var res;
  try {
    res = args ? handler.apply(context, args) : handler.call(context);
    if (res && !res._isVue && isPromise(res) && !res._handled) {
      res.catch(function (e) { return handleError(e, vm, info + " (Promise/async)"); });
      // issue #9511
      // avoid catch triggering multiple times when nested calls
      res._handled = true;
    }
  } catch (e) {
    handleError(e, vm, info);
  }
  return res
}
```

走进` handler.apply(context, args)`

![WX20190907-165937@2x](http://www.qinhanwen.xyz/WX20190907-165937@2x.png)

触发`App`组件的`click`事件，不过触发之前，先回从`_vm`上取`clickHandler`，所以走到了`get`

![WX20190907-170314@2x](http://www.qinhanwen.xyz/WX20190907-170314@2x.png)

这里补充一下`initMethods`方法吧，在初始化调用`initState`的时候会走进`initMethods`方法

```javascript
function initMethods (vm, methods) {
  var props = vm.$options.props;
  for (var key in methods) {
    if (process.env.NODE_ENV !== 'production') {
      if (typeof methods[key] !== 'function') {
        warn(
          "Method \"" + key + "\" has type \"" + (typeof methods[key]) + "\" in the component definition. " +
          "Did you reference the function correctly?",
          vm
        );
      }
      if (props && hasOwn(props, key)) {
        warn(
          ("Method \"" + key + "\" has already been defined as a prop."),
          vm
        );
      }
      if ((key in vm) && isReserved(key)) {
        warn(
          "Method \"" + key + "\" conflicts with an existing Vue instance method. " +
          "Avoid defining component methods that start with _ or $."
        );
      }
    }
    vm[key] = typeof methods[key] !== 'function' ? noop : bind(methods[key], vm);
  }
}
```

这个方法，其实就是在`vm`上添加属性方法（`bind(methods[key],vm)`返回是个绑定了`this`方法）

```javascript
function nativeBind (fn, ctx) {
  return fn.bind(ctx)
}
```



继续说触发了`App`组件`click`方法

![WX20190907-171130@2x](http://www.qinhanwen.xyz/WX20190907-171130@2x.png)

先打印出`Button clicked,xxxxxx`，之后调用`this.$emit`

```javascript
  Vue.prototype.$emit = function (event) {
    var vm = this;
    if (process.env.NODE_ENV !== 'production') {
      var lowerCaseEvent = event.toLowerCase();
      if (lowerCaseEvent !== event && vm._events[lowerCaseEvent]) {
        tip(
          "Event \"" + lowerCaseEvent + "\" is emitted in component " +
          (formatComponentName(vm)) + " but the handler is registered for \"" + event + "\". " +
          "Note that HTML attributes are case-insensitive and you cannot use " +
          "v-on to listen to camelCase events when using in-DOM templates. " +
          "You should probably use \"" + (hyphenate(event)) + "\" instead of \"" + event + "\"."
        );
      }
    }
    var cbs = vm._events[event];
    if (cbs) {
      cbs = cbs.length > 1 ? toArray(cbs) : cbs;
      var args = toArray(arguments, 1);
      var info = "event handler for \"" + event + "\"";
      for (var i = 0, l = cbs.length; i < l; i++) {
        invokeWithErrorHandling(cbs[i], vm, args, vm, info);
      }
    }
    return vm
  };
```

获取`vm._events[event]`，然后遍历它，调用每一项的方法，那么`vm._events`在什么时候加的呢？在`initEvents(vm);`的时候

```javascript
function initEvents (vm) {
  vm._events = Object.create(null);
  vm._hasHookEvent = false;
  // init parent attached events
  var listeners = vm.$options._parentListeners;
  if (listeners) {
    updateComponentListeners(vm, listeners);
  }
}
```



接着就是调用`selectHandler`方法

![WX20190907-222530@2x](http://www.qinhanwen.xyz/WX20190907-222530@2x.png)

然后冒泡，触发

![WX20190907-222753@2x](http://www.qinhanwen.xyz/WX20190907-222753@2x.png)





### **双向绑定原理**

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
      message:"123"
    }
  },
  methods: {

  },
  template: `<div>
    <input v-model="message" />
    {{message}}
  </div>`
})
```

从`var code = generate(ast, options);`开始

```javascript
var createCompiler = createCompilerCreator(function baseCompile (
  template,
  options
) {
  var ast = parse(template.trim(), options);
  if (options.optimize !== false) {
    optimize(ast, options);
  }
  var code = generate(ast, options);
  return {
    ast: ast,
    render: code.render,
    staticRenderFns: code.staticRenderFns
  }
});
```

一直往下走，这是调用栈

![WX20190906-151714@2x](http://www.qinhanwen.xyz/WX20190906-151714@2x.png)

当`el`是`input`的时候

![WX20190906-151816@2x](http://www.qinhanwen.xyz/WX20190906-151816@2x.png)

走进`genData$2`

![WX20190906-152105@2x](http://www.qinhanwen.xyz/WX20190906-152105@2x.png)

```javascript
function genData$2 (el, state) {
  var data = '{';

  // directives first.
  // directives may mutate the el's other properties before they are generated.
  var dirs = genDirectives(el, state);
  if (dirs) { data += dirs + ','; }

  // key
  if (el.key) {
    data += "key:" + (el.key) + ",";
  }
  // ref
  if (el.ref) {
    data += "ref:" + (el.ref) + ",";
  }
  if (el.refInFor) {
    data += "refInFor:true,";
  }
  // pre
  if (el.pre) {
    data += "pre:true,";
  }
  // record original tag name for components using "is" attribute
  if (el.component) {
    data += "tag:\"" + (el.tag) + "\",";
  }
  // module data generation functions
  for (var i = 0; i < state.dataGenFns.length; i++) {
    data += state.dataGenFns[i](el);
  }
  // attributes
  if (el.attrs) {
    data += "attrs:" + (genProps(el.attrs)) + ",";
  }
  // DOM props
  if (el.props) {
    data += "domProps:" + (genProps(el.props)) + ",";
  }
  // event handlers
  if (el.events) {
    data += (genHandlers(el.events, false)) + ",";
  }
  if (el.nativeEvents) {
    data += (genHandlers(el.nativeEvents, true)) + ",";
  }
  // slot target
  // only for non-scoped slots
  if (el.slotTarget && !el.slotScope) {
    data += "slot:" + (el.slotTarget) + ",";
  }
  // scoped slots
  if (el.scopedSlots) {
    data += (genScopedSlots(el, el.scopedSlots, state)) + ",";
  }
  // component v-model
  if (el.model) {
    data += "model:{value:" + (el.model.value) + ",callback:" + (el.model.callback) + ",expression:" + (el.model.expression) + "},";
  }
  // inline-template
  if (el.inlineTemplate) {
    var inlineTemplate = genInlineTemplate(el, state);
    if (inlineTemplate) {
      data += inlineTemplate + ",";
    }
  }
  data = data.replace(/,$/, '') + '}';
  // v-bind dynamic argument wrap
  // v-bind with dynamic arguments must be applied using the same v-bind object
  // merge helper so that class/style/mustUseProp attrs are handled correctly.
  if (el.dynamicAttrs) {
    data = "_b(" + data + ",\"" + (el.tag) + "\"," + (genProps(el.dynamicAttrs)) + ")";
  }
  // v-bind data wrap
  if (el.wrapData) {
    data = el.wrapData(data);
  }
  // v-on data wrap
  if (el.wrapListeners) {
    data = el.wrapListeners(data);
  }
  return data
}
```

走进`  var dirs = genDirectives(el, state);`

```javascript
function genDirectives (el, state) {
  var dirs = el.directives;
  if (!dirs) { return }
  var res = 'directives:[';
  var hasRuntime = false;
  var i, l, dir, needRuntime;
  for (i = 0, l = dirs.length; i < l; i++) {
    dir = dirs[i];
    needRuntime = true;
    var gen = state.directives[dir.name];
    if (gen) {
      // compile-time directive that manipulates AST.
      // returns true if it also needs a runtime counterpart.
      needRuntime = !!gen(el, dir, state.warn);
    }
    if (needRuntime) {
      hasRuntime = true;
      res += "{name:\"" + (dir.name) + "\",rawName:\"" + (dir.rawName) + "\"" + (dir.value ? (",value:(" + (dir.value) + "),expression:" + (JSON.stringify(dir.value))) : '') + (dir.arg ? (",arg:" + (dir.isDynamicArg ? dir.arg : ("\"" + (dir.arg) + "\""))) : '') + (dir.modifiers ? (",modifiers:" + (JSON.stringify(dir.modifiers))) : '') + "},";
    }
  }
  if (hasRuntime) {
    return res.slice(0, -1) + ']'
  }
}
```

在这里遍历了`el.directives`数组，获取指令对应的`gen`，走进`!!gen(el, dir, state.warn);`

```javascript
function model (
  el,
  dir,
  _warn
) {
  warn$1 = _warn;
  var value = dir.value;
  var modifiers = dir.modifiers;
  var tag = el.tag;
  var type = el.attrsMap.type;

  if (process.env.NODE_ENV !== 'production') {
    // inputs with type="file" are read only and setting the input's
    // value will throw an error.
    if (tag === 'input' && type === 'file') {
      warn$1(
        "<" + (el.tag) + " v-model=\"" + value + "\" type=\"file\">:\n" +
        "File inputs are read only. Use a v-on:change listener instead.",
        el.rawAttrsMap['v-model']
      );
    }
  }

  if (el.component) {
    genComponentModel(el, value, modifiers);
    // component v-model doesn't need extra runtime
    return false
  } else if (tag === 'select') {
    genSelect(el, value, modifiers);
  } else if (tag === 'input' && type === 'checkbox') {
    genCheckboxModel(el, value, modifiers);
  } else if (tag === 'input' && type === 'radio') {
    genRadioModel(el, value, modifiers);
  } else if (tag === 'input' || tag === 'textarea') {
    genDefaultModel(el, value, modifiers);
  } else if (!config.isReservedTag(tag)) {
    genComponentModel(el, value, modifiers);
    // component v-model doesn't need extra runtime
    return false
  } else if (process.env.NODE_ENV !== 'production') {
    warn$1(
      "<" + (el.tag) + " v-model=\"" + value + "\">: " +
      "v-model is not supported on this element type. " +
      'If you are working with contenteditable, it\'s recommended to ' +
      'wrap a library dedicated for that purpose inside a custom component.',
      el.rawAttrsMap['v-model']
    );
  }

  // ensure runtime directive metadata
  return true
}
```

走进`genDefaultModel(el, value, modifiers);`

```javascript
function genDefaultModel (
  el,
  value,
  modifiers
) {
  var type = el.attrsMap.type;

  // warn if v-bind:value conflicts with v-model
  // except for inputs with v-bind:type
  if (process.env.NODE_ENV !== 'production') {
    var value$1 = el.attrsMap['v-bind:value'] || el.attrsMap[':value'];
    var typeBinding = el.attrsMap['v-bind:type'] || el.attrsMap[':type'];
    if (value$1 && !typeBinding) {
      var binding = el.attrsMap['v-bind:value'] ? 'v-bind:value' : ':value';
      warn$1(
        binding + "=\"" + value$1 + "\" conflicts with v-model on the same element " +
        'because the latter already expands to a value binding internally',
        el.rawAttrsMap[binding]
      );
    }
  }

  var ref = modifiers || {};
  var lazy = ref.lazy;
  var number = ref.number;
  var trim = ref.trim;
  var needCompositionGuard = !lazy && type !== 'range';
  var event = lazy
    ? 'change'
    : type === 'range'
      ? RANGE_TOKEN
      : 'input';

  var valueExpression = '$event.target.value';
  if (trim) {
    valueExpression = "$event.target.value.trim()";
  }
  if (number) {
    valueExpression = "_n(" + valueExpression + ")";
  }

  var code = genAssignmentCode(value, valueExpression);
  if (needCompositionGuard) {
    code = "if($event.target.composing)return;" + code;
  }

  addProp(el, 'value', ("(" + value + ")"));
  addHandler(el, event, code, null, true);
  if (trim || number) {
    addHandler(el, 'blur', '$forceUpdate()');
  }
}
```

```javascript
  addProp(el, 'value', ("(" + value + ")"));
  addHandler(el, event, code, null, true);
```

其中的这两个方法，给 `el` 添加一个 `prop`，相当于我们在 `input` 上动态绑定了 `value`，又给 `el` 添加了事件处理，相当于在 `input` 上绑定了 `input` 事件

![WX20190906-161250@2x](http://www.qinhanwen.xyz/WX20190906-161250@2x.png)



然后上面`  var dirs = genDirectives(el, state);`返回的`dirs`就是这样的

![WX20190906-161533@2x](http://www.qinhanwen.xyz/WX20190906-161533@2x.png)



最终获得的`render`方法是这样的

```javascript
"with(this){return _c('div',[_c('input',{directives:[{name:"model",rawName:"v-model",value:(message),expression:"message"}],domProps:{"value":(message)},on:{"input":function($event){if($event.target.composing)return;message=$event.target.value}}}),_v("\n    "+_s(message)+"\n  ")])}"
```



往后走到调用`render`方法的地方

![WX20190906-165558@2x](http://www.qinhanwen.xyz/WX20190906-165558@2x.png)

顺便补充一下这个`vm._renderProxy`，其实就是`new Proxy(vm,handlers)`返回的实例

```javascript
  initProxy = function initProxy (vm) {
    if (hasProxy) {
      // determine which proxy handler to use
      var options = vm.$options;
      var handlers = options.render && options.render._withStripped
        ? getHandler
        : hasHandler;
      vm._renderProxy = new Proxy(vm, handlers);
    } else {
      vm._renderProxy = vm;
    }
  };
```

这个是`input`生成的`vnode`

![WX20190906-171617@2x](http://www.qinhanwen.xyz/WX20190906-171617@2x.png)

走进去`update`方法看下`patch`阶段做了什么，进入`createElm`方法里的`createChildren`方法（这里是创建的是`div`标签的`children`）

![WX20190906-174914@2x](http://www.qinhanwen.xyz/WX20190906-174914@2x.png)

![WX20190906-172113@2x](http://www.qinhanwen.xyz/WX20190906-172113@2x.png)

又会走到`createElm`，这里是`input`标签的了

![WX20190906-175418@2x](http://www.qinhanwen.xyz/WX20190906-175418@2x.png)

进入`invokeCreateHooks`方法

```javascript
  function invokeCreateHooks (vnode, insertedVnodeQueue) {
    for (var i$1 = 0; i$1 < cbs.create.length; ++i$1) {
      cbs.create[i$1](emptyNode, vnode);
    }
    i = vnode.data.hook; // Reuse variable
    if (isDef(i)) {
      if (isDef(i.create)) { i.create(emptyNode, vnode); }
      if (isDef(i.insert)) { insertedVnodeQueue.push(vnode); }
    }
  }
```

![WX20190906-180033@2x](http://www.qinhanwen.xyz/WX20190906-180033@2x.png)

`updateDOMListeners`方法

```javascript
function updateDOMListeners (oldVnode, vnode) {
  if (isUndef(oldVnode.data.on) && isUndef(vnode.data.on)) {
    return
  }
  var on = vnode.data.on || {};
  var oldOn = oldVnode.data.on || {};
  target$1 = vnode.elm;
  normalizeEvents(on);
  updateListeners(on, oldOn, add$1, remove$2, createOnceHandler$1, vnode.context);
  target$1 = undefined;
}
```

走进`updateListeners`

```javascript
function updateListeners (
  on,
  oldOn,
  add,
  remove$$1,
  createOnceHandler,
  vm
) {
  var name, def$$1, cur, old, event;
  for (name in on) {
    def$$1 = cur = on[name];
    old = oldOn[name];
    event = normalizeEvent(name);
    if (isUndef(cur)) {
      process.env.NODE_ENV !== 'production' && warn(
        "Invalid handler for event \"" + (event.name) + "\": got " + String(cur),
        vm
      );
    } else if (isUndef(old)) {
      if (isUndef(cur.fns)) {
        cur = on[name] = createFnInvoker(cur, vm);
      }
      if (isTrue(event.once)) {
        cur = on[name] = createOnceHandler(event.name, cur, event.capture);
      }
      add(event.name, cur, event.capture, event.passive, event.params);
    } else if (cur !== old) {
      old.fns = cur;
      on[name] = old;
    }
  }
  for (name in oldOn) {
    if (isUndef(on[name])) {
      event = normalizeEvent(name);
      remove$$1(event.name, oldOn[name], event.capture);
    }
  }
}
```

走进`add(event.name, cur, event.capture, event.passive, event.params);`

```javascript
function add$1 (
  name,
  handler,
  capture,
  passive
) {
  if (useMicrotaskFix) {
    var attachedTimestamp = currentFlushTimestamp;
    var original = handler;
    handler = original._wrapper = function (e) {
      if (
        e.target === e.currentTarget ||
        e.timeStamp >= attachedTimestamp ||
        e.timeStamp <= 0 ||
        e.target.ownerDocument !== document
      ) {
        return original.apply(this, arguments)
      }
    };
  }
  target$1.addEventListener(
    name,
    handler,
    supportsPassive
      ? { capture: capture, passive: passive }
      : capture
  );
}

```

在这里添加绑定事件

![WX20190906-185324@2x](http://www.qinhanwen.xyz/WX20190906-185324@2x.png)



触发一下绑定`input`事件

![WX20190906-185653@2x](http://www.qinhanwen.xyz/WX20190906-185653@2x.png)

![WX20190906-185841@2x](http://www.qinhanwen.xyz/WX20190906-185841@2x.png)

就是调用

```javascript
function($event){if($event.target.composing)return;message=$event.target.value}
```

然后又触发`setter`

![WX20190906-190059@2x](http://www.qinhanwen.xyz/WX20190906-190059@2x.png)

后面就不写了



**总的来说就是**

1）在模板解析，生成`render`方法的时候，为`input`的`el`先添加了`events`属性，之后又判断有`events`属性的话，生成的`data`做字符串拼接(最后拼接到`render`方法里的字符串)

![WX20190907-123004@2x](http://www.qinhanwen.xyz/WX20190907-123004@2x.png)

2）在调用`render`方法生成`vnode`以后，在`update`的`patch`阶段，为DOM元素增加事件





## 数组

为什么不能直接通过索引，或者通过`length`属性修改值，在响应式原理有写。

其实Object.defineProperty本身有一定的监控到数组下标变化的能力，因为性能问题，所以没有使用到。

Vue 的响应式原理中 Object.defineProperty 有什么缺陷？为什么在 Vue3.0 采用了 Proxy，抛弃了 Object.defineProperty？

```
Object.defineProperty只能劫持对象的属性，从而需要对每个对象，每个属性进行遍历，如果，属性值是对象，还需要深度遍历。Proxy可以劫持整个对象，并返回一个新的对象。

Proxy不仅可以代理对象，还可以代理数组。还可以代理动态增加的属性。
```



## nextTick

这个方法接收到参数后，把参数包在匿名函数中`push`进`callbacks`数组，然后把一个方法加到微任务队列或者宏任务队列，加进去的方法就是遍历` callbacks`调用匿名函数。



## Vue 组件中 data 为什么必须是函数

先看错误提示在哪

在子组件做`mergeOptions`的时候

![WX20190908-233124@2x](http://www.qinhanwen.xyz/WX20190908-233124@2x.png)

如果`childVal`不是`function`类型就会提示

![WX20190908-233206@2x](http://www.qinhanwen.xyz/WX20190908-233206@2x.png)

因为：

在组件创建实例做参数合并的时候，看一下`data`，合并参数后返回的是这个方法，其中的`childVal`是`new Vue(options)`，中的`options`中`data`的引用地址

![WX20190908-234713@2x](http://www.qinhanwen.xyz/WX20190908-234713@2x.png)

然后在`initData`的时候，获取`data`

![WX20190908-234921@2x](http://www.qinhanwen.xyz/WX20190908-234921@2x.png)

然后我去修改`data`的值

![WX20190908-235121@2x](http://www.qinhanwen.xyz/WX20190908-235121@2x.png)



然后看下`new Vue(options)`

![WX20190908-235121@2x](http://www.qinhanwen.xyz/WX20190908-235121@2x.png)



也就是说，在组件创建实例的时候，`data`如果是对象的话，每个组件的实例修改`data`的值，会有影响，所以使用方法可以让每个实例的`data`独立



## 自定义事件通知

emm…，意思其实就是子组件通知父组件啦

main.js

```javascript
import Vue from 'vue'
import App from './App';
Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  methods:{
    childEmitHandler() {
      console.log('Child select!')
    },
    childEmitHandler1() {
      console.log('Child select1!')
    }
  },
  components:{
    App
  },
  template: `<div> 
  <App @childEmitHandler="childEmitHandler();childEmitHandler1()" />
  </div>`
})
```

App.vue

```javascript
<template>
  <button @click="clickHandler($event)">click me</button>
</template>

<script>
export default {
  name: "App",
  props: {
    firstName: String
  },
  methods: {
    clickHandler(e) {
      console.log('child clicked');
      this.$emit("childEmitHandler");
    }
  }
};
</script>
```

写个大概吧。

1）父组件初始化的时候，`initState`的时候，`childEmitHandler`强绑定`this`后挂载到`vm`实例上，在`update`阶段的时候为元素添加事件

![WX20190909-082624@2x](http://www.qinhanwen.xyz/WX20190909-082624@2x.png)

2）子组件初始化的时候，`initEvents`的时候，添加自定义事件进`vm._events`对象

![WX20190909-084215@2x](http://www.qinhanwen.xyz/WX20190909-084215@2x.png)

![WX20190909-002952@2x](http://www.qinhanwen.xyz/WX20190909-002952@2x.png)

然后`initState`的时候`clickHandler`绑定`this`后挂载在实例上，在`update`阶段的时候为元素添加事件。

3）触发子组件点击事件，调用`$emit`方法通知调用自定义事件，遍历`vm._events[event]`，调用每一项的方法调用

![WX20190909-084428@2x](http://www.qinhanwen.xyz/WX20190909-084428@2x.png)

![WX20190909-084607@2x](http://www.qinhanwen.xyz/WX20190909-084607@2x.png)

并且得知，事件是可以绑定多个方法的



**总结结**

- 子组件创建实例的时候，传入了options里包含父vnode，父vnode里包含了自定义事件。子组件init进入initEvent的时候挂载到了子组件实例的_events上
- $emit方法调用的时候，从vm._events上取得回调，并调用

​	



## vue中 `key` 值的作用

写两个例子，分别`带key`和`不带key`的情况

**不带key**

```javascript
import Vue from 'vue'
import App from './App';
Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data:{
    userList:[{
      name:"qinhanwen",
      id:1,
    },{
      name:"zenghua",
      id:2,
    }]
  },
  methods:{
    reverseUserList() {
      this.userList.reverse();
    },
  },
  template: `<ul @click="reverseUserList()"> 
    <li v-for="item in userList">{{item.name}}</li>
  </ul>`
})
```

1）触发点击事件

2）更新`vnode`

3）触发`DOM`更新

当没有key的时候（`key`是`undefined`），`sameVnode(oldStartVnode, newStartVnode)`方法的返回值为`true`，选择复用之前的`DOM`结构，也就是`DOM`元素不变，去改变它的`children`（当前这个元素是第一个`li`元素）

![WX20190909-101749@2x](http://www.qinhanwen.xyz/WX20190909-101749@2x.png)

当前元素的`children`的值更新了

![WX20190909-102602@2x](http://www.qinhanwen.xyz/WX20190909-102602@2x.png)

之后再改变另外一个`li`元素的`children`

![WX20190909-103118@2x](http://www.qinhanwen.xyz/WX20190909-103118@2x.png)



**带key**

```javascript
import Vue from 'vue'
import App from './App';
Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data:{
    userList:[{
      name:"qinhanwen",
      id:1,
    },{
      name:"zenghua",
      id:2,
    }]
  },
  methods:{
    reverseUserList() {
      this.userList.reverse();
    },
  },
  template: `<ul @click="reverseUserList()"> 
    <li v-for="item in userList" key="item.id">{{item.name}}</li>
  </ul>`
})
```

1）触发点击事件

2）更新`vnode`

3）触发`DOM`更新

这时候因为有`key`，所以`sameVnode(oldStartVnode, newStartVnode)`方法返回值为`false`

![WX20190909-103558@2x](http://www.qinhanwen.xyz/WX20190909-103558@2x.png)

所以走进了另外一个处理分支，`sameVnode(oldStartVnode, newEndVnode)`返回为`true`，把现有元素通过`insertBefore`移动到前面

![WX20190909-104124@2x](http://www.qinhanwen.xyz/WX20190909-104124@2x.png)



**总结：**

`带key`：根据key的变化，来重新排列元素顺序，更新`DOM`结构。

`不带key`：没有 `key` 的值会采取一种“就地更新策略”，更新现有的`DOM`节点和它的`children`，（有可能会做很多创建、删除节点之类的，相对来说效率更低一点）。



## 为什么key不建议使用index

写两个例子

**`key`为`index`**

main.js

```javascript
import Vue from 'vue'
import App from './App';
Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data:{
    userList:[{
      name:"qinhanwen",
      id:1,
    },{
      name:"zenghua",
      id:2,
    },{
      name:"bao",
      id:3,
    },{
      name:"guo",
      id:4,
    }]
  },
  methods:{
    reverseUserList() {
      this.userList.splice(1,1);
    },
  },
  template: `<ul @click="reverseUserList()"> 
    <li v-for="(item,index) in userList" :key="index">{{item.name}}{{index}}</li>
  </ul>`
})

```

1）触发点击事件

2）更新`vnode`

3）

- 第一个`li`无变化，没有重新渲染

- 第二个`li`被重新渲染

  ![WX20190909-125426@2x](http://www.qinhanwen.xyz/WX20190909-125426@2x.png)

- 第三个`li`被重新渲染

  ![WX20190909-125530@2x](http://www.qinhanwen.xyz/WX20190909-125530@2x.png)

- 第四个`li`被移除

  

**`key`为`id`**

main.js

```javascript
import Vue from 'vue'
import App from './App';
Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data:{
    userList:[{
      name:"qinhanwen",
      id:1,
    },{
      name:"zenghua",
      id:2,
    },{
      name:"bao",
      id:3,
    },{
      name:"guo",
      id:4,
    }]
  },
  methods:{
    reverseUserList() {
      this.userList.splice(1,1);
    },
  },
  template: `<ul @click="reverseUserList()"> 
    <li v-for="(item,index) in userList" :key="item.id">{{item.name}}{{index}}</li>
  </ul>`
})

```

1）触发点击事件

2）更新`vnode`

3）

- 第一个`li`无变化，没有重新渲染

- 第四个`li`的更新内容

  ![WX20190909-130108@2x](http://www.qinhanwen.xyz/WX20190909-130108@2x.png)

- 第三个`li`更新内容

  ![WX20190909-130256@2x](http://www.qinhanwen.xyz/WX20190909-130256@2x.png)

- 移除第二个`li`



**总结：**

`key`为`index`：当移除数组的某一项，某项开始到之后的所有节点都会被重新渲染

`key`为`id`：尽可能复用相同的`id`的节点（因为插值表达式写了`index`，才做了渲染），只移除需要移除的那一项。



## v-show和v-if指令的共同点和不同点

`v-show`指令是通过修改元素的`display`CSS属性让其显示或者隐藏

`v-if`指令是直接销毁和重建DOM达到让元素显示和隐藏的效果（注意：v-if 可以实现组件的重新渲染）



## \$route与\$router

`Vue.use(VueRouter)`后，在执行`install`方法的时候，除了安装了两个生命周期函数和全局组件，还为`vm.$router`和`vm.$route`添加了`get`

![WX20190909-221338@2x](http://www.qinhanwen.xyz/WX20190909-221338@2x.png)

在`new VueRouter({routes})`的时候，创建路由注册表，下面是创建的实例

![WX20190909-222713@2x](http://www.qinhanwen.xyz/WX20190909-222713@2x.png)

它的`__proto__`属性：

![WX20190909-225609@2x](http://www.qinhanwen.xyz/WX20190909-225609@2x.png)



在`init`初始化，执行` callHook(vm, 'beforeCreate');`，也就是调用`install`的时候添加的`beforeCreate`生命周期钩子函数，在这里做路由初始化

`$router`就是`VueRouter`的实例

![WX20190909-224918@2x](http://www.qinhanwen.xyz/WX20190909-224918@2x.png)

监听路由变化

![WX20190909-224041@2x](http://www.qinhanwen.xyz/WX20190909-224041@2x.png)

然后`$route`在这里赋值

![WX20190909-225316@2x](http://www.qinhanwen.xyz/WX20190909-225316@2x.png)

![WX20190909-225405@2x](http://www.qinhanwen.xyz/WX20190909-225405@2x.png)



**总结：**

`$router`：路由器对象，提供了系列跳转相关的方法。

`$route`：相当于当前正在跳转的路由对象。





## 插槽

main.js

```javascript
import Vue from 'vue'
import App from './App';
Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data: {
    name: 'qinhanwen'
  },
  components: {
    App
  },
  methods: {

  },
  template: `<App><p slot="content">{{name}}</p></App>`
})
```

app.vue

```vue
<template>
  <div>
    <slot name="content"></slot>
  </div>
</template>

<script>
export default {
  name: "App",
  props: {
    firstName: String
  },
  methods: {}
};
</script>
```



在父组件模板解析生成`render`方法

```javascript
with(this){return _c('App',[_c('p',{attrs:{"slot":"content"},slot:"content"},[_v(_s(name))])])}
```

因为`App`是个组件，所以开始创建组件，进入组件`init`，在`initInternalComponent`的时候，拿到`App`组件`vnode`里的`子vnode`，就是那个`p`标签的`vnode`

![WX20190910-210255@2x](http://www.qinhanwen.xyz/WX20190910-210255@2x.png)



在`initRender`的时候为子组件`vm`添加`$slots`属性

![WX20190910-204120@2x](http://www.qinhanwen.xyz/WX20190910-204120@2x.png)



然后进入子组件挂载阶段

```javascript
 child.$mount(hydrating ? vnode.elm : undefined, hydrating);
```

进入`_render`方法，`vm.$scopedSlots`等于这么个方法

```javascript
  if (_parentVnode) {
      vm.$scopedSlots = normalizeScopedSlots(
        _parentVnode.data.scopedSlots,
        vm.$slots,
        vm.$scopedSlots
      );
    }
```

![WX20190910-205350@2x](http://www.qinhanwen.xyz/WX20190910-205350@2x.png)

生成`vnode`

`   vnode = render.call(vm._renderProxy, vm.$createElement);`

进入`render`方法

```javascript
var render = function() {
  var _vm = this
  var _h = _vm.$createElement
  var _c = _vm._self._c || _h
  return _c("div", [_vm._t("content")], 2)
}
```

看一下`_vm._t`方法

```javascript
function renderSlot (
  name,
  fallback,
  props,
  bindObject
) {
  var scopedSlotFn = this.$scopedSlots[name];
  var nodes;
  if (scopedSlotFn) { // scoped slot
    props = props || {};
    if (bindObject) {
      if (process.env.NODE_ENV !== 'production' && !isObject(bindObject)) {
        warn(
          'slot v-bind without argument expects an Object',
          this
        );
      }
      props = extend(extend({}, bindObject), props);
    }
    nodes = scopedSlotFn(props) || fallback;
  } else {
    nodes = this.$slots[name] || fallback;
  }

  var target = props && props.slot;
  if (target) {
    return this.$createElement('template', { slot: target }, nodes)
  } else {
    return nodes
  }
}
```

返回的`nodes`就是这个，就是那个`p`标签的`vnode`

![WX20190910-212407@2x](http://www.qinhanwen.xyz/WX20190910-212407@2x.png)

然后调用`_c`，就是`_createElement`方法，最后生成的`vnode`大概是这样的

```json
{
		tag:"div",
		children:[
			{
				tag:"p",
				children:[
					{
						tag:undefined,
						text:"qinhanwen"
						...
					}
				]
				...
			}
		]
		...
}
```

后面就不写了



**总结：**

也就是一开始父组件里使用组件的地方，组件标签里的模板，变成了`vnode`被存了起来，在子组件`_render`的时候被拿出来插到插槽的位置。



## 计算属性与监听属性

**计算属性：**

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
  methods:{
    changeFirstName(){
      this.firstName = 'q';
    }
  },
  computed: {
    totalName: function () {
      return this.firstName + this.lastName
    }
  },
  template: `<div @click="changeFirstName()">
  {{totalName}}
  </div>`
})
```



- `initState`进入到`initComputed`的时候，遍历计算属性
  - 添加计算属性`watcher`，`push`到`vm._watchers`，`watcher`的`getter`方法就是计算属性的函数
  - 为`vm`上的`计算属性键名`，添加属性描述符（`getter`、`setter`、`configurable`、`enumerable`)

- 调用`vm._render()`，生成`vnode`的过程中

  - 取`totalName`的值，触发`getter`（这里就是触发`totalName`计算属性的方法调用，其中又触发`firstName`和`lastName`的`getter`，就把当前这个`totalName`的`watcher`收集了，另外还收集了渲染`watcher`）

  - 生成`vnode`

- 触发点击事件，修改`firstName`的值，触发派发更新的过程
  - 因为计算属性的`watcher`是`lazy`，所以没能加到`queue`队列里
  - 将渲染`watcher`加入到`queue`队列
  - 之后通过`nextTick`添加异步任务，遍历`queue`执行里面`watcher`的`run`方法，也就是重新执行`vm._update(vm._render(), hydrating);`，生成`vnode`的时候，更新`totalName`的数据
  - 重新渲染



**监听属性：**

```javascript
import Vue from 'vue'

Vue.config.productionTip = false

/* eslint-disable no-new */

new Vue({
  el: '#app',
  data: function () {
    return {
      firstName: 'qin',
      totalName: 'total'
    }
  },
  methods:{
    changeFirstName(){
      this.firstName = 'q';
    }
  },
  watch: {
    firstName: function (newVal,oldVal) {
       this.totalName =  newVal + oldVal;
    }
  },
  template: `<div @click="changeFirstName()">
  {{totalName}}
  </div>`
})
```



- `initState`进入到`initWatch`的时候，遍历监听属性

  - 添加`user watcher`，`push`到`vm._watchers`，`user watcher`的`cb`属性是`watch`属性的回调函数，`getter`是这个方法

    ```javascript
    function (obj) {
        for (var i = 0; i < segments.length; i++) {
          if (!obj) { return }
          obj = obj[segments[i]];
        }
        return obj
      }
    ```

  - 创建`user watcher`实例的最后，调用`this.get() -> this.getter.call(vm, vm)`，`getter`就是上面这个方法，`obj[segments[i]]`的时候触发`firstName`的`getter`，把这个`user watcher`做依赖收集，添加到了`firstName`的`dep`里

  - 判断是否有`immediate`属性，有就立即执行`cb`

- 触发点击事件，改变`firstName`的值，触发派发更新

  - 调用刚才依赖收集到的`user watcher`的`run`方法，先获取了`val`，也获取了`oldVal`，然后调用`user watcher`的`cb`函数

    ```javascript
    this.cb.call(this.vm, value, oldValue);
    ```

    

**所以监听属性与计算属性的区别：**

**计算属性：**在`render`生成`vnode`的时候做依赖收集，也是在依赖是属性发生变化的时候，才重新生成`vnode`，重新渲染。

**监听属性：**在创建`user watcher`实例的时候，就做了依赖收集，在`watch`的值改变的时候，会调用`watch`属性的回调（监听属性比较适合做一些复杂的逻辑）。



## v-if与v-for不建议使用在同一个模板

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
      arr:[1,2,3]
    }
  },

  template: `<div>
    <p v-if="!arr.length" v-for="item in arr">{{item}}</p>
  </div>`
})
```



生成的`render`函数

```javascript
(function anonymous(
) {
with(this){return _c('div',_l((arr),function(item){return (!arr.length)?_c('p',[_v(_s(item))]):_e()}),0)}
})
```

优先调用

```javascript
_l((arr),function(item){return (!arr.length)?_c('p',[_v(_s(item))]):_e()})
```

`_l`也就是下面这个方法

```javascript
function renderList (
  val,
  render
) {
  var ret, i, l, keys, key;
  if (Array.isArray(val) || typeof val === 'string') {
    ret = new Array(val.length);
    for (i = 0, l = val.length; i < l; i++) {
      ret[i] = render(val[i], i);
    }
  } else if (typeof val === 'number') {
    ret = new Array(val);
    for (i = 0; i < val; i++) {
      ret[i] = render(i + 1, i);
    }
  } else if (isObject(val)) {
    if (hasSymbol && val[Symbol.iterator]) {
      ret = [];
      var iterator = val[Symbol.iterator]();
      var result = iterator.next();
      while (!result.done) {
        ret.push(render(result.value, ret.length));
        result = iterator.next();
      }
    } else {
      keys = Object.keys(val);
      ret = new Array(keys.length);
      for (i = 0, l = keys.length; i < l; i++) {
        key = keys[i];
        ret[i] = render(val[key], key, i);
      }
    }
  }
  if (!isDef(ret)) {
    ret = [];
  }
  (ret)._isVList = true;
  return ret
}
```

如果传入的`val`是数组，就会遍历数组，调用`ret[i] = render(val[i], i);`，而这里的遍历操作是无意义的



## keep-alive

main.js

```javascript
import Vue from 'vue'

var child1 = {
  template: '<div><button @click="add">add</button><p>{{num}}</p></div>',
  data () {
    return {
      num: 1
    }
  },
  activated () {
    console.log('active')
  },
  created () {
    console.log('created')
  },
  mounted () {
    console.log('mounted')
  },
  methods: {
    add () {
      this.num++
    }
  }
}
var child2 = {
  template: '<div>child2</div>'
}

/* eslint-disable no-new */
new Vue({
  el: '#app',
  components: {
    child1, child2
  },
  data () {
    return {
      chooseTabs: 'child1'
    }
  },
  methods: {
    changeTabs (tab) {
      this.chooseTabs = tab
    }
  },
  template: `
  <div>
    <button @click="changeTabs('child1')">child1</button>
    <button @click="changeTabs('child2')">child2</button>
    <keep-alive>
        <component :is="chooseTabs">
        </component>
    </keep-alive>
  </div>
  `
})

```



首先声明了`KeepAlive`对象，之后作为`builtInComponents`对象的元素

```javascript
var KeepAlive = {
  name: 'keep-alive',
  abstract: true,

  props: {
    include: patternTypes,
    exclude: patternTypes,
    max: [String, Number]
  },

  created: function created () {
    this.cache = Object.create(null);
    this.keys = [];
  },

  destroyed: function destroyed () {
    for (var key in this.cache) {
      pruneCacheEntry(this.cache, key, this.keys);
    }
  },

  mounted: function mounted () {
    var this$1 = this;

    this.$watch('include', function (val) {
      pruneCache(this$1, function (name) { return matches(val, name); });
    });
    this.$watch('exclude', function (val) {
      pruneCache(this$1, function (name) { return !matches(val, name); });
    });
  },

  render: function render () {
    var slot = this.$slots.default;
    var vnode = getFirstComponentChild(slot);
    var componentOptions = vnode && vnode.componentOptions;
    if (componentOptions) {
      // check pattern
      var name = getComponentName(componentOptions);
      var ref = this;
      var include = ref.include;
      var exclude = ref.exclude;
      if (
        // not included
        (include && (!name || !matches(include, name))) ||
        // excluded
        (exclude && name && matches(exclude, name))
      ) {
        return vnode
      }

      var ref$1 = this;
      var cache = ref$1.cache;
      var keys = ref$1.keys;
      var key = vnode.key == null
        // same constructor may get registered as different local components
        // so cid alone is not enough (#3269)
        ? componentOptions.Ctor.cid + (componentOptions.tag ? ("::" + (componentOptions.tag)) : '')
        : vnode.key;
      if (cache[key]) {
        vnode.componentInstance = cache[key].componentInstance;
        // make current key freshest
        remove(keys, key);
        keys.push(key);
      } else {
        cache[key] = vnode;
        keys.push(key);
        // prune oldest entry
        if (this.max && keys.length > parseInt(this.max)) {
          pruneCacheEntry(cache, keys[0], keys, this._vnode);
        }
      }

      vnode.data.keepAlive = true;
    }
    return vnode || (slot && slot[0])
  }
};
var builtInComponents = {
  KeepAlive: KeepAlive
};
```

在`initGlobalAPI`方法调用中`extend(Vue.options.components, builtInComponents);`，把`keepAlive`组件添加到`Vue.options.components`

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

**首次渲染：**

根据`ast`生成的`render`函数

```javascript
(function anonymous(
) {
with(this){return _c('div',[_c('button',{on:{"click":function($event){return changeTabs('child1')}}},[_v("child1")]),_v(" "),_c('button',{on:{"click":function($event){return changeTabs('child2')}}},[_v("child2")]),_v(" "),_c('keep-alive',[_c(chooseTabs,{tag:"component"})],1)],1)}
})
```

在`patch`阶段

生成`keep-alive`组件的`vnode`的时候，`render`函数就是`keep-alive`组件的`render`函数

![微信截图_20190927223928](http://www.qinhanwen.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20190927223928.png)

```javascript
  render: function render () {
    var slot = this.$slots.default;
    var vnode = getFirstComponentChild(slot);
    var componentOptions = vnode && vnode.componentOptions;
    if (componentOptions) {
      // check pattern
      var name = getComponentName(componentOptions);
      var ref = this;
      var include = ref.include;
      var exclude = ref.exclude;
      if (
        // not included
        (include && (!name || !matches(include, name))) ||
        // excluded
        (exclude && name && matches(exclude, name))
      ) {
        return vnode
      }

      var ref$1 = this;
      var cache = ref$1.cache;
      var keys = ref$1.keys;
      var key = vnode.key == null
        // same constructor may get registered as different local components
        // so cid alone is not enough (#3269)
        ? componentOptions.Ctor.cid + (componentOptions.tag ? ("::" + (componentOptions.tag)) : '')
        : vnode.key;
      if (cache[key]) {
        vnode.componentInstance = cache[key].componentInstance;
        // make current key freshest
        remove(keys, key);
        keys.push(key);
      } else {
        cache[key] = vnode;
        keys.push(key);
        // prune oldest entry
        if (this.max && keys.length > parseInt(this.max)) {
          pruneCacheEntry(cache, keys[0], keys, this._vnode);
        }
      }

      vnode.data.keepAlive = true;
    }
    return vnode || (slot && slot[0])
  }
```

在这里对组件`vnode`进行了缓存，之后有用到的时候会取出来

**缓存渲染：**

点击添加按钮，并且切换视图两次，触发派发更新

![微信截图_20190928191902](http://www.qinhanwen.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20190928191902.png)

走进更新视图的`patch`，会走进`prepatch`方法

```javascript
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
```

然后走进`updateChildComponent`

```javascript
function updateChildComponent (
  vm,
  propsData,
  listeners,
  parentVnode,
  renderChildren
) {
  if (process.env.NODE_ENV !== 'production') {
    isUpdatingChildComponent = true;
  }

  // determine whether component has slot children
  // we need to do this before overwriting $options._renderChildren.

  // check if there are dynamic scopedSlots (hand-written or compiled but with
  // dynamic slot names). Static scoped slots compiled from template has the
  // "$stable" marker.
  var newScopedSlots = parentVnode.data.scopedSlots;
  var oldScopedSlots = vm.$scopedSlots;
  var hasDynamicScopedSlot = !!(
    (newScopedSlots && !newScopedSlots.$stable) ||
    (oldScopedSlots !== emptyObject && !oldScopedSlots.$stable) ||
    (newScopedSlots && vm.$scopedSlots.$key !== newScopedSlots.$key)
  );

  // Any static slot children from the parent may have changed during parent's
  // update. Dynamic scoped slots may also have changed. In such cases, a forced
  // update is necessary to ensure correctness.
  var needsForceUpdate = !!(
    renderChildren ||               // has new static slots
    vm.$options._renderChildren ||  // has old static slots
    hasDynamicScopedSlot
  );

  vm.$options._parentVnode = parentVnode;
  vm.$vnode = parentVnode; // update vm's placeholder node without re-render

  if (vm._vnode) { // update child tree's parent
    vm._vnode.parent = parentVnode;
  }
  vm.$options._renderChildren = renderChildren;

  // update $attrs and $listeners hash
  // these are also reactive so they may trigger child update if the child
  // used them during render
  vm.$attrs = parentVnode.data.attrs || emptyObject;
  vm.$listeners = listeners || emptyObject;

  // update props
  if (propsData && vm.$options.props) {
    toggleObserving(false);
    var props = vm._props;
    var propKeys = vm.$options._propKeys || [];
    for (var i = 0; i < propKeys.length; i++) {
      var key = propKeys[i];
      var propOptions = vm.$options.props; // wtf flow?
      props[key] = validateProp(key, propOptions, propsData, vm);
    }
    toggleObserving(true);
    // keep a copy of raw propsData
    vm.$options.propsData = propsData;
  }

  // update listeners
  listeners = listeners || emptyObject;
  var oldListeners = vm.$options._parentListeners;
  vm.$options._parentListeners = listeners;
  updateComponentListeners(vm, listeners, oldListeners);

  // resolve slots + force update if has children
  if (needsForceUpdate) {
    vm.$slots = resolveSlots(renderChildren, parentVnode.context);
    vm.$forceUpdate();
  }
```

调用`vm.$forceUpdate`，为`queue`新增了一个`wathcer`

![微信截图_20190928101915](http://www.qinhanwen.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20190928101915.png)

之后调用新增这个`wathcer`的`run`方法(最后就是`  vm._update(vm._render(), hydrating);`)，在生成`vnode`的时候，从`cache`中取出缓存的`vnode`

![微信截图_20190928185150](http://www.qinhanwen.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20190928185150.png)

之后调用`update`更新视图，这次走的不是新增的流程，而是比较的

这次在创建子组件的时候

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

进入组建`init`方法的时候，逻辑是走的这里

进入`prepatch`方法,在`updateChildComponent`做了一些更新操作

```javascript
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
```

回到`createComponent`，在`initComponent`方法里，把组建实例的`$el`赋值给了`vnode.elm`

```javascript
 vnode.elm = vnode.componentInstance.$el;
```

之后调用

```javascript
insert(parentElm, vnode.elm, refElm);
```

把缓存的`dom`结构插入到了插槽中

![微信截图_20190928192812](http://www.qinhanwen.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20190928192812.png)

之后再移除旧的节点

![微信截图_20190928193001](http://www.qinhanwen.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20190928193001.png)



看一下`componentInstance`在什么时候添加的

![微信截图_20190929010011](http://www.qinhanwen.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20190929010011.png)

看一下`createComponentInstanceForVnode`

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



需要注意的就是组建在**重新渲染**的时候不会调用`created`和`mounted`钩子函数，会调用`activated `钩子函数

而**首次渲染**也会调用`activated `钩子函数





## Vue.directive

改一下例子

```javascript
import Vue from 'vue'

Vue.directive('qhw', function (el, binding, vnode) {
  console.log(el, binding, vnode)
  el.style = 'color:' + binding.value
})
/* eslint-disable no-new */
new Vue({
  el: '#app',
  data: {
    color: 'red'
  },
  template: `
  <div v-qhw="color">
    123
  </div>
  `
})
```





![微信截图_20190929235816](http://www.qinhanwen.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20190929235816.png)

看一下`updateDirectives`方法，其实就是遍历`directives`，调用`_update`

```javascript
function updateDirectives (oldVnode, vnode) {
  if (oldVnode.data.directives || vnode.data.directives) {
    _update(oldVnode, vnode);
  }
}
```

看一下`_update`方法

```javascript
function _update (oldVnode, vnode) {
  var isCreate = oldVnode === emptyNode;
  var isDestroy = vnode === emptyNode;
  var oldDirs = normalizeDirectives$1(oldVnode.data.directives, oldVnode.context);
  var newDirs = normalizeDirectives$1(vnode.data.directives, vnode.context);

  var dirsWithInsert = [];
  var dirsWithPostpatch = [];

  var key, oldDir, dir;
  for (key in newDirs) {
    oldDir = oldDirs[key];
    dir = newDirs[key];
    if (!oldDir) {
      // new directive, bind
      callHook$1(dir, 'bind', vnode, oldVnode);
      if (dir.def && dir.def.inserted) {
        dirsWithInsert.push(dir);
      }
    } else {
      // existing directive, update
      dir.oldValue = oldDir.value;
      dir.oldArg = oldDir.arg;
      callHook$1(dir, 'update', vnode, oldVnode);
      if (dir.def && dir.def.componentUpdated) {
        dirsWithPostpatch.push(dir);
      }
    }
  }

  if (dirsWithInsert.length) {
    var callInsert = function () {
      for (var i = 0; i < dirsWithInsert.length; i++) {
        callHook$1(dirsWithInsert[i], 'inserted', vnode, oldVnode);
      }
    };
    if (isCreate) {
      mergeVNodeHook(vnode, 'insert', callInsert);
    } else {
      callInsert();
    }
  }

  if (dirsWithPostpatch.length) {
    mergeVNodeHook(vnode, 'postpatch', function () {
      for (var i = 0; i < dirsWithPostpatch.length; i++) {
        callHook$1(dirsWithPostpatch[i], 'componentUpdated', vnode, oldVnode);
      }
    });
  }

  if (!isCreate) {
    for (key in oldDirs) {
      if (!newDirs[key]) {
        // no longer present, unbind
        callHook$1(oldDirs[key], 'unbind', oldVnode, oldVnode, isDestroy);
      }
    }
  }
}
```

在这里调用`bind`钩子，其实就是调用`directive`传入的函数

![微信截图_20190930000934](http://www.qinhanwen.xyz/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20190930000934.png)





## ref在什么时候添加到$refs上的

在`patch`阶段

![WX20191007-130322@2x](http://www.qinhanwen.xyz/WX20191007-130322@2x.png)



![WX20191007-130349@2x](http://www.qinhanwen.xyz/WX20191007-130349@2x.png)



![WX20191007-130403@2x](http://www.qinhanwen.xyz/WX20191007-130403@2x.png)



![WX20191007-130428@2x](http://www.qinhanwen.xyz/WX20191007-130428@2x.png)



```javascript
  create: function create (_, vnode) {
    registerRef(vnode);
  },
```

进入`registerRef`

```javascript
function registerRef (vnode, isRemoval) {
  var key = vnode.data.ref;
  if (!isDef(key)) { return }

  var vm = vnode.context;
  var ref = vnode.componentInstance || vnode.elm;
  var refs = vm.$refs;
  if (isRemoval) {
    if (Array.isArray(refs[key])) {
      remove(refs[key], ref);
    } else if (refs[key] === ref) {
      refs[key] = undefined;
    }
  } else {
    if (vnode.data.refInFor) {
      if (!Array.isArray(refs[key])) {
        refs[key] = [ref];
      } else if (refs[key].indexOf(ref) < 0) {
        // $flow-disable-line
        refs[key].push(ref);
      }
    } else {
      refs[key] = ref;
    }
  }
}
```

这个方法获取了`vm.$refs`，并且传值给`refs`，之后为`refs`添加属性



## 组件style标签中scoped的作用

在创建占位符的时候，调用setScope的时候，为标签设置了一个属性

```javascript
  function setScope (vnode) {
    var i;
    if (isDef(i = vnode.fnScopeId)) {
      nodeOps.setStyleScope(vnode.elm, i);
    } else {
      var ancestor = vnode;
      while (ancestor) {
        if (isDef(i = ancestor.context) && isDef(i = i.$options._scopeId)) {
          nodeOps.setStyleScope(vnode.elm, i);
        }
        ancestor = ancestor.parent;
      }
    }
    // for slot content they should also get the scopeId from the host instance.
    if (isDef(i = activeInstance) &&
      i !== vnode.context &&
      i !== vnode.fnContext &&
      isDef(i = i.$options._scopeId)
    ) {
      nodeOps.setStyleScope(vnode.elm, i);
    }
  }
```

![WX20191024-000600@2x](http://www.qinhanwen.xyz/WX20191024-000600@2x.png)

![WX20191024-000626@2x](http://www.qinhanwen.xyz/WX20191024-000626@2x.png)

![WX20191024-000744@2x](http://www.qinhanwen.xyz/WX20191024-000744@2x.png)





