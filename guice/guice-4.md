## 第4章 Guice 的绑定 Bindings ##

在 Guice 中，一个绑定 binding 是一个对象，对应 "Guice map" 中的一个条目。客户端通过创建绑定的方式向 "Guice map" 中添加新的条目。


<br/><br/>

#### <font size=5 color=green><b>创建绑定</b></font> ####

要创建绑定，从 `AbstractModule` 抽象类继承并重写其 `configure` 方法。在 `configure` 方法体内，调用 `bind()` 方法指定每一个绑定。这些方法都是类型检查的，因此如果使用了错误的类型，编译器会报告错误。创建好模块之后，将它们作为参数传递给 Guice.createInjector() 方法构建注入器 Injector 实例，通过 Injector 就可以启动应用程序了。

使用模块可以创建 linked bindings, instance bindings, @Provides methods, provider bindings, constructor bindings 以及 untargetted bindings。



<br/><br/>

#### <font size=5 color=green><b>其它绑定 More Bindings</b></font> ####

除了指定的绑定，注入器 injector 也包含内置绑定 [built-in bindings](https://github.com/google/guice/wiki/BuiltInBindings) 。当请求一个依赖但没有找到时，Guice 会尝试创建一个 [just-in-time binding](https://github.com/google/guice/wiki/JustInTimeBindings)。注入器也为 [provider](https://github.com/google/guice/wiki/InjectingProviders)包含其它绑定。


<br/><br/>
<a id="1"></a>

## 4.1 链式绑定 Linked Binding ##

链式绑定 Linked Binding 将一个类型映射到它的实现。下面的例子将接口 `TransactionLog` 映射到它的实现 `DatabaseTransactionLog` 类：

```java
public class BillingModule extends AbstractModule {
  @Provides
  TransactionLog provideTransactionLog(DatabaseTransactionLog impl) {
    return impl;
  }
}
```

现在，当调用 `injector.getInstance(TransactionLog.class)` 时，或者注入器遇到一个 `TransactionLog` 依赖时，它会使用 `DatabaseTransactionLog` 类。

链式绑定可以链接起来形成绑定链：

```java
public class BillingModule extends AbstractModule {
  @Provides
  TransactionLog provideTransactionLog(DatabaseTransactionLog databaseTransactionLog) {
    return databaseTransactionLog;
  }

  @Provides
  DatabaseTransactionLog provideDatabaseTransactionLog(MySqlDatabaseTransactionLog impl) {
    return impl;
  }
}
```

这种情况下，当请求一个 TransactionLog 对象时，注入器 injector 会返回一个 `MySqlDatabaseTransactionLog` 实例。

除了使用 @Provides 注解方法的形式创建链式绑定，也可以在 Module 中使用 `bind()` 方法创建。例如，下面的代码将具体的 `DatabaseTransactionLog` 链接到它的一个子类 `MySqlDatabaseTransactionLog`：

```java
    bind(DatabaseTransactionLog.class).to(MySqlDatabaseTransactionLog.class);
```


<br/><br/>
<a id="2"></a>

## 4.2 绑定注解 Binding Annotations ##

偶尔，可能想要对同一类型进行多个绑定。例如，可能同时想要 PayPal 信用卡处理器和 Google Checkout 处理器。绑定支持一个可选的绑定注解（binding annotation）用于解决这个问题。注解和类型一起唯一地标识了一个绑定，这样的一对信息称为一个**键（key）**。


<br/><br/>

#### <font size=5 color=green><b>定义绑定注解 Defining binding annotations</b></font> ####

绑定注解是通过元注解 `@Qualifier` 或 `@BindingAnnotation` 注解的 Java 注解。

定义一个绑定注解要求两行代码，把它放到自己的 .java 源文件或者它要注解的类型的源文件中。如下示例展示：

```java
package example.pizza;

import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

import java.lang.annotation.Target;
import java.lang.annotation.Retention;
import javax.inject.Qualifier;

@Qualifier
@Target({ FIELD, PARAMETER, METHOD })
@Retention(RUNTIME)
public @interface PayPal {}

// Older code may still use Guice `BindingAnnotation` in place of the standard
// `@Qualifier` annotation. New code should use `@Qualifier` instead.
@BindingAnnotation
@Target({ FIELD, PARAMETER, METHOD })
@Retention(RUNTIME)
public @interface GoogleCheckout {}

```

其中：

- `@Qualifier` 是 JSR-330 规范定义的元注解（meta-annotation），由 [javax.inject](http://javax-inject.github.io/javax-inject/) 项目维护，它告诉 Guice 这是一个绑定注解。如果有多个绑定注解应用到同一个类成员上，Guice 会产生一个错误。Guice 的 `@BindingAnnotation` 在旧代码中也用于同一个目的。
- `@Target({FIELD, PARAMETER, METHOD})` 元注解防止 @PayPal 注解不小心被应用到没有目标的地方。
- `@Retention(RUNTIME)` 使注解在运行时可用，这是 Guice 所要求的。

为了依赖被注解的绑定，将注解应用到被注入的参数上：

```java
public class RealBillingService implements BillingService {

  @Inject
  public RealBillingService(@PayPal CreditCardProcessor processor,
      TransactionLog transactionLog) {
    ...
  }
```

最后，使用定义的注解创建绑定：

```java
final class CreditCardProcessorModule extends AbstractModule {
  @Override
  protected void configure() {
    // This uses the optional `annotatedWith` clause in the `bind()` statement
    bind(CreditCardProcessor.class)
        .annotatedWith(PayPal.class)
        .to(PayPalCreditCardProcessor.class);
  }

  // This uses binding annotation with a @Provides method
  @Provides
  @GoogleCheckout
  CreditCardProcessor provideCheckoutProcessor(
      CheckoutCreditCardProcessor processor) {
    return processor;
  }
}
```


<br/><br/>

#### <font size=5 color=green><b>@Named</b></font> ####

Guice 自带了一个内置的绑定注解 `@Named`，带一个字符串型参数。用法如下：

```java
public class RealBillingService implements BillingService {

  @Inject
  public RealBillingService(@Named("Checkout") CreditCardProcessor processor,
      TransactionLog transactionLog) {
    ...
  }
```

要绑定到具体的名字，使用 `Names.named()` 方法创建一个实例传递给 `annotatedWith()` 方法：

```java
final class CreditCardProcessorModule extends AbstractModule {
  @Override
  protected void configure() {
    bind(CreditCardProcessor.class)
      .annotatedWith(Names.named("Checkout"))
      .to(CheckoutCreditCardProcessor.class);
  }
}
```

由于编译器不能检查字符串表示的类型，因此建议少使用 `@Named` 注解。定义自己目的明确的注解提供更好的类型安全。



<br/><br/>
<a id="3"></a>

## 4.3 实例绑定 Instance Bindings ##

可以将一个类型绑定到这个类型的一个具体的实例上。通常只对其自身没有依赖的对象有用，例如值对象：

```java
    bind(String.class)
        .annotatedWith(Names.named("JDBC URL"))
        .toInstance("jdbc:mysql://localhost/pizza");
    bind(Integer.class)
        .annotatedWith(Names.named("login timeout seconds"))
        .toInstance(10);
```

避免将 `toInstance()` 方法用于复杂对象的创建，因为它会降低应用启动的速度，可以通过 `@Provides` 注解方法的方式替代这种需求。

也可以使用 `bindConstant()` 方法绑定常量：

```java
  bindConstant()
      .annotatedWith(HttpPort.class)
      .to(8080);
```

`bindConstant()` 方法是绑定原生类型，以及其它常量类型如 String，enum，Class 的快捷方式。




<br/><br/>
<a id="4"></a>

## 4.4 使用 @Provides 注解方法 @Provides Methods ##

在需要通过代码创建一个对象的时候，使用 `@Provides` 注解方法。这个方法必须定义在一个模块中，而且方法必须有 `@Provides` 注解。方法的返回类型绑定到要创建的实例类型上。无论何时 injector 需要一个这个类型的实例，它都会调用这个方法：

```java
public class BillingModule extends AbstractModule {
  @Override
  protected void configure() {
    ...
  }

  @Provides
  static TransactionLog provideTransactionLog() {
    DatabaseTransactionLog transactionLog = new DatabaseTransactionLog();
    transactionLog.setJdbcUrl("jdbc:mysql://localhost/pizza");
    transactionLog.setThreadPoolSize(30);
    return transactionLog;
  }
}

```

被 `@Provides` 注解的方法可以是静态方法，也可以是实例方法。


<br/><br/>

#### <font size=5 color=green><b>带有绑定注解 With Binding Annotation </b></font> ####

如果使用 `@Provides` 注解的方法有绑定注解，例如 `@PayPal` 或者 `@Named("Checkout")`，Guice 绑定这个被注解的类型。依赖可以作为参数传递给方法，注入器 injector 会在调用这个方法之前把每一个参数进行绑定，如下所示：

```java
  @Provides @PayPal
  CreditCardProcessor providePayPalCreditCardProcessor(
      @Named("PayPal API key") String apiKey) {
    PayPalCreditCardProcessor processor = new PayPalCreditCardProcessor();
    processor.setApiKey(apiKey);
    return processor;
  }

```

<br/><br/>

#### <font size=5 color=green><b>异常抛出</b></font> ####

Guice 不允许从 Provider 抛出异常。由 `@Provides` 注解的方法抛出的异常会被封装成 `ProvisionException`。从 `@Provides` 注解的方法中允许抛出任何类型的异常（runtime 或 checked）都是一个很差的实践。如果由于某些原因，确实需要抛出一个异常，可以使用 [ThrowingProviders extension](https://github.com/google/guice/wiki/ThrowingProviders) 的 `@CheckedProvides` 注解方法。




<br/><br/>
<a id="5"></a>

## 4.5 使用 Provider 绑定 ##

在 `@Provides` 注解的方法开始变得越来越复杂的时候，可以考虑把它们转移到它们自己的 provider 类中。provider 类实现 Guice 的 `Provider<T>` 接口，这个接口是个简单的、专门提供值的通用接口：

```java

public interface Provider<T> {
  T get();
}

```

我们的 provider 实现类有其自己的依赖，通过 `@Inject` 注解的构造器接收这些依赖。它通过实现 `Provider<T>` 接口来定义要返回的对象是完全类型安全的：

```java
public class DatabaseTransactionLogProvider implements Provider<TransactionLog> {
  private final Connection connection;

  @Inject
  public DatabaseTransactionLogProvider(Connection connection) {
    this.connection = connection;
  }

  public TransactionLog get() {
    DatabaseTransactionLog transactionLog = new DatabaseTransactionLog();
    transactionLog.setConnection(connection);
    return transactionLog;
  }
}

```

最后，在模块中通过 `.toProvider()` 子句绑定到 provider 类：

```java
public class BillingModule extends AbstractModule {
  @Override
  protected void configure() {
    bind(TransactionLog.class)
        .toProvider(DatabaseTransactionLogProvider.class);
  }
}

```

<br/><br/>

#### <font size=5 color=green><b>异常抛出</b></font> ####

Guice 不允许从 Provider 类抛出异常。`Provider` 接口也不允许抛出 checked 异常。`RuntimeExceptions` 可以封装为 `ProvisionException` 或 `CreationException`，如果因为某些原因确实需要抛出异常，可以使用 [ThrowingProviders extension](https://github.com/google/guice/wiki/ThrowingProviders)。


<br/><br/>
<a id="6"></a>

## 4.6 无目标绑定 Untargeted Binding ##

创建没有目标的绑定。

可以创建没有指定目标的绑定。这对具体的类，和被 `@ImplementedBy` 或者 `@ProvidedBy` 注解的类型最有用。一个无目标的绑定通知注入器 injector 一个类型，因此它可以渴望地准备依赖。无目标绑定没有 `to()` 子句，像这样：

```java
    bind(MyConcreteClass.class);
    bind(AnotherConcreteClass.class).in(Singleton.class);
```

在指定绑定注解时，必须要添加目标绑定，即使它与具体类是同一个类，如下所示：

```java
    bind(MyConcreteClass.class)
        .annotatedWith(Names.named("foo"))
        .to(MyConcreteClass.class);
    bind(AnotherConcreteClass.class)
        .annotatedWith(Names.named("foo"))
        .to(AnotherConcreteClass.class)
        .in(Singleton.class);
```

无目标的绑定经常用于在模块中注册一个类型，以使 Guice 感知到这个类型。在 [JIT](https://github.com/google/guice/wiki/JustInTimeBindings) 禁用的时候，为了使这些类型适合注入，注册这些类型是必须的。



<br/><br/>
<a id="7"></a>

## 4.7 构造器绑定 Constructor Binding ##

有些时候，将一个类型绑定到任意一个构造器上是由必要的。在 `@Inject` 注解不能应用到目标构造器上的时候，会出现下面这些问题：因为它是一个第三方提供的类，或者因为有多个构造器参与到依赖注入。通过 `@Provides` 对方法加注解为这个问题提供了最好的解决方案！通过显式地调用目标构造器，我们不需要反射及其相关的缺点。但这个方法有些限制：手动构造实例不能使它参与到 AOP 中。

为即解决这个问题，Guice 提供了 `toConstructor()` 绑定。它们要求通过反射选择目标构造器，并在不能找到构造器时处理异常：

```java
public class BillingModule extends AbstractModule {
  @Override
  protected void configure() {
    try {
      bind(TransactionLog.class).toConstructor(
          DatabaseTransactionLog.class.getConstructor(DatabaseConnection.class));
    } catch (NoSuchMethodException e) {
      addError(e);
    }
  }
}

```

在这个例子中，`DatabaseTransactionLog` 必须有一个带有单独一个 `DatabaseConnection` 参数的构造器。这个构造器不需要使用 `@Inject` 进行注解。Guice 会调用那个构造器以满足绑定。

每个 `toConstructor()` 绑定是独立进行 scope 的，如果创建多个 singleton 绑定，将目标设置到同一个构造器上，每一个绑定产生它自己的实例。


<br/><br/>
<a id="8"></a>

## 4.8 内置的绑定 Built-in Binding ##

更多可以使用的绑定。

警告：应该避免使用这些内置的绑定，除了 `Provider` 绑定。

除了显式地绑定和 [just-in-time bindings](https://github.com/google/guice/wiki/JustInTimeBindings)，伴随一些附加的绑定自动被包含到注入器 injector 中。只有 injector 能创建这些绑定，尝试自己绑定它们是错误的。

<br/><br/>

#### <font size=5 color=green><b>Logger</b></font> ####

Guice 为 java.util.logging.Logger 提供了内置绑定，是为了节省一些样板代码。绑定自动将 logger 的名字设置为 Logger 被注入的类的名字：

```java
@Singleton
public class ConsoleTransactionLog implements TransactionLog {

  private final Logger logger;

  @Inject
  public ConsoleTransactionLog(Logger logger) {
    this.logger = logger;
  }

  public void logConnectException(UnreachableException e) {
    /* the message is logged to the "ConsoleTransacitonLog" logger */
    logger.warning("Connect exception failed, " + e.getMessage());
  }
```




<br/><br/>

#### <font size=5 color=green><b>Injector</b></font> ####

在框架代码中，有时候并不知道需要的类型，直到运行时。在这种很少的情况下，应该注入 injector。注入 injector 的代码不能自我描述它的依赖，因此这种方法应少用。




<br/><br/>

#### <font size=5 color=green><b>Providers</b></font> ####

对 Guice 知道的每一个类型，它也能够注入那个类型的一个 Provider，[Injecting Providers ](https://github.com/google/guice/wiki/InjectingProviders) 详细阐述。




<br/><br/>

#### <font size=5 color=green><b>TypeLiterals</b></font> ####

Guice 对它注入的任何东西都有完全的类型信息。如果注入参数化类型，可以注入一个 `TypeLiteral<T>` 以反射的方式告诉我们元素的类型。



<br/><br/>

#### <font size=5 color=green><b>阶段 Stage</b></font> ####

Guice 支持一个 stage 枚举以区分开发和生产运行。



<br/><br/>

#### <font size=5 color=green><b>MembersInjector</b></font> ####

在绑定到 provider 或编写扩展时，可能想要 Guice 将依赖注入到一个自己构建的对象中。为此，在一个 `MembersInjector<T>` 上添加一个依赖，这里 T 是对象的类型，然后调用 `membersInjector.injectMembers(myNewObject)` 方法。`membersInjector.injectMembers()` 被调用时，Guice 会在 `myNewObject` 上执行[ field and method injection](https://github.com/google/guice/wiki/Injections)。



<br/><br/>
<a id="9"></a>

## 4.9 Just-in-time Binding ##

Guice 自动创建的绑定。

在注入器 injector 需要一个类型的实例时，它需要绑定。在模块中的绑定称为显式绑定（explicit binding），injector 在它们可用时使用这些绑定的类型。如果需要某个类型，但它没有被显式绑定，injector 会尝试创建一个**即时绑定 （Just-In-Time binding）**，也被称为 **JIT binding** 或者**隐式绑定（implicit binding）**。


<br/><br/>

#### <font size=5 color=green><b>使用 @Inject 注解构造器</b></font> ####

通过类型的**可注入构造器（injectable constructor）**，Guice 能为具体类型创建绑定。Guice 认为下面情况的构造器是可注入的构造器：

- 构造器显式由 @Inject 注解，同时支持 com.google.inject.Inject 和 javax.inject.Inject 注解。这是**推荐的方式**。
- 或者，构造器带有 0 个参数，并且：
  - 构造器是**非 private** 的，并且定义在一个**非 private** 的类中。（Guice 中支持在 pivate 类中的 private 构造器，然而，private 构造器是不建议使用的，因为它们在反射方面的开销，会使 Guice 变慢）。
  - injector 没有选择要求显式通过 `@Inject` 注解的构造器。

可注入构造器示例：

```java
public final class Foo {
  // An @Inject annotated constructor.
  @Inject
  Foo(Bar bar) {
    ...
  }
}

public final class Bar {
  // A no-arg non private constructor.
  Bar() {}

  private static class Baz {
    // A private constructor to a private class is also usable by Guice, but
    // this is not recommended since it can be slow.
    private Baz() {}
  }
}

```

如果是下面的情况，则构造器是不可注入的（not injectable）：

- 构造器带有一个或多个参数，并且没有通过 `@Inject` 进行注解
- 有多于一个被 `@Inject` 注解的构造器
- 构造器定义在一个非静态的内部类中（non-static nested class）。内部类有一个外部包含类的隐式引用，因此不能被注入。

非可注入构造器示例：

```java
public final class Foo {
  // Not injectable because the construct takes an argument and there is no
  // @Inject annotation.
  Foo(Bar bar) {
    ...
  }
}

public final class Bar {
  // Not injectable because the constructor is private
  private Bar() {}

  class Baz {
    // Not injectable because Baz is not a static inner class
    Baz() {}
  }
}

```

<br/><br/>

#### <font size=3 color=green><b>显式 @Inject Constructor 特性</b></font> ####

应用程序可以选择性地强制 Guice 只使用 `@Inject` 注解构造器：通过在安装到 injector 的模块中调用 `binder().requireAtInjectRequired()` 方法。如果做了这样的选择，Guice 只认为带有 `@Inject` 注解的构造器是可注入的构造器，并且如果没有找到，就会报告 `MISSING_CONSTRUCTOR` 错误。

**提示**：调用 `Modules.requireAtInjectOnConstructorsModule()` 方法来选择要求在构造器上使用 `@Inject` 注解。


<br/><br/>

#### <font size=5 color=green><b>@ImplementedBy</b></font> ####

给类型加注解（Annotate type）告诉 injector 某个类型的默认实现是什么。`@ImplementedBy` 注解看起来像[链式绑定（linked binding）](guice/guice-4.md#1)，在构建类型实例时使用的子类型：

```java
@ImplementedBy(PayPalCreditCardProcessor.class)
public interface CreditCardProcessor {
  ChargeResult charge(String amount, CreditCard creditCard)
      throws UnreachableException;
}

```

上面的注解与下面的 bind() 语句等价：

```java
  bind(CreditCardProcessor.class).to(PayPalCreditCardProcessor.class);
```

如果一个类型，既通过 bind() 语句绑定，也通过 `@ImplementedBy` 注解，则使用 `bind()` 语句的绑定。这个注解是建议一个默认的实现（a default implementation），但可以被一个具体的绑定覆写（override）。使用 `@ImplementedBy` 要小心，它加入了一个从接口到其实现的编译时依赖。



<br/><br/>

#### <font size=5 color=green><b>@ProvidedBy</b></font> ####

`@ProvidedBy` 告诉 injector 产生实例的 `Provider` 类：

```java
@ProvidedBy(DatabaseTransactionLogProvider.class)
public interface TransactionLog {
  void logConnectException(UnreachableException e);
  void logChargeResult(ChargeResult result);
}

```

该注解等同于一个 `toProvider()` 绑定：

```java
  bind(TransactionLog.class)
      .toProvider(DatabaseTransactionLogProvider.class);
```

与 `@ImplementedBy` 类似，如果一个类型被 `@ProvidedBy` 注解，同时通过 `bind()` 语句绑定，则使用 `bind()` 语句的绑定。


<br/><br/>

#### <font size=5 color=green><b>强制显式的绑定</b></font> ####

要禁用隐式的绑定，可以使用 `requireExplicitBindings` API：

```java
final class ExplicitBindingModule extends AbstractModule {
  @Override
  protected void configure() {
    binder().requireExplicitBindings();
  }
}

```

将上述模块安装到 injector 中，将导致 Guice 强制所有的绑定必须列在一个 Module 中才能被注入。


<br/><br/>
<a id="10"></a>

## 4.10 多绑定 Multibindings  ##

注意：从 Guice 4.2 开始，multibindings 支持已移入 Guice 核心包，在此之前，需要依赖 `guice-multibindings` 扩展包。

[Multibinder](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/multibindings/Multibinder.html) 类和 [MapBinder](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/multibindings/MapBinder.html) 类是为了插件类型架构（plugin-type architectures）设计的，其中已获得了由 Servlets, Actions, Filters, Components 提供的多个模块。

[OptionalBinder](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/multibindings/OptionalBinder.html) 是为了框架用于：

- 定义一个注入点（injection point）可以或不必被用户绑定（may or may not be bound by users）。
- 提供一个默认值可以由用户改变。



<br/><br/>
<a id="101"></a>

## 4.10.1 多绑定 Multibinding  ##

由 `Multibinder` 或 `MapBinder` 类存储插件（to host plugins）。



<br/><br/>

#### <font size=5 color=green><b>Multibinder 类</b></font> ####

多绑定使得在应用程序中支持插件架构变得很容易。插件由 IDE 和浏览器变得流行起来，这种范式向外提供 API 以扩展应用程序的行为。

使用 Guice 框架，不管是插件的消费者还是插件作者，都不需要写大量的启动代码（setup code）。简单地定义一个接口，绑定实现，然后注入一组实现即可！任何模块都可以创建一个新的 `Multibinder` 以提供绑定到一组实现上（a set of implementations）。为了演示，这里使用插件来对 Twitter 上类似 `http://bit.ly/1mzgW1` 这样的 URI 提取某些可读的摘要信息。

首先，定义一个插件作者能实现的接口，通常是一个能使它自己对应几个实现的接口。例如，为每个要收集摘要信息的 web 站点写一个不同的实现。

```java
interface UriSummarizer {
  /**
   * Returns a short summary of the URI, or null if this summarizer doesn't
   * know how to summarize the URI.
   */
  String summarize(URI uri);
}

```

下一步，拿到插件作者实现这个接口的实现。下面是网络相簿（Flickr photo）URL 的实现：

```java
class FlickrPhotoSummarizer implements UriSummarizer {
  private static final Pattern PHOTO_PATTERN
      = Pattern.compile("http://www\\.flickr\\.com/photos/[^/]+/(\\d+)/");

  public String summarize(URI uri) {
    Matcher matcher = PHOTO_PATTERN.matcher(uri.toString());
    if (!matcher.matches()) {
      return null;
    } else {
      String id = matcher.group(1);
      Photo photo = lookupPhoto(id);
      return photo.getTitle();
    }
  }
}

```

插件作者通过一个 multibinder 对他们的实现进行注册。有些插件可以绑定多个实现，或者实现了多个扩展点接口（implementations of several extension-point interfaces）。

```java
public class FlickrPluginModule extends AbstractModule {
  public void configure() {
    Multibinder<UriSummarizer> uriBinder = Multibinder.newSetBinder(binder(), UriSummarizer.class);
    uriBinder.addBinding().to(FlickrPhotoSummarizer.class);

    ... // bind plugin dependencies, such as our Flickr API key
  }
}

```

现在就可以使用由插件提供的服务了。这次，对 tweets 提取摘要：

```java
public class TweetPrettifier {

  private final Set<UriSummarizer> summarizers;
  private final EmoticonImagifier emoticonImagifier;

  @Inject TweetPrettifier(Set<UriSummarizer> summarizers,
      EmoticonImagifier emoticonImagifier) {
    this.summarizers = summarizers;
    this.emoticonImagifier = emoticonImagifier;
  }

  public Html prettifyTweet(String tweetMessage) {
    ... // split out the URIs and call prettifyUri() for each
  }

  public String prettifyUri(URI uri) {
    // loop through the implementations, looking for one that supports this URI
    for (UrlSummarizer summarizer : summarizers) {
      String summary = summarizer.summarize(uri);
      if (summary != null) {
        return summary;
      }
    }

    // no summarizer found, just return the URI itself
    return uri.toString();
  }
}

```

注意，`Multibinder.newSetBinder(binder, type)` 方法可能会让人困惑，这个操作创建一个新的 binder，但没有覆写任何已存在的绑定。通过这种方式创建的 binder 为那个类型提供了现有的实现集。它只有在还没有一个被绑定时才创建一个新的 set。

最后，必须把插件自己注册到注入器 injector。最简单的机制是以编程方式把它们依次传递给 `Guice.createInjector()` 方法：

```java
public class PrettyTweets {
  public static void main(String[] args) {
    Injector injector = Guice.createInjector(
        new GoogleMapsPluginModule(),
        new BitlyPluginModule(),
        new FlickrPluginModule()
        ...
    );

    injector.getInstance(Frontend.class).start();
  }
}

```

如果每次插件集改变都重新编译主程序是不可行的，可以从一个配置文件载入插件模块列表。

注意，这种机制不能在系统运行时加载或卸载插件。如果需要热交换的（hot-swap）应用组件，可以探究 [Guice's OSGi](https://github.com/google/guice/wiki/OSGi)。


<br/><br/>

#### <font size=5 color=green><b>Multibinder 中的重复元素 Duplicate elements in Multibinder</b></font> ####

默认情况下，`Multibinder` 中不允许有重复的元素，并且一旦有重复的元素加入到 `Multibinder` 中，会抛出 `DUPLICATE_ELEMENT` 错误。注意，如果绑定同一个常量两次，Guice 本身会去重，这个错误只有在 provide 期间遇到时才会抛出，例如两个 provider 返回同一个值。为了允许重复的元素，可以在 `Multibinder` 上调用 `permitDuplicates()` 方法：

```java
public class FlickrPluginModule extends AbstractModule {
  @Override
  protected void configure() {
    Multibinder<UriSummarizer> uriBinder =
        Multibinder.newSetBinder(binder(), UriSummarizer.class);
    uriBinder.permitDuplicates();
  }
}

```

<br/><br/>
<a id="102"></a>

## 4.10.2 多绑定 MapBinder 类 ##

前面的示例展示了如何通过 `Multibinder` 绑定一个带有多个模块的 `Set<UrlSummarizer>`。Guice 也支持通过 `MapBinder` 绑定 `Map<K, V>`，例如 `Map<String, UrlSummarizer>`：

```java
public class FlickrPluginModule extends AbstractModule {
  public void configure() {
    MapBinder<String, UriSummarizer> uriBinder =
        MapBinder.newMapBinder(binder(), String.class, UriSummarizer.class);
    uriBinder.addBinding("Flickr").to(FlickrPhotoSummarizer.class);

    ... // bind plugin dependencies, such as our Flickr API key
  }
}

```

然后，应用程序就可以像这样注入 `Map<String, UriSummarizer>`：

```java
public class TweetPrettifier {

  private final Map<String, UriSummarizer> summarizers;

  @Inject TweetPrettifier(Map<String, UriSummarizer> summarizers) {
    this.summarizers = summarizers;
    ...
  }
}

```


<br/><br/>

#### <font size=5 color=green><b>MapBinder 中的重复的键 Duplicate keys in MapBinder </b></font> ####

与 `Multibinder` 类似，`MapBinder` 默认不允许重复的元素，并且如果有重复的元素加入到 `MapBinder`，会抛出 `DUPLICATE_ELEMENT` 错误。与 `Multibinder` 不同的是，唯一性要求是作用在 Map 条目的 **Key** 上，而非它的值上。

要允许重复，可以调用 `MapBinder` 的 `permitDuplicates()` 方法。当存在重复的 key 时，在最终 Map 中的实际值是无法确定的：

```java
public final class FooModule extends AbstractModule {
  @Override
  protected void configure() {
    MapBinder.newMapBinder(binder(), String.class, String.class)
        .permitDuplicates();
  }
}

public final class BarModule extends AbstractModule {
  @ProvidesIntoMap
  @StringMapKey("letter")
  String provideKeyValue() {
    return "a";
  }
}

public final class BazModule extends AbstractModule {
  @ProvidesIntoMap
  @StringMapKey("letter")
  String provideKeyValue() {
    return "b";
  }
}

```

在上面的例子中，key `letter` 在最终的 `Map<String, String>` 中可能值为 a 或 b。


<br/><br/>
<a id="103"></a>

## 4.10.3 多绑定 OptionalBinder 类 ##

`OptionalBinder` 类可用于提供可选的绑定。

框架经常为应用程序开发者提供配置 API 以自定义框架的行为。在 Guice 绑定用于自定义类型配置时，`OptionalBinder` 类能使可选绑定变得很容易。

例如，一个 web 框架可能有为应用提供可选的 `RequestLogger` 的 API，以记录请求和响应。

```java
 public class FrameworkLoggingModule extends AbstractModule {
   protected void configure() {
     OptionalBinder.newOptionalBinder(binder(), RequestLogger.class);
   }
 }

 ```

通过这个模块，一个 `Optional<RequestLogger>` 可用由框架代码注入，以在处理完响应之后，记录请求和响应：

```java
public class RequestHandler {
  private final Optional<RequestLogger> requestLogger;

  @Inject
  RequestHandler(Optional<RequestLogger> requestLogger) {
    this.requestLogger = requestLogger;
  }

  void handleRequest(Request request) {
    Response response = ...;
    if (requestLogger.isPresent()) {
      requestLogger.get().logRequest(request, response);
    }
  }
}

```

如果应用程序没有提供 `RequestLogger`，不会记录日志。如果应用程序像下面这样安装了一个：

```java
public class ConsoleLoggingModule extends AbstractModule {
  @Provides
  RequestLogger provideRequestLogger() {
    return new ConsoleLogger(System.out);
  }
}

```

框架代码会获得一个存在的值，`requestLogger.isPresent()` 返回 true 值，包含 `ConsoleLogger` 的一个实例来记录请求和响应的日志。


<br/><br/>

#### <font size=5 color=green><b>带有默认值的 OptionalBinder</b></font> ####

在上面的例子中，一个 `RequestLogger` 对框架是可选的，但并非总是这种情况。当需要一个绑定时，框架能使用 `OptionalBinder` 设置一个默认的绑定，可以被应用程序覆盖：

```java
public class FrameworkLoggingModule extends AbstractModule {
 protected void configure() {
   OptionalBinder.newOptionalBinder(binder(), RequestLogger.class)
       .setDefault()
       .to(DefaultRequestLoggerImpl.class);
 }
}

```

使用上面的模块，在应用程序没有为 `RequestLogger` 提供绑定时，框架使用 `DefaultRequestLoggerImpl` 作为默认值。 


<br/><br/>

#### <font size=5 color=green><b>覆盖默认值 Overriding the default</b></font> ####

如果模块使用 `setDefault()` 提供了默认值，覆盖默认值的唯一途径是使用 `setBinding()` 方法。如果用户没有通过 `OptionalBinder` 而调用 `setDefault()` 或 `setBinding()` 方法来指定绑定会出现错误。

因此要覆盖框架提供的默认 logger 绑定，应用程序可以像下面这样安装一个模块：

```java
public class ConsoleLoggingModule extends AbstractModule {
  @Override
  protected void configure() {
    OptionalBinder.newOptionalBinder(binder(), RequestLogger.class)
        .setBinding()
        .to(ConsoleLogger.class);
  }
}

```

<br/><br/>
<a id="104"></a>

## 4.10.4 使用 @Provides-like 方法 ##

除了在一个模块的 `configure()` 的方法中使用 `Multibinder` 和 `MapBinder` 创建多绑定之外，也可以使用 @Provides-like 方法来绑定到 `Multibinder`，`MapBinder` 或 `OptionalBinder`。



<br/><br/>

#### <font size=5 color=green><b>@ProvidesIntoSet</b></font> ####

```java
public class FlickrPluginModule extends AbstractModule {
  @ProvidesIntoSet
  UriSummarizer provideFlickerUriSummarizer() {
    return new FlickrPhotoSummarizer(...);
  }
}
```


<br/><br/>

#### <font size=5 color=green><b>@ProvidesIntoMap</b></font> ####

```java
public class FlickrPluginModule extends AbstractModule {
  @StringMapKey("Flickr")
  @ProvidesIntoMap
  UriSummarizer provideFlickrUriSummarizer() {
    return new FlickrPhotoSummarizer(...);
  }
}

```

`@ProvidesIntoMap` 要求一个额外的注解，以指定与这个绑定关联的 key。上面的例子使用 `@StringMapKey` 注解，它是一个内置的可以和 `@ProvidesIntoMap` 注解使用的注解，以将由 `provideFlickerUriSummarizer` 提供的绑定与 key "Flickr" 关联起来。

可以创建和 @ProvidesIntoMap 一起使用的自定义注解，通过使用 `MapKey` 注解进行注解，类似下面这样：

```java
@MapKey(unwrapValue=true)
@Retention(RUNTIME)
public @interface MyCustomEnumKey {
  MyCustomEnum value();
}

```

如果 unwrapValue = true，则自定义注解的 value 被用作 map 的 key，否则整个注解被用作 key。上面的例子 `MyCustomEnumKey` 设置 unwrapValue = true，因此对应的 `MapBinder` 使用 `MyCustomEnum` 作为 key，而不是 `MyCustomEnumKey `自己。



<br/><br/>

#### <font size=5 color=green><b>@ProvidesIntoOptional</b></font> ####

框架代码可以使用 `@ProvidesIntoOptional(ProvidesIntoOptional.Type.DEFAULT)` 来提供一个默认绑定，而应用程序代码可以使用 `@ProvidesIntoOptional(ProvidesIntoOptional.Type.ACTUAL)` 覆盖默认绑定。

```java
public class FrameworkModule extends AbstractModule {
  @ProvidesIntoOptional(ProvidesIntoOptional.Type.DEFAULT)
  @Singleton
  RequestLogger provideConsoleLogger() {
    return new DefaultRequestLoggerImpl();
  }
}

```

注意，当前 `@ProvidesIntoOptional` 不能用于创建一个不存在的或者空的选项绑定（absent/empty optional binding），而必须使用 `OptionalBinder.newOptionalBinder()` 方法。



<br/><br/>
<a id="105"></a>

## 4.10.5 限制 ##

在使用 `PrivateModule` 的多绑定时，所有的元素必须绑定到同一环境。不能使用跨私有模块内的元素创建集合，否则注入器 injector 创建会失败。



<br/><br/>
<a id="106"></a>

## 4.10.6 观察多绑定内部信息 Inspecting Multibindings ##

有时候，需要观察组成 Multibinder 或 MapBinder 的元素。例如，可能需要测试从一个 MapBinder 中取出一系列的模块。可以通过 [MultibindingsTargetVisitor](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/multibindings/MultibindingsTargetVisitor.html) 类访问一个绑定以获取有关 Multibindings 或 MapBindings 的细节。有了一个 [MapBinderBinding](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/multibindings/MapBinderBinding.html) 或 [MultibinderBinding](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/multibindings/MultibinderBinding.html) 实例之后，就可以探索更多的信息。

```java
   // Find the MapBinderBinding and use it to remove elements within it.
   Module stripMapBindings(Key<?> mapKey, Module... modules) {
     MapBinderBinding<?> mapBinder = findMapBinder(mapKey, modules);
     List<Element> allElements = Lists.newArrayList(Elements.getElements(modules));
     if (mapBinder != null) {
       List<Element> mapElements = getMapElements(mapBinder, modules);
       allElements.removeAll(mapElements);
     }
     return Elements.getModule(allElements);
  }

  // Look through all Elements in the module and, if the key matches,
  // then use our custom MultibindingsTargetVisitor to get the MapBinderBinding
  // for the matching binding.
  MapBinderBinding<?> findMapBinder(Key<?> mapKey, Module... modules) {
    for(Element element : Elements.getElements(modules)) {
      MapBinderBinding<?> binding =
          element.acceptVisitor(new DefaultElementVisitor<MapBinderBinding<?>>() {
            MapBinderBinding<?> visit(Binding<?> binding) {
              if(binding.getKey().equals(mapKey)) {
                return binding.acceptTargetVisitor(new Visitor());
              }
              return null;
            }
          });
      if (binding != null) {
        return binding;
      }
    }
    return null;
  }

  // Get all elements in the module that are within the MapBinderBinding.
  List<Element> getMapElements(MapBinderBinding<?> binding, Module... modules) {
    List<Element> elements = Lists.newArrayList();
    for(Element element : Elements.getElements(modules)) {
      if(binding.containsElement(element)) {
        elements.add(element);
      }
    }
    return elements;
  }

  // A visitor that just returns the MapBinderBinding for the binding.
  class Visitor
      extends DefaultBindingTargetVisitor<Object, MapBinderBinding<?>>
      implements MultibindingsTargetVisitor<Object, MapBinderBinding<?>> {
    MapBinderBinding<?> visit(MapBinderBinding<?> mapBinder) {
      return mapBinder;
    }

    MapBinderBinding<?> visit(MultibinderBinding<?> multibinder) {
      return null;
    }
  }

```





