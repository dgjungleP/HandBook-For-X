	之前我们的创建都是围绕着一个`Flowable`来进行的，那么我们可以将多个`Flowable`组合在一起么，
	例如我们日常工作中的一种场景，当我们要去组装一条数据的时候，由于历史或者设计原因，我们不得
	不进行多次网络调用或者计较耗时的操作，但是每个操作基本上都是可以异步执行的，那么对于这种情况
	`RxJava`又能怎么样帮助我们简单的来处理呢？

### merge
从方法声明中我们看到有一个叫做`merge`的静态方法，他的入参就是我们在学习`subscribe`的时候看到的`Publisher`：
``` java 
public static <@NonNull T> Flowable<T> merge(@NonNull Iterable<@NonNull ? extends Publisher<? extends T>> sources)
```
而在观察我们Flowable的类声明的时候我们可以看到以下关系：
``` java
public abstract class Flowable<@NonNull T> implements Publisher<T>
```

所以，merge方法可以将我们的多个Flowable聚合在一起，至于它有什么效果，我们先动手试一下：
``` java
@Test  
public void generateFlowable() {  
    List<Publisher<Integer>> publisherList = new ArrayList<>();  
    for (int i = 0; i < 10; i++) {  
        publisherList.add(Flowable.just(i));  
    }  
    Flowable<Integer> mergeFlowable = Flowable.merge(publisherList);  
    mergeFlowable.subscribe(new ConsoleSubscribe());  
  
}
```
我们得到了以下结果：
``` console
Thread:【main】 在 2023-04-04T14:42:53.590Z 开启注册消费！
Thread:【main】 在 2023-04-04T14:42:53.605Z 消费了 data【0】
Thread:【main】 在 2023-04-04T14:42:53.605Z 消费了 data【1】
Thread:【main】 在 2023-04-04T14:42:53.605Z 消费了 data【2】
Thread:【main】 在 2023-04-04T14:42:53.605Z 消费了 data【3】
Thread:【main】 在 2023-04-04T14:42:53.605Z 消费了 data【4】
Thread:【main】 在 2023-04-04T14:42:53.605Z 消费了 data【5】
Thread:【main】 在 2023-04-04T14:42:53.605Z 消费了 data【6】
Thread:【main】 在 2023-04-04T14:42:53.605Z 消费了 data【7】
Thread:【main】 在 2023-04-04T14:42:53.605Z 消费了 data【8】
Thread:【main】 在 2023-04-04T14:42:53.605Z 消费了 data【9】
Thread:【main】 在 2023-04-04T14:42:53.605Z 完成了消费
```
可以看出merge看起来是将所有的数据，进行了有机的聚合，甚至于还是有顺序的，那么我们稍微再做一点尝试，为么个`Flowable`增加一点随机的延迟，这里会用到`map`操作
``` java
@Test  
public void generateFlowable() {  
    List<Publisher<Integer>> publisherList = new ArrayList<>();  
    for (int i = 0; i < 10; i++) {  
        publisherList.add(Flowable.just(i).map(data -> {  
            TimeUnit.MILLISECONDS.sleep((long) (200 + Math.random() * 1000));  
            return data;  
        }));  
    }  
    Flowable<Integer> mergeFlowable = Flowable.merge(publisherList);  
    mergeFlowable.subscribe(new ConsoleSubscribe());  
  
}
```

结果为：
``` console
Thread:【main】 在 2023-04-04T14:49:24.360Z 开启注册消费！
Thread:【main】 在 2023-04-04T14:49:24.840Z 消费了 data【0】
Thread:【main】 在 2023-04-04T14:49:25.165Z 消费了 data【1】
Thread:【main】 在 2023-04-04T14:49:25.487Z 消费了 data【2】
Thread:【main】 在 2023-04-04T14:49:26.028Z 消费了 data【3】
Thread:【main】 在 2023-04-04T14:49:26.371Z 消费了 data【4】
Thread:【main】 在 2023-04-04T14:49:27.210Z 消费了 data【5】
Thread:【main】 在 2023-04-04T14:49:28.257Z 消费了 data【6】
Thread:【main】 在 2023-04-04T14:49:28.702Z 消费了 data【7】
Thread:【main】 在 2023-04-04T14:49:28.904Z 消费了 data【8】
Thread:【main】 在 2023-04-04T14:49:29.696Z 消费了 data【9】
Thread:【main】 在 2023-04-04T14:49:29.696Z 完成了消费
```
从结果中我们看得出来，它确实是按照我们添加`Flowable`的顺序进行的消息消费，纵使每个消息都有各自的延迟，它还是会等待着前一个消息消费完成，才进行后面的消息消费，这样是不符合我们的预期的，我们希望的是这10个消息能够尽可能快地在同一时间并行消费，那么我们有没有什么办法做到这一点呢？
	答案是没有，对于`merge`的定义，其实只是将所有`Publisher`中所需要传递的数据扁平化，供给给下游使
	用。
所以它适用的场景，是对于多方`同源数据`或者`异构数据`进行合并处理，以满足下游的消费。、
所以我们还得继续跟踪`Flowable`的下一种创建方式
