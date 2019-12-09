---
title: vue-ssr-2
date: 2019-12-06 11:57:34
tags: 
- vue-ssr
categories: 
- vue-ssr
---

之前的例子集成了vue-router和vuex，但是有一些项目项目如果没有使用到这两者，光引入就又需要改造成本，并不是一个好的体验，所以这边讲的是没有集成vuex的处理方式



## 思路

构造一个Vue的实例，那么我们可以用这个实例的data来存储我们的预取数据，而用methods中的方法去做数据的异步获取，这样我们只在需要预取数据的组件中去调用这个方法就可以了。



## 开始

首先需要让我们的组件“共享”这个EventBus，封装一个plugin：

```javascript
export default {
 install (Vue) {
   const EventBus = new Vue({
     data () {
       return {
	      list: [],
	      nav: []
       }
     },
     methods: {
       getList () {
	      // get list
		},
       getNav () {
         // get nav
       }
     }
   })
   
   Vue.prototype.$events = EventBus
   Vue.$events = EventBus
 }
}
```



然后我们需要在app.js中export出我们的EventBus以便两个entry使用。这样我们的app.js就像下面这样：

```javascript
import Vue from 'vue'
import App from './App'
import EventBus from './event'

Vue.use(EventBus)
Vue.config.devtools = true

export function createApp () {
 const app = new Vue({
   // 注入 router 到根 Vue 实例
   router,
   render: h => h(App)
 })
 
 return { app, router, eventBus: app.$events }
}
```











## 参考资料

[解密Vue SSR](https://juejin.im/post/5b063962f265da0ddb63dac3)