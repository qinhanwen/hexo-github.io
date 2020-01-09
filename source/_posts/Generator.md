---
title: Generator
date: 2019-03-05 20:27:48
tags: 
- es6
categories: 
- es6
---

## Generator

> 参考资料	https://juejin.im/post/5a6db41351882573351a8d72

#### 概念

##### 1）函数声明方式：在function关键字与函数名之间有个 * 号

##### 2）函数体内：可以使用yield表达式

##### 3）执行时：返回一个iterator对象，可以用过调用next方法，遍历iterator对象里的每个yield表达式定义的状态





#### yield表达式

```javascript
function *test(){
    console.log(1);
    yield 1
    yield 2
    return 3
}
let g = test();
```

并没有打印出 1 



```javascript
function *test(){
    yield 1
    console.log(2);
    yield 2
    return 3
}
let g = test();
console.log(g.next());//{ value: 1, done: false }
```

未打印出  2



```javascript
function *test(){
    yield 1
    yield 2
    return 3
}
let g = test();
console.log(g.next());//{ value: 1, done: false }
console.log(g.next());//{ value: 2, done: false }
console.log(g.next());//{ value: 3, done: true }
```





**yield表达式特点**：

1）调用Generator函数并不会马上执行，而是返回一个Iterator对象。

2）在Generator函数体内，yield可以有多个，并且yield会暂停函数执行。

3）通过yield定义内部状态





#### next方法

通过上面的了解，调用Generator函数会返回一个Iterator对象，而不会马上执行函数。



1）调用Iterator对象的next方法，就会从函数体头部开始执行，到第一个yield后结束。再次调用next方法的时候会从中断的地方继续往下执行。

2）调用next方法会返回一个包含value和done的对象，value的值就是yield后面跟随的值。

3）调用next之后会一直往下执行，如果遇到yield就被暂停，如果遇到return，就将return后面跟的值作为返回对象的value的值。

4）如果没有return语句，那么返回的值是undefined。



```javascript
function *test(){
}
let g = test();
console.log(g.next());//{ value: undefined, done: true }
```

```javascript
function *test(){
    yield 1
    return 3
}
let g = test();
console.log(g.next());//{ value: 1, done: false }
console.log(g.next());//{ value: 3, done: true }
console.log(g.next());//{ value: undefined, done: true }
```

```javascript
function *test(){
    var a = yield 1
    console.log(a);//undefined
    return a
}
let g = test();
console.log(g.next());//{ value: 1, done: false }
console.log(g.next());//{ value: undefined, done: true }
```

**从上面得知，返回值会有3种，分别是**：

1）yield后的值

2）return的值

3）undefined

并且yield是没有返回值的





#### next方法的参数

而next方法传递的参数，该参数会被当成yield的返回值

```javascript
function *test(){
    var a = yield 1
    console.log(a);//3
    var b = yield 2;
    console.log(b);//4
    return a+b;//7
}
let g = test();
console.log(g.next(2));//{ value: 1, done: false }
console.log(g.next(3));//{ value: 2, done: false }
console.log(g.next(4));//{ value: 7, done: true }
```

**通过对上面的调用的观察**：

1）第一次调用next传入的参数是无效的。

2）next方法的参数被当成上一个yield的返回值。

3）使用next方法遍历Iterator里的所有状态的次数，会比yield的个数多一次，因为第一次是启动。





#### yield*

在一个Generator函数里调用另外一个Generator函数

```javascript
function *test(){
    yield 1
    return 2
}
function *test1(){
    test();
    yield 3
    return 4
}
let g = test1();
console.log(g.next());//{ value: 3, done: false }
console.log(g.next());//{ value: 4, done: true }
console.log(g.next());//{ value: undefined, done: true }
```

发现并无卵用，没能在`yield 1`的地方停止执行



使用`yield*`

```javascript
function *test(){
    yield 1
    return 2
}
function *test1(){
    yield* test();
    yield 3
    return 4
}
let g = test1();
console.log(g.next());//{ value: 1, done: false }
console.log(g.next());//{ value: 3, done: false }
console.log(g.next());//{ value: 4, done: true }
```



那么问题来了，怎么取得test函数里的返回值

```javascript
function *test(){
    yield 1
    return 2
}
function *test1(){
    let num = yield* test();
    console.log(num);//2
    yield 3
    return 4
}
let g = test1();
console.log(g.next());//{ value: 1, done: false }
console.log(g.next());//{ value: 3, done: false }
console.log(g.next());//{ value: 4, done: true }
```



`yield*`与`for...of`

```javascript
function *test(){
    yield 1
    return 2
}
function *test1(){
    for(value of test()){
        yield value
    }
    yield 3
    return 4
}
let g = test1();
console.log(g.next());//{ value: 1, done: false }
console.log(g.next());//{ value: 3, done: false }
console.log(g.next());//{ value: 4, done: true }
```



再看`yield`与`yield*`

```javascript
function *test(){
    yield 1
    return 2
}
function *test1(){
    yield test();
    yield 3
    return 4
}
let g = test1();
console.log(g.next());//{ value: {}, done: false }
console.log(g.next());//{ value: 3, done: false }
console.log(g.next());//{ value: 4, done: true }
```

可以看到`yield test()`返回的是一个Iterator对象



另外，只要有Iterator的数据类型都可以使用`yield*`

```javascript
var arr = [1,2,3];
function *test1(){
    yield* arr;
    yield 3
    return 4
}
let g = test1();
console.log(g.next());//{ value: 1, done: false }
console.log(g.next());//{ value: 2, done: false }
console.log(g.next());//{ value: 3, done: false }
console.log(g.next());//{ value: 3, done: false }
console.log(g.next());//{ value: 4, done: true }
```



另外可以使用`yield*`，将[多维数组扁平化](https://qinhanwen.github.io/2019/03/03/%E6%AF%8F%E6%97%A5%E4%B8%80%E9%81%93%E7%AE%97%E6%B3%95%E9%A2%98/#more)





#### Generator函数与Iterator

```javascript
var obj = {
    name: 'qinhanwen',
    age: '25'
}
for(let key of obj){
    
}
//报错obj[Symbol.iterator] is not a function
```

因为对象是没有`Iterator`接口的，可以将Generator函数赋值给对象的`Symbol.iterator`，这样就让对象获得了Iterator接口。

```javascript
var obj = {
    name: 'qinhanwen',
    age: '25'
}
function *test(){
    const arr = Object.keys(this)
    for (let item of arr) {
      yield [item, this[item]]
    }
}
obj[Symbol.iterator] = test;

for(let [key,value] of obj){
    console.log(key,value);
}
//name qinhanwen
//age 25
```



稍微看一下具有`Iterator`接口的数据类型

```javascript
//数组
var arr = ['qinhanwen','25'];
for(let key of arr){
    console.log(key);
}
//qinhanwen
//25



//字符串
var str = 'qinhanwen';
for(let key of str){
    console.log(key);
}
//q
//i
//n
//h
//a
//n
//w
//e
//n
```



对有Iterator接口的数据类型，可以使用`展开运算符`，`解构赋值`，`for…of`，`Array.from`。[了解一下字符串反转](![image-20190306211217543](http://118.24.241.76/image-20190306211217543.png))



