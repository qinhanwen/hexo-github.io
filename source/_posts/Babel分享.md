---
title: Babel分享
date: 2019-12-10 21:05:05
tags: 
- Babel
categories: 
- Babel
---

![WX20191202-201539@2x](http://www.qinhanwen.xyz/WX20191202-201539@2x.png)

- Babel 
  - Babel是什么
  - ast是什么
  - 访问者模式
  - charCodeAt
- 插件
  - @babel/parser 7.x（babylon 6.x）
  - @babel/traverse 7.x（babel-traverse 6.x）
  - @babel/generator 7.x （babel-generator 6.x）
- 过程
  - 解析
    - 词法分析
    - 语法分析
  - 转换
    - 访问者模式
  - 生成



## Babel

**Babel是什么**

简单来说把 JavaScript 中 es6/7/8 之类的的新语法转化为 es5，让低端运行环境(如浏览器和 node )能够认识并执行。babel 也可以转化为更低的规范。但以目前情况来说，es5 规范已经足以覆盖绝大部分浏览器，因此常规来说转到 es5 是一个安全且流行的做法。



**[ast（Abstract Syntax Tree）](https://astexplorer.net/)**

抽象语法树：

源代码语法结构的一种抽象表示。它以树状的形式表现编程语言的语法结构，树上的每个节点都表示源代码中的一种结构。之所以说语法是“抽象”的，是因为这里的语法并不会表示出真实语法中出现的每个细节。

简单来说就是对象描述语法结构。



应用场景：

- eslint对代码错误或风格的检查，发现一些潜在的错误
- IDE的错误提示、格式化、高亮、自动补全等
- UglifyJS压缩代码
- 代码打包工具webpack



**访问者模式**

访问者模式：封装一些作用于某种数据结构中的各元素的操作，它可以在不改变这个数据结构的前提下定义作用于这些元素的新的操作。

举个例子：我去朋友家做客，那么朋友属于主人，我则属于访问者。这时刚好朋友在炒菜，却没得酱油了。如果朋友下去买酱油将会很麻烦而且会影响炒菜。这时就到我这个访问者出马了。一溜烟的出去打着酱油就回来了。简单理解的来说就是，访问者在主人原来的基础上帮助主人去完成主人不方便或者完不成的东西。

```javascript
var Visitor = {} // 访问者对象
Visitor.push = function() { // 定义新的操作
    return Array.prototype.push.apply( this, arguments );
}
var obj = {};
obj.push = Visitor.push;
obj.push('first');
console.log(obj[0]) // "first"
console.log(obj.length);  // 1
```



> [Array.prototype.push](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/push):
>
> push方法将值追加到数组中，`push` 方法具有通用性。该方法和 [`call()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call) 或 [`apply()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 一起使用时，可应用在类似数组的对象上。`push` 方法根据 `length` 属性来决定从哪里开始插入给定的值。如果 `length` 不能被转成一个数值，则插入的元素索引为 0，包括 `length` 不存在时。当 `length` 不存在时，将会创建它。
>

```javascript
 var obj = {length:2}
 Array.prototype.push.call(obj,1);
 console.log(obj);// { 2:1, length:3 }
```



**[String.prototype.charCodeAt](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt)：返回字符所代表的unicode编码，返回值是0~65535之间的整数**

```javascript
'a'.charCodeAt(0); //97
'A'.charCodeAt(0); //65
```

[unicode字符列表](http://www.52unicode.com/)



## 插件

**[@babel/parser](https://babeljs.io/docs/en/next/babel-parser.html)：Babel解析器（以前是Babylon）是Babel中使用的JavaScript解析器，以前基于acorn**



**[@babel/traverse](https://babeljs.io/docs/en/next/babel-traverse.html)：Babel Traverse（遍历）模块维护了整棵树的状态，并且负责替换、移除和添加节点**



**[@babel/generator](https://babeljs.io/docs/en/next/babel-generator.html)：Babel 的代码生成器，它读取AST并将其转换为代码** 



## 过程

举个🌰

```javascript
const {parse} = require('babylon');

const code = `a==b`;
const ast = parse(code);
```



**解析**

- 词法分析：把字符串形式的代码转换为令牌（token）

- 语法分析：把令牌（token）转换成 AST的形式，同时这个阶段会把令牌中的信息转换成AST的表述结构



1. Parser 实例

实例化的过程中做 state 和其他属性的初始化，之后生成之后调用 parse 方法

```javascript
function parse(input, options) {
  return new _parser2["default"](options, input).parse();
}
```

```javascript
  function Parser(options, input) {
    _classCallCheck(this, Parser);

    options = _options.getOptions(options);
    
    // state 初始化的地方
    _Tokenizer.call(this, options, input);

    this.options = options;
    this.inModule = this.options.sourceType === "module";
    this.isReservedWord = _utilIdentifier.reservedWords[6];
    this.input = input;
    this.plugins = this.loadPlugins(this.options.plugins);

    if (this.state.pos === 0 && this.input[0] === "#" && this.input[1] === "!") {
      this.skipLineComment(2);
    }
  }
```



2. state 属性

用于保存解析的指针、值与其他位置状态信息等

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
      value: "a" // 属性
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



3. next 方法

更新state指针、tokens状态。最后并且调用nextToken开始解析后面的内容

```javascript
  Tokenizer.prototype.next = function next() {
    if (!this.isLookahead) {
      this.state.tokens.push(new Token(this.state));
    }

    // 存储上一个被解析内容的开始结束位置等信息
    this.state.lastTokEnd = this.state.end;
    this.state.lastTokStart = this.state.start;
    this.state.lastTokEndLoc = this.state.endLoc;
    this.state.lastTokStartLoc = this.state.startLoc;

    // 解析后面内容
    this.nextToken();
  };
```



4. nextToken 方法

```javascript
  Tokenizer.prototype.nextToken = function nextToken() {
    var curContext = this.curContext();
    if (!curContext || !curContext.preserveSpace) this.skipSpace();

    this.state.containsOctal = false;
    this.state.octalPosition = null;
    
    // 更新解析起点位置信息
    this.state.start = this.state.pos;
    this.state.startLoc = this.state.curPosition();
    
    // 如果解析完了，就结束
    if (this.state.pos >= this.input.length) return this.finishToken(_types.types.eof);

    if (curContext.override) {
      return curContext.override(this);
    } else {
      return this.readToken(this.fullCharCodeAtPos());
    }
  };
```



5. readWord1 方法

词法分析过程，根据unicode的值比较，遇到`\` 、`.`之类终止循环。

```javascript
  Tokenizer.prototype.readWord1 = function readWord1() {
    this.state.containsEsc = false;
    var word = "",
        first = true,
        chunkStart = this.state.pos;
    
    // 循环从起始位置开始
    while (this.state.pos < this.input.length) {
      var ch = this.fullCharCodeAtPos();
      if (_utilIdentifier.isIdentifierChar(ch)) {
        this.state.pos += ch <= 0xffff ? 1 : 2;
      } else if (ch === 92) {
        // "\"
        this.state.containsEsc = true;

        word += this.input.slice(chunkStart, this.state.pos);
        var escStart = this.state.pos;

        if (this.input.charCodeAt(++this.state.pos) !== 117) {
          // "u"
          this.raise(this.state.pos, "Expecting Unicode escape sequence \\uXXXX");
        }

        ++this.state.pos;
        var esc = this.readCodePoint();
        if (!(first ? _utilIdentifier.isIdentifierStart : _utilIdentifier.isIdentifierChar)(esc, true)) {
          this.raise(escStart, "Invalid Unicode escape");
        }

        word += codePointToString(esc);
        chunkStart = this.state.pos;
      } else {
        break;
      }
      first = false;
    }
    // 从 input 表达式中，根据位置信息截取出
    return word + this.input.slice(chunkStart, this.state.pos);
  };
```



6. finishToken 方法

词法分析结束，更新 state 属性上的信息

```javascript
  Tokenizer.prototype.finishToken = function finishToken(type, val) {
    // 更新位置信息、值和type
    this.state.end = this.state.pos;
    this.state.endLoc = this.state.curPosition();
    var prevType = this.state.type;
    this.state.type = type;
    this.state.value = val;

    this.updateContext(prevType);
  };
```



finishNode 方法

生成节点

```javascript
pp.finishNode = function (node, type) {
  return finishNodeAt.call(this, node, type, this.state.lastTokEnd, this.state.lastTokEndLoc);
};
```



7. finishNodeAt 方法

设置node（节点）属性，并返回node

```javascript
function finishNodeAt(node, type, pos, loc) {
  node.type = type;
  node.end = pos;
  node.loc.end = loc;
  this.processComment(node);
  return node;
}
```

![WX20191211-161817@2x](http://www.qinhanwen.xyz/WX20191211-161817@2x.png)



**转换**

第二个🌰

```javascript
const {
    parse
} = require('babylon');

const code = `a==b`;
const ast = parse(code);

// 新增部分
const traverse = require("babel-traverse").default;

const visitor = {
	  // 对语法树中特定的节点进行操作
    Identifier: {
        // 进入节点事件
        enter(path) {
            console.log(path.node.name)
        }
    }
}

traverse(ast, visitor);
```



1. visit 方法

会对节点遍历，单个节点和多个节点不同的处理，对具体的节点调用 visit 方法处理

对遍历到的节点处理

```javascript
function visit() {
  if (!this.node) {
    return false;
  }

  if (this.isBlacklisted()) {
    return false;
  }

  if (this.opts.shouldSkip && this.opts.shouldSkip(this)) {
    return false;
  }

  if (this.call("enter") || this.shouldSkip) {
    this.debug(function () {
      return "Skip...";
    });
    return this.shouldStop;
  }

  this.debug(function () {
    return "Recursing into...";
  });
  _index2.default.node(this.node, this.opts, this.scope, this.state, this, this.skipKeys);

  this.call("exit");

  return this.shouldStop;
}
```

a 节点

![WX20191211-170048@2x](http://www.qinhanwen.xyz/WX20191211-170048@2x.png)

call 方法

![WX20191211-171102@2x](http://www.qinhanwen.xyz/WX20191211-171102@2x.png)

调用访问者中设置的 enter 方法

![WX20191211-171601@2x](http://www.qinhanwen.xyz/WX20191211-171601@2x.png)

![WX20191211-171728@2x](http://www.qinhanwen.xyz/WX20191211-171728@2x.png)



**生成**

第三个🌰

```javascript
const {
    parse
} = require('babylon');

const code = `a==b`;
const ast = parse(code);

const traverse = require("babel-traverse").default;

const visitor = {
    Identifier: {
        enter(path) {
            // 修改类型是 Identifier， name 是 a 的节点
            if (path.isIdentifier({
                    name: "a"
                })) {
                path.node.name = "x";
            }
        }
    }
}

traverse(ast, visitor);

const generate = require('babel-generator').default;

const output = generate(ast);
console.log(output);// { code: 'x == b;', map: null, rawMappings: null }
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

![WX20191211-175602@2x](http://www.qinhanwen.xyz/WX20191211-175602@2x.png)

```javascript
function Identifier(node) {
  if (node.variance) {
    if (node.variance === "plus") {
      this.token("+");
    } else if (node.variance === "minus") {
      this.token("-");
    }
  }

  this.word(node.name); // node.name 之后会被 push 到 _buf 中
}
```



3. append 方法

在 _append 方法里，向 _buf 数组里 push 解析出的内容，就是node.name

```javascript
  Buffer.prototype.append = function append(str) {
    this._flush();
    var _sourcePosition = this._sourcePosition,
        line = _sourcePosition.line,
        column = _sourcePosition.column,
        filename = _sourcePosition.filename,
        identifierName = _sourcePosition.identifierName;

    this._append(str, line, column, identifierName, filename);
  };
```



3. get 方法

合并并返回结果

![WX20191211-172510@2x](http://www.qinhanwen.xyz/WX20191211-172510@2x.png)

```javascript
  Buffer.prototype.get = function get() {
    this._flush();

    var map = this._map;
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

[String.prototype.codePointAt 与  String.prototype.charCodeAt](https://www.jianshu.com/p/91dad99d238c)







