---
title: react入门-高阶组件
date: 2019-03-30 15:33:57
tags: 
- react入门
categories: 
- react入门
---

## 高阶组件

接受个函数，并且返回也是个函数



举个🌰，需要从localStorage取得数据，在页面中展示

```jsx
import React, { Component } from 'react';

export default class Example extends Component {

    componentWillMount() {
        let name = localStorage.getItem('name');
        this.setState({ name });
    }

    render() {
        return (
            <div>{this.state.name}</div>
        )
    }
}
```

当其他组件也需要从localStorage里取数据，重复添加componentWillMount里的代码就显得很繁琐。所以修改一下：

```jsx
import React, { Component } from 'react';
import WarpComponent from './component';

class Example extends Component {
    render() {
        return (
            <div>{this.props.name}</div>
        )
    }
}
const NewExample = WarpComponent(Example);
export default NewExample;
```

调用高阶组件返回了新的组件

```jsx
import React, { Component } from 'react';

export default function WarpComponent(InnerComponent){
    return class extends Component{
        componentWillMount() {
            let name = localStorage.getItem('name');
            this.setState({ name });
        }
        render(){
            return (
                <InnerComponent name={this.state.name} />
            )
        }
    }
}
```

1）高阶组件可以对数据进行处理，再传给被包装的组件

2）再将props传入被包装组件



## 使用场景

##### 1）对传入数据props拦截，进行修改、增加、删除操作

可以看上面那个例子



##### 2）通过ref，可以在高阶组件中获取组件的实例，就可以使用实例的方法和属性

在类组件中定义了个changeName方法，用于修改state中的name

```jsx
import React, { Component } from 'react';
import WrapComponent from './wrapComponent';

class Example extends Component {
    constructor(props){
        super(props);
        this.state = {
            name:'qinhanwen'
        }
        this.changeName = this.changeName.bind(this);
    }

    changeName(){
        this.setState({name:'qinhanwen2'})
    }
    
    render() {
        return (
            <div>{this.state.name}</div>
        )
    }
}
const NewExample = WrapComponent(Example);
export default NewExample;
```



在高阶组件中可以拿到传入的组件的实例，并且可以调用到实例上的方法

```jsx
import React, { Component } from 'react';

export default function WrapComponent(InnerComponent){
    return class extends Component{
        componentDidMount(){
            this.InnerComponent.changeName();
        }
        render(){
            return (
                <InnerComponent ref={InnerComponent => this.InnerComponent = InnerComponent}
                />
            )
        }
    }
}
```





##### 3）组件状态提升

无状态的组件更容易被复用，将状态以及业务逻辑提升到高阶组件中

```jsx
import React, { Component } from 'react';
import WrapComponent from './wrapComponent';

class Example extends Component {
    render() {
        console.log(this.props);
        return (
            <input {...this.props.statusAndCB} />
        )
    }
}
const NewExample = WrapComponent(Example);
export default NewExample;
```

这个组件只有一个input标签，以及输入属性，这是一个无状态的组件



```jsx
import React, { Component } from 'react';

export default function WrapComponent(InnerComponent){
    return class extends Component{
        constructor(props){
            super(props);
            this.state = {
                name:''
            }
            this.handleChange = this.handleChange.bind(this);
        }

        handleChange(){
            this.setState({
                name:'qinhanwen'
            });
        }

        render(){
            const newProps ={
                statusAndCB:{
                    value:this.state.name,
                    onChange:this.handleChange
                }
            }
            return (
                <InnerComponent {...newProps}
                />
            )
        }
    }
}
```

这个高阶组件中，将value的值，以及onChange的回调传入了InnerComponent的props中，保证了无状态组件中没有复杂的业务逻辑和状态改变。





##### 4）其他元素包裹组件

```jsx
import React, { Component } from 'react';
import WrapComponent from './wrapComponent';

class Example extends Component {
    render() {
        return (
            <div>color</div>
        )
    }
}
const NewExample = WrapComponent(Example);
export default NewExample;
```



这个高阶函数调加了一个div包裹了组件，并且设置了新的样式

```jsx
import React, { Component } from 'react';

export default function WrapComponent(InnerComponent) {
    return class extends Component {
        render() {
            return (
                <div style={{color:"red"}}>
                    <InnerComponent />
                </div>
            )
        }
    }
}
```





## 参数传递

比如说有个父组件列表，获取的是用户信息的数据。

```jsx
import React, { Component } from 'react';
import NewItem from './item';
class List extends Component {
    constructor(props) {
        super(props);
        this.state = {
            user: [],
        }
    }

    componentDidMount() {
        // 请求数据成功
        this.setState({
            user: [{ name: 'qinhanwen', age: 25, desc: '简介1' }, { name: 'zh', age: 26, desc: '简介2' }],
        })
    }

    render() {
        return (
            <div>
                <NewItem user={this.state.user} />
            </div>
        );
    }
}

export default List;
```



数据传递给子组件

```jsx
import React, { Component } from 'react';
import  WrapComponent from './wrapComponent';

class Item extends Component {
    render() {
        return (
            <div>
                <ul>
                    {
                        this.props.user.map((item, index) => <li key={index}>name:{item.name} age: {item.age}</li>)
                    }
                </ul>
            </div>
        );
    }
}

const NewItem = WrapComponent(Item,'red');
export default NewItem;
```



再增加一个高阶组件，这里添加了一个外层标签，简单的设置了一下颜色的值，由调用的地方传参传入。这里需要注意的就是要将this.props再传入组件。

```jsx
import React, { Component } from 'react';

export default function WrapComponent(InnerComponent,key) {
    return class extends Component {
        render() {
            return (
                <div style={{color:key}}>
                    <InnerComponent  {...this.props} />
                </div>
            )
        }
    }
}
```













