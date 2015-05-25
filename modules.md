# 模块

    稳定性: 5 - 锁定

Node 有简单的模块加载系统。在 Node 里，文件和模块是一一对应的。下面例子里，`foo.js` 加载同一个文件夹里的 `circle.js` 模块。

 `foo.js` 内容:

    var circle = require('./circle.js');
    console.log( 'The area of a circle of radius 4 is '
               + circle.area(4));

`circle.js` 内容:

    var PI = Math.PI;

    exports.area = function (r) {
      return PI * r * r;
    };

    exports.circumference = function (r) {
      return 2 * PI * r;
    };

`circle.js` 模块输出了 `area()` 和 `circumference()` 函数。想要给根模块添加函数和对象，你可以将他们添加到特定的 `exports` 对象。

加载到模块里的变量是私有的，仿佛模块是包含在一个函数里。在这个例子里， `PI` 是  `circle.js` 的私有变量。

如果你想模块里的根像一个函数一样的输出（比如 构造函数），或者你想输出一个完整对象,那就分派给 `module.exports`，而不是 `exports`。

`bar.js` 使用 `square` 模块, 它输出了构造函数:

    var square = require('./square.js');
    var mySquare = square(2);
    console.log('The area of my square is ' + mySquare.area());

`square` 定义在 `square.js` 文件里:

    // assigning to exports will not modify module, must use module.exports
    module.exports = function(width) {
      return {
        area: function() {
          return width * width;
        }
      };
    }

模块系统在 `require("module")` 模块里实现。

## Cycles

环形调用 `require()` ，当返回时模块可能都没执行结束。

考虑以下场景:

`a.js`:

    console.log('a starting');
    exports.done = false;
    var b = require('./b.js');
    console.log('in a, b.done = %j', b.done);
    exports.done = true;
    console.log('a done');

`b.js`:

    console.log('b starting');
    exports.done = false;
    var a = require('./a.js');
    console.log('in b, a.done = %j', a.done);
    exports.done = true;
    console.log('b done');

`main.js`:

    console.log('main starting');
    var a = require('./a.js');
    var b = require('./b.js');
    console.log('in main, a.done=%j, b.done=%j', a.done, b.done);

当  `main.js` 加载  `a.js`，`a.js`加载 `b.js`。此时，`b.js`试着加载 `a.js`。为了阻止循环调用，`a.js`输出对象的不完全拷贝返回给`b.js` 模块。 `b.js` 会结束加载，并且它的`exports` 对象提供给 `a.js` 模块。

`main.js`加载完两个模块时，它们都会结束。这个程序的输出如下： 

    $ node main.js
    main starting
    a starting
    b starting
    in b, a.done = false
    b done
    in a, b.done = true
    a done
    in main, a.done=true, b.done=true

如果你的程序有环形模块依赖，需要保证是线性的。

## 核心模块

Node 有很多模块编译成二进制。这些模块在本文档的其他地方有更详细的描述。

核心模块定义在 Node 的源代码 `lib/` 目录里。

`require()` 总是会优先加载核心模块。例如，`require('http')` 总是返回编译好的 HTTP 模块，而不管这个文件的名字。

## 文件模块

如果按照文件名没有找到模块，那么 Node 会试着加载添加了后缀 `.js`, `.json` 的文件，如果还没好到，再试着加载添加了后缀`.node` 的文件。

`.js`  会解析为 JavaScript 的文本文件， `.json` 会解析为 JSON 文本文件，`.node` 会解析为编译过的插件模块，由 `dlopen` 负责加载。

模块的前缀`'/'` 表示绝对路径。例如 `require('/home/marco/foo.js')` 将会加载 `/home/marco/foo.js`文件。

模块的前缀`'./'` 表示相对于调用 `require()`的路径。就是说，`circle.js`必须和  `foo.js` 在 同一个目录里，`require('./circle')` 才能找到。

文件前没有 `/` 或 `./` 前缀，表示模块可能是 `core module`，或者已经从 `node_modules` 文件夹里加载过了。

如果指定的路径不存在，`require()`将会抛出一个 `code` 属性为 `'MODULE_NOT_FOUND'` 的异常。

## 从 `node_modules` 目录里加载

如传递给 `require()` 的模块不是一个本地模块，并且不以 `'/'`, `'../'`, 或 `'./'` 开头，那么 Node 会从当前模块的父目录开始，尝试在它的 `node_modules` 文件夹里加载模块。

如果没有找到，那么会到父目录，直到到文件系统的根目录里找。

例如，如果 `'/home/ry/projects/foo.js'` 里的文件加载 `require('bar.js')`，那么 Node 将会按照下面的顺序查找:


* `/home/ry/projects/node_modules/bar.js`
* `/home/ry/node_modules/bar.js`
* `/home/node_modules/bar.js`
* `/node_modules/bar.js`

这样允许程序独立，不会产生冲突。

