	上次我们已经定义好了插件的一些大致实现方案，那么接下来我们就将上诉方案进行整合，来完成我们的数据
	脱敏插件

## 整合

### 第一步将集合类型和对象类型分开处理

由于我们使用查询接口的时候，有时候会使用`List`等集合来接收数据，也有时候用单一的对象类型去接收数据，所以我们在处理返回值的时候需要分别对这两种情况进行处理，所以我们就需要将他们分开来处理：
``` java
@Override  
public Object intercept(Invocation invocation) throws Throwable {  
    Object proceed = invocation.proceed();  
    Class<?> resultClass = proceed.getClass();  
    if (Collection.class.isAssignableFrom(resultClass)) {  
        Collection<?> result = (Collection<?>) proceed;  
        if (result.isEmpty()) {  
            return result;  
        }  
        Class<?> resultMapClazz = result.stream().findFirst().get().getClass();  
        result.forEach(data -> doDesensitize(resultMapClazz, data));  
    } else {  
        doDesensitize(resultClass, proceed);  
    }  
    return proceed;  
}
```
由以上方法，我们就可以将集合类型的数据和对象类型的数据分别处理，同时对于集合类型内的数据我们其实，处理方式和对象类型数据处理方式一样，所以我们只需要使用`forEach`方法对其进行迭代处理即可

### 抽象处理方法

通过上一个步骤，我们发现其实集合类型的处理和对象类型的处理已经被我们抽象成一致的方式了，所以我们只需要定义好对应的方法：
``` java
private void doDesensitize(Class<?> resultMapClazz, Object data)
```
接下来我们就是逐步去实现该方法即可

#### 获取当前对象需要处理的字段

由于我们在对接收对象进行定义的时候已经对数据进行了标记，所以我们只需要通过获取字段身上的注解，即可判断当前的字段需不需要进行脱敏处理，首先我们这里需要用到一些反射的知识，所以我们提前做一些工具类来方便后期的使用，如下：
``` java
public class ClassUtil {  
  
    public static Method getMethod(Class<?> clazz, MethodTYpe type, Field field) throws Exception {  
        String name = field.getName();  
        name = name.substring(0, 1).toUpperCase() + name.substring(1);  
        name = type.name().toLowerCase() + name;  
        return type.equals(MethodTYpe.GET) ?  
                clazz.getDeclaredMethod(name) :  
                clazz.getDeclaredMethod(name, field.getType());  
    }  
  
    public static List<Field> filterFieldList(List<Field> fieldList, Class<?> annotationType) {  
        return fieldList.stream()  
                .filter(field -> Arrays.stream(field.getAnnotations())  
                        .anyMatch(annotation -> annotation.annotationType().equals(annotationType)))  
                .collect(Collectors.toList());  
  
    }  
  
    public enum MethodTYpe {  
        GET, SET  
    }  
  
}
```

所以我们很方便就可以得到我们想要的需要脱敏的字段：
``` java
List<Field> desensitizedFields = ClassUtil  
        .filterFieldList(Arrays.asList(resultMapClazz.getDeclaredFields()), Desensitized.class);
```

#### 获取当前字段的GET/SET方法
同理我们运用我们的工具类也可以很轻易的得到：
``` java
Method getMethod = ClassUtil.getMethod(resultMapClazz, ClassUtil.MethodTYpe.GET, field);  
Object invoke = getMethod.invoke(data);  
if (invoke == null) {  
    continue;  
}  
Method setMethod = ClassUtil.getMethod(resultMapClazz, ClassUtil.MethodTYpe.SET, field);
```

当然我们也可以看出，如果获取到的数据是`null`的话，那么我们就没必要进行处理了（当然我们也可以自动帮他填充成一个默认对象，但是这个一般有风险，不建议使用）

#### 获取策略类
当所有的基础方法都准备完毕之后，我们还记得，我们保留了一个策略类在我们的注解中，那么我们同样需要从注解中拿到对应的策略类
``` java
Desensitized annotation = field.getAnnotation(Desensitized.class);  
Class<?> strategy = annotation.strategy();  
DesensitizeStrategy desensitizeStrategy = (DesensitizeStrategy) strategy.newInstance();
```

至此我们将整串代码组合在一起，那么就形成了我们所需要的模板代码:
``` java
private void doDesensitize(Class<?> resultMapClazz, Object data) {  
    List<Field> desensitizedFields = ClassUtil  
            .filterFieldList(Arrays.asList(resultMapClazz.getDeclaredFields()), Desensitized.class);  
    if (desensitizedFields.isEmpty()) {  
        return;  
    }  
  
    for (Field field : desensitizedFields) {  
        try {  
            Method getMethod = ClassUtil.getMethod(resultMapClazz, ClassUtil.MethodTYpe.GET, field);  
            Object value = getMethod.invoke(data);  
            if (value == null) {  
                continue;  
            }  
            Method setMethod = ClassUtil.getMethod(resultMapClazz, ClassUtil.MethodTYpe.SET, field);  
  
            Desensitized annotation = field.getAnnotation(Desensitized.class);  
            Class<?> strategy = annotation.strategy();  
            DesensitizeStrategy desensitizeStrategy = (DesensitizeStrategy) strategy.newInstance();  
            setMethod.invoke(data, desensitizeStrategy.doDesensitize(value));  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
}
```

剩下的就是留给我们的策略类的具体实现了，那么下次我们再去实现我们自己的策略类，来完善我们当前的这个插件