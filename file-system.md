# 文件系统

    稳定性: 3 - 稳定

文件系统模块是一个封装了标准的 POSIX 文件 I/O 操作的集合。通过 `require('fs')` 使用这个模块。所有的方法都有同步和异步两种模式。


异步方法最后一个参数都是回调函数，这个回调的参数取决于方法，不过第一个参数一般都是异常。如果操作成功，那么第一个参数就是 `null` 或 `undefined`。

当使用一个同步操作的时候，任意的异常都立即抛出，可以用 try/catch 来处理异常，使得程序正常运行。

这是异步操作的例子:

    var fs = require('fs');

    fs.unlink('/tmp/hello', function (err) {
      if (err) throw err;
      console.log('successfully deleted /tmp/hello');
    });

这是同步操作的例子:

    var fs = require('fs');

    fs.unlinkSync('/tmp/hello');
    console.log('successfully deleted /tmp/hello');

异步方法不能保证操作顺序，因此下面的例子很容易出错：

    fs.rename('/tmp/hello', '/tmp/world', function (err) {
      if (err) throw err;
      console.log('renamed complete');
    });
    fs.stat('/tmp/world', function (err, stats) {
      if (err) throw err;
      console.log('stats: ' + JSON.stringify(stats));
    });

可能先执行了 `fs.stat` 方法。正确的方法：

    fs.rename('/tmp/hello', '/tmp/world', function (err) {
      if (err) throw err;
      fs.stat('/tmp/world', function (err, stats) {
        if (err) throw err;
        console.log('stats: ' + JSON.stringify(stats));
      });
    });

在繁忙的进程里，强烈建议使用异步方法。同步方法会阻塞整个进程，直到方法完成。

可能会用到相对路径，路径是相对 `process.cwd() ` 来说的。

大部分 fs 函数会忽略回调参数，如果忽略，将会用默认函数抛出异常。如果想得到原调用点的堆栈信息，需要设置环境变量 NODE_DEBUG；

    $ cat script.js
    function bad() {
      require('fs').readFile('/');
    }
    bad();

    $ env NODE_DEBUG=fs node script.js
    fs.js:66
            throw err;
                  ^
    Error: EISDIR, read
        at rethrow (fs.js:61:21)
        at maybeCallback (fs.js:79:42)
        at Object.fs.readFile (fs.js:153:18)
        at bad (/path/to/script.js:2:17)
        at Object.<anonymous> (/path/to/script.js:5:1)
        <etc.>


## fs.rename(oldPath, newPath, callback)

异步函数 rename(2)。回调函数只有一个参数：可能出现的异常。  

## fs.renameSync(oldPath, newPath)

同步函数 rename(2)。 返回 `undefined`。

## fs.ftruncate(fd, len, callback)

异步函数 ftruncate(2)。 回调函数只有一个参数：可能出现的异常。

## fs.ftruncateSync(fd, len)

同步函数 ftruncate(2)。 返回 `undefined`。

## fs.truncate(path, len, callback)

异步函数 truncate(2)。 回调函数只有一个参数：可能出现的异常。 文件描述符也可以作为第一个参数，如果这种情况，调用 `fs.ftruncate()` 。

## fs.truncateSync(path, len)

同步函数 truncate(2)。 返回 `undefined`。

## fs.chown(path, uid, gid, callback)

异步函数 chown(2)。 回调函数只有一个参数：可能出现的异常。

## fs.chownSync(path, uid, gid)

同步函数 chown(2)。 返回 `undefined`。

## fs.fchown(fd, uid, gid, callback)

异步函数 fchown(2)。 回调函数只有一个参数：可能出现的异常。

## fs.fchownSync(fd, uid, gid)

同步函数 fchown(2)。 返回 `undefined`。

## fs.lchown(path, uid, gid, callback)

异步函数 lchown(2)。 回调函数只有一个参数：可能出现的异常。

## fs.lchownSync(path, uid, gid)

