---
title: TypeScript-3
date: 2019-08-19 12:13:24
tags: 
- TS
categories: 
- TS
---

## tsconfig.json

如果一个目录下存在一个`tsconfig.json`文件，那么它意味着这个目录是TypeScript项目的根目录。 `tsconfig.json`文件中指定了用来编译这个项目的根文件和编译选项。

可以手动创建`tsconfig.json`文件，也也可以通过`tsc —init`创建。

可以通过 `compilerOptions` 来定制编译选项，这边就写一点常见（我常见）的：

```javascript
{
    // "files": [ //显式指定需要编译的文件
    //     "./src/index.ts"
    // ],
    "include": [//包含某个目录下
        "./src/"
    ],
    "exclude": [//剔除某个目录下
        "./src/untils/*.ts",
    ],
    "compilerOptions": {
        "outDir": "./dist",//输出到某个目录
        "noImplicitAny": false,//没有声明类型的时候，不爆红
        "target": "es5",//
        "experimentalDecorators": true//这个是装饰器爆红的
    }
}
```

[其他配置信息](https://jkchao.github.io/typescript-book-chinese/project/compilationContext.html#tsconfig-json)



## 模块

**全局模块**

两个`ts`文件在同一个目录下，这样声明`a`相当于在全局中声明

![WX20190820-225555@2x](http://114.55.30.96/WX20190820-225555@2x.png)



**文件模块**

使用`export`或者`import`，会在文件中创建一个本地作用域

![WX20190820-230729@2x](http://114.55.30.96/WX20190820-230729@2x.png)

正确的使用方式：

![WX20190820-230807@2x](http://114.55.30.96/WX20190820-230807@2x.png)

[尽量使用 ES 模块导入导出语法]([https://qinhanwen.github.io/2019/02/13/import-export%E7%AD%89%E7%AD%89/](https://qinhanwen.github.io/2019/02/13/import-export等等/))



**相对模块路径：**

当使用`import { b } from "./moduleB"`的时候，在`/root/src/moduleA.ts`里

1. `/root/src/moduleB.ts`
2. `/root/src/moduleB.tsx`
3. `/root/src/moduleB.d.ts`
4. `/root/src/moduleB/package.json` (如果指定了`"types"`属性)
5. `/root/src/moduleB/index.ts`
6. `/root/src/moduleB/index.tsx`
7. `/root/src/moduleB/index.d.ts`



**非相对模块路径：**

当导入不是相对模块路径的时候，当你使用 `import { b } from "moduleB"`，它是在`/root/src/folder/A.ts`文件里，会按照下面顺序查找

1. `/root/src/node_modules/moduleB.ts`

2. `/root/src/node_modules/moduleB.tsx`

3. `/root/src/node_modules/moduleB.d.ts`

4. `/root/src/node_modules/moduleB/package.json` (如果指定了`"types"`属性)

5. `/root/src/node_modules/moduleB/index.ts`

6. `/root/src/node_modules/moduleB/index.tsx`

7. `/root/src/node_modules/moduleB/index.d.ts` 

   

8. `/root/node_modules/moduleB.ts`

9. `/root/node_modules/moduleB.tsx`

10. `/root/node_modules/moduleB.d.ts`

11. `/root/node_modules/moduleB/package.json` (如果指定了`"types"`属性)

12. `/root/node_modules/moduleB/index.ts`

13. `/root/node_modules/moduleB/index.tsx`

14. `/root/node_modules/moduleB/index.d.ts` 

    

15. `/node_modules/moduleB.ts`

16. `/node_modules/moduleB.tsx`

17. `/node_modules/moduleB.d.ts`

18. `/node_modules/moduleB/package.json` (如果指定了`"types"`属性)

19. `/node_modules/moduleB/index.ts`

20. `/node_modules/moduleB/index.tsx`

21. `/node_modules/moduleB/index.d.ts`

在步骤（8）和（15）向上跳了两次目录



**NodeJS如何解析模块：**

在Node.js里导入是通过`require`函数调用进行的。 Node.js会根据 `require`的是相对路径还是非相对路径做出不同的行为。

相对路径：

假设有一个文件路径为 `/root/src/moduleA.js`，包含了一个导入`var x = require("./moduleB");` Node.js以下面的顺序解析这个导入：

1. 检查`/root/src/moduleB.js`文件是否存在。
2. 检查`/root/src/moduleB`目录是否包含一个`package.json`文件，且`package.json`文件指定了一个`"main"`模块。 在我们的例子里，如果Node.js发现文件 `/root/src/moduleB/package.json`包含了`{ "main": "lib/mainModule.js" }`，那么Node.js会引用`/root/src/moduleB/lib/mainModule.js`。
3. 检查`/root/src/moduleB`目录是否包含一个`index.js`文件。 这个文件会被隐式地当作那个文件夹下的"main"模块。



非相对路径：

Node会在一个特殊的文件夹 `node_modules`里查找你的模块。`node_modules`可能与当前文件在同一级目录下，或者在上层目录里。 Node会向上级目录遍历，查找每个`node_modules`直到它找到要加载的模块。

还是用上面例子，但假设`/root/src/moduleA.js`里使用的是非相对路径导入`var x = require("moduleB");`。 Node则会以下面的顺序去解析 `moduleB`，直到有一个匹配上。

1. `/root/src/node_modules/moduleB.js`

2. `/root/src/node_modules/moduleB/package.json` (如果指定了`"main"`属性)

3. `/root/src/node_modules/moduleB/index.js` 

   

4. `/root/node_modules/moduleB.js`

5. `/root/node_modules/moduleB/package.json` (如果指定了`"main"`属性)

6. `/root/node_modules/moduleB/index.js` 

   

7. `/node_modules/moduleB.js`

8. `/node_modules/moduleB/package.json` (如果指定了`"main"`属性)

9. `/node_modules/moduleB/index.js`

注意Node.js在步骤（4）和（7）会向上跳一级目录。



TS调用Js

index.ts

```typescript
import { b } from './untils/untils';
console.log(b);
```

untils.d.ts

```typescript
export declare const b:number
```

untils.js

```javascript
export const b = 1;
```



实际的项目里，建议把声明放在`.d.ts`文件内，当然放在`ts`文件里也可以



## 资料

[工程配置](https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook/tsconfig.json.html)

[项目地址](https://github.com/qinhanwen/qhw-learning-typescript.git)

[TypeScript](https://www.tslang.cn/docs/handbook/module-resolution.html)

[深入了解TypeScript](https://jkchao.github.io/typescript-book-chinese/)