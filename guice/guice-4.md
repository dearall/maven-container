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


