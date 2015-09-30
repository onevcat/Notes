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

## 操作组合

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
