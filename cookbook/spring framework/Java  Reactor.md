## [反应式编程](https://projectreactor.io/docs/core/release/reference/#flux)

在传统的数据流中，可以分为两个部分 状态变动方， 受影响方

比如下面的例子

```java
a + b = c;
// 当前 a 或者 b变动时, 作为受影响方c 直接的接受了a或者b变动对于他带来的影响
```

反应式编程就是斩断剥离**这直接影响的联系**以及**耦合性**

主要使用状态机机制

```java
a + b = c;
// 针对 a,b 设计对应的状态机 sa, sb

// 提供针对sa,sb状态变更的监控对象  stateChangeListener & stateChangeHandler 
stateChangeLisnter.linsen();
stateChangeHandler.hander(); // update C
```

经过上述的改造，那么C的更新就不再是直接更新，而是由状态变更监控方 `stateChangeListener` 控制. 

**总体是基于注册监听的做法，但是反应式编程统一使用状态机制来控制时间的触发并形成了统一的标准, 如果stateChangeListener是拉模式，它一直不拉取变更状态信息，那么c永远不会更新**

### 优势

- **Composability** and **readability**
- Data as a **flow** manipulated with a rich vocabulary of **operators**
- Nothing happens until you **subscribe**
- **Backpressure** or *the ability for the consumer to signal the producer that the rate of emission is too high*
- **High level** but **high value** abstraction that is *concurrency-agnostic*

### Flux 和 Mono

Flux 和 Mono 是 Reactor 中的两个基本概念。**Flux 表示的是包含 0 到 N 个元素的异步序列**。在该序列中可以包含**三种不同类型的消息**通知：**正常的包含**

* **元素的消息**

* **序列结束的消息**

* **序列出错的消息**

当消息通知产生时，订阅者中对应的方法 onNext(), onComplete()和 onError()会被调用。

**Mono 表示的是包含 0 或者 1 个元素的异步序列**。该序列中同样可以包含与 Flux 相同的三种类型的消息通知。

Flux 和 Mono 之间可以进行转换。对一个 Flux 序列进行计数操作，得到的结果是一个 Mono<Long>对象。把两个 Mono 序列合并在一起，得到的是一个 Flux 对象。



### 创建 Flux

有多种不同的方式可以创建 Flux 序列。

#### Flux 类的静态方法

第一种方式是通过 Flux 类中的静态方法。

- just()：可以指定序列中包含的全部元素。创建出来的 Flux 序列在发布这些元素之后会自动结束。

- fromArray()，fromIterable()和 fromStream()：可以从一个数组、Iterable 对象或 Stream 对象中创建 Flux 对象。

- empty()：创建一个不包含任何元素，只发布结束消息的序列。

- error(Throwable error)：创建一个只包含错误消息的序列。

- never()：创建一个不包含任何消息通知的序列。

- range(int start, int count)：创建包含从 start 起始的 count 个数量的 Integer 对象的序列。

- interval(Duration period)和 interval(Duration delay, Duration period)：创建一个包含了从 0 开始递增的 Long 对象的序列。其中包含的元素按照指定的间隔来发布。除了间隔时间之外，还可以指定起始元素发布之前的延迟时间。

- intervalMillis(long period)和 intervalMillis(long delay, long period)：与 interval()方法的作用相同，只不过该方法通过毫秒数来指定时间间隔和延迟时间。

  

#### generate()方法

generate()方法通过同步和逐一的方式来产生 Flux 序列。序列的产生是通过调用所提供的 SynchronousSink 对象的 next()，complete()和 error(Throwable)方法来完成的。逐一生成的含义是在具体的生成逻辑中，next()方法只能最多被调用一次。在有些情况下，序列的生成可能是有状态的，需要用到某些状态对象。此时可以使用 generate()方法的另外一种形式 generate(Callable<S> stateSupplier, BiFunction<S,SynchronousSink<T>,S> generator)，其中 stateSupplier 用来提供初始的状态对象。在进行序列生成时，状态对象会作为 generator 使用的第一个参数传入，可以在对应的逻辑中对该状态对象进行修改以供下一次生成时使用。

#### create()方法

create()方法与 generate()方法的不同之处在于所使用的是 FluxSink 对象。FluxSink 支持同步和异步的消息产生，并且可以在一次调用中产生多个元素。在代码清单 3 中，在一次调用中就产生了全部的 10 个元素。

**清单 3. 使用 create()方法生成 Flux 序列**

```java
Flux.create(sink -> {
     for (int i = 0; i < 10; i++) {
         sink.next(i);
     }
     sink.complete();
}).subscribe(System.out::println);
```



参考文档

[论坛文章]: https://blog.csdn.net/simple_chao/article/details/73648238?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_link&amp;depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1.no_search_





