---
title: Babel 插件开发
date: 2020-02-06 13:50:29
tags:
  - Babel
categories:
  - Babel
---



## 开发第一个 Babel 插件

**一个接收当前 babel 对象作为参数的 function，(而我们经常使用的方式是从 babel 中解构拿到 types)**

```javascript
export default function(babel) {
  // plugin contents
}

// export default function({ types: t }) {
// }
```



**返回一个对象，visitor属性是这个插件的主要访问者，visitor 中每个函数接收 2 个参数**

```javascript
export default function({ types: t }) {
  return {
    visitor: {
       Identifier(path, state) {},
       CallExpression (path, state) {}
    }
  };
};
```



**从一个简单例子看看插件是如何工作的，将 foo 转换为 a ，bar 转换为 b**

```javascript
foo === bar;

// ast
{
  type: "BinaryExpression",
  operator: "===",
  left: {
    type: "Identifier",
    name: "foo"
  },
  right: {
    type: "Identifier",
    name: "bar"
  }
}
```



创建项目，安装依赖

```shell
$ npm i @babel/cli @babel/core -D
```

目录结构

```
├── index.js // 需要编译的文件
├── lib // 编译后文件目录
│   └── index.js
├── package-lock.json
├── package.json
├── .babelrc // 配置文件
└── translate // 插件目录
    └── index.js
```



.babelrc 

```json
{
    "plugins": ["./translate"]
}
```



package.json 

```json
{
  "name": "babel-plugin",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "rm -rf lib && babel ./index.js --out-dir lib"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@babel/cli": "^7.8.4",
    "@babel/core": "^7.8.4"
  }
}
```



translate/index.js

```javascript
module.exports = (babel) => {
    console.log(babel);
    const { types } = babel;
    return {
        visitor: {
            BinaryExpression (path, state) {// 类型为 BinaryExpression 的节点，通过节点的构造器方法builder得到新的值
                console.log(types);
                console.log(path);
                console.log(state);
                path.node.left = t.identifier("a");
							  path.node.right = t.identifier("b");
            }
        }
    }
}
```



编译出来的文件

```javascript
a === b;
```



## 转换操作

**访问方法**

```javascript
            BinaryExpression (path, state) {// 类型为 BinaryExpression 的节点，通过节点的构造器方法builder得到新的值
                console.log('子节点的内部path', path.get('left'));// 获得子节点的内部path

                if (t.isIdentifier(path.node.left, { name: "n" })) {// 检查节点类型
                    // ...
                }
                // 等价于
                // if (
                //     path.node.left != null &&
                //     path.node.left.type === "Identifier" &&
                //     path.node.left.name === "n"
                // ) {
                //     // ...
                // }

                if (path.node.operator !== '===') return;// 停止遍历，如果你的插件需要在某种情况下不运行，最简单的做法是尽早写回
            },

```



**处理方法**

```javascript
            // 替换单节点
            // function square (n) {
            //     -   return n * n;
            //     +   return n ** 2;
            // }
            BinaryExpression (path) {
                path.replaceWith(
                    t.binaryExpression("**", path.node.left, t.numberLiteral(2))
                );
            },

            //替换多节点
            // function square (n) {
            //     -   return n * n;
            //     +   "Is this the real life?";
            //     +   "Is this just fantasy?";
            //     +   "(Enjoy singing the rest of the song in your head)";
            // }
            ReturnStatement (path) {
                path.replaceWithMultiple([
                    t.expressionStatement(t.stringLiteral("Is this the real life?")),
                    t.expressionStatement(t.stringLiteral("Is this just fantasy?")),
                    t.expressionStatement(t.stringLiteral("(Enjoy singing the rest of the song in your head)")),
                ]);
            },

            // 字符串源码代替节点
            // - function square(n) {
            //     -   return n * n;
            //     + function add(a, b) {
            //     +   return a + b;
            // }
            FunctionDeclaration (path) {
                path.replaceWithSourceString(function add (a, b) { return a + b; });
            },

            // 插入兄弟节点
            // + "Because I'm easy come, easy go.";
            // function square(n) {
            //     return n * n;
            // }
            // + "A little high, little low.";
            FunctionDeclaration (path) {
                path.insertBefore(t.expressionStatement(t.stringLiteral("Because I'm easy come, easy go.")));
                path.insertAfter(t.expressionStatement(t.stringLiteral("A little high, little low.")));
            },

            // 插入容器
            // class A {
            //     constructor() {
            //   +   "before"
            //       var a = 'middle';
            //   +   "after"
            //     }
            // }
            ClassMethod (path) {
                path.get('body').unshiftContainer('body', t.expressionStatement(t.stringLiteral('before')));
                path.get('body').pushContainer('body', t.expressionStatement(t.stringLiteral('after')));
            },

            // 删除节点
            FunctionDeclaration (path) {
                path.remove();
            },

            // 删除父节点
            BinaryExpression (path) {
                path.parentPath.remove();
            },
```



**插件选项**

```javascript
					 // 配置文件中的参数
            //    {
            //         plugins: [
            //         ["my-plugin", {
            //             "option1": true,
            //             "option2": false
            //         }]
            //         ]
            //     }
            FunctionDeclaration (path, state) {
                console.log(state.opts);
                // { option1: true, option2: false }
            },
```



**构建节点**

```javascript
            /** 
            * 编写转换时，通常需要构建一些要插入的节点进入AST，可以使用 babel-types 包中的 builder 方法
            * 构建器的方法名称就是您想要的节点类型的名称，除了第一个字母小写。 例如，如果您想建立一个 MemberExpression 您可以使用 t.memberExpression（...）
            */
            BinaryExpression (path) {
                // member.expression.property
                path.node.right = t.memberExpression(
                    t.memberExpression(
                        t.identifier('member'),
                        t.identifier('expression')
                    ),
                    t.identifier('property')
                )
            }
```



## 参考资料

https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#toc-writing-your-first-babel-plugin