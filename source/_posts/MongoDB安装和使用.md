---
title: MongoDB安装和使用Mac OSX
date: 2019-04-08 20:22:16
tags: 
- 拓展
categories: 
- 拓展
---
> 参考   	https://jspang.com/post/mongodb.html
>

[TOC]



## 下载与安装

[下载地址](https://www.mongodb.com/download-center#community)

```shell
# 下载之后，将解压后的文件夹重命名为 MongoDB 放入 /usr/local ,可以打开Finder后按 shift + command + G 输入 /usr/local 回车

#配置环境变量，打开终端，输入
$ open -e .bash_profile

#在打开的文件中加入
$ export PATH=${PATH}:/usr/local/MongoDB/bin

# command + s保存配置

#在终端输入，使配置生效
$ source .bash_profile

#输入
$ mongod -version

#生成 data 文件夹，里面再建一个 db 文件夹
$ sudo mkdir -p /data/db

```





## 运行

```shell
#命令行输入
$ mongod
```

这时候我报错了。

![WechatIMG321](http://www.qinhanwen.xyz/WechatIMG321.png)

解决方法：

```shell
$ sudo chown -R $USER /data/db
```



```shell
#继续输入
$ mongod
#或者
$ /usr/local/mongodb/bin/mongod
```



打开http://localhost:27017/，看到

![WX20190408-234819@2x](http://www.qinhanwen.xyz/WX20190408-234819@2x.png)





## 创建数据库

##### 1.添加超级管理员账号

```shell
# 进入admin库
$ use admin

#添加用户
$ db.createUser(
     {
       user:"qhw",
       pwd:"123456"
       roles:[{role:"root",db:"admin"}]
     }
  )
  
#查看是否添加成功
$ show users
```

如图

![WX20190412-135301@2x](http://www.qinhanwen.xyz/WX20190412-135301@2x.png)



##### 2.登录

```shell
#查询数据库
$ show dbs

#切换到某个库
$ use admin

#登录，返回1就是登录成功
$ db.auth('qhw','123456')
```



##### 3.创建数据库，以及为单个数据库添加管理员

```shell
#新增
$ use interview
#switched to db iterview

#
$ db.createUser({
      user:'test',
      pwd:'123456',
      roles:[{role:'readWrite',db:'iterview'}]
    })
```

![WX20190412-141012@2x](http://www.qinhanwen.xyz/WX20190412-141012@2x.png)







## 连接数据库

##### 1.连接语法

```
$ mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```

- **mongodb://** 这是固定的格式，必须要指定。
- **username:password@** 可选项，如果设置，在连接数据库服务器之后，驱动都会尝试登陆这个数据库
- **host1** 必须的指定至少一个host, host1 是这个URI唯一要填写的。它指定了要连接服务器的地址。如果要连接复制集，请指定多个主机地址。
- **portX** 可选的指定端口，如果不填，默认为27017
- **/database** 如果指定username:password@，连接并验证登陆指定数据库。若不指定，默认打开 test 数据库。
- **?options** 是连接选项。如果不使用/database，则前面需要加上/。所有连接选项都是键值对name=value，键值对之间通过&或;（分号）隔开



##### 2.连接

用户名字：test

密码：123456

连接服务器的地址：localhost

数据库：iterview

```shell
$ mongodb://test:123456@localhost/iterview
```





## update修改器

了解修改器之前，先插入一笔数据

```shell
$ var workmate = {name:'qinhanwen',age:23};
$ db.workmate.insert(workmate);
$ db.workmate.find();
```

![WX20190415-085524@2x](http://www.qinhanwen.xyz/WX20190415-085524@2x.png)

添加成功了。



##### $set

用来修改一个指定的键值(key)

```shell
$ db.workmate.update({"name":"qinhanwen"},{"$set":{age:25}})
```

![WX20190415-085625@2x](http://www.qinhanwen.xyz/WX20190415-085625@2x.png)



修改嵌套内容

```shell
#删除表中的数据
$ db.workmate.drop();

$ var workmate = {name:'qinhanwen',age:23,skill:{name:'javascript'}};
$ db.workmate.insert(workmate);
$ db.workmate.find();
$ db.workmate.update({"name":"qinhanwen"},{"$set":{"skill.name":"css"}});
$ db.workmate.find();
```

![WX20190415-090452@2x](http://www.qinhanwen.xyz/WX20190415-090452@2x.png)



##### **$unset**

将制定key删除

```shell
$ db.workmate.update({"name":"qinhanwen"},{"$unset":{"age":""}});
$ db.workmate.find();
```

![WX20190415-090709@2x](http://www.qinhanwen.xyz/WX20190415-090709@2x.png)



##### **$inc**

对数字进行运算

```shell
$ db.workmate.update({"name":"qinhanwen"},{"$set":{"age":25}});
$ db.workmate.update({"name":"qinhanwen"},{$inc:{"age":-2}});
```

![WX20190415-091029@2x](http://www.qinhanwen.xyz/WX20190415-091029@2x.png)



##### **multi选项**

是否修改全部数据，默认值false

```shell
$ var workmate1 = {name:'zenghua',age:26,skill:{name:'lalala'}};
$ db.workmate.insert(workmate1);
$ db.workmate.find();

#如果是multi为false，只会修改第一笔数据的值
$ db.workmate.update({},{$set:{favorite:[]}},{multi:true});
$ db.workmate.find();
```

![WX20190415-091558@2x](http://www.qinhanwen.xyz/WX20190415-091558@2x.png)



**upsert选项**

找不到值，直接插入新数据

```shell
$ db.workmate.find();
$ db.workmate.update({name:'baoge'},{$set:{age:32}},{upsert:false});
$ db.workmate.find();
$ db.workmate.update({name:'baoge'},{$set:{age:32}},{upsert:true})
```

![WX20190415-091922@2x](http://www.qinhanwen.xyz/WX20190415-091922@2x.png)





## update数组修改器

##### **$push追加数组/内嵌文档值**

```shell
$ db.workmate.update({name:'qinhanwen'},{$push:{favorite:'draw'}});
```

![WX20190416-055710@2x](http://www.qinhanwen.xyz/WX20190416-055710@2x.png)



```shell
$ db.workmate.update({name:'qinhanwen'},{$push:{"skill.name1":'css'}});
```

![WX20190416-055946@2x](http://www.qinhanwen.xyz/WX20190416-055946@2x.png)



##### **$ne查找是否存在**

不存在在执行操作，存在就不执行

```shell
$ db.workmate.update({name:'qinhanwen',favorite:{$ne:'swimming'}},{$push:{favorite:'swimming'}})
```

![WX20190416-060659@2x](http://www.qinhanwen.xyz/WX20190416-060659@2x.png)



##### **$addToSet**

查看是否存在，不存在直接$push上去

```shell
$ db.workmate.update({name:'qinhanwen'},{$addToSet:{favorite:'sleep'}})
```



##### **$each 批量追加**

```shell
$ var favorite = ['book','basketball'];
$ db.workmate.update({name:'qinhanwen'},{$addToSet:{favorite:{$each:favorite}}})
```

![WX20190416-062354@2x](http://www.qinhanwen.xyz/WX20190416-062354@2x.png)



##### **$pop 删除数组值**

删除一次

- 1：从数组末端进行删除
- -1：从数组开端进行删除

```shell
$ db.workmate.update({name:'qinhanwen'},{$pop:{favorite:1}})
$ db.workmate.update({name:'qinhanwen'},{$pop:{favorite:-1}})
```

![WX20190416-062639@2x](http://www.qinhanwen.xyz/WX20190416-062639@2x.png)



**修改数组中某个下标的值**

```shell
$  db.workmate.update({name:'qinhanwen'},{$set:{"favorite.2":"Code"}})
```

![WX20190416-062921@2x](http://www.qinhanwen.xyz/WX20190416-062921@2x.png)





## 修改和返回参数

##### **findAndModify**

```shell
$ var myModify={
    findAndModify:"workmate",
    query:{name:'qinhanwen'},
    update:{$set:{age:18}},
    new:true    //更新完成，需要查看结果，如果为false不进行查看结果
}
$ var ResultMessage=db.runCommand(myModify);
$ printjson(ResultMessage)
```



**findAndModify属性值：**

- query：需要查询的条件/文档
- sort: 进行排序
- remove：[boolean]是否删除查找到的文档，值填写true，可以删除。
- new:[boolean]返回更新前的文档还是更新后的文档。
- fields：需要返回的字段
- upsert：没有这个值是否增加。







## 查询

先造数据

```shell
$ var workmate = {
  name:'qinhanwen',
  age:25,
  sex:1,
  favorite:['swimming'],
  skill:{
    name:'css',
    name1:'html',
    name2:'js'
  }
}
$ var workmate1 = {
  name:'meilaoban',
  age:30,
  sex:1,
  favorite:[''],
  skill:{
    name:'css',
    name1:'html',
    name2:'js'
  }
}
$ var workmate2 = {
  name:'zenghua',
  age:26,
  sex:0,
  favorite:['sleep'],
  skill:{
    name:'draw',
  }
}
db.workmate.insert([workmate,workmate1,workmate2]);
```



**查询会html的，只需要人名和爱好**

```shell
$ db.workmate.find({"skill.name1":"html"});
$ db.workmate.find({"skill.name1":"html"},{name:true,favorite:true});
$ db.workmate.find({"skill.name1":"html"},{name:true,favorite:true,_id:false});
```

![WX20190416-072453@2x](http://www.qinhanwen.xyz/WX20190416-072453@2x.png)



##### **不等修饰符**

- 小于($lt):英文全称less-than
- 小于等于($lte)：英文全称less-than-equal
- 大于($gt):英文全称greater-than
- 大于等于($gte):英文全称greater-than-equal
- 不等于($ne):英文全称not-equal 我们现在要查找一下，公司内年龄小于30大于25岁的人员。看下面的代码。





**查询会html，岁数大于26的**

```shell
$ db.workmate.find({"skill.name1":"html",age:{$gt:25}},{name:true,favorite:true,_id:false});
```



**查询会html，岁数小于等于25的**

```shell
$ db.workmate.find({"skill.name1":"html",age:{$lte:25}},{name:true,favorite:true,_id:false});
```

![WX20190416-072944@2x](http://www.qinhanwen.xyz/WX20190416-072944@2x.png)





## 多条件查询

**会html，年龄小于28**

```shell
$ db.workmate.find({"skill.name1":"html",age:{$lt:28}});
```

![WX20190417-061311@2x](http://www.qinhanwen.xyz/WX20190417-061311@2x.png)



##### **$in修饰符**

接受个数组

**查询年龄25和30的同事，输出名字和技能1**

```shell
$ db.workmate.find({age:{$in:[25,30]}},{"skill.name1":1,_id:0,name:1})
```

![WX20190417-062008@2x](http://www.qinhanwen.xyz/WX20190417-062008@2x.png)



##### **$or修饰符**

接受个数组

**查询年纪大于26，或者技能有画画的**

```shell
$ db.workmate.find({$or:[{age:{$gt:26}},{"skill.name":"draw"}]})
```

![WX20190417-063157@2x](http://www.qinhanwen.xyz/WX20190417-063157@2x.png)



##### **$and修饰符**

接受个数组

**查询男性，年纪大于26，输出名字**

```shell
$ db.workmate.find({$and:[{sex:1},{age:{$gt:26}}]},{name:1,_id:0})
```





##### **$not修饰符**

用法有点不一样

**查询年龄25到30，也就是小于25，大于30之外的**

```shell
$ db.workmate.find({age:{$not:{$lt:25,$gt:30}}},{name:1,_id:0})
```





## 数组查询

重新造下数据

```shell
$ db.workmate.drop();

$ var workmate = {
  name:'qinhanwen',
  age:25,
  sex:1,
  favorite:['swimming','coding','sleep'],
  skill:{
    name:'css',
    name1:'html',
    name2:'js'
  }
}
$ var workmate1 = {
  name:'meilaoban',
  age:30,
  sex:1,
  favorite:['1','2','3'],
  skill:{
    name:'css',
    name1:'html',
    name2:'js'
  }
}
$ var workmate2 = {
  name:'zenghua',
  age:26,
  sex:0,
  favorite:['sleep','draw'],
  skill:{
    name:'draw',
  }
}
db.workmate.insert([workmate,workmate1,workmate2]);
```



**查询爱好有游泳的**

```shell
$ db.workmate.find({favorite:'swimming'},{name:1,_id:0})
```

![WX20190417-070917@2x](http://www.qinhanwen.xyz/WX20190417-070917@2x.png)

如果favorite后面使用个[]包含，那就变成完整匹配



##### **$all-数组多项查询**

接受个数组，需要满足所有条件

**查询喜欢睡觉和游泳的**

```shell
$ db.workmate.find({favorite:{$all:['sleep','swimming']}})
```

![WX20190417-072512@2x](http://www.qinhanwen.xyz/WX20190417-072512@2x.png)



##### **$in-数组的或者查询**

接受个数组，只需满足一项条件

**查看喜欢睡觉或者喜欢游泳的**

```shell
$ db.workmate.find({favorite:{$in:['sleep','swimming']}},{name:1,_id:0})
```

![WX20190417-072313@2x](http://www.qinhanwen.xyz/WX20190417-072313@2x.png)



##### **$size-数组个数查询**

根据数组长度返回结果

**查询有3个爱好的人**

```shell
$ db.workmate.find({favorite:{$size:3}},{name:1,_id:0})
```

![WX20190417-072759@2x](http://www.qinhanwen.xyz/WX20190417-072759@2x.png)



##### **$slice-显示选项**

**查询每个人爱好的第一项**

```shell
$ db.workmate.find({},{name:1,favorite:{$slice:1},_id:false})
```

![WX20190417-073025@2x](http://www.qinhanwen.xyz/WX20190417-073025@2x.png)





## find的参数

- query：这个就是查询条件，MongoDB默认的第一个参数。
- fields：（返回内容）查询出来后显示的结果样式，可以用true和false控制是否显示。
- limit：返回的数量，后边跟数字，控制每次查询返回的结果数量。
- skip:跳过多少个显示，和limit结合可以实现分页。
- sort：排序方式，从小到大排序使用1，从大到小排序使用-1。



##### 分页例子

limit为0，目测是跳过第一条显示之后的所有。

```shell
$ db.workmate.find({},{name:true,age:true,_id:false}).limit(0).skip(1);
```

![WX20190417-081159@2x](http://www.qinhanwen.xyz/WX20190417-081159@2x.png)











