## 构成情况

- 若干个`Listener`组件
- 若干个`GlobalNamingResource`组件
- 若干个[Service](./Service)组件
- 一个`ServerSocket`组件

## 作用
- 提供`监听器`机制，用于整个Tomcat的生命周期中对不同的事件进行处理
- 提供Tomcat容器全局的`命名资源`实现
- 监听某个端口，以接受`SHUTDOWN`命令


## 组件作用

### 生命周期监听器

> 默认的情况下，会`自动配置6个`监听器组件，如果要自己实现一个生命周期监听器，只需要实现`LifecycleListener`接口

#### AprLifecyleListener 
有时候Tomcat会使用[APR](/appendix/APR.md)本地库进行优化的，并通过[JNI](/appendix/JNI.md)方式调用本地库能大幅提高对静态文件的处理能力，该监听器就针对初始化前的事件和销毁后事件进行监听，进行APR库的创建和销毁

#### JasperListener
用于初始化Tomcat的JSP编译器核心引擎`Jasper`

#### JreMemoryLeakPreventionListener
提供`JRE内存泄露`和`锁文件`的一种措施，在初始化时使用`系统类加载器`加载一些类和设置缓存属性，以避免内存泄露和锁文件

##### JRE内存泄露问题
产生原因：
	垃圾回收期要回收时无法回收本该回收的对象
产生的方式：
	- 上下文类加载器导致的内存泄露
	- 线程启动另一个线程并且新线程无止尽的执行

Tomcat的解决办法：
	1. 先将当前线程的上下文类加载器设置为系统类加载器
	2. 使用系统类加载器加载这些特殊的类
	3. 在将线程的上下文类加载器复原

##### 锁文件
产生原因：
	主要是因为URLConnection默认的缓存机制导致的

Tomcat的解决方法：
	将URLConnection默认设置为不缓存

#### GlobalResourceLifecycleListener
初始化Server组件里面`JNDI资源的MBean`，并提交给`JMX`管理

#### ThreadlocalLeakPreventionListener
解决`Threadlocal`带来的内存泄露问题

##### Threadlocal内存泄露问题
产生原因：
	垃圾回收器回收时，由于使用Threadlocal的对象被一个运行时间很长的线程引用，导致该对象无法被回收
场景：
	- Web应用的重加载

Tomcat的解决办法：
	在Web应用重新加载时，将所有的工作线程销毁并再创建

#### NamingContextListener
负责Server组件内全局命名资源在不同生命周期的不同操作


### 全局命名资源
> 通过`ResourceLink`将命名对象给所有Web应用使用，在Server启动的时候创建NamingResource和NamingContextListener，在利用ContextResource创建树状的命名上下文（通过Digester框架解析）

### 命名资源
- ContextResource
- ContextEjb
- ContextEnviroment
- ContextLocalEjb
- MessageDestinationRef
- ContextResourceEnvRef
- ContextResourceLink
- ContextService


### 监听SHUTDOWN命令
> Server自身会开放一个监听关闭端口默认为8005。






