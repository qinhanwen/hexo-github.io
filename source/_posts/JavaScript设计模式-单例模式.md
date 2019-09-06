---
title: JavaScript设计模式-单例模式
date: 2019-08-24 13:47:08
tags: 
- JavaScript设计模式
categories: 
- JavaScript设计模式
---

## 定义

保证一个类只有仅有一个实例，并提供一个全局访问它的访问点。

其实就是把实例存下来，下次获取的时候，返回被存下的实例。



## 实现方式

**不透明的单例模式**

必须通过`Singleton.getInstance`获取`Singleton`的实例，跟以往的使用`new`关键字获取实例不同

```javascript
var Singleton = function(name){
     this.name = name;
     this.instance = null;
}

Singleton.prototype.getName = function(){
    return this.name;
}
Singleton.getInstance = function(name){
    if(!this.instance){
        this.instance = new Singleton(name)
    }
    return this.instance;
}
var singleA =Singleton.getInstance('132');
var singleB =Singleton.getInstance('123');
console.log(singleA === singleB);
```



**透明的单例模式**

可以使用`new`关键字获取实例，就跟使用其他构造函数一样

```javascript
var CreateDiv = (function(){
    var instance;
    var CreateDiv = function(html){
        if(instance){
            return instance;
        }
        this.html = html;
        this.init();
        return instance = this;
    }
    CreateDiv.prototype.init = function(){
        var div = document.createElement('div');
        div.innerHTML = this.html;
        document.body.appendChild(div);
    }
    return CreateDiv;
})();

var singleA = new CreateDiv('div');
var singleB = new CreateDiv('div');
console.log(singleA === singleB)
```



**代理实现单例模式**

将管理单例的逻辑抽出到了代理类`ProxySingletonCreateDiv`中，`CreateDiv`变为一个普通的类，职责单一。

```javascript
var CreateDiv = function (html) {
    this.html = html;
    this.init();
};
CreateDiv.prototype.init = function () {
    var div = document.createElement('div');
    div.innerHTML = this.html;
    document.body.appendChild(div);
}
var ProxySingletonCreateDiv = (function () {
    var instance;
    return function (html) {
        if (!instance) {
            instance = new CreateDiv(html)
        }
        return instance;
    }
})();
var singleA = new ProxySingletonCreateDiv('div');
var singleB = new ProxySingletonCreateDiv('div1');
console.log(singleA === singleB)
```



**惰性单例**

当需要的时候才去创建实例

比如点击创建标签，再第一次点击的时候使用变量保存是否创建过对象，如果有的话，下次直接使用

```javascript
let single = function(fn) {
    let flag;
    return flag || (flag = fn.apply(this,arguments));
}

let createIframe = (function(){
    let iframe;
    return function(){
        if(!iframe) {
            iframe = document.createElement('iframe');
            iframe.style.display = 'none';
            document.body.appendChild(iframe);
        }
        return iframe;
    }
})();

document.getElementById("btn").onclick = function(){
    // 调用单例方法
    var createIframeRef = createIframe();
    createIframeRef.style.display = "block";    
};
```

