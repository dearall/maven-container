## 第6章 注入 Injections ##

Guice 怎样初始化对象。

依赖注入模式（The dependency injection pattern）将行为从依赖解析中分离开来。不是直接查找依赖或从工厂获取依赖，这个模式建议依赖被传递进去。把依赖设置到对象中的过程称为**注入（injection）**。

<br/><br/>
<a id="1"></a>

## 6.1 构造器注入 Constructor Injection ##

构造器注入将实例化与注入联合起来。要使用构造器注入，使用 `@Inject` 注解构造器。这个构造器会以参数的形式接受类的依赖（class dependencies）。大多数构造器会将参数赋值给 final 修饰的字段。如下所示：

```java
public class RealBillingService implements BillingService {
  private final CreditCardProcessor processorProvider;
  private final TransactionLog transactionLogProvider;

  @Inject
  RealBillingService(CreditCardProcessor processorProvider,
      TransactionLog transactionLogProvider) {
    this.processorProvider = processorProvider;
    this.transactionLogProvider = transactionLogProvider;
  }

```

如果没有 `@Inject` 注解的构造器，如果存在的话，Guice 会使用一个 public 修饰的，无参数构造器。推荐使用注解方式，这样可以文档的方式表明这个类参与到依赖注入中。

构造器注入在单元测试中工作得很好。如果某个类在一个构造器上接受了它所有的依赖，就不会偶尔忘记给它设置某个依赖。当引入一个新的依赖时，所有的调用代码都会很便利地中断（break），修复编译错误然后可以确信所有的东西都正确连接了（properly wired up）。


<br/><br/>
<a id="2"></a>

## 6.2 方法注入 Method Injection ##

**警告**：方法注入暗示着被注入的类是易变的，通常情况下应该避免使用，使用构造器注入方式比方法注入更好。

Guice 可以注入带有 `@Inject` 注解的方法。依赖以方法参数的形式注入，在调用该方法之前，injector 解决这些参数。被注入的方法可以由任何数量的参数，并且方法名不影响注入。如下所示：

```java
public class PayPalCreditCardProcessor implements CreditCardProcessor {

  private static final String DEFAULT_API_KEY = "development-use-only";

  private String apiKey = DEFAULT_API_KEY;

  @Inject
  public void setApiKey(@Named("PayPal API key") String apiKey) {
    this.apiKey = apiKey;
  }

```

<br/><br/>
<a id="3"></a>

## 6.3 字段注入 Field Injection ##

**警告**：与方法注入相同，字段注入隐含着被注入的类是易变的，通常情况下应该避免使用，使用构造器注入方式比字段注入更好。

Guice 通过 `@Inject` 注解注入字段。这是最简洁的注入，但是有最差的可测试性。如下代码所示：

```java
public class DatabaseTransactionLogProvider implements Provider<TransactionLog> {
  @Inject Connection connection;

  public TransactionLog get() {
    return new DatabaseTransactionLog(connection);
  }
}

```

避免对 `final` 修饰的字段使用字段注入，这会造成[弱语义（weak semantics）](https://docs.oracle.com/javase/6/docs/api/java/lang/reflect/Field.html#set(java.lang.Object,%20java.lang.Object))。


<br/><br/>
<a id="4"></a>

## 6.4 可选注入 Optional Injections ##

**提示**：优先选择使用 [OptionalBinder](guice/guice-4.md#103) 而非 `@Inject(optional = true)`。

有时候，在依赖存在时使用这个依赖，而在它不存在时回退到使用默认值是很方便的。方法注入和字段注入可以为可选的，这会使 Guice 在依赖不可用时简单地忽略它们。使用可选注入，应用 `@Inject(optional=true)` 注解进行注入：

```java
public class PayPalCreditCardProcessor implements CreditCardProcessor {
  private static final String SANDBOX_API_KEY = "development-use-only";

  private String apiKey = SANDBOX_API_KEY;

  @Inject(optional=true)
  public void setApiKey(@Named("PayPal API key") String apiKey) {
    this.apiKey = apiKey;
  }

```

混合可选注入和 just-in-time 绑定可能产生令人吃惊的结果。例如，下面的字段总是会被注入，即使 Date 没有被显式绑定。这是因为 Date 有一个 public 无参数构造器，这适合于 just-in-time 绑定。

```java
  @Inject(optional=true) Date launchDate;
```


<br/><br/>
<a id="5"></a>

## 6.5 按需注入 On-demand Injection ##

方法注入和字段注入可以被用于初始化一个已存在的实例。可以使用 `Injector.injectMembers()` API：

```java
  public static void main(String[] args) {
    Injector injector = Guice.createInjector(...);

    CreditCardProcessor creditCardProcessor = new PayPalCreditCardProcessor();
    injector.injectMembers(creditCardProcessor);
```


<br/><br/>
<a id="6"></a>

## 6.6 静态注入 Static Injections ##

在将一个应用从静态工厂迁移到 Guice 时，很可能一步步改变，这里静态注入就很有帮助。它使对象部分参与到依赖注入称为可能，通过对被注入的类型获得访问而它们自己无需被注入。在一个模块内部使用 `requestStaticInjection()` 方法指定在 injector 创建时被注入的类。

```java
  @Override public void configure() {
    requestStaticInjection(ProcessorFactory.class);
    ...
  }
```

Guice 会注入类的带有 `@Inject` 注解的静态成员：

```java
class ProcessorFactory {
  @Inject static Provider<Processor> processorProvider;

  /**
   * @deprecated prefer to inject your processor instead.
   */
  @Deprecated
  public static Processor getInstance() {
    return processorProvider.get();
  }
}

```

静态成员不会在实例注入时被注入。这个 API 在一般情况下不建议使用，因为它遭受很多和静态工厂同样的问题：不方便的测试，使依赖不透明，以及受全局状态控制。



<br/><br/>
<a id="7"></a>

## 6.7 自动注入 Automatic  Injections ##

Guice 在下列对象类型上自动执行字段和方法注入：

- 在一个绑定语句上传递给 `toInstance​(T instance)` 方法的实例
- 在一个绑定语句上传递给 `toProvider()` 方法的 provider 实例

这些注入被作为 injector 创建的一部分执行。


<br/><br/>
<a id="8"></a>

## 6.8 注入点 Injection Points ##

一个**注入点（injection point）** 是代码中的一个位置，其中 Guice 已被请求注入一个依赖。

注入点例子：

- 可注入构造器的参数（[parameters of an injectable constructor](guice/guice-4.md#9)）
- @Provides 注解方法的参数（[parameters of a @Provides method](guice/guice-4.md#4)）
- @Inject 注解的方法（[parameters of an @Inject annotated method](guice/guice-6.md#2)）
- @Inject 注解的字段（[fields annotated with @Inject](guice/guice-6.md#3)）