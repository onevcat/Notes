### [RAC Marbles](http://neilpa.me/rac-marbles/)

可视化帮助理解 RAC 流。

### Event

Event 流一定满足

> Next* (Interrupted | Error | Completed)?

若干个 Next 后，跟一个终结事件 (Interrupted | Error | Completed 三者之一)。

RAC 保证事件是串行到达的，并且事件不会被循环发送。但是如果一个事件的发送依赖于另一个信号，但是那个信号又依赖于这个事件的话，是可以产生死锁的。一般来说死锁都是人为造成的。当需要循环发送事件的时候，建议使用 delay 来做时间推移，以保证是从不同的 handler 中发送的。

### [Signal](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/v4.0-alpha.1/ReactiveCocoa/Swift/Signal.swift)

热信号。观察者不能影响信号的产生和终结，信号也无法重启。Signal 是引用类型，我们可以对 signal 进行标记和状态更改。

使用 `init` 可以直接创建一个热信号，并立即执行传入的参数 (一个闭包)：

```swift
Signal {
    sink in
    NSTimer.schedule(repeatInterval: 1.0) { timer in
      sendNext(sink, "tick #\(count++)")
    }
    return nil
}
```

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

### [Signal Producers](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/v4.0-alpha.1/ReactiveCocoa/Swift/SignalProducer.swift)

冷信号。

信号源用来创建和发生信号，用来指代网络请求等任务。使用 `toSignalProducer` 将一个 `RACSignal` 转换为 `SignalProducer`。

信号源是信号产生的蓝图，它们自己并不会工作，而需要外界启动以产生信号。SingalProducer 是值类型。我们通常所使用的 start 或者 cancel 一个 signal producer，实际上指的是由 Signal Producer 开始一个 Signal，或者取消 (停止) 这个 Signal。因为 Signal Producer 是蓝图，因此它可以被 start 任意多次，并对应任意多个 Signal，每个将独立存在并发送事件。


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

### 操作和组合

#### lift

对 Producer 的产生的 Signal 进行操作，并映射到一个新的 Producer。

因为 Signal 是由 Signal Producer 产生的，因此我们有可能将某个针对 Signal 的操作应用到 (抬举到) 一个 Producer 产生的所有 Signal 上。这就是 lift 的作用 -- 将某个应用于特定 Signal 的规则应用到整个 Signal Producer 上，然后得到应用规则以后的新的 Signal Producer。

与 `map` 的区别：`map` 对 signal 的 Value 进行操作，而 lifting 是对整个信号操作。

