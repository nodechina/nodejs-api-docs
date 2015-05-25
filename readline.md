# 逐行读取

    稳定性: 2 - 不稳定

使用 `require('readline')`，可以使用这个模块。逐行读取（Readline）可以逐行读取流（比如`process.stdin`）

一旦你开启了这个模块，node 程序将不会终止，直到你关闭接口。以下的代码展示了如何优雅的退出程序：

    var readline = require('readline');

    var rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    });

    rl.question("What do you think of node.js? ", function(answer) {
      // TODO: Log the answer in a database
      console.log("Thank you for your valuable feedback:", answer);

      rl.close();
    });

## readline.createInterface(options)

创建一个逐行读取（Readline） `Interface` 实例. 参数 "options" 对象有以下值:

 - `input` - 监听的可读流 (必填).

 - `output` - 逐行读取（Readline）数据要写入的可写流(可选).

 - `completer` - 用于 Tab 自动补全的可选函数。参见下面的例子。

 - `terminal` - 如果希望和 TTY 一样，对待  `input` 和 `output` 流，设置为 true。 并且由 ANSI/VT100 转码。默认情况下，检查  `isTTY` 是否在 `output` 流上实例化。

`completer` 给出当前行的入口，应该返回包含2条记录的数组

 1. 一个匹配当前输入补全的字符串数组

 2. 用来匹配的子字符串

最终像这样:`[[substr1, substr2, ...], originalsubstring]`.

例子：

    function completer(line) {
      var completions = '.help .error .exit .quit .q'.split(' ')
      var hits = completions.filter(function(c) { return c.indexOf(line) == 0 })
      // show all completions if none found
      return [hits.length ? hits : completions, line]
    }

同时， `completer` 可以异步运行，此时接收到2个参数:

    function completer(linePartial, callback) {
      callback(null, [['123'], linePartial]);
    }

为了接受用户输入，`createInterface` 通常和  `process.stdin` ，`process.stdout`一起使用：

    var readline = require('readline');
    var rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    });

如果你有逐行读取（Readline）实例, 通常会监听`"line"` 事件.

如果这个实例参数 `terminal` = `true`，而且定义了 `output.columns` 属性，那么 `output` 流将会最佳兼容性，并且，当 columns 变化时（当它是 TTY 时，`process.stdout` 会自动这么做），会在 `output`  流上触发 `"resize"` 事件。


## Class: Interface

代表一个包含输入/输出流的逐行读取（Readline）接口的类，

### rl.setPrompt(prompt)

设置提示符，比如当你再命令行里运行 `node` 时，可以看到 node 的提示符 `> `。

### rl.prompt([preserveCursor])

为用户输入准备好逐行读取（Readline），将当前 `setPrompt` 选项方法哦新的行中，让用户有新的地方输入。设置 `preserveCursor` 为 `true`，防止当前的游标重置为  `0`。

如果暂停，使用 `createInterface`也可以重置`input` 输入流。
  
调用 `createInterface` 时，如果 `output` 设置为 `null` 或 `undefined` ，不会重新写提示符。  

### rl.question(query, callback)

预先提示 `query`，用户应答后触发 `callback`。给用户显示 query 后，用户应答被输入后，调用 `callback`。

如果暂停，使用 `createInterface`也可以重置`input` 输入流。
  
调用 `createInterface` 时，如果 `output` 设置为 `null` 或 `undefined` ，不会重新写提示符。  

例子:

    interface.question('What is your favorite food?', function(answer) {
      console.log('Oh, so your favorite food is ' + answer);
    });

### rl.pause()

暂停逐行读取（Readline）的 `input` 输入流, 如果需要可以重新启动。

注意，这不会立即暂停流。调用 `pause` 后还会有很多事件触发，包含 `line`。

### rl.resume()

恢复 逐行读取（Readline） `input` 输入流.

### rl.close()

关闭 `Interface` 实例, 放弃控制输入输出流。会触发"close" 事件。

### rl.write(data[, key])

调用` createInterface` 后，将数据 `data` 写到 `output`  输出流，除非 `output` 为 `null`，或未定义`undefined` 。`key` 是一个代表键序列的对象；当终端是一个 TTY 时可用。

暂停 `input` 输入流后，这个方法可以恢复。

