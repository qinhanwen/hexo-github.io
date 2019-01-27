---
title: github.io+hexo
date: 2018-10-04 15:22:49
tags: 
- hexo
categories: 
- hexo
---

# github.io+hexo

1. 创建一个库，这里的名字叫 **xxx.github.io**，xxx 就是你的账号名字，同时可以选择 “Initialize this … README”。

![image-20181003183048661](http://www.qinhanwen.xyz/images/WX20181204-175001@2x.png)

2. 可以通过访问**xxx.github.io**，访问你的页面了。

3. 我们只需要把这个项目克隆到本地，在上传本地的代码就可以了。可以添加个页面试试

```html
<!DOCTYPE html>
<html>
<body>
  <h1>Hello World</h1>
</body>
</html>
```

上传之后访问 **xxx.github.io**

![image-20181003190309249](http://www.qinhanwen.xyz/images/image-20181003190309249.png)

顺带提一下配置 SSH Key，配置 SSH Key 的目的主要配置是为了让本地和远端可以连接。

1. 首先查看是否存在ssh key

```shell
$ ls -al ~/.ssh
```

如果 key 存在，会有 id_rsa 和 id_rsa.pub 可以直接将 key 复制到远程 GitHub 上。如果不存在就生成 key。

2. 创建 ssh

```shell
$ ssh-keygen -t rsa -C "your_email@example.com"
```

//密码名称什么的一直按回车即可

3. 在 Github 上设置

```shell
$ cd ~/.ssh # 切换目录到这个路径
$ cat id_rsa.pub # 将这个文件的内容显示到终端上
```

4. 打开 Github，点击头像，点击下面的 settings，进入个人设置选择SSH and GPG keys，创建新的SSH：New SSH key，将 key（id_rsa.pub）复制到远程 GitHub 里，同时可以给你的 key 起一个名字。

若需了解更多的内容，可以阅读阮老师 [SSH原理与运用](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html ) 这篇文章。

# hexo部分

##### 1.安装

```shell
$ sudo npm install -g hexo
```

##### 2.初始化

```shell
$ hexo init 
$ npm install
$ hexo s
```

在[http://localhost:4000](http://localhost:4000/)就能看见本地网页

##### 3.修改 _config.yml文件

```shell
$ vim _config.yml

#修改内容
deploy:
  type: github
  repository: https://github.com/qinhanwen/用户名.github.io.git
  branch: master
```

```shell
//保存之后
$ hexo clean
$ hexo g
$ hexo d
#在执行 hexo d 后,出现 error deployer not found:github 的错误
#解决方法npm install hexo-deployer-git --save
#之后修改成
deploy:
  type: git
  repository: https://github.com/qinhanwen/用户名.github.io.git
  branch: master
#再执行就可以了
```

##### 4.发布一条博客

```shell
$ hexo new "这是一条博客哦"
$ hexo g   #生成网页
$ hexo d  #部署到远端(github)
```

##### 5.更换主题

```shell
$ mkdir themes/next  
$ curl -L https://api.github.com/repos/iissnan/hexo-theme-next/tarball | tar -zxv -C themes/next --strip-components=1
#新建一个文件夹，并且下载主题
#修改_config.yml文件配置如下
```

![image-20181004194051654](http://www.qinhanwen.xyz/images/image-20181004194051654.png)

```shell
$ hexo clean  #清理缓存
$ hexo g    #重新生成博客代码
$ hexo d   #部署到本地，之后再次打开https://xxx.github.io即可
```

[这是更换主题参考](https://github.com/iissnan/hexo-theme-next )



##### 6.添加分类栏与标签栏

1）在写的md文件头部添加tags和categories

```markdown
tags: 
- 前端
- hexo
categories: 
- 前端
```



2）新建分类和标签

```shell
$ hexo new page tags
$ hexo new page categories
```



3）打开`tags/index.md`,设置

```markdown
title: 标签
date: 日期
type: "tags"
```



4）打开`category/index.md`,设置

```markdown
title: 分类
date: 日期
type: "categories"
```





