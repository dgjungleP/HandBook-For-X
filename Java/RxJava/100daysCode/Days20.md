> 前面我们讲过了很多的`SPI`的使用，其实RPC就是其集大成者，在继续探索之前，我们先来看一下
> `Dubbo`利用`SPI`如何快速的帮助我们创建一个RPC的

## 准备
首先我们只是简单的尝试在本地构建一个简单的PRC，所以我们并没有采用`SpringBoot`来集成`Dubbo`，而是采用最小单位，使用`Spring`来做集成,所以我们需要添加对应的依赖，（我们暂时不用最新的版本）

``` xml
<dependencies>  
    <dependency> 
        <groupId>org.apache.dubbo</groupId>  
        <artifactId>dubbo</artifactId>  
        <version>2.7.7</version>  
    </dependency>    
    <dependency>        
	    <groupId>org.springframework</groupId>  
        <artifactId>spring-core</artifactId>  
        <version>4.3.16.RELEASE</version>  
    </dependency>
</dependencies>
```
然后我们则需要创建一个需要具备RPC能力的Sevice,并给出一个简单的实现类
``` java
public interface DemoService {  
    String sayHello(String name);  
}
public class DemoServiceImpl implements DemoService {  
    @Override  
    public String sayHello(String name) {  
        System.out.println("【" + new SimpleDateFormat("HH:mm:ss")  
                .format(new Date()) + "】 Hello " + name + ". request from:【" + RpcContext.getContext()  
                .getRemoteAddressString() + "】");  
        return "Hello " + name + ". request from:【" + RpcContext.getContext()  
                .getRemoteAddressString() + "】";  
    }  
}
```
那么接下来，我们就需要将这个Sevice暴露在RPC的访问中，那么我们就采用XML的配置方式来配置这个服务
``` xml
<context:property-placeholder></context:property-placeholder>  

<dubbo:application name="provider"></dubbo:application>  

<dubbo:registry address="multicast://224.5.6.7:1234"></dubbo:registry>  

<dubbo:protocol name="dubbo" port="20880"></dubbo:protocol>  

<bean id="demoService" class="com.jungle.dubbo.api.impl.DemoServiceImpl"></bean>  

<dubbo:service interface="com.jungle.dubbo.api.DemoService" ref="demoService" group="test"></dubbo:service>  

```
我们可以看到里面有一个我们不太熟悉的东西`multicast`,这是一种网络协议，它允许将一台主机发送的数据通过网络路由器和交换机复制到多个加入此组播的主机，因为我们知道Dubbo必备的就是一个注册中心，这里就巧妙地利用组播协议来充当一个注册中心，关于组播具体地一些细则和相关知识，我们在后续学习网络知识地时候再做深入了解（现在先记在小本本上）。

同时服务提供出来了，那么我们也需要一个能够找到它地消费者来进行RPC操作：
``` xml
<context:property-placeholder></context:property-placeholder>  
  
<dubbo:application name="consumer"></dubbo:application>  
  
<dubbo:reference interface="com.jungle.dubbo.api.DemoService" id="demoService" check="false" url="127.0.0.1:20880"  group="test"></dubbo:reference>
```
那么这里是采用直连地方式去进行访问，当然在产线上我们肯定不是这样做

## 使用
准备工作做好了，那么我们现在就需要两个启动类，来分别启动我们地`Provider`和`Consumer`
``` java
public class DemoProvider {  
  
    public static void main(String[] args) throws InterruptedException {  
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("/dubbo-provider.xml");  
  
        context.start();  
        System.out.println("Provider service start");  
        new CountDownLatch(1).await();  
    }  
}
public class DemoConsumer {  
  
    public static void main(String[] args) throws InterruptedException {  
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("/dubbo-consumer.xml");  
  
        context.start();  
        DemoService service = (DemoService) context.getBean("demoService");  
        String hello = service.sayHello("Jungle");  
        System.out.println("hello = " + hello);  
    }  
}
```
同时也非常简单，这里要说明一下我们看到在`Povider`中我们使用了一个`new CountDownLatch(1).await()`,利用这种方式让主线程永远处于锁释放的状态，从而不会因为执行完`main`函数之后主线程就挂掉了。
分别执行完之后我们会得到以下结果
``` console
Provider service start
【23:01:36】 Hello Jungle. request from:【10.1.1.1:14348】
```
那么，一次简单的Dubbo帮助我们构造的RPC就完成了，那么至于Dubbo是怎么利用`SPI`来完成这个过程的，我们接下来进一步再看看吧。