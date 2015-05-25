# TLS/SSL

    Stability: 3 - Stable

可以使用 `require('tls')` 来访问这个模块。


`tls` 模块 使用 OpenSSL 来提供传输层（Transport Layer）安全性和（或）安全套接层（Secure Socket Layer）：加密过的流通讯。

TLS/SSL 是一种公钥/私钥基础架构。每个客户端和服务端都需要一个私钥。私钥可以用以下方法创建的：

    openssl genrsa -out ryans-key.pem 2048

所有服务器和某些客户端需要证书。证书由认证中心（Certificate Authority）签名，或者自签名。获得证书第一步是创建一个证书签名请求 "Certificate Signing Request" (CSR)文件。证书可以用以下方法创建的：

    openssl req -new -sha256 -key ryans-key.pem -out ryans-csr.pem

使用CSR创建一个自签名的证书：

    openssl x509 -req -in ryans-csr.pem -signkey ryans-key.pem -out ryans-cert.pem

或者你可以发送 CSR 给认证中心（Certificate Authority）来签名。

(TODO: 创建 CA 的文档, 感兴趣的读者可以在 Node 源码 `test/fixtures/keys/Makefile` 里查看)

创建 .pfx 或 .p12，可以这么做：

    openssl pkcs12 -export -in agent5-cert.pem -inkey agent5-key.pem \
        -certfile ca-cert.pem -out agent5.pfx

  - `in`:  certificate
  - `inkey`: private key
  - `certfile`: all CA certs concatenated in one file like
    `cat ca1-cert.pem ca2-cert.pem > ca-cert.pem`

## 协议支持

Node.js 默认遵循 SSLv2 和 SSLv3 协议，不过这些协议被禁用。因为他们不太可靠，很容易受到威胁，参见 [CVE-2014-3566][]。某些情况下，旧版本客户端/服务器（比如 IE6）可能会产生问题。如果你想启用 SSLv2 或 SSLv3 ，使用参数`--enable-ssl2` 或 `--enable-ssl3` 运行 Node。Node.js 的未来版本中不会再默认编译 SSLv2 和 SSLv3。

有一个办法可以强制 node 进入仅使用  SSLv3 或 SSLv2 模式，分别指定`secureProtocol` 为 `'SSLv3_method'` 或 `'SSLv2_method'`。

Node.js 使用的默认协议方法准确名字是 `AutoNegotiate_method`， 这个方法会尝试并协商客户端支持的从高到底协议。为了提供默认的安全级别，Node.js(v0.10.33 版本之后)通过将 `secureOptions` 设为`SSL_OP_NO_SSLv3|SSL_OP_NO_SSLv2` ，明确的禁用了 SSLv3 和 SSLv2（除非你给`secureProtocol` 传值`--enable-ssl3`, 或 `--enable-ssl2`, 或 `SSLv3_method` ）。

如果你设置了 `secureOptions`，我们不会重新这个参数。


改变这个行为的后果：

 * 如果你的应用被当做为安全服务器，`SSLv3` 客户端不能协商建立连接，会被拒绝。这种情况下，你的服务器会触发 `clientError` 事件。错误消息会包含错误版本数字( `'wrong version number'`).
 * 如果你的应用被当做安全客户端，和一个不支持比 SSLv3 更高安全性的方法的服务器通讯，你的连接不会协商成功。这种情况下，你的客户端会触发 `clientError` 事件。错误消息会包含错误版本数字( `'wrong version number'`).

## Client-initiated renegotiation attack mitigation

<!-- type=misc -->

TLS 协议让客户端协商 TLS 会话的某些方法内容。但是，会话协商需要服务器端响应的资源，这回让它成为阻断服务攻击（denial-of-service attacks）的潜在媒介。

为了降低这种情况的发生，重新协商被限制为每10分钟3次。当超出这个界限时，在 [tls.TLSSocket][] 实例上会触发错误。这个限制可设置：

  - `tls.CLIENT_RENEG_LIMIT`: 重新协商 limit, 默认是 3.

  - `tls.CLIENT_RENEG_WINDOW`: 重新协商窗口的时间，单位秒, 默认是 10 分钟.

