# 断言测试

    稳定性: 5 - 锁定

这个模块可用于应用的单元测试，通过 `require('assert')` 可以使用这个模块。


## assert.fail(actual, expected, message, operator)

使用参数 operator 测试参数 `actual` (实际值) 和 `expected` （期望值）是否相等。

## assert(value[, message]), assert.ok(value[, message])

测试参数 `value` 是否为 `true`,此函数和 `assert.equal(true, !!value, message);` 等价。

## assert.equal(actual, expected[, message])

判断实际值 `actual` 和期望值 `expected` 是否相等。

## assert.notEqual(actual, expected[, message])

判断实际值 `actual` 和期望值 `expected` 是否不等。

## assert.deepEqual(actual, expected[, message])

执行深度比较，判断实际值 `actual` 和期望值 `expected` 是否相等。

## assert.notDeepEqual(actual, expected[, message])

深度比较两个参数是否不相等。

## assert.strictEqual(actual, expected[, message])

深度比较两个参数是否相等。  

## assert.notStrictEqual(actual, expected[, message])

此函数使用操作符 ‘!==’ 严格比较是否两参数不相等。


## assert.throws(block[, error][, message])
声明一个 `block` 用来抛出错误（`error`）， `error`可以是构造函数，正则表达式或其他验证器。

使用构造函数验证实例：

```
    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      Error
    );
```

使用正则表达式验证错误信息：

```
    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      /value/
    );
```

用户自定义的错误验证器：

```
    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      function(err) {
        if ( (err instanceof Error) && /value/.test(err) ) {
          return true;
        }
      },
      "unexpected error"
    );
```

## assert.doesNotThrow(block[, message])

声明 `block` 不抛出错误，详细信息参见 `assert.throws`。

## assert.ifError(value)

判断参数 value 是否为 false ，如果是 true 抛出异常。通常用来测试回调中第一个参数 error。
