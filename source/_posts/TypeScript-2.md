---
title: TypeScript-2
date: 2019-08-15 22:49:11
tags: 
- TS
categories: 
- TS
---

## TypeScript面向对象编程

##### SOLID（下面的首字母简写）原则

- **职责单一原则（SRP）：**组件（函数、类、模块）专注于单一任务，这样容易看出类做了哪些事情。类的属性通常描述对象的特征，方法通常描述对象行为。如果拥有过多属性和方法，就要考虑是否职责单一（比如Person类有名字、性别、电话、说hello的方法。那么验证电话号码正确性的方法，应该抽成另外一个类里。）
- **开/闭原则（OCP）**：时刻考虑到代码的扩展性（就是新增功能时候，能够最小程度修改代码）
- **里氏替换原则（LSP）**：派生类对象替换基类对象（比如b是基类有个save方法，a是派生类可以有自己的save方法）
- **接口隔离原则（ISP）**：将特别大的接口，拆分成小的。
- **依赖反转原则（DIP）**：一个方法遵从依赖于抽象而不是实例。



## 新建项目

```shell
$ npm init
$ npm install gulp gulp-typescript -S
```

目录结构，这里选择了比较熟悉的`gulp`，而没使用`webpack`。

```
├── gulpfile.js
├── package-lock.json
├── package.json
├── src
│   └── main.ts
└── tsconfig.json
```

tsconfig.json

```shell
#可以通过这种方式创建
$ tsc —init
```

```json
{
    "files": [
        "src/main.ts"
    ],
    "compilerOptions": {
        "noImplicitAny": false,
        "target": "es5"
    }
}
```

gulpfile.js

```javascript
const gulp = require("gulp");
const ts = require("gulp-typescript");
const tsProject = ts.createProject("tsconfig.json");

gulp.task("default", function () {
    return tsProject.src()
        .pipe(tsProject())
        .js.pipe(gulp.dest("dist"));
});
```

main.ts

```typescript
class Test {
    name: string = 'qinhanwen';
    constructor() {

    }
}
```

编译一下试试

```shell
$ npx gulp
```

编译出来文件在`dist/main.js`

```javascript
var Test = /** @class */ (function () {
    function Test() {
        this.name = 'qinhanwen';
    }
    return Test;
}());
```



## 装饰器

装饰器有`类装饰器`、`参数装饰器`、`方法装饰器`和`属性装饰器`。它们的作用就是在不修改原有对象（接口）的情况下，对类及其方法、入参、属性行为的修改。

**类装饰器**

main.ts

```typescript
function logClass(target: any) {
    // 保存原构造函数的引用
    var original = target;
    //用来生成类的实例的工具方法
    function construct(constructor, args) {
        var c: any = function () {
            return constructor.apply(this, args);
        }
        c.prototype = constructor.prototype;
        return new c();
    }

    //新的构造函数行为
    var f: any = function (...args) {
        console.log("New: " + original.name);
        return construct(original, args);
    }
    //复制原型，使 instanceof操作能正常使用
    f.prototype = original.prototype;
    //返回新的构造函数(将会覆盖原构造函数)
    return f
}

@logClass
class Person {
    name: string;
    age: number;
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    getName() {
        return this.name;
    }
}
```

遇到提示

