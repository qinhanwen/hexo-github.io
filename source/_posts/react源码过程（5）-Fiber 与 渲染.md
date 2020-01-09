---
title: react源码过程（5）- Fiber 与 渲染
date: 2019-12-23 23:47:08
tags:
  - react源码过程
categories:
  - react源码过程
---

## 例子

**需要先了解的内容**

**挂载**

```jsx
ReactDOM.render(<App />, document.getElementById('container'))
```

这段代码的含义是想在容器中渲染出一个组件。

**fiber 对象**

在 `createFiberRoot` 函数内部，分别创建了两个 `root`，一个 `root` 叫做 `FiberRoot`，另一个 `root` 叫做 `RootFiber`，并且它们两者还是相互引用的。

对于 `FiberRoot` 对象来说，我们现在只需要了解两个属性，分别是 `containerInfo` 及 `current`。前者代表着容器信息，也就是 `document.querySelector('#container')`；后者指向 `RootFiber`。

对于 `RootFiber` 对象来说，我们需要了解的属性稍微多点

```javascript
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode
) {
  this.stateNode = null
  this.return = null
  this.child = null
  this.sibling = null
  this.effectTag = NoEffect
  this.alternate = null
}
```

`return`、`child`、`sibling` 这三个属性很重要，它们是构成 `fiber` 树的主体数据结构。`fiber` 树其实是一个单链表树结构，`return` 及 `child` 分别对应着树的父子节点，并且父节点只有一个 `child` 指向它的第一个子节点，即便是父节点有好多个子节点。那么多个子节点如何连接起来呢？答案是 `sibling`，每个子节点都有一个 `sibling` 属性指向着下一个子节点，都有一个 `return` 属性指向着父节点。

从代码来看

```jsx
const APP = () => (
  <div>
    <span></span>
    <span></span>
  </div>
)
ReactDom.render(<APP />, document.querySelector('#root'))
```

