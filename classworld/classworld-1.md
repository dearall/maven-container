## 第1章 Plexus Classworld 项目 ##

Plexus Classworlds 是一个为需要对 Java 的类加载器（ClassLoader）进行复杂操作的容器开发者提供的框架。Java 自带的类加载器机制和类，对某些类型的应用开发者来说，可能带来麻烦和混乱。动态加载组件或其它表现为 “容器（container）” 的项目，可以从 Classworlds 提供的类加载控制获得益处。

Plexus Classworlds 为类加载提供了一组比 Java 普通机制更丰富的语义，而它仍能提供 ClassLoader 接口，以使它可以与 Java 环境无缝集成。

Classworlds 模型摒弃了与 ClassLoader 关联的分层结构，相反，ClassWorld 类提供了一个 ClassRealm 池，可以从其他 ClassRealm 导入任意的包。实际上，ClassLoader 是将旧风格的分层结构转换为一个有向图（directed graph）。

Realm 中文的意思是 “领域，范围，王国，地带”。这里可以理解为类加载的一个具体模块，比如一个 jar 包可以作为一个 Realm，一个 URL 也可以指向一个 Realm。ClassRealm 就是以 jar 包或者 URL 为划分的类加载器，归根结底它是一个经过优化的类加载器。

在一个应用程序容器环境，容器可以有一个加载容器或组件约定接口和类的领域（realm）能力。从容器的领域 realm，为每一个导入约定类的组件创建另外的领域 realm。

这个模型允许对使用哪个类加载器导入某个特定的类进行精细地控制。这种部分隔离的形式，能降低大量由多个加载器加载类产生的怪异错误。

另外，Plexus Classworlds 还提供一个 launcher，从一个配置文件创建 ClassWorld 和类加载器 ClassRealm，以正确的加载器加载正确的类，启动应用程序的 main 方法。



<br/><br/>
<a id="1"></a>

## 1.1 Classworld API 详解 ##

plexus-classworlds 是一个很小的项目模块，其中包含的主要的 API 类如下：

- **ClassRealm**：继承自 java.net.URLClassLoader，是 Classworld 定义的类加载器，是类的加载工具（gateway）。
- **Strategy**：控制类加载器的查询顺序。
- **ClassWorld**：是 ClassRealm 的集合，内部维护一个 `Map<String, ClassRealm>`，其中 key 为 String 型的 realmId，value 是 ClassRealm 实例。
- **ClassWorldListener**：定义 ClassWorld 创建和销毁 ClassRealm 时的监听器。
- **Launcher**：从配置文件创建 ClassWorld 和类加载器 ClassRealm，以正确的加载器加载正确的类，启动应用程序 main 方法。


<br/><br/>
<a id="11"></a>

## 1.1.1 ClassRealm 类 ##


ClassRealm 继承自 java.net.URLClassLoader 类，是 Classworld 定义的类加载器，是类的加载工具（gateway）。每一个 ClassRealm 类加载器都可以访问基本类加载器（base class loader），导入形式的 0 个或多个类加载器，可选的上级类加载器（parent class loader），以及自己的类路径。在查找一个类或者资源时，ClassRealm 总是先从基本类加载器开始查找，如果没有找到，则委托给其内部的策略 Strategy 实例按既定的顺序查找，使用 Strategy 查找类或资源的顺序，是 ClassRealm 与 Java 原生类加载器根本性区别所在。ClassRealm 默认使用 SelfFirstStrategy 策略定义的顺序查找类或资源。它会按照如下的类加载顺序对一个类进行加载：从外部导入查找类加载器加载、自身加载器中查找已加载的类或通过全路径名加载、从上级类加载器（parent class loader）加载。按照这样顺序查找，一旦找到并加载了该类，就立即停止，不会继续后面的查找。基本类加载器（base class loader）假定其具有加载引导类（bootstrap class）的能力。如下代码所示：

