## 调优的核心领域
1. 编辑器和垃圾收集器等的调优参数
2. Java平台（类比API）的最佳实践

## 约定
### JVM调优标志
- 布尔标记：-XX:+FlagName 开启 -XX:-FlagName 关闭
- 附带参数标记：-XX:FlagName=something
> 添加标记-XX:+PrintFlagsFinal 可以打印出当前JVM的标记
> -XX:+PrintCommandLineFlags 答应被修改过的标记

### 性能优化需要注意的点
- 编写更好的算法
- 编写更少的代码
- 拒绝过早的优化
- `"数据库"`可能就是瓶颈

### 性能分析的对象
- CPU使用率
- I/O延迟
- 系统整体的吞吐量

## 性能分析的方法
- 借助性能分析来优化代码，重点关注性能分析中最耗时的操作
- 利用[奥卡姆剃刀原则](/appendix/OccamsRazor.md)诊断性能问题
- 为应用中最常用的操作编写简单的算法

## [Next 测试方法](./TestMethod.md)