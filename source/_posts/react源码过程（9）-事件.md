---
title: react源码过程（9）- 事件
date: 2019-12-31 13:56:08
tags:
  - react源码过程
categories:
  - react源码过程
---

## 事件

React组件上声明的事件没有绑定在React组件对应的原生DOM节点上，而是绑定在document节点上，触发的事件是对原生事件的包装。



## 例子

```html
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
        debugger
        this.setState({ count: ++this.state.count });
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

组件在创建的时候会调用实例的 render方法，通过 createElement 创建 ReactConponent ，得到组件上的属性

![WX20191231-142031](http://118.24.241.76/WX20191231-142031.png)



在 **workLoop** 方法创建完整的 **Fiber** 树的链接关系。后在 **completeWork** 方法里的 **createInstance**方法 生成 stateNode后，添加绑定事件

```javascript
function setInitialDOMProperties(tag, domElement, rootContainerElement, nextProps, isCustomComponentTag) {
  // nextProps 就是 上面图里的 props
  for (var propKey in nextProps) {
    if (!nextProps.hasOwnProperty(propKey)) {
      continue;
    }
    var nextProp = nextProps[propKey];
   
    ...
    
    else if (registrationNameModules.hasOwnProperty(propKey)) {
      if (nextProp != null) {
        if (true && typeof nextProp !== 'function') {
          warnForInvalidEventListener(propKey, nextProp);
        }
        ensureListeningTo(rootContainerElement, propKey);
      }
    } else if (nextProp != null) {
      setValueForProperty(domElement, propKey, nextProp, isCustomComponentTag);
    }
  }
}


// 进入 ensureListeningTo
function ensureListeningTo(rootContainerElement, registrationName) {
  // rootContainerElement 是 document.getElementById('container')，registrationName 是 onClick
  var isDocumentOrFragment = rootContainerElement.nodeType === DOCUMENT_NODE || rootContainerElement.nodeType === DOCUMENT_FRAGMENT_NODE;
  var doc = isDocumentOrFragment ? rootContainerElement : rootContainerElement.ownerDocument;
  listenTo(registrationName, doc);
}

// 进入 listenTo
function listenTo(registrationName, mountAt) {
  // registrationName 是 onClick，mountAt 是 document
  var isListening = getListeningForDocument(mountAt);
  var dependencies = registrationNameDependencies[registrationName];

  for (var i = 0; i < dependencies.length; i++) {
    var dependency = dependencies[i];
    if (!(isListening.hasOwnProperty(dependency) && isListening[dependency])) {
      switch (dependency) {
       
        ...  
          
        default:
          var isMediaEvent = mediaEventTypes.indexOf(dependency) !== -1;
          if (!isMediaEvent) {
            trapBubbledEvent(dependency, mountAt);
          }
          break;
      }
      isListening[dependency] = true;
    }
  }
}


// 进入 trapBubbledEvent
function trapBubbledEvent(topLevelType, element) {
  // topLevelType 是 click ， element 是document
  if (!element) {
    return null;
  }
  var dispatch = isInteractiveTopLevelEventType(topLevelType) ? dispatchInteractiveEvent : dispatchEvent;

  addEventBubbleListener(element, getRawEventName(topLevelType),
  dispatch.bind(null, topLevelType));
}


// 进入 addEventBubbleListener
function addEventBubbleListener(element, eventType, listener) {
  // 为 document 添加绑定事件，点击的时候 触发 dispatchInteractiveEvent 方法
  element.addEventListener(eventType, listener, false);
}
```



触发点击事件

```javascript
function dispatchInteractiveEvent(topLevelType, nativeEvent) {
  interactiveUpdates(dispatchEvent, topLevelType, nativeEvent);
}


// 进入interactiveUpdates
function interactiveUpdates(fn, a, b) {
  return _interactiveUpdatesImpl(fn, a, b);
}


// 进入 interactiveUpdates$1
function interactiveUpdates$1(fn, a, b) {
  if (isBatchingInteractiveUpdates) {
    return fn(a, b);
  }
  if (!isBatchingUpdates && !isRendering && lowestPriorityPendingInteractiveExpirationTime !== NoWork) {
    performWork(lowestPriorityPendingInteractiveExpirationTime, false);
    lowestPriorityPendingInteractiveExpirationTime = NoWork;
  }
  var previousIsBatchingInteractiveUpdates = isBatchingInteractiveUpdates;
  var previousIsBatchingUpdates = isBatchingUpdates;
  isBatchingInteractiveUpdates = true;
  isBatchingUpdates = true;
  try {
    return fn(a, b);
  } finally {
    isBatchingInteractiveUpdates = previousIsBatchingInteractiveUpdates;
    isBatchingUpdates = previousIsBatchingUpdates;
    if (!isBatchingUpdates && !isRendering) {
      performSyncWork();
    }
  }
}


// 进入 dispatchEvent
function dispatchEvent(topLevelType, nativeEvent) {
  if (!_enabled) {
    return;
  }

  // 获得点击事件的target
  var nativeEventTarget = getEventTarget(nativeEvent);
  // 获得节点的 FiberNode
  var targetInst = getClosestInstanceFromNode(nativeEventTarget);
  if (targetInst !== null && typeof targetInst.tag === 'number' && !isFiberMounted(targetInst)) {
    targetInst = null;
  }

  var bookKeeping = getTopLevelCallbackBookKeeping(topLevelType, nativeEvent, targetInst);

  try {
    batchedUpdates(handleTopLevel, bookKeeping);
  } finally {
    releaseTopLevelCallbackBookKeeping(bookKeeping);
  }
}