```java

public Class<?> loadClass( String name )
    throws ClassNotFoundException
{
    Class<?> clazz = realm.loadClassFromImport( name );

    if ( clazz == null )
    {
        clazz = realm.loadClassFromSelf( name );

        if ( clazz == null )
        {
            clazz = realm.loadClassFromParent( name );

            if ( clazz == null )
            {
                throw new ClassNotFoundException( name );
            }
        }
    }

    return clazz;
}
```

加载资源时，按如下顺序查找：

```java
public URL getResource( String name )
{
    URL resource = realm.loadResourceFromImport( name );

    if ( resource == null )
    {
        resource = realm.loadResourceFromSelf( name );

        if ( resource == null )
        {
            resource = realm.loadResourceFromParent( name );
        }
    }

    return resource;
}
```

ClassRealm 类代码实现：

```java
public class ClassRealm
    extends URLClassLoader
{
    // 内部有对外部 ClassWorld 容器的引用
    private ClassWorld world;

    // 构造器传递进来的一个 ClassRealm 标识，与 ClassWorld 中 `Map<String, ClassRealm>` 的 key 一致
    private String id;

    /**
    Entry 是 org.codehaus.plexus.classworlds.realm 包一个具体的用于存储类加载器 ClassLoader 实例和对应包名的实体类，并实现 Comparable<Entry> 接口，便于在 SortedSet<Entry> 中排序。
    */

    // 外部导入包的类加载器集合
    private SortedSet<Entry> foreignImports;

    // 上级类加载器导入包集合
    private SortedSet<Entry> parentImports;

    // 类加载顺序策略，默认为 SelfFirstStrategy
    private Strategy strategy;

    // 上一级类加载器（parent class loader），可能为 null
    private ClassLoader parentClassLoader;

    private static final boolean isParallelCapable = Closeable.class.isAssignableFrom( URLClassLoader.class );

    private final ConcurrentMap<String, Object> lockMap;

    /**
     * Creates a new class realm.
     *
     * @param world           The class world this realm belongs to, must not be <code>null</code>.
     * @param id              The identifier for this realm, must not be <code>null</code>.
     * @param baseClassLoader The base class loader for this realm, may be <code>null</code> to use the bootstrap class
     *                        loader.
     */
    public ClassRealm( ClassWorld world, String id, ClassLoader baseClassLoader )
    {
        super( new URL[0], baseClassLoader );

        this.world = world;

        this.id = id;

        foreignImports = new TreeSet<Entry>();

        strategy = StrategyFactory.getStrategy( this );

        lockMap = isParallelCapable ? new ConcurrentHashMap<String, Object>() : null;

        if ( isParallelCapable ) {
        	// We must call super.getClassLoadingLock at least once
        	// to avoid NPE in super.loadClass.
        	super.getClassLoadingLock(getClass().getName());
        }
    }

    ...

}
```

<br/><br/>

#### <font size=4 color=green><b>ClassRealm 构造器</b></font> ####

ClassRealm 提供如下构造器：

- **ClassRealm(ClassWorld world, String id, ClassLoader baseClassLoader)**：创建新的 ClassRealm 实例<br />
  *ClassWorld world* ：类加载器集合 ClassWorld 对象，一定不能为 null 值。<br />
  *String id* ：这个类加载器 realm 的标识符，一定不能为 null 值。<br />
  *ClassLoader baseClassLoader* ：基础类加载器，如果为 null 值，则使用引导类加载器（the bootstrap class loader）作为基本类加载器。


<br/><br/>

#### <font size=4 color=green><b>ClassRealm 主要方法阐释</b></font> ####

&emsp;**Class\<?\> loadClass( String name )** 和 **loadClass( String name, boolean resolve )** ：ClassRealm 直接重写了这两个方法，委托给 Strategy 按既定顺序加载类，从根本上改变了 Java 原生类加载器的分层结构，它是加载类的核心方法。

&emsp;**getResource( String name )** 和 **getResources( String name )** ：ClassRealm 重写了这两个方法，也是委托给 Strategy 按既定顺序加载类资源数据。

&emsp;**createChildRealm(String id)**：创建具有父子关系的下一级 ClassRealm 实例，实际上是委托给内部引用的 ClassWorld 实例创建新 ClassRealm 实例，并通过调用 `childRealm.setParentRealm( this );` 维护父子关系。新创建的 ClassRealm 实例，自然也由同一个 ClassWorld 负责管理。

