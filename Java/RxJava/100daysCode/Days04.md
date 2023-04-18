	在探索能够用已学习的知识能够用来做些什么东西的时候，本来想着来制作一个`心跳器`的，却发现还没有很好的办法去创建一个`心跳`，本着对于心跳的理解，所以心跳信号应该是用`Flowable`这种消息发布体来进行，提供所以，又得去探索一下关于`Flowable`的一些操作知识。

> 在第一次创建`Flowable`的时候看到他有很多的静态方法，那么就浅看一下有没有适合`心跳`的创建方式

### interval

我们找到他看上去最简单的方法签名：
- `public static Flowable<Long> interval(long period, @NonNull TimeUnit unit)`
结合到方法的说明文档。我们知道这个方法可以让我们创建一个不断输出有序数字的发射器。
``` java
@Test  
public void generateFlowable() {  
    Flowable<Long> intervalFlowable = Flowable.interval(1, TimeUnit.SECONDS);  
  
    intervalFlowable.subscribe(new ConsoleSubscribe());  
  
}
```

但是我们运行这段代码，只得到了一串打印信息（`ConsoleSubscribe`是我编写的一个会打印出模板信息`Thead - Time - Data`的`Subscribe`实现）:
``` console
Thread:【main】 在 2023-04-03T12:54:23.243Z 开启注册消费！
```

所以可以看到我们确实注册成功了，但是并没有执行消费。我们猜测是不是因为消息的传递不是在`main`线程执行的，所以我们添加一个死循环，来监控一下，最后得到了我们预取的打印数据：
``` console
Thread:【main】 在 2023-04-03T12:57:13.927Z 开启注册消费！
Thread:【RxComputationThreadPool-1】 在 2023-04-03T12:57:14.937Z 消费了 data【0】
Thread:【RxComputationThreadPool-1】 在 2023-04-03T12:57:15.942Z 消费了 data【1】
Thread:【RxComputationThreadPool-1】 在 2023-04-03T12:57:16.951Z 消费了 data【2】
Thread:【RxComputationThreadPool-1】 在 2023-04-03T12:57:17.942Z 消费了 data【3】
```
确实是在一个线程池中进行了数据消费。那么自此我们的一个微型的`心跳`就制作完成了，目前只会进行简单的打印（取决于你对`Subscribe`的具体实现）