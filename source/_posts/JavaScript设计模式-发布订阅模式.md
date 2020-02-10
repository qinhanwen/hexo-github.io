---
title: JavaScript设计模式-发布订阅模式
date: 2019-08-25 01:10:41
tags: 
- JavaScript设计模式
categories: 
- JavaScript设计模式
---

## 定义

发布订阅模式理念和观察者模式相同，但是处理方式上不同：订阅者把自己想订阅的事件注册到调度中心，当该事件触发时候，发布者发布该事件到调度中心（顺带上下文），由调度中心统一调度订阅者注册到调度中心的处理代码。

![WX20190825-011607@2x](http://114.55.30.96/WX20190825-011607@2x.png)

双向解耦，发布者和订阅者都不用清楚对方，全部由订阅中心做处理。



## 实现

```javascript
var Event = (function () {
    var clientList = {}, listen, trigger, remove;
    listen = function (key, fn) {
        if (!clientList[key]) {
            clientList[key] = [];
        }
        clientList[key].push(fn);
    }
    trigger = function (key, ...args) {
        var key = Array.prototype.shift.call(arguments);
        var fns = clientList[key];
        if (fns && fns.length) {
            fns.forEach(fn => {
                fn(...args);
            });
        } else {
            return false
        }
    }
    remove = function (key, fn) {
        var fns = clientList[key];
        if (fns && fns.length) {
            var index = fns.findIndex(_fn => {
                return _fn !== fn
            });
            fns.splice(index, 1);
        }
    }
    return {
        listen,
        trigger,
        remove
    }
})()

function getSalary(salary) {
    console.log(salary);
}

Event.listen('A', getSalary);
Event.trigger('A', 1000);
Event.remove('A', getSalary);
Event.trigger('A', 1000);
```



`ionic`中的`Events`实现

```javascript
(function (factory) {
    if (typeof module === "object" && typeof module.exports === "object") {
        var v = factory(require, exports);
        if (v !== undefined) module.exports = v;
    }
    else if (typeof define === "function" && define.amd) {
        define(["require", "exports", "./scroll-view"], factory);
    }
})(function (require, exports) {
    "use strict";
    Object.defineProperty(exports, "__esModule", { value: true });
    var scroll_view_1 = require("./scroll-view");
    /**
     * @name Events
     * @description
     * Events is a publish-subscribe style event system for sending and responding to application-level
     * events across your app.
     *
     * @usage
     * ```ts
     * import { Events } from 'ionic-angular';
     *
     * // first page (publish an event when a user is created)
     * constructor(public events: Events) { }
     *
     * createUser(user) {
     *   console.log('User created!')
     *   this.events.publish('user:created', user, Date.now());
     * }
     *
     *
     * // second page (listen for the user created event after function is called)
     * constructor(public events: Events) {
     *   events.subscribe('user:created', (user, time) => {
     *     // user and time are the same arguments passed in `events.publish(user, time)`
     *     console.log('Welcome', user, 'at', time);
     *   });
     * }
     *
     * ```
     * @demo /docs/demos/src/events/
     */
    var Events = (function () {
        function Events() {
            this._channels = [];
        }
        /**
         * Subscribe to an event topic. Events that get posted to that topic will trigger the provided handler.
         *
         * @param {string} topic the topic to subscribe to
         * @param {function} handler the event handler
         */
        Events.prototype.subscribe = function (topic) {
            var _this = this;
            var handlers = [];
            for (var _i = 1; _i < arguments.length; _i++) {
                handlers[_i - 1] = arguments[_i];
            }
            if (!this._channels[topic]) {
                this._channels[topic] = [];
            }
            handlers.forEach(function (handler) {
                _this._channels[topic].push(handler);
            });
        };
        /**
         * Unsubscribe from the given topic. Your handler will no longer receive events published to this topic.
         *
         * @param {string} topic the topic to unsubscribe from
         * @param {function} handler the event handler
         *
         * @return true if a handler was removed
         */
        Events.prototype.unsubscribe = function (topic, handler) {
            if (handler === void 0) { handler = null; }
            var t = this._channels[topic];
            if (!t) {
                // Wasn't found, wasn't removed
                return false;
            }
            if (!handler) {
                // Remove all handlers for this topic
                delete this._channels[topic];
                return true;
            }
            // We need to find and remove a specific handler
            var i = t.indexOf(handler);
            if (i < 0) {
                // Wasn't found, wasn't removed
                return false;
            }
            t.splice(i, 1);
            // If the channel is empty now, remove it from the channel map
            if (!t.length) {
                delete this._channels[topic];
            }
            return true;
        };
        /**
         * Publish an event to the given topic.
         *
         * @param {string} topic the topic to publish to
         * @param {any} eventData the data to send as the event
         */
        Events.prototype.publish = function (topic) {
            var args = [];
            for (var _i = 1; _i < arguments.length; _i++) {
                args[_i - 1] = arguments[_i];
            }
            var t = this._channels[topic];
            if (!t) {
                return null;
            }
            var responses = [];
            t.forEach(function (handler) {
                responses.push(handler.apply(void 0, args));
            });
            return responses;
        };
        return Events;
    }());
    exports.Events = Events;
    /**
     * @hidden
     */
    function setupEvents(plt, dom) {
        var events = new Events();
        var win = plt.win();
        var doc = plt.doc();
        // start listening for resizes XXms after the app starts
        plt.timeout(function () {
            win.addEventListener('online', function (ev) {
                events.publish('app:online', ev);
            }, false);
            win.addEventListener('offline', function (ev) {
                events.publish('app:offline', ev);
            }, false);
            win.addEventListener('orientationchange', function (ev) {
                events.publish('app:rotated', ev);
            });
            // When that status taps, we respond
            win.addEventListener('statusTap', function () {
                // TODO: Make this more better
                var el = doc.elementFromPoint(plt.width() / 2, plt.height() / 2);
                if (!el) {
                    return;
                }
                var contentEle = el.closest('.scroll-content');
                if (contentEle) {
                    var style = contentEle.style;
                    var scroll = new scroll_view_1.ScrollView(null, plt, dom);
                    scroll._el = contentEle;
                    // We need to stop scrolling if it's happening and scroll up
                    style['WebkitBackfaceVisibility'] = 'hidden';
                    style['WebkitTransform'] = 'translate3d(0,0,0)';
                    dom.write(function () {
                        style.overflow = 'hidden';
                        function finish() {
                            style.overflow = '';
                            style['WebkitBackfaceVisibility'] = '';
                            style['WebkitTransform'] = '';
                        }
                        var didScrollTimeout = plt.timeout(function () {
                            finish();
                        }, 400);
                        scroll.scrollTo(0, 0, 300).then(function () {
                            plt.cancelTimeout(didScrollTimeout);
                            finish();
                        });
                    });
                }
            });
        }, 2000);
        return events;
    }
    exports.setupEvents = setupEvents;
    /**
     * @hidden
     */
    function setupProvideEvents(plt, dom) {
        return function () {
            return setupEvents(plt, dom);
        };
    }
    exports.setupProvideEvents = setupProvideEvents;
});
```

去除一些不太重要的东西

```javascript
var Events = (function () {
    function Events() {
        this._channels = [];
    }
    /**
     * Subscribe to an event topic. Events that get posted to that topic will trigger the provided handler.
     *
     * @param {string} topic the topic to subscribe to
     * @param {function} handler the event handler
     */
    Events.prototype.subscribe = function (topic) {
        var _this = this;
        var handlers = [];
        for (var _i = 1; _i < arguments.length; _i++) {
            handlers[_i - 1] = arguments[_i];
        }
        if (!this._channels[topic]) {
            this._channels[topic] = [];
        }
        handlers.forEach(function (handler) {
            _this._channels[topic].push(handler);
        });
    };
    /**
     * Unsubscribe from the given topic. Your handler will no longer receive events published to this topic.
     *
     * @param {string} topic the topic to unsubscribe from
     * @param {function} handler the event handler
     *
     * @return true if a handler was removed
     */
    Events.prototype.unsubscribe = function (topic, handler) {
        if (handler === void 0) {
            handler = null;
        }
        var t = this._channels[topic];
        if (!t) {
            // Wasn't found, wasn't removed
            return false;
        }
        if (!handler) {
            // Remove all handlers for this topic
            delete this._channels[topic];
            return true;
        }
        // We need to find and remove a specific handler
        var i = t.indexOf(handler);
        if (i < 0) {
            // Wasn't found, wasn't removed
            return false;
        }
        t.splice(i, 1);
        // If the channel is empty now, remove it from the channel map
        if (!t.length) {
            delete this._channels[topic];
        }
        return true;
    };
    /**
     * Publish an event to the given topic.
     *
     * @param {string} topic the topic to publish to
     * @param {any} eventData the data to send as the event
     */
    Events.prototype.publish = function (topic) {
        var args = [];
        for (var _i = 1; _i < arguments.length; _i++) {
            args[_i - 1] = arguments[_i];
        }
        var t = this._channels[topic];
        if (!t) {
            return null;
        }
        var responses = [];
        t.forEach(function (handler) {
            responses.push(handler.apply(void 0, args));
        });
        return responses;
    };
    return Events;
}());
```

