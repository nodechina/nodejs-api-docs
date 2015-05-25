# 进程

 `process` 是全局对象，能够在任意位置访问，是 [EventEmitter][] 的实例。

## 退出状态码

当没有新的异步的操作等待处理时，Node 正常情况下退出时会返回状态码 `0` 。下面的状态码表示其他状态：

* `1` **未捕获的致命异常-Uncaught Fatal Exception** - 有未捕获异常，并且没有被域或 `uncaughtException` 处理函数处理。  
* `2` - Unused (保留)
* `3` **JavaScript解析错误-Internal JavaScript Parse Error** - JavaScript的源码启动 Node 进程时引起解析错误。非常罕见，仅会在开发 Node 时才会有。   
* `4` **JavaScript评估失败-Internal JavaScript Evaluation Failure** - JavaScript的源码启动 Node 进程，评估时返回函数失败。非常罕见，仅会在开发 Node 时才会有。  
* `5` **致命错误-Fatal Error** - V8 里致命的不可恢复的错误。通常会打印到 stderr ，内容为： `FATAL ERROR`
* `6` **Non-function 异常处理-Non-function Internal Exception Handler** - 未捕获异常，内部异常处理函数不知为何设置为on-function，并且不能被调用。
* `7` **异常处理函数运行时失败-Internal Exception Handler Run-Time Failure** - 未捕获的异常， 并且异常处理函数处理时自己抛出了异常。例如，如果 `process.on('uncaughtException')` 或 `domain.on('error')` 抛出了异常。  
* `8` - Unused保留.  之前版本的 Node， 8 有时表示未捕获异常。  
* `9` - **参数非法-Invalid Argument** - 可能是给了未知的参数，或者给的参数没有值。
* `10` **运行时失败-Internal JavaScript Run-Time Failure** - JavaScript的源码启动 Node 进程时抛出错误，非常罕见，仅会在开发 Node 时才会有。
* `12` **无效的 Debug 参数-Invalid Debug Argument** - 设置了参数`--debug` 和/或 `--debug-brk`，但是选择了错误端口。
* `>128` **信号退出-Signal Exits** - 如果 Node 接收到致命信号，比如`SIGKILL` 或 `SIGHUP`，那么退出代码就是`128` 加信号代码。这是标准的 Unix 做法，退出信号代码放在高位。

## 事件: 'exit'

当进程准备退出时触发。此时已经没有办法阻止从事件循环中推出。因此，你必须在处理函数中执行同步操作。这是一个在固定事件检查模块状态（比如单元测试）的好时机。回调函数有一个参数，它是进程的退出代码。

监听 `exit` 事件的例子:

    process.on('exit', function(code) {
      // do *NOT* do this
      setTimeout(function() {
        console.log('This will not run');
      }, 0);
      console.log('About to exit with code:', code);
    });


## 事件: 'beforeExit'

当 node 清空事件循环，并且没有其他安排时触发这个事件。通常来说，当没有进程安排时 node 退出，但是  'beforeExit' 的监听器可以异步调用，这样 node 就会继续执行。

'beforeExit' 并不是明确退出的条件，`process.exit()` 或异常捕获才是，所以不要把它当做'exit' 事件。除非你想安排更多的工作。


## 事件: 'uncaughtException'

当一个异常冒泡回到事件循环，触发这个事件。如果给异常添加了监视器，默认的操作（打印堆栈跟踪信息并退出）就不会发生。

监听 `uncaughtException` 的例子:

    process.on('uncaughtException', function(err) {
      console.log('Caught exception: ' + err);
    });

    setTimeout(function() {
      console.log('This will still run.');
    }, 500);

    // Intentionally cause an exception, but don't catch it.
    nonexistentFunc();
    console.log('This will not run.');

注意，`uncaughtException` 是非常简略的异常处理机制。

尽量不要使用它，而应该用 [domains](domain.html) 。如果你用了，每次未处理异常后，重启你的程序。

不要使用 node.js 里诸如 `On Error Resume Next` 这样操作。每个未处理的异常意味着你的程序，和你的 node.js 扩展程序，一个未知状态。盲目的恢复意味着 *任何事情* 都可能发生


你在升级的系统时拉掉了电源线，然后恢复了。可能10次里有9次没有问题，但是第10次，你的系统可能就会挂掉。