除非你明确知道自己在干什么，否则不要改变默认值。


要测试你的服务器的话，使用`openssl s_client -connect address:port` 连接服务器，并敲 `R<CR>`（字母 R 键加回车）几次。


## NPN and SNI

<!-- type=misc -->

NPN (Next Protocol Negotiation 下次协议协商) 和 SNI (Server Name Indication 域名指示) 都是 TLS 握手扩展，运行你：

  * NPN - 同一个TLS服务器使用多种协议 (HTTP, SPDY)
  * SNI - 同一个TLS服务器使用多个主机名（不同的 SSL 证书）。
    certificates.


## 完全正向保密

<!-- type=misc -->
"[Forward Secrecy]" 或 "Perfect Forward Secrecy-完全正向保密" 协议描述了秘钥协商（比如秘钥交换）方法的特点。实际上这意味着及时你的服务器的秘钥有危险，通讯仅有可能被一类人窃听，他们必须设法获的每次会话都会生成的秘钥对。

完全正向保密是通过每次握手时为秘钥协商随机生成密钥对来完成（和所有会话一个 key 相反）。实现这个技术（提供完全正向保密-Perfect Forward Secrecy）的方法被称为 "ephemeral"。

通常目前有2个方法用于完成完全正向保密（Perfect Forward Secrecy）:

  * [DHE] - 一个迪菲-赫尔曼密钥交换密钥协议（Diffie Hellman key-agreement protocol）短暂（ephemeral）版本。
  * [ECDHE] - 一个椭圆曲线密钥交换密钥协议（ Elliptic Curve Diffie Hellman key-agreement protocol）短暂（ephemeral）版本。

短暂（ephemeral）方法有性能缺点，因为生成 key 非常耗费资源。


## tls.getCiphers()

返回支持的 SSL 密码名数组。

例子：

    var ciphers = tls.getCiphers();
    console.log(ciphers); // ['AES128-SHA', 'AES256-SHA', ...]

<a name="tls_tls_createserver_options_secureconnectionlistener"></a>
## tls.createServer(options[, secureConnectionListener])

