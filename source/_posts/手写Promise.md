---
title: 实现Promise
date: 2018-12-04 20:11:07
tags: 
- es6
categories: 
- es6
---



# 手写[Promise](https://qinhanwen.github.io/2018/11/12/Promise%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)

> 参考资料：[掘金，手写Promise](https://juejin.im/post/5b83cb5ae51d4538cc3ec354)
>
> ​		    [阮一峰promise](http://es6.ruanyifeng.com/#docs/promise)
>
> ​		    [this的指向问题](https://qinhanwen.github.io/2018/12/05/this/)
>
> ​			





#### 1.先看个Promise例子

```javascript
new Promise(function (resolve, reject) {
    console.log(1);
    resolve('resolve');
})
```



##### 1） 看到Promise接收一个参数，并且有三种状态 pending，fulfilled，rejected

调用resolve   状态从pending  => fulfilled

调用reject      状态从pending => rejected

并且不可逆转



##### 2）先声明个promise函数,接受个一个函数,有个状态`pending`,resolve和reject方法都接受一个参数,并且要保存这个参数。并且有`pending`的状态改变

```javascript
function MyPromise(exector) {
    var self = this;
    self.status = 'pending';
    self.value = '';
    function resolve(value) {
        if (self.status == 'pending') {
            self.status = 'resolved';
            self.value = value;
            console.log('resolved');
        }
    }

    function reject(value) {
        if (self.status == 'pending') {
            self.status = 'rejected';
            self.value = value;
            console.log('rejected');
        }
    }
    exector(resolve, reject);
}

//然后调用
new MyPromise(function (resolve, reject) {
    console.log(1);
    resolve('resolve');
})
//打印出
//1
//resolved
```



#### 2.实现Promise.prototype.then

##### 1）先看then方法

```javascript
new Promise(function (resolve, reject) {
    console.log(1);
    resolve('resolve');
}).then(function(data){console.log(data)},function(err){console.log(err)})
//打印出
//1
//resolve
```



##### 2）then方法实现：它接受两个参数，onFulfilled和onRejected，两个方法都有接收参数

```javascript
MyPromise.prototype.then = function(onFulfilled,onRejected){
     if(this.status == 'resolved') {//这里this指向Promise
         onFulfilled(this.value);
     }
     
     if(this.status == 'rejected') {
         onRejected(this.value);
     }
 }
//调用
new MyPromise(function (resolve, reject) {
    console.log(1);
    resolve('resolve');
}).then(function(data){console.log(data)},function(err){console.log(err)});
//打印出
//1
//resolved
//resolve
```

这里调用的then方法相当于 Promise.then()的调用

```javascript
MyPromise = {
    then:function(){...},
    status:'',
    value:''
}
```

所以this指向Promise



#### 3.实现异步处理

##### 1）promise经常用来处理异步操作。

#####  添加两个数组，用来存放成功与失败的回调函数。

```javascript
function MyPromise(exector) {
    var self = this;
    self.status = 'pending';
    self.value = '';

    //存放成功或失败回调数组
    self.successCB = [];
    self.errorCB = [];

    function resolve(value) {
        if (self.status == 'pending') {
            self.status = 'resolved';
            self.value = value;
            console.log('resolved');
            self.successCB.forEach(function(cb){
                cb();
            })
        }
    }

    function reject(value) {
        if (self.status == 'pending') {
            self.status = 'rejected';
            self.value = value;
            console.log('rejected');
            self.errorCB.forEach(function(cb){
                cb();
            })
        }
    }
    exector(resolve, reject);
}

MyPromise.prototype.then = function (onFulfilled, onRejected) {
    const self = this;
    if (this.status == 'resolved') {
        onFulfilled(self.value);
    }

    if (this.status == 'rejected') {
        onRejected(self.value);
    }

    //如果是pending就先存起来函数调用
    if (this.status == 'pending') {
        this.successCB.push(function () {
            onFulfilled(self.value);
        })
        this.errorCB.push(function () {
            onRejected(self.value);
        })
    }
}

new MyPromise(function (resolve, reject) {
    console.log(1);
    setTimeout(function () {
        resolve('resolve');
    }, 2000);
}).then(function (data) { console.log(data) }, function (err) { console.log(err) });
//打印出
//1
//resolved
//resolve
```



#### 4.实现多个then的链式调用方法

1）

`Promise` 对象的 `then` 方法接受两个参数：

```javascript
promise.then(onFulfilled, onRejected)
```

`onFulfilled` 和 `onRejected` 都是可选参数。

- 如果 `onFulfilled` 或 `onRejected` 不是函数，其必须被忽略

**onFulfilled 特性**

    如果 `onFulfilled` 是函数：

- 当 `promise` 状态变为成功时必须被调用，其第一个参数为 `promise` 成功状态传入的值（ `resolve` 执行时传入的值）
- 在 `promise` 状态改变前其不可被调用
- 其调用次数不可超过一次

**onRejected 特性**

    如果 `onRejected` 是函数：

- 当 `promise` 状态变为失败时必须被调用，其第一个参数为 `promise` 失败状态传入的值（ `reject` 执行时传入的值）
- 在 `promise` 状态改变前其不可被调用
- 其调用次数不可超过一次

**多次调用**

    `then` 方法可以被同一个 `promise` 对象调用多次

- 当 `promise` 成功状态时，所有 `onFulfilled` 需按照其注册顺序依次回调
- 当 `promise` 失败状态时，所有 `onRejected` 需按照其注册顺序依次回调

**返回**

`then` 方法必须返回一个新的 `promise` 对象



2）

如果 `onFulfilled` 或者 `onRejected` 返回一个值 `x` ，则运行下面的 `Promise` 解决过程：`[[Resolve]](promise2, x)`

- 若 `x` 不为 `Promise` ，则使 `x` 直接作为新返回的 `Promise` 对象的值， 即新的`onFulfilled` 或者 `onRejected` 函数的参数.
- 若 `x` 为 `Promise` ，这时后一个回调函数，就会等待该 `Promise` 对象(即 `x` )的状态发生变化，才会被调用，并且新的 `Promise` 状态和 `x` 的状态相同。



```javascript
MyPromise.prototype.then = function (onFulfilled, onRejected) {
    const self = this;
    return new MyPromise(function (resolve, reject) {
        if (self.status == 'resolved') {
            if (typeof onFulfilled == "function") {
                //新增部分
                let result = onFulfilled(self.value);
                if (result instanceof MyPromise) {
                    result.then(resolve, reject);
                } else {
                    resolve(result);
                }
            } else {
                resolve(self.value);
            }
        }

        if (self.status == 'rejected') {
            if (typeof onRejected == "function") {
                //新增部分
                let result = onRejected(self.value);
                if (result instanceof MyPromise) {
                    result.then(resolve, reject);
                } else {
                    reject(result);
                }
            } else {
                reject(self.value);
            }
        }

        if (self.status == 'pending') {
            self.successCB.push(function () {
                if (typeof onFulfilled == "function") {
                    //新增部分
                    let result = onFulfilled(self.value);
                    if (result instanceof MyPromise) {
                        result.then(resolve, reject);
                    } else {
                        resolve(result);
                    }
                } else {
                    resolve(self.value);
                }
            })
            self.errorCB.push(function () {
                if (typeof onRejected == "function") {
                    //新增部分
                    let result = onRejected(self.value);
                    if (result instanceof MyPromise) {
                        result.then(resolve, reject);
                    } else {
                        reject(result);
                    }
                } else {
                    reject(self.value);
                }
            })
        }
    })
}
```



抽出一下重复部分

```javascript

```



