## Signal 事件

<!--type=event-->
<!--name=SIGINT, SIGHUP, etc.-->

当进程接收到信号时就触发。信号列表详见标准的 POSIX 信号名，如 SIGINT、SIGUSR1 等


监听 `SIGINT` 的例子:

    // Start reading from stdin so we don't exit.
    process.stdin.resume();

    process.on('SIGINT', function() {
      console.log('Got SIGINT.  Press Control-D to exit.');
    });

在大多数终端程序里，发送 `SIGINT` 信号的简单方法是按下 `信号Control-C`。

注意:

- `SIGUSR1` node.js 接收这个信号开启调试模式。可以安装一个监听器，但开始时不会中断调试。  
- `SIGTERM` 和 `SIGINT` 在非 Windows 系统里，有默认的处理函数，退出（伴随退出代码 `128 + 信号码`）前，重置退出模式。如果这些信号有监视器，默认的行为将会被移除。  
- `SIGPIPE` 默认情况下忽略，可以加监听器。
- `SIGHUP` 当 Windowns 控制台关闭的时候生成，其他平台的类似条件，参见signal(7)。可以添加监听者，Windows 平台上 10 秒后会无条件退出。在非 Windows 平台上，`SIGHUP` 的默认操作是终止 node，但是一旦添加了监听器，默认动作将会被移除。  `SIGHUP` is to terminate node, but once a listener has been installed its
- `SIGTERM`  Windows 不支持, 可以被监听。
- `SIGINT` 所有的终端都支持，通常由`CTRL+C` 生成（可能需要配置）。当终端原始模式启用后不会再生成。  
- `SIGBREAK` Windows 里，按下 `CTRL+BREAK` 会发送。非 Windows 平台，可以被监听，但是不能发送或生成。  
- `SIGWINCH` - 当控制台被重设大小时发送。Windows 系统里，仅会在控制台上输入内容时，光标移动，或者可读的 tty在原始模式上使用。 
- `SIGKILL` 不能有监视器，在所有平台上无条件关闭 node。  
- `SIGSTOP` 不能有监视器。

Windows 不支持发送信号，但是 node 提供了很多`process.kill()` 和 `child_process.kill()` 的模拟：
- 发送 Sending 信号 `0` 可以查找运行中得进程
- 发送  `SIGINT`, `SIGTERM`, 和 `SIGKILL` 会引起目标进程无条件退出。

## process.stdout

一个 `Writable Stream` 执向 `stdout` (on fd `1`).

例如: `console.log`的定义：

    console.log = function(d) {
      process.stdout.write(d + '\n');
    };

`process.stderr` 和 `process.stdout` 和 node 里的其他流不同，他们不会被关闭（`end()` 将会被抛出），它们不会触发 `finish` 事件，并且写是阻塞的。

- 引用指向常规文件或 TTY 文件描述符时，是阻塞的。
- 引用指向 pipe 管道时:
  - 在 Linux/Unix 里阻塞.
  - 在 Windows 像其他流一样，不被阻塞

检查 Node  是否运行在 TTY 上下文中，从`process.stderr`, `process.stdout`, 或 `process.stdin`里读取 `isTTY` 属性。

    $ node -p "Boolean(process.stdin.isTTY)"
    true
    $ echo "foo" | node -p "Boolean(process.stdin.isTTY)"
    false

    $ node -p "Boolean(process.stdout.isTTY)"
    true
    $ node -p "Boolean(process.stdout.isTTY)" | cat
    false

