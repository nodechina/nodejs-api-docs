# Smalloc

    稳定性: 1 - 试验

## 类: smalloc

由简单内存分配器(处理扩展原始内存的分配)支持的缓存。Smalloc 有以下函数：

### smalloc.alloc(length[, receiver][, type])

* `length` {Number} `<= smalloc.kMaxLength`
* `receiver` {Object} 默认: `new Object`
* `type` {Enum} 默认: `Uint8`

返回  `receiver` 对象，包含分配的外部数组数据。如果没有传入`receiver` ，将会创建并返回一个新的对象。

这可用于创建你自己的类似 buffer 的类。不会设置其他属性，因此使用者需要跟踪其他所需信息（比如分配的长度）。

    function SimpleData(n) {
      this.length = n;
      smalloc.alloc(this.length, this);
    }

    SimpleData.prototype = { /* ... */ };

仅检查 `receiver` 是否是非数组的对象。因此，可以分配扩展数据数据，不仅是普通对象。

    function allocMe() { }
    smalloc.alloc(3, allocMe);

    // { [Function allocMe] '0': 0, '1': 0, '2': 0 }

v8 不支持给数组分配扩展数组对象，如果这么做，将会抛出。

你可以指定外部数组数据的类型。所有可用类型在 `smalloc.Types` 列出，例如：

    var doubleArr = smalloc.alloc(3, smalloc.Types.Double);

    for (var i = 0; i < 3; i++)
      doubleArr = i / 10;

    // { '0': 0, '1': 0.1, '2': 0.2 }

使用 `Object.freeze`, `Object.seal` 和 `Object.preventExtensions`不能冻结，封锁，阻止对象的使用扩展数据扩展。

### smalloc.copyOnto(source, sourceStart, dest, destStart, copyLength);

* `source` {Object} 分配了外部数组的对象
* `sourceStart` {Number} 负责的起始位置
* `dest` {Object} 分配了外部数组的对象
* `destStart` {Number} 拷贝到目标的起始位置
* `copyLength` {Number} 需要拷贝的长度

从一个外部数组拷贝内存到另外一个，所有的参数都必填，否则会抛出异常。

    var a = smalloc.alloc(4);
    var b = smalloc.alloc(4);

    for (var i = 0; i < 4; i++) {
      a[i] = i;
      b[i] = i * 2;
    }

    // { '0': 0, '1': 1, '2': 2, '3': 3 }
    // { '0': 0, '1': 2, '2': 4, '3': 6 }

    smalloc.copyOnto(b, 2, a, 0, 2);

    // { '0': 4, '1': 6, '2': 2, '3': 3 }


`copyOnto` 会在内部自动检测分配的长度，因此不必设置任何附加参数。

### smalloc.dispose(obj)

* `obj` Object

释放通过 `smalloc.alloc` 给对象分配的内存。

    var a = {};
    smalloc.alloc(3, a);

    // { '0': 0, '1': 0, '2': 0 }

    smalloc.dispose(a);

    // {}

有利于减轻垃圾回收器的负担，但是开发时候还是要小心。程序里可能会出现难以跟踪的错误。

    var a = smalloc.alloc(4);
    var b = smalloc.alloc(4);

    // perform this somewhere along the line
    smalloc.dispose(b);

    // now trying to copy some data out
    smalloc.copyOnto(b, 2, a, 0, 2);

    // now results in:
    // RangeError: copy_length > source_length

调用 `dispose()` ，对象依旧拥有外部数据，例如 `smalloc.hasExternalData()` 会返回 `true`。`dispose()`不支持缓存，如果传入将会抛出。

### smalloc.hasExternalData(obj)

* `obj` {Object}

如果 `obj`拥有外部分配的内存，返回`true`。

### smalloc.kMaxLength

可分配的最大数量。同样适用于缓存创建。

### smalloc.Types

外部数组的类型，包含：

* `Int8`
* `Uint8`
* `Int16`
* `Uint16`
* `Int32`
* `Uint32`
* `Float`
* `Double`
* `Uint8Clamped`
