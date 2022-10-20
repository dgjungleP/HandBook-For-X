## 构成情况

- 多个`Wrapper`组件
- 一个`AccessLog`组件
- 一个`Pipeline`组件
- 一个`Cluster`组件
- 一个`Realm`组件
- 一个`ErrorPage`组件
- 一个`Manager`组件
- 一个`DirContext`组件
- 一个安全认证组件
- 一个`JarScanner`组件
- 多个过滤器组件
- 一个`NamingResource`组件
- 一个`Mapper`组件
- 一个`WebappLoader`组件
- 一个`ApplicationContext`组件
- 一个`InstanceManager`组件
- 一个`ServletContainerInitializer`组件
- 一个`Listeners`组件


## 构件说明

### Wapper
> Tomcat中的最小的容器，对应一个Servlet

### Realm
> Context级别的用户、密码及权限的数据对象

### AccessLog
> Context级别的访问日志

### ErrorPage
>  利用StandardHostValve实现的，错误页面的处理器

### Manager
>  管理对应的Web容器中的会话

### DirContext
>  利用JNDI技术，来对目录对象的相关属性进行操作

### 安全认证
>  实现对资源的访问约束

### JarScanner
>  扫描Context对应的Web应用中的Jar包，使用的`回调机制`实现

### 过滤器
> 为Web应用的所有请求和响应做统一逻辑处理，其自身由Wrapper容器中的管道负责调用

### NamingResource
>  将配置中声明的不同的资源以及属性映射到内存中，与NamingContextListener协作完成这个功能

### Mapper
> Context级别的路由导航，与RequestDispatcher协作，分发请求到对应的Servlet

## Pipeline
> Context级别的管道

### WebappLoader
> 作为应用的类加载器，实现应用的相互隔离和重加载

注：
1. 这里的WebappLoader并没有遵循`双亲委派机制`，而是按照自己的策略顺序加载类
2. 可以看一下WebappLoader与其他加载器的关系
3. 重加载，是直接用新的WebappLoader来替换旧的就能实现

### ApplicationContext
> 对各种资源和功能进行管理，并使用门面模式提供了一个类似代理的访问模式

### InstanceManager
> 对Context中的实例进行管理，通过`反射`来实例化对象，并且包含两个类加载器

### ServletContainerInitializer
> 给第三方组件执行一些初始化工作，配合`HandlesTypes`注解使用

注：
1. 注意其使用方式，必须在`META_INF/services`目录理创建一个`javax.servlet.ServletContainerinitializer`文件

### Listeners
> 提供可插拔可扩展的能力，这里使用的`监听者模式`

注：
1. 默认会注入四个监听器
	1. ContextConfig
	2. TldConfig
	3. NamingContextListener
	4. MemoryLeakTrackingListener