---
title: SVG
date: 2019-03-01 14:47:29
tags:
---

## SVG

SVG，即可缩放矢量图形(Scalable Vector Graphics)，是一种 XML 应用，可以以一种简洁、可移植的形式表示图形信息。目前，人们对 SVG 越来越感兴趣。大多数现代浏览器都能显示 SVG 图形，并且大多数矢量绘图软件都能导出 SVG 图形。SVG 主要可以概括为以下几点：

- SVG 指可伸缩矢量图形
- SVG 用来定义网络的基于矢量的图形
- SVG 使用 XML 格式定义图形
- SVG 图像在放大或改变尺寸的情况下其图形质量不会有所损失
- SVG 是万维网联盟的标准
- SVG 与诸如 DOM 和 XSL 之类的 W3C 标准是一个整体

## SVG 与 Canvas 区别

![WX20191220-230618@2x](http://118.24.241.76/WX20191220-230618@2x.png)

## 创建 SVG 图像

```html
<svg width="10" height="10">
  <rect x="0" y="0" width="10" height="10" fill="blue" />
</svg>
```

## SVG 元素

最常用的：

- `text`: 创建一个 text 元素

- `circle`: 创建一个圆

- `rect`: 创建一个矩形

- `line`: 创建一条线

- `path`: 在两点之间创建一条路径

- `textPath`: 在两点之间创建一条路径，并创建一个链接文本元素

- `polygon`: 允许创建任意类型的多边形

- `g`: 单独的元素

> 坐标从绘图区域左上角的 0，0 开始，并 **从左到右**表示 `x`，**从上到下**表示 `y`。

```html
<!-- text 元素添加文本。可以使用鼠标选择文本。x 和 y 定义文本的起始点 -->
<svg>
  <text x="5" y="30">A nice rectangle</text>
</svg>

<!-- cx 和 cy 是中心坐标，r 是半径。 fill 是一个常用属性，表示图形颜色。 -->
<svg>
  <circle cx="50" cy="50" r="50" fill="#529fca" />
</svg>

<!-- 定义矩形。 x ， y 是起始坐标，width 和 height 是自解释的。 -->
<svg>
  <rect x="0" y="0" width="100" height="100" fill="#529fca" />
</svg>

<!-- x1 和 y1 定义起始坐标。x2 和 y2 定义结束坐标。stroke 是一个常用属性，表示线条颜色。 -->
<svg>
  <line x1="0" y1="0" x2="100" y2="100" stroke="#529fca" />
</svg>

<!-- 路径是一系列的直线和曲线。它是所有 SVG 绘制工具中最强大的，因此也是最复杂的。
    d 包含方向命令。这些命令以命令名和一组坐标开始：
      M 表示移动，它接受一组 x，y 坐标
      L 表示直线将绘制到它接受一组 x，y
      H 是一条水平线，它只接受 x 坐标
      V 是一条垂直线，它只接受 y 坐标
      Z 表示关闭路径，并将其放回起始位置
      A 表示 Arch，它自己需要一个完整的教程
      Q 是一条二次 Bezier 曲线，同样，它自己也需要一个完整的教程 -->
<svg height="300" width="300">
  <path
    d="M 100 100 L 200 200 H 10 V 40 H 70"
    fill="#59fa81"
    stroke="#d85b49"
    stroke-width="3"
  />
</svg>

<!-- 使用 polygon 绘制任意多边形。 points 代表一组 x，y 坐标多边形应该链接： -->
<svg>
  <polygon points="9.9, 1.1, 3.3, 21.78, 19.8, 8.58, 0, 8.58, 16.5, 21.78" />
</svg>

<!-- 使用 g 元素，您可以对多个元素进行分组： -->
<svg width="200" height="200">
  <rect x="0" y="0" width="100" height="100" fill="#529fca" />
  <g id="my-group">
    <rect x="0" y="100" width="100" height="100" fill="#59fa81" />
    <rect x="100" y="0" width="100" height="100" fill="#59fa81" />
  </g>
</svg>
```

![WX20191220-232354@2x](http://118.24.241.76/WX20191220-232354@2x.png)

## SVG viewport 和 viewBox

**viewport**：SVG 相对于其容器的大小由 `svg` 元素的 `width` 和 `height` 属性设置。这些单位默认为像素，但您可以使用任何其他常用单位，如 `%` 或 `em`。

**viewBox**：它允许您在 SVG 画布中定义一个新的坐标系统。

假设在 200x200px SVG 中有一个简单的圆：

```html
<svg width="200" height="200">
  <circle cx="100" cy="100" r="100" fill="#529fca" />
</svg>
```

![WX20191220-233336@2x](http://118.24.241.76/WX20191220-233336@2x.png)

通过指定 **viewBox** ，您可以选择 **只显示此 SVG 的一部分**。例如，您可以从 0，0 点开始，只显示一个 100 x 100 px 画布：

```html
<svg width="200" height="200" viewBox="0 0 100 100">
  <circle cx="100" cy="100" r="100" fill="#529fca" />
</svg>
```

![WX20191220-233348@2x](http://118.24.241.76/WX20191220-233348@2x.png)

如果应用的时候需要设置 viewport 宽高，可以通过 viewbox 来辅助控制大小

```html
<svg width="100" height="100">
  <circle cx="100" cy="100" r="50" style="stroke:#000;fill:none" />
  <circle cx="85" cy="85" r="5" style="fill:#000" />
  <circle cx="115" cy="85" r="5" style="fill:#000" />
  <g id="whiskers">
    <line x1="105" y1="105" x2="165" y2="95" style="stroke:black"></line>
    <line x1="105" y1="105" x2="165" y2="115" style="stroke:black"></line>
  </g>
  <use
    xlink:href="#whiskers"
    href="#whiskers"
    transform="scale(-1 1) translate(-200 0)"
  />
</svg>
```

![WX20191221-205250@2x](http://118.24.241.76/WX20191221-205250@2x.png)

```html
<svg width="100" height="100" viewBox="0,0,200,200">
  ....
</svg>
```

![WX20191221-205406@2x](http://118.24.241.76/WX20191221-205406@2x.png)

## 在 Web 网页中插入 SVG

最常见的是：

- 带有 `img` 标签

```html
<img src="flag.svg" alt="Flag" />
```

- 带有 CSS `background-image` 属性

```html
<style>
  .svg-background {
    background-image: url(flag.svg);
    height: 200px;
    width: 300px;
  }
</style>
<div class="svg-background"></div>
```

- 在 HTML 中内联

```html
<svg
  width="300"
  height="200"
  viewBox="0 0 300 200"
  version="1.1"
  xmlns="http://www.w3.org/2000/svg"
  xmlns:xlink="http://www.w3.org/1999/xlink"
>
  <title>Italian Flag</title>
  <desc>By Flavio Copes https://flaviocopes.com</desc>
  <g id="flag">
    <rect fill="green" x="0" y="0" width="100" height="200"></rect>
    <rect fill="white" x="100" y="0" width="100" height="200"></rect>
    <rect fill="red" x="200" y="0" width="100" height="200"></rect>
  </g>
</svg>
```

- 带有 `object`、 `iframe` 或 `embed` 标签

```html
<object data="flag.svg" type="image/svg+xml"></object>

<iframe src="flag.svg" frameborder="0"></iframe>

<embed src="flag.svg" type="" />
```

## 样式元素

任何 SVG 元素都可以接受 `style` 属性，就像 HTML 标签一样。并非所有的 CSS 属性都能像您预期的那样工作。例如，要更改文本元素的颜色，请使用 `fill` 而不是 `color`。

```html
<svg>
  <text x="5" y="30" style="fill: green">A nice text</text>
</svg>

<svg>
  <text x="5" y="70" style="fill: green; font-family: Courier New">
    A nice text
  </text>
</svg>

<!-- 也可以使用 fill 作为元素属性 -->
<svg>
  <text x="5" y="70" fill="green">A nice text</text>
</svg>
```

![WX20191220-235534@2x](http://118.24.241.76/WX20191220-235534@2x.png)

其他公共属性包括

- `fill-opacity`，背景颜色不透明度
- `stroke`，定义边框颜色
- `stroke-width`，设置边框宽度

CSS 可以针对 SVG 元素，就像您以 HTML 标签为目标一样：

```css
rect {
  fill: red;
}
circle {
  fill: blue;
}
```

## 使用 CSS 或 JavaSCript 与 SVG 交互

SVG 图像可以使用 CSS 进行样式化，或者使用 JavaScript 编写脚本，这种情况下：

- **当 SVG 在 HTML 中内联**
- 通过 `object` 、`embed` 或 `iframe` 标签加载图像时

**CSS 嵌入 SVG**

将 CSS 加至 CDATA:

```html
<svg>
  <style>
    <![CDATA[
      #my-rect { fill: blue; }
    ]]>
  </style>
  <rect id="my-rect" x="0" y="0" width="10" height="10" />
</svg>
```

![WX20191221-141217@2x](http://118.24.241.76/WX20191221-141217@2x.png)

SVG 文件还可以包括外部样式表

```html
<?xml version="1.0" standalone="no"?>
<?xml-stylesheet type="text/css" href="style.css"?>
<svg
  xmlns="http://www.w3.org/2000/svg"
  version="1.1"
  width=".."
  height=".."
  viewBox=".."
>
  <rect id="my-rect" x="0" y="0" width="10" height="10" />
</svg>
```

**JavaScript 嵌入 SVG**

你可以将 JavaScript 放在第一个位置上，并包装在一个 `load` 事件中，以便在页面完全加载并在 DOM 中插入 SVG 时执行它：

```html
<svg>
  <script>
    <![CDATA[ window.addEventListener("load", () => { //... }, false) ]]>
  </script>
  <rect x="0" y="0" width="10" height="10" fill="blue" />
</svg>
```

或者如果您将 JS 放在其他 SVG 代码的末尾，可以避免添加事件监听，确保当 SVG 出现在页面时 JavaSCript 会执行。

```html
<svg>
  <rect x="0" y="0" width="10" height="10" fill="blue" />
  <script>
    <![CDATA[ //... ]]>
  </script>
</svg>
```

与 HTML 元素一样，SVG 元素也可以有 `id` 和 `class` 属性。

```html
<svg>
  <rect
    x="0"
    y="0"
    width="10"
    height="10"
    fill="blue"
    id="my-rect"
    class="a-rect"
  />
  <script>
    <![CDATA[ console.log(document.getElementsByTagName('rect'))
    console.log(document.getElementById('my-rect'))
    console.log(document.querySelector('.a-rect'))
    console.log(document.querySelectorAll('.a-rect')) ]]>
  </script>
</svg>
```

**SVG 外部的 JavaScript**

如果可以与 SVG 交互（SVG 在 HTML 中是内联的），则可以使用 JavaScript 更改任何 SVG 属性，

```javascript
document.getElementById('my-svg-rect').setAttribute('fill', 'black')
```

**SVG 外部的 CSS**

```html
<style>
  #my-rect {
    fill: red;
  }
</style>
<svg>
  <rect x="0" y="0" width="10" height="10" fill="blue" id="my-rect" />
</svg>
```

## SVG 符号

符号使您可以定义一次 SVG 图像，并在多个地方重用它。如果您需要重用一个图像，这是一个很大的帮助，可能只是改变一点它的一些属性

您可以通过添加一个 `symbol` 元素并分配一个 `id` 属性来完成此操作：

```html
<svg class="hidden">
  <symbol id="rectangle" viewBox="0 0 20 20">
    <rect x="0" y="0" width="300" height="300" fill="rgb(255,159,0)" />
  </symbol>
</svg>
```

如何使用：

```html
<svg>
  <use xlink:href="#rectangle" href="#rectangle" />
</svg>
```

(`xlink:href` 用于 Safari 支持，即使它是一个已废弃的属性)

如果您希望对这两个矩形使用不同的样式，例如，对每个矩形使用不同的颜色？您可以使用[CSS 变量](https://flaviocopes.com/css-variables/).

```html
<svg class="hidden">
  <symbol id="rectangle" viewBox="0 0 20 20">
    <rect x="0" y="0" width="300" height="300" fill="var(--color)" />
  </symbol>
</svg>
```

```html
<svg class="blue">
  <use xlink:href="#rectangle" href="#rectangle" />
</svg>

<svg class="red">
  <use xlink:href="#rectangle" href="#rectangle" />
</svg>

<style>
  svg.red {
    --color: red;
  }
  svg.blue {
    --color: blue;
  }
</style>
```

## 参考资料

[svg](https://juejin.im/post/5ad84f296fb9a045e8011be1)

[svg](https://juejin.im/post/5deee313518825121c330c8e)

[SVG 之 ViewBox](https://segmentfault.com/a/1190000009226427)
