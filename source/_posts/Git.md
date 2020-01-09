---
title: Git
date: 2018-11-06 21:22:04
tags: 
- 优化
categories: 
- 优化
---

#### Git常用命令

##### 1.忽略已经在版本控制里的文件

```shell
$ git update-index --assume-unchanged 文件路径
#这样 Git 会忽略文件的修改。

$ git update-index --no-assume-unchanged 文件路径
# 需要提交的时候需要重置改标识，.gitignore无法忽略在添加到版本中的文件
```

某目录下所有文件可以写成 `src/**`

##### 2.更改签名

```shell
$ git config user.name [AAA]
$ git config user.email [邮箱地址]

#设置记录
$ git config --global credential.helper store
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
$ git reset --hard [哈希索引值]
$ git reset --hard HEAD~[数值]
```



##### 5.切换分支

```shell
$ git checkout [分支名]

$ git checkout -b [name]
#创建新分支，并且切换到新分支

$ git checkout -b [本地分支名] origin/[远程分支名]
#拉去远程分支到本地某分支，这会配置关联关系，后面会说
```



##### 6.检出仓库

```shell
$ git clone xxxxxxxx
```



##### 7.历史

```shell
$ git log
$ git log [Branches]

#最近3笔提交改动文件
$ git log -3 --stat

#具体某一笔的文件变动
$ git show [哈希索引值] --stat
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


##### 13.cherry-pick
```shell
$ git cherry-pick [哈希索引值]
```



##### 14.撤销某笔操作

```shell
$ git revert [哈希索引值]
```



##### 15.rebase

```shell
$ git rebase -i [startpoint]  [endpoint]
```

`-i`的意思是`--interactive`，即弹出交互式的界面让用户编辑完成合并操作，

![WX20190719-003950@2x](http://118.24.241.76/WX20190719-003950@2x.png)

合并图中的两笔

```shell
$ git rebase -i 5f81895032add1c5f003519b059e67d971d5bdd8 f4544e9b890b988f51fcff211d2399d203ef57fc
```

![WX20190719-004054@2x](http://118.24.241.76/WX20190719-004054@2x.png)

```
pick：保留该commit（缩写:p）

reword：保留该commit，但我需要修改该commit的注释（缩写:r）

edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）

squash：将该commit和前一个commit合并（缩写:s）

fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）

exec：执行shell命令（缩写:x）

drop：我要丢弃该commit（缩写:d）
```

修改内容，第二个pick改成s或者squash

![WX20190719-004236@2x](http://118.24.241.76/WX20190719-004236@2x.png)

wq保存

![WX20190719-004331@2x](http://118.24.241.76/WX20190719-004331@2x.png)



合并最后几笔记录log：

```shell
$ git rebase -i HEAD~2
#合并最后两笔
#跟上面一样最后一个pick改成s或者squash，wq修改保存退出
#提示确认消息，q保存退出
$ git push -f
#与远程同步
```







需要将多笔提交，复制到其他分支的时候，如图

![WX20190719-005203@2x](http://118.24.241.76/WX20190719-005203@2x.png)

```shell
$ git rebase  [startpoint]   [endpoint]  --onto  [branchName]
```



```shell
#放弃合并，回到rebase操作之前的状态，之前的提交的不会丢弃
$ git rebase --abort

#合并冲突，结合"git add 文件"命令一起用与修复冲突，提示开发者，一步一步地有没有解决冲突
$ git rebase --continue 
```



##### 16. pull

```shell
$ git pull
#等价于	
# git fetch + git merge FETCH_HEAD 

$ git pull --rebase
#等价于	
# git fetch + git rebase FETCH_HEAD 

$ git pull origin develop:develop
#远端develop更新到本地的develop
```

**FETCH_HEAD：** 是一个版本链接，记录在本地的一个文件中，指向着目前已经从远程仓库取下来的分支的末端版本。



比如有两个分支

```
      D---E develop
      /
 A---B---C---F--- master
```



执行

```shell
$ git merge develop
```

```
       D--------E
      /          \
 A---B---C---F----G---   develop, master
```

merge会产生新节点



执行

```shell
$ git rebase test
```

```
A---B---D---E---C‘---F‘---   develop, master
```

rebase会将两个分支合并。rebase 操作的话，会中断rebase,同时会提示去解决冲突。
解决冲突后,将修改add后执行`git rebase --continue`继续操作，或者`git rebase --skip`忽略冲突。



```shell
$ git rebase --continue
#处理冲突后继续操作

$ git rebase --abort 
#会回到rebase操作之前的状态，之前的提交的不会丢弃；
```





###### **碰到的问题**

![WX20190719-233059@2x](http://118.24.241.76/WX20190719-233059@2x.png)



在git push的时候发现当前分支没在远程上，解决方案：切换到需要提交的分支，cherry-pick过来。



###### 可能碰到的问题

本地和远端分支没有关联，导致下拉代码有问题(比如因为没建立关联，develop分支下拉的是别的分支的代码，不是远端的develop)

```shell
$ git branch -vv
#查看本地分支与远程分支关联

#解决方案：
#1.下拉时候指定下拉分支
$ git pull origin develop:develop

#2.检出时候设置关联关系(不过我使用这个不知道为什么失效了)
$ git checkout -b [本地分支名] origin/[远程分支名]

#3.提交时配置关联关系
$ git push -u origin [remote_branch]
```

![6F00020A81A132B95C9D3FE24E431754](http://118.24.241.76/6F00020A81A132B95C9D3FE24E431754.png)



##### 17.查看分支

```shell
$ git branch

#所有分支
$ git branch -a
```



##### 18.删除分支

```shell
$ git branch -D  要删除的分支
```





#### 备注：

##### .gitignore文件

.gitignore只能忽略那些原来没有被追踪的文件，如果需要新增忽略文件的话

```shell
$ git rm -r --cached .
$ git add .
$ git commit -m 'update .gitignore'
```



**误删本地分支恢复**

```shell
#删除分支
$ git branch -D develop 

#找到提交log
$ git log -g

#找到最后一笔commit的记录，或者删除分支的索引值
$ git branch [分支名]  [哈希索引值]
```

删除分支的本质，其实是删除了.git/refs/heads/目录下的一个与分支同名文件，文件内容是一个分支所指向commit对象的sha-1值。也就是说真正删除的并不是commit对象本身，更为形象的说就是揭掉贴在commit对象上的标签，当然我们可以再贴上一个标签。