---
title: react源码过程（3）- 更新
date: 2019-10-31 16:45:15
tags: 
- react源码过程
categories: 
- react源码过程
---

## 创建更新方式

##### **ReactDom.render**

```javascript
render(
  element: React$Element<any>,//ReactElement
  container: DOMContainer,//要挂载的节点
  callback: ?Function,//挂载之后调用的方法
) {
  return legacyRenderSubtreeIntoContainer(
    null,
    element,//ReactElement
    container,//要挂载的节点
    false,
    callback,//挂载之后调用的方法
  );
},
```

legacyRenderSubtreeIntoContainer

```javascript
function legacyRenderSubtreeIntoContainer(
  parentComponent: ?React$Component<any, any>,//null
  children: ReactNodeList,//ReactElement
  container: DOMContainer,//要挂载的节点
  forceHydrate: boolean,//false
  callback: ?Function,//挂载之后调用的方法
) {
  invariant(
    isValidContainer(container),
    'Target container is not a DOM element.',
  );
	
	...
  
  let root: Root = (container._reactRootContainer: any);//第一次渲染不存在_reactRootContainer属性
  if (!root) {// 不存在走进这里
    //创建一个root，
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      container,//要挂载的节点
      forceHydrate,//false
    );
    if (typeof callback === 'function') {// 如果有传入callback
      const originalCallback = callback;//保存callback
      callback = function() {//封装了一下最后调用到被保存的callback
        const instance = DOMRenderer.getPublicRootInstance(root._internalRoot);
        originalCallback.call(instance);
      };
    }
    DOMRenderer.unbatchedUpdates(() => {//更新
      if (parentComponent != null) {
        root.legacy_renderSubtreeIntoContainer(
          parentComponent,
          children,
          callback,
        );
      } else {
        root.render(children, callback);
      }
    });
  } else {//第一次渲染没走这边
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = DOMRenderer.getPublicRootInstance(root._internalRoot);
        originalCallback.call(instance);
      };
    }
    if (parentComponent != null) {
      root.legacy_renderSubtreeIntoContainer(
        parentComponent,
        children,
        callback,
      );
    } else {
      root.render(children, callback);
    }
  }
  return DOMRenderer.getPublicRootInstance(root._internalRoot);
}
```

- 生成root

legacyCreateRootFromDOMContainer

```javascript
function legacyCreateRootFromDOMContainer(
  container: DOMContainer,//要挂载的节点
  forceHydrate: boolean,//false
): Root {
  const shouldHydrate =
    forceHydrate || shouldHydrateDueToLegacyHeuristic(container);//是否需要调和

  if (!shouldHydrate) {//不需要的话
    let warned = false;
    let rootSibling;
    while ((rootSibling = container.lastChild)) {//循环
     
      ...
      
      container.removeChild(rootSibling);//删除子节点
    }
  }
 
  ...  
    
  const isConcurrent = false;
  return new ReactRoot(
    container,//要挂载的节点
    isConcurrent, // false
    shouldHydrate// false
  );
}
```

ReactRoot

```javascript
function ReactRoot(
  container: Container,//要挂载的节点
  isConcurrent: boolean,// false
  hydrate: boolean,// false
) {
  const root = DOMRenderer.createContainer(container, isConcurrent, hydrate);
  this._internalRoot = root;
}
```

createContainer

```javascript
export function createContainer(
  containerInfo: Container,//要挂载的节点
  isConcurrent: boolean,// false
  hydrate: boolean,// false
): OpaqueRoot {
  return createFiberRoot(containerInfo, isConcurrent, hydrate);
}
```

createFiberRoot

```javascript
export function createFiberRoot(
  containerInfo: any,//要挂载的节点
  isConcurrent: boolean,// false
  hydrate: boolean,// false
): FiberRoot {
  const uninitializedFiber = createHostRootFiber(isConcurrent);

  let root;
  if (enableSchedulerTracing) {
    root = ({
      current: uninitializedFiber,//Fiber对象
      containerInfo: containerInfo,//要挂载的节点
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
      hydrate,
      nextExpirationTimeToWorkOn: NoWork,
      expirationTime: NoWork,
      firstBatch: null,
      nextScheduledRoot: null,

      interactionThreadID: unstable_getThreadID(),
      memoizedInteractions: new Set(),
      pendingInteractionMap: new Map(),
    }: FiberRoot);
  } else {
    root = ({
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
      hydrate,
      nextExpirationTimeToWorkOn: NoWork,
      expirationTime: NoWork,
      firstBatch: null,
      nextScheduledRoot: null,
    }: BaseFiberRootProperties);
  }

  uninitializedFiber.stateNode = root;

  return ((root: any): FiberRoot);
}
```

