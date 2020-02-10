---
title: '浅析babel-plugin-component'
date: 2020-02-10 13:19:10
tags:
  - Babel
categories:
  - Babel
---

## 浅析babel-plugin-component

vue + element-ui 使用 babel-plugin-component 来实现按需加载组件及样式。



### 一、[官网按需引入示例](https://element.eleme.io/2.0/#/zh-CN/component/quickstart)

借助 [babel-plugin-component](https://github.com/QingWei-Li/babel-plugin-component)，我们可以只引入需要的组件，以达到减小项目体积的目的。

首先，安装 babel-plugin-component：

```bash
npm install babel-plugin-component -D
```

然后，将 .babelrc 修改为：

```json
{
  	...
  
    "plugins": [
        [
            "component",
            {
                "libraryName": "element-ui",
                "styleLibraryName": "theme-chalk"
            }
        ]
    ]
}
```

接下来，如果你只希望引入部分组件，比如 Button 和 Select，那么需要在 main.js 中写入以下内容：

```javascript
import Vue from 'vue'
import { Button, Select } from 'element-ui'
import App from './App.vue'

Vue.component(Button.name, Button)
Vue.component(Select.name, Select)
/* 或写为
 * Vue.use(Button)
 * Vue.use(Select)
 */

new Vue({
  el: '#app',
  render: h => h(App)
})
```



### 二、[构建过程概述]()

使用 @babel/cli 与 @babel/core 编译文件的主要过程

- 加载配置文件
  - 尝试加载 babel.config.js、babel.config.cjs、babel.config.mjs、babel.config.json文件
  - 加载 package.json 文件
  - 尝试加载 .bebelrc、.babelrc.js、.babelrc.cjs、.babelrc.mjs、.babelrc.json 文件
  - 合并参数

- 加载 plugins 与 presets，分别遍历他们（plugins 每一项会调用它的回调，返回包含 visitor 的对象）
- 解析部分
  - 以 utf8 格式读入口文件得到代码
  - 之后解析生成 ast
- 遍历与转换部分
  - 遍历插件数组，生成最后的访问者（visitor）对象
  - 开始遍历节点，碰到感兴趣的节点就调用回调
- 生成部分
  - 遍历 ast，将得到的代码保存在数组中，最后拼接



### 三、babel-plugin-component

通过一个[简单的项目](https://github.com/qinhanwen/babel-plugin-component-demo.git)可以看一下上面的配置在编译过程中做了哪些事情

**1.项目结构**

目录结构

```
├── index.js
├── .babelrc
├── .gitignore
├── package-lock.json
└── package.json
```

package.json

```json
{
  
  ...
  
  "scripts": {
    "build": "rm -rf lib && babel ./index.js --out-dir lib"
  },
  
  ...
  
  "dependencies": {
    "@babel/cli": "^7.8.4",
    "@babel/core": "^7.8.4",
    "babel-plugin-component": "^1.1.1"
  }
}
```

.babelrc 文件

```json
{
  "plugins": [
      [
          "component",
          {
              "libraryName": "element-ui",
              "styleLibraryName": "theme-chalk"
          }
      ]
  ]
}
```

index.js 文件

```javascript
import Vue from 'vue'
import { Button, Select } from 'element-ui'
import App from './App.vue'

Vue.component(Button.name, Button)
Vue.component(Select.name, Select)
new Vue({
  el: '#app',
  render: h => h(App)
})
```

编译生成文件

```javascript
import _Select2 from "element-ui/lib/theme-chalk/select.css";
import "element-ui/lib/theme-chalk/base.css";
import _Select from "element-ui/lib/select";
import _Button2 from "element-ui/lib/theme-chalk/button.css";
import "element-ui/lib/theme-chalk/base.css";
import _Button from "element-ui/lib/button";
import Vue from 'vue';
import App from './App.vue';
Vue.component(_Button.name, _Button);
Vue.component(_Select.name, _Select);
new Vue({
  el: '#app',
  render: h => h(App)
});
```



