---
title: Vue入门（3）- 组件
date: 2019-08-26 21:54:32
tags: 
- Vue入门
categories: 
- Vue入门
---

## 定义组件

**全局注册**

因为组件是可复用的 Vue 实例，所以它们与 `new Vue` 接收相同的选项，例如 `data(必须是一个函数)`、`computed`、`watch`、`methods`以及生命周期钩子等

需要注意的是组件模板需要被一个根元素包裹，而且组件名称建议使用`短横线分隔命名`：

```html
<div id="learning-vue">
  <button-counter></button-counter>
</div>
```

```javascript
// 定义一个名为 button-counter 的新组件
Vue.component('button-counter', {
  data: function () {//是一个函数
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```



**局部注册**

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
  template: '<App :name="name" />'
})
```



## prop

**父组件向子组件传递数据**

使用 DOM 中的模板时，camelCase (驼峰命名法) 的 prop 名需要使用其等价的 kebab-case (短横线分隔命名) 命名

```html
<div id="learning-vue">
  <post-list v-for="post in posts" :post="post" :my-name="post.name"></post-list>
</div>
```

```javascript
Vue.component('post-list', {
	// props: ['post','myName'],
  props:{
    post:Object,
    myName:String
  },
  template: '<p>{{post.title}}{{post.id}}</p>'
})
```

所有的 prop 都使得其父子 prop 之间形成了一个**单向下行绑定**：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样会防止从子组件意外改变父级组件的状态，从而导致你的应用的数据流向难以理解。



## 监听子组件事件

**子组件向父子间传值**

```html
 <post-list v-on:click-me="clickMe" v-for="post in posts" :post="post"></post-list>
```

```javascript
Vue.component('post-list', {
  props: ['post'],
  template: `<p v-on:click="$emit('click-me','{a:1,b:2}')">{{post.title}}{{post.id}}</p>`
})
...
  methods: {
    clickMe: function (params) {
      console.log(params)
    }
  }
...
```



## 插槽

和 HTML 元素一样，我们经常需要向一个组件传递内容

```html
<slot-box>
  slot 的内容
</slot-box>
```

```javascript
Vue.component('slot-box', {
	template: `<div><slot></slot></div>`
})
```



**具名插槽**

`<slot>` 元素有一个特殊的特性：`name`。这个特性可以用来定义额外的插槽

```html
<slot-name-box>
  <template v-slot:header>
    <header>这里是header哦</header>
  </template>
  <template v-slot:footer>
    <footer>这里是footer哦</footer>
  </template>
</slot-name-box>
```

```javascript
Vue.component('slot-name-box', {
    template: `
    <div>
    <slot name="header"></slot>
    <slot name="footer"></slot>
    </div>`
})
```



具名插槽缩写：

```html
<slot-name-box>
  <template #header>
    <header>这里是header哦</header>
  </template>
  <template #footer>
    <footer>这里是footer哦</footer>
  </template>
</slot-name-box>
```



**作用域插槽**

插槽中内容能够访问子组件中的数据

```html
<slot-box :name1="name" :name2="name">
  <template v-slot:default="slotProps">
    {{slotProps.name1}}
    {{slotProps.name2}}
  </template>
</slot-box>
```

将包含所有插槽 prop 的对象命名为 `slotProps`，但你也可以使用任意你喜欢的名字。

```javascript
Vue.component('slot-box', {
  props: ['name1','name2'],
  template: `<div>my name is <slot :name1="name1" :name2="name2"></slot></div>`
})
...
	data:{
      name: 'qinhanwen',
  }
...
```

绑定在` <slot> `元素上的特性被称为插槽 `prop`



## 访问元素或组件

`$root`、`$parent`和`$refs`

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>learning-vue</title>
</head>
<script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
<!-- <script src="https://unpkg.com/vue-router/dist/vue-router.js"></script> -->

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
</script>

</html>
```



