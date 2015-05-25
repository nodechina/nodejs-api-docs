# 集群

    稳定性: 2 - 不稳定

单个 Node 实例运行在一个线程中。为了更好的利用多核系统的能力，可以启动 Node 集群来处理负载。

在集群模块里很容易就能创建一个共享所有服务器接口的进程。  

    var cluster = require('cluster');
    var http = require('http');
    var numCPUs = require('os').cpus().length;

    if (cluster.isMaster) {
      // Fork workers.
      for (var i = 0; i < numCPUs; i++) {
        cluster.fork();
      }

      cluster.on('exit', function(worker, code, signal) {
        console.log('worker ' + worker.process.pid + ' died');
      });
    } else {
      // Workers can share any TCP connection
      // In this case its a HTTP server
      http.createServer(function(req, res) {
        res.writeHead(200);
        res.end("hello world\n");
      }).listen(8000);
    }

运行 Node 后，将会在所有工作进程里共享 8000 端口。  

    % NODE_DEBUG=cluster node server.js
    23521,Master Worker 23524 online
    23521,Master Worker 23526 online
    23521,Master Worker 23523 online
    23521,Master Worker 23528 online

这个特性是最近才引入的，大家可以试试并提供反馈。

还要注意，在 Windows 系统里还不能在工作进程中创建一个被命名的管道服务器。  

## 如何工作

`child_process.fork` 方法派生工作进程，所以它能通过 IPC 和父进程通讯，并相互传递句柄。

集群模块通过2种分发模式来处理连接。  

第一种（默认方法，除了 Windows 平台）为循环式。主进程监听一个端口，接收新的连接，再轮流的分发给工作进程。

第二种，主进程监听 socket，并发送给感兴趣的工作进程，工作进程直接接收连接。  

第二种方法理论上性能最高。实际上，由于操作系统各式各样，分配往往分配不均。列如，70%的连接终止于2个进程，实际上共有8个进程。

因为 `server.listen()` 将大部分工作交给了主进程，所以一个普通的 Node.js 进程和一个集群工作进程会在三种情况下有所区别：

1. `server.listen({fd: 7})` 由于消息被传回主进程，所以将会监听***主进程***里的文件描述符，而不是其他工作进程里的文件描述符 7。
2. `server.listen(handle)` 监听一个明确地句柄，会使得工作进程使用指定句柄，而不是与主进程通讯。如果工作进程已经拥有了该句柄，前提是您知道在做什么。
3. `server.listen(0)` 通常它会让服务器随机监听端口。然而在集群里每个工作进程 `listen(0)` 时会收到相同的端口。实际上仅第一次是随机的，之后是可预测的。如果你想监听一个特定的端口，可以根据集群的工作进程的ID生产一个端口ID 。 

在 Node.js 或你的程序里没有路由逻辑，工作进程见也没有共享状态。因此，像登录和会话这样的工作，不要设计成过度依赖内存里的对象。

因为工作线程都是独立的，你可以根据需求来杀死或者派生而不会影响其他进程。只要仍然有工作进程，服务器还会接收连接。Node 不会自动管理工作进程的数量，这是你的责任，你可以根据自己需求来管理。

## cluster.schedulingPolicy

调度策略 `cluster.SCHED_RR` 表示轮流制，`cluster.SCHED_NONE` 表示操作系统处理。这是全局性的设定，一旦你通过 `cluster.setupMaster()` 派生了第一个工作进程，它就不可更改了。

`SCHED_RR` 是除 Windows 外所有系统的默认设置。只要 libuv 能够有效地分配 IOCP 句柄并且不产生巨大的性能损失，Windows 也会改为 SCHED_RR 方式。

`cluster.schedulingPolicy` 也可通过环境变量 `NODE_CLUSTER_SCHED_POLICY` 来更改。有效值为 `"rr"` 和 `"none"`。

## cluster.settings

* {Object}
  * `execArgv` {Array} 传给可执行的 Node 的参数列表(默认=`process.execArgv`)
  * `exec` {String} 执行文件的路径。  (默认=`process.argv[1]`)
  * `args` {Array} 传给工作进程的参数列表(默认=`process.argv.slice(2)`)
  * `silent` {Boolean}是否将输出发送给父进程的 stdio。
    (默认=`false`)
  * `uid` {Number} 设置用户进程的ID。 (See setuid(2)。)
  * `gid` {Number} 设置进程组的ID。 (See setgid(2)。)

