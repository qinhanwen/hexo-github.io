---
title: puppeteer入门
date: 2018-11-16 16:24:26
tags:
---

## Puppeteer入门

##### 1.背景需求

刚好公司的app，定制包比较多，当初经常出现app里二维码页面图片丢失或者有些白屏，打出来的包比较多又不好人工验证，所以可以使用Puppeteer。跳转到二维码页面截图，识别二维码，是否与预设的地址一样。还有每次有新客户的时候，就有友盟，微信，极光7，8个key需要申请，也可以使用puppeteer来完成。缺点就是维护成本比较高，需要经常更新。



##### 2.Puppeteer介绍

`Puppeteer`是一个`Nodejs`的库，可以用来做爬虫，加上百度文字识别，用于识别验证码，可以过一些简单表单提交，验证码比如一些字母加数字的比较好识别，一些不好识别图片就没办法了。也可以监听网络请求，比如监听到某些请求，然后去抓取里面的数据。应用场景还是蛮多的。



##### 3.Puppeteer入门

######  1.安装

```shell
npm i puppeteer
# or "yarn add puppeteer"
#会同时安装一个Chromium
```



###### 2.目录结构：

![](http://pi8irywwe.bkt.clouddn.com/WX20181117-164728@2x.png)



###### 3.简单的例子

```javascript
//一个截取百度图片的小例子
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch({headless: false});//无头模式默认是true，设置为false，还有很多其他属性就不介绍了
  const page = await browser.newPage();
  await page.goto('https://baidu.com');
  await page.screenshot({path: 'images/baidu.png'});

  await browser.close();
})();
```



##### 4.介绍几个比较实用的

###### 1.第一个当然是截屏（可以截取想要的位置，比如想识别一张验证码，需要先截取验证码的图片，再去识别）。

```javascript
const puppeteer = require('puppeteer');

(async () => {
    const browser = await puppeteer.launch({ headless: true });//无头模式默认是true，设置为false
    const page = await browser.newPage();
    await page.goto('http://www.exexm.com/');

    await page.waitForSelector('.logo>a>img');//等待元素
    const img = await page.$('.logo>a>img');//其实就是document.querySelector
    const boundingBox = await img.boundingBox();//获取一个对象有x，y，width，height属性
    await page.screenshot({ path: 'images/exe-logo.png', clip: boundingBox });//clip 指定页面剪切部分，对象，属性有x,y,width,height

    await browser.close();
})();
//这里是截取官网的logo。
```



###### 2.iframe标签里面的内容(登陆qq邮箱的简单例子)

```javascript
const puppeteer = require('puppeteer');

(async () => {
    const browser = await puppeteer.launch({ headless: false });//无头模式默认是true，设置为false
    const page = await browser.newPage();
    await page.goto('https://mail.qq.com/');
    
    //因为元素iframe标签内，所以要获取一下
    const frame = await page.frames().find(frame => frame.name() === 'login_frame');//找到name=login_frame的iframe
    const inputs = await frame.$$('.inputstyle');//其实就是document.querySelectorAll
    await inputs[0].type('你的邮箱');//在输入框中输入邮箱账号
    await inputs[1].type('你的邮箱密码');//在输入框中输入邮箱密码
    await frame.tap('.login_button');

    await browser.close();
})();
```



###### 3.鼠标事件（比如有些网站验证比较简单，输入账号密码之后，只需要拖动滑块便可完成验证）

```javascript
await page.mouse.move(0, 0);
await page.mouse.down();
await page.mouse.move(0, 100);
await page.mouse.up();
//代码片段
//我只需要获取滑块的x，y坐标，以及宽高，将鼠标移动过去，在按下，在继续移动，最后放开
```



###### 4.监听事件

```javascript
page.on('response', response => {
    console.log(response.url());
    if (response.url().indexOf('https://mail.qq.com')) {
    //需要做的操作
    }
});
//我可以在打开网页的时候，开始监听某些响应，并且可以获得响应内容
```



以上这些感觉是比较常用的东西。





