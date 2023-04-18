	上一次我浅用了以下merge方法，它确实可以把多个`Flowable`聚合在一起，但是却不能满足我们预设的要求，
	将多个耗时操作异步执行，并且，并利用其结果去组装我们想要的下游数据，那么接下来，我们再看一下另一
	个能够组合`Flowable`的创建方式

### zip

我们可以看到最简单易用的`zip`的方法签名如下
- `public static <@NonNull T, @NonNull R> Flowable<R> zip(@NonNull Iterable<@NonNull ? extends Publisher<? extends T>> sources, @NonNull Function<? super Object[], ? extends R> zipper)`
可以很明显的看到，我们不仅要将元数据放置进来，还需要提供一个`zipper`的方法，对放入的元数据进行处理
``` java
@Test  
public void generateFlowable() {  
    List<Publisher<Integer>> publisherList = new ArrayList<>();  
    for (int i = 0; i < 10; i++) {  
        publisherList.add(getDelayFlowable(i, 0));  
    }  
    Flowable<Integer> zipFlowable = Flowable  
            .zip(publisherList, (dataList) -> Arrays.stream(dataList).mapToInt(data -> {  
                if (data instanceof Number) {  
                    return ((Number) data).intValue();  
                } else {  
                    return 0;  
                }  
            }).sum());  
    zipFlowable.subscribe(new ConsoleSubscribe());  
}
```
以上例子，我们拿到了结果
``` console
Thread:【main】 在 2023-04-07T13:54:26.106Z 开启注册消费！
Thread:【main】 在 2023-04-07T13:54:26.106Z 消费了 data【45】
Thread:【main】 在 2023-04-07T13:54:26.106Z 完成了消费
```
确实它帮我们将0-9之间的数的总和进行的求解，那么我们可以看到`zip`可以让我们对多个`Flowable`的数据进行求解，然后同过`zipper`函数，让我们得到我们想要的数据，这种针对于有时我们需要`向多方拿取数据，然后整合数据`的场景中我们可以适用zip来帮助我们。
那么接下一个问题，所有入参的`Flowable`是并行执行的吗？为了验证这一点，我们给每个`Flowable`增加一个1秒的延迟，如果是并行执行的，那么我们将在1-2秒内拿到我们想要的结果。
``` console
Thread:【main】 在 2023-04-07T13:59:15.050Z 开启注册消费！
Thread:【main】 在 2023-04-07T13:59:25.145Z 消费了 data【45】
Thread:【main】 在 2023-04-07T13:59:25.145Z 完成了消费
```
结果我们看到，我们足足等了10秒，也就是所有的`Flowable`并不是并行执行的，而且`zip`需要等待到所有的`Flowable`准备就绪之后才能执行`zipper`函数。
所以接下来我们如何让每一个`Flowable`并行执行呢？接下来我们将继续取探索。