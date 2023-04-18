	经过前面的学习，我们已经了解到一些`RxJava`的使用方式，今天我们在这里借助一个小小的案例来综合使用 
	一下我们学到的知识，首先这里我们会使用到`Vert.x`的`RxJava`实现

## 案例

我们需要创建一个Http请求工具，能够满足我们的一些特殊的处理方式
- 并行访问
- 失败重试

## 准备

### 依赖

这里涉及到`Http`请求工具所以我们借用一下`Vert.x`的`RxJava`实现帮助我们简化我们的实现细节。

#### MAVEN

``` xml
	<dependency>  
	    <groupId>io.reactivex.rxjava3</groupId>  
	    <artifactId>rxjava</artifactId>  
	</dependency>  
	  
	<dependency>  
	    <groupId>io.vertx</groupId>  
	    <artifactId>vertx-web-client</artifactId>  
	</dependency>  
	<dependency>  
	    <groupId>io.vertx</groupId>  
	    <artifactId>vertx-rx-java</artifactId>  
	</dependency>
```


### 接口

> Server.java 模拟单个的请求访问
``` java
@FunctionalInterface  
public interface Server {  
  
    Single<?> service(WebClient client);  
}
```
> JClient.java 对请求进行封装   
``` java
public class JClient {  
    private final WebClient client;  
  
    public JClient() {  
        Vertx vertx = Vertx.vertx();  
        this.client = WebClient.create(vertx);  
    } 
    ... 
}
```

## 实现：并行访问

首先我们需要了解到，我们是需要将多个请求访问并行执行,并且每个请求都会得到相应的返回值，那么我们很轻易的想到，我们有两个可以使用的创建方式`merge`和`zip`。所以我们可以很简单的就想到我们的方法声明应该是：
``` java
public Single<List<?>> multiDoServer(Server... servers);

```

	 这里所用到`Single`是另一种`RxJava`的创建方式，使用场景和`Flowable`不一样但是，大体的思路一样地，
	 而且我们用到的`WebClient`使用的就是这个，所以我们先暂时用这个创建方式来示意。

所以我们能够很轻松的想到，接下来我们需要将每一个`server`转化成对应的`RxJava`能够聚合的实体即可，所以我们的完整实现如下：
``` java
public Single<List<?>> multiDoServer(Server... servers) {  
    List<? extends Single<?>> list = Arrays.stream(servers).map(server -> server.service(client))  
            .collect(Collectors.toList());  
    return Single.zip(list, Arrays::asList);  
}
```

	 我们会使用一个OPEN Api来帮助我们模拟网络请求

``` java
public static Server makeService(Integer time) {  
    return client -> client.getAbs("http://httpstat.us/200?sleep=" + time)  
            .rxSend()  
            .doOnSuccess(data -> System.out.println(" " + time + " is finish at" + new Date()))  
            .map(data -> data.bodyAsString() + " with " + time + " at" + new Date());  
}

```

那么接下来我们通过一个简单的测试，来试一下我们创建的并行请求具体使用情况
``` java
@Test  
public void test() throws InterruptedException {  
    JClient client = new JClient();  
    long start = System.currentTimeMillis();  
    Single<List<?>> server = client  
            .multiDoServer(makeService(1000), makeService(2000), makeService(1000), makeService(1000), makeService(3000));  
    Subscription subscribe = server.subscribe();  
    while (!subscribe.isUnsubscribed()) {  
  
    }  
    System.out.println("Used:" + (System.currentTimeMillis() - start) + " ms");  
}
  
```

我们得到以下结果：
``` console
 1000 is finish atMon Apr 10 22:31:13 CST 2023
 1000 is finish atMon Apr 10 22:31:13 CST 2023
 1000 is finish atMon Apr 10 22:31:13 CST 2023
 2000 is finish atMon Apr 10 22:31:14 CST 2023
 3000 is finish atMon Apr 10 22:31:15 CST 2023
Used:4613 ms
```

我们可以看到，它并不是按照我们添加servise的顺序取请求的，而且总共消耗了4.6s，小于五个请求的延迟总和8s,，其中有一些时间消耗属于`网络I/O`和`Vert.x`内部的调度，所以总和来看，我们确实简单的实现了一个并行请求的Client，可能还有很多需要改进的地方。