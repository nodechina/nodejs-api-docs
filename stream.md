# 流

    稳定性: 2 - 不稳定

流是一个抽象接口，在 Node 里被不同的对象实现。例如[request to an HTTP
server](http.html#http_http_incomingmessage) 是流，[stdout][] 是流。流是可读，可写，或者可读写。所有的流是 [EventEmitter][] 的实例。

你可以通过 `require('stream')` 加载 Stream 基类。其中包括了 `Readable` 流、`Writable` 流、`Duplex` 流和 `Transform` 流的基类。

这个文档分为 3 个章节。第一个章节解释了在你的程序中使用流时候需要了解的部分。如果你不用实现流式 API，可以只看这个章节。

如果你想实现你自己的流，第二个章节解释了这部分 API。这些 API 让你的实现更加简单。

第三个部分深入的解释了流是如何工作的，包括一些内部机制和函数，这些内容不要改动，除非你明确知道你要做什么。


## 面向流消费者的 API

<!--type=misc-->

流可以是可读（Readable），可写（Writable），或者兼具两者（Duplex，双工）的。

所有的流都是事件分发器（EventEmitters），但是也有自己的方法和属性，这取决于他它们是可读（Readable），可写（Writable），或者兼具两者（Duplex，双工）的。

如果流式可读写的，则它实现了下面的所有方法和事件。因此，这个章节 API 完全阐述了[Duplex][] 或 [Transform][] 流，即便他们的实现有所不同。

没有必要为了消费流而在你的程序里实现流的接口。如果你正在你的程序里实现流接口，请同时参考下面的[API for Stream Implementors][]。

基本所有的 Node 程序，无论多简单，都会使用到流。这有一个使用流的例子。  
  
  
```
javascript
var http = require('http');

var server = http.createServer(function (req, res) {
  // req is an http.IncomingMessage, which is 可读流（Readable stream）
  // res is an http.ServerResponse, which is a Writable Stream

  var body = '';
  // we want to get the data as utf8 strings
  // If you don't set an encoding, then you'll get Buffer objects
  req.setEncoding('utf8');

  // 可读流（Readable stream） emit 'data' 事件 once a 监听器（listener） is added
  req.on('data', function (chunk) {
    body += chunk;
  });

  // the end 事件 tells you that you have entire body
  req.on('end', function () {
    try {
      var data = JSON.parse(body);
    } catch (er) {
      // uh oh!  bad json!
      res.statusCode = 400;
      return res.end('error: ' + er.message);
    }

    // write back something interesting to the user:
    res.write(typeof data);
    res.end();
  });
});

server.listen(1337);

// $ curl localhost:1337 -d '{}'
// object
// $ curl localhost:1337 -d '"foo"'
// string
// $ curl localhost:1337 -d 'not json'
// error: Unexpected token o
```

### 类： stream.Readable

<!--type=class-->

可读流（Readable stream）接口是对你正在读取的数据的来源的抽象。换句话说，数据来来自

可读流（Readable stream）不会分发数据，直到你表明准备就绪。  

可读流（Readable stream） 有2种模式: **流动模式（flowing mode）** 和  **暂停模式（paused mode）**.  流动模式（flowing mode）时,尽快的从底层系统读取数据并提供给你的程序。 暂停模式（paused mode）时, 你必须明确的调用  `stream.read()` 来读取数据。 暂停模式（paused mode） 是默认模式。

**注意**: 如果没有绑定数据处理函数，并且没有 [`pipe()`][] 目标，流会切换到流动模式（flowing mode），并且数据会丢失。

可以通过下面几个方法，将流切换到流动模式（flowing mode）。

* 添加一个 [`'data'` 事件][] 事件处理器来监听数据.
* 调用 [`resume()`][] 方法来明确的开启数据流。
* 调用 [`pipe()`][] 方法来发送数据给[Writable][].

可以通过以下方法来切换到暂停模式（paused mode）:

* 如果没有 导流（pipe） 目标，调用 [`pause()`][]方法.
* 如果有 导流（pipe） 目标, 移除所有的 [`'data'` 事件][]处理函数, 调用 [`unpipe()`][] 方法移除所有的 导流（pipe） 目标。

注意, 为了向后兼容考虑， 移除 'data' 事件监听器并不会自动暂停流。同样的，当有导流目标时，调用 pause() 并不能保证流在那些目标排空后，请求更多数据时保持暂停状态。

可读流（Readable stream）例子包括:

* [http responses, on the client](http.html#http_http_incomingmessage)
* [http requests, on the server](http.html#http_http_incomingmessage)
* [fs read streams](fs.html#fs_class_fs_readstream)
* [zlib streams][]
* [crypto streams][]
* [tcp sockets][]
* [child process stdout and stderr][]
* [process.stdin][]

#### 事件: 'readable'

当一个数据块可以从流中读出，将会触发`'readable'` 事件.`


某些情况下, 如果没有准备好，监听一个 `'readable'` 事件将会导致一些数据从底层系统读取到内部缓存。

```  
javascript
var readble = getReadableStreamSomehow();
readable.on('readable', function() {
  // there is some data to read now
});
```

一旦内部缓存排空，一旦有更多数据将会再次触发 `readable` 事件。

#### 事件: 'data'

* `chunk` {Buffer | String} 数据块

绑定一个 `data` 事件的监听器（listener）到一个未明确暂停的流，会将流切换到流动模式。数据会尽额能的传递。

如果你像尽快的从流中获取数据，这是最快的方法。

```  
javascript
var readable = getReadableStreamSomehow();
readable.on('data', function(chunk) {
  console.log('got %d bytes of data', chunk.length);
});
  
```

#### 事件: 'end'

如果没有更多的可读数据，将会触发这个事件。

注意，除非数据已经被完全消费， the `end` 事件**才会触发**。 可以通过切换到流动模式（flowing mode）来实现，或者通过调用重复调用 `read()`获取数据，直到结束。    
  
```    
javascript
    var readable = getReadableStreamSomehow();
    readable.on('data', function(chunk) {
        console.log('got %d bytes of data', chunk.length);
    });
    readable.on('end', function() {
        console.log('there will be no more data.');
    });  
``` 


#### 事件: 'close'

当底层资源（例如源头的文件描述符）关闭时触发。并不是所有流都会触发这个事件。

#### 事件: 'error'

* {Error Object}

当接收数据时发生错误触发。  

#### readable.read([size])

* `size` {Number} 可选参数， 需要读入的数据量
* 返回 {String | Buffer | null}

`read()` 方法从内部缓存中拉取数据。如果没有可用数据，将会返回`null`

如果传了 `size`参数，将会返回相当字节的数据。如果`size`不可用，将会返回 `null`

如果你没有指定 `size` 参数。将会返回内部缓存的所有数据。

这个方法仅能再暂停模式（paused mode）里调用.  流动模式（flowing mode）下这个方法会被自动调用直到内存缓存排空。

```  
javascript
var readable = getReadableStreamSomehow();
readable.on('readable', function() {
  var chunk;
  while (null !== (chunk = readable.read())) {
    console.log('got %d bytes of data', chunk.length);
  }
});
```

如果这个方法返回一个数据块, 它同时也会触发[`'data'` 事件][].

#### readable.setEncoding(encoding)

* `encoding` {String} 要使用的编码.
* 返回: `this`

调用此函数会使得流返回指定编码的字符串，而不是 Buffer 对象。例如，如果你调用`readable.setEncoding('utf8')`，输出数据将会是UTF-8 编码，并且返回字符串。如果你调用 `readable.setEncoding('hex')`，将会返回2进制编码的数据。

该方法能正确处理多字节字符。如果不想这么做，仅简单的直接拉取缓存并调`buf.toString(encoding)` ，可能会导致字节错位。因此，如果你想以字符串读取数据，请使用这个方法。
  

```  
javascript
var readable = getReadableStreamSomehow();
readable.setEncoding('utf8');
readable.on('data', function(chunk) {
  assert.equal(typeof chunk, 'string');
  console.log('got %d characters of string data', chunk.length);
});
```

#### readable.resume()

* 返回: `this`

这个方法让可读流（Readable stream）继续触发 `data` 事件.

这个方法会将流切换到流动模式（flowing mode）.  如果你不想从流中消费数据，而想得到`end` 事件，可以调用 [`readable.resume()`][]  来打开数据流。  

```  
javascript
var readable = getReadableStreamSomehow();
readable.resume();
readable.on('end', function(chunk) {
  console.log('got to the end, but did not read anything');
});
```

#### readable.pause()

* 返回: `this`

这个方法会使得流动模式（flowing mode）的流停止触发 `data` 事件, 切换到流动模式（flowing mode）. 并让后续可用数据留在内部缓冲区中。

```  
javascript
var readable = getReadableStreamSomehow();
readable.on('data', function(chunk) {
  console.log('got %d bytes of data', chunk.length);
  readable.pause();
  console.log('there will be no more data for 1 second');
  setTimeout(function() {
    console.log('now data will start flowing again');
    readable.resume();
  }, 1000);
});
```

#### readable.isPaused()

* 返回: `Boolean`

这个方法返回`readable` 是否被客户端代码 **明确**的暂停（调用 `readable.pause()`）。

``javascript
var readable = new stream.Readable

readable.isPaused() // === false
readable.pause()
readable.isPaused() // === true
readable.resume()
readable.isPaused() // === false
```

#### readable.pipe(destination[, options])

* `destination` {[Writable][] Stream} 写入数据的目标
* `options` {Object} 导流（pipe） 选项
  * `end` {Boolean} 读取到结束符时，结束写入者。默认 = `true`

这个方法从可读流（Readable stream）拉取所有数据, 并将数据写入到提供的目标中。自动管理流量，这样目标不会快速的可读流（Readable stream）淹没。

可以导流到多个目标。

```  
javascript
var readable = getReadableStreamSomehow();
var writable = fs.createWriteStream('file.txt');
// All the data from readable goes into 'file.txt'
readable.pipe(writable);
```

这个函数返回目标流, 因此你可以建立导流链：

```  
javascript
var r = fs.createReadStream('file.txt');
var z = zlib.createGzip();
var w = fs.createWriteStream('file.txt.gz');
r.pipe(z).pipe(w);
```

例如, 模拟 Unix 的 `cat` 命令:

```  
javascript
process.stdin.pipe(process.stdout);
```
默认情况下，当源数据流触发 `end`的时候调用[`end()`][]，所以 `destination` 不可再写。传 `{ end:false }`作为`options`，可以保持目标流打开状态。


这会让 `writer`保持打开状态，可以在最后写入"Goodbye" 。

```  
javascript
reader.pipe(writer, { end: false });
reader.on('end', function() {
  writer.end('Goodbye\n');
});
```

注意 `process.stderr` 和 `process.stdout` 直到进程结束才会关闭，无论是否指定  

#### readable.unpipe([destination])

* `destination` {[Writable][] Stream} 可选，指定解除导流的流

这个方法会解除之前调用 `pipe()` 设置的钩子（ `pipe()` ）。   

如果没有指定 `destination`,所有的 导流（pipe） 都会被移除。

如果指定了 `destination`，但是没有建立如果没有指定 `destination`，则什么事情都不会发生。

```  
javascript
var readable = getReadableStreamSomehow();
var writable = fs.createWriteStream('file.txt');
// All the data from readable goes into 'file.txt',
// but only for the first second
readable.pipe(writable);
setTimeout(function() {
  console.log('stop writing to file.txt');
  readable.unpipe(writable);
  console.log('manually close the file stream');
  writable.end();
}, 1000);
```

#### readable.unshift(chunk)

* `chunk` {Buffer | String} 数据块插入到读队列中

这个方法很有用，当一个流正被一个解析器消费，解析器可能需要将某些刚拉取出的数据“逆消费”，返回到原来的源，以便流能将它传递给其它消费者。

如果你在程序中必须经常调用 `stream.unshift(chunk)` ，那你可以考虑实现[Transform][]来替换（参见下文API for Stream Implementors）。

```  
javascript
// Pull off a header delimited by \n\n
// use unshift() if we get too much
// Call the callback with (error, header, stream)
var StringDecoder = require('string_decoder').StringDecoder;
function parseHeader(stream, callback) {
  stream.on('error', callback);
  stream.on('readable', onReadable);
  var decoder = new StringDecoder('utf8');
  var header = '';
  function onReadable() {
    var chunk;
    while (null !== (chunk = stream.read())) {
      var str = decoder.write(chunk);
      if (str.match(/\n\n/)) {
        // found the header boundary
        var split = str.split(/\n\n/);
        header += split.shift();
        var remaining = split.join('\n\n');
        var buf = new Buffer(remaining, 'utf8');
        if (buf.length)
          stream.unshift(buf);
        stream.removeListener('error', callback);
        stream.removeListener('readable', onReadable);
        // now the body of the message can be read from the stream.
        callback(null, header, stream);
      } else {
        // still reading the header.
        header += str;
      }
    }
  }
}
```

#### readable.wrap(stream)

* `stream` {Stream} 一个旧式的可读流（Readable stream）

v0.10 版本之前的 Node 流并未实现现在所有流的API（更多信息详见下文“兼容性”章节）。

如果你使用的是旧的 Node 库，它触发 `'data'` 事件，并拥有仅做查询用的[`pause()`][] 方法，那么你能使用`wrap()` 方法来创建一个[Readable][] 流来使用旧版本的流，作为数据源。

你应该很少需要用到这个函数，但它会留下方便和旧版本的 Node 程序和库交互。

例如:

```  
javascript
var OldReader = require('./old-api-module.js').OldReader;
var oreader = new OldReader;
var Readable = require('stream').Readable;
var myReader = new Readable().wrap(oreader);

myReader.on('readable', function() {
  myReader.read(); // etc.
});
```


### 类： stream.Writable

<!--type=class-->

可写流（Writable stream ）接口是你正把数据写到一个目标的抽象。  

可写流（Writable stream ）的例子包括:

* [http requests, on the client](http.html#http_class_http_clientrequest)
* [http responses, on the server](http.html#http_class_http_serverresponse)
* [fs write streams](fs.html#fs_class_fs_writestream)
* [zlib streams][]
* [crypto streams][]
* [tcp sockets][]
* [child process stdin](child_process.html#child_process_child_stdin)
* [process.stdout][], [process.stderr][]

#### writable.write(chunk[, encoding][, callback])

* `chunk` {String | Buffer} 准备写的数据
* `encoding` {String} 编码方式（如果`chunk` 是字符串）
* `callback` {Function} 数据块写入后的回调
* 返回: {Boolean} 如果数据已被全部处理返回true

这个方法向底层系统写入数据，并在数据处理完毕后调用所给的回调。

返回值表示你是否应该继续立即写入。如果数据要缓存在内部，将会返回`false`。否则返回 `true`。

返回值仅供参考。即使返回 `false`，你也可能继续写。但是写会缓存在内存里，所以不要做的太过分。最好的办法是等待`drain` 事件后，再写入数据。

#### 事件: 'drain'

如果调用 [`writable.write(chunk)`][] 返回 false, `drain` 事件会告诉你什么时候将更多的数据写入到流中。

```  
javascript
// Write the data to the supplied 可写流（Writable stream ） 1MM times.
// Be attentive to back-pressure.
function writeOneMillionTimes(writer, data, encoding, callback) {
  var i = 1000000;
  write();
  function write() {
    var ok = true;
    do {
      i -= 1;
      if (i === 0) {
        // last time!
        writer.write(data, encoding, callback);
      } else {
        // see if we should continue, or wait
        // don't pass the callback, because we're not done yet.
        ok = writer.write(data, encoding);
      }
    } while (i > 0 && ok);
    if (i > 0) {
      // had to stop early!
      // write some more once it drains
      writer.once('drain', write);
    }
  }
}
```

#### writable.cork()

强制缓存所有写入。

调用 `.uncork()` 或 `.end()`后，会把缓存数据写入。

#### writable.uncork()

写入所有 .cork() 调用之后缓存的数据。

#### writable.setDefaultEncoding(encoding)

* `encoding` {String} 新的默认编码
* 返回: `Boolean`

给写数据流设置默认编码方式，如编码有效，返回 `true` ，否则返回 `false`。

#### writable.end([chunk][, encoding][, callback])

* `chunk` {String | Buffer} 可选，要写入的数据
* `encoding` {String} 编码方式（如果 `chunk` 是字符串）
* `callback` {Function} 可选， stream 结束时的回调函数  

当没有更多的数据写入的时候调用这个方法。如果给出，回调会被用作 finish 事件的监听器。

调用 [`end()`][] 后调用 [`write()`][] 会产生错误。

```  
javascript
// write 'hello, ' and then end with 'world!'
var file = fs.createWriteStream('example.txt');
file.write('hello, ');
file.end('world!');
// writing more now is not allowed!
```

#### 事件: 'finish'

调用[`end()`][] 方法后，并且所有的数据已经写入到底层系统，将会触发这个事件。


```  
javascript
var writer = getWritableStreamSomehow();
for (var i = 0; i < 100; i ++) {
  writer.write('hello, #' + i + '!\n');
}
writer.end('this is the end\n');
writer.on('finish', function() {
  console.error('all writes are now complete.');
});
```

#### 事件: 'pipe'

* `src` {[Readable][] Stream} 是导流（pipe）到可写流的源流  
  
无论何时在可写流（Writable stream ）上调用`pipe()` 方法，都会触发 'pipe' 事件，添加这个流到目标。

```  
javascript
var writer = getWritableStreamSomehow();
var reader = getReadableStreamSomehow();
writer.on('pipe', function(src) {
  console.error('something is piping into the writer');
  assert.equal(src, reader);
});
reader.pipe(writer);
```

#### 事件: 'unpipe'

* `src` {[Readable][] Stream} The source stream that [unpiped][] this writable

无论何时在可写流（Writable stream ）上调用`unpipe()` 方法,都会触发 'unpipe' 事件,将这个流从目标上移除。


```  
javascript
var writer = getWritableStreamSomehow();
var reader = getReadableStreamSomehow();
writer.on('unpipe', function(src) {
  console.error('something has stopped piping into the writer');
  assert.equal(src, reader);
});
reader.pipe(writer);
reader.unpipe(writer);
```

#### 事件: 'error'

* {Error object}

写或导流（pipe）数据时，如果有错误会触发。

### 类： stream.Duplex


双工流（Duplex streams）是同时实现了 [Readable][] and [Writable][] 接口。用法详见下文。

双工流（Duplex streams） 的例子包括:

* [tcp sockets][]
* [zlib streams][]
* [crypto streams][]


### 类： stream.Transform

转换流（Transform streams） 是双工 [Duplex][] 流，它的输出是从输入计算得来。 它实现了[Readable][] 和 [Writable][] 接口.  用法详见下文.

转换流（Transform streams） 的例子包括:

* [zlib streams][]
* [crypto streams][]


## API for Stream Implementors

<!--type=misc-->

无论实现什么形式的流，模式都是一样的:

1. 在你的子类中扩展适合的父类.  ([`util.inherits`][] 方法很有帮助)
2. 在你的构造函数中调用父类的构造函数，以确保内部的机制初始化正确。
2. 实现一个或多个方法，如下所列

所扩展的类和要实现的方法取决于你要编写的流类。

<table>
  <thead>
    <tr>
      <th>
        <p>Use-case</p>
      </th>
      <th>
        <p>Class</p>
      </th>
      <th>
        <p>方法(s) to implement</p>
      </th>
    </tr>
  </thead>
  <tr>
    <td>
      <p>Reading only</p>
    </td>
    <td>
      <p>[Readable](#stream_class_stream_readable_1)</p>
    </td>
    <td>
      <p><code>[_read][]</code></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Writing only</p>
    </td>
    <td>
      <p>[Writable](#stream_class_stream_writable_1)</p>
    </td>
    <td>
      <p><code>[_write][]</code></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Reading and writing</p>
    </td>
    <td>
      <p>[Duplex](#stream_class_stream_duplex_1)</p>
    </td>
    <td>
      <p><code>[_read][]</code>, <code>[_write][]</code></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>Operate on written data, then read the result</p>
    </td>
    <td>
      <p>[Transform](#stream_class_stream_transform_1)</p>
    </td>
    <td>
      <p><code>_transform</code>, <code>_flush</code></p>
    </td>
  </tr>
</table>

在你的代码里，千万不要调用 [API for Stream Consumers][] 里的方法。否则可能会引起消费流的程序副作用。

### 类： stream.Readable

<!--type=class-->

`stream.Readable` 是一个可被扩充的、实现了底层 `_read(size)` 方法的抽象类。  

参照之前的[API for Stream Consumers][]查看如何在你的程序里消费流。底下内容解释了在你的程序里如何实现可读流（Readable stream）。

#### Example: 计数流

<!--type=example-->

这是可读流（Readable stream）的基础例子.  它将从 1 至 1,000,000 递增地触发数字，然后结束。

```  
javascript
var Readable = require('stream').Readable;
var util = require('util');
util.inherits(Counter, Readable);

function Counter(opt) {
  Readable.call(this, opt);
  this._max = 1000000;
  this._index = 1;
}

Counter.prototype._read = function() {
  var i = this._index++;
  if (i > this._max)
    this.push(null);
  else {
    var str = '' + i;
    var buf = new Buffer(str, 'ascii');
    this.push(buf);
  }
};
```

#### Example: 简单协议 v1 (初始版)

和之前描述的 `parseHeader` 函数类似, 但它被实现为自定义流。注意这个实现不会将输入数据转换为字符串。

实际上，更好的办法是将他实现为 [Transform][] 流。下面的实现方法更好。

```  
javascript
// A parser for a simple data protocol.
// "header" is a JSON object, followed by 2 \n characters, and
// then a message body.
//
// 注意: This can be done more simply as a Transform stream!
// Using Readable directly for this is sub-optimal.  See the
// alternative example below under Transform section.

var Readable = require('stream').Readable;
var util = require('util');

util.inherits(SimpleProtocol, Readable);

  function SimpleProtocol(source, options) {
  if (!(this instanceof SimpleProtocol))
    return new SimpleProtocol(source, options);

  Readable.call(this, options;
  
  this._inBody = false;
  this._sawFirstCr = false;

  // source is 可读流（Readable stream）, such as a socket or file
  this._source = source;

  var self = this;
  source.on('end', function() {
    self.push(null);
  });

  // give it a kick whenever the source is readable
  // read(0) will not consume any bytes
  source.on('readable', function() {
    self.read(0);
  });

  this._rawHeader = [];
  this.header = null;
}

SimpleProtocol.prototype._read = function(n) {
  if (!this._inBody) {
    var chunk = this._source.read();

    // if the source doesn't have data, we don't have data yet.
    if (chunk === null)
      return this.push('');

    // check if the chunk has a \n\n
    var split = -1;
    for (var i = 0; i < chunk.length; i++) {
      if (chunk[i] === 10) { // '\n'
        if (this._sawFirstCr) {
          split = i;
          break;
        } else {
          this._sawFirstCr = true;
        }
      } else {
        this._sawFirstCr = false;
      }
    }

    if (split === -1) {
      // still waiting for the \n\n
      // stash the chunk, and try again.
      this._rawHeader.push(chunk);
      this.push('');
    } else {
      this._inBody = true;
      var h = chunk.slice(0, split);
      this._rawHeader.push(h);
      var header = Buffer.concat(this._rawHeader).toString();
      try {
        this.header = JSON.parse(header);
      } catch (er) {
        this.emit('error', new Error('invalid simple protocol data'));
        return;
      }
      // now, because we got some extra data, unshift the rest
      // back into the 读取队列 so that our consumer will see it.
      var b = chunk.slice(split);
      this.unshift(b);

      // and let them know that we are done parsing the header.
      this.emit('header', this.header);
    }
  } else {
    // from there on, just provide the data to our consumer.
    // careful not to push(null), since that would indicate EOF.
    var chunk = this._source.read();
    if (chunk) this.push(chunk);
  }
};

// Usage:
// var parser = new SimpleProtocol(source);
// Now parser is 可读流（Readable stream） that will emit 'header'
// with the parsed header data.
```


#### new stream.Readable([options])

* `options` {Object}
  * `highWaterMark` {Number} 停止从底层资源读取数据前，存储在内部缓存的最大字节数。默认=16kb,  `objectMode` 流是16.
  * `encoding` {String} 若指定，则 Buffer 会被解码成所给编码的字符串。缺省为 null
  * `objectMode` {Boolean} 该流是否为对象的流。意思是说 stream.read(n) 返回一个单独的值，而不是大小为 n 的 Buffer。

Readable 的扩展类中，确保调用了 Readable 的构造函数，这样才能正确初始化。

#### readable.\_read(size)

* `size` {Number} 异步读取的字节数

注意: **实现这个函数, 但不要直接调用.**

这个函数不要直接调用.  在子类里实现,仅能被内部的 Readable 类调用。

所有可读流（Readable stream） 的实现必须停供一个 `_read` 方法，从底层资源里获取数据。

这个方法以下划线开头，是因为对于定义它的类是内部的，不会被用户程序直接调用。 你可以在自己的扩展类中实现。

当数据可用时，通过调用`readable.push(chunk)` 将之放到读取队列中。再次调用 `_read` ，需要继续推出更多数据。

`size` 参数仅供参考. 调用 “read” 可以知道知道应当抓取多少数据；其余与之无关的实现，比如 TCP 或 TLS，则可忽略这个参数，并在可用时返回数据。例如，没有必要“等到” size 个字节可用时才调用stream.push(chunk)。

#### readable.push(chunk[, encoding])

* `chunk` {Buffer | null | String} 推入到读取队列的数据块
* `encoding` {String} 字符串块的编码。必须是有效的 Buffer 编码，比如 utf8 或 ascii。
* 返回 {Boolean} 是否应该继续推入

注意: **这个函数必须被 Readable 实现者调用, 而不是可读流（Readable stream）的消费者.**

`_read()` 函数直到调用`push(chunk)` 后才能被再次调用。  

`Readable` 类将数据放到读取队列，当 `'readable'` 事件触发后，被 `read()` 方法取出。`push()` 方法会插入数据到读取队列中。如果调用了 `null` ，会触发	数据结束信号 (EOF)。


这个 API 被设计成尽可能地灵活。比如说，你可以包装一个低级别的，具备某种暂停/恢复机制，和数据回调的数据源。这种情况下，你可以通过这种方式包装低级别来源对象：

```  
javascript
// source is an object with readStop() and readStart() 方法s,
// and an `ondata` member that gets called when it has data, and
// an `onend` member that gets called when the data is over.

util.inherits(SourceWrapper, Readable);

function SourceWrapper(options) {
  Readable.call(this, options);

  this._source = getLowlevelSourceObject();
  var self = this;

  // Every time there's data, we push it into the internal buffer.
  this._source.ondata = function(chunk) {
    // if push() 返回 false, then we need to stop reading from source
    if (!self.push(chunk))
      self._source.readStop();
  };

  // When the source ends, we push the EOF-signaling `null` chunk
  this._source.onend = function() {
    self.push(null);
  };
}

// _read will be called when the stream wants to pull more data in
// the advisory size 参数 is ignored in this case.
SourceWrapper.prototype._read = function(size) {
  this._source.readStart();
};
```


### 类： stream.Writable

<!--type=class-->

`stream.Writable` 是个抽象类，它扩展了一个底层的实现[`_write(chunk, encoding, callback)`][] 方法.

参考上面的[API for Stream Consumers][]，来了解在你的程序里如何消费可写流。下面内容介绍了如何在你的程序里实现可写流。

#### new stream.Writable([options])

* `options` {Object}
  * `highWaterMark` {Number} 当 [`write()`][] 返回 false 时的缓存级别. 默认=16kb,`objectMode` 流是 16.
  * `decodeStrings` {Boolean} 传给  [`_write()`][] 前是否解码为字符串。  默认=true
  * `objectMode` {Boolean} `write(anyObj)` 是否是有效操作.如果为 true，可以写任意数据，而不仅仅是`Buffer` / `String`.  默认=false

请确保 Writable 类的扩展类中，调用构造函数以便缓冲设定能被正确初始化。

#### writable.\_write(chunk, encoding, callback)

* `chunk` {Buffer | String} 要写入的数据块。总是 buffer， 除非 `decodeStrings` 选项为 `false`。
* `encoding` {String} 如果数据块是字符串，这个参数就是编码方式。如果是缓存，则忽略。注意，除非`decodeStrings` 被设置为 `false` ，否则这个数据块一直是buffer。
* `callback` {函数} 当你处理完数据后调用这个函数 (错误参数为可选参数)。

所以可写流（Writable stream ） 实现必须提供一个 [`_write()`][]方法，来发送数据给底层资源。 

注意: **这个函数不能直接调用** ,由子类实现， 仅内部可写方法可以调用。

使用标准的 `callback(error)` 方法调用回调函数，来表明写入完成或遇到错误。

如果构造函数选项中设定了 `decodeStrings` 标识，则 `chunk` 可能会是字符串而不是 Buffer， `encoding` 表明了字符串的格式。这种设计是为了支持对某些字符串数据编码提供优化处理的实现。如果你没有明确的设置`decodeStrings` 为  `false`，这样你就可以安不管 `encoding` 参数，并假定 `chunk` 一直是一个缓存。

该方法以下划线开头，是因为对于定义它的类来说，这个方法是内部的，并且不应该被用户程序直接调用。你应当在你的扩充类中重写这个方法。

### writable.\_writev(chunks, callback)

* `chunks` {Array} 准备写入的数据块，每个块格式如下: `{ chunk: ..., encoding: ... }`.
* `callback` {函数} 当你处理完数据后调用这个函数 (错误参数为可选参数)。

注意: **这个函数不能直接调用。**  由子类实现，仅内部可写方法可以调用.

这个函数的实现是可选的。多数情况下，没有必要实现。如果实现，将会在所有数据块缓存到写队列后调用。

### 类： stream.Duplex

<!--type=class-->

双工流（duplex stream）同时兼具可读和可写特性，比如一个 TCP socket 连接。

注意 `stream.Duplex` 可以像 Readable 或 Writable 一样被扩充，实现了底层 _read(sise) 和 _write(chunk, encoding, callback) 方法的抽象类。

由于 JavaScript 并没有多重继承能力，因此这个类继承自 Readable，寄生自 Writable.从而让用户在双工扩展类中同时实现低级别的`_read(n)` 方法和低级别的[`_write(chunk, encoding, callback)`][]方法。

#### new stream.Duplex(options)

* `options` {Object} 传递 Writable and Readable 构造函数，有以下的内容：
  * `allowHalfOpen` {Boolean} 默认=true.  如果设置为 `false`, 当写端结束的时候，流会自动的结束读端，反之亦然。
  * `readableObjectMode` {Boolean} 默认=false. 将 `objectMode` 设为读端的流，如果为 `true`，将没有效果。
  * `writableObjectMode` {Boolean} 默认=false. 将 `objectMode`设为写端的流，如果为 `true`，将没有效果。

扩展自 Duplex 的类，确保调用了父亲的构造函数，保证缓存设置能正确初始化。


### 类： stream.Transform

转换流（transform class) 是双工流（duplex stream），输入输出端有因果关系，比如[zlib][] 流或 [crypto][] 流。

输入输出没有要求大小相同，块数量相同，到达时间相同。例如，一个 Hash 流只会在输入结束时产生一个数据块的输出；一个 zlib 流会产生比输入小得多或大得多的输出。

转换流（transform class) 必须实现`_transform()` 方法，而不是[`_read()`][] 和 [`_write()`][] 方法，也可以实现`_flush()` 方法（参见如下）。

#### new stream.Transform([options])

* `options` {Object} 传递给 Writable and Readable 构造函数。

扩展自 转换流（transform class) 的类，确保调用了父亲的构造函数，保证缓存设置能正确初始化。

#### transform.\_transform(chunk, encoding, callback)

* `chunk` {Buffer | String} 准备转换的数据块。是buffer，除非 `decodeStrings`  选项设置为 `false`。
* `encoding` {String} 如果数据块是字符串, 这个参数就是编码方式，否则就忽略这个参数  
* `callback` {函数} 当你处理完数据后调用这个函数 (错误参数为可选参数)。


注意: **这个函数不能直接调用。**  由子类实现，仅内部可写方法可以调用.

所有的转换流（transform class) 实现必须提供 `_transform`方法来接收输入，并生产输出。

`_transform` 可以做转换流（transform class)里的任何事，处理写入的字节，传给接口的写端，异步 I/O,处理事情等等。

调用 `transform.push(outputChunk)`0或多次，从这个输入块里产生输出，依赖于你想要多少数据作为输出。

仅在当前数据块完全消费后调用这个回调。注意，输入块可能有，也可能没有对应的输出块。如果你提供了第二个参数，将会传给push 方法。如底下的例子

```  
javascript
transform.prototype._transform = function (data, encoding, callback) {
  this.push(data);
  callback();
}

transform.prototype._transform = function (data, encoding, callback) {
  callback(null, data);
}
```
该方法以下划线开头，是因为对于定义它的类来说，这个方法是内部的，并且不应该被用户程序直接调用。你应当在你的扩充类中重写这个方法。

#### transform.\_flush(callback)

* `callback` {函数} 当你处理完数据后调用这个函数 (错误参数为可选参数)

注意: **这个函数不能直接调用。**  由子类实现，仅内部可写方法可以调用.

某些情况下，转换操作可能需要分发一点流最后的数据。例如， `Zlib`流会存储一些内部状态，以便优化压缩输出。

有些时候，你可以实现 `_flush` 方法，它可以在最后面调用，当所有的写入数据被消费后，分发`end`告诉读端。和 `_transform` 一样，当刷新操作完毕， `transform.push(chunk)` 为0或更多次数，。

该方法以下划线开头，是因为对于定义它的类来说，这个方法是内部的，并且不应该被用户程序直接调用。你应当在你的扩充类中重写这个方法。

#### 事件: 'finish' and 'end'
  
[`finish`][] 和 [`end`][] 事件 分别来自 Writable 和 Readable 类。`.end()`事件结束后调用 `finish` 事件，所有的数据已经被`_transform`处理完毕，调用 `_flush` 后，所有的数据输出完毕，触发`end`。


#### Example: `SimpleProtocol` parser v2

上面的简单协议分析例子列子可以通过使用高级别的[Transform][] 流来实现，和 `parseHeader` ， `SimpleProtocol v1`列子类似。

在这个示例中，输入会被导流到解析器中，而不是作为参数提供。这种做法更符合 Node 流的惯例。

```  
javascript
var util = require('util');
var Transform = require('stream').Transform;
util.inherits(SimpleProtocol, Transform);

function SimpleProtocol(options) {
  if (!(this instanceof SimpleProtocol))
    return new SimpleProtocol(options);

  Transform.call(this, options);
  this._inBody = false;
  this._sawFirstCr = false;
  this._rawHeader = [];
  this.header = null;
}

SimpleProtocol.prototype._transform = function(chunk, encoding, done) {
  if (!this._inBody) {
    // check if the chunk has a \n\n
    var split = -1;
    for (var i = 0; i < chunk.length; i++) {
      if (chunk[i] === 10) { // '\n'
        if (this._sawFirstCr) {
          split = i;
          break;
        } else {
          this._sawFirstCr = true;
        }
      } else {
        this._sawFirstCr = false;
      }
    }

    if (split === -1) {
      // still waiting for the \n\n
      // stash the chunk, and try again.
      this._rawHeader.push(chunk);
    } else {
      this._inBody = true;
      var h = chunk.slice(0, split);
      this._rawHeader.push(h);
      var header = Buffer.concat(this._rawHeader).toString();
      try {
        this.header = JSON.parse(header);
      } catch (er) {
        this.emit('error', new Error('invalid simple protocol data'));
        return;
      }
      // and let them know that we are done parsing the header.
      this.emit('header', this.header);

      // now, because we got some extra data, emit this first.
      this.push(chunk.slice(split));
    }
  } else {
    // from there on, just provide the data to our consumer as-is.
    this.push(chunk);
  }
  done();
};

// Usage:
// var parser = new SimpleProtocol();
// source.pipe(parser)
// Now parser is 可读流（Readable stream） that will emit 'header'
// with the parsed header data.
```


### 类： stream.PassThrough

这是[Transform][] 流的简单实现，将输入的字节简单的传递给输出。它的主要用途是测试和演示。偶尔要构建某种特殊流时也会用到。


## 流: 内部细节

<!--type=misc-->

### 缓冲

<!--type=misc-->

可写流（Writable streams ） 和 可读流（Readable stream）都会缓存数据到内部对象上，叫做 `_writableState.buffer` 或 `_readableState.buffer`。  

缓存的数据量，取决于构造函数是传入的 `highWaterMark` 参数。


调用 [`stream.push(chunk)`][] 时，缓存数据到可读流（Readable stream）。在数据消费者调用 `stream.read()` 前，数据会一直缓存在内部队列中。

调用 [`stream.write(chunk)`][] 时，缓存数据到可写流（Writable stream）。即使 `write()` 返回 `false`。

流（尤其是`pipe()` 方法）得目的是限制数据的缓存量到一个可接受的水平，使得不同速度的源和目的不会淹没可用内存。

### `stream.read(0)`

某些时候，你可能想不消费数据的情况下，触发底层可读流（Readable stream）机制的刷新。这种情况下可以调用 stream.read(0)，它总会返回 null。

如果内部读取缓冲低于  `highWaterMark`，并且流当前不在读取状态，那么调用 `read(0)`  会触发一个低级  `_read` 调用。

虽然基本上没有必要这么做。但你在 Node 内部的某些地方看到它确实这么做了，尤其是在 Readable 流类的内部。

### `stream.push('')`

推一个0字节的字符串或缓存 (不在[Object mode][]时)会发送有趣的副作用.  因为它是一个对
[`stream.push()`][] 的调用, 它将会结束 `reading` 进程.  然而，它没有添加任何数据到可读缓冲区中，所以没有东西可供用户消费。  

少数情况下，你当时没有提供数据，但你的流的消费者（或你的代码的其它部分）会通过调用 `stream.read(0)` 得知何时再次检查。在这种情况下，你可以调用 `stream.push('')`。  
  
到目前为止，这个功能唯一一个使用情景是在 [tls.CryptoStream][] 类中，但它将在 Node v0.12 中被废弃。如果你发现你不得不使用 `stream.push('')`，请考虑另一种方式。

<a name="stream_compatibility_with_older_node_versions"></a>
### 和老版本的兼容性

<!--type=misc-->

v0.10 版本前，可读流（Readable stream）接口比较简单，因此功能和用处也小。

*  `'data'`事件会立即开始触发，而不会等待你调用 `read()` 方法。如果你需要进行某些 I/O 来决定如何处理数据，那么你只能将数据块储存到某种缓冲区中以防它们流失。   
*  [`pause()`][] 方法仅供参考，而不保证生效。这意味着，即便流处于暂停状态时，你仍然需要准备接收 'data' 事件。

在 Node v0.10中, 加入了下文所述的 Readable 类。为了考虑向后兼容，添加了 'data' 事件监听器或 resume() 方法被调用时，可读流（Readable stream）会切换到 "流动模式（flowing mode）"。其作用是，即便你不使用新的 `read()`  方法和`'readable'`事件，你也不必担心丢失`'data'` 数据块。


大多数程序会维持正常功能。然而，下列条件下也会引入边界情况：

* 没有添加 [`'data'` 事件][] 处理器
* 从来没有调用 [`resume()`][] 方法
* 流从来没有被倒流（pipe）到任何可写目标上、  

例如：

```  
javascript
// WARNING!  BROKEN!
net.createServer(function(socket) {

  // we add an 'end' 方法, but never consume the data
  socket.on('end', function() {
    // It will never get here.
    socket.end('I got your message (but didnt read it)\n');
  });

}).listen(1337);
```

v0.10 版本前的 Node, 流入的消息数据会被简单的抛弃。之后的版本，socket 会一直保持暂停。  

这种情形下，调用`resume()` 方法来开始工作：

```  
javascript
// Workaround
net.createServer(function(socket) {

  socket.on('end', function() {
    socket.end('I got your message (but didnt read it)\n');
  });

  // start the flow of data, discarding it.
  socket.resume();

}).listen(1337);
```

可读流（Readable stream）切换到流动模式（flowing mode）,v0.10 版本前，可以使用`wrap()` 方法将风格流包含在一个可读类里。



### Object Mode

<!--type=misc-->

通常情况下，流仅操作字符串和缓存。

处于 **object mode** 的流，除了 缓存和字符串，还可以可以读出普通 JavaScript值。

在对象模式里，可读流（Readable stream） 调用 `stream.read(size)`总会返回单个项目，无论是什么参数。

在对象模式里， 可写流（Writable stream ） 总会忽略传给`stream.write(data, encoding)`的 `encoding`参数。

特殊值 `null` 在对象模式里，依旧保持它的特殊性。也就说，对于对象模式的可读流（Readable stream），`stream.read()` 返回 null 意味着没有更多数据，同时[`stream.push(null)`][] 会告知流数据结束（EOF）。

Node 核心不存在对象模式的流，这种设计只被某些用户态流式库所使用。

应该在你的子类构造函数里，设置`objectMode` 。在过程中设置不安全。


对于双工流（Duplex streams），`objectMode`可以用`readableObjectMode` 和 `writableObjectMode` 分别为读写端分别设置。这些选项，被转换流（Transform streams）用来实现解析和序列化。

```  
javascript
var util = require('util');
var StringDecoder = require('string_decoder').StringDecoder;
var Transform = require('stream').Transform;
util.inherits(JSONParseStream, Transform);

// Gets \n-delimited JSON  string data, and emits the parsed objects
function JSONParseStream() {
  if (!(this instanceof JSONParseStream))
    return new JSONParseStream();

  Transform.call(this, { readableObjectMode : true });

  this._buffer = '';
  this._decoder = new StringDecoder('utf8');
}

JSONParseStream.prototype._transform = function(chunk, encoding, cb) {
  this._buffer += this._decoder.write(chunk);
  // split on newlines
  var lines = this._buffer.split(/\r?\n/);
  // keep the last partial line buffered
  this._buffer = lines.pop();
  for (var l = 0; l < lines.length; l++) {
    var line = lines[l];
    try {
      var obj = JSON.parse(line);
    } catch (er) {
      this.emit('error', er);
      return;
    }
    // push the parsed object out to the readable consumer
    this.push(obj);
  }
  cb();
};

JSONParseStream.prototype._flush = function(cb) {
  // Just handle any leftover
  var rem = this._buffer.trim();
  if (rem) {
    try {
      var obj = JSON.parse(rem);
    } catch (er) {
      this.emit('error', er);
      return;
    }
    // push the parsed object out to the readable consumer
    this.push(obj);
  }
  cb();
};
```


[EventEmitter]: events.html#events_class_events_eventemitter
[Object mode]: #stream_object_mode
[`stream.push(chunk)`]: #stream_readable_push_chunk_encoding
[`stream.push(null)`]: #stream_readable_push_chunk_encoding
[`stream.push()`]: #stream_readable_push_chunk_encoding
[`unpipe()`]: #stream_readable_unpipe_destination
[unpiped]: #stream_readable_unpipe_destination
[tcp sockets]: net.html#net_class_net_socket
[zlib streams]: zlib.html
[zlib]: zlib.html
[crypto streams]: crypto.html
[crypto]: crypto.html
[tls.CryptoStream]: tls.html#tls_class_cryptostream
[process.stdin]: process.html#process_process_stdin
[stdout]: process.html#process_process_stdout
[process.stdout]: process.html#process_process_stdout
[process.stderr]: process.html#process_process_stderr
[child process stdout and stderr]: child_process.html#child_process_child_stdout
[API for Stream Consumers]: #stream_api_for_stream_consumers
[API for Stream Implementors]: #stream_api_for_stream_implementors
[Readable]: #stream_class_stream_readable
[Writable]: #stream_class_stream_writable
[Duplex]: #stream_class_stream_duplex
[Transform]: #stream_class_stream_transform
[`end`]: #stream_event_end
[`finish`]: #stream_event_finish
[`_read(size)`]: #stream_readable_read_size_1
[`_read()`]: #stream_readable_read_size_1
[_read]: #stream_readable_read_size_1
[`writable.write(chunk)`]: #stream_writable_write_chunk_encoding_callback
[`write(chunk, encoding, callback)`]: #stream_writable_write_chunk_encoding_callback
[`write()`]: #stream_writable_write_chunk_encoding_callback
[`stream.write(chunk)`]: #stream_writable_write_chunk_encoding_callback
[`_write(chunk, encoding, callback)`]: #stream_writable_write_chunk_encoding_callback_1
[`_write()`]: #stream_writable_write_chunk_encoding_callback_1
[_write]: #stream_writable_write_chunk_encoding_callback_1
[`util.inherits`]: util.html#util_util_inherits_constructor_superconstructor
[`end()`]: #stream_writable_end_chunk_encoding_callback
[`'data'` event]: #stream_event_data
[`resume()`]: #stream_readable_resume
[`readable.resume()`]: #stream_readable_resume
[`pause()`]: #stream_readable_pause
[`unpipe()`]: #stream_readable_unpipe_destination
[`pipe()`]: #stream_readable_pipe_destination_options