- 调用  root.render(children, callback);

```javascript
ReactRoot.prototype.render = function(
  children: ReactNodeList,
  callback: ?() => mixed,
): Work {
  const root = this._internalRoot;
  const work = new ReactWork();
  callback = callback === undefined ? null : callback;
	
  ...
  
  if (callback !== null) {
    work.then(callback);
  }
  DOMRenderer.updateContainer(
    children,//ReactElement
    root,
    null,
    work._onCommit);
  return work;
};
```

updateContainer

```javascript
export function updateContainer(
  element: ReactNodeList,
  container: OpaqueRoot,
  parentComponent: ?React$Component<any, any>,
  callback: ?Function,
): ExpirationTime {
  const current = container.current;
  const currentTime = requestCurrentTime();
  const expirationTime = computeExpirationForFiber(currentTime, current);//计算expirationTime，优先级任务更新
  return updateContainerAtExpirationTime(
    element,
    container,
    parentComponent,
    expirationTime,
    callback,
  );
}
```

requestCurrentTime

```javascript
function requestCurrentTime() {

  if (isRendering) {

    return currentSchedulerTime;
  }

  findHighestPriorityRoot();//调度队列中找到权限最高的root
  if (
    nextFlushedExpirationTime === NoWork ||
    nextFlushedExpirationTime === Never
  ) {
    recomputeCurrentRendererTime();
    currentSchedulerTime = currentRendererTime;
    return currentSchedulerTime;
  }
  return currentSchedulerTime;
}
```

recomputeCurrentRendererTime

```javascript
function recomputeCurrentRendererTime() {
  const currentTimeMs = now() - originalStartTimeMs;
  currentRendererTime = msToExpirationTime(currentTimeMs);
}
```

computeExpirationForFiber

```javascript
function computeExpirationForFiber(currentTime: ExpirationTime, fiber: Fiber) {
  let expirationTime;
  if (expirationContext !== NoWork) {
    expirationTime = expirationContext;
  } else if (isWorking) {
    if (isCommitting) {
      expirationTime = Sync;
    } else {
      expirationTime = nextRenderExpirationTime;
    }
  } else {
    if (fiber.mode & ConcurrentMode) {
      if (isBatchingInteractiveUpdates) {
        expirationTime = computeInteractiveExpiration(currentTime);
      } else {
        expirationTime = computeAsyncExpiration(currentTime);
      }
      if (nextRoot !== null && expirationTime === nextRenderExpirationTime) {
        expirationTime -= 1;
      }
    } else {
      expirationTime = Sync;
    }
  }
  if (isBatchingInteractiveUpdates) {
    if (
      lowestPriorityPendingInteractiveExpirationTime === NoWork ||
      expirationTime < lowestPriorityPendingInteractiveExpirationTime
    ) {
      lowestPriorityPendingInteractiveExpirationTime = expirationTime;
    }
  }
  return expirationTime;
}
```

updateContainerAtExpirationTime

```javascript
export function updateContainerAtExpirationTime(
  element: ReactNodeList,
  container: OpaqueRoot,
  parentComponent: ?React$Component<any, any>,
  expirationTime: ExpirationTime,
  callback: ?Function,
) {
  const current = container.current;

  ...

  const context = getContextForSubtree(parentComponent);
  if (container.context === null) {
    container.context = context;
  } else {
    container.pendingContext = context;
  }

  return scheduleRootUpdate(current, element, expirationTime, callback);
}
```

scheduleRootUpdate

```javascript
function scheduleRootUpdate(
  current: Fiber,
  element: ReactNodeList,
  expirationTime: ExpirationTime,
  callback: ?Function,
) {
	...

  const update = createUpdate(expirationTime);//需要更新的地方

  update.payload = {element};//设置属性

  callback = callback === undefined ? null : callback;
  if (callback !== null) {
    warningWithoutStack(
      typeof callback === 'function',
      'render(...): Expected the last optional `callback` argument to be a ' +
        'function. Instead received: %s.',
      callback,
    );
    update.callback = callback;
  }

  flushPassiveEffects();
  enqueueUpdate(current, update);//把update添加到queue里面
  scheduleWork(current, expirationTime);//任务调用，说明有更新产生了

  return expirationTime;
}
```

