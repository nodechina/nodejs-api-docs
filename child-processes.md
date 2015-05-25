# 子进程

    稳定性: 3 - 稳定

Node 通过 `child_process` 模块提供了 `popen(3)` 数据流。

它能在非阻塞的方式中，通过 `stdin`, `stdout`, 和 `stderr` 传递数据。 (请注意，某些程序使用内部线性缓冲 I/O， 它并不妨碍 node.js，只是你发送给子进程的数据不会被立即消。)

使用 `require('child_process').spawn()` 或 `require('child_process').fork()` 创建一个子进程。这两种方法有区别，底下会解释 [below](#child_process_asynchronous_process_creation).

开发过程中查看
[synchronous counterparts](#child_process_synchronous_process_creation) 效率会更高。

## 类: ChildProcess

`ChildProcess` 是一个 [EventEmitter][]。

子进程有三个相关的流 `child.stdin`,
`child.stdout`, 和 `child.stderr`。他们可能和会父进程的 stdio
streams 共享, 也可作为独立的对象。

不能直接调用 ChildProcess 类，使用
`spawn()`, `exec()`, `execFile()`, 或 `fork()` 方法来创建子进程的实例。

### 事件:  'error'

* `err` {Error Object} 错误。

发生于:

1. 无法创建进程。
2. 无法杀死进程。
3. 无论什么原因导致给子进程发送消息失败。

注意，`exit` 事件有可能在错误发生后调用，也可能不调用，所以如果你监听这两个事件来触发函数，记得预防函数会被调用2次。

参考 [`ChildProcess#kill()`](#child_process_child_kill_signal) 和
[`ChildProcess#send()`](#child_process_child_send_message_sendhandle)。

### 事件: 'exit'

* `code` {Number} 退出代码, 正常退出时才有效。
* `signal` {String} 如果是被父进程杀死，则它为传递给子进程的信号

子进程结束的时候触发这个事件。如果子进程正常终止，则 `code` 为最终的退出代码，否则为 `null` 。如果是由 `signal` 引起的终止，则 `signal` 为字符串，否则为 `null`。

注意，子进程的 stdio 流可能仍为开启模式。

注意，node 为 `'SIGINT'` 和 `'SIGTERM`' 建立句柄，所以当信号来临的时候，他们不会终止而是退出。

参见 `waitpid(2)`。

### 事件: 'close'

* `code` {Number} 退出代码, 正常退出时才有效。
* `signal` {String} 如果是被父进程杀死，则它为传递给子进程的信号。

子进程里所有 stdio 流都关闭时触发这个事件。要和 'exit' 区分开, 因为多进程可以共享一个 stdio 流。

### Event: 'disconnect'    
  
父进程或子进程中调用`.disconnect()` 方法后触发这个事件。 断开后不会在互发消息，并且 `.connected` 属性值为 false。

### Event: 'message'

* `message` {Object} 一个解析过的 JSON 对象，或者一个原始值
* `sendHandle` {Handle object} 一个 Socket 或 Server 对象

通过 `.send(message, [sendHandle])` 传递消息。

### child.stdin

* {Stream object}

子进程的 `stdin` 是 `Writable Stream`（可写流）。
如果子进程在等待输入，它就会暂停直到通过调用 `end()` 来关闭。

`child.stdin` 是 `child.stdio[0]` 的缩写。这两个都指向同一个对象，或者 null。

### child.stdout

* {Stream object}

子进程的 `stdout` 是 `Readable Stream`（可读流）。


`child.stdout` 是 `child.stdio[1]` 的缩写。 这两个都指向同一个对象，或者 null。

### child.stderr

* {Stream object}

子进程的 `stderr` 是 `Readable Stream`（可写流）。

`child.stderr` 是 `child.stdio[2]` 缩写。 这两个都指向同一个对象，或者 null。

### child.stdio

* {Array}

子进程的管道数组和 [spawn](#child_process_child_process_spawn_command_args_options) 的 [stdio](#child_process_options_stdio) 里设置为 `'pipe'` 的内容次序相对应。
  
注意，流[0-2]也能分别用 ChildProcess.stdin,ChildProcess.stdout, 和 ChildProcess.stderrNote 来表示。

在下面的例子里，只有子进程的 fd `1` 设置为 pipe管道，所以父进程的 `child.stdio[1]` 是流（stream）, 数组里其他值为`null`。

    child = child_process.spawn("ls", {
        stdio: [
          0, // use parents stdin for child
          'pipe', // pipe child's stdout to parent
          fs.openSync("err.out", "w") // direct child's stderr to a file
        ]
    });

    assert.equal(child.stdio[0], null);
    assert.equal(child.stdio[0], child.stdin);

    assert(child.stdout);
    assert.equal(child.stdio[1], child.stdout);

    assert.equal(child.stdio[2], null);
    assert.equal(child.stdio[2], child.stderr);

### child.pid

* {Integer}

子进程的 PID。

例子:

    var spawn = require('child_process').spawn,
        grep  = spawn('grep', ['ssh']);

    console.log('Spawned child pid: ' + grep.pid);
    grep.stdin.end();

### child.connected

* {Boolean} 调用 `.disconnect' 后设置为 false

如果 `.connected` 为 false, 消息不再可用。

### child.kill([signal])

* `signal` {String}

发送信号给子进程。如果没有参数，会发送 `'SIGTERM'`，参见 `signal(7)` 里的可用的信号列表。

    var spawn = require('child_process').spawn,
        grep  = spawn('grep', ['ssh']);

    grep.on('close', function (code, signal) {
      console.log('child process terminated due to receipt of signal '+signal);
    });

    // send SIGHUP to process
    grep.kill('SIGHUP');

当信号无法传递的时候会触发 `'error'` 事件。给已经终止的进程发送信号不会触发 `'error'` 事件，但是可以能引起不可预知的后果: 因为有可能 PID (进程 ID) 已经重新分配给其他进程, 信号就会被发送到新的进程里，无法想象这样会引发什么样的事情。

注意，当函数调用 `kill` 信号的时候，它实际并并不会杀死进程，只是发送信号给进程。
  
参见 `kill(2)`

### child.send(message[, sendHandle])

* `message` {Object}
* `sendHandle` {Handle object}

使用 `child_process.fork()` 的时候，你能用 `child.send(message, [sendHandle])` 给子进程写数据，子进程通过 `'message'` 接收消息。

例如:

    var cp = require('child_process');

    var n = cp.fork(__dirname + '/sub.js');

    n.on('message', function(m) {
      console.log('PARENT got message:', m);
    });

    n.send({ hello: 'world' });

子进程的代码 `'sub.js'` :

    process.on('message', function(m) {
      console.log('CHILD got message:', m);
    });

    process.send({ foo: 'bar' });

子进程代码里的 `process` 对象拥有 `send()` 方法，当它通过信道接收到信息时会触发，并返回对象。
  
注意，父进程和子进程 `send()` 是同步的，不要用来发送大块的数据（可以用管道来代替，参见[`child_process.spawn`](#child_process_child_process_spawn_command_args_options))。

不过发送 `{cmd: 'NODE_foo'}` 消息是特殊情况。所有包含 `NODE_` 前缀的消息都不会被触发，因为它们是 node 的内部的核心消息，它们会在 `internalMessage` 事件里触发，尽量避免使用这个特性。

`child.send()` 里的 `sendHandle` 属性用来发送 TCP 服务或 socket 对象给其他的进程，子进程会用接收到的对象作为 `message` 事件的第二个参数。

如果不能发出消息会触发 `'error'` 事件，比如子进程已经退出。

#### 例子: 发送 server 对象

以下是例子:

    var child = require('child_process').fork('child.js');

    // Open up the server object and send the handle.
    var server = require('net').createServer();
    server.on('connection', function (socket) {
      socket.end('handled by parent');
    });
    server.listen(1337, function() {
      child.send('server', server);
    });

子进程将会收到这个 server 对象:

    process.on('message', function(m, server) {
      if (m === 'server') {
        server.on('connection', function (socket) {
          socket.end('handled by child');
        });
      }
    });

注意，现在父子进程共享了server，某些连接会被父进程处理，某些会被子进程处理。

`dgram` 服务器，工作流程是一样的，监听的是 `message` 事件，而不是 `connection`，使用 `server.bind` 而不是 `server.listen`。(目前仅支持 UNIX 平台)

#### 例子: 发送 socket 对象

以下是发送 socket 对象的例子。 他将会创建 2 个子线程，并且同时处理连接，一个将远程地址 `74.125.127.100` 当做 VIP 发送到一个特殊的子进程，另外一个发送到正常进程。
    var normal = require('child_process').fork('child.js', ['normal']);
    var special = require('child_process').fork('child.js', ['special']);

    // Open up the server and send sockets to child
    var server = require('net').createServer();
    server.on('connection', function (socket) {

      // if this is a VIP
      if (socket.remoteAddress === '74.125.127.100') {
        special.send('socket', socket);
        return;
      }
      // just the usual dudes
      normal.send('socket', socket);
    });
    server.listen(1337);

`child.js` 代码如下:

    process.on('message', function(m, socket) {
      if (m === 'socket') {
        socket.end('You were handled as a ' + process.argv[2] + ' person');
      }
    });

注意，当 socket 发送给子进程后，如果这个 socket 被销毁，父进程不再跟踪它，相应的 `.connections` 属性会变为 `null`。这种情况下，不建议使用 `.maxConnections` 。

### child.disconnect()

关闭父子进程间的所有 IPC 通道，能让子进程优雅的退出。调用这个方法后，父子进程里的`.connected` 标志会变为 `false`，之后不能再发送消息。

当进程里没有消息需要处理的时候，会触发 'disconnect' 事件。

注意，在子进程还有 IPC 通道的情况下（如 `fork()` ），也可以调用 `process.disconnect()` 来关闭它。

## 创建异步处理

这些方法遵从常用的异步处理模式（比如回调，或者返回一个事件处理）。

### child_process.spawn(command[, args][, options])

* `command` {String} 要运行的命令
* `args` {Array} 字符串参数表
* `options` {Object}
  * `cwd` {String} 子进程的工作目录
  * `env` {Object} 环境
  * `stdio` {Array|String} 子进程的 stdio 配置。 (见
    [below](#child_process_options_stdio))
  * `customFds` {Array} **Deprecated** 作为子进程 stdio 使用的 文件标示符。(见 [below](#child_process_options_customFds))
  * `detached` {Boolean} 子进程将会变成一个进程组的领导者。(参见
    [below](#child_process_options_detached))
  * `uid` {Number} 设置用户进程的ID。 (参见 setuid(2))
  * `gid` {Number} 设置进程组的ID。 (参见 setgid(2))
* 返回: {ChildProcess object}

用指定的 `command` 发布一个子进程, `args` 是命令行参数。如果忽略, `args` 是空数组。
            
第三个参数用来指定附加设置，默认值:

    { cwd: undefined,
      env: process.env
    }

创建的子进程里使用 `cwd` 指定工作目录，如果没有指定，默认继承自当前的工作目录。

使用 `env` 来指定新进程可见的环境变量。默认是 `process.env`。

例如，运行 `ls -lh /usr`, 获取 `stdout`, `stderr`, 和退出代码:

    var spawn = require('child_process').spawn,
        ls    = spawn('ls', ['-lh', '/usr']);

    ls.stdout.on('data', function (data) {
      console.log('stdout: ' + data);
    });

    ls.stderr.on('data', function (data) {
      console.log('stderr: ' + data);
    });

    ls.on('close', function (code) {
      console.log('child process exited with code ' + code);
    });


例如: 一个非常精巧的方法执行 'ps ax | grep ssh'

    var spawn = require('child_process').spawn,
        ps    = spawn('ps', ['ax']),
        grep  = spawn('grep', ['ssh']);

    ps.stdout.on('data', function (data) {
      grep.stdin.write(data);
    });

    ps.stderr.on('data', function (data) {
      console.log('ps stderr: ' + data);
    });

    ps.on('close', function (code) {
      if (code !== 0) {
        console.log('ps process exited with code ' + code);
      }
      grep.stdin.end();
    });

    grep.stdout.on('data', function (data) {
      console.log('' + data);
    });

    grep.stderr.on('data', function (data) {
      console.log('grep stderr: ' + data);
    });

    grep.on('close', function (code) {
      if (code !== 0) {
        console.log('grep process exited with code ' + code);
      }
    });


### options.stdio

 `stdio` 可能是以下几个参数之一:

* `'pipe'` - `['pipe', 'pipe', 'pipe']`, 默认值
* `'ignore'` - `['ignore', 'ignore', 'ignore']`
* `'inherit'` - `[process.stdin, process.stdout, process.stderr]` 或 `[0,1,2]`

`child_process.spawn()` 里的 'stdio' 参数是一个数组，它和子进程的 fd 相对应，它的值如下:

1. `'pipe'` - 创建在父进程和子进程间的 pipe。管道的父进程端以 `child_process` 的属性形式暴露给父进程，例如 `ChildProcess.stdio[fd]` 。为 fds 0 - 2 创建的管道也可以通过 ChildProcess.stdin, ChildProcess.stdout 和 ChildProcess.stderr 来独立的访问。

2. `'ipc'` - 在父进程和子进程间创建一个 IPC 通道来传递消息/文件描述符。 一个子进程最多有**1个** IPC stdio 文件标识。设置这个选项会激活 ChildProcess.send() 方法。如果子进程向此文件标识写入 JSON 消息，则会触发 ChildProcess.on('message') 。如果子进程是 Node.js 程序，那么 IPC 通道会激活 process.send() 和 process.on('message')。

3. `'ignore'` - 在子进程里不要设置这个文件标识，注意，Node 总会为其 spawn 的进程打开 fd 0-2。如果任何一个被 ignored，node 将会打开 `/dev/null` 并赋给子进程的 fd。
   
4. `Stream` 对象 - 共享一个tty, file, socket, 或刷（pipe）可读或可写流给子进程。该流底层（underlying）的文件标识在子进程中被复制给stdio数组索引对应的文件标识（fd）。
   
5. 正数 - 这个整数被理解为一个在父进程中打开的文件标识，它和子进程共享，就和共享 `Stream` 对象类似。
   
6. `null`, `undefined` - 使用默认值。 对于 stdio fds 0, 1 and 2 (换句话说, stdin, stdout, and stderr) ，pipe管道被建立。 对于 fd 3 及之后, 默认是 `'ignore'`。

例如:

    var spawn = require('child_process').spawn;

    // Child will use parent's stdios
    spawn('prg', [], { stdio: 'inherit' });

    // Spawn child sharing only stderr
    spawn('prg', [], { stdio: ['pipe', 'pipe', process.stderr] });

    // Open an extra fd=4, to interact with programs present a
    // startd-style interface.
    spawn('prg', [], { stdio: ['pipe', null, null, null, 'pipe'] });

### options.detached

如果设置了 `detached` 选项, 子进程将会被作为新进程组的leader，这使得子进程可以在父进程退出后继续运行。

缺省情况下父进程会等 detached 的子进程退出。要阻止父进程等待一个这样的子进程，调用 child.unref() 方法，则父进程的事件循环引用计数中将不会包含这个子进程。

detaching 一个长期运行的进程，并重新将输出指向文件：

     var fs = require('fs'),
         spawn = require('child_process').spawn,
         out = fs.openSync('./out.log', 'a'),
         err = fs.openSync('./out.log', 'a');

     var child = spawn('prg', [], {
       detached: true,
       stdio: [ 'ignore', out, err ]
     });

     child.unref();

使用  `detached`  选项来启动一个长时间运行的进程时，进程不会在后台保持运行，除非他提供了一个不连接到父进程的`stdio` 。如果继承了父进程的  `stdio`，则子进程会继续控制终端。

### options.customFds

已废弃， `customFds` 允许指定特定文件描述符作为子进程的 `stdio`。该 API 无法移植到所有平台，因此被废弃。使用 `customFds` 可以将新进程的 [`stdin`, `stdout`, `stderr`] 钩到已有流上；-1 表示创建新流。自己承担使用风险。

参见: `child_process.exec()` and `child_process.fork()`

### child_process.exec(command[, options], callback)

* `command` {String} 要执行的命令，空格分割
* `options` {Object}
  * `cwd` {String} 子进程的当前工作目录
  * `env` {Object} 环境变量
  * `encoding` {String} (默认: 'utf8')
  * `shell` {String} 运行命令的 shell
    (默认: '/bin/sh'  UNIX, 'cmd.exe'  Windows,  该 shell 必须接收 UNIX上的 `-c` 开关 ，或者 Windows上的`/s /c` 开关 。Windows 上，命令解析必须兼容 `cmd.exe`。)
  * `timeout` {Number} (默认: 0)
  * `maxBuffer` {Number} (默认: `200*1024`)
  * `killSignal` {String} (默认: 'SIGTERM')
  * `uid` {Number} 设置进程里的用户标识。 (见 setuid(2)。)
  * `gid` {Number} 设置进程里的群组标识。(见 setgid(2)。)
* `callback` {Function} 进程终止的时候调用
  * `error` {Error}
  * `stdout` {Buffer}
  * `stderr` {Buffer}
* 返回: ChildProcess 对象

在 shell 里执行命令，并缓冲输出。

    var exec = require('child_process').exec,
        child;

    child = exec('cat *.js bad_file | wc -l',
      function (error, stdout, stderr) {
        console.log('stdout: ' + stdout);
        console.log('stderr: ' + stderr);
        if (error !== null) {
          console.log('exec error: ' + error);
        }
    });

回调参数是 `(error, stdout, stderr)`。 如果成功 , `error`
值为 `null`。 如果失败, `error` 变为 `Error` 的实例， `error.code`
等于子进程退出码, 并且 `error.signal` 会被设置为结束进程的信号名。

第二个参数可以设置一些选项。 缺省是：

    { encoding: 'utf8',
      timeout: 0,
      maxBuffer: 200*1024,
      killSignal: 'SIGTERM',
      cwd: null,
      env: null }

如果 `timeout` 大于 0, 子进程运行时间超过 `timeout` 时会被终止。 `killSignal` (默认: `'SIGTERM'`) 能杀死子进程。 `maxBuffer` 设定了 stdout 或 stderr 的最大数据量，如果子进程的数量量超过了，将会被杀死。

### (file[, args][, options][, callback])

* `file` {String} 要运行的程序的文件名
* `args` {Array} 参数列表
* `options` {Object}
  * `cwd` {String} 子进程的工作目录
  * `env` {Object} 环境
  * `encoding` {String} (默认: 'utf8')
  * `timeout` {Number} (默认: 0)
  * `maxBuffer` {Number} (默认: 200\*1024)
  * `killSignal` {String} (默认: 'SIGTERM')
  * `uid` {Number} 设置进程里的用户标识。 (见 setuid(2)。)
  * `gid` {Number} 设置进程里的群组标识。(见 setgid(2)。)
* `callback` {Function} 进程终止的时候调用
  * `error` {Error}
  * `stdout` {Buffer}
  * `stderr` {Buffer}
* 返回: ChildProcess 对象

和 `child_process.exec()` 类似，不同之处在于这是执行一个指定的文件，因此它比`child_process.exec` 精简些，参数相同。


### child_process.fork(modulePath[, args][, options])

* `modulePath` {String} 子进程里运行的模块
* `args` {Array} 参数列表
* `options` {Object}
  * `cwd` {String} 子进程的工作目录
  * `env` {Object} 环境
  * `execPath` {String} 执行文件路径
  * `execArgv` {Array} 执行参数
    (默认: `process.execArgv`)
  * `silent` {Boolean} 如果是 true ，子进程将会用父进程的 stdin, stdout, and stderr ，否则，将会继承自父进程, 更多细节，参见 `spawn()` 的 `stdio` 参数里的 "pipe" 和 "inherit" 选项(默认 false)
  * `uid` {Number} 设置进程里的用户标识。 (见 setuid(2)。)
  * `gid` {Number} 设置进程里的群组标识。 (见 setgid(2)。)
* 返回: ChildProcess 对象

这是 `spawn()` 的特殊例子，用于派生 Node 进程。除了拥有子进程的所有方法，它的返回对象还拥有内置通讯通道。参见 `child.send(message, [sendHandle])`。

这些 Nodes 是全新的 V8 实例化，假设每个 Node 最少需要 30ms 的启动时间，10mb 的存储空间，可想而知，创建几千个 Node 是不太现实的。

`options` 对象中的 `execPath` 属性可以用于执行文件（非当前 `node` ）创建子进程。这需要小心使用，缺省情况下 fd 表示子进程的 `NODE_CHANNEL_FD` 环境变量。该 fa 的输入和输出是以行分割的 JSON 对象。

## 创建同步进程

以下这些方法是**同步**的，意味着他会**阻塞**事件循环，并暂停执行代码，直到 spawned 的进程退出。

同步方法简化了任务进程，比如大为简化在应用初始化加载/处理过程。

### child_process.spawnSync(command[, args][, options])

* `command` {String} 要执行的命令
* `args` {Array} 参数列表
* `options` {Object}
  * `cwd` {String} 子进程的当前工作目录
  * `input` {String|Buffer} 传递给spawned 进程的值，这个值将会重写 `stdio[0]`
  * `stdio` {Array}  子进程的 stdio 配置。 
  * `env` {Object} 环境变量
  * `uid` {Number}   设置用户进程的ID。 (参见 setuid(2)。)
  * `gid` {Number} 设置进程组的ID。 (参见 setgid(2)。)
  * `timeout` {Number} 子进程运行最大毫秒数。 (默认: undefined)
  * `killSignal` {String} 用来终止子进程的信号。 (默认: 'SIGTERM')
  * `maxBuffer` {Number}
  * `encoding` {String} stdio 输入和输出的编码方式。 (默认: 'buffer')
* 返回: {Object}
  * `pid` {Number} 子进程的 pid
  * `output` {Array} stdio 输出的结果数组
  * `stdout` {Buffer|String}  `output[1]` 的内容
  * `stderr` {Buffer|String} `output[2]` 的内容
  * `status` {Number} 子进程的退出代码
  * `signal` {String} 用来杀死子进程的信号
  * `error` {Error} 子进程错误或超时的错误代码  

`spawnSync` 直到子进程关闭才会返回。超时或者收到 `killSignal` 信号，也不会返回，直到进程完全退出。进程处理完 `SIGTERM` 信号后并不会结束，直到子进程完全退出。


### child_process.execFileSync(command[, args][, options])

* `command` {String} 要执行的命令
* `args` {Array} 参数列表
* `options` {Object}
  * `cwd` {String} 子进程的当前工作目录
  * `input` {String|Buffer}传递给spawned 进程的值，这个值将会重写 `stdio[0]`
  * `stdio` {Array}子进程的 stdio 配置。 (默认: 'pipe')
    - `stderr` 默认情况下会输出给父进程的' stderr 除非指定了 `stdio`
  * `env` {Object} 环境变量
  * `uid` {Number} 设置用户进程的ID。 (参见 setuid(2)。)
  * `gid` {Number} 设置进程组的ID。 (参见 setgid(2)。)
  * `timeout` {Number} 进程运行最大毫秒数。 (默认: undefined)
  * `killSignal` {String} 用来终止子进程的信号。 (默认: 'SIGTERM')
  * `maxBuffer` {Number}
  * `encoding` {String} stdio 输入和输出的编码方式。 (默认: 'buffer')
* 返回: {Buffer|String} 来自命令的 stdout 

直到子进程完全退出，`execFileSync` 才会返回。超时或者收到 `killSignal` 信号，也不会返回，直到进程完全退出。进程处理完 `SIGTERM` 信号后并不会结束，直到子进程完全退出。

如果进程超时，或者非正常退出，这个方法将会抛出异常。`Error` 会包含整个 [`child_process.spawnSync`](#child_process_child_process_spawnsync_command_args_options)结果。


### child_process.execSync(command[, options])

* `command` {String} 要执行的命令
* `options` {Object}
  * `cwd` {String} 子进程的当前工作目录
  * `input` {String|Buffer} 传递给spawned 进程的值，这个值将会重写 `stdio[0]`
  * `stdio` {Array} 子进程的 stdio 配置。 (默认: 'pipe')
    - `stderr` 默认情况下会输出给父进程的' stderr 除非指定了 `stdio`
  * `env` {Object} 环境变量
  * `uid` {Number} 设置用户进程的ID。 (参见 setuid(2)。)
  * `gid` {Number} 设置进程组的ID。 (参见 setgid(2)。)
  * `timeout` {Number} 进程运行最大毫秒数。 (默认: undefined)
  * `killSignal` {String} 用来终止子进程的信号。 (默认: 'SIGTERM')
  * `maxBuffer` {Number}
  * `encoding` {String} stdio 输入和输出的编码方式。 (默认: 'buffer')
* 返回: {Buffer|String} 来自命令的 stdout 

直到子进程完全退出，`execSync` 才会返回。超时或者收到 `killSignal` 信号，也不会返回，直到进程完全退出。进程处理完 `SIGTERM`  信号后并不会结束，直到子进程完全退出。

如果进程超时，或者非正常退出，这个方法将会抛出异常。`Error` 会包含整个[`child_process.spawnSync`](#child_process_child_process_spawnsync_command_args_options)结果。

[EventEmitter]: events.html#events_class_events_eventemitter
