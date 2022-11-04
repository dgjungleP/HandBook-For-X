>  1.  利用`可观测序列`来组合异步和基于时间的程序
>  2. 倡导`函数组合`
>  3. 避免出现`全局状态`和`副作用`
>  4. 以流的方式思考
>  5. 仅仅是对命令式回调对的抽象


## 使用场景
> 对于需要处理的事件复杂且相互关联的时候，使用效果更佳

1. 处理用户事件
2. 响应和处理来自磁盘或网络的所有`延迟受限`的I/O事件
3. 在应用程序中处理由该应用程序`无法控制的生产者`推送过来的事件或数据

## 如何运行的
> 借助于`Observable`/`Producer`类，来代替数据或事件的流，然后进行推送和拉取。其本身是延迟执行的，同时可以既可以同步使用，也可以异步使用

说明：
1. 自身并不关心是同步还是异步的，而是关注生成事件的过程是阻塞的还是非阻塞的
2. 单个的Observable既不允许并发，也不允许并行，必须始终都是序列化和线程安全的
3. 一般不建议在Observable中启动线程，而是使用`调度器`
4. 可以使用组合的方式来使用并发和并行
5. 一份`Observable`由于是延迟执行地（仅对于Cold类型的流），所以意味着，它可以被`重复使用`

### 为什么不允许并发地调用OnNext
1. 因为方法本意是提供给人类使用地，并发比较困难
2. 有些操作无法实现并发发布
3. 同步开销会影响性能
4. 进行一般地细粒度并行，通常会更慢


## 操作符

### 创建型
- create
- defer
- range
- interval
- timer
- empty
- error
- never
- just
- from

### 过滤型
- filter
- distinct

### 转换型
- map
- flatmap

说明：
1. 并不能保证原始事件的顺序

适用场景：
1. 对map的结果必须是Observable，想要执行一些异步操作（可以控制并发度）
2. 需要进行一对多的转换

- scan

### 聚合型
- count
- reduce

### 其他操作
- take
- skip
- window


## 调度器
> 用来对RxJava流操作进行调度的类，一般从`Scheduler`工厂方法获取实现

### 实现
- Scheduler.io()
- Scheduler.newThread()
- Scheduler.computation()
- Scheduler.trampoline()
- Scheduler.single()

## 背压
> 一般出现在响应式编程中，由于上游弹射数据高于下游接收数据的速度，而造成的数据积压，最后导致内存溢出

### 应对方式
> - LATEST：如果消费跟不上，那么仅仅缓存最近弹射出来的数据，将老旧的数据直接丢弃
> -  DROP：如果消费更不是，就会将除开第一个元素以外的其他元素丢弃
> -  NONE：不使用背压，如果消费产生积压，就会抛出异常
> -  ERROR：不使用背压，如果消费产生积压，就会抛出异常
> -  BUFFER：如果消费不上，会无线空阔缓冲区，知道内存消耗完


## 说明
1. 通过`Observable`和`Subscriber`来实现订阅关系的
2. 通过`Observable`中的弹射器`Emitter`来实现的消息发布
3. `Observable`还会负责消息序列的缓存，类似于`生产者消费者模式`
4. 支持函数式编程，设计了多个函数时接口
	1. Action0：无参数无返回
	   > 替代Subscriber的`onCompleted`方法
	2. Action1：有参数物返回
	   > 1. 替代`Subscriber`使用`onNext`
	   > 2. 替代`Subscriber`使用`onErrorAction`
5.  可以通过 `subscribeOn`和`observeOn`来改变RxJava的流操作的调度器



## 观察者模式
> 当`被观察的主题对象`的`状态发生变化`时，会自动通知这些`订阅者`，令他们做出响应，又叫做发布订阅模式

#### 组成部分
- Subject：抽象的被观察对象
- Concrete Subject：具体的被观察对象
- Observer：抽象的观察者
- Concrete Observer：具体的观察者