createUpdate

```javascript
export function createUpdate(expirationTime: ExpirationTime): Update<*> {
  return {
    expirationTime: expirationTime,

    tag: UpdateState,
    payload: null,
    callback: null,

    next: null,
    nextEffect: null,
  };
}
```

enqueueUpdate

```javascript
export function enqueueUpdate<State>(fiber: Fiber, update: Update<State>) {

  const alternate = fiber.alternate;
  let queue1;
  let queue2;
  if (alternate === null) {//未渲染过

    queue1 = fiber.updateQueue;
    queue2 = null;
    if (queue1 === null) {
      queue1 = fiber.updateQueue = createUpdateQueue(fiber.memoizedState);//创建一个队列
    }
  } else { //渲染过

    queue1 = fiber.updateQueue;
    queue2 = alternate.updateQueue;
    if (queue1 === null) {
      if (queue2 === null) {

        queue1 = fiber.updateQueue = createUpdateQueue(fiber.memoizedState);
        queue2 = alternate.updateQueue = createUpdateQueue(
          alternate.memoizedState,
        );
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

	...
}
```

createUpdateQueue

```javascript
export function createUpdateQueue<State>(baseState: State): UpdateQueue<State> {
  const queue: UpdateQueue<State> = {
    baseState,
    firstUpdate: null,
    lastUpdate: null,
    firstCapturedUpdate: null,
    lastCapturedUpdate: null,
    firstEffect: null,
    lastEffect: null,
    firstCapturedEffect: null,
    lastCapturedEffect: null,
  };
  return queue;
}
```

appendUpdateToQueue

```javascript
function appendUpdateToQueue<State>(
  queue: UpdateQueue<State>,
  update: Update<State>,
) {

  if (queue.lastUpdate === null) {

    queue.firstUpdate = queue.lastUpdate = update;
  } else {
    queue.lastUpdate.next = update;
    queue.lastUpdate = update;
  }
}
```



##### **setState**

```javascript
Component.prototype.setState = function(partialState, callback) {
  invariant(
    typeof partialState === 'object' ||
      typeof partialState === 'function' ||
      partialState == null,
    'setState(...): takes an object of state variables to update or a ' +
      'function which returns an object of state variables.',
  );//判断partialState类型
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};
```

enqueueSetState

```javascript
  enqueueSetState(inst, payload, callback) {
    const fiber = ReactInstanceMap.get(inst);
    const currentTime = requestCurrentTime();
    const expirationTime = computeExpirationForFiber(currentTime, fiber);

    const update = createUpdate(expirationTime);
    update.payload = payload;
    if (callback !== undefined && callback !== null) {
      if (__DEV__) {
        warnOnInvalidCallback(callback, 'setState');
      }
      update.callback = callback;
    }

    flushPassiveEffects();
    enqueueUpdate(fiber, update);
    scheduleWork(fiber, expirationTime);
  },
```



##### **forceUpdate**

```javascript
  enqueueForceUpdate(inst, callback) {
    const fiber = ReactInstanceMap.get(inst);
    const currentTime = requestCurrentTime();
    const expirationTime = computeExpirationForFiber(currentTime, fiber);

    const update = createUpdate(expirationTime);
    update.tag = ForceUpdate;

    if (callback !== undefined && callback !== null) {
      if (__DEV__) {
        warnOnInvalidCallback(callback, 'forceUpdate');
      }
      update.callback = callback;
    }

    flushPassiveEffects();
    enqueueUpdate(fiber, update);
    scheduleWork(fiber, expirationTime);
  }
```



## 补充



**什么是FiberRoot**

1. 整个应用的起点
2. 包含应用挂载的目标节点
3. 记录整个应用更新过程的各种信息



**什么是Fiber**

1. 每一个ReactElement对应一个Fiber对象
2. 记录节点的各种状态
3. 串联整个应用形成树结构



**什么是Update**

1. 用于记录组件状态的改变
2. 存放于UpdateQueue中
3. 多个Update可以同时存在

 