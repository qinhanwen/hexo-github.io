---
title: ES6学习笔记
date: 2018-12-03 20:58:38
tags: 
- es6
categories: 
- es6
---

# ES6学习笔记

>  [Babel](https://www.babeljs.cn/)



#### 1.const和let

###### let特点：

1.在编译过程中没有变量提升

2.不允许重复声明（同个代码块内）

3.块级作用域(只在声明的代码块内有效)

4.暂时性死区



###### 1）说个问题

```javascript
for (var i = 0; i < 5; i++) {
    setTimeout(() => {
        console.log(i);
    }, i * 1000)
}
// 每隔1秒打印一个5
```

其实是想每隔一秒打印出0到4，使用let

```javascript
for (let i = 0; i < 5; i++) {
    setTimeout(() => {
        console.log(i);
    }, i * 1000)
}
// 每隔1秒打印一个
//0
//1
//2
//3
//4
```

代码转成es5的结果：

![WX20190123-131241@2x](http://www.qinhanwen.xyz/images/WX20190123-131241@2x.png)



当然也可以使用闭包（在函数调用的时候，首先会在函数内寻找变量，找不到就会在定义函数的作用域内找）

```javascript
for (var i = 0; i < 5; i++) {
    ((j) => {
        (setTimeout(() => {
            console.log(j);
        }, j * 1000))
    })(i)
}
```



###### 2）临时性死区

```javascript
var a = 1;
if(true){
    a = 2;
    let a;
}
//a is not defined
```

在es6中，在区域中存在let和const声明的变量，在声明之前使用就会报错。





###### const特点：

1.在编译过程中没有变量提升

2.不允许重复声明（同个代码块内）

3.块级作用域(只在声明的代码块内有效)

4.暂时性死区

5.声明常量，必须在初始化的时候赋值，不可再改变值。



但是const比较奇怪的地方就是：

保存的是常量的时候，不允许修改，如果保存的是复合类型数据的引用地址，可以修改数据的属性。

```javascript
const obj = {
    key:'value'
}
obj.key = 'value1';
console.log(obj);//{ key: 'value1' }
```





#### 2.对象和数组的解构

特点：

1.按照一定的模式，从数组和对象中取得值。

2.解构不成功，值为undefined。

3.指定默认值



###### 1）一般使用方法

对象：

```javascript
var obj = {
    a:1,b:2,c:3
}
const {a,b,c,d} = obj;
console.log(a);//1
console.log(b);//2
console.log(c);//3
console.log(d);//undefined
```

数组：

```javascript
var arr = [1,2,3];
var [a,b,c] = arr;
console.log(a);
console.log(b);
console.log(c);
```



###### 2）指定默认值

```javascript
var obj = {
    a:undefined,b:null
}
const {a=1,b=2} = obj;
console.log(a);//1
console.log(b);//null
```

null和undefined有区别。



###### 3）对象解构赋值的特点 (变量名与属性名不一样)

```javascript
let { foo: baz } = { foo: 'aaa', bar: 'bbb' };
console.log(baz);//aaa
```

可以理解为：

```javascript
let { foo } = { foo: 'aaa', bar: 'bbb' };
//其实就是
let { foo:foo } = { foo: 'aaa', bar: 'bbb' };
```

先找同名属性，再赋值给对应变量。



解构用于嵌套对象：

```javascript
const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

let { loc, loc: { start }, loc: { start: { line }} } = node;
console.log(line);// 1
console.log(loc);//{ start: { line: 1, column: 5 } }
console.log(start);//{line: 1, column: 5}
```

转换成es5：

![WX20190123-132440@2x](http://www.qinhanwen.xyz/images/WX20190123-132440@2x.png)

这样子可能更易于理解



#### 3.字符串

###### 1）模板字符串（支持换行）

```javascript
const name = "qinhanwen";
const age = 25;
let str = `${name} is ${age} years old`;
console.log(str);//qinhanwen is 25 years old
```



###### 2）字符串方法

```javascript
var URI = 'file://xxxxxxx.png';
console.log(URI.startsWith('file://'));//true
console.log(URI.endsWith('.png'));//true
console.log(URI.includes('xxx'));//true
```





#### 4.函数

###### 1）参数的解构赋值与默认参数

```javascript
function move({x = 0, y = 0} = {}) {
  return [x, y];
}
move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
```

编译结果：

![WX20190123-165322@2x](http://www.qinhanwen.xyz/images/WX20190123-165322@2x.png)

和

```javascript
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}
move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```

编译结果：

![WX20190123-165534@2x](http://www.qinhanwen.xyz/images/WX20190123-165534@2x.png)



这两个都是为move方法指定默认值，表现略微有点不同。看到编译结果就很好理解。



###### 2）剩余操作符

```javascript
function sum(a, ...arr) {
    console.log(a + arr.reduce(function (pre, item) {
        return pre + item;
    }));
}
sum(1, 2, 3);//6
```

[手写reduce方法](https://qinhanwen.github.io/2019/01/23/%E5%BE%AA%E7%8E%AF/)



###### 3）展开运算符

```javascript
var arr1 = [1,2,3];
var arr2 = [4,5,6];
var arr3 = [...arr1,...arr2];
console.log(arr3);//[ 1, 2, 3, 4, 5, 6 ]
```

编译一下：

![WX20190124-193403@2x](http://www.qinhanwen.xyz/images/WX20190124-193403@2x.png)



##### 这是es7！！！！哈哈哈哈哈

```javascript
var obj1 = {name:'qinhanwen'};
var obj2 = {age:25};
var obj3 = {...obj1,...obj2};
console.log(obj3);
```



##### 说一下深拷贝和浅拷贝，其实就是传值和传址的问题。

```javascript
var obj = {name:'qinhanwen'};
var obj1 = obj;
obj1.name = '啦啦啦';
console.log(obj);//{ name: '啦啦啦' }
console.log(obj1);//{ name: '啦啦啦' }
```

`var obj1 = obj;`是传递obj的引用地址，指向栈中。



##### 如何深拷贝？

想到2个办法

1.序列化与反序列化

```javascript
var obj = {name:'qinhanwen'};
var obj1 = JSON.parse(JSON.stringify(obj));
obj1.name = '啦啦啦';
console.log(obj);//{ name: 'qinhanwen' }
console.log(obj1);//{ name: '啦啦啦' }
```

不过这有个缺点，在序列化的时候，undefined与函数等会丢失

```javascript
var obj = { 
    name: 'qinhanwen',
    age:undefined,
    cb:function(){
        console.log(this.age);
    }
 };
var obj1 = JSON.parse(JSON.stringify(obj));
obj1.name = '啦啦啦';
console.log(obj);//{ name: 'qinhanwen', age: undefined, cb: [Function: cb] }
console.log(obj1);//{ name: '啦啦啦' }
```



2.递归实现一个简单的深拷贝

```javascript
function ObjectDeepClone(obj) {
    var newObj = Array.isArray(obj) ? [] : {};
    for (var key in obj) {
        if (Object.prototype.toString.call(obj[key]) == "[object Object]" || Object.prototype.toString.call(obj[key]) == "[object Array]") {
            newObj[key] = ObjectDeepClone(obj[key]);
        } else {
            newObj[key] = obj[key];
        }
    }
    return newObj;
}

var obj = {
    name: 'qinhanwen',
    age: undefined,
    cb: function () {
        console.log(this.age);
    },
    arr: [1, 2, 3],
    obj: {
        name: 'qinhanwen',
        age: undefined,
        cb: function () {
            console.log(this.age);
        },
        arr: [1, 2, 3],
    }
};

var obj1 = ObjectDeepClone(obj);
obj1.arr[0] = 0;
obj1.obj.name = '啦啦啦';

console.log(obj);
/*{ name: 'qinhanwen',
  age: undefined,
  cb: [Function: cb],
  arr: [ 1, 2, 3 ],
  obj: 
   { name: 'qinhanwen',
     age: undefined,
     cb: [Function: cb],
     arr: [ 1, 2, 3 ] } }
     */
console.log(obj1);
/*
{ name: 'qinhanwen',
  age: undefined,
  cb: [Function: cb],
  arr: [ 0, 2, 3 ],
  obj: 
   { name: '啦啦啦',
     age: undefined,
     cb: [Function: cb],
     arr: [ 1, 2, 3 ] } }
*/
```

其实typeof 数组和对象，返回都是object，直接用typeof也可以





###### 4）箭头函数

箭头函数没有this，它会使用上层的this

```javascript
var obj = {
    name:1,
    func(){setTimeout(()=>{
        console.log(this);
    },1000)}
}
obj.func();
//1秒后打印{name: 1, func: ƒ}
```

编译:

![WX20190125-001233@2x](http://www.qinhanwen.xyz/images/WX20190125-001233@2x.png)

箭头函数中的this，与es5中this指向有区别，[es5中this](https://qinhanwen.github.io/2018/12/05/this/)只与调用的上下文有关，与声明的时候无关，而箭头函数的this，在函数声明的时候就定死了。



如果this需要随调用环境变化，就不适合使用箭头函数。



#### 5.数组方法

###### 1）filter

```javascript
var arr = [1, 2, 3];
var obj = {
    arr1: []
}
var newArr = arr.filter(function (item, index, array) {
    array[index] = 0;
    this.arr1.push(item);
    return item > 2;
}, obj);
console.log(newArr);//[3]
```



###### 2）find和findIndex

###### 3）includes

###### 4）every

###### 5）some





#### 6.对象拓展

###### 1）属性和方法简写

```javascript
const a = 1;
const b = 2;
var obj = {
    a,
    b,
    method(){
        console.log('method');
    }
}
console.log(obj);//{ a: 1, b: 2, method: [Function: method] }
```



###### 2）对象方法

###### 1.setPrototypeOf

```javascript
var obj = {
    name: 'qinhanwen',
    age: 25
}
var obj1 = {};
var obj2 = Object.setPrototypeOf(obj1, obj);
console.log(obj1);//{}
console.log(obj1.name, obj1.age);//qinhanwen 25
console.log(obj2.name, obj2.age);//qinhanwen 25
```

那么setPrototypeOf做了什么，相当于：

```javascript
Object.prototype.setPrototypeOf = function (obj, proto) {
    obj.__proto__ = proto;
    return obj;
}
```



###### 2.super

```javascript
var obj = {
    name: 'qinhanwen',
    getName(){
        console.log('name1'+this.name);
    }
}
var obj1 = {
    name:'啦啦啦',
    __proto__:obj,
    getName(){
        console.log('name2'+this.name);
        super.getName();
    }
};
obj1.getName();
//name2啦啦啦
//name1啦啦啦
```





#### 7.类

##### 1）类

```javascript
class Parent{
    constructor(name){
        this.name = name
    }
    getName(){
        console.log(this.name);
    }
	static getAge(){//私有属性
    }
}
```

编译成es5：

```javascript
"use strict";

var _createClass = function () {
    function defineProperties(target, props) {//这个方法做的事情，其实就是在Constructor上绑定原型链的属性，和私有属性
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            if ("value" in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor);
        }
    }
    return function (Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);//如果存在原型上的属性
        if (staticProps) defineProperties(Constructor, staticProps);//如果存在私有属性
        return Constructor;
    };
}();

function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {//为了保证调用方式是以构造函数方式调用
        throw new TypeError("Cannot call a class as a function");
    }
}

var Parent = function () {
    function Parent(name) {
        _classCallCheck(this, Parent);

        this.name = name;
    }

    _createClass(Parent, [{
        key: "getName",
        value: function getName() {
            console.log(this.name);
        }
    }], [{
        key: "getAge",
        value: function getAge() {}
    }]);

    return Parent;
}();
```

其实总的来说做了几件事

1）函数表达式声明Parent，在它内部声明了一个函数Parent

2）绑定内部函数Parent的`原型上的属性`和`静态属性(类上的属性)`

3）返回Parent





##### 2）继承

```javascript
class Parent{
    constructor(name){
        this.name = name
    }
    getName(){
        console.log(this.name);
    }
	static getAge(){//私有属性
    }
}

class Child extends Parent{
    constructor(name,age){
        super(name);
        this.age = age;
    }
    getAge(){
        console.log(this.age);
    }
}

var child = new Child('qinhanwen',25);
child.getName();//qinhanwen
```

编译成es5：

嗯，确实有点长看起来有点吓人，看下它做了什么吧。不过看之前最好先了解一下[Object构造函数](https://qinhanwen.github.io/2019/02/18/%E5%AF%B9%E8%B1%A1/)

```javascript
'use strict';

var _createClass = function () {
    function defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            if ("value" in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor);
        }
    }
    return function (Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
}();

function _possibleConstructorReturn(self, call) {
    if (!self) {
        throw new ReferenceError("this hasn't been initialised - super() hasn't been called");
    }
    return call && (typeof call === "object" || typeof call === "function") ? call : self;
}

function _inherits(subClass, superClass) {//这个方法的目的就是让subCLass获得superClass的静态属性和原型属性
    if (typeof superClass !== "function" && superClass !== null) {
        throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);
    }
    subClass.prototype = Object.create(superClass && superClass.prototype, { constructor: { value: subClass, enumerable: false, writable: true, configurable: true } });
    //这里设置了Child的prototype指向一个新创建的对象，并且新对象的__proto__指向superClass的prototype，并且设置了constructor的属性描述符，不可枚举，constructor键名的值是subClass
    //一句话说就是  subClass.prototype.__proto__ === superClass.prototype,也就是
    if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;
    //Object.setPrototypeOf是es6新增的方法，其实也就是 subClass.__proto__ = superClass
}


function _classCallCheck(instance, Constructor) {//保证调用方式
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

var Parent = function () {
    function Parent(name) {
        _classCallCheck(this, Parent);//保证调用方式是通过new关键字

        this.name = name;
    }

    //将静态属性和原型属性挂载在Parent上
    _createClass(Parent, [{
        key: 'getName',
        value: function getName() {
            console.log(this.name);
        }
    }], [{
        key: 'getAge',
        value: function getAge() {//私有属性
        }
    }]);

    return Parent;
}();

var Child = function (_Parent) {
    _inherits(Child, _Parent);//将_Parent的原型属性和静态属性挂载在Child.prototype.__proto__上和Child.__ptoto__上
    //也就是说调用之后 Child.prototype.__proto__ = { getName:function(){} }
    //Child.__proto__ = _Parent  
    function Child(name, age) {
        _classCallCheck(this, Child);//保证调用的方式是通过new关键字

        var _this = _possibleConstructorReturn(this, (Child.__proto__ || Object.getPrototypeOf(Child)).call(this, name));//其实就是获得Child的__proto__

        _this.age = age;
        return _this;
    }

    //将静态属性和原型属性挂载在Child上
    _createClass(Child, [{
        key: 'getAge',
        value: function getAge() {
            console.log(this.age);
        }
    }]);

    return Child;
}(Parent);

var child = new Child('qinhanwen', 25);
child.getName(); //qinhanwen
```

总的来说这里做了哪些事情
1.首先将Parent的静态属性和原型属性绑定到Parent上
2.将Parent的静态属性和原型属性绑定到`Child.__proto__`上和`Child.prototype.__proto__`上
3.将静态属性和原型属性绑定在Child上



看一下child实例，如图：

![WX20190219-232039@2x](http://www.qinhanwen.xyz/images/WX20190219-232039@2x.png)



那么温习一下，es5来实现一个简易的继承吧

```javascript
function Parent(name){
    this.name = name;
    this.getAge = function(){ 
    }
}
Parent.prototype.getName = function(){
    console.log(this.name);	
}

function Child(name,age){
    this.name = name;
    this.age = age;
}

Child.prototype = Object.create(Parent.prototype,{
    constructor: {
        value: Child,
        enumerable: false,
        writable: true,
        configurable: true
    }
});
Child.__proto__ = Parent;

Child.prototype.getAge = function(){
    console.log(this.age);
}

var child = new Child('qinhanwen', 25);
child.getName(); //qinhanwen
child.getAge();//25
console.log(child);
```

child实例如图：![WX20190220-223013@2x](http://www.qinhanwen.xyz/images/WX20190220-223013@2x.png)

与上面的打印出的实例有一点差异，这里暂时不深入了。





#### 8.[生成器（Generator)](https://qinhanwen.github.io/2019/03/05/Generator/#more)与[迭代器（iterator）](https://qinhanwen.github.io/2019/03/06/Iterator/)

##### 迭代器

```javascript
let books = ['JavaScript','Node'];
function read(books){
    let index = 0;
    return {
        next(){
            return index<books.length?{value:books[index++],done:false}:{value:undefined,done:true}
        },
    }
}
let it = read(books);
let it1 = it.next();
console.log(it1);//{ value: 'JavaScript', done: false }
let it2 = it.next();
console.log(it2);//{ value: 'Node', done: false }
let it3 = it.next();
console.log(it3);//{ value: undefined, done: true }
```

可以看出来迭代器可以不停的调用next方法，会得到{value,done}的结果，当done为true的时候，遍历结束。



##### 生成器函数

声明方式：`function`关键词与函数名中有一个`*`号

```javascript
function * test(){}
function *test(){}
function* test(){}
```



使用方式：碰到`yield`表达式，便会停止执行，必须调用遍历器对象的`next`方法，使得指针移向下一个状态。也就是说，每次调用`next`方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个`yield`表达式（或`return`语句）为止。

```javascript
let books = ['JavaScript','Node'];
function * read(books) {
    for (let i = 0; i < books.length; i++) { 
        yield books[i];
    }
}
let it = read(books);
let it1 = it.next();
console.log(it1);//{ value: 'JavaScript', done: false }
let it2 = it.next();
console.log(it2);//{ value: 'Node', done: false }
let it3 = it.next();
console.log(it3);//{ value: undefined, done: true }
```

也就是说，遇到`yield`是暂停的标志，`next`可以恢复执行。

##### 

#### 9.[Promise](https://qinhanwen.github.io/2018/11/12/Promise%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)





#### 10.Symbol

新的基础数据类型，表示独一无二的值，可以接受一个字符串作为参数

```javascript
let s = Symbol();
typeof s;//symbol
```



可以转成字符串或者布尔值

```javascript
let s = Symbol('booleanOrString');
let str = s.toString();
let bool = Boolean(s);
console.log(str);//Symbol(booleanOrString)
console.log(bool);//true
```



作为属性名来用，可以避免键名被覆盖

```javascript
let s = Symbol('booleanOrString');
let obj = {
    [s]:1
}
console.log(obj[s]);//1
//不可以使用点运算符读取
```



for…in ， Object.getOwnPropertyName，以及Object.keys都无法获得键名，因为它不是私有属性，要使用Object.getOwnPropertySymbols方法

```javascript
let s = Symbol('booleanOrString');
let obj = {
    [s]:1,
    a:2
}
for(i in obj){
    console.log(i)//a
}
let arr = Object.keys(obj);
console.log(arr);//[ 'a' ]

let arr1 = Object.getOwnPropertyNames(obj);
console.log(arr1);//[ 'a' ]

let arr2 = Object.getOwnPropertySymbols(obj);
console.log(arr2);//[ Symbol(booleanOrString) ]
```









#### 11.Set与Map

##### Set是一个构造函数，用于生成Set数据结构，类似于数组，但是成员的值都是唯一的。

```javascript
let arr = new Set([1,1,2,3,3,4,5,5]);
console.log(arr);
```

![WX20190228-132317@2x](http://www.qinhanwen.xyz/images/WX20190228-132317@2x.png)

实例的`__proto__`上的方法和属性。



也可以使用add方法

```javascript
let set = new Set();
[1,1,2,3,3,4,5,5].forEach((value)=> set.add(value));
console.log(set);
```

![WX20190228-132723@2x](http://www.qinhanwen.xyz/images/WX20190228-132723@2x.png)



可是使用它来去重

```javascript
var arr = [...new Set([1,1,2,3,3,4,5,5])];
console.log(arr);// [1, 2, 3, 4, 5]

var string = [...new Set('abcabc')].join('');
console.log(string);//abc
```



实例的`__proto__`属性上的方法：

- `add(value)`：添加某个值，返回 Set 结构本身。
- `delete(value)`：删除某个值，返回一个布尔值，表示删除是否成功。
- `has(value)`：返回一个布尔值，表示该值是否为`Set`的成员。
- `clear()`：清除所有成员，没有返回值。

- `keys()`：返回键名的遍历器
- `values()`：返回键值的遍历器
- `entries()`：返回键值对的遍历器
- `forEach()`：使用回调函数遍历每个成员

```javascript
let s = new Set();
s.add("red").add("yellow").add("blue");
console.log(s);//Set(3) {"red", "yellow", "blue"}

s.delete("red");
console.log(s);//Set(2) {"yellow", "blue"}

console.log(s.has("yellow"));//true

s.clear();
console.log(s);//Set(0) {}

s.add("red").add("yellow").add("blue");
console.log(s.keys());//console.log(s);//Set(0) {}

console.log(s.values());//console.log(s);//Set(0) {}

console.log(s.entries());//console.log(s);//Set(0) {}

s.forEach((value,key)=>console.log(value,key));
//red red 
//yellow yellow
//blue blue
```





##### Map

类似于对象，也是键值对的集合，但是它`各种类型的数据`都可以当做键名

```javascript
var obj = {}
var obj1 = new Object({[obj]:1});
var obj2 = new Map();
obj2.set(obj,1);
console.log(obj1['[object Object]']);//1
console.log(obj2.get(obj));//1
```

对象会把键名调toString方法



```javascript
var m = new Map();
m.set('name','qinhanwen');
m.set('age','25');

m.has('name');//true

m.get('name');//qinhanwen

m.delete('name');
console.log(m);//Map(1) {"age" => "25"}

m.clear();
console.log(m);//Map(0) {}
```







#### 总结

不要问我为什么有的地方用var有的地方用let，调试方便。。













