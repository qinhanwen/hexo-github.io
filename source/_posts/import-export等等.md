---
title: import export 和 export default
date: 2019-02-13 21:08:35
tags:
---

模块功能主要由两个命令构成：`export`和`import`。`export`命令用于规定模块的对外接口，`import`命令用于输入其他模块提供的功能

## export与import

test.js

```javascript
export var name = "qinhanwen";
export var age = 25;
//等价于
var name = "qinhanwen";
var age = 25;
export { name, age };
```



test1.js

```javascript
import { name, age } from './test.js';
console.log(name);//qinhanwen
console.log(age);//25
```



缩写:

```javascript
export { name, age } from './a.js';
```



## 模块整体加载

用星号（`*`）指定一个对象，所有输出值都加载在这个对象上面

test.js

```javascript
var name = "qinhanwen";
var age = 25;
export { name, age };
```



test1.js

```javascript
import * as obj from './test.js';//所有输出值，都在obj对象上
console.log(obj.name);//qinhanwen
console.log(obj.age);//25
```



## export default

`export default`命令用于指定模块的默认输出。一个模块只能有一个默认输出，因此`export default`命令只能使用一次。所以，import命令后面才不用加大括号，因为只可能唯一对应`export default`命令。

本质上，`export default`就是输出一个叫做`default`的变量或方法，然后系统允许你为它取任意名字。

test.js

```javascript
var name = "qinhanwen";
export var age = 25;
export default name;//这个等价于 export {name as default};
```



test1.js

```javascript
import name, { age } from './test.js';//等价于 import { default as name,age } from './test.js';
console.log(name);//qinhanwen
console.log(age);//25
```





## as关键字

as关键字可以为模块重命名

test.js

```javascript
var name = "qinhanwen";
export { name as name1 };
```



test1.js

```javascript
import { name1 as name2 } from './test.js';
console.log(name2);//qinhanwen
```













