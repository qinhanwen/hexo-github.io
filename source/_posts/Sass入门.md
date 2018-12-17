---
title: Sass入门
date: 2018-11-23 18:09:08
tags: 
- css
categories: 
- css
---

# Sass入门

##### 1.安装Sass（window需要安装[Ruby](https://rubyinstaller.org/downloads/),安装过程中请注意勾选`Add Ruby executables to your PATH`添加到系统环境变量）

```shell
gem install sass
```



##### 2.编译Sass

```shell
#单文件转换命令
sass test/test.scss output/output.css

#单文件监听命令
sass --watch test/test.scss:output/output.css

#如果你有很多的sass文件的目录，你也可以告诉sass监听整个目录：
sass --watch test:output

```



##### 3.拓展

######   1.嵌套 

```scss
#main p {
    color: #00ff00;
    width: 97%;
  
    .redbox {
      background-color: #ff0000;
      color: #000000;
    }
  }
```
编译为

```css
#main p {
  color: #00ff00;
  width: 97%; }
  #main p .redbox {
    background-color: #ff0000;
    color: #000000; }
```



###### 2.引用父选择器（&）

```scss
a {
    font-weight: bold;
    &:hover { text-decoration: underline; }
  }
/*&层级很深，会先替换里面的*/
p{
    height: 200px;
    &:hover{
        background: red;
    }
    a{
        &:hover{
            text-decoration: underline;
        }
    }
}
```
编译为

```css
a {
  font-weight: bold; }
  a:hover {
    text-decoration: underline; }
/**/
p {
  height: 200px; }
  p:hover {
    background: red; }
  p a:hover {
    text-decoration: underline; }

```



###### 3.嵌套属性

```scss
.funky {
    font: {
      family: fantasy;
      size: 30em;
      weight: bold;
    }
  }
```
编译为

```css
.funky {
  font-family: fantasy;
  font-size: 30em;
  font-weight: bold; }
```



###### 4.变量

```scss
$width: 5em;
#main {
    width: $width;
}  
/*变量写在嵌套之中，就不会在全局中，可以加上 !global*/
#main {
    $width: 5em!global;
    width: $width;
}  
p{
    width: $width;
}
```

编译为

```css
#main {
  width: 5em; }
/**/
#main {
  width: 5em; }

p {
  width: 5em; }

```



###### 5.运算

```scss
$width:20px;
#main {
    width: $width/2;
    margin-left: 5px + 8px/2px;
}  
/*有点类似es6的`${}`*/
p {
    $font-size: 12px;
    $line-height: 30px;
    font: #{$font-size}/#{$line-height};
  }
/**/
p:before {
    content: "Foo " + Bar;
    font-family: sans- + "serif";
  }
/*
*   #{}内支持运算
*/
$attr: border;
p {
     width: #{5 + 10}px;
     #{$attr}-color: blue;
  }
```

编译为

```css
#main {
  width: 10px;
  margin-left: 9px; }
/**/

p {
  font: 12px/30px; }
/**/

p:before {
  content: "Foo Bar";
  font-family: sans-serif; }
/**/

p {
  width: 15px;
  border-color: blue; }

```



###### 6.@extend

```scss
.error {
  border: 1px #f00;
  background-color: #fdd;
}
.seriousError {
  @extend .error;
  border-width: 3px;
}
.criticalError {
  @extend .seriousError;
  position: fixed;
  top: 10%;
  bottom: 10%;
  left: 10%;
  right: 10%;
}
```

编译成

```css
.error, .seriousError, .criticalError {
  border: 1px #f00;
  background-color: #fdd; }

.seriousError, .criticalError {
  border-width: 3px; }

.criticalError {
  position: fixed;
  top: 10%;
  bottom: 10%;
  left: 10%;
  right: 10%; }

```



###### 7.@if

```scss
$type: monster;
p {
  @if $type == ocean {
    color: blue;
  } @else if $type == matador {
    color: red;
  } @else if $type == monster {
    color: green;
  } @else {
    color: black;
  }
}
```

编译成

```css
p {
  color: green; }
```



###### 8.@for

