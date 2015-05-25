# 域

    稳定性: 2 - 不稳定

域提供了一种方法，它能把多个不同的 IO 操作看成一个单独组。如果任何一个注册到域的事件或者回调触发 `error` 事件，或者抛出一个异常，域就会接收到通知，而不是在`process.on('uncaughtException')`处理函数一样丢失错误的上下文，也不会使程序立即退出。

## 警告：不要忽视错误!

<!-- type=misc -->

域错误处理程序并不是一个错误发生时关闭你的进程的替代品。

基于 JavaScript 中抛出异常的工作原理，基本上不可能在不泄露引用，或者不造成一些其他未定义的状态下，完全重现现场。

响应抛出错误最安全的方法就是关闭进程。一个正常的服务器会可能有很多活跃的连接，因为某个错误就关闭所有连接显然是不合理的。

比较好的方法是给触发错误的请求发送回应，让其他连接正常工作时，停止监听触发错误的人的新请求。

按这种方法，`域`和集群（cluster）模块可以协同工作，当某个进程遇到错误时，主进程可以复制一个新的进程。对于 Node 程序，终端代理或者注册的服务，可以留意错误并做出反应。

举例来说，下面的代码就不是好办法:

```
javascript
// XXX WARNING!  BAD IDEA!

var d = require('domain').create();
d.on('error', function(er) {
  // The error won't crash the process, but what it does is worse!
  // Though we've prevented abrupt process restarting, we are leaking
  // resources like crazy if this ever happens.
  // This is no better than process.on('uncaughtException')!
  console.log('error, but oh well', er.message);
});
d.run(function() {
  require('http').createServer(function(req, res) {
    handleRequest(req, res);
  }).listen(PORT);
});
```  

通过使用域的上下文，并将程序切为多个工作进程，我们能够更合理的响应，处理错误更安全。

  
```
javascript
// 好一些的做法!

var cluster = require('cluster');
var PORT = +process.env.PORT || 1337;

if (cluster.isMaster) {
  // In real life, you'd probably use more than just 2 workers,
  // and perhaps not put the master and worker in the same file.
  //
  // You can also of course get a bit fancier about logging, and
  // implement whatever custom logic you need to prevent DoS
  // attacks and other bad behavior.
  //
  // See the options in the cluster documentation.
  //
  // The important thing is that the master does very little,
  // increasing our resilience to unexpected errors.

  cluster.fork();
  cluster.fork();

  cluster.on('disconnect', function(worker) {
    console.error('disconnect!');
    cluster.fork();
  });

} else {
  // the worker
  //
  // This is where we put our bugs!

  var domain = require('domain');

  // See the cluster documentation for more details about using
  // worker processes to serve requests.  How it works, caveats, etc.

  var server = require('http').createServer(function(req, res) {
    var d = domain.create();
    d.on('error', function(er) {
      console.error('error', er.stack);

      // Note: we're in dangerous territory!
      // By definition, something unexpected occurred,
      // which we probably didn't want.
      // Anything can happen now!  Be very careful!

      try {
        // make sure we close down within 30 seconds
        var killtimer = setTimeout(function() {
          process.exit(1);
        }, 30000);
        // But don't keep the process open just for that!
        killtimer.unref();

        // stop taking new requests.
        server.close();

        // Let the master know we're dead.  This will trigger a
        // 'disconnect' in the cluster master, and then it will fork
        // a new worker.
        cluster.worker.disconnect();

        // try to send an error to the request that triggered the problem
        res.statusCode = 500;
        res.setHeader('content-type', 'text/plain');
        res.end('Oops, there was a problem!\n');
      } catch (er2) {
        // oh well, not much we can do at this point.
        console.error('Error sending 500!', er2.stack);
      }
    });

    // Because req and res were created before this domain existed,
    // we need to explicitly add them.
    // See the explanation of implicit vs explicit binding below.
    d.add(req);
    d.add(res);

    // Now run the handler function in the domain.
    d.run(function() {
      handleRequest(req, res);
    });
  });
  server.listen(PORT);
}

// This part isn't important.  Just an example routing thing.
// You'd put your fancy application logic here.
function handleRequest(req, res) {
  switch(req.url) {
    case '/error':
      // We do some async stuff, and then...
      setTimeout(function() {
        // Whoops!
        flerb.bark();
      });
      break;
    default:
      res.end('ok');
  }
}
```

## 错误对象的附加内容

<!-- type=misc -->

任何时候一个错误被路由传到一个域的时，会添加几个字段。  

* `error.domain` 第一个处理错误的域
* `error.domainEmitter`  用这个错误对象触发 'error' 事件的事件分发器
* `error.domainBound` 绑定到 domain 的回调函数，第一个参数是 error。
* `error.domainThrown` boolean 值，表明是抛出错误，分发，或者传递给绑定的回到函数。  

## 隐式绑定

<!--type=misc-->

如果域正在使用中，所有新分发的对象（包括 流对象，请求，响应等）将会隐式的绑定到这个域。

另外，传递给底层事件循环（比如 fs.open 或其他接收回调的方法）的回调函数将会自动的绑定到这个域。如果他们抛出异常，域会捕捉到错误信息。

为了避免过度使用内存，域对象不会象隐式的添加为有效域的子对象。如果这样做的话，很容易影响到请求和响应对象的垃圾回收。

如果你想将域对象作为子对象嵌入到父域里，就必须显式的添加它们。  
  
隐式绑定路由抛出的错误和 `'error'` 事件，但是不会注册事件分发器到域，所以  `domain.dispose()` 不会关闭事件分发器。隐式绑定仅需注意抛出的错误和 `'error'` 事件。 

## 显式绑定

<!--type=misc-->

