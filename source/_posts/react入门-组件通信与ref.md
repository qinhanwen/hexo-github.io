---
title: react入门-组件通信与ref
date: 2019-03-27 13:37:45
tags: 
- react入门
categories: 
- react入门
---

> 资料	<https://github.com/hujiulong/blog/issues/7>

[TOC]

## 组件的state

#### 设置合适的state

##### state里面适用于放置一些与页面渲染UI有关的状态集，其他的属性可以直接绑定在this上。

```jsx
import React, { Component } from 'react';
class Timer extends Component {
    constructor(props) {
        super(props);
        this.state = {
            date: new Date()//渲染属性
        }
        this.timer = null;//普通属性
        this.changeTime = this.changeTime.bind(this);
    }
    
    changeTime() {
        this.setState({ date: new Date() });
    }

    componentDidMount() {
        this.timer = setInterval(this.changeTime, 1000);
    }

    componentWillUnmount() {
        this.timer = null;
    }

    render() {
        return (
            <h3>当前时间：{this.state.date.toString()}</h3>
        );
    }
}
export default Timer;
```

![timer](http://114.55.30.96/timer.gif)

timer作为普通属性，直接挂载在this上。



#### state更新

一个合并的过程，只需要传入发生改变的state，它会与之前组建中的其他状态合并。



#### state更新是异步的

[后面再研究吧。](<https://github.com/hujiulong/blog/issues/7>)



#### 修改state的正确方式

使用`this.state.date = new Date();`这种方式修改状态，不会触发render，页面上的状态不会被更新。

```jsx
import React, { Component } from 'react';
class Timer extends Component {
    constructor(props) {
        super(props);
        this.state = {
            date: new Date()
        }
        this.timer = null;
        this.changeTime = this.changeTime.bind(this);
    }
    
    changeTime() {
        this.state.date = new Date();
    }

    componentDidMount() {
        this.timer = setInterval(this.changeTime, 1000);
    }

    componentWillUnmount() {
        this.timer = null;
    }

    render() {
        return (
            <h3>当前时间：{this.state.date.toString()}</h3>
        );
    }
}
export default Timer;
```

所以还是要使用`this.setState({ date: new Date() });`。



当state的某个状态发生改变的时候，应该要重新创建这个状态，而不是直接修改原来的状态。

```jsx
import React, { Component } from 'react';
class Book extends Component {
    constructor(props) {
        super(props);
        this.state = {
            number: 123456,
            title: '标题',
            success: true,
            books: ['JavaScript', 'Node'],
            person: {
                name: 'qinhanwen',
            }
        }
        this.timer = null;
        this.changeState = this.changeState.bind(this);
    }

    changeState() {
        this.setState({
            number: 23456,
            title: '新标题',
            success: false
        })

        this.setState((preState) => ({
            books: [...preState.books, 'JAVA']
        }))

        this.setState((preState) => ({
            person: Object.assign(preState.person, { age: 25 })
        }))
    }

    render() {
        return (
            <div>
                <h3>{this.state.title}</h3>
                <h3>{this.state.number}</h3>
                <h3>{this.state.success}</h3>

                <ul>
                    {this.state.books.map((item,index)=><li key={index}>{item}</li>)}
                </ul>

                <p>{this.state.person.name}:{this.state.person.age}</p>
                <button onClick={this.changeState}>修改</button>
            </div>
        );
    }
}
export default Book;
```

对不同的数据类型，使用不同的方式重新创建状态对象。







## 组件通信

##### 父组件向子组件通信

###### 父组件

```jsx
import React, { Component } from 'react';
import PostItem from './postItem';

class PostList extends Component {
    constructor(props) {
        super(props);
        this.state = {
            user: []
        }
    }
    componentDidMount() {
        // 请求数据成功
        this.setState({
            user: [{ name: 'qinhanwen', age: 25 }, { name: 'zh', age: 26 }]
        })
    }
    render() {
        return (
            <PostItem user={this.state.user} />
        );
    }
}
export default PostList;
```



###### 子组件

```jsx
import React, { Component } from 'react';

class PostItem extends Component {
    constructor(props) {
        super(props);
        this.state = {
        }
    }

    render() {
        return (
            <ul>
                {
                    this.props.user.map((item, index) => <li key={index}>name:{item.name} age: {item.age}</li>)
                }
            </ul>

        );
    }
}
export default PostItem;
```



###### 父组件向子组件通信方式：通过props传递数据。



##### 子组件向父组件通信

###### 父组件

```jsx
import React, { Component } from 'react';
import PostItem from './postItem';

class PostList extends Component {
    constructor(props) {
        super(props);
        this.state = {
            user: []
        }
        this.commit = this.commit.bind(this);
    }
    componentDidMount() {
        // 请求数据成功
        this.setState({
            user: [{ name: 'qinhanwen', age: 25 }, { name: 'zh', age: 26 }]
        })
    }

    commit(info){
        console.log(info);
        //上传数据
    }

    render() {
        return (
            <PostItem user={this.state.user} cb={this.commit} />
        );
    }
}
export default PostList;
```

父组件通过props传递进回调函数



###### 子组件

```jsx
import React, { Component } from 'react';

class PostItem extends Component {
    constructor(props) {
        super(props);
        this.state = {
            userInfo: ''
        }
        this.handleChange = this.handleChange.bind(this);
        this.update = this.update.bind(this);
    }

    handleChange(event) {
        this.setState({
            userInfo:event.target.value
        })
    }

    update(){
        this.props.cb(this.state.userInfo);
    }

    render() {
        return (
            <div>
                <ul>
                    {
                        this.props.user.map((item, index) => <li key={index}>name:{item.name} age: {item.age}</li>)
                    }
                </ul>
                <input name='userInfo' value={this.state.userInfo} onChange={this.handleChange} />
                <button onClick={this.update}>通知父组件</button>
            </div>
        );
    }
}
export default PostItem;
```

子组件更新状态调用父组件传递进的cb方法



###### 子组件向父组件通信方式：父组件通过props传入回调函数，子组件调用回调函数通知父组件。



##### 兄弟组件通信

B和C是兄弟组件，它们拥有共同的父组件

![WX20190327-202433@2x](http://114.55.30.96/WX20190327-202433@2x.png)

##### 方式一：props

###### 父组件PostList

```jsx
import React, { Component } from 'react';
import PostItem from './postItem';
import Detail from './detail'

class PostList extends Component {
    constructor(props) {
        super(props);
        this.state = {
            user: [],
            index: 0
        }
        this.choose = this.choose.bind(this);
    }
  
    componentDidMount() {
        // 请求数据成功
        this.setState({
            user: [{ name: 'qinhanwen', age: 25, desc: '简介1' }, { name: 'zh', age: 26, desc: '简介2' }],
        })
    }

    choose(index) {
        this.setState({ index });
    }

    render() {
        const desc = this.state.user && this.state.user[this.state.index] && this.state.user[this.state.index].desc;
        return (
            <div>
                <PostItem user={this.state.user} index={this.state.index} choose={this.choose} />
                <Detail desc={desc} />
            </div>
        );
    }
}
export default PostList;
```

1）请求数据之后改变状态集中user的值

2）新增的状态集中的index用来标识选中列表哪一项的下标

3）新增choose方法，传入子组件，用于更新选中列表的下标

4）传入Detail组件desc数据



###### 子组件PostItem

```jsx
import React, { Component } from 'react';
import './postItem.css'

class PostItem extends Component {
    constructor(props) {
        super(props);
        this.state = {
        }
    }

    choose(index) {
        this.props.choose(index);
    }

    render() {
        return (
            <div>
                <ul>
                    {
                        this.props.user.map((item, index) => <li key={index} onClick={this.choose.bind(this, index)}
                            className={this.props.index === index ? 'choosed' : null}>name:{item.name} age: {item.age}</li>)
                    }
                </ul>
            </div>
        );
    }
}
export default PostItem;
```

1）choose方法，调用父组件传入的choose方法，更新父组件状态集中的index的值

2）设置选中状态的类名

3）引入postItem.css模块



###### 子组件Detail

```jsx
import React from 'react';

function Detail(props) {
    return (
        <div>
            {props.desc}
        </div>
    )
}

export default Detail;
```

1）它不用维护自身的状态，所以使用函数组件



###### postItem.css

```css
.choosed{
    border:1px solid red;
}
```





##### 方式二：Context

postList.js

```jsx
import React, { Component } from 'react';
import PostItem from './postItem';
import Detail from './detail'
import PropTypes from 'prop-types';

class PostList extends Component {
    constructor(props) {
        super(props);
        this.state = {
            user: [],
            index: 0
        }
        this.choose = this.choose.bind(this);
    }

    getChildContext() {
        return {
            choose: this.choose
        }
    }
    
    static childContextTypes = {
        choose: PropTypes.func
    }

    componentDidMount() {
        // 请求数据成功
        this.setState({
            user: [{ name: 'qinhanwen', age: 25, desc: '简介1' }, { name: 'zh', age: 26, desc: '简介2' }],
        })
    }

    choose(index) {
        this.setState({ index });
    }

    render() {
        const desc = this.state.user && this.state.user[this.state.index] && this.state.user[this.state.index].desc;
        return (
            <div>
                <PostItem user={this.state.user} index={this.state.index} />
                <Detail desc={desc} />
            </div>
        );
    }
}

export default PostList;
```

1）新增了一个静态childContextTypes对象属性

2）返回Context对象，约定好方法名



postItem.js

```jsx
import React, { Component } from 'react';
import './postItem.css'
import PropTypes from 'prop-types';

class PostItem extends Component {
    constructor(props) {
        super(props);
        this.state = {
        }
    }

    static contextTypes = {
        choose: PropTypes.func
    }

    choose(index) {
        this.context.choose(index);
    }

    render() {
        return (
            <div>
                <ul>
                    {
                        this.props.user.map((item, index) => <li key={index} onClick={this.choose.bind(this, index)}
                            className={this.props.index === index ? 'choosed' : null}>name:{item.name} age: {item.age}</li>)
                    }
                </ul>
            </div>
        );
    }
}

export default PostItem;
```

1）新增一个静态contextTypes属性

2）访问父组件的Context对象的属性



###### 兄弟组件之间通信方式：子组件通过父组件传入的props里的回调通知父组件更新状态集，会调用render方法，重新传入另一个子组件新的props。





## ref

可以用来获取DOM元素，或者组件的实例



##### 在DOM上使用ref

ref接收一个回调函数作为值，在调用这个回调函数的时候，这个DOM元素会被当作回调函数的参数。

```jsx
import React,{Component} from 'react';

export default class InputAutoFocus extends Component{
    componentDidMount(){
        this.input.focus();
    }
    render(){
        return (
            <div>
                <input ref={input => this.input = input} />
            </div>
        )
    }
}
```

选择在`componentDidMount`的时候调用focus方法，因为这个生命周期的时候，组件已经挂载到DOM上了。可以进行DOM操作。



##### 在组件上使用ref

**1）类组件**

回调函数接受的参数是当前组件的实例

###### 父组件

```jsx
import React,{Component} from 'react';
import InputAutoFocus from './ref-component';
export default class InputAutoFocusParent extends Component{
    componentDidMount(){
        this.inputComponent.focus();
    }
    render(){
        return (
            <InputAutoFocus ref={inputComponent => this.inputComponent = inputComponent} />
        )
    }
}
```

this.inputComponent就是子组件的实例，所以能调用到子组件的原型方法



###### 子组件

```jsx
import React,{Component} from 'react';

export default class InputAutoFocus extends Component{
    focus(){
        this.input.focus();
    }
    render(){
        return (
            <div>
                <input ref={input => this.input = input} />
            </div>
        )
    }
}
```

子组件声明了一个focus方法，让父组件通过子组件实例去调用



**2）函数组件**

**函数组件定义ref是不生效的！！！函数组件定义ref是不生效的！！！函数组件定义ref是不生效的！！！**

示例：

父组件

```jsx
import React,{Component} from 'react';
import InputAutoFocus from './ref-component';
export default class InputAutoFocusParent extends Component{
    render(){
        return (
            <InputAutoFocus ref={inputComponent => this.inputComponent = inputComponent} />
        )
    }
}


```

子组件

```jsx
import React, { Component } from 'react';

function InputAutoFocus(props) {
    return (
        <input />
    )
}

export default InputAutoFocus
```

![WX20190329-235653@2x](http://114.55.30.96/WX20190329-235653@2x.png)





**顺便说个小问题**

当我注释了引入React的时候，报错了，为什么呢。

![WX20190329-235959@2x](http://114.55.30.96/WX20190329-235959@2x.png)



通过Babel编译可以看到`React.createElement`，JSX的语法会编译成这个，最后转成一个reactElement。所以注释了引入React就报错了。

![WX20190330-000218@2x](http://114.55.30.96/WX20190330-000218@2x.png)





##### 父组件访问子组件的DOM节点

###### 父组件

```jsx
import React,{Component} from 'react';
import Child from './child';

export default class Parent extends Component{
    getInput(inputElment){
        this.inputElment = inputElment;
        console.log(this.inputElment);
    }
    render(){
        return (
            <Child getInput={(inputElment) => this.getInput(inputElment)} />
        )
    }
}
```



###### 子组件

```jsx
import React,{Component} from 'react';

export default class Child extends Component{
    constructor(props){
        super(props);
        this.state = {

        }
    }
    render(){
        return (
            <input ref={this.props.getInput} />
        )
    }
}
```



























#### 

























