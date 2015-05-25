# 虚拟机

    稳定性: 3 - 稳定

<!--name=vm-->

可以通过以下方法访问该模块:

    var vm = require('vm');

JavaScript 可以立即编译立即执行，也可以编译，保存，之后再运行。

## vm.runInThisContext(code[, options])

`vm.runInThisContext()` 对 参数`code` 编译，运行并返回结果。 运行的代码没有权限访问本地作用域（local scope），但是可以访问全局对象。

使用 `vm.runInThisContext` 和 `eval` 方法运行同样代码的例子：

    var localVar = 'initial value';

    var vmResult = vm.runInThisContext('localVar = "vm";');
    console.log('vmResult: ', vmResult);
    console.log('localVar: ', localVar);

    var evalResult = eval('localVar = "eval";');
    console.log('evalResult: ', evalResult);
    console.log('localVar: ', localVar);

    // vmResult: 'vm', localVar: 'initial value'
    // evalResult: 'eval', localVar: 'eval'

`vm.runInThisContext` 没有访问本地作用域，所以没有改变 `localVar`。 `eval` 范围了本地作用域，所以改变了 `localVar`。

`vm.runInThisContext` 用起来很像间接调用 `eval`，比如  `(0,eval)('code')`。但是，`vm.runInThisContext` 也包含以下选项:

- `filename`: 允许更改显示在站追踪（stack traces）的文件名。
- `displayErrors`: 是否在 stderr 上打印错误，抛出异常的代码行高亮显示。会捕获编译时的语法错误，和执行时抛出的错误。默认为 `true`。
- `timeout`: 中断前代码执行的毫秒数。如果执行终止，将会抛出错误。

[1]: http://es5.github.io/#x10.4.2


## vm.createContext([sandbox])

如果参数 `sandbox` 不为空，调用 `vm.runInContext` 或 `script.runInContext` 时可以调用沙箱的上下文。以此方式运行的脚本，`sandbox` 是全局对象，它保留自己的属性同时拥有标准全局对象（[global object][2]）拥有的内置对象和函数。

如果参数 sandbox 对象为空，返回一个可用的新且空的上下文相关的沙盒对象。


这个函数对于创建一个可运行多脚本的沙盒非常有用。比如，在模拟浏览器的时候可以使用该函数创建一个用于表示 window 全局对象的沙箱，并将所有 `<script>` 标签放入沙箱执行。

[2]: http://es5.github.io/#x15.1


## vm.isContext(sandbox)

沙箱对象是否已经通过调用 `vm.createContext` 上下文化。


## vm.runInContext(code, contextifiedSandbox[, options])

`vm.runInContext` 编译代码，运行在 `contextifiedSandbox` 并返回结果。运行代码不能访问本地域。`contextifiedSandbox` 对象必须通过 `vm.createContext` 上下文化；`code` 会通过全局变量使用它。

`vm.runInContext` 和 `vm.runInThisContext` 参数相同。

在同一个上下文中编译并执行不同的脚本，例子：

    var util = require('util');
    var vm = require('vm');

    var sandbox = { globalVar: 1 };
    vm.createContext(sandbox);

    for (var i = 0; i < 10; ++i) {
        vm.runInContext('globalVar *= 2;', sandbox);
    }
    console.log(util.inspect(sandbox));

    // { globalVar: 1024 }

注意，执行不被信任的代码是需要技巧且要非常的小心。`vm.runInContext` 非常有用，不过想要安全些，最好还是在独立的进程里运行不被信任的代码。


## vm.runInNewContext(code[, sandbox][, options])

`vm.runInNewContext` 编译代码, 如果提供了 sandbox ，则将 sandbox 上下文化，否则创建一个新的上下文化过的沙盒，将沙盒作为全局变量运行代码并返回结果。

`vm.runInNewContext` 和 `vm.runInThisContext` 参数相同。

编译并执行代码，增加全局变量值，并设置一个新的。这些全局变量包含在一个新的沙盒里。

    var util = require('util');
    var vm = require('vm'),

    var sandbox = {
      animal: 'cat',
      count: 2
    };

    vm.runInNewContext('count += 1; name = "kitty"', sandbox);
    console.log(util.inspect(sandbox));

    // { animal: 'cat', count: 3, name: 'kitty' }

注意，执行不被信任的代码是需要技巧且要非常的小心。`vm.runInNewContext` 非常有用，不过想要安全些，最好还是在独立的进程里运行不被信任的代码。


