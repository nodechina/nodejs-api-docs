# Zlib

    稳定性: 3 - 文档

可以通过以下方式访问这个模块:

    var zlib = require('zlib');

这个模块提供了对 Gzip/Gunzip, Deflate/Inflate, 和 DeflateRaw/InflateRaw 类的绑定。每个类都有相同的参数和可读/写的流。 

## 例子

压缩/解压缩一个文件，可以通过倒流（piping）一个 fs.ReadStream 到 zlib 流里来，再到一个 fs.fs.WriteStream.

    var gzip = zlib.createGzip();
    var fs = require('fs');
    var inp = fs.createReadStream('input.txt');
    var out = fs.createWriteStream('input.txt.gz');

    inp.pipe(gzip).pipe(out);

一步压缩/解压缩数据可以通过一个简便方法来实现。

    var input = '.................................';
    zlib.deflate(input, function(err, buffer) {
      if (!err) {
        console.log(buffer.toString('base64'));
      }
    });

    var buffer = new Buffer('eJzT0yMAAGTvBe8=', 'base64');
    zlib.unzip(buffer, function(err, buffer) {
      if (!err) {
        console.log(buffer.toString());
      }
    });

要在一个 HTTP 客户端或服务器中使用这个模块，可以在请求时使用 [accept-encoding](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.3)，响应时使用 [content-encoding](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.11) 头。