```scss
@for $i from 1 through 3 {
  .item-#{$i} { width: 2em * $i; }
}
/**/
@for $i from 1 to 3 {
    .item-#{$i} { width: 2em * $i; }
}
```

编译成

```css
.item-1 {
  width: 2em; }

.item-2 {
  width: 4em; }

.item-3 {
  width: 6em; }
/**/
.item-1 {
  width: 2em; }

.item-2 {
  width: 4em; }

```



###### 9.@each

```scss
@each $animal in puma, sea-slug, egret, salamander {
  .#{$animal}-icon {
    background-image: url('/images/#{$animal}.png');
  }
}

/*每一项对应*/
@each $animal, $color, $cursor in (puma, black, default),
                                  (sea-slug, blue, pointer),
                                  (egret, white, move) {
  .#{$animal}-icon {
    background-image: url('/images/#{$animal}.png');
    border: 2px solid $color;
    cursor: $cursor;
  }
}

/**/
@each $header, $size in (h1: 2em, h2: 1.5em, h3: 1.2em) {
  #{$header} {
    font-size: $size;
  }
}
```

编译成

```css
.puma-icon {
  background-image: url("/images/puma.png"); }

.sea-slug-icon {
  background-image: url("/images/sea-slug.png"); }

.egret-icon {
  background-image: url("/images/egret.png"); }

.salamander-icon {
  background-image: url("/images/salamander.png"); }
/**/

.puma-icon {
  background-image: url("/images/puma.png");
  border: 2px solid black;
  cursor: default; }

.sea-slug-icon {
  background-image: url("/images/sea-slug.png");
  border: 2px solid blue;
  cursor: pointer; }

.egret-icon {
  background-image: url("/images/egret.png");
  border: 2px solid white;
  cursor: move; }

/**/
h1 {
  font-size: 2em; }

h2 {
  font-size: 1.5em; }

h3 {
  font-size: 1.2em; }

```



###### 10.@mixin

```scss
@mixin large-text {
  font: {
    family: Arial;
    size: 20px;
    weight: bold;
  }
  color: #ff0000;
}
.page-title{
    @include large-text
}
/*mixin可以执行并且可以接受参数*/
@mixin sexy-border($color, $width) {
  border: {
    color: $color;
    width: $width;
    style: dashed;
  }
}
p { @include sexy-border(blue, 20px); }

/*@content*/
@mixin button { 
  font-size: 1em; 
  padding: 0.5em 1.0em;
  text-decoration: none; 
  color: #fff; @content;
  @content;  
} 
.button-green { 
    @include button { 
      background: green
    } 
}
```

编译成

```css
.page-title {
  font-family: Arial;
  font-size: 20px;
  font-weight: bold;
  color: #ff0000; }
/**/

p {
  border-color: blue;
  border-width: 20px;
  border-style: dashed; }
/**/

.button-green {
  font-size: 1em;
  padding: 0.5em 1.0em;
  text-decoration: none;
  color: #fff;
  background: green; }
```



###### 11.Function

```scss
$grid-width: 40px;
$gutter-width: 10px;

@function grid-width($n) {
  @return $n * $grid-width + ($n - 1) * $gutter-width;
}

#sidebar { width: grid-width(5); }
```

编译成

```css
#sidebar {
  width: 240px; }
```



##### 比较

@mixin与@extend的区别：

```scss
/*@mixin*/
@mixin button { 
  background-color: green; 
}
.button-1 {
   @include button; 
} 
.button-2 {
   @include button;
}

/*@extend
*有时候写的样式只想继承，不想编译出来的时候可以使用，只需要把button改成%button
*/
button { 
  background-color: green; 
}
.button-1 {
   @extend button; 
} 
.button-2 {
  @extend button;
}

```

编译成

```css
/*@mixin*/
.button-1 {
  background-color: green; }

.button-2 {
  background-color: green; }

/*@extend*/
button, .button-1, .button-2 {
  background-color: green; }

```

1.@mixin可以传入参数，如果@mixin里样式代码很多，编译后会生成大量的代码。

2.@extend编译不会重复花括号内的内容。