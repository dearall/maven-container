## 第2章 Guice 的内部模型 Mental Model ##

学习有关 `key`，`Provider`的概念，以及 Guice 怎么只是一个 map 结构。

在阅读关于 Guice 文章时，会经常看到很多时髦用语："控制反转（Inversion of control）"、"好莱坞原则（Hollywood principle）"、"注入（injection）"，这些听起来令人困惑。但在依赖注入术语里面，这些概念不是很复杂。实际上，我们可能已经编写了非常类似的代码。本文通过一个简化的 Guice 实现模型，以使其更易于理解 Guice 是如何工作的。


<br/><br/>
<a id="1"></a>

## 2.1 Guice 是一个 map ##

从根本上说，Guice 帮助创建和获取对象以为应用来使用。应用程序所需的对象称为**依赖（dependencies）**。

可以把 Guice 理解为一个 map[^guice-map] 结构。应用程序代码声明所需的依赖，而 Guice 为应用从它的 map 中获取依赖对象。"Guice map" 中的每一个条目（entry）由两部分构成：

- **Guice key** ：map 中第一个 key，用于从 map 中获取特定的值。
- **Provider** ：map 中第一个值 value，用于为应用创建对象。

[^guice-map]：Guice 的实际实现要更加复杂得多，map 结构可以大致描述 Guice 的行为。


<br/><br/>

#### <font size=5 color=green><b>Key</b></font> ####

Guice 使用 `com.google.inject.Key<T>` 类来标识一个依赖，它能被使用的 "Guice map" 解析。在上一章中的 Greeter 类，在其构造器中声明了两个依赖，而这两个依赖在 Guice 中被表示为 Key：

-  `@Message String` --> `Key<String>` 
-  `@Count int` --> `Key<Integer>`

一个 Key 最简单的形式表示为 Java 指定的一个类型：

```java
// Identifies a dependency that is an instance of String.
Key<String> databaseKey = Key.get(String.class);
```

然而，应用程序经常有同一类型的多个依赖，例如：

```java
final class MultilingualGreeter {
  private String englishGreeting;
  private String spanishGreeting;

  MultilingualGreeter(String englishGreeting, String spanishGreeting) {
    this.englishGreeting = englishGreeting;
    this.spanishGreeting = spanishGreeting;
  }
}

```

Guice 使用绑定注解（binding annotations）来区分同一类型的不同依赖，以使类型更具体：

```java
final class MultilingualGreeter {
  private String englishGreeting;
  private String spanishGreeting;

  @Inject
  MultilingualGreeter(
      @English String englishGreeting, @Spanish String spanishGreeting) {
    this.englishGreeting = englishGreeting;
    this.spanishGreeting = spanishGreeting;
  }
}
```

带有绑定注解的 Key 可以创建为：

```java
Key<String> englishGreetingKey = Key.get(String.class, English.class);
Key<String> spanishGreetingKey = Key.get(String.class, Spanish.class);
```

当应用程序调用 `injector.getInstance(MultilingualGreeter.class)` 来创建 MultilingualGreeter 的一个实例时，与下面的代码等同：

```java
// Guice internally does this for you so you don't have to wire up those
// dependencies manually.
String english = injector.getInstance(Key.get(String.class, English.class));
String spanish = injector.getInstance(Key.get(String.class, Spanish.class));
MultilingualGreeter greeter = new MultilingualGreeter(english, spanish);
```

总而言之：Guice 的 **Key** 是一个类型，联合一个可选的绑定注解（binding annotation），用于标识依赖。



<br/><br/>

#### <font size=5 color=green><b>Provider</b></font> ####

Guice 使用 `javax.inject.Provider<T>` 接口来表示 "Guice map" 中的对象工厂，具有创建对象的能力以满足依赖的需要。

Provider 接口只有一个方法：

```java
interface Provider<T> {
  /** Provides an instance of T.**/
  T get();
}
```

每个实现 Provider 的类只用很少的代码来提供 T 的实例。可以调用 new T()，或者其它构造 T 实例的方式，或者从一个缓存中返回一个建立好实例。

多数应用程序并不是直接实现 `Provider` 接口，而是使用 `Module` 来配置 Guice 注入器 injector，而 Guice 注入器在内部为所有它知道如何创建的对象创建 `Provider`。

例如，下面的 Guice 模块创建两个 `Provider`：

```java
class DemoModule extends AbstractModule {
  @Provides
  @Count
  static Integer provideCount() {
    return 3;
  }

  @Provides
  @Message
  static String provideMessage() {
    return "hello world";
  }
}
```

- `Provider<String>` 由调用 `provideMessage()` 方法并返回 "hello world" 实现
- `Provider<Integer>` 由调用 `provideCount()` 方法并返回 3 实现





