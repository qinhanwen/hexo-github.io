---
title: new的时候到底做了什么
date: 2018-12-04 23:05:16
tags:
---

# new到底做了什么

> 参考资料：https://juejin.im/post/584e1ac50ce463005c618ca2
>
> ​                   https://juejin.im/entry/59196a3ea0bb9f005ff7bee5
>
> ​		   https://xiaogliu.github.io/2018/06/28/what-happened-when-using-new/

##### 1）new做了什么

![](http://39.108.238.15:97/static/images/images/WX20181204-231216@2x.png)

1.创建了一个临时对象

2.临时对象的`__proto__` 等于士兵的`prototype`

3.执行函数，执行中this指向这个对象

4.如果没有return `对象类型数据`，则返回这个临时对象



##### 2）new的时候this的指向

直接看🌰

```javascript
this.alia = 'window';
this.win1 = 'window property';
var a = new fun1();

function fun1(){
    //fun1内的this指的是 a 对象，是fun1{};

    this.alia = 'fun1'; 
    this.ofun1 ="only fun1 have"
    console.log("fun1的alia : " + this.alia);   //"fun1的alia :fun1"
    console.log(this.win1);   // "undefine"

    function fun2 (){
        // this指的是window。
        //window没有ofun1属性,所以输出了undefine
        console.log(this.ofun1); // "undefine"

        //下面都是window已有的属性
        console.log("fun2的alia :" + this.alia);    //"fun2的alia :window"
        console.log(this.win1);   // "window property"

        //给window添加ofun1属性
        this.ofun1 = "change in fun2"

    }
    fun2();
    // 这时this 是 a对象本身的fun1{}
    console.log("this.ofun1 :" +this.ofun1 );   //this.ofun1 :only fun1 have

    //window.ofun1刚刚已经在fun2里面赋值了，所以输出有值
    console.log("window.ofun1 :" +window.ofun1 );   //window.ofun1 :change in fun2
}
```

调用函数里面的this属性赋值都是给window赋值的。