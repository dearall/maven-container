## 第3章 SpaceModule ##

SpaceModule 是 bean 实现的可自定义扫描（Customizable scanning of bean implementations）。

`SpaceModule` 类内部提供了三个静态的 `ClassFinder` 接口类型字段：

- public static final **ClassFinder LOCAL_INDEX** = new IndexedClassFinder( NAMED_INDEX, false );
- public static final **ClassFinder GLOBAL_INDEX** = new IndexedClassFinder( NAMED_INDEX, true );
- public static final **ClassFinder LOCAL_SCAN** = SpaceScanner.DEFAULT_FINDER;

其中，`IndexedClassFinder` 和 `SpaceScanner.DEFAULT_FINDER` 都是 `ClassFinder` 接口的实现。
`NAMED_INDEX` 常量字符串是由下面的代码连接而成的：

```java
private static final String NAMED_INDEX = AbstractSisuIndex.INDEX_FOLDER + AbstractSisuIndex.NAMED;

```

其中 `AbstractSisuIndex` 类中定义了相关字符串常量：

```java
abstract class AbstractSisuIndex
{
    static final String INDEX_FOLDER = "META-INF/sisu/";

    static final String QUALIFIER = "javax.inject.Qualifier";

    static final String NAMED = "javax.inject.Named";

    ...
}
```

`ClassFinder` 的功能是从 `ClassSpace` 中查找（可选地过滤）类资源。

`SpaceModule` 提供了三个构造器：

- **SpaceModule(ClassSpace space)** ：
- **SpaceModule(ClassSpace space, ClassFinder finder)** 
- **SpaceModule(ClassSpace space, BeanScanning scanning)** 

其中，`BeanScanning` 是枚举类型，有如下这些枚举值：

<table width="100%">
    <tr>
      <td>BeanScanning </td>
      <td>CACHE </td>
      <td>Scan once and cache results  </td>
    </tr>
    <tr>
      <td>BeanScanning </td>
      <td>GLOBAL_INDEX </td>
      <td>Use global index (application)  </td>
    </tr>
    <tr>
      <td>BeanScanning </td>
      <td>INDEX </td>
      <td>Use local index (plug-ins)  </td>
    </tr>
    <tr>
      <td>BeanScanning </td>
      <td>OFF </td>
      <td>Never scan  </td>
    </tr>
    <tr>
      <td>BeanScanning </td>
      <td>ON </td>
      <td>Always scan </td>
    </tr>
</table>

使用枚举值创建 `SpaceModule` 通过如下逻辑：

```java
    public SpaceModule( final ClassSpace space, final BeanScanning scanning )
    {
        caching = BeanScanning.CACHE == scanning;

        this.space = space;
        switch ( scanning )
        {
            case OFF:
                finder = null;
                break;
            case INDEX:
                finder = LOCAL_INDEX;
                break;
            case GLOBAL_INDEX:
                finder = GLOBAL_INDEX;
                break;
            default:
                finder = LOCAL_SCAN;
                break;
        }
    }
```

使用了上面提到的三个常量 `ClassFinder`，`INDEX` 对应 `LOCAL_INDEX`，`GLOBAL_INDEX` 对应 `GLOBAL_INDEX`，默认对应 `LOCAL_SCAN`，`OFF` 对应的 `finder = null;`

没有 `ClassFinder` 参数的构造器通过如下代码构建：

```java
    public SpaceModule( final ClassSpace space )
    {
        this( space, BeanScanning.ON );
    }
```

其中 `BeanScanning.ON` 枚举值对应到上面构造器逻辑的 `default:` 标签，即 `finder = LOCAL_SCAN;`

第二个构造器的逻辑更加简单，如下所示：

```java
    public SpaceModule( final ClassSpace space, final ClassFinder finder )
    {
        caching = false;

        this.space = space;
        this.finder = finder;
    }
```

直接把构造器参数赋值给实例变量，并设置 `caching = false;`。



[SpaceModule](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/space/SpaceModule.html) 需要给出一个 `ClassSpace` 接口的实例，表示要扫描的类和资源：

