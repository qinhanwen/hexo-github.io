---
title: VDOM 和 Render方法
date: 2018-11-23 22:47:56
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







##### 顺带一提Mutation Observer API，稍微了解一点点

就是这个api可以监听DOM中的变化，不过不是实时监听，而是一段时间的变化的结果。如果文档中连续插入1000个`<p>`元素，就会连续触发1000个插入事件，执行每个事件的回调函数，这很可能造成浏览器的卡顿；而 Mutation Observer 完全不同，只在1000个段落都插入结束后才会触发，而且只触发一次。

- 它等待所有脚本任务完成后，才会运行（即异步触发方式）。
- 它把 DOM 变动记录封装成一个数组进行处理，而不是一条条个别处理 DOM 变动。
- 它既可以观察 DOM 的所有类型变动，也可以指定只观察某一类变动。

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>

<body>
    <p id="p">改变下试试</p>
</body>
<script>
    var p = document.getElementById('p');
    //创建一个回调
    var callback = function (mutations) {
        console.log(mutations);
    }
    //实例话的时候传入回调
    var observer = new MutationObserver(callback);
    //这个options是需要观察哪些变化
    var options = {
        'childList': true,//子节点的变动（指新增，删除或者更改）。
        'attributes': true,//属性的变动。
        'characterData': true//节点内容或节点文本的变动。
    };
    //开始观察
    observer.observe(p,options);

    p.innerHTML = '改变了内容';
    //取消观察
    observer.disconnect();
    p.innerHTML = '再次改变内容';
</script>

</html>
```



> 参考资料 
>
> https://engineering.hexacta.com/didact-learning-how-react-works-by-building-it-from-scratch-51007984e5c5
>
> http://javascript.ruanyifeng.com/dom/mutationobserver.html