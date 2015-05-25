# 路径

    稳定性: 3 - 稳定

本模块包含一系列处理和转换文件路径的工具集。基本所有的反复都仅对字符串转换。文件系统不会检查路径是否有效。


通过 `require('path')` 来访问这个模块。提供了以下方法：

## path.normalize(p)

规范化路径，注意`'..'` 和 `'.'`。

发现多个斜杠时，会替换成一个斜杠。当路径末尾包含一个斜杠时，保留。Windows 系统使用反斜杠。

例如:

    path.normalize('/foo/bar//baz/asdf/quux/..')
    // returns
    '/foo/bar/baz/asdf'

## path.join([path1][, path2][, ...])

连接所有的参数，并规范化输出路径。

参数必须是字符串。在 v0.8 版本，非字符参数会被忽略。v0.10之后的版本后抛出异常。

例如:

    path.join('/foo', 'bar', 'baz/asdf', 'quux', '..')
    // returns
    '/foo/bar/baz/asdf'

    path.join('foo', {}, 'bar')
    // throws exception
    TypeError: Arguments to path.join must be strings

## path.resolve([from ...], to)

将 `to` 参数解析为绝对路径。

如果参数 `to` 不是一个相对于参数 `from` 的绝对路径，`to`会添加到 `from` 右侧，直到找到一个绝对路径为止。如果使用所有  `from`  参数后，还是没有找到绝对路径，将会使用当前工作目录。返回的路径已经规范化过，并且去掉了尾部的斜杠（除非是根目录）。非字符串的参数会被忽略。

另一种思路就是在shell里执行一系列的  `cd` 命令。

    path.resolve('foo/bar', '/tmp/file/', '..', 'a/../subfile')

类似于:

    cd foo/bar
    cd /tmp/file/
    cd ..
    cd a/../subfile
    pwd

不同点是，不同的路径不需要存在的，也可能是文件。

例如:

    path.resolve('/foo/bar', './baz')
    // returns
    '/foo/bar/baz'

    path.resolve('/foo/bar', '/tmp/file/')
    // returns
    '/tmp/file'

    path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif')
    // if currently in /home/myself/node, it returns
    '/home/myself/node/wwwroot/static_files/gif/image.gif'

## path.isAbsolute(path)

判断参数 `path` 是否是绝对路径。一个绝对路径解析后都会指向相同的位置，无论当前的工作目录在哪里。

Posix 例子:

    path.isAbsolute('/foo/bar') // true
    path.isAbsolute('/baz/..')  // true
    path.isAbsolute('qux/')     // false
    path.isAbsolute('.')        // false

Windows 例子:

    path.isAbsolute('//server')  // true
    path.isAbsolute('C:/foo/..') // true
    path.isAbsolute('bar\\baz')   // false
    path.isAbsolute('.')         // false

## path.relative(from, to)

解决从 `from` 到 `to`的相对路径。

有时我们会有2个绝对路径，需要从中找到相对目录。这是 `path.resolve` 的逆实现：

    path.resolve(from, path.relative(from, to)) == path.resolve(to)

例如:

    path.relative('C:\\orandea\\test\\aaa', 'C:\\orandea\\impl\\bbb')
    // returns
    '..\\..\\impl\\bbb'

    path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb')
    // returns
    '../../impl/bbb'

## path.dirname(p)

返回路径 `p` 所在的目录。和 Unix `dirname` 命令类似。

例如：

    path.dirname('/foo/bar/baz/asdf/quux')
    // returns
    '/foo/bar/baz/asdf'

## path.basename(p[, ext])

返回路径的最后一个部分。和 Unix `basename` 命令类似。

例如：

    path.basename('/foo/bar/baz/asdf/quux.html')
    // returns
    'quux.html'

    path.basename('/foo/bar/baz/asdf/quux.html', '.html')
    // returns
    'quux'

## path.extname(p)

返回路径 `p` 的扩展名，从最后一个 '.' 到字符串的末尾。如果最后一个部分没有 '.' ，或者路径是以 '.' 开头，则返回空字符串。例如。  

    path.extname('index.html')
    // returns
    '.html'

    path.extname('index.coffee.md')
    // returns
    '.md'

    path.extname('index.')
    // returns
    '.'

    path.extname('index')
    // returns
    ''

## path.sep

特定平台的文件分隔符，`'\\'` 或 `'/'`。

*nix 上的例子:

    'foo/bar/baz'.split(path.sep)
    // returns
    ['foo', 'bar', 'baz']

Windows 的例子:

    'foo\\bar\\baz'.split(path.sep)
    // returns
    ['foo', 'bar', 'baz']

## path.delimiter

特定平台的分隔符, `;` or `':'`.

 *nix 上的例子:

    console.log(process.env.PATH)
    // '/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin'

    process.env.PATH.split(path.delimiter)
    // returns
    ['/usr/bin', '/bin', '/usr/sbin', '/sbin', '/usr/local/bin']

Windows 例子:

    console.log(process.env.PATH)
    // 'C:\Windows\system32;C:\Windows;C:\Program Files\nodejs\'

    process.env.PATH.split(path.delimiter)
    // returns
    ['C:\\Windows\\system32', 'C:\\Windows', 'C:\\Program Files\\nodejs\\']

## path.parse(pathString)

返回路径字符串的对象。

*nix 上的例子:

    path.parse('/home/user/dir/file.txt')
    // returns
    {
        root : "/",
        dir : "/home/user/dir",
        base : "file.txt",
        ext : ".txt",
        name : "file"
    }

Windows 例子:

    path.parse('C:\\path\\dir\\index.html')
    // returns
    {
        root : "C:\\",
        dir : "C:\\path\\dir",
        base : "index.html",
        ext : ".html",
        name : "index"
    }

## path.format(pathObject)

从对象中返回路径字符串，和 `path.parse` 相反。

    path.format({
        root : "/",
        dir : "/home/user/dir",
        base : "file.txt",
        ext : ".txt",
        name : "file"
    })
    // returns
    '/home/user/dir/file.txt'

## path.posix

提供上述 `path` 路径访问，不过总是以 posix 兼容的方式交互。

## path.win32

提供上述 `path` 路径访问，不过总是以 win32 兼容的方式交互。