// 进入 batchedUpdates
function batchedUpdates(fn, bookkeeping) {
  if (isBatching) {
    return fn(bookkeeping);
  }
  isBatching = true;
  try {
    return _batchedUpdatesImpl(fn, bookkeeping);
  } finally {
    isBatching = false;
    var controlledComponentsHavePendingUpdates = needsStateRestore();
    if (controlledComponentsHavePendingUpdates) {
      _flushInteractiveUpdatesImpl();
      restoreStateIfNeeded();
    }
  }
}


// 进入 _batchedUpdatesImpl
function batchedUpdates$1(fn, a) {
  var previousIsBatchingUpdates = isBatchingUpdates;
  isBatchingUpdates = true;
  try {
    return fn(a);
  } finally {
    isBatchingUpdates = previousIsBatchingUpdates;
    if (!isBatchingUpdates && !isRendering) {
      performSyncWork();
    }
  }
}


// 进入 handleTopLevel
function handleTopLevel(bookKeeping) {
  var targetInst = bookKeeping.targetInst;

  var ancestor = targetInst;
  do {
    if (!ancestor) {
      bookKeeping.ancestors.push(ancestor);
      break;
    }
    var root = findRootContainerNode(ancestor);
    if (!root) {
      break;
    }
    bookKeeping.ancestors.push(ancestor);
    ancestor = getClosestInstanceFromNode(root);
  } while (ancestor);

  for (var i = 0; i < bookKeeping.ancestors.length; i++) {
    targetInst = bookKeeping.ancestors[i];
    runExtractedEventsInBatch(bookKeeping.topLevelType, targetInst, bookKeeping.nativeEvent, getEventTarget(bookKeeping.nativeEvent));
  }
}


// 进入 runExtractedEventsInBatch
function runExtractedEventsInBatch(topLevelType, targetInst, nativeEvent, nativeEventTarget) {
  // 在这里为 events 添加 _dispatchListeners(事件回调) 与 _dispatchInstances(FiberNode) 属性
  var events = extractEvents(topLevelType, targetInst, nativeEvent, nativeEventTarget);
  runEventsInBatch(events);
}


// 补充一下 traverseTwoPhase， 从 extractEvents 方法进入后，为 events 添加属性
function traverseTwoPhase(inst, fn, arg) {
  var path = [];
  while (inst) {
    path.push(inst);
    inst = getParent(inst);
  }
  var i = void 0;
  for (i = path.length; i-- > 0;) {
    fn(path[i], 'captured', arg);
  }
  for (i = 0; i < path.length; i++) {
    fn(path[i], 'bubbled', arg);
  }
}


// 进入 runEventsInBatch
function runEventsInBatch(events) {
  if (events !== null) {
    eventQueue = accumulateInto(eventQueue, events);
  }

  var processingEventQueue = eventQueue;
  eventQueue = null;

  if (!processingEventQueue) {
    return;
  }

  forEachAccumulated(processingEventQueue, executeDispatchesAndReleaseTopLevel);
  !!eventQueue ? invariant(false, 'processEventQueue(): Additional events were enqueued while processing an event queue. Support for this has not yet been implemented.') : void 0;
  rethrowCaughtError();
}


// 进入 forEachAccumulated
function forEachAccumulated(arr, cb, scope) {
  if (Array.isArray(arr)) {
    arr.forEach(cb, scope);
  } else if (arr) {
    cb.call(scope, arr);
  }
}


// 进入 executeDispatchesAndReleaseTopLevel
var executeDispatchesAndReleaseTopLevel = function (e) {
  return executeDispatchesAndRelease(e);
};


// 进入 executeDispatchesAndRelease
var executeDispatchesAndRelease = function (event) {
  if (event) {
    executeDispatchesInOrder(event);

    if (!event.isPersistent()) {
      event.constructor.release(event);
    }
  }
};


// 进入 executeDispatchesInOrder
function executeDispatchesInOrder(event) {
  // 拿到事件回调
  var dispatchListeners = event._dispatchListeners;
  // 拿到FiberNode
  var dispatchInstances = event._dispatchInstances;
  {
    validateEventDispatches(event);
  }
  if (Array.isArray(dispatchListeners)) {
    for (var i = 0; i < dispatchListeners.length; i++) {
      if (event.isPropagationStopped()) {
        break;
      }
      executeDispatch(event, dispatchListeners[i], dispatchInstances[i]);
    }
  } else if (dispatchListeners) {
    executeDispatch(event, dispatchListeners, dispatchInstances);
  }
  event._dispatchListeners = null;
  event._dispatchInstances = null;
}


// 进入 executeDispatch
function executeDispatch(event, listener, inst) {
  var type = event.type || 'unknown-event';
  event.currentTarget = getNodeFromInstance(inst);
  invokeGuardedCallbackAndCatchFirstError(type, listener, undefined, event);
  event.currentTarget = null;
}


// 进入 invokeGuardedCallbackAndCatchFirstError
function invokeGuardedCallbackAndCatchFirstError(name, func, context, a, b, c, d, e, f) {
  invokeGuardedCallback.apply(this, arguments);
  if (hasError) {
    var error = clearCaughtError();
    if (!hasRethrowError) {
      hasRethrowError = true;
      rethrowError = error;
    }
  }
}


// 进入 invokeGuardedCallback
function invokeGuardedCallback(name, func, context, a, b, c, d, e, f) {
  hasError = false;
  caughtError = null;
  invokeGuardedCallbackImpl$1.apply(reporter, arguments);
}


// 最后进入 invokeGuardedCallbackDev
// 就是调用了传入的 func
```