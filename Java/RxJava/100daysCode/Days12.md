	在上次的实例中，我们已经使用过了`retry`,那么我们今天来看一下这个操作到底在帮助我们做些什么事情

我们很轻易的就能创造一个错误
``` java 
@Test  
public void generateFlowable() {  
    Flowable<Integer> retryFlowable = Flowable.range(0, 10).map(data -> 1000 / data).retry(3);  
    retryFlowable.subscribe(new ConsoleSubscribe());  
}
```
于是我们就得到了以下的打印结果：
``` console
Thread:【main】 在 2023-04-10T15:07:24.186Z 开启注册消费！
Thread:【main】 在 2023-04-10T15:07:24.196Z 发现了异常错误！java.lang.ArithmeticException: / by zero
```
根绝这个结果我们看到，一旦发生了错误，整个`Flowable`就停止了，并且程序貌似并没有像我们预期的方式重试三次，这个是为什么呢？
经过对`Flowable`进一步研究，我们猜想是不是并没有进入到我们预期的`onNext`方法中，是不是与我们的`map`操作有关，所以我们在`map`操作之前加入一个监听方法：
``` java
@Test  
public void generateFlowable() {  
    Flowable<Integer> retryFlowable = Flowable.range(0, 10)  
            .doOnEach(data -> System.out.println("Thread:【" + Thread.currentThread().getName() + "】 在 " + Instant  
                    .now() + " 消费了 data【" + data.getValue() + "】"))  
            .map(data -> 1000 / data).retry(3);  
    retryFlowable.subscribe(new ConsoleSubscribe());  
}
```
我们又得到了以下结果：
``` console
Thread:【main】 在 2023-04-10T15:11:24.690Z 开启注册消费！
Thread:【main】 在 2023-04-10T15:11:24.699Z 消费了 data【0】
Thread:【main】 在 2023-04-10T15:11:24.700Z 消费了 data【0】
Thread:【main】 在 2023-04-10T15:11:24.700Z 消费了 data【0】
Thread:【main】 在 2023-04-10T15:11:24.700Z 消费了 data【0】
Thread:【main】 在 2023-04-10T15:11:24.701Z 发现了异常错误！java.lang.ArithmeticException: / by zero
```
很明显的可以看得到，`RxJava`在我们自己执行错误的情况下，确实帮助我们重试了三次，而且重试点应该是从操作的头部开始，并且尾部的接收方法`onNext`并不会捕获到执行情况，所以在使用的时候需要特别注意。
其次并不是`retry`中断了`Flowale`中的数据传递，是因为错误被`onError`捕获到了，所以才会推出当前的数据传递。