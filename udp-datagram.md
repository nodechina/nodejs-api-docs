# UDP/Datagram Sockets

    稳定性: 3 - 稳定

<!-- name=dgram -->

调用 `require('dgram')` ，可以使用数据报文 sockets(Datagram sockets)。

重要提醒： `dgram.Socket#bind()` 的行为在 v0.10 做了改动 ，它总是异步的。如果你的代码像下面的一样：

    var s = dgram.createSocket('udp4');
    s.bind(1234);
    s.addMembership('224.0.0.114');

现在需要改为：

    var s = dgram.createSocket('udp4');
    s.bind(1234, function() {
      s.addMembership('224.0.0.114');
    });


## dgram.createSocket(type[, callback])

* `type` 字符串.  'udp4' 或 'udp6'
* `callback` 函数. 附加到 `message` 事件的监听器。可选参数。
* 返回: Socket 对象

创建指定类型的数据报文(datagram) Socket。有效类型是`udp4` 和 `udp6`

接受一个可选的回调，会被添加为 `message` 的监听事件。

如果你想接收数据报文(datagram)可以调用 `socket.bind()`。`socket.bind()` 将会绑定到所有接口（"all interfaces"）的随机端口上（ `udp4` 和 `udp6` sockets 都适用）。你可以通过`socket.address().address` 和 `socket.address().port` 获取地址和端口。

## dgram.createSocket(options[, callback])
* `options` 对象
* `callback` 函数. 给 `message` 事件添加事件监听器.
* 返回: Socket 对象

参数 `options` 必须包含 `type` 值(`udp4` 或 `udp6`),或可选的 boolean 值  `reuseAddr`。

当 `reuseAddr`  为 true 时， `socket.bind()` 将会重用地址，即使另一个进程已经绑定 socket。 `reuseAddr`  默认为 `false`。

回调函数为可选参数，作为 `message` 事件的监听器。

如果你想接受数据报文(datagram)，可以调用 `socket.bind()` 。`socket.bind()` 将会绑定到所有接口（ "all interfaces" ）地址的随机端口上（ `udp4` 和 `udp6` sockets 都适用）。你可以通过`socket.address().address` 和 `socket.address().port` 获取地址和端口。

## Class: dgram.Socket

报文数据 Socket 类封装了数据报文(datagram) 函数。必须通过 `dgram.createSocket(...)` 函数创建。

### Event: 'message'

* `msg` 缓存对象. 消息。
* `rinfo` 对象. 远程地址信息。

当 socket 上新的数据报文(datagram)可用的时候，会触发这个事件。`msg` 是一个缓存，`rinfo` 是一个包含发送者地址信息的对象

    socket.on('message', function(msg, rinfo) {
      console.log('Received %d bytes from %s:%d\n',
                  msg.length, rinfo.address, rinfo.port);
    });

### Event: 'listening'

当 socket 开始监听数据报文(datagram)时触发。在 UDP socket 创建时触发。

### Event: 'close'

当 socket 使用 `close()` 关闭时触发。在这个 socket 上不会触发新的消息事件。

### Event: 'error'

* `exception` Error 对象

当发生错误时触发。

### socket.send(buf, offset, length, port, address[, callback])

* `buf` 缓存对象 或 字符串.  要发送的消息。
* `offset` 整数. 消息在缓存中得偏移量。
* `length` 整数. 消息的比特数。
* `port` 整数. 端口的描述。
* `address` 字符串. 目标的主机名或 IP 地址。
* `callback` 函数. 当消息发送完毕的时候调用。可选。

对于 UDP socket，必须指定目标端口和地址。 `address` 参数可能是字符串，它会被 DNS 解析。

如果忽略地址或者地址是空字符串，将使用 `'0.0.0.0'` 或 `'::0'` 替代。依赖于网络配置，这些默认值有可能行也可能不行。

如果 socket 之前没被调用 `bind` 绑定，则它会被分配一个随机端口并绑定到所有接口（ "all interfaces" ）地址(`udp4` sockets 的`'0.0.0.0'` , `udp6` sockets 的`'::0'`)

回调函数可能用来检测 DNS 错误，或用来确定什么时候重用 `buf` 对象。注意，DNS 查询会导致发送tick延迟。通过回调函数能确认数据报文(datagram)是否已经发送的

