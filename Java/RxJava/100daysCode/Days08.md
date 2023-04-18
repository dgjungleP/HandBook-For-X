	在学习异步和并行处理之前，由于业务原因，我同时又遇到了一个问题，
	当我有很多数据要进行处理时，由于数据量过大，内存可能无法支持到这么多数据，这个时候我需要对数据
	进行分组批处理，那么这个时候`RxJava`能够怎么样轻易的帮助我们完成这项工作呢

### buffer

``` java
@Test  
public void generateFlowable() {  
    Flowable<Integer> rangeFlowable = Flowable.range(1, 100000);  
    rangeFlowable.subscribe(new ConsoleSubscribe());  
}
```
我们假设现在的一个场景，我们有大概10w条数需要批处理，但是如果一次性全部都在内存中处理，我们的内存可能就支撑不住，从而导致堆溢出等情况发生，那么为了缓解这一压力，我们能够想到的就是，希望能够将这10w条数据拆分成可能大的批次，然后进行分批次处理，我们可以用到各种方式对这批数据进行分批，那利用`RxJava`，我们可以如何轻松的做到呢？
``` java
@Test  
public void generateFlowable() {  
    Flowable<Integer> rangeFlowable = Flowable.range(1, 10);  
    Flowable<List<Integer>> bufferFlowable = rangeFlowable.buffer(5);  
    bufferFlowable.subscribe(new ConsoleSubscribe());  
}
```
可以看到就这样简单的一句(由于篇幅原因，我们将数据量改小了)，我们就轻松的办到了
``` console
Thread:【main】 在 2023-04-07T14:13:38.900Z 开启注册消费！
Thread:【main】 在 2023-04-07T14:13:38.916Z 消费了 data【[1, 2, 3, 4, 5]】
Thread:【main】 在 2023-04-07T14:13:38.916Z 消费了 data【[6, 7, 8, 9, 10]】
Thread:【main】 在 2023-04-07T14:13:38.916Z 完成了消费
```
何以很明确的看到，确实达到了分组的效果，确实很方便。除了简单的分组以外，`buffer`还给我们提供了`时间窗口`的功能。
``` java
@Test  
public void generateFlowable2() {  
    Flowable<Integer> rangeFlowable = Flowable.range(1, 10).map(data -> {  
        TimeUnit.MILLISECONDS.sleep(200);  
        return data;  
    }).subscribeOn(Schedulers.io());  
    Flowable<List<Integer>> bufferFlowable = rangeFlowable.buffer(500, TimeUnit.MILLISECONDS);  
    bufferFlowable.subscribe(new ConsoleSubscribe());  
    while (true) {  
    }  
}
```
同时我们得到了由时间窗口分批的结果
``` console
Thread:【main】 在 2023-04-07T14:27:06.453Z 开启注册消费！
Thread:【RxComputationThreadPool-1】 在 2023-04-07T14:27:06.976Z 消费了 data【[1, 2]】
Thread:【RxComputationThreadPool-1】 在 2023-04-07T14:27:07.478Z 消费了 data【[3, 4, 5]】
Thread:【RxComputationThreadPool-1】 在 2023-04-07T14:27:07.974Z 消费了 data【[6, 7]】
Thread:【RxComputationThreadPool-1】 在 2023-04-07T14:27:08.480Z 消费了 data【[8, 9]】
Thread:【RxCachedThreadScheduler-1】 在 2023-04-07T14:27:08.528Z 消费了 data【[10]】
Thread:【RxCachedThreadScheduler-1】 在 2023-04-07T14:27:08.528Z 完成了消费

```
所以这里我们可以看到，针对于一些大流量或者大批量的数据需要进行分批处理的时候，我们可以分方便的适用`RxJava`的`buffer`功能，当然除了执行分批大小和时间窗口以外，他们还能随意组合，以及更多高级功能可以适用，这个后面我们在使用到的时候在进行说明吧。
