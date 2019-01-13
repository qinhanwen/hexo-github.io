---
title: 事件循环
date: 2018-12-20 18:44:49
tags: 
- 深入学习
categories: 
- 深入学习
---

# 事件循环（Event Loop）入门

### 这里只说浏览器环境下,在不同浏览器下表现不同，这里是在**chrome!!!!**

> 参考资料		 https://www.codercto.com/a/18956.html
>
>  			 https://github.com/aooy/blog/issues/5
>
> ​			 [这个地址，可以看到具体的执行过程，还有详细的教程，中文字幕哈哈哈哈](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)

#### 1.单线程的JavaScript

负责解析和执行JavaScript的线程只有一个，同一时间只能执行一段JavaScript代码。JavaScript设计的时候就是一种浏览器脚本，负责与用户进行交互，主要用来操作DOM，如果是多线程的，同时操作同个DOM就不合适了。



#### 2.[调用栈](https://medium.freecodecamp.org/understanding-the-javascript-call-stack-861e41ae61d4)（call stack）

当调用函数时，其参数和变量被推入调用堆栈以形成堆栈帧。 此堆栈帧是堆栈中的内存位置。最后一个被推入堆栈的函数是第一个被弹出的函数， 当所有函数从堆栈中弹出时，内存被清除。

![](http://39.108.238.15:97/static/images/images/WX20181224-231326@2x.png)

###### 举个🌰，执行栈是怎样处理函数调用的：

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

1）执行secondFunction调用，创建一个执行栈，并且压入栈。

2）之后firstFunction被调用，firstFunction被压入执行栈中。

3）返回打印出 Hello from firstFunction之后， firstFunction 被弹出栈。

4）返回打印 The end from secondFunction之后，secondFunction被弹出栈，清除内存。



#### 3.webapis

是一种浏览器提供给我们的api，不在v8引擎中



#### 4.任务队列

![](http://39.108.238.15:97/static/images/images/WX20181224-232444@2x.png)

##### 1）任务分为两种：

同步任务：同步任务在主线程中排队执行的任务。

异步任务：异步任务不进入主线程，异步任务有结果的后将具体任务放入任务队列（task queue）。只有"任务队列"通知主线程，某个异步任务可以执行了，该任务才会进入主线程执行。



##### 2）运行机制（我的理解）

1.同步任务在主线程执行，形成一个`执行栈`。

2.异步任务，有结果的时候，将具体任务放入任务队列。

3.主线程中所有同步任务执行完毕，就会从任务队列中取出任务执行。

4.主线程不断重复上面步骤。



##### 3）任务队列分类

1.macro-task（宏任务）：script（整体代码)，setTimeout，setInterval，I/O，UI Rendering

2.micro-task（微任务）：Promise，[MutationObserver](https://qinhanwen.github.io/2018/11/23/VDOM-%E5%92%8C-render/)



##### 举个🌰，问输出顺序

```javascript
console.log(1);
setTimeout(function(){
    console.log(2);
},0);
console.log(3);
new Promise(function(resolve,reject){
    console.log(4);
    resolve();
}).then(function(){
    console.log(5)
});
console.log(6);
//1 3 4 6 5 2
```

（ps：这里顺序加入了一个setTImeout第一个参数，因为传入的是一个函数立即调用，所以视为`setTimeout(undefined,...)`）

1）首先console.log(1),压入执行栈，打印出1，将它弹出执行栈。

2）发现`setTimeout`，压入执行栈，执行`setTimeout`，执行的时候，将它放入 `webapis`等待结果，并且把`setTimeout`弹出执行栈，0秒后有结果了将具体的任务（回调）放入Macro-task

3）接着console.log(3)压入执行栈，打印出3，弹出执行栈

4）整个new Promise调用被压入栈， console.log(4)压入栈，打印出4，console.log(4)弹出栈，之后resolve()入栈，执行完之后resolve()弹出栈，then方法里的具体任务放入Micro-task，

5）console.log(6)压入栈，打印出6，弹出栈

6）主线程（当前的同步任务执行完了），立刻执行Micro-task中的任务，console.log(5)压入栈，打印5,执行完弹出栈

7）主线程（当前的同步任务执行完了）并且Micro-task中无任务，从Macro-task中取任务，console.log(2)压入栈，打印2。执行完弹出栈



##### 感觉文字太多就变好好复杂，看图

![](http://39.108.238.15:97/static/images/images/event-loop.gif)





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





##### 第二个🌰，其实写到这的时候，我觉得我已经对事件循环了如指掌了，然鹅我又看到了这样的一道题



![WX20181225-200839@2x](http://39.108.238.15:97/static/images/images/WX20181225-202805@2x.png)

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<style>
.outer{
    width: 200px;
    height: 200px;
    background: #ddd;
    position: relative;
}
.inner{
    width: 150px;
    height: 150px;
    position: absolute;
    left: 50%;
    top: 50%;
    margin: -75px 0 0 -75px;
    background: #eee;
}
</style>
<body>
    <div class="outer">
        <div class="inner"></div>
    </div>
</body>
<script>
    var outer = document.querySelector('.outer');
    var inner = document.querySelector('.inner');

    new MutationObserver(function () {
        console.log('mutate');
    }).observe(outer, {
        attributes: true
    });


    function onClick() {
        console.log('click');

        setTimeout(function () {
            console.log('timeout');
        }, 0);

        Promise.resolve().then(function () {
            console.log('promise');
        });

        outer.setAttribute('data-random', Math.random());
    }


    inner.addEventListener('click', onClick);
    outer.addEventListener('click', onClick);

</script>

</html>
```

打印结果

```javascript
//click
//promise
//mutate
//click
//promise
//mutate
//timeout
//timeout
```

那么点击的时候发生了什么（没写调用进栈出栈的部分）

1）inner的监听事件触发，触发的函数执行，将函数压入执行栈

2）打印`click`

3）setTImeout是webApis，将方法里的具体任务放入**Macro-task**

4）.then里的具体任务放入**Micro-task**

5）outer.setAttribute触发MutationObserver，将具体的任务放入**Micro-task**

6）这时候inner监听事件的函数执行完，被弹出执行栈

7）主线程（当前的同步任务执行完了），执行**Micro-task**任务

8）**Micro-task**任务被执行，先打印`promise`，再打印`mutate`

9）主线程（当前的同步任务执行完了），**Micro-task**任务队列空了，这时候并没有执行**Macro-task**里的任务！！！因为这时候冒泡被触发了，导致outer的监听事件被触发

10）重复 2）～ 8）

11）主线程（当前的同步任务执行完了），**Micro-task**任务队列空了，执行**Macro-task**里的任务

12）先打印`timeout`，再打印`timeout`。



> Execution of a Job can be initiated only when there is no running execution context and the execution context stack is empty…  — ECMAScript: Jobs and Job Queues

其实重要的是 **Micro-task**任务的执行时机，一有机会立马执行！非常勤劳，**Macro-task**可能就比较懒了。





##### 第三个🌰，一样的，当我看完这题后我觉得我又懂了，然后看下一题

```javascript
//再上一个题目的基础上在script标签的结尾加上
inner.click();

//打印结果
//click
//click
//promise
//mutate
//promise
//timeout
//timeout
```

发生了什么。（没写调用进栈出栈的部分）

首先，这个与上面执行的情况，是它在script中执行，这个很重要

1）执行栈压入script里的整体

2）inner.click的onClick压入栈

3）打印`click`

4）setTimeout的具体任务放入**Macro-task**

5）.then方法里的具体任务放入**Micro-task**

6）outer.setAttribute触发MutationObserver，将具体的任务放入**Micro-task**

7）这时候冒泡被触发，script仍然在主线程中，没有被弹出栈，第一个onclick调用被弹出栈，第二个outer的onclick调用被压入栈（**Micro-task**里的任务没办法插入执行）

8）重复 3）～ 5）

9）第二次没有增加mutation（MutationObserver比较特别，多次添加或者删除子元素之类的操作或者添加属性等等，都只会触发一次回调）

10）然后outer的onclick调用被弹出栈，script也被弹出栈

11）主线程（当前的同步任务执行完了），执行**Micro-task**任务，先打印promise，再打印mutate，接着打印promise

12）主线程（当前的同步任务执行完了），**Micro-task**任务队列空了，执行**Macro-task**里的任务

13）先打印`timeout`，再打印`timeout`。





#### 总结

事件循环不止这么一点，这一点之中可能个人理解会有误，如果发现有理解错误再回来改。

