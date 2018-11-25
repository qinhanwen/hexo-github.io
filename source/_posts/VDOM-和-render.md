---
title: VDOM 和 Render方法
date: 2018-11-23 22:47:56
tags:
---

# VDOM和Render方法

##### 1.用普通的JS对象去描述我们需要渲染的内容。

```javascript
const element = {
  type: "div",
  props: {
    id: "container",
    children: [
      { type: "input", props: { value: "foo", type: "text" } },
      {
        type: "a",
        props: {
          href: "/bar",
          children: [{ type: "TEXT ELEMENT", props: { nodeValue: "bar" } }]
        }
      },
      {
        type: "span",
        props: {
          onClick: "alert('Hi')",
          children: [{ type: "TEXT ELEMENT", props: { nodeValue: "click me" } }]
        }
      }
    ]
  }
};
```

对象所描述的DOM结构

```html
<div id="container">
    <input value="foo" type="text" />
    <a href="/bar">bar</a>
    <span onClick="alert('Hi')">click me</span>
</div>
```



##### 2.实现对象转成DOM的Render方法

1）先实现一个简单的，只有type和props里的属性

```javascript
const element = {
  type: "div", 
  props: {
    id: "container",
  }
};
```
render方法

```javascript
function render(element, parent) {

    const { type, props } = element;
    const dom = document.createElement(type);

    Object.keys(props).forEach(function (item) {
        dom.setAttribute(item, props[item]);
    });
    parent.appendChild(dom);
}
```

所描述的DOM结构

```html
<div id="container"></div>
```



2）然后加入children属性

```javascript
const element = {
    type: "div",
    props: {
        id: "container",
        children: [
            { type: "input", props: { value: "foo", type: "text" } },
            { type: "a", props: { href: "/bar" } },
            { type: "span", }
        ]
    }
};
```

render方法

```javascript
        function render(element, parent) {

            const { type, props } = element;
            const dom = document.createElement(type);

            props instanceof Object && Object.keys(props).filter(function (item) { return !Array.isArray(props[item]) }).forEach(function (item) {//处理props里非数组的，就是属性或者监听事件
                dom.setAttribute(item, props[item]);
            });

            if (props && props.children && props.children.length) {
                props.children.forEach(function (item) {
                    render(item, dom);
                })
            }
            parent.appendChild(dom);
        }
```

所描述的DOM结构

```html
<div id="container">
  <input value="foo" type="text">
  <a href="/bar"></a>
  <span></span>
</div>
```



3) 如果里面有纯文本类型

```javascript
const element = {
    type: "div",
    props: {
        id: "container",
        children: [
            { type: "input", props: { value: "foo", type: "text" } },
            { type: "a", props: { href: "/bar" } },
            {
                type: "span",
                props: {
                    children: [
                        {
                            type: "TEXT ELEMENT",
                            props: { nodeValue: "Foo" }
                        }
                    ]
                }
            }
        ]
    }
};
```

render方法

```javascript
function render(element, parent) {
        const TEXT_TYPE = 'TEXT ELEMENT';//纯文本类型

        const { type, props } = element;
        let dom;
        if (type === TEXT_TYPE) {
            dom = document.createTextNode(props.nodeValue)
        } else {
            dom = document.createElement(type);
            props instanceof Object && Object.keys(props).filter(function (item) { return !Array.isArray(props[item]) 				}).forEach(function (item) {//处理props里非数组的，就是属性或者监听事件
                dom.setAttribute(item, props[item]);
            });
            if (props && props.children && props.children.length) {
                props.children.forEach(function (item) {
                    render(item, dom);
                })
            }
        }
        parent.appendChild(dom);
 }
```

所描述的DOM结构

```html
<div id="container">
  <input value="foo" type="text">
  <a href="/bar"></a>
  <span>Foo</span>
</div>
```



4）对象中的监听事件处理

```java
const element = {
    type: "div",
    props: {
        id: "container",
        children: [
            { type: "input", props: { value: "foo", type: "text" } },
            { type: "a", props: { href: "/bar" } },
            {
                type: "span",
                props: {
                    onClick: "alert('Hi')",
                    children: [
                        {
                            type: "TEXT ELEMENT",
                            props: { nodeValue: "Foo" }
                        }
                    ]
                }
            }
        ]
    }
};
```

render方法

```javascript
    function render(element, parent) {
        const TEXT_TYPE = 'TEXT ELEMENT';//纯文本类型

        const { type, props } = element;
        let dom;
        if (type === TEXT_TYPE) {
            dom = document.createTextNode(props.nodeValue)
        } else {
            dom = document.createElement(type);
            Object.keys(props).filter(function (item) { return !Array.isArray(props[item]) }).forEach(function (item) {//处理props里非数组的，就是属性或者监听事件
                const key = item.startsWith("on") ? item.toUpperCase() : item;
                dom.setAttribute(key, props[item]);

            });
            if (props && props.children && props.children.length) {
                props.children.forEach(function (item) {
                    render(item, dom);
                })
            }
        }
        parent.appendChild(dom);
    }
```

所描述的DOM结构

```html
<div id="container">
  <input value="foo" type="text">
  <a href="/bar"></a>
  <span onclick="alert('Hi')">Foo</span>
</div>
```



> 参考资料 
>
> https://engineering.hexacta.com/didact-learning-how-react-works-by-building-it-from-scratch-51007984e5c5
>