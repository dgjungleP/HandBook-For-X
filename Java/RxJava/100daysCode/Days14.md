	为了巩固之前对Mybatis插件的学习，同时也想到了一个之前经常遇到的问题，数据库的数据需要脱敏（如用户
	手机号，密码等），那么今天我们就来一个创建一个Mybatis的数据脱敏插件

## 数据脱敏插件

### 分析
首先通过对需求的分析，我们可以轻易得知以下几个要点
1. 我们需要对查询的返回值进行数据脱敏
2. 脱敏的数据需要被标记，不然插件无法得知哪些数据是需要脱敏的
3. 每个数据的脱敏策略可能不一样，所以在标记的时候或许需要预设好脱敏的策略

### 方案

#### 对查询返回值进行脱敏
> 那么我们结合上次所学到的知识，我们可以确定到我们需要用到的插入点是ResultSetHandler

``` java
public interface ResultSetHandler {  
  
  <E> List<E> handleResultSets(Statement stmt) throws SQLException;  
  
  <E> Cursor<E> handleCursorResultSets(Statement stmt) throws SQLException;  
  
  void handleOutputParameters(CallableStatement cs) throws SQLException;  
  
}
```
那么很自然的可以看到有一个`handleResultSets`方法，那么我们就对这个方法进行拦截。那么就得到了以下的拦截注解
``` java
@Intercepts({@Signature(  
        type = ResultSetHandler.class,  
        method = "handleResultSets",  
        args = {Statement.class}  
)})
```

#### 标记数据
标记数据我们最常用的就是自定义注解了，所以这里我们就创建一个自定义注解，用来标记我们需要脱敏的数据
``` java
@Documented  
@Inherited  
@Retention(RetentionPolicy.RUNTIME)  
@Target({ElementType.FIELD})  
public @interface Desensitized {  
}
```

#### 多种脱敏策略
我们可以借助在我们创建的自定义注解`Desensitized`添加策略字段，来标记使用哪一种策略，我们这里创建一个策略接口，并提供一个默认实现类
``` java
public interface DesensitizeStrategy<T, R> {  
  
    R doDesensitize(T data);  
}

public class DefaultDesensitizeStrategy implements DesensitizeStrategy<String, String> {  
    @Override  
    public String doDesensitize(String data) {  
        return data;  
    }  
}
```

OK,方案已经定好了，那么下一次我们来将整体的方案进行整合和调整，来实现我们自己DIY的一个数据脱敏插件