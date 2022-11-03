## 第12章 自定义注入 ##

通过类型和注入监听器执行自定义注入。

除了使用标准的 `@Inject` 驱动注入，Guice 为自定义注入提供了钩子（hook）。这使 Guice 驻留其它具有自己的注入语义或注解的框架。大多数用户不直接使用自定义注入，但他们可能在扩展或第三方类库中看到过它们的使用。每个自定义注入需要一个类型监听器（a type listener），一个注入监听器（an injection listener），并进行监听器注册。

[TypeListener](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/TypeListener.html) 获得 Guice 注入类型的通知。因为类型监听器每个类型只通知一次（once-per-type），它们是执行潜在比较慢的操作最好的地方，例如列举类型的成员。随着它们检查的完成，类型监听器可以为值注入注册实例监听器。

[MembersInjector](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/MembersInjector.html) 和 [InjectionListener](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/InjectionListener.html) 可用于在 Guice 注入一个实例之后接收回调。实例首先由 Guice 注入，然后自定义成员 injector，最后，注入监听器被通知。因为它们是每个实例（once-per-instance）通知的，所以它们的执行应该尽可能地快。


<br/><br/>
<a id="1"></a>

## 12.1 示例：注入一个 Log4J Logger ##

Guice 对注入 `java.util.Logger` 实例提供内置支持，使用被注入实例的类型名命名。通过类型监听器 API，可以使用相同的命名规则注入一个 org.apache.log4j.Logger。我们以这种格式注入字段：

```java
import org.apache.log4j.Logger;
import com.publicobject.log4j.InjectLogger;

public class PaymentService {
  @InjectLogger Logger logger;

  ...
}

```

在一个模块中注册自定义的类型侦听器 `Log4JTypeListener`。使用一个 matcher 来选择要侦听哪种类型：

```java
  @Override protected void configure() {
    bindListener(Matchers.any(), new Log4JTypeListener());
  }
```

可以实现 [TypeListener](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/TypeListener.html) 来扫描一个类型的全部字段，以查找 Log4J logger。对每一个遇到的 logger 字段，在传递进来的 [TypeEncounter](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/TypeEncounter.html) 上注册一个 `Log4JMembersInjector `：

```java
  class Log4JTypeListener implements TypeListener {
    public <T> void hear(TypeLiteral<T> typeLiteral, TypeEncounter<T> typeEncounter) {
      Class<?> clazz = typeLiteral.getRawType();
      while (clazz != null) {
        for (Field field : clazz.getDeclaredFields()) {
          if (field.getType() == Logger.class &&
            field.isAnnotationPresent(InjectLogger.class)) {
            typeEncounter.register(new Log4JMembersInjector<T>(field));
          }
        }
        clazz = clazz.getSuperclass();
      }
    }
  }
```

最后，实现 `Log4JMembersInjector` 来设置 logger。在本例中，我们总是设置这个字段为同一个实例。在自己的应用中，可能需要计算或从一个 provider 请求一个值：

```java
  class Log4JMembersInjector<T> implements MembersInjector<T> {
    private final Field field;
    private final Logger logger;

    Log4JMembersInjector(Field field) {
      this.field = field;
      this.logger = Logger.getLogger(field.getDeclaringClass());
      field.setAccessible(true);
    }

    public void injectMembers(T t) {
      try {
        field.set(t, logger);
      } catch (IllegalAccessException e) {
        throw new RuntimeException(e);
      }
    }
  }
  
```