## vm.runInDebugContext(code)

`vm.runInDebugContext` 在 V8 的调试上下文中编译并执行。最主要的应用场景是获得 V8 调试对象访问权限。


    var Debug = vm.runInDebugContext('Debug');
    Debug.scripts().forEach(function(script) { console.log(script.name); });

注意，调试上下文和对象内部绑定到 V8 的调试实现里，并可能在没有警告时改变（或移除）。

可以通过 `--expose_debug_as=` 开关暴露调试对象。


## Class: Script

包含预编译脚本的类，并在指定的沙盒里执行。


### new vm.Script(code, options)

创建一个新的脚本编译代码，但是不运行。使用被创建的 `vm.Script` 来表示编译完的代码。这个代码可以使用以下的方法调用多次。返回的脚本没有绑定到任何全局变量。在运行前绑定，执行后释放。

创建脚本的选项有:

- `filename`: 允许更改显示在站追踪（stack traces）的文件名。
- `displayErrors`: 是否在 stderr 上打印错误，抛出异常的代码行高亮显示。只会捕获编译时的语法错误，执行时抛出的错误由脚本的方法的选项来控制。默认为 `true`。


### script.runInThisContext([options])

和 `vm.runInThisContext` 类似，只是作为 `Script` 脚本对象的预编译方法。`script.runInThisContext` 执行编译过的脚本并返回结果。被运行的代码没有本地作用域访问权限，但是拥有权限访问全局对象。

以下例子，使用 `script.runInThisContext` 编译代码一次，并运行多次：

    var vm = require('vm');

    global.globalVar = 0;

    var script = new vm.Script('globalVar += 1', { filename: 'myfile.vm' });

    for (var i = 0; i < 1000; ++i) {
      script.runInThisContext();
    }

    console.log(globalVar);

    // 1000

所运行的代码选项：

- `displayErrors`: 是否在 stderr 上打印错误，抛出异常的代码行高亮显示。仅适用于执行时抛出的错误。不能创建一个语法错误的 `Script` 实例，因为构造函数会抛出。
- `timeout`: 中断前代码执行的毫秒数。如果执行终止，将会抛出错误。


### script.runInContext(contextifiedSandbox[, options])

和 `vm.runInContext` 类似，只是作为预编译的 `Script` 对象方法。`script.runInContext` 运行脚本（在 `contextifiedSandbox` 中编译）并返回结果。运行的代码没有权限访问本地域。

`script.runInContext` 的选项和 `script.runInThisContext` 类似。

例子: 编译一段代码，并执行多次，这段代码实现了一个全局变量的自增，并创建一个新的全局变量。这些全局变量保存在沙盒里。

    var util = require('util');
    var vm = require('vm');

    var sandbox = {
      animal: 'cat',
      count: 2
    };

    var script = new vm.Script('count += 1; name = "kitty"');

    for (var i = 0; i < 10; ++i) {
      script.runInContext(sandbox);
    }

    console.log(util.inspect(sandbox));

    // { animal: 'cat', count: 12, name: 'kitty' }

注意，执行不被信任的代码是需要技巧且要非常的小心。`script.runInContext` 非常有用，不过想要安全些，最好还是在独立的进程里运行不被信任的代码。

### script.runInNewContext([sandbox][, options])

和 `vm.runInNewContext` 类似，只是作为预编译的 `Script` 对象方法。 若提供 sandbox 则 script.runInNewContext 将 sandbox 上下文化，若未提供，则创建一个新的上下文化的沙箱。

`script.runInNewContext` 和 `script.runInThisContext` 的参数类似。

例子: 编译代码（设置了一个全局变量）并在不同的上下文里执行多次。这些全局变量会被保存在沙箱中。

    var util = require('util');
    var vm = require('vm');

    var sandboxes = [{}, {}, {}];

    var script = new vm.Script('globalVar = "set"');

    sandboxes.forEach(function (sandbox) {
      script.runInNewContext(sandbox);
    });

    console.log(util.inspect(sandboxes));

    // [{ globalVar: 'set' }, { globalVar: 'set' }, { globalVar: 'set' }]

注意，执行不被信任的代码是需要技巧且要非常的小心。`script.runInNewContext` 非常有用，不过想要安全些，最好还是在独立的进程里运行不被信任的代码。