调用 `.setupMaster()` (或 `.fork()`) 方法后，这个 settings 对象会包含设置内容，包括默认值。 

设置后会立即冻结，因为`.setupMaster()`只能调用一次。  

这个对象不应该被手动改变或设置。  

## cluster.isMaster

* {Boolean}

如果是主进程，返回 true。如果 `process.env.NODE_UNIQUE_ID` 未定义，`isMaster` 为 `true`。

## cluster.isWorker

* {Boolean}

如果不是主进程返回 true (和 `cluster.isMaster` 相反)。

## 事件: 'fork'

* `worker` {Worker object}

当一个新的工作进程被分支出来，集群模块会产生 'fork' 事件。它可用于记录工作进程，并创建自己的超时管理。

    var timeouts = [];
    function errorMsg() {
      console.error("Something must be wrong with the connection ...");
    }

    cluster.on('fork', function(worker) {
      timeouts[worker.id] = setTimeout(errorMsg, 2000);
    });
    cluster.on('listening', function(worker, address) {
      clearTimeout(timeouts[worker.id]);
    });
    cluster.on('exit', function(worker, code, signal) {
      clearTimeout(timeouts[worker.id]);
      errorMsg();
    });

## 事件: 'online'

* `worker` {Worker object}

分支出一个新的工作进程后，它会响应在线消息。当主线程接收到在线消息后，它会触发这个事件。'fork' 和 'online' 之间的区别在于，主进程分支一个工作进程后会调用 fork，而工作进程运行后会调用 emitted。  

    cluster.on('online', function(worker) {
      console.log("Yay, the worker responded after it was forked");
    });

## 事件: 'listening'

* `worker` {Worker object}
* `address` {Object}

工作进程调用 `listen()` 时，服务器会触发'listening'事件，同时也会在主进程的集群里触发。

事件处理函数有两个参数，`worker` 包含工作进程对象，`address` 包含以下属性：`address`, `port` 和 `addressType`。如果工作进程监听多个地址的时候，这些东西非常有用。

    cluster.on('listening', function(worker, address) {
      console.log("A worker is now connected to " + address.address + ":" + address.port);
    });

 `addressType` 是以下内容:

* `4` (TCPv4)
* `6` (TCPv6)
* `-1` (unix domain socket)
* `"udp4"` or `"udp6"` (UDP v4 or v6)*  

## 事件: 'disconnect'

* `worker` {Worker object}

当一个工作进程的 IPC 通道关闭时会触发这个事件。当工作进程正常退出，被杀死，或者手工关闭（例如worker.disconnect()）时会调用。

`disconnect` 和 `exit` 事件间可能存在延迟。 这些事件可以用来检测进程是否卡在清理过程中，或者存在长连接。

    cluster.on('disconnect', function(worker) {
      console.log('The worker #' + worker.id + ' has disconnected');
    });

## 事件: 'exit'

* `worker` {Worker object}
* `code` {Number} 如果正常退出，则为退出代码.
* `signal` {String} 使得进程被杀死的信号名 (比如. `'SIGHUP'`)  

当任意一个工作进程终止的时候，集群模块会触发 'exit' 事件。  

可以调用 `.fork()` 重新启动工作进程。

    cluster.on('exit', function(worker, code, signal) {
      console.log('worker %d died (%s). restarting...',
        worker.process.pid, signal || code);
      cluster.fork();
    });

