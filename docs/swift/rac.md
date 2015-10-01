## [RAC Marbles](http://neilpa.me/rac-marbles/)

可视化帮助理解 RAC 流。

## [Signal Producers](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/v4.0-alpha.1/ReactiveCocoa/Swift/SignalProducer.swift)

冷信号。

创建和发生信号，用来指代网络请求等任务。使用 `toSignalProducer` 将一个 `RACSignal` 转换为 `SignalProducer`。

使用 `map` `flatMap` 等对信号/信号源变形，使用 `start` 来使信号源开始产生信号并监视。

使用 `buffer` 创建可控冷信号，`observe` 用来观察 event：

```swift
let signal: Signal<Int, NoError>
let (producer, sink) = SignalProducer<Int, NoError>.buffer()

// Saves observed values in the buffer
signal.observe(sink)

// Prints each value buffered
producer.start(next: { value in
    println(value)
})
```

## [Signal](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/v4.0-alpha.1/ReactiveCocoa/Swift/Signal.swift)

热信号。

使用 `pipe` 创建可控热信号：

```swift
let (signal, sink) = Signal<Int, NoError>.pipe()

signal.observe(next: { value in
    println(value)
})

// Prints each number
sendNext(sink, 0)
sendNext(sink, 1)
sendNext(sink, 2)
```

使用 `.on` 来注入信号，执行 side effect：

```swift
let producer = signalProducer.on(sink)
```

pipe 和 buffer 都是非 RAC 世界与 RAC 世界连接的桥梁。在开发中，尽量尝试减少这种桥梁的数量，而更多地保持在 RAC 的世界中。

## 操作和组合

### lift

对 Producer 的产生的 Signal 进行操作，并映射到一个新的 Producer。与 `map` 的区别：`map` 对 signal 的 Value 进行操作，而 lifting 是对整个信号操作。

关于 Lifting，[这里](http://stackoverflow.com/questions/2395697/haskell-newbie-question-what-is-lifting)有一个不错的答案解释。

### map 和 filter

相对于 `lift` 来说，`map` 和 `filter` 更加容易理解。`map` 将信号值进行映射，`filter` 将信号值进行过滤，仅留下满足条件的信号，其他不满足的将在新的 Signal 流中被忽略 (注意新的和旧的信号共享同一组 sink)。

### reduce

将信号 reduce 到一个。reduce 后的信号在原信号源 complete 时被 send 及 complete。

```swift
let (signal, sink) = Signal<Int, NoError>.pipe()

signal
    .reduce(1) { $0 * $1 }
    .observe(next: println)

sendNext(sink, 1)     // nothing printed
sendNext(sink, 2)     // nothing printed
sendNext(sink, 3)     // nothing printed
sendCompleted(sink)   // prints 6
```

### collect

将信号收集为一个 Array，在原信号源 complete 时被 send 及 complete。

```swift
let (signal, sink) = Signal<Int, NoError>.pipe()
signal.collect().observe(next: println)

sendNext(sink, 1)     // nothing printed
sendNext(sink, 2)     // nothing printed
sendNext(sink, 3)     // nothing printed
sendCompleted(sink)   // prints [1, 2, 3]
```

### combineLatest

直到两个信号输出了至少一次 next 时，发送第一次 next (将两个信号最新的 next 合并)；之后每次任意信号 next 则触发新信号的 next。[图例](http://neilpa.me/rac-marbles/#combineLatest)

```
combineLatest
  [0, 1,        2                  ]
  [      A,            B,     C    ]
->[      (1,A), (2,A), (2,B), (2,C)]
```

### zip

直到两个信号输出了至少一次 next 时，发送第一次 next (将两个信号最新的 next 合并)；之后将两个信号的后一个 next 合并触发新信号的 next。[图例](http://neilpa.me/rac-marbles/#zip)

```
zip
  [0, 1,        2              ]
  [      A,        B,     C    ]
->[      (0,A),    (1,B), (2,C)]
```

### flatten/flatMap

将多个信号/信号源展平为一个信号。

#### .Merge

按照信号 next 被发送的顺序组织结果信号，不同信号按照时间插值。[图例](http://neilpa.me/rac-marbles/)

#### .Concat

按照信号整体的顺序组织信号，信号按照加入时间串联。[图例](http://neilpa.me/rac-marbles/#concat)

#### .Latest

只关心最后加入的信号值。


### flatMapError

将 Error 映射为 Value，并原地开始一个新的 signal producer，相当于化解 Error。

### retry

遇到错误时重新开始 signal producer，直到达到重试错误次数。

### promoteErrors

让一个不能产生错误的 signal producer 可以产生某个错误。

## Action

## Property

存储变量，通常是用来做 binding 的 entrypoint。

用 `<~` 来绑定一个属性，操作符左侧需要是 `MutablePropertyType`：

* `property <~ signal` 将信号绑定到属性上，信号的最新值将被设定到属性上
* `property <~ signal producer` 将信号源绑定到属性上，并开始信号源，将得到的信号值设定到属性上
* `property <~ otherProperty` 将另一个属性绑定到属性上，源属性变化时，左侧属性也跟随变化

## Disposables
