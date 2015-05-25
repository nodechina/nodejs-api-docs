# Punycode

    稳定性: 2 - 不稳定

[Punycode.js](http://mths.be/punycode) 从 Node.js v0.6.2+ 开始内置. 使用
`require('punycode')` 来访问。 (要在其他 Node.js 版本中访问,先用 npm 来 `punycode` 安装)。

## punycode.decode(string)

将一个纯 ASCII 的 Punycode 字符串转换为 Unicode  字符串。

    // decode domain name parts
    punycode.decode('maana-pta'); // 'mañana'
    punycode.decode('--dqo34k'); // '☃-⌘'

## punycode.encode(string)

将一个纯 Unicode  Punycode 字符串转换为 纯 ASCII 字符串。

    // encode domain name parts
    punycode.encode('mañana'); // 'maana-pta'
    punycode.encode('☃-⌘'); // '--dqo34k'

## punycode.toUnicode(domain)

将一个表示域名的 Punycode 字符串转换为 Unicode。只有域名中的 Punycode 部分会被转换，也就是说你也可以在一个已经转换为 Unicode 的字符串上调用它。

    // decode domain names
    punycode.toUnicode('xn--maana-pta.com'); // 'mañana.com'
    punycode.toUnicode('xn----dqo34k.com'); // '☃-⌘.com'

## punycode.toASCII(domain)
将一个表示域名的 Unicode 字符串转换为 Punycode。只有域名中的非 ASCII 部分会被转换，也就是说你也可以在一个已经转换为 ASCII 的字符串上调用它。

    // encode domain names
    punycode.toASCII('mañana.com'); // 'xn--maana-pta.com'
    punycode.toASCII('☃-⌘.com'); // 'xn----dqo34k.com'

## punycode.ucs2

### punycode.ucs2.decode(string)

创建一个包含字符串中每个 Unicode 符号的数字编码点的数组，。由于 [JavaScript uses UCS-2
internally](http://mathiasbynens.be/notes/javascript-encoding) 在内部使用 UCS-2， 该函数会按照 UTF-16 将一对代半数（UCS-2 暴露的单独的字符）转换为单独一个编码点。


    punycode.ucs2.decode('abc'); // [0x61, 0x62, 0x63]
    // surrogate pair for U+1D306 tetragram for centre:
    punycode.ucs2.decode('\uD834\uDF06'); // [0x1D306]

### punycode.ucs2.encode(codePoints)

创建以一组数字编码点为基础一个字符串。

    punycode.ucs2.encode([0x61, 0x62, 0x63]); // 'abc'
    punycode.ucs2.encode([0x1D306]); // '\uD834\uDF06'

## punycode.version

表示当前 Punycode.js 版本数字的字符串。
