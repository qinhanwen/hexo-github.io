---
title: react入门-路由
date: 2019-03-30 22:20:33
tags: 
- react入门
categories: 
- react入门
---



## 单页应用与多页应用

##### 单页应用

![WX20190116-233425@2x](http://118.24.241.76/WX20190116-233425@2x.png)

1）一次性载入公共资源（js，css）

2）页面局部更新

3）页面间传递数据比较容易



##### 多页应用

![WX20190116-233621@2x](http://118.24.241.76/WX20190116-233621@2x.png)

1）每次都重新载入公共资源

2）页面刷新

3）页面间传递数据比较不方便（url，本地存储等）



​	没有太多的介绍区别，反正就是多页应用每次都请求一个页面的地址，加载这个页面以及有关的所有资源，服务器压力比较大，而单页应用首次加载的时候比较久，之后变换URL只是更新视图。





## react Router安装

有多个库，这里安装react-router-dom

```shell
$ npm install react-router-dom
```



## react路由

两种方式：

##### 1）HashRouter：利用hash实现路由切换

##### 2）BrowserRouter：利用h5 api实现路由切换



##### 实现一个简单的路由切换，三个分类，切换对应的视图。

![WX20190331-120145@2x](http://118.24.241.76/WX20190331-120145@2x.png)

目录结构：

![WX20190331-230234@2x](http://118.24.241.76/WX20190331-230234@2x.png)





index.js

```jsx
import React, { Component } from 'react';
import { render } from 'react-dom';
import { HashRouter as Router, Route, Link, Redirect, Switch } from './react-router-dom';
import Home from './home';
import My from './my';
import List from './list';

export default class App extends Component {
    render() {
        return (
            <Router>
                <div>
                    <Link to="/home">首页</Link>
                    <Link to="/list">列表</Link>
                    <Link to="/my">我的</Link>
                </div>
                <Switch>
                    {/* extract严格匹配 */}
                    <Route path="/home" extract={true} component={Home}></Route>
                    <Route path="/list" component={List}></Route>
                    <Route path="/my" component={My}></Route>
                    <Redirect to="/home"></Redirect>
                </Switch>
            </Router>
        )
    }
}
render(<App />, document.getElementById('root'));
```



my.js

```jsx
import React, { Component } from 'react'

export default class My extends Component {
  render() {
    return (
      <div>
        my
      </div>
    )
  }
}
```



home.js

```jsx
import React, { Component } from 'react'

export default class Home extends Component {
  render() {
    return (
      <div>
          home
      </div>
    )
  }
}
```



list.js

```jsx
import React, { Component } from 'react'

export default class List extends Component {
  render() {
    return (
      <div>
        list
      </div>
    )
  }
}
```



这时候hash值切换对应的路径，就会看到不同的视图。



##### 同时简单实现一下的react-router-dom

context.js

```jsx
import React, { Component } from 'react'

let { Provider, Consumer } = React.createContext();

export { Provider, Consumer };
```



HashRouter.js

```jsx
import React, { Component } from 'react'
import { Provider } from './context';
export default class HashRouter extends Component {
    constructor() {
        super();
        this.state = {
            location: {
                pathName: window.location.hash.slice(1) || '/'
            }
        }
    }
    componentDidMount() {
        window.location.hash = window.location.hash.slice(1) || '/';
        window.addEventListener('hashchange', () => {
            this.setState({
                location: {
                    pathName: window.location.hash.slice(1) || '/'
                }
            })
        })
    }
    render() {
        let value = {
            location:this.state.location,
            history:{
                push(to){
                    window.location.hash = to;
                }
            }
        };
        return (
            <Provider value={value}>
                {this.props.children}
            </Provider>
        )
    }
}

```



Link.js

```jsx
import React, { Component } from 'react'
import { Consumer } from './context';

export default class Link extends Component {
    
  render() {
    return (
        <Consumer>
            {state => {
                return <a onClick={()=>{
                    state.history.push(this.props.to);
                }}>{this.props.children}</a>
            }}
        </Consumer>
    )
  }
}

```



Redirect.js

```jsx
import React, { Component } from 'react'
import { Consumer } from './context'

export default class Redirect extends Component {
    render() {
        return (
            <Consumer>
                {state => {
                    state.history.push(this.props.to);
                    return null;
                }}
            </Consumer>
        )
    }
}
```



Route.js

```jsx
import React, { Component } from 'react';
import { Consumer } from './context';
import pathToReg from 'path-to-regexp';

export default class Route extends Component {
    render() {
        return (
            <Consumer>
                {state => {
                    let { path, component: Component, extract = false } = this.props;
                    let pathName = state.location.pathName;
                    if (pathToReg(path, [], { end: extract }).test(pathName)) {
                        return <Component></Component>;
                    }

                    return null;
                }}
            </Consumer>
        )
    }
}

```



Switch.js

```jsx
import React, { Component } from 'react';
import { Consumer } from './context';
import pathToReg from 'path-to-regexp';

export default class Switch extends Component {
    render() {
        return (
            <Consumer>
                {state => {
                    let { pathName } = state.location;
                    let children = this.props.children;
                    for(let i=0;i<children.length;i++){
                        let child = children[i];
                        // Redirect可能没有path
                        let path = child.props.path || '';
                        if(pathToReg(pathName,[],{end:false}).test(path)){
                            return child;
                        }
                    }
                    return null;
                }}
            </Consumer>
        )
    }
}

```



index.js

```jsx
import HashRouter from './HashRouter';
import Route from './Route';
import Link from './Link';
import Redirect from './Redirect';
import Switch from './Switch';

export { HashRouter, Route, Link, Redirect, Switch };
```

导入各个模块，再导出。



**当然缺失了很多东西，只是简单实现。**





## 路由跳转传参

写个简单例子吧

```jsx
import React, { Component } from 'react'
import { Link, Route } from 'react-router-dom';
import userInfo from './userInfo'

export default class List extends Component {
  render() {
    return (
      <div>
        <Link to='/list/userInfo/1'>用户详情</Link>
        <Route path="/list/userInfo/:id" component={userInfo}></Route>
      </div>
    )
  }
}
```

点击Link的时候，会携带id



或者使用` this.props.history.push('/list/userInfo/1');`

```jsx
import React, { Component } from 'react'
import { Link, Route } from 'react-router-dom';
import userInfo from './userInfo'

export default class List extends Component {
  constructor(){
    super();
    this.gotoUserInfo = this.gotoUserInfo.bind(this);
  }

  gotoUserInfo(){
    this.props.history.push('/list/userInfo/1');
  }

  render() {
    return (
      <div>
        {/* <Link to='/list/userInfo/1'>用户详情</Link> */}
        <div onClick={this.gotoUserInfo}>用户详情</div>
        <Route path="/list/userInfo/:id" component={userInfo}></Route>
      </div>
    )
  }
}
```



userInfo.js

```jsx
import React, { Component } from 'react'

export default class userInfo extends Component {
  render() {
      console.log(this.props);
    return (
      <div>
        用户id:{this.props.match.params.id}
      </div>
    )
  }
}
```