**注意: 这些例子只是简单展示了基本概念。**  Zlib 编码可能消耗非常大，并且结果可能要被缓存。更多使用 zlib 相关的速度/内存/压缩的权衡选择细节参见后面的  [Memory Usage Tuning](#zlib_memory_usage_tuning)。

    // client request example
    var zlib = require('zlib');
    var http = require('http');
    var fs = require('fs');
    var request = http.get({ host: 'izs.me',
                             path: '/',
                             port: 80,
                             headers: { 'accept-encoding': 'gzip,deflate' } });
    request.on('response', function(response) {
      var output = fs.createWriteStream('izs.me_index.html');

      switch (response.headers['content-encoding']) {
        // or, just use zlib.createUnzip() to handle both cases
        case 'gzip':
          response.pipe(zlib.createGunzip()).pipe(output);
          break;
        case 'deflate':
          response.pipe(zlib.createInflate()).pipe(output);
          break;
        默认：
          response.pipe(output);
          break;
      }
    });

    // server example
    // Running a gzip operation on every request is quite expensive.
    // It would be much more efficient to cache the compressed buffer.
    var zlib = require('zlib');
    var http = require('http');
    var fs = require('fs');
    http.createServer(function(request, response) {
      var raw = fs.createReadStream('index.html');
      var acceptEncoding = request.headers['accept-encoding'];
      if (!acceptEncoding) {
        acceptEncoding = '';
      }

      // Note: this is not a conformant accept-encoding parser.
      // See http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.3
      if (acceptEncoding.match(/\bdeflate\b/)) {
        response.writeHead(200, { 'content-encoding': 'deflate' });
        raw.pipe(zlib.createDeflate()).pipe(response);
      } else if (acceptEncoding.match(/\bgzip\b/)) {
        response.writeHead(200, { 'content-encoding': 'gzip' });
        raw.pipe(zlib.createGzip()).pipe(response);
      } else {
        response.writeHead(200, {});
        raw.pipe(response);
      }
    }).listen(1337);

## zlib.createGzip([options])


根据参数 [options](#zlib_options) 返回一个新的  [Gzip](#zlib_class_zlib_gzip) 对象。

## zlib.createGunzip([options])

根据参数 [options](#zlib_options) 返回一个新的  [Gunzip](#zlib_class_zlib_gunzip) 对象。

## zlib.createDeflate([options])

根据参数 [options](#zlib_options) 返回一个新的  [Deflate](#zlib_class_zlib_deflate) 对象。

## zlib.createInflate([options])

根据参数 [options](#zlib_options) 返回一个新的  [Inflate](#zlib_class_zlib_inflate) 对象。

## zlib.createDeflateRaw([options])

根据参数 [options](#zlib_options) 返回一个新的  [DeflateRaw](#zlib_class_zlib_deflateraw) 对象。

## zlib.createInflateRaw([options])

根据参数 [options](#zlib_options) 返回一个新的  [InflateRaw](#zlib_class_zlib_inflateraw) 对象。

## zlib.createUnzip([options])

根据参数 [options](#zlib_options) 返回一个新的  [Unzip](#zlib_class_zlib_unzip) 对象。


## Class: zlib.Zlib

这个类未被 `zlib` 模块导出。之所以写在这，是因为这是压缩/解压缩类的基类。

### zlib.flush([kind], callback)

参数 `kind` 默认为 `zlib.Z_FULL_FLUSH`。

刷入缓冲数据。不要轻易调用这个方法，过早的刷会对压缩算法产生负面影响。

### zlib.params(level, strategy, callback)

动态更新压缩基本和压缩策略。仅对 deflate 算法有效。

### zlib.reset()

重置压缩/解压缩为默认值。仅适用于 inflate 和 deflate 算法。

## Class: zlib.Gzip

使用 gzip 压缩数据。

## Class: zlib.Gunzip

使用 gzip 解压缩数据。

## Class: zlib.Deflate

使用 deflate 压缩数据。

## Class: zlib.Inflate

解压缩 deflate 流。

## Class: zlib.DeflateRaw

使用 deflate 压缩数据，不需要拼接 zlib 头。

## Class: zlib.InflateRaw

解压缩一个原始 deflate 流。

## Class: zlib.Unzip

通过自动检测头解压缩一个 Gzip- 或 Deflate-compressed 流。

## 简便方法

<!--type=misc-->

所有的这些方法第一个参数为字符串或缓存，第二个可选参数可以供 zlib 类使用，回调函数为 `callback(error, result)`。

每个方法都有一个 `*Sync` 伴随方法，它接收相同参数，不过没有回调。

## zlib.deflate(buf[, options], callback)
## zlib.deflateSync(buf[, options])

使用 Deflate 压缩一个字符串。

## zlib.deflateRaw(buf[, options], callback)
## zlib.deflateRawSync(buf[, options])

使用 DeflateRaw 压缩一个字符串。

## zlib.gzip(buf[, options], callback)
## zlib.gzipSync(buf[, options])

使用 Gzip 压缩一个字符串。

## zlib.gunzip(buf[, options], callback)
## zlib.gunzipSync(buf[, options])

使用 Gunzip 解压缩一个原始的 Buffer。

## zlib.inflate(buf[, options], callback)
## zlib.inflateSync(buf[, options])

使用 Inflate 解压缩一个原始的 Buffer。

## zlib.inflateRaw(buf[, options], callback)
## zlib.inflateRawSync(buf[, options])

使用 InflateRaw 解压缩一个原始的 Buffer。

## zlib.unzip(buf[, options], callback)
## zlib.unzipSync(buf[, options])

使用 Unzip 解压缩一个原始的 Buffer。

## Options

<!--type=misc-->

每个类都有一个选项对象。所有选项都是可选的。

注意：某些选项仅在压缩时有用，解压缩时会被忽略。

* flush (默认： `zlib.Z_NO_FLUSH`)
* chunkSize (默认： 16*1024)
* windowBits
* level (仅压缩有效)
* memLevel (仅压缩有效)
* strategy (仅压缩有效)
* dictionary (仅 deflate/inflate 有效, 默认为空字典)

参见 `deflateInit2` 和 `inflateInit2` 的描述，它们位于<http://zlib.net/manual.html#Advanced>。

## 使用内存调优

<!--type=misc-->

来自 `zlib/zconf.h`,修改为 node's 的用法:

deflate 的内存需求（单位：字节）：

    (1 << (windowBits+2)) +  (1 << (memLevel+9))

windowBits=15 的 128K 加  memLevel = 8 的 128K （缺省值），加其他对象的若干 KB。

例如，如果你想减少默认的内存需求（从 256K 减为 128k），设置选项：

    { windowBits: 14, memLevel: 7 }

当然这通常会降低压缩等级。

inflate 的内存需求（单位：字节）：

    1 << windowBits

windowBits=15 (默认值)32K 加其他对象的若干 KB。

这是除了内部输出缓冲外 chunkSize 的大小，缺省为 16K
  
影响 zlib 的压缩速度最大因素为 `level` 压缩级别。 `level` 越大，压缩率越高，速度越慢，`level` 越小，压缩率越小，速度会更快。

通常来说，使用更多的内存选项，意味着 node 必须减少对 zlib 掉哟过，因为可以在一个  `write` 操作里可以处理更多的数据。所以，这是另一个影响速度和内存使用率的因素，

## 常量

<!--type=misc-->

所有常量定义在 zlib.h ，也定义在 `require('zlib')` 。
  
通常的操作，基本用不到这些常量。写到文档里是想你不会对他们的存在感到惊讶。这个章节基本都来自 [zlib
documentation](http://zlib.net/manual.html#Constants)。更多细节参见 <http://zlib.net/manual.html#Constants>。

允许 flush 的值：

* `zlib.Z_NO_FLUSH`
* `zlib.Z_PARTIAL_FLUSH`
* `zlib.Z_SYNC_FLUSH`
* `zlib.Z_FULL_FLUSH`
* `zlib.Z_FINISH`
* `zlib.Z_BLOCK`
* `zlib.Z_TREES`

压缩/解压缩函数的返回值。负数代表错误，正数代表特殊但正常的事件：

* `zlib.Z_OK`
* `zlib.Z_STREAM_END`
* `zlib.Z_NEED_DICT`
* `zlib.Z_ERRNO`
* `zlib.Z_STREAM_ERROR`
* `zlib.Z_DATA_ERROR`
* `zlib.Z_MEM_ERROR`
* `zlib.Z_BUF_ERROR`
* `zlib.Z_VERSION_ERROR`

压缩级别：

* `zlib.Z_NO_COMPRESSION`
* `zlib.Z_BEST_SPEED`
* `zlib.Z_BEST_COMPRESSION`
* `zlib.Z_DEFAULT_COMPRESSION`

压缩策略：

* `zlib.Z_FILTERED`
* `zlib.Z_HUFFMAN_ONLY`
* `zlib.Z_RLE`
* `zlib.Z_FIXED`
* `zlib.Z_DEFAULT_STRATEGY`

data_type 字段的可能值：

* `zlib.Z_BINARY`
* `zlib.Z_TEXT`
* `zlib.Z_ASCII`
* `zlib.Z_UNKNOWN`

deflate 的压缩方法：

* `zlib.Z_DEFLATED`

初始化 zalloc, zfree, opaque：

* `zlib.Z_NULL`
