# 控制台

    稳定性: 4 - 冻结

* {Object}

用于打印输出字符到 stdout 和 stderr。和多数浏览器提供的 console 对象函数一样，Node 也是输出到 stdout 和 stderr。

当输出目标是终端或文件的时候，console 函数是同步的（为了防止意外退出数据丢失），输出是管道的时候是异步的（防止阻塞时间太长）。

下面的例子里，stdout 是非阻塞的，而 stderr 是阻塞的。

    $ node script.js 2> error.log | tee info.log

平常使用过程中，不用考虑阻塞或非阻塞问题，除非有大批量的数据。


## console.log([data][, ...])

输出到 stdout 并新起一行。和 `printf()` 类似，stdout 可以传入多个参数。例如：

    var count = 5;
    console.log('count: %d', count);
    // prints 'count: 5'

如果第一个字符里没有找到格式化的元素， `util.inspect` 将会应用到各个参数，参见[util.format()][]

## console.info([data][, ...])

参见 `console.log`。

## console.error([data][, ...])

参见 `console.log` ，不同的是打印到 stderr。

## console.warn([data][, ...])

参见 `console.error`。

## console.dir(obj[, options])

在 `obj` 使用 `util.inspect`，并打印结果到 stdout，而这个函数绕过 `inspect()`。`options`参数可能传入以下几种：

- `showHidden` -  如果是`true`，将会展示对象的非枚举属性，默认是 `false` 。

- `depth` - `inspect`对象递归的次数，对于复杂对象的扫描非常有用。默认是 `2`。想要严格递归，传入 `null`。

- `colors` - 如果是 `true`, 输出会格式化为 ANSI 颜色代码。默认是 `false`。颜色可以定制，下面会介绍。

## console.time(label)

标记一个时间点。

## console.timeEnd(label)

计时器结束的时候，记录输出，例如：

    console.time('100-elements');
    for (var i = 0; i < 100; i++) {
      ;
    }
    console.timeEnd('100-elements');
    // prints 100-elements: 262ms

## console.trace(message[, ...])

输出当前位置的栈跟踪到 stderr `'Trace :'`。

## console.assert(value[, message][, ...])

和 [assert.ok()][] 类似, 但是错误的输出格式为：
`util.format(message...)`。

[assert.ok()]: assert.html#assert_assert_value_message_assert_ok_value_message
[util.format()]: util.html#util_util_format_format
