# 网络

    稳定性: 3 - 稳定

 `net` 模块提供了异步网络封装，它包含了创建服务器/客户端的方法（调用 streams）。可以通过调用  `require('net')` 包含这个模块。

## net.createServer([options][, connectionListener])

创建一个 TCP 服务器。参数  `connectionListener` 自动给 ['connection'][] 事件创建监听器。

`options` 包含有以下默认值:

    {
      allowHalfOpen: false,
      pauseOnConnect: false
    }

如果  `allowHalfOpen` =  `true`， 当另一端 socket 发送 FIN 包时 socket 不会自动发送 FIN 包。socket 变为不可读，但仍可写。你需要显式的调用 `end()` 方法。更多信息参见 ['end'][] 事件。

如果 `pauseOnConnect` =  `true`,当连接到来的时候相关联的 socket 将会暂停。它允许在初始进程不读取数据情况下，让连接在进程间传递。调用 `resume()` 从暂停的 socket 里读取数据。

下面是一个监听 8124 端口连接的应答服务器的例子：

    var net = require('net');
    var server = net.createServer(function(c) { //'connection' listener
      console.log('client connected');
      c.on('end', function() {
        console.log('client disconnected');
      });
      c.write('hello\r\n');
      c.pipe(c);
    });
    server.listen(8124, function() { //'listening' listener
      console.log('server bound');
    });

使用 `telnet` 来测试:

    telnet localhost 8124