&emsp;**addURL(URL url)**：ClassRealm 重写了该方法，处理消除以 "jar:" 开头，并以 "!/" 结尾的 URL 字符，并调用父类 URLClassLoader.addURL(URL url) 方法，以向该类加载器添加要加载类的 URL，然后 ClassRealm 就可以从该 URL 指定的位置加载类了。

&emsp;**importFrom(ClassLoader classLoader, String packageName)**：将 classLoader 类加载器对应的包 packageName，加入到该领域类加载器的外部导入集合中，形成对外部某个包及其对应的领域类加载器的链接关系。

&emsp;**importFrom( String realmId, String packageName )**：与上个方法类似，通过所提供的 realmId 在内部引用的 ClassWorld 集合中找到对应的类加载器，然后以找到的 classLoader 调用上面的方法。



<br/><br/>
<a id="12"></a>

## 1.1.2 ClassWorld 类 ##


ClassWorld 是 ClassRealm 的集合，实现对 ClassRealm 的管理。其内部维护一个 `Map<String, ClassRealm> realms`，其中 key 为 realmId，value 是 ClassRealm 实例。ClassRealm 要求内部必须维护包含它的 ClassWorld 实例的引用，并且每个 ClassRealm 实例通过其内部的 id 来保证它在 ClassWorld 中的唯一性，称为 realmId。ClassWorld 类是线程安全的，其方法都进行了同步处理。


ClassWorld 部分代码如下所示：

```java
public class ClassWorld
{
    private Map<String, ClassRealm> realms;

    private final List<ClassWorldListener> listeners = new ArrayList<ClassWorldListener>();

    public ClassWorld( String realmId, ClassLoader classLoader )
    {
        this();

        try
        {
            newRealm( realmId, classLoader );
        }
        catch ( DuplicateRealmException e )
        {
            // Will never happen as we are just creating the world.
        }
    }

    public ClassWorld()
    {
        this.realms = new LinkedHashMap<String, ClassRealm>();
    }

    
    public ClassRealm newRealm( String id )
        throws DuplicateRealmException
    {
        return newRealm( id, getClass().getClassLoader() );
    }

    public synchronized ClassRealm newRealm( String id, ClassLoader classLoader )
        throws DuplicateRealmException
    {
        if ( realms.containsKey( id ) )
        {
            throw new DuplicateRealmException( this, id );
        }

        ClassRealm realm;

        realm = new ClassRealm( this, id, classLoader );

        realms.put( id, realm );

        for ( ClassWorldListener listener : listeners )
        {
            listener.realmCreated( realm );
        }

        return realm;
    }
}
```

ClassWorld 在内部除了维护 realmId 到 ClassRealm 实例的映射集合外，还维护一个监听器列表 `List<ClassWorldListener> listeners`，可以通过如下方法管理 ClassWorld 监听器：

- **addListener(ClassWorldListener listener)** ：向 ClassWorld 添加监听器
- **removeListener(ClassWorldListener listener)**：将监听器移出 ClassWorld 

ClassWorldListener 监听器接口定义如下：

```java
public interface ClassWorldListener
{
    public void realmCreated( ClassRealm realm );

    public void realmDisposed( ClassRealm realm );
}
```

ClassWorld 在创建新 ClassRealm 和将 ClassRealm 移除 ClassWorld 管理时，触发监听器调用，这对包含 ClassWorld 实例的组件提供了良好的通信机制。


<br/><br/>

#### <font size=4 color=green><b>ClassWorld 构造器</b></font> ####

ClassWorld 提供两个构造器：

- **ClassWorld()**：创建一个空的 ClassWorld 实例。
- **ClassWorld(String realmId, ClassLoader classLoader)**：创建一个 ClassWorld 实例之后，再以 realmId 和 classLoader 参数创建一个 ClassRealm 类加载器，并把它加入到 ClassWorld 实例进行管理。




<br/><br/>

