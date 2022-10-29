## 第1章 org.eclipse.sisu.plexus 项目 ##

sisu.plexus 是在 sisu.inject 和 Google 的 guice 基础上对 Plexus container 容器的重新实现。并且直接对原始的 Plexus 容器接口进行重新定义。因为 sisu.inject 项目的原因，基于 sisu.inject 重新实现的容器具有类路径扫描（classpath scanning）、自动绑定（auto-binding）、以及动态自动装配（dynamic auto-wiring）特性。因此传统的使用 Plexus container 容器的项目可以像之前一样使用，无需改变任何代码，如下所示：

```java

ContainerConfiguration config = new DefaultContainerConfiguration();
// ... configure ...
PlexusContainer container = null;
try {
  container = new DefaultPlexusContainer( config );
  // ... execute/wait ...
} finally {
  if ( container != null ) {
    container.dispose();
  }
}

```

sisu 应用想要重用 plexus 容器管理的组件，可以使用 PlexusSpaceModule 对其进行封装，如下所示：

```java
binder.install( new PlexusSpaceModule( space ) );
```

sisu plexus 的实现，都在 org.eclipse.sisu.plexus 中。




<br/><br/>
<a id="1"></a>

## 1.1 sisu.plexus IOC 容器初体验 ##

把之前使用 Plexus IOC 容器测试的例子，原封不动地复制一份，使用 sisu.plexus 类库初步体验一下：


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

与使用传统 plexus 容器的 plexus-container-default 类库不同，项目的 pom.xml 文件中需使用如下依赖项替换掉对 plexus-container-default 的依赖：

```xml
    <dependency>
        <groupId>org.eclipse.sisu</groupId>
        <artifactId>org.eclipse.sisu.plexus</artifactId>
        <version>0.3.5</version>
    </dependency>
    <dependency>
        <groupId>com.google.inject</groupId>
        <artifactId>guice</artifactId>
        <version>5.1.0</version>
    </dependency>
```

运行代码，程序成功运行，并正确输出如下内容：

```shell
Parmesan is strong
```

使用 sisu.plexus 实现的 PlexusContainer 容器，成功获取到组件实例，并调用了该组件的方法。除了 pom.xml 中依赖的类库，由传统的 plexus-container-default 被替换成 org.eclipse.sisu.plexus 和 guice 之外，源代码没有做任何改变，这就是 sisu.plexus 容器的能力。而 sisu.plexus 容器的能力不仅仅如此，它还有比传统 plexus 容器更优秀的特性，下一篇文章继续探讨。




























