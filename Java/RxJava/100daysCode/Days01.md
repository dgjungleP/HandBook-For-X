
	在我的工作中，我使用最多的场景就是用RxJava帮助我处理异步任务的问题，所以对于RxJava的起步我应该是从`Flowable`开始切入的。那么这里就浅讲一下`Flowable`的使用方式（总的来说RxJava的几乎所有的`被观察者`都是这样创建的）。
> 它可能不对，但是我确实是这样入坑的




``` java
@Test  
public void createFlowable() {  
    Flowable<Integer> flowable = Flowable.just(1);  
}
```

好了这样的一个简单的包含一个可被监控元素的Flowable的被监听者已经创建好了，当运行代码的时候，会发现什么也没有发生，是因为只有被监听者准备好了，他还没有被监听。所以我们就需要让他被监听起来。

``` java
@Test  
public void createFlowable() {  
    Flowable<Integer> flowable = Flowable.just(1);  
    flowable.subscribe();  
}
```

这样运行之后发现，还是什么变化也没有，是因为监听者并没有做任何的事情，那么就需要给他定制一个当监听到对应的数据的时候的一个行为。

``` java
@Test  
public void createFlowable() {  
    Flowable<Integer> flowable = Flowable.just(1);  
    Disposable subscribe = flowable.subscribe(System.out::println);  
}
```

这样执行后，我们就会发现，我们声明了他只需要简单的打印一下拿到的数据就可以了，运行之后我们也拿到了对应的结果
``` console
1
```

至此一个简易的RxJava的模型就已经完成了，他可以说是RxJava的全部内容，也可以说是冰山一角，剩下的中间操作、监听者的配置以及`Disposable`则是接下来需要深入理解的东西。
  

代码地址:[100daysCode](https://github.com/dgjungleP/100days-code-round1)