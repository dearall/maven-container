## 第7章 注入 Provider ##

使用普通的依赖注入，每一个类型精确地获得它所依赖每一个类型的一个实例。`RealBillingService` 获取一个 `CreditCardProcessor` 实例和一个 `TransactionLog` 实例。有时想要获得所依赖类型多于一个的实例，如果需要这种灵活性，Guice 可以绑定一个提供者（provider）。provider 在每次调用 get() 方法时产生一个值。`Provider` 接口定义如下所示：

```java
public interface Provider<T> {
  T get();
}

```

provider 的参数化类型 `T` 将 `Provider<TransactionLog>` 和 `Provider<CreditCardProcessor>` 区分开来。在需要注入一个值的地方，可以为这个值注入一个 provider：

```java
public class RealBillingService implements BillingService {
  private final Provider<CreditCardProcessor> processorProvider;
  private final Provider<TransactionLog> transactionLogProvider;

  @Inject
  public RealBillingService(Provider<CreditCardProcessor> processorProvider,
      Provider<TransactionLog> transactionLogProvider) {
    this.processorProvider = processorProvider;
    this.transactionLogProvider = transactionLogProvider;
  }

  public Receipt chargeOrder(PizzaOrder order, CreditCard creditCard) {
    CreditCardProcessor processor = processorProvider.get();
    TransactionLog transactionLog = transactionLogProvider.get();

    /* use the processor and transaction log here */
  }
}

```

对每一个绑定，加注解的或者没加注解的，injector 为它的 provider 提供一个内置绑定。



<br/><br/>
<a id="1"></a>

## 7.1 多实例提供者 provider ##

在需要同一类型的多个实例时使用 provider，假设应用程序要在一个披萨的收费发生失败时，保存这次失败收费项（entry）的简要信息和详细描述，可以使用 provider，在需要时获取一个新的收费项：

```java
public class LogFileTransactionLog implements TransactionLog {

  private final Provider<LogFileEntry> logFileProvider;

  @Inject
  public LogFileTransactionLog(Provider<LogFileEntry> logFileProvider) {
    this.logFileProvider = logFileProvider;
  }

  public void logChargeResult(ChargeResult result) {
    LogFileEntry summaryEntry = logFileProvider.get();
    summaryEntry.setText("Charge " + (result.wasSuccessful() ? "success" : "failure"));
    summaryEntry.save();

    if (!result.wasSuccessful()) {
      LogFileEntry detailEntry = logFileProvider.get();
      detailEntry.setText("Failure result: " + result);
      detailEntry.save();
    }
  }

```


<br/><br/>
<a id="2"></a>

## 7.2 延迟加载的 provider ##

如果已经获得了某个类型上的一个产生代价特别大的依赖，可以使用 provider 来延迟这个产生的工作。这在不总是需要某个依赖的情况下特别有用：

```java
public class DatabaseTransactionLog implements TransactionLog {

  private final Provider<Connection> connectionProvider;

  @Inject
  public DatabaseTransactionLog(Provider<Connection> connectionProvider) {
    this.connectionProvider = connectionProvider;
  }

  public void logChargeResult(ChargeResult result) {
    /* only write failed charges to the database */
    if (!result.wasSuccessful()) {
      Connection connection = connectionProvider.get();
    }
  }

```

<br/><br/>
<a id="3"></a>

## 7.3 混合 scope 的 provider ##

在应用中使用比较窄小的 scope 直接注入一个对象，通常会导致意想不到的行为。在下面的例子中，假设有一个单例的 `ConsoleTransactionLog` 依赖当前用户的 request 级别的 scope。如果把用户 user 直接注入到 `ConsoleTransactionLog` 构造器，在应用程序的整个生命周期中，user 只会被一次求值。这种行为是不对的，因为用户的每个请求 request 都会发生变化。相反，应该使用 Provider。因为 Provider 是按需产生值的（produce values on-demand），这使得混合使用 scope 是安全的：

```java
@Singleton
public class ConsoleTransactionLog implements TransactionLog {

  private final AtomicInteger failureCount = new AtomicInteger();
  private final Provider<User> userProvider;

  @Inject
  public ConsoleTransactionLog(Provider<User> userProvider) {
    this.userProvider = userProvider;
  }

  public void logConnectException(UnreachableException e) {
    failureCount.incrementAndGet();
    User user = userProvider.get();
    System.out.println("Connection failed for " + user + ": " + e.getMessage());
    System.out.println("Failure count: " + failureCount.incrementAndGet());
  }
  
```















