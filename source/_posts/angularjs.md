---
title: angularjs
date: 2018-11-08 21:14:23
tags: 
- angularjs
categories: 
- angularjs
---

## angularInit方法

属性选择器选中[ng-app]作为appElement的值

![WX20191023-172022@2x](http://118.24.241.76/WX20191023-172022@2x.png)

![WX20191023-172049@2x](http://118.24.241.76/WX20191023-172049@2x.png)

然后进去compileNodes，深度优先遍历子节点获取子节点的directive（比如p标签的ng-click，和input的ng-model）

![WX20191023-172650@2x](http://118.24.241.76/WX20191023-172650@2x.png)

之后先做watchers的收集，后面更新变化用的

![WX20191023-233311@2x](http://118.24.241.76/WX20191023-233311@2x.png)

之后为input添加input事件，为div添加点击事件，比如添加点击事件的地方，先获取调用的事件，封装了一下

![WX20191023-173812@2x](http://118.24.241.76/WX20191023-173812@2x.png)

再添加事件

![WX20191023-214405@2x](http://118.24.241.76/WX20191023-214405@2x.png)





## angularjs双向绑定原理

1.构造函数

```javascript
function Scope() {
  this.$id = nextUid();
  this.$$phase = this.$parent = this.$$watchers =
                 this.$$nextSibling = this.$$prevSibling =
                 this.$$childHead = this.$$childTail = null;
  this.$root = this;
  this.$$destroyed = false;
  this.$$listeners = {};
  this.$$listenerCount = {};
  this.$$watchersCount = 0;
  this.$$isolateBindings = null;
}
```
2.属性挂载到Scope实例，也就是\$scope上

3.$$wachers收集

4.为input绑定监听事件，输入的时候，最后会调用scope.\$apply，触发\$rootScope.\$digest()，先设置dirty为false， 遍历wathchers，判断当前的value与watch.last是否相等，不相等就把dirty设置为true，之后设置nodeValue更新值。





## angularjs事件触发更新

为元素添加绑定事件，触发更新跟上面一样