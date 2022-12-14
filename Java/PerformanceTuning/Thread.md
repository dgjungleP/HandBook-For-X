## ThreadPool 
- 优化线程池的时候，不能盲目的添加新的线程，可能会造成性能更差
	- I/O密集型任务
	- CPU密集型任务
- 给ThreadPoolExcutor设置更简单的选项会带来最佳的（可预测的）性能
- 尽量避免使用Excutors来创建线程池

### ForkJoinPool
#### 适用场景
- 算法的合并部分会做一些工作
- 算法的叶子计算所执行的工作足以抵消创建任务的开销
- 任务分割不均匀的时候
##### 注意
- fork/join范式的递归应该在什么时候结束（对性能有所影响）
- 使用自动化并行化特征的时候，使用的是公共的ForkJoinPool的实例，注意调整大小
##### 配置
- Djava.util.concurrent.ForkJoinPool.common.parallrlism=N

## 线程同步

### 工具
- synchronized关键字
- java.util.concurrent包
- `CAS`
- volatile

### 同步的代价

#### 同步和可扩展性
> 一个应用程序被拆分为多线程运行后，能实现的`加速比`由`阿姆达尔定律`定义

#### 锁对象的开销
- 获取同步锁的开销
- 寄存器的刷新(取决于Java内存模型)

### 避免同步
- 在每个线程栈中使用不同的对象，避免访问对象竞争(例如，使用`TreadLocal`)
- 使用基于`CAS`的替代方案
	- 比较CAS和传统同步的性能的准则
		- 对资源的访问无竞争，CAS较优，如果始终没有竞争，不保护更好
		- 对资源的访问有轻度或适度的竞争，CAS较优
		- 资源访问剧烈，传统的同步有时候会更优
		- 当只读不写的情况，CAS不会受到竞争的影响

### 伪共享
> 缓存行共享，与CPU处理缓存的方式有关

#### 避免方式
- 检查代码修改，减少涉及到的变量的频繁写入
- 对变量进行填充，避免加载到同一个缓存行
- `@Contended`注解
	- 指令 -XX:+RestrictContemded -XX:-EnableContended
- 将数据移动到具备，稍后再存储他们

### JVM线程优化
>  每个线程都有一个`原生栈`，操作系统会在这里存储线程的`调用栈信息`

#### 优化线程栈大小
> 指令，-Xss=N
> 在内存空间不足的机器上，可以减少线程栈的大小

#### 偏向锁
> 指令，-XX:-UseBiasedLocking
> 如果一个线程最近使用了某个锁，那么下次它执行被`同一个锁保护的代码`的时候,处理器的缓存更有可能还包含`线程所需要的数据`
> 在使用`线程池`的应用程程序中，使用偏向锁，往往会让机器性能下降

#### 线程优先级
> 只会给操作系统一种提示，并不真正决定它的优先级和运行频率
> 如果需要严格的优先级，需要使用`应用程序逻辑`来控制

### 监控线程和锁

- 查看线程
- 查看阻塞线程（使用性能分析器）
	- 阻塞线程和JFR
	- 阻塞线程和jstack
	> - JVM只在`安全点转储线程栈`
	> - 栈的转储是`一次转储一个线程`



##  拓展
- 读取JAR文件(SAX解析器)产生了竞争我们应该怎么办
> 设置 -Djavax.xml.parsers.SAXParserFactory
- 线程性能的最佳实践准则
	- 管理线程数
	- 限制同步影响



## [Next Java服务器](./JavaService.md)
