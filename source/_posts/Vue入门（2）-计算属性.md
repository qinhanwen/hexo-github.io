---
title: Vue入门（2）- 计算属性
date: 2019-08-25 23:11:57
tags: 
- Vue入门
categories: 
- Vue入门
---

## 计算属性

就是依赖的值更新的时候，这个值也会更新，与之相关的DOM也会更新。(在插值中的复杂逻辑运算，应该使用计算属性)。

```html
<p>{{message}}</p>
<p>reversed message: "{{ reversedMessage }}"</p>
<p>function reversed message :{{reversedMessageFunc()}}</p>
```

```javascript
data: {
  message: 'hello qinhanwen',
},
methods: {
  reversedMessageFunc: function () {
    return this.message.split('').reverse().join('');
  }
},
computed: {
      // 计算属性的 getter
  reversedMessage: function () {
    // `this` 指向 vm 实例
    return this.message.split('').reverse().join('');
  }
 }
```

在表达式中使用`计算属性`或者`调用方法`都可以实现，它们的区别：

- 计算属性是基于它们的**响应式依赖进行缓存**的。只在相关响应式依赖发生**改变**时它们才会重新求值。

- 而每当触发重新渲染时，调用方法将总会再次执行函数。

我们为什么需要缓存？假设我们有一个性能开销比较大的计算属性 **A**，它需要遍历一个巨大的数组并做大量的计算。然后我们可能有其他的计算属性依赖于 **A** 。如果没有缓存，我们将不可避免的多次执行 **A** 的 getter！如果你不希望有缓存，请用方法来替代。



**侦听属性**

```html
<input type="text" v-model="name">
<p>{{fullName}}</p>
```

```javascript
data: {
    name:'qinhanwen',
    fullName:''
},
  watch: {
    name: function (newName, oldName) {
      this.fullName = 'new name ' + newName;
    },
  }
```



**计算属性的setter**

计算属性默认只有 getter ，不过在需要时你也可以提供一个 setter 

```javascript
computed: {
    // 计算属性的 getter
    reversedMessage: function () {
        // `this` 指向 vm 实例
        return this.message.split('').reverse().join('');
    },
    myAge: {
        set: function (value) {
            console.log(value);
        },
        get: function () {
            return this.myAge;
        }
    }
},
```