同步函数 lchown(2)。 返回 `undefined`。

## fs.chmod(path, mode, callback)

异步函数 chmod(2)。回调函数只有一个参数：可能出现的异常。

## fs.chmodSync(path, mode)

同步函数 chmod(2)。 返回 `undefined`。

## fs.fchmod(fd, mode, callback)

异步函数 fchmod(2)。 回调函数只有一个参数：可能出现的异常。

## fs.fchmodSync(fd, mode)

同步函数 fchmod(2)。 返回 `undefined`。

## fs.lchmod(path, mode, callback)

异步函数 lchmod(2)。 回调函数只有一个参数：可能出现的异常。

仅在 Mac OS X 可用。

## fs.lchmodSync(path, mode)

同步函数 lchmod(2)。 返回 `undefined`。

## fs.stat(path, callback)

异步函数 stat(2)。 回调函数有两个参数： (err, stats) ，其中 `stats` 是一个 `fs.Stats` 对象。 详情请参考 fs.Stats。  

## fs.lstat(path, callback)

异步函数 lstat(2)。 回调函数有两个参数： (err, stats) ，其中 `stats` 是一个 `fs.Stats` 对象。 `lstat()` 与 `stat()` 基本相同, 区别在于，如果 `path` 是链接，读取的是链接本身，而不是它所链接到的文件。

## fs.fstat(fd, callback)

异步函数 fstat(2)。 回调函数有两个参数： (err, stats) ，其中 `stats` 是一个 `fs.Stats` 对象。 

## fs.statSync(path)

同步函数 stat(2)。 返回 `fs.Stats` 实例。

## fs.lstatSync(path)

同步函数 lstat(2)。 返回 `fs.Stats` 实例。

## fs.fstatSync(fd)

同步函数 fstat(2)。 返回 `fs.Stats` 实例。

## fs.link(srcpath, dstpath, callback)

异步函数 link(2)。 回调函数只有一个参数：可能出现的异常。

## fs.linkSync(srcpath, dstpath)

同步函数 link(2)。 返回 `undefined`。

## fs.symlink(srcpath, dstpath[, type], callback)

异步函数 symlink(2)。 回调函数只有一个参数：可能出现的异常。

`type` 可能是 `'dir'`, `'file'`, 或 `'junction'` (默认 `'file'`) ，仅在 Windows（不考虑其他系统）有效。注意， Windows junction 要求目的地址需要绝对的。当使用 `'junction'` 的时候，`destination` 参数将会自动转换为绝对路径。

## fs.symlinkSync(srcpath, dstpath[, type])

同步函数 symlink(2)。 返回 `undefined`。

## fs.readlink(path, callback)

异步函数 readlink(2)。 回调函数有2个参数 `(err, linkString)`.

## fs.readlinkSync(path)

同步函数 readlink(2)。 返回符号链接的字符串值。

## fs.realpath(path[, cache], callback)

异步函数 realpath(2)。 回调函数有2个参数 `(err,resolvedPath)`。可以使用 `process.cwd`来解决相对路径问题。

例如:

    var cache = {'/etc':'/private/etc'};
    fs.realpath('/etc/passwd', cache, function (err, resolvedPath) {
      if (err) throw err;
      console.log(resolvedPath);
    });

## fs.realpathSync(path[, cache])

同步函数 realpath(2)。 返回解析出的路径。

## fs.unlink(path, callback)

异步函数 unlink(2)。 回调函数只有一个参数：可能出现的异常.

## fs.unlinkSync(path)

同步函数 unlink(2)。 返回 `undefined`。

## fs.rmdir(path, callback)

异步函数 rmdir(2)。 回调函数只有一个参数：可能出现的异常.

## fs.rmdirSync(path)

同步函数 rmdir(2)。 返回 `undefined`。

## fs.mkdir(path[, mode], callback)

