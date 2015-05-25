# HTTP

    稳定性: 3 - 稳定
  
使用 HTTP 服务器或客户端功能必须调用 `require('http')`。  

Node 里的 HTTP 接口支持协议里原本比较难用的特性。特别是很大的或块编码的消息。这些接口不会完全缓存整个请求或响应，这样用户可以在请求或响应中使用数据流。


HTTP消息头对象和下面的例子类似：  

    { 'content-length': '123',
      'content-type': 'text/plain',
      'connection': 'keep-alive',
      'host': 'mysite.com',
      'accept': '*/*' }

Keys 都是小写，值不能修改。

为了能支持尽可能多的 HTTP 应用程序，Node 提供的 HTTP API 接口都是底层的。仅能处理流和消息。它把消息解析成报文头和报文体，但是不解析实际的报文头和报文体内容。

定义报文头的时候，多个值间可用 `,` 分隔。除了 `set-cookie` 和 `cookie` 头，因为它们表示值的数组。诸如 `content-length` 的只有一个值的报文头，直接解析，并且只有单值可以表示成已经解析好的对象。

接收到的原始头信息以数组（`[key, value, key2, value2, ...]`）的形式保存在 `rawHeaders` 里。例如，之前提到的消息对象会有如下的 `rawHeaders`：

    [ 'ConTent-Length', '123456',
      'content-LENGTH', '123',
      'content-type', 'text/plain',
      'CONNECTION', 'keep-alive',
      'Host', 'mysite.com',
      'accepT', '*/*' ]

## http.METHODS

* {Array}


解析器支持的 HTTP 方法列表。

## http.STATUS_CODES

* {Object}

全部标准 HTTP 响应状态码的和描述的集合。例如，`http.STATUS_CODES[404] === 'Not Found'`。

## http.createServer([requestListener])

