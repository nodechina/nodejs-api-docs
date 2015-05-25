# 实用工具

    稳定性: 4 - 锁定

这些函数都在`'util'` 模块里。使用 `require('util')` 来访问他们。

`util` 模块原先设计的初衷是用来支持 node 的内部 API 的。这里的很多的函数对你的程序来说都非常有用。如果你觉得这些函数不能满足你的要求，那你可以写自己的工具函数。我们不希望 `'util'` 模块里添加对于 node 内部函数无用的扩展。


## util.debuglog(section)

* `section` {字符串} 被调试的程序节点部分
* Returns: {Function} 日志函数

用来创建一个有条件的写到 stderr 的函数（基于 `NODE_DEBUG`  环境变量）。如果 `section` 出现在环境变量里，返回函数将会和 `console.error()` 类似。否则，返回一个空函数。  

例如:
  

```  
javascript
var debuglog = util.debuglog('foo');

var bar = 123;
debuglog('hello from foo [%d]', bar);
  
```  

如果这个程序以 `NODE_DEBUG=foo` 的环境运行，将会输出：

    FOO 3245: hello from foo [123]

`3245` 是进程 ID。如果没有运行在这个环境变量里，将不会打印任何东西。

可以用逗号切割多个 `NODE_DEBUG` 环境变量。例如：`NODE_DEBUG=fs,net,tls`。

## util.format(format[, ...])

使用第一个参数返回一个格式化的字符串，类似 `printf`。

第一个参数是字符串，它包含 0 或更多的占位符。每个占位符被替换成想要参数转换的值。支持的占位符包括：

* `%s` - 字符串.
* `%d` - 数字 (整数和浮点数).
* `%j` - JSON.  如果参数包含循环引用，将会用字符串替换R
* `%%` - 单独一个百分号 (`'%'`)。 不会消耗一个参数。

如果占位符没有包含一个相应的参数，占位符不会被替换。

    util.format('%s:%s', 'foo'); // 'foo:%s'

如果参数超过占位符，多余的参数将会用 `util.inspect()` 转换成字符串，并拼接在一起，用空格隔开。

    util.format('%s:%s', 'foo', 'bar', 'baz'); // 'foo:bar baz'

如果第一个参数不是格式化字符串，那么 `util.format()` 会返回所有参数拼接成的字符串（空格分割）。每个参数都会用 `util.inspect()` 转换成字符串。

    util.format(1, 2, 3); // '1 2 3'


## util.log(string)

在 `stdout` 输出并带有时间戳.

    require('util').log('Timestamped message.');

## util.inspect(object[, options])

返回一个对象的字符串表现形式，在代码调试的时候非常有用。

通过加入一些可选选项，来改变对象的格式化输出形式：

 - `showHidden` - 如果为 `true`，将会显示对象的不可枚举属性。默认为 `false`。

 - `depth` - 告诉 `inspect` 格式化对象时递归多少次。这在格式化大且复杂对象时非常有用。默认为 `2`。如果想无穷递归的话，传 `null` 。

 - `colors` - 如果为 `true`, 输出内容将会格式化为有颜色的代码。默认为 `false`， 颜色可以自定义，参见下文。

 - `customInspect` - 如果为 `false`, 那么定义在被检查对象上的inspect(depth, opts) 方法将不会被调用。 默认为true。

检查 `util` 对象上所有属性的例子：

    var util = require('util');

    console.log(util.inspect(util, { showHidden: true, depth: null }));

当被调用的时候，参数值可以提供自己的自定义inspect(depth, opts)方法。该方法会接收当前的递归检查深度，以及传入util.inspect()的其他参数。

### 自定义 `util.inspect` 颜色

<!-- type=misc -->

`util.inspect` 通过 `util.inspect.styles` 和 `util.inspect.colors` 对象，自定义全局的输出颜色，

`util.inspect.styles` 和 `util.inspect.colors` 组成风格颜色的一对映射。

高亮风格和他们的默认值：
 * `数字` (黄色)
 * `boolean` (黄色)
 * `字符串` (绿色)
 * `date` (洋红)
 * `regexp` (红色)
 * `null` (粗体)
 * `undefined` (斜体)
 * `special` - (青绿色)
 * `name` (内部用，不是风格)