异步函数 mkdir(2)。 回调函数只有一个参数：可能出现的异常. `mode` 默认s to `0777`.

## fs.mkdirSync(path[, mode])

同步函数 mkdir(2)。 返回 `undefined`。

## fs.readdir(path, callback)

异步函数 readdir(3)。  读取文件夹的内容。回调有2个参数 `(err, files) `files  是文件夹里除了名字为`，`'.'` 和 `'..'`之外的所有文件名。

## fs.readdirSync(path)

同步函数 readdir(3)。 返回除了文件名为 `'.'` 和 `'..'`之外的所有文件.

## fs.close(fd, callback)

异步函数 close(2)。  回调函数只有一个参数：可能出现的异常.

## fs.closeSync(fd)

同步函数 close(2)。 返回 `undefined`。

## fs.open(path, flags[, mode], callback)

异步函数 file open. 参见 open(2)。 `flags` 是:

* `'r'` - 以只读模式打开.如果文件不存在，抛出异常。

* `'r+'` -以读写模式打开.如果文件不存在，抛出异常。

* `'rs'` - 同步的，以只读模式打开. 指令绕过操作系统直接使用本地文件系统缓存。
  这个功能主要用来打开 NFS 挂载的文件，因为它能让你跳过可能过时的本地缓存。如果对 I/O 性能很在乎，就不要使用这个标志位。

  这里不是调用 `fs.open()` 变成同步阻塞请求，如果你想要这样，可以调用 `fs.openSync()`。

* `'rs+'` - 同步模式下以读写方式打开文件。注意事项参见 `'rs'`.

* `'w'` - 以只写模式打开。文件会被创建 (如果文件不存在) 或者覆盖 (如果存在)。

* `'wx'` - 和 `'w'`类似，如果文件存储操作失败

* `'w+'` - 以可读写方式打开。文件会被创建 (如果文件不存在) 或者覆盖 (如果存在)  

* `'wx+'` - 和 `'w+'`类似，如果文件存储操作失败。

* `'a'` - 以附加的形式打开。如果文件不存在则创建一个。

* `'ax'` - 和 `'a'`类似，如果文件存储操作失败。

* `'a+'` - 以只读和附加的形式打开文件.若文件不存在，则会建立该文件

* `'ax+'` - 和 `'a+'`类似，如果文件存储操作失败.

如果文件存在，参数`mode` 设置文件模式 (permission 和 sticky bits)。 默认是 `0666`, 可读写.

回调有2个参数 `(err, fd)`.

排除标记 `'x'` (对应 open(2)的`O_EXCL` 标记) 保证 `path` 是新创建的。在 POSIX 系统里，即使文件不存在，也会被认定为文件存在。 排除标记不能确定在网络文件系统中是否有效。


Linux系统里，无法对以追加模式打开的文件进行指定位置写。系统核心忽略了位置参数，每次把数据写到文件的最后。

## fs.openSync(path, flags[, mode])

`fs.open()` 的同步版本. 返回整数形式的文件描述符。.

## fs.utimes(path, atime, mtime, callback)

改变指定路径文件的时间戳。

## fs.utimesSync(path, atime, mtime)

`fs.utimes()` 的同步版本. 返回 `undefined`。


## fs.futimes(fd, atime, mtime, callback)

改变传入的文件描述符指向文件的时间戳。

## fs.futimesSync(fd, atime, mtime)

`fs.futimes()` 的同步版本. 返回 `undefined`。

## fs.fsync(fd, callback)

异步函数 fsync(2)。 回调函数只有一个参数：可能出现的异常.

## fs.fsyncSync(fd)

同步 fsync(2)。 返回 `undefined`。

## fs.write(fd, buffer, offset, length[, position], callback)

将  `buffer` 写到 `fd`指定的文件里。

参数 `offset` 和 `length` 确定写哪个部分的缓存。

参数 `position` 是要写入的文件位置。如果  `typeof position !== 'number'`，将会在当前位置写入。参见 pwrite(2)。

回调函数有三个参数 `(err, written, buffer)`，`written` 指定 `buffer`的多少字节用来写。

注意，如果 `fs.write` 的回调还没执行，就多次调用 `fs.write` ，这样很不安全。因此，推荐使用 `fs.createWriteStream` 。

Linux系统里，无法对以追加模式打开的文件进行指定位置写。系统核心忽略了位置参数，每次把数据写到文件的最后。


## fs.write(fd, data[, position[, encoding]], callback)

将  `buffer` 写到 `fd`指定的文件里。如果 `data` 不是 buffer,那么它就会被强制转换为字符串。

参数 `position` 是要写入的文件位置。如果  `typeof position !== 'number'`，将会在当前位置写入。参见 pwrite(2)。

参数 `encoding` ：字符串的编码方式.

回调函数有三个参数 `(err, written, buffer)`，`written` 指定 `buffer`的多少字节用来写。注意写入的字节（bytes）和字符（string characters）不同。参见[Buffer.byteLength](buffer.html#buffer_class_method_buffer_bytelength_string_encoding)。

和写入 `buffer` 不同，必须写入整个字符串，不能截取字符串。这是因为返回的字节的位移跟字符串的位移是不一样的。

注意，如果 `fs.write` 的回调还没执行，就多次调用 `fs.write` ，这样很不安全。因此，推荐使用 `fs.createWriteStream` 

Linux系统里，无法对以追加模式打开的文件进行指定位置写。系统核心忽略了位置参数，每次把数据写到文件的最后。


## fs.writeSync(fd, buffer, offset, length[, position])

## fs.writeSync(fd, data[, position[, encoding]])

`fs.write()`  的同步版本. 返回要写的bytes数.

## fs.read(fd, buffer, offset, length, position, callback)

读取 `fd`指定文件的数据。

`buffer` 是缓冲区，数据将会写入到这里.

`offset` 写入的偏移量

`length` 需要读的文件长度

`position` 读取的文件起始位置，如果是 `position` 是  `null`， 将会从当前位置读。

回调函数有3个参数, `(err, bytesRead, buffer)`.

## fs.readSync(fd, buffer, offset, length, position)

`fs.read`  的同步版本. 返回 `bytesRead` 的数量.

## fs.readFile(filename[, options], callback)

* `filename` {String}
* `options` {Object}
  * `encoding` {String | Null} 默认 = `null`
  * `flag` {String} 默认 = `'r'`
* `callback` {Function}

异步读取整个文件的内容。例如：

    fs.readFile('/etc/passwd', function (err, data) {
      if (err) throw err;
      console.log(data);
    });

回调函数有2个参数 `(err, data)`, 参数 `data` 是文件的内容。
如果没有指定参数 `encoding`, 返回原生 buffer


## fs.readFileSync(filename[, options])

`fs.readFile`  的同步版本. 返回整个文件的内容.

如果没有指定参数 `encoding`, 返回buffer。


## fs.writeFile(filename, data[, options], callback)

* `filename` {String}
* `data` {String | Buffer}
* `options` {Object}
  * `encoding` {String | Null} 默认 = `'utf8'`
  * `mode` {Number} 默认 = `438` (aka `0666` in Octal)
  * `flag` {String} 默认 = `'w'`
* `callback` {Function}

异步写文件，如果文件已经存在则替换。 `data` 可以是缓存或者字符串。  

如果参数 `data` 是 buffer，会忽略参数  `encoding`。默认值是 `'utf8'`。

列如:

    fs.writeFile('message.txt', 'Hello Node', function (err) {
      if (err) throw err;
      console.log('It\'s saved!');
    });

## fs.writeFileSync(filename, data[, options])

`fs.writeFile`  的同步版本. 返回 `undefined`。

## fs.appendFile(filename, data[, options], callback)

* `filename` {String}
* `data` {String | Buffer}
* `options` {Object}
  * `encoding` {String | Null} 默认 = `'utf8'`
  * `mode` {Number} 默认 = `438` (aka `0666` in Octal)
  * `flag` {String} 默认 = `'a'`
* `callback` {Function}

异步的给文件添加数据，如果文件不存在，就创建一个。 `data` 可以是缓存或者字符串。  

例如:

    fs.appendFile('message.txt', 'data to append', function (err) {
      if (err) throw err;
      console.log('The "data to append" was appended to file!');
    });

## fs.appendFileSync(filename, data[, options])

`fs.appendFile` 的同步版本. 返回 `undefined`。

## fs.watchFile(filename[, options], listener)

    稳定性: 2 - 不稳定。  尽可能的用 fs.watch 来替换。

监视 `filename` 文件的变化。每当文件被访问的时候都会调用`listener`。

第二个参数可选。如果有，它必须包含两个 boolean 参数（`persistent` 和 `interval`）的对象。 `persistent` 指定文件被监视时进程是否继续运行。 `interval`指定了查询文件的间隔，以毫秒为单位。缺省值为{ persistent: true, interval: 5007 }。

listener 有两个参数，第一个为文件现在的状态，第二个为文件的前一个状态：

    fs.watchFile('message.text', function (curr, prev) {
      console.log('the current mtime is: ' + curr.mtime);
      console.log('the previous mtime was: ' + prev.mtime);
    });

listener中的文件状态对象类型为 fs.Stat。

如果想修改文件时被通知，而不是访问的时候就通知，可以比较 `curr.mtime` 和 `prev.mtime`。

## fs.unwatchFile(filename[, listener])

    稳定性: 2 - 不稳定. 尽可能的用 fs.watch 来替换。


停止监视 `filename` 文件的变化。如果指定了 `listener`，那只会移除这个 `listener`。否则，移除所有的 listener，并会停止监视 `filename`。

调用 `fs.unwatchFile()`停止监视一个没被监视的文件，不会触发错误，而会发生一个no-op。 

## fs.watch(filename[, options][, listener])

    稳定性: 2 - 不稳定.

观察 `filename` 指定的文件或文件夹的改变。返回对象是 [fs.FSWatcher](#fs_class_fs_fswatcher)。

第二个参数可选。如果有，它必须是包含两个 boolean 参数（`persistent` 和 `recursive`）的对象。 `persistent` 指定文件被监视时进程是否继续运行。 `recursive`表明是监视所有的子文件夹还是当前文件夹，这个参数只有监视对象是文件夹时才有效，而且仅在支持的系统里有效（参见下面注意事项）。

默认值 `{ persistent: true, recursive: false }`.

回调函数有2个参数 `(event, filename)`。`event` 是 `rename` 或 `change`。`filename` 是触发事件的文件名。


### 注意事项

<!--type=misc-->

`fs.watch` API 不是 100% 的跨平台兼容，可能在某些情况下不可用。

`recursive` 参数仅在 OS X 上可用。仅 `FSEvents` 支持这个类型文件的监视，所以未来也不太可能有新的平台加入。

#### 可用性

<!--type=misc-->

这些特性依赖于底层系统提供文件系统变动的通知。

* Linux 系统,使用 `inotify`.
* BSD 系统,使用 `kqueue`.
* OS X,文件使用 `kqueue` ，文件夹使用 `FSEvents`.
* SunOS 系统(包括 Solaris 和 SmartOS),使用 `event ports`.
* Windows 系统, 依赖与 `ReadDirectoryChangesW`.

如果底层系统函数不可用，那么`fs.watch` 就无法工作。例如，监视网络文件系统(NFS, SMB, 等)经常不能用。你仍然可以用 `fs.watchFile` 查询，但是会比较慢，且不可靠。  

#### 文件名参数

<!--type=misc-->

回调函数中提供文件名参数，不是每个平台都能用（Linux 和 Windows 就不行）。即使在可用的平台，也不能保证都能提供。所以不要假设回调函数中 `filename` 参数有效，要在代码里添加一些为空的逻辑判断。

    fs.watch('somedir', function (event, filename) {
      console.log('event is: ' + event);
      if (filename) {
        console.log('filename provided: ' + filename);
      } else {
        console.log('filename not provided');
      }
    });

## fs.exists(path, callback)

判断文件是否存在，回调函数参数是 bool 值。例如：

    fs.exists('/etc/passwd', function (exists) {
      util.debug(exists ? "it's there" : "no passwd!");
    });

`fs.exists()` 是老版本的函数，因此在代码里不要用。

另外，打开文件前判断是否存在有漏洞，在`fs.exists()` 和 `fs.open()` 调用中间，另外一个进程有可能已经移除了文件。最好用 `fs.open()` 来打开文件，根据回调函数来判断是否有错误。

`fs.exists()` 未来会被移除。

## fs.existsSync(path)

 `fs.exists()` 的同步版本. 如果文件存在返回 `true`, 否则返回`false`。

`fs.existsSync()` 未来会被移除。

## fs.access(path[, mode], callback)

测试由参数 `path` 指向的文件的用户权限。可选参数 `mode` 为整数，它表示需要检查的权限。下面列出了所有值。`mode` 可以是单个值，或者可以通过或运算，掩码运算实现多个权限检查。

- `fs.F_OK` - 文件是对于进程可见，可以用来检查文件是否存在。参数 `mode` 的默认值。  
- `fs.R_OK` - 文件对于进程是否可读。
- `fs.W_OK` - 文件对于进程是否可写。
- `fs.X_OK` - 文件对于进程是否可执行。（Windows系统不可用，执行效果等同`fs.F_OK`）  

第三个参数是回调函数。如果检查失败，回调函数的参数就是响应的错误。下面的例子检查文件`/etc/passwd` 是否能被当前的进程读写。

    fs.access('/etc/passwd', fs.R_OK | fs.W_OK, function(err) {
      util.debug(err ? 'no access!' : 'can read/write');
    });

## fs.accessSync(path[, mode])

`fs.access`  的同步版本. 如果发生错误抛出异常，否则不做任何事情。  

## 类: fs.Stats

`fs.stat()`, `fs.lstat()` 和 `fs.fstat()` 以及同步版本的返回对象。

 - `stats.isFile()`
 - `stats.isDirectory()`
 - `stats.isBlockDevice()`
 - `stats.isCharacterDevice()`
 - `stats.isSymbolicLink()` (only valid with  `fs.lstat()`)
 - `stats.isFIFO()`
 - `stats.isSocket()`

对普通文件使用 `util.inspect(stats)`，返回的字符串和下面类似：

    { dev: 2114,
      ino: 48064969,
      mode: 33188,
      nlink: 1,
      uid: 85,
      gid: 100,
      rdev: 0,
      size: 527,
      blksize: 4096,
      blocks: 8,
      atime: Mon, 10 Oct 2011 23:24:11 GMT,
      mtime: Mon, 10 Oct 2011 23:24:11 GMT,
      ctime: Mon, 10 Oct 2011 23:24:11 GMT,
      birthtime: Mon, 10 Oct 2011 23:24:11 GMT }

 `atime`, `mtime`, `birthtime`, 和 `ctime` 都是 [Date][MDN-Date] 的实例，需要使用合适的方法来比较这些值。通常使用 [getTime()][MDN-Date-getTime] 来获取时间戳（毫秒，从 _1 January 1970 00:00:00 UTC_ 开始算），这个整数基本能满足任何比较条件。也有一些其他方法来显示额外信息。更多参见[MDN JavaScript Reference][MDN-Date]  

[MDN-Date]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Date
[MDN-Date-getTime]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Date/getTime

### Stat Time Values

状态对象（stat object）有以下语义：

* `atime` 访问时间 - 文件的最后访问时间.  `mknod(2)`, `utimes(2)`, 和 `read(2)` 等系统调用可以改变.
* `mtime` 修改时间 - 文件的最后修改时间. `mknod(2)`, `utimes(2)`, 和 `write(2)`等系统调用可以改变.
* `ctime` 改变时间 - 文件状态(inode)的最后修改时间. `chmod(2)`, `chown(2)`,`link(2)`, `mknod(2)`, `rename(2)`, `unlink(2)`, `utimes(2)`, `read(2)`, 和 `write(2)`等系统调用可以改变.
* `birthtime` "Birth Time" -  文件创建时间，文件创建时生成。 在一些不提供文件 birthtime 的文件系统中, 这个字段会使用 ctime 或 1970-01-01T00:00Z (ie, unix epoch timestamp 0)来填充。 在 Darwin 和其他 FreeBSD 系统变体中, 也将 atime 显式地设置成比它现在的 birthtime 更早的一个时间值，这个过程使用了 utimes(2) 系统调用。

在 Node v0.12 版本之前, Windows 系统里 ctime 有 birthtime 值. 注意在v.0.12版本中, ctime 不再是"creation time", 而且在Unix系统中，他一直都不是。

## fs.createReadStream(path[, options])

返回可读流对象 (见 `Readable Stream`)。

`options` 默认值如下:

    { flags: 'r',
      encoding: null,
      fd: null,
      mode: 0666,
      autoClose: true
    }

参数 `options` 提供 `start` 和 `end` 位置来读取文件的特定范围内容，而不是整个文件。`start` 和 `end` 都在文件范围里，并从 0 开始，	`encoding` 是 `'utf8'`, `'ascii'`, 或 `'base64'`。

如果给了  `fd` 值， `ReadStream` 将会忽略 `path` 参数，而使用文件描述，这样不会触发任何 `open` 事件。

如果 `autoClose` 为 false，即使发生错误文件也不会关闭，需要你来负责关闭，避免文件描述符泄露。如果 `autoClose` 是 true（默认值），遇到 `error` 或 `end` ，文件描述符将会自动关闭。  

例如，从100个字节的文件里，读取最少10个字节：

    fs.createReadStream('sample.txt', {start: 90, end: 99});


## Class: fs.ReadStream

`ReadStream` 是 [Readable Stream](stream.html#stream_class_stream_readable)。

### Event: 'open'

* `fd` {Integer} ReadStream 所使用的文件描述符。

当创建文件的ReadStream时触发。


## fs.createWriteStream(path[, options])

返回一个新的写对象 (参见 `Writable Stream`)。

`options` 是一个对象，默认值:

    { flags: 'w',
      encoding: null,
      fd: null,
      mode: 0666 }

options 也可以包含一个 start 选项，在指定文件中写入数据开始位置。 修改而不替换文件需要 flags 的模式指定为 r+ 而不是默值的 w.

和之前的 `ReadStream` 类似，如果 `fd` 不为空，`WriteStream` 将会忽略 `path` 参数，转而使用文件描述，这样不会触发任何 `open` 事件。


## 类: fs.WriteStream

`WriteStream` 是 [Writable Stream](stream.html#stream_class_stream_writable)。

### Event: 'open'

* `fd` {Integer} WriteStream 所用的文件描述符

打开 WriteStream file 时触发。

### file.bytesWritten

目前写入的字节数，不含等待写入的数据。

## Class: fs.FSWatcher

`fs.watch()` 返回的对象就是这个类.

### watcher.close()

停止观察 `fs.FSWatcher` 对象中的更改。

### Event: 'change'

* `event` {String} fs 改变的类型
* `filename` {String} 改变的文件名 (if relevant/available)

当监听的文件或文件夹改变的时候触发，参见[fs.watch](#fs_fs_watch_filename_options_listener)。

### Event: 'error'

* `error` {Error object}

错误发生时触发。
