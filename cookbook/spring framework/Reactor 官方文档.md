## [Introduction to Reactive Programming](https://projectreactor.io/docs/core/release/reference/#intro-reactive)

官方原文

Reactor is an implementation of the Reactive Programming paradigm, which can be summed up as follows:

> Reactive programming is an asynchronous programming paradigm concerned with data streams and the propagation of change. This means that it becomes possible to express static (e.g. arrays) or dynamic (e.g. event emitters) data streams with ease via the employed programming language(s).

— https://en.wikipedia.org/wiki/Reactive_programming

反应式编程是一种**异步编程范式**，涉及**数据流**和**变化**的传播。这意味着可以通过所使用的编程语言轻松地表示静态（例如阵列）或动态（例如事件发射器）数据流。

The reactive programming paradigm is often presented in object-oriented languages as an extension of the Observer design pattern. You can also compare the main reactive streams pattern with the familiar Iterator design pattern, as there is a duality to the Iterable-Iterator pair in all of these libraries. One major difference is that, while an Iterator is pull-based, reactive streams are push-based.

反应式编程范式通常在面向对象语言中作为**观察者设计模式的扩展**而出现。您还可以将主反应流模式与熟悉的迭代器设计模式进行比较，因为在所有这些库中，Iterable迭代器对都具有双重性。一个主要的区别是，**迭代器是基于拉**的，而**反应流是基于推**的。

Using an iterator is an imperative programming pattern, even though the method of accessing values is solely the responsibility of the Iterable. Indeed, it is up to the developer to choose when to access the next() item in the sequence. In reactive streams, the equivalent of the above pair is Publisher-Subscriber. But it is the Publisher that notifies the Subscriber of newly available values as they come, and this push aspect is the key to being reactive. Also, operations applied to pushed values are expressed declaratively rather than imperatively: The programmer expresses the logic of the computation rather than describing its exact control flow.

使用迭代器是一种命令式编程模式，即使访问值的方法完全由Iterable负责。实际上，**由开发人员选择何时访问序列中的下一个项**。在反应流中，上述对的等价物是**发布者-订阅者**。但发布者会在新的可用值出现时通知订阅者，**这一推送特性是反应性的关键**。此外，应用于推送值的操作以声明方式而不是强制方式表示：程序员**表示计算的逻辑**，而不是描述其确切的控制流。

In addition to pushing values, the error-handling and completion aspects are also covered in a well defined manner. A `Publisher` can push new values to its `Subscriber` (**by calling `onNext`**) but can also signal an error (**by calling `onError`**) or completion (**by calling `onComplete`**). Both errors and completion terminate the sequence. This can be summed up as follows:

```none
onNext x 0..N [onError | onComplete]
```

This approach is very flexible. The pattern supports use cases where there is no value, one value, or n values (including an infinite sequence of values, such as the continuing ticks of a clock).

###  3.1. Blocking Can Be Wasteful

 见官方， 说Blocking不好的地方

###  3.2. Asynchronicity to the Rescue?

 见官方， 说Callbacks和Futures 不好的地方，主要是代码太复杂，各种嵌套

###  3.3. From Imperative to Reactive Programming

Reactive libraries, such as Reactor, aim to address these drawbacks of “classic” asynchronous approaches on the JVM while also focusing on a few additional aspects:

- **Composability** and **readability**
- Data as a **flow** manipulated with a rich vocabulary of **operators**
- Nothing happens until you **subscribe**
- **Backpressure** or *the ability for the consumer to signal the producer that the rate of emission is too high*
- **High level** but **high value** abstraction that is *concurrency-agnostic*

#### 3.3.1. Composability and Readability

By “composability”, we mean the ability to orchestrate multiple asynchronous tasks, in which we use results from previous tasks to feed input to subsequent ones. Alternatively, we can run several tasks in a fork-join style. In addition, we can reuse asynchronous tasks as discrete components in a higher-level system.

The ability to orchestrate tasks is tightly coupled to the readability and maintainability of code. As the layers of asynchronous processes increase in both number and complexity, being able to compose and read code becomes increasingly difficult. As we saw, the callback model is simple, but one of its main drawbacks is that, for complex processes, you need to have a callback executed from a callback, itself nested inside another callback, and so on. That mess is known as “Callback Hell”. As you can guess (or know from experience), such code is pretty hard to go back to and reason about.

Reactor offers rich composition options, wherein code mirrors the organization of the abstract process, and everything is generally kept at the same level (nesting is minimized).

#### 3.3.2. The Assembly Line Analogy