关于 Lifting，[这里](http://stackoverflow.com/questions/2395697/haskell-newbie-question-what-is-lifting)有一个不错的答案解释。

#### map 和 filter

相对于 `lift` 来说，`map` 和 `filter` 更加容易理解。`map` 将信号值进行映射，`filter` 将信号值进行过滤，仅留下满足条件的信号，其他不满足的将在新的 Signal 流中被忽略 (注意新的和旧的信号共享同一组 sink)。

#### reduce

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

#### collect

将信号收集为一个 Array，在原信号源 complete 时被 send 及 complete。

```swift
let (signal, sink) = Signal<Int, NoError>.pipe()
signal.collect().observe(next: println)

sendNext(sink, 1)     // nothing printed
sendNext(sink, 2)     // nothing printed
sendNext(sink, 3)     // nothing printed
sendCompleted(sink)   // prints [1, 2, 3]
```

#### combineLatest

直到两个信号输出了至少一次 next 时，发送第一次 next (将两个信号最新的 next 合并)；之后每次任意信号 next 则触发新信号的 next。[图例](http://neilpa.me/rac-marbles/#combineLatest)

```
combineLatest
  [0, 1,        2                  ]
  [      A,            B,     C    ]
->[      (1,A), (2,A), (2,B), (2,C)]
```

#### zip

直到两个信号输出了至少一次 next 时，发送第一次 next (将两个信号最新的 next 合并)；之后将两个信号的后一个 next 合并触发新信号的 next。[图例](http://neilpa.me/rac-marbles/#zip)

```
zip
  [0, 1,        2              ]
  [      A,        B,     C    ]
->[      (0,A),    (1,B), (2,C)]
```

#### flatten/flatMap

将多个信号/信号源展平为一个信号。

###### .Merge

按照信号 next 被发送的顺序组织结果信号，不同信号按照时间插值。[图例](http://neilpa.me/rac-marbles/)

###### .Concat

按照信号整体的顺序组织信号，信号按照加入时间串联。[图例](http://neilpa.me/rac-marbles/#concat)

###### .Latest

只关心最后加入的信号值。


#### flatMapError

将 Error 映射为 Value，并原地开始一个新的 signal producer，相当于化解 Error。

#### retry

遇到错误时重新开始 signal producer，直到达到重试错误次数。

#### promoteErrors

让一个不能产生错误的 signal producer 可以产生某个错误。

### Action

### Property

存储变量，通常是用来做 binding 的 entrypoint。

用 `<~` 来绑定一个属性，操作符左侧需要是 `MutablePropertyType`：

* `property <~ signal` 将信号绑定到属性上，信号的最新值将被设定到属性上
* `property <~ signal producer` 将信号源绑定到属性上，并开始信号源，将得到的信号值设定到属性上
* `property <~ otherProperty` 将另一个属性绑定到属性上，源属性变化时，左侧属性也跟随变化

### Disposables

`Disposable` 接口表示。

开始一个 producer 或者开始监视一个 signal 时可以获取到对应的 Disposable。可以通过 Disposables 开取消 (Interrupted) 任务。

### Schedulers

`SchedulerType` 接口表示。

串行执行队列，

### 最佳实践

#### 尽可能短的生命周期

一旦你不再需要某个 Signal 或者 Signal Producer 的事件，就可以关掉它。比如只关心前几个事件的话，使用 `take` 或者 `takeUntil`。

#### 线程

虽然 Event 是串行到达的，但是对于未知来源的 signal，你并不能确定它会在哪个线程被发送，这在进行 UI 绑定的时候会有危险。通过使用 `observeOn` 来制定观察的线程。

但是应该避免频繁切换线程，只在必要的时候指定观察线程。`observeOn` 只应当被用在开始观察信号前或者开始一个信号源前。

#### 扩展

因为 lift 的存在，所有针对 Singal 的操作都可以应用在 Signal Producer 上，所以要跑扩展 RAC 操作，最好是为 Signal 编写操作，这样我们就能在 Signal Producer 上免费使用它们。

#### Switch

由于 Event 其实是 enum，所以尽量使用 `switch` 来让编译器保证各个分支都得到了适当处理。

#### 避免并行

RAC 的世界其实是厌恶并行的，使用 Signal Producer 可以同时开启若干个信号，所以 RAC 中并行需求并没有那么多，特别是对于并行时共享状态的情况。减少并行可以有效减少人为 bug。

### 源码阅读

#### [Atomic.swift](https://github.com/ReactiveCocoa/ReactiveCocoa/blob/master/ReactiveCocoa/Swift/Atomic.swift)

使用 Atomic 的方式来对值进行带锁操作：使用了 Spin 锁 `OSSpinLockLock`，主要用来存储 Signal 的 Observer。

#### Event.swift

RAC 由 event 构成，除了工具类型，Event 是最基础的类型。Event 枚举定义了事件类型：Next，Error，Completed 和 Interrupted。

另外实现了基本的转换操作，即将一个 Value 值转换为另一个 Value 值，或者将一个 Error 转换为另一个 Error。这是所有 Signal 和 SignalProducer 操作的基础。

```swift
func map<U>(f: Value -> U) -> Event<U, Err>
```

```swift
func mapError<F>(f: Err -> F) -> Event<Value, F>
```

#### Signal.swift

`init` 中使用一个 observer (sink) 来在遇到事件时向其他之后添加的 observers 分发事件。

`pipe` 将 `init` 中的 `observer` 获取为 tuple 中的 `observer` 并允许外层用户直接通过给这个 `observer` 发送事件来控制 Signal。

`observe` 将新的 observer 加入到 `atomicObservers` 中，并返回一个 `ActionDisposable`。在 `ActionDisposable` 被 dispose 的时候，action closure 将被执行以移除当前 observer。对于 Signal 来说，观察者 dispose 的只有 observer 本身，而不能影响原来的热信号发生。
