# 调试器

    稳定性: 3 - 稳定

<!-- type=misc -->

V8 提供了强大的调试工具，可以通过 [TCP protocol](http://code.google.com/p/v8/wiki/DebuggerProtocol) 从外部访问。Node 内置这个调试工具客户端。要使用这个调试器，以`debug`参数启动 Node，出现提示：

    % node debug myscript.js
    < debugger listening on port 5858
    connecting... ok
    break in /home/indutny/Code/git/indutny/myscript.js:1
      1 x = 5;
      2 setTimeout(function () {
      3   debugger;
    debug>

Node 的调试器不支持所有的命令，但是简单的步进和检查还是可以的。在代码里嵌入 `debugger;`，可以设置断点。

例如,  `myscript.js` 代码如下：

    // myscript.js
    x = 5;
    setTimeout(function () {
      debugger;
      console.log("world");
    }, 1000);
    console.log("hello");

如果启动 debugger，它会断在第四行：

    % node debug myscript.js
    < debugger listening on port 5858
    connecting... ok
    break in /home/indutny/Code/git/indutny/myscript.js:1
      1 x = 5;
      2 setTimeout(function () {
      3   debugger;
    debug> cont
    < hello
    break in /home/indutny/Code/git/indutny/myscript.js:3
      1 x = 5;
      2 setTimeout(function () {
      3   debugger;
      4   console.log("world");
      5 }, 1000);
    debug> next
    break in /home/indutny/Code/git/indutny/myscript.js:4
      2 setTimeout(function () {
      3   debugger;
      4   console.log("world");
      5 }, 1000);
      6 console.log("hello");
    debug> repl
    Press Ctrl + C to leave debug repl
    > x
    5
    > 2+2
    4
    debug> next
    < world
    break in /home/indutny/Code/git/indutny/myscript.js:5
      3   debugger;
      4   console.log("world");
      5 }, 1000);
      6 console.log("hello");
      7
    debug> quit
    %


 `repl` 命令能执行远程代码；`next` 能步进到下一行。此外可以输入 `help` 查看哪些命令可用。

## 监视器-Watchers

调试的时候可以查看表达式和变量。每个断点处，监视器都会显示上下文。  

输入  `watch("my_expression")` 开始监视表达式，`watchers` 显示活跃的监视器。输入`unwatch("my_expression")` 可以移除监视器。  

## 命令参考-Commands reference

### 步进-Stepping

* `cont`, `c` - 继续执行
* `next`, `n` - Step next
* `step`, `s` - Step in
* `out`, `o` - Step out
* `pause` - 暂停 (类似开发工具的暂停按钮)

### 断点Breakpoints

* `setBreakpoint()`, `sb()` - 当前行设置断点
* `setBreakpoint(line)`, `sb(line)` - 在指定行设置断点
* `setBreakpoint('fn()')`, `sb(...)` - 在函数里的第一行设置断点
* `setBreakpoint('script.js', 1)`, `sb(...)` - 在 script.js 第一行设置断点。
* `clearBreakpoint`, `cb(...)` - 清除断点

也可以在尚未加载的文件里设置断点。

    % ./node debug test/fixtures/break-in-module/main.js
    < debugger listening on port 5858
    connecting to port 5858... ok
    break in test/fixtures/break-in-module/main.js:1
      1 var mod = require('./mod.js');
      2 mod.hello();
      3 mod.hello();
    debug> setBreakpoint('mod.js', 23)
    Warning: script 'mod.js' was not loaded yet.
      1 var mod = require('./mod.js');
      2 mod.hello();
      3 mod.hello();
    debug> c
    break in test/fixtures/break-in-module/mod.js:23
     21
     22 exports.hello = function() {
     23   return 'hello from module';
     24 };
     25
    debug>

### 信息Info

* `backtrace`, `bt` - 打印当前执行框架的backtrace
* `list(5)` - 显示脚本代码的 5 行上下文（之前 5 行和之后 5 行）
* `watch(expr)` - 监视列表里添加表达式
* `unwatch(expr)` - 从监视列表里删除表达式
* `watchers` - 显示所有的监视器和它们的值（每个断点都会自动列出）  
* `repl` - 在所调试的脚本的上下文中，打开调试器的 repl 
  
### 执行控制Execution control

* `run` - 运行脚本 (开始调试的时候自动运行)
* `restart` - 重新运行脚本
* `kill` - 杀死脚本

### 杂项Various

* `scripts` - 列出所有已经加载的脚本
* `version` - 显示 v8 版本

## 高级应用Advanced Usage

V8 调试器可以用两种方法启用和访问，`--debug`命令启动调试，或向已经启动 Node 发送 `SIGUSR1`。

一旦一个进程进入调试模式，它可以被 node 调试器连接。调试器可以通过`pid` 或 URI 来连接。

* `node debug -p <pid>` - 通过 `pid` 连接进程
* `node debug <URI>` - 通过 URI （比如localhost:5858） 连接进程w
