## 第5章 限制绑定源 Restricting the Binding Source ##

如果自己有一个提供（provide）了一组绑定的 Guice 类库，想要防止类库之外的代码提供（provide）这些绑定，原因包括：

- 防止用户将他们自己仿制绑定（fake）实现带入到测试中，强制它们使用类库维护的权威仿制（fake）模块进行测试。
- 让另外的一组绑定实现出现在自己的代码基线上，使得客户端代码更难以重构。
- 为下游类库提供一个保证的约定。
  
`@RestrictedBindingSource` 注解可以完成这项任务。它限制哪些模块可以提供我们的绑定。使用 `@RestrictedBindingSource` 注解我们的绑定，让我们指定**许可的注解（permit annotation）**，模块需要通过它们进行注解。例如，下面的示例展示如何确保 `@IpAddress Integer` 和 `RoutingTable` 绑定只能通过 `NetworkModule` 提供：

```java
// Modules annotated with this Permit can provide Network bindings.
@RestrictedBindingSource.Permit
@Retention(RetentionPolicy.RUNTIME)
@interface NetworkPermit {}

// Bindings with the @IpAddress qualifier annotation can only be provided by
// modules with the NetworkPermit annotation.
@RestrictedBindingSource(
  explanation = "Please install NetworkModule instead of binding network bindings yourself.",
  permits = {NetworkPermit.class})
@Qualifier
@Retention(RetentionPolicy.RUNTIME)
public @interface IpAddress {}

// The RoutingTable binding can only be provided by modules annotated with the
// NetworkPermit annotation.
@RestrictedBindingSource(
  explanation = "Please install NetworkModule instead of binding network bindings yourself.",
  permits = {NetworkPermit.class})
public interface RoutingTable {
  int getNextHopIpAddress(int destinationIpAddress);
}

@NetworkPermit
public final class NetworkModule extends AbstractModule {
  @Provides @IpAddress int provideIp( ... ) { ... }

  @Override
  protected void configure() {
    // RoutingModule is permitted to provide the RoutingTable binding because
    // it is installed by NetworkModule, which is annotated with NetworkPermit
    // - ie. it's enough for any module providing a binding (directly or
    // indirectly) to have the right permit.
    install(new RoutingModule());
  }
}

private final RoutingModule extends AbstractModule {
  @Provides RoutingTable provideRoutingTable( ... ) { ... }
}

```

如果违反了这个限制，`explanation` 应包含错误消息，指示用户安装正确的模块来提供被限制的绑定。如果一个被限制的绑定不存在，解释内容也包含在错误消息中。

<br/><br/>

#### <font size=5 color=green><b>ModuleAnnotatedMethodScanner as a Source of Bindings</b></font> ####

除了模块，`ModuleAnnotatedMethodScanner` 也是一个 source 绑定，并可以被注解为许可，以授权它们许可提供被限制的绑定。

例如，如果一个 scanner 扫描 @FooProvides 方法，并把这些方法提供的类型绑定到 `@Foo Type`，则 `@Foo` 能被这样限制只有在扫描器能提供被 @Foo 注解的绑定。
































