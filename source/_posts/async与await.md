---
title: async与await
date: 2019-02-18 12:39:59
tags: 
- es7
categories: 
- es7
---

在了解了[事件循环](https://qinhanwen.github.io/2018/12/20/%E5%88%9D%E8%AF%86macrotask%E5%92%8Cmicrotask/)与[Promise](https://qinhanwen.github.io/2018/11/12/Promise%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)之后，觉得有必要了解一下async与await。

https://juejin.im/post/5c0dcf70518825765548502b

### 描述

​	当调用一个 async 函数时，会返回一个 Promise 对象。当这个 async 函数返回一个值时，Promise 的 resolve 方法会负责传递这个值；当 async 函数抛出异常时，Promise 的 reject 方法也会传递这个异常值。
async 函数中可能会有 await 表达式，这会使 async 函数暂停执行，等待 Promise  的结果出来，然后恢复async函数的执行并返回解析值（resolved）。
​	async/await的用途是简化使用 promises 异步调用的操作，并对一组 Promises执行某些操作。正如Promises类似于结构化回调，async/await类似于组合生成器和 promises。

​	await  操作符用于等待一个Promise 对象。它只能在异步函数 async function 中使用。
[return_value] = await expression;
​	await 表达式会暂停当前 async function 的执行，等待 Promise 处理完成。若 Promise 正常处理(fulfilled)，其回调的resolve函数参数作为 await 表达式的值，继续执行 async function。
若 Promise 处理异常(rejected)，await 表达式会把 Promise 的异常原因抛出。
另外，如果 await 操作符后的表达式的值不是一个 Promise，则返回该值本身。



### async

#### 定义一个异步函数

```javascript
async function getName(){
    return 'qinhanwen'
}
```



#### 返回值

```javascript
async function getName(){
    return 'qinhanwen'
}
let name = getName();
console.log(name);
name.then(function(data){console.log(data)})//qinhanwen
```

打印的name如图：

![WX20190220-181141@2x](http://www.qinhanwen.xyz/WX20190220-181141@2x.png)

可以看到返回的是一个Promise对象（async函数默认返回就是一个Promise对象），它的`status`是`resolved`，`value`是`qinhanwen`。当`status`为`resolved`的时候，`value`是会往下传递的，下一个`.then(onfullfiled)`的中的`onfullfiled`的参数。



如果异步函数体内报错，则会返回一个`status`为`rejected`的Promise对象

```javascript
async function getName(){
   	throw new Error();
}
let name = getName();
console.log(name);
name.catch(reason => console.log(reason));
	//Error
    //at getName (VM25244 Script snippet %232:2)
	//at VM25244 Script snippet %232:6
```

打印的name如图：

![WX20190220-182856@2x](http://www.qinhanwen.xyz/WX20190220-182856@2x.png)

抛出的错误会被catch方法的回调接收到



### await

1.await只能在async函数内使用

2.所有的await的函数执行完之后，才能知道async函数返回的Promise对象的`status`是`resolved`还是`rejected`

3.如果await之后跟随的如果不是一个promise，会被转换成一个立即resolve的promise



### 实现一个简单的async函数

```javascript
function spawn(genF){
    return new Promise((resolve,reject)=>{
        const gen = genF();
        function step(){
            try{
               let next = gen.next();   
            }catch(e){
                reject(e);
            }
            if(next.done){
               return resolve(next.value);
            }
            Promise.resolve(next.value).then(function(){
                step();
            })
        }
        step();
    })
}

function *gen(){
    yield new Promise(resolve=>{
        console.log(1);
        setTimeout(resolve,2000)
    })
    yield new Promise(resolve=>{
        console.log(2);
        setTimeout(resolve,2000)
    })
    console.log('finished');
}
spawn(gen);
```









