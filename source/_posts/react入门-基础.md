---
title: react入门-基础
date: 2019-02-02 20:38:58
tags: 
- react入门
categories: 
- react入门
---

> 资料	<https://github.com/semlinker/reactjs-interview-questions>

[TOC]

使用node版本10.11.0

## 1.开始

#### 安装

```shell
$ npm install -g create-react-app
```



#### 创建项目

```shell
$ create-react-app app
```



#### 运行项目

```shell
$ cd app
$ npm start
```



## 2.JSX

JSX是一种用于描述UI的JavaScript拓展语法。



#### 标签类型

DOM类型标签

```jsx
const element = <p>Hello World</p>;
```



React组件类型标签

```jsx
const element = <HelloWorld />;
```



嵌套使用

```jsx
const element = (
  <p>
  	<HelloWorld />
  </p>
);
```



#### JavaScript表达式

在JSX中使用JavaScript表达式需要使用{}包裹

```jsx
const list = ['qinhanwen',25];
const element = (
	<ul>
  		{list.map(item => <Item item={item} />)}
  </ul>
)
```

 只能使用JavaScript表达式，不能使用多行JavaScript语句。



#### 标签属性

class改写为className

事件属性改为驼峰命名onClick

JSX标签是React组件类型的时候，可以自定义标签的属性



#### 注释

```jsx
const element = (
<div>
  {/* 注释 */}
</div>
)
```





## 3.组件



#### 组件定义

方式一：使用class定义组件

1）class继承React.Component

2）内部实现render方法

postList.js

```jsx
import React,{Component} from 'react';

class PostList extends Component{
    render(){
        return (
            <ul>
                <li>列表1</li>
                <li>列表2</li>
            </ul>
        );
    }
}
export default PostList;
```

App.js

```jsx
import React, { Component } from 'react';
import logo from './logo.svg';
import './App.css';
import './postList'
import PostList from './postList';

class App extends Component {
  render() {
    return (
      <PostList />
    );
  }
}

export default App;
```

