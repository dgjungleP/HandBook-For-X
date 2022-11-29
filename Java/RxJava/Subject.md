> 即实现了`Observer`又扩展了`Observable`，所以即能订阅上游事件，又能推送下游事件
> 一般用以共享订阅和缓存事件

## 细节
1. 再调用了Subject.onError()之后，后续的所有onError通知都会被吞噬掉
2. 很容易破坏RX的并发原则（onNext的调用必须是序列化的），所以有必要的时候需要使用toSerialized方法


## 类型

### PublishSubject
> 会立即从上游系统接收事件，并简单地将它们推送到所有地Subscriber


### AsyncSubject
> 会记住`最后一个值`，并在onComplete调用后，将其推送给所有订阅者，如果它没有完成，那么除了`最后一个事件`以外，其他地事件都会被抛弃

### BehaviorSubject
> 在订阅之后，将所有的事件推送给订阅者，它会首先发布在订阅之前发布的最新事件

### ReplaySubject
>  缓存贯穿整个历史的推送事件，默认情况下所有的事件都会被缓存

注：
1. 为了安全期间可以选择使用重载版本的ReplaySubject
	1. createWithSize,控制需要保留的数量
	2. createWithTime,控制最近的时间窗口
	3. createWithTimAndSize，同时控制时间和数量
