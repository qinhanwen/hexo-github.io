---
title: Koa插件之类
date: 2019-06-05 11:27:58
tags: 
- koa
categories: 
- koa
---

## 基本组成

1. application.js：框架入口；负责管理中间件，以及处理请求
2. context.js：context对象的原型，代理request与response对象上的方法和属性
3. request.js：request对象的原型，提供请求相关的方法和属性
4. response.js：response对象的原型，提供响应相关的方法和属性



## 开始

新建`test.js`文件

```javascript
const koa = require('koa');
const app = new koa();

app.use(async (ctx, next) => {
    console.log(`1 start`);
    await next();
    console.log('1 end');
})

app.use(async (ctx, next) => {
    console.log(`2 start`);
    await next();
    console.log('2 end');
})

app.use(async (ctx, next) => {
    console.log(`3 start`);
    await next();
    console.log('3 end');
})

app.listen(3000);
console.log('koa server is listening port 3000');
```

通过` node --inspect-brk test`运行，进入调试模式。

![WX20191005-125153@2x](http://118.24.241.76/WX20191005-125153@2x.png)

浏览器打开`http://127.0.0.1:9229/`，点击这个

![WX20191005-125329@2x](http://118.24.241.76/WX20191005-125329@2x.png)



**首先通过`new Koa()`创建实例**

```javascript
module.exports = class Application extends Emitter {
  constructor() {
    super();

    this.proxy = false;
    this.middleware = [];
    this.subdomainOffset = 2;
    this.env = process.env.NODE_ENV || 'development';
    this.context = Object.create(context);
    this.request = Object.create(request);
    this.response = Object.create(response);
    if (util.inspect.custom) {
      this[util.inspect.custom] = this.inspect;
    }
  }
  ...
}
```

对象实例：

![WX20191005-190445@2x](http://118.24.241.76/WX20191005-190445@2x.png)



**`app.use`方法**

```javascript
  use(fn) {
    if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
    if (isGeneratorFunction(fn)) {
      deprecate('Support for generators will be removed in v3. ' +
                'See the documentation for examples of how to convert old middleware ' +
                'https://github.com/koajs/koa/blob/master/docs/migration.md');
      fn = convert(fn);
    }
    debug('use %s', fn._name || fn.name || '-');
    this.middleware.push(fn);
    return this;
  }
```

- 判断了传入的`fn`类型是否是`function`
- 然后判断是否是`generator`函数
- 之后`push`进`middleware`数组里



**`app.listen`方法**

```javascript
  listen(...args) {
    debug('listen');
    const server = http.createServer(this.callback());
    return server.listen(...args);
  }
```

- 先执行`this.callback()`，返回一个`handleRequest`函数

```javascript
  callback() {
    const fn = compose(this.middleware);
    if (!this.listenerCount('error')) this.on('error', this.onerror);
    const handleRequest = (req, res) => {
      const ctx = this.createContext(req, res);
      return this.handleRequest(ctx, fn);
    };

    return handleRequest;
  }
```

`compose`是由一个名为`koa-compose`的库,它可以将多个中间件组合成洋葱式调用的对象。

```javascript
function compose (middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (const fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  /**
   * @param {Object} context
   * @return {Promise}
   * @api public
   */

  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
}
```



- 调用`http.createServer`，该函数用来创建一个HTTP服务器，并将 `handleRequest` 作为 `request` 事件的监听函数。

- `server.listen(...args)`启动服务器监听连接，`3000`端口号



**发送请求到`http://localhost:3000/`，执行回调函数**

![WX20191005-201601@2x](http://118.24.241.76/WX20191005-201601@2x.png)

```javascript
    const handleRequest = (req, res) => {
      const ctx = this.createContext(req, res);
      return this.handleRequest(ctx, fn);
    };
```

- 首先调用`createContext`创建上下文对象

```javascript
  createContext(req, res) {
    const context = Object.create(this.context);
    const request = context.request = Object.create(this.request);
    const response = context.response = Object.create(this.response);
    context.app = request.app = response.app = this;
    context.req = request.req = response.req = req;
    context.res = request.res = response.res = res;
    request.ctx = response.ctx = context;
    request.response = response;
    response.request = request;
    context.originalUrl = request.originalUrl = req.url;
    context.state = {};
    return context;
  }
```

- 调用`this.handleRequest(ctx, fn)`

```javascript
  handleRequest(ctx, fnMiddleware) {
    const res = ctx.res;
    res.statusCode = 404;
    const onerror = err => ctx.onerror(err);
    const handleResponse = () => respond(ctx);
    onFinished(res, onerror);
    return fnMiddleware(ctx).then(handleResponse).catch(onerror);
  }
```

进入`fnMiddleware(ctx).then(handleResponse).catch(onerror);`

```javascript
  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
```



**小结**

1）首先先创建`koa`实例

2）`app.use`是往`middleware`数组里`push`进回调函数

3）`app.listen`启动服务端器监听，并且添加了回调函数

4）触发接口请求回调函数的时候，会将创建的上下文对象与`next`（next其实就是`dispatch`方法），作为参数传入到`middleware`中每项的函数调用中，如果调用了`next`就是调用的`dispatch.bind(null, i + 1)`



## koa-router

```javascript
const koa = require('koa');
const app = new koa();

const Router = require('koa-router');
const router = new Router();

router.get('/list/:id', async (ctx, next) => {
  ctx.body = {
    success: true,
    data: [{
        name: 'name1',
      },
      {
        name: 'name2',
      }
    ]
  };
})

app.use(router.routes());

app.listen(3000);
console.log('koa server is listening port 3000');
```



**看实例化`Router`的过程之前，先看一下，遍历数组为`Router`构造函数添加`prototype`属性**

![WX20191005-224823@2x](http://118.24.241.76/WX20191005-224823@2x.png)

看`Router`构造函数

```javascript
function Router(opts) {
  if (!(this instanceof Router)) {
    return new Router(opts);
  }

  this.opts = opts || {};
  this.methods = this.opts.methods || [
    'HEAD',
    'OPTIONS',
    'GET',
    'PUT',
    'PATCH',
    'POST',
    'DELETE'
  ];

  this.params = {};
  this.stack = [];
};
```

`Router`的实例

![WX20191005-225339@2x](http://118.24.241.76/WX20191005-225339@2x.png)



**`router.get`方法**

```javascript
  Router.prototype[method] = function (name, path, middleware) {
    var middleware;

    if (typeof path === 'string' || path instanceof RegExp) {
      middleware = Array.prototype.slice.call(arguments, 2);
    } else {
      middleware = Array.prototype.slice.call(arguments, 1);
      path = name;
      name = null;
    }

    this.register(path, [method], middleware, {
      name: name
    });

    return this;
  };
```

在这里调用`register`并且传入`path`、`method`、`name`和`middleware`（这里有对`router.get`传入的值做了一些修改）,进入`regiseter`方法

```javascript
Router.prototype.register = function (path, methods, middleware, opts) {
  opts = opts || {};

  var router = this;
  var stack = this.stack;

  // support array of paths
  if (Array.isArray(path)) {
    path.forEach(function (p) {
      router.register.call(router, p, methods, middleware, opts);
    });

    return this;
  }

  // create route
  var route = new Layer(path, methods, middleware, {
    end: opts.end === false ? opts.end : true,
    name: opts.name,
    sensitive: opts.sensitive || this.opts.sensitive || false,
    strict: opts.strict || this.opts.strict || false,
    prefix: opts.prefix || this.opts.prefix || "",
    ignoreCaptures: opts.ignoreCaptures
  });

  if (this.opts.prefix) {
    route.setPrefix(this.opts.prefix);
  }

  // add parameter middleware
  Object.keys(this.params).forEach(function (param) {
    route.param(param, this.params[param]);
  }, this);

  stack.push(route);

  return route;
};
```

这里实例化了`route`，并且`push`到了`stack`里



**`app.use(router.routes())`**

先看`router.routes()`

```javascript
Router.prototype.routes = Router.prototype.middleware = function () {
  var router = this;

  var dispatch = function dispatch(ctx, next) {
    debug('%s %s', ctx.method, ctx.path);

    var path = router.opts.routerPath || ctx.routerPath || ctx.path;
    var matched = router.match(path, ctx.method);
    var layerChain, layer, i;

    if (ctx.matched) {
      ctx.matched.push.apply(ctx.matched, matched.path);
    } else {
      ctx.matched = matched.path;
    }

    ctx.router = router;

    if (!matched.route) return next();

    var matchedLayers = matched.pathAndMethod
    var mostSpecificLayer = matchedLayers[matchedLayers.length - 1]
    ctx._matchedRoute = mostSpecificLayer.path;
    if (mostSpecificLayer.name) {
      ctx._matchedRouteName = mostSpecificLayer.name;
    }

    layerChain = matchedLayers.reduce(function(memo, layer) {
      memo.push(function(ctx, next) {
        ctx.captures = layer.captures(path, ctx.captures);
        ctx.params = layer.params(path, ctx.captures, ctx.params);
        ctx.routerName = layer.name;
        return next();
      });
      return memo.concat(layer.stack);
    }, []);

    return compose(layerChain)(ctx, next);
  };

  dispatch.router = this;

  return dispatch;
};
```

声明`dispatch`方法并且把`Router`实例挂载到了这个方法上

之后调用`app.use`把`dispatch`方法`push`进去了`middleware`数组



**`app.listen`方法**

与上面一样，就是传入回调，启动个服务。



**访问`http://localhost:3000/list/1`触发回调**

进入回调函数

```javascript
 return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
```

在这里，取得`middleware`的第一个回调并且调用



```javascript
  var dispatch = function dispatch(ctx, next) {
    debug('%s %s', ctx.method, ctx.path);

    var path = router.opts.routerPath || ctx.routerPath || ctx.path;
    var matched = router.match(path, ctx.method);
    var layerChain, layer, i;

    if (ctx.matched) {
      ctx.matched.push.apply(ctx.matched, matched.path);
    } else {
      ctx.matched = matched.path;
    }

    ctx.router = router;

    if (!matched.route) return next();

    var matchedLayers = matched.pathAndMethod
    var mostSpecificLayer = matchedLayers[matchedLayers.length - 1]
    ctx._matchedRoute = mostSpecificLayer.path;
    if (mostSpecificLayer.name) {
      ctx._matchedRouteName = mostSpecificLayer.name;
    }

    layerChain = matchedLayers.reduce(function(memo, layer) {
      memo.push(function(ctx, next) {
        ctx.captures = layer.captures(path, ctx.captures);
        ctx.params = layer.params(path, ctx.captures, ctx.params);
        ctx.routerName = layer.name;
        return next();
      });
      return memo.concat(layer.stack);
    }, []);

    return compose(layerChain)(ctx, next);
  };
```

最后调用到`compose`方法，传入的`layerChina`是个数组，第一个是匿名函数

```javascript
function(ctx, next) {
        ctx.captures = layer.captures(path, ctx.captures);
        ctx.params = layer.params(path, ctx.captures, ctx.params);
        ctx.routerName = layer.name;
        return next();
      }
```

第二个是访问`/list`的回调函数

![WX20191006-231742@2x](http://118.24.241.76/WX20191006-231742@2x.png)



最后又调用进这个方法

```javascript
  return function (context, next) {
    // last called middleware #
    let index = -1
    return dispatch(0)
    function dispatch (i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
      index = i
      let fn = middleware[i]
      if (i === middleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, function next () {
          return dispatch(i + 1)
        }))
      } catch (err) {
        return Promise.reject(err)
      }
    }
  }
```

也就是先调用了上面的匿名函数，这个匿名函数为`上下文对象`添加了一些属性

![WX20191006-234337@2x](http://118.24.241.76/WX20191006-234337@2x.png)

之后调用`next`方法，走到我们为访问`/list`添加的回调那里

![WX20191006-234604@2x](http://118.24.241.76/WX20191006-234604@2x.png)



**小结**

1）创建`Router`实例

2）`router.get`，注册路由规则，并且`push`进`stack`数组

3）`app.use(router.routes())`，将`router.routes()`返回的`dispatch`方法`push`到`middleware`数组里

4）`app.listen(3000)`，添加回调函数，并且启动服务器监听

5）访问`http://localhost:3000/list/1`，调用`middleware`数组中第一个函数调用，这个函数调用返回`compose(layerChain)(ctx, next);`

6）`layerChain`是由两个函数组成的数组，里面的第一个函数被调用时候，解析`path`，为`ctx`上下文对象添加一些属性，之后调用`next()`

7）也就是走进`layerChain`里的第二个函数调用，也就是`router.get`的回调函数





## koa-cors

test.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>

<body>
</body>
<script>
    function send() {
        var xmlHttp;
        if (XMLHttpRequest) {
            xmlHttp = new XMLHttpRequest();
        } else {
            xmlHttp = new ActiveXObject();
        }

        xmlHttp.onreadystatechange = function () {
            if (xmlHttp.readyState == 4) {
                if ((xmlHttp.status >= 200 && xmlHttp.status <= 300) || xmlHttp.status == 304) {
                    console.log(xmlHttp.response);
                }
            }
        }
        xmlHttp.open('GET', 'http://127.0.0.1:3000/list/1', true);
        xmlHttp.send();
    }

    send();
</script>

</html>
```

test.js

```javascript
const koa = require('koa');
const app = new koa();

const Router = require('koa-router');
const router = new Router();

var cors = require('koa2-cors');

router.get('/list/:id', async (ctx, next) => {
  ctx.body = {
    success: true,
    data: [{
        name: 'name1',
      },
      {
        name: 'name2',
      }
    ]
  };
})

app.use(cors());
app.use(router.routes());

app.listen(3000);
console.log('koa server is listening port 3000');
```



**`app.use(cors())`调用**

先进入`cors()`

```javascript
module.exports = function crossOrigin(options = {}) {
  const defaultOptions = {
    allowMethods: ['GET', 'PUT', 'POST', 'PATCH', 'DELETE', 'HEAD', 'OPTIONS']
  };

  // set defaultOptions to options
  options = Object.assign({}, defaultOptions, options); // eslint-disable-line no-param-reassign

  // eslint-disable-next-line consistent-return
  return async function cors(ctx, next) {
    // always set vary Origin Header
    // https://github.com/rs/cors/issues/10
    ctx.vary('Origin');

    let origin;
    if (typeof options.origin === 'function') {
      origin = options.origin(ctx);
    } else {
      origin = options.origin || ctx.get('Origin') || '*';
    }
    if (!origin) {
      return await next();
    }

    // Access-Control-Allow-Origin
    ctx.set('Access-Control-Allow-Origin', origin);

    if (ctx.method === 'OPTIONS') {
      // Preflight Request
      if (!ctx.get('Access-Control-Request-Method')) {
        return await next();
      }

      // Access-Control-Max-Age
      if (options.maxAge) {
        ctx.set('Access-Control-Max-Age', String(options.maxAge));
      }

      // Access-Control-Allow-Credentials
      if (options.credentials === true) {
        // When used as part of a response to a preflight request,
        // this indicates whether or not the actual request can be made using credentials.
        ctx.set('Access-Control-Allow-Credentials', 'true');
      }

      // Access-Control-Allow-Methods
      if (options.allowMethods) {
        ctx.set('Access-Control-Allow-Methods', options.allowMethods.join(','));
      }

      // Access-Control-Allow-Headers
      if (options.allowHeaders) {
        ctx.set('Access-Control-Allow-Headers', options.allowHeaders.join(','));
      } else {
        ctx.set('Access-Control-Allow-Headers', ctx.get('Access-Control-Request-Headers'));
      }

      ctx.status = 204; // No Content
    } else {
      // Request
      // Access-Control-Allow-Credentials
      if (options.credentials === true) {
        if (origin === '*') {
          // `credentials` can't be true when the `origin` is set to `*`
          ctx.remove('Access-Control-Allow-Credentials');
        } else {
          ctx.set('Access-Control-Allow-Credentials', 'true');
        }
      }

      // Access-Control-Expose-Headers
      if (options.exposeHeaders) {
        ctx.set('Access-Control-Expose-Headers', options.exposeHeaders.join(','));
      }

      try {
        await next();
      } catch (err) {
        throw err;
      }
    }
  };
```

返回了一个异步函数，之后`app.use`把这个函数`push`到`middleware`数组里



刷新`html`页面，触发`send`方法，发送请求，触发回调，之后先调用`middleware`数组里第一个函数调用，就是上面的这个方法，这个方法（在这个例子中）做的事情：

- 获取`origin`，通过`ctx.set('Access-Control-Allow-Origin', origin);`设置头部
- 调用`next()`

这里的`next`调用就是路由`Router.routes()`返回的那个方法，调用步骤与上面路由的地方一样



**小结**

1）`app.use(cors())`把回调传入`middleware`数组

2）发送请求，设置响应头

因为这边是简单请求，所以没有`OPTIONS`预检请求

- 当替换请求方法为`PUT`的时候，判断是`OPTIONS`请求的时候，会设置这个响应头

![WX20191007-153846@2x](http://118.24.241.76/WX20191007-153846@2x.png)

- 当添加请求头`name`属性的时候，会设置这个响应头

![WX20191007-154107@2x](http://118.24.241.76/WX20191007-154107@2x.png)



## koa-bodyparser

```javascript
...
const bodyParser = require('koa-bodyparser')
...
app.use(bodyParser())
...
```

先进入`bodyParser()`

最后往middleware数组push进去这个函数

```javascript
async function bodyParser(ctx, next) {
    if (ctx.request.body !== undefined) return await next();
    if (ctx.disableBodyParser) return await next();
    try {
      const res = await parseBody(ctx);
      ctx.request.body = 'parsed' in res ? res.parsed : {};
      if (ctx.request.rawBody === undefined) ctx.request.rawBody = res.raw;
    } catch (err) {
      if (onerror) {
        onerror(err, ctx);
      } else {
        throw err;
      }
    }
    await next();
  };
```

收到请求后根据不同的`content-type`调用不同的方法解析请求体，这边使用的是`application/x-www-form-urlencoded`

![WX20191024-215420@2x](http://118.24.241.76/WX20191024-215420@2x.png)

获取请求的`content-encoding`、`content-length`和编码方式`utf-8`

- 先根据`contentn-encoding`的值判断是否需要解压，还是直接返回

![WX20191028-095029@2x](http://118.24.241.76/WX20191028-095029@2x.png)

- 根据编码方式，Buffer转字符串

![WX20191028-095418@2x](http://118.24.241.76/WX20191028-095418@2x.png)

![WX20191028-094333@2x](http://118.24.241.76/WX20191028-094333@2x.png)

- 解析完成后返回str

![WX20191028-093318@2x](http://118.24.241.76/WX20191028-093318@2x.png)

最后挂载在ctx.request.body上

![WX20191024-220208@2x](http://118.24.241.76/WX20191024-220208@2x.png)