创建一个新的 [tls.Server][]。参数 `connectionListener` 会自动设置为 [secureConnection][]  事件的监听器。参数 `options` 对象有以下可能性：

  - `pfx`: 包含私钥，证书和服务器的 CA 证书（PFX 或 PKCS12 格式）字符串或缓存`Buffer`。（`key`, `cert` 和 `ca`互斥）。

  - `key`: 包含服务器私钥（PEM 格式）字符串或缓存`Buffer`。（可以是keys的数组）（必传）。

  - `passphrase`: 私钥或 pfx 的密码字符串  

  - `cert`: 包含服务器证书key（PEM 格式）字符串或缓存`Buffer`。（可以是certs的数组）（必传）。

  - `ca`: 信任的证书（PEM 格式）的字符串/缓存数组。如果忽略这个参数，将会使用"root" CAs ，比如 VeriSign。用来授权连接。

  - `crl` : 不是 PEM 编码 CRLs （证书撤销列表 Certificate Revocation List）的字符串就是字符串列表.

  - `ciphers`: 要使用或排除的密码（cipher）字符串

    为了减轻[BEAST attacks] ，推荐使用这个参数和之后会提到的 `honorCipherOrder` 参数来优化non-CBC 密码（cipher）

    默认：`ECDHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA256:AES128-GCM-SHA256:RC4:HIGH:!MD5:!aNULL`.
    格式上更多细节参见 [OpenSSL cipher list format documentation]

    `ECDHE-RSA-AES128-SHA256`, `DHE-RSA-AES128-SHA256` 和 `AES128-GCM-SHA256` 都是 TLS v1.2 密码（cipher），当 node.js 连接 OpenSSL 1.0.1 或更早版本（比如）时使用。注意， `honorCipherOrder` 设置为 enabled 后，现在任然可以和 TLS v1.2 客户端协商弱密码（cipher），

    `RC4` 可作为客户端和老版本 TLS 协议通讯的备用方法。`RC4` 这些年受到怀疑，任何对信任敏感的对象都会考虑其威胁性。国家级别（state-level）的参与者拥有中断它的能力。

    **注意**: 早些版本的修订建议， `AES256-SHA` 作为可以接受的密码（cipher）.Unfortunately, `AES256-SHA` 是一个 CBC 密码（cipher），容易受到 [BEAST attacks] 攻击。 *不要* 使用它。  

  - `ecdhCurve`: 包含用来 ECDH 秘钥交换弧形（curve）名字符串，或者 false 禁用 ECDH。

    默认 `prime256v1`. 更多细节参考 [RFC 4492] 。

  - `dhparam`: DH 参数文件，用于 DHE 秘钥协商。使用 `openssl dhparam` 命令来创建。如果加载文件失败，会悄悄的抛弃它。

  - `handshakeTimeout`: 如果 SSL/TLS 握手事件超过这个参数，会放弃里连接。 默认是 120 秒.

    握手超时后， `tls.Server` 对象会触发 `'clientError`' 事件。

  - `honorCipherOrder` : 当选择一个密码（cipher） 时, 使用服务器配置，而不是客户端的。  

    虽然这个参数默认不可用，还是推荐你用这个参数，和  `ciphers` 参数连接使用，减轻 BEAST 攻击。

    注意，如果使用了 SSLv2，服务器会发送自己的配置列表给客户端，客户端会挑选密码（cipher）。默认不支持 SSLv2，除非 node.js 配置了`./configure --with-sslv2`。

  - `requestCert`: 如果设为 `true`，服务器会要求连接的客户端发送证书，并尝试验证证书。默认：`false`。

  - `rejectUnauthorized`: 如果为 `true` ，服务器将会拒绝任何不被 CAs 列表授权的连接。仅  `requestCert` 参数为  `true` 时这个参数才有效。默认： `false`。

  - `checkServerIdentity(servername, cert)`: 提供一个重写的方法来检查证书对应的主机名。如果验证失败，返回 error。如果验证通过，返回 `undefined` 。

  - `NPNProtocols`: NPN 协议的 `Buffer` 数组（协议需按优先级排序）。

  - `SNICallback(servername, cb)`: 如果客户端支持 SNI TLS 扩展会调用这个函数。会传入2个参数： `servername` 和 `cb`。 `SNICallback` 必须调用 `cb(null, ctx)` ，其中 `ctx` 是 SecureContext 实例。（你可以用 `tls.createSecureContext(...)`  来获取相应的 SecureContext 上下文）。如果  `SNICallback` 没有提供，将会使用高级的 API（参见下文）.

  - `sessionTimeout`: 整数，设定了服务器创建TLS 会话标示符（TLS session identifiers）和 TLS 会话票据（TLS session tickets）后的超时时间（单位：秒）。更多细节参见：[SSL_CTX_set_timeout]。

  - `ticketKeys`: 一个 48 字节的 `Buffer` 实例，由 16 字节的前缀，16 字节的 hmac key，16 字节的 AES key 组成。可用用它来接受 tls 服务器实例上的 tls会话票据（tls session tickets）。

    注意: 自动在集群模块（ `cluster` module）工作进程间共享。

  - `sessionIdContext`: 会话恢复（session resumption）的标示符字符串。如果 `requestCert` 为  `true`。默认值为命令行生成的 MD5 哈希值。否则不提供默认值。

  - `secureProtocol`: SSL 使用的方法，例如，`SSLv3_method` 强制 SSL 版本为3。可能的值定义于你所安装的 OpenSSL 中的常量[SSL_METHODS][]。

  - `secureOptions`: 设置服务器配置。例如设置 `SSL_OP_NO_SSLv3` 可用禁用 SSLv3 协议。所有可用的参数见[SSL_CTX_set_options]

