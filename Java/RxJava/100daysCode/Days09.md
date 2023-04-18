	在之前的操作中，我们经常需要为每一个Flowable进行模拟延迟的时候，都使用了一个`map`的操作符，那么今
	天，我们就来看一下这个map到底可以为我们做些什么。

### map

如果大家都使用过Java8提供的`Stream`的话，那么多以这个map就很熟悉了，它就是可以让我们在对流的处理过程中，可以将流中的数据转化成我们所需要的数据，那么`RxJava`提供的map呢？
``` java
@Test  
public void generateFlowable() {  
    Flowable<Integer> mapFlowable = Flowable.range(1, 10).map(data -> 2 * data);  
    mapFlowable.subscribe(new ConsoleSubscribe());  
}
```
我们可以看到结果，确实如我们所想，它确实可以将我们的数据进行一些处理：
``` console
Thread:【main】 在 2023-04-07T14:37:58.044Z 开启注册消费！
Thread:【main】 在 2023-04-07T14:37:58.044Z 消费了 data【2】
Thread:【main】 在 2023-04-07T14:37:58.044Z 消费了 data【4】
Thread:【main】 在 2023-04-07T14:37:58.044Z 消费了 data【6】
Thread:【main】 在 2023-04-07T14:37:58.044Z 消费了 data【8】
Thread:【main】 在 2023-04-07T14:37:58.044Z 消费了 data【10】
Thread:【main】 在 2023-04-07T14:37:58.044Z 消费了 data【12】
Thread:【main】 在 2023-04-07T14:37:58.044Z 消费了 data【14】
Thread:【main】 在 2023-04-07T14:37:58.044Z 消费了 data【16】
Thread:【main】 在 2023-04-07T14:37:58.044Z 消费了 data【18】
Thread:【main】 在 2023-04-07T14:37:58.044Z 消费了 data【20】
Thread:【main】 在 2023-04-07T14:37:58.044Z 完成了消费
```

那么讲到了`map`肯定也会联想到`flatmap`,那么`RxJava`中的`flatmap` 是不是也和`Stream`提供的功能类似呢

### flatmap

从方法签名中我们可以看到，它确实类似于`Stream`,是将当前的数据转化成另一个`Flowable`并展开使用
- `public final <@NonNull R> Flowable<R> flatMap(@NonNull Function<? super T, @NonNull ? extends Publisher<? extends R>> mapper)`
我们尝试着使用一下:
``` java
@Test  
public void generateFlowable2() {  
    Flowable<Integer> mapFlowable = Flowable.range(1, 4).flatMap(data -> Flowable.range(1, data));  
    mapFlowable.subscribe(new ConsoleSubscribe());  
}
```
得到了我们预期的结果：
``` console
Thread:【main】 在 2023-04-07T14:43:47.394Z 开启注册消费！
Thread:【main】 在 2023-04-07T14:43:47.409Z 消费了 data【1】
Thread:【main】 在 2023-04-07T14:43:47.409Z 消费了 data【1】
Thread:【main】 在 2023-04-07T14:43:47.409Z 消费了 data【2】
Thread:【main】 在 2023-04-07T14:43:47.409Z 消费了 data【1】
Thread:【main】 在 2023-04-07T14:43:47.409Z 消费了 data【2】
Thread:【main】 在 2023-04-07T14:43:47.409Z 消费了 data【3】
Thread:【main】 在 2023-04-07T14:43:47.409Z 消费了 data【1】
Thread:【main】 在 2023-04-07T14:43:47.409Z 消费了 data【2】
Thread:【main】 在 2023-04-07T14:43:47.409Z 消费了 data【3】
Thread:【main】 在 2023-04-07T14:43:47.409Z 消费了 data【4】
Thread:【main】 在 2023-04-07T14:43:47.409Z 完成了消费
```

但是`Flowable`中的`flatmap`提供了更多其他的重载方法，可以说更加丰富，我们在后续的使用中遇到了，再去细细斟酌