返回 [http.Server](#http_class_http_server) 的新实例。

`requestListener` 函数自动加到`'request'` 事件里。

## http.createClient([port][, host])

这个函数已经被抛弃，用 [http.request()][] 替换。创建一个新的 HTTP 客户端，连接到服务器的 `port` 和 `host` 。

## 类： http.Server

这是事件分发器 [EventEmitter][]，有以下事件：

### 事件： 'request'

`function (request, response) { }`

每当有请求的时候触发。注意：每个连接可以有多个请求（在 keep-alive 连接中）。`request` 是 [http.IncomingMessage][] 实例，`response` 是 [http.ServerResponse][] 的实例。

### 事件： 'connection'

`function (socket) { }`

当建立新的 TCP 流的时候。 `socket` 是一个 `net.Socket` 对象。通常用户不会访问这个事件。协议解析器绑定套接字时采用的方式使套接字不会出发 readable 事件。也能通过 `request.connection` 访问 `socket`。

### 事件： 'close'

`function () { }`

服务器关闭的时候触发。

### 事件： 'checkContinue'

`function (request, response) { }`

当 http 收到 100-continue 的 http 请求时会触发。如果没有监听这个事件，服务器将会自动发送 100 Continue 的响应。

如果客户端需要继续发送请求主题，或者生成合适的 HTTP 响应（如，400 请求无效），可以通过调用 [response.writeContinue()][] 来处理。

注意：触发并处理这个事件的时候，不会再触发 `request` 事件。 

### 事件： 'connect'

`function (request, socket, head) { }`

当客户端请求 http 连接时触发。如果没有监听这个事件，客户端请求连接的时候会被关闭。

* `request` 是 http 请求的参数，与 request 事件参数相同。
* `socket` 是服务器和客户端间的 socket。
* `head` 是 buffer 的实例。网络隧道的第一个包，可能为空。

这个事件触发后，请求的 socket 不会有  `data` 事件监听器，也就是说你需要绑定一个监听器到 `data` 上，来处理在发送到服务器上的 socket 数据。

### 事件： 'upgrade'

`function (request, socket, head) { }`

当客户端请求 http upgrage 时候会触发。如果没有监听这个事件，客户端请求一个连接的时候会被关闭。

* `request`  是 http 请求的参数，与 request 事件参数相同。
* `socket` 是服务器和客户端间的 socket。
* `head` 是 buffer 的实例。网络隧道的第一个包，可能为空。

这个事件触发后，请求的 socket 不会有  `data` 事件监听器，也就是说你需要绑定一个监听器到 `data` 上，来处理在发送到服务器上的 socket 数据。

### 事件： 'clientError'

`function (exception, socket) { }`

如果一个客户端连接触发了一个 'error' 事件, 它就会转发到这里.

`socket` 是导致错误的 `net.Socket` 对象。

<a name="http_server_listen_port_hostname_backlog_callback"></a>
### server.listen(port[, hostname][, backlog][, callback])

在指定的的端口和主机名上开始接收连接。 如果忽略主机名，服务器将会接收指向任意 IPv4 的地址(`INADDR_ANY`)。

监听一个 unix socket，需要提供一个文件名而不是主机名和端口。

积压量 backlog 为等待连接队列的最大长度。实际的长度由你的操作系统的 sysctl 设置决定（比如 linux 上的 `tcp_max_syn_backlog` and `somaxconn`）。默认参数值为 511 (不是 512)

这是异步函数。最后一个参数 `callback` 会作为事件监听器添加到 `listening` 事件。参见[net.Server.listen(port)][]。


### server.listen(path[, callback])

启动一个 UNIX socket 服务器所给路径 `path`   


这是异步函数。最后一个参数 `callback` 会作为事件监听器添加到 `listening`事件。参见[net.Server.listen(port)][]。


### server.listen(handle[, callback])

* `handle` {Object}
* `callback` {Function}

 `handle` 对象可以是 server 或 socket（任意以下划线 `_handle`开头的成员），或者`{fd: <n>}`对象。

这会导致 server 用参数 `handle` 接收连接，前提条件是文件描述符或句柄已经连接到端口或域 socket。

Windows 不能监听文件句柄。  

这是异步函数。最后一个参数 `callback` 会作为事件监听器添加到 `listening` 事件。参见[net.Server.listen(port)][]。

<a name="http_server_close_callback"></a>
### server.close([callback])

禁止 server 接收连接。参见 [net.Server.close()][].


### server.maxHeadersCount

最大请求头的数量限制，默认 1000。如果设置为 0，则不做任何限制。

<a name="http_server_settimeout_msecs_callback"></a>
### server.setTimeout(msecs, callback)

* `msecs` {Number}
* `callback` {Function}

为 socket 设置超时时间，单位为毫秒，如果发生超时，在 server 对象上触发 `'timeout'` 事件，参数为 socket 。

如果在 Server 对象上有一个 `'timeout'` 事件监听器，超时的时候，将会调用它，参数为 socket 。

默认情况下，Server 的超时为 2 分钟，如果超时将会销毁 socket。如果你给 Server 的超时事件设置了回调函数，那你就得负责处理 socket 超时。

<a name="http_server_timeout"></a>
### server.timeout

* {Number} Default = 120000 (2 minutes)

超时的时长，单位为毫秒。

注意，socket 的超时逻辑在连接时设定，所以有新的连接时才能改变这个值。

设为 0 时，建立连接的自动超时将失效。

## 类： http.ServerResponse

这是一个由 HTTP 服务器（而不是用户）内部创建的对象。作为第二个参数传递给 `'request'`事件。

该响应实现了 [Writable Stream][] 接口。这是一个包含下列事件的 [EventEmitter][] ：

### 事件： 'close'

`function () { }`

在调用 [response.end()][]，或准备 flush 前，底层连接结束。

### 事件： 'finish'

`function () { }`

发送完响应触发。响应头和响应体最后一段数据被剥离给操作系统后，通过网络来传输时被触发。这并不代表客户端已经收到数据。

这个事件之后，响应对象不会再触发任何事件。

### response.writeContinue()

发送 HTTP/1.1 100 Continue 消息给客户端，表示请求体可以发送。可以在服务器上查看['checkContinue'][] 事件。

### response.writeHead(statusCode[, statusMessage][, headers])

发送一个响应头给请求。状态码是 3 位数字，如 `404`。最后一个参数 `headers` 是响应头。建议第二个参数设置为可以看的懂的消息。

例如:

    var body = 'hello world';
    response.writeHead(200, {
      'Content-Length': body.length,
      'Content-Type': 'text/plain' });

这个方法仅能在消息中调用一次，而且必须在 [response.end()][] 前调用。

如果你在这之前调用 [response.write()][] 或 [response.end()][],将会计算出不稳定的头。

Content-Length 是字节数，而不是字符数。上面的例子 `'hello world'` 仅包含一个字节字符。如果 body 包含高级编码的字符， `Buffer.byteLength()` 就必须确定指定编码的字符数。Node 不会检查Content-Length 和 body 的长度是否相同。

### response.setTimeout(msecs, callback)

* `msecs` {Number}
* `callback` {Function}

设置 socket 超时时间，单位为毫秒。如果提供了回调函数，将会在 response 对象的 `'timeout'` 事件上添加监听器。  

如果没有给请求、响应、服务器添加 `'timeout'` 监视器，超时的时候将会销毁 socket。如果你给请求、响应、服务器加了处理函数，就需要负责处理 socket 超时。

### response.statusCode

使用默认的 headers 时（没有显式的调用 [response.writeHead()][] ），这个属性表示将要发送给客户端状态码。

例如:

    response.statusCode = 404;

响应头发送给客户端的后，这个属性表示状态码已经发送。

### response.statusMessage


使用默认 headers 时（没有显式的调用 [response.writeHead()][] ）, 这个属性表示将要发送给客户端状态信息。 如果这个没有定义，将会使用状态码的标准消息。

例如:

    response.statusMessage = 'Not found';

当响应头发送给客户端的时候，这个属性表示状态消息已经发送。

### response.setHeader(name, value)

设置默认头某个字段内容。如果这个头即将被发送，内容会被替换。如果你想设置更多的头， 就使用一个相同名字的字符串数组。

例如:

    response.setHeader("Content-Type", "text/html");

或

    response.setHeader("Set-Cookie", ["type=ninja", "language=javascript"]);

### response.headersSent

Boolean (只读)。如果headers发送完毕,则为 true,反之为 false。

### response.sendDate

默认值为 true。若为 true,当 headers 里没有 Date 值时，自动生成 Date 并发送。

只有在测试环境才能禁用; 因为 HTTP 要求响应包含 Date 头.

### response.getHeader(name)

读取一个在队列中但是还没有被发送至客户端的header。名字是大小写敏感。仅能再头被flushed前调用。

例如:

    var contentType = response.getHeader('content-type');

### response.removeHeader(name)

从即将发送的队列里移除头。

例如:

    response.removeHeader("Content-Encoding");


### response.write(chunk[, encoding][, callback])

如果调用了这个方法，且还没有调用 [response.writeHead()][]，将会切换到默认的 header，并更新这个header。

这个方法将发送响应体数据块。可能会多次调用这个方法，以提供 body 成功的部分内容。

`chunk` 可以是字符串或 buffer。如果 `chunk` 是字符串，第二个参数表明如何将它编码成字节 流。`encoding` 的默认值是`'utf8'`。最后一个参数在刷新这个数据块时调用。

注意：这个是原始的 HTTP body，和高级的multi-part body 编码无关。

第一次调用 `response.write()` 的时候，将会发送缓存的头信息和第一个 body 给客户端。第二次，将会调用 `response.write()`。Node 认为你将会独立发送流数据。这意味着，响应缓存在第一个数据块中。

如果成功的刷新全部数据到内核缓冲区，返回 `true` 。如果部分或全部数据在用户内存中还处于排队状况，返回 `false` 。当缓存再次释放的时候，将会触发 `'drain'`。

### response.addTrailers(headers)

这个方法给响应添加 HTTP 的尾部 header（消息末尾的 header）。

**只有**数据块编码用于响应体时，才会触发 Trailers；如果不是（例如，请求是HTTP/1.0），它们将会被自动丢弃。

如果你想触发 trailers， HTTP 会要求发送 `Trailer` 头，它包含一些信息，比如：

    response.writeHead(200, { 'Content-Type': 'text/plain',
                              'Trailer': 'Content-MD5' });
    response.write(fileData);
    response.addTrailers({'Content-MD5': "7895bf4b8828b55ceaf47747b4bca667"});
    response.end();


### response.end([data][, encoding][, callback])

这个方法告诉服务器，所有的响应头和响应体已经发送；服务器可以认为消息结束。`response.end()` 方法必须在每个响应中调用。

如果指定了参数 `data`，将会在响应流结束的时候调用。

<a name="http_http_request_options_callback"></a>
## http.request(options[, callback])

Node 维护每个服务器的连接来生成 HTTP 请求。这个函数让你可以发布请求。

参数`options`是对象或字符串。如果 `options` 是字符串，会通过 [url.parse()][] 自动解析。

`options` 值:

- `host`: 请求的服务器域名或 IP 地址，默认：`'localhost'`
- `hostname`: 用于支持 `url.parse()`。 `hostname` 优于 `host`
- `port`: 远程服务器端口。 默认： 80.
- `localAddress`: 用于绑定网络连接的本地接口
- `socketPath`: Unix域 socket（使用host:port或socketPath
- `method`: 指定 HTTP 请求方法。 默认： `'GET'`.
- `path`: 请求路径。 默认： `'/'`。如果有查询字符串，则需要包含。例如'/index.html?page=12'。请求路径包含非法字符时抛出异常。目前，只有空格不行，不过在未来可能改变。
- `headers`: 包含请求头的对象
- `auth`: 用于计算认证头的基本认证，即 `user:password`
- `agent`: 控制Agent的行为。当使用了一个Agent的时候，请求将默认为`Connection: keep-alive`。可能的值为：
 - `undefined` (default): 在这个主机和端口上使用 [global Agent][]
 - `Agent` object: 在`Agent`中显式使用 passed .
 - `false`: 选择性停用连接池,默认请求为： `Connection: close`.
 - `keepAlive`: {Boolean} 持资源池周围的socket，用于未来其它请求。默认值为`false`。
 - `keepAliveMsecs`: {Integer} 使用HTTP KeepAlive 的时候，通过正在保持活动的sockets发送TCP KeepAlive包的频繁程度。默认值为1000。仅当keepAlive为true时才相关。  

可选参数  `callback` 将会作为一次性的监视器，添加给 ['response'][] 事件。

http.request() 返回一个 http.ClientRequest类的实例。ClientRequest实例是一个可写流对象。如果需要用 POST 请求上传一个文件的话，就将其写入到ClientRequest对象。

例如：

    var postData = querystring.stringify({
      'msg' : 'Hello World!'
    });

    var options = {
      hostname: 'www.google.com',
      port: 80,
      path: '/upload',
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Content-Length': postData.length
      }
    };

    var req = http.request(options, function(res) {
      console.log('STATUS: ' + res.statusCode);
      console.log('HEADERS: ' + JSON.stringify(res.headers));
      res.setEncoding('utf8');
      res.on('data', function (chunk) {
        console.log('BODY: ' + chunk);
      });
    });

    req.on('error', function(e) {
      console.log('problem with request: ' + e.message);
    });

    // write data to request body
    req.write(postData);
    req.end();

