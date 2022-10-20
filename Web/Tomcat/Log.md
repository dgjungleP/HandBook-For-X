> Tomcat 提供了统一的Log接口，其自身的实现类`DirectJDKLog`

注：
1. 通过工厂模式，如过需要添加其他的Log实现类，就可以无缝衔接
2. 可以通过配置`%JAVA_HOME%\jre\lib\logging.properties`来配置默认版本的Tomcat日志框架
3. 在国际化处理中Tomcat使用`StaingManager`来处理


### 访问日志
核心功能：
1. 将信息记录下载
2. 信息记录到哪里
3. 信息以那种形式记录（定义访问日志格式）
4. 考虑适配器
> Tomcat 考虑到模块化和可配置扩展，它把访问日志组件作为管道中的`阀门`


## Next [类加载器](./ClassLoader.md)