考虑到多字节字符串情况，偏移量和长度是字节长度[byte length](buffer.html#buffer_class_method_buffer_bytelength_string_encoding)，而不是字符串长度。

下面的例子是在  `localhost` 上发送一个 UDP 包给随机端口：

    var dgram = require('dgram');
    var message = new Buffer("Some bytes");
    var client = dgram.createSocket("udp4");
    client.send(message, 0, message.length, 41234, "localhost", function(err) {
      client.close();
    });

**关于 UDP 数据报文(datagram) 尺寸**

`IPv4/v6` 数据报文(datagram)的最大长度依赖于`MTU` (_Maximum Transmission Unit_)和 `Payload Length` 的长度。


-  `Payload Length` 内容为 16 位宽，它意味着 Payload 的最大字节说不超过 64k，其中包括了头信息和数据（65,507 字节 = 65,535 − 8 字节 UDP 头 − 20 字节 IP 头）;对于环回接口（loopback interfaces）这是真的，但对于多数主机和网络来说不太现实。

-  `MTU` 能支持数据报文(datagram)的最大值（以目前链路层技术来说）。对于任何连接，`IPv4` 允许的最小值为 `68` 的 `MTU`，推荐值为 `576` （通常推荐作拨号应用的 `MTU`）,无论他们是完整接收还是碎片接收。

  对于 `IPv6`，`MTU` 的最小值为 `1280` 字节，最小碎片缓存大小为 `1500` 字节。16 字节实在是太小，所以目前链路层一般最小 `MTU` 大小为 `1500`。

  
我们不可能知道一个包可能进过的每个连接的MTU。通常发送一个超过接收端 MTU 大小的数据报文(datagram)会失效。（数据包会被悄悄的抛弃，不会通知发送端数据包没有到达接收端）。

### socket.bind(port[, address][, callback])

* `port` 整数
* `address` 字符串, 可选
* `callback` 没有参数的函数, 可选。绑定时会调用回调。

对于 UDP socket，在一个端口和可选地址上监听数据报文(datagram)。如果没有指定地点，系统将会参数监听所有的地址。绑定完毕后，会触发 "listening"  事件，并会调用传入的回调函数。指定监听事件和回调函数非常有用。

一个绑定了的数据报文 socket 会保持 node 进程运行来接收数据。

如果绑定失败，会产生错误事件。极少数情况（比如绑定一个关闭的 socket）。这个方法会抛出一个错误。

以下是 UDP 服务器监听端口 41234 的例子：

    var dgram = require("dgram");

    var server = dgram.createSocket("udp4");

    server.on("error", function (err) {
      console.log("server error:\n" + err.stack);
      server.close();
    });

    server.on("message", function (msg, rinfo) {
      console.log("server got: " + msg + " from " +
        rinfo.address + ":" + rinfo.port);
    });

    server.on("listening", function () {
      var address = server.address();
      console.log("server listening " +
          address.address + ":" + address.port);
    });

    server.bind(41234);
    // server listening 0.0.0.0:41234


### socket.bind(options[, callback])

* `options` {对象} - 必需. 有以下的属性:
  * `port` {Number} - 必需.
  * `address` {字符串} - 可选.
  * `exclusive` {Boolean} - 可选.
* `callback` {函数} - 可选.

`options` 的可选参数`port` 和 `address`，以及可选参数 `callback`，好像在调用 [socket.bind(port, \[address\], \[callback\])
](#dgram_socket_bind_port_address_callback)。

如果 `exclusive` 是  `false` （默认），集群进程将会使用相同的底层句柄，允许连接处理共享的任务。当`exclusive` 为  `true` 时，句柄不会共享，尝试共享端口也会失败。监听 `exclusive` 端口的例子如下：

    socket.bind({
      address: 'localhost',
      port: 8000,
      exclusive: true
    });


### socket.close()

关闭底层 socket 并且停止监听数据。

### socket.address()

返回一个包含套接字地址信息的对象。对于 UDP socket，这个对象会包含`address` , `family` 和 `port`。

### socket.setBroadcast(flag)

* `flag` Boolean

设置或清除 `SO_BROADCAST` socket 选项。设置这个选项后，UDP 包可能会发送给一个本地的接口广播地址。

### socket.setTTL(ttl)

* `ttl` 整数

设置 `IP_TTL` socket 选项。 TTL 表示生存时间（Time to Live），但是在这个上下文中它指的是报文允许通过的 IP 跃点数。各个转发报文的路由器或者网关都会递减 TTL。如果 TTL 被路由器递减为0，则它将不会被转发。改变 TTL 的值通常用于网络探测器或多播。

`setTTL()` 的参数为 1 到 255 的跃点数。多数系统默认值为 64.

### socket.setMulticastTTL(ttl)

* `ttl` 整数

设置 `IP_MULTICAST_TTL` socket 选项.   TTL 表示生存时间（Time to Live），但是在这个上下文中它指的是报文允许通过的 IP 跃点数。 各个转发报文的路由器或者网关都会递减 TTL。如果 TTL 被路由器递减为0，则它将不会被转发。改变 TTL 的值通常用于网络探测器或多播。

`setMulticastTTL()` 的参数为 1 到 255 的跃点数。多数系统默认值为 1.

### socket.setMulticastLoopback(flag)

* `flag` Boolean

设置或清空 `IP_MULTICAST_LOOP` socket 选项。设置完这个选项后，当该选项被设置时，组播报文也会被本地接口收到。

### socket.addMembership(multicastAddress[, multicastInterface])

* `multicastAddress` 字符串
* `multicastInterface` 字符串, 可选

告诉内核加入广播组，选项为 `IP_ADD_MEMBERSHIP` socket

如果没有指定 `multicastInterface`，操作系统会给所有可用的接口添加关系。

### socket.dropMembership(multicastAddress[, multicastInterface])

* `multicastAddress` 字符串
* `multicastInterface` 字符串, 可选

和 `addMembership` 相反 - 用 `IP_DROP_MEMBERSHIP` 选项告诉内核离开广播组 。
如果没有指定 `multicastInterface`，操作系统会移除所有可用的接口关系。


### socket.unref()

在 socket 上调用 `unref` 允许程序退出，如果这是在事件系统中唯一的活动 socket。如果 socket 已经  `unref`，再次调用 `unref` 将会无效。

### socket.ref()

和 `unref` 相反，如果这是唯一的 socket，在一个之前被 unref 了的 socket 上调用 ref 将不会让程序退出（缺省行为）。如果一个 socket 已经被 ref，则再次调用 ref 将会无效。