注意，例子里调用了 `req.end()`。`http.request()` 必须调用 `req.end()` 来表明请求已经完成，即使没有数据写入到请求 body 里。

如果在请求的时候遇到错误（DNS 解析、TCP 级别的错误或实际 HTTP 解析错误），在返回的请求对象时会触发一个 'error' 事件。

有一些特殊的头需要注意：


* 发送 `Connection: keep-alive` 告诉服务器保持连接，直到下一个请求到来。

* 发送 `Content-length` 头将会禁用chunked编码。

* 发送一个 `Expect` 头，会立即发送请求头，一般来说，发送 `Expect: 100-continue` ,你必须设置超时，并监听 `continue` 事件。更多细节参见 RFC2616 Section 8.2.3 。

* 发送一个授权头，将会使用 `auth` 参数重写，来计算基本的授权。
  

## http.get(options[, callback])

因为多数请求是没有报文体的 GET 请求， Node 提供了这个简便的方法。和 `http.request()` 唯一不同点在于，这个方法自动设置 GET，并自动调用 `req.end()`。

例如：

    http.get("http://www.google.com/index.html", function(res) {
      console.log("Got response: " + res.statusCode);
    }).on('error', function(e) {
      console.log("Got error: " + e.message);
    });

<a name="http_class_http_agent"></a>
## 类： http.Agent

