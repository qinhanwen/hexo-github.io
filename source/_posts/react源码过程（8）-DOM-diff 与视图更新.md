---
title: react源码过程（7）- DOM diff 与视图更新
date: 2019-12-31 12:40:41
tags:
  - react源码过程
categories:
  - react源码过程
---## DOM diff

**无 key**

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
        list: [{name:'1',id:'1'},{name:'2',id:'2'},{name:'3',id:'3'}]
      };
      reverse = () => {
        debugger
        const {list} = this.state;
        this.setState({ list:list.splice(1,1).reverse() });
      }
      render() {
        const {list} = this.state;
        return (
          <div onClick={this.reverse}>
            {list.map(item=><div>{item.name}</div>)}
          </div>
        )
      }
    }
    ReactDOM.render(
      <App/>,
      document.getElementById('container')
    );
  </script>
</body>

</html>
```

从触发 setState 进入到 WorkLoop 后进入 beginWork 开始，这里生成新的 Fiber 树

```javascript
function beginWork(current$$1, workInProgress, renderExpirationTime) {
  var updateExpirationTime = workInProgress.expirationTime;

  if (current$$1 !== null) {
    // 分别拿到旧的和新的 props 属性
    var oldProps = current$$1.memoizedProps;
    var newProps = workInProgress.pendingProps;
    if (oldProps === newProps && !hasContextChanged() && updateExpirationTime < renderExpirationTime) {// 首次更新的时候没有走这里，触发更新的时候走了这里
      switch (workInProgress.tag) {
        case HostRoot:
          pushHostRootContext(workInProgress);
          resetHydrationState();
          break;
          ...
      }
      return bailoutOnAlreadyFinishedWork(current$$1, workInProgress, renderExpirationTime);
    }
  }

  ...

}


// 进入 bailoutOnAlreadyFinishedWork
function bailoutOnAlreadyFinishedWork(current$$1, workInProgress, renderExpirationTime) {
		...
    cloneChildFibers(current$$1, workInProgress);// workInProgress 是 RootFiber
    return workInProgress.child; // 返回复制出来的 child
}


// 进入 cloneChildFibers
function cloneChildFibers(current$$1, workInProgress) {

  ...

  var currentChild = workInProgress.child;// 保存 child
  var newChild = createWorkInProgress(currentChild, currentChild.pendingProps, currentChild.expirationTime);// 复制了一份 child
  workInProgress.child = newChild;// 复制的 child 设置为了 RootFiber 的child

  newChild.return = workInProgress; // 复制的 child 的父亲为 RootFiber
  while (currentChild.sibling !== null) {// 如果有兄弟节点也要更新
    currentChild = currentChild.sibling;
    newChild = newChild.sibling = createWorkInProgress(currentChild, currentChild.pendingProps, currentChild.expirationTime);
    newChild.return = workInProgress;
  }
  newChild.sibling = null;
}


// 进入 createWorkInProgress，这个方法是做复制的
function createWorkInProgress(current, pendingProps, expirationTime) {
  var workInProgress = current.alternate;
  if (workInProgress === null) {
    workInProgress = createFiber(current.tag, pendingProps, current.key, current.mode);
    workInProgress.elementType = current.elementType;
    workInProgress.type = current.type;
    workInProgress.stateNode = current.stateNode;

    {
      workInProgress._debugID = current._debugID;
      workInProgress._debugSource = current._debugSource;
      workInProgress._debugOwner = current._debugOwner;
    }

    workInProgress.alternate = current;
    current.alternate = workInProgress;
  } else {
    workInProgress.pendingProps = pendingProps;

    workInProgress.effectTag = NoEffect;

    workInProgress.nextEffect = null;
    workInProgress.firstEffect = null;
    workInProgress.lastEffect = null;

    if (enableProfilerTimer) {
      workInProgress.actualDuration = 0;
      workInProgress.actualStartTime = -1;
    }
  }

  workInProgress.childExpirationTime = current.childExpirationTime;
  workInProgress.expirationTime = current.expirationTime;

  workInProgress.child = current.child;
  workInProgress.memoizedProps = current.memoizedProps;
  workInProgress.memoizedState = current.memoizedState;
  workInProgress.updateQueue = current.updateQueue;
  workInProgress.firstContextDependency = current.firstContextDependency;

  workInProgress.sibling = current.sibling;
  workInProgress.index = current.index;
  workInProgress.ref = current.ref;

  if (enableProfilerTimer) {
    workInProgress.selfBaseDuration = current.selfBaseDuration;
    workInProgress.treeBaseDuration = current.treeBaseDuration;
  }

  return workInProgress;
}

