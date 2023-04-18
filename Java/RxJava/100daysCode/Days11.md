
## 实现： 失败重试

	接上次的案例，我们需要构建一个失败请求的后的重试方法。

很简单我们能想到的方法声明：
``` java
public Single<?> retryableDoServer(Server server, Long times)
```

他的具体实现也很简单，因为我们能够在`RxJava`中找到一个我们还没有学习过的转换方法`retry`

``` java
public Single<?> retryableDoServer(Server server, Long times) {  
    return server.service(client).retry(times);  
}
```
至此他的重试方式我们就完成了，那么接下来我们对这个方法进行测试。  

	我们会使用一个OPEN Api来帮助我们模拟网络请求

``` java
public static Server makeService(Integer time) {  
    return client -> client  
            .getAbs("http://httpstat.us/200?sleep=" + time)  
            .rxSend()  
            .doOnSuccess(data -> System.out.println(" " + time + " is finish at" + new Date()))  
            .timeout(900, TimeUnit.MILLISECONDS)  
            .doOnError(data -> System.err.println(Thread.currentThread().getName() + ":" + data))  
            .map(data -> data.bodyAsString() + " with " + time + " at" + new Date());  
}
```
我们在里面加入了一个超时，当我们请求等待的时间超过阈值的时候，这个请求就会失败，我们先试一下如果我们不超过等待时间

``` java 
@Test  
public void testRetry() throws InterruptedException {  
    JClient client = new JClient();  
    Single<?> single = client.retryableDoServer(makeService(500), 3L);  
    Subscription subscribe = single  
            .subscribe(System.out::println, System.err::println);  
    while (!subscribe.isUnsubscribed()) {  
  
    }  
}
```
我们会得到以下结果
``` console
RxComputationScheduler-1:java.util.concurrent.TimeoutException
RxComputationScheduler-2:java.util.concurrent.TimeoutException
 500 is finish atMon Apr 10 22:50:23 CST 2023
200 OK with 500 atMon Apr 10 22:50:23 CST 2023
```

可以看到，虽然我们的请求延迟并没有超过等待时间，但是它还是给我们报错了，并且帮我们重试了2次后成功了，这样变相的证明了我们的重试的成功的，至于报错信息，有可能是因为我们用的这个OPEN API 不太稳定的原因。那么当我们把请求等待时间改变成2000ms：
``` java
Single<?> single = client.retryableDoServer(makeService(2000), 3L);
```
我们会得到以下结果
``` console
RxComputationScheduler-1:java.util.concurrent.TimeoutException
RxComputationScheduler-2:java.util.concurrent.TimeoutException
RxComputationScheduler-3:java.util.concurrent.TimeoutException
RxComputationScheduler-4:java.util.concurrent.TimeoutException
java.util.concurrent.TimeoutException
```
可以看到`RxJava`除开我们自己请求的一次以外，确实帮我们重试了3次，符合我们的预期，并且在第三次重试失败之后，结束了当前的信息消费。
至此我们的一个多功能的Client就完成了，后期如果还有其他的实现，或者补充修改的地方我们会继续持续更新。

