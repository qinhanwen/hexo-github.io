---
title: npm
date: 2019-06-29 16:37:06
tags: 
- npm
categories: 
- npm
---

> [前端工程化（5）：你所需要的npm知识储备都在这了](https://juejin.im/post/5d08d3d3f265da1b7e103a4d)
>
> https://juejin.im/post/5ab3f77df265da2392364341
>
> [semver规范](https://www.jianshu.com/p/a7490344044f)
>
> [package-lock.json](https://juejin.im/entry/5ae00835518825673277e460)



## npm 是什么

npm是随同NodeJS一起安装的包管理工具。

常见使用场景：

- 允许用户从NPM服务器下载别人编写的第三方包到本地使用。
- 允许用户从NPM服务器下载并安装别人编写的命令行程序到本地使用。
- 允许用户将自己编写的包或命令行程序上传到NPM服务器供别人使用。



## 发布npm包



一、安装npm（新版的NodeJS已经集成了npm，所以安装的时候npm也一并安装好了）

二、[注册npm账号](https://www.npmjs.com/)，注册完会收到邮件，需要点击验证一下，不然不能上传包

三、新建一个文件夹`npm-study-qhw`，打开命令行，输入

```shell
$ npm init --yes
```

npm init 是执行默认行为，输出一个初始化的 package.json 文件， 添加--yes参数（或-y），则生成一个默认的配置文件

![WX20190701-093012@2x](http://114.55.30.96/WX20190701-093012@2x.png)

要补全一下`author`

四、新建index.js文件

![WX20190701-093100@2x](http://114.55.30.96/WX20190701-093100@2x.png)

五、添加用户

```shell
$ npm adduser
```

六、发布

```shell
$ npm publish 
```

遇到报错如下：

![WX20190701-093931@2x](http://114.55.30.96/WX20190701-093931@2x.png)

因为使用的是淘宝源，需要切换到npmjs的网址

```shell
$ npm config set registry http://registry.npmjs.org/
```

之后再发布一次

![WX20190701-095008@2x](http://114.55.30.96/WX20190701-095008@2x.png)



七、查看与安装

![WX20190701-095236@2x](http://114.55.30.96/WX20190701-095236@2x.png)

新建另外一个文件夹

```shell
#新增package.json文件
$ npm init -y
#安装依赖包
$ npm install npm-study-qhw

#新建index.js文件

#运行index.js文件
$ node index
```

![569D362A18DC9925A04BADB20F480C37](http://114.55.30.96/569D362A18DC9925A04BADB20F480C37.png)

`npm install packageName`的时候，会自动添加到`dependencies`下面（与npm版本有关）。

![QQ20190701-0](http://114.55.30.96/QQ20190701-0.png)

八、更新版本

```shell
$ npm version <update_type>
#update_type为patch, minor, or major其中之一，分别表示补丁，小版本，大版本
$ npm publish
```

九、删除

强制删除，不要占用npm包名

```shell
$ npm unpublish -f
```



## 常见命令

```shell
#查看npm版本
$ npm -v

#创建package.json文件
$ npm init [--yes/-y]

#安装依赖包
$ npm install packageName [-S/--save/-D/--save-dev/--save-exact/-E/--save-optional/-O/-g]

#查看帮助命令
$ npm -h

#更新依赖包
$ npm update packageName

#卸载依赖包
$ npm uninstall packageName

#设置依赖包源
$ npm config set registry https://registry.npm.taobao.org

#查看安装模块
$ npm ls

#运行script命令
$ npm run <command> [--<args>]

#创建软链
$ npm link <packageName>
```

**npm link使用**

项目A和项目B，B依赖A

在本地调试的时候，在项目A目录下，创建软链

```shell
$ npm link
```

在项目B目录下

```shell
$ npm link A
```

这里的A和B分别对应项目`package.json`文件里的`name`字段（取名当然不能为A和B）




## package.json文件

#### **name**：包名，全部小写，没有空格，可以使用下划线或者横线

#### **version**：版本号

npm采用了[semver](https://www.jianshu.com/p/a7490344044f)规范作为依赖版本管理方案。

按照semver的约定，一个npm依赖包的版本格式一般为：**主版本号.次版本号.修订号**（`x.y.z`），每个号的含义是：

**主版本号**（也叫大版本，`major version`）

大版本的改动很可能是一次颠覆性的改动，也就意味着可能存在与低版本不兼容的`API`或者用法，（比如 `vue 2 -> 3`)。

**次版本号**（也叫小版本，`minor version`）

小版本的改动应当兼容同一个大版本内的`API`和用法，因此应该让开发者无感。所以我们通常只说大版本号，很少会精确到小版本号。

**修订号**（也叫补丁，`patch`）

一般用于修复`bug`或者很细微的变更，也需要保持向前兼容。

###### 常见版本号格式：

**"1.2.3"**：精确版本号。

**"^1.2.3"**：兼容小版本和补丁更新的版本号，`大版本号为 0 版本号的时候处理机制不同`

```
"^1.2.3" 等价于 ">= 1.2.3 < 2.0.0"。即只要最左侧的 "1" 不变，其他都可以改变。所以 "1.2.4", "1.3.0" 都可以兼容。

"^0.2.3" 等价于 ">= 0.2.3 < 0.3.0"。因为最左侧的是 "0"，那么只要第二位 "2" 不变，其他的都兼容，比如 "0.2.4" 和 "0.2.99"。

"^0.0.3" 等价于 ">= 0.0.3 < 0.0.4"。大版本号和小版本号都为 "0" ，所以也就等价于精确的 "0.0.3"
```

**"~1.2.3"**：表示只兼容补丁更新的版本号。关于 `~` 的定义分为两部分：如果列出了小版本号（第二位），则只兼容补丁（第三位）的修改；如果没有列出小版本号，则兼容第二和第三位的修改。

```
"~1.2.3" 列出了小版本号 "2"，因此只兼容第三位的修改，等价于 ">= 1.2.3 < 1.3.0"。

"~1.2" 也列出了小版本号 "2"，因此和上面一样兼容第三位的修改，等价于 ">= 1.2.0 < 1.3.0"。

"~1" 没有列出小版本号，可以兼容第二第三位的修改，因此等价于 ">= 1.0.0 < 2.0.0"
```

**"1.x" 、"1.X"、1.\*"、"1"、"\*"**：表示使用通配符的版本号。x、X、* 和 （空） 的含义相同，都表示可以匹配任何内容

```
"*" 、"x" 或者 （空） 表示可以匹配任何版本。

"1.x", "1.*" 和 "1" 表示匹配主版本号为 "1" 的所有版本，因此等价于 ">= 1.0.0 < 2.0.0"。

"1.2.x", "1.2.*" 和 "1.2" 表示匹配版本号以 "1.2" 开头的所有版本，因此等价于 ">= 1.2.0 < 1.3.0"。
```



#### description：描述信息，有助于搜索

#### main: 入口文件，一般都是 `index.js`

#### keywords：关键字，有助于在人们使用 `npm search` 搜索时发现你的项目

#### author：作者信息

#### scripts：用于定义脚本命令


#### dependencies：业务依赖

这种依赖在项目最终上线或者发布npm包时所需要， 通过`npm install/i packageName -S/—save`，把包装在依赖项里，如果没有没有指定版本，则安装npm仓库中最新版本的包，可以把版本号写在包名最后来指定版本`npm i vue@3.0.1 -S`。

从`npm 5.x`开始，可以不用手动添加`-S/--save`指令，直接执行`npm i packageName`把依赖包添加到`dependencies`中去。



#### devDependencies：开发依赖

这种依赖只在项目开发时所需要，即其中的依赖项不应该属于线上代码的一部分。比如构建工具`webpack`、`gulp`，预处理器`babel-loader`、`scss-loader`，测试工具`e2e`、`chai`等，这些都是辅助开发的工具包，无须在生产环境使用。通过`npm install/i packageName -D/--save-dev`安装。

并不是只有在dependencies中的模块才会被一起打包，而在devDependencies中的不会！**模块能否被打包，取决于项目里是否被引入了该模块！** 在业务项目中`dependencies`和`devDependencies`没有什么本质区别，只是单纯的一个规范作用，在执行`npm i`时两个依赖下的模块都会被下载；而在发布`npm`包的时候，包中的`dependencies`依赖项在安装该包的时候会被一起下载，`devDependencies`依赖项则不会。



#### peerDependencies：同伴依赖

这种依赖的作用是提示宿主环境去安装插件在`peerDependencies`中所指定依赖的包，然后插件所依赖的包永远都是宿主环境统一安装的`npm`包，最终解决插件与所依赖包不一致的问题。

比如：`element-ui@2.6.3`只是提供一套基于`vue`的`ui`组件库，但它要求宿主环境需要安装指定的`vue`版本，所以你可以看到`element`项目中的`package.json`中具有一项配置：

```json
"peerDependencies": {
    "vue": "^2.5.16"
}
```

反正就是安装了一个包，要按照它的需要，去安装别的包。



#### bundledDependencies / bundleDependencies：打包依赖

#### optionalDependencies：可选依赖



## npm对node_modules的处理

依赖管理是 npm 的核心功能，原理就是执行 `npm install` 从 package.json 中的 dependencies, devDependencies 将依赖包安装到当前目录的 ./node_modules 文件夹中。

看下不同版本npm的表现

#### 一、npm版本2.15.9

![WX20190701-221036@2x](http://114.55.30.96/WX20190701-221036@2x.png)

先发布4个npm包

```
npm-study2@1.0.0 依赖于 npm-study1@1.0.0

npm-study3@1.0.0 依赖于 npm-study1@2.0.0

npm-study4@1.0.0 依赖于 npm-study1@1.0.0
```

新建项目，package.json文件如下：

```json
{
  "name": "ls",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "npm-study2": "^1.0.0",
    "npm-study3": "^1.0.0",
    "npm-study4": "^1.0.0"
  }
}
```

安装依赖

```shell
$ npm install
```

node_modules结构：

```
├── node_modules
│   ├── npm-study2@1.0.0
│   │   └── node_modules
│   │   │   └── npm-study1@1.0.0
│   ├── npm-study3@1.0.0
│   │   └── node_modules
│   │   │   └── npm-study1@2.0.0
│   └── npm-study4@1.0.0
│   │   └── node_modules
│   │   │   └── npm-study1@1.0.0
```

`npm 2.x`安装依赖方式比较简单直接，以递归的方式按照包依赖的树形结构下载填充本地目录结构，也就是说每个包都会将该包的依赖安装到当前包所在的`node_modules`目录中。



#### 二、npm版本3.10.10

![WX20190701-232152@2x](http://114.55.30.96/WX20190701-232152@2x.png)

删除node_modules，并重新安装依赖

```shell
$ npm install
```

node_modules结构：

```
├── node_modules
│   ├── npm-study1@1.0.0
│   ├── npm-study2@1.0.0
│   ├── npm-study3@1.0.0
│   │   └── node_modules
│   │   │   └── npm-study1@2.0.0
│   └── npm-study4@1.0.0
```

`npm 3.x`则采用了扁平化的结构来安装组织`node_modules`。也就是在执行`npm install`的时候，按照`package.json` 里依赖的顺序依次解析，遇到新的包就把它安装在第一级目录，后续安装如果遇到一级目录已经存在的包，会先按照约定版本判断版本，如果符合版本约定则忽略，否则会按照npm 2.x的方式依次挂在依赖包目录下。





## package-lock.json文件

npm更新到v5.x.x以后, 会出现一种新的自动生成文件 `package-lock.json`。

npm为了让开发者在安全的前提下使用最新的依赖包，在`package.json`中通常做了锁定大版本的操作，这样在每次`npm install`的时候都会拉取依赖包大版本下的最新的版本。这种机制最大的一个缺点就是当有**依赖包有小版本更新时**，可能会出现协同开发者的**依赖包版本不一致**的问题。

`package-lock.json`文件精确描述了`node_modules` 目录下所有的包的树状依赖结构，每个包的版本号都是完全精确的。它的的详细描述主要由`version`、`resolved`、`integrity`、`dev`、`requires`、`dependencies`这几个字段构成：

- `version`：包唯一的版本号
- `resolved`：安装源
- `integrity`：表明包完整性的hash值（验证包是否已失效）
- `dev`：如果为true，则此依赖关系仅是顶级模块的开发依赖关系或者是一个的传递依赖关系
- `requires`：依赖包所需要的所有依赖项，对应依赖包`package.json`里`dependencies`中的依赖项
- `dependencies`：依赖包`node_modules`中依赖的包，与顶层的`dependencies`一样的结构



切换到npm版本`6.4.1`，在大版本相同的前提下，以`^`版本为例，`npm-study3`依赖包版本有`1.3.0`，`1.4.0`，`1.6.0`还有`1.9.0`，`node_modules`下没有该依赖包的情况，那么测试一下`package-lock.json`的表现：

1）如果一个模块在`package.json`中的小版本要**小于**`package-lock.json`中的小版本，则被`package-lock.json`中的版本锁定

![WX20190703-091915@2x](http://114.55.30.96/WX20190703-091915@2x.png)

安装的`npm-study3`依赖包的实际版本为`1.4.0`



2）如果一个模块在`package.json`中的小版本要**大于**`package-lock.json`中的小版本

![WX20190703-092353@2x](http://114.55.30.96/WX20190703-092353@2x.png)

则在执行`npm install`时，会将该模块更新到大版本下的最新的版本，并将版本号更新至`package-lock.json`。

![WX20190703-093419@2x](http://114.55.30.96/WX20190703-093419@2x.png)

安装的`npm-study3`依赖包的实际版本为`1.9.0`



更新某个模块大版本下的最新版本（升级小版本号），可以执行以下命令

```shell
$ npm install packageName
## 或者
$ npm update packageName

#指定版本号
$ npm install packageName@x.x.x
```

`package.json`和`package-lock.json`中的版本号都将会随之更新。



禁用`package-lock.json`

```shell
$ npm config set package-lock false
```



## .npmrc文件

通过修改 `.npmrc `文件可以直接修改配置。系统中存在多个`npmrc`文件，这些`npmrc`文件被访问的优先级从高到低的顺序为：

1.项目级的`.npmrc`文件

2.用户级的`.npmrc`文件（`npm config get userconfig`可以看到存放的路径）

3.全局级的`.npmrc`文件（`npm config get globalconfig`可以看到存放的路径）

4.`npm`内置的`.npmrc`文件















