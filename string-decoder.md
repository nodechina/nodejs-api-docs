# 字符串解码器

    稳定性: 3 - 稳定

通过 `require('string_decoder')` ，可以使用这个模块。字符串解码器（StringDecoder）将缓存（buffer）解码为字符串。这是 `buffer.toString()` 的简单接口，提供了 utf8 支持。

    var StringDecoder = require('string_decoder').StringDecoder;
    var decoder = new StringDecoder('utf8');

    var cent = new Buffer([0xC2, 0xA2]);
    console.log(decoder.write(cent));

    var euro = new Buffer([0xE2, 0x82, 0xAC]);
    console.log(decoder.write(euro));

## Class: StringDecoder

接受一个参数 `encoding`，默认值 `utf8`。

### decoder.write(buffer)

返回解码后的字符串。

### decoder.end()

返回 buffer 里剩下的末尾字节。
