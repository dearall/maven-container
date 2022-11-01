## 第1章 Google Guice 入门 Getting Started ##



<br/><br/>
<a id="1"></a>

## 1.1 Google Guice 概述 Overview ##


Guice 是 Google 开发的一个轻量级的，基于 Java5 泛型和注解特性的依赖注入（dependency injection，DI ）框架。Guice 非常小巧且运行速度快，性能和运行效率优于 Spring，有人说 guice 比 spring 快十倍左右。使用 spring 很容易写成 service locator 的风格，而用 guice，会很自然的形成 DI 风格。

&emsp;&emsp;Guice 是类型安全的，它能够对构造函数，属性，方法（包含任意个参数的任意方法，而不仅仅是 setter 方法）进行注入。Java 5 引入泛型和注解特性之后，Guice 采用泛型加注解的方式对托管对象进行配置。

&emsp;&emsp;Guice 提供模块对应的抽象 `Module` 接口，将类与类之间的关系隔离到 `Module` 中，使得架构和设计的模块概念产物与代码中的 module 类一一对应，更加便利的组织和梳理模块依赖关系，利于整体应用内部的依赖关系维护，而其他 IOC 框架是没有对应物的。此外，借助 privateModule 的功能，可以实现模块接口的明确导出和实现封装，使得支持多数据源这类需求实现起来异常简单。

&emsp;&emsp;简单来说，Guice 减轻 Java 代码中使用对象工厂和 new 关键字创建对象。可以把 Guice 的 @Inject 注解视为 Java 原生语法的 new 关键字。在某些情况下可能仍需要编写对象工厂代码，但应用代码不直接依赖它们，这使得应用代码更容易修改、单元测试、和在其他环境中的重用。

Guice 可以帮助开发者设计更好的 API，并且 Guice API 本身也是非常好的 API 设计案例。Guice 只构建通用的功能，而不是把所有的特性都加入到核心框架中，如果有需要，可以自行扩展。


<br/><br/>
<a id="2"></a>

## 1.2 什么是依赖注入 What is dependency injection ##

依赖注入（Dependency injection, DI）是一种设计模式（design pattern ），在其中，类把它们的依赖声明为参数，而不是自己直接创建这些依赖。例如，一个客户端 client 希望调用一个服务 service，客户端不必知道如何创建这个服务，而是由某些外部代码负责提供服务 service 给客户端 client。

下面是一个简单的没有使用依赖注入的示例代码：

```java
class Foo {
  private Database database;  // We need a Database to do some work

  Foo() {
    // Ugh. How could I test this? What if I ever want to use a different
    // database in another application?
    this.database = new Database("/path/to/my/data");
  }
}
```

上面的类 Foo 在内部直接创建了一个固定的 Database 对象。这种方式阻止了 Foo 类使用其它的 Database 对象，并且也不允许在测试中使用一个测试数据库替换掉这个真实的数据库。可以利用依赖注入模式来解决这些问题，而不是编写这种不可测试或者说不灵活的代码。

下面是同一个例子，这一次，使用依赖注入模式：

```java
class Foo {
  private Database database;  // We need a Database to do some work

  // The database comes from somewhere else. Where? That's not my job, that's
  // the job of whoever constructs me: they can choose which database to use.
  Foo(Database database) {
    this.database = database;
  }
}

```

上面的 Foo 类可以使用任何 Database 对象，因为 Foo 类对如何创建 Database 对象毫不知情。例如，可以在测试环境中创建一个测试版本的 Database 实现，使用内存数据库来使测试不受外界环境影响，并且运行速度很快。



<br/><br/>
<a id="3"></a>

## 1.3 Guice 核心概念 Core Guice concepts ##


<br/><br/>

#### <font size=5 color=green><b>@Inject constructor</b></font> ####

由 @Inject 注解的类构造器，能被 Guice 通过一个被称为 **构造器注入（constructor injection）** 的过程调用，期间，构造器的参数会由 Guice 创建并提供。

下面是一个使用构造器注入的例子：

```java
class Greeter {
  private final String message;
  private final int count;

  // Greeter declares that it needs a string message and an integer
  // representing the number of time the message to be printed.
  // The @Inject annotation marks this constructor as eligible to be used by
  // Guice.
  @Inject
  Greeter(@Message String message, @Count int count) {
    this.message = message;
    this.count = count;
  }

  void sayHello() {
    for (int i=0; i < count; i++) {
      System.out.println(message);
    }
  }
}

```

