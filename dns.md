# DNS

    稳定性: 3 - 稳定

调用 `require('dns')` 可以访问这个模块。

这个模块包含的函数属于2个不同的分类：

1）使用系统底层的特性，完成名字解析，这个过程不需要网络通讯，这个分类仅有一个函数： `dns.lookup`。**开发者在同一个系统里名字解析都是用 `dns.lookup`.**

下面的例子，解析 `www.google.com`.

    var dns = require('dns');

    dns.lookup('www.google.com', function onLookup(err, addresses, family) {
      console.log('addresses:', addresses);
    });

2）连接到 DNS 服务器进行名字解析，**始终**使用网络来进行域名查询。这个分类包含除了  `dns.lookup` 外的所有函数。这些函数不会和 `dns.lookup` 使用同一套配置文件。如果你不想使用系统底层的特性来进行名字解析，而想进行 DNS 查询的话，可以用这个分类的函数。

下面的例子，解析了`'www.google.com'`，并反向解析返回的 IP 地址。

    var dns = require('dns');

    dns.resolve4('www.google.com', function (err, addresses) {
      if (err) throw err;

      console.log('addresses: ' + JSON.stringify(addresses));

      addresses.forEach(function (a) {
        dns.reverse(a, function (err, hostnames) {
          if (err) {
            throw err;
          }

          console.log('reverse for ' + a + ': ' + JSON.stringify(hostnames));
        });
      });
    });