例子：

    rl.write('Delete me!');
    // Simulate ctrl+u to delete the line written previously
    rl.write(null, {ctrl: true, name: 'u'});

## Events

### 事件: 'line'

`function (line) {}`

`input` 输入流收到 `\n` 后触发，通常因为用户敲回车或返回键。这是监听用户输入的好办法。

监听 `line` 的例子:

    rl.on('line', function (cmd) {
      console.log('You just typed: '+cmd);
    });

### 事件: 'pause'

`function () {}`

暂停 `input` 输入流后，会触发这个方法。

当输入流未被暂停，但收到 `SIGCONT` 也会触发。 (详见 `SIGTSTP` 和 `SIGCONT` 事件)


监听 `pause` 的例子:

    rl.on('pause', function() {
      console.log('Readline paused.');
    });

### 事件: 'resume'

`function () {}`

恢复 `input` 输入流后，会触发这个方法。

监听 `resume` 的例子:

    rl.on('resume', function() {
      console.log('Readline resumed.');
    });

### 事件: 'close'

`function () {}`

调用 `close()` 方法时会触发。

当 `input` 输入流收到 "end" 事件时会触发。一旦触发，可以认为 `Interface` 实例结束。例如当`input` 输入流收到 `^D`，被当做 `EOT`。

如果没有`SIGINT` 事件监听器，当`input` 输入流接收到`^C`（被当做`SIGINT`），也会触发这个事件。

### 事件: 'SIGINT'

`function () {}`

当 `input` 输入流收到 `^C` 时会触发， 被当做`SIGINT`。如果没有`SIGINT` 事件监听器，当`input` 输入流接收到 `SIGINT`（被当做`SIGINT`），会触发 `pause` 事件。

监听 `SIGINT` 的例子:

    rl.on('SIGINT', function() {
      rl.question('Are you sure you want to exit?', function(answer) {
        if (answer.match(/^y(es)?$/i)) rl.pause();
      });
    });

### 事件: 'SIGTSTP'

`function () {}`

**Windows 里不可用**

当 `input` 输入流收到 `^Z` 时会触发，被当做`SIGTSTP`。如果没有`SIGINT` 事件监听器，当`input` 输入流接收到 `SIGTSTP`，程序将会切换到后台。


当程序通过 `fg` 恢复，将会触发 `pause` 和 `SIGCONT` 事件。你可以使用两者中任一事件来恢复流。
  
程切换到后台前，如果暂停了流，`pause` 和 `SIGCONT` 事件不会被触发。

监听 `SIGTSTP` 的例子:

    rl.on('SIGTSTP', function() {
      // This will override SIGTSTP and prevent the program from going to the
      // background.
      console.log('Caught SIGTSTP.');
    });

### 事件: 'SIGCONT'

`function () {}`

**Windows 里不可用**

一旦 input 流中含有 ^Z并被切换到后台就会触发。被当做`SIGTSTP`，然后继续执行 `fg(1)`。程切换到后台前，如果流没被暂停，这个事件可以被触发。


监听 `SIGCONT` 的例子:

    rl.on('SIGCONT', function() {
      // `prompt` will automatically resume the stream
      rl.prompt();
    });


## 例子： Tiny CLI

以下的例子，展示了如何所有这些方法的命令行接口:

    var readline = require('readline'),
        rl = readline.createInterface(process.stdin, process.stdout);

    rl.setPrompt('OHAI> ');
    rl.prompt();

    rl.on('line', function(line) {
      switch(line.trim()) {
        case 'hello':
          console.log('world!');
          break;
        default:
          console.log('Say what? I might have heard `' + line.trim() + '`');
          break;
      }
      rl.prompt();
    }).on('close', function() {
      console.log('Have a great day!');
      process.exit(0);
    });

## readline.cursorTo(stream, x, y)

在TTY 流里，移动光标到指定位置。

## readline.moveCursor(stream, dx, dy)

在TTY 流里，移动光标到当前位置的相对位置。

## readline.clearLine(stream, dir)

清空 TTY 流里指定方向的行。`dir` 是以下值：

* `-1` - 从光标到左边
* `1` - 从光标到右边
* `0` - 整行

## readline.clearScreenDown(stream)

清空屏幕上从当前光标位置起的内容。