参见 [child_process event: 'exit'](child_process.html#child_process_event_exit).

## 事件: 'setup'

* `settings` {Object}

调用`.setupMaster()` 后会被触发。

 `settings` 对象就是 `cluster.settings` 对象。

详细内容参见 `cluster.settings`。

## cluster.setupMaster([settings])

* `settings` {Object}
  * `exec` {String} 执行文件的路径。  (默认=`process.argv[1]`)
  * `args` {Array}传给工作进程的参数列表(默认=`process.argv.slice(2)`)
  * `silent` {Boolean} 是否将输出发送给父进程的 stdio.

`setupMaster`用来改变默认的 'fork' 。 一旦调用，settings值将会出现在`cluster.settings`里。

注意:

* 改变任何设置，仅会对未来的工作进程产生影响，不会影响对目前已经运行的进程
* 工作进程里，仅能改变传递给 `.fork()` 的 `env` 属性。  
* 以上的默认值，仅在第一次调用的时候有效，之后的默认值是调用 `cluster.setupMaster()` 后的值。 

例如:

    var cluster = require('cluster');
    cluster.setupMaster({
      exec: 'worker.js',
      args: ['--use', 'https'],
      silent: true
    });
    cluster.fork(); // https worker
    cluster.setupMaster({
      args: ['--use', 'http']
    });
    cluster.fork(); // http worker

仅能在主进程里调用。

## cluster.fork([env])

* `env` {Object} 添加到子进程环境变量中的键值。
* return {Worker object}

派生一个新的工作进程。

仅能在主进程里调用。

## cluster.disconnect([callback])

* `callback` {Function} 当所有工作进程都断开连接，并且关闭句柄后被调用。

`cluster.workers` 里的每个工作进程可调用 `.disconnect()` 关闭。

关闭所有的内部句柄连接，并且没有任何等待处理的事件时，允许主进程优雅的退出。

这个方法有一个可选参数，会在完成时被调用。

仅能在主进程里调用。

## cluster.worker

* {Object}

对当前工作进程对象的引用。主进程中不可用。

    var cluster = require('cluster');

    if (cluster.isMaster) {
      console.log('I am master');
      cluster.fork();
      cluster.fork();
    } else if (cluster.isWorker) {
      console.log('I am worker #' + cluster.worker.id);
    }

## cluster.workers

* {Object}

存储活跃工作对象的哈希表，主键是 `id`，能方便的遍历所有工作进程，仅在主进程可用。

当工作进程关闭连接并退出后，将会从 cluster.workers 里移除。这两个事件的次序无法确定，仅能保证从cluster.workers 移除会发生在 `'disconnect'` 或 `'exit'` 之后。


    // Go through all workers
    function eachWorker(callback) {
      for (var id in cluster.workers) {
        callback(cluster.workers[id]);
      }
    }
    eachWorker(function(worker) {
      worker.send('big announcement to all workers');
    });

如果希望通过通讯通道引用工作进程，那么使用工作进程的 id 来查询最简单。

    socket.on('data', function(id) {
      var worker = cluster.workers[id];
    });

## Class: Worker

一个 Worker 对象包含工作进程所有公开的信息和方法。在主进程里可用通过 `cluster.workers` 来获取，在工作进程里可以通过 `cluster.worker` 来获取。

### worker.id

* {String}

每一个新的工作进程都有独立的唯一标示，它就是 `id`。

当工作进程可用时，`id` 就是 cluster.workers 里的主键。 

### worker.process

* {ChildProcess object}

所有工作进程都是通用 child_process.fork() 创建的，该函数返回的对象被储存在 process 中。  

参见: [Child Process module](
child_process.html#child_process_child_process_fork_modulepath_args_options)

注意，当 `process` 和 `.suicide` 不是 `true` 的时候，会触发 `'disconnect'` 事件，并使得工作进程调用 `process.exit(0)` 。它会保护意外的连接关闭。

### worker.suicide

* {Boolean}

调用 `.kill()` 或 `.disconnect()` 后设置，在这之前是 `undefined`。

`worker.suicide` 能让你区分出是自愿的还是意外退出，主进程可以根据这个值，来决定是否是重新派生成工作进程。

    cluster.on('exit', function(worker, code, signal) {
      if (worker.suicide === true) {
        console.log('Oh, it was just suicide\' – no need to worry').
      }
    });

    // kill worker
    worker.kill();

### worker.send(message[, sendHandle])

* `message` {Object}
* `sendHandle` {Handle object}

这个函数和 child_process.fork() 提供的 send 方法相同。主进程里你必须使用这个函数给指定工作进程发消息。 

在工作进程里，你也可以用 `process.send(message)`。

这个例子会回应所有来自主进程的消息:

    if (cluster.isMaster) {
      var worker = cluster.fork();
      worker.send('hi there');

    } else if (cluster.isWorker) {
      process.on('message', function(msg) {
        process.send(msg);
      });
    }

### worker.kill([signal='SIGTERM'])

* `signal` {String}发送给工作进程的杀死信号的名称

这个函数会杀死工作进程。在主进程里，它会关闭 `worker.process`，一旦关闭会发送杀死信号。在工作进程里，关闭通道，退出，返回代码`0`。
  
会导致 `.suicide` 被设置。  

为了保持兼容性，这个方法的别名是`worker.destroy()`。

注意，在工作进程里有`process.kill()`，于此不同，参见[kill](process.html#process_process_kill_pid_signal)。

### worker.disconnect()

在工作进程里，这个函数会关闭所有服务器，等待 'close' 事件，关闭 IPC 通道。

在主进程里，发给工作进程一个内部消息，用来调用`.disconnect()`

会导致 `.suicide` 被设置。

注意，服务器关闭后，不再接受新的连接，但可以接受新的监听。已经存在的连接允许正常退出。当连接为空得时候，工作进程的 IPC 通道运行优雅的退出（参见[server.close()](net.html#net_event_close)）。

以上仅能适用于服务器的连接，客户端的连接由工作进程关闭。

注意，在工作进程里，存在 `process.disconnect`，但并不是这个函数，它是 [disconnect](child_process.html#child_process_child_disconnect)。

由于长连接可能会阻塞进程关闭连接，有一个较好的办法是发消息给应用，这样应用会想办法关闭它们。超时管理也是不错，如果超过一定时间后还没有触发 `disconnect` 事件，将会杀死进程。

    if (cluster.isMaster) {
      var worker = cluster.fork();
      var timeout;

      worker.on('listening', function(address) {
        worker.send('shutdown');
        worker.disconnect();
        timeout = setTimeout(function() {
          worker.kill();
        }, 2000);
      });

      worker.on('disconnect', function() {
        clearTimeout(timeout);
      });

    } else if (cluster.isWorker) {
      var net = require('net');
      var server = net.createServer(function(socket) {
        // connections never end
      });

      server.listen(8000);

      process.on('message', function(msg) {
        if(msg === 'shutdown') {
          // initiate graceful close of any connections to server
        }
      });
    }

### worker.isDead()

工作进程结束，返回 `true`， 否则返回 `false`。

### worker.isConnected()

当工作进程通过 IPC 通道连接主进程时，返回 `true` ，否则 `false` 。工作进程创建后会连接到主进程。当`disconnect`事件触发后会关闭连接。

### 事件: 'message'

* `message` {Object}

该事件和 `child_process.fork()` 所提供的一样。在主进程中您应当使用该事件，而在工作进程中您也可以使用 `process.on('message')`。

例如，有一个集群使用消息系统在主进程中统计请求的数量:

    var cluster = require('cluster');
    var http = require('http');

    if (cluster.isMaster) {

      // Keep track of http requests
      var numReqs = 0;
      setInterval(function() {
        console.log("numReqs =", numReqs);
      }, 1000);

      // Count requestes
      function messageHandler(msg) {
        if (msg.cmd && msg.cmd == 'notifyRequest') {
          numReqs += 1;
        }
      }

      // Start workers and listen for messages containing notifyRequest
      var numCPUs = require('os').cpus().length;
      for (var i = 0; i < numCPUs; i++) {
        cluster.fork();
      }

      Object.keys(cluster.workers).forEach(function(id) {
        cluster.workers[id].on('message', messageHandler);
      });

    } else {

      // Worker processes have a http server.
      http.Server(function(req, res) {
        res.writeHead(200);
        res.end("hello world\n");

        // notify master about the request
        process.send({ cmd: 'notifyRequest' });
      }).listen(8000);
    }

### 事件: 'online'

和 `cluster.on('online')` 事件类似, 仅能在特定工作进程里触发。

    cluster.fork().on('online', function() {
      // Worker is online
    });

不会在工作进程里触发。

### 事件: 'listening'

* `address` {Object}

和 `cluster.on('listening')` 事件类似, 仅能在特定工作进程里触发。

    cluster.fork().on('listening', function(address) {
      // Worker is listening
    });

不会在工作进程里触发。

### 事件: 'disconnect'

和 `cluster.on('disconnect')` 事件类似, 仅能在特定工作进程里触发。

    cluster.fork().on('disconnect', function() {
      // Worker has disconnected
    });

### 事件: 'exit'

* `code` {Number} 正常退出时的退出代码.
* `signal` {String}  使得进程被终止的信号的名称（比如 `SIGHUP`）。

和`cluster.on('exit')` 事件类似, 仅能在特定工作进程里触发。

    var worker = cluster.fork();
    worker.on('exit', function(code, signal) {
      if( signal ) {
        console.log("worker was killed by signal: "+signal);
      } else if( code !== 0 ) {
        console.log("worker exited with error code: "+code);
      } else {
        console.log("worker success!");
      }
    });

### 事件: 'error'

和 `child_process.fork()`事件类似。

工作进程里，你也可以用 `process.on('error')`。
