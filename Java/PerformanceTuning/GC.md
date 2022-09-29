> 很多时候，我们没有机会重写代码，又面临需要提高Java应用的性能压力，这种情况下对垃圾收集器的调优就变得至关重要

## 概述
### 垃圾收集的步骤
1. 查找不再使用的对象
> 可达性分析，从`GC Root`对象开始搜索，可达就是活跃，不可达就是无效对象
3. 释放对这些对象所管理的内存
4. 对堆的内存布局进行压缩整理
> - 简单的清理不足以合理的时候内存，有时候我们还需要进行内存整理，防止内存碎片
> - `STW`对性能的影响最大，尽量减少这种停顿是最关键的考虑因素
### 垃圾回收器运行时的线程
- `mutator线程`，执行引用程序的逻辑
- GC线程

### 类型
- Serial收集器（单CPU环境）
- Throughout（Parallel）收集器
- Concurrent收集器（CMS）
- G1收集器

## 指令
- -XX:+PrintGCDetails 打印GC日志

## 垃圾收集器
> - 所有的垃圾收集器都遵循同一个方式，划分`老年代`、`新生代`,`新生代`划分为`Eden`和`Survivor`
> - 所有的垃圾收集算法，在对`新生代`进行垃圾回收时都存在`STW`的现象
> - 各种收集器，使用各种不同的方法对`老年代`进行收集和压缩

### 分代垃圾收集器
> 初衷：
> - 处理大量的临时对象 
#### 特性
- 垃圾收集器会在执行回收的时候，暂停所有的应用线程（`STW`）
- 能带来更短的`STW`时间
- 发生的频率更高
- 垃圾收集的时候自动进行了一次压缩整理

### 考虑的因素
> 对单个请求的响应事件有要求

- 单个请求会受停顿的事件影响--对受`Full GC`长时间的停顿影响更大，想尽可能地缩短响应事件，那么选择使用`Concurrent收集器`
- 如果平均响应事件比最大响应事件(百分位)更重要，采用`Throughput收集器`
- 使用`Concurrent收集器`避免长时间的停顿，会额外消耗`CPU`

> 为批量请求的应用选择的原则

- 如果CPU足够强，使用`Concurrent收集器`可以避免发生`Full GC`，可以让任务更快
- 如果CPU有限，使用`Concurrent收集器`额外的CPU消耗会让批量任务消耗更多的时间

### GC算法
- Serial垃圾收集器
> 指令 ， -XX:+UseSerialGC 
> - `单线程`清理
> - 执行任何GC都会`STW`
> - 进行`Full GC`时，还会对老年代空间的对象进行`压缩`
- [Throughput垃圾收集器](./GCAlgorithm/Throughput.md)
> 指令，-XX:+UseParallelOldGC -XX:+UseParallelGC
> - `多线程`回收新生代们可以`多线程`回收老年代
> - 执行任何GC都会`STW`
> - 进行`Full GC`时，还会对老年代空间的对象进行`压缩`
- [CMS收集器](./GCAlgorithm/CMS.md)
> 指令，-XX:+UseParNewGC  -XX:+UseConcMarkSweepGC
> - 多线程清理
> - 在执行Minor GC才会`STW`,对于Full GC使用后台线程定期对老年代进行扫描和回收
> - CPU想好更高
> - 进行`Full GC`时，不会进行任何压缩，所以碎片化严重
> - 碎片化严重或CPU资源不足时，会退化为`Serial收集器`，处理后再恢复
- [G1收集器](./GCAlgorithm/G1.md)
> 指令，-XX:+UseG1GC
> - 多线程清理
> - 在执行Minor GC才会`STW`,对于Full GC使用后台线程定期对老年代进行扫描和回收
> - CPU资源更高
> - 对于老年代的整理，采用的时复制的方式，进行压缩整理(无法完全避免碎片化)
#### 备注
> - 可以通过`Systen.gc()`手动触发GC
> - 通过指令，-XX:+DisableExplicitGC 禁止手动GC
> - 通过jcmd GC.run 可以触发gc
> - 远程方法调用（RMI）,也可以触发gc

### 如何选择GC算法
> - 应用程序的特征
> - 应用的性能目标
> - 可用的硬件

#### GC与批量任务
>- 主要考虑收集器引入的停顿的影响
>- CPU可用资源的影响
#### GC与吞吐量
>- 主要考虑收集器引入的停顿的影响
>- CPU可用资源的影响
#### GC与响应时间
> - 平均时间
> - 90位、99位
#### CMS和G1
> - 当前应用堆大小


## GC调优
- 调整堆的大小
> 指令，-XmsN  -XmxN
> - 过大，影响`STW`的效率
> - 过小，长期用于执行GC对应用代码执行效率降低
> 尽可能通过设定`性能目标`，来调整：
> - 能忍受的`STW`时间
> - 期望的垃圾回收占整个运行时间的百分比
- 代空间的调整
> 指令，-XX:NewRatio=N -XX:NewSize=N -XX:MaxNewSize=N -XmnN
> 新生代过大，发生垃圾回收的频率会降低，但是Full GC会更加频繁
> JVM会`自适应变`化代空间大小
- 永久代和元空间的调整
> 指令，-XX:PermSize=N -XX:MaxPermSize=N  -XX:MetaspaceSize=N -XX:MaxMetaspaceSize=N 
> 保存的数据只对编译器或者JVM的运行时有用
> 与程序的类的数量成正比
> 频繁`重新载入类`的环境(类加载器泄露)，容易造成永久代/元空间耗尽，触发Full GC
- 控制并发
> 指令，-XX:ParallelGCThreads=N
> - 几乎所有的垃圾收集算法中的基本垃圾回收线程数都依据机器上的CPU数目计算得出
> - 多个JVM运行再同一台物理机上是，需特别注意线程数的优化
> - 如果Docker容器有限制，那么就需要线程数的优化
- 自适应调整
> 指令，-XX:+UseAdaptiveSizePolicy -XX:+PrintAdaptiveSizePolicy
> JVM再运行的过程中也会不断地尝试和寻找优化性能地机会
> - 小型应用程序不需要为指定了过大地堆而担心
> - 大部分应用程序不需要担心他们地堆太小
> - 一般不需要关闭自适应，除非已经对堆进行了精细化调优

