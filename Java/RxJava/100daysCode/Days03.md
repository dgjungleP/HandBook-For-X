经过昨天的学习，我们认识到了可以如何去消费我们`Flowable`创建出来的消息，那么现在存在一个疑问，那么它是在那个线程去执行的呢。我们用一个新的方式`range`去创建一个例子尝试一下
``` java
@Test  
public void testSchedule() {  
    Flowable<Integer> flowable = Flowable.range(0, 10);  
  
    Disposable subscribe = flowable  
            .subscribe(data -> System.out  
                    .println("Thread:【" + Thread.currentThread().getName() + "】 消费了 data【" + data + "】"));  
  
}
```
结果：
``` console
Thread:【main】 消费了 data【0】
Thread:【main】 消费了 data【1】
Thread:【main】 消费了 data【2】
Thread:【main】 消费了 data【3】
Thread:【main】 消费了 data【4】
Thread:【main】 消费了 data【5】
Thread:【main】 消费了 data【6】
Thread:【main】 消费了 data【7】
Thread:【main】 消费了 data【8】
Thread:【main】 消费了 data【9】
```
那么如果我们要让他`异步执行`该怎么办呢？经过对API的排查我们发现了几个特别的方法签名：
> 1.  `public final Flowable<T> subscribeOn(@NonNull Scheduler scheduler)`
> 2.  `public final Flowable<T> subscribeOn(@NonNull Scheduler scheduler, boolean requestOn)`
> 3.  `public final Flowable<T> unsubscribeOn(@NonNull Scheduler scheduler)`

从这个`Scheduler`的文档里面我们发现这样一句话：
	You can get various standard, RxJava-specific instances of this class via the static methods of the io.reactivex.rxjava3.schedulers.Schedulers utility class.

那么在不太熟悉的情况下，我们就按照官方文档的建议我们去看一下这个`Schedulers`里面携带了什么标准内容给我们使用：

``` java
@NonNull  
static final Scheduler IO;  
  
@NonNull  
static final Scheduler COMPUTATION;  

@NonNull  
static final Scheduler SINGLE;  
   
@NonNull  
static final Scheduler TRAMPOLINE;  
  
@NonNull  
static final Scheduler NEW_THREAD;
```

我们分别来使用一下看一下效果

### IO
``` java
@Test  
public void testScheduleIo() {  
    Flowable<Integer> flowable = Flowable.range(0, 10);  
    flowable = flowable  
            .subscribeOn(Schedulers.io());  
    Disposable subscribe = flowable  
            .subscribe(data -> System.out  
                    .println("Thread:【" + Thread.currentThread().getName() + "】 消费了 data【" + data + "】"));  
}
```

#### 结果：
``` console
Thread:【RxCachedThreadScheduler-1】 消费了 data【0】
Thread:【RxCachedThreadScheduler-1】 消费了 data【1】
Thread:【RxCachedThreadScheduler-1】 消费了 data【2】
Thread:【RxCachedThreadScheduler-1】 消费了 data【3】
Thread:【RxCachedThreadScheduler-1】 消费了 data【4】
Thread:【RxCachedThreadScheduler-1】 消费了 data【5】
Thread:【RxCachedThreadScheduler-1】 消费了 data【6】
Thread:【RxCachedThreadScheduler-1】 消费了 data【7】
Thread:【RxCachedThreadScheduler-1】 消费了 data【8】
Thread:【RxCachedThreadScheduler-1】 消费了 data【9】
```
#### 使用的结论：
建立一单独的线程来处理数据，看起来时在背后建立了一个线程池，那么我们看一下官方文档是怎么介绍的
	Returns a default, shared Scheduler instance intended for IO-bound work.
	This can be used for asynchronously performing blocking IO.
	The implementation is backed by a pool of single-threaded ScheduledExecutorService instances that will try to reuse previously started instances used by the worker returned by Scheduler


### COMPUTATION
``` java
@Test  
public void testScheduleIo() {  
    Flowable<Integer> flowable = Flowable.range(0, 10);  
    flowable = flowable  
            .subscribeOn(Schedulers.io());  
    Disposable subscribe = flowable  
            .subscribe(data -> System.out  
                    .println("Thread:【" + Thread.currentThread().getName() + "】 消费了 data【" + data + "】"));  
}
```

#### 结果：
``` console
Thread:【RxComputationThreadPool-1】 消费了 data【0】
Thread:【RxComputationThreadPool-1】 消费了 data【1】
Thread:【RxComputationThreadPool-1】 消费了 data【2】
Thread:【RxComputationThreadPool-1】 消费了 data【3】
Thread:【RxComputationThreadPool-1】 消费了 data【4】
Thread:【RxComputationThreadPool-1】 消费了 data【5】
Thread:【RxComputationThreadPool-1】 消费了 data【6】
Thread:【RxComputationThreadPool-1】 消费了 data【7】
Thread:【RxComputationThreadPool-1】 消费了 data【8】
Thread:【RxComputationThreadPool-1】 消费了 data【9】
```
#### 使用的结论：
建立一单独的线程来处理数据，看起来时在背后建立了一个线程池，那么我们看一下官方文档是怎么介绍的
	Returns a default, shared Scheduler instance intended for computational work.
	This can be used for event-loops, processing callbacks and other computational work.
	It is not recommended to perform blocking, IO-bound work on this scheduler. Use io() instead.
