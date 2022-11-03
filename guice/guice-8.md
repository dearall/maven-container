## 第8章 Aspect Oriented Programming, AOP ##

使用 Guice 拦截方法（Intercepting methods with Guice）。

为了完成依赖注入，Guice 支持**方法拦截（method interception）**。这个特性使我们可以编写每次执行调用一个**匹配方法（matching method ）** 的代码。非常适合横切相关的（cross cutting concerns ("切面 aspects")）需求，例如事务（transaction）、安全（security）和日志记录（logging）。因为拦截器（interceptor）将一个问题切分（divide）为多个切面（aspect），而不是对象，因此它们的使用称为**面向切面编程（Aspect Oriented Programming (AOP)）** 。

大多数开发者不会直接编写方法拦截器，但他们可能在集成的类库上看到过拦截器的使用，例如 Warp Persist。所有需要做的就是，选择匹配的方法，创建拦截器，然后配置它们，全部在一个模块中进行。

[Matcher](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/matcher/Matcher.html) 是一个简单的接口，或者接受或者拒绝一个值。对于 AOP，需要两个 matcher：一个用于定义哪些类参与进来，另一个类定义这些类的方法。为了使其容易实现，提供了一个[工厂类 factory class](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/matcher/Matchers.html) 来满足通用的场景。

当一个匹配方法被调用时，[MethodInterceptors](https://aopalliance.sourceforge.net/doc/org/aopalliance/intercept/MethodInterceptor.html)被执行。他们有机会检查其调用：调用的方法、它的参数、以及接收的实例。他们可以执行它们的横切逻辑（cross-cutting logic），然后委托给底层方法。最后，可以检查其返回值，或者抛出异常，或者返回。因为拦截器可以被应用到很多方法上并接收很多调用，它们的实现应该高效和非侵入式的。


<br/><br/>
<a id="1"></a>

## 8.1 实例：拒绝在周末进行方法调用 ##

为了演示方法拦截器如何与 Guice 进行工作，我们将拒绝披萨支持系统在周末调用。送货员只能在周一至周五工作，因此在披萨不能送货时，我们防止披萨下订单。这个示例在结构上与使用 AOP 授权类似。

为了标记选择方法只能在工作日调用，定义一个注解：

```java
@Retention(RetentionPolicy.RUNTIME) @Target(ElementType.METHOD)
@interface NotOnWeekends {}

```

然后把它应用到需要被拦截的方法上：

```java
public class RealBillingService implements BillingService {

  @NotOnWeekends
  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    ...
  }
}
```

下一步，通过实现 org.aopalliance.intercept.MethodInterceptor 接口来定义拦截器。当需要通过底层调用的时候，通过调用 `invocation.proceed()`：

```java
public class WeekendBlocker implements MethodInterceptor {
  public Object invoke(MethodInvocation invocation) throws Throwable {
    Calendar today = new GregorianCalendar();
    if (today.getDisplayName(DAY_OF_WEEK, LONG, ENGLISH).startsWith("S")) {
      throw new IllegalStateException(
          invocation.getMethod().getName() + " not allowed on weekends!");
    }
    return invocation.proceed();
  }
}

```

最后，配置所有的东西。这就是为类和方法被拦截创建 matcher 的地方。在本例中，我们匹配任何类，但只有带自己定义的 `@NotOnWeekends` 注解的方法：

```java
public class NotOnWeekendsModule extends AbstractModule {
  protected void configure() {
    bindInterceptor(Matchers.any(), Matchers.annotatedWith(NotOnWeekends.class),
        new WeekendBlocker());
  }
}
```

把它组合到整体程序中，等到周六，会看到方法被拦截了，并且订单被拒绝：

```java
Exception in thread "main" java.lang.IllegalStateException: chargeOrder not allowed on weekends!
  at com.publicobject.pizza.WeekendBlocker.invoke(WeekendBlocker.java:65)
  at com.google.inject.internal.InterceptorStackCallback.intercept(...)
  at com.publicobject.pizza.RealBillingService$$EnhancerByGuice$$49ed77ce.chargeOrder(<generated>)
  at com.publicobject.pizza.WeekendExample.main(WeekendExample.java:47)
```


<br/><br/>
<a id="2"></a>

## 8.2 约束 ##

在幕后，方法拦截是通过在运行时生成字节码实现的。Guice 为应用的拦截器动态创建一个子类，来重写方法。如果在一个不支持字节码生成的系统上（例如 Android）使用，应该使用没有 AOP 支持的 Guice（[Guice without AOP support](https://github.com/google/guice/wiki/OptionalAOP)）。


这个方法强制什么样的类和方法可被拦截的约束：

- 类必须是 public 或 package-private
- 类必须是非 final 的
- 方法必须是 public, package-private, 或 protected 的
- 方法必须是非 final 的
- 实例必须由 Guice 通过 `@Inject` 注解或无参数构造器创建的。在不是由 Guice 创建的实例上使用方法拦截是不可能的。



<br/><br/>
<a id="3"></a>

## 8.3 注入拦截器 Injecting Interceptors ##

如果需要将依赖注入拦截器，使用 `requestInjection` API：

```java
public class NotOnWeekendsModule extends AbstractModule {
  protected void configure() {
    WeekendBlocker weekendBlocker = new WeekendBlocker();
    requestInjection(weekendBlocker);
    bindInterceptor(Matchers.any(), Matchers.annotatedWith(NotOnWeekends.class),
       weekendBlocker);
  }
}

```

另一个选择是使用 Binder.getProvider 并把依赖传递给拦截器的构造器：

```java
public class NotOnWeekendsModule extends AbstractModule {
  protected void configure() {
    bindInterceptor(any(),
                    annotatedWith(NotOnWeekends.class),
                    new WeekendBlocker(getProvider(Calendar.class)));
  }
}

```

使用注入拦截器时要小心，如果拦截器调用一个其本身就在拦截的方法，可能会收到 `StackOverflowException` 异常，这是由于无结束递归调用。


<br/><br/>
<a id="4"></a>

## 8.4 AOP 联盟 AOP Alliance ##

Guice 实现的方法拦截器 API 是 AOP 联盟公开规范的一部分。这使得跨各种不同框架间使用同一拦截器成为可能。








