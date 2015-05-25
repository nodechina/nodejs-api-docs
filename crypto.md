# 加密

    稳定性: 2 - 不稳定; 正在讨论未来版本的 API 改进，会尽量减少重大变化。详见后文。

使用 `require('crypto')` 来访问这个模块。

加密模块提供了 HTTP 或 HTTPS 连接过程中封装安全凭证的方法。

它也提供了 OpenSSL 的哈希，hmac, 加密（cipher）, 解密（decipher）, 签名（sign） 和 验证（verify） 方法的封装。


## crypto.setEngine(engine[, flags])

为某些/所有 OpenSSL 函数加载并设置引擎（根据参数 flags 来设置）。

`engine` 可能是 id，或者是指向引擎共享库的路径。

`flags` 是可选参数，默认值是`ENGINE_METHOD_ALL` ，它可以是以下一个或多个参数的组合（在`constants`里定义）:

* `ENGINE_METHOD_RSA`
* `ENGINE_METHOD_DSA`
* `ENGINE_METHOD_DH`
* `ENGINE_METHOD_RAND`
* `ENGINE_METHOD_ECDH`
* `ENGINE_METHOD_ECDSA`
* `ENGINE_METHOD_CIPHERS`
* `ENGINE_METHOD_DIGESTS`
* `ENGINE_METHOD_STORE`
* `ENGINE_METHOD_PKEY_METH`
* `ENGINE_METHOD_PKEY_ASN1_METH`
* `ENGINE_METHOD_ALL`
* `ENGINE_METHOD_NONE`


## crypto.getCiphers()

返回支持的加密算法名数组。

例如：

    var ciphers = crypto.getCiphers();
    console.log(ciphers); // ['AES-128-CBC', 'AES-128-CBC-HMAC-SHA1', ...]


## crypto.getHashes()

返回支持的哈希算法名数组。

例如：

    var hashes = crypto.getHashes();
    console.log(hashes); // ['sha', 'sha1', 'sha1WithRSAEncryption', ...]


## crypto.createCredentials(details)

    稳定性: 0 - 抛弃. 用 [tls.createSecureContext][] 替换.
根据参数 details，创建一个加密凭证对象。参数为字典，key 包括:

* `pfx` : 字符串或者buffer对象，表示经PFX或PKCS12编码产生的私钥、证书以及CA证书
* `key` : 进过 PEM 编码的私钥
* `passphrase` : 私钥或 pfx 的密码
* `cert` : PEM 编码的证书
* `ca` : 字符串或字符串数组，PEM 编码的可信任的 CA 证书。
* `crl` : 字符串或字符串数组，PEM 编码的 CRLs（证书吊销列表Certificate Revocation List）。
* `ciphers`: 字符串，使用或者排除的加密算法。参见<http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT>。

如果没有指定 'ca'，Node.js将会使用下面列表中的CA<http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt>。


## crypto.createHash(algorithm)

创建并返回一个哈希对象，使用指定的算法来生成哈希摘要。

参数 `algorithm` 取决于平台上 OpenSSL 版本所支持的算法。例如，`'sha1'`, `'md5'`,
`'sha256'`, `'sha512'` 等等。在最近的版本中，`openssllist-message-digest-algorithms` 会显示所有算法。

例如： 这个程序会计算文件的 sha1 的和。

    var filename = process.argv[2];
    var crypto = require('crypto');
    var fs = require('fs');

    var shasum = crypto.createHash('sha1');

    var s = fs.ReadStream(filename);
    s.on('data', function(d) {
      shasum.update(d);
    });

    s.on('end', function() {
      var d = shasum.digest('hex');
      console.log(d + '  ' + filename);
    });

## 类： Hash

用来生成数据的哈希值。

它是可读写的流 [stream](stream.html) 。写入的数据来用计算哈希值。当写入流结束后，使用 `read()` 方法来获取计算后的哈希值。也支持老的 `update` 和 `digest` 方法。

通过 `crypto.createHash` 返回。

### hash.update(data[, input_encoding])

根据 `data` 来更新哈希内容，编码方式根据 `input_encoding` 来定，有 `'utf8'`, `'ascii'` 或
`'binary'`。如果没有传入值，默认编码方式是`'binary'`。如果 `data` 是 `Buffer`， `input_encoding` 将会被忽略。  

因为它是流式数据，所以可以使用不同的数据调用很多次。  

### hash.digest([encoding])

