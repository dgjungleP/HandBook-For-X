## 测量内存
### 工具
- Linux
	- top
	- ps
- windows
	- perfmon
	- VMMap
### 类型
- 分配的内存
- 保留的内存



### 最小化内存占用
- 堆
- 线程栈
- 代码缓存
- 原生库分配

### 原生内存跟踪(NMT)
> 指令，-XX:NativeMemoryTracking=off|summary|detail  -XX:+PrintNMTStatistic -XX:-AutoShutdownNMT
> 命令行：
> - jcmd pid VM.native_memoty summary
> - jcmd pid VM.native_memory baseline
> - jcmd pid VM.native_memoty summary.diff

##### 内存的使用情况
- 堆本身
- 元数据的原生内存
- 线程栈
- JIT编译代码缓存
- GC使用的堆外内存
- JVM内部操作
	- 直接字节缓冲区
- 符号表引用
- `NMT`(原生内存跟踪)本身的操作内存
- JVM簿记

##### 关键信息
- 总提交大小
- 单个提交大小

### 共享库原生内存
> 通过`System.loadLobrary()`加载共享库
> 导致原生内存被大量使用的操作
> - 使用Inflater对象和Deflater对象
> - 使用NIO缓冲区
> 	- 指令，-XX:MaxDirectMemorySize=N
> 原生内存不会进行压缩的，所以会存在碎片化严重的情况


## 针对于操作系统的JVM优化
### 大页
> 指令，-XX:+UseLargePages


## [Next 线程和同步性能](./Thread.md)