HTTP Agent 用于 socket 池，用于 HTTP 客户端请求。

HTTP Agent 也把客户端请求默认为使用 Connection:keep-alive 。如果没有 HTTP 请求正在等着成为空闲 socket 的话，那么 socket 将关闭。这意味着，Node 的资源池在负载的情况下对 keep-alive 有利，但是仍然不需要开发人员使用 KeepAlive 来手动关闭 HTTP 客户端。
 
如果你选择使用 HTTP KeepAlive， 可以创建一个 Agent 对象，将 flag 设置为 `true`.  (参见下面的 [constructor options](#http_new_agent_options) ) ，这样 Agent 会把没用到的 socket 放到池里，以便将来使用。他们会被显式的标志，让 Node 不运行。但是，当不再使用它的时候，需要显式的调用[`destroy()`](#http_agent_destroy)，这样 socket 将会被关闭。

当 socket 事件触发  `close`  事件或特殊的 `agentRemove` 事件时，socket 将会从 agent 池里移除。如果你要保持 HTTP 请求保持长时间打开，并且不希望他们在池里，可以参考以下代码：

    http.get(options, function(res) {
      // Do stuff
    }).on("socket", function (socket) {
      socket.emit("agentRemove");
    });

另外，你可以使用 `agent:false` 让资源池停用：

    http.get({
      hostname: 'localhost',
      port: 80,
      path: '/',
      agent: false  // create a new agent just for this one request
    }, function (res) {
      // Do stuff with response
    })

### new Agent([options])

* `options` {Object} agent 上的设置选项集合，有以下字段内容:
  * `keepAlive` {Boolean} 持资源池周围的 socket，用于未来其它请求。默认值为 `false`。
  * `keepAliveMsecs` {Integer} 使用 HTTP KeepAlive 的时候，通过正在保持活动的 sockets 发送 TCP KeepAlive 包的频繁程度。默认值为 1000。仅当 keepAlive 为 true 时才相关。.】
  * `maxSockets` {Number} 在空闲状态下,还依然开启的 socket 的最大值。仅当 `keepAlive` 设置为 true 的时候有效。默认值为 256。

被  `http.request` 使用的默认的 `http.globalAgent` ,会设置全部的值为默认。

必须在创建你自己的 `Agent`  对象后，才能配置这些值。

```javascript
var http = require('http');
var keepAliveAgent = new http.Agent({ keepAlive: true });
options.agent = keepAliveAgent;
http.request(options, onResponseCallback);
```

### agent.maxSockets

默认值为 Infinity。决定了每台主机上的 agent 可以拥有的并发 socket 的打开数量，主机可以是 `host:port` 或 `host:port:localAddress`。

### agent.maxFreeSockets

默认值 256.  对于支持 HTTP KeepAlive 的 Agent 而言，这个方法设置了空闲状态下仍然打开的套接字数的最大值。 

### agent.sockets

这个对象包含了当前 Agent 使用中的 socket 数组。不要修改它。

### agent.freeSockets

使用 HTTP KeepAlive 的时候，这个对象包含等待当前 Agent 使用的 socket 数组。不要修改它。

### agent.requests

这个对象包含了还没分配给 socket 的请求数组。不要修改它。

### agent.destroy()

销毁任意一个被 agent 使用 的socket。

通常情况下不要这么做。如果你正在使用一个允许 KeepAlive 的 agent，当你知道不在使用它的时候，最好关闭 agent。否则，socket 会一直保存打开状态，直到服务器关闭。


### agent.getName(options)

获取一组请求选项的唯一名，来确定某个连接是否可重用。在 http agent 里，它会返回 `host:port:localAddress`。在 http agent 里， name 包括 CA，cert, ciphers, 和其他 HTTPS/TLS 特殊选项来决定 socket 是否可以重用。


## http.globalAgent

Agent 的全局实例，是 http 客户端的默认请求。


## 类： http.ClientRequest

该对象在内部创建并从 `http.request()` 返回。他是 _正在_ 处理的请求，其头部已经在队列中。使用  `setHeader(name, value)`, `getHeader(name)`, `removeHeader(name)` API 可以改变header。当关闭连接的时候，header将会和第一个数据块一起发送。

为了获取响应，可以给请求对象的 `'response'` 添加监听器。当接收到响应头的时候将会从请求对象里触发`'response'`。`'response'` 事件执行时有一个参数，该参数为 [http.IncomingMessage][] 的实例。

在  `'response'` 事件期间，可以给响应对象添加监视器，监听 `'data'` 事件。

如果没有添加 `'response'` 处理函数，响应将被完全忽略。如果你添加了 `'response'` 事件处理函数，那你 **必须** 消费掉从响应对象获取的数据，可以在 `'readable'` 事件里调用 `response.read()` ，或者添加一个  `'data'` 处理函数，或者调用  `.resume()`  方法。如果未读取数据，它将会消耗内存，最终产生 `process out of memory` 错误。


Node 不会检查 Content-Length 和 body 的长度是否相同。

该请求实现了 [Writable Stream][] 接口。这是一个包含下列事件的 [EventEmitter][]。

### 事件： 'response'

`function (response) { }`

当接收到请求的时候会触发，仅会触发一次。 `response` 的参数是 [http.IncomingMessage][] 的实例。

Options:

- `host`: 要请求的服务器域名或 IP 地址
- `port`: 远程服务器的端口
- `socketPath`: Unix 域 Socket (使用 host:port 或 socketPath 之一)

### 事件： 'socket'

`function (socket) { }`

Socket 附加到这个请求的时候触发。

### 事件： 'connect'

`function (response, socket, head) { }`

每次服务器使用 CONNECT 方法响应一个请求时触发。如果这个这个事件未被监听，接收 CONNECT 方法的客户端将关闭他们的连接。

下面的例子展示了一对匹配的客户端/服务器如何监听 `connect` 事件。
    var http = require('http');
    var net = require('net');
    var url = require('url');

    // Create an HTTP tunneling proxy
    var proxy = http.createServer(function (req, res) {
      res.writeHead(200, {'Content-Type': 'text/plain'});
      res.end('okay');
    });
    proxy.on('connect', function(req, cltSocket, head) {
      // connect to an origin server
      var srvUrl = url.parse('http://' + req.url);
      var srvSocket = net.connect(srvUrl.port, srvUrl.hostname, function() {
        cltSocket.write('HTTP/1.1 200 Connection Established\r\n' +
                        'Proxy-agent: Node-Proxy\r\n' +
                        '\r\n');
        srvSocket.write(head);
        srvSocket.pipe(cltSocket);
        cltSocket.pipe(srvSocket);
      });
    });

    // now that proxy is running
    proxy.listen(1337, '127.0.0.1', function() {

      // make a request to a tunneling proxy
      var options = {
        port: 1337,
        hostname: '127.0.0.1',
        method: 'CONNECT',
        path: 'www.google.com:80'
      };

      var req = http.request(options);
      req.end();

      req.on('connect', function(res, socket, head) {
        console.log('got connected!');

        // make a request over an HTTP tunnel
        socket.write('GET / HTTP/1.1\r\n' +
                     'Host: www.google.com:80\r\n' +
                     'Connection: close\r\n' +
                     '\r\n');
        socket.on('data', function(chunk) {
          console.log(chunk.toString());
        });
        socket.on('end', function() {
          proxy.close();
        });
      });
    });

### 事件： 'upgrade'

`function (response, socket, head) { }`

每当服务器响应 upgrade 请求时触发。如果没有监听这个事件,客户端会收到 upgrade 头后关闭连接。

下面的例子展示了一对匹配的客户端/服务器如何监听 `upgrade` 事件。

    var http = require('http');

    // Create an HTTP server
    var srv = http.createServer(function (req, res) {
      res.writeHead(200, {'Content-Type': 'text/plain'});
      res.end('okay');
    });
    srv.on('upgrade', function(req, socket, head) {
      socket.write('HTTP/1.1 101 Web Socket Protocol Handshake\r\n' +
                   'Upgrade: WebSocket\r\n' +
                   'Connection: Upgrade\r\n' +
                   '\r\n');

      socket.pipe(socket); // echo back
    });

    // now that server is running
    srv.listen(1337, '127.0.0.1', function() {

      // make a request
      var options = {
        port: 1337,
        hostname: '127.0.0.1',
        headers: {
          'Connection': 'Upgrade',
          'Upgrade': 'websocket'
        }
      };

      var req = http.request(options);
      req.end();

      req.on('upgrade', function(res, socket, upgradeHead) {
        console.log('got upgraded!');
        socket.end();
        process.exit(0);
      });
    });

### 事件： 'continue'

`function () { }`

当服务器发送 '100 Continue'  HTTP 响应的时候触发，通常因为请求包含 'Expect: 100-continue'。该指令表示客户端应发送请求体。

### request.flushHeaders()

刷新请求的头。

考虑效率因素，Node.js 通常会缓存请求的头直到你调用 `request.end()`，或写入请求的第一个数据块。然后，包装请求的头和数据到一个独立的 TCP 包里。


### request.write(chunk[, encoding][, callback])

发送一个请求体的数据块。通过多次调用这个函数，用户能流式的发送请求给服务器，这种情况下，建议使用`['Transfer-Encoding', 'chunked']`  头。

`chunk` 参数必须是 [Buffer][] 或字符串。

回调参数可选，当这个数据块被刷新的时候会被调用。

### request.end([data][, encoding][, callback])

发送请求完毕。如果 body 的数据没被发送，将会将他们刷新到流里。如果请求是分块的，该方法会发送终结符0\r\n\r\n 。

如果指定了 `data`，等同于先调用 `request.write(data, encoding)`，再调用 `request.end(callback)`。

如果有 `callback`，将会在请求流结束的时候调用。

### request.abort()

终止一个请求. (v0.3.8 开始新加入)。

### request.setTimeout(timeout[, callback])

如果 socket 被分配给这个请求，并完成连接，将会调用 [socket.setTimeout()][] 。

### request.setNoDelay([noDelay])

如果 socket 被分配给这个请求，并完成连接，将会调用 [socket.setNoDelay()][]。

### request.setSocketKeepAlive([enable][, initialDelay])

如果 socket 被分配给这个请求，并完成连接，将会调用 [socket.setKeepAlive()][]。


## http.IncomingMessage

[http.Server][] 或 [http.ClientRequest][] 创建了 `IncomingMessage` 对象，作为第一个参数传递给 `'response'`。它可以用来访问应答的状态，头文件和数据。

它实现了 [Readable Stream][] 接口，以及以下额外的事件，方法和属性。

### 事件： 'close'

`function () { }`

表示底层连接已经关闭。
和 `'end'`类似，这个事件每个应答只会发送一次。

### message.httpVersion

客户端向服务器发送请求时，客户端发送的 HTTP 版本；或者服务器想客户端返回应答时，服务器的 HTTP 版本。通常是 `'1.1'` 或 `'1.0'` 。

另外， `response.httpVersionMajor` 是第一个整数，`response.httpVersionMinor` 是第二个整数。

### message.headers

请求/响应头对象。

只读的头名称和值的映射。头的名字是小写，比如：

    // Prints something like:
    //
    // { 'user-agent': 'curl/7.22.0',
    //   host: '127.0.0.1:8000',
    //   accept: '*/*' }
    console.log(request.headers);

### message.rawHeaders

接收到的请求/响应头字段列表。

注意，键和值在同一个列表中。它并非一个元组列表。所以，偶数偏移量为键，奇数偏移量为对应的值。

头名字不是小写敏感，也没用合并重复的头。
    // Prints something like:
    //
    // [ 'user-agent',
    //   'this is invalid because there can be only one',
    //   'User-Agent',
    //   'curl/7.22.0',
    //   'Host',
    //   '127.0.0.1:8000',
    //   'ACCEPT',
    //   '*/*' ]
    console.log(request.rawHeaders);

### message.trailers

请求/响应 的尾部对象。只在 'end' 事件中存在。

### message.rawTrailers

接收到的原始的请求/响应尾部键和值。仅在 'end' 事件中存在。

### message.setTimeout(msecs, callback)

* `msecs` {Number}
* `callback` {Function}

调用 `message.connection.setTimeout(msecs, callback)`.

### message.method

**仅对从 [http.Server][] 获得的请求有效。**

请求方法如果一个只读的字符串。
例如：`'GET'`, `'DELETE'`.

### message.url

**仅对从 [http.Server][] 获得的请求有效。**

请求的 URL 字符串。它仅包含实际的 HTTP 请求中所提供的 URL，比如请求如下：

    GET /status?name=ryan HTTP/1.1\r\n
    Accept: text/plain\r\n
    \r\n

`request.url` 就是:

    '/status?name=ryan'

如果你想将 URL 分解，可以用 `require('url').parse(request.url)`，例如：

    node> require('url').parse('/status?name=ryan')
    { href: '/status?name=ryan',
      search: '?name=ryan',
      query: 'name=ryan',
      pathname: '/status' }

如果想从查询字符串中解析出参数，可以用 `require('querystring').parse` 函数，或者将`true` 作为第二个参数传递给 `require('url').parse`。 例如：

    node> require('url').parse('/status?name=ryan', true)
    { href: '/status?name=ryan',
      search: '?name=ryan',
      query: { name: 'ryan' },
      pathname: '/status' }

### message.statusCode

**仅对从 `http.ClientRequest` 获取的响应有效。**

3位数的 HTTP 响应状态码 `404`。

### message.statusMessage

**仅对从 `http.ClientRequest` 获取的响应有效。**

HTTP 的响应消息。比如， `OK` 或 `Internal Server Error`.

### message.socket

和连接相关联的 `net.Socket` 对象。

通过 HTTPS 支持，使用 `request.connection.verifyPeer()` 和 `request.connection.getPeerCertificate()` 获取客户端的身份信息。


['checkContinue']: #http_event_checkcontinue
['listening']: net.html#net_event_listening
['response']: #http_event_response
[Agent]: #http_class_http_agent
[Buffer]: buffer.html#buffer_buffer
[EventEmitter]: events.html#events_class_events_eventemitter
[Readable Stream]: stream.html#stream_class_stream_readable
[Writable Stream]: stream.html#stream_class_stream_writable
[global Agent]: #http_http_globalagent
[http.ClientRequest]: #http_class_http_clientrequest
[http.IncomingMessage]: #http_http_incomingmessage
[http.ServerResponse]: #http_class_http_serverresponse
[http.Server]: #http_class_http_server
[http.request()]: #http_http_request_options_callback
[http.request()]: #http_http_request_options_callback
[net.Server.close()]: net.html#net_server_close_callback
[net.Server.listen(path)]: net.html#net_server_listen_path_callback
[net.Server.listen(port)]: net.html#net_server_listen_port_host_backlog_callback
[response.end()]: #http_response_end_data_encoding
[response.write()]: #http_response_write_chunk_encoding
[response.writeContinue()]: #http_response_writecontinue
[response.writeHead()]: #http_response_writehead_statuscode_reasonphrase_headers
[socket.setKeepAlive()]: net.html#net_socket_setkeepalive_enable_initialdelay
[socket.setNoDelay()]: net.html#net_socket_setnodelay_nodelay
[socket.setTimeout()]: net.html#net_socket_settimeout_timeout_callback
[stream.setEncoding()]: stream.html#stream_stream_setencoding_encoding
[url.parse()]: url.html#url_url_parse_urlstr_parsequerystring_slashesdenotehost
