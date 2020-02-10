---
title: react源码过程（4）- 任务调度
date: 2019-11-03 11:51:39
tags:
  - react源码过程
categories:
  - react源码过程
---

## 任务调度

##### scheduleWork

- 找到更新对应的 FiberRoot 节点

- 如果符合条件重置 stack

- 如果符合条件就请求工作调度

![WX20191103-121127@2x](http://114.55.30.96/WX20191103-121127@2x.png)

```javascript
function scheduleWork(
   fiber: Fiber, //Fiber对象，
   expirationTime: ExpirationTime//创建更新时候计算的过期时间
) {
  const root = scheduleWorkToRoot(fiber, expirationTime);
  if (root === null) {

    ...

    return;
  }

  if (
    !isWorking &&
    nextRenderExpirationTime !== NoWork &&
    expirationTime > nextRenderExpirationTime
  ) {

    interruptedBy = fiber;
    resetStack();//
  }
  markPendingPriorityLevel(root, expirationTime);
  if (
    !isWorking ||
    isCommitting ||

    nextRoot !== root
  ) {
    const rootExpirationTime = root.expirationTime;
    requestWork(root, rootExpirationTime);//
  }
  if (nestedUpdateCount > NESTED_UPDATE_LIMIT) {

    nestedUpdateCount = 0;
    invariant(
      false,
      'Maximum update depth exceeded. This can happen when a ' +
        'component repeatedly calls setState inside ' +
        'componentWillUpdate or componentDidUpdate. React limits ' +
        'the number of nested updates to prevent infinite loops.',
    );
  }
}
```

scheduleWorkToRoot

```javascript
function scheduleWorkToRoot(fiber: Fiber, expirationTime): FiberRoot | null {//根据传入的fiber节点找到rootFiber对象

  recordScheduleUpdate();

  ...

  if (fiber.expirationTime < expirationTime) {//更新expirationTime
    fiber.expirationTime = expirationTime;
  }
  let alternate = fiber.alternate;
  if (alternate !== null && alternate.expirationTime < expirationTime) {//更新expirationTime
    alternate.expirationTime = expirationTime;
  }

  let node = fiber.return;//得到它的上级节点
  let root = null;
  if (node === null && fiber.tag === HostRoot) {//如果node为null说明是RootFiber
    root = fiber.stateNode;
  } else {
    while (node !== null) {//循环更新expirationTime，直到node为RootFiber
      alternate = node.alternate;
      if (node.childExpirationTime < expirationTime) {
        node.childExpirationTime = expirationTime;
        if (
          alternate !== null &&
          alternate.childExpirationTime < expirationTime
        ) {
          alternate.childExpirationTime = expirationTime;
        }
      } else if (
        alternate !== null &&
        alternate.childExpirationTime < expirationTime
      ) {
        alternate.childExpirationTime = expirationTime;
      }
      if (node.return === null && node.tag === HostRoot) {
        root = node.stateNode;
        break;
      }
      node = node.return;
    }
  }

  if (enableSchedulerTracing) {
    if (root !== null) {
      const interactions = __interactionsRef.current;
      if (interactions.size > 0) {
        const pendingInteractionMap = root.pendingInteractionMap;
        const pendingInteractions = pendingInteractionMap.get(expirationTime);
        if (pendingInteractions != null) {
          interactions.forEach(interaction => {
            if (!pendingInteractions.has(interaction)) {
              interaction.__count++;
            }

            pendingInteractions.add(interaction);
          });
        } else {
          pendingInteractionMap.set(expirationTime, new Set(interactions));


          interactions.forEach(interaction => {
            interaction.__count++;
          });
        }

        const subscriber = __subscriberRef.current;
        if (subscriber !== null) {
          const threadID = computeThreadID(
            expirationTime,
            root.interactionThreadID,
          );
          subscriber.onWorkScheduled(interactions, threadID);
        }
      }
    }
  }
  return root;
}
```

resetStack

```javascript
function resetStack() {
  if (nextUnitOfWork !== null) {
    let interruptedWork = nextUnitOfWork.return;
    while (interruptedWork !== null) {
      unwindInterruptedWork(interruptedWork);
      interruptedWork = interruptedWork.return;
    }
  }

  ...

  nextRoot = null;
  nextRenderExpirationTime = NoWork;
  nextLatestAbsoluteTimeoutMs = -1;
  nextRenderDidError = false;
  nextUnitOfWork = null;
}
```

##### requestWork

- 加入到 root 调度队列
- 判断是否批量更新
- 根据 expirationTime 判断调度类型

```javascript
function requestWork(root: FiberRoot, expirationTime: ExpirationTime) {
  addRootToSchedule(root, expirationTime)
  if (isRendering) {
    return
  }

  if (isBatchingUpdates) {
    if (isUnbatchingUpdates) {
      nextFlushedRoot = root
      nextFlushedExpirationTime = Sync
      performWorkOnRoot(root, Sync, false)
    }
    return
  }

  if (expirationTime === Sync) {
    performSyncWork() //同步调用一直执行不会被打断
  } else {
    scheduleCallbackWithExpirationTime(root, expirationTime) //异步调度，在浏览器有空闲的时候执行一些任务，并且设置了deadline，把主动权交给浏览器，有空闲时候再来执行
  }
}
```

addRootToSchedule

```javascript
function addRootToSchedule(root: FiberRoot, expirationTime: ExpirationTime) {
  if (root.nextScheduledRoot === null) {
    root.expirationTime = expirationTime
    if (lastScheduledRoot === null) {
      firstScheduledRoot = lastScheduledRoot = root
      root.nextScheduledRoot = root
    } else {
      lastScheduledRoot.nextScheduledRoot = root
      lastScheduledRoot = root
      lastScheduledRoot.nextScheduledRoot = firstScheduledRoot
    }
  } else {
    const remainingExpirationTime = root.expirationTime
    if (expirationTime > remainingExpirationTime) {
      root.expirationTime = expirationTime
    }
  }
}
```

##### batchedUpdates

```javascript
function batchedUpdates<A, R>(fn: (a: A) => R, a: A): R {
  const previousIsBatchingUpdates = isBatchingUpdates //记录之前的值
  isBatchingUpdates = true
  try {
    return fn(a)
  } finally {
    isBatchingUpdates = previousIsBatchingUpdates //设置为之前记录的值
    if (!isBatchingUpdates && !isRendering) {
      performSyncWork() //更新
    }
  }
}
```

##### React Scheduler

- 维护时间片
- 模拟 requestIdelCallback（requestIdelCallback 优先级比 requestAinimationFrame 低）
- 调度列表和超时判断

**performWork**

- 是否有 deadline 的区别
- 循环渲染 Root 的条件
- 超过时间片的处理

```javascript
function performAsyncWork() {
  try {
    if (!shouldYieldToRenderer()) {
      if (firstScheduledRoot !== null) {
        recomputeCurrentRendererTime()
        let root: FiberRoot = firstScheduledRoot
        do {
          didExpireAtExpirationTime(root, currentRendererTime)

          root = (root.nextScheduledRoot: any)
        } while (root !== firstScheduledRoot)
      }
    }
    performWork(NoWork, true)
  } finally {
    didYield = false
  }
}

function performSyncWork() {
  performWork(Sync, false)
}

function performWork(minExpirationTime: ExpirationTime, isYieldy: boolean) {
  findHighestPriorityRoot()

  if (isYieldy) {
    recomputeCurrentRendererTime()
    currentSchedulerTime = currentRendererTime

    if (enableUserTimingAPI) {
      const didExpire = nextFlushedExpirationTime > currentRendererTime
      const timeout = expirationTimeToMs(nextFlushedExpirationTime)
      stopRequestCallbackTimer(didExpire, timeout)
    }

    while (
      nextFlushedRoot !== null &&
      nextFlushedExpirationTime !== NoWork &&
      minExpirationTime <= nextFlushedExpirationTime &&
      !(didYield && currentRendererTime > nextFlushedExpirationTime)
    ) {
      performWorkOnRoot(
        nextFlushedRoot,
        nextFlushedExpirationTime,
        currentRendererTime > nextFlushedExpirationTime
      )
      findHighestPriorityRoot()
      recomputeCurrentRendererTime()
      currentSchedulerTime = currentRendererTime
    }
  } else {
    while (
      nextFlushedRoot !== null &&
      nextFlushedExpirationTime !== NoWork &&
      minExpirationTime <= nextFlushedExpirationTime
    ) {
      performWorkOnRoot(nextFlushedRoot, nextFlushedExpirationTime, false)
      findHighestPriorityRoot()
    }
  }

  if (isYieldy) {
    callbackExpirationTime = NoWork
    callbackID = null
  }

  if (nextFlushedExpirationTime !== NoWork) {
    scheduleCallbackWithExpirationTime(
      ((nextFlushedRoot: any): FiberRoot),
      nextFlushedExpirationTime
    )
  }

  finishRendering()
}
```

performWorkOnRoot

```javascript
function performWorkOnRoot(
  root: FiberRoot,
  expirationTime: ExpirationTime,
  isYieldy: boolean
) {
  invariant(
    !isRendering,
    'performWorkOnRoot was called recursively. This error is likely caused ' +
      'by a bug in React. Please file an issue.'
  )

  isRendering = true

  if (!isYieldy) {
    let finishedWork = root.finishedWork
    if (finishedWork !== null) {
      completeRoot(root, finishedWork, expirationTime)
    } else {
      root.finishedWork = null
      const timeoutHandle = root.timeoutHandle
      if (timeoutHandle !== noTimeout) {
        root.timeoutHandle = noTimeout

        cancelTimeout(timeoutHandle)
      }
      renderRoot(root, isYieldy)
      finishedWork = root.finishedWork
      if (finishedWork !== null) {
        completeRoot(root, finishedWork, expirationTime)
      }
    }
  } else {
    let finishedWork = root.finishedWork
    if (finishedWork !== null) {
      completeRoot(root, finishedWork, expirationTime)
    } else {
      root.finishedWork = null
      const timeoutHandle = root.timeoutHandle
      if (timeoutHandle !== noTimeout) {
        root.timeoutHandle = noTimeout

        cancelTimeout(timeoutHandle)
      }
      renderRoot(root, isYieldy)
      finishedWork = root.finishedWork
      if (finishedWork !== null) {
        if (!shouldYieldToRenderer()) {
          completeRoot(root, finishedWork, expirationTime)
        } else {
          root.finishedWork = finishedWork
        }
      }
    }
  }

  isRendering = false
}
```

##### renderRoot

- 调用 workLoop 进行循环单元更新
- 捕获错误并进行处理

```javascript
function renderRoot(root: FiberRoot, isYieldy: boolean): void {

  ...

  flushPassiveEffects();

  isWorking = true;
  if (enableHooks) {
    ReactCurrentOwner.currentDispatcher = Dispatcher;
  } else {
    ReactCurrentOwner.currentDispatcher = DispatcherWithoutHooks;
  }

  const expirationTime = root.nextExpirationTimeToWorkOn;

  if (
    expirationTime !== nextRenderExpirationTime ||
    root !== nextRoot ||
    nextUnitOfWork === null
  ) {

    resetStack();
    nextRoot = root;
    nextRenderExpirationTime = expirationTime;
    nextUnitOfWork = createWorkInProgress(//当前的节点拷贝了一份
      nextRoot.current,
      null,
      nextRenderExpirationTime,
    );
    root.pendingCommitExpirationTime = NoWork;

    if (enableSchedulerTracing) {
      const interactions: Set<Interaction> = new Set();
      root.pendingInteractionMap.forEach(
        (scheduledInteractions, scheduledExpirationTime) => {
          if (scheduledExpirationTime >= expirationTime) {
            scheduledInteractions.forEach(interaction =>
              interactions.add(interaction),
            );
          }
        },
      );

      root.memoizedInteractions = interactions;

      if (interactions.size > 0) {
        const subscriber = __subscriberRef.current;
        if (subscriber !== null) {
          const threadID = computeThreadID(
            expirationTime,
            root.interactionThreadID,
          );
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

  let prevInteractions: Set<Interaction> = (null: any);
  if (enableSchedulerTracing) {
    prevInteractions = __interactionsRef.current;
    __interactionsRef.current = root.memoizedInteractions;
  }

  let didFatal = false;

  startWorkLoopTimer(nextUnitOfWork);

  do {
    try {
      workLoop(isYieldy);//执行每个节点更新
    } catch (thrownValue) {
      resetContextDependences();
      resetHooks();

      let mayReplay;

      ...

      if (nextUnitOfWork === null) {
        didFatal = true;
        onUncaughtError(thrownValue);
      } else {
        if (enableProfilerTimer && nextUnitOfWork.mode & ProfileMode) {
          stopProfilerTimerIfRunningAndRecordDelta(nextUnitOfWork, true);
        }

        ...

        invariant(
          nextUnitOfWork !== null,
          'Failed to replay rendering after an error. This ' +
            'is likely caused by a bug in React. Please file an issue ' +
            'with a reproducing case to help us find it.',
        );

        const sourceFiber: Fiber = nextUnitOfWork;
        let returnFiber = sourceFiber.return;
        if (returnFiber === null) {
          didFatal = true;
          onUncaughtError(thrownValue);
        } else {
          throwException(
            root,
            returnFiber,
            sourceFiber,
            thrownValue,
            nextRenderExpirationTime,
          );
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
  ReactCurrentOwner.currentDispatcher = null;
  resetContextDependences();
  resetHooks();

  if (didFatal) {
    const didCompleteRoot = false;
    stopWorkLoopTimer(interruptedBy, didCompleteRoot);
    interruptedBy = null;

    ...

    nextRoot = null;
    onFatal(root);
    return;
  }

  if (nextUnitOfWork !== null) {
    const didCompleteRoot = false;
    stopWorkLoopTimer(interruptedBy, didCompleteRoot);
    interruptedBy = null;
    onYield(root);
    return;
  }

  const didCompleteRoot = true;
  stopWorkLoopTimer(interruptedBy, didCompleteRoot);
  const rootWorkInProgress = root.current.alternate;
  invariant(
    rootWorkInProgress !== null,
    'Finished root should have a work-in-progress. This error is likely ' +
      'caused by a bug in React. Please file an issue.',
  );

  nextRoot = null;
  interruptedBy = null;

  if (nextRenderDidError) {
    if (hasLowerPriorityWork(root, expirationTime)) {
      markSuspendedPriorityLevel(root, expirationTime);
      const suspendedExpirationTime = expirationTime;
      const rootExpirationTime = root.expirationTime;
      onSuspend(
        root,
        rootWorkInProgress,
        suspendedExpirationTime,
        rootExpirationTime,
        -1, // Indicates no timeout
      );
      return;
    } else if (
      !root.didError &&
      isYieldy
    ) {
      root.didError = true;
      const suspendedExpirationTime = (root.nextExpirationTimeToWorkOn = expirationTime);
      const rootExpirationTime = (root.expirationTime = Sync);
      onSuspend(
        root,
        rootWorkInProgress,
        suspendedExpirationTime,
        rootExpirationTime,
        -1, // Indicates no timeout
      );
      return;
    }
  }

  if (isYieldy && nextLatestAbsoluteTimeoutMs !== -1) {
    const suspendedExpirationTime = expirationTime;
    markSuspendedPriorityLevel(root, suspendedExpirationTime);

    const earliestExpirationTime = findEarliestOutstandingPriorityLevel(
      root,
      expirationTime,
    );
    const earliestExpirationTimeMs = expirationTimeToMs(earliestExpirationTime);
    if (earliestExpirationTimeMs < nextLatestAbsoluteTimeoutMs) {
      nextLatestAbsoluteTimeoutMs = earliestExpirationTimeMs;
    }

    const currentTimeMs = expirationTimeToMs(requestCurrentTime());
    let msUntilTimeout = nextLatestAbsoluteTimeoutMs - currentTimeMs;
    msUntilTimeout = msUntilTimeout < 0 ? 0 : msUntilTimeout;


    const rootExpirationTime = root.expirationTime;
    onSuspend(
      root,
      rootWorkInProgress,
      suspendedExpirationTime,
      rootExpirationTime,
      msUntilTimeout,
    );
    return;
  }

  onComplete(root, rootWorkInProgress, expirationTime);
}
```

workLoop

```javascript
function workLoop(isYieldy) {
  if (!isYieldy) {
    while (nextUnitOfWork !== null) {
      nextUnitOfWork = performUnitOfWork(nextUnitOfWork)
    }
  } else {
    while (nextUnitOfWork !== null && !shouldYieldToRenderer()) {
      nextUnitOfWork = performUnitOfWork(nextUnitOfWork)
    }
  }
}
```

performUnitOfWork

```javascript
function performUnitOfWork(workInProgress: Fiber): Fiber | null {
  const current = workInProgress.alternate;
  startWorkTimer(workInProgress);

  ...

  let next;
  if (enableProfilerTimer) {
    if (workInProgress.mode & ProfileMode) {
      startProfilerTimer(workInProgress);
    }

    next = beginWork(current, workInProgress, nextRenderExpirationTime);//每个节点的更新完返回下一个节点
    workInProgress.memoizedProps = workInProgress.pendingProps;

    if (workInProgress.mode & ProfileMode) {
      stopProfilerTimerIfRunningAndRecordDelta(workInProgress, true);
    }
  } else {
    next = beginWork(current, workInProgress, nextRenderExpirationTime);
    workInProgress.memoizedProps = workInProgress.pendingProps;
  }

	...

  if (next === null) {//说明更新完了当前层级的全部
    next = completeUnitOfWork(workInProgress);//具体更新节点渲染到DOM里
  }
  ReactCurrentOwner.current = null;
  return next;
}
```

beginWork

```javascript
function beginWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderExpirationTime: ExpirationTime
): Fiber | null {
  const updateExpirationTime = workInProgress.expirationTime //Fiber节点上的expirationTime

  if (current !== null) {
    const oldProps = current.memoizedProps
    const newProps = workInProgress.pendingProps
    if (
      oldProps === newProps &&
      !hasLegacyContextChanged() &&
      updateExpirationTime < renderExpirationTime
    ) {
      //无更新或者优先级不高
      switch (workInProgress.tag) {
        case HostRoot:
          pushHostRootContext(workInProgress)
          resetHydrationState()
          break
        case HostComponent:
          pushHostContext(workInProgress)
          break
        case ClassComponent: {
          const Component = workInProgress.type
          if (isLegacyContextProvider(Component)) {
            pushLegacyContextProvider(workInProgress)
          }
          break
        }
        case HostPortal:
          pushHostContainer(
            workInProgress,
            workInProgress.stateNode.containerInfo
          )
          break
        case ContextProvider: {
          const newValue = workInProgress.memoizedProps.value
          pushProvider(workInProgress, newValue)
          break
        }
        case Profiler:
          if (enableProfilerTimer) {
            workInProgress.effectTag |= Update
          }
          break
        case SuspenseComponent: {
          const state: SuspenseState | null = workInProgress.memoizedState
          const didTimeout = state !== null
          if (didTimeout) {
            const primaryChildFragment: Fiber = (workInProgress.child: any)
            const primaryChildExpirationTime =
              primaryChildFragment.childExpirationTime
            if (
              primaryChildExpirationTime !== NoWork &&
              primaryChildExpirationTime >= renderExpirationTime
            ) {
              return updateSuspenseComponent(
                current,
                workInProgress,
                renderExpirationTime
              )
            } else {
              const child = bailoutOnAlreadyFinishedWork(
                current,
                workInProgress,
                renderExpirationTime
              )
              if (child !== null) {
                return child.sibling
              } else {
                return null
              }
            }
          }
          break
        }
      }
      return bailoutOnAlreadyFinishedWork(
        current,
        workInProgress,
        renderExpirationTime
      )
    }
  }

  workInProgress.expirationTime = NoWork

  switch (workInProgress.tag) {
    case IndeterminateComponent: {
      const elementType = workInProgress.elementType
      return mountIndeterminateComponent(
        current,
        workInProgress,
        elementType,
        renderExpirationTime
      )
    }
    case LazyComponent: {
      const elementType = workInProgress.elementType
      return mountLazyComponent(
        current,
        workInProgress,
        elementType,
        updateExpirationTime,
        renderExpirationTime
      )
    }
    case FunctionComponent: {
      const Component = workInProgress.type
      const unresolvedProps = workInProgress.pendingProps
      const resolvedProps =
        workInProgress.elementType === Component
          ? unresolvedProps
          : resolveDefaultProps(Component, unresolvedProps)
      return updateFunctionComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderExpirationTime
      )
    }
    case ClassComponent: {
      const Component = workInProgress.type
      const unresolvedProps = workInProgress.pendingProps
      const resolvedProps =
        workInProgress.elementType === Component
          ? unresolvedProps
          : resolveDefaultProps(Component, unresolvedProps)
      return updateClassComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderExpirationTime
      )
    }
    case HostRoot:
      return updateHostRoot(current, workInProgress, renderExpirationTime)
    case HostComponent:
      return updateHostComponent(current, workInProgress, renderExpirationTime)
    case HostText:
      return updateHostText(current, workInProgress)
    case SuspenseComponent:
      return updateSuspenseComponent(
        current,
        workInProgress,
        renderExpirationTime
      )
    case HostPortal:
      return updatePortalComponent(
        current,
        workInProgress,
        renderExpirationTime
      )
    case ForwardRef: {
      const type = workInProgress.type
      const unresolvedProps = workInProgress.pendingProps
      const resolvedProps =
        workInProgress.elementType === type
          ? unresolvedProps
          : resolveDefaultProps(type, unresolvedProps)
      return updateForwardRef(
        current,
        workInProgress,
        type,
        resolvedProps,
        renderExpirationTime
      )
    }
    case Fragment:
      return updateFragment(current, workInProgress, renderExpirationTime)
    case Mode:
      return updateMode(current, workInProgress, renderExpirationTime)
    case Profiler:
      return updateProfiler(current, workInProgress, renderExpirationTime)
    case ContextProvider:
      return updateContextProvider(
        current,
        workInProgress,
        renderExpirationTime
      )
    case ContextConsumer:
      return updateContextConsumer(
        current,
        workInProgress,
        renderExpirationTime
      )
    case MemoComponent: {
      const type = workInProgress.type
      const unresolvedProps = workInProgress.pendingProps
      const resolvedProps = resolveDefaultProps(type.type, unresolvedProps)
      return updateMemoComponent(
        current,
        workInProgress,
        type,
        resolvedProps,
        updateExpirationTime,
        renderExpirationTime
      )
    }
    case SimpleMemoComponent: {
      return updateSimpleMemoComponent(
        current,
        workInProgress,
        workInProgress.type,
        workInProgress.pendingProps,
        updateExpirationTime,
        renderExpirationTime
      )
    }
    case IncompleteClassComponent: {
      const Component = workInProgress.type
      const unresolvedProps = workInProgress.pendingProps
      const resolvedProps =
        workInProgress.elementType === Component
          ? unresolvedProps
          : resolveDefaultProps(Component, unresolvedProps)
      return mountIncompleteClassComponent(
        current,
        workInProgress,
        Component,
        resolvedProps,
        renderExpirationTime
      )
    }
    default:
      invariant(
        false,
        'Unknown unit of work tag. This error is likely caused by a bug in ' +
          'React. Please file an issue.'
      )
  }
}
```

bailoutOnAlreadyFinishedWork

```javascript
function bailoutOnAlreadyFinishedWork(
  current: Fiber | null,
  workInProgress: Fiber,
  renderExpirationTime: ExpirationTime
): Fiber | null {
  cancelWorkTimer(workInProgress)

  if (current !== null) {
    workInProgress.firstContextDependency = current.firstContextDependency
  }

  if (enableProfilerTimer) {
    stopProfilerTimerIfRunning(workInProgress)
  }
  const childExpirationTime = workInProgress.childExpirationTime
  if (childExpirationTime < renderExpirationTime) {
    //说明无更新
    return null
  } else {
    cloneChildFibers(current, workInProgress) //自己无更新，子节点也无更新，拷贝子节点就好了
    return workInProgress.child
  }
}
```

##### mountChildFibers 和 reconcileChildFibers

```javascript
var reconcileChildFibers = ChildReconciler(true);
var mountChildFibers = ChildReconciler(false);

...

function ChildReconciler(shouldTrackSideEffects) {

    ...

    function reconcileChildFibers(returnFiber, currentFirstChild, newChild, expirationTime) {

      var isUnkeyedTopLevelFragment = typeof newChild === 'object' && newChild !== null && newChild.type === REACT_FRAGMENT_TYPE && newChild.key === null;
      if (isUnkeyedTopLevelFragment) {
        newChild = newChild.props.children;
      }

      // Handle object types
      var isObject = typeof newChild === 'object' && newChild !== null;

      if (isObject) {
        switch (newChild.$$typeof) {
          case REACT_ELEMENT_TYPE:
            return placeSingleChild(reconcileSingleElement(returnFiber, currentFirstChild, newChild, expirationTime));
          case REACT_PORTAL_TYPE:
            return placeSingleChild(reconcileSinglePortal(returnFiber, currentFirstChild, newChild, expirationTime));
        }
      }

      if (typeof newChild === 'string' || typeof newChild === 'number') {
        return placeSingleChild(reconcileSingleTextNode(returnFiber, currentFirstChild, '' + newChild, expirationTime));
      }

      if (isArray(newChild)) {
        return reconcileChildrenArray(returnFiber, currentFirstChild, newChild, expirationTime);
      }

      if (getIteratorFn(newChild)) {
        return reconcileChildrenIterator(returnFiber, currentFirstChild, newChild, expirationTime);
      }

      if (isObject) {
        throwOnInvalidObjectType(returnFiber, newChild);
      }

      {
        if (typeof newChild === 'function') {
          warnOnFunctionType();
        }
      }
      if (typeof newChild === 'undefined' && !isUnkeyedTopLevelFragment) {
        switch (returnFiber.tag) {
          case ClassComponent:
            {
              {
                var instance = returnFiber.stateNode;
                if (instance.render._isMockFunction) {
                  break;
                }
              }
            }
          case FunctionComponent:
            {
              var Component = returnFiber.type;
              invariant(false, '%s(...): Nothing was returned from render. This usually means a return statement is missing. Or, to render nothing, return null.', Component.displayName || Component.name || 'Component');
            }
        }
      }

      return deleteRemainingChildren(returnFiber, currentFirstChild);
    }

    return reconcileChildFibers;
  }
```

##### updateSlot

```javascript
function updateSlot(returnFiber, oldFiber, newChild, expirationTime) {
  var key = oldFiber !== null ? oldFiber.key : null

  if (typeof newChild === 'string' || typeof newChild === 'number') {
    //如果是string或者number
    if (key !== null) {
      //老的节点是有key
      return null
    }
    return updateTextNode(returnFiber, oldFiber, '' + newChild, expirationTime) //直接复用
  }

  if (typeof newChild === 'object' && newChild !== null) {
    switch (newChild.$$typeof) {
      case REACT_ELEMENT_TYPE: {
        if (newChild.key === key) {
          //key一样复用
          if (newChild.type === REACT_FRAGMENT_TYPE) {
            return updateFragment(
              returnFiber,
              oldFiber,
              newChild.props.children,
              expirationTime,
              key
            )
          }
          return updateElement(returnFiber, oldFiber, newChild, expirationTime)
        } else {
          return null
        }
      }
      case REACT_PORTAL_TYPE: {
        if (newChild.key === key) {
          //key一样复用
          return updatePortal(returnFiber, oldFiber, newChild, expirationTime)
        } else {
          return null
        }
      }
    }

    if (isArray(newChild) || getIteratorFn(newChild)) {
      if (key !== null) {
        return null
      }

      return updateFragment(
        returnFiber,
        oldFiber,
        newChild,
        expirationTime,
        null
      )
    }

    throwOnInvalidObjectType(returnFiber, newChild)
  }

  {
    if (typeof newChild === 'function') {
      warnOnFunctionType()
    }
  }

  return null
}
```

**updateClassComponent**

```javascript
function updateClassComponent(
  current: Fiber | null,
  workInProgress: Fiber,
  Component: any,
  nextProps,
  renderExpirationTime: ExpirationTime
) {
  let hasContext
  if (isLegacyContextProvider(Component)) {
    hasContext = true
    pushLegacyContextProvider(workInProgress)
  } else {
    hasContext = false
  }
  prepareToReadContext(workInProgress, renderExpirationTime)

  const instance = workInProgress.stateNode
  let shouldUpdate
  if (instance === null) {
    //没被创建为null
    if (current !== null) {
      current.alternate = null
      workInProgress.alternate = null
      workInProgress.effectTag |= Placement
    }
    constructClassInstance(
      workInProgress,
      Component,
      nextProps,
      renderExpirationTime
    )
    mountClassInstance(
      workInProgress,
      Component,
      nextProps,
      renderExpirationTime
    )
    shouldUpdate = true
  } else if (current === null) {
    shouldUpdate = resumeMountClassInstance(
      workInProgress,
      Component,
      nextProps,
      renderExpirationTime
    )
  } else {
    shouldUpdate = updateClassInstance(
      current,
      workInProgress,
      Component,
      nextProps,
      renderExpirationTime
    )
  }
  return finishClassComponent(
    current,
    workInProgress,
    Component,
    shouldUpdate,
    hasContext,
    renderExpirationTime
  )
}
```

## 参考

[requestIdleCallback](https://juejin.im/post/5ad71f39f265da239f07e862)