预定义的颜色为: `white`, `斜体`, `black`, `blue`, `cyan`, 
`绿色`, `洋红`, `红色` 和 `黄色`.
以及 `粗体`, `斜体`, `下划线` 和 `反选` 风格.

### 对象上德自定义 `inspect()` 函数

<!-- type=misc -->

对象也能自定义 `inspect(depth)` 函数， 当使用util.inspect()检查该对象的时候，将会执行对象自定义的检查方法:

    var util = require('util');

    var obj = { name: 'nate' };
    obj.inspect = function(depth) {
      return '{' + this.name + '}';
    };

    util.inspect(obj);
      // "{nate}"

你可以返回另外一个对象，返回的字符串会根据返回的对象格式化。这和 `JSON.stringify()` 的工作流程类似。
You may also return another Object entirely, and the returned 字符串 will be
formatted according to the returned Object. This is similar to how
`JSON.stringify()` works:

    var obj = { foo: 'this will not show up in the inspect() output' };
    obj.inspect = function(depth) {
      return { bar: 'baz' };
    };

    util.inspect(obj);
      // "{ bar: 'baz' }"


## util.isArray(object)

Array.isArray 的内部别名。

如果参数 "object" 是数组，返回 `true` ，否则返回  `false` 。

    var util = require('util');

    util.isArray([])
      // true
    util.isArray(new Array)
      // true
    util.isArray({})
      // false


## util.isRegExp(object)

如果参数 "object" 是 `RegExp` 返回 `true` ，否则返回 `false`。

    var util = require('util');

    util.isRegExp(/some regexp/)
      // true
    util.isRegExp(new RegExp('another regexp'))
      // true
    util.isRegExp({})
      // false


## util.isDate(object)

如果参数 "object" 是 `Date` 返回 `true` ，否则返回 `false`。

    var util = require('util');

    util.isDate(new Date())
      // true
    util.isDate(Date())
      // false (without 'new' returns a String)
    util.isDate({})
      // false


## util.isError(object)

如果参数 "object" 是 `Error` 返回 `true` ，否则返回 `false`。

    var util = require('util');

    util.isError(new Error())
      // true
    util.isError(new TypeError())
      // true
    util.isError({ name: 'Error', message: 'an error occurred' })
      // false


## util.inherits(constructor, superConstructor)

从一个构造函数[constructor](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Object/constructor)继承原型方法到另一个。构造函数的原型将被设置为一个新的从超类（`superConstructor`）创建的对象。

通过 `constructor.super_` 属性可以访问 `superConstructor` 。

    var util = require("util");
    var events = require("events");

    function MyStream() {
        events.EventEmitter.call(this);
    }

    util.inherits(MyStream, events.EventEmitter);

    MyStream.prototype.write = function(data) {
        this.emit("data", data);
    }

    var stream = new MyStream();

    console.log(stream instanceof events.EventEmitter); // true
    console.log(MyStream.super_ === events.EventEmitter); // true

    stream.on("data", function(data) {
        console.log('Received data: "' + data + '"');
    })
    stream.write("It works!"); // Received data: "It works!"


## util.deprecate(function, string)

标明该方法不要再使用。

    exports.puts = exports.deprecate(function() {
      for (var i = 0, len = arguments.length; i < len; ++i) {
        process.stdout.write(arguments[i] + '\n');
      }
    }, 'util.puts: Use console.log instead')

返回一个修改过的函数，默认情况下仅警告一次。如果设置了 `--no-deprecation` 该函数不做任何事。如果设置了`--throw-deprecation`，如果使用了该 API 应用将会抛出异常

## util.debug(string)

    稳定性: 0 - 抛弃: 使用 console.error() 替换。

`console.error` 的前身。

## util.error([...])

    稳定性: 0 - 抛弃: 使用 console.error() 替换。

`console.error` 的前身。

## util.puts([...])

    稳定性: 0 - 抛弃:使用 console.log() 替换。

`console.log` 的前身。

## util.print([...])

    稳定性: 0 - 抛弃: 使用 console.log() 替换。

`console.log` 的前身。


## util.pump(readableStream, writableStream[, callback])

    稳定性: 0 - 抛弃: Use readableStream.pipe(writableStream)

`stream.pipe` 的前身。
