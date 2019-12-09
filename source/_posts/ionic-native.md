---
title: ionic-native
date: 2019-08-13 20:03:32
tags: 
- ionic-native
categories: 
- ionic-native
---

Ionic-native5.x版本以上处理装饰器，在`npm run build:esm`的scripts命令里，有使用到`scripts/build/transformers/plugin-class.ts`文件里的方法来处理，这个后面再来研究。先写4.x版本的。



## 新建项目

```shell
$ npm install -g ionic
$ ionic start myApp
$ npm i -g cordova

#安装插件
$ ionic cordova plugin add cordova-plugin-camera
#移除插件
# ionic cordova plugin remove cordova-plugin-camera
$ npm install @ionic-native/camera --save
```

将插件引入`app.module.ts`文件的providers中。

![WX20190814-204936@2x](http://www.qinhanwen.xyz/WX20190814-204936@2x.png)



业务模块内导入

![WX20190814-205237@2x](http://www.qinhanwen.xyz/WX20190814-205237@2x.png)



页面

![WX20190814-205308@2x](http://www.qinhanwen.xyz/WX20190814-205308@2x.png)

这边安装的@ionic-native与@ionic-native版本都是4.14.0



## 打包

连接手机

```shell
$ npm install -g ios-deploy
$ ionic cordova platform add ios
$ ionic cordova build ios
```

碰到第一个报错

```javascript
npm ERR! ios-deploy@1.9.4 preinstall: `./src/scripts/check_reqs.js && xcodebuild`
npm ERR! Exit status 65
```

解决方案：

```shell
$ brew install ios-deploy
```



第二个问题

![WX20190814-205910@2x](http://www.qinhanwen.xyz/WX20190814-205910@2x.png)

打包失败，没有证书，打开xcode，打开`platforms/ios/MyApp.codeproj`，配置证书，继续打包。



## @ionic-native与@ionic-native/camera

@ionic-native/plugins/camera/index.ts

```javascript
import { Injectable } from '@angular/core';
import { Cordova, IonicNativePlugin, Plugin } from '@ionic-native/core';

export interface CameraOptions {
  quality?: number;
  destinationType?: number;
  sourceType?: number;
  allowEdit?: boolean;
  encodingType?: number;
  targetWidth?: number;
  targetHeight?: number;
  mediaType?: number;
  correctOrientation?: boolean;
  saveToPhotoAlbum?: boolean;
  cameraDirection?: number;
  popoverOptions?: CameraPopoverOptions;
}

export interface CameraPopoverOptions {
  x: number;
  y: number;
  width: number;
  height: number;
  arrowDir: number;
}

export enum DestinationType {
  DATA_URL = 0,
  FILE_URL,
  NATIVE_URI
}

export enum EncodingType {
  JPEG = 0,
  PNG
}

export enum MediaType {
  PICTURE = 0,
  VIDEO,
  ALLMEDIA
}

export enum PictureSourceType {
  PHOTOLIBRARY = 0,
  CAMERA,
  SAVEDPHOTOALBUM
}

export enum PopoverArrowDirection {
  ARROW_UP = 1,
  ARROW_DOWN,
  ARROW_LEFT,
  ARROW_RIGHT,
  ARROW_ANY
}

export enum Direction {
  BACK = 0,
  FRONT
}

@Plugin({
  pluginName: 'Camera',
  plugin: 'cordova-plugin-camera',
  pluginRef: 'navigator.camera',
  repo: 'https://github.com/apache/cordova-plugin-camera',
  platforms: ['Android', 'Browser', 'iOS', 'Windows']
})
@Injectable()
export class Camera extends IonicNativePlugin {
  DestinationType = {
    DATA_URL: 0,
    FILE_URI: 1,
    NATIVE_URI: 2
  };

  EncodingType = {
    JPEG: 0,
    PNG: 1
  };

  MediaType = {
    PICTURE: 0,
    VIDEO: 1,
    ALLMEDIA: 2
  };

  PictureSourceType = {
    PHOTOLIBRARY: 0,
    CAMERA: 1,
    SAVEDPHOTOALBUM: 2
  };

  PopoverArrowDirection = {
    ARROW_UP: 1,
    ARROW_DOWN: 2,
    ARROW_LEFT: 4,
    ARROW_RIGHT: 8,
    ARROW_ANY: 15
  };

  Direction = {
    BACK: 0,
    FRONT: 1
  };


  @Cordova({
    callbackOrder: 'reverse'
  })
  getPicture(options?: CameraOptions): Promise<any> {
    return;
  }

  @Cordova({
    platforms: ['iOS']
  })
  cleanup(): Promise<any> {
    return;
  }
}

```

主要看Plugin和Cordova装饰器

Plugin遍历了传入的config配置文件，为cls添加静态方法

```javascript
export function Plugin(config: PluginConfig): ClassDecorator {
  return (cls: any) => {
    // Add these fields to the class
    for (const prop in config) {
      cls[prop] = config[prop];
    }

    cls['installed'] = (printWarning?: boolean) => {
      return !!getPlugin(config.pluginRef);
    };

    cls['getPlugin'] = () => {
      return getPlugin(config.pluginRef);
    };

    cls['checkInstall'] = () => {
      return checkAvailability(cls) === true;
    };

    cls['getPluginName'] = () => {
      return config.pluginName;
    };

    cls['getPluginRef'] = () => {
      return config.pluginRef;
    };

    cls['getPluginInstallName'] = () => {
      return config.plugin;
    };

    cls['getPluginRepo'] = () => {
      return config.repo;
    };

    cls['getSupportedPlatforms'] = () => {
      return config.platforms;
    };

    return cls;
  };
}
```

而Cordova装饰器，在调用方法的时候会调用到wrap方法

```javascript
export function Cordova(opts: CordovaOptions = {}) {
  return (
    target: Object,
    methodName: string,
    descriptor: TypedPropertyDescriptor<any>
  ) => {
    return {
      value(...args: any[]) {
        return wrap(this, methodName, opts).apply(this, args);
      },
      enumerable: true
    };
  };
}
```



## 调试

触发点击事件

![WX20191030-212013@2x](http://www.qinhanwen.xyz/WX20191030-212013@2x.png)

获取插件实例

![WX20191030-220527@2x](http://www.qinhanwen.xyz/WX20191030-220527@2x.png)

![WX20191030-220613@2x](http://www.qinhanwen.xyz/WX20191030-220613@2x.png)

调用实例方法

![WX20191030-220841@2x](http://www.qinhanwen.xyz/WX20191030-220841@2x.png)

![WX20191030-220713@2x](http://www.qinhanwen.xyz/WX20191030-220713@2x.png)

postMessage通信

![WX20191030-221034@2x](http://www.qinhanwen.xyz/WX20191030-221034@2x.png)



如果兼容多平台，可以考虑在get方法那配个映射表，修改获得的插件实例。