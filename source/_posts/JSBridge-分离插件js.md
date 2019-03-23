---
title: JSBridge-分离插件js
date: 2018-12-11 23:04:36
tags: 
- JSBridge
categories: 
- JSBridge
---

# JSBridge-分离插件js

宝哥总是说做Hybird应用开发的，这个要了解一下，那就了解一下哈哈哈哈。刚好有机会写个接入的文档，总结一下流程。

#### 1.JSBridge

其实就是Native和Web之间通信的桥梁。



#### 2.需求

1）引入lauch.js，根据ua判断，引入不同平台的cordova.js。

2）引入的cordova.js会根据配置文件cordova.plugin.js里的配置数据（数组），然后再动态创建script标签，引入不同的插件js。（cordova.js、cordova.plugin.js以及插件js全都会放在cdn上，根据用户配置信息载入不同的文件，变成可配置的）。

3）调用插件，文档需要标明有哪些输入与输出属性。

4）了解不同平台之间的通信方式(IOS与Android)。



####3.准备工作（这里不怎么重要。。）

##### 1）首先，新建一个ionic1的项目

```shell
$ sudo npm install -g cordova ionic
$ ionic start myApp tabs --type ionic1
```



##### 2）ngCordova安装配置（也可以忽略）

其实就是多封装了一层

```shell
##首先安装bower，用于安装和卸载如JavaScript、HTML、CSS之类的网络资源。
$ npm install -g bower
##安装ngCordova
$ bower install ngCordova
```

在www/index.html引入

![WX20190128-183902@2x](http://www.qinhanwen.xyz/images/WX20190128-183902@2x.png)

并在www/js/app.js注入

![WX20190128-184112@2x](http://www.qinhanwen.xyz/images/WX20190128-184112@2x.png)

##### 3）装个相机插件

```shell
##添加平台
$ ionic cordova platform add ios
##安装插件
$ cordova plugin add cordova-plugin-camera
```





##### 4）真机调试

使用xcode安装到手机上，接着碰到一个问题导致调用插件的时候app直接闪退

![WX20190129-124901@2x](http://www.qinhanwen.xyz/images/WX20190129-124901@2x.png)

解决方案，增加权限，在下面的文件以`Source Code`方式打开并加这一段

```
<key>NSCameraUsageDescription</key>
<string>The camera is used to scan QR codes.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string/>
<key>NSMainNibFile</key>
<string/>
<key>NSMainNibFile~ipad</key>
<string/>
<key>NSPhotoLibraryUsageDescription</key>
<string/>
```

如图：

![WX20190129-125233@2x](http://www.qinhanwen.xyz/images/WX20190129-125233@2x.png)



到这里准备工作就完成了。





#### 4.实现流程（因为cdn上传文件夹不大方便，于是我放到了自己的服务器上，直接放在了之前放图片的images目录下，忽略这个尴尬的路径名称）

##### 1.单页应用只需要在`index.html`引入`lauch.js`，这个js做的事情，就是根据不同平台，引入不同的`cordova.js`

```javascript
(function($window){ 'use strict';
    var ua = $window.navigator.userAgent;
    var isIOS = false;
    var isAndroid = false;
    var platform = 'web';
    var filePath = '';
    if(new RegExp('\\biPhone.*Mobile|\\biPod|\\biPad|AppleCoreMedia', 'i').test(ua)) {
        isIOS = true;
        platform = 'ios';
    }
    if(new RegExp('Android', 'i').test(ua)) {
        isAndroid = true;
        platform = 'android';
    }

    if(isIOS || isAndroid) {
        filePath = 'http://www.qinhanwen.xyz/images/';
        setTimeout(function(){
            createAndAddScript(filePath + platform + '/cordova.js');
        }, 0);
    }

    function createAndAddScript(url, onload, onerror) {
        var script = document.createElement('script');
        // onload fires even when script fails loads with an error.
        script.onload = onload;
        // onerror fires for malformed URLs.
        script.onerror = onerror;
        script.src = url;
        document.head.appendChild(script);
    };
})(this);
```

之后上传一份IOS平台引用的`cordova.js`文件，地址为`http://www.qinhanwen.xyz/images/ios/cordova.js`



更改`index.html`，引入`lauch.js`，如图：

![WX20190129-172110@2x](http://www.qinhanwen.xyz/images/WX20190129-172110@2x.png)



打到真机上调试，当`lauch.js`加载成功后，加载了ios平台的`cordova.js`文件之后，加载`cordova_plugins.js`文件的时候加载失败

![WX20190129-172759@2x](http://www.qinhanwen.xyz/images/WX20190129-172759@2x.png)



###### 那么这个`cordova_plugins.js`文件是干嘛的，它是在什么时候加载？

1）首先cordova.js执行过程中，会获取了页面中所有`script`标签的`src`，并且找出了`cordova.js`的前缀，保存前缀为`pathPrefix`字段，拼接`pathPrefix`+`cordova_plugins.js`得到一个`url`，这个`url`也就是`cordova_plugins.js`文件的地址

2）接着动态创建`script`标签，`src`设置为`url`，之后把`script`标签加入`head`内



##### 2.所以我又在服务器上的`ios`目录下上传了`cordova_plugins.js`文件

之后`cordova_plugins.js`是加载成功了，但是又出现一大堆的404。

![WX20190129-173610@2x](http://www.qinhanwen.xyz/images/WX20190129-173610@2x.png)

其实`cordova_plugin.js`里有一个数组，里面有插件的信息，解析数组之后，会动态创建script标签，引入这些插件js，如图：

![WX20190129-173750@2x](http://www.qinhanwen.xyz/images/WX20190129-173750@2x.png)



##### 3.接着我把整个插件文件夹都放到了服务器ios目录下

重新加载应用，加载插件js成功了，可以看到`index.html`审查元素变成了如图：

![WechatIMG174](http://www.qinhanwen.xyz/images/WechatIMG174.png)





#### 5.通信方式

##### 在安装的时候默认是使用WKWebView，通过在config.xml文件中配置，这样子可以使用UIWebView

```xml
 <preference name="CordovaWebViewEngine" value="CDVUIWebViewEngine" />
```

![WX20190130-185249@2x](http://www.qinhanwen.xyz/images/WX20190130-185249@2x.png)



##### IOS端：

首先在调用过程中，传递参数的时候发现会把传入的成功与失败的回调，存在一个对象内，key是一个callbackId

![WX20190130-213640@2x](http://www.qinhanwen.xyz/images/WX20190130-213640@2x.png)

###### 1）UIWebView的通信方式：

在从controller开始调用的地方进行断点，一步一步往内层调用走，发现了一个pokeNative函数，这个是轮训。从上面一张图可以看到一个commandQueue，压入了一个command数组，其实就是调用的一些信息，还有标识callbackId，原生接收到消息，就可以取得这些数据。然后通过callbackId找到对应的回调。

![WX20190130-212710@2x](http://www.qinhanwen.xyz/images/WX20190130-212710@2x.png)



###### 2）WKWebView的通信方式

![WX20190130-232029@2x](http://www.qinhanwen.xyz/images/WX20190130-232029@2x.png)

command直接通过postMessage方法传入。

##### Android端：

重写了prompt方法



至于Android端和IOS端的处理方式，等同事上完课再补。



#### 补充

iframe.src 和 locaiton.href：iframe.src每次都会发送请求，而location.href，连续调用会丢失一些请求。

