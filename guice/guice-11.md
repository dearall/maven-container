## 第11章 自定义 Scope ##


通常建议用户不要编写自定义的 scope，内置的 scope 足够大多数应用程序的使用。如果编写 web 应用，`ServletModule ` 模块为 HTTP request 和 HTTP session 提供了简单的、经过测试的 scope 实现。

创建自定义的 scope 分为几个步骤：

1. 定义 scope 注解
2. 实现 Scope 接口
3. 将 scope 注解连接到实现
4. 触发 scope 进入 entry 和退出 exit



<br/><br/>
<a id="1"></a>

## 11.1 定义 scope 注解 ##

scope 注解标识出我们的 scope。要用它来注解 Guice 构造的类型、`@Provides` 注解的方法、以及在一个 `bind()` 语句中的 `in()` 子句。复制并自定义（Copy-and-customize）下面的代码来定义自己的 scoping 注解：

```java
import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.ElementType.TYPE;
import java.lang.annotation.Retention;
import static java.lang.annotation.RetentionPolicy.RUNTIME;
import java.lang.annotation.Target;

@Target({ TYPE, METHOD })
@Retention(RUNTIME)
@ScopeAnnotation
public @interface BatchScoped {}
```



<br/><br/>
<a id="2"></a>

## 11.2 实现 Scope 接口 ##

Scope 接口保证每个 scope 实例最多有一个类型实例。`SimpleScope` 是一个每线程（per-thread）实现的正规起始点。复制下面的代码并调整以适应子句的需要：

```java
import static com.google.common.base.Preconditions.checkState;
import com.google.common.collect.Maps;
import com.google.inject.Key;
import com.google.inject.OutOfScopeException;
import com.google.inject.Provider;
import com.google.inject.Scope;
import java.util.Map;

/**
 * Scopes a single execution of a block of code. Apply this scope with a
 * try/finally block: <pre><code>
 *
 *   scope.enter();
 *   try {
 *     // explicitly seed some seed objects...
 *     scope.seed(Key.get(SomeObject.class), someObject);
 *     // create and access scoped objects
 *   } finally {
 *     scope.exit();
 *   }
 * </code></pre>
 *
 * The scope can be initialized with one or more seed values by calling
 * <code>seed(key, value)</code> before the injector will be called upon to
 * provide for this key. A typical use is for a servlet filter to enter/exit the
 * scope, representing a Request Scope, and seed HttpServletRequest and
 * HttpServletResponse.  For each key inserted with seed(), you must include a
 * corresponding binding:

 *  <pre><code>
 *   bind(key)
 *       .toProvider(SimpleScope.&lt;KeyClass&gt;seededKeyProvider())
 *       .in(ScopeAnnotation.class);
 * </code></pre>
 *
 * @author Jesse Wilson
 * @author Fedor Karpelevitch
 */
public class SimpleScope implements Scope {

  private static final Provider<Object> SEEDED_KEY_PROVIDER =
      new Provider<Object>() {
        public Object get() {
          throw new IllegalStateException("If you got here then it means that" +
              " your code asked for scoped object which should have been" +
              " explicitly seeded in this scope by calling" +
              " SimpleScope.seed(), but was not.");
        }
      };
  private final ThreadLocal<Map<Key<?>, Object>> values
      = new ThreadLocal<Map<Key<?>, Object>>();

  public void enter() {
    checkState(values.get() == null, "A scoping block is already in progress");
    values.set(Maps.<Key<?>, Object>newHashMap());
  }

  public void exit() {
    checkState(values.get() != null, "No scoping block in progress");
    values.remove();
  }

  public <T> void seed(Key<T> key, T value) {
    Map<Key<?>, Object> scopedObjects = getScopedObjectMap(key);
    checkState(!scopedObjects.containsKey(key), "A value for the key %s was " +
        "already seeded in this scope. Old value: %s New value: %s", key,
        scopedObjects.get(key), value);
    scopedObjects.put(key, value);
  }

  public <T> void seed(Class<T> clazz, T value) {
    seed(Key.get(clazz), value);
  }

  public <T> Provider<T> scope(final Key<T> key, final Provider<T> unscoped) {
    return new Provider<T>() {
      public T get() {
        Map<Key<?>, Object> scopedObjects = getScopedObjectMap(key);

        @SuppressWarnings("unchecked")
        T current = (T) scopedObjects.get(key);
        if (current == null && !scopedObjects.containsKey(key)) {
          current = unscoped.get();

          // don't remember proxies; these exist only to serve circular dependencies
          if (Scopes.isCircularProxy(current)) {
            return current;
          }

          scopedObjects.put(key, current);
        }
        return current;
      }
    };
  }

  private <T> Map<Key<?>, Object> getScopedObjectMap(Key<T> key) {
    Map<Key<?>, Object> scopedObjects = values.get();
    if (scopedObjects == null) {
      throw new OutOfScopeException("Cannot access " + key
          + " outside of a scoping block");
    }
    return scopedObjects;
  }

  /**
   * Returns a provider that always throws exception complaining that the object
   * in question must be seeded before it can be injected.
   *
   * @return typed provider
   */
  @SuppressWarnings({"unchecked"})
  public static <T> Provider<T> seededKeyProvider() {
    return (Provider<T>) SEEDED_KEY_PROVIDER;
  }
}

```

<br/><br/>
<a id="3"></a>

## 11.3 注册 Scope ##

必须将 scoping 注解与对应的 scope 实现连接起来。类似于绑定，可以在模块的 `configure()` 方法中配置：

```java
public final class BatchScopeModule extends AbstractModule {
  private final SimpleScope batchScope = new SimpleScope();

  @Override
  protected void configure() {
    // tell Guice about the scope
    bindScope(BatchScoped.class, batchScope);
  }

  @Provides
  @Named("batchScope")
  SimpleScope provideBatchScope() {
    return batchScope;
  }
}

public final class ApplicationModule extends AbstractModule {
  @Override
  protected void configure() {
    install(new BatchScopeModule());
  }

  // Foo is bound in BatchScoped
  @Provides
  @BatchScoped
  Foo provideFoo() {
    return FooFactory.createFoo();
  }
}

```

也将 scope 实现绑定为 `@Named("batchScope") SimpleScope`。这是必要的，因为下一步需要使用 `batchScope` 触发 scope。


<br/><br/>
<a id="4"></a>

## 11.4 触发 Scope ##

自定义的 Scope 实现要求手动进入和退出 scope。通常这些存在于低级的架构代码，例如 web 服务器的进入点。例如，要使用 `SimpleScope ` 运行一段代码：

```java
  @Inject @Named("batchScope") SimpleScope scope;

  /**
   * Runs {@code runnable} in batch scope.
   */
  public void scopeRunnable(Runnable runnable) {
    scope.enter();
    try {
      // explicitly seed some seed objects...
      scope.seed(Key.get(SomeObject.class), someObject);

      // create and access scoped objects
      runnable.run();

    } finally {
      scope.exit();
    }
  }
```

确保在 finally 子句中调用 exit() 方法，否则抛出异常时 scope 会一直打开。




















