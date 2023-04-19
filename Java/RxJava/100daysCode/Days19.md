> 之前我们一起看来一下Java原生是如何支持`SPI`的，那么更进一步，我们来看一下我们常用的
> Spring是如何实现他自己的`SPI`的，它又有什么不一样的地方呢

在使用SpringBoot的时候我们都会有疑惑，SpringBoot的`自动装配`到底是如何实现的，为什么我用一个注解或者甚至不用注解直接引入包就能实现我们编写的第三方组件能够自动注入，其实其原理的背后就是`SPI`,那么SpringBoot又是怎么样来支持`SPI`的呢？

## SpringFactoriesLoader

### 发现它
如有有兴趣去追查一下`SpringAplication`的`run`方法的话，我们可以看到接下来的一段代码：
``` java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {  
   return run(new Class<?>[] { primarySource }, args);  
}  
  
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {  
   return new SpringApplication(primarySources).run(args);  
}
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {  
   this.resourceLoader = resourceLoader;  
   Assert.notNull(primarySources, "PrimarySources must not be null");  
   this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));  
   this.webApplicationType = WebApplicationType.deduceFromClasspath();  
   setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));  
   setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));  
   this.mainApplicationClass = deduceMainApplicationClass();  
}
```
`Spring`的启动最后，还是会落到这个`SpringApplication`类的初始化里面来，同时我们看到，`SpringApplication`在类初始化的过程中，执行了一些类的注册，那么我们就继续追查一下这个`getSpringFactoriesInstances`方法：
``` java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {  
   ClassLoader classLoader = getClassLoader();  
   // Use names and ensure unique to protect against duplicates  
   Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));  
   List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);  
   AnnotationAwareOrderComparator.sort(instances);  
   return instances;  
}
```
那么到这里我们就看到，我们需要的主角出现了，我们看到除开下面是构造我们需要的实例之外，最重要的就是去找到，这些实例具体是那些实现类，那么`SpringFactoriesLoader`就在这里充当了这样一个服务发现的功能。

### 挖掘它

那么现在我们就看看，这个`loadFactoryNames`做了些什么事情：
``` java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {  
   String factoryTypeName = factoryType.getName();  
   return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());  
}  
  
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {  
   MultiValueMap<String, String> result = cache.get(classLoader);  
   if (result != null) {  
      return result;  
   }  
  
   try {  
      Enumeration<URL> urls = (classLoader != null ?  
            classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :  
            ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));  
      result = new LinkedMultiValueMap<>();  
      while (urls.hasMoreElements()) {  
         URL url = urls.nextElement();  
         UrlResource resource = new UrlResource(url);  
         Properties properties = PropertiesLoaderUtils.loadProperties(resource);  
         for (Map.Entry<?, ?> entry : properties.entrySet()) {  
            String factoryTypeName = ((String) entry.getKey()).trim();  
            for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {  
               result.add(factoryTypeName, factoryImplementationName.trim());  
            }  
         }  
      }  
      cache.put(classLoader, result);  
      return result;  
   }  
   catch (IOException ex) {  
      throw new IllegalArgumentException("Unable to load factories from location [" +  
            FACTORIES_RESOURCE_LOCATION + "]", ex);  
   }  
}
```
其实从上面我们就能看得到，它其实和`ServerLoader`做了同样的事情，都是去找到一个资源文件，然后将资源文件的数据解析出来，只不过这里的资源文件路径是:
``` java
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
```
> 但是在SpringBoot 2.7之后官方已经不建议使用这个路径了，改为了`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`,并且在废弃通知中也做出了回应
> `Loading auto-configurations from `spring.factories` is deprecated`

所以很多自定义starter包的朋友都对这个文件夹不陌生，这就是为什么我们只要在这个路径中指明我们的实现类之后，就可以通过第三方插件的方式来改变我们SpringBoot的一些行为了，当然具体的自动装配还有更多的其他的一些组件来组合才完成了最终效果，这个我们后期再来细细的品味。

### 它改变了什么
- 我们可以看到它提供了一个类加载器和装配类的缓存，可以避免多次重复的读取配置配置，但是类的装载还是直接调用`ClassLoader`进行的装载