响应服务器的简单例子：

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      key: fs.readFileSync('server-key.pem'),
      cert: fs.readFileSync('server-cert.pem'),

      // This is necessary only if using the client certificate authentication.
      requestCert: true,

      // This is necessary only if the client uses the self-signed certificate.
      ca: [ fs.readFileSync('client-cert.pem') ]
    };

    var server = tls.createServer(options, function(socket) {
      console.log('server connected',
                  socket.authorized ? 'authorized' : 'unauthorized');
      socket.write("welcome!\n");
      socket.setEncoding('utf8');
      socket.pipe(socket);
    });
    server.listen(8000, function() {
      console.log('server bound');
    });

或

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      pfx: fs.readFileSync('server.pfx'),

      // This is necessary only if using the client certificate authentication.
      requestCert: true,

    };

    var server = tls.createServer(options, function(socket) {
      console.log('server connected',
                  socket.authorized ? 'authorized' : 'unauthorized');
      socket.write("welcome!\n");
      socket.setEncoding('utf8');
      socket.pipe(socket);
    });
    server.listen(8000, function() {
      console.log('server bound');
    });

你可以通过 `openssl s_client` 连接服务器来测试：


    openssl s_client -connect 127.0.0.1:8000

<a name="tls_tls_connect_options_callback"></a>
## tls.connect(options[, callback])

## tls.connect(port[, host][, options][, callback])

创建一个新的客户端连接到指定的端口和主机（`port` and `host`）（老版本 API），或者`options.port` 和 `options.host` （如果忽略 `host`，默认为 `localhost`）。`options` 是一个包含以下值得对象：

  - `host`: 客户端需要连接到的主机

  - `port`: 客户端需要连接到的端口

  - `socket`: 在指定的 socket （而非新建）上建立安全连接。如果这个参数有值，将忽略 `host` 和 `port` 参数。

  - `path`: 创建 到参数`path` 的 unix socket 连接。如果这个参数有值，将忽略 `host` 和 `port` 参数。

  - `pfx`: 包含私钥，证书和客户端（ PFX 或 PKCS12 格式）的 CA 证书的字符串或 `Buffer` 缓存。

  - `key`: 包含客户端（ PEM 格式）的 私钥的字符串或 `Buffer` 缓存。可以是 keys 数组。

  - `passphrase`: 私钥或 pfx 的密码字符串。

  - `cert`: 包含客户端证书key（PEM 格式）字符串或缓存`Buffer`。（可以是certs的数组）。  

  - `ca`: 信任的证书（PEM 格式）的字符串/缓存数组。如果忽略这个参数，将会使用"root" CAs ，比如 VeriSign。用来授权连接。


  - `rejectUnauthorized`: 如果为 `true` ，服务器证书根据 CAs 列表授权列表验证。如果验证失败，触发  `'error'`  事件；`err.code` 包含 OpenSSL 错误代码。默认： `true`。

  - `NPNProtocols`: NPN 协议的字符串或`Buffer` 数组。`Buffer` 必须有以下格式`0x05hello0x05world`，第一个字节是下一个协议名字的长度。（传的数组通常非常简单，比如： `['hello', 'world']`）。

  - `servername`: SNI（域名指示 Server Name Indication） TLS 扩展的服务器名。

  - `secureProtocol`: SSL 使用的方法，例如，`SSLv3_method` 强制 SSL 版本为3。可能的值定义于你所安装的 OpenSSL 中的常量[SSL_METHODS][]。

  - `session`: 一个 `Buffer` 实例, 包含 TLS 会话.


参数 `callback` 添加到 ['secureConnect'][] 事件上，其效果如同监听器。

`tls.connect()` 返回一个 [tls.TLSSocket][] 对象。


这是一个简单的客户端应答服务器例子：

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      // These are necessary only if using the client certificate authentication
      key: fs.readFileSync('client-key.pem'),
      cert: fs.readFileSync('client-cert.pem'),

      // This is necessary only if the server uses the self-signed certificate
      ca: [ fs.readFileSync('server-cert.pem') ]
    };

    var socket = tls.connect(8000, options, function() {
      console.log('client connected',
                  socket.authorized ? 'authorized' : 'unauthorized');
      process.stdin.pipe(socket);
      process.stdin.resume();
    });
    socket.setEncoding('utf8');
    socket.on('data', function(data) {
      console.log(data);
    });
    socket.on('end', function() {
      server.close();
    });