计算传入的数据的哈希摘要。`encoding` 可以是 `'hex'`, `'binary'` 或 `'base64'`，如果没有指定`encoding` ，将返回 buffer。  
注意：调用  `digest()`  后不能再用 `hash` 对象。


## crypto.createHmac(algorithm, key)

创建并返回一个 hmac 对象，用指定的算法和秘钥生成 hmac 图谱。

它是可读写的流 [stream](stream.html) 。写入的数据来用计算 hmac。当写入流结束后，使用 `read()` 方法来获取计算后的值。也支持老的 `update` 和 `digest` 方法。

参数 `algorithm` 取决于平台上 OpenSSL 版本所支持的算法，参见前面的 createHash。`key`是 hmac 算法中用的 key。  

## 类： Hmac

用来创建 hmac 加密图谱。

通过 `crypto.createHmac` 返回。

### hmac.update(data)

根据 `data` 更新 hmac 对象。因为它是流式数据，所以可以使用新数据调用多次。

### hmac.digest([encoding])

计算传入数据的 hmac 值。`encoding`可以是 `'hex'`, `'binary'` 或 `'base64'`，如果没有指定`encoding` ，将返回 buffer。 

注意：调用  `digest()`  后不能再用 `hmac` 对象。


## crypto.createCipher(algorithm, password)

使用传入的算法和秘钥来生成并返回加密对象。    

`algorithm` 取决于 OpenSSL，例如`'aes192'`等。最近发布的版本中， `openssl list-cipher-algorithms` 将会展示可用的加密算法。`password` 用来派生 key 和 IV，它必须是一个`'binary'` 编码的字符串或者一个[buffer](buffer.html)。

它是可读写的流 [stream](stream.html) 。写入的数据来用计算 hmac。当写入流结束后，使用 `read()` 方法来获取计算后的值。也支持老的`update` 和 `digest` 方法。

注意，OpenSSL 函数[EVP_BytesToKey][]摘要算法如果是一次迭代（one iteration），无需盐值（no salt）的 MD5 时， `createCipher` 为它派生秘钥。缺少盐值使得字典攻击，相同的密码总是生成相同的key，低迭代次数和非加密的哈希算法，使得密码测试非常迅速。  

OpenSSL推荐使用 pbkdf2 来替换 EVP_BytesToKey，推荐使用 [crypto.pbkdf2][] 来派生 key 和 
 iv ，推荐使用 [createCipheriv()][] 来创建加密流。


## crypto.createCipheriv(algorithm, key, iv)

创建并返回一个加密对象，用指定的算法，key 和 iv。

