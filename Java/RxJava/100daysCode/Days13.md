> 最近在做项目的时候，由于需要为实体类生成对应的雪花ID，但是又不可能每次在创建实例的时候再去
> 调用方法，所以，想到了能不能借助于Myabtis的插件来实现，因为在之前的工作过程中也用到过它的
> 分页插件`PageHelper`,所以接下来将要学习以下Myabtis的插件开发


## 概述

Myabtis的插件允许我们在映射语句执行的过程中的某一个点进行拦截，并且提供了以下几个拦截点（具体的更多的信息，后面再去深读一下源码）
- Executor
- ParameterHandler
- ResultSetHandler
- StatmentHandler
所以它主要也是提供了一个拦截器`Intercaptor`接口来供我们使用，以方便接入我们实现的插件，同时需要配合`@intercepts`注解和`@Signature`注解使用

首先们根据官方的说明简易的创建一个插件：
``` java
@Intercepts({@Signature(  
        type = Executor.class,  
        method = "query",  
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}  
)})  
@Slf4j  
public class UsedTimeAnalysePlugin implements Interceptor {  
    @Override  
    public Object intercept(Invocation invocation) throws Throwable {  
        log.info("Pre executor");  
        Object proceed = invocation.proceed();  
        log.info("After executor");  
        return proceed;  
    }  
}
```
下面我们再来说明一下每个地方到底是怎么回事

### @Intercepts
这里是为了指定哪些方法需要被当前我们声明的拦截器拦截到，具体的拦截信息是在内部的`@Signature`来定义的，这里相当于是一个标识注解

### @Signature
这里是声明具体哪个方法需要被拦截到，当前例子的说明如下：
#### type
这里是指代的我们上面所说的拦截点，这里我们选择的是`Executor`
#### method
这里是指代的我们拦截点上的某个方法的名称，这里我们选择的是拦截`Executor`的`query`方法
#### args
这里可能会比较疑惑，为什么我们这里会填写这四个类呢？我们需要去找到`Executor`的`query`方法的声明

``` java
<E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException;
```
可以看到我们这里的所填写的四个类信息，就是对应的方法声明的四个入参的类信息

所以综上所述，`@Signature`其实就是去指明我们要拦截的方法到底是哪一个

### UsedTimeAnalysePlugin
这里就是我们通过实现Mybatis提供的接口来实现的`自定义的Plugin`，最简易的方法就是去实现他的`intercept`方法，像在这里我们就是简单的在语句允许的前后，打印了一些信息通过执行我们可以得到以下结果：
``` console
2023-04-11 22:45:00.028  INFO 17896 --- [           main] c.j.c.plugins.UsedTimeAnalysePlugin      : Pre executor
JDBC Connection [HikariProxyConnection@4491252 wrapping conn0: url=jdbc:h2:mem:testdb user=SA] will not be managed by Spring
==>  Preparing: select * from user
==> Parameters: 
<==    Columns: ID, USER_NAME
<==        Row: 1, user1
<==        Row: 2, user2
<==        Row: 3, user3
<==        Row: 4, user4
<==        Row: 5, user5
<==        Row: 6, user6
<==      Total: 6
2023-04-11 22:45:42.161  INFO 17896 --- [           main] c.j.c.plugins.UsedTimeAnalysePlugin      : After executor
```
所以一个简易的Mybatis Plugin就这样被我们实现了。
在这里我们其实需要注意的是，他的实现很简单，但是我们需要分析到我们到底想要做些什么，才能找到对应的方法去给他装配插件，最后才能达到我们想要的效果，所以如果要去实现Mybatis的插件，我们就得了解到，我们到底需要去拦截哪个拦截点的哪个方法。