---
title: Iterator
date: 2019-03-06 21:24:06
tags: 
- es6
categories: 
- es6
---

[了解Generator，以及为对象添加Iterator接口](https://qinhanwen.github.io/2019/03/05/Generator/#more)



#### 概念

为数据结构提供的访问机制，数据只要部署了Iterator接口，就能完成遍历。



#### Symbol.iterator属性与for…of

首先看个🌰，已知对象是没有Iterator接口的，那么让它获得一个

```javascript
var obj = {
}
for (var val of obj) {
    console.log(val);
}
//报错  TypeError: obj[Symbol.iterator] is not a function
//于是我们为对象添加一个 Symbol.iterator 属性


var obj = {
    [Symbol.iterator]: function () {
        return this
    },
}
for (var val of obj) {
    console.log(val);
}
//报错  TypeError: .iterator.next is not a function
//接着为对象添加next方法，需要有个返回对象，否则会报错


var obj = {
    [Symbol.iterator]: function () {
        return this
    },
    next: function () {
        return {}
    }
}
for (var val of obj) {
    console.log(val);
}
//不过这样会一直打印undefined，如果返回的对象是{value:1,done:false}，那么会一直打印1
//修改一下return的对象


var obj = {
    [Symbol.iterator]: function () {
        return this
    },
    next: function () {
        return { value: 1, done: true }
    }
}
for (var val of obj) {
    console.log(val);
}


//修改一下
var obj = {
    index: 0,
    [Symbol.iterator]: function () {
        return this
    },
    next: function () {
        if (this.index < 2) {
            return { value: this.index++, done: false }
        }else{
            return { value: this.index, done: true }
        }
    }
}
for (var val of obj) {
    console.log(val);
}
//0
//1
```



从上面可以看到`for…of`做的事情

1）先调用了数据的`Symbol.iterator`方法

2）接着调用next方法，打印出返回对象里的value





#### 数据结构默认的Iterator接口

```javascript
var arr = [1,2,3];
var g = arr[Symbol.iterator]();
console.log(g.next());//{ value: 1, done: false }
console.log(g.next());//{ value: 2, done: false }
console.log(g.next());//{ value: 3, done: false }

for(let key of arr){
    console.log(key);// 1 2 3
}
```





#### 对拥有Iterator接口的数据结构可以使用哪些（在顶上的Generator函数的链接里有介绍这些）

1）展开操作符

2）Array.from

3）for…of

4）yield*



