或

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      pfx: fs.readFileSync('client.pfx')
    };

    var socket = tls.connect(8000, options, function() {
      console.log('client connected',
                  socket.authorized ? 'authorized' : 'unauthorized');
      process.stdin.pipe(socket);
      process.stdin.resume();
    });
    socket.setEncoding('utf8');
    socket.on('data', function(data) {
      console.log(data);
    });
    socket.on('end', function() {
      server.close();
    });

## 类: tls.TLSSocket

[net.Socket][] 实例的封装，取代内部 socket 读写程序，执行透明的输入/输出数据的加密/解密。

## new tls.TLSSocket(socket, options)

从现有的 TCP socket 里构造一个新的 TLSSocket 对象。

`socket` 一个 [net.Socket][] 的实例

`options` 一个包含以下属性的对象:

  - `secureContext`: 来自 `tls.createSecureContext( ... )` 的可选 TLS 上下文对象。

  - `isServer`: 如果为 true， TLS socket 将会在服务器模式(server-mode)初始化。  

  - `server`: 一个可选的 [net.Server][] 实例

  - `requestCert`: 可选, 参见 [tls.createSecurePair][]

  - `rejectUnauthorized`: 可选, 参见 [tls.createSecurePair][]

  - `NPNProtocols`: 可选, 参见 [tls.createServer][]

  - `SNICallback`: 可选, 参见 [tls.createServer][]

  - `session`: 可选, 一个 `Buffer` 实例, 包含 TLS 会话

  - `requestOCSP`: 可选, 如果为 `true` - OCSP 状态请求扩展将会被添加到客户端 hello，并且  `OCSPResponse` 事件将会在 socket 上建立安全通讯前触发。


## tls.createSecureContext(details)

创建一个凭证（credentials）对象，包含字典有以下的key：

* `pfx` : 包含 PFX 或 PKCS12 加密的私钥，证书和服务器的 CA 证书（PFX 或 PKCS12 格式）字符串或缓存`Buffer`。（`key`, `cert` 和 `ca`互斥）。
* `key` : 包含 PEM 加密过的私钥的字符串。
* `passphrase` : 私钥或 pfx 的密码字符串。  
* `cert` : 包含 PEM 加密过的证书的字符串。
* `ca` : 信任的 PEM 加密过的可信任的证书（PEM 格式）字符串/缓存数组。  
* `crl` :PEM 加密过的 CRLs（证书撤销列表）
* `ciphers`: 要使用或排除的密码（cipher）字符串。更多格式上的细节参见 <http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT>
* `honorCipherOrder` : 当选择一个密码（cipher） 时, 使用服务器配置，而不是客户端的。  更多细节参见 `tls` 模块文档。

如果没给 'ca' 细节，node.js 将会使用默认的公开信任的 CAs 列表（参见<http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt>）。


## tls.createSecurePair([context][, isServer][, requestCert][, rejectUnauthorized])

创建一个新的安全对（secure pair）对象，包含2个流，其中一个读/写加密过的数据，另外一个读/写明文数据。通常加密端数据来自是从输入的加密数据流，另一端被当做初始加密流。

 - `credentials`: 来自 tls.createSecureContext( ... ) 的安全上下文对象。

 - `isServer`: 是否以服务器/客户端模式打开这个 tls 连接。

 - `requestCert`: 是否服务器需要连接的客户端发送证书。仅适用于服务端连接。  

 - `rejectUnauthorized`:非法证书时，是否服务器需要自动拒绝客户端。启用 `requestCert`后，才适用于服务器。

`tls.createSecurePair()` 返回一个安全对（SecurePair）对象，包含明文 `cleartext` 和 密文 `encrypted` 流 。

注意: `cleartext` 和 [tls.TLSSocket][] 拥有相同的 API。

## 类: SecurePair

通过 tls.createSecurePair 返回。

