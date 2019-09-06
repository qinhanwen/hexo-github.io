---
title: Vue入门（5）- class与style绑定
date: 2019-08-27 12:35:36
tags: 
- Vue入门
categories: 
- Vue入门
---

操作元素的 class 列表和内联样式是数据绑定的一个常见需求。因为它们都是属性，所以我们可以用 `v-bind` 处理它们：只需要通过表达式计算出字符串结果即可。不过，字符串拼接麻烦且易错。因此，在将 `v-bind` 用于 `class` 和 `style` 时，Vue.js 做了专门的增强。表达式结果的类型除了字符串之外，还可以是对象或数组。

## Class

**对象语法**

```html
<div class="static" v-bind:class="{ active: isActive, 'text-danger': hasError }">class</div>
```

```javascript
data: {
  isActive: true,
  hasError: true,
}
```

![WX20190827-124617@2x](http://www.qinhanwen.xyz/WX20190827-124617@2x.png)

当`isActive`与`hasError`变化的时候，class 列表将相应地更新。



绑定的数据对象不必内联定义在模板里

```html
<div class="static" v-bind:class="classObject">class</div>
```

```javascript
data: {
  classObject: {
		active: true,
		'text-danger': true
  }
}
```



也可以绑定一个返回对象的`计算属性`

```html
<div v-bind:class="classObjectComputed">class2</div>
```

```javascript
computed: {
  classObjectComputed: function () {
    return {
      active: this.isActive,
      'text-danger': this.hasError
    }
  }
}
```



**数组语法**

```html
<div v-bind:class="[activeClass, errorClass]">class4</div>
```

```javascript
data: {
  activeClass:'active',
  errorClass:'text-danger'
},
```

![WX20190827-125519@2x](http://www.qinhanwen.xyz/WX20190827-125519@2x.png)



也可以使用三元表达式或者对象语法

```html
<div v-bind:class="[isActive ? activeClass : '', errorClass]">class5</div>
<div v-bind:class="[{active:isActive}, errorClass]">class6</div>
```

```javascript
data: {
  isActive: true,
	activeClass:'active',
	errorClass:'text-danger'
},
```



**在组件上使用**

```html
<class-box :class="{active:isActive}"></class-box>
```

```javascript
Vue.component('class-box',{
  template:'<p class="inner-tag">component</p>'
})
...
  data: {
    isActive: true,
  },
...
```

![WX20190827-130200@2x](http://www.qinhanwen.xyz/WX20190827-130200@2x.png)





## style

`v-bind:style` 的对象语法十分直观——看着非常像 CSS，但其实是一个 JavaScript 对象。CSS 属性名可以用驼峰式 (camelCase) 或短横线分隔 (kebab-case，记得用引号括起来) 来命名：

```HTML
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

```javascript
data: {
  activeColor: 'red',
  fontSize: 30
}
```

![WX20190827-130549@2x](http://www.qinhanwen.xyz/WX20190827-130549@2x.png)



也可以不内联在模板中

```html
<div v-bind:style="styleObject"></div>
```

```javascript
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```



**数组语法**

```html
<div v-bind:style="[baseStyles,overridingStyles]">style2</div>
```

```javascript
data: {
  baseStyles: {
    color: 'green',
      fontSize: '30px'
  },
    overridingStyles: {
      'font-weight': 'bold'
    }
},
```

![WX20190827-130955@2x](http://www.qinhanwen.xyz/WX20190827-130955@2x.png)