![WX20191225-131528](http://118.24.241.76/WX20191225-131528.png)

## 例子

通过一个例子看一下整个流程做了什么事情

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
        data: [{ key: 1, value: 1 }, { key: 2, value: 2 }]
      };
      componentDidMount() {
        setTimeout(() => {
          const data = [{ key: 0, value: 0 }, { key: 2, value: 2 }]
          this.setState({
            data
          })
        }, 3000);
      }
      render() {
        const { data } = this.state;
        return (
          <div>
            {data.map(item => <p key={item.key}>{item.value}</p>)}
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

**首先 \<App /\> 会生成 ReactELement**

```javascript
function createElement(type, config, children) {
  var propName = void 0;

  var props = {};

  var key = null;
  var ref = null;
  var self = null;
  var source = null;

	// 判断是否传入配置，比如 <div className='11'></div> 中的 className 会被解析到配置中
  if (config != null) {
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    if (hasValidKey(config)) {
      key = '' + config.key;
    }

    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    for (propName in config) {
      if (hasOwnProperty$1.call(config, propName) && !RESERVED_PROPS.hasOwnProperty(propName)) {
        props[propName] = config[propName];
      }
    }
  }

	// 处理 children 的几个操作，长度2之后的全都当成children
  var childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    var childArray = Array(childrenLength);
    for (var i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    {
      if (Object.freeze) {
        Object.freeze(childArray);
      }
    }
    props.children = childArray;
  }


  if (type && type.defaultProps) {
    var defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  {
    if (key || ref) {
      var displayName = typeof type === 'function' ? type.displayName || type.name || 'Unknown' : type;
      if (key) {
        defineKeyPropWarningGetter(props, displayName);
      }
      if (ref) {
        defineRefPropWarningGetter(props, displayName);
      }
    }
  }
  return ReactElement(type, key, ref, self, source, ReactCurrentOwner.current, props);
}

// 进入 ReactElement
// 工厂函数，帮助我们创建 React Element 的，内部代码很简单，多了一个 $$typeof 帮助我们标识它的类型
var ReactElement = function (type, key, ref, self, source, owner, props) {
  var element = {
    $$typeof: REACT_ELEMENT_TYPE,

    type: type,
    key: key,
    ref: ref,
    props: props,

    _owner: owner
  };

  {
    element._store = {};

    ...

  return element;
};
```

生成的 ReactElement

![WX20191226-140330](http://118.24.241.76/WX20191226-140330.png)

**之后进入 ReactDOM.render，第一个参数就是 ReactElement，第二个是 DOM 引用**

![WX20191225-233334@2x](http://118.24.241.76/WX20191225-233334@2x.png)

**进入 legacyRenderSubtreeIntoContainer 方法**

```javascript
function legacyRenderSubtreeIntoContainer(parentComponent, children, container, forceHydrate, callback) {

  !isValidContainer(container) ? invariant(false, 'Target container is not a DOM element.') : void 0;

  {
    topLevelUpdateWarnings(container);
  }

// 刚开始调用 container 上是肯定没有这个属性的
  var root = container._reactRootContainer;
  if (!root) {
 // 创建一个 root 出来，类型是 ReactRoot
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(container, forceHydrate);

    ...

		// batchedUpdate ，也就是批量更新
    unbatchedUpdates(function () {
      if (parentComponent != null) {
        root.legacy_renderSubtreeIntoContainer(parentComponent, children, callback);
      } else {
        root.render(children, callback);
      }
    });
  } else {

    ...

    if (parentComponent != null) {
      root.legacy_renderSubtreeIntoContainer(parentComponent, children, callback);
    } else {
      root.render(children, callback);
    }
  }
  return getPublicRootInstance(root._internalRoot);
}


// 创建 ReactRoot 赋值给 root 和 container._reactRootContainer
function legacyCreateRootFromDOMContainer(container, forceHydrate) {

  ...

  var isConcurrent = false;
  return new ReactRoot(container, isConcurrent, shouldHydrate);
}


// 进入 ReactRoot
function ReactRoot(container, isConcurrent, hydrate) {
  var root = createContainer(container, isConcurrent, hydrate);
  this._internalRoot = root;
}


// 进入createContainer
function createContainer(containerInfo, isConcurrent, hydrate) {
  return createFiberRoot(containerInfo, isConcurrent, hydrate);
}


// 进入createFiberRoot
function createFiberRoot(containerInfo, isConcurrent, hydrate) {

  // 创建 RootFiber
  var uninitializedFiber = createHostRootFiber(isConcurrent);

  var root = void 0;
  if (enableSchedulerTracing) {
    root = {
      current: uninitializedFiber, // 挂载了 RootFiber
      containerInfo: containerInfo,
      pendingChildren: null,

      earliestPendingTime: NoWork,
      latestPendingTime: NoWork,
      earliestSuspendedTime: NoWork,
      latestSuspendedTime: NoWork,
      latestPingedTime: NoWork,

      didError: false,

      pendingCommitExpirationTime: NoWork,
      finishedWork: null,
      timeoutHandle: noTimeout,
      context: null,
      pendingContext: null,
      hydrate: hydrate,
      nextExpirationTimeToWorkOn: NoWork,
      expirationTime: NoWork,
      firstBatch: null,
      nextScheduledRoot: null,

      interactionThreadID: unstable_getThreadID(),
      memoizedInteractions: new Set(),
      pendingInteractionMap: new Map()
    };
  } else {
    root = {
      current: uninitializedFiber,
      containerInfo: containerInfo,
      pendingChildren: null,

      earliestPendingTime: NoWork,
      latestPendingTime: NoWork,
      earliestSuspendedTime: NoWork,
      latestSuspendedTime: NoWork,
      latestPingedTime: NoWork,

      didError: false,

      pendingCommitExpirationTime: NoWork,
      finishedWork: null,
      timeoutHandle: noTimeout,
      context: null,
      pendingContext: null,
      hydrate: hydrate,
      nextExpirationTimeToWorkOn: NoWork,
      expirationTime: NoWork,
      firstBatch: null,
      nextScheduledRoot: null
    };
  }

  uninitializedFiber.stateNode = root;

	// RootFiber
  return root;
}


// 进入 createHostRootFiber
function createHostRootFiber(isConcurrent) {
  ...
  return createFiber(HostRoot, null, null, mode);
}


// 进入createFiber
var createFiber = function (tag, pendingProps, key, mode) {
  return new FiberNode(tag, pendingProps, key, mode);
};


// 进入FiberNode
// 对于 FiberNode 中的属性，我们当下只需要以下几点
// stateNode 保存了每个节点的 DOM 信息
// return、child、sibling、index 组成了单链表树结构
// return 代表父 fiber，child 代表子 fiber、sibling 代表下一个兄弟节点，和链表中的 next 一个含义
// index 代表了当前 fiber 的索引
function FiberNode(tag, pendingProps, key, mode) {
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;

  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

	...
}


// 进入unbatchedUpdates
function unbatchedUpdates(fn, a) {

  ...

  return fn(a); // 进入回调
}
```

**进入回调调用**

```javascript
unbatchedUpdates(function() {
  // 创建 root 的时候不可能存在 parentComponent，所以进入 root.render(children, callback)
  if (parentComponent != null) {
    root.legacy_renderSubtreeIntoContainer(parentComponent, children, callback)
  } else {
    root.render(children, callback)
  }
})
```

**进入 render**

```javascript
ReactRoot.prototype.render = function (children, callback) {
  var root = this._internalRoot; // 这个是刚才创建的 FiberRoot
  var work = new ReactWork();

  ...

  updateContainer(children, root, null, work._onCommit);
  return work;
};
```

**进入 updateContainer**

```javascript
function updateContainer(element, container, parentComponent, callback) {
  var current$$1 = container.current // 这个是RootFiber
  var currentTime = requestCurrentTime() // 计算时间

  // expirationTime 代表优先级，数字越大优先级越高
  // sync 的数字是最大的，所以优先级也是最高的
  var expirationTime = computeExpirationForFiber(currentTime, current$$1)
  return updateContainerAtExpirationTime(
    element,
    container,
    parentComponent,
    expirationTime,
    callback
  )
}
```

**进入 updateContainerAtExpirationTime**

```javascript
function updateContainerAtExpirationTime(element, container, parentComponent, expirationTime, callback) {
  var current$$1 = container.current; // 获得 RootFiber

  ...

  // 获取 context 并赋值，因为 parentComponent 为 null，所以是个空对象

  var context = getContextForSubtree(parentComponent);
  if (container.context === null) {
    container.context = context;
  } else {
    container.pendingContext = context;
  }

  return scheduleRootUpdate(current$$1, element, expirationTime, callback);
}
```

**进入 scheduleRootUpdate**

```javascript
function scheduleRootUpdate(current$$1, element, expirationTime, callback) {

  ...

  var update = createUpdate(expirationTime);

  update.payload = { element: element };

  ...


  enqueueUpdate(current$$1, update);
  scheduleWork(current$$1, expirationTime);

  return expirationTime;
}


// 创建的 update 对象
function createUpdate(expirationTime) {
  return {
    expirationTime: expirationTime,

    tag: UpdateState,
    payload: null,
    callback: null,

    next: null,
    nextEffect: null
  };
}


// 进入 enqueueUpdate
function enqueueUpdate(fiber, update) {
  // 获取 fiber 的镜像
  var alternate = fiber.alternate;
  var queue1 = void 0;
  var queue2 = void 0;

  // 第一次 render 的时候肯定是没有这个镜像的，所以进第一个条件
  if (alternate === null) {

    queue1 = fiber.updateQueue;
    queue2 = null;
    if (queue1 === null) {
      // UpdateQueue 是一个链表组成的队列
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

  // 获取队列操作完毕以后，就开始入队了
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

	...
}


// 进入 createUpdateQueue
function createUpdateQueue(baseState) {
  var queue = {
    baseState: baseState,
    firstUpdate: null,
    lastUpdate: null,
    firstCapturedUpdate: null,
    lastCapturedUpdate: null,
    firstEffect: null,
    lastEffect: null,
    firstCapturedEffect: null,
    lastCapturedEffect: null
  };
  return queue;
}


// 进入 appendUpdateToQueue
function appendUpdateToQueue(queue, update) {

  if (queue.lastUpdate === null) {
		// 第一次走了这里
    queue.firstUpdate = queue.lastUpdate = update;
  } else {
    queue.lastUpdate.next = update;
    queue.lastUpdate = update;
  }
}


// 进入 scheduleWork
function scheduleWork(fiber, expirationTime) {
  // 获得 FiberRoot
  var root = scheduleWorkToRoot(fiber, expirationTime);

  if (root === null) {
    {
      switch (fiber.tag) {
        case ClassComponent:
          warnAboutUpdateOnUnmounted(fiber, true);
          break;
        case FunctionComponent:
        case ForwardRef:
        case MemoComponent:
        case SimpleMemoComponent:
          warnAboutUpdateOnUnmounted(fiber, false);
          break;
      }
    }
    return;
  }

  if (!isWorking && nextRenderExpirationTime !== NoWork && expirationTime > nextRenderExpirationTime) {

    interruptedBy = fiber;
    resetStack();
  }
  markPendingPriorityLevel(root, expirationTime);
  if (

  !isWorking || isCommitting$1 ||
  nextRoot !== root) {
    var rootExpirationTime = root.expirationTime;
    requestWork(root, rootExpirationTime);
  }
  if (nestedUpdateCount > NESTED_UPDATE_LIMIT) {

    nestedUpdateCount = 0;
    invariant(false, 'Maximum update depth exceeded. This can happen when a component repeatedly calls setState inside componentWillUpdate or componentDidUpdate. React limits the number of nested updates to prevent infinite loops.');
  }
}


// 进入 requestWork
function requestWork(root, expirationTime) {
  // 添加 root 到调度
  addRootToSchedule(root, expirationTime);
  if (isRendering) {

    return;
  }

  // 判断是否批量更新
  if (isBatchingUpdates) {

    if (isUnbatchingUpdates) {

      nextFlushedRoot = root;
      nextFlushedExpirationTime = Sync;
      performWorkOnRoot(root, Sync, false);
    }
    return;
  }

 // 判断优先级是同步还是异步，异步的话需要调度
  if (expirationTime === Sync) {
    performSyncWork();
  } else {
    // 实现了 requestIdleCallback 的 polyfill 版本
    scheduleCallbackWithExpirationTime(root, expirationTime);
  }
}


// 进入 addRootToSchedule
function addRootToSchedule(root, expirationTime) {
  // 添加 root 到调度中
  // 判断是否调度过
  if (root.nextScheduledRoot === null) {
		// 如果没有，就添加进去
    root.expirationTime = expirationTime;
    if (lastScheduledRoot === null) {
      firstScheduledRoot = lastScheduledRoot = root;
      root.nextScheduledRoot = root;
    } else {
      lastScheduledRoot.nextScheduledRoot = root;
      lastScheduledRoot = root;
      lastScheduledRoot.nextScheduledRoot = firstScheduledRoot;
    }
  } else {
		// 已经调度过，判断是否需要更新优先级
    var remainingExpirationTime = root.expirationTime;
    if (expirationTime > remainingExpirationTime) {
      root.expirationTime = expirationTime;
    }
  }
}


//  进入 performSyncWork
function performSyncWork() {
  performWork(Sync, false);
}


//  进入 performWork
function performWork(minExpirationTime, isYieldy) {

	...

  if (isYieldy) {

    ...

  } else {
    while (nextFlushedRoot !== null && nextFlushedExpirationTime !== NoWork && minExpirationTime <= nextFlushedExpirationTime) {
      performWorkOnRoot(nextFlushedRoot, nextFlushedExpirationTime, false);
      findHighestPriorityRoot();
    }
  }

	...

  if (nextFlushedExpirationTime !== NoWork) {
    scheduleCallbackWithExpirationTime(nextFlushedRoot, nextFlushedExpirationTime);
  }

  finishRendering();
}


// 进入 performWorkOnRoot
function performWorkOnRoot(root, expirationTime, isYieldy) {
  !!isRendering ? invariant(false, 'performWorkOnRoot was called recursively. This error is likely caused by a bug in React. Please file an issue.') : void 0;

  isRendering = true;


  if (!isYieldy) {// 不可打断任务

    // 判断是否存在已完成的 finishedWork，存在话就完成它
    var finishedWork = root.finishedWork;
    if (finishedWork !== null) {

      completeRoot(root, finishedWork, expirationTime);
    } else {
      root.finishedWork = null;

      var timeoutHandle = root.timeoutHandle;
      if (timeoutHandle !== noTimeout) {
        root.timeoutHandle = noTimeout;

        cancelTimeout(timeoutHandle);
      }

			// 渲染成 DOM
      renderRoot(root, isYieldy);
      finishedWork = root.finishedWork;
      if (finishedWork !== null) {

        completeRoot(root, finishedWork, expirationTime);
      }
    }
  } else {
		// 可打断任务
    var _finishedWork = root.finishedWork;
    if (_finishedWork !== null) {

      completeRoot(root, _finishedWork, expirationTime);
    } else {
      root.finishedWork = null;

      var _timeoutHandle = root.timeoutHandle;
      if (_timeoutHandle !== noTimeout) {
        root.timeoutHandle = noTimeout;

        cancelTimeout(_timeoutHandle);
      }
      renderRoot(root, isYieldy);
      _finishedWork = root.finishedWork;
      if (_finishedWork !== null) {

        if (!shouldYieldToRenderer()) {

          completeRoot(root, _finishedWork, expirationTime);
        } else {

          root.finishedWork = _finishedWork;
        }
      }
    }
  }

  isRendering = false;
}


// 进入 renderRoot
function renderRoot(root, isYieldy) {
  !!isWorking ? invariant(false, 'renderRoot was called recursively. This error is likely caused by a bug in React. Please file an issue.') : void 0;

  flushPassiveEffects();

  isWorking = true;
  if (enableHooks) {
    ReactCurrentOwner$2.currentDispatcher = Dispatcher;
  } else {
    ReactCurrentOwner$2.currentDispatcher = DispatcherWithoutHooks;
  }

  var expirationTime = root.nextExpirationTimeToWorkOn;

  if (expirationTime !== nextRenderExpirationTime || root !== nextRoot || nextUnitOfWork === null) {
    resetStack();
    nextRoot = root;
    nextRenderExpirationTime = expirationTime;

    // 获取下一个需要工作的单元
    nextUnitOfWork = createWorkInProgress(nextRoot.current, null, nextRenderExpirationTime);
    root.pendingCommitExpirationTime = NoWork;

    if (enableSchedulerTracing) {
      var interactions = new Set();
      root.pendingInteractionMap.forEach(function (scheduledInteractions, scheduledExpirationTime) {
        if (scheduledExpirationTime >= expirationTime) {
          scheduledInteractions.forEach(function (interaction) {
            return interactions.add(interaction);
          });
        }
      });

      root.memoizedInteractions = interactions;

      if (interactions.size > 0) {
        var subscriber = __subscriberRef.current;
        if (subscriber !== null) {
          var threadID = computeThreadID(expirationTime, root.interactionThreadID);
          try {
            subscriber.onWorkStarted(interactions, threadID);
          } catch (error) {
            if (!hasUnhandledError) {
              hasUnhandledError = true;
              unhandledError = error;
            }
          }
        }
      }
    }
  }

  var prevInteractions = null;
  if (enableSchedulerTracing) {
    prevInteractions = __interactionsRef.current;
    __interactionsRef.current = root.memoizedInteractions;
  }

  var didFatal = false;

  startWorkLoopTimer(nextUnitOfWork);

  // 更新节点
  do {
    try {
      workLoop(isYieldy);
    } catch (thrownValue) {
      resetContextDependences();
      resetHooks();

      var mayReplay = void 0;
      if (true && replayFailedUnitOfWorkWithInvokeGuardedCallback) {
        mayReplay = mayReplayFailedUnitOfWork;
        mayReplayFailedUnitOfWork = true;
      }

      if (nextUnitOfWork === null) {
        didFatal = true;
        onUncaughtError(thrownValue);
      } else {
        if (enableProfilerTimer && nextUnitOfWork.mode & ProfileMode) {
          stopProfilerTimerIfRunningAndRecordDelta(nextUnitOfWork, true);
        }

        {
          resetCurrentlyProcessingQueue();
        }

        if (true && replayFailedUnitOfWorkWithInvokeGuardedCallback) {
          if (mayReplay) {
            var failedUnitOfWork = nextUnitOfWork;
            replayUnitOfWork(failedUnitOfWork, thrownValue, isYieldy);
          }
        }

        !(nextUnitOfWork !== null) ? invariant(false, 'Failed to replay rendering after an error. This is likely caused by a bug in React. Please file an issue with a reproducing case to help us find it.') : void 0;

        var sourceFiber = nextUnitOfWork;
        var returnFiber = sourceFiber.return;
        if (returnFiber === null) {
          didFatal = true;
          onUncaughtError(thrownValue);
        } else {
          throwException(root, returnFiber, sourceFiber, thrownValue, nextRenderExpirationTime);
          nextUnitOfWork = completeUnitOfWork(sourceFiber);
          continue;
        }
      }
    }
    break;
  } while (true);

  if (enableSchedulerTracing) {
    __interactionsRef.current = prevInteractions;
  }

  isWorking = false;
  ReactCurrentOwner$2.currentDispatcher = null;
  resetContextDependences();
  resetHooks();

  if (didFatal) {
    var _didCompleteRoot = false;
    stopWorkLoopTimer(interruptedBy, _didCompleteRoot);
    interruptedBy = null;

    {
      resetStackAfterFatalErrorInDev();
    }

    nextRoot = null;
    onFatal(root);
    return;
  }

  if (nextUnitOfWork !== null) {

    var _didCompleteRoot2 = false;
    stopWorkLoopTimer(interruptedBy, _didCompleteRoot2);
    interruptedBy = null;
    onYield(root);
    return;
  }


  var didCompleteRoot = true;
  stopWorkLoopTimer(interruptedBy, didCompleteRoot);
  var rootWorkInProgress = root.current.alternate;
  !(rootWorkInProgress !== null) ? invariant(false, 'Finished root should have a work-in-progress. This error is likely caused by a bug in React. Please file an issue.') : void 0;


  nextRoot = null;
  interruptedBy = null;

  if (nextRenderDidError) {
    if (hasLowerPriorityWork(root, expirationTime)) {

      markSuspendedPriorityLevel(root, expirationTime);
      var suspendedExpirationTime = expirationTime;
      var rootExpirationTime = root.expirationTime;
      onSuspend(root, rootWorkInProgress, suspendedExpirationTime, rootExpirationTime, -1
      );
      return;
    } else if (

    !root.didError && isYieldy) {
      root.didError = true;
      var _suspendedExpirationTime = root.nextExpirationTimeToWorkOn = expirationTime;
      var _rootExpirationTime = root.expirationTime = Sync;
      onSuspend(root, rootWorkInProgress, _suspendedExpirationTime, _rootExpirationTime, -1
      );
      return;
    }
  }

  if (isYieldy && nextLatestAbsoluteTimeoutMs !== -1) {
    var _suspendedExpirationTime2 = expirationTime;
    markSuspendedPriorityLevel(root, _suspendedExpirationTime2);


    var earliestExpirationTime = findEarliestOutstandingPriorityLevel(root, expirationTime);
    var earliestExpirationTimeMs = expirationTimeToMs(earliestExpirationTime);
    if (earliestExpirationTimeMs < nextLatestAbsoluteTimeoutMs) {
      nextLatestAbsoluteTimeoutMs = earliestExpirationTimeMs;
    }

    var currentTimeMs = expirationTimeToMs(requestCurrentTime());
    var msUntilTimeout = nextLatestAbsoluteTimeoutMs - currentTimeMs;
    msUntilTimeout = msUntilTimeout < 0 ? 0 : msUntilTimeout;

    var _rootExpirationTime2 = root.expirationTime;
    onSuspend(root, rootWorkInProgress, _suspendedExpirationTime2, _rootExpirationTime2, msUntilTimeout);
    return;
  }

  onComplete(root, rootWorkInProgress, expirationTime);
}


// 进入 resetStack
function resetStack() {
  // nextUnitOfWork：下一个需要执行的 fiber 节点
  if (nextUnitOfWork !== null) {
    // 往上找 fiber 节点
    var interruptedWork = nextUnitOfWork.return;
    // 如果存在父节点的话，就清掉父节点的 valueStack
    // 发现这个数组应该是用来存储数据的
    // 这个做法应该是为了重头开始一个新的任务。因为打断一个任务的时候
    // 被打断的任务可能已经改变一部分节点的数据，这时候新的任务开始时
    // 不应该被之前的任务所影响，需要清掉之前任务的影响。
    while (interruptedWork !== null) {
      unwindInterruptedWork(interruptedWork);
      interruptedWork = interruptedWork.return;
    }
  }

  {
    ReactStrictModeWarnings.discardPendingWarnings();
    checkThatStackIsEmpty();
  }
	// 重置
  nextRoot = null;
  nextRenderExpirationTime = NoWork;
  nextLatestAbsoluteTimeoutMs = -1;
  nextRenderDidError = false;
  nextUnitOfWork = null;
}


// 进入 workLoop
function workLoop(isYieldy) {
  if (!isYieldy) {
    // 对 nextUnitOfWork 循环进行判断，直到没有 nextUnitOfWork
    // 一开始进来 nextUnitOfWork 是 root，每次执行 performUnitOfWork 后
    // 都会生成下一个工作单元
    while (nextUnitOfWork !== null) {
      nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    }
  } else {
    while (nextUnitOfWork !== null && !shouldYieldToRenderer()) {
      nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    }
  }
}


// 进入 performUnitOfWork
// 开始组件更新
function performUnitOfWork(workInProgress) {
  var current$$1 = workInProgress.alternate;

  startWorkTimer(workInProgress);
  {
    setCurrentFiber(workInProgress);
  }

  if (true && replayFailedUnitOfWorkWithInvokeGuardedCallback) {
    stashedWorkInProgressProperties = assignFiberPropertiesInDEV(stashedWorkInProgressProperties, workInProgress);
  }

  var next = void 0;
  if (enableProfilerTimer) {
    if (workInProgress.mode & ProfileMode) {
      startProfilerTimer(workInProgress);
    }

    // 获得要更新FiberNode
    next = beginWork(current$$1, workInProgress, nextRenderExpirationTime);
    workInProgress.memoizedProps = workInProgress.pendingProps;

    if (workInProgress.mode & ProfileMode) {
      stopProfilerTimerIfRunningAndRecordDelta(workInProgress, true);
    }
  } else {
    next = beginWork(current$$1, workInProgress, nextRenderExpirationTime);
    workInProgress.memoizedProps = workInProgress.pendingProps;
  }

  {
    resetCurrentFiber();
    if (isReplayingFailedUnitOfWork) {
      rethrowOriginalError();
    }
  }
  if (true && ReactFiberInstrumentation_1.debugTool) {
    ReactFiberInstrumentation_1.debugTool.onBeginWork(workInProgress);
  }

  if (next === null) {
    next = completeUnitOfWork(workInProgress);
  }

  ReactCurrentOwner$2.current = null;

  return next;
}


// 进入 beginwork
function beginWork(current$$1, workInProgress, renderExpirationTime) {
  var updateExpirationTime = workInProgress.expirationTime;

  if (current$$1 !== null) {
    var oldProps = current$$1.memoizedProps;
    var newProps = workInProgress.pendingProps;
    if (oldProps === newProps && !hasContextChanged() && updateExpirationTime < renderExpirationTime) {

      // 根据节点类型优化
      switch (workInProgress.tag) {
        case HostRoot:
          pushHostRootContext(workInProgress);
          resetHydrationState();
          break;
        case HostComponent:
          pushHostContext(workInProgress);
          break;
        case ClassComponent:
          {
            var Component = workInProgress.type;
            if (isContextProvider(Component)) {
              pushContextProvider(workInProgress);
            }
            break;
          }
        case HostPortal:
          pushHostContainer(workInProgress, workInProgress.stateNode.containerInfo);
          break;
        case ContextProvider:
          {
            var newValue = workInProgress.memoizedProps.value;
            pushProvider(workInProgress, newValue);
            break;
          }
        case Profiler:
          if (enableProfilerTimer) {
            workInProgress.effectTag |= Update;
          }
          break;
        case SuspenseComponent:
          {
            var state = workInProgress.memoizedState;
            var didTimeout = state !== null;
            if (didTimeout) {
              var primaryChildFragment = workInProgress.child;
              var primaryChildExpirationTime = primaryChildFragment.childExpirationTime;
              if (primaryChildExpirationTime !== NoWork && primaryChildExpirationTime >= renderExpirationTime) {
                return updateSuspenseComponent(current$$1, workInProgress, renderExpirationTime);
              } else {
                var child = bailoutOnAlreadyFinishedWork(current$$1, workInProgress, renderExpirationTime);
                if (child !== null) {
                  return child.sibling;
                } else {
                  return null;
                }
              }
            }
            break;
          }
      }
      return bailoutOnAlreadyFinishedWork(current$$1, workInProgress, renderExpirationTime);
    }
  }

  workInProgress.expirationTime = NoWork;

  // 根据节点处理
  switch (workInProgress.tag) {
    case IndeterminateComponent:
      {
        var elementType = workInProgress.elementType;
        return mountIndeterminateComponent(current$$1, workInProgress, elementType, renderExpirationTime);
      }
    case LazyComponent:
      {
        var _elementType = workInProgress.elementType;
        return mountLazyComponent(current$$1, workInProgress, _elementType, updateExpirationTime, renderExpirationTime);
      }
    case FunctionComponent:
      {
        var _Component = workInProgress.type;
        var unresolvedProps = workInProgress.pendingProps;
        var resolvedProps = workInProgress.elementType === _Component ? unresolvedProps : resolveDefaultProps(_Component, unresolvedProps);
        return updateFunctionComponent(current$$1, workInProgress, _Component, resolvedProps, renderExpirationTime);
      }
    case ClassComponent:
      {
        var _Component2 = workInProgress.type;
        var _unresolvedProps = workInProgress.pendingProps;
        var _resolvedProps = workInProgress.elementType === _Component2 ? _unresolvedProps : resolveDefaultProps(_Component2, _unresolvedProps);
        return updateClassComponent(current$$1, workInProgress, _Component2, _resolvedProps, renderExpirationTime);
      }
    case HostRoot:
      return updateHostRoot(current$$1, workInProgress, renderExpirationTime);
    case HostComponent:
      return updateHostComponent(current$$1, workInProgress, renderExpirationTime);
    case HostText:
      return updateHostText(current$$1, workInProgress);
    case SuspenseComponent:
      return updateSuspenseComponent(current$$1, workInProgress, renderExpirationTime);
    case HostPortal:
      return updatePortalComponent(current$$1, workInProgress, renderExpirationTime);
    case ForwardRef:
      {
        var type = workInProgress.type;
        var _unresolvedProps2 = workInProgress.pendingProps;
        var _resolvedProps2 = workInProgress.elementType === type ? _unresolvedProps2 : resolveDefaultProps(type, _unresolvedProps2);
        return updateForwardRef(current$$1, workInProgress, type, _resolvedProps2, renderExpirationTime);
      }
    case Fragment:
      return updateFragment(current$$1, workInProgress, renderExpirationTime);
    case Mode:
      return updateMode(current$$1, workInProgress, renderExpirationTime);
    case Profiler:
      return updateProfiler(current$$1, workInProgress, renderExpirationTime);
    case ContextProvider:
      return updateContextProvider(current$$1, workInProgress, renderExpirationTime);
    case ContextConsumer:
      return updateContextConsumer(current$$1, workInProgress, renderExpirationTime);
    case MemoComponent:
      {
        var _type = workInProgress.type;
        var _unresolvedProps3 = workInProgress.pendingProps;
        var _resolvedProps3 = resolveDefaultProps(_type.type, _unresolvedProps3);
        return updateMemoComponent(current$$1, workInProgress, _type, _resolvedProps3, updateExpirationTime, renderExpirationTime);
      }
    case SimpleMemoComponent:
      {
        return updateSimpleMemoComponent(current$$1, workInProgress, workInProgress.type, workInProgress.pendingProps, updateExpirationTime, renderExpirationTime);
      }
    case IncompleteClassComponent:
      {
        var _Component3 = workInProgress.type;
        var _unresolvedProps4 = workInProgress.pendingProps;
        var _resolvedProps4 = workInProgress.elementType === _Component3 ? _unresolvedProps4 : resolveDefaultProps(_Component3, _unresolvedProps4);
        return mountIncompleteClassComponent(current$$1, workInProgress, _Component3, _resolvedProps4, renderExpirationTime);
      }
    default:
      invariant(false, 'Unknown unit of work tag. This error is likely caused by a bug in React. Please file an issue.');
  }
}

  
// 进入 finishClassComponent
function finishClassComponent(current$$1, workInProgress, Component, shouldUpdate, hasContext, renderExpirationTime) {


  var instance = workInProgress.stateNode;

...  
  
  else {
    {
      setCurrentPhase('render');
      nextChildren = instance.render();// 调用组件实例的 render 方法
      if (debugRenderPhaseSideEffects || debugRenderPhaseSideEffectsForStrictMode && workInProgress.mode & StrictMode) {
        instance.render();
      }
      setCurrentPhase(null);
    }
  }
  
  workInProgress.effectTag |= PerformedWork;
  if (current$$1 !== null && didCaptureError) {
    forceUnmountCurrentAndReconcile(current$$1, workInProgress, nextChildren, renderExpirationTime);
  } else {
    // 设置 workInProgress 的 child 属性
    reconcileChildren(current$$1, workInProgress, nextChildren, renderExpirationTime);
  }

  workInProgress.memoizedState = instance.state;

  if (hasContext) {
    invalidateContextProvider(workInProgress, Component, true);
  }

  return workInProgress.child;
}
  

// 进入 completeRoot
function completeRoot(root, finishedWork, expirationTime) {
  var firstBatch = root.firstBatch;
  if (firstBatch !== null && firstBatch._expirationTime >= expirationTime) {
    if (completedBatches === null) {
      completedBatches = [firstBatch];
    } else {
      completedBatches.push(firstBatch);
    }
    if (firstBatch._defer) {
      root.finishedWork = finishedWork;
      root.expirationTime = NoWork;
      return;
    }
  }

  root.finishedWork = null;

  if (root === lastCommittedRootDuringThisBatch) {
    nestedUpdateCount++;
  } else {

    lastCommittedRootDuringThisBatch = root;
    nestedUpdateCount = 0;
  }
  commitRoot(root, finishedWork);
}


// 进入commitRoot
function commitRoot(root, finishedWork) {

	...

  while (nextEffect !== null) {
    var _didError = false;
    var _error = void 0;
    {
      invokeGuardedCallback(null, commitAllHostEffects, null);
      if (hasCaughtError()) {
        _didError = true;
        _error = clearCaughtError();
      }
    }
    if (_didError) {
      !(nextEffect !== null) ? invariant(false, 'Should have next effect. This error is likely caused by a bug in React. Please file an issue.') : void 0;
      captureCommitPhaseError(nextEffect, _error);
      if (nextEffect !== null) {
        nextEffect = nextEffect.nextEffect;
      }
    }
  }
  stopCommitHostEffectsTimer();

  resetAfterCommit(root.containerInfo);

  root.current = finishedWork;

  nextEffect = firstEffect;
  startCommitLifeCyclesTimer();
  while (nextEffect !== null) {
    var _didError2 = false;
    var _error2 = void 0;
    {
      invokeGuardedCallback(null, commitAllLifeCycles, null, root, committedExpirationTime);
      if (hasCaughtError()) {
        _didError2 = true;
        _error2 = clearCaughtError();
      }
    }
    if (_didError2) {
      !(nextEffect !== null) ? invariant(false, 'Should have next effect. This error is likely caused by a bug in React. Please file an issue.') : void 0;
      captureCommitPhaseError(nextEffect, _error2);
      if (nextEffect !== null) {
        nextEffect = nextEffect.nextEffect;
      }
    }
  }

  if (enableHooks && firstEffect !== null && rootWithPendingPassiveEffects !== null) {
    var callback = commitPassiveEffects.bind(null, root, firstEffect);
    if (enableSchedulerTracing) {
      callback = unstable_wrap(callback);
    }
    passiveEffectCallbackHandle = unstable_scheduleCallback(callback);
    passiveEffectCallback = callback;
  }

  isCommitting$1 = false;
  isWorking = false;
  stopCommitLifeCyclesTimer();
  stopCommitTimer();
  onCommitRoot(finishedWork.stateNode);
  if (true && ReactFiberInstrumentation_1.debugTool) {
    ReactFiberInstrumentation_1.debugTool.onCommitWork(finishedWork);
  }

  var updateExpirationTimeAfterCommit = finishedWork.expirationTime;
  var childExpirationTimeAfterCommit = finishedWork.childExpirationTime;
  var earliestRemainingTimeAfterCommit = childExpirationTimeAfterCommit > updateExpirationTimeAfterCommit ? childExpirationTimeAfterCommit : updateExpirationTimeAfterCommit;
  if (earliestRemainingTimeAfterCommit === NoWork) {
    legacyErrorBoundariesThatAlreadyFailed = null;
  }
  onCommit(root, earliestRemainingTimeAfterCommit);

  if (enableSchedulerTracing) {
    __interactionsRef.current = prevInteractions;

    var subscriber = void 0;

    try {
      subscriber = __subscriberRef.current;
      if (subscriber !== null && root.memoizedInteractions.size > 0) {
        var threadID = computeThreadID(committedExpirationTime, root.interactionThreadID);
        subscriber.onWorkStopped(root.memoizedInteractions, threadID);
      }
    } catch (error) {
      if (!hasUnhandledError) {
        hasUnhandledError = true;
        unhandledError = error;
      }
    } finally {
      var pendingInteractionMap = root.pendingInteractionMap;
      pendingInteractionMap.forEach(function (scheduledInteractions, scheduledExpirationTime) {
        if (scheduledExpirationTime > earliestRemainingTimeAfterCommit) {
          pendingInteractionMap.delete(scheduledExpirationTime);

          scheduledInteractions.forEach(function (interaction) {
            interaction.__count--;

            if (subscriber !== null && interaction.__count === 0) {
              try {
                subscriber.onInteractionScheduledWorkCompleted(interaction);
              } catch (error) {

                if (!hasUnhandledError) {
                  hasUnhandledError = true;
                  unhandledError = error;
                }
              }
            }
          });
        }
      });
    }
  }
}


// 进入invokeGuardedCallback
function invokeGuardedCallback(name, func, context, a, b, c, d, e, f) {
  hasError = false;
  caughtError = null;
  invokeGuardedCallbackImpl$1.apply(reporter, arguments);
}


// 进入 invokeGuardedCallbackDev
  var invokeGuardedCallbackDev = function (name, func, context, a, b, c, d, e, f) {

      !(typeof document !== 'undefined') ? invariant(false, 'The `document` global was defined when React was initialized, but is not defined anymore. This can happen in a test environment if a component schedules an update from an asynchronous callback, but the test has already finished running. To solve this, you can either unmount the component at the end of your test (and ensure that any asynchronous operations get canceled in `componentWillUnmount`), or you can change the test itself to be asynchronous.') : void 0;
	    // 创建个 event
  	  var evt = document.createEvent('Event');

      var didError = true;

      var windowEvent = window.event;

      var windowEventDescriptor = Object.getOwnPropertyDescriptor(window, 'event');

      var funcArgs = Array.prototype.slice.call(arguments, 3);
      function callCallback() {

        // 移除监听事件
        fakeNode.removeEventListener(evtType, callCallback, false);

        if (typeof window.event !== 'undefined' && window.hasOwnProperty('event')) {
          window.event = windowEvent;
        }

        func.apply(context, funcArgs);
        didError = false;
      }

      var error = void 0;
      var didSetError = false;
      var isCrossOriginError = false;

      function handleWindowError(event) {
        error = event.error;
        didSetError = true;
        if (error === null && event.colno === 0 && event.lineno === 0) {
          isCrossOriginError = true;
        }
        if (event.defaultPrevented) {
          if (error != null && typeof error === 'object') {
            try {
              error._suppressLogging = true;
            } catch (inner) {
            }
          }
        }
      }

      var evtType = 'react-' + (name ? name : 'invokeguardedcallback');

      window.addEventListener('error', handleWindowError);

    	// 添加监听事件
      fakeNode.addEventListener(evtType, callCallback, false);

    	// 初始化事件名
      evt.initEvent(evtType, false, false);

    	// 发出事件通知
      fakeNode.dispatchEvent(evt);

      if (windowEventDescriptor) {
        Object.defineProperty(window, 'event', windowEventDescriptor);
      }

    	...

      window.removeEventListener('error', handleWindowError);
    };


// 进入commitAllHostEffects
function commitAllHostEffects() {
  while (nextEffect !== null) {
    {
      setCurrentFiber(nextEffect);
    }
    recordEffect();

    var effectTag = nextEffect.effectTag;

    if (effectTag & ContentReset) {
      commitResetTextContent(nextEffect);
    }

    if (effectTag & Ref) {
      var current$$1 = nextEffect.alternate;
      if (current$$1 !== null) {
        commitDetachRef(current$$1);
      }
    }

    var primaryEffectTag = effectTag & (Placement | Update | Deletion);
    switch (primaryEffectTag) {
      case Placement:
        {
          commitPlacement(nextEffect);

          nextEffect.effectTag &= ~Placement;
          break;
        }
      case PlacementAndUpdate:
        {
          // 占位符
          commitPlacement(nextEffect);

          nextEffect.effectTag &= ~Placement;

          var _current = nextEffect.alternate;
          commitWork(_current, nextEffect);
          break;
        }
      case Update:
        {
          var _current2 = nextEffect.alternate;
          commitWork(_current2, nextEffect);
          break;
        }
      case Deletion:
        {
          commitDeletion(nextEffect);
          break;
        }
    }
    nextEffect = nextEffect.nextEffect;
  }

  {
    resetCurrentFiber();
  }
}


// 进入 commitPlacement 在这里做挂载
function commitPlacement(finishedWork) {
  if (!supportsMutation) {
    return;
  }

  var parentFiber = getHostParentFiber(finishedWork);

  var parent = void 0;
  var isContainer = void 0;

  switch (parentFiber.tag) {
    case HostComponent:
      parent = parentFiber.stateNode;
      isContainer = false;
      break;
    case HostRoot:
      parent = parentFiber.stateNode.containerInfo;
      isContainer = true;
      break;
    case HostPortal:
      parent = parentFiber.stateNode.containerInfo;
      isContainer = true;
      break;
    default:
      invariant(false, 'Invalid host parent fiber. This error is likely caused by a bug in React. Please file an issue.');
  }
  if (parentFiber.effectTag & ContentReset) {
    resetTextContent(parent);
    parentFiber.effectTag &= ~ContentReset;
  }

  var before = getHostSibling(finishedWork);
  var node = finishedWork;
  while (true) {
    if (node.tag === HostComponent || node.tag === HostText) {
      if (before) {
        if (isContainer) {
          insertInContainerBefore(parent, node.stateNode, before);
        } else {
          insertBefore(parent, node.stateNode, before);
        }
      } else {
        if (isContainer) {
          appendChildToContainer(parent, node.stateNode);
        } else {
          appendChild(parent, node.stateNode);
        }
      }
    } else if (node.tag === HostPortal) {
    } else if (node.child !== null) {
      node.child.return = node;
      node = node.child;
      continue;
    }
    if (node === finishedWork) {
      return;
    }
    while (node.sibling === null) {
      if (node.return === null || node.return === finishedWork) {
        return;
      }
      node = node.return;
    }
    node.sibling.return = node.return;
    node = node.sibling;
  }
}
```

进入在 renderRoot 方法里的 createWorkInProgress 方法后进入 createWorkInProgress 方法后，这里创建 FiberNode。而在 **workLoop** 方法创建完整的 **Fiber** 树的链接关系（主要是 beginWork 方法）。在 **completeWork** 方法里的 **createInstance**方法 生成 stateNode

```javascript
function createWorkInProgress(current, pendingProps, expirationTime) {
  var workInProgress = current.alternate
  if (workInProgress === null) {
    // 这里创建
    workInProgress = createFiber(
      current.tag,
      pendingProps,
      current.key,
      current.mode
    )
    workInProgress.elementType = current.elementType
    workInProgress.type = current.type
    workInProgress.stateNode = current.stateNode

    {
      workInProgress._debugID = current._debugID
      workInProgress._debugSource = current._debugSource
      workInProgress._debugOwner = current._debugOwner
    }

    workInProgress.alternate = current
    current.alternate = workInProgress
  } else {
    workInProgress.pendingProps = pendingProps

    workInProgress.effectTag = NoEffect

    workInProgress.nextEffect = null
    workInProgress.firstEffect = null
    workInProgress.lastEffect = null

    if (enableProfilerTimer) {
      workInProgress.actualDuration = 0
      workInProgress.actualStartTime = -1
    }
  }

  workInProgress.childExpirationTime = current.childExpirationTime
  workInProgress.expirationTime = current.expirationTime

  workInProgress.child = current.child
  workInProgress.memoizedProps = current.memoizedProps
  workInProgress.memoizedState = current.memoizedState
  workInProgress.updateQueue = current.updateQueue
  workInProgress.firstContextDependency = current.firstContextDependency

  workInProgress.sibling = current.sibling
  workInProgress.index = current.index
  workInProgress.ref = current.ref

  if (enableProfilerTimer) {
    workInProgress.selfBaseDuration = current.selfBaseDuration
    workInProgress.treeBaseDuration = current.treeBaseDuration
  }

  return workInProgress
}

// 进入createFiber
var createFiber = function(tag, pendingProps, key, mode) {
  return new FiberNode(tag, pendingProps, key, mode)
}

// completeWork
function completeWork(current, workInProgress, renderExpirationTime) {
  var newProps = workInProgress.pendingProps

  switch (workInProgress.tag) {
    case IndeterminateComponent:
      break
    case LazyComponent:
      break
    case SimpleMemoComponent:
    case FunctionComponent:
      break
    case ClassComponent: {
      var Component = workInProgress.type
      if (isContextProvider(Component)) {
        popContext(workInProgress)
      }
      break
    }
    case HostRoot: {
      popHostContainer(workInProgress)
      popTopLevelContextObject(workInProgress)
      var fiberRoot = workInProgress.stateNode
      if (fiberRoot.pendingContext) {
        fiberRoot.context = fiberRoot.pendingContext
        fiberRoot.pendingContext = null
      }
      if (current === null || current.child === null) {
        popHydrationState(workInProgress)
        workInProgress.effectTag &= ~Placement
      }
      updateHostContainer(workInProgress)
      break
    }
    case HostComponent: {
      popHostContext(workInProgress)
      var rootContainerInstance = getRootHostContainer()
      var type = workInProgress.type
      if (current !== null && workInProgress.stateNode != null) {
        updateHostComponent$1(
          current,
          workInProgress,
          type,
          newProps,
          rootContainerInstance
        )

        if (current.ref !== workInProgress.ref) {
          markRef$1(workInProgress)
        }
      } else {
        if (!newProps) {
          !(workInProgress.stateNode !== null)
            ? invariant(
                false,
                'We must have new props for new mounts. This error is likely caused by a bug in React. Please file an issue.'
              )
            : void 0
          break
        }

        var currentHostContext = getHostContext()
        var wasHydrated = popHydrationState(workInProgress)
        if (wasHydrated) {
          if (
            prepareToHydrateHostInstance(
              workInProgress,
              rootContainerInstance,
              currentHostContext
            )
          ) {
            markUpdate(workInProgress)
          }
        } else {
          var instance = createInstance(
            type,
            newProps,
            rootContainerInstance,
            currentHostContext,
            workInProgress
          )

          appendAllChildren(instance, workInProgress, false, false)

          if (
            finalizeInitialChildren(
              instance,
              type,
              newProps,
              rootContainerInstance,
              currentHostContext
            )
          ) {
            markUpdate(workInProgress)
          }
          workInProgress.stateNode = instance
        }

        if (workInProgress.ref !== null) {
          markRef$1(workInProgress)
        }
      }
      break
    }
    case HostText: {
      var newText = newProps
      if (current && workInProgress.stateNode != null) {
        var oldText = current.memoizedProps
        updateHostText$1(current, workInProgress, oldText, newText)
      } else {
        if (typeof newText !== 'string') {
          !(workInProgress.stateNode !== null)
            ? invariant(
                false,
                'We must have new props for new mounts. This error is likely caused by a bug in React. Please file an issue.'
              )
            : void 0
        }
        var _rootContainerInstance = getRootHostContainer()
        var _currentHostContext = getHostContext()
        var _wasHydrated = popHydrationState(workInProgress)
        if (_wasHydrated) {
          if (prepareToHydrateHostTextInstance(workInProgress)) {
            markUpdate(workInProgress)
          }
        } else {
          workInProgress.stateNode = createTextInstance(
            newText,
            _rootContainerInstance,
            _currentHostContext,
            workInProgress
          )
        }
      }
      break
    }
    case ForwardRef:
      break
    case SuspenseComponent: {
      var nextState = workInProgress.memoizedState
      if ((workInProgress.effectTag & DidCapture) !== NoEffect) {
        workInProgress.expirationTime = renderExpirationTime
        return workInProgress
      }

      var nextDidTimeout = nextState !== null
      var prevDidTimeout = current !== null && current.memoizedState !== null

      if (current !== null && !nextDidTimeout && prevDidTimeout) {
        var currentFallbackChild = current.child.sibling
        if (currentFallbackChild !== null) {
          var first = workInProgress.firstEffect
          if (first !== null) {
            workInProgress.firstEffect = currentFallbackChild
            currentFallbackChild.nextEffect = first
          } else {
            workInProgress.firstEffect = workInProgress.lastEffect = currentFallbackChild
            currentFallbackChild.nextEffect = null
          }
          currentFallbackChild.effectTag = Deletion
        }
      }

      if (
        nextDidTimeout !== prevDidTimeout ||
        ((workInProgress.effectTag & ConcurrentMode) === NoContext &&
          nextDidTimeout)
      ) {
        workInProgress.effectTag |= Update
      }
      break
    }
    case Fragment:
      break
    case Mode:
      break
    case Profiler:
      break
    case HostPortal:
      popHostContainer(workInProgress)
      updateHostContainer(workInProgress)
      break
    case ContextProvider:
      popProvider(workInProgress)
      break
    case ContextConsumer:
      break
    case MemoComponent:
      break
    case IncompleteClassComponent: {
      var _Component = workInProgress.type
      if (isContextProvider(_Component)) {
        popContext(workInProgress)
      }
      break
    }
    default:
      invariant(
        false,
        'Unknown unit of work tag. This error is likely caused by a bug in React. Please file an issue.'
      )
  }

  return null
}
```



## 补充

updatequeue

```javascript
function processUpdateQueue(workInProgress, queue, props, instance, renderExpirationTime) {
  hasForceUpdate = false;

  queue = ensureWorkInProgressQueueIsAClone(workInProgress, queue);

  {
    currentlyProcessingQueue = queue;
  }

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
```

看起来好像是合并多次操作的地方，到 setState 的地方再来看



## 小结

- createElement 生成 ReactComponent
- 进入 React.render 方法，创建 ReactRoot，它的 current 是 RootFiber，containerInfo 是模板插槽的引用
- 创建 updateQueue，单向链表，包含需要更新的
- 任务调度阶段
- 从 RootFiber 开始生成 Fiber 树，遇到组件会调用 render 方法生成新的 ReactComponent（从上到下）
- 当完成 Fiber 树结构之后，开始生成实例（stateNode）（从下到上，儿子实例被添加到父亲实例里）
- 提交阶段
- 把 DomElement 实例（就是stateNode）添加到模板插槽的位置



## 参考资料

[Document.createEvent()](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/createEvent)

语法：

```javascript
var event = document.createEvent(type)
```

- `event` 就是被创建的 [Event](https://developer.mozilla.org/zh-CN/docs/DOM/event) 对象.
- `type` 是一个字符串，表示要创建的事件类型。事件类型可能包括`"UIEvents"`, `"MouseEvents"`, `"MutationEvents"`, 或者 `"HTMLEvents"`。请查看 [Notes](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/createEvent#Notes) 章节获取详细信息 。

[Event.initEvent()](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/initEvent)

用来初始化由[`Document.createEvent()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/createEvent) 创建的 [`event`](https://developer.mozilla.org/zh-CN/docs/Web/API/Event) 实例

语法：

```javascript
event.initEvent(type, bubbles, cancelable)
```

- `type`一个 [`DOMString`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMString) 类型的字段，定义了事件的类型.

- `bubbles`一个 [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean) 值，决定是否事件是否应该向上冒泡. 一旦设置了这个值，只读属性[`Event.bubbles`](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/bubbles)也会获取相应的值.

- `cancelable`一个 [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean) 值，决定该事件的默认动作是否可以被取消. 一旦设置了这个值, 只读属性 [`Event.cancelable`](https://developer.mozilla.org/zh-CN/docs/Web/API/Event/cancelable) 也会获取相应的值.

[EventTarget.dispatchEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/EventTarget/dispatchEvent)

向一个指定的事件目标派发一个[事件](https://developer.mozilla.org/zh-CN/docs/Web/API/Event), 并以合适的顺序**同步调用**目标元素相关的事件处理函数。标准事件处理规则(包括事件捕获和可选的冒泡过程)同样适用于通过手动的使用`dispatchEvent()`方法派发的事件。

语法：

```javascript
target.dispatchEvent(event)
```

- `event` 是要被派发的事件对象。
- `target` 被用来初始化 事件 和 决定将会触发 目标

整合上面 3 个的例子

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>event</title>
  </head>
  <body>
    <button onclick="dispatch()">dispatch</button>
  </body>
  <script>
    function dispatch() {
      const evtType = 'eventName'
      var fakeNode = document.createElement('react')
      fakeNode.addEventListener(
        evtType,
        e => {
          console.log(e)
        },
        false
      )

      var evt = document.createEvent('Event')
      evt.initEvent(evtType, false, false)

      fakeNode.dispatchEvent(evt)
    }
  </script>
</html>
```

[剖析 React 源码：render 流程（一）](https://juejin.im/post/5cca5ad2e51d456e6154b4c7)
