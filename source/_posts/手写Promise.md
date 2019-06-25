---
title: 手写Promise
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

上面的都还比较好理解。。直到链式调用，简直脑袋打结。

上面的then方法调用之后就再也无法调用then方法了，我必须在调用then方法之后返回一个带有then方法的对象的新的实例，与之前那个的无关了。

实现下面🌰

#### 先写 我 ! 自! 己 ! 的 ! 想 ! 法 ! ：

```javascript
new Promise(function (resolve, reject) {
    console.log(1);
    setTimeout(function () {
        resolve('resolve');
    }, 2000);
})
.then(function (data) { console.log(data); })
.then(function (data) { console.log(data); })
```



##### `.then`需要返回一个新的`promise`，我一直在想执行器里面要放什么。首先，`.then`方法里放的是成功和失败的回调，那么我得知道成功了还是失败了。

拆分一下一步一步写。。脑子不太好用怕自己搞晕。

###### 1) 那么我的第一步，把判断放进新的promise的执行器里。这样`status`变化成了`resolved`或者`rejected`，那么会我立刻知道，然后可以立刻执行回调。执行回调的同时调用`reject`或者`resolve`，才能让下一个`.then`里的回调执行（说的好绕，反正就是`resolve`或者`reject`调用之后，会调用`.then`里的回调，然后循环）

```javascript
MyPromise.prototype.then = function (onFulfilled, onRejected) {
    const self = this;
    return new MyPromise(function (resolve, reject) {
        if (self.status == 'resolved') {
            onFulfilled(self.value);
            resolve();//还没写参数
        }
    
        if (self.status == 'rejected') {
            onRejected(self.value);
            reject();//还没写参数
        }
    
        if (self.status == 'pending') {
            self.successCB.push(function () {
                onFulfilled(self.value);
          	    resolve();//还没写参数
            })
            self.errorCB.push(function () {
                onRejected(self.value);
                reject();//还没写参数
            })
        }
    })
}
```



###### 2）第二步，`resolve`和`reject`方法里的填什么参数？？ 我们知道，如果`.then`方法里接受到的不是函数，就视为null，会把上一次传递的值继续往下传递。如果接受到的是函数，就返回函数的返回值（没有返回值就视为`undefined`），所以修改了一下上面的代码。

```javascript
MyPromise.prototype.then = function (onFulfilled, onRejected) {
    const self = this;
    return new MyPromise(function (resolve, reject) {
        if (self.status == 'resolved') {
            if (typeof onFulfilled == "function") {
                resolve(onFulfilled(self.value));
            } else {
                resolve(self.value);
            }
        }

        if (self.status == 'rejected') {
            if (typeof onFulfilled == "function") {
                reject(onRejected(self.value));
            } else {
                reject(self.value);
            }
        }

        if (self.status == 'pending') {
            self.successCB.push(function () {
                if (typeof onFulfilled == "function") {
                  	let result = onFulfilled(self.value);
                  	if(result instanceof MyPromise){
                       
                    }else{
                   		 resolve(result);
                    }
                } else {
                    resolve(self.value);
                }
            })
            self.errorCB.push(function () {
                if (typeof onFulfilled == "function") {
                    reject(onRejected(self.value));
                } else {
                    reject(self.value);
                }
            })
        }
    })
}
```



##### 我们稍微写几个🌰试试输出结果

第一个🌰，`.then`方法里不是个函数

```javascript
new MyPromise(function (resolve, reject) {
    console.log(1);
    setTimeout(function () {
        resolve('resolve');
    }, 2000);
})
.then(1)
.then(function (data) { console.log(data); })
//打印出
//1
//resolve
```



第二个🌰,`.then`方法是个函数，但是没有返回值

```javascript
new Promise(function (resolve, reject) {
    console.log(1);
    setTimeout(function () {
        resolve('resolve');
    }, 2000);
})
.then(function (data) { console.log(data);})
.then(function (data) { console.log(data);})
//打印出
//1
//resolve
//undefined
```