#### <font size=4 color=green><b>ClassWorld 方法</b></font> ####

ClassWorld 对外提供如下方法：

- **ClassRealm newRealm( String id, ClassLoader classLoader )**：以提供的 classLoader 作为新 ClassRealm 实例的基本加载器，并以 id 为标识符，创建新的 ClassRealm 实例，将实例加入到 ClassWorld 的管理中，并触发所有监听器的 realmCreated( ClassRealm realm ) 方法。
- **ClassRealm 	newRealm(String id)**：以加载该 ClassWorld 的类加载器作为新 ClassRealm 实例的基本加载器，并以 id 为标识符，创建新的 ClassRealm 实例，将实例加入到 ClassWorld 的管理中，最后触发所有监听器的 realmCreated( ClassRealm realm ) 方法。
- **disposeRealm(String id)**：将指定 id 的 ClassRealm 实例移除 ClassWorld 的管理，并触发所有监听器的 realmDisposed( ClassRealm realm ) 方法。
- **ClassRealm getClassRealm( String id )**：如果在 `Map<String, ClassRealm> realms` 找到指定 id 的 ClassRealm 实例，返回该实例，否则返回 null 值。
- **ClassRealm getRealm( String id )**：如果在 `Map<String, ClassRealm> realms` 找到指定 id 的 ClassRealm 实例，返回该实例，否则抛出 NoSuchRealmException 异常。
- **Collection<ClassRealm> getRealms()**：返回 ClassWorld 中管理的全部 ClassRealm 实例集合。





<br/><br/>
<a id="2"></a>

## 1.2 使用 Classworlds API ##

Java API 可以用于创建新的 realm 类加载器，并通过具体包的导入将这些 realm 连接起来。

Classworlds 的核心基础结构是 ClassWorld 类。应用程序必须创建 ClassWorld 的实例，最好把它作为单例实例存储。

```java

ClassWorld world = new ClassWorld();

```

创建了 ClassWorld 之后，就可以在 ClassWorld 内创建某个具体领域的类加载器 ClassRealm 了。

```java
    ClassWorld world = new ClassWorld();
    ClassRealm containerRealm    = world.newRealm( "container" );
    ClassRealm logComponentRealm = world.newRealm( "logComponent" );
```

这时，新创建的 ClassRealm 实际上只能加载核心 JVM 类，要使每个 ClassRealm 变得真正有用，必须要给每一个领域类加载器 ClassRealm 添加要加载的部分，才能提供特定范围内类的加载。通过 ClassRealm.addURL(URL url) 方法为 ClassRealm 类加载器添加领域 URL：

```java
    containerRealm.addURL( containerJarUrl );
    logComponentRealm.addURL( logComponentJarUrl );
```

现在，需要创建各种领域间的链接，以使从一个 ClassRealm 加载的类，可以被由另一个 ClassRealm 加载的类所用。

```java
    logComponentRealm.importFrom( "container", 
                                  "com.werken.projectz.component" );
```

代码将 "container" 对应的领域类加载器，即 containerRealm 及其对应的包 "com.werken.projectz.component" 加入到 logComponentRealm 领域类加载器内的 foreignImports 集合。这时，如果通过 `ClassRealm.loadClass( String name )` 方法加载某个全限定类名指定的类，就会调用 `ClassRealm.loadClassFromImport( String name )` 方法，该方法首先在 foreignImports 集合中查找与所提供的全限定类名具有相同包名的领域类加载器，如果找到，就以该领域类加载器加载 name 所指定的类。

容器实现，可以由容器的领域类加载器加载容器类，然后使用加载完成的容器类：

```java
    Class containerClass = containerRealm.loadClass( CONTAINER_CLASSNAME );
    MyContainer container = (MyContainer) containerClass.newInstance();
    Thread.currentThread().setContextClassLoader( containerRealm.getClassLoader() );
    container.run();
```

理想情况下，容器本身应该负责为每个它要加载的组件创建一个 ClassRealm，并将组件约定接口导入到组件的 ClassRealm 中，并使用 loadClass(String name) 方法获得进入沙箱组件领域的入口。



