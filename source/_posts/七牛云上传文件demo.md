---
title: 七牛云上传文件demo
date: 2019-05-21 23:27:34
tags:
---

#### 一、服务端获取凭证

```shell
$ npm i qiniu -S
```

```javascript
var qiniu = require('qiniu');
// 1、定义鉴权对象mac
var accessKey = 'your accessKey';
var secretKey = 'your secretKey';//secretKey
var mac = new qiniu.auth.digest.Mac(accessKey, secretKey);
// 2、上传凭证
var options = {
  scope: 'your bucket', // Bucket
  expires: 3600, // 默认有效期时间
};
var putPolicy = new qiniu.rs.PutPolicy(options);
var uploadToken = putPolicy.uploadToken(mac);
console.log(uploadToken);
```



#### 二、客户端上传

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
    <input type="file" id="file" value="上传" onchange="change()" />
</body>
<script src="https://unpkg.com/qiniu-js@2.5.4/dist/qiniu.min.js"></script>

<script>
    var input = document.getElementById('file');
    function change() {
        let file = input.files[0];
        // 定义被观察者
        let observable = qiniu.upload(file, file.name, "your token")//你的token
        // 订阅被观察者，开始上传
        let subscription = observable.subscribe({
            next(res) {
                console.log('上传中')
            },
            error(err) {
            },
            complete(res) {
                console.log(res);
            }
        })
    }
</script>

</html>
```