第三个🌰,`.then`方法是个函数，并且有返回值

```javascript
new Promise(function (resolve, reject) {
    console.log(1);
    setTimeout(function () {
        resolve('resolve');
    }, 2000);
})
.then(function (data) { console.log(data);return '有返回值';})
.then(function (data) { console.log(data);});
//打印出
//1
//resolve
//有返回值
```



emm...其实这个的调用过程真的很绕，因为有一堆的`self`，于是我决定画个图，就拿上面第一个🌰的调用过程来画吧

![59EF3428467C4B9F81D0CA9C20DBE35C](http://www.qinhanwen.xyz/59EF3428467C4B9F81D0CA9C20DBE35C.png)



如果`.then`里是函数并且有返回值，也就是黄色箭头的那个数组变成

`[function(){resolve(onFulfilled(self.value))}]`，就会让第二个promise对象的value变成onFulfilled(self.value)的返回值。





#### 实现的then发现一个问题

1、如果 `onFulfilled` 或者 `onRejected` 返回一个值 `x` ，则运行下面的 `Promise` 解决过程：`[[Resolve]](promise2, x)`

- 若 `x` 不为 `Promise` ，则使 `x` 直接作为新返回的 `Promise` 对象的值， 即新的`onFulfilled` 或者 `onRejected` 函数的参数.
- 若 `x` 为 `Promise` ，这时后一个回调函数，就会等待该 `Promise` 对象(即 `x` )的状态发生变化，才会被调用，并且新的 `Promise` 状态和 `x` 的状态相同。



就是说如果then方法里面的回调返回值为Promise，则要等它的状态变为`resolved`或者`rejected`，才会执行之后的回调，举个例子

```javascript
new Promise((resolve)=>{
    setTimeout(()=>{
        resolve();
    },1000)
}).then((data)=>{
    return new Promise((resolve)=>{
            setTimeout(()=>{
                resolve(2000);
            },2000)
        })
}).then((data)=>{
    console.log(data);
})
//3秒后打印出
//2000
```

而

```javascript
new MyPromise((resolve)=>{
    setTimeout(()=>{
        resolve();
    },1000)
}).then((data)=>{
    return new MyPromise((resolve)=>{
            setTimeout(()=>{
                resolve(2000);
            },2000)
        })
}).then((data)=>{
    console.log(data);
})
//1秒后打印出
//MyPromise对象
```



修改一下then方法

```javascript
MyPromise.prototype.then = function (onFulfilled, onRejected) {
    const self = this;
    return new MyPromise(function (resolve, reject) {
        if (self.status == 'resolved') {
            if (typeof onFulfilled == "function") {
              	//新增部分
                let result = onFulfilled(self.value);
                if (result instanceof MyPromise) {
                    result.then(resolve,reject);
                } else {
                    resolve(result);
                }
            } else {
                resolve(self.value);
            }
        }

        if (self.status == 'rejected') {
            if (typeof onFulfilled == "function") {
              	//新增部分
                let result = reject(onRejected(self.value));
                if(result instanceof MyPromise){
                    result.then(resolve,reject);
                }else{
                    reject(onRejected(self.value));
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
                        result.then(resolve,reject);
                    } else {
                        resolve(result);
                    }
                } else {
                    resolve(self.value);
                }
            })
            self.errorCB.push(function () {
                if (typeof onFulfilled == "function") {
	                  //新增部分
                    let result = reject(onRejected(self.value));
                    if(result instanceof MyPromise){
                        result.then(resolve,reject);
                    }else{
                        reject(onRejected(self.value));
                    }
                } else {
                    reject(self.value);
                }
            })
        }
    })
}
```

新增的部分其实就是`onFulfilled`或者`onRejected`调用返回的是一个`Promise`，则当这个`Promise`的状态变成`resolved`或者`reject`的时候，再去执行之后的回调。





























