## 第2章 WireModule ##

WireModule 对未解决依赖的可自定义的连接（Customizable wiring of unresolved dependencies）。使用这个类进行跨注入器（injector）共享组件、应用配置、以及形成按需组件集合（form on-demand collections）。这个 Guice 模块解决的是依赖注入的问题。

[WireModule](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/wire/WireModule.html) 类是一个 Guice `Module` 的实现，为未解析的依赖自动加入 [BeanLocator](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/inject/BeanLocator.html) 绑定。`BeanLocator` 接口用于查找和跟踪带有 `@Qualifier` 注解的 bean 实现。默认使用 `DefaultBeanLocator` 实现类。

WireModule 类提供如下构造器：

- **WireModule(Module... modules)** 
- **WireModule(Iterable<Module> modules)** 


**应该把应用程序中所有的组件模块都包装到`WireModule`中**，如：

```java
Guice.createInjector( new WireModule( bootModule, configModule, mainModule ) );
```

要想对子注入器进行依赖的连接，应使用 [ChildWireModule](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/wire/ChildWireModule.html)：

```java
injector.createChildInjector( new ChildWireModule( serviceModule, subModule ) );
```

默认的连接策略（wiring strategy）是使用 [LocatorWiring](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/wire/LocatorWiring.html)，它通过[BeanLocator](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/inject/BeanLocator.html)提供如下绑定：


<br/>

#### <font size=3><b>实例绑定（Instances）</b></font> ####

```java
 @Inject MyType bean
 
 @Inject @Named("hint") MyType namedBean
 
 @Inject @MyQualifier MyType qualifiedBean
 
 @Inject Provider<MyType> beanProvider
```


<br/>

#### <font size=3><b>配置（Configuration）</b></font> ####

```java
 @Inject @Named("${my.property.name}") File file                      // supports basic type conversion
 
 @Inject @Named("${my.property.name:-http://example.org/}") URL url   // can give default in case property is not set
 
 @Inject @Named("${my.property.name:-development}") MyType bean       // can be used to pick specific @Named beans
 
 @Inject @Named("my.property.name") int port                          // shorthand syntax
```

可以按如下方式在运行时绑定配置：

```java
 bind( ParameterKeys.PROPERTIES ).toInstance( myConfiguration );      // multiple bindings are merged into one view
```

<br/>

#### <font size=3><b>集合（Collections）</b></font> ####

下面的集合是动态的并且线程安全，其中的元素在注入器 injector 加入到 `BeanLocator`时进来，在注入器 injector 从 `BeanLocator`移出时，元素也从集合中移出。

其中的元素也是延迟性的，意思是在访问集合元素时才创建实例，这些实例然后在同一集合中重用。

```java
@Inject List<MyType> list
 
 @Inject List<Provider<MyType>> providers
 
 @Inject Iterable<BeanEntry<MyQualifier, MyType>> entries             // gives access to additional metadata

```

```java
 @Inject Map<String, MyType> stringMap                                // strings are taken from @Named values
 
 @Inject Map<Named, MyType> namedMap
 
 @Inject Map<MyQualifier, MyType> qualifiedMap
 
 @Inject Map<String, Provider<MyType>> providerMap

 ```





