---
title: setTimeout、setInterval、requestAnimationFrame与requestIdleCallback
date: 2019-01-01 23:43:28
tags: 
- 小记
categories: 
- 小记
---

## setTimeout

setTimeout(code, millseconds) 用于延时执行参数指定的代码，如果在指定的延迟时间之前，你想取消这个执行，那么直接用clearTimeout(timeoutId)来清除任务，timeoutID 是 setTimeout 时返回的；



## setInterval

setInterval(code, millseconds)用于每隔一段时间执行指定的代码，永无停歇，除非你反悔了，想清除它，可以使用 clearInterval(intervalId)，这样从调用 clearInterval 开始，就不会在有重复执行的任务，intervalId 是 setInterval 时返回的；

虽然你在setInterval的里指定的周期是100毫秒，但它并不能保证两个函数之间调用的间隔一定是一百毫秒

```javascript
setInterval(function () {
    func(i++);
}, 100)
```

如果func执行的时间很长，超过了100ms

![WX20191101-235706@2x](http://118.24.241.76/WX20191101-235706@2x.png)

如果队列中的第二个函数时在第450毫秒处结束的话，在第500毫秒时，它会继续执行下一轮func，也就是说这之间的间隔只有50毫秒，而非周期100毫秒

想保证每次执行的间隔应该怎么办？用setTimeout

```javascript
 var i = 1
 var timer = setTimeout(function() { 
   alert(i++) 
   timer = setTimeout(arguments.callee, 2000)
 }, 2000)
```



## requestAnimationFrame

requestAnimationFrame(code)，一般用于动画，与 setTimeout 方法类似，区别是 setTimeout 是用户指定的，而 requestAnimationFrame 是浏览器刷新频率决定的，requestAnimationFrame 会把每一帧中的所有DOM操作集中起来，在一次重绘或回流中就完成，并且重绘或回流的时间间隔紧紧跟随浏览器的刷新频率，一般来说，这个频率为每秒60帧。在隐藏或不可见的元素中，requestAnimationFrame将不会进行重绘或回流，这当然就意味着更少的的cpu，gpu和内存使用量。

怎么停止requestAnimationFrame？cancelAnimationFrame()接收一个参数 requestAnimationFrame默认返回一个id，cancelAnimationFrame只需要传入这个id就可以停止了。

示例：

```html
<!doctype html>
<html lang="en">
<head>
    <title>Document</title>
    <style>
        #e{
            width: 100px;
            height: 100px;
            background: red;
            position: absolute;
            left: 0;
            top: 0;
            zoom: 1;
        }
    </style>
</head>
<body>
<div id="e"></div>
<script>


    var e = document.getElementById("e");
    var flag = true;
    var left = 0;

    function render() {
        if(flag == true){
            if(left>=100){
                flag = false
            }
            e.style.left = ` ${left++}px`
        }else{
            if(left<=0){
                flag = true
            }
            e.style.left = ` ${left--}px`
        }
    }

    //requestAnimationFrame效果
    (function animloop() {
        render();
        window.requestAnimationFrame(animloop);
    })();

</script>
</body>
</html>
```



## requestIdleCallback

当关注用户体验，不希望因为一些不重要的任务（如统计上报）导致用户感觉到卡顿的话，就应该考虑使用requestIdleCallback。因为requestIdleCallback回调的执行的前提条件是当前浏览器处于空闲状态。

示例：

```javascript
requestIdelCallback(myNonEssentialWork);

function myNonEssentialWork (deadline) {

  // deadline.timeRemaining()可以获取到当前帧剩余时间
  while (deadline.timeRemaining() > 0 && tasks.length > 0) {
    doWorkIfNeeded();
  }
  if (tasks.length > 0){
    requestIdleCallback(myNonEssentialWork);
  }
}
```

我们所看到的网页，都是浏览器一帧一帧绘制出来的，通常认为FPS为60的时候是比较流畅的，而FPS为个位数的时候就属于用户可以感知到的卡顿了，那么一帧里面浏览器都要做哪些事情

![WX20191103-120503@2x](http://118.24.241.76/WX20191103-120503@2x.png)

一帧包含了用户的交互、js的执行、以及requestAnimationFrame的调用，布局计算以及页面的重绘等工作，假如某一帧里面要执行的任务不多，一帧的的时间内就完成了上述任务的话，那么这一帧就会有一定的空闲时间，这段时间就恰好可以用来执行requestIdleCallback的回调。

![WX20191103-172528@2x](http://118.24.241.76/WX20191103-172528@2x.png)

由于requestIdleCallback利用的是帧的空闲时间，所以就有可能出现浏览器一直处于繁忙状态，导致回调一直无法执行，那么这种情况我们就需要在调用requestIdleCallback的时候传入第二个配置参数timeout了

示例：

```javascript
requestIdleCallback(myNonEssentialWork, { timeout: 2000 });

function myNonEssentialWork (deadline) {
  // 当回调函数是由于超时才得以执行的话，deadline.didTimeout为true
  while ((deadline.timeRemaining() > 0 || deadline.didTimeout) &&
         tasks.length > 0) {
       doWorkIfNeeded();
    }
  if (tasks.length > 0) {
    requestIdleCallback(myNonEssentialWork);
  }
}
```

但是需要注意的就是这个API兼容性不好。



## 参考

http://qingbob.com/difference-between-settimeout-setinterval/

https://segmentfault.com/a/1190000020639465?utm_source=tag-newest

[requestIdleCallback](https://juejin.im/post/5ad71f39f265da239f07e862)