## 垃圾回收工具
> 分析垃圾回收堆应用程序性能影响最好地方法就是尽量熟悉垃圾回收的日志

- 开启日志
> 指令
> - -verbose:gc 
> - -XX:+PrintGC 
> - -XX:+PrintGCDetails
> - -XX:+PrintGCTimeStamps
> - -XX:+PrintGCDateStamps
- GC日志输出到标准输出
> 指令
> - -Xloggc:filename
- 日志的循环
> 指令
> - -XX:+UseGCLogfileRotation
> - -XX:+NumberOfGCLogfiles=N
> - -XX:+GCLogfileSize=N
- 读取日志信息
> - 工具[GC Histogram](./Tool/GCHistogram.md)可以解析GC日志形成图表
> - `jconsole`可以查看分区的使用情况
> - `jstat` 可以使用 `-gcutil` 查看gc的情况
- 原生内存跟踪工具（NMT）
### 其他指令
>  -XX:MaxGCPauseMilis=N -XX:GCTimeRatio=N

### 总结
- 堆的动态优化是调整堆大小的第一步
- 动态优化后，以动态优化结果开始静态优化是一个不错的选择

## 高级优化

### 晋升和Survivor空间
- 指令，-XX:InitialSurvivorRatio=N
- 指令，-XX:MinSurvivorTatio=N
- 指令，-XX:TargetSurvivorRatio=N
- 指令，-XX:InitailTenuringThresdhold=N
- 指令，-XX:MaxTenuringThreshold=N
- 指令，-XX:+AlwaysTenure
- 指令，-XX:+NeverTenure
> 什么情况下优化什么值，我们需要参考一下晋升统计
> - 指令，-XX:+PrintTenuringDistribution

### 分配大对象
> `大`是相对于JVM中特定的缓冲区(`线程本地分配缓冲区` TLAB)的大小

##### 线程本地分配缓存
> - 指令，-XX:+UseTLAB -XX:+PrintTLAB -XX:TLABSize=N -XX:+ResizeTLAB
> - 每一个线程都有一个本地分配缓存区
> - 在专用的分配区域中，线程在分配对象的时候就不需要进行任何同步（是一种`防止锁竞争`的变体），否则在`共享空间`中分配的话，就需要利用`同步机制`来管理当前空间中的`空闲指针`
> - 本地分配内存很好，不能分配大对象
> - `TLAB`满了之后，JVM所需要做出的抉择
> 	- 清退当前`TLAB`，并重新给相关的线程分配一个新的
> 	- 直接在堆上分配对象
> - `TLAB`大小决定的因素：
> 	- 应用程序中线程数量
> 	- Eden空间的大小
> 	- 线程的分配速率
> - 能从调优中获取收益的程序类型
> 	- 分配很多大对象的应用程序
> 	- 于Eden空间相比，线程数量相对较多的程序

##### 修改线程本地分配缓存的大小
> - 指令, -XX:TLABSize=N -XX:+ResizeTLAB -XX:TLABWasteTargetPercent=N -XX:TLABWasteIncrement=N -XX:MinTLABSize=N

##### 巨型对象
> - 针对于短期巨型对象造成的负面影响，除了改变程序，使其不需要短期的巨型对象以外， 没有其他办法优化
> - 在G1 GC中需要特殊的优化来补偿损失
> 	- 在G1 GC中定义巨型对象为大于等于区域一般大小的对象
> 	- 在并发周期的清理阶段就会被释放

##### G1 GC的区域大小
> - 指令，-XX:G1HeapRegionSize=N
> - 是固定值
> - 由启动时堆大小的最小值确定
> - 调优场景
> 	- 处理巨型对象
> 	- 区域过多（最好让区域的数量接近2048）

##### AggressiveHeap标志
> 不建议使用，甚至会导致性能下降
> - 包含以下几个方面
> 	- 晋升本地分配缓冲区 (`PLAB`)的大小
> 	- 编译策略
> 	- 在`Full GC`之前禁用`Young GC`
> 	- 将GC线程绑定在CPU

##### 全面控制堆的大小 
> - 指令，-XX:MaxRAM=N  -XX:MaxRAMFraction=N -XX:ErgoHeapSizeLimit=N -XX:MinRAMFraction=N

### 新型的GC目标
- 并发的压缩（[ZGC](./GCAlgorithm/ZGC.md)，[Shenandoah](./GCAlgorithm/Shenandoah.md)）
- 无回收（[Epsilon GC](./GCAlgorithm/EpsilonGC.md)）

## 选择什么样的方式优化垃圾回收器
- 你的应用程序能容忍Full GC的停顿吗
- 在默认设置下，你是否已经获得了所需的性能
- 停顿时间是否在某种程度上界定你的预期
- 尽管GC停顿时间很短，但吞吐量仍然上不去
- 是否在使用并发回收器
	- 遇到了`并发模式失败`而导致的Full GC
	- 遇到了`晋升失败`而导致的Full GC


## [Next 堆内存最佳实践](./Heap.md)
