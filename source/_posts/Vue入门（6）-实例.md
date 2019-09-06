---
title: Vue入门（6）- 实例
date: 2019-08-27 22:44:43
tags: 
- Vue入门
categories: 
- Vue入门
---

[API文档](https://cn.vuejs.org/v2/api/#silent)

## 实例属性

| 实例属性  | 类型                              | 描述                                                         |
| --------- | --------------------------------- | ------------------------------------------------------------ |
| $data     | Object                            | Vue 实例代理了对其 data 对象属性的访问                       |
| $el       | Element                           | Vue 实例使用的根 DOM 元素。                                  |
| $parent   | Vue instance                      | 父实例，如果当前实例有的话。                                 |
| $root     | Vue instance                      | 当前组件树的根 Vue 实例。如果当前实例没有父实例，此实例将会是其自己。 |
| $children | Array<Vue instance>               | 当前实例的直接子组件。**需要注意 $children 并不保证顺序，也不是响应式的。**如果你发现自己正在尝试使用 `$children`来进行数据绑定，考虑使用一个数组配合 `v-for` 来生成子组件，并且使用 Array 作为真正的来源。 |
| $slot     | { [name: string]: ?Array<VNode> } | 用来访问被[插槽分发](https://cn.vuejs.org/v2/guide/components.html#通过插槽分发内容)的内容。每个[具名插槽](https://cn.vuejs.org/v2/guide/components-slots.html#具名插槽) 有其相应的属性 (例如：`v-slot:foo` 中的内容将会在 `vm.$slots.foo` 中被找到)。`default` 属性包括了所有没有被包含在具名插槽中的节点，或 `v-slot:default` 的内容。 |
| $refs     | Object                            | 一个对象，持有注册过 [`ref` 特性](https://cn.vuejs.org/v2/api/#ref) 的所有 DOM 元素和组件实例。 |

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>learning-vue</title>
</head>
<script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>

<body>
    <div id="learning-vue">
        <div>
            <input-box ref="user-input"></input-box>
        </div>
    </div>
</body>
<script>
    Vue.component('input-box', {
        template: `<input />`
    })
    const app = new Vue({
        el: "#learning-vue",
        data: {
            name: 'qinhanwen'
        },
    })
    console.log(app.$refs['user-input'].$parent);
    console.log(app.$refs);
    console.log(app.$root);
    console.log(app.$data);
    console.log(app.$el);
</script>

</html>
```

打印如图：

![WX20190827-232916@2x](http://www.qinhanwen.xyz/WX20190827-232916@2x.png)



## 实例方法

**vm.$watch(expOrFn,callBack,options)**

- `{string | Function} expOrFn`
- `{Function | Object} callback`
-  `{Object} [options]`
  - `{boolean} deep`发现对象内部值的变化，可以在选项参数中指定 `deep: true` 。注意监听数组的变动不需要这么做。
  - `{boolean} immediate`在选项参数中指定 `immediate: true` 将立即以表达式的当前值触发回调（就是立即调用回调时候使用初始值）。

```javascript
var unwatch = vm.$watch('message', function (newVal, oldVal) {
  console.log(newVal);
})
unwatch();//取消侦听
```

示例：

```javascript
const app = new Vue({
  el: "#learning-vue",
  data: {
    name: 'qinhanwen'
  },
})
const unwatch = app.$watch('name',function(newVal){
  console.log(newVal);//zeng
})
app.name = "zeng";
```



**vm.$set(target,propertyName/index,value)**

- `{Object | Array} target`

- `{string | number} propertyName/index`

- `{any} value`

  

**vm.$delete(target,propertyName/index)**

- `{Object | Array} target`
- `{string | number} propertyName/index`



## 实例事件

**vm.$on('event',callback)**

**vm.emit('event',[…args])**

**vm.$once('event',callback)**

监听一个自定义事件，但是只触发一次，在第一次触发之后移除监听器。

**vm.$off('event',callback)**

- 如果没有提供参数，则移除所有的事件监听器；
- 如果只提供了事件，则移除该事件所有的监听器；
- 如果同时提供了事件与回调，则只移除这个回调的监听器。



## 实例方法/生命周期

 **vm.$mount([elementOrSelector])**

如果 Vue 实例在实例化时没有收到 el 选项，则它处于“未挂载”状态，没有关联的 DOM 元素。可以使用 `vm.$mount()` 手动地挂载一个未挂载的实例。

```javascript
var MyComponent = Vue.extend({
	template: '<div>$mount!</div>'
})

// 创建并挂载到 #learning-vue (会替换 #learning-vue)
new MyComponent().$mount('#learning-vue');
```



**vm.$forceUpdate()**

迫使 Vue 实例重新渲染。注意它仅仅影响实例本身和插入插槽内容的子组件，而不是所有子组件。



![](https://cn.vuejs.org/images/lifecycle.png)