// 之后的流程就是在 workLoop 里循环

// 进入 updateClassComponent，当前的 FiberNode 是 App
function updateClassComponent(current$$1, workInProgress, Component, nextProps, renderExpirationTime) {

  ...

	else {
    shouldUpdate = updateClassInstance(current$$1, workInProgress, Component, nextProps, renderExpirationTime);// updateClassInstance 方法里有对 updateQueue 里的 state 数据进行合并
  }
  return finishClassComponent(current$$1, workInProgress, Component, shouldUpdate, hasContext, renderExpirationTime);
}


// 当前 FiberNode 是 div 的时候
// 进入 updateHostComponent，并且 workInProgress 的 memoizedProps 和 pendingProps 分别代表旧的 props 和新的 props（一个是 list 有 3 项，一个是 splice 之后的只有 2 项）
function updateHostComponent(current$$1, workInProgress, renderExpirationTime) {
  pushHostContext(workInProgress);

 	...

  var type = workInProgress.type;
  var nextProps = workInProgress.pendingProps;// 新的 props
  var prevProps = current$$1 !== null ? current$$1.memoizedProps : null;// 旧的 props

  var nextChildren = nextProps.children; // 这里的 children 有 2 个，prevProps.children 有 3 个
  var isDirectTextChild = shouldSetTextContent(type, nextProps);

  ...


  reconcileChildren(current$$1, workInProgress, nextChildren, renderExpirationTime);
  return workInProgress.child;
}


// 进入 reconcileChildren

function reconcileChildren(current$$1, workInProgress, nextChildren, renderExpirationTime) {

  ...

    workInProgress.child = reconcileChildFibers(workInProgress, current$$1.child, nextChildren, renderExpirationTime);

  ...

}


// 进入 reconcileChildFibers
  function reconcileChildFibers(returnFiber, currentFirstChild, newChild, expirationTime) {
  // currentFirstChild 是第一个子 FiberNode
		...

    if (isArray(newChild)) {// 根据 newChild 的类型来做不同的处理，这边是数组
      return reconcileChildrenArray(returnFiber, currentFirstChild, newChild, expirationTime);
    }

    ...

  }


