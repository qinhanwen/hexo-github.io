---
title: react源码过程（2）- 基础
date: 2019-10-31 09:15:34
tags: 
- react源码过程
categories: 
- react源码过程
---

**createElement**

`ReactElement`通过`createElement`创建

该方法需要传入三个参数：

- type
- config
- children

```javascript
export function createElement(type, config, children) {
  let propName;

  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

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
      //如果propName为key、ref、__self、__source就不会处理
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }
	//超过2的参数都被认为是子节点，会放到一个数组里
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    
    ...
    
    props.children = childArray;
  }

	//读取设置的默认值，如果值是undefined就设置成默认值的属性
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  
  ...
  
  //最后返回了reactElement
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
```



**ReactElement**

返回一个存储节点信息的对象

```javascript
const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {

    $$typeof: REACT_ELEMENT_TYPE,

    type: type,
    key: key,
    ref: ref,
    props: props,

    _owner: owner,
  };

  ...
  
  return element;
}
```



**Component**

```javascript
function Component(props, context, updater) {
  this.props = props;
  this.context = context;
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}

Component.prototype.isReactComponent = {};

//状态更新的方法，支持传入方法与对象，之后会调用callback
Component.prototype.setState = function(partialState, callback) {
  invariant(
    typeof partialState === 'object' ||
      typeof partialState === 'function' ||
      partialState == null,
    'setState(...): takes an object of state variables to update or a ' +
      'function which returns an object of state variables.',
  );
  this.updater.enqueueSetState(this, partialState, callback, 'setState');
};

//强制更新
Component.prototype.forceUpdate = function(callback) {
  this.updater.enqueueForceUpdate(this, callback, 'forceUpdate');
};

...
```



**PureComponent**

继承于Component

```javascript
function ComponentDummy() {}
ComponentDummy.prototype = Component.prototype;

function PureComponent(props, context, updater) {
  this.props = props;
  this.context = context;
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}

const pureComponentPrototype = (PureComponent.prototype = new ComponentDummy());
// 也就是说
// pureComponentPrototype === PureComponent.prototype
// PureComponent.prototype.__proto__ === Component.prototype

pureComponentPrototype.constructor = PureComponent;
//constructor指向自己

Object.assign(pureComponentPrototype, Component.prototype);
//合并 PureComponent.prototype 与 Component.prototype的属性

pureComponentPrototype.isPureReactComponent = true;
// 添加 isPureReactComponent 属性
```



**createRef**

```javascript
export function createRef(): RefObject {
  const refObject = {
    current: null,
  };
  ...
  return refObject;
}
//返回一个对象
```



**forwardRef**

```javascript
export default function forwardRef<Props, ElementType: React$ElementType>(
  render: (props: Props, ref: React$Ref<ElementType>) => React$Node,
) {
	...
  return {
    $$typeof: REACT_FORWARD_REF_TYPE,
    render,
  };
}
```



**Contenxt**

createContext

```javascript
export function createContext<T>(
  defaultValue: T,
  calculateChangedBits: ?(a: T, b: T) => number,
): ReactContext<T> {
  if (calculateChangedBits === undefined) {
    calculateChangedBits = null;
  } else {
    if (__DEV__) {
      warningWithoutStack(
        calculateChangedBits === null ||
          typeof calculateChangedBits === 'function',
        'createContext: Expected the optional second argument to be a ' +
          'function. Instead received: %s',
        calculateChangedBits,
      );
    }
  }

  const context: ReactContext<T> = {
    $$typeof: REACT_CONTEXT_TYPE,
    _calculateChangedBits: calculateChangedBits,
    _currentValue: defaultValue,
    _currentValue2: defaultValue,
    _threadCount: 0,
    Provider: (null: any),
    Consumer: (null: any),
  };

  context.Provider = {
    $$typeof: REACT_PROVIDER_TYPE,
    _context: context,
  };

  let hasWarnedAboutUsingNestedContextConsumers = false;
  let hasWarnedAboutUsingConsumerProvider = false;

  if (__DEV__) {
    const Consumer = {
      $$typeof: REACT_CONTEXT_TYPE,
      _context: context,
      _calculateChangedBits: context._calculateChangedBits,
    };

    Object.defineProperties(Consumer, {
      Provider: {
        get() {
          if (!hasWarnedAboutUsingConsumerProvider) {
            hasWarnedAboutUsingConsumerProvider = true;
            warning(
              false,
              'Rendering <Context.Consumer.Provider> is not supported and will be removed in ' +
                'a future major release. Did you mean to render <Context.Provider> instead?',
            );
          }
          return context.Provider;
        },
        set(_Provider) {
          context.Provider = _Provider;
        },
      },
      _currentValue: {
        get() {
          return context._currentValue;
        },
        set(_currentValue) {
          context._currentValue = _currentValue;
        },
      },
      _currentValue2: {
        get() {
          return context._currentValue2;
        },
        set(_currentValue2) {
          context._currentValue2 = _currentValue2;
        },
      },
      _threadCount: {
        get() {
          return context._threadCount;
        },
        set(_threadCount) {
          context._threadCount = _threadCount;
        },
      },
      Consumer: {
        get() {
          if (!hasWarnedAboutUsingNestedContextConsumers) {
            hasWarnedAboutUsingNestedContextConsumers = true;
            warning(
              false,
              'Rendering <Context.Consumer.Consumer> is not supported and will be removed in ' +
                'a future major release. Did you mean to render <Context.Consumer> instead?',
            );
          }
          return context.Consumer;
        },
      },
    });

    context.Consumer = Consumer;
  } else {
    context.Consumer = context;
  }

  if (__DEV__) {
    context._currentRenderer = null;
    context._currentRenderer2 = null;
  }

  return context;
}
```



**ReactChildren**

```javascript
...
function forEachChildren(children, forEachFunc, forEachContext) {
  if (children == null) {
    return children;
  }
  const traverseContext = getPooledTraverseContext(
    null,
    null,
    forEachFunc,
    forEachContext,
  );
  traverseAllChildren(children, forEachSingleChild, traverseContext);
  releaseTraverseContext(traverseContext);
}
...
function mapChildren(children, func, context) {
  if (children == null) {
    return children;
  }
  const result = [];
  mapIntoWithKeyPrefixInternal(children, result, null, func, context);
  return result;
}
...
function countChildren(children) {
  return traverseAllChildren(children, () => null, null);
}
...
function onlyChild(children) {
  invariant(
    isValidElement(children),
    'React.Children.only expected to receive a single React element child.',
  );
  return children;
}
...
function toArray(children) {
  const result = [];
  mapIntoWithKeyPrefixInternal(children, result, null, child => child);
  return result;
}
...
export {
  forEachChildren as forEach,
  mapChildren as map,
  countChildren as count,
  onlyChild as only,
  toArray,
};
```



