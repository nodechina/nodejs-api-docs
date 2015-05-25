# 事件

    文档: 4 - API 冻结

Node 里很多对象会分发事件： 每次有连接的时候 `net.Server` 会分发事件，当文件打开的时候 `fs.readStream` 会分发事件。所有能分发事件的对象都是 `events.EventEmitter` 的实例。通过 `require("events");` 能访问这个模块。

一般来说，事件名都遵照驼峰规则，但这不是强制规定，任何形式的字符串都可以做为事件名。

为了处理事件，通常将函数关联到对象上。这些函数也叫监听者(listeners)。在这个函数里，`this` 指向  监听者所关联的 `EventEmitter`。

<a name="events_class_events_eventemitter"></a>
## 类: events.EventEmitter

通过 `require('events').EventEmitter` 获取 EventEmitter 类。

`EventEmitter` 实例遇到错误后，通常会触发一个错误事件。错误事件在 node 里是特殊例子。如果没有监听者，默认的操作是打印一个堆栈信息并退出程序。

当添加新的监听者时， EventEmitters 会触发`'newListener'` 事件，当移除时会触发`'removeListener'`。

### emitter.addListener(event, listener)
### emitter.on(event, listener)

添加一个监听者到特定 `event` 的监听数组的尾部，触发器不会检查是否已经添加过这个监听者。 多次调用相同的 `event` 和 `listener` 将会导致 `listener` 添加多次。

    server.on('connection', function (stream) {
      console.log('someone connected!');
    });

返回 emitter。

### emitter.once(event, listener)

给事件添加一个一次性的 listener，这个 listener 只会被触发一次，之后就会被移除。

    server.once('connection', function (stream) {
      console.log('Ah, we have our first user!');
    });

返回emitter。

### emitter.removeListener(event, listener)

从一个某个事件的 listener 数组中移除一个 listener。**注意**，这个操作会改变 listener 数组内容的次序。

    var callback = function(stream) {
      console.log('someone connected!');
    };
    server.on('connection', callback);
    // ...
    server.removeListener('connection', callback);

`removeListener` 最多会移除数组里的一个 listener。如果多次添加同一个 listener 到数组，那就需要多次调用 `removeListener` 来移除每一个实例。
  
返回emitter。

### emitter.removeAllListeners([event])

移除所有的 listener，或者某个事件的 listener。最好不要移除全部 listener，尤其是那些不是你传入的（比如 socket 或 文件流）。

返回emitter。  

### emitter.setMaxListeners(n)

默认情况下，给单个事件添加超过 10 个 listener ，事件分发器会打印警告。这样有利于检查内存泄露。不过不是所有的分发器都应该限制在 10 个，这个函数允许改变 listener 数量，无论是 0 还是更多。

返回emitter。  

### EventEmitter.defaultMaxListeners

`emitter.setMaxListeners(n)` 设置一个分发器的最大 listener 数，而这个函数会立即设置**所有**`EventEmitter` 的当前值和默认值。要小心使用。

请注意， `emitter.setMaxListeners(n)` 的优先级高于 `EventEmitter.defaultMaxListeners`.


### emitter.listeners(event)

返回事件的 listener 数组。

    server.on('connection', function (stream) {
      console.log('someone connected!');
    });
    console.log(util.inspect(server.listeners('connection'))); // [ [Function] ]


### emitter.emit(event[, arg1][, arg2][, ...])

使用指定的参数顺序的执行每一个 listener.

如果事件有 listener,返回  `true`， 否则 `false` 


### 类方法: EventEmitter.listenerCount(emitter, event)

返回指定事件的 listener 数量。


### Event: 'newListener'

* `event` {String} 事件名
* `listener` {Function} 事件处理函数
  
添加 listener 的时候会触发这个事件。当这个事件触发的时候，listener 可能还没添加到 listener 数组。


### Event: 'removeListener'

* `event` {String} 事件名
* `listener` {Function} 事件处理函数

删除 listener 的时候会触发这个事件。当这个事件触发的时候，listener 可能还还没从 listener 数组移除。