### 事件: 'secure'

一旦安全对（SecurePair）成功建立一个安全连接，安全对（SecurePair）将会触发这个事件。

和检查服务器 'secureConnection' 事件一样，pair.cleartext.authorized 必须检查确认是否适用的证书是授权过的。  

## 类: tls.Server

这是 `net.Server`  的子类，拥有相同的方法。这个类接受适用 TLS 或 SSL 的加密连接，而不是接受原始 TCP 连接。

### 事件: 'secureConnection'

`function (tlsSocket) {}`

新的连接握手成功后回触发这个事件。参数是 [tls.TLSSocket][] 实例。它拥有常用的流方法和事件。

`socket.authorized` 是否客户端被证书（服务器提供）授权。如果 `socket.authorized` 为 false，`socket.authorizationError` 是如何授权失败。值得一提的是，依赖于 TLS 服务器的设置，你的未授权连接可能也会被接受。
`socket.authorizationError` 如何授权失败。值得一提的是，依赖于 TLS 服务器的设置，你的未授权连接可能也会被接受。
`socket.npnProtocol` 包含选择的 NPN 协议的字符串.
`socket.servername` 包含 SNI 请求的服务器名的字符串。


### 事件: 'clientError'

`function (exception, tlsSocket) { }`

在安全连接建立前，客户端连接触发 'error' 事件会转发到这里来。

`tlsSocket` 是 [tls.TLSSocket][]，错误是从这里触发的。


### 事件: 'newSession'

`function (sessionId, sessionData, callback) { }`
  
创建 TLS 会话的时候会触发。可能用来在外部存储器里存储会话。`callback` 必须最后调用，否则没法从安全连接发送/接收数据。


注意： 添加这个事件监听器仅会在连接连接时有效果。


### 事件: 'resumeSession'

`function (sessionId, callback) { }`

当客户端想恢复之前的 TLS 会话时会触发。事件监听器可能会使用 `sessionId` 到外部存储器里查找，一旦结束会触发 `callback(null, sessionData)`。如果会话不能恢复（比如不存在这个会话），可能会调用 `callback(null, null)`。调用 `callback(err)` 将会终止连接，并销毁 socket。

注意： 添加这个事件监听器仅会在连接连接时有效果。


### 事件: 'OCSPRequest'

`function (certificate, issuer, callback) { }`

当客户端发送证书状态请求时会触发。你可以解析服务器当前的证书，来获取 OCSP 网址和证书 id，获取 OCSP 响应调用 `callback(null, resp)`，其中  `resp`  是 `Buffer`  实例。 证书（`certificate`）和发行者（`issuer`） 都是初级表达式缓存（`Buffer` DER-representations of the primary）和证书的发行者。它可以用来获取 OCSP 证书和 OCSP 终点网址。


可以调用`callback(null, null)`，表示没有 OCSP 响应。

调用 `callback(err)` 可能会导致调用 `socket.destroy(err)`。

典型流程:

1. 客户端连接服务器，并发送 `OCSPRequest`(通过 ClientHello 里的状态信息扩展)。
2. 服务器收到请求，调用 `OCSPRequest` 事件监听器。
3. 服务器从 `certificate` 或 `issuer` 获取 OCSP 网址，并执行 [OCSP request]  到 CA
4. 服务器 CA 收到 `OCSPResponse`， 并通过 `callback` 参数送回到客户端
5. 客户端验证响应，并销毁 socket 或 执行握手

注意: 如果证书是自签名的，或者如果发行者不再根证书列表里（你可以通过参数提供一个发行者）。 `issuer` 就可能为 null。  

注意： 添加这个事件监听器仅会在连接连接时有效果。

注意： 你可以能想要使用 npm 模块（比如 [asn1.js]）来解析证书。

### server.listen(port[, host][, callback])

在指定的端口和主机上开始接收连接。如果 `host` 参数没传，服务接受通过 IPv4 地址(`INADDR_ANY`)的直连。

这是异步函数。当服务器已经绑定后回调用最后一个参数`callback` 。

