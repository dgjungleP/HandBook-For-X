



# 扩展点

### [ApplicationContextInitializer](./ApplicationContextInitializer.md)
> 整个容器初始化之前调用，可以激活一些配置，或者进行动态字节码注入

### [BeanDefinitionRegistryPostProcessor](./BeanDefinitionRegistryPostProcessor.md)
> 读取完项目中的`BeanDefinition`之后触发，可以动态注册自己的`BeanDefinition`,记载cplasspath之外的Bean


### [BeanFactoryPostProcessor](./BeanFactoryPostProcessor.md)
> 读取`BeanDetinition`之后，实例化Bean之前触发，可以修改已注册的`BeanDefinition`元信息


### [InstantiationAwareBeanPostProcessor](./InstantiationAwareBeanPostProcessor.md) #Important
> 在初始化阶段和实例化阶段调用，可以针对于某一类Bean在整个生命周期期间进行收集，或者对某个类型的Bean进行统一的设值


### [SmartInstantiationAwareBeanPostProcessor](./SmartInstantiationAwareBeanPostProcessor.md)
> 


### [BeanFactoryAware](./BeanFactoryAware.md)
> 在Bean对象实例化时候，注入属性之前调用，可以对每个Bean做特殊化定制，或者拿到`BeanFactory`进行缓存，以后使用


### [ApplicationContextAwareProcessor](./ApplicationContextAwareProcessor.md)
>在Bean实例化之后，初始化之前，一般是提供自己内部的扩展到，给别人使用


### [BeanNameAware](./BeanNameAware.md)
>在Bean初始化之前，可以用来修改BeanName


### [@PostConstruct](./@PostConstruct.md)
>在初始化的阶段，可以用来初始化一些属性


### [InitializingBean](./InitializingBean.md)
> 用于系统启动的时候做一个业务指标的初始化工作


### [FactoryBean](./FactoryBean.md)
> 用以定制实例化Bean的逻辑，比如制作代理类


### [SmartInitializingSingleton](./SmartInitializingSingleton.md)
> 所有的单例对象初始化完成之后进行触发，可以对所有的单例对象做一些后置的业务逻辑


### [CommandLineRunner](./CommandLineRunner.md)
> 整个项目启动完毕之后自动执行，也可以用`@Order`进行排序，主要用作启动项目完成之后的一些业务预处理

### [DisposableBean](./DisposableBean.md)
>在Bean对象被销毁时，自动触发

### [ApplicationListener](./ApplicationListener.md)
> 用以监听一些Spring的内置事件、用户自定义事件、

