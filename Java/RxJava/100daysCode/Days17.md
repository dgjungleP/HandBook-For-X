	在开始了解RPC的时候，我们常常会学会去用到`SpringCloud`和`Dubbo`,那么就必不可少的会去接触到
	`SPI`，一个熟悉而又陌生的概念，那么今天我们就先来看看这个神秘的`SPI`到底是什么，它能给我们带来什
	么有意思的东西

# 基本概念

`SPI`  英文全称`Service provider interface`，从名字我们就可以看的出来，它应该是作用于`接口`（不一定是Java中的那个接口），同时也是提供给对应的供应商的，从字面意思我们可以解读出来，它应该是用作为一些第三方需要接入的实现或者组件预留的一套工具，是一套服务发现机制。


# 如何使用

其实使用`SPI`相对来说比理解它更为简单，我们基本上只需要遵循它给到的一些模板，我们就能很好的去利用这样的一个工具。  

## 固定的SPI加载工具类

JDK给我们预留了一个用于寻找`SPI`实现的工具类`java.util.ServiceLoader`,我们只需要简单的使用它就能够得到我们想要的结果，我们可以看看它的一个方法签名：
``` java
public static <S> ServiceLoader<S> load(Class<S> service,  
                                        ClassLoader loader)
```

## 固定的文件路径

当我们再进一步去读`ServiceLoader`的源码的时候，我们就会再开头就看到，它有一个静态变量:
``` java
private static final String PREFIX = "META-INF/services/";
```

我们我们去追寻它的使用的地方的时候会看到这样一段代码：
``` java
private boolean hasNextService() {  
    if (nextName != null) {  
        return true;  
    }  
    if (configs == null) {  
        try {  
            String fullName = PREFIX + service.getName();  
            if (loader == null)  
                configs = ClassLoader.getSystemResources(fullName);  
            else                
	            configs = loader.getResources(fullName);  
        } catch (IOException x) {  
            fail(service, "Error locating configuration files", x);  
        }  
    }  
    while ((pending == null) || !pending.hasNext()) {  
        if (!configs.hasMoreElements()) {  
            return false;  
        }  
        pending = parse(service, configs.nextElement());  
    }  
    nextName = pending.next();  
    return true;}
```
从这里我们可以进一步确认到，当去寻找`SPI`具体的实现的时候，我们就会用到这个文件夹，并且再读取文件的时候，使用的是需要寻找的类的全限定名。

# 实践

在看到如何使用后，我们开始大胆的尝试，来利用`SPI`来发现我们定义的服务

## 准备

首先我们创建一个`People`接口，来指定我们的需要实现的`SPI`，并且实现两个他的实现类：

``` java
public interface People {  
    void name();  
}

public class Boy implements People {  
    @Override  
    public void name() {  
        System.out.println("I`m boy");  
    }  
}

public class Girl implements People {  
    @Override  
    public void name() {  
        System.out.println("I`m girl");  
    }  
}
```

然后在`resource`目录下创建对应的目录文件`META-INF/services/com.jungle.challenge.spi.interfaces.People`
然后创建一个工厂方法，用来执行`SPI`的具体实现的操作：
``` java
public class PeopleFactory {  
    public void invoker() {  
        ServiceLoader<People> loader = ServiceLoader.load(People.class);  
  
        boolean notFound = true;  
        for (People people : loader) {  
            notFound = false;  
            people.name();  
        }  
        if (notFound) {  
            throw new RuntimeException("Not found any People instant");  
        }  
    }  
}
```

## 执行验证

接下来我们直接调用`PeopleFactory`的`invoker`方法，可以看到我们报错了：
``` console
java.lang.RuntimeException: Not found any People instant

	at com.jungle.challenge.spi.core.PeopleFactory.invoker(PeopleFactory.java:18)
```
然后我们在`com.jungle.challenge.spi.interfaces.People`文件中添加我们两个实现类的路径地址
``` txt
com.jungle.challenge.spi.v2.Girl  
com.jungle.challenge.spi.v1.Boy
```
再次执行
``` console
I`m girl
I`m boy
```
我们确实拿到了我们需要的两个实体类，并执行了他们的`name`方法


## 结论

所以以上就是我们简单的将`SPI`这个强大的机制使用起来了，当然真实的用法肯定不是在同一个包里面进行使用，这样太过于复杂，何不直接利用简单的多态就行了呢，所以一般都是在一些公共的组件里面预留一些接口给第三方实现的时候使用。一般我们接触到的最多的就是JDBC和日志模块，当然在一些框架里面也能经常看到他的身影：
1. Spring MVC
2. Spring Boot
3. Dubbo
在这些框架里面还有一些不同的地方，我们在后面再去摸索以下。

在查阅`ServiceLoader`的源码的时候，我们发现它利用到了类加载器，并且在默认的方法是先用还是用到的线程上下文的类加载器:
``` java
public static <S> ServiceLoader<S> load(Class<S> service) {  
    ClassLoader cl = Thread.currentThread().getContextClassLoader();  
    return ServiceLoader.load(service, cl);  
}
```
那么接下来我们就来说说这个类加载器到底在干些什么事情，为什么我们需要在这里指定。