在上面的例子中，Greeter 类有一个构造器，会在应用请求 Guice 创建 Greeter 实例时被调用。Guice 会创建这两个要求的参数，然后调用构造器。Greeter 类构造器的两个参数就是它的依赖（dependencies），并且应用使用 Module 来告知 Guice 如何满足这些依赖。


<br/><br/>

#### <font size=5 color=green><b>Guice 模块 </b></font> ####

应用程序包含对象，对象通过声明依赖其它对象，而这些依赖构成了图。例如上面的例子，Greeter 类有两个依赖，声明在它的构造器上：

- 一个 String 对象用于打印的消息
- 一个 Integer 对象用于打印消息的次数

Guice 的模块允许应用指定如何满足这些依赖。例如，下面的 DemoModule 为 Greeter 类配置了全部必要的依赖：

```java
/**
 * Guice module that provides bindings for message and count used in
 * {@link Greeter}.
 */
import com.google.inject.Provides;

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

DemoModule 类使用 @Provides 对方法加注解的方式指定依赖。

在实际的应用程序中，对象的依赖图会更加复杂，而 Guice 通过自动创建所有传递性依赖，来使创建复杂对象变得容易。


<br/><br/>

#### <font size=5 color=green><b>Guice 的注入器 injectors </b></font> ####

为了引导启动应用程序，需要利用一个或多个模块创建一个 Guice 注入器。例如，一个 web 服务器应用应用有一个类似如下的 `main` 方法：

```java
public final class MyWebServer {
  public void start() {
    ...
  }

  public static void main(String[] args) {
    // Creates an injector that has all the necessary dependencies needed to
    // build a functional server.
    Injector injector = Guice.createInjector(
        new RequestLoggingModule(),
        new RequestHandlerModule(),
        new AuthenticationModule(),
        new DatabaseModule(),
        ...);
    // Bootstrap the application by creating an instance of the server then
    // start the server to handle incoming requests.
    injector.getInstance(MyWebServer.class)
        .start();
  }
}
```

注入器 Injector 在内部持有应用程序的依赖图。在向 Guice 请求一个给定类型的实例时，Injector 找出要构建的对象，解析它们的依赖，并把所有的整合在一起。要指定如何解析依赖，可以通过绑定（bindings）来配置注入器。


<br/><br/>

#### <font size=5 color=green><b>简单的 Guice 应用 </b></font> ####

下面简单的 Guice 应用将所有必要的部分整合在一起：

```java
package guicedemo;

import static java.lang.annotation.RetentionPolicy.RUNTIME;

import com.google.inject.AbstractModule;
import com.google.inject.Guice;
import com.google.inject.Injector;
import com.google.inject.Key;
import com.google.inject.Provides;
import java.lang.annotation.Retention;
import javax.inject.Inject;
import javax.inject.Qualifier;

public class GuiceDemo {
  @Qualifier
  @Retention(RUNTIME)
  @interface Message {}

  @Qualifier
  @Retention(RUNTIME)
  @interface Count {}

  /**
   * Guice module that provides bindings for message and count used in
   * {@link Greeter}.
   */
  static class DemoModule extends AbstractModule {
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

  static class Greeter {
    private final String message;
    private final int count;

    // Greeter declares that it needs a string message and an integer
    // representing the number of time the message to be printed.
    // The @Inject annotation marks this constructor as eligible to be used by
    // Guice.
    @Inject
    Greeter(@Message String message, @Count int count) {
      this.message = message;
      this.count = count;
    }

    void sayHello() {
      for (int i=0; i < count; i++) {
        System.out.println(message);
      }
    }
  }

  public static void main(String[] args) {
    /*
     * Guice.createInjector() takes one or more modules, and returns a new Injector
     * instance. Most applications will call this method exactly once, in their
     * main() method.
     */
    Injector injector = Guice.createInjector(new DemoModule());

    /*
     * Now that we've got the injector, we can build objects.
     */
    Greeter greeter = injector.getInstance(Greeter.class);

    // Prints "hello world" 3 times to the console.
    greeter.sayHello();
  }
}
```

GuiceDemo 应用通过 Guice 构造了一个小型的依赖图，具有构建 Greeter 类实例的能力。大小应用通常具有很多的 Module 可以构建复杂的对象。








