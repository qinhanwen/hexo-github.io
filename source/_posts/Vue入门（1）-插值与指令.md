---
title: Vue入门（1）-插值与指令
date: 2019-08-25 13:53:44
tags: 
- Vue入门
categories: 
- Vue入门
---

[项目git地址，按标题提交版本](https://github.com/qinhanwen/learning-vue.git)



## 安装

- 通过script标签引入CDN文件
- npm install vue --save-dev
- bower install vue --save-dev



或使用`vue-cli`工具

```shell
$ npm install -g @vue/cli
```



这里选择引入CDN的JS文件

```javascript
<script src="https://cdn.jsdelivr.net/npm/vue@2.5.22/dist/vue.js"></script>
```

这个是未压缩版本，压缩版本只需把`vue.js`改成`vue.min.js`，当然压缩版本少了一些关键的错误提示信息。



## 插值与表达式

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>learning-vue</title>
</head>
<script src="https://cdn.jsdelivr.net/npm/vue@2.5.22/dist/vue.js"></script>

<body>
    <div id="learning-vue">
        <ul>
            <li v-for="tab in tabs">{{tab}}</li>
        </ul>
        <div v-html="bindHTML">
        </div>
        <div :id="id1">
            123
        </div>
        <div>
            multiply:{{number * 10}}
        </div>
    </div>
</body>
<script>
    new Vue({
        el: "#learning-vue",
        data: {
            tabs: [
                'name1',
                'name2'
            ],
            bindHTML: "<p>p标签</p>",
            id1: 'myid',
            number: 100,
        },
    })
</script>

</html>
```





## 指令

原生标签前面`v-`前缀开头



**v-show**

值为false的时候，元素被隐藏，审查元素发现元素上多了`display:none`



**v-if**

值为false的时候，不会在DOM中



**v-else-if**

跟在`v-if`后面



**v-else**

跟在`v-if`后面

```html
<p v-if="directive">if directive</p>
<p v-else="directive">else directive</p>
```



**v-model**

在表单控件上创建双向绑定

```html
<form>
  <input type="text" name="userName" v-model="value" />
  <p>{{value}}</p>
</form>
```



**v-for**

基于源数据重复渲染元素

```html
<ul>
  	<li v-for="(tab,index) in tabs" :class="tab+index" v-on:click="change(index)">{{tab}}		   {{index}}</li>
</ul>
```

当数组数据出现变化时，只有部分操作会触发视图更新：

- `push()`

- `pop()`

- `shift()`

- `unshift()`

- `splice()`

- `sort()`

- `reverse()`

  

那如果需要修改某项的值

```javascript
change(index) {
    // this.tabs[index] = 'index-change';//直接根据索引值修改是无效的
    this.$set(this.tabs, index, 'index-change');
}
```

也可以使用filter、slice、map方法返回的新数组替换原来的数组

```javascript
change(index) {
 	 this.tabs = this.tabs.filter((item, i) => i == index);
}
```

Vue 为了使得 DOM 元素得到最大范围的重用而实现了一些智能的启发式方法，所以用一个含有相同元素的数组去替换原来的数组是非常高效的操作，不一定会重新渲染整个列表。

```html
<ul>
  	<li v-for="(tab,index) in tabs" :class="tab+index" v-on:click="change(index)" key="tab.id">{{tab}}		   {{index}}</li>
</ul>
```



**v-html**

更新元素的innerHTML



**v-bind**

用于响应更新HTML特性

```html
<img v-bind:src="src" />
<!-- 简写 -->
<img :src="src" />
```



```html
<div :id="id1">
	123
</div>
<div :class="[classA,{classB:isB}]">
  class
</div>
<div v-bind="{id:id1,class:[classA,{classB:isB}]}">
  other
</div>
```

![WX20190825-195925@2x](http://118.24.241.76/WX20190825-195925@2x.png)

**动态参数**

```html
<a v-bind:[attributename]="src"> ... </a>
```

```javascript
 data: {
   attributename:'href',
   src:'xx.png'
 }
```



**v-on**

绑定事件监听

```html
<div v-on:click="clickMe(2,$event)">
	event2
</div>
<!-- 简写 -->
<div @click="clickMe(1,$event)">
	event
</div>
```

修饰符

- `.stop`  调用event.stopPropagation()
- `.prevent`  调用 event.preventDefault()
- `指定按钮触发` enter:13，space:32 等等

```html
<a href="http://www.baidu.com" v-on:click.prevent="clickMe(1,$event)">www.baidu.com</a>

<div @click="clickMe(1,$event)">
  <div v-on:click.stop="clickMe(2,$event)">
	  event2
  </div>
</div>

<input type="text" value="space" @keyup.enter="clickMe(1,$event)">
<!-- 等价于 -->
<input type="text" value="space" @keyup.13="clickMe(1,$event)">
```



 **v-cloak** 

防止页面加载时出现 vuejs 的变量名而设计，一般也用不到

```html
<style>
  [v-cloak]{
      display: none;
  }
</style>

<div v-cloak>
  {{ message }}
</div>
```



**v-ref与v-el**

使用vue版本是`2.5.22`

```html
<p rel=ele @click="clickMe">v-el</p>
<p ref=oneRef  @click="clickMe">v-ref</p>
```

```javascript
clickMe() {
  console.log(this.$refs);
  console.log(this.$eles);
}
```



**自定义指令**

钩子函数 (均为可选)：

- `bind`：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
- `inserted`：被绑定元素插入父节点时调用 (仅保证父节点存在，但不一定已被插入文档中)。
- `update`：所在组件的 VNode 更新时调用，**但是可能发生在其子 VNode 更新之前**。指令的值可能发生了改变，也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新。

- `componentUpdated`：指令所在组件的 VNode **及其子 VNode** 全部更新后调用。
- `unbind`：只调用一次，指令与元素解绑时调用。



钩子参数：

`el`：指令所绑定的元素，可以用来直接操作 DOM 。

`binding`：一个对象，包含以下属性：

- `name`：指令名，不包括 `v-` 前缀。
- `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 `2`。
- `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
- `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"` 中，表达式为 `"1 + 1"`。
- `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`。
- `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar` 中，修饰符对象为 `{ foo: true, bar: true }`。

`vnode`：Vue 编译生成的虚拟节点。移步 [VNode API](https://cn.vuejs.org/v2/api/#VNode-接口) 来了解更多详情。

`oldVnode`：上一个虚拟节点，仅在 `update` 和 `componentUpdated` 钩子中可用。



```html
...
<div v-demo:a.b.c="message" @click="clickDirec()"></div>
...
```

```javascript
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify;
    el.innerHTML =
      'name: ' + s(binding.name) + '<br>' +
      'value: ' + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: ' + s(binding.arg) + '<br>' +
      'modifiers: ' + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  },
  update: function (el, value) {
    console.log(el);
    console.log(value);
  }
})

...
data: {
  message: {name:'hello qinhanwen',age:25},
},
methods: {
  clickDirec() {
    this.message = 'hello'
  }
}
...
```

![WX20190825-221351@2x](http://118.24.241.76/WX20190825-221351@2x.png)



**动态指令参数**

这里版本不支持，vue版本改为`2.6.0`

```javascript
v-mydirective:[argument]="value"
```

`argument` 参数可以根据组件实例数据进行更新！这使得自定义指令可以在应用中被灵活使用。

示例1：

```html
 <p v-pin="200">Stick me 200px from the top of the page</p>
```

```javascript
 Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    el.style.top = binding.value + 'px'
  }
})
```

示例2：

```html
<p v-pin1:[direction]="200">I am pinned onto the page at 200px to the left.</p>
```

```javascript
Vue.directive('pin1', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    var s = (binding.arg == 'left' ? 'left' : 'top')
    el.style[s] = binding.value + 'px'
  }
})
...
data:{
  direction: 'left'
}
...
```

![WX20190825-222731@2x](http://118.24.241.76/WX20190825-222731@2x.png)

**对象字面量**

如果指令需要多个值，可以传入一个 JavaScript 对象字面量，像上面的`value`就是个对象



