# 全局对象

这些对象在所有模块里都可用。有些对象不是在全局作用域而是在模块作用域里，这些情况下面文档都会标注出来。

## global

* {Object} 全局命名空间对象。

浏览器里，全局作用域就是顶级域。如果在全局域内定义变量 `var something` 将会是全局变量。 Node 里不同，顶级域并不是全局域；在模块里定义变量 `var something` 只是模块内可用。

## process

* {Object}

进程对象。 参见 [process object][] 章节.

## console

* {Object}

用来打印 stdout 和 stderr. 参见[console][] 章节.

## Class: Buffer

* {Function}

用来处理二进制数据。 参见[buffer 章节][]。

## require()

* {Function}

引入模块。 参见[Modules][] 章节.  `require` 实际上并非全局的，而是各个本地模块有效。

### require.resolve()

使用内部 `require()` 机制来查找 module 位置，但是不加载模块，只是返回解析过的文件名。

### require.cache

* {Object}

引入模块时会缓存到这个对象。通过删除该对象键值，下次调用`require` 将会重载该模块。

### require.extensions

    稳定性: 0 - 抛弃

* {Object}

指导  `require` 如何处理特定的文件扩展名。

将 `.sjs` 文件当 `.js` 文件处理:

    require.extensions['.sjs'] = require.extensions['.js'];

**抛弃**  以前这个列表用来加载按需编译的非 JavaScript 模块到 node。实际上，有更好的办法来解决这个问题，比如通过其他 node 程序来加载模块，或者提前编译成 JavaScript。

由于模块系统已经锁定，该功能可能永远不会去掉。改动它可能会产生 bug，所以最好不要动它。

## __filename

* {String}

被执行的代码的文件名是相对路径。对于主程序来说，这和命令行里未必用同一个文件名。模块里的值是模块文件的路径。

列如，运行  `/Users/mjr` 里的 `node example.js`：

    console.log(__filename);
    // /Users/mjr/example.js

`__filename` 不是全局的，而是模块本地的。

## __dirname



* {String}

执行的 script 代码所在的文件夹的名字。

列如，运行  `/Users/mjr` 里的 `node example.js`：

    console.log(__dirname);
    // /Users/mjr

`__dirname` 不是全局的，而是模块本地的。


## module



* {Object}

当前模块的引用。通过 `require()`，`module.exports`定义了哪个模块输出可用。

`module` 不是全局的，而是模块本地的。

更多信息参见[module system documentation][]。

## exports



`module.exports` 的引用。何时用 `exports` 和 `module.exports` 可参加[module system documentation][]。

`module` 不是全局的，而是模块本地的。  

更多信息参见 [module system documentation][]。

更多信息参见[module 章节][]。

## setTimeout(cb, ms)

最少 `ms` 毫秒后调回调函数。实际的延迟依赖于外部因素，比如操作系统的粒度和负载。

timeout 值有效范围 1-2,147,483,647。如果超过范围，将会变为 1 毫秒。通常，定时器不应该超过 24.8 天。

返回一个代表定时器的句柄值。  

## clearTimeout(t)

停止一个之前通过 `setTimeout()` 创建的定时器。不会再被执行回调。  

## setInterval(cb, ms)

每隔 `ms` 毫秒调用回调函数 `cb` 。实际的间隔依赖于外部因素，比如操作系统的粒度和系统负载。通常会大于`ms`。

间隔值有效范围 1-2,147,483,647。如果超过范围，将会变为 1 毫秒。通常，定时器不应该超过 24.8 天。

返回一个代表该定时器的句柄值。  

## clearInterval(t)

停止一个之前通过 `setInterval()` 创建的定时器。不会再被执行回调。  


timer 函数是全局变量。 参见[timers][] 章节。

[buffer 章节]: buffer.html
[module 章节]: modules.html
[module system documentation]: modules.html
[Modules]: modules.html#modules_modules
[process object]: process.html#process_process
[console]: console.html
[timers]: timers.html
