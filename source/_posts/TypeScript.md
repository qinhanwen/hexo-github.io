---
title: TypeScript学习笔记
date: 2019-03-07 22:43:08
tags: 
- TS
categories: 
- TS
---

在学习了[es6的常见常用的知识](https://qinhanwen.github.io/2018/12/03/es6%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)之后，工作需要好好学习一下TS。一些与es6相同的特性就不写了。

在开始之前，可以通过npm安装TypeScript

```shell
$ npm install -g typescript
```

之后通过

```shell
$ tsc greeter.ts
```

会输出一个greeter.js文件



## 基础类型

```typescript
let bool:boolean = true;
let num:number = 1;
let string:string = 'string';
let array:Array<number> = [1,2];
let array1:number[] = [2,3];
let obj:Object = {
    name:'qinhanwen',
    age:'25'
}
```

在声明的时候，可以为声明的变量设置类型，设置了某个类型之后，就无法再赋值为其他类型。

![WX20190308-122836@2x](http://www.qinhanwen.xyz/images/WX20190308-122836@2x.png)

如果在声明的时候没有设置类型，那么会有个隐性的类型推断，也是不允许再赋值成其他类型的数据。



**any**

```typescript
let any:any = true;
any = "true";
```

any还有个特点，

```typescript
let obj:any = 1;
obj.test();

let obj1:object = {};
obj1.test();//报错
```

![WX20190308-124431@2x](http://www.qinhanwen.xyz/images/WX20190308-124431@2x.png)





**void**

```typescript
let unusable:void = undefined;
unusable = null;
```

只能赋值为null或者undefined，也可以用来表示函数没有返回值

```typescript
function test(): void {
    console.log("This is my warning message");
}
```



**null**和**undefined**

默认情况下`null`和`undefined`是所有类型的子类型，也就是说可以赋值成其他类型的数据，而不会报错。

```typescript
let bool:boolean = true;
let num:number = 1;
let string:string = 'string';
let array:Array<number> = [1,2];
let array1:number[] = [2,3];
let obj:Object = {
    name:'qinhanwen',
    age:'25'
}
bool = null;
num = undefined;
string = null;
array = undefined;
array1 = null;
obj = undefined;
```



**never**

```typescript
// 返回never的函数必须存在无法达到的终点
function error(message: string): never {
    throw new Error(message);
}

// 推断的返回值类型为never
function fail() {
    return error("Something failed");
}

// 返回never的函数必须存在无法达到的终点
function infiniteLoop(): never {
    while (true) {
    }
}
```



**类型断言**：手动的指定一个值的类型

比如有个联合类型，是数值和字符串，会访问联合类型共有的属性，但是数值类型没有length属性。

```typescript
function getLength(something: string | number): number {
    return something.length;
}
```

![WX20190309-170925@2x](http://www.qinhanwen.xyz/images/WX20190309-170925@2x.png)



一般会先判断是否存在length属性，再做响应的操作。

```typescript
function getLength(something: string | number): number {
    if (something.length) {
        return something.length;
    } else {
        return something.toString().length;
    }
}
```

![WX20190309-171743@2x](http://www.qinhanwen.xyz/images/WX20190309-171743@2x.png)

这时候就需要类型断言

```typescript
function getLength(something: string | number): number {
    if ((something as string).length) {
        return (something as string).length;
    } else {
        return something.toString().length;
    }
}
```

或者

```typescript
function getLength(something: string | number): number {
    if ((<string>something).length) {
        return (<string>something).length;
    } else {
        return something.toString().length;
    }
}
```



**元组**

```typescript
let person: [string, number] = ['qinhanwen', 25];
```



## 变量声明

**let**



**const**



**数组解构**



**对象解构**



**可选参数**

```typescript
function test(a: number = 1, b?: number) {
    console.log(a, b);
}
test(1);//1 undefined
```



**展开操作符-对象**

```typescript
//
var obj = {
    name:'qinhanwen',
    age:'25'
}
var obj1 = {...obj};
console.log(obj1);//{ name: 'qinhanwen', age: '25' }

//合并对象
var obj = {
    name:'qinhanwen',
}
var obj1 = {
    age:25,
    ...obj
}
console.log(obj1);//{ age: 25, name: 'qinhanwen' }
```





## 接口

定义一个接口

```typescript
interface qinhanwen {
    name: string,
    age?: number,
}
```

使用这个接口

```typescript
function test(info: qinhanwen) {
    console.log(info);
}
test({ name: 'qinhanwen', age: 25 });
test({ name: 'qinhanwen' })
```

age作为可选属性，也就是说这个函数调用的时候，必须传入一个数据结构与定义的接口一致的参数。



**只读属性**

```typescript
interface qinhanwen {
    readonly name: string,
    age: number,
}
```

在属性前面添加readonly，也就是说属性在刚创建的时候才可以修改值。

```typescript
interface qinhanwen {
    readonly name: string,
    age: number,
}

function test(info: qinhanwen) {
    info.age = 24;
    info.name = 'zenghua';//这里报错
}
test({ name: 'qinhanwen', age: 25 })
```

![WX20190309-163147@2x](http://www.qinhanwen.xyz/images/WX20190309-163147@2x.png)



如果还有可能附带其他任意数量的属性，这么定义

```typescript
interface qinhanwen {
    readonly name: string,
    age: number,
    [propName: string]: any
}
function test(info: qinhanwen) {
    console.log(info);
}
test({ name: 'qinhanwen', age: 25, sex: 'male' })//{ name: 'qinhanwen', age: 25, sex: 'male' }
```



**函数类型**

```typescript
interface func {
    (name: string, age: number): string;
}

let getAge: func;
getAge = function (name, age) {
    return `${name} is ${age} years old`;
}
getAge('qinhanwen', 25);
```

会对入参，以及返回值做检查。



**接口继承**

```typescript
interface Person {
    name: string;
}

interface Child extends Person {
    age: number;
}

let child = <Child>{};
child.name = "qinhanwen";
child.age = 25;
```



## 类

 **继承**



**修饰符**

**public**

```typescript
class Person{
    public name:string = 'qinhanwen';
    public age:number = 25;
    constructor(){

    }
    public getName(){
        console.log(this.name);
    }
}
new Person().getName();//qinhanwen
```

继承可以访问到父类的共有方法

```typescript
class Person{
    public name:string = 'qinhanwen';
    public age:number = 25;
    constructor(){

    }
    public getName(){
        console.log(this.name);
    }
}
class Child extends Person{
    constructor(){
        super();
    }
    getParentName(){
        super.getName();
    }
}
new Child().getParentName();
```



**private**

```typescript
class Person{
    private name:string = 'qinhanwen';
    private age:number = 25;
    constructor(){

    }
    private getName(){
        console.log(this.name);
    }
}
new Person().getName();//报错，getName为私有属性，只能在Person类内使用
```

并且继承的也无法使用私有变量

```typescript
class Person{
    private name:string = 'qinhanwen';
    private age:number = 25;
    constructor(){

    }
    private getName(){
        console.log(this.name);
    }
}
class Child extends Person{
    constructor(){
        super();
    }
    getParentName(){
        super.getName();
    }
}
```

如图：

![WX20190309-210038@2x](http://www.qinhanwen.xyz/images/WX20190309-210038@2x.png)



**protected**

```typescript
class Person{
    protected name:string = 'qinhanwen';
    protected age:number = 25;
    constructor(){

    }
    protected getName(){
        console.log(this.name);
    }
}
new Person().getName();//报错
```

![WX20190309-215118@2x](http://www.qinhanwen.xyz/images/WX20190309-215118@2x.png)

继承可以使用父类的属性和方法

```typescript
class Person{
    protected name:string = 'qinhanwen';
    protected age:number = 25;
    constructor(){

    }
    protected getName(){
        console.log(this.name);
    }
}
class Child extends Person{
    constructor(){
        super();
    }
    getParentName(){
        super.getName();
    }
}
new Child().getParentName();
```





**readonly**

必须在声明时或者初始化的时候被赋值





**存取器**

可以使用getters/setters来截取对对象成员的访问

```typescript
class Person{
    _age:number = 25;

    set changeAge(age){
        this._age = age;
    }
    get getAge(){
        return this._age;
    }
}

let person = new Person();
console.log(person.getAge);//25
person.changeAge = 26;
console.log(person.getAge);//26
```



**静态属性**

```typescript
class Person {
   static myName: string = 'qinhanwen';
   public myName1: string = 'qinhanwen';
}

console.log(Person.myName);
console.log(Person.myName1);//报错
```

如图：

![WX20190309-214559@2x](http://www.qinhanwen.xyz/images/WX20190309-214559@2x.png)

static与上面的public，private还有protected不同，区别在于属性是在`类本身上`，还是在`类的实例上`



##### 区分一下static，public，private，protected

**public：**通过实例访问的属性，派生类也能访问到。

**private：**只能在类中使用，实例访问不到，派生类也无法访问。

**protected：**只能在类中使用，实例访问不到，派生类可以访问。

**static：**这个是静态属性，直接通过类本身才能访问到。



**重载**

```typescript
function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}
```



## 泛型

不使用泛型的函数

```typescript
function getName(name:string):string{
    return name;
}
```

使用泛型的函数

```typescript
function getName<T>(name:T):T{
    return name;
}
```



调用方式：

1）传入参数类型和参数

```typescript
function getName<T>(name:T):T{
    return name;
}
getName<string>('qinhanwen');
```

2）类型推断，根据传入的数据类型，推断T的类型

```typescript
function getName<T>(name:T):T{
    return name;
}
getName('qinhanwen');
```



**泛型类**

```typescript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```



## 枚举

枚举值和枚举名反向映射

```typescript
enum Days {Sun, Mon, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 0); // true
console.log(Days["Mon"] === 1); // true
console.log(Days["Tue"] === 2); // true
console.log(Days["Sat"] === 6); // true

console.log(Days[0] === "Sun"); // true
console.log(Days[1] === "Mon"); // true
console.log(Days[2] === "Tue"); // true
console.log(Days[6] === "Sat"); // true	
```

看一下枚举转成es5

```typescript
enum Days {Sun, Mon, Tue, Wed, Thu, Fri, Sat};
//转换后
var Days;
(function (Days) {
    Days[Days["Sun"] = 0] = "Sun";
    Days[Days["Mon"] = 1] = "Mon";
    Days[Days["Tue"] = 2] = "Tue";
    Days[Days["Wed"] = 3] = "Wed";
    Days[Days["Thu"] = 4] = "Thu";
    Days[Days["Fri"] = 5] = "Fri";
    Days[Days["Sat"] = 6] = "Sat";
})(Days || (Days = {}));
```



如果改变一下，就会从3开始计数

```typescript
enum Days {Sun = 3, Mon, Tue, Wed, Thu, Fri, Sat};
//转换后
var Days;
(function (Days) {
    Days[Days["Sun"] = 3] = "Sun";
    Days[Days["Mon"] = 4] = "Mon";
    Days[Days["Tue"] = 5] = "Tue";
    Days[Days["Wed"] = 6] = "Wed";
    Days[Days["Thu"] = 7] = "Thu";
    Days[Days["Fri"] = 8] = "Fri";
    Days[Days["Sat"] = 9] = "Sat";
})(Days || (Days = {}));
```



## 类型推断

```typescript
let num = 1;
num = '1';//报错
```

没有明确的给出数值的类型，类型推断会提供数值类型。



## 生成器与迭代器

**Generator**



**Iterator**



















