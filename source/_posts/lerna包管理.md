---
title: lerna包管理
date: 2019-04-07 11:41:23
tags: 
- lerna
categories: 
- lerna
---

## 简介

[lerna](https://github.com/lerna/lerna)是一个用于管理包含多个package结构的代码仓库的工具。



## 安装

```shell
$ npm install lerna -g
```



## 初始化项目

```shell
$ lerna init 
```

设置lerna.json文件

```json
{
  "version": "1.0.0",
  "npmClient": "yarn",
  "command": {
    "publish": {
      "ignoreChanges": ["ignored-file", "*.md"]
    }
  },
  "packages": [
    "packages/*"
  ]
}
```

字段说明：

| key                             | value                                                        |
| :------------------------------ | :----------------------------------------------------------- |
| `version`                       | 当前仓库版本，当设为`independent`时开启独立模式              |
| `npmClient`                     | 执行命令的client，默认为`npm`，可以设为`yarn`                |
| `command.publish.ignoreChanges` | 设置不会包含进`lerna change/publish`操作的文件路径，使用它来避免一些非重要改动时的版本更新，比如更新`README.md`中的拼写错误 |
| `packages`                      | 用于定位package的文件路径                                    |



## 使用

创建包，会在packages下面创建这个依赖包

```shell
$ lerna create xxx
```

添加依赖

```shell
$ lerna add [@version] [--dev] [--exact]
```

`--dev` 和 `--exact` 等同于 `npm install` 里的 `--dev` 和 `—exact`

当我们执行此命令后，将会执行下面那 2 个动作：

1. 在每一个符合要求的模块里安装指明的依赖包，类似于在指定模块文件夹中执行 `npm install <package>`。
2. 更新每个安装了该依赖包的模块中的 `package.json` 中的依赖包信息。