要监听 socket t `/tmp/echo.sock`，仅需要改倒数第三行代码：

    server.listen('/tmp/echo.sock', function() { //'listening' listener

使用 `nc` 连接到一个 UNIX domain socket 服务器:

    nc -U /tmp/echo.sock必须

## net.connect(options[, connectionListener])
## net.createConnection(options[, connectionListener])

工厂方法，返回一个新的 ['net.Socket'](#net_class_net_socket)，并连接到指定的地址和端口。

当 socket 建立的时候，将会触发  ['connect'][] 事件。

和['net.Socket'](#net_class_net_socket)有相同的方法。

对于 TCP sockets，参数 `options` 因为下列参数的对象：

  - `port`: 客户端连接到 Port 的端口（必须）。

  - `host`: 客户端要连接到得主机。默认 `'localhost'`.

  - `localAddress`: 网络连接绑定的本地接口。

  - `localPort`: 网络连接绑定的本地端口。

  - `family` : IP 栈版本。默认 `4`.

对于本地域socket，参数  `options`因为下列参数的对象：

  - `path`: 客户端连接到得路径(必须).

通用选项:

  - 如果  `allowHalfOpen` =  `true`， 当另一端 socket 发送 FIN 包时 socket 不会自动发送 FIN 包。socket 变为不可读，但仍可写。你需要显式的调用 `end()` 方法。更多信息参见 ['end'][] 事件。

 `connectListener`  参数将会作为监听器添加到['connect'][]事件上。

下面是一个用上述方法应答服务器的客户端例子：

    var net = require('net');
    var client = net.connect({port: 8124},
        function() { //'connect' listener
      console.log('connected to server!');
      client.write('world!\r\n');
    });
    client.on('data', function(data) {
      console.log(data.toString());
      client.end();
    });
    client.on('end', function() {
      console.log('disconnected from server');
    });

要连接到 socket `/tmp/echo.sock`，仅需将第二行代码改为：

    var client = net.connect({path: '/tmp/echo.sock'});

## net.connect(port[, host][, connectListener])
## net.createConnection(port[, host][, connectListener])

创建一个到端口 `port` 和 主机 `host`的 TCP 连接。如果忽略主机 `host`，则假定为`'localhost'`。参数 `connectListener` 将会作为监听器添加到 ['connect'][] 事件。

这是工厂方法，返回一个新的 ['net.Socket'](#net_class_net_socket)。

## net.connect(path[, connectListener])
## net.createConnection(path[, connectListener])

创建到 `path` 的 unix socket 连接。
参数 `connectListener` 将会作为监听器添加到 ['connect'][] 事件上。

这是工厂方法，返回一个新的 ['net.Socket'](#net_class_net_socket)。

## Class: net.Server

这个类用来创建一个 TCP 或本地服务器。

### server.listen(port[, host][, backlog][, callback])

开始接受指定端口 `port` 和 主机 `host` 的连接。如果忽略主机 `host`， 服务器将会接受任何 IPv4 地址(`INADDR_ANY`)的直接连接。端口为 0，则会分配一个随机端口。

积压量（Backlog）为连接等待队列的最大长度。实际长度由您的操作系统通过 sysctl 设定，比如 linux 上的`tcp_max_syn_backlog` 和 `somaxconn`。这个参数默认值是 511 (不是 512)。  

这是异步函数。当服务器被绑定时会触发 ['listening'][] 事件。最后一个参数 `callback` 将会作为['listening'][] 事件的监听器。

有些用户会遇到 `EADDRINUSE` 错误，它表示另外一个服务器已经运行在所请求的端口上。处理这个情况的办法是等一段事件再重试：

    server.on('error', function (e) {
      if (e.code == 'EADDRINUSE') {
        console.log('Address in use, retrying...');
        setTimeout(function () {
          server.close();
          server.listen(PORT, HOST);
        }, 1000);
      }
    });

(注意: Node 中的所有 socket 已设置了 `SO_REUSEADDR` )


### server.listen(path[, callback])

* `path` {String}
* `callback` {Function}

启动一个本地 socket 服务器，监听指定 `path` 的连接。

这是异步函数。绑定服务器后，会触发 ['listening'][] 事件。最后一个参数 `callback` 将会作为['listening'][] 事件的监听器。


UNIX 上，本地域通常默认为 UNIX 域。参数 `path` 是文件系统路径，就和创建文件时一样，它也遵从命名规则和权限检查，并且在文件系统里可见，并持续到关闭关联。


Windows上，本地域通过命名管道实现。路径必须是以 `\\?\pipe\` 或 `\\.\pipe\` 入口。任意字符串都可以，不过之后进行相同的管道命名处理，比如解决 `..` 序列。管道命名空间是平的。管道不会一直持久，当最后一个引用关闭的时候，管道将会移除。不要忘记 javascript 字符字符串转义要求路径使用双反斜杠，比如：

    net.createServer().listen(
        path.join('\\\\?\\pipe', process.cwd(), 'myctl'))

### server.listen(handle[, callback])

* `handle` {Object}
* `callback` {Function}

 `handle` 对象可以设置成 server 或 socket（任意以下划线 `_handle` 开头的成员），或者是 `{fd: <n>}` 对象。

这将是服务器用指定的句柄接收连接，前提是文件描述符或句柄已经绑定到端口或域 socket。

Windows 不支持监听文件句柄。

这是异步函数。当服务器已经被绑定，将会触发['listening'][]  事件。最后一个参数 `callback` 将会作为 ['listening'][] 事件的监听器。

### server.listen(options[, callback])

* `options` {Object} - 必须. 支持以下属性:
  * `port` {Number} - 可选.
  * `host` {String} - 可选.
  * `backlog` {Number} - 可选.
  * `path` {String} - 可选.
  * `exclusive` {Boolean} - 可选.
* `callback` {Function} - 可选.

`options` 的属性：端口 `port`, 主机 `host`, 和 `backlog`, 以及可选参数 callback 函数, 他们在一起调用[server.listen(port, \[host\], \[backlog\], \[callback\])](#net_server_listen_port_host_backlog_callback)。还有，参数 `path` 可以用来指定 UNIX socket。

如果参数 `exclusive` 是`false`（默认值），集群进程将会使用同一个句柄，允许连接共享。当参数`exclusive` 是 `true` 时，句柄不会共享，如果共享端口会返回错误。监听独家端口例子如下：  

    server.listen({
      host: 'localhost',
      port: 80,
      exclusive: true
    });

### server.close([callback])

服务器停止接收新的连接，保持现有连接。这是异步函数，当所有连接结束的时候服务器会关闭，并会触发 `'close'` 事件。你可以传一个回调函数来监听 `'close'` 事件。如果存在，将会调用回调函数，错误（如果有）作为唯一参数。

### server.address()

操作系统返回绑定的地址，协议族名和服务器端口。查找哪个端口已经被系统绑定时，非常有用。返回的对象有3个属性，比如：
`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`

例如:

    var server = net.createServer(function (socket) {
      socket.end("goodbye\n");
    });

    // grab a random port.
    server.listen(function() {
      address = server.address();
      console.log("opened server on %j", address);
    });

在 `'listening'` 事件触发前，不要调用 `server.address()`。

### server.unref()

如果这是事件系统中唯一一个活动的服务器，调用 `unref` 将允许程序退出。如果服务器已被 unref，则再次调用 unref 并不会产生影响。

### server.ref()

与 `unref` 相反，如果这是唯一的服务器，在之前被 `unref` 了的服务器上调用 `ref` 将不会让程序退出（默认行为）。如果服务器已经被 `ref`，则再次调用 `ref` 并不会产生影响。

### server.maxConnections

设置这个选项后，当服务器连接数超过数量时拒绝新连接。

一旦已经用 `child_process.fork()` 方法将 socket 发送给子进程， 就不推荐使用这个选项。

### server.connections

已经抛弃这个函数。请用 [server.getConnections()][] 代替。服务器上当前连接的数量。

当调用 `child_process.fork()` 发送一个 socket 给子进程时，它将变为 `null` 。 要轮询子进程来获取当前活动连接的数量，请用 `server.getConnections` 代替.


### server.getConnections(callback)

异步获取服务器当前活跃连接的数量。当 socket 发送给子进程后才有效；  

回调函数有 2 个参数 `err` 和 `count`。

`net.Server` 是事件分发器 [EventEmitter][]， 有以下事件:

### 事件： 'listening'

当服务器调用 `server.listen` 绑定后会触发。

<a name="net_event_connection"></a>
### 事件：'connection'

* {Socket object} 连接对象

当新连接创建后会被触发。`socket` 是 `net.Socket`实例。

### 事件： 'close'

服务器关闭时会触发。注意，如果存在连接，这个事件不会被触发直到所有的连接关闭。

### 事件： 'error'

* {Error Object}

发生错误时触发。'close' 事件将被下列事件直接调用。请查看 `server.listen`  例子。

## Class: net.Socket

这个对象是 TCP 或 UNIX Socket 的抽象。`net.Socket` 实例实现了一个双工流接口。 他们可以在用户创建客户端(使用 connect())时使用, 或者由 Node 创建它们，并通过 `connection` 服务器事件传递给用户。

### new net.Socket([options])

构造一个新的 socket 对象。

`options` 对象有以下默认值:

    { fd: null
      allowHalfOpen: false,
      readable: false,
      writable: false
    }

参数 `fd` 允许你指定一个存在的文件描述符。将 `readable` 和（或） `writable` 设为 `true` ，允许在这个 socket 上读和（或）写（注意，仅在参数 `fd` 有效时）。关于 `allowHalfOpen`，参见`createServer()` 和 `'end'` 事件。

### socket.connect(port[, host][, connectListener])
### socket.connect(path[, connectListener])

使用传入的 socket 打开一个连接。如果指定了端口 `port` 和 主机 `host`，TCP socket 将打开 socket 。如果忽略参数 `host`，则默认为 `localhost`。如果指定了 `path` ，socket 将会被指定路径的 unix socket 打开。

通常情况不需要使用这个函数，比如使用 `net.createConnection` 打开 socket。只有你实现了自己的 socket 时才会用到。


这是异步函数。当 ['connect'][] 事件被触发时， socket 已经建立。如果这是问题连接， `'connect'` 事件不会被触发， 将会抛出 `'error'`事件。
  
参数 `connectListener` 将会作为监听器添加到 ['connect'][] 事件。


### socket.bufferSize

是 `net.Socket` 的一个属性，用于 `socket.write()` 。帮助用户获取更快的运行速度。计算机不能一直处于写入大量数据状态--网络连接可能太慢。Node 在内部会将排队数据写入到socket，并在网络可用时发送。（内部实现：轮询 socket 的文件描述符直到变为可写）。


这种内部缓冲的缺点是会增加内存使用量。这个属性表示当前准备写的缓冲字符数。（字符的数量等于准备写入的字节的数量，但是缓冲区可能包含字符串，这些字符串是惰性编码的，所以准确的字节数还无法知道）。

遇到很大增长很快的 `bufferSize` 时，用户可用尝试用`pause()` 和 `resume()`来控制字符流。


### socket.setEncoding([encoding])

设置 socket 的编码为可读流。更多信息参见[stream.setEncoding()][]

### socket.write(data[, encoding][, callback])

在 socket 上发送数据。第二个参数指定了字符串的编码，默认是 UTF8 编码。

如果所有数据成功刷新到内核缓冲区，返回  `true`。如果数据全部或部分在用户内存里，返回 `false` 。当缓冲区为空的时候会触发 `'drain'` 。

当数据最终被完整写入的的时候，可选的 `callback` 参数会被执行，但不一定会马上执行。

<a name="net_event_end"></a>
### socket.end([data][, encoding])

半关闭 socket。例如，它发送一个 FIN 包。可能服务器仍在发送数据。

如果参数 `data` 不为空，等同于调用 `socket.write(data, encoding)` 后再调用 `socket.end()`。

### socket.destroy()

确保没有 I/O 活动在这个套接字上。只有在错误发生情况下才需要。（处理错误等等）。  

### socket.pause()

暂停读取数据。就是说，不会再触发 `data` 事件。对于控制上传非常有用。

### socket.resume()

调用  `pause()` 后想恢复读取数据。

### socket.setTimeout(timeout[, callback])

socket 闲置时间超过 `timeout` 毫秒后 ，将 socket 设置为超时。

触发空闲超时事件时，socket 将会收到 `'timeout'`事件，但是连接不会被断开。用户必须手动调用 `end()` 或 `destroy()` 这个socket。

如果 `timeout` = 0, 那么现有的闲置超时会被禁用

可选的 callback 参数将会被添加成为 'timeout' 事件的一次性监听器。

### socket.setNoDelay([noDelay])

禁用纳格（Nagle）算法。默认情况下 TCP 连接使用纳格算法，在发送前他们会缓冲数据。将 `noDelay` 设置为 `true` 将会在调用 `socket.write()` 时立即发送数据。`noDelay` 默认值为 `true`。

### socket.setKeepAlive([enable][, initialDelay])

禁用/启用长连接功能，并在发送第一个在闲置 socket 上的长连接 probe 之前，可选地设定初始延时。默认为 false。

设定 `initialDelay` （毫秒），来设定收到的最后一个数据包和第一个长连接probe之间的延时。将 initialDelay 设为0，将会保留默认（或者之前）的值。默认值为0.

### socket.address()
  
操作系统返回绑定的地址，协议族名和服务器端口。返回的对象有 3 个属性，比如`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`。

### socket.unref()

如果这是事件系统中唯一一个活动的服务器，调用 `unref` 将允许程序退出。如果服务器已被 unref，则再次调用 unref 并不会产生影响。

### socket.ref()

与 `unref` 相反，如果这是唯一的服务器，在之前被 `unref` 了的服务器上调用 `ref` 将不会让程序退出（默认行为）。如果服务器已经被 `ref`，则再次调用 `ref` 并不会产生影响。

### socket.remoteAddress

远程的 IP 地址字符串，例如：`'74.125.127.100'` or `'2001:4860:a005::68'`.

### socket.remoteFamily

远程IP协议族字符串，比如 `'IPv4'` or `'IPv6'`.

### socket.remotePort
远程端口，数字表示，例如：`80` or `21`.

### socket.localAddress
网络连接绑定的本地接口
远程客户端正在连接的本地 IP 地址，字符串表示。例如，如果你在监听'0.0.0.0'而客户端连接在'192.168.1.1'，这个值就会是 '192.168.1.1'。

### socket.localPort

本地端口地址，数字表示。例如：`80` or `21`.

### socket.bytesRead

接收到得字节数。

### socket.bytesWritten

发送的字节数。

`net.Socket` 是事件分发器 [EventEmitter][]的实例， 有以下事件:

### 事件： 'lookup'
在解析域名后，但在连接前，触发这个事件。对 UNIX sokcet 不适用。

* `err` {Error | Null} 错误对象。  参见 [dns.lookup()][].
* `address` {String} IP 地址。
* `family` {String | Null} 地址类型。  参见 [dns.lookup()][].

### 事件： 'connect'

当成功建立 socket 连接时触发。
参见 `connect()`.

### 事件： 'data'

* {Buffer object}

当接收到数据时触发。参数 `data` 可以是  `Buffer` 或 `String`。使用 `socket.setEncoding()`设定数据编码。(更多信息参见  [Readable Stream][] )。

当  `Socket` 触发一个 `'data'` 事件时，如果没有监听器，数据将会丢失。

### 事件： 'end'

当 socket 另一端发送 FIN 包时，触发该事件。

默认情况下(`allowHalfOpen == false`)，一旦 socket 将队列里的数据写完毕，socket 将会销毁它的文件描述符。如果 `allowHalfOpen == true`,socket 不会从它这边自动调用 end()，使的用户可以随意写入数据，而让用户端自己调用 end()。


### 事件： 'timeout'

当 socket 空闲超时时触发，仅是表明 socket 已经空闲。用户必须手动关闭连接。

参见 : `socket.setTimeout()`


### 事件： 'drain'

当写缓存为空得时候触发。可用来控制上传。

参见 : `socket.write()` 的返回值。

### 事件： 'error'

* {Error object}

错误发生时触发。以下事件将会直接触发 `'close'` 事件。

### 事件： 'close'

* `had_error` {Boolean} 如果 socket 传输错误，为 `true`

当 socket 完全关闭时触发。参数 `had_error` 是 boolean，它表示是否因为传输错误导致 socket 关闭。

## net.isIP(input)

测试是否输入的为 IP 地址。字符串无效时返回 0。 IPV4 情况下返回 4， IPV6情况下返回 6.


## net.isIPv4(input)

如果输入的地址为 IPV4， 返回 true，否则返回 false。


## net.isIPv6(input)
  
如果输入的地址为 IPV6， 返回 true，否则返回 false。


['connect']: #net_event_connect
['connection']: #net_event_connection
['end']: #net_event_end
[EventEmitter]: events.html#events_class_events_eventemitter
['listening']: #net_event_listening
[server.getConnections()]: #net_server_getconnections_callback
[Readable Stream]: stream.html#stream_class_stream_readable
[stream.setEncoding()]: stream.html#stream_stream_setencoding_encoding
[dns.lookup()]: dns.html#dns_dns_lookup_domain_family_callback
