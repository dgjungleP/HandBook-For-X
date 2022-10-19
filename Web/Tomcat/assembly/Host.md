## 构成情况
- 多个`Context`容器
- 一个`AccessLog`组件
- 一个`Pipeline`组件
- 一个`Cluster`组件
- 一个`Realm`组件
- 一个`HostConfig`组件
- 一个`Log`组件


## 组件说明

### Context
> 对Web应用的抽象
- AccessLog
> Host级别的访问日志
- Pipeline
> Host级别的串联通道
- Cluster
> Host级别的集群会话
- Realm
> Host级别的用户、密码及权限等的数据对象
- HostConfig
> Host的生命周期监听器，将Web项目加载到Host容器中


## 其他说明
- Web应用有三种部署类型
	- Descriptor描述符
		- 一般地址为 `%CATALINA_HOME%/conf/[EngineName]/[HostName]/MyTomcat.xml`
	- WAR包
		- 一般地址为 `%CATALINA_HOME%/webapps`下所有的WAR包项目
	- 目录
		- - 一般地址为 `%CATALINA_HOME%/webapps`下所有的项目
- HostConfig会使用`Future`和线程池来优化部署的耗时