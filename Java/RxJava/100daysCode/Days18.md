	紧接着上次说到的这个线程上下文类加载器，我们看一下他在`ServceLoader`中到底做了些什么事情

## 使用`Thread.currentThread().getContextClassLoader()`

在源码中追踪的时候，我们发现最后这个类加载器被注入到了`LazyIterator`
``` java
public void reload() {  
    providers.clear();  
    lookupIterator = new LazyIterator(service, loader);  
}  
  
private ServiceLoader(Class<S> svc, ClassLoader cl) {  
    service = Objects.requireNonNull(svc, "Service interface cannot be null");  
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;  
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;  
    reload();  
}
```
而在`LayzIterator`中我们发现它主要用到了两个地方
-  资源数据的加载
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
            else                configs = loader.getResources(fullName);  
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

- 类加载
``` java
private S nextService() {  
    if (!hasNextService())  
        throw new NoSuchElementException();  
    String cn = nextName;  
    nextName = null;  
    Class<?> c = null;  
    try {  
        c = Class.forName(cn, false, loader);  
    } catch (ClassNotFoundException x) {  
        fail(service,  
             "Provider " + cn + " not found");  
    }  
    if (!service.isAssignableFrom(c)) {  
        fail(service,  
             "Provider " + cn  + " not a subtype");  
    }  
    try {  
        S p = service.cast(c.newInstance());  
        providers.put(cn, p);  
        return p;  
    } catch (Throwable x) {  
        fail(service,  
             "Provider " + cn + " could not be instantiated",  
             x);  
    }  
    throw new Error();          // This cannot happen  
}
```

那么为了类加载需要用这个特殊的类加载器呢，这个我们就不得不来聊一下，Java的类加载机制`双亲委派加载`

# 双亲委派加载

学习Java的时候，我们经常会听到这个词`双亲委派`，那么为什么叫`双亲委派`呢，它又给我们带来了什么？帮助我们解决了什么问题呢？

### 类加载器

我们耳熟能详的类加载器有以下几种：
- `BootstrapClassLoader`：主要负责以加载核心类库，构造`ExtClassLoader`和`AppClassLoader`
- `ExtClassLoader`：主要负责加载`jre/lib/ext`目录下的一些扩展jar包
- `AppClassLoader`：主要负责加载应用程序的主函数类

我们根据这些类加载器的源码可以看到它的加载逻辑：
``` java
Class<?> c = findLoadedClass(name);  
if (c == null) {  
  
    try {  
        if (parent != null) {  
            c = parent.loadClass(name, false);  
        } else {  
            c = findBootstrapClassOrNull(name);  
        }  
    } catch (ClassNotFoundException e) {  
	 ....
	}
	if (c == null) {
        c = findClass(name);
    }
    }
```
这里是精简版的逻辑，我们可以看到，他的实际逻辑就是
1. 先确认当前类是不是以及加载过了，加载过了就不再处理
2. 如果没有加载先调用父加载（这里的父子关系，并不是继承中的父子关系，而是组合中的父子关系）器进行加载
3. 父加载器还是找不到的情况下，才调用自己的加载方法进行类加载。

### 双亲委派给我们带来了什么好处
- 避免了自己吗的重复加载
- 程序更安全，核心的API不会被替换

### 为什么要打破双亲委派

从上述中给我们可以得知，所有的类加载最终都会从启动类加载器加载，但是如果我们使用`SPI`机制去引入一些第三方类库的话，启动类加载器肯定会加载失败，但是
> 例如`java.sql.Driver`这种由启动类加载器加载的，所以它的实现类也得是启动类加载器加载

所以我们不得不找一个类加载器让我们的接口和实现类由同一个类加载器加载，那么`线程上下文加载器`就应运而生，从此打破双亲委派模型，让`SPI`的接口类和实现类都由同一个类加载器加载，其实就是让

所以每一次破坏双亲委派都是因为对于类加载的一些特殊要求，历史上有也有这么几次破坏双亲委派的场景：
1. 由于历史原因，我们可以继承CLassLoader类然后重写loaderClass方法，从而打破双亲委派，例如：Tomcat就这样做了
2. 为了更好的使用`SPI`，从而引入了线程上下文类加载器，来特意大打破双亲委派
3. 用户对程序的动态性的极致追求，希望做到代码的热替代，模块的热部署，例如：我也不太熟悉的`OSGi`(粗略的了解了一下，感觉很有意思，后面会持续去跟进看一下)，就是通过自己的类加载器来实现的，不再是一个树状的双亲委派模型，而是一个网状结构
4. JDK9引入的模块化，这个从源码里面就可以看出，它会在提交给父加载器之前，先判断系统模块的归属问题(后面学习的时候也需要看看)

## 总结

总体来说，由于各种各样的原因，我们不得不打破双亲委派模式，但是打破的同时，也要注意代码的安全性，不要盲目的自定义类加载器等等。合理得打破可以提高我们得代码活性，更加灵活。