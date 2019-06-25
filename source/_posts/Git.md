---
title: Git
date: 2018-11-06 21:22:04
tags: 
- 基础知识
categories: 
- 基础知识
---

#### Git常用命令

##### 1.忽略已经在版本控制里的文件

```shell
$ git update-index --assume-unchanged 文件路径
#这样 Git 会忽略文件的修改。
```



```shell
$ git update-index --no-assume-unchanged 文件路径
# 需要提交的时候需要重置改标识，.gitignore无法忽略在添加到版本中的文件
```



##### 2.更改签名

```shell
$ git config user.name [AAA]
$ git config user.email [邮箱地址]
```



##### 3.提交操作

```shell
$ git add 文件名
#将工作区新建/修改的内容添加到暂存区

$git commit -m “commit message” 
#将暂存区的内容提交到本地库

$git push
#上传

$git push --set-upstream origin xxxx
#第一次提交到远程分支
```



##### 4.撤回操作

```shell
$ git reset --hard 哈希索引值
```



##### 5.切换分支

```shell
$ git checkout 分支名

$ git checkout -b [name]
#创建新分支，并且切换到新分支
```



##### 6.检出仓库

```shell
$ git clone xxxxxxxx
```



##### 7.历史

```shell
$ git log
```



##### 8.暂存

```shell
$ git stash
#将当前未提交的工作存入Git工作栈中，时机成熟的时候再应用回来，

$ git stash list
#会显示这个栈的list.

$git stash pop
#将工作栈中，最后一笔弹出

$ git stash clear
#清空栈
```



##### 9.合并

```shell
$ git merge [brance]
#合并进当前分支
```



##### 10.更新

```shell
$ git fetch
#会取到本地没有的数据
```



##### 11.git引起的变化

```shell
$ git reflog
```



##### 12.回滚远程

假设3笔提交

```
commit 3
commit 2
commit 1
```

```shell
$ git reset --hard HEAD~1
$ git push --force
```









