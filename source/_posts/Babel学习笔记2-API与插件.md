---
title: Babel学习笔记3 - API与插件
date: 2019-12-04 21:40:25
tags: 
- Babel
categories: 
- Babel
---

## API

**[@babel/parser](https://babeljs.io/docs/en/next/babel-parser.html)：**Babel 的解析器。最初是 从Acorn项目fork出来的。Acorn非常快，易于使用，并且针对非标准特性(以及那些未来的标准特性) 设计了一个基于插件的架构。

```javascript
const {
    parse
} = require('@babel/parser');

const code = `function square(n) {
    return n * n;
  }`;
const ast = parse(code);
```



**[@babel/traverse](https://babeljs.io/docs/en/next/babel-traverse.html)：**Babel Traverse（遍历）模块维护了整棵树的状态，并且负责替换、移除和添加节点。

我们可以和 **@babel/parser** 一起使用来遍历和更新节点：

```javascript
const {
    parse
} = require('@babel/parser');
const traverse = require('@babel/traverse').default;

const code = `function square(n) {
    return n * n;
  }`;
const ast = parse(code);
traverse(ast, {
    FunctionDeclaration(path) {
        const param = path.node.params[0];
        paramName = param.name;
        param.name = "x";
    },

    Identifier(path) {
        if (path.node.name === paramName) {
            path.node.name = "x";
        }
    }
});
```



**[@babel/types](https://babeljs.io/docs/en/next/babel-types.html)：**Babel Types模块是一个用于 AST 节点的 Lodash 式工具库， 它包含了构造、验证以及变换 AST 节点的方法。 该工具库包含考虑周到的工具方法，对编写处理AST逻辑非常有用。

```javascript
const {
    parse
} = require('@babel/parser');
const traverse = require('@babel/traverse').default;
const {
    is
} = require('@babel/types');

const code = `function square(n) {
    return n * n;
  }`;
const ast = parse(code);
traverse(ast, {
    enter(path) {
        if (is("Identifier", path.node)) {
            console.log(path.node.name);
        }
    }
});
```



**[@babel/generator](https://babeljs.io/docs/en/next/babel-generator.html)：** Babel 的代码生成器，它读取AST并将其转换为代码和源码映射（sourcemaps）。

```javascript
const {
    parse
} = require('@babel/parser');
const generate = require('@babel/generator').default;

const code = `function square(n) {
    return n * n;
  }`;
const ast = parse(code);
const output = generate(ast);
console.log(output)
```



## 编写第一个Babel插件

先从一个接收了当前`babel`对象作为参数的 `function` 开始。

```javascript
export default function(babel) {
  // plugin contents
}
```

由于你将会经常这样使用，所以直接取出 `babel.types` 会更方便

```javascript
export default function({ types: t }) {
  // plugin contents
}
```

接着返回一个对象，其 `visitor` 属性是这个插件的主要访问者。

```javascript
export default function({ types: t }) {
  return {
    visitor: {
      // visitor contents
    }
  };
};
```

Visitor 中的每个函数接收2个参数：`path` 和 `state`

```javascript
export default function({ types: t }) {
  return {
    visitor: {
      Identifier(path, state) {},
      ASTNodeTypeHere(path, state) {}
    }
  };
};
```

让我们快速编写一个可用的插件来展示一下它是如何工作的。下面是我们的源代码：

```javascript
foo === bar;
```

其 AST 形式如下：

```json
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

![WX20191205-104742@2x](http://118.24.241.76/WX20191205-104742@2x.png)

我们从添加 `BinaryExpression` 访问者方法开始：

```javascript
export default function({ types: t }) {
  return {
    visitor: {
      BinaryExpression(path) {
        // ...
      }
    }
  };
}
```

然后我们更确切一些，只关注哪些使用了 `===` 的 `BinaryExpression`。

```javascript
visitor: {
  BinaryExpression(path) {
    if (path.node.operator !== "===")return;

    // ...
  }
}
```

现在我们用新的标识符来替换 `left` 属性：

```javascript
BinaryExpression(path) {
  if (path.node.operator !== "===")return;
  path.node.left = t.identifier("sebmck");
  // ...
}
```

于是如果我们运行这个插件我们会得到：

```javascript
sebmck === bar;
```

现在只需要替换 `right` 属性了

```javascript
BinaryExpression(path) {
  if (path.node.operator !== "===")return;
  path.node.left = t.identifier("sebmck");
  path.node.right = t.identifier("dork");
}
```

这就是我们的最终结果了：

```javascript
sebmck === dork;
```



**使用过程**

index.js

```javascript
foo === bar;
```



babel-plugin-translate.js

这里改成了使用CommonJS

```javascript
module.exports = function ({
    types: t
}) {
    return {
        visitor: {
            BinaryExpression(path) {
                if (path.node.operator !== "===") return;
                path.node.left = t.identifier("sebmck");
                path.node.right = t.identifier("dork");
            }
        }
    };
}
```



babel.config.js

```javascript
module.exports = { plugins: ['./babel-plugin-translate.js'] };
```



执行

```shell
$ npx babel index.js --out-file script-index.js
```



输出文件script-index.js

```javascript
sebmck === dork;
```








## 参考资料

[插件手册](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#toc-visitors)