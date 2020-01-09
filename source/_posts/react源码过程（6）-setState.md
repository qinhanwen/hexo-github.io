---
title: react源码过程（6）- setState
date: 2019-12-31 12:51:45
tags:
  - react源码过程
categories:
  - react源码过程
---

## setState

关于 `setState()` 你应该了解三件事：



- 不要直接修改 State

```javascript
// Wrong
// 此代码不会重新渲染组件
this.state.comment = 'Hello';

// Correct
// 而是应该使用 setState()
this.setState({comment: 'Hello'});
```



- State 的更新可能是异步的

出于性能考虑，React 可能会把多个 `setState()` 调用合并成一个调用。

因为 `this.props` 和 `this.state` 可能会异步更新，所以你不要依赖他们的值来更新下一个状态。

```javascript
// Wrong
// 此代码可能会无法更新计数器
this.setState({
  counter: this.state.counter + this.props.increment,
});

// Correct
// 要解决这个问题，可以让 setState() 接收一个函数而不是一个对象。这个函数用上一个 state 作为第一个参数，将此次更新被应用时的 props 做为第二个参数 (也可以使用普通函数，不只是箭头函数)
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```



- State 的更新会被合并

```javascript
 componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
```



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
      state = {
        count: 1
      };
      imcrement = () => {
        this.setState({ count: this.state.count + 1 });
      }
      render() {
        const { count } = this.state;
        return (
          <div onClick={this.imcrement}>
            {count}
          </div>
        )
      }
    }
    ReactDOM.render(
      <App />,
      document.getElementById('container')
    );
  </script>
</body>

</html>
```



触发点击事件

```javascript
Component.prototype.setState = function (partialState, callback) {
	
  ...
  
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};


// 进入 enqueueSetState
  enqueueSetState: function (inst, payload, callback) {
    // inst 是 App 的实例，从它的 _reactInternalFiber 属性上拿到FiberNode
    var fiber = get(inst);
    var currentTime = requestCurrentTime();
    var expirationTime = computeExpirationForFiber(currentTime, fiber);

    var update = createUpdate(expirationTime);
    update.payload = payload;
    if (callback !== undefined && callback !== null) {
      {
        warnOnInvalidCallback$1(callback, 'setState');
      }
      update.callback = callback;
    }

    flushPassiveEffects();
    // update 的 payload 就是要更新的对象
    // fiber 是 FiberNode
    enqueueUpdate(fiber, update);
    scheduleWork(fiber, expirationTime);
  },
    
    
// 进入 scheduleWork
function scheduleWork(fiber, expirationTime) {
  // 获得 FiberRoot
  var root = scheduleWorkToRoot(fiber, expirationTime);
 
  ...

  if (
  !isWorking || isCommitting$1 ||
  nextRoot !== root) {
    var rootExpirationTime = root.expirationTime;
    requestWork(root, rootExpirationTime);
  }
  
  ...
  
}
```

当点击事件触发完之后进入 performWork 调用，之后进入 workLoop 。最后再做更新操作。



## 异步更新例子

```javascript
...
      imcrement = () => {
        this.setState({ count: this.state.count + 1 });
        this.setState({ count: this.state.count + 1 });
        this.setState({ count: this.state.count + 1 });
      }
