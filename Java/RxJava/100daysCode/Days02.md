在昨天的使用我们，我们是用了最简单的RxJava的模型，在这里我们回顾一下我们是如何订阅了消息

``` java
Disposable subscribe = flowable.subscribe(System.out::println);
```
这里使用了subscibe方法的一个重载方法，其实它总共有其中不同的声明

> 1.  `public final Disposable subscribe()`
> 2. `public final Disposable subscribe(@NonNull Consumer<? super T> onNext)`
> 3.  `public final Disposable subscribe(@NonNull Consumer<? super T> onNext, @NonNull Consumer<? super Throwable> onError)`
> 4.  `public final Disposable subscribe(@NonNull Consumer<? super T> onNext, @NonNull Consumer<? super Throwable> onError,@NonNull Action onComplete)`
> 5.  `public final Disposable subscribe(@NonNull Consumer<? super T> onNext,@NonNull Consumer<? super Throwable> onError,@NonNull Action onComplete,@NonNull DisposableContainer container)`
> 6.  `public final void subscribe(@NonNull Subscriber<? super T> subscriber)`
> 7.  `public final void subscribe(@NonNull FlowableSubscriber<? super T> subscriber)`

其中我们主要来看一下序号`6`的方法重载，其他的方法重载，是为了方便取使用而做的一些简化。    
从签名中我们就注意到了一个对象`Subscriber<? super T>`，那么它的组成部分到底是什么样的，我们来看一下：
``` java
Subscriber<Integer> subscriber = new Subscriber<Integer>() {  
    @Override  
    public void onSubscribe(Subscription s) {  
  
    }  
  
    @Override  
    public void onNext(Integer o) {  
  
    }  
  
    @Override  
    public void onError(Throwable t) {  
  
    }  
  
    @Override  
    public void onComplete() {  
  
    }  
};
```
从接口的方法签名上，我们就可以明确的看到，这里有四个接口方法需要我们实现，那么就可以`Flowable`的消息进行订阅了。如果要达到昨天效果，我们只需要取实现对应的`onNext`和`onSubscribe`方法
``` java
Flowable<Integer> flowable = Flowable.just(1);  
Subscriber<Integer> subscriber = new Subscriber<Integer>() {  
    @Override  
    public void onSubscribe(Subscription s) {  
		s.request(Long.MAX_VALUE);
    }  
  
    @Override  
    public void onNext(Integer o) {  
        System.out.println(o);  
    }  
  
    @Override  
    public void onError(Throwable t) {  
  
    }  
  
    @Override  
    public void onComplete() {  
  
    }  
};  
flowable.subscribe(subscriber);

```
一定要注意`onSubscribe`方法，按照接口的说明，如果没有对`Subscription`进行request操作的话，那么其实`Publisher`并不会发送消息，这里为什么用`s.request(Long.MAX_VALUE)`,其实当你去看对应的重载发放的时候，你就会发现当我们用其他的重载方法的时候，他会去做这样一个组装，帮我们构造一个`LambdaSubscriber`
``` java
LambdaSubscriber<T> ls = new LambdaSubscriber<>(onNext, onError, onComplete, FlowableInternalHelper.RequestMax.INSTANCE);
```

那么这里就看得到有一个`单例对象FlowableInternalHelper.RequestMax.INSTANCE`,那我们再看一下它指代的是什么
``` java
public enum RequestMax implements Consumer<Subscription> {  
    INSTANCE;  
    @Override  
    public void accept(Subscription t) {  
        t.request(Long.MAX_VALUE);  
    }  
}
```
所以这就是为什么我们也用这个的原因，那我们可不可以用其他的数值来代替呢？，当我们把他换成任意一个数值
``` java
@Override  
public void onSubscribe(Subscription s) {  
    s.request(1);  
}
```
我们会发现，它在现在的上下文上，确实也得到了我们的预期，但是如果我们将上下文做一个处理
``` java 
Flowable<Integer> flowable = Flowable.just(1, 2, 3, 4, 5, 6);  
```
那么这个时候，你就会发现它打印了1之后就再也没有其他的打印了。所以我们再去看一下 `Subscription`的`request(long)`方法的说明文档，我们就会发现，它说明了这里是用来限制到底能够最多拿到`Publisher`发送的多少个消息，所以我们在这里可以大胆的假设它的使用场景：
- 定量的心跳包发送
- 请求限流
那么剩下的`onNext`方法、`onError`方法、`onComplete`方法我们就见名知意了，分别是用于处理每一次获取到消息、发生异常时的处理、消息完全接收完毕之后的处理。到后续我们需要的时候再回过头来继续看看，他们有没有什么可以利用的点。