更多信息参见 `net.Server` 。


### server.close()

停止服务器，不再接收新连接。这是异步函数，当服务器触发 `'close'` 事件后回最终关闭。

### server.address()

返回绑定的地址，地址家族名和服务器端口。更多信息参见 [net.Server.address()][]

### server.addContext(hostname, context)

如果客户端请求 SNI 主机名和传入的 `hostname` 相匹配，将会用到安全上下文（secure context）。 `context`  可以包含`key`, `cert`, `ca` 和/或`tls.createSecureContext` `options` 参数的其他任何属性。

### server.maxConnections

设置这个属性可以在服务器的连接数达到最大值时拒绝连接。

### server.connections

当前服务器连接数。


## 类: CryptoStream

    稳定性: 0 - 抛弃. 使用 tls.TLSSocket 替代.

这是一个加密的流

### cryptoStream.bytesWritten

底层 socket 写字节访问器（bytesWritten accessor）的代理，将会返回写到 socket 的全部字节数。*包括 TLS 的开销*。

## 类: tls.TLSSocket

[net.Socket][] 实例的封装，透明的加密写数据和所有必须的 TLS 协商。


这个接口实现了一个双工流接口。它包含所有常用的流方法和事件。

### 事件: 'secureConnect'

新的连接成功握手后回触发这个事件。无论服务器证书是否授权，都会调用监听器。用于用户测试 `tlsSocket.authorized`看看如果服务器证书已经被指定的 CAs 签名。如果 `tlsSocket.authorized === false` ，可以在 `tlsSocket.authorizationError` 里找到错误。如果使用了 NPN，你可以检`tlsSocket.npnProtocol` 获取协商协议（negotiated protocol）。

### 事件: 'OCSPResponse'

`function (response) { }`

如果启用 `requestOCSP` 参赛会触发这个事件。 `response` 是缓存对象，包含服务器的 OCSP 响应。

一般来说， `response`是服务器 CA 签名的对象，它包含服务器撤销证书状态的信息。

### tlsSocket.encrypted

静态 boolean 变量，一直是 `true`。可以用来区别 TLS socket 和 常规对象。

### tlsSocket.authorized

boolean 变量，如果  对等实体证书（ peer's certificate）被指定的某个 CAs 签名，返回 `true`，否则 `false`。

### tlsSocket.authorizationError

对等实体证书（ peer's certificate）没有验证通过的原因。当 `tlsSocket.authorized === false`时，这个属性才可用。

### tlsSocket.getPeerCertificate([ detailed ])

返回一个代表对等实体证书（ peer's certificate）的对象。这个返回对象有一些属性和证书内容相对应。如果参数 `detailed` 是 `true`，将会返回包含发行者`issuer` 完整链。如果`false`，仅有顶级证书没有发行者`issuer`属性。

例子：

    { subject:
       { C: 'UK',
         ST: 'Acknack Ltd',
         L: 'Rhys Jones',
         O: 'node.js',
         OU: 'Test TLS Certificate',
         CN: 'localhost' },
      issuerInfo:
       { C: 'UK',
         ST: 'Acknack Ltd',
         L: 'Rhys Jones',
         O: 'node.js',
         OU: 'Test TLS Certificate',
         CN: 'localhost' },
      issuer:
       { ... another certificate ... },
      raw: < RAW DER buffer >,
      valid_from: 'Nov 11 09:52:22 2009 GMT',
      valid_to: 'Nov  6 09:52:22 2029 GMT',
      fingerprint: '2A:7A:C2:DD:E5:F9:CC:53:72:35:99:7A:02:5A:71:38:52:EC:8A:DF',
      serialNumber: 'B9B0D332A1AA5635' }

如果 peer 没有提供证书，返回 `null`或空对象。

### tlsSocket.getCipher()
返回一个对象，它代表了密码名和当前连接的 SSL/TLS 协议的版本。


例子：
{ name: 'AES256-SHA', version: 'TLSv1/SSLv3' }

