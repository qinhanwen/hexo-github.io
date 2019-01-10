---
title: this
date: 2018-12-05 23:30:20
tags: 
- JavaScript
categories: 
- JavaScript
---

# this

在手写Promise的时候被this折腾的不清，网上this又一堆资料看了也还是不会，于是决定认真的看一下`你不知道的JavaScript`。用的例子基本都是原文偷的。



#### 1.了解调用位置

就是函数被调用的位置

```javascript
function baz() {
    // 当前调用栈是:baz
    // 因此，当前调用位置是全局作用域
    console.log("baz");
    bar(); // <-- bar 的调用位置 
}
function bar() {// 当前调用栈是 baz -> bar
    // 因此，当前调用位置在 baz 中
    console.log("bar");
    foo(); // <-- foo 的调用位置
}
function foo() {
    // 当前调用栈是 baz -> bar -> foo // 因此，当前调用位置在 bar 中
    console.log("foo");
}
baz(); // <-- baz 的调用位置
```



在图片里也可以看到，调用到foo函数的时候`baz -> bar -> foo `

![](http://39.105.62.145/assets/images/WX20181206-234609@2x.png)





#### 2.绑定规则

##### 1）默认绑定

```javascript
var a = 2;
function test(){
    console.log(this.a);
}
test();//2
```

这里的this指向window。

在严格模式下

```javascript
var a = 2;
function test(){
	"use strict";
    console.log(this.a);
}
test();
//TypeError: Cannot read property 'a' of undefined
```



##### 2）隐式绑定

```javascript
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2,
    foo: foo
};
obj.foo(); // 2
```

当 foo() 被调用时，它的落脚点确实指向 obj 对象。当函数引用有上下文对象时，隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象。



###### 隐式丢失

```javascript
var a = 'global';

function test(){
    console.log(this.a);
}

var obj = {
    a:'1',
    test:test
}

var test1 = obj.test;
test1();//global
```

test1和obj.test指向相同。都是指向test函数，因为test1()不带任何修饰的函数调用，所以使用`默认绑定`的方式。



关于setTimeout这种里面this指向的问题

```javascript
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2,
    foo: foo
};
var a = "oops, global"; // a 是全局对象的属性 setTimeout( obj.foo, 100 ); // "oops, global"
setTimeout(obj.foo, 100); // "oops, global"
```

看一下setTImeout的伪代码

```javascript
function setTimeout(fn,delay) {
    // 等待 delay 毫秒
	fn(); // <-- 调用位置!
}
```



##### 3）显式绑定

**显式绑定，其实我觉得就是强制把this绑定在某个对象上。**

可以通过使用`apply`和`call`：这两个方法的区别就是接受参数，第一个是上下文context，apply第二个参数是个数组，call是将参数都放在后面。

看🌰，猫吃鱼，狗吃肉

```javascript
var cat = {
    food: 'fish',
    eat: function (a,b,c) {
        console.log(this.food);
        console.log(a,b,c);
    }
}
var dog = {
    food: 'meat',
    eat: function (a,b,c) {
        console.log(this.food);
        console.log(a,b,c);
    }
}
//猫想吃肉了。
cat.eat.apply(dog,[1,2,3])
//meat
//1,2,3
cat.eat.call(dog,1,2,3)
//meat
//1,2,3
```

在调用 cat.eat 的时候将this强制绑定到了dog上。



**硬绑定**：显式绑定的变种

看🌰

```javascript
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2
};
var bar = function () {
    foo.call(obj);
};
bar(); // 2
setTimeout(bar, 100); // 2
// 硬绑定的 bar 不可能再修改它的 this bar.call( window ); // 2
```

其实就是在使用一个函数包裹foo.call(obj)，之后怎么调用bar，都会将foo的this绑定在obj上。



**硬绑定**的应用，es5提供了内置方法`Function.prototype.bind`

```javascript
function foo(something) {
    console.log(this.a, something);
}
var obj = {
    a: 2
};
var bar = foo.bind(obj);
bar(3);//2 3
```

实现一个bind方法

```javascript
function bind(fn,ctx){
    return function(){
        fn.apply(ctx,arguments);
    }
}

function foo(something){
    console.log(this.a,something);
}
var obj = {
    a:2
}
var bar = bind(foo,obj);
bar(3);//2 3
```



##### 4）new绑定

[new的时候做了什么](https://qinhanwen.github.io/2018/12/04/new%E7%9A%84%E6%97%B6%E5%80%99%E5%88%B0%E5%BA%95%E5%81%9A%E4%BA%86%E4%BB%80%E4%B9%88/)

```javascript
function foo(a) {
    this.a = a;
}
var bar = new foo(2);
console.log(bar.a); // 2
```





#### 3.优先级

以上我们知道了4种绑定方式：默认绑定，显式绑定，隐式绑定，new绑定。现在对比他们的优先级，默认绑定优先级最低（忽略）。

我们需要找到调用的位置，并且应用哪种绑定方式。

##### 1）`显式绑定`和`隐式绑定`对比

```javascript
function test(){
    console.log(this.a);
}

var obj = {
    a:1,
    test:test
}

var obj1 = {
    a:2,
    test:test
}
obj.test();//1
obj.test.apply(obj1);//2
```

优先级：显式绑定 > 隐式绑定



##### 2）`隐式绑定`和`new绑定`对比

```javascript
function test(a){
    this.a = a;
}

var obj = {
    a:1,
    test:test
}

obj.test(3);
var obj1 = new obj.test(4);//new 绑定的时候this指向 test{}
console.log(obj.a);//3   
console.log(obj1.a);//4
```



优先级：new绑定 > 隐式绑定



##### 3）`显式绑定`与`new绑定`

```javascript
function foo(something) {
    this.a = something;
}
var obj1 = {};
var bar = foo.bind(obj1);
bar(2);
console.log(obj1.a); // 2
var baz = new bar(3);
console.log(obj1.a); // 2
console.log( baz.a ); // 3
```

优先级：new绑定 > 显式绑定



#### 4.判断this

判断this 现在我们可以根据优先级来判断函数在某个调用位置应用的是哪条规则。可以按照下面的 

顺序来进行判断: 

1. 函数是否在new中调用(new绑定)?如果是的话this绑定的是新创建的对象。 

   ```javascript
        var bar = new foo()
   ```

2. 函数是否通过call、apply(显式绑定)或者硬绑定调用?如果是的话，this绑定的是 指定的对象。 

   ```javascript
        var bar = foo.call(obj2)
   ```

3. 函数是否在某个上下文对象中调用(隐式绑定)?如果是的话，this绑定的是那个上 下文对象。 

   ```javascript
        var bar = obj1.foo()
   ```

4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到 全局对象。 

   ```javascript
        var bar = foo()
   ```



#### 5.绑定例外

##### 1）apply(null)

使用null的时候，一般是为了展开数组

```javascript
var arr = [1,2,3];
function test(a,b,c){
    console.log(a,b,c);
}
test.apply(null,arr);
```



##### 2）间接引用

```
function foo() {
    console.log(this.a);
}
var a = 2;
var o = {
    a: 3,
    foo: foo
};
var p = {
    a: 4
};
o.foo(); // 3
(p.foo = o.foo)(); // 2
```

赋值表达式p.foo = o.foo的返回值是目标函数的引用，因此调用位置是foo()而不是p.foo() 或者 o.foo()。根据我们之前说过的，这里会应用默认绑定。













