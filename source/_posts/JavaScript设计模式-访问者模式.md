---
title: JavaScript设计模式-访问者模式
date: 2019-11-25 09:57:28
tags: 
- JavaScript设计模式
categories: 
- JavaScript设计模式
---

## 定义

![WX20191204-220227@2x](http://114.55.30.96/WX20191204-220227@2x.png)

角色介绍:

**Visitor**：表示访问者的抽象类，用于声明对数据结构中xxx元素访问的visit(xxx)方法。
**ConcreteVisitor**：表示具体的访问者，继承Visitor并对其声明的抽象方法提供具体实现。
**Element**：表示元素的抽象类，即访问者实际要访问的对象，Element角色需要对访问者提供一个开放的接口，即accept方法，该方法的参数就是Visitor角色。
**ConcreteElement**：表示具体的元素，提供accept方法的实现。
**ObjectStructure**：负责处理Element元素的集合，即表示数据结构的类

总的来说就是：针对于对象结构的元素，定义新方法，在不改变该对象的前提下访问对象结构中的元素。



## 实现

使用访问者模式，使对象拥有像数组的push pop和splice方法。

```javascript
    var Visitor = (function() {
      return {
        splice: function(){
          var args = Array.prototype.splice.call(arguments, 1)
          return Array.prototype.splice.apply(arguments[0], args)
        },
        push: function(){
          var len = arguments[0].length || 0
          var args = this.splice(arguments, 1)
          arguments[0].length = len + arguments.length - 1
          return Array.prototype.push.apply(arguments[0], args)
        },
        pop: function(){
          return Array.prototype.pop.apply(arguments[0])
        }
      }
    })()
    
    var a = new Object()
    console.log(a.length)
    Visitor.push(a, 1, 2, 3, 4)
    console.log(a.length)
    Visitor.push(a, 4, 5, 6)
    console.log(a.length)
    Visitor.pop(a)
    console.log(a)
    console.log(a.length)
    Visitor.splice(a, 2)
    console.log(a)
```



