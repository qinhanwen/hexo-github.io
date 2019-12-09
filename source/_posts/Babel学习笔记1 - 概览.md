---
title: Babel学习笔记1
date: 2019-11-30 21:05:05
tags: 
- Babel
categories: 
- Babel
---

## Babel是什么

Babel 是一个 JavaScript 编译器，是一个工具链。主要用于将 ECMAScript 2015+ 版本的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。它能做的事情：

- 语法转换
- 通过 Polyfill 方式在目标环境中添加缺失的特性 (通过 [@babel/polyfill](https://www.babeljs.cn/docs/babel-polyfill) 模块)
- 源码转换 (codemods)



## 插件

Babel 是一个编译器（输入源码 => 输出编译后的代码）。就像其他编译器一样，编译过程分为三个阶段：解析、转换和打印输出。

##### 转换插件：用于转换代码



**语法插件：只允许Babel解析特定类型的语法（不是转换）**

转换插件会自动启用语法插件。因此，如果你已经使用了相应的转换插件，则不需要指定语法插件。



**插件/Preset路径：如果插件在 npm 上，你可以输入插件的名称，babel 会自动检查它是否已经被安装到 `node_modules` 目录下**

```json
{
  "plugins": ["babel-plugin-myPlugin"]
}

也可以指定插件的相对/绝对路径
{
  "plugins": ["./node_modules/asdf/plugin"]
}

如果插件名称的前缀为 babel-plugin-，还可以使用它的短名称
{
  "plugins": [
    "myPlugin",
    "babel-plugin-myPlugin" // 两个插件实际是同一个
  ]
}
```



**插件顺序：转换插件都将处理“程序（Program）”的某个代码片段，则将根据转换插件或 preset 的排列顺序依次执行**

```json
{
  "plugins": ["transform-decorators-legacy", "transform-class-properties"]
}
```

先执行 `transform-decorators-legacy` ，在执行 `transform-class-properties`。



**插件参数：插件和 preset 都可以接受参数，参数由插件名和参数对象组成一个数组，可以在配置文件中设置。**

```json
{
  "plugins": [
    [
      "transform-async-to-module-method",
      {
        "module": "bluebird",
        "method": "coroutine"
      }
    ]
  ]
}
```

要指定参数，请传递一个以参数名作为键（key）的对象。

preset 的设置参数的工作原理完全相同：

```json
{
  "presets": [
    [
      "env",
      {
        "loose": true,
        "modules": false
      }
    ]
  ]
}
```



## 预设（Presets）

preset 可以作为 Babel 插件的组合

**Preset的路径：如果 preset 在 npm 上，你可以输入 preset 的名称，babel 将检查是否已经将其安装到 `node_modules` 目录下了**

```json
{
  "presets": ["babel-preset-myPreset"]
}

指定绝对/相对路径
{
  "presets": ["./myProject/myPreset"]
}
```



**Preset的短名称：如果 preset 名称的前缀为 `babel-preset-`，你还可以使用它的短名称**

```json
{
  "presets": [
    "myPreset",
    "babel-preset-myPreset" // equivalent
  ]
}
```



**Preset的排列顺序：Preset是逆序排列的（从后往前）**

```json
{
  "presets": [
    "a",
    "b",
    "c"
  ]
}
```

将按如下顺序执行： `c`、`b` 然后是 `a`



**Preset参数：插件和 preset 都可以接受参数，参数由插件名和参数对象组成一个数组，可以在配置文件中设置。**

```json
{
  "presets": [
    ["@babel/preset-env", {
      "loose": true,
      "modules": false
    }]
  ]
}
```



## 配置Babel

**.babelrc**

使用场景

- 只是需要一个简单的并且只用于单个软件包的配置

在你的项目中创建名为 `.babelrc` 的文件，并输入以下内容。

```json
{
  "presets": [...],
  "plugins": [...]
}
```



**babel.config.js**	

使用场景

- 希望以编程的方式创建配置文件
- 希望编译 `node_modules` 目录下的模块

在项目的根目录（`package.json` 文件所在目录）下创建一个名为 `babel.config.js` 的文件，并输入如下内容。

```javascript
module.exports = function (api) {
  api.cache(true);

  const presets = [ ... ];
  const plugins = [ ... ];

  return {
    presets,
    plugins
  };
}
```



**.babelrc.js**

与 [`.babelrc`](https://www.babeljs.cn/docs/configuration#babelrc) 的配置相同，但你可以使用 JavaScript 编写。

```javascript
const presets = [ ... ];
const plugins = [ ... ];

module.exports = { presets, plugins };
```



**package.json**

还可以选择将 [`.babelrc`](https://www.babeljs.cn/docs/configuration#babelrc) 中的配置信息作为 `babel` 键（key）的值添加到 `package.json` 文件中，如下所示：

```json
{
  "name": "my-package",
  "version": "1.0.0",
  "babel": {
    "presets": [ ... ],
    "plugins": [ ... ],
  }
}
```



## 命令行工具

**[@babel/cli](https://www.babeljs.cn/docs/babel-cli)：**Babel 自带了一个内置的 CLI 命令行工具，可通过命令行编译文件。

```shell
$ npm install @babel/core @babel/cli -D
```

部分操作说明

```shell
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



## 工具

**[@babel/parser](https://www.babeljs.cn/docs/babel-parser)：在Babel中使用的JavaScript解析器，生成ast抽象语法树**

- 默认支持ES2017
- 支持JSX，Flow，TypeScript
- 支持实验性的语言特性

```javascript
const {parse} = require('@babel/parser');
const code = 'const a = 1';
const ast = parse(code);
```



**[@babel/core](https://www.babeljs.cn/docs/babel-core)：核心模块**

- transform，转换传入的代码，回调两个参数一个是error一个是result

```javascript
babel.transform(code, options, function(err, result) {
  result; // => { code, map, ast }
});
```

- transformSync，返回一个Promise

```javascript
babel.transformAsync(code, options) // => Promise<{ code, map, ast }>
```



**[@babel/generator](https://www.babeljs.cn/docs/babel-generator)：ast生成代码**

```javascript
const {parse} = require('@babel/parser');

const generate = require('@babel/generator').default;

const code = 'const a = 1';
const ast = parse(code);

const output = generate(ast, { /* options */ }, code);
console.log(output)
```



**[@babel/runtime](https://www.babeljs.cn/docs/babel-runtime)：包含了babel模块运行时规则**



**[@babel/traverse](https://www.babeljs.cn/docs/babel-traverse)：可以和babel解析器一起使用来遍历和更新节点**

```javascript
const ast = parse(code);
traverse(ast, {
  enter(path) {
    if (path.isIdentifier({ name: "n" })) {
      path.node.name = "x";
    }
  }
});
```