可以请求指定的文件或分布子目录里的模块，在模块名后添加路径后缀。例如，`require('example-module/path/to/file')` 会解决 `path/to/file` 相对于`example-module` 的加载位置。路径后缀使用相同语法。

## 文件夹作为模块

可以把程序和库放到独立的文件夹里，并提供单一的入口指向他们。有三种方法可以将文件夹作为参数传给 `require()` 。

第一个方法是，在文件夹的根创建一个 `package.json` 文件，它指定了 `main` 模块。`package.json` 的例子如下：

    { "name" : "some-library",
      "main" : "./lib/some-library.js" }

如果这是在 `./some-library` 里的文件夹，`require('./some-library')` 将会试着加载 `./some-library/lib/some-library.js`。

如果文件夹里没有 `package.json` 文件，Node 会试着加载 `index.js` 或 `index.node` 文件。例如，如果上面的例子里没有 package.json 文件。那么  `require('./some-library')` 将会试着加载：

* `./some-library/index.js`
* `./some-library/index.node`

## 缓存

模块第一次加载后会被被缓存。这就是说，每次调用 `require('foo')` 都会返回同一个对象，当然，必须每次都要解析到同一个文件。

多次调用 `require('foo')` 也许不会导致模块代码多次执行。这是很重要的特性，这样就可以返回 "partially done" 对象，允许加载过渡性的依赖关系，即使可能会引起环形调用。

如果你希望多次调用一个模块，那么就输出一个函数，然后调用这个函数。

### 模块换成预警

模块的缓存依赖于解析后的文件名。因此随着调用位置的不同，模块可能解析到不同的文件（例如，从 `node_modules` 文件夹加载）。如果解析为不同的文件，`require('foo')` 可能会返回不同的对象。  

## `module` 对象

* {Object}

在每个模块中，变量 `module` 是一个代表当前模块的对象的引用。为了方便，`module.exports` 可以通过 `exports` 全局模块访问。`module` 不是事实上的全局对象，而是每个模块内部的。

### module.exports

* {Object}

模块系统创建 `module.exports` 对象。很多人希望自己的模块是某个类的实例。因此，把将要导出的对象赋值给  `module.exports`。注意，将想要的对象赋值给 `exports` ，只是简单的将它绑定到本地 `exports`  变量，这可能并不是你想要的。

例如，假设我们有一个模块叫 `a.js`。  

    var EventEmitter = require('events').EventEmitter;

    module.exports = new EventEmitter();

    // Do some work, and after some time emit
    // the 'ready' event from the module itself.
    setTimeout(function() {
      module.exports.emit('ready');
    }, 1000);

另一个文件可以这么写：

    var a = require('./a');
    a.on('ready', function() {
      console.log('module a is ready');
    });

注意，赋给 `module.exports` 必须马上执行，并且不能在回调中执行。

x.js:

    setTimeout(function() {
      module.exports = { a: "hello" };
    }, 0);

y.js:

    var x = require('./x');
    console.log(x.a);

#### exports alias

 `exports` 变量在引用到 `module.exports` 的模块里可用。和其他变量一样，如果你给他赋一个新的值，它不再指向老的值。

为了展示这个特性，假设实现：
`require()`:

    function require(...) {
      // ...
      function (module, exports) {
        // Your module code here
        exports = some_func;        // re-assigns exports, exports is no longer
                                    // a shortcut, and nothing is exported.
        module.exports = some_func; // makes your module export 0
      } (module, module.exports);
      return module;
    }

如果你对 `exports` 和 `module.exports` 间的关系感到迷糊，那就只用`module.exports` 就好。

### module.require(id)

* `id` {String}
* 返回: {Object} 已经解析模块的`module.exports` 

`module.require` 方法提供了一种像 `require()` 一样从最初的模块加载一个模块的方法。

为了能这样做，你必须获得`module` 对象的引用。`require()` 返回 `module.exports`，并且 `module` 是一个典型的只能在特定模块作用域内有效的变量，如果要使用它，就必须明确的导出。


### module.id

* {String}

模块的标识符。通常是完全解析的文件名。


### module.filename

* {String}

模块完全解析的文件名。


### module.loaded

* {Boolean}

模块是已经加载完毕，还是在加载中。


### module.parent

* {Module Object}

引入这个模块的模块。


### module.children

* {Array}

由这个模块引入的模块。

## 其他...

为了获取即将用 `require()` 加载的准确文件名，可以使用 `require.resolve()` 函数。

