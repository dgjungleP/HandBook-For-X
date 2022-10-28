> 延迟和容错的组件

# 功能

## 失败回退
> 当目标`Provider`实例发生故障时，RPC的会得到一个后备的结果

### 如何使用
1. 定义和使用一个`Fallback`回退处理类
	1. 编写Fallback处理类，将RPC失败后的回退处理具体实现
	2. 在`@FeginClient`中配置失败处理类
2. 定义和使用一个`FallbackFactory`回退处理工厂
	1. 编写FallbackFactory处理类，实现`FallbackFactory`接口，实现抽象方法`Create`创建一个`Fegin`客户端返回
	2. 在`@FeginClient`中配置失败处理工厂
区别：
1. 回退类中已经将所有的异常彻底屏蔽，不方便应用程序干预和排查问题
2. 回退工厂，可以对RPC的异常进行拦截和处理，包括日志输出

## 熔断
>  统计最近RPC调用发生错误的次数，根据统计信息中的`失败比例`等信息，判断是否允许后面的RPC调用`继续或者快速失败`

### 如何使用
1. 在配置文件中进行配置，可以配置全局的，也可以单独配置

### 熔断器模式

### 滑动窗口和时间桶
> 时间桶就是滑动窗口的最小单元


## 重试

## 舱壁隔离

### 隔离方式

#### 线程池隔离

#### 信号量隔离
> 适用于一些非网络API调用或者耗时较小的API调用


### 舱壁模式
> 将应用整体切分为多个不同的区域，防止一块区域不可能用之后，影响到其他可用区域



# RPC保护原理

> 使用`命令模式`和[RxJava](../../RxJava/RxJava.md)响应式编程和滑动窗口技术实现的对外保护

## HystrixCommand
> 不再Spring Cloud的环境中也可以单独使用

使用步骤：
1. 继承`HystrixCommand`,实现器命令当打
2. 使用`HystrixCommand`提供的启动方法启动命令
	1. execute
	2. queue
	3. observe
	4. toObservable
3. 使用`Setter`进行配置或者使用`ComfigurationManager`进行参数配置



