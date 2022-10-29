## 第1章 Plexus 项目 ##


<br/><br/>
<a id="1"></a>

## 1.1 Plexus 项目的由来 ##

Maven 项目刚开始启动的时候，开发团队意识到需要一个组件框架，一种标准化机制来实例化插件，并基于一组配置点对组件进行配置。以下是 maven 开发者 Jason van Zyl 在文章 [From Plexus to Guice (#1): Why Guice?](https://blog.sonatype.com/2010/01/from-plexus-to-guice-1-why-guice/) 中所写的内容：

>We knew we needed some sort of component framework, some standard mechanism to instantiate plugins and configure them based on a set of configuration points, and, at the time, Plexus filled the gap. Plexus was exactly what we needed because it evolved with the requirements of Maven,and I think that Plexus served us well for the past few years but it's time to let go.

Jason van Zyl 是个了不起的开发者，Plexus 容器的默认实现就是由他和 Kenney Westerhof 设计的。

Maven 的第一个版本发布于 2002 年 3 月 30 日。而那时（来自同一篇文章）：

>When we started the Maven project, dependency injection was still developing. Spring was just starting out and the Avalon project at Apache was really the only IoC framework around

也就是说，在开发团队开始启动 Maven 项目的时候，依赖注入思想还在发展中。Spring 刚起步，Avalon 项目，也仅仅只是一个 Ioc 框架。即没有成熟的容器框架可用。因此 Plexus 项目应运而生，并且是按照 Maven 的需求而演变的。

Plexus 项目是 Maven 开发团队早期为 Maven 项目开发的 IOC 容器。

>Plexus: 发音(ˈpleksəs)，a network of nerves or vessels in the body.

意思是身体里的神经或血管网络。

项目官网：[https://codehaus-plexus.github.io](https://codehaus-plexus.github.io)
github: [https://github.com/codehaus-plexus/plexus-containers](https://github.com/codehaus-plexus/plexus-containers)


<br/><br/>
<a id="2"></a>

## 1.2 Plexus 项目概述 ##

Plexus 是一个由 Maven 项目使用的组件集合，其中包括如下子项目：

- [Modello](https://codehaus-plexus.github.io/modello/) 从简单模型生成代码的框架
- [Plexus Classworlds](https://codehaus-plexus.github.io/plexus-classworlds/) 要求对 Java 类加载器进行复杂操作的容器开发者提供的框架
- Plexus Components:
  - [plexus-archiver](https://codehaus-plexus.github.io/plexus-archiver/):操纵归档的组件
  - [plexus-cli](https://codehaus-plexus.github.io/plexus-cli/): 便于创建 CLI 程序的组件
  - [plexus-compiler](https://codehaus-plexus.github.io/plexus-compiler/): 操纵编译器的组件
  - [plexus-digest](https://codehaus-plexus.github.io/plexus-digest/): 
  - [plexus-i18n](https://codehaus-plexus.github.io/plexus-i18n/): 
  - [plexus-interactivity](https://codehaus-plexus.github.io/plexus-interactivity/): 
  - [plexus-interpolation](https://codehaus-plexus.github.io/plexus-interpolation/): 拦截框架
  - [plexus-io](https://codehaus-plexus.github.io/plexus-io/): I/O 操作专用组件
  - [plexus-languages](https://codehaus-plexus.github.io/plexus-languages/): ,
  - [plexus-resources](https://codehaus-plexus.github.io/plexus-resources/): 从文件系统、类路径、或者互联网上透明获取资源的组件
  - [plexus-swizzle](https://codehaus-plexus.github.io/plexus-swizzle/): 从一个问题跟踪系统生成报告的组件 an issue tracking system (JIRA)
  - [plexus-velocity](https://codehaus-plexus.github.io/plexus-velocity/): 渲染 velocity 模板的组件
- [Plexus Utils](https://codehaus-plexus.github.io/plexus-utils/),各种实用工具类集合，便于对字符串、文件、命令行、XML 等进行操作。


Plexus 的核心子项目是 Plexus Container，它是 Plexus' inversion-of-control (IoC) container。即 Plexus 的 IoC 容器。由核心模块 [**plexus-container-default**](http://codehaus-plexus.github.io/plexus-containers/plexus-container-default) 及以下辅助工具模块组成：

- [plexus-component-metadata](http://codehaus-plexus.github.io/plexus-containers/plexus-component-metadata) 是一个 Maven 插件，用于从源 javadoc 标记和 Java 类注解生成 plexus 的 component.xml 文件
- [plexus-component-annotations](http://codehaus-plexus.github.io/plexus-containers/plexus-component-annotations) 为 plexus 提供 Java 注解
- [plexus-component-javadoc](http://codehaus-plexus.github.io/plexus-containers/plexus-component-javadoc) 为 javadoc 提供 taglets，以将 plexus 文档加入到 javadoc


Plexus 项目提供了一个创建和执行软件项目的全软件栈。基于 Plexus 容器，应用程序可以利用面向组件编程，构建易于组合和使用的模块化的、可复用的组件。

尽管 Plexus 与其它 inversion-of-control (IoC) 或 dependency injection 框架类似，例如 Spring Framework，但它是一个完全成熟的容器，支持更多的特性，例如：

- 组件生命周期
- 组件实例化策略
- 嵌入式容器 Nested containers 
- 组件配置
- 自动装配 Auto-wiring
- 组件依赖 Component dependencies
- 各种依赖注入技术，包括构造器注入、setter 注入、私有字段注入。

Plexus 中的组件不必由 Java 编写，现有的组件工厂支持 Jython, JRuby, Beanshell, Groovy 编写的组件。





<br/><br/>
<a id="3"></a>

## 1.3 Plexus 项目现状 ##

Plexus Container 在 Maven 1.x 和 2.x 中工作了很多年，直到 Maven 3 版本，被 Eclipse Sisu 项目所取代，作为 Google Guice 项目的扩展对 Plexus Container 进行了重写。

其它组件仍在 Maven 项目或其子项目中使用。例如 maven 3.8.6 的 boot 目录下有：

- plexus-classworlds-2.6.0.jar

lib 目录下有：

- plexus-cipher-2.0.jar
- plexus-component-annotations-2.1.0.jar
- plexus-interpolation-1.26.jar
- plexus-sec-dispatcher-2.0.jar
- plexus-utils-3.3.1.jar

而 lib 目录下的容器部分由下面两个类库所取代：

- org.eclipse.sisu.inject-0.3.5.jar
- org.eclipse.sisu.plexus-0.3.5.jar

另外，在 Maven 子项目 maven-indexer 中，使用了 org.eclipse.sisu.inject，org.eclipse.sisu.plexus，plexus-cli，plexus-utils 几个 Plexus 相关的模块。




<br/><br/>
<a id="4"></a>

## 1.4 Plexus IOC 容器初体验 ##

这是 Plexus 项目 Quick Start 展示的简单示例。


<br/><br/>

#### <font size=4 color=green><b>创建组件接口</b></font> ####


创建组件的第一个任务是定义组件的角色 role。在 Java 中，通常是定义一个具有向外界暴露组件功能性的接口。

注意，Plexus 不严格要求使用接口来定义组件角色，但这是帮助提升应用程序设计所强烈建议的。

接口定义如下所示:

```java

public interface Cheese
{
    /** The Plexus role identifier. */
    String ROLE = Cheese.class.getName();

    /**
     * Slices the cheese for apportioning onto crackers.
     * @param slices the number of slices
     */
    void slice( int slices );

    /**
     * Get the description of the aroma of the cheese.
     * @return the aroma
     */
    String getAroma();
}
```

接口通过 ROLE 静态字段为角色声明了一个 String 型标识符。这个字段的名字和是一个简单的约定，并且可以为任何其它值，只要保证它在容器中是唯一的就可以，使用包和类名保证了这一点。

接口中的其它方法声明了组件的功能：将 cheese 按指定数量切成片的能力，以及获取 cheese 味道描述的方法。


<br/><br/>

#### <font size=4 color=green><b>创建组件实现</b></font> ####

一旦声明了接口，就需要创建一个或多个接口的功能实现，如下所示：

```java
public class ParmesanCheese implements Cheese
{
    public void slice( int slices )
    {
        throw new UnsupportedOperationException( "No can do" );
    }

    public String getAroma()
    {
        return "strong";
    }
}
```

ParmesanCheese 类实现了 Cheese 接口，提供了接口声明的两个方法：slice() 和 getAroma() 的具体实现，这样就完成了组件的设计工作。


<br/><br/>

#### <font size=4 color=green><b>创建组件描述符</b></font> ####

下一步，是在最终包含该组件的 JAR 包或类加载器 classloader 上创建组件描述符文件：META-INF/plexus/components.xml，开发时可以在项目源码的 resources 目录下创建组件描述符文件：META-INF/plexus/components.xml。


按如下内容配置该文件：

```xml
<component-set>
    <components>
        <component>
            <role>org.codehaus.plexus.examples.tutorial.container.Cheese</role>
            <role-hint>parmesan</role-hint>
            <implementation>org.codehaus.plexus.examples.tutorial.container.ParmesanCheese</implementation>
        </component>
    </components>
</component-set>
```

这个描述符只包含一个组件，其角色 role 声明为 org.codehaus.plexus.examples.tutorial.container.Cheese 接口。描述符通过给定的角色标识 role hint 指明了使用接口的哪个实现。角色标识 role-hint 是强制性的元素，用以区分一个给定组件的不同实现，并用于之后对该组件的引用。

注意，没有必要手动创建这个描述符文件，可以通过 Component Descriptor Creator (CDC) 工具从 Java 类代码创建 components.xml 文件，[Component Descriptor Creator](https://codehaus-plexus.github.io/guides/quick-start/getting-started.html)。

其中，每个组件配置一个 `<component>` 元素，在 `<component>` 内，可以配置多个子元素，下面展示其中重要的三个，更多详细信息，参考第4章 Plexus IOC 容器的组件配置文件：

- **role**: 组件的角色，就是接口定义或基类的全限定类名。
- **role-hint**：组件的角色提示。从含义上理解，有点类似于 id, name 之类的东西，但又没有那么强的用途。在一个接口有多个实现类时，用以区分不同的实现类。可以使用任意字符串值，把同一接口的不同实现类区分开来就可以了。
- **implementation**：组件的实现。设置该接口实现类的完全限定名。普通 Java 组件的 FQCN 字符串，或者某些其它组件工厂实现的名字或文件。


<br/><br/>

#### <font size=4 color=green><b>创建 Plexus 应用程序</b></font> ####

最后一步，使用 Plexus 容器管理组件，如下代码所示：

```java

public class App {
    public static void main(String args[]) throws Exception {
        // 1  定义一个容器，容器会去加载 classpath 下的 META-INF/Plexus/component.xml 中的组件
        PlexusContainer container= new DefaultPlexusContainer();

        // 2 获取组件，完成依赖注入等工作
        Cheese cheese = (Cheese) container.lookup( Cheese.ROLE, "parmesan" );

        // 3 使用组件
        System.out.println( "Parmesan is " + cheese.getAroma() );

        // 4 销毁容器
        container.dispose();
    }
}

```

项目的 pom.xml 文件中需添加如下依赖项：

```xml
    <dependency>
        <groupId>org.codehaus.plexus</groupId>
        <artifactId>plexus-container-default</artifactId>
        <version>2.1.1</version>
    </dependency>
```

运行代码，输出：

```shell
Parmesan is strong
```

通过 PlexusContainer 容器，成功获取到组件实例，并调用了该组件的方法。






