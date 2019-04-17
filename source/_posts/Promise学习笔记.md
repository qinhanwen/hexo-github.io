---
title: Promise学习笔记
date: 2018-11-12 22:19:12
tags: 
- es6
categories: 
- es6
---

# Promise学习笔记

##### 1.Promise适用场景

通常调用某些异步，都是传入回调函数，等待异步执行完之后再调用回调函数。可是当回调过多的时候就不方便阅读与维护

Promise链式调用的风格，书写方式都让程序变得更好维护



##### 2.Promise

######  1.它有三种状态

- pending：进行中
- fulfilled :已经成功
- rejected 已经失败

###### 2.状态改变

- 从pending变为fulfilled
- 从pending变为rejected。

   状态被改变之后就不能再改变，无法逆向转变

###### 3.then方法

then方法接受两个参数，第一个参数是成功时的回调，在Promise由“等待”态转换到“完成”态时调用，另一个是失败时的回调，在Promise由“等待”态转换到“拒绝”态时调用

![](http://www.qinhanwen.xyz/WX20181116-172309@2x.png)



##### 3.Promise Methods

######  1. Promise.all(iterable)   //将多个Promise包装成一个Promise适合可以处理多个异步事件，有成功和失败两种情况

```javascript
//成功的时候
let p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success')
  },2000)
})

let p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success1')
  },1000)
})

Promise.all([p1,p2]).then((result) => {
    console.log(result)   //[ 'success', 'success1' ]
}).catch((error) => {
    console.log(error)      
})
//成功的时候会返回一个成功的数组。返回成功数组的顺序与Promise.all接收的数组顺序一定是一致的。与异步操作返回结果先后顺序无关
```

```javascript
let p1 = new Promise((resolve, reject) => {
    resolve('成功了')
})

let p2 = Promise.reject('失败');

let p3 = Promise.reject('失败1');

Promise.all([p1,p2,p3]).then((result) => {
    console.log(result)
}).catch((error) => {
    console.log(error)      // 失败
})
//失败的时候，返回第一个失败的值
```



###### 2.Promise.race(iterable)

```javascript
let p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('success')
    },2000)
})

let p2 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('success1')
    },1000)
})

let p3 = Promise.reject('failed');

Promise.race([p1,p2]).then((result) => {
    console.log(result)   //failed
}).catch((error) => {
    console.log(error)      
})
//返回最快的结果,无论成功与失败
```

##### 

###### 3 . Promise.resolve(value)

```javascript
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```



###### 4 . Promise.reject()   //生成错误的一个promise对象

```javascript
Promise.reject(new Error("error"));

//在一个promise链中，只要任何一个promise被reject，promise链就被破坏了，reject之后的promise都不会再执行，而是直接调用.catch方法

```



###### 一般情况下我们都会使用new Promise()来创建promise对象，但是我们也可以使用promise.resolve 和 promise.reject这两个方法



##### 4.Promise prototype

######  1.Promise.prototype.catch(onRejected)

```javascript
function p1(){
    return new Promise((resolve,reject)=>{
        console.log('p1 resolve');
        resolve('resolve');
    })
}

function p2(){
    return new Promise((resolve,reject)=>{
        console.log('p2 reject');
        reject('reject');
    })
}

function p3(){
    return new Promise((resolve,reject)=>{
        console.log('p3 resolve');
        resolve('resolve1');
    })
}
p1().then(p2).then(p3).catch(err=>{
    console.log(err);
});
//输出结果
//p1 resolve
//p2 reject
//reject
//primise链中任何地方被reject，就直接调用catch，之后的都不执行了
```



###### 2.Promise.prototype.then(onFulfilled, onRejected)

```javascript
//then 方法接受两个参数：onFulfilled 和 onRejected 都是可选参数
//如果 onFulfilled 不是函数，其必须被忽略
//如果 onFulfilled 是函数：当 promise 执行结束后其必须被调用，其第一个参数为 promise 的终值，在 promise 执行结束前其不可被调用

//总的来说!!!!!!如果传入的不是函数或者没有返回值就忽略当前的then，会使用上一个then的结果。
//我们需要关心的只是
//1.上一个then中传入了回调函数吗？
//2.上一个then中提供了返回值吗?
//3.若上一个then中若提供了返回值，返回了什么？
```

![](http://www.qinhanwen.xyz/WX20190111-172541@2x.png)

如果没有返回值，那就是undefined。

```javascript
let func = function() {
    return new Promise((resolve, reject) => {
        resolve('返回值');
    });
};

let cb = function() {
    return '新的值';
}

func().then(function () {
    return cb();
}).then(resp => {
    console.warn(resp);
    console.warn('1 =========<');
});

func().then(function () {
    cb();
}).then(resp => {
    console.warn(resp);
    console.warn('2 =========<');
});

func().then(cb()).then(resp => {
    console.warn(resp);
    console.warn('3 =========<');
});

func().then(cb).then(resp => {
    console.warn(resp);
    console.warn('4 =========<');
});
//新的值
//1 =========<

//undefined
//2 =========<

//返回值
//3 =========<

//新的值
//4 =========<
```

[传送门](https://segmentfault.com/a/1190000010420744)



##### 5.创建一个Promise

```javascript
const myFirstPromise = new Promise((resolve, reject) => {
  // do something asynchronous which eventually calls either:
  //
  //   resolve(someValue); // fulfilled
  // or
  //   reject("failure reason"); // rejected
});
```

使用关键字new创建Promise对象，构造函数接受一个函数作为参数，这个函数又接受两个函数作为参数(resolve与reject)。

resolve在异步操作成功的时候调用，并将异步操作的结果作为参数传出。

reject在异步操作失败的时候调用，将失败结果传出



##### 6.Promise的面试题

[传送门](https://juejin.im/post/5bd697cfe51d454c791cd1d5)

































