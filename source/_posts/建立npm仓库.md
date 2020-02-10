---
title: 建立npm仓库
date: 2020-02-06 13:49:25
tags: 
- npm
categories: 
- npm
---

## 私有npm仓库建立

```shell
$ npm install -g verdaccio --unsafe-perm
$ verdaccio
$ vi /root/.config/verdaccio/config.yaml

# 启动
$ verdaccio 
```

访问 域名+端口号

![WX20200130-170757@2x](http://114.55.30.96/WX20200130-170757@2x.png)



使用 pm2 启动进程

```shell
$ npm i pm2 -g
$ pm2 start verdaccio
```

