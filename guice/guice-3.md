## 第3章 Guice 的生命范围 Scope ##

默认情况下，Guice 每次提供值的时候返回一个新的实例。这个行为可以通过 scope 来配置。Scope 允许在一定范围内重用实例：应用程序生命周期（lifetime of an application）、会话生命周期（a session）、或者一个请求（a request）。



<br/><br/>
<a id="1"></a>

## 3.1 Guice 内置的 scope（Built-in scopes） ##


<br/><br/>

#### <font size=5 color=green><b>Singleton</b></font> ####

Guice 自带的内置 @Singleton scope，在应用程序整个生命周期期间，重用一个单独的注入器 injector 内的同一个实例。Guice 同时支持 `javax.inject.Singleton` 和 `com.google.inject.Singleton` 注解，但最好使用 javax.inject.Singleton，因为这样也可以被其它框架支持。

<br/><br/>

#### <font size=5 color=green><b>RequestScoped</b></font> ####

[servlet extension](https://github.com/google/guice/wiki/Servlets) 为 web app 提供了另外的 scope，例如 @RequestScoped。



<br/><br/>
<a id="2"></a>

## 3.2 应用 Scope（Applying Scope） ##

Guice 使用注解标识 scope。为一个类型指定 scope，将 scope 注解应用到实现类上。与函数接口类似，这个注解也作为文档的一部分。例如，下面的 @Singleton 指明该类要求是线程安全的。

```java
@Singleton
public class InMemoryTransactionLog implements TransactionLog {
  /* everything here should be threadsafe! */
}
```

scope 也可以配置在 `bind` 语句中：

```java
bind(TransactionLog.class).to(InMemoryTransactionLog.class).in(Singleton.class);
```

也可以注解到 @Provides 注解的方法上：

```java
  @Provides @Singleton
  TransactionLog provideTransactionLog() {
    ...
  }
```

如果在类型上指定的 scope 和 `bind()` 语句上指定的 scope 发生冲突，则使用 `bind` 语句的 scope。

如果一个类型已经通过 scope 进行了注解，而我们不想要这个注解，可以通过 `bind` 语句把它绑定到 `Scopes.NO_SCOPE`。

在链接的绑定上（In linked bindings），scope 应用到绑定源（binding source）上，而不是绑定的目标上（binding target）。假设有一个 `Applebees` 类，它同时实现了 `Bar` 和 `Grill` 接口。下面的绑定会产生 `Applebees` 的两个实例，一个用于 `Bar` 接口，另一个用于 `Grill` 接口：

```java
  bind(Bar.class).to(Applebees.class).in(Singleton.class);
  bind(Grill.class).to(Applebees.class).in(Singleton.class);
```

这是因为，scope 应用于绑定的类型（`Bar`, `Grill`）上，而不是满足这个绑定的实现上，即 Applebees 类。为了只创建单个实例，可以将 `@Singleton` 注解到那个类的声明上。或者添加另一个绑定：

```java
  bind(Applebees.class).in(Singleton.class);
```

这个绑定使上面的两个 `.in(Singleton.class)` 子句就没必要存在了。

`in()` 子句可以接受一个 scope 注解类型，例如 `RequestScoped.class`，也可以接受一个 `Scope` 类型的实例，如 `ServletScopes.REQUEST`：

```java
  bind(UserPreferences.class)
      .toProvider(UserPreferencesProvider.class)
      .in(ServletScopes.REQUEST);
```

首选使用注解，因为注解可以使模块在不同的应用中重用。例如，一个通过 `@RequestScoped` 注解的对象可以用在一个 web app 中的 HTTP 请求的范围上，也可用于 API 服务器的 RPC 请求上。


<br/><br/>
<a id="3"></a>

## 3.3 渴望型单例 Eager Singleton ##

Guice 通过特殊的语法定义尽早地构建 Singleton 实例：

```java
  bind(TransactionLog.class).to(InMemoryTransactionLog.class).asEagerSingleton();
```

渴望型单例（Eager Singleton）尽早地暴露出该实例的初始化问题，并且保证最终用户得到一致且快捷的体验。延迟型（Lazy singleton）具有更快的编辑-编译-运行（edit-compile-run）开发周期。使用 `Stage` 枚举指定使用哪种类型的策略。

<br />
<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center></th>
        <th align=center>生产场景 PRODUCTION</th>
        <th align=center>开发场景 DEVELOPMENT</th>
    </tr>
    <tr>
      <td>.asEagerSingleton()</td>
      <td>eager</td>
      <td>eager</td>
    </tr>
    <tr>
      <td>.in(Singleton.class)</td>
      <td>eager</td>
      <td>lazy</td>
    </tr>
    <tr>
      <td>.in(Scopes.SINGLETON)</td>
      <td>eager</td>
      <td>lazy</td>
    </tr>
    <tr>
      <td>@Singleton</td>
      <td>eager*</td>
      <td>lazy</td>
    </tr>
</table>

Guice 只对它知道的类型进行渴望构建 singleton。这些类型是模块中提供的类型，加上这些类型的传递性依赖的类型。


<br/><br/>
<a id="4"></a>

## 3.4 scope 的选择 ##

如果对象是有状态的（stateful），很明显需要给对象加 scope，例如对应用程序生命周期的对象（per-application）设置 `@Singleton`，对每个请求内的对象（per-request）设置 `@RequestScoped`，等等。如果对象是无状态的（stateless）并且创建它耗费并不太多（inexpensive），给它指定 scope 就没有必要了。如果绑定时不指定 scope，在每次请求该对象时，Guice 会创建新的实例返回给调用者。

单例模式在 Java 应用程序中流行很广，但它们也没有太大的价值，特别是涉及到依赖注入的时候。虽然单例模式节约了对象的创建，但是单例的初始化要求同步，获取单个初始化的实例只要求读取一个易变的 volatile 型的变量。单例实例在下列情况很有用：

- 有状态对象，例如配置（configuration ）或者计数器（counter）。
- 构建或查找过程资源耗费很大的对象
- 与资源捆绑在一起的对象，例如数据库连接池



<br/><br/>
<a id="5"></a>

## 3.5 scope 与并发性 ##

由 @Singleton 和 @SessionScoped 注解的类必须是**线程安全的（must be threadsafe）**。注入到这种类的所有的对象也必须是线程安全的。[Minimize mutability](https://github.com/google/guice/wiki/MinimizeMutability) 阐述了并发性保护所要求的状态数量限制。


@RequestScoped 注解的对象不需要线程安全。如果对一个注解为 @Singleton 或 @SessionScoped 的对象再依赖一个 @RequestScoped 注解的对象是错误的。


<br/><br/>
<a id="6"></a>

## 3.6 在测试中使用 NO_SCOPE ##

如果测试 Guice 模块使用 scope（特别是使用[自定义 scope](https://github.com/google/guice/wiki/CustomScopes)），但并不真正关心测试中绑定的 scope，可以使用 Guice 的 `Scopes.NO_SCOPE` 覆盖某个 scope。`NO_SCOPE` 是 `Scope` 的实现，每次请求一个对象时返回一个新的实例。

下面是将 `BatchScoped` 绑定到 `NO_SCOPE` 的测试：

```java
import com.google.inject.Scopes;

@RunWith(Junit4.class)
final class FooTest {
  static class TestModule extends AbstractModule {
    @Override
    protected void configure() {
      bindScope(BatchScoped.class, Scopes.NO_SCOPE);
    }
  }
}
```