![WX20190817-000855@2x](http://118.24.241.76/WX20190817-000855@2x.png)

解决方案，修改`tsconfig.json`文件：

```json
{
		...
    "compilerOptions": {
       	...
        "experimentalDecorators": true
    }
}
```

编译ts，main.js文件：

```javascript
var __decorate = (this && this.__decorate) || function (decorators, target, key, desc) {
    var c = arguments.length, r = c < 3 ? target : desc === null ? desc = Object.getOwnPropertyDescriptor(target, key) : desc, d;
    if (typeof Reflect === "object" && typeof Reflect.decorate === "function") r = Reflect.decorate(decorators, target, key, desc);
    else for (var i = decorators.length - 1; i >= 0; i--) if (d = decorators[i]) r = (c < 3 ? d(r) : c > 3 ? d(target, key, r) : d(target, key)) || r;
    return c > 3 && r && Object.defineProperty(target, key, r), r;
};
function logClass(target) {
    // 保存原构造函数的引用
    var original = target;
    //用来生成类的实例的工具方法
    function construct(constructor, args) {
        var c = function () {
            return constructor.apply(this, args);
        };
        c.prototype = constructor.prototype;
        return new c();
    }
    //新的构造函数行为
    var f = function () {
        var args = [];
        for (var _i = 0; _i < arguments.length; _i++) {
            args[_i] = arguments[_i];
        }
        console.log("New: " + original.name);
        return construct(original, args);
    };
    //复制原型，使 instanceof操作能正常使用
    f.prototype = original.prototype;
    //返回新的构造函数(将会覆盖原构造函数)
    return f;
}
var Person = /** @class */ (function () {
    function Person(name, age) {
        this.name = name;
        this.age = age;
    }
    Person.prototype.getName = function () {
        return this.name;
    };
    Person = __decorate([
        logClass
    ], Person);
    return Person;
}());
```

类装饰器接收一个参数（构造函数），为类添加一些额外的逻辑。



**方法装饰器**·

main.ts

```typescript
function logMethod(target: any, key: string, descriptor: any) {
    //保存方法引用
    var originMethod = descriptor.value;
    descriptor.value = function (...args: any[]) {
      	//用于打印，演示部分
        var a = args.map(item => JSON.stringify(item)).join();
        var result = originMethod.apply(this, args);
        var r = JSON.stringify(result);
        console.log(`call ${key}(${a}=>${r})`);
        return result;//返回方法调用的返回值
    }
    return descriptor;
}

class Person {
    name: string;
    age: number;
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    @logMethod
    getName(desc:String) {
        return desc + this.name;
    }
}
```



main.js

```javascript
var __decorate = (this && this.__decorate) || function (decorators, target, key, desc) {
    var c = arguments.length, r = c < 3 ? target : desc === null ? desc = Object.getOwnPropertyDescriptor(target, key) : desc, d;
    if (typeof Reflect === "object" && typeof Reflect.decorate === "function") r = Reflect.decorate(decorators, target, key, desc);
    else for (var i = decorators.length - 1; i >= 0; i--) if (d = decorators[i]) r = (c < 3 ? d(r) : c > 3 ? d(target, key, r) : d(target, key)) || r;
    return c > 3 && r && Object.defineProperty(target, key, r), r;
};
function logMethod(target, key, descriptor) {
    //保存方法引用
    var originMethod = descriptor.value;
    descriptor.value = function () {
        var args = [];
        for (var _i = 0; _i < arguments.length; _i++) {
            args[_i] = arguments[_i];
        }
        var a = args.map(function (item) { return JSON.stringify(item); }).join();
        var result = originMethod.apply(this, args);
        var r = JSON.stringify(result);
        console.log("call " + key + "(" + a + "=>" + r + ")");//call getName("name is "=>"name is qinhanwen")
        return result;
    };
    return descriptor;
}
var Person = /** @class */ (function () {
    function Person(name, age) {
        this.name = name;
        this.age = age;
    }
    Person.prototype.getName = function (desc) {
        return desc + this.name;
    };
    __decorate([
        logMethod
    ], Person.prototype, "getName", null);
    return Person;
}());
```

方法装饰器接收3个参数，属性对象，属性名和属性描述符，为方法增加一些额外的逻辑。



**属性装饰器**

main.ts

```typescript
function logProperty(target: any, key: string) {
    var _val = this[key];
    var getter = function () {
        console.log(`get ${key} ${_val}`);
        return _val;
    }
    var setter = function (newVal) {
        console.log(`set ${key} ${newVal}`);
        _val = newVal;
    }
    if (delete this[key]) {
        Object.defineProperty(target, key, {
            get: getter,
            set: setter,
            configurable: true,
            enumerable: true
        })
    }
}

class Person {
    @logProperty
    name: string;
    age: number;
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    getName(desc: String) {
        return desc + this.name;
    }
}
```

main.js

```javascript
var __decorate = (this && this.__decorate) || function (decorators, target, key, desc) {
    var c = arguments.length, r = c < 3 ? target : desc === null ? desc = Object.getOwnPropertyDescriptor(target, key) : desc, d;
    if (typeof Reflect === "object" && typeof Reflect.decorate === "function") r = Reflect.decorate(decorators, target, key, desc);
    else for (var i = decorators.length - 1; i >= 0; i--) if (d = decorators[i]) r = (c < 3 ? d(r) : c > 3 ? d(target, key, r) : d(target, key)) || r;
    return c > 3 && r && Object.defineProperty(target, key, r), r;
};
function logProperty(target, key) {
    var _val = this[key];
    var getter = function () {
        console.log("get " + key + " " + _val);
        return _val;
    };
    var setter = function (newVal) {
        console.log("set " + key + " " + newVal);
        _val = newVal;
    };
    if (delete this[key]) {
        Object.defineProperty(target, key, {
            get: getter,
            set: setter,
            configurable: true,
            enumerable: true
        });
    }
}
var Person = /** @class */ (function () {
    function Person(name, age) {
        this.name = name;
        this.age = age;
    }
    Person.prototype.getName = function (desc) {
        return desc + this.name;
    };
    __decorate([
        logProperty
    ], Person.prototype, "name", void 0);
    return Person;
}());
```

属性装饰器接收2个参数，属性对象与属性名，为属性的set和get操作添加一些逻辑



**参数装饰器**

main.ts

```typescript
function addMetadata(target: any, key: string, index: number) {

}
class Person {
    name: string;
    age: number;
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }

    getName(@addMetadata desc: String) {
        return desc + this.name;
    }
}
```

main.js

```javascript
var __decorate = (this && this.__decorate) || function (decorators, target, key, desc) {
    var c = arguments.length, r = c < 3 ? target : desc === null ? desc = Object.getOwnPropertyDescriptor(target, key) : desc, d;
    if (typeof Reflect === "object" && typeof Reflect.decorate === "function") r = Reflect.decorate(decorators, target, key, desc);
    else for (var i = decorators.length - 1; i >= 0; i--) if (d = decorators[i]) r = (c < 3 ? d(r) : c > 3 ? d(target, key, r) : d(target, key)) || r;
    return c > 3 && r && Object.defineProperty(target, key, r), r;
};
var __param = (this && this.__param) || function (paramIndex, decorator) {
    return function (target, key) { decorator(target, key, paramIndex); }
};
function addMetadata(target, key, index) {
}
var Person = /** @class */ (function () {
    function Person(name, age) {
        this.name = name;
        this.age = age;
    }
    Person.prototype.getName = function (desc) {
        return desc + this.name;
    };
    __decorate([
        __param(0, addMetadata)
    ], Person.prototype, "getName", null);
    return Person;
}());
```



## 装饰器工厂

装饰器工厂能够鉴别使用哪种装饰器

![WX20190818-000319@2x](http://118.24.241.76/WX20190818-000319@2x.png)

使用`@log`装饰器

![WX20190818-000811@2x](http://118.24.241.76/WX20190818-000811@2x.png)



## 带参装饰器

特殊的装饰器工厂

```typescript
function logMethod(options: any) {
    return function (target: any, key: string, descriptor: any) {
        console.log(options);
    }
}

class Person {
    name: string;
    age: number;
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    @logMethod('method')
    getName(desc: String) {
        return desc + this.name;
    }
}
```



## 新语法

`declare var`声明全局变量

`declare function` 声明全局方法

`declare class` 声明全局类

`declare enum` 声明全局枚举类型

`declare namespace`声明（含有子属性的）全局对象

`interface` 和 `type` 声明全局类型

`export` 导出变量

`export namespace` 导出（含有子属性的）对象

`export default`ES6 默认导出

`export =` commonjs 导出模块

`export as namespace`UMD 库声明全局变量

`declare global` 扩展全局变量

`declare module`扩展模块

`/// <reference />`三斜线指令



**全局变量的声明文件主要有以下几种语法：**

- `declare var`声明全局变量

用来定义一个全局变量的类型

```typescript
declare var functionA: (param: any) => any; 
declare let functionB: (param: any) => any; 
declare const functionC: (param: any) => any; 
```

- `declare function` 声明全局方法

```typescript
declare function functionA(selector: string): any;
```

- `declare class`声明全局类

```typescript
declare class Animal {
    name: string;
    constructor(name: string);
    sayHi(): string;
}
```

- `declare enum` 声明全局枚举类型

```typescript
declare enum Directions {
    Up,
    Down,
    Left,
    Right
}
```

- `declare namespace` 声明（含有子属性的）全局对象

```typescript
declare namespace jQuery {
    function ajax(url: string, settings?: any): void;
}
//声明这个拥有多个子属性的全局变量
```

- `interface` 和 `type`声明全局类型

```typescript
interface AjaxSettings {
    method?: 'GET' | 'POST'
    data?: any;
}
```

`type` 与 `interface` 类似

```typescript
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === 'string') {
        return n;
    } else {
        return n();
    }
}
```

声明语句中只能定义类型，不要在声明语句中定义具体实现。





## 声明文件

**什么是`.d.ts文件`**

这个文件是对第三方库类型API的类型声明，`JavaScript`和`TypeScript`能集成在一起依赖于它。



**@types**

使用 `@types` 统一管理第三方库的声明文件，可以使用`npm`直接安装对应依赖包的声明模块

```shell
$ npm install @types/jquery -S
```



**书写声明文件**

不同场景下，声明文件的内容和使用方式也不同。



## npm包

npm包的声明文件可能存在两个地方

- `package.json` 中有 `types` 字段，或者有一个 `index.d.ts` 声明文件
- 我们只需要尝试安装一下对应的 `@types` 包

如果没有描述文件，可以自己写声明文件





## 抽象类

`abstract`用于定义抽象类和其中的抽象方法，它有以下特点

- 抽象类是不允许被实例化
- 抽象方法必须被子类实现

```typescript
abstract class Animal {
    name: string
    constructor(name) {
        this.name = name;
    }
    sayHi(string) {

    }
}
class Dog extends Animal {
    sayHi() {
        console.log(`hi ${this.name}`)
    }
}
new Dog('dog').sayHi();//hi dog
```



看一下类的继承，使用 `extends` 关键字实现继承

```typescript
class Animal {
    name: string
    constructor(name) {
        this.name = name;
    }
    sayHi(string) {

    }
}
class Dog extends Animal {
    sayHi() {
        console.log(`hi ${this.name}`);
    }
}
new Dog('dog').sayHi();//hi dog
```

`Dog`类里没有构造函数，`this`指向`Dog`实例，看下编译成JavaScript

```javascript
var __extends = (this && this.__extends) || (function () {
    var extendStatics = function (d, b) {//这个方法让d.__proto__ = b
        extendStatics = Object.setPrototypeOf ||
            ({ __proto__: [] } instanceof Array && function (d, b) { d.__proto__ = b; }) ||
            function (d, b) { for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p]; };
        return extendStatics(d, b);
    };
    return function (d, b) {
        extendStatics(d, b);
        function __() { this.constructor = d; }
        d.prototype = b === null ? Object.create(b) : (__.prototype = b.prototype, new __());
    };
})();
var Animal = /** @class */ (function () {
    function Animal(name) {
        this.name = name;
    }
    Animal.prototype.sayHi = function (string) {
    };
    return Animal;
}());
var Dog = /** @class */ (function (_super) {
    __extends(Dog, _super);
    function Dog() {
        return _super !== null && _super.apply(this, arguments) || this;//这里调用时候走进Animal方法，this指向Dog，所以name属性在Dog实例上
    }
    Dog.prototype.sayHi = function () {
        console.log("hi " + this.name); 
    };
    return Dog;
}(Animal));
new Dog('dog').sayHi(); //hi dog
```

类的继承特点：

- 可以不需要实现父类的方法
- 可以使用 `super` 关键字来调用父类的构造函数和方法



## 资料

[tsconfig.js文件](https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook/tsconfig.json.html)

https://ts.xcatliu.com/basics/declaration-files