...
```



触发点击事件，进入 setState 调用后进入 enqueueUpdate 方法

```javascript
function enqueueUpdate(fiber, update) {
  var alternate = fiber.alternate;
  var queue1 = void 0;
  var queue2 = void 0;
  if (alternate === null) {
    queue1 = fiber.updateQueue;
    queue2 = null;
    if (queue1 === null) {
      queue1 = fiber.updateQueue = createUpdateQueue(fiber.memoizedState);
    }
  } else {
    queue1 = fiber.updateQueue;
    queue2 = alternate.updateQueue;
    if (queue1 === null) {
      if (queue2 === null) {
        queue1 = fiber.updateQueue = createUpdateQueue(fiber.memoizedState);
        queue2 = alternate.updateQueue = createUpdateQueue(alternate.memoizedState);
      } else {
        queue1 = fiber.updateQueue = cloneUpdateQueue(queue2);
      }
    } else {
      if (queue2 === null) {
        queue2 = alternate.updateQueue = cloneUpdateQueue(queue1);
      } else {
      }
    }
  }
  if (queue2 === null || queue1 === queue2) {
    appendUpdateToQueue(queue1, update);
  } else {
    if (queue1.lastUpdate === null || queue2.lastUpdate === null) {
      appendUpdateToQueue(queue1, update);
      appendUpdateToQueue(queue2, update);
    } else {
      appendUpdateToQueue(queue1, update);
      queue2.lastUpdate = update;
    }
  }

  {
    if (fiber.tag === ClassComponent && (currentlyProcessingQueue === queue1 || queue2 !== null && currentlyProcessingQueue === queue2) && !didWarnUpdateInsideUpdate) {
      warningWithoutStack$1(false, 'An update (setState, replaceState, or forceUpdate) was scheduled ' + 'from inside an update function. Update functions should be pure, ' + 'with zero side-effects. Consider using componentDidUpdate or a ' + 'callback.');
      didWarnUpdateInsideUpdate = true;
    }
  }
}
```

- 第一次调用，走这条分支

```javascript
function enqueueUpdate(fiber, update) {
  var alternate = fiber.alternate; // 这个值是 null
  var queue1 = void 0;
  var queue2 = void 0;
  if (alternate === null) {
    queue1 = fiber.updateQueue; // 这个值也是 null
    queue2 = null;
    if (queue1 === null) {
      queue1 = fiber.updateQueue = createUpdateQueue(fiber.memoizedState); // fiber.memoizedState 是 {count: 1}
    }
  }
  
  ... 
  
  if (queue2 === null || queue1 === queue2) {// queue2 是 null
    appendUpdateToQueue(queue1, update); // 设置 queue.firstUpdate = queue.lastUpdate = update  update 的值是下面图里的
  }
  
...


// 进入 appendUpdateToQueue
function appendUpdateToQueue(queue, update) {
  if (queue.lastUpdate === null) {// queue.lastUpdate 是 null
    queue.firstUpdate = queue.lastUpdate = update;
  } 
  
  ...
  
}
```

![WX20200101-192226@2x](http://118.24.241.76/WX20200101-192226@2x.png)



- 第二次调用

```javascript
function enqueueUpdate(fiber, update) {
  var alternate = fiber.alternate;// 这个是 null
  var queue1 = void 0;
  var queue2 = void 0;
  if (alternate === null) {
    queue1 = fiber.updateQueue; // 这里上一次调用的时候做了赋值操作，在下面图里
    queue2 = null;

    ...
    
  }
  
  ...
  
  if (queue2 === null || queue1 === queue2) {// queue2 是 null
    appendUpdateToQueue(queue1, update);
  }
    
    
// 
function appendUpdateToQueue(queue, update) {
  
  ...
  
  else { // 第二次 queue.lastUpdate 不是 null 了，所以走这里
    queue.lastUpdate.next = update;
    queue.lastUpdate = update;
  }
}
```

![WX20200101-192650@2x](http://118.24.241.76/WX20200101-192650@2x.png)



- 第三次调用，与第二次类似。最后 queue1.firstUpdate.next 有 n 个 next 组成一条链

![WX20200101-193931@2x](http://118.24.241.76/WX20200101-193931@2x.png)



之后进入 workLoop，在这里整合之前 setState 的操作。是个循环取 queue1.firstUpdate 的 next 属性，一直取到为 null

```javascript
function processUpdateQueue(workInProgress, queue, props, instance, renderExpirationTime) {

  ...
  
  var newBaseState = queue.baseState;
  var newFirstUpdate = null;
  var newExpirationTime = NoWork;

  var update = queue.firstUpdate;
  var resultState = newBaseState;
  while (update !== null) {
    var updateExpirationTime = update.expirationTime;
    if (updateExpirationTime < renderExpirationTime) {
      if (newFirstUpdate === null) {
        newFirstUpdate = update;
        newBaseState = resultState;
      }
      if (newExpirationTime < updateExpirationTime) {
        newExpirationTime = updateExpirationTime;
      }
    } else {
      resultState = getStateFromUpdate(workInProgress, queue, update, resultState, props, instance);
      var _callback = update.callback;
      if (_callback !== null) {
        workInProgress.effectTag |= Callback;
        update.nextEffect = null;
        if (queue.lastEffect === null) {
          queue.firstEffect = queue.lastEffect = update;
        } else {
          queue.lastEffect.nextEffect = update;
          queue.lastEffect = update;
        }
      }
    }
    update = update.next;
  }

  var newFirstCapturedUpdate = null;
  update = queue.firstCapturedUpdate;
  while (update !== null) {
    var _updateExpirationTime = update.expirationTime;
    if (_updateExpirationTime < renderExpirationTime) {
      if (newFirstCapturedUpdate === null) {
        newFirstCapturedUpdate = update;
        if (newFirstUpdate === null) {
          newBaseState = resultState;
        }
      }
      if (newExpirationTime < _updateExpirationTime) {
        newExpirationTime = _updateExpirationTime;
      }
    } else {
      // 合并出的最后的 state 属性
      resultState = getStateFromUpdate(workInProgress, queue, update, resultState, props, instance);
      var _callback2 = update.callback;
      if (_callback2 !== null) {
        workInProgress.effectTag |= Callback;
        update.nextEffect = null;
        if (queue.lastCapturedEffect === null) {
          queue.firstCapturedEffect = queue.lastCapturedEffect = update;
        } else {
          queue.lastCapturedEffect.nextEffect = update;
          queue.lastCapturedEffect = update;
        }
      }
    }
    update = update.next;
  }

  if (newFirstUpdate === null) {
    queue.lastUpdate = null;
  }
  if (newFirstCapturedUpdate === null) {
    queue.lastCapturedUpdate = null;
  } else {
    workInProgress.effectTag |= Callback;
  }
  if (newFirstUpdate === null && newFirstCapturedUpdate === null) {
    newBaseState = resultState;
  }

  queue.baseState = newBaseState;
  queue.firstUpdate = newFirstUpdate;
  queue.firstCapturedUpdate = newFirstCapturedUpdate;
  workInProgress.expirationTime = newExpirationTime;
  workInProgress.memoizedState = resultState;

  {
    currentlyProcessingQueue = null;
  }
}


// 进入 getStateFromUpdate
function getStateFromUpdate(workInProgress, queue, update, prevState, nextProps, instance) {
  switch (update.tag) {
    
    ...  
      
    case UpdateState:
      {
        var _payload2 = update.payload;
        var partialState = void 0;
        if (typeof _payload2 === 'function') { // 这边就是同步 setState 与异步 setState 的区别的地方，因为类型不是 funtion ，所以不关注它做了什么，在异步时候看
        ... 
         
        } else {
          partialState = _payload2;
        }
        if (partialState === null || partialState === undefined) {
          return prevState;
        }
        return _assign({}, prevState, partialState);
      }
    
      ...
        
  }
  return prevState;
}
```

之后走更新的流程



## 同步更新例子

```javascript
...
  imcrement = () => {
        this.setState(pre => {
          return { count: pre.count + 1 }
        });
        this.setState(pre => {
          return { count: pre.count + 1 }
        });
      }
...
```

进入 setState 调用，前面都类似，进入 scheduleWork

```javascript
function scheduleWork(fiber, expirationTime) {
  var root = scheduleWorkToRoot(fiber, expirationTime);
  
  ...
  
  if (
  !isWorking || isCommitting$1 ||
  nextRoot !== root) {
    var rootExpirationTime = root.expirationTime;
    requestWork(root, rootExpirationTime);
  }

  ...
  
}
  
  
// 进入 requestWork
function requestWork(root, expirationTime) {
  addRootToSchedule(root, expirationTime);
  if (isRendering) {
    return;
  }

  if (isBatchingUpdates) {
    if (isUnbatchingUpdates) {
      nextFlushedRoot = root;
      nextFlushedExpirationTime = Sync;
      performWorkOnRoot(root, Sync, false);
    }
    return;
  }

  if (expirationTime === Sync) {
    performSyncWork();
  } else {
    scheduleCallbackWithExpirationTime(root, expirationTime);
  }
}
```



进入 workLoop 后，在 processUpdateQueue 里的 getStateFromUpdate 方法

```javascript
function getStateFromUpdate(workInProgress, queue, update, prevState, nextProps, instance) {
  switch (update.tag) {
 
    ...  
      
    case UpdateState:
      {
        var _payload2 = update.payload;
        var partialState = void 0;
        if (typeof _payload2 === 'function') {
          {
            if (debugRenderPhaseSideEffects || debugRenderPhaseSideEffectsForStrictMode && workInProgress.mode & StrictMode) {
              _payload2.call(instance, prevState, nextProps);
            }
          }
          partialState = _payload2.call(instance, prevState, nextProps); // 这里的 _payload2 是同步 setState 的回调，有2个参数分别是 prevState 与 nextProps
        } 
        
        ...
        
        if (partialState === null || partialState === undefined) {
          return prevState;
        }
        return _assign({}, prevState, partialState);// 在合并参数的时候 count 就被更新了
      }
    case ForceUpdate:
      {
        hasForceUpdate = true;
        return prevState;
      }
  }
  return prevState;
}
```

之后走到更新流程