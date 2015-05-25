# REPL

    稳定性: 3 - 稳定

Read-Eval-Print-Loop (REPL 读取-执行-输出循环)即可作为独立程序，也可以集成到其他程序中。REPL 提供了一种交互的执行 JavaScript 并查看输出结果的方法。可以用来调试，测试，或仅是用来试试。

在命令行中不带任何参数的执行  `node` ，就是 REPL 模式。它提供了简单的 emacs 行编辑。

    mjr:~$ node
    Type '.help' for options.
    > a = [ 1, 2, 3];
    [ 1, 2, 3 ]
    > a.forEach(function (v) {
    ...   console.log(v);
    ...   });
    1
    2
    3

若想使用高级的编辑模式，使用环境变量 `NODE_NO_READLINE=1` 打开 node。这样会开启 REPL 模式，允许你使用 `rlwrap`。

例如，你可以添加以下代码到你的 bashrc 文件里。

    alias node="env NODE_NO_READLINE=1 rlwrap node"


## repl.start(options)

启动并返回一个 `REPLServer` 实例。它继承自[Readline Interface][]。接收的参数 "options"  有以下值：

 - `prompt` - 所有输入输出的提示符和流。默认是 `> `.

 - `input` - 需要监听的可读流，默认 `process.stdin`.

 - `output` - 用来输出数据的可写流，默认为 `process.stdout`.

 - `terminal` - 如果 `stream`  被当成 TTY，并且有 ANSI/VT100 转义， 传`true`。默认在实例的输出流上检查`isTTY`。

 - `eval` - 用来对每一行进行求值的函数。默认为 `eval()` 的异步封装。参见后面的自定义 `eval`例子。

 - `useColors` - 写函数输出是否有颜色。如果设定了不同的  `writer` 函数则无效。默认为 repl 的 `terminal` 值。

 - `useGlobal` - 如果为 `true` ，则 repl 将会使用全局对象，而不是在独立的上下文中运行scripts。默认为 `false`。 

 - `ignoreUndefined` - 如果为 `true`，repl 不会输出未定义命令的返回值。默认为 `false`。  

 - `writer` - 每个命令行被求值时都会调用这个函数，它会返回格式化显示内容（包括颜色）。默认是 `util.inspect`。

如果有以下特性，可以使用自己的 `eval`函数：

    function eval(cmd, context, filename, callback) {
      callback(null, result);
    }

在同一个 node 的运行实例上，可以打开多个 REPLs。每个都会共享一个全局对象，但会有独立的 I/O。

以下的例子，在stdin,  Unix socket, 和  TCP socket 上开启 REPL ：

    var net = require("net"),
        repl = require("repl");

    connections = 0;

    repl.start({
      prompt: "node via stdin> ",
      input: process.stdin,
      output: process.stdout
    });

    net.createServer(function (socket) {
      connections += 1;
      repl.start({
        prompt: "node via Unix socket> ",
        input: socket,
        output: socket
      }).on('exit', function() {
        socket.end();
      })
    }).listen("/tmp/node-repl-sock");

    net.createServer(function (socket) {
      connections += 1;
      repl.start({
        prompt: "node via TCP socket> ",
        input: socket,
        output: socket
      }).on('exit', function() {
        socket.end();
      });
    }).listen(5001);

从命令行运行这个程序，将会在 stdin 上启动 REPL。其他的 REPL 客户端可能通过 Unix socket 或 TCP socket 连接。`telnet` 常用于连接 TCP socket， `socat` 用于连接Unix 和 TCP sockets

从Unix socket-based 服务器启动 REPL（而非stdin），你可以建立长连接，不用重启它们。

通过 `net.Server` 和 `net.Socket` 实例运行"full-featured" (`terminal`) REPL的例子，参见: https://gist.github.com/2209310

通过 `curl(1)` 实例运行 REPL的例子，参见: https://gist.github.com/2053342

### Event: 'exit'

`function () {}`

当用户通过预定义的方式退出 REPL 将会触发这个事件。预定义的方式包括，在 repl 里输入 `.exit`，按 Ctrl+C 两次来发送 SIGINT 信号，或者在 `input` 流上按 Ctrl+D 来发送"end"。


监听 `exit` 的例子：

    r.on('exit', function () {
      console.log('Got "exit" event from repl!');
      process.exit();
    });


### Event: 'reset'

`function (context) {}`

重置 REPL 的上下文的时候触发。当你输入`.clear`会重置。如果你用 `{ useGlobal: true }` 启动 repl，那这个事件永远不会被触发。


监听 `reset` 的例子：

    // Extend the initial repl context.
    r = repl.start({ options ... });
    someExtension.extend(r.context);

    // When a new context is created extend it as well.
    r.on('reset', function (context) {
      console.log('repl has a new context');
      someExtension.extend(context);
    });


## REPL 特性

<!-- type=misc -->

在 REPL 里，  Control+D 会退出。可以输入多行表达式。支持全局变量和局部变量的 TAB 自动补全。

特殊变量`_` (下划线)包含上一个表达式的结果。

    > [ "a", "b", "c" ]
    [ 'a', 'b', 'c' ]
    > _.length
    3
    > _ += 1
    4

REPL支持在全局域里访问任何变量。将变量赋值个和`REPLServer`关联的上下文对象，你可以显示的讲变量暴露给 REPL，例如：

    // repl_test.js
    var repl = require("repl"),
        msg = "message";

    repl.start("> ").context.m = msg;

`context` 对象里的东西，会以局部变量的形式出现：

    mjr:~$ node repl_test.js
    > m
    'message'

有一些特殊的REPL命令：

  - `.break` - 当你输入多行表达式时，也许你走神了或者不想完成了，`.break` 可以重新开始。  
  - `.clear` - 重置 `context`  对象为空对象，并且清空多行表达式。  
  - `.exit` - 关闭输入/输出流，会让 REPL 退出。
  - `.help` - 打印这些特殊命令。
  - `.save` - 保存当前 REPL 会话到文件。
    >.save ./file/to/save.js
  - `.load` - 加载一个文件到当前REPL 会话
    >.load ./file/to/load.js

下面的组合键在 REPL 中有以下效果：

  - `<ctrl>C` - 和 `.break` 键类似. 在一个空行连按两次会强制退出。
  - `<ctrl>D` - 和 `.exit` 键类似。

