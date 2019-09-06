---
title: Vue入门（4）- 过滤器
date: 2019-08-26 23:51:03
tags: 
- Vue入门
categories: 
- Vue入门
---

## 过滤器

Vue.js 允许你自定义过滤器，可被用于一些常见的文本格式化。过滤器可以用在两个地方：**双花括号插值和 v-bind 表达式** (后者从 2.1.0+ 开始支持)。过滤器应该被添加在 JavaScript 表达式的尾部，由“管道”符号指示

```html
<!-- 在双花括号中 -->
{{ message | capitalize }}

<!-- 在 `v-bind` 中 -->
<div v-bind:id="rawId | formatId"></div>
```



## 自定义过滤器

```javascript
Vue.filter('capitalize', function (value) {
  if (!value) return ''
  value = value.toString()
  return value.toUpperCase();
})

...
    data: {
      message: 'QInhAnwen'
    },
	  filters: {
            capitalize1: function (value) {
                if (!value) return ''
                value = value.toString()
                return value.substr(2);
            }
    }
...
```

```html
{{message | capitalize}}
{{message | capitalize1}}
```



**接收参数**

```javascript
capitalize2: function (value, value2, value3) {
		return value + value2 + value3;
}
```

```html
{{message | capitalize2(2,3)}}
```

`message`作为第一个参数之后的入参按顺序。