从说明里面看，如果执行过程中没有阻塞行为的话，可以使用这个类型


### SINGLE
``` java
@Test  
public void testScheduleComputation() {  
    Flowable<Integer> flowable = Flowable.range(0, 10);  
    flowable = flowable  
            .subscribeOn(Schedulers.computation());  
    Disposable subscribe = flowable  
            .subscribe(data -> System.out  
                    .println("Thread:【" + Thread.currentThread().getName() + "】 消费了 data【" + data + "】"));  
}
```

#### 结果：
``` console
Thread:【RxSingleScheduler-1】 消费了 data【0】
Thread:【RxSingleScheduler-1】 消费了 data【1】
Thread:【RxSingleScheduler-1】 消费了 data【2】
Thread:【RxSingleScheduler-1】 消费了 data【3】
Thread:【RxSingleScheduler-1】 消费了 data【4】
Thread:【RxSingleScheduler-1】 消费了 data【5】
Thread:【RxSingleScheduler-1】 消费了 data【6】
Thread:【RxSingleScheduler-1】 消费了 data【7】
Thread:【RxSingleScheduler-1】 消费了 data【8】
Thread:【RxSingleScheduler-1】 消费了 data【9】
```
#### 使用的结论：
建立一单独的线程来处理数据，看起来时在背后建立了一个线程池，那么我们看一下官方文档是怎么介绍的
	Returns a default, shared, single-thread-backed Scheduler instance for work requiring strongly-sequential execution on the same background thread.


### TRAMPOLINE
``` java
@Test  
public void testScheduleComputation() {  
    Flowable<Integer> flowable = Flowable.range(0, 10);  
    flowable = flowable  
            .subscribeOn(Schedulers.computation());  
    Disposable subscribe = flowable  
            .subscribe(data -> System.out  
                    .println("Thread:【" + Thread.currentThread().getName() + "】 消费了 data【" + data + "】"));  
}
```

#### 结果：
``` console
Thread:【main】 消费了 data【0】
Thread:【main】 消费了 data【1】
Thread:【main】 消费了 data【2】
Thread:【main】 消费了 data【3】
Thread:【main】 消费了 data【4】
Thread:【main】 消费了 data【5】
Thread:【main】 消费了 data【6】
Thread:【main】 消费了 data【7】
Thread:【main】 消费了 data【8】
Thread:【main】 消费了 data【9】
```
#### 使用的结论：
从结果来看，它还是使用了当前线程处理，因为现在的执行线程是Main线程，所以我们拿到的也是Main线程
	The default implementation's Scheduler.scheduleDirect(Runnable) methods execute the tasks on the current thread without any queueing and the timed overloads use blocking sleep as well

### NEW_THREAD
``` java
@Test  
public void testScheduleComputation() {  
    Flowable<Integer> flowable = Flowable.range(0, 10);  
    flowable = flowable  
            .subscribeOn(Schedulers.computation());  
    Disposable subscribe = flowable  
            .subscribe(data -> System.out  
                    .println("Thread:【" + Thread.currentThread().getName() + "】 消费了 data【" + data + "】"));  
}
```

#### 结果：
``` console
Thread:【RxNewThreadScheduler-1】 消费了 data【0】
Thread:【RxNewThreadScheduler-1】 消费了 data【1】
Thread:【RxNewThreadScheduler-1】 消费了 data【2】
Thread:【RxNewThreadScheduler-1】 消费了 data【3】
Thread:【RxNewThreadScheduler-1】 消费了 data【4】
Thread:【RxNewThreadScheduler-1】 消费了 data【5】
Thread:【RxNewThreadScheduler-1】 消费了 data【6】
Thread:【RxNewThreadScheduler-1】 消费了 data【7】
Thread:【RxNewThreadScheduler-1】 消费了 data【8】
Thread:【RxNewThreadScheduler-1】 消费了 data【9】
```
#### 使用的结论：
建立一单独的线程来处理数据，看起来时在背后建立了一个线程池，那么我们看一下官方文档是怎么介绍的
	Returns a default, shared Scheduler instance that creates a new Thread for each unit of work.

综上俩看，基本上每一个特定的标准都是创建了一个线程池来办这个事情，那么现实真的是不是呢，我们后面在真正的使用的时候，我们再去追一下源码看看具体情况，但是现在有一个问题，如果我自己有一个线程池，我能不能使用我自己的线程池呢，从方法签名中我们有找到几个特定的方法：

> 1.  `public static Scheduler from(@NonNull Executor executor)`
> 2.  `public static Scheduler from(@NonNull Executor executor, boolean interruptibleWorker)`
> 3.  `public static Scheduler from(@NonNull Executor executor, boolean interruptibleWorker, boolean fair`

从这里我们看到，确实我们能够使用自己的线程池，所以接下来我们看一下我们应该如何去使用我们现在学习过的知识点。
