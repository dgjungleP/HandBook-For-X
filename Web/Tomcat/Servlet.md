## 概述
> Java体系的Web服务器基本上都会遵守`Sevlet规范`，该规范描述了HTTP请求及响应处理过程相关的对象及其作用
   

### Servlet接口
> - 所有的Servletl类`必须`实现该接口
> - 一般Servlet容器中，每个Servlet只能对应一个Servlet对象，请求都是由同一个Servlet对象处理
> - 实现了`SingleThreadModel`接口的Servlet可能会在Web容器中存在多个对象，该Servlet对应一个线程，不存在线程安全问题
> - 所有的Servlet及他们使用的类需要由一个`单独的类加载器加载`


方便开发这使用的Servlet类
- GenericServlet，通用的、协议无关的
- HttpServlet，基于HTTP

#### 生命周期
- 加载实例化，主要由`Web容器`完成
- 初始化
- 处理客户端请求
- 销毁


### ServletRequest接口
> - 封装了客户端请求的所有信息
> - Web容器通常会出于`性能原因`而不销毁ServletRequest对象,而是选择`复用`
> - 一般只在Servlet的service方法和`过滤器`的doFilter方法作用域中有效
> - 开启异步调用startAsync后，该对象会一直有效，直到`AsyncContext`调用`complete`方法

信息包括
- 一些HTTP请求头部获取
- 一些获取请求路径
- 获取Cookie
- 获取客户端语言环境
- 获取客户端编码方法


### ServletContext接口
> - 运行所有Servlet的Web应用的`视图`
> - 每个ServletContext对象都需要一个`临时存储目录`,由Servlet分配

包含
- Web应用的Servlet全局存储空间，所有Servlet共有的各种资源和功能
- 获取Web应用的部署描述配置文件
- 添加Servlet到ServletContext
- 添加Filter到ServletContext
- 添加Listener到ServletContext
- 全局的属性保存和获取
- 访问Web应用的静态内容(以路径作为参数是`/`相当于`WEB-INF/lib`目录下jar包中的`META-INF/resources`)


### ServletResponse接口
> - 封装了服务器要返回给客户端的所有信息
> - 为了`提高效率`，一般ServletResponse接口对响应提供了`输出缓存`  
> - Web容器通常会出于`性能原因`而不销毁ServletResponse对象,而是选择`复用`
> - 一般只在Servlet的service方法和`过滤器`的doFilter方法作用域中有效
> - 开启异步调用startAsync后，该对象会一直有效，直到`AsyncContext`调用`complete`方法


### Filter接口
> - 对Web容器的请求和响应做`统一处理`
> - 可以使用`@WebFilter`来定义
> - 也可以使用部署描述文件定义
> - 一般是链式调用
> - Web容器部署完成后，必须实例化过滤器并调用其init方法


### 会话
> - 并没有提出`协议无关`的会话规定,每个通信协议自己规定
> - HTTP的会话接口HttpSession
> - 会话的属性不能在`不同的ServletContext之间共享`
> - 如果想监听某个对象在会话中的情况，可以让该对象实现`HttpSessionBindingListener`
> - 注意设置超时事件
> - 在`分布式`环境中，会话的所有请求必须仅被同一个JVM处理
> - 分布式容器迁移会话是会通知实现了`HttpSessionActivationListener`的会话属性

会话跟踪机制
- Cookie
	- 标准名称必须为JSESSIONID
- URL重写
	- 添加jseesionid参数

### 注解
> - 使用了注解的类只有被方法`WEB-INF/classes`或`WEB-INF/lib`下的jar中才能会Web容器处理
> - 需要开启`web.xml`中的`<web-app>中的metadata-complete`

- @WebServlet，被注解的类必须继承`javax.servlet.http.HttpServlet`
- @WebFiler，被注解的类必须继承`javax.servlet.Filter`
- @WebInitParam，用于给WebServlet和WebFilter传递参数
- @WebListener，被注解的类必须继承`javax.servlet`或`javax.servlet.http`包下的XXListener
- @MultipartConfig,指定期望的mime/multipart类型


### 可插拔性
> - 可以定义`web-fragment.xml`，组成Web的逻辑分区，同样可以生成主键

加载顺序定义
- 在web.xml中定义绝对加载顺序
- 在web-fragment.xml中定义相对顺序

### 请求分发器（RequestDispatcher接口）
> - 将请求/响应转发给另一个Servlet处理

### Web应用
> - Web应用和ServletContext对象是`一对一的关系`
> - 一般`WEB-INF`目录下的文件都不能由容器直接提供给客户端，只能通过ServletContext结合RequesuDispatcher公开这些内容
> - 部署每个Web容器的ClassLoader必须是一个`单独的实例`
> - 服务器应该能在不重启Web容器的情况下更新一个Web应用程序
> - 更新Web应用程序时，Web容器应该提供可靠的方法保存这些Web应用的会话

Web应用程序部署在容器中时的请求步骤
- 实例化所有`事件监听器`
- 调用已经实例化且实现了`ServletContextListener`的监听器实例的`contextInitialized`方法
- 实例化所有`过滤器`，并调用`init`方法
- 根据顺序，实例化所有Servlet实例，并调用`init`方法

### Servlet映射
> 根据最长的上下文路径匹配请求URL，然后匹配Servlet

匹配规则
- 匹配精确的路径
- 递归尝试最长的路径前缀
- 如果结尾包含扩展名，尝试匹配一个专属的Servlet
- 都不满足，匹配一个默认的Servlet


### 部署描述文件

配置和部署信息
- ServletContext的初始化参数
- Session配置
- Servlet声明
- Servlet映射
- 应用程序生命周期监听器类
- 过滤器定义和过滤器映射
- MIME类映射
- 欢迎文件列表
- 错误页面
- 语言环境的编码映射
- 安全配置


## [NEXT Tomcat](Tomcat.md)