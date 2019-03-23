---
title: 简单实现call、apply、bind
date: 2019-02-26 14:45:24
tags: 
- JavaScript
categories: 
- JavaScript
---



### Function.prototype.call()

#### 先看下call的表现，第一个🌰

```javascript
var cat = {
    food:'fish',
    eat:function(){
        console.log(this.food);
    }
}

var dog = {
    food:'meat',
}
cat.eat.call(dog);//meat
cat.eat.call();//undefined
console.log(dog);//{ food: 'meat' }
```



#### 实现：

##### 第一版本实现：

```javascript
Function.prototype.myCall = function () {
    var ctx = arguments[0] || window;
    ctx.fn = this;
    ctx.fn();
}
var cat = {
    food:'fish',
    eat:function(){
        console.log(this.food);
    }
}

var dog = {
    food:'meat',
}
cat.eat.myCall(dog);//meat
console.log(dog);//{ food: 'meat', fn: [Function: eat] }
```

这里将传入的`ctx`添加了`eat`方法，并不是我想要的，于是我想到了创建一个新的对象。



##### 第二版本实现：

我得为传入的对象创建一个副本，马上想到`Object.create()方法`，这个方法可以让我通过`obj.__proto__`访问到原型上的我需要用到的其他属性，又不会改变原来的对象，并且在调用过`obj.fn()`之后，`obj`就会被回收销毁，看没有副作用。但是之后出现了使用这个方式创建副本的缺点。

```javascript
Function.prototype.myCall = function () {
    var ctx = arguments[0] || window;
    var obj = Object.create(ctx);
    obj.fn = this;
    obj.fn();
}
var cat = {
    food:'fish',
    eat:function(){
        console.log(this.food);
    }
}

var dog = {
    food:'meat',
}
cat.eat.myCall(dog);//meat
console.log(dog);//{ food: 'meat' }
```



#### 第二个🌰，为call方法加入多个参数

```javascript
function Parent(a,b,c){
    this.a = a;
    this.b = b;
    this.c = c;
}

function Child(a,b,c,d){
    Parent.call(this,a,b,c);
    this.d = d;
}

var child = new Child(1,2,3,4);
console.log(child);//{a: 1, b: 2, c: 3, d: 4}
```



##### 第三版本实现，稍微改造一下`myCall`方法

```javascript
Function.prototype.myCall = function () {
    var ctx = arguments[0] || window;
    var args = [...arguments].slice(1);
    var obj = Object.create(ctx);
    obj.fn = this;
    obj.fn(...args);
}
function Parent(a,b,c){
    this.a = a;
    this.b = b;
    this.c = c;
}

function Child(a,b,c,d){
    Parent.myCall(this,a,b,c);
    this.d = d;
}

var child = new Child(1,2,3,4);
console.log(child);//{ d: 4 }
```

发现问题了，打印的`child`只有属性`d`，这是因为在调用`Parent.myCall(this,a,b,c);`的时候，传入的this问题，如图：

![WX20190227-100303@2x](http://www.qinhanwen.xyz/images//WX20190227-100303@2x.png)

但是在`myCall`方法里，创建的`obj.__proto__ = ctx`，而`obj.fn()`调用的时候，Parent方法里this指向这个新创建的对象，如图：

![WX20190227-100341@2x](http://www.qinhanwen.xyz/images//WX20190227-100341@2x.png)

所以使用`Object.create()`方法似乎不是一个很好的选择



##### 第四版本实现：改变一下`myCall`方法中创建对象的方式

```javascript
Function.prototype.myCall = function () {
    var ctx = arguments[0] || window;
    var args = [...arguments].slice(1);
    var obj = Object(ctx);
    obj.fn = this;
    obj.fn(...args);
    delete obj.fn;
}
function Parent(a,b,c){
    this.a = a;
    this.b = b;
    this.c = c;
}

function Child(a,b,c,d){
    Parent.myCall(this,a,b,c);
    this.d = d;
}

var child = new Child(1,2,3,4);
console.log(child);//{a: 1, b: 2, c: 3, d: 4}
```

`Object()`与`new Object()`是一样的。稍微测试一下，看起来似乎没什么问题



#### 看下一个🌰，发现一个有趣事情

```javascript
console.log(window.a);//undefined
console.log(window.b);//undefined
console.log(window.c);//undefined
function Parent(a,b,c){
    this.a = a;
    this.b = b;
    this.c = c;
}

function Child(a,b,c,d){
    Parent.call(null,a,b,c);
    this.d = d;
}

var child = new Child(1,2,3,4);
console.log(child);//{d: 4}
console.log(window.a);//1
console.log(window.b);//2
console.log(window.c);//3
```

看下[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)的介绍得知，当传入的第一个参数为null或者undefined，就会默认指向全局，也就导致了上面的这些值暴露到了window对象上。

![WX20190227-102830@2x](http://www.qinhanwen.xyz/images//WX20190227-102830@2x.png)



上面的实现使用了展开运算符，虽然大部分浏览器都支持es6，但是总是感觉还是得用个什么东西来替代一下。

首先，看一下展开运算符通过Babel转成es5的代码

```javascript
var arr = [1,2,3];
console.log(...arr);
//转换
var _console;
var arr = [1, 2, 3];
(_console = console).log.apply(_console, arr);
```

然后发现它是通过apply展开的。



然后想到了几个蠢办法，于是想想我还是放弃了，不过首先了解到arguments是个类数组，缺少Array的方法，可以将它转成数组。

```javascript
Array.prototype.slice.call(arr);
```

同时arguments是可以枚举的。



### Function.prototype.apply()

这个方法与call方法只是接收参数不一样，其他基本一致，稍微改造一下`myCall`方法成`myApply`方法

```javascript
Function.prototype.myApply = function () {
    var ctx = arguments[0] || window;
    var args = [...arguments].slice(1)[0];
    var obj = Object(ctx);
    obj.fn = this;
    obj.fn(...args);
    delete obj.fn;
}
function Parent(a,b,c){
    this.a = a;
    this.b = b;
    this.c = c;
}

function Child(a,b,c,d){
    Parent.myApply(this,[a,b,c]);
    this.d = d;
}

var child = new Child(1,2,3,4);
console.log(child);//{a: 1, b: 2, c: 3, d: 4}
```



我如此大胆的直接使用 `[...arguments].slice(1)[0]`，是因为将arguments展开，即使没有传入参数，也是个空数组，调用slice方法会返回一个数组，我可以直接取第一个数据

```javascript
function test(){
    console.log([...arguments].slice(1)[0]);
}
test();//undefined
```



### Function.prototype.bind()

举个🌰

```javascript
function getFood(){
    console.log(this.food)
}
var cat = {
    food:'fish'
}

var getFood1 = getFood.bind(cat);
getFood1();//fish
console.log(cat);//{ food: 'fish' }
```



实现`myBind`方法：

```javascript
Function.prototype.myBind = function () {
    var ctx = arguments[0] || window;
    var fn = this;
    return function () {
        ctx.fn = fn;
        ctx.fn();
        delete ctx.fn;
    }
}
function getFood(){
    console.log(this.food)
}
var cat = {
    food:'fish'
}

var getFood1 = getFood.bind();
getFood1();//fish
console.log(cat);//{ food: 'fish' }
```





### 总结

1.在严格模式下和Node环境，是没有window对象的。

2.ctx.fn可能会覆盖之前的同名fn方法，所以最好加个单独标识。

### 





#### 附件题

```javascript
var big = '123';
var obj = {
    big:'234',
    getBig:function(){
        console.log(this.big);
    }
}
obj.getBig.call(big);
```