```java
Guice.createInjector( new SpaceModule( new URLClassSpace( classloader ) ) );
```

**URLClassSpace** 是 ClassSpace 接口的实现，有一个强引用的（strongly-referenced）ClassLoader 和一个 URL 表示的类路径。

构造器：

- **URLClassSpace(ClassLoader loader)**：创建由一个 ClassLoader 支持的，并使用默认类路径的实例。而系统默认类路径是 "."。
- **URLClassSpace(ClassLoader loader, URL[] path)**： 创建由一个 ClassLoader 支持的，并使用所提供类路径的实例。

loader ：是获取或查找资源时使用的类加载器（class loader）。
path ：查找资源时使用的类路径（class path）。

<br />

可以通过使用一个 [index](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/space/SisuIndex.html) 或提供自己的 [ClassFinder](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/space/ClassFinder.html) 减少扫描时间：

```java
Guice.createInjector( new SpaceModule( new URLClassSpace( classloader ), BeanScanning.INDEX ) );
```

默认的访问策略（visitor strategy）是通过[QualifiedTypeBinder](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/space/QualifiedTypeBinder.html)使用 [QualifiedTypeVisitor](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/space/QualifiedTypeVisitor.html) 查找由 `@Named` 或其它 `@Qualifier` 注解的类型，并按如下方式绑定它们：


<br/>

#### <font size=3><b>组件（Components）</b></font> ####

任何 qualified 的组件通过一个特殊的通配符 key（"wildcard" key）绑定，通配符 key 是 BeanLocator 在查找时用于检查类型兼容性（通配符 key 查找避免了遍历类型层次结构，和为每一个类型以及其超类注册绑定，以及调节 injector 对象图的过程）：

```java
 @Named("example") public class MyTypeImpl implements MyType {
   // ...
 }

```

如果使用一个空的 `@Named` 或一个不同的 `@Qualifier` 注解，则 sisu 会基于实现类型选择一个规范化的名字。

有时候需要为外部集成显式地进行类型化绑定，可以在一个 @Typed 注解中列出类型，或者让它为空以使用声明的接口：

```java
 @Named @Typed public class MyTypeImpl implements MyType {
   // ...
 }
```

默认的实现可以通过使用 "default" 作为一个绑定名指定：

```java
 @Named("default") public class MyTypeImpl implements MyType {
   // ...
 }
```

或者通过实现名以 "Default" 开头：

```java
 @Named public class DefaultMyType implements MyType {
   // ...
 }
```

默认组件被绑定到没有 qualifier 的实现上，并且比非默认的组件有更高的排名（a higher ranking）。


<br/>

#### <font size=3><b>Providers</b></font> ####

任何 qualified providers 使用与组件形同的探索方法绑定：

```java
 @Named public class MyProvider implements Provider<MyType> {
   public MyType get() {
     // ...
   }
 }
```

使用 `@Singleton` 将提供的绑定设定为 singleton scope：

```java
 @Named @Singleton public class MyProvider implements Provider<MyType> {
   public MyType get() {
     // ...
   }
 }
```

注意，这与普通的 Guice 行为有所不同，Guice 的 singleton scope 只应用给 provider 自己。


<br/>

#### <font size=3><b>模块（Modules）</b></font> ####

任何 qualified 的模块通过当前 binder 安装:

```java
 @Named public class MyModule extends AbstractModule {
   @Override protected void configure() {
     // ...
   }
 }

```

<br/>

#### <font size=3><b>调停者（Mediator）</b></font> ####

任何 qualified 的[Mediator](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/Mediator.html)都通过 BeanLocator 注册：

```java
 @Named public class MyMediator implements Mediator<Named, MyType, MyWatcher> {
   public void add( BeanEntry<Named, MyType> entry, MyWatcher watcher ) throws Exception {
     // ...
   }
 
   public void remove( BeanEntry<Named, MyType> entry, MyWatcher watcher ) throws Exception {
     // ...
   }
 }
```