![WX20190324-153910@2x](http://114.55.30.96/WX20190324-153910@2x.png)

方式二：使用ReactDom.render()



#### props

用于把父组件的值传递给子组件，供子组件使用

postList.js

```jsx
import React, { Component } from 'react';
import PostListItem from './postListItem';
const data = [{ name: 'qinhanwen', age: 25 }, { name: 'zh', age: 26 }];
class PostList extends Component {
    render() {
        return (
            <ul>
                {data.map(item => <PostListItem name={item.name} age={item.age} key={item.age} />)}
            </ul>
        );
    }
}
export default PostList;
```

postListItem.js

```jsx
import React,{Component} from 'react';

class PostListItem extends Component{
    render(){
        const {name,age} = this.props;
        return (
            <li>
                name:{name} age: {age}
            </li>
        );
    }
}
export default PostListItem;
```

![WX20190324-174124@2x](http://114.55.30.96/WX20190324-174124@2x.png)

#### state

组件的内部状态



实现个点击累加

postListItem.js

```jsx
import React, { Component } from 'react';

class PostListItem extends Component {
    constructor(props) {
        super(props);
        this.state = {
            vote: 0
        }
    }

    handleClick() {
        let vote = this.state.vote;
        vote++;
        this.setState({
            vote
        });
    }

    render() {
        const { name, age } = this.props;
        return (
            <li onClick={() => this.handleClick()}>
                name:{name} age: {age}
                vote:{this.state.vote}
            </li>
        );
    }
}
export default PostListItem;
```



#### 组件样式

##### 外部css样式表

方式一：通过模块引入

```jsx
import React, { Component } from 'react';
import './postListItem.css';
class PostListItem extends Component {
    constructor(props) {
        super(props);
        this.state = {
            vote: 0
        }
    }

    handleClick() {
        let vote = this.state.vote;
        vote++;
        this.setState({
            vote
        });
    }

    render() {
        const { name, age } = this.props;
        return (
            <li className="item" onClick={() => this.handleClick()}>
                name:{name} age: {age}
                vote:{this.state.vote}
            </li>
        );
    }
}
export default PostListItem;
```

使用className定义类名，并且引入css文件。



postListItem.css

```css
.item{
    height: 80px;
    line-height: 80px;
    font-size: 20px;
    color:red;
}
```

![WX20190325-234559@2x](http://114.55.30.96/WX20190325-234559@2x.png)

方式二：在使用组件的HTML页面通过link标签引入

```html
<link rel="stylesheet" type="text/css" href="./postListItem.css"  />
```





##### 内联样式

用JS对象描述CSS，属性名使用驼峰形式

```jsx
import React, { Component } from 'react';

class PostListItem extends Component {
    constructor(props) {
        super(props);
        this.state = {
            vote: 0
        }
    }

    handleClick() {
        let vote = this.state.vote;
        vote++;
        this.setState({
            vote
        });
    }

    render() {
        const { name, age } = this.props;
        return (
            <li className="item" style={{color:"red",backgroundColor:"yellow"}} onClick={() => this.handleClick()}>
                name:{name} age: {age}
                vote:{this.state.vote}
            </li>
        );
    }
}
export default PostListItem;
```

![WX20190325-235727@2x](http://114.55.30.96/WX20190325-235727@2x.png)





## 4.组件生命周期

### 挂载阶段

##### 从创建，初始化，挂载到DOM中，完成组件第一次渲染，分别经历了

##### 1）constructor

​	class的构造方法，组件被创建的时候首先调用到它，它接受一个参数props(从父组件传入的值)，需要在组件内调用super(props)才会将值传入子组件。

​	

##### 2）componentWillMount

​	组件被挂载到DOM前只会调用一次，在这个方法内调用this.setState不会引起组件重新渲染。



##### 3）render

​	这个方法是必要的，根据组件的props和state返回一个React元素，用于描述UI组件(render不负责渲染，只是返回一个UI描述)。



##### 4）componentDidMount

​	组件挂载到DOM的时候调用，这时候可以做DOM操作，向服务器请求通常也放在这个方法里，这个方法里调用this.setState会引起组件的重新渲染。





### 更新阶段

##### 组件挂载到DOM后，props或者state会引起组件更新，组件更新阶段经历了

##### 1）componentWillReceiveProps(nextProps)

​	这个方法只是props变化引起的组件更新才会被调用，state引起的组件更新不会触发这个方法。`nextProps`是父组件传递给当前组件的新的props。



##### 2）shouldComponentUpdate(nextProps,nextState)

​	这个方法决定组件是否继续更新，方法为true的时候，组件继续更新。方法返回为false的时候，后续的componentWillUpdate，render，componentDidUpdate，不再调用。



##### 3）componentWillUpdate(nextProps,nextState)

​	在render调用前执行。



##### 4）componentDidUpdate(prevProps,prevState)

​	组件更新后调用，两个参数是更新前的值。





### 卸载阶段

##### 组件从DOM中被卸载的过程，只有一个方法

##### componentWillUnmount

在这个方法中，需要做的事情是清除一些定时器，`componentDidMount`中创建的DOM元素，避免内存泄漏。





## 5.列表和keys

```jsx
import React, { Component } from 'react';
import PostListItem from './postListItem';
const data = [{ name: 'qinhanwen', age: 25 }, { name: 'zh', age: 26 }];
class PostList extends Component {
    render() {
        return (
            <ul>
                {data.map(item => <PostListItem name={item.name} age={item.age}  />)}
            </ul>
        );
    }
}
export default PostList;
```

列表数据渲染，如果没有添加key，就会有警告提示。

![WX20190326-224147@2x](http://114.55.30.96/WX20190326-224147@2x.png)

当列表数据变化的时候，React通过key知道哪些元素变化，并且只重新渲染变化的元素。



可以使用单独标识，比如索引值作为key

```jsx
import React, { Component } from 'react';
import PostListItem from './postListItem';
const data = [{ name: 'qinhanwen', age: 25 }, { name: 'zh', age: 26 }];
class PostList extends Component {
    render() {
        return (
            <ul>
                {data.map((item,index) => <PostListItem name={item.name} age={item.age} key={index} />)}
            </ul>
        );
    }
}
export default PostList;
```





## 6.事件处理

##### 事件驼峰命名：onClick、onChange

##### 

##### 1）箭头函数

业务处理逻辑不直接放在onClick的括号中，而是抽出，通过箭头函数调用另外一个定义的方法

```jsx
import React, { Component } from 'react';

class PostListItem extends Component {
    constructor(props) {
        super(props);
        this.state = {
            vote: 0
        }
    }

    handleClick(event) {
        console.log(event);
        let vote = this.state.vote;
        vote++;
        this.setState({
            vote
        });
    }

    render() {
        const { name, age } = this.props;
        return (
            <li className="item" style={{ color: "red", backgroundColor: "yellow" }} onClick={(event) => this.handleClick(event)}>
                 name:{name} age: {age}
                vote:{this.state.vote}
            </li>
        );
    }
}
export default PostListItem;
```

在render方法中定义事件处理函数，每一次render调用的时候都会创建新的事件处理函数，是不大合理的。



##### 2）组件方法

将组件方法赋值给组件属性。

```jsx
import React, { Component } from 'react';

class PostListItem extends Component {
    constructor(props) {
        super(props);
        this.state = {
            vote: 0
        }
        this.handleClick = this.handleClick.bind(this);
    }

    handleClick(event) {
        console.log(event);
        let vote = this.state.vote;
        vote++;
        this.setState({
            vote
        });
    }

    render() {
        const { name, age } = this.props;
        return (
            <li className="item" style={{ color: "red", backgroundColor: "yellow" }} onClick={this.handleClick}>
                 name:{name} age: {age}
                vote:{this.state.vote}
            </li>
        );
    }
}
export default PostListItem;
```

每次render的时候不会重新创建一个回调函数。

另一种方式：

```jsx
import React, { Component } from 'react';

class PostListItem extends Component {
    constructor(props) {
        super(props);
        this.state = {
            vote: 0
        }
    }

    handleClick(name,event) {
        console.log(name);
        console.log(event);
        let vote = this.state.vote;
        vote++;
        this.setState({
            vote
        });
    }

    render() {
        const { name, age } = this.props;
        return (
            <li className="item" style={{ color: "red", backgroundColor: "yellow" }} onClick={this.handleClick.bind(this,name)}>
                 name:{name} age: {age}
                vote:{this.state.vote}
            </li>
        );
    }
}
export default PostListItem;
```

这种每次render就会创建一个新函数。



##### 3）属性初始化语法

略。





## 7.表单

##### 受控组件：表单的元素的值由React管理，需要为每个表单元素定义onchange事件，然后把表单状态更新到React组件的state

```jsx
import React, { Component } from 'react';

class LoginForm extends Component {
    constructor(props) {
        super(props);
        this.state = {
            userName: '',
            passWord: ''
        }
    }

    handleSubmit(event) {
        event.preventDefault();
        console.log(this.state);
    }

    handleChange(event) {
        this.setState({
            [event.target.name]: event.target.value
        })
    }

    render() {
        return (
            <form onSubmit={this.handleSubmit.bind(this)}>
                <label>账号
                    <input name="userName" onChange={this.handleChange.bind(this)}
                     value={this.state.userName} />
                </label>
                <label>密码
                    <input name="passWord" onChange={this.handleChange.bind(this)}
                     value={this.state.passWord} />
                </label>
                <input type="submit" value="登录" />
            </form>
        );
    }
}
export default LoginForm;
```

使用同一个handleChange函数对数据变化进行修改。



##### 非受控组件：有一种自己的方式获得表单元素的值(书上说不建议使用，哈哈哈哈哈)

```jsx
import React, { Component } from 'react';

class LoginForm extends Component {
    handleSubmit(event) {
        event.preventDefault();
        console.log(this.userNameInput.value);
        console.log(this.passWordInput.value);
    }

    render() {
        return (
            <form onSubmit={this.handleSubmit.bind(this)}>
                <label>账号
                    <input name="userName" ref={userNameInput => this.userNameInput = userNameInput}
                    />
                </label>
                <label>密码
                    <input name="passWord" ref={passWordInput => this.passWordInput = passWordInput}
                    />
                </label>
                <input type="submit" value="登录" />
            </form>
        );
    }
}
export default LoginForm;
```

