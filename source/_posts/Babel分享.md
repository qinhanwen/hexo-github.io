---
title: Babel分享
date: 2019-12-10 21:05:05
tags:
  - Babel
categories:
  - Babel
---

![WX20191202-201539@2x](http://114.55.30.96/WX20191202-201539@2x.png)

- **Babel**
  - **Babel 是什么**
  - **AST 是什么**
  - **访问者模式**
  - **charCodeAt**
- 其他内容简单介绍
  - cli
  - plugin
  - presets
  - polyfill
  - babel-runtime & babel-plugin-transform-runtime
  - .babelrc、babel.config.js、package.json 等配置
- **插件**
  - **@babel/parser 7.x（babylon 6.x）**
  - **@babel/traverse 7.x（babel-traverse 6.x）**
  - **@babel/generator 7.x （babel-generator 6.x）**
  - @babel/types
- **过程**
  - **解析**
    - **词法分析**
    - **语法分析**
  - **转换**
    - **访问者模式**
  - **生成**
- 补充
  - String.prototype.codePointAt 与 String.prototype.charCodeAt

## Babel

**Babel 是什么**

Babel 是 JavaScript 编译器 ， 简单来说是把 JavaScript 中 es6/7/8 之类的的新语法转化为 es5，让低端运行环境(如浏览器和 node )能够认识并执行。babel 也可以转化为更低的规范。但以目前情况来说，es5 规范已经足以覆盖绝大部分浏览器，因此常规来说转到 es5 是一个安全且流行的做法。

**[AST（Abstract Syntax Tree）](https://astexplorer.net/)**

抽象语法树：

源代码语法结构的一种抽象表示。它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。之所以说语法是“抽象”的，是因为这里的语法并不会表示出真实语法中出现的每个细节。

简单来说就是对象描述语法结构。

应用场景：

- eslint 对代码错误或风格的检查，发现一些潜在的错误
- IDE 的错误提示、格式化、高亮、自动补全等
- UglifyJS 压缩代码
- 代码打包工具 webpack，从入口模块解析，寻找依赖模块

![](http://114.55.30.96/WX20191020-140848@2x.png)

**访问者模式**

访问者模式：封装一些作用于某种数据结构中的各元素的操作，它可以在不改变这个数据结构的前提下定义作用于这些元素的新的操作。

举个例子：我去朋友家做客，那么朋友属于主人，我则属于访问者。这时刚好朋友在炒菜，却没得酱油了。如果朋友下去买酱油将会很麻烦而且会影响炒菜。这时就到我这个访问者出马了。一溜烟的出去打着酱油就回来了。简单理解的来说就是，访问者在主人原来的基础上帮助主人去完成主人不方便或者完不成的东西。

```javascript
var Visitor = {} // 访问者对象
Visitor.push = function() {
  // 定义新的操作
  return Array.prototype.push.apply(this, arguments)
}
var obj = {}
obj.push = Visitor.push
obj.push('first')
console.log(obj[0]) // "first"
console.log(obj.length) // 1
```

> [Array.prototype.push](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/push):
>
> push 方法将值追加到数组中，`push` 方法具有通用性。该方法和 [`call()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call) 或 [`apply()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 一起使用时，可应用在类似数组的对象上。`push` 方法根据 `length` 属性来决定从哪里开始插入给定的值。如果 `length` 不能被转成一个数值，则插入的元素索引为 0，包括 `length` 不存在时。当 `length` 不存在时，将会创建它。

```javascript
var obj = { length: 2 }
Array.prototype.push.call(obj, 1)
console.log(obj) // { 2:1, length:3 }
```

**[String.prototype.charCodeAt](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt)：返回字符所代表的 unicode 编码，返回值是 0~65535 之间的整数**

```javascript
'a'.charCodeAt(0) //97
'A'.charCodeAt(0) //65
```

[unicode 字符列表](http://www.52unicode.com/)

## 其他内容简单介绍

**[cli]()**

命令行工具，可通过命令行编译文件。

```shell
# 安装
$ npm install @babel/core @babel/cli -D

# 编译 script.js 文件并 输出到标准输出设备（stdout）。
$ npx babel script.js

# 如果你希望 输出到文件 ，可以使用 --out-file 或 -o 参数。
$ npx babel script.js --out-file script-compiled.js

# 要在 每次文件修改后 编译该文件，请使用 --watch 或 -w 参数：
$ npx babel script.js --watch --out-file script-compiled.js

# 如果你希望输出 源码映射表（source map）文件 ，你可以使用 --source-maps 或 -s 参数。
$ npx babel script.js --out-file script-compiled.js --source-maps

# 编译整个 src 目录下的文件并输出到 lib 目录，输出目录可以通过 --out-dir 或 -d 指定。这不会覆盖 lib 目录下的任何其他文件或目录。
$ npx babel src --out-dir lib

# 编译整个 src 目录下的文件并将输出合并为一个文件。
$ npx babel src --out-file script-compiled.js
```

**[plugin](https://www.babeljs.cn/docs/plugins)**

Babel 作为 JavaScript 编译器，有三个过程：解析、转换、生成。如果什么都不做。它基本上只是将代码解析之后再输出同样的代码。如果想要 Babel 做一些实际的工作，就需要为其添加插件。

插件的作用：用于转换代码

[babel-plugin-transform-remove-console](https://www.babeljs.cn/docs/babel-plugin-transform-remove-console)，用于移除代码中的 console，上生产环境其实这些都是不需要的。可以减少包体积。

**[presets（预设）](https://www.babeljs.cn/docs/presets)**

presets 可以作为 Babel 插件的套餐。

预设的作用：不用自己组合插件

[@babel/preset-env](https://www.babeljs.cn/docs/babel-preset-env)：这个插件整合了很多插件（比如要转义箭头函数，需要使用到 `@babel/plugin-transform-arrow-functions` 这个插件，转义 `class` 需要使用 `@babel/plugin-transform-classes`）。但是一些 API 没有转换（ Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise），导致一些新的 API 老版浏览器不兼容。所以这个时候就需要一些工具来为浏览器做这个兼容。

![WX20191211-222825@2x](http://114.55.30.96/WX20191211-222825@2x.png)

**[@babel/polyfill](https://babeljs.io/docs/en/6.26.3/babel-polyfill)**

可以使用新的 API（例如 Promise 或 WeakMap），静态方法（例如 Array.from 或 Object.assign），实例方法（例如 Array.prototype.includes）等。为了做到这一点，polyfill 把这些添加到了全局环境上。在 CommonJS 或者 ES module 环境下使用，都需要在入口顶部引入它。

缺点：

1. babel-polyfill 会污染全局变量，给很多类的原型链上都作了修改，这就有不可控的因素存在。
2. 使用 babel-polyfill 会导致打出来的包非常大，很多其实没有用到，对资源来说是一种浪费。

因为上面的问题，所以在 Babel7 中增加了 babel-preset-env，我们设置 `"useBuiltIns": "usage"` 这个参数值就可以实现按需加载 babel-polyfill （也就是有使用到的 API 才引入 polyfill 版本）

Babel 7.4.0 开始不推荐使用这个软件包

**[@babel-runtime & babel-plugin-transform-runtime](https://babeljs.io/docs/en/babel-runtime)**

因为 babel-polyfill 会污染全局作用域，还有 Array、String 等的静态与实例属性。

Babel 为了解决这个问题，提供了单独的包 babel-runtime 用以提供编译模块的工具函数。将 Babel 的运行时的 “ polyfilling” 行为分开了。也就是说不再污染全局，**将运行时的帮助函数抽离出来管理**。

使用`类转换`插件：

```javascript
class Circle {}

// 编译的结果 ，包含了辅助函数，也就是说每个文件里面都会包含这个辅助函数

function _classCallCheck(instance, Constructor) {
  //...
}

var Circle = function Circle() {
  _classCallCheck(this, Circle)
}
```

如果使用@ babel / plugin-transform-runtime，它将把对该函数的引用替换为 @babel / runtime 版本：

```javascript
var _classCallCheck = require('@babel/runtime/helpers/classCallCheck')

var Circle = function Circle() {
  _classCallCheck(this, Circle)
}
```

**.babelrc、babel.config.js、package.json 等配置**

.babelrc

使用场景

- 只是需要一个简单的并且只用于单个软件包的配置

在你的项目中创建名为 `.babelrc` 的文件，并输入以下内容。

```
{
  "presets": [...],
  "plugins": [...]
}
```

babel.config.js

使用场景

- 希望以编程的方式创建配置文件
- 希望编译 `node_modules` 目录下的模块

在项目的根目录（`package.json` 文件所在目录）下创建一个名为 `babel.config.js` 的文件，并输入如下内容。

```javascript
module.exports = function (api) {
  const presets = [ ... ];
  const plugins = [ ... ];

  return {
    presets,
    plugins
  };
}
```

package.json

如下所示：

```json
{
	...

  "babel": {
    "presets": [ ... ],
    "plugins": [ ... ],
  }
}
```

## 插件

**[@babel/parser](https://babeljs.io/docs/en/next/babel-parser.html)：Babel 解析器（以前是 Babylon）是 Babel 中使用的 JavaScript 解析器，以前基于 acorn 库 fork 出来的**

**[@babel/traverse](https://babeljs.io/docs/en/next/babel-traverse.html)：Babel Traverse（遍历）模块维护了整棵树的状态，并且负责替换、移除和添加节点**

**[@babel/generator](https://babeljs.io/docs/en/next/babel-generator.html)：Babel 的代码生成器，它读取 AST 并将其转换为代码**

**[@babel/types](https://babeljs.io/docs/en/next/babel-types.html)：**Babel Types 模块是一个用于 AST 节点的 Lodash 式工具库， 它包含了构造、验证以及变换 AST 节点的方法。 该工具库包含考虑周到的工具方法，对编写处理 AST 逻辑非常有用。

```javascript
// 依赖版本 7.x
const { parse } = require('@babel/parser')
const traverse = require('@babel/traverse').default
const { is } = require('@babel/types')

const code = `function square(n) {
    return n * n;
  }`
const ast = parse(code)
traverse(ast, {
  enter(path) {
    if (is('Identifier', path.node)) {
      // 判断节点类型
      console.log(path.node.name)
    }
  }
})
```

## 过程

使用插件版本 6.x

举个 🌰

```javascript
const { parse } = require('babylon')

const code = `a==b`
const ast = parse(code)
```

解析出的 AST

```json
{
  "type": "Program",
  "start": 0,
  "end": 4,
  "body": [
    {
      "type": "ExpressionStatement",
      "start": 0,
      "end": 4,
      "expression": {
        "type": "BinaryExpression",
        "start": 0,
        "end": 4,
        "left": {
          "type": "Identifier",
          "start": 0,
          "end": 1,
          "name": "a"
        },
        "operator": "==",
        "right": {
          "type": "Identifier",
          "start": 3,
          "end": 4,
          "name": "b"
        }
      }
    }
  ],
  "sourceType": "module"
}
```

**解析**

- 词法分析：把字符串形式的代码转换为令牌（token）

在解析生成节点之前，会使用 state 属性的值生成 token

![WX20191217-203855@2x](http://114.55.30.96/WX20191217-203855@2x.png)

生成的 token 的样子

![WX20191214-153943@2x](http://114.55.30.96/WX20191214-153943@2x.png)

- 语法分析：这个阶段会把令牌中的信息转换成 AST 的表述结构

next 里生成 token，finishNode 生成节点

![WX20191217-204104@2x](http://114.55.30.96/WX20191217-204104@2x.png)

1. Parser 实例

实例化的过程中做 state 和其他属性的初始化，之后生成之后调用 parse 方法

```javascript
function parse(input, options) {
  return new _parser2['default'](options, input).parse()
}
```

```javascript
  function Parser(options, input) {
    ...

    // state 初始化的地方
    _Tokenizer.call(this, options, input);

    ...

    this.input = input;

		...
  }

  Parser.prototype.parse = function parse() {
    var file = this.startNode();
    var program = this.startNode();
    this.nextToken();
    return this.parseTopLevel(file, program);
  };
```

得到的 Parser 实例

```json
{
    ...

    input: 'a==b',
    state: {
      ...
    }

    ...
}
```

2. state 属性

用于在解析过程中，用于保存解析的指针、值与其他位置状态信息等

```json
{
			...
      input: "a==b"// 表达式
      start: 0 // 起始位置
  		end: 1 // 结束位置
      pos: 1 // 解析到达的位置，会作为下一个解析位置的起点
      startLoc: Position {line: 1, column: 0} // 起始位置信息 行与列
			endLoc: Position {line: 1, column: 1} // 结束位置信息 行与列
      tokens: [] // tokens数组
      value: "a" // 属性值
			lastTokEnd: 1 // 上一个被解析的结束位置
			lastTokStart: 0 // 上一个被解析的起始位置
			lastTokStartLoc:  Position {line: 1, column: 0} // 上一个被解析的起始位置的位置信息 行与列
			lastTokEndLoc: Position {line: 1, column: 1} // 上一个被解析的结束位置的位置信息 行与列
			type: TokenType { // token类型，根据不同类型做处理
        label: "name",
        keyword: undefined,
        beforeExpr: false,
        startsExpr: true,
        rightAssociative: false,
        …
      }
			...
}
```

3. readWord1 方法

词法分析过程，根据 unicode 的值比较，遇到`\` 、`.`之类终止循环。

```javascript
Tokenizer.prototype.readWord1 = function readWord1() {
  this.state.containsEsc = false
  var word = '',
    first = true,
    chunkStart = this.state.pos

  // 循环从起始位置开始
  while (this.state.pos < this.input.length) {
    var ch = this.fullCharCodeAtPos()
    if (_utilIdentifier.isIdentifierChar(ch)) {
      this.state.pos += ch <= 0xffff ? 1 : 2
    } else if (ch === 92) {
      // "\"
      this.state.containsEsc = true

      word += this.input.slice(chunkStart, this.state.pos)
      var escStart = this.state.pos

      if (this.input.charCodeAt(++this.state.pos) !== 117) {
        // "u"
        this.raise(this.state.pos, 'Expecting Unicode escape sequence \\uXXXX')
      }

      ++this.state.pos
      var esc = this.readCodePoint()
      if (
        !(first
          ? _utilIdentifier.isIdentifierStart
          : _utilIdentifier.isIdentifierChar)(esc, true)
      ) {
        this.raise(escStart, 'Invalid Unicode escape')
      }

      word += codePointToString(esc)
      chunkStart = this.state.pos
    } else {
      break
    }
    first = false
  }
  // 从 input 表达式中，根据位置信息截取出
  return word + this.input.slice(chunkStart, this.state.pos)
}
```

4. next 方法

更新 state 指针、tokens 状态。最后并且调用 nextToken 开始解析后面的内容

```javascript
Tokenizer.prototype.next = function next() {
  if (!this.isLookahead) {
    this.state.tokens.push(new Token(this.state))
  }

  // 存储上一个被解析内容的开始结束位置等信息
  this.state.lastTokEnd = this.state.end
  this.state.lastTokStart = this.state.start
  this.state.lastTokEndLoc = this.state.endLoc
  this.state.lastTokStartLoc = this.state.startLoc

  // 解析后面内容
  this.nextToken()
}
```

5. nextToken 方法

```javascript
Tokenizer.prototype.nextToken = function nextToken() {
  var curContext = this.curContext()
  if (!curContext || !curContext.preserveSpace) this.skipSpace()

  this.state.containsOctal = false
  this.state.octalPosition = null

  // 更新解析起点位置信息
  this.state.start = this.state.pos
  this.state.startLoc = this.state.curPosition()

  // 如果解析完了，就结束
  if (this.state.pos >= this.input.length)
    return this.finishToken(_types.types.eof)

  if (curContext.override) {
    return curContext.override(this)
  } else {
    return this.readToken(this.fullCharCodeAtPos())
  }
}
```

6. finishToken 方法

词法分析结束，更新 state 属性上的信息

```javascript
Tokenizer.prototype.finishToken = function finishToken(type, val) {
  // 更新位置信息、值和type
  this.state.end = this.state.pos
  this.state.endLoc = this.state.curPosition()
  var prevType = this.state.type
  this.state.type = type
  this.state.value = val

  this.updateContext(prevType)
}
```

7. finishNode 方法

生成节点

```javascript
pp.finishNode = function(node, type) {// 传入的 node 节点上的参数，都是从 state 属性上拿的
  return finishNodeAt.call(
    this,
    node,
    type,
    this.state.lastTokEnd,
    this.state.lastTokEndLoc
  )
}
```

8. finishNodeAt 方法

设置 node（节点）属性，并返回 node

```javascript
function finishNodeAt(node, type, pos, loc) { // pos loc 也是 state属性 上拿的
  node.type = type
  node.end = pos
  node.loc.end = loc
  this.processComment(node)
  return node
}
```

![WX20191211-161817@2x](http://114.55.30.96/WX20191211-161817@2x.png)

简单理解

```javascript
const input = 'a'

let state = {
  start: 0,
  end: 0,
  pos: 0,
  value: '',
  type: '',
  input: 'a'
}

parse()

function parse() {
  // 词法解析
  let chunkStart = state.pos // 解析开始的位置
  while (state.pos <= input.length) {
    let ch = input.charCodeAt(state.pos) // 起始位置的 unicode 值 97
    if (ch < 123 && ch >= 97) {
      //  a 是 0x7b，判断条件是 a-z 之间
      state.pos += 1
    } else {
      // 如果不满足就更新状态
      const word = input.substr(chunkStart, state.pos) // 这边解析出 a
      state.value = word;
      
      var node = new Node(state.start, state.pos) // 生成 node
      node.name = state.value
      console.log(finishNode(node, 'Indentifier'))
      // 输出节点
      // Node { start: 0, end: 1, type: 'Indentifier', name: 'a' }
      
      break
    }
  }
}

function Node(start, end) {
  // 节点构造函数
  this.start = start
  this.end = end
}

function finishNode(node, type) {
  node.type = type
  return node
}
```

**转换**

第二个 🌰

```javascript
const { parse } = require('babylon')

const code = `a==b`
const ast = parse(code)

// 新增部分
const traverse = require('babel-traverse').default

const visitor = {
  // 对语法树中特定的节点进行操作
  Identifier: {
    // 进入节点事件
    enter(path) {
      console.log(path.node.name)
    }
  }
}

traverse(ast, visitor)
```

1. visit 方法

会对节点遍历，单个节点和多个节点不同的处理，对具体的节点调用 visit 方法处理

对遍历到的节点处理

```javascript
function visit() {

  ...

  if (this.call("enter") || this.shouldSkip) {
    this.debug(function () {
      return "Skip...";
    });
    return this.shouldStop;
  }

	...

  this.call("exit");

  return this.shouldStop;
}

```

a 节点

![WX20191211-170048@2x](http://114.55.30.96/WX20191211-170048@2x.png)

call 方法

![WX20191211-171102@2x](http://114.55.30.96/WX20191211-171102@2x.png)

调用访问者中设置的 enter 方法

![WX20191211-171601@2x](http://114.55.30.96/WX20191211-171601@2x.png)

![WX20191211-171728@2x](http://114.55.30.96/WX20191211-171728@2x.png)

**生成**

第三个 🌰

```javascript
const { parse } = require('babylon')

const code = `a==b`
const ast = parse(code)

const traverse = require('babel-traverse').default

const visitor = {
  Identifier: {
    enter(path) {
      // 修改类型是 Identifier， name 是 a 的节点
      if (
        path.isIdentifier({
          name: 'a'
        })
      ) {
        path.node.name = 'x'
      }
    }
  }
}

traverse(ast, visitor)

const generate = require('babel-generator').default

const output = generate(ast)
console.log(output) // { code: 'x == b;', map: null, rawMappings: null }
```

1. print 方法

遍历节点

```javascript
  Printer.prototype.print = function print(node, parent) {
    var _this = this;

    ...

    this.withSource("start", loc, function () {
      // 根据节点的 type 处理，最后还是调用到 print 方法
      _this[node.type](node, parent);
    });

    ...

  };

```

2. Identifier 方法

当节点类型是 Identifier 的时候

![WX20191211-175602@2x](http://114.55.30.96/WX20191211-175602@2x.png)

```javascript
function Identifier(node) {

  ...

  this.word(node.name); // node.name 之后会被 push 到 _buf 中
}

```

3. append 方法

在 \_append 方法里，向 \_buf 数组里 push 解析出的内容，就是 node.name

```javascript
  Buffer.prototype.append = function append(str) {

    ...

    this._append(str, line, column, identifierName, filename);
  };

```

4. get 方法

合并并返回结果

![WX20191211-172510@2x](http://114.55.30.96/WX20191211-172510@2x.png)

```javascript
  Buffer.prototype.get = function get() {

    ...

    var result = {
      // 数组 join 操作合并
      code: (0, _trimRight2.default)(this._buf.join("")),
      map: null,
      rawMappings: map && map.getRawMappings()
    };

    ...

    return result;
  };

```

## 补充

[String.prototype.codePointAt 与 String.prototype.charCodeAt](https://www.jianshu.com/p/91dad99d238c)
