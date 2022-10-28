> 基于`观察者模式`实现的



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