有时候正在使用的域并不是某个事件分发器的域。或者说，事件分发器可能在某个域里创建，但是被绑定到另外一个域里。

例如，HTTP 服务器使用正一个域对象，但我们希望可以每一个请求使用一个不同的域。

这可以通过显式绑定来实现。

例如:

```
// create a top-level domain for the server
var serverDomain = domain.create();

serverDomain.run(function() {
  // server is created in the scope of serverDomain
  http.createServer(function(req, res) {
    // req and res are also created in the scope of serverDomain
    // however, we'd prefer to have a separate domain for each request.
    // create it first thing, and add req and res to it.
    var reqd = domain.create();
    reqd.add(req);
    reqd.add(res);
    reqd.on('error', function(er) {
      console.error('Error', er, req.url);
      try {
        res.writeHead(500);
        res.end('Error occurred, sorry.');
      } catch (er) {
        console.error('Error sending 500', er, req.url);
      }
    });
  }).listen(1337);
});
```

## domain.create()

* return: {Domain}

返回一个新的域对象。

## Class: Domain

这个类封装了将错误和没有捕捉到的异常到有效对象功能。

域是 [EventEmitter][] 的子类.  监听它的 `error`事件来处理捕捉到的错误。  

### domain.run(fn)

* `fn` {Function}

在域的上下文运行提供的函数，隐式的绑定了所有的事件分发器，计时器和底层请求。

这是使用域的基本方法。

例如:

```
var d = domain.create();
d.on('error', function(er) {
  console.error('Caught error!', er);
});
d.run(function() {
  process.nextTick(function() {
    setTimeout(function() { // simulating some various async stuff
      fs.open('non-existent file', 'r', function(er, fd) {
        if (er) throw er;
        // proceed...
      });
    }, 100);
  });
});
```

这个例子里程序不会崩溃，而会触发`d.on('error')`。  

### domain.members

* {Array}

显式添加到域里的计时器和事件分发器数组。


### domain.add(emitter)

* `emitter` {EventEmitter | Timer} 添加到域里的计时器和事件分发器

显式将一个分发器添加到域。如果分发器调用的事件处理函数抛出错误，或者分发器遇到 `error` 事件，将会导向域的 `error` 事件，和隐式绑定一样。  

对于 `setInterval` 和 `setTimeout` 返回的计时器同样适用。如果这些回调函数抛出错误，将会被域的 'error' 处理器捕捉到。

如果计时器或分发器已经绑定到域，那它将会从上一个域移除，绑定到当前域。

### domain.remove(emitter)

* `emitter` {EventEmitter | Timer} 要移除的分发器或计时器

与 domain.add(emitter) 函数恰恰相反，这个函数将分发器移除出域。

### domain.bind(callback)

* `callback` {Function} 回调函数
* return: {Function}被绑定的函数

返回的函数是一个对于所提供的回调函数的包装函数。当调用这个返回的函数被时，所有被抛出的错误都会被导向到这个域的 `error` 事件。

#### Example

    var d = domain.create();

    function readSomeFile(filename, cb) {
      fs.readFile(filename, 'utf8', d.bind(function(er, data) {
        // if this throws, it will also be passed to the domain
        return cb(er, data ? JSON.parse(data) : null);
      }));
    }

    d.on('error', function(er) {
      // an error occurred somewhere.
      // if we throw it now, it will crash the program
      // with the normal line number and stack message.
    });

### domain.intercept(callback)

* `callback` {Function} 回调函数
* return: {Function} 被拦截的函数

和 `domain.bind(callback)` 类似。除了捕捉被抛出的错误外，它还会拦截 Error 对象作为参数传递到这个函数。  

这种方式下，常见的 `if (er) return callback(er);` 模式，能被一个地方一个错误处理替换。

#### Example

    var d = domain.create();

    function readSomeFile(filename, cb) {
      fs.readFile(filename, 'utf8', d.intercept(function(data) {
        // note, the first argument is never passed to the
        // callback since it is assumed to be the 'Error' argument
        // and thus intercepted by the domain.

        // if this throws, it will also be passed to the domain
        // so the error-handling logic can be moved to the 'error'
        // event on the domain instead of being repeated throughout
        // the program.
        return cb(null, JSON.parse(data));
      }));
    }

    d.on('error', function(er) {
      // an error occurred somewhere.
      // if we throw it now, it will crash the program
      // with the normal line number and stack message.
    });

### domain.enter()

这个函数就像 `run`, `bind`, 和 `intercept` 的管道系统，它设置有效域。它设定了域的`domain.active` and `process.domain`，还隐式的将域推到域模块管理的域栈（关于域栈的细节详见`domain.exit()`）。enter函数的调用，分隔了异步调用链以及绑定到一个域的I/O操作的结束或中断。


调用`enter`仅改变活动的域，而不改变域本身。在一个单独的域里可以调用任意多次`Enter` 和 `exit`。

### domain.exit()

`exit` 函数退出当前域，并从域的栈里移除。每当程序的执行流程要切换到不同的异步调用链的时候，要保证退出当前域。调用 exit 函数，分隔了异步调用链，和绑定到一个域的I/O操作的结束或中断。


如果有多个嵌套的域绑定到当前的上下文，`exit` 函数将会退出所有嵌套。

调用  `exit` 仅改变活跃域，不会改变自身域。在一个单独的域里可以调用任意多次`Enter` 和 `exit`。

如果在这个域名下 exit 已经被设置，exit 将不退出域返回。


### domain.dispose()

    稳定性: 0 - 抛弃。通过域里设置的错误事件来显示的消除失败的 IO 操作。

调用 `dispos` 后，通过 run，bind 或 intercept 绑定到域的回调函数不再使用这个域，并且分发 `dispose` 事件。


[EventEmitter]: events.html#events_class_events_eventemitter
