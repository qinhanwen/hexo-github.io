---
title: XML学习笔记
date: 2018-11-18 14:53:55
tags: 
- XML
categories: 
- XML
---

# XML基础学习笔记

##### 1.XML（Extensible Markup Language）可扩展标记语言。



##### 2.XML和HTML的区别

###### 1.设计目的不同：XML 被设计用来传输和存储数据，HTML 被设计用来显示数据。

###### 2.XML 不是 HTML 的替代。

###### 3.XML 被设计为传输和存储数据，其焦点是数据的内容。HTML 被设计用来显示数据，其焦点是数据的外观。

###### 4.XML没有预定义的标签，HTML则有



##### 3.XML的特点

###### 1.XML 仅仅是纯文本。

###### 2.可以自定义标签

###### 3.必须有个根元素

###### 4.XML 标签对大小写敏感，标签 <Letter> 与标签 <letter> 是不同的。

###### 5字符拥有特殊的意义，放在元素中会出错（>  <   &    '    "）符号。例如

```xml
<message>if salary < 1000 then</message>
<!-- 这样就会有问题 --> 
```



##### 4.XMLHttpRequest 对象

###### 1.XMLHttpRequest 对象用于在后台与服务器交换数据。

```javascript
//创建一个XMLHttpRequest对象
var xmlhttp;
if (window.XMLHttpRequest)
{
    //  IE7+, Firefox, Chrome, Opera, Safari 浏览器执行代码
    xmlhttp=new XMLHttpRequest();
}
else
{
    // IE6, IE5 浏览器执行代码
    xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
}
```

之后代码都使用`xmlhttp`这个对象



###### 2.设置请求头（POST请求通常应设置Content-Type请求头）

```javascript
xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
```



###### 3.发送请求

```javascript
xmlhttp.open("GET","url",true);// method,url,async
xmlhttp.send();//POST参数放在方法里参数形式为：name=qinhanwen&age=18
```



###### 4.onreadystatechange事件中就绪执行的函数

```javascript
xmlhttp.onreadystatechange=function()
{
    if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
        //do something
    }
}
```



###### 5.服务器响应（ XMLHttpRequest 对象的 responseText 或 responseXML 属性）

```javascript
xmlhttp.responseText//字符串形式
xmlhttp.responseXML//XML形式
```



##### 5.XMl解析

```javascript
if (window.XMLHttpRequest)
  {// code for IE7+, Firefox, Chrome, Opera, Safari
  xmlhttp=new XMLHttpRequest();
  }
else
  {// code for IE6, IE5
  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
  }

xmlhttp.open("GET","books.xml",false);
xmlhttp.send();
xmlDoc=xmlhttp.responseXML; 
```



##### 6.XML DOM

```javascript
xmlDoc.getElementsByTagName("to")[0].childNodes[0].nodeValue
//xmlDoc -由解析器创建的 XML 文档
//getElementsByTagName("to")[0] - 第一个 <to> 元素
//childNodes[0] - <to> 元素的第一个子元素（文本节点）
//nodeValue - 节点的值（文本本身
```