更多信息参见 [the tty docs](tty.html#tty_tty)。

## process.stderr

一个指向 stderr (on fd `2`)的可写流。

`process.stderr` 和 `process.stdout` 和 node 里的其他流不同，他们不会被关闭（`end()` 将会被抛出），它们不会触发 `finish` 事件，并且写是阻塞的。

- 引用指向常规文件或 TTY 文件描述符时，是阻塞的。
- 引用指向 pipe 管道时:
  - 在 Linux/Unix 里阻塞.
  - 在 Windows 像其他流一样，不被阻塞


## process.stdin

一个指向 stdin (on fd `0`)的可读流。


以下例子：打开标准输入流，并监听两个事件:

    process.stdin.setEncoding('utf8');

    process.stdin.on('readable', function() {
      var chunk = process.stdin.read();
      if (chunk !== null) {
        process.stdout.write('data: ' + chunk);
      }
    });

    process.stdin.on('end', function() {
      process.stdout.write('end');
    });

`process.stdin` 可以工作在老模式里，和 v0.10 之前版本的 node 代码兼容。

更多信息参见[Stream compatibility](stream.html#stream_compatibility_with_older_node_versions).

在老的流模式里，stdin流默认暂停，必须调用 `process.stdin.resume()` 读取。可以调用 `process.stdin.resume()` 切换到老的模式。

如果开始一个新的工程，最好选择新的流，而不是用老的流。

## process.argv

包含命令行参数的数组。第一个元素是'node'，第二个参数是 JavaScript 文件的名字，第三个参数是任意的命令行参数。

    // print process.argv
    process.argv.forEach(function(val, index, array) {
      console.log(index + ': ' + val);
    });

将会生成:

    $ node process-2.js one two=three four
    0: node
    1: /Users/mjr/work/node/process-2.js
    2: one
    3: two=three
    4: four


## process.execPath

开启当前进程的执行文件的绝对路径。

例子:

    /usr/local/bin/node


## process.execArgv

启动进程所需的 node 命令行参数。这些参数不会在 `process.argv` 里出现，并且不包含 node 执行文件的名字，或者任何在名字之后的参数。这些用来生成子进程，使之拥有和父进程有相同的参数。

例子:

    $ node --harmony script.js --version

process.execArgv 的参数:

    ['--harmony']

process.argv 的参数:

    ['/usr/local/bin/node', 'script.js', '--version']


## process.abort()

这将导致 node 触发 abort 事件。会让 node 退出并生成一个核心文件。

## process.chdir(directory)

改变当前工作进程的目录，如果操作失败抛出异常。

    console.log('Starting directory: ' + process.cwd());
    try {
      process.chdir('/tmp');
      console.log('New directory: ' + process.cwd());
    }
    catch (err) {
      console.log('chdir: ' + err);
    }



## process.cwd()

返回当前进程的工作目录

    console.log('Current directory: ' + process.cwd());


## process.env

包含用户环境的对象，参见 environ(7).

这个对象的例子：

    { TERM: 'xterm-256color',
      SHELL: '/usr/local/bin/bash',
      USER: 'maciej',
      PATH: '~/.bin/:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin',
      PWD: '/Users/maciej',
      EDITOR: 'vim',
      SHLVL: '1',
      HOME: '/Users/maciej',
      LOGNAME: 'maciej',
      _: '/usr/local/bin/node' }

你可以写入这个对象，但是不会改变当前运行的进程。以下的命令不会成功：

    node -e 'process.env.foo = "bar"' && echo $foo

这个会成功:

    process.env.foo = 'bar';
    console.log(process.env.foo);


## process.exit([code])

使用指定的 code 结束进程。如果忽略，将会使用 code  `0`

使用失败的代码退出：

    process.exit(1);

Shell 将会看到退出代码为1.


## process.exitCode

进程退出时的代码，如果进程优雅的退出，或者通过 `process.exit()` 退出，不需要指定退出码。

设定 `process.exit(code)` 将会重写之前设置的 `process.exitCode`。


## process.getgid()

注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。

获取进程的群组标识（参见 getgid(2)）。获取到得时群组的数字 id，而不是名字。

    if (process.getgid) {
      console.log('Current gid: ' + process.getgid());
    }


## process.setgid(id)

注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。

设置进程的群组标识（参见 setgid(2)）。可以接收数字 ID 或者群组名。如果指定了群组名，会阻塞等待解析为数字 ID 。

    if (process.getgid && process.setgid) {
      console.log('Current gid: ' + process.getgid());
      try {
        process.setgid(501);
        console.log('New gid: ' + process.getgid());
      }
      catch (err) {
        console.log('Failed to set gid: ' + err);
      }
    }


## process.getuid()

注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。

获取进程的用户标识(参见 getuid(2))。这是数字的用户 id，不是用户名

    if (process.getuid) {
      console.log('Current uid: ' + process.getuid());
    }


## process.setuid(id)

注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。

设置进程的用户标识（参见setuid(2)）。接收数字 ID或字符串名字。果指定了群组名，会阻塞等待解析为数字 ID 。

    if (process.getuid && process.setuid) {
      console.log('Current uid: ' + process.getuid());
      try {
        process.setuid(501);
        console.log('New uid: ' + process.getuid());
      }
      catch (err) {
        console.log('Failed to set uid: ' + err);
      }
    }


## process.getgroups()

注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。

返回进程的群组 iD 数组。POSIX 系统没有保证一定有，但是 node.js 保证有。


## process.setgroups(groups)

注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。

设置进程的群组 ID。这是授权操作，所有你需要有 root 权限，或者有 CAP_SETGID 能力。

列表可以包含群组 IDs，群组名，或者两者都有。


## process.initgroups(user, extra_group)

注意：这个函数仅在 POSIX 平台上可用(例如，非Windows 和 Android)。

读取 /etc/group ，并初始化群组访问列表，使用成员所在的所有群组。这是授权操作，所有你需要有 root 权限，或者有 CAP_SETGID 能力。

`user` 是用户名或者用户 ID， `extra_group` 是群组名或群组 ID。

当你在注销权限 (dropping privileges) 的时候需要注意. 例子:

    console.log(process.getgroups());         // [ 0 ]
    process.initgroups('bnoordhuis', 1000);   // switch user
    console.log(process.getgroups());         // [ 27, 30, 46, 1000, 0 ]
    process.setgid(1000);                     // drop root gid
    console.log(process.getgroups());         // [ 27, 30, 46, 1000 ]


## process.version

一个编译属性，包含 `NODE_VERSION`.

    console.log('Version: ' + process.version);

## process.versions

一个属性，包含了 node 的版本和依赖.

    console.log(process.versions);

打印出来：

    { http_parser: '1.0',
      node: '0.10.4',
      v8: '3.14.5.8',
      ares: '1.9.0-DEV',
      uv: '0.10.3',
      zlib: '1.2.3',
      modules: '11',
      openssl: '1.0.1e' }

## process.config

一个包含用来编译当前 node 执行文件的 javascript 配置选项的对象。它与运行 ./configure 脚本生成的 "config.gypi" 文件相同。


一个可能的输出:

    { target_defaults:
       { cflags: [],
         default_configuration: 'Release',
         defines: [],
         include_dirs: [],
         libraries: [] },
      variables:
       { host_arch: 'x64',
         node_install_npm: 'true',
         node_prefix: '',
         node_shared_cares: 'false',
         node_shared_http_parser: 'false',
         node_shared_libuv: 'false',
         node_shared_v8: 'false',
         node_shared_zlib: 'false',
         node_use_dtrace: 'false',
         node_use_openssl: 'true',
         node_shared_openssl: 'false',
         strict_aliasing: 'true',
         target_arch: 'x64',
         v8_use_snapshot: 'true' } }

## process.kill(pid[, signal])

发送信号给进程. `pid` 是进程id，并且 `signal` 是发送的信号的字符串描述。信号名是字符串，比如
'SIGINT' 或 'SIGHUP'。如果忽略，信号会是 'SIGTERM'.更多信息参见 [Signal 事件](#process_signal_事件) 和 kill(2) .

如果进程没有退出，会抛出错误。信号 `0` 可以用来测试进程是否存在。

注意，虽然这个这个函数名叫`process.kill`，它真的仅是信号发射器，就像`kill`  系统调用。信号发射可以做其他事情，不仅是杀死目标进程。

例子， 给自己发信号:

    process.on('SIGHUP', function() {
      console.log('Got SIGHUP signal.');
    });

    setTimeout(function() {
      console.log('Exiting.');
      process.exit(0);
    }, 100);

    process.kill(process.pid, 'SIGHUP');

注意: 当 Node.js 接收到 SIGUSR1 信号，它会开启 debugger 调试模式, 参见
[Signal Events](#process_signal_events).

## process.pid

当前进程的 PID

    console.log('This process is pid ' + process.pid);


## process.title

获取/设置(Getter/setter) 'ps' 中显示的进程名。

使用 setter 时，字符串的长度由系统指定，可能会很短。

在 Linux 和 OS X 上，它受限于名称的长度加上命令行参数的长度，因为它会覆盖参数内存(argv memory)。

v0.8 版本允许更长的进程标题字符串，也支持覆盖环境内存，但是存在潜在的不安全和混乱（很难说清楚）。


## process.arch

当前 CPU 的架构：'arm'、'ia32' 或者 'x64'.

    console.log('This processor architecture is ' + process.arch);


## process.platform

运行程序所在的平台系统 `'darwin'`, `'freebsd'`, `'linux'`, `'sunos'` or `'win32'`

    console.log('This platform is ' + process.platform);


## process.memoryUsage()

返回一个对象，描述了 Node 进程所用的内存状况，单位为字节。

    var util = require('util');

    console.log(util.inspect(process.memoryUsage()));

将会生成:

    { rss: 4935680,
      heapTotal: 1826816,
      heapUsed: 650472 }

`heapTotal` and `heapUsed` refer to V8's memory usage.


## process.nextTick(callback)

* `callback` {Function}

一旦当前事件循环结束，调用回到函数。

这不是 `setTimeout(fn, 0)` 的简单别名，这个效率更高。在任何附加 I/O 事件在子队列事件循环中触发前，它就会运行。

    console.log('start');
    process.nextTick(function() {
      console.log('nextTick callback');
    });
    console.log('scheduled');
    // Output:
    // start
    // scheduled
    // nextTick callback

在对象构造后，在 I/O 事件发生前，你又想改变附加事件处理函数时，这个非常有用。

    function MyThing(options) {
      this.setupOptions(options);

      process.nextTick(function() {
        this.startDoingStuff();
      }.bind(this));
    }

    var thing = new MyThing();
    thing.getReadyForStuff();

    // thing.startDoingStuff() gets called now, not before.

要保证你的函数一定是 100% 同步执行，或者 100% 异步执行。例子:

    // WARNING!  DO NOT USE!  BAD UNSAFE HAZARD!
    function maybeSync(arg, cb) {
      if (arg) {
        cb();
        return;
      }

      fs.stat('file', cb);
    }

这个 API 非常危险.  如果你这么做:

    maybeSync(true, function() {
      foo();
    });
    bar();

不清楚`foo()` 或 `bar()` 哪个先执行。

更好的方法：

    function definitelyAsync(arg, cb) {
      if (arg) {
        process.nextTick(cb);
        return;
      }

      fs.stat('file', cb);
    }

注意：nextTick 队列会在完全执行完毕之后才调用 I/O 操作。因此，递归设置 nextTick 的回调就像一个 `while(true);` 循环一样，将会阻止任何 I/O 操作的发生。

## process.umask([mask])

设置或读取进程文件的掩码。子进程从父进程继承掩码。如果`mask` 参数有效，返回旧的掩码。否则，返回当前掩码。  

    var oldmask, newmask = 0022;

    oldmask = process.umask(newmask);
    console.log('Changed umask from: ' + oldmask.toString(8) +
                ' to ' + newmask.toString(8));


## process.uptime()

返回 Node 已经运行的秒数。


## process.hrtime()

返回当前进程的高分辨时间，形式为 `[seconds, nanoseconds]`数组。它是相对于过去的任意事件。该值与日期无关，因此不受时钟漂移的影响。主要用途是可以通过精确的时间间隔，来衡量程序的性能。

你可以将之前的结果传递给当前的 `process.hrtime()` ，会返回两者间的时间差，用来基准和测量时间间隔。

    var time = process.hrtime();
    // [ 1800216, 25 ]

    setTimeout(function() {
      var diff = process.hrtime(time);
      // [ 1, 552 ]

      console.log('benchmark took %d nanoseconds', diff[0] * 1e9 + diff[1]);
      // benchmark took 1000000527 nanoseconds
    }, 1000);


## process.mainModule

 [`require.main`](modules.html#modules_accessing_the_main_module) 的备选方法。不同点，如果主模块在运行时改变，`require.main`可能会继续返回老的模块。可以认为，这两者引用了同一个模块。

Alternate way to retrieve
[`require.main`](modules.html#modules_accessing_the_main_module).
The difference is that if the main module changes at runtime, `require.main`
might still refer to the original main module in modules that were required
before the change occurred. Generally it's safe to assume that the two refer
to the same module.

和 `require.main` 一样, 如果没有入口脚本，将会返回`undefined` 。

[EventEmitter]: events.html#events_class_events_eventemitter