// 进入 reconcileChildrenArray
  function reconcileChildrenArray(returnFiber, currentFirstChild, newChildren, expirationTime) {

    ...

    var resultingFirstChild = null;
    var previousNewFiber = null;

    var oldFiber = currentFirstChild;// 老的 FiberNode 它有两个兄弟 FIberNode 所以oldFiber.sibling.sibling 是存在的，一共 3 个
    var lastPlacedIndex = 0;
    var newIdx = 0;
    var nextOldFiber = null; // 用于保存下一个老的兄弟节点的 FiberNode
    for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
      if (oldFiber.index > newIdx) {
        nextOldFiber = oldFiber;
        oldFiber = null;
      } else {
        nextOldFiber = oldFiber.sibling;
      }
      var newFiber = updateSlot(returnFiber, oldFiber, newChildren[newIdx], expirationTime);
      if (newFiber === null) {
        if (oldFiber === null) {
          oldFiber = nextOldFiber;
        }
        break;
      }
      if (shouldTrackSideEffects) {
        if (oldFiber && newFiber.alternate === null) {
          deleteChild(returnFiber, oldFiber);
        }
      }
      lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      previousNewFiber = newFiber;
      oldFiber = nextOldFiber;
    }

    if (newIdx === newChildren.length) {
      deleteRemainingChildren(returnFiber, oldFiber);
      return resultingFirstChild;
    }

    if (oldFiber === null) {
      for (; newIdx < newChildren.length; newIdx++) {
        var _newFiber = createChild(returnFiber, newChildren[newIdx], expirationTime);
        if (!_newFiber) {
          continue;
        }
        lastPlacedIndex = placeChild(_newFiber, lastPlacedIndex, newIdx);
        if (previousNewFiber === null) {
          resultingFirstChild = _newFiber;
        } else {
          previousNewFiber.sibling = _newFiber;
        }
        previousNewFiber = _newFiber;
      }
      return resultingFirstChild;
    }

    var existingChildren = mapRemainingChildren(returnFiber, oldFiber);
    for (; newIdx < newChildren.length; newIdx++) {
      var _newFiber2 = updateFromMap(existingChildren, returnFiber, newIdx, newChildren[newIdx], expirationTime);
      if (_newFiber2) {
        if (shouldTrackSideEffects) {
          if (_newFiber2.alternate !== null) {
            existingChildren.delete(_newFiber2.key === null ? newIdx : _newFiber2.key);
          }
        }
        lastPlacedIndex = placeChild(_newFiber2, lastPlacedIndex, newIdx);
        if (previousNewFiber === null) {
          resultingFirstChild = _newFiber2;
        } else {
          previousNewFiber.sibling = _newFiber2;
        }
        previousNewFiber = _newFiber2;
      }
    }

    if (shouldTrackSideEffects) {
      existingChildren.forEach(function (child) {
        return deleteChild(returnFiber, child);
      });
    }

    return resultingFirstChild;
  }


