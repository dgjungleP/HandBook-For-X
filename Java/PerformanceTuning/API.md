## 字符串
指令
- -XX:+CompactStrings
- -XX:StringTableSize=N
- -XX:+PrintStringTableStatistics
#### 重复字符串和字符串保留
去重的方法
- 使用G1 GC执行自动去重
> 指令 ，-XX:+UseStringDeduplication  -XX:+PrintStringDeduplicationStatistics -XX:StringDeduplicationAgeThreshold=N
- 使用String类的`intern()`方法来创建字符串的标准版本
- 使用自定义方法来创建字符串`标准版本`
> 将重要的字符串直接保存在哈希映射中
> - CustomConcurrentHashMap
> - ConcurrentHashMap

#### 字符串连接
指令
- -XX:OptimizeStringConcat
> 对于多次的连接操作，一定要使用StringBuilder
> 字符串的`单行连接`，会产生良好的性能


## 缓冲I/O
>- 使用`二进制文件I/O`，总是需要使用`BufferedInputStream或BufferedOutputStream`
>- 对于使用`字符数据的文件I/O`，总是需要使用`BufferedReader或BufferedWriter`
>- 对于文件和套接字，还有像压缩和字符串编码这样的内部擦操作，必须适当地对I/O进行缓冲


## 类加载
- `CDS` - java11开始地新特性

## 随机数
生成器
- Radom
- ThreadLocalTandom
- SecureRandom


## Java原生接口
> 避免使用Java原生接口（`JNI`）


## 异常
指令
- -XX:-StackTraceInThrowable
> 异常处理的成本在现在不一定很高，但是一般不应该用作通用的机制
> 	- 因为创建代码块
> 	- 获取栈轨迹

## 日志
- JVM日志
- 服务器访问日志
	- 使用IP不用主机名
	- 使用时间戳不用时间字符串
>基本原则
>- 要在日志的数据和日志的级别之间找平衡
>- 使用细粒度的日志记录器
>- 即使没有开启日志，也很统一无意间写出具有副作用的日志代码


## 集合
- 同步和非同步
- 设置集合的大小
- 集合和内存的效率

## Lambda和匿名类
- Lambda并不是匿名类的语法糖

## 流和过滤器的性能
重要特性
- 延迟遍历

注意
- 即使对于整个数据集，的那个过滤器的性能也会略优于迭代器
- 多个过滤器是由开销的，要确保编写良好的过滤器

## 对象序列化

- 瞬时字段
- 覆盖默认的序列化
- 压缩序列化数据
- 跟踪重复对象