综上所述，下面用伪代码的高级算法形式演示了 require.resolve 的工作流程：

    require(X) from module at path Y
    1. If X is a core module,
       a. return the core module
       b. STOP
    2. If X begins with './' or '/' or '../'
       a. LOAD_AS_FILE(Y + X)
       b. LOAD_AS_DIRECTORY(Y + X)
    3. LOAD_NODE_MODULES(X, dirname(Y))
    4. THROW "not found"

    LOAD_AS_FILE(X)
    1. If X is a file, load X as JavaScript text.  STOP
    2. If X.js is a file, load X.js as JavaScript text.  STOP
    3. If X.json is a file, parse X.json to a JavaScript Object.  STOP
    4. If X.node is a file, load X.node as binary addon.  STOP

    LOAD_AS_DIRECTORY(X)
    1. If X/package.json is a file,
       a. Parse X/package.json, and look for "main" field.
       b. let M = X + (json main field)
       c. LOAD_AS_FILE(M)
    2. If X/index.js is a file, load X/index.js as JavaScript text.  STOP
    3. If X/index.json is a file, parse X/index.json to a JavaScript object. STOP
    4. If X/index.node is a file, load X/index.node as binary addon.  STOP

    LOAD_NODE_MODULES(X, START)
    1. let DIRS=NODE_MODULES_PATHS(START)
    2. for each DIR in DIRS:
       a. LOAD_AS_FILE(DIR/X)
       b. LOAD_AS_DIRECTORY(DIR/X)

    NODE_MODULES_PATHS(START)
    1. let PARTS = path split(START)
    2. let I = count of PARTS - 1
    3. let DIRS = []
    4. while I >= 0,
       a. if PARTS[I] = "node_modules" CONTINUE
       c. DIR = path join(PARTS[0 .. I] + "node_modules")
       b. DIRS = DIRS + DIR
       c. let I = I - 1
    5. return DIRS

## 从全局文件夹加载

如果环境变量 `NODE_PATH` 设置为冒号分割的绝对路径列表，并且在模块在其他地方没有找到，Node 将会搜索这些路径。（注意，Windows 里，`NODE_PATH`用分号分割 ）。


另外， Node 将会搜索这些路径。

* 1: `$HOME/.node_modules`
* 2: `$HOME/.node_libraries`
* 3: `$PREFIX/lib/node`

`$HOME` 是用户的 home 文件夹，`$PREFIX`  是 Node 里配置的 `node_prefix`。

这大多是历史原因照成的。强烈建议将所以来的模块放到 `node_modules` 文件夹里。这样加载会更快。

## 访问主模块

当 Node 运行一个文件时， `require.main` 就会设置为它的 `module`。也就是说你可以通过测试判断文件是否被直接运行。  

    require.main === module

对于 `foo.js` 文件。 如果直接运行 `node foo.js`，返回  `true`， 如果通过 `require('./foo')`是间接运行。

因为 `module` 提供了 `filename` 属性（通常等于`__filename`），程序的入口点可以通过检查  `require.main.filename` 来获得。

## 附录: 包管理技巧

Node 的 `require()` 函数语义定义的足够通用，它能支持各种常规目录结构。诸如 `dpkg`, `rpm`, 和  `npm` 包管理程序，不用修改就可以从 Node 模块构建本地包。

下面我们介绍一个可行的目录结构：

假设我们有一个文件夹 `/usr/lib/node/<some-package>/<some-version>`，包含指定版本的包内容。

一个包可以依赖于其他包。为了安装包 foo，可能需要安装特定版本的 `bar` 包。 `bar`  包可能有自己的包依赖，某些条件下，依赖关系可能会发生冲突或形成循环。


因为 Node 会查找他所加载的模块的 `realpath`（也就是说会解析符号链接），然后按照上文描述的方式在 node_modules 目录中寻找依赖关系，这种情形跟以下体系结构非常相像：


* `/usr/lib/node/foo/1.2.3/` -  `foo` 包, version 1.2.3.
* `/usr/lib/node/bar/4.3.2/` -   `foo` 依赖的 `bar` 包内容
* `/usr/lib/node/foo/1.2.3/node_modules/bar` - 指向 `/usr/lib/node/bar/4.3.2/` 的符号链接
* `/usr/lib/node/bar/4.3.2/node_modules/*` - 指向 `bar` 包所依赖的包的符号链接

因此，即使存在循环依赖或依赖冲突，每个模块还可以获得他所依赖的包得可用版本。

当`foo` 包里的代码调用 `foo` ，将会获得符号链接`/usr/lib/node/foo/1.2.3/node_modules/bar` 指向的版本。然后，当 bar 包中的代码调用 `require('queue')`，将会获得符号链接  `/usr/lib/node/bar/4.3.2/node_modules/quux` 指向的版本。

另外，为了让模块搜索更快些，不要将包直接放在 `/usr/lib/node` 目录中，而是将它们放在 `/usr/lib/node_modules/<name>/<version>` 目录中。 这样在依赖的包找不到的情况下，就不会一直寻找 /`usr/node_modules` 目录或 `/node_modules` 目录了。基于调用 require() 的文件所在真实路径，因此包本身可以放在任何位置。

为了让 Node 模块对 Node REPL 可用，可能需要将  `/usr/lib/node_modules`  文件夹路径添加到环境变量  `$NODE_PATH`  。由于模块查找 `$NODE_PATH` 文件夹都是相对路径，因此包可以放到任何位置。
