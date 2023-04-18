	上次我们制作了简单的`心跳包`，那除了`心跳`以外我们还能做些什么呢。在对创建方法的探索中，我们又发现了一个带有类似`interval`方法签名的一个方法

### timer

其最简单的方法签名为：
- `public static Flowable<Long> timer(long delay, @NonNull TimeUnit unit)`
从其方法的说明文档中，我们可以看到它的声明，是可以在延迟一定的时间后，发送一个`0L`供给下游消费
``` java
@Test  
public void generateFlowable() {  
    Flowable<Long> flowable = Flowable.timer(10, TimeUnit.SECONDS);  
    flowable.subscribe(new ConsoleSubscribe());  
}
```

我们同样去简单的使用一下，我们只得到了一句注册成功的打印：
``` console
Thread:【main】 在 2023-04-03T13:09:49.346Z 开启注册消费！
```

那么加上死循环之后，我们再次运行：
``` console
Thread:【main】 在 2023-04-03T13:11:26.225Z 开启注册消费！
Thread:【RxComputationThreadPool-1】 在 2023-04-03T13:11:36.236Z 消费了 data【0】
Thread:【RxComputationThreadPool-1】 在 2023-04-03T13:11:36.238Z 完成了消费
```
我们得到了以上结果，那么这种结合上一次的`interval`我们貌似得到了前端同学经常使用的两个工具，一个`定时任务`和一个`延迟任务`，那么有了这两个方便的工具我们能用来做一些什么有趣的东西呢？