<br/><br/>
<a id="2"></a>

## 2.2 使用 Guice ##

使用 Guice 的过程分为两个部分：

1. **配置 Configuration**：应用程序把必要的内容加入到 "Guice map"
2. **注入 Injection**：应用程序向 Guice 请求从 map 中创建并获取所需的对象


<br/><br/>

#### <font size=5 color=green><b>配置 Configuration</b></font> ####

Guice map 通过 Guice 的模块以及 Just-In-Time bindings 来配置。Guice 模块是向 Guice map 中添加内容的配置逻辑单元。可以通过两种途径进行配置：

- 添加方法注解，例如 `@Provides`
- 使用 Guice 的 DSL（Domain Specific Language）

从概念上讲，这些 API 简单地提供操纵 Guice map 的途径。它们所做的操纵相当直接。下面是一些转换示例，为了简洁和清晰，使用了 Java 8 语法：

<br />
<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center>Guice DSL 语法</th>
        <th align=center>内部模型 Mental model</th>
        <th align=center>说明</th>
    </tr>
    <tr>
      <td>bind(key).toInstance(value)</td>
      <td>map.put(key, () -> value)</td>
      <td>实例绑定（instance binding）</td>
    </tr>
    <tr>
      <td>bind(key).toProvider(provider)</td>
      <td>map.put(key, provider)</td>
      <td>provider 绑定（provider binding）</td>
    </tr>
    <tr>
      <td>bind(key).to(anotherKey)</td>
      <td>map.put(key, map.get(anotherKey))</td>
      <td>链接绑定(linked binding)</td>
    </tr>
    <tr>
      <td>@Provides Foo provideFoo() {...}</td>
      <td>map.put(Key.get(Foo.class), module::provideFoo)</td>
      <td>provider 方法绑定(provider method binding)</td>
    </tr>
</table>


`DemoModule` 向 Guice map 添加两个条目：

- `@Message String` --> `() -> DemoModule.provideMessage()` 
- `@Count Integer` --> `() -> DemoModule.provideCount()`







<br/><br/>

#### <font size=5 color=green><b>注入 Injection</b></font> ####

不是从 map 中拉出什么东西，而是声明（declare）需要它们，这是依赖注入的本质。如果需要什么东西，不要到别处获取它，或者请求某个类来返回想要的东西。而是简单的声明没有它就无法工作，并依赖 Guice 提供想要的对象。

这种模型来自于大多数人们关于代码的思考：更具声明性的模型而非命令式的模型。这也就是为什么依赖注入经常被描述为一类控制反转（inversion of control, IoC）。


有几种方法可以声明需要某些东西：

1. 由 @Inject 注解的构造器参数：

```java
class Foo {
  private Database database;

  @Inject
  Foo(Database database) {  // We need a database, from somewhere
    this.database = database;
  }
}
```

2. 由 @Provides 注解的方法参数：

```java
@Provides
Database provideDatabase(
    // We need the @DatabasePath String before we can construct a Database
    @DatabasePath String databasePath) {
  return new Database(databasePath);
}
```





<br/><br/>
<a id="3"></a>

## 2.3 依赖构成一个有向图 Dependencies form a graph ##

当注入某个东西，它自己也有依赖，Guice 递归注入这些依赖。想象一下，为了注入一个 Foo 类的实例，Guice 创建 Provider 实现，与如下类似：

```java
class FooProvider implements Provider<Foo> {
  @Override
  public Foo get() {
    Provider<Database> databaseProvider = guiceMap.get(Key.get(Database.class));
    Database database = databaseProvider.get();
    return new Foo(database);
  }
}

class ProvideDatabaseProvider implements Provider<Database> {
  @Override
  public Database get() {
    Provider<String> databasePathProvider =
        guiceMap.get(Key.get(String.class, DatabasePath.class));
    String databasePath = databasePathProvider.get();
    return module.provideDatabase(databasePath);
  }
}

```

依赖构成了一个有向图（directed graph），而注入 injection 工作是，从想要的对象开始做深度优先（depth-first）遍历这个图，直到找到它的全部依赖。

Guice 的 Injector 对象表示整个依赖图。为了创建一个 Injector 对象，Guice 需要验证整个依赖图工作正常，其中不能有任何悬而未决的节点（node），需要某个依赖却没有提供依赖。不管什么原因，如果依赖图无效，Guice 会抛出 `CreationException` 异常，并描述出错的原因。

相反的情况是没有任何错误：提供了某些东西，即使没有使用它，依赖图也会表现工作正常。只是一段无用的代码，如果没有任何代码还再使用这些依赖，最好删除这些无用的 provider。