// 进入
  function updateSlot(returnFiber, oldFiber, newChild, expirationTime) {
    var key = oldFiber !== null ? oldFiber.key : null;// 先拿到老的 FiberNode 的 key
		// 不同类型的节点处理不一样
    if (typeof newChild === 'string' || typeof newChild === 'number') {
      if (key !== null) {
        return null;
      }
      return updateTextNode(returnFiber, oldFiber, '' + newChild, expirationTime);
    }

    if (typeof newChild === 'object' && newChild !== null) { // 第一次走进这里
      switch (newChild.$$typeof) {
        case REACT_ELEMENT_TYPE:
          {
            if (newChild.key === key) {// 因为不存在 key 所以都是 null，就相等了
              if (newChild.type === REACT_FRAGMENT_TYPE) {
                return updateFragment(returnFiber, oldFiber, newChild.props.children, expirationTime, key);
              }
              return updateElement(returnFiber, oldFiber, newChild, expirationTime);// 接着走进这里，返回一个 FiberNode，克隆了 oldFiber 同时包含了新的属性
            }

		...

    return null;
  }


// 进入 updateElement
  function updateElement(returnFiber, current$$1, element, expirationTime) {
    if (current$$1 !== null && current$$1.elementType === element.type) {
      var existing = useFiber(current$$1, element.props, expirationTime);// 克隆了一个新的出来，包含了新的属性
      existing.ref = coerceRef(returnFiber, current$$1, element);
      existing.return = returnFiber;
      {
        existing._debugSource = element._source;
        existing._debugOwner = element._owner;
      }
      return existing;
    }

    ...

  }
```

**小结-无 key 的时候**

- setState 会更新某个 FiberNode 的 updateQueue，之后可以得到被更新的数据。

- FiberNode 新的与旧的之间比较，是遍历新生成 newChildren 数组（只有 2 项）

  ![WechatIMG456](http://118.24.241.76/WechatIMG456.png)

刚进来的时候，声明了几个变量

oldFiber （是旧的 FiberNode，它是第一项，它有两个兄弟 FiberNode）

newIdx = 0

nextOldFiber = null;

lastPlacedIndex = 0;

更新过程中复用了 oldFiber

- 首先 nextOldFiber 变为了 oldFiber.sibling，之后更新 oldFiber 为 newFiber，更新过程：先拿到 oldFiber 的 key，不存在就是 null。如果 newFiber 的 key 与 oldFIber 的 key 相等，就把 oldFiber 复用，克隆成一个新的 FiberNode，并且合并进去了新的数据，让它成为 newFiber。

- 第二次循环与第一次类似。
- 第三次进入 deleteRemainingChildren 方法，移除了最后一个节点，就是将倒数第二个的 sibling 指向 null

之后更新 Fiber 树，再进入 commit 阶段

先将列表最后一项从 dom 中移除，之后再逐个更新列表其他元素。需要注意的就是需要更新的节点会是一个链表，比如这边的 **firstEffect.nextEffct.nextEffct**，是在 renderRoot 阶段里的 workLoop 里生成的

![WX20200108-232929@2x](http://118.24.241.76/WX20200108-232929@2x.png)

看下具体生成的步骤

这里的 firstEffect 是在 reconcileChildrenArray 里生成的，在第三个 FiberNode 删除的时候添加到 returnFiber 上

![WX20200109-001720@2x](http://118.24.241.76/WX20200109-001720@2x.png)

在 completeUnitOfWork 方法做的链接（也就是生成 stateNode 与做链接关系是一件事）

![WX20200109-000620@2x](http://118.24.241.76/WX20200109-000620@2x.png)

**有 key**

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
        list: [{name:'1',id:'1'},{name:'2',id:'2'},{name:'3',id:'3'}]
      };
      reverse = () => {
        const {list} = this.state;
        list.splice(1,1)
        this.setState({ list:list.reverse() });
      }
      render() {
        const {list} = this.state;
        return (
          <div onClick={this.reverse}>
            {list.map(item=><div key={item.id}>{item.name}</div>)}
          </div>
        )
      }
    }
    ReactDOM.render(
      <App/>,
      document.getElementById('container')
    );
  </script>
</body>

</html>
```

主要区别应该是在 reconcileChildrenArray 方法，更新 Fiber 树的地方

在进入到 updateSlot

```javascript
  function updateSlot(returnFiber, oldFiber, newChild, expirationTime) {

    ...

    var key = oldFiber !== null ? oldFiber.key : null;

  	...

    if (typeof newChild === 'object' && newChild !== null) {
      switch (newChild.$$typeof) {
        case REACT_ELEMENT_TYPE:
          {
            if (newChild.key === key) {// key 不相等
              if (newChild.type === REACT_FRAGMENT_TYPE) {
                return updateFragment(returnFiber, oldFiber, newChild.props.children, expirationTime, key);
              }
              return updateElement(returnFiber, oldFiber, newChild, expirationTime);
            } else {
              return null;// 走到了这
            }
          }

		...

  }
```

用 map 通过 key 把对应的 FiberNode 存起来

```javascript
function mapRemainingChildren(returnFiber, currentFirstChild) {
  var existingChildren = new Map()

  var existingChild = currentFirstChild
  while (existingChild !== null) {
    if (existingChild.key !== null) {
      existingChildren.set(existingChild.key, existingChild)
    } else {
      existingChildren.set(existingChild.index, existingChild)
    }
    existingChild = existingChild.sibling
  }
  return existingChildren
}
```

![WX20200109-102218](http://118.24.241.76/WX20200109-102218.png)

之后取出对应的 FiberNode 做更新，还剩下的 existingChildren 里的被删除

```javascript
for (; newIdx < newChildren.length; newIdx++) {
  var _newFiber2 = updateFromMap(
    existingChildren,
    returnFiber,
    newIdx,
    newChildren[newIdx],
    expirationTime
  )
  if (_newFiber2) {
    if (shouldTrackSideEffects) {
      if (_newFiber2.alternate !== null) {
        existingChildren.delete(
          _newFiber2.key === null ? newIdx : _newFiber2.key
        )
      }
    }
    lastPlacedIndex = placeChild(_newFiber2, lastPlacedIndex, newIdx)
    if (previousNewFiber === null) {
      resultingFirstChild = _newFiber2
    } else {
      previousNewFiber.sibling = _newFiber2
    }
    previousNewFiber = _newFiber2
  }
}

if (shouldTrackSideEffects) {
  existingChildren.forEach(function(child) {
    return deleteChild(returnFiber, child)
  })
}
```
