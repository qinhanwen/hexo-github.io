---
title: Git
date: 2018-11-06 21:22:04
tags: 
- 杂
categories: 
- 杂
---

##### 忽略已经在版本控制里的文件

1. 先对其 `git update-index --assume-unchanged 文件路径`，这样 Git 会忽略文件的修改。

2. 需要提交的时候需要重置改标识：`git update-index --no-assume-unchanged 文件路径`，于是 Git 只需要做一次更新。

   (.gitignore无法忽略在添加到版本中的文件)
