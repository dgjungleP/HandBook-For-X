## 概念
`Just-In-Time`

### 编译器类型
- Client
- Server
- 分层编译
> 指令，-XX:-TieredCompilation

### 考虑因素
- 优化启动
- 优化批处理
- 优化长时间运行的应用

### 编译器调优的方式
- 调优代码缓存
> - 指令 -XX:ReservedCodeCache=N
- 编译阈值
> - 指令 -XX:CompileThreshold=N (N=`回边计数器`+`方法调用计数器`) （过时了）
> - 调整的原因：
> 	- 节约应用热身的事件
> 	- 使原本不能被编译器编译的方法得以编译
> 新指令，-XX:Tier3InvocationThreshold=N -XX:Tier4InvocationThreshold=N
> - 禁止分层编译时，这些阈值才有意义 
- 检测编译过程
> - 指令 -XX:+PrintCompilation
- 编译线程
> - 指令 -XX:CICompilerCount=N
> - 指令 -XX:BackgroundCompilation
> - 指令 -XBatch
> 在Docker容器中运行旧版本的JDK8可能需要手动设置值
- 内联（对编译器最有利的优化）
> - 指令 -XX:+Inline
> - 指令 -XX:+PrintInlining
> - 指令 -XX:MaxFreqInlineSize=N
> - 指令 -XX:MaxInlineSize=N
- 逃逸分析(最复杂的优化)
> - 指令 -XX:+DoEscapeAnalysis
- CPU相关
> - 指令，-XX:UseAVX=N
> - 指令，-XX:useSSE=N
### 逆优化
> 产生逆优化的情况
> - `made not entrant`  (代码被丢弃)
> - `made zombie` (产生僵尸代码)
- 代码被丢弃
> 可能是和`类与接口的工作方式`相关，也可能与`分层编译的实现细节`有关
#### 注意
> - 先前的优化不再有效时（如，所设计的对象类型发生了改变），才会发生逆优化
> - 逆优化的时候，会对性能产生一些小而短暂的影响
> - 分层编译时，如果先由client编译器编译，而后用server编译器编译，会触发逆优化


## 建议
- 只要有必要的时候，就应该使用`final`，在当前的java中是不对的，`fianl`不会影响到应用的性能
- 不用担心小方法，他们很容易内联
- 需要编译的代码在编译队列中
- 虽然代码缓存的大小可以调整，但是它任然是有限的西元
- 代码越简单，优化越多

### 一些关于javac编译器
> 它完全不会影响性能
> - 包含额外调试信息的-g标志不会影响性能
> - 在java应用程序中使用final关键字并不会更快的编译代码
> - 用较新的javac版本编译通常不会使应用程序更快


##### 未来的虚拟机[GraalVM](./VM/GraalVM.md)

## 预编译
- 提前编译（AOT compilation）
> - 启动速度更快
> - 工具，jaotc
> - 获取系统实际会用到的方法列表
> 	- java -XX:+UnlockDiagnosticVMOptions -XX:+LogTouchedMethods -XX:+PrintTouchedMethodsAtExit
> - 指令，-XX:+PrintAOT
- GraalVM原生编译
> - 对短期程序是理想的选择
> - 在程序运行期间不会使用C2编译器编译类
> - 内存占用在开始的时候比传统的JVM少很多，但是随着程序的运行，这种优势会消失
> 可以使用java的特性会受到限制
> - 动态类加载
> - Finalizer
> - Java安全管理器
> - `JMX`和`JVMTI`
> - 使用`反射`常常需要特殊的编码和配置
> - 使用`动态代理`常常需要特殊的配置
> - 使用`JNI`需要特殊的编码或配置
## [Next 垃圾收集器](./GC.md)