更多细节参考[Implementation considerations section](#dns_implementation_considerations)。

## dns.lookup(hostname[, options], callback)

将域名（比如 `'google.com'`）解析为第一条找到的记录 A （IPV4）或 AAAA(IPV6)。参数 `options`可以是一个对象或整数。如果没有提供 `options`，IP v4 和 v6 地址都可以。如果 `options` 是整数，则必须是 `4` 或 `6`。

`options` 参数可能是包含 `family` 和 `hints` 两个属性的对象。这两个属性都是可选的。如果提供了 `family`，则必须是 `4` 或 `6`，否则，IP v4 和 v6 地址都可以。如果提供了 `hints`，可以是一个或者多个 `getaddrinfo` 标志，若不提供，没有标志会传给 `getaddrinfo`。多个标志位可以通过或运算来整合。以下的例子展示如何使用 `options`。

```
{
  family: 4,
  hints: dns.ADDRCONFIG | dns.V4MAPPED
}
```

参见 [supported `getaddrinfo` flags](#dns_supported_getaddrinfo_flags) 查看更多的标志位。

回调函数包含参数 `(err, address, family)`。 `address`参数表示 IP v4 或 v6 地址。`family` 参数是4 或 6，表示 `address` 家族（不一定是之前传入 lookup 的值）。

出错时，参数 `err` 是 `Error` 对象，`err.code`是错误代码。请记住，`err.code`等于`'ENOENT'`，不仅可能是因为域名不存在，还有可能是是其他原因，比如没有可用文件描述符。

`dns.lookup` 不必和 DNS 协议有关系。它使用了操作系统的特性，能将名字和地址关联。

实现这些东西也许很简单，但是对于 Node.js 程序来说都重要，所以在使用前请花点时间阅读[Implementation considerations section](#dns_implementation_considerations)。

# dns.lookupService(address, port, callback)

使用 `getnameinfo` 解析传入的地址和端口为域名和服务。

这个回调函数的参数是 `(err, hostname, service)`。 `hostname` 和 `service` 都是字符串 (比如 `'localhost'` 和 `'http'`）。

出错时，参数`err` 是 `Error` 对象，`err.code`是错误代码。

## dns.resolve(hostname[, rrtype], callback)

将一个域名（如 'google.com'）解析为一个 rrtype 指定记录类型的数组。

有效的 rrtypes 值为:

 * `'A'` (IPV4 地址, 默认)
 * `'AAAA'` (IPV6 地址)
 * `'MX'` (邮件交换记录)
 * `'TXT'` (text 记录)
 * `'SRV'` (SRV 记录)
 * `'PTR'` (用来反向 IP 查找)
 * `'NS'` (域名服务器 记录)
 * `'CNAME'` (别名 记录)
 * `'SOA'` (授权记录的初始值)

回调参数为 `(err, addresses)`.  其中 `addresses` 中每一项的类型都取决于记录类型, 详见下文对应的查找方法。

出错时，参数`err` 是 `Error` 对象，`err.code`是错误代码。


## dns.resolve4(hostname, callback)

和 `dns.resolve()` 类似, 仅能查询 IPv4 (`A` 记录）。
`addresses` IPv4 地址数组 (比如，`['74.125.79.104', '74.125.79.105', '74.125.79.106']`）。

## dns.resolve6(hostname, callback)

和 `dns.resolve4()` 类似， 仅能查询 IPv4( `AAAA` 查询）。


## dns.resolveMx(hostname, callback)

和 `dns.resolve()` 类似, 仅能查询邮件交换(`MX` 记录)。

`addresses` 是 MX 记录数组, 每一个包含优先级和交换属性(比如， `[{'priority': 10, 'exchange': 'mx.example.com'},...]`）。

## dns.resolveTxt(hostname, callback)

和 `dns.resolve()` 类似, 仅能进行文本查询 (`TXT` 记录）。
`addresses` 是 2-d 文本记录数组。(比如，`[ ['v=spf1 ip4:0.0.0.0 ', '~all' ] ]`）。 每个子数组包含一条记录的 TXT 块。根据使用情况可以连接在一起，也可单独使用。
## dns.resolveSrv(hostname, callback)

和 `dns.resolve()` 类似, 仅能进行服务记录查询 (`SRV` 记录）。
`addresses` 是 `hostname`可用的 SRV 记录数组。 SRV 记录属性有优先级（priority），权重（weight）, 端口（port）, 和名字（name） (比如，`[{'priority': 10, 'weight': 5, 'port': 21223, 'name': 'service.example.com'}, ...]`）。

## dns.resolveSoa(hostname, callback)

和 `dns.resolve()` 类似, 仅能查询权威记录(`SOA` 记录）。

`addresses`是包含以下结构的对象:

```
{
  nsname: 'ns.example.com',
  hostmaster: 'root.example.com',
  serial: 2013101809,
  refresh: 10000,
  retry: 2400,
  expire: 604800,
  minttl: 3600
}
```

## dns.resolveNs(hostname, callback)

和 `dns.resolve()` 类似, 仅能进行域名服务器记录查询(`NS` 记录）。
`addresses` 是域名服务器记录数组（`hostname` 可以使用） (比如, `['ns1.example.com', 'ns2.example.com']`）。

## dns.resolveCname(hostname, callback)

和 `dns.resolve()` 类似, 仅能进行别名记录查询 (`CNAME`记录)。`addresses` 是对 `hostname` 可用的别名记录数组 (比如，, `['bar.example.com']`）。

## dns.reverse(ip, callback)

反向解析 IP 地址，返回指向该 IP 地址的域名数组。  

回调函数参数 `(err, hostnames)`。

出错时，参数`err` 是 `Error` 对象，`err.code`是错误代码。

## dns.getServers()

返回一个用于当前解析的 IP 地址的数组的字符串。
	
## dns.setServers(servers)

指定一组 IP 地址作为解析服务器。

如果你给地址指定了端口，端口会被忽略，因为底层库不支持。

传入无效参数，会抛出以下错误:

## Error codes
  
每个 DNS 查询都可能返回以下错误:

- `dns.NODATA`: DNS 服务器返回无数据应答。 
- `dns.FORMERR`: DNS 服务器声称查询格式错误。 
- `dns.SERVFAIL`: DNS 服务器返回一般失败。 
- `dns.NOTFOUND`: 没有找到域名。 
- `dns.NOTIMP`: DNS 服务器未实现请求的操作。 
- `dns.REFUSED`: DNS 服务器拒绝查询。 
- `dns.BADQUERY`: DNS 查询格式错误。 
- `dns.BADNAME`:  域名格式错误。 
- `dns.BADFAMILY`: 地址协议不支持。 
- `dns.BADRESP`: DNS 回复格式错误。 
- `dns.CONNREFUSED`: 无法连接到DNS 服务器。 
- `dns.TIMEOUT`: 连接DNS 服务器超时。 
- `dns.EOF`: 文件末端。 
- `dns.FILE`: 读文件错误。 
- `dns.NOMEM`: 内存溢出。 
- `dns.DESTRUCTION`: 通道被摧毁。 
- `dns.BADSTR`: 字符串格式错误。 
- `dns.BADFLAGS`: 非法标识符。 
- `dns.NONAME`: 所给主机不是数字。 
- `dns.BADHINTS`: 非法HINTS标识符。 
- `dns.NOTINITIALIZED`: c c-ares 库尚未初始化。 
- `dns.LOADIPHLPAPI`: 加载 iphlpapi.dll 出错。 
- `dns.ADDRGETNETWORKPARAMS`: 无法找到 GetNetworkParams 函数。 
- `dns.CANCELLED`: 取消 DNS 查询。 

## 支持的 getaddrinfo 标志

以下内容可作为 hints 标志传给 `dns.lookup`  

- `dns.ADDRCONFIG`: 返回当前系统支持的地址类型。例如，如果当前系统至少配置了一个 IPv4 地址，则返回 IPv4地址。
- `dns.V4MAPPED`: 如果指定了 IPv6 家族， 但是没有找到 IPv6 地址，将返回 IPv4 映射的 IPv6地址。

## Implementation considerations

虽然 `dns.lookup` 和 `dns.resolve*/dns.reverse`函数都能实现网络名和网络地址的关联，但是他们的行为不太一样。这些不同点虽然很巧妙，但是会对 Node.js 程序产生显著的影响。

### dns.lookup

`dns.lookup` 和绝大多数程序一样使用了相同的系统特性。例如，`dns.lookup` 和 `ping` 命令用相同的方法解析了一个指定的名字。多数类似 POSIX 的系统，`dns.lookup` 函数可以通过改变`nsswitch.conf(5)` 和/或 `resolv.conf(5)` 的设置调整。如果改变这些文件将会影响系统里的其他应用。


虽然，JavaScript 调用是异步的，它的实现是同步的调用 libuv 线程池里的`getaddrinfo(3)` 。因为 libuv 线程池固定大小，所以如果调用 `getaddrinfo(3)`  的时间太长，会使的池里的其他操作（比如文件操作）性能降低。为了降低这个风险，可以通过增加 'UV_THREADPOOL_SIZE' 的值，让它超过4，来调整libuv线程池大小，更多信息参见[the official libuv
documentation](http://docs.libuv.org/en/latest/threadpool.html）。

### dns.resolve, functions starting with dns.resolve and dns.reverse

这些函数的实现和`dns.lookup` 不大相同。他们不会用到 `getaddrinfo(3)`，而是始终进行网络查询。这些操作都是异步的，和libuv线程池无关。

因此，这些操作对于其他线程不会产生负面影响，这和 `dns.lookup` 不同。

它们不会用到 `dns.lookup` 的配置文件（例如 `/etc/hosts`_）。
