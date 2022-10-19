## 构建情况
- 多个`Host`组件
- 一个`AccessLog`组件
- 一个`Pipeline`组件
- 一个`Cluster`组件
- 一个`Realm`组件
- 一个`LifecycleListener`组件
- 一个`Log`组件


## 组件说明

### [Host](Host.md)
> 子容器，报表一个虚拟主机
### Accesslog
> 记录访问日志
### pipeline
> 将不同的容器串联起来的通道
### Cluster
> 全局引擎容器，将不同的JVM上的全局引擎容器内所有的应用抽象成集群，让他们能在不同的JVM之间`互相通信`
### Realm
> 存储了用户、密码及权限等的数据对象，为了实现资源认证模块
### LifeCycleListener
> 建通Tomcat的生命周期
### Log
> 不同级别的日志输出

## 其他说明
- 合理使用`Pipeline模式`，搭配`阀门Valve`使用