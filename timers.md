# 定时器

    稳定性: 5 - 锁定

所有的定时器函数都是全局的。不需要通过 `require()` 就可以访问。

## setTimeout(callback, delay[, arg][, ...])

`delay` 毫秒之后执行 `callback`。返回 `timeoutObject` 对象，可能会用来 `clearTimeout()`。你也可以给回调函数传参数。

需要注意，你的回调函数可能不会非常准确的在 `delay` 毫秒后执行，Node.js 不保证回调函数的精确时间和执行顺序。回调函数会尽量的靠近指定的时间。

## clearTimeout(timeoutObject)

阻止一个  timeout 被触发。

## setInterval(callback, delay[, arg][, ...])

每隔 `delay` 毫秒就重复执行 `callback` 。返回 `timeoutObject` 对象，可能会用来 `clearTimeout()`。你也可以给回调函数传参数。

## clearInterval(intervalObject)

阻止一个  interval 被触发。

## unref()

 `setTimeout` 和 `setInterval` 所返回的值，拥有 `timer.unref()` 方法，它能让你创建一个活动的定时器，但是它所在的事件循环中如果仅剩它一个计时器，将不会保持程序运行。如果计时器已经调用了 `unref`，再次调用将无效。

在 `setTimeout` 场景中，当你使用 `unref` 并创建了一个独立定时器它将会唤醒事件循环。创建太多的这样的东西会影响事件循环性能，所以谨慎使用。


## ref()

如果你之前已经使用 `unref()` 一个定时器，就可以使用 `ref()` 来明确的请求定时器保持程序打开状态。果计时器已经调用了 `ref（）`，再次调用将无效。

## setImmediate(callback[, arg][, ...])

在 `setTimeout` 和 `setInterval` 事件前，在输入/输出事件后，安排一个 `callback` "immediate" 立即执行。

immediates 的回调以它们创建的顺序加入队列。整个回调队列会在事件循环迭代中执行。如果你将immediates 加入到一个正在执行回调中，那么将不会触发immediate，直到下次事件循环迭代。

## clearImmediate(immediateObject)

停止一个 immediate 的触发。
