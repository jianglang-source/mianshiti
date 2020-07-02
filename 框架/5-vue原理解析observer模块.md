# Vue原理解析之observer模块

`observer`是Vue核心中最重要的一个模块（个人认为），能够实现视图与数据的响应式更新，底层全凭`observer`的支持。

`observer`模块在Vue项目中的代码位置是`src/core/observer`，模块共分为这几个部分：

- `Observer`: 数据的观察者，让数据对象的读写操作都处于自己的监管之下
- `Watcher`: 数据的订阅者，数据的变化会通知到`Watcher`，然后由`Watcher`进行相应的操作，例如更新视图
- `Dep`: `Observer`与`Watcher`的纽带，当数据变化时，会被`Observer`观察到，然后由`Dep`通知到`Watcher`

示意图如下：

![clipboard.png](https://segmentfault.com/img/bVJiAp?w=651&h=327)















## Observer

`Observer`类定义在`src/core/observer/index.js`中，先来看一下`Observer`的构造函数

```
constructor (value: any) {
  this.value = value
  this.dep = new Dep()
  this.vmCount = 0
  def(value, '__ob__', this)
  if (Array.isArray(value)) {
      const augment = hasProto
      ? protoAugment
      : copyAugment
    augment(value, arrayMethods, arrayKeys)
    this.observeArray(value)
  } else {
    this.walk(value)
  }
}
```

`value`是需要被观察的数据对象，在构造函数中，会给`value`增加`__ob__`属性，作为数据已经被`Observer`观察的标志。如果`value`是数组，就使用`observeArray`遍历`value`，对`value`中每一个元素调用`observe`分别进行观察。如果`value`是对象，则使用`walk`遍历`value`上每个key，对每个key调用`defineReactive`来获得该key的`set/get`控制权。

解释下上面用到的几个函数的功能：

- `observeArray`: 遍历数组，对数组的每个元素调用`observe`
- `observe`: 检查对象上是否有`__ob__`属性，如果存在，则表明该对象已经处于`Observer`的观察中，如果不存在，则`new Observer`来观察对象（其实还有一些判断逻辑，为了便于理解就不赘述了）
- `walk`: 遍历对象的每个key，对对象上每个key的数据调用`defineReactive`
- `defineReactive`: 通过`Object.defineProperty`设置对象的key属性，使得能够捕获到该属性值的`set/get`动作。一般是由`Watcher`的实例对象进行`get`操作，此时`Watcher`的实例对象将被自动添加到`Dep`实例的依赖数组中，在外部操作触发了`set`时，将通过`Dep`实例的`notify`来通知所有依赖的`watcher`进行更新。

如果不太理解上面的文字描述可以看一下图：

![clipboard.png](https://segmentfault.com/img/bVJiJC?w=844&h=337)

## Dep

`Dep`是`Observer`与`Watcher`之间的纽带，也可以认为`Dep`是服务于`Observer`的订阅系统。`Watcher`订阅某个`Observer`的`Dep`，当`Observer`观察的数据发生变化时，通过`Dep`通知各个已经订阅的`Watcher`。

`Dep`提供了几个接口：

- `addSub`: 接收的参数为`Watcher`实例，并把`Watcher`实例存入记录依赖的数组中
- `removeSub`: 与`addSub`对应，作用是将`Watcher`实例从记录依赖的数组中移除
- `depend`: `Dep.target`上存放这当前需要操作的`Watcher`实例，调用`depend`会调用该`Watcher`实例的`addDep`方法，`addDep`的功能可以看下面对`Watcher`的介绍
- `notify`: 通知依赖数组中所有的`watcher`进行更新操作

## Watcher

`Watcher`是用来订阅数据的变化的并执行相应操作（例如更新视图）的。`Watcher`的构造器函数定义如下：

```
constructor (vm, expOrFn, cb, options) {
  this.vm = vm
  vm._watchers.push(this)
  // options
  if (options) {
    this.deep = !!options.deep
    this.user = !!options.user
    this.lazy = !!options.lazy
    this.sync = !!options.sync
  } else {
    this.deep = this.user = this.lazy = this.sync = false
  }
  this.cb = cb
  this.id = ++uid // uid for batching
  this.active = true
  this.dirty = this.lazy // for lazy watchers
  this.deps = []
  this.newDeps = []
  this.depIds = new Set()
  this.newDepIds = new Set()
  this.expression = process.env.NODE_ENV !== 'production'
    ? expOrFn.toString()
    : ''
  if (typeof expOrFn === 'function') {
    this.getter = expOrFn
  } else {
    this.getter = parsePath(expOrFn)
    if (!this.getter) {
      this.getter = function () {}
      process.env.NODE_ENV !== 'production' && warn(
        `Failed watching path: "${expOrFn}" ` +
        'Watcher only accepts simple dot-delimited paths. ' +
        'For full control, use a function instead.',
        vm
      )
    }
  }
  this.value = this.lazy
    ? undefined
    : this.get()
}
```

参数中，`vm`表示组件实例，`expOrFn`表示要订阅的数据字段（字符串表示，例如`a.b.c`）或是一个要执行的函数，`cb`表示watcher运行后的回调函数，`options`是选项对象，包含`deep`、`user`、`lazy`等配置。

`watcher`实例上有这些方法：

- `get`: 将`Dep.target`设置为当前`watcher`实例，在内部调用`this.getter`，如果此时某个被`Observer`观察的数据对象被取值了，那么当前`watcher`实例将会自动订阅数据对象的`Dep`实例
- `addDep`: 接收参数`dep`(Dep实例)，让当前`watcher`订阅`dep`
- `cleanupDeps`: 清除`newDepIds`和`newDep`上记录的对dep的订阅信息
- `update`: 立刻运行`watcher`或者将`watcher`加入队列中等待统一flush
- `run`: 运行`watcher`，调用`this.get()`求值，然后触发回调
- `evaluate`: 调用`this.get()`求值
- `depend`: 遍历`this.deps`，让当前`watcher`实例订阅所有`dep`
- `teardown`: 去除当前`watcher`实例所有的订阅

## Array methods

在`src/core/observer/array.js`中，Vue框架对数组的`push`、`pop`、`shift`、`unshift`、`sort`、`splice`、`reverse`方法进行了改造，在调用数组的这些方法时，自动触发`dep.notify()`，解决了调用这些函数改变数组后无法触发更新的问题。在Vue的官方文档中对这个也有说明：[http://cn.vuejs.org/v2/guide/list.html#变异方法](http://cn.vuejs.org/v2/guide/list.html#)