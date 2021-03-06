---
title: react源码过程（5）- 生命周期
date: 2019-12-31 12:37:49
tags:
  - react源码过程
categories:
  - react源码过程
---

## 生命周期

组件的生命周期可分成三个状态：

- Mounting：已插入真实 DOM
- Updating：正在被重新渲染
- Unmounting：已移出真实 DOM

生命周期的方法有：

- **componentWillMount** 在渲染前调用,在客户端也在服务端。
- **componentDidMount** : 在第一次渲染后调用，只在客户端。之后组件已经生成了对应的DOM结构，可以通过this.getDOMNode()来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作(防止异步操作阻塞UI)。
- **componentWillReceiveProps** 在组件接收到一个新的 prop (更新后)时被调用。这个方法在初始化render时不会被调用。
- **shouldComponentUpdate** 返回一个布尔值。在组件接收到新的props或者state时被调用。在初始化时或者使用forceUpdate时不被调用。
  可以在你确认不需要更新组件时使用。
- **componentWillUpdate**在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。
- **componentDidUpdate** 在组件完成更新后立即调用。在初始化时不会被调用。
- **componentWillUnmount**在组件从 DOM 中移除之前立刻被调用。



## 例子

```jsx
<html>

<body>
  <script src="../../../build/dist/react.development.js"></script>
  <script src="../../../build/dist/react-dom.development.js"></script>
  <script src="https://unpkg.com/babel-standalone@6/babel.js"></script>
  <div id="container"></div>
  <script type="text/babel">
    class App extends React.Component {
      constructor(props) {
        super(props)
      }
      state = {
        count: 1
      };

      componentWillMount() {
        
        console.log('Component WILL MOUNT!')
      }
      componentDidMount() {
        
        console.log('Component DID MOUNT!')
      }
      componentWillUpdate(nextProps, nextState) {
        
        console.log('Component WILL UPDATE!');
      }
      componentDidUpdate(prevProps, prevState) {
        
        console.log('Component DID UPDATE!')
      }

      imcrement = () => {
        this.setState(pre => {
          return { count: pre.count + 1 }
        });
      }
      render() {
        const { count } = this.state;
        return (
          <div onClick={this.imcrement}>
            {this.props.name}{count}
          </div>
        )
      }
    }
    ReactDOM.render(
      <App name="count:" />,
      document.getElementById('container')
    );
  </script>
</body>

</html>
```

只加了 4 个钩子函数，看下在什么时机点被调用的

- componentWillMount

![WX20200102-153304](http://114.55.30.96/WX20200102-153304.png)



- componentDidMount

![WX20200102-154037](http://114.55.30.96/WX20200102-154037.png)



- componentWillUpdate

![WX20200102-154253](http://114.55.30.96/WX20200102-154253.png)



- componentDidUpdate

![WX20200102-154332](http://114.55.30.96/WX20200102-154332.png)



都是从 组件实例上调用 原型方法，有两个钩子会判断类型是否是 function 在调用。

**componentDidUpdate** ，这个钩子有点不一样，在触发点击事件的时候，nextFlushedRoot 的值 是在 findHighestPriorityRoot 方法中得到的

```javascript
function performWork(minExpirationTime, isYieldy) {
 
  ...
  
  findHighestPriorityRoot();

  ...
  
  else {
    while (nextFlushedRoot !== null && nextFlushedExpirationTime !== NoWork && minExpirationTime <= nextFlushedExpirationTime) {
      performWorkOnRoot(nextFlushedRoot, nextFlushedExpirationTime, false);
      findHighestPriorityRoot();
    }
  }
}


// 进入 findHighestPriorityRoot
function findHighestPriorityRoot() {
  var highestPriorityWork = NoWork;
  var highestPriorityRoot = null;
  if (lastScheduledRoot !== null) {
    var previousScheduledRoot = lastScheduledRoot;
    var root = firstScheduledRoot;
    while (root !== null) {
      var remainingExpirationTime = root.expirationTime;
     
      ...
      
      else {
        if (remainingExpirationTime > highestPriorityWork) {
          highestPriorityWork = remainingExpirationTime;
          highestPriorityRoot = root;
        }
        if (root === lastScheduledRoot) {
          break;
        }
      
        ...
        
      }
    }
  }

  nextFlushedRoot = highestPriorityRoot; // 其实就是 FiberRoot
  nextFlushedExpirationTime = highestPriorityWork;
}
```