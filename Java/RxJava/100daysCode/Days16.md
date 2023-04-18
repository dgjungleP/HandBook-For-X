	上会我们已经将整体的插件模板创建完成了，那么今天我们继续来完成最后的策略实现，以及一些简单的优化

### 策略类实现

首先我们知道，我们定义了一个策略接口`DesensitizeStrategy`，同时再我们的模板代码，以及注解定义中，我们都看到了策略接口的身影：
``` java
DesensitizeStrategy desensitizeStrategy = (DesensitizeStrategy) strategy.newInstance();  
setMethod.invoke(data, desensitizeStrategy.doDesensitize(value));
```
``` java
@Documented  
@Inherited  
@Retention(RetentionPolicy.RUNTIME)  
@Target({ElementType.FIELD})  
public @interface Desensitized {  
  
    Class<? extends DesensitizeStrategy> strategy() default DefaultDesensitizeStrategy.class;  
}
```
根据注解的定义，只要实现了`DesensitizeStrategy`的注解接口，那么就能在我们的插件中实例化出来，所以我们现在就来实现一个手机号脱敏策略：
``` java
public class PhoneDesensitizeStrategy implements DesensitizeStrategy<String, String> {  
      
    @Override  
    public String doDesensitize(String data) {  
         
  
    }  
}
```
我们假定所有的手机号都是通过字符串存储的，那么接下来我们使用常用的手机号脱敏方式就是我们常看到了的`136****9958`这样的一个格式，那么我们这里用一下正则表达式来实现：
``` java
@Override  
public String doDesensitize(String data) {  
    return data.replaceAll("(\\d{3})\\d{4}(\\d+)", "$1****$2");  
}
```
现在我们给我们的`User`的`userPhone`字段打上对应的策略类：
``` java
@Data  
public class User {  
    private Long id;  
    private String userName;  
    @Desensitized(strategy = PhoneDesensitizeStrategy.class)  
    private String userPhone;  
}
```
允许之后我们得到以下以下结果：
``` console
Creating a new SqlSession
SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@e1ce44] was not registered for synchronization because synchronization is not active
JDBC Connection [HikariProxyConnection@31007053 wrapping conn0: url=jdbc:h2:mem:testdb user=SA] will not be managed by Spring
==>  Preparing: select * from user
==> Parameters: 
<==    Columns: ID, USER_NAME, USER_PHONE
<==        Row: 1, user1, 110112113114
<==        Row: 2, user2, 110112113114
<==        Row: 3, user3, 110112113114
<==        Row: 4, user4, 110112113114
<==        Row: 5, user5, 110112113114
<==        Row: 6, user6, 110112113114
<==      Total: 6
Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@e1ce44]
User(id=1, userName=user1, userPhone=110****13114)
User(id=2, userName=user2, userPhone=110****13114)
User(id=3, userName=user3, userPhone=110****13114)
User(id=4, userName=user4, userPhone=110****13114)
User(id=5, userName=user5, userPhone=110****13114)
User(id=6, userName=user6, userPhone=110****13114)
```
可以看到，虽然我们从数据库里面查到的是明文，但是我们最终得到的数据是通过了我们脱敏的数据

### 优化
目前我们的插件很轻易的就能发现由很多可以优化的点

#### 1. 策略类不需要每次都创建，我们可以使用全局单例类来处理
``` java
DesensitizeStrategy desensitizeStrategy = STRATEGY_MAP.computeIfAbsent(annotation.strategy(), (key) -> {  
    try {  
        return (DesensitizeStrategy) key.newInstance();  
    } catch (InstantiationException | IllegalAccessException e) {  
        return new DefaultDesensitizeStrategy();  
    }  
});
```
我们可以使用一个`STRATEGY_MAP`来装载策略类，避免每次都需要重新创建

#### 2.对于类的反射操作也是一样，不需要每次都使用反射的方式去获取
处理方式同上方一样，这里就不再赘述了。


## 总结
通过以上方式，我们就创建了一个可让用户自己实现脱敏策略的Mybatis插件,当然还有很多不足和值得优化的地方，后续有时间，我们再继续进行调整