You can think of data processed by a reactive application as moving through an assembly line. Reactor is both the conveyor belt and the workstations. The raw material pours from a source (the original `Publisher`) and ends up as a finished product ready to be pushed to the consumer (or `Subscriber`).

The raw material can go through various transformations and other intermediary steps or be part of a larger assembly line that aggregates intermediate pieces together. If there is a glitch or clogging at one point (perhaps boxing the products takes a disproportionately long time), the afflicted workstation can signal upstream to limit the flow of raw material.

#### 3.3.3. Operators

In Reactor, operators are the workstations in our assembly analogy. Each operator adds behavior to a `Publisher` and wraps the previous step’s `Publisher` into a new instance. The whole chain is thus linked, such that data originates from the first `Publisher` and moves down the chain, transformed by each link. Eventually, a `Subscriber` finishes the process. Remember that nothing happens until a `Subscriber` subscribes to a `Publisher`, as we see shortly.

|      | Understanding that operators create new instances can help you avoid a common mistake that would lead you to believe that an operator you used in your chain is not being applied. See this [item](https://projectreactor.io/docs/core/release/reference/#faq.chain) in the FAQ. |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

While the Reactive Streams specification does not specify operators at all, one of the best added values of reactive libraries, such as Reactor, is the rich vocabulary of operators that they provide. These cover a lot of ground, from simple transformation and filtering to complex orchestration and error handling.

#### 3.3.4. Nothing Happens Until You `subscribe()`

In Reactor, when you write a `Publisher` chain, data does not start pumping into it by default. Instead, you create an abstract description of your asynchronous process (which can help with reusability and composition).

By the act of **subscribing**, you tie the `Publisher` to a `Subscriber`, which triggers the flow of data in the whole chain. This is achieved internally by a single `request` signal from the `Subscriber` that is propagated upstream, all the way back to the source `Publisher`.

#### 3.3.5. Backpressure

Propagating signals upstream is also used to implement **backpressure**, which we described in the assembly line analogy as a feedback signal sent up the line when a workstation processes more slowly than an upstream workstation.

The real mechanism defined by the Reactive Streams specification is pretty close to the analogy: A subscriber can work in *unbounded* mode and let the source push all the data at its fastest achievable rate or it can use the `request` mechanism to signal the source that it is ready to process at most `n` elements.

Intermediate operators can also change the request in-transit. Imagine a `buffer` operator that groups elements in batches of ten. If the subscriber requests one buffer, it is acceptable for the source to produce ten elements. Some operators also implement **prefetching** strategies, which avoid `request(1)` round-trips and is beneficial if producing the elements before they are requested is not too costly.

This transforms the push model into a **push-pull hybrid**, where the downstream can pull n elements from upstream if they are readily available. But if the elements are not ready, they get pushed by the upstream whenever they are produced.

#### 3.3.6. Hot vs Cold

The Rx family of reactive libraries distinguishes two broad categories of reactive sequences: **hot** and **cold**. This distinction mainly has to do with how the reactive stream reacts to subscribers:

- A **Cold** sequence starts anew for each `Subscriber`, including at the source of data. For example, if the source wraps an HTTP call, a new HTTP request is made for each subscription.
- A **Hot** sequence does not start from scratch for each `Subscriber`. Rather, late subscribers receive signals emitted *after* they subscribed. Note, however, that some hot reactive streams can cache or replay the history of emissions totally or partially. From a general perspective, a hot sequence can even emit when no subscriber is listening (an exception to the “nothing happens before you subscribe” rule).

For more information on hot vs cold in the context of Reactor, see [this reactor-specific section](https://projectreactor.io/docs/core/release/reference/#reactor.hotCold).

[Suggest Edit](https://github.com/reactor/reactor-core/edit/main/docs/asciidoc/reactiveProgramming.adoc) to "[Introduction to Reactive Programming](https://projectreactor.io/docs/core/release/reference/#intro-reactive)"

## Reactor Core Features

The Reactor project main artifact is `reactor-core`, a reactive library that focuses on the Reactive Streams specification and **targets Java 8**.

Reactor introduces composable reactive types that implement `Publisher` but also provide a rich vocabulary of operators: `Flux` and `Mono`.

 **A `Flux` object represents a reactive sequence of 0..N items, while a `Mono` object represents a single-value-or-empty (0..1) result.**

This distinction carries a bit of semantic information into the type, indicating the rough cardinality of the asynchronous processing. For instance, an HTTP request produces only one response, so there is not much sense in doing a `count` operation. Expressing the result of such an HTTP call as a `Mono<HttpResponse>` thus makes more sense than expressing it as a `Flux<HttpResponse>`, as it offers only operators that are relevant to a context of zero items or one item.

这种区别在类型中携带了一些语义信息，表示异步处理的大致基数。例如，一个HTTP请求只产生一个响应，因此执行计数操作没有多大意义。因此，将此类HTTP调用的结果表示为Mono<HttpResponse>比将其表示为Flux<HttpResponse>更有意义，因为它只提供与零项或一项上下文相关的运算符。

> 所以结合返回值数量，合理选择使用Flux和Mono

Operators that change the maximum cardinality of the processing also switch to the relevant type. For instance, the `count` operator exists in `Flux`, but it returns a `Mono<Long>`.

> ? 没懂

### 4.1. `Flux`, an Asynchronous Sequence of 0-N Items

The following image shows how a `Flux` transforms items:

![Flux](E:\personal site\trainkg.github.io\cookbook\spring framework\images\flux.svg)

A `Flux<T>` is a standard `Publisher<T>` that represents an **asynchronous sequence of 0 to N emitted items**, optionally **terminated** by either a **completion signal or an error**. As in the Reactive Streams spec, these **three types of signal translate** to calls to a **downstream** Subscriber’s `onNext`, `onComplete`, and `onError` methods.

> 总体来说是不是直接调用，而是通过信号的方式去处理下游监听对象的逻辑。3中方式

With this large scope of possible signals, `Flux` is the general-purpose reactive type. Note that all events, even terminating ones, are optional: no `onNext` event but an `onComplete` event represents an *empty* finite sequence, but remove the `onComplete` and you have an *infinite* empty sequence (not particularly useful, except for tests around cancellation). Similarly, infinite sequences are not necessarily empty. For example, `Flux.interval(Duration)` produces a `Flux<Long>` that is infinite and emits regular ticks from a clock.

###  4.2. `Mono`, an Asynchronous 0-1 Result

The following image shows how a `Mono` transforms an item:

![Mono](E:\personal site\trainkg.github.io\cookbook\spring framework\images\mono.svg)

A `Mono<T>` is a specialized `Publisher<T>` that emits at most one item *via* the `onNext` signal then terminates with an `onComplete` signal (successful `Mono`, with or without value), or only emits a single `onError` signal (failed `Mono`).

Most `Mono` implementations are expected to immediately call `onComplete` on their `Subscriber` after having called `onNext`. `Mono.never()` is an outlier: it doesn’t emit any signal, which is not technically forbidden although not terribly useful outside of tests. On the other hand, a combination of `onNext` and `onError` is explicitly forbidden.

`Mono` offers only a subset of the operators that are available for a `Flux`, and some operators (notably those that combine the `Mono` with another `Publisher`) switch to a `Flux`. For example, `Mono#concatWith(Publisher)` returns a `Flux` while `Mono#then(Mono)` returns another `Mono`.

Note that you can use a `Mono` to represent no-value asynchronous processes that only have the concept of completion (similar to a `Runnable`). To create one, you can use an empty `Mono<Void>`.

###  4.3. Simple Ways to Create a Flux or Mono and Subscribe to It

见项目代码 `ReactorDemo`

#### 4.3.1. `subscribe` Method Examples

见项目代码 `ReactorDemo`

#### 4.3.2. Cancelling a `subscribe()` with Its `Disposable`

All these lambda-based variants of `subscribe()` have a `Disposable` return type. In this case, the `Disposable` interface represents the fact that the subscription can be *cancelled*, by calling its `dispose()` method.

For a `Flux` or `Mono`, cancellation is a signal that the source should stop producing elements. However, it is NOT guaranteed to be immediate: Some sources might produce elements so fast that they could complete even before receiving the cancel instruction.

Some utilities around `Disposable` are available in the `Disposables` class. Among these, `Disposables.swap()` creates a `Disposable` wrapper that lets you atomically cancel and replace a concrete `Disposable`. This can be useful, for instance, in a UI scenario where you want to cancel a request and replace it with a new one whenever the user clicks on a button. Disposing the wrapper itself closes it. Doing so disposes the current concrete value and all future attempted replacements.

Another interesting utility is `Disposables.composite(…)`. This composite lets you collect several `Disposable` — for instance, multiple in-flight requests associated with a service call — and dispose all of them at once later on. Once the composite’s `dispose()` method has been called, any attempt to add another `Disposable` immediately disposes it.

#### 4.3.3. An Alternative to Lambdas: `BaseSubscriber`

