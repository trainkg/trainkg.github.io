## [Promise](https://es6.ruanyifeng.com/#docs/promise#Promise-resolve) 

`Promise`对象代表一个异步操作，有三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）

。如果改变已经发生了，你再对`Promise`对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

> Promise 可以重复监听

### 基本用法



`resolve`函数的作用是，将`Promise`对象的状态从“未完成”变为“成功”

`reject`函数的作用是，将`Promise`对象的状态从“未完成”变为“失败”

`Promise`实例生成以后，可以用`then`方法分别指定`resolved`状态和`rejected`状态的回调函数。

```javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

它们都接受`Promise`对象传出的值作为参数。

Promise 新建后就会立即执行。

> 主要指构建Promise对象过程会立刻执行

### Promise.resolve()

有时需要将现有对象转为 Promise 对象，`Promise.resolve()`方法就起到这个作用。