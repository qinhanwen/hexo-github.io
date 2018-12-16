---
title: 手写Promise
date: 2018-12-04 20:11:07
tags: 
- es6
categories: 
- es6
---



# 手写[Promise](https://qinhanwen.github.io/2018/11/12/Promise%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)

> 参考资料：https://juejin.im/post/5b5d0ac5f265da0f574df709



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
function Promise(exector) {
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
new Promise(function (resolve, reject) {
    console.log(1);
    resolve('resolve');
})
//打印出
//1
//resolved
```



#### 2.实现Promise.then

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
Promise.prototype.then = function(onFulfilled,onRejected){
     if(this.status == 'resolved') {//这里this指向Promise
         onFulfilled(this.value);
     }
     
     if(this.status == 'rejected') {
         onRejected(this.value);
     }
 }
//调用
new Promise(function (resolve, reject) {
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
Promise = {
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
function Promise(exector) {
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

Promise.prototype.then = function (onFulfilled, onRejected) {
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

new Promise(function (resolve, reject) {
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

上面的then方法调用之后就再也无法调用then方法了，我必须在调用then方法之后返回一个带有then方法的对象的新的实例，与之前那个的无关了。

实现下面🌰

```javascript
new Promise(function (resolve, reject) {
    console.log(1);
    setTimeout(function () {
        resolve('resolve');
    }, 2000);
})
.then(function (data) { console.log(data);return 'resolve'; }, function (err) { console.log(err) })
.then(function (data) { console.log(data) }, function (err) { console.log(err) });
```



我们又知道then方法里如果接收到的不是函数，那么将视为null，并把上一个传递下来的值，继续往下传递。修改一下上面的then方法



emm..写到这感觉知识不够写不下去了,看的资料理解不了。后面再补上





