更多信息参见http://www.openssl.org/docs/ssl/ssl.html#DEALING_WITH_CIPHERS 里的 SSL_CIPHER_get_name() 和 SSL_CIPHER_get_version() 。
  

### tlsSocket.renegotiate(options, callback)

初始化 TLS 重新协商进程。参数 `options` 可能包含以下内容： `rejectUnauthorized`, `requestCert` (细节参见 [tls.createServer][])。一旦重新协商成（renegotiation）功完成，将会执行 `callback(err)`，其中 `err` 为 `null`。



注意： 当安全连接建立后，可以用这来请求对等实体证书（ peer's certificate）。

注意： 作为服务器运行时，`handshakeTimeout`  超时后，socket 将会被销毁。

### tlsSocket.setMaxSendFragment(size)

设置最大的 TLS 碎片大小（默认最大值为：`16384`，最小值为：`512`）。成功的话，返回 `true`，否则返回 `false`。
  
小的碎片包会减少客户端的缓存延迟：大的碎片直到接收完毕后才能被 TLS 层完全缓存，并且验证过完整性；大的碎片可能会有多次往返，并且可能会因为丢包或重新排序导致延迟。而小的碎片会增加额外的 TLS 帧字节和 CPU 负载，这会减少 CPU 的吞吐量。


### tlsSocket.getSession()

返回 ASN.1 编码的 TLS 会话，如果没有协商，会返回。连接到服务器时，可以用来加速握手的建立。 

### tlsSocket.getTLSTicket()

注意： 仅和客户端 TLS socket 打交道。仅在调试时有用，会话重用是，提供 `session`  参数给 `tls.connect`。

返回 TLS 会话票据（ticket），或如果没有协商（negotiated），返回 `undefined`。

### tlsSocket.address()

返回绑定的地址，地址家族名和服务器端口。更多信息参见 [net.Server.address()][]。返回三个属性, 比如：
`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`

### tlsSocket.remoteAddress

表示远程 IP 地址（字符串表示），例如：`'74.125.127.100'` 或  `'2001:4860:a005::68'`.

### tlsSocket.remoteFamily

表示远程 IP 家族， `'IPv4'` 或 `'IPv6'`.

### tlsSocket.remotePort

远程端口（数字表示），列如, `443`.

### tlsSocket.localAddress

本地 IP 地址（字符串表示）。

### tlsSocket.localPort

本地端口号（数字表示）。

[OpenSSL cipher list format documentation]: http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT
[BEAST attacks]: http://blog.ivanristic.com/2011/10/mitigating-the-beast-attack-on-tls.html
[tls.createServer]: #tls_tls_createserver_options_secureconnectionlistener
[tls.createSecurePair]: #tls_tls_createsecurepair_credentials_isserver_requestcert_rejectunauthorized
[tls.TLSSocket]: #tls_class_tls_tlssocket
[net.Server]: net.html#net_class_net_server
[net.Socket]: net.html#net_class_net_socket
[net.Server.address()]: net.html#net_server_address
['secureConnect']: #tls_event_secureconnect
[secureConnection]: #tls_event_secureconnection
[Stream]: stream.html#stream_stream
[SSL_METHODS]: http://www.openssl.org/docs/ssl/ssl.html#DEALING_WITH_PROTOCOL_METHODS
[tls.Server]: #tls_class_tls_server
[SSL_CTX_set_timeout]: http://www.openssl.org/docs/ssl/SSL_CTX_set_timeout.html
[RFC 4492]: http://www.rfc-editor.org/rfc/rfc4492.txt
[Forward secrecy]: http://en.wikipedia.org/wiki/Perfect_forward_secrecy
[DHE]: https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange
[ECDHE]: https://en.wikipedia.org/wiki/Elliptic_curve_Diffie%E2%80%93Hellman
[asn1.js]: http://npmjs.org/package/asn1.js
[OCSP request]: http://en.wikipedia.org/wiki/OCSP_stapling
[SSL_CTX_set_options]: https://www.openssl.org/docs/ssl/SSL_CTX_set_options.html
[CVE-2014-3566]: https://access.redhat.com/articles/1232123
