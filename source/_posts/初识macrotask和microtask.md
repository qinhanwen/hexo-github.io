---
title: 事件循环
date: 2018-12-20 18:44:49
tags: 
- 深入学习
categories: 
- 深入学习
---

# 事件循环（Event Loop）入门

### 这里只说浏览器环境下



#### 1.单线程的JavaScript

负责解析和执行JavaScript的线程只有一个，同一时间只能执行一段JavaScript代码。JavaScript设计的时候就是一种浏览器脚本，负责与用户进行交互，主要用来操作DOM，如果是多线程的，同时操作同个DOM就不合适了。



#### 2.[调用栈](https://medium.freecodecamp.org/understanding-the-javascript-call-stack-861e41ae61d4)（call stack）

当调用函数时，其参数和变量被推入调用堆栈以形成堆栈帧。 此堆栈帧是堆栈中的内存位置。最后一个被推入堆栈的函数是第一个被弹出的函数， 当所有函数从堆栈中弹出时，内存被清除。

![](http://39.105.62.145/assets/images/WX20181224-231326@2x.png)

###### 举个🌰，调用栈是怎样处理函数调用的：

```javascript
function firstFunction(){
 console.log("Hello from firstFunction");
}
function secondFunction(){
 firstFunction();
 console.log("The end from secondFunction");
}
secondFunction();
```

1）执行secondFunction调用，创建一个空堆栈帧。

2）之后firstFunction被调用，firstFunction被压入调用栈中。

3）返回打印出 Hello from firstFunction之后， firstFunction 被弹出栈。

4）返回打印 The end from secondFunction之后，secondFunction被弹出栈，清除内存。



#### 3.任务队列

![](http://39.105.62.145/assets/images/WX20181224-232444@2x.png)

##### 1）任务分为两种：

同步任务：同步任务在主线程中排队执行的任务。

异步任务：异步任务不进入主线程，异步任务有结果的后将具体任务放入任务队列（task queue）。只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。



##### 2）运行机制（我的理解）

1.同步任务在主线程执行，形成一个`执行栈`。

2.异步任务，有结果的时候，将具体任务放入任务队列。

3.主线程中所有同步任务执行完毕，就会从任务队列中取出任务执行。

4.主线程不断重复上面步骤。



##### 3）任务队列分类

1.macro-task（宏任务）：setTimeout，setInterval，I/O，UI Rendering

2.micro-task（微任务）：Promise，[MutationObserver](https://qinhanwen.github.io/2018/11/23/VDOM-%E5%92%8C-render/)



举个🌰，问输出顺序

```javascript
console.log(1);
setTimeout(function(){
    console.log(2);
},0);
setTimeout((function(){
    console.log(3);
})(),0);
console.log(4);
new Promise(function(resolve,reject){
    console.log(5);
    resolve();
}).then(function(){
    console.log(6)
});
console.log(7);
//1 3 4 5 7 6 2
```

（ps：这里顺序加入了一个setTImeout第一个参数，因为传入的是一个函数立即调用，所以视为`setTimeout(undefined,...)`）

1）首先打印出1

2）发现`setTimeout`是 webApi，0秒后将具体的任务放入Macro-task

3）第二个`setTImeout`里的函数调用立刻执行了，打印出3

4）接着打印出4

5）new Promise是同步的调用，打印出5，then方法里的具体任务放入Micro-task

6）打印出7

8）主线程任务空了，首先从Micro-task中取任务执行，打印6

9）主线程和Micro-task中都无任务，从Macro-task中取任务，打印2。



如果写了一个死循环，那么主线程不空的时候，永远不会执行Macro-task和Micro-task里的任务

```javascript
setTimeout(function(){
    console.log('不输出')
},0)
for(i=0;i<1;i--)
{
    console.log('死循环')
}
//我运行了一下风扇狂飙然后重启电脑。。。要谨慎尝试
```



#### 总结

可能理解有偏差，如果发现有理解错误再回来改。