`algorithm` 参数和 `createCipher()` 一致。`key` 在算法中用到.`iv` 是一个[initialization vector](http://en.wikipedia.org/wiki/Initialization_vector).

`key` 和 `iv` 必须是 `'binary'` 的编码字符串或[buffers](buffer.html).

## 类： Cipher

加密数据的类。.

通过 `crypto.createCipher` 和 `crypto.createCipheriv` 返回。	

它是可读写的流 [stream](stream.html) 。写入的数据来用计算 hmac。当写入流结束后，使用 `read()` 方法来获取计算后的值。也支持老的`update` 和 `digest` 方法。  

### cipher.update(data[, input_encoding][, output_encoding])

根据 `data` 来更新哈希内容，编码方式根据 `input_encoding` 来定，有 `'utf8'`, `'ascii'` or
`'binary'`。如果没有传入值，默认编码方式是`'binary'`。如果`data` 是 `Buffer`，`input_encoding` 将会被忽略。  

`output_encoding` 指定了输出的加密数据的编码格式，它可用是 `'binary'`, `'base64'` 或 `'hex'`。如果没有提供编码，将返回 buffer 。

返回加密后的内容，因为它是流式数据，所以可以使用不同的数据调用很多次。

### cipher.final([output_encoding])

返回加密后的内容，编码方式是由 `output_encoding` 指定，可以是 `'binary'`, `'base64'` 或 `'hex'`。如果没有传入值，将返回 buffer。  

注意：`cipher` 对象不能在 `final()` 方法之后调用。

### cipher.setAutoPadding(auto_padding=true)

你可以禁用输入数据自动填充到块大小的功能。如果 `auto_padding` 是false， 那么输入数据的长度必须是加密器块大小的整倍数，否则 `final` 会失败。这对非标准的填充很有用，例如使用0x0而不是PKCS的填充。这个函数必须在 `cipher.final` 之前调用。

### cipher.getAuthTag()

加密认证模式（目前支持：GCM），这个方法返回经过计算的_认证标志_ `Buffer`。必须使用`final`方法完全加密后调用。

### cipher.setAAD(buffer)

加密认证模式（目前支持：GCM），这个方法设置附加认证数据（ AAD ）。

## crypto.createDecipher(algorithm, password)

根据传入的算法和密钥，创建并返回一个解密对象。这是 [createCipher()][] 的镜像。

## crypto.createDecipheriv(algorithm, key, iv)

根据传入的算法，密钥和 iv，创建并返回一个解密对象。这是 [createCipheriv()][] 的镜像。

## 类： Decipher

解密数据类。

通过 `crypto.createDecipher` 和 `crypto.createDecipheriv` 返回。

解密对象是可读写的流 [streams](stream.html) 。用写入的加密数据生成可读的纯文本数据。也支持老的`update` 和 `digest` 方法。 

### decipher.update(data[, input_encoding][, output_encoding])

使用参数 `data` 更新需要解密的内容，其编码方式是 `'binary'`,`'base64'` 或 `'hex'`。如果没有指定编码方式，则把 `data` 当成 `buffer` 对象。

如果 `data` 是 `Buffer`，则忽略 `input_encoding` 参数。

参数 `output_decoding` 指定返回文本的格式，是 `'binary'`, `'ascii'` 或 `'utf8'` 之一。如果没有提供编码格式，则返回 buffer。

### decipher.final([output_encoding])

返回剩余的解密过的内容，参数 `output_encoding` 是 `'binary'`, `'ascii'` 或 `'utf8'`，如果没有指定编码方式，返回 buffer。

注意，`decipher`对象不能在 `final()` 方法之后使用。

### decipher.setAutoPadding(auto_padding=true)

如果加密的数据是非标准块，可以禁止其自动填充，防止 `decipher.final` 检查并移除。仅在输入数据长度是加密块长度的整数倍的时才有效。你必须在 `decipher.update` 前调用。

### decipher.setAuthTag(buffer)

对于加密认证模式（目前支持：GCM），必须用这个方法来传递接收到的_认证标志_。如果没有提供标志，或者密文被篡改，将会抛出 `final` 标志，认证失败，密文会被抛弃，


### decipher.setAAD(buffer)

对于加密认证模式（目前支持：GCM），用这个方法设置附加认证数据（ AAD ）。


## crypto.createSign(algorithm)

根据传入的算法创建并返回一个签名数据。 OpenSSL 的最近版本里，`openssl list-public-key-algorithms` 会列出所有算法，比如`'RSA-SHA256'`。

## 类： Sign

生成数字签名的类。

通过 `crypto.createSign` 返回。

签名对象是可读写的流 [streams](stream.html)。可写数据用来生成签名。当所有的数据写完，`sign` 签名方法会返回签名。也支持老的 `update` 和 `digest` 方法。 


### sign.update(data)

用参数 `data` 来更新签名对象。因为是流式数据，它可以被多次调用。

### sign.sign(private_key[, output_format])

根据传送给sign的数据来计算电子签名。  
  
`private_key` 可以是一个对象或者字符串。如果是字符串，将会被当做没有密码的key。

`private_key`:

* `key` : 包含 PEM 编码的私钥
* `passphrase` : 私钥的密码

返回值`output_format` 包含数字签名， 格式是 `'binary'`,`'hex'` 或 `'base64'` 之一。如果没有指定 `encoding` ，将返回 buffer。  
  
注意：`sign` 对象不能在 `sign()` 方法之后调用。


## crypto.createVerify(algorithm)

根据传入的算法，创建并返回验证对象。是签名对象（signing object）的镜像。

## 类： Verify

用来验证签名的类。

通过 `crypto.createVerify` 返回。

是可写流 [streams](stream.html)。可写数据用来验证签名。一旦所有数据写完后，如签名正确 `verify` 方法会返回 `true` 。  
  
也支持老的 `update` 方法。

### verifier.update(data)

用参数 `data` 来更新验证对象。因为是流式数据，它可以被多次调用。

### verifier.verify(object, signature[, signature_format])

使用 `object` 和  `signature` 验证签名数据。参数 `object` 是包含了 PEM 编码对象的字符串，它可以是 RSA 公钥, DSA 公钥, 或 X.509 证书。`signature` 是之前计算出来的数字签名。`signature_format` 可以是 `'binary'`, `'hex'` 或 `'base64'` 之一，如果没有指定编码方式 ，则默认是buffer 对象。 

根据数据和公钥验证签名有效性，来返回 true 或 false。  

注意：`verifier` 对象不能在 `verify()` 方法之后调用。

## crypto.createDiffieHellman(prime_length[, generator])

创建一个 Diffie-Hellman 密钥交换(Diffie-Hellman key exchange)对象，并根据给定的位长度生成一个质数。如果没有指定参数 `generator`，默认为 `2`。

## crypto.createDiffieHellman(prime[, prime_encoding][, generator][, generator_encoding])

使用传入的 `prime` 和 `generator` 创建 Diffie-Hellman 秘钥交互对象。  
  
`generator` 可以是数字，字符串或Buffer。  
  
如果没有指定 `generator`，使用 `2`.  
  
`prime_encoding` 和 `generator_encoding` 可以是 `'binary'`, `'hex'`, 或 `'base64'`。  
  
如果没有指定 `prime_encoding`， 则 Buffer 为 `prime`。
  
如果没有指定 `generator_encoding` ，则 Buffer 为 `generator`。

## 类： DiffieHellman

创建 Diffie-Hellman 秘钥交换的类。

通过 `crypto.createDiffieHellman` 返回。	

### diffieHellman.verifyError

在初始化的时候，如果有警告或错误，将会反应到这。它是以下值（定义在 `constants` 模块）：

* `DH_CHECK_P_NOT_SAFE_PRIME`
* `DH_CHECK_P_NOT_PRIME`
* `DH_UNABLE_TO_CHECK_GENERATOR`
* `DH_NOT_SUITABLE_GENERATOR`

### diffieHellman.generateKeys([encoding])

生成秘钥和公钥，并返回指定格式的公钥。这个值必须传给其他部分。编码方式： `'binary'`, `'hex'`,
或 `'base64'`。如果没有指定编码方式，将返回 buffer。 


### diffieHellman.computeSecret(other_public_key[, input_encoding][, output_encoding])

使用 `other_public_key` 作为第三方公钥来计算并返回共享秘密（shared secret）。秘钥用`input_encoding` 编码。编码方式为：`'binary'`, `'hex'`, 或 `'base64'`。如果没有指定编码方式 ，默认为 buffer。   
  
如果没有指定返回编码方式，将返回 buffer。

### diffieHellman.getPrime([encoding])

用参数 encoding 指明的编码方式返回 Diffie-Hellman 质数，编码方式为: `'binary'`, `'hex'`, 或 `'base64'`。 如果没有指定编码方式，将返回 buffer。

### diffieHellman.getGenerator([encoding])

用参数 encoding 指明的编码方式返回 Diffie-Hellman 生成器，编码方式为: `'binary'`, `'hex'`, 或 `'base64'`. 如果没有指定编码方式 ，将返回 buffer。

### diffieHellman.getPublicKey([encoding])

用参数 encoding 指明的编码方式返回 Diffie-Hellman 公钥，编码方式为: `'binary'`, `'hex'`, 或 `'base64'`. 如果没有指定编码方式 ，将返回 buffer。  

### diffieHellman.getPrivateKey([encoding])

用参数 encoding 指明的编码方式返回 Diffie-Hellman 私钥，编码方式为: `'binary'`, `'hex'`, 或 `'base64'`. 如果没有指定编码方式 ，将返回 buffer。 


### diffieHellman.setPublicKey(public_key[, encoding])

设置 Diffie-Hellman 的公钥，编码方式为: `'binary'`, `'hex'`, 或 `'base64'`，如果没有指定编码方式 ，默认为 buffer。 

### diffieHellman.setPrivateKey(private_key[, encoding])

设置 Diffie-Hellman 的私钥，编码方式为: `'binary'`, `'hex'`, 或 `'base64'`，如果没有指定编码方式 ，默认为 buffer。 

## crypto.getDiffieHellman(group_name)

创建一个预定义的 Diffie-Hellman 秘钥交换对象。支持的组： `'modp1'`, `'modp2'`, `'modp5'` (定义于[RFC 2412][]) and `'modp14'`, `'modp15'`, `'modp16'`, `'modp17'`,`'modp18'` (定义于[RFC 3526][]). 返回对象模仿了上述创建的[crypto.createDiffieHellman()][]对象，但是不允许修改秘钥交换（例如，[diffieHellman.setPublicKey()][]）。使用这套流程的好处是，双方不需要生成或交换组组余数，节省了计算和通讯时间。

例如 (获取一个共享秘密):

    var crypto = require('crypto');
    var alice = crypto.getDiffieHellman('modp5');
    var bob = crypto.getDiffieHellman('modp5');

    alice.generateKeys();
    bob.generateKeys();

    var alice_secret = alice.computeSecret(bob.getPublicKey(), null, 'hex');
    var bob_secret = bob.computeSecret(alice.getPublicKey(), null, 'hex');

    /* alice_secret and bob_secret should be the same */
    console.log(alice_secret == bob_secret);

## crypto.createECDH(curve_name)

使用传入的参数 `curve_name`,创建一个 Elliptic Curve (EC) Diffie-Hellman 秘钥交换对象。

## 类： ECDH
这个类用来创建  EC Diffie-Hellman 秘钥交换。

通过 `crypto.createECDH` 返回。

### ECDH.generateKeys([encoding[, format]])

生成 EC Diffie-Hellman 的秘钥和公钥，并返回指定格式和编码的公钥，它会传递给第三方。

参数 `format` 是 `'compressed'`, `'uncompressed'`, 或 `'hybrid'`. 如果没有指定，将返回`'uncompressed'` 格式.

参数`encoding`是 `'binary'`, `'hex'`, 或 `'base64'`. 如果没有指定编码方式 ，将返回 buffer。 


### ECDH.computeSecret(other_public_key[, input_encoding][, output_encoding])
  
以`other_public_key` 作为第三方公钥计算共享秘密，并返回。秘钥会以`input_encoding`来解读。编码是：`'binary'`, `'hex'`, 或 `'base64'`。如果没有指定编码方式 ，默认为 buffer。   
  
如果没有指定编码方式 ，将返回 buffer。

### ECDH.getPublicKey([encoding[, format]])
  
用参数 encoding 指明的编码方式返回 EC Diffie-Hellman 公钥，编码方式为: `'compressed'`, `'uncompressed'`, 或
`'hybrid'`. 如果没有指定编码方式 ，将返回`'uncompressed'` 。
 
编码是：`'binary'`, `'hex'`, 或 `'base64'`。如果没有指定编码方式 ，默认为 buffer。   


### ECDH.getPrivateKey([encoding])

用参数 encoding 指明的编码方式返回 EC Diffie-Hellman 私钥，编码是：`'binary'`, `'hex'`, 或 `'base64'`。如果没有指定编码方式 ，默认为 buffer。

### ECDH.setPublicKey(public_key[, encoding])

设置  EC Diffie-Hellman 的公钥，编码方式为: `'binary'`, `'hex'`, 或 `'base64'`，如果没有指定编码方式 ，默认为 buffer。

### ECDH.setPrivateKey(private_key[, encoding])

设置  EC Diffie-Hellman 的私钥，编码方式为: `'binary'`, `'hex'`, 或 `'base64'`，如果没有指定编码方式 ，默认为 buffer。

例如 (包含一个共享秘密):

    var crypto = require('crypto');
    var alice = crypto.createECDH('secp256k1');
    var bob = crypto.createECDH('secp256k1');

    alice.generateKeys();
    bob.generateKeys();

    var alice_secret = alice.computeSecret(bob.getPublicKey(), null, 'hex');
    var bob_secret = bob.computeSecret(alice.getPublicKey(), null, 'hex');

    /* alice_secret and bob_secret should be the same */
    console.log(alice_secret == bob_secret);

## crypto.pbkdf2(password, salt, iterations, keylen[, digest], callback)

异步 PBKDF2 提供了一个伪随机函数 HMAC-SHA1，根据给定密码的长度，salt 和 iterations 来得出一个密钥。回调函数得到两个参数 (err, derivedKey)。

例如：

    crypto.pbkdf2('secret', 'salt', 4096, 512, 'sha256', function(err, key) {
      if (err)
        throw err;
      console.log(key.toString('hex'));  // 'c5e478d...1469e50'
    });

在 [crypto.getHashes()](#crypto_crypto_gethashes) 里有支持的摘要函数列表。

## crypto.pbkdf2Sync(password, salt, iterations, keylen[, digest])

异步 PBKDF2 函数.  返回 derivedKey 或抛出错误。

## crypto.randomBytes(size[, callback])

生成一个密码强度随机的数据：

    // async
    crypto.randomBytes(256, function(ex, buf) {
      if (ex) throw ex;
      console.log('Have %d bytes of random data: %s', buf.length, buf);
    });

    // sync
    try {
      var buf = crypto.randomBytes(256);
      console.log('Have %d bytes of random data: %s', buf.length, buf);
    } catch (ex) {
      // handle error
      // most likely, entropy sources are drained
    }

注意：如果没有足够积累的熵来生成随机强度的密码，将会抛出错误，或调用回调函数返回错误。换句话说，没有回调函数的 `crypto.randomBytes` 不会阻塞，即使耗尽所有的熵。

## crypto.pseudoRandomBytes(size[, callback])

生成非密码学强度的伪随机数据。如果数据足够长会返回一个唯一数据，但是这个数可能是可以预期的。因此，当不可预期很重要的时候，不要用这个函数。例如，在生成加密的秘钥时。

用法和 `crypto.randomBytes` 相同。

## 类： Certificate

这个类和签过名的公钥打交道。最重要的场景是处理 `<keygen>` 元素，http://www.openssl.org/docs/apps/spkac.html。

通过 `crypto.Certificate` 返回.

### Certificate.verifySpkac(spkac)

根据 SPKAC 返回 true 或 false。

### Certificate.exportChallenge(spkac)

根据提供的SPKAC，返回加密的公钥。

### Certificate.exportPublicKey(spkac)

输出和 SPKAC 关联的编码 challenge。  

## crypto.publicEncrypt(public_key, buffer)

使用 `public_key` 加密  `buffer`。目前仅支持 RSA。
  
`public_key` 可以是对象或字符串。如果 `public_key` 是一个字符串，将会当做没有密码的key，并会用`RSA_PKCS1_OAEP_PADDING`。

`public_key`:

* `key` : 包含有 PEM 编码的私钥。
* `padding` : 填充值，如下
  * `constants.RSA_NO_PADDING`
  * `constants.RSA_PKCS1_PADDING`
  * `constants.RSA_PKCS1_OAEP_PADDING`

注意: 所有的填充值 定义于`constants` 模块.

## crypto.privateDecrypt(private_key, buffer)

使用 `private_key` 来解密 `buffer`.

`private_key`:

* `key` : 包含有 PEM 编码的私钥
* `passphrase` : 私钥的密码
* `padding` : 填充值，如下:
  * `constants.RSA_NO_PADDING`
  * `constants.RSA_PKCS1_PADDING`
  * `constants.RSA_PKCS1_OAEP_PADDING`

注意: 所有的填充值 定义于`constants` 模块.

## crypto.DEFAULT_ENCODING

函数所用的编码方式可以是字符串或 buffer ，默认值是 'buffer'。这是为了加密模块兼容默认 'binary' 为编码方式的遗留程序。

注意，新程序希望用 buffer 对象，所以这是暂时手段。


## Recent API Changes

在统一的流 API 概念出现前，在引入 Buffer 对象来处理二进制数据之前，Crypto 模块就已经添加到 Node。    
  
因此，流相关的类里没有其他的 Node 类里的典型方法，并且很多方法接收并返回二级制编码的字符串，而不是 Buffers。在最近的版本中，这些函数改成默认使用 Buffers。
	
对于一些场景来说这是重大变化。  
  
例如，如果你使用默认参数给签名类，将结果返回给认证类，中间没有验证数据，程序会正常工作。之前你会得到二进制编码的字符串，并传递给验证类，现在则是 Buffer。
  
如果你之前使用的字符串数据在 Buffers 对象不能正常工作（比如，连接数据，并存储在数据库里 ）。或者你传递了二进制字符串给加密函数，但是没有指定编码方式，现在就需要提供编码参数。如果想切换回原来的风格，将 `crypto.DEFAULT_ENCODING` 设置为  'binary'。注意，新的程序希望是 buffers,所以之前的方法只能作为临时的办法。


[createCipher()]: #crypto_crypto_createcipher_algorithm_password
[createCipheriv()]: #crypto_crypto_createcipheriv_algorithm_key_iv
[crypto.createDiffieHellman()]: #crypto_crypto_creatediffiehellman_prime_encoding
[tls.createSecureContext]: tls.html#tls_tls_createsecurecontext_details
[diffieHellman.setPublicKey()]: #crypto_diffiehellman_setpublickey_public_key_encoding
[RFC 2412]: http://www.rfc-editor.org/rfc/rfc2412.txt
[RFC 3526]: http://www.rfc-editor.org/rfc/rfc3526.txt
[crypto.pbkdf2]: #crypto_crypto_pbkdf2_password_salt_iterations_keylen_callback
[EVP_BytesToKey]: https://www.openssl.org/docs/crypto/EVP_BytesToKey.html
