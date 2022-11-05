## 第4章 BeanLocator ##

执行跨多个 injector 动态 bean 查找。

[BeanLocator](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/inject/BeanLocator.html) 执行查找和对 bean 的实现保持跟踪，是通过从一个或多个 [BindingPublisher](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/inject/BindingPublisher.html) 绑定信息的处理来实现的，例如 injector。

可以通过 [MutableBeanLocator](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/inject/MutableBeanLocator.html) 视图来添加或移除 BindingPublisher。任何存在的监视器或返回的集合都被更新，以反映最新的绑定信息。

[DefaultBeanLocator](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/inject/DefaultBeanLocator.html) 会通过其优美的 `@Inject` 注解的设置器方法，自动添加任何它绑定进去的 injector。这使它通过一个简单的实例绑定就可以轻易地在多个 injector 间共享，定义如下所示：

```java
    /**
     * Automatically publishes any {@link Injector} that contains a binding to this {@link BeanLocator}.
     * 
     * @param injector The injector
     */
    @Inject
    void autoPublish( final Injector injector )
    {
        staticAutoPublish( this, injector );
    }

    @Inject
    static void staticAutoPublish( final MutableBeanLocator locator, final Injector injector )
    {
        locator.add( new InjectorBindings( injector ) );
    }
```

例如下面的代码：

```java
 Module locatorModule = new AbstractModule() {
   private final DefaultBeanLocator locator = new DefaultBeanLocator();
   
   @Override protected void configure() {
     bind( DefaultBeanLocator.class ).toInstance( locator );
   }
 };
 
 Injector injectorA = Guice.createInjector( new WireModule( locatorModule, spaceModuleA ) );   // adds injectorA to locator
 Injector injectorB = Guice.createInjector( new WireModule( locatorModule, spaceModuleB ) );   // adds injectorB to locator
```

如果想要在一个给定的 injector 中使用 DefaultBeanLocator，但不想把这个 injector 自动加入，可以把 locator 封装到一个 provider 中来隐藏从 Guice 注入的设置器：

```java
bind( DefaultBeanLocator.class ).toProvider( Providers.of( locator ) );
```

默认情况下，在一个 injector 中的所有绑定被分为两个部分：默认的和非默认的，并根据它们的序列号（sequence number）排名（ranked）。这样，来自多个 injector 的绑定就可以被交叉存取，以保持默认组件优先在非默认组件之前，而仍能在不同的 injector 之间维护整体的排序。要覆盖默认的排名，可以绑定自定义的 [RankingFunction](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/inject/RankingFunction.html) 排名函数：

```java
bind( RankingFunction.class ).to( MyRankingFunction.class );
```




















