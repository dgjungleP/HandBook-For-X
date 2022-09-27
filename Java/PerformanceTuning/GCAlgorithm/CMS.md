## 具体操作
- 对新生代的对象进行回收（STW）
- 启动一个并发线程对老年代空间的垃圾进行回收
- 如果有必要，CMS会发起Full GC

## 并发收集阶段
- 初始化标记(STW)
> CMS-initial-mark
> 主要任务是找到堆中所有的垃圾回收根节点对象
- 标记阶段
> CMS-concurrent-mark-start
> CMS-concurrent-mark
> 仅作标记工作，不会对堆的使用情况产生实质性变化
- 预清理
> CMS-concurrent-preclean-start
> CMS-concurrent-preclean
- 重新标记(STW)
> CMS-concurrent-abortable-preclean-start
> CMS-concurrent-abortable-preclean
> CMS Final Remark
> 	- weak refs processing
> 	- class unloading
> 	- scrub symbol table
> 	- scrub string table
- 清除
> CMS-concurrent-sweep-start
> CMS-concurrent-sweep
- 并发重置
> CMS-concurrent-reset-start
> CMS-concurrent-reset

### 特殊情况
#### 并发模式失败
> concurrent mode failure
> - 新生代发生垃圾回收，同时老年代没有足够的空间容纳晋升
> - 老年代有足够的空间可以容纳晋升的对象，但是由于空闲空间的碎片化，不能晋升
> - 永久代/元空间用尽，需要回收（默认情况下CMS不会触发回收）

解决办法
- 增大老年代空间，要么只移动部分新生代对象到老年代，要么增加更多的堆空间
- 以更高的频率运行后台回收程序，给后台线程更过的运行机会
	 - 更早地启动并发收集周期
	  > 指令，-XX:CMSInitiatingOccupancyFraction=N  -XX:+UseCMSInitiatingOccupancyOnly
	  - CMS后台线程会持续运行，会消耗大量地CPU时钟，100%地CPU资源，机器必须预留出足够地CPU周期来运行CMS线程
	  - CMS在特定地阶段会STW,所以频繁地无效CMS周期会起到反作用
- 使用更多的后台回收线程
>指令，-XX:ConcGCThreads=N
#### 永久代调优
> 指令，-XX:CMSPermGenSweepingEnabled -XX:CMSInitiatingPermOccupancyFraction=N -XX:+CMSClassUnloadingEnabled
> 频繁部署应用或频繁定义（或回收）类地应用中，容易产生永久代垃圾回收
#### 增量式CMS垃圾收集
> 指令， -XX:CmsIncrementalMode -XX:CMSIncrementalSafetyFactor=N -XX:CMSIncrementalDutyCycleMin=N -XX:CMSIncrementalPacing

	

