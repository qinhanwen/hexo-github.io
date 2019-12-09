---
title: Babel学习笔记2 - 处理流程
date: 2019-12-01 11:32:03
tags: 
- Babel
categories: 
- Babel
---

![WX20191202-201539@2x](http://www.qinhanwen.xyz/WX20191202-201539@2x.png)

## 前置知识

开始之前补充一些知识

-  `String.prototype.codePointAt` 方法，返回 一个 Unicode 编码点值的非负整数，如果在指定的位置没有元素则返回 `undefined`

示例：

```javascript
str.codePointAt(pos)
// pos 这个字符串中需要转码的元素的位置。

'ABC'.codePointAt(1); // 66
'XYZ'.codePointAt(3); // undefined
```



- ast（抽象语法树）

可以在这个网站尝试代码转成ast ：https://astexplorer.net/

例如：

```javascript
console.log('hello qhw')
```

转换成json的结果

```json
{
  "type": "Program",
  "start": 0,
  "end": 24,
  "body": [
    {
      "type": "ExpressionStatement",
      "start": 0,
      "end": 24,
      "expression": {
        "type": "CallExpression",
        "start": 0,
        "end": 24,
        "callee": {
          "type": "MemberExpression",
          "start": 0,
          "end": 11,
          "object": {
            "type": "Identifier",
            "start": 0,
            "end": 7,
            "name": "console"
          },
          "property": {
            "type": "Identifier",
            "start": 8,
            "end": 11,
            "name": "log"
          },
          "computed": false
        },
        "arguments": [
          {
            "type": "Literal",
            "start": 12,
            "end": 23,
            "value": "hello qhw",
            "raw": "'hello qhw'"
          }
        ]
      }
    }
  ],
  "sourceType": "module"
}
```

ast是由单个或多个节点组成的，节点在不同层级有相似的结构。

节点中的type字段表示节点的类型。



- [Babylon](https://github.com/babel/babel/tree/master/packages/babel-parser)，仓库被移动了，它的新身份的是`@babel/parser`。它将代码解析成ast，在解析过程中有两个阶段：**词法分析**和**语法分析**，下面会说。



- visitor

当Babel处理一个节点时，是以访问者的形式获取节点信息，并进行相关操作，这种方式是通过一个visitor对象来完成的，在visitor对象中定义了对于各种节点的访问函数，这样就可以针对不同的节点做出不同的处理。我们编写的Babel插件其实也是通过定义一个实例化visitor对象处理一系列的AST节点来完成我们对代码的修改操作。



- path

从上面的visitor对象中，可以看到每次访问节点方法时，都会传入一个path参数，这个path参数中包含了节点的信息以及节点和所在的位置，以供对特定节点进行操作。具体来说Path 是表示两个节点之间连接的对象。这个对象不仅包含了当前节点的信息，也有当前节点的父节点的信息，同时也包含了添加、更新、移动和删除节点有关的其他很多方法。具体地，Path对象包含的属性和方法主要如下：

```
── 属性      
  - node   当前节点
  - parent  父节点
  - parentPath 父path
  - scope   作用域
  - context  上下文
  - ...
── 方法
  - get   当前节点
  - findParent  向父节点搜寻节点
  - getSibling 获取兄弟节点
  - replaceWith  用AST节点替换该节点
  - replaceWithMultiple 用多个AST节点替换该节点
  - insertBefore  在节点前插入节点
  - insertAfter 在节点后插入节点
  - remove   删除节点
  - ...
```



- state

state是visitor对象中每次访问节点方法时传入的第二个参数。state就是一系列状态的集合，包含诸如当前plugin的信息、plugin传入的配置参数信息，甚至当前节点的path信息也能获取到，当然也可以把babel插件处理过程中的自定义状态存储到state对象中。



- scopes作用域

这里的作用域其实跟js说的作用域是一个道理，也就是说babel在处理AST时也需要考虑作用域的问题，比如函数内外的同名变量需要区分开来。



## 通过一个示例来看过程

```javascript
const {
    parse
} = require('@babel/parser');
const traverse = require('@babel/traverse').default;
const generate = require('@babel/generator').default;

const code = `function fn(x,y){console.log(x,y)}`;
const ast = parse(code);
traverse(ast, {
    enter(path) {
        console.log(path.node);
        if (path.isIdentifier({
                name: "x"
            })) {
            path.node.name = 'z';
        }
        
    }
});
const output = generate(ast);
console.log(output) 
```



**解析阶段**

将代码解析成抽象语法树（AST），每个js引擎（比如Chrome浏览器中的V8引擎）都有自己的AST解析器，而Babel是通过[Babylon](https://github.com/babel/babel/tree/master/packages/babel-parser)实现的。在解析过程中有 **词法分析** 和 **语法分析**。词法分析阶段把字符串形式的代码转换为**令牌**，令牌类似于AST中节点；而语法分析阶段则会把令牌转换成 AST的形式，同时这个阶段会把令牌中的信息转换成AST的表述结构。



- 词法分析生成token

从`parse(code)`方法调用进入，断点走到这个方法

![WX20191201-204659@2x](http://www.qinhanwen.xyz/WX20191201-204659@2x.png)

`getTokenFromCode`方法，根据不同的Unicode编码值，进行不同的处理

```javascript
  getTokenFromCode(code) {
    switch (code) {
      case 46:
        this.readToken_dot();
        return;

      case 40:
        ++this.state.pos;
        this.finishToken(types.parenL);
        return;

      case 41:
        ++this.state.pos;
        this.finishToken(types.parenR);
        return;

      case 59:
        ++this.state.pos;
        this.finishToken(types.semi);
        return;

      case 44:
        ++this.state.pos;
        this.finishToken(types.comma);
        return;

      case 91:
        ++this.state.pos;
        this.finishToken(types.bracketL);
        return;

      case 93:
        ++this.state.pos;
        this.finishToken(types.bracketR);
        return;

      case 123:
        ++this.state.pos;
        this.finishToken(types.braceL);
        return;

      case 125:
        ++this.state.pos;
        this.finishToken(types.braceR);
        return;

      case 58:
        if (this.hasPlugin("functionBind") && this.input.charCodeAt(this.state.pos + 1) === 58) {
          this.finishOp(types.doubleColon, 2);
        } else {
          ++this.state.pos;
          this.finishToken(types.colon);
        }

        return;

      case 63:
        this.readToken_question();
        return;

      case 96:
        ++this.state.pos;
        this.finishToken(types.backQuote);
        return;

      case 48:
        {
          const next = this.input.charCodeAt(this.state.pos + 1);

          if (next === 120 || next === 88) {
            this.readRadixNumber(16);
            return;
          }

          if (next === 111 || next === 79) {
            this.readRadixNumber(8);
            return;
          }

          if (next === 98 || next === 66) {
            this.readRadixNumber(2);
            return;
          }
        }

      case 49:
      case 50:
      case 51:
      case 52:
      case 53:
      case 54:
      case 55:
      case 56:
      case 57:
        this.readNumber(false);
        return;

      case 34:
      case 39:
        this.readString(code);
        return;

      case 47:
        this.readToken_slash();
        return;

      case 37:
      case 42:
        this.readToken_mult_modulo(code);
        return;

      case 124:
      case 38:
        this.readToken_pipe_amp(code);
        return;

      case 94:
        this.readToken_caret();
        return;

      case 43:
      case 45:
        this.readToken_plus_min(code);
        return;

      case 60:
      case 62:
        this.readToken_lt_gt(code);
        return;

      case 61:
      case 33:
        this.readToken_eq_excl(code);
        return;

      case 126:
        this.finishOp(types.tilde, 1);
        return;

      case 64:
        ++this.state.pos;
        this.finishToken(types.at);
        return;

      case 35:
        this.readToken_numberSign();
        return;

      case 92:
        this.readWord();
        return;

      default:
        if (isIdentifierStart(code)) {
          this.readWord();
          return;
        }

    }

    throw this.raise(this.state.pos, `Unexpected character '${String.fromCodePoint(code)}'`);
  }
```

看一下`readWord`方法

![WX20191201-205528@2x](http://www.qinhanwen.xyz/WX20191201-205528@2x.png)

进入`readWord1`方法，比如`console.log`解析，就先拿`console`，然后返回

![WX20191201-205600@2x](http://www.qinhanwen.xyz/WX20191201-205600@2x.png)



- 语法分析

`createIdentifier`方法，创建节点

![WX20191204-100126@2x](http://www.qinhanwen.xyz/WX20191204-100126@2x.png)

`finishNode`方法

![WX20191204-103613@2x](http://www.qinhanwen.xyz/WX20191204-103613@2x.png)

`finishNodeAt`方法

![WX20191204-103656@2x](http://www.qinhanwen.xyz/WX20191204-103656@2x.png)



解析出的ast大概是这样的

```json
{
  "type": "Program",
  "start": 0,
  "end": 34,
  "body": [
    {
      "type": "FunctionDeclaration",
      "start": 0,
      "end": 34,
      "id": {
        "type": "Identifier",
        "start": 9,
        "end": 11,
        "name": "fn"
      },
      "expression": false,
      "generator": false,
      "async": false,
      "params": [
        {
          "type": "Identifier",
          "start": 12,
          "end": 13,
          "name": "x"
        },
        {
          "type": "Identifier",
          "start": 14,
          "end": 15,
          "name": "y"
        }
      ],
      "body": {
        "type": "BlockStatement",
        "start": 16,
        "end": 34,
        "body": [
          {
            "type": "ExpressionStatement",
            "start": 17,
            "end": 33,
            "expression": {
              "type": "CallExpression",
              "start": 17,
              "end": 33,
              "callee": {
                "type": "MemberExpression",
                "start": 17,
                "end": 28,
                "object": {
                  "type": "Identifier",
                  "start": 17,
                  "end": 24,
                  "name": "console"
                },
                "property": {
                  "type": "Identifier",
                  "start": 25,
                  "end": 28,
                  "name": "log"
                },
                "computed": false
              },
              "arguments": [
                {
                  "type": "Identifier",
                  "start": 29,
                  "end": 30,
                  "name": "x"
                },
                {
                  "type": "Identifier",
                  "start": 31,
                  "end": 32,
                  "name": "y"
                }
              ]
            }
          }
        ]
      }
    }
  ],
  "sourceType": "module"
}
```



**转换阶段**

在这个阶段，Babel接受得到AST并通过babel-traverse对其进行深度优先遍历，在此过程中对节点进行添加、更新及移除操作。这部分也是Babel插件介入工作的部分。

进入`traverse`方法调用

![WX20191203-155921@2x](http://www.qinhanwen.xyz/WX20191203-155921@2x.png)

进入`traverse.node`方法调用，从根开始的深度优先遍历

![WX20191203-171623@2x](http://www.qinhanwen.xyz/WX20191203-171623@2x.png)

进入`context.visit`方法调用，根据nodes是否是数组，调用不同的visit方法

![WX20191203-171845@2x](http://www.qinhanwen.xyz/WX20191203-171845@2x.png)

进入`visitSingle`方法

![WX20191204-151112@2x](http://www.qinhanwen.xyz/WX20191204-151112@2x.png)

进入`visitQueue`方法

![WX20191204-151212@2x](http://www.qinhanwen.xyz/WX20191204-151212@2x.png)

这里调用`enter`

![WX20191203-173447@2x](http://www.qinhanwen.xyz/WX20191203-173447@2x.png)

从opts里拿到enter的数组

![WX20191204-151428@2x](http://www.qinhanwen.xyz/WX20191204-151428@2x.png)

遍历fns，并调用

![WX20191204-151501@2x](http://www.qinhanwen.xyz/WX20191204-151501@2x.png)

进入enter函数调用

![WX20191204-151517@2x](http://www.qinhanwen.xyz/WX20191204-151517@2x.png)

打印出来的enter的路径，也就是深度遍历的顺序。

![WX20191203-173148@2x](http://www.qinhanwen.xyz/WX20191203-173148@2x.png)



**生成阶段**

将经过转换的AST通过babel-generator再转换成js代码，过程就是深度优先遍历整个AST，然后构建可以表示转换后代码的字符串。

进入`generate`方法

![WX20191204-152451@2x](http://www.qinhanwen.xyz/WX20191204-152451@2x.png)

进入`gen.generate`调用，`this.ast`是在`new Generator(ast, opts, code)`的时候挂载在实例上的

![WX20191204-152756@2x](http://www.qinhanwen.xyz/WX20191204-152756@2x.png)

进入`generate`方法

![WX20191204-165751@2x](http://www.qinhanwen.xyz/WX20191204-165751@2x.png)

在`print`方法里做的对节点解析并且`push`到`_buf`数组里

![WX20191204-165552@2x](http://www.qinhanwen.xyz/WX20191204-165552@2x.png)

最后调用`this._buf.get`方法，把字符串合并

![WX20191204-170009@2x](http://www.qinhanwen.xyz/WX20191204-170009@2x.png)



## 参考资料

[深入Babel，这一篇就够了](https://juejin.im/post/5c21b584e51d4548ac6f6c99)

[babel文档](https://babeljs.io/)

[一口(很长的)气了解 babel](https://juejin.im/post/5c19c5e0e51d4502a232c1c6)

[深入浅出 Babel 上篇：架构和原理 + 实战](https://juejin.im/post/5d94bfbf5188256db95589be)