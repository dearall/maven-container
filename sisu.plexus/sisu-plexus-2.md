## 第2章 sisu.plexus 容器接口及默认实现 ##

sisu.plexus 重新定义了 **PlexusContainer** 接口，而没有直接使用传统 Plexus container 类库定义的接口，因此它不依赖传统 Plexus container 类库：plexus-container-default。但它保留了原始的包名 org.codehaus.plexus，并且在该包及其子包下重新定义了传统 plexus 容器的大部分接口和类。**MutablePlexusContainer** 接口继承自 PlexusContainer，也进行了重新定义。sisu.plexus 提供的容器默认实现 **DefaultPlexusContainer** 类，同样位于 org.codehaus.plexus 包，同时实现 PlexusContainer 和 MutablePlexusContainer 接口，但移除了 **LogEnabled** 日志控制接口的实现。



<br/><br/>
<a id="1"></a>

## 2.1 sisu.plexus 的 PlexusContainer 接口 ##

观察 sisu.plexus 的 PlexusContainer 接口定义，并注意它与传统 Plexus PlexusContainer 接口定义的不同之处。sisu.plexus 的 PlexusContainer 接口定义如下所示：

```java

package org.codehaus.plexus;

public interface PlexusContainer
{
    /** 
     * Returns this container's context. A Context is a simple data store used to hold values which may alter the
     * execution of the Container.
     * @return this container's context.
     */
    Context getContext();

    // ------------------------------------------------------------------------
    // Lookup
    // ------------------------------------------------------------------------

    /**
     * Looks up and returns a component object with the given unique key or role.
     * @param role a unique key for the desired component
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    Object lookup( String role )
        throws ComponentLookupException;

    /**
     * Looks up and returns a component object with the given unique role/role-hint combination.
     * @param role a non-unique key for the desired component
     * @param roleHint a hint for the desired component implementation
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    Object lookup( String role, String hint )
        throws ComponentLookupException;
    /**
     * Looks up and returns a component object with the given unique key or role.
     * @param type the unique type of the component within the container
     * @param <T> The type.
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> T lookup( Class<T> role )
        throws ComponentLookupException;
    /**
     * Looks up and returns a component object with the given unique role/role-hint combination.
     * @param type the non-unique type of the component
     * @param roleHint a hint for the desired component implementation
     * @param <T> The type.
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> T lookup( Class<T> role, String hint )
        throws ComponentLookupException;
    /**
     * Looks up and returns a component object with the given unique role/role-hint combination.
     * @param type the non-unique type of the component
     * @param role a non-unique key for the desired component
     * @param roleHint a hint for the desired component implementation
     * @param <T> The type.
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> T lookup( Class<T> type, String role, String hint )
        throws ComponentLookupException;
    /**
     * Looks up and returns a List of component objects with the given role.
     * @param role a non-unique key for the desired components
     * @return a List of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    List<Object> lookupList( String role )
        throws ComponentLookupException;
    /**
     * Looks up and returns a List of component objects with the given role.
     * @param type the non-unique type of the components
     * @param <T> The type.
     * @return a List of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> List<T> lookupList( Class<T> role )
        throws ComponentLookupException;
    /**
     * Looks up and returns a Map of component objects with the given role, keyed by all available role-hints.
     * @param role a non-unique key for the desired components
     * @return a Map of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    Map<String, Object> lookupMap( String role )
        throws ComponentLookupException;
    /**
     * Looks up and returns a Map of component objects with the given role, keyed by all available role-hints.
     * @param type the non-unique type of the components
     * @param <T> The type.
     * @return a Map of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> Map<String, T> lookupMap( Class<T> role )
        throws ComponentLookupException;
    /**
     * Returns true if this container has the keyed component.
     * @param role a non-unique key for the desired component
     * @return true if this container has the keyed component
     */
    boolean hasComponent( String role );
    /**
     * Returns true if this container has a component with the given role/role-hint.
     * @param role a non-unique key for the desired component
     * @param roleHint a hint for the desired component implementation
     * @return true if this container has a component with the given role/role-hint
     */
    boolean hasComponent( String role, String hint );
    /**
     * Returns true if this container has a component with the given role/role-hint.
     * @param type the non-unique type of the component
     * @return true if this container has a component with the given role/role-hint
     */
    boolean hasComponent( Class<?> role );
    /**
     * Returns true if this container has a component with the given role/role-hint.
     * @param type the non-unique type of the component
     * @param roleHint a hint for the desired component implementation
     * @return true if this container has a component with the given role/role-hint
     */
    boolean hasComponent( Class<?> role, String hint );
    /**
     * Returns true if this container has a component with the given role/role-hint.
     * @param type the non-unique type of the component
     * @param role a non-unique key for the desired component
     * @param roleHint a hint for the desired component implementation
     * @return true if this container has a component with the given role/role-hint
     */
    boolean hasComponent( Class<?> type, String role, String hint );

    void addComponent( Object component, String role );
    /**
     * Adds live component instance to this container.
     *
     * Component instance is not associated with any class realm and will
     * be ignored during lookup is lookup realm is provided using thread context
     * classloader.
     * @param component The component.
     * @param role The role.
     * @param roleHint The hint.
     * @param <T> The type.
     */
    <T> void addComponent( T component, Class<?> role, String hint );
    /**
     * Adds a component descriptor to this container. componentDescriptor should have realmId set.
     * @param componentDescriptor {@link ComponentDescriptor}
     * @throws CycleDetectedInComponentGraphException In case of an error.
     */
    <T> void addComponentDescriptor( ComponentDescriptor<T> descriptor )
        throws CycleDetectedInComponentGraphException;
 
    // ----------------------------------------------------------------------
    // Component Descriptor Lookup
    // ----------------------------------------------------------------------
    /**
     * Returns the ComponentDescriptor with the given component role and hint.
     * Searches up the hierarchy until one is found, null if none is found.
     * @param role a unique role for the desired component's descriptor
     * @param roleHint a hint showing which implementation should be used
     * @return the ComponentDescriptor with the given component role
     */
    ComponentDescriptor<?> getComponentDescriptor( String role, String hint );
    /**
     * Returns the ComponentDescriptor with the given component role and hint.
     * Searches up the hierarchy until one is found, null if none is found.
     * @param type the Java type of the desired component
     * @param role a unique role for the desired component's descriptor
     * @param roleHint a hint showing which implementation should be used
     * @param <T> The type.
     * @return the ComponentDescriptor with the given component role
     */
    <T> ComponentDescriptor<T> getComponentDescriptor( Class<T> type, String role, String hint );
    
    /**
     * Returns a List of ComponentDescriptors with the given role. Searches up the hierarchy until all are found, an
     * empty List if none are found.
     * @param role a non-unique key for the desired components
     * @return a List of component descriptors
     */
    List<ComponentDescriptor<?>> getComponentDescriptorList( String role );

    /**
     * Returns a List of ComponentDescriptors with the given role. Searches up the hierarchy until all are found, an
     * empty List if none are found.
     * @param type the Java type of the desired components
     * @param role a non-unique key for the desired components
     * @param <T> The type.
     * @return a List of component descriptors
     */
    <T> List<ComponentDescriptor<T>> getComponentDescriptorList( Class<T> type, String role );

    /**
     * Returns a Map of ComponentDescriptors with the given role, keyed by role-hint. Searches up the hierarchy until
     * all are found, an empty Map if none are found.
     * @param role a non-unique key for the desired components
     * @return a Map of component descriptors keyed by role-hint
     */
    Map<String, ComponentDescriptor<?>> getComponentDescriptorMap( String role );
    
    /**
     * Returns a Map of ComponentDescriptors with the given role, keyed by role-hint. Searches up the hierarchy until
     * all are found, an empty Map if none are found.
     * @param type the Java type of the desired components
     * @param role a non-unique key for the desired components
     * @param <T> The type.
     * @return a Map of component descriptors keyed by role-hint
     */
    <T> Map<String, ComponentDescriptor<T>> getComponentDescriptorMap( Class<T> type, String role );

    /**
     * Discovers components in the given realm.
     * @param childRealm {@link ClassRealm}
     * @return list {@link ComponentDescriptor}
     * @throws PlexusConfigurationException in case of an error.
     * @throws CycleDetectedInComponentGraphException in case of an error.
     */
    List<ComponentDescriptor<?>> discoverComponents( ClassRealm classRealm )
        throws PlexusConfigurationException;

    /**
     * Returns the Classworld's ClassRealm of this Container, which acts as the default parent for all contained
     * components.
     * @return the ClassRealm of this Container
     */
    ClassRealm getContainerRealm();

    /**
     * Sets the lookup realm to use for lookup calls that don't have a ClassRealm parameter.
     * @param realm the new realm to use.
     * @return The previous lookup realm. It is advised to set it back once the old-style lookups have completed.
     */
    ClassRealm setLookupRealm( ClassRealm realm );

    /**
     * Returns the lookup realm for this container, which is either
     * the container realm or the realm set by {@link MutablePlexusContainer#setLookupRealm(ClassRealm)}.
     * @return {@link ClassRealm}
     */
    ClassRealm getLookupRealm();

    ClassRealm createChildRealm( String id );

    /**
     * Releases the component from the container. This is dependent upon how the implementation manages the component,
     * but usually enacts some standard lifecycle shutdown procedure on the component. In every case, the component is
     * no longer accessible from the container (unless another is created).
     * @param component the plexus component object to release
     * @throws ComponentLifecycleException in case of an error.
     */
    void release( Object component )
        throws ComponentLifecycleException;

    /**
     * Releases all Mapped component values from the container.
     * @see PlexusContainer#release( Object component )
     * @param components Map of plexus component objects to release
     * @throws ComponentLifecycleException in case of an error.
     */
    void releaseAll( Map<String, ?> components )
        throws ComponentLifecycleException;

    /**
     * Releases all Listed components from the container.
     * @see PlexusContainer#release( Object component )
     * @param components List of plexus component objects to release
     * @throws ComponentLifecycleException in case of an error.
     */
    void releaseAll( List<?> components )
        throws ComponentLifecycleException;


    /**
     * Disposes of this container, which in turn disposes all of it's components. This container should also remove
     * itself from the container hierarchy.
     */
    void dispose();
}

```

可以看到，sisu.plexus 定义的 PlexusContainer 接口与传统 PlexusContainer 接口定义有不小的差别：

1. sisu.plexus 定义的 PlexusContainer 接口移除了 **String ROLE = PlexusContainer.class.getName()** 静态字段的定义
<br/>

2. sisu.plexus 定义的 PlexusContainer 接口移除了 **\<T\> T lookup( ComponentDescriptor\<T\> componentDescriptor )** 方法
<br/>

3. sisu.plexus 定义的 PlexusContainer 接口移除了 **List\<Object\> lookupList( String role, List\<String\> roleHints )** 方法
<br/>

4. sisu.plexus 定义的 PlexusContainer 接口移除了 **\<T\> List\<T\> lookupList( Class\<T\> type, List\<String\> roleHints )** 方法
<br/>

5. sisu.plexus 定义的 PlexusContainer 接口移除了 **Map\<String, Object\> lookupMap( String role, List\<String\> roleHints )** 方法
<br/>

6. sisu.plexus 定义的 PlexusContainer 接口移除了 **\<T\> Map\<String, T\> lookupMap( Class\<T\> type, List\<String\> roleHints )** 方法
<br/>

7. sisu.plexus 定义的 PlexusContainer 接口移除了 **ComponentDescriptor\<?\> getComponentDescriptor( String role )** 方法
<br/>

8. sisu.plexus 定义的 PlexusContainer 接口移除了 **List<ComponentDescriptor\<?\>\> discoverComponents( ClassRealm realm, Object data )** 方法
<br/>

9. sisu.plexus 定义的 PlexusContainer 接口移除了 **ClassRealm getComponentRealm( String realmId )** 方法
<br/>

看得出，sisu.plexus 的 PlexusContainer 接口定义，相当于传统 Plexus PlexusContainer 接口定义的一个子集，没有增加额外的方法，但使用时要注意被移除的那些方法在 sisu.plexus 中是不存在的。







<br/><br/>
<a id="2"></a>

## 2.2 sisu.plexus 的 MutablePlexusContainer 接口 ##

与 PlexusContainer 接口类似，sisu.plexus 定义的 MutablePlexusContainer 接口和 Plexus 中定义的 MutablePlexusContainer 接口也有很大不同，如下代码所示：

```java

package org.codehaus.plexus;

public interface MutablePlexusContainer
    extends PlexusContainer
{
    LoggerManager getLoggerManager();

    void setLoggerManager( LoggerManager loggerManager );

    Logger getLogger();

    ClassWorld getClassWorld();
}

```

只提供了日志和 Classword 的管理，移除了 Plexus 中定义的大部分对容器内部组件管理的方法，因为 sisu.plexus 使用底层的 guice 管理组件，plexus 容器本身不需要再建立自己的组件管理机制了。

1. sisu.plexus 定义的 MutablePlexusContainer 接口移除了 **ComponentRegistry getComponentRegistry()** 和 **void setComponentRegistry( ComponentRegistry componentRegistry )** 方法
<br />

2. sisu.plexus 定义的 MutablePlexusContainer 接口移除了 **ComponentDiscovererManager getComponentDiscovererManager()** 和 **void setComponentDiscovererManager( ComponentDiscovererManager componentDiscovererManager )** 方法
<br />

3. sisu.plexus 定义的 MutablePlexusContainer 接口移除了 **ComponentFactoryManager getComponentFactoryManager()** 和 **void setComponentFactoryManager( ComponentFactoryManager componentFactoryManager )** 方法
<br />

4. sisu.plexus 定义的 MutablePlexusContainer 接口移除了 **void setConfigurationSource( ConfigurationSource configurationSource )** 和 **ConfigurationSource getConfigurationSource()** 方法
<br />

5. sisu.plexus 定义的 MutablePlexusContainer 接口移除了 **void setConfiguration( PlexusConfiguration configuration )** 和 **PlexusConfiguration getConfiguration()** 方法
<br />

6. sisu.plexus 定义的 MutablePlexusContainer 接口移除了 **ClassRealm getComponentRealm( String realmId )** 方法


&emsp;&emsp;&emsp;&emsp;\*&emsp;&emsp;&emsp;&emsp;\*&emsp;&emsp;&emsp;&emsp;\*&emsp;&emsp;&emsp;&emsp;
<br />


**因此，提醒注意：** 对于使用 sisu.plexus 容器的应用程序来说，应该使用这里定义的 PlexusContainer 和 MutablePlexusContainer 接口提供的方法访问 plexus 容器，而不能依据传统 Plexus container 项目提供的那两个接口与容器交互，因为有些方法已经被移除了。




<br/><br/>
<a id="3"></a>

## 2.3 sisu.plexus 提供的 Plexus 容器的默认实现 DefaultPlexusContainer 类 ##

sisu.plexus 也使用 DefaultPlexusContainer 类提供容器实现，但此 DefaultPlexusContainer 类非彼 DefaultPlexusContainer 类，二者除了包和类名之外，是两个完全不同的类。下面给出 sisu.plexus 实现的 DefaultPlexusContainer 类部分代码进行简单的分析：

```java

package org.codehaus.plexus;

public final class DefaultPlexusContainer
    implements MutablePlexusContainer
{
    // ----------------------------------------------------------------------
    // Static initialization
    // ----------------------------------------------------------------------

    static
    {
        System.setProperty( "guice.disable.misplaced.annotation.check", "true" );
    }

    // ----------------------------------------------------------------------
    // Constants
    // ----------------------------------------------------------------------

    private static final String DEFAULT_REALM_NAME = "plexus.core";

    private static final Module[] NO_CUSTOM_MODULES = {};

    // ----------------------------------------------------------------------
    // Implementation fields
    // ----------------------------------------------------------------------

    final AtomicInteger plexusRank = new AtomicInteger();

    final Map<ClassRealm, List<ComponentDescriptor<?>>> descriptorMap =
        new IdentityHashMap<ClassRealm, List<ComponentDescriptor<?>>>();

    final ThreadLocal<ClassRealm> lookupRealm = new ThreadLocal<ClassRealm>();

    final LoggerManagerProvider loggerManagerProvider = new LoggerManagerProvider();

    final MutableBeanLocator qualifiedBeanLocator = new DefaultBeanLocator();

    final Context context;

    final Map<?, ?> variables;

    final ClassRealm containerRealm;

    final ClassRealmManager classRealmManager;

    final PlexusBeanLocator plexusBeanLocator;

    final BeanManager plexusBeanManager;

    private final String componentVisibility;

    private final boolean isAutoWiringEnabled;

    private final BeanScanning scanning;

    private final Module containerModule = new ContainerModule();

    private final Module defaultsModule = new DefaultsModule();

    private LoggerManager loggerManager = new ConsoleLoggerManager();

    private Logger logger;

    private boolean disposing;

    // ----------------------------------------------------------------------
    // Constructors
    // ----------------------------------------------------------------------

    public DefaultPlexusContainer()
        throws PlexusContainerException
    {
        this( new DefaultContainerConfiguration() );
    }

    public DefaultPlexusContainer( final ContainerConfiguration configuration )
        throws PlexusContainerException
    {
        this( configuration, NO_CUSTOM_MODULES );
    }

    @SuppressWarnings( "finally" )
    public DefaultPlexusContainer( final ContainerConfiguration configuration, final Module... customModules )
        throws PlexusContainerException
    {
        ...
    }
}

```

由代码可以看出，sisu.plexus 的 DefaultPlexusContainer 类没有从 AbstractLogEnabled 类继承，也没有实现 LogEnabled 接口。

sisu.plexus 的 DefaultPlexusContainer 类内部通过更多的组件来管理其它组件的创建和管理工作，但它们都是轻量级的对象，没有传统 Plexus container 那么厚重的组件注册中心 ComponentRegistry 组件。 


<br/><br/>

#### <font size=4 color=green>sisu.plexus 的 <b>DefaultPlexusContainer 构造器</b></font> ####

DefaultPlexusContainer 提供了三个个构造器：

- **DefaultPlexusContainer()** ：通过 ContainerConfiguration 的默认实现 DefaultContainerConfiguration 调用第二个构造器。
<br/>

- **DefaultPlexusContainer( final ContainerConfiguration configuration )**：通过提供的自定义配置 ContainerConfiguration 和内置的 0 长度 Module 数组调用第三个构造器。
<br/>

- **DefaultPlexusContainer( final ContainerConfiguration configuration, final Module... customModules )**：这是真正执行容器创建的构造器，前两个构造器通过提供默认的容器配置参数 ContainerConfiguration 和一个内置的 0 长度 Module 数组参数 NO_CUSTOM_MODULES 调用这个构造器。ContainerConfiguration 是容器配置接口，sisu.plexus 也对该接口进行了重新定义，参考下一篇《sisu.plexus 容器配置接口及其默认实现》。Module 是 Google guice 容器定义的模块接口，后面会单独探讨它的概念，先研究这个构造器的逻辑。

下面是构造器的代码实现：


```java

    @SuppressWarnings( "finally" )
    public DefaultPlexusContainer( final ContainerConfiguration configuration, final Module... customModules )
        throws PlexusContainerException
    {
        final URL plexusXml = lookupPlexusXml( configuration );

        context = getContextComponent( configuration );
        context.put( PlexusConstants.PLEXUS_KEY, this );
        variables = new ContextMapAdapter( context );

        containerRealm = lookupContainerRealm( configuration );
        classRealmManager = new ClassRealmManager( qualifiedBeanLocator );
        containerRealm.getWorld().addListener( classRealmManager );

        componentVisibility = configuration.getComponentVisibility();
        isAutoWiringEnabled = configuration.getAutoWiring();

        scanning = parseScanningOption( configuration.getClassPathScanning() );

        plexusBeanLocator = new DefaultPlexusBeanLocator( qualifiedBeanLocator, componentVisibility );
        final BeanManager jsr250Lifecycle = configuration.getJSR250Lifecycle() ? new LifecycleManager() : null;
        plexusBeanManager = new PlexusLifecycleManager( Providers.of( context ), loggerManagerProvider, //
                                                        new SLF4JLoggerFactoryProvider(), jsr250Lifecycle );

        setLookupRealm( containerRealm );

        final List<PlexusBeanModule> beanModules = new ArrayList<PlexusBeanModule>();

        final ClassSpace space = new URLClassSpace( containerRealm );
        beanModules.add( new PlexusXmlBeanModule( space, variables, plexusXml ) );
        final BeanScanning global = BeanScanning.INDEX == scanning ? BeanScanning.GLOBAL_INDEX : scanning;
        beanModules.add( new PlexusAnnotatedBeanModule( space, variables, global ) );

        try
        {
            addPlexusInjector( beanModules, new BootModule( customModules ) );
        }
        catch ( final RuntimeException e )
        {
            try
            {
                dispose(); // cleanup as much as possible
            }
            finally
            {
                throw e; // always report original failure
            }
        }
    }
```

从代码可以看出，容器没有一直持有 PlexusConfiguration 对象，容器创建完成之后，configuration 对象在容器中就不存在了。

代码首先通过 **URL lookupPlexusXml( final ContainerConfiguration configuration )** 方法获取配置文件的 URL:

```java
final URL plexusXml = lookupPlexusXml( configuration );
```

查找 ContainerConfiguration 提供的额外配置文件，ContainerConfiguration 通过 getContainerConfigurationURL() 和 getContainerConfiguration() 两个方法为容器提供额外配置文件，默认都为 null 值，因此 plexusXml 返回 null 值。

下面这段代码：

```java
    context = getContextComponent( configuration );
    context.put( PlexusConstants.PLEXUS_KEY, this );
    variables = new ContextMapAdapter( context );
```

查找或创建上下文环境数据 Context context 对象，并将容器自身加入到 context 中。ContextMapAdapter 是一个简单的 Map\<Object, Object\> 实现，存放 Context 中的 contextData 数据。ContainerConfiguration 通过 getContext() 和 getContextComponent() 为容器提供 Context 对象或 contextData 数据，默认为 null 值。

代码分析继续：

```java
    containerRealm = lookupContainerRealm( configuration );
    classRealmManager = new ClassRealmManager( qualifiedBeanLocator );
    containerRealm.getWorld().addListener( classRealmManager );
```

查找或创建容器的 ClassRealm 和 ClassWorld 对象。容器中没有直接保留 ClassWorld 对象，因为 ClassRealm 实例中已经持有了该 ClassWorld 对象的引用。这两个对象都可以通过 ContainerConfiguration 配置，默认为 null 值。

qualifiedBeanLocator 的类型为 MutableBeanLocator，由 DefaultBeanLocator 创建，这两个类型来自于 sisu.inject 类库，后面会探讨。ClassRealmManager 类位于 org.eclipse.sisu.plexus 包，是 sisu.plexus 容器的后台实现类之一，管理组件的 ClassRealm。它实现了 ClassWorldListener 接口，并把它作为监听器安装到 containerRealm.getWorld() 中。

代码分析继续：

```java
    componentVisibility = configuration.getComponentVisibility();
    isAutoWiringEnabled = configuration.getAutoWiring();
```

这两行代码直接从容器配置获取值，容器的组件可视性 componentVisibility 和 isAutoWiringEnabled 都是 sisu.inject 中的特性，引入到 sisu.plexus 中，由 sisu.inject 提供支持。

代码分析继续：

```java
scanning = parseScanningOption( configuration.getClassPathScanning() );
```

scanning 的类型为 BeanScanning，来自于 sisu.inject 项目的 org.eclipse.sisu.space.BeanScanning，是扫描枚举类型，取值如下：

<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center>类型</th>
        <th align=center>枚举值</th>
        <th align=center>说明</th>
    </tr>
    <tr>
      <td>BeanScanning </td>
      <td>CACHE</td>
      <td>扫描一次并把结果缓存起来</td>
    </tr>
    <tr>
      <td>BeanScanning </td>
      <td>GLOBAL_INDEX</td>
      <td>使用全局目录（应用程序）- index (application) </td>
    </tr>
    <tr>
      <td>BeanScanning </td>
      <td>INDEX </td>
      <td>使用本地目录（插件）- index (plug-ins) </td>
    </tr>
    <tr>
      <td>BeanScanning </td>
      <td>OFF</td>
      <td>从不不扫描</td>
    </tr>
    <tr>
      <td>BeanScanning </td>
      <td>ON</td>
      <td>总是扫描</td>
    </tr>
</table>

**scanning = parseScanningOption( configuration.getClassPathScanning() )** 调用将容器配置选项中取出的类路径扫描字符串 configuration.getClassPathScanning()，转换为可用的 BeanScanning 类型，存储在变量 scanning 中。如果没有找到匹配的扫描方式，scanning 设置为 BeanScanning.OFF 值。

代码分析继续：

```java
    plexusBeanLocator = new DefaultPlexusBeanLocator( qualifiedBeanLocator, componentVisibility );
    final BeanManager jsr250Lifecycle = configuration.getJSR250Lifecycle() ? new LifecycleManager() : null;
    plexusBeanManager = new PlexusLifecycleManager( Providers.of( context ), loggerManagerProvider, //
                                                    new SLF4JLoggerFactoryProvider(), jsr250Lifecycle );

```

plexusBeanLocator 的类型为 org.eclipse.sisu.plexus.PlexusBeanLocator 接口，是 sisu.plexus 容器的底层实现定义的接口之一，提供各种类型 bean 的定位服务，使用可选的 plexus hint 标识。默认实现为 org.eclipse.sisu.plexus.DefaultPlexusBeanLocator 类。

BeanManager 是 sisu.inject 定义的接口，并由 LifecycleManager 提供 JSR250 规范定义的 bean 和调度器声明周期事件的实现。plexusBeanManager 的类型也是 BeanManager，sisu.plexus 通过对 sisu.inject 实现的封装，提供了 plexus 容器的实现： org.eclipse.sisu.plexus.PlexusLifecycleManager，用于管理 Plexus 容器托管要求生命周期管理的组件。 

代码分析继续：

```java
setLookupRealm( containerRealm );
```

使用线程本地变量存储容器 ClassRealm 对象。

代码分析继续：

```java
    final ClassSpace space = new URLClassSpace( containerRealm );
    beanModules.add( new PlexusXmlBeanModule( space, variables, plexusXml ) );
    final BeanScanning global = BeanScanning.INDEX == scanning ? BeanScanning.GLOBAL_INDEX : scanning;
    beanModules.add( new PlexusAnnotatedBeanModule( space, variables, global ) );
```

类空间 ClassSpace 来自于 sisu.inject 的定义，URLClassSpace 实现 ClassSpace 接口，它们都定义于 org.eclipse.sisu.space 包。beanModules 的类型为 `List<PlexusBeanModule>`，是 sisu.plexus 容器管理 PlexusBeanModule 的链表数据结构，PlexusXmlBeanModule 和 PlexusAnnotatedBeanModule 都是 PlexusBeanModule 的具体实现。其中 PlexusXmlBeanModule 在它的 configure() 方法中调用了 PlexusXmlScanner 的 scan() 方法，而 **PlexusXmlScanner.scan( final ClassSpace space, final boolean root )** 方法正是执行扫描和解析 **"META-INF/plexus/components.xml"** 配置文件的工作。这个文件是传统 Plexus 容器标准的组件配置文件。而 PlexusAnnotatedBeanModule 内部使用 SpaceModule 针对给定的 `URLClassSpace( containerRealm )` 执行类路径扫描，自动绑定由 `@Qualifier` 注解的注解标注的，或者由 `@Named` 注解的类型。

这样，**sisu.plexus 容器同时支持 "META-INF/plexus/components.xml" 配置组件和类路径扫描配置组件**。


代码分析继续：

```java
addPlexusInjector( beanModules, new BootModule( customModules ) );
```

注意，customModules 被包装到 `BootModule` 引导模块，因此可以 customModules 中提供自己的引导模块来构建容器。进入 `addPlexusInjector()` 的方法实现：

```java
    public Injector addPlexusInjector( final List<? extends PlexusBeanModule> beanModules,
                                       final Module... customModules )
    {
        final List<Module> modules = new ArrayList<Module>();

        modules.add( containerModule );
        Collections.addAll( modules, customModules );
        modules.add( new PlexusBindingModule( plexusBeanManager, beanModules ) );
        modules.add( defaultsModule );

        return Guice.createInjector( isAutoWiringEnabled ? new WireModule( modules ) : new MergedModule( modules ) );
    }
```

其中，在类的数据定义部分，有如下定义：

```java
    private final Module containerModule = new ContainerModule();
    private final Module defaultsModule = new DefaultsModule();
```

ContainerModule 和 DefaultsModule 都来自 DefaultPlexusContainer 内部类定义。PlexusBindingModule 是 guice 的 Module 实现，支持 plexus bean 的注册、注入、及管理。PlexusBindingModule 配置了 PlexusBeanModule 接口，PlexusXmlBeanModule 实现了 PlexusBeanModule 接口，它的功能就是通过扫描 XML 资源绑定 Plexus 组件。

`addPlexusInjector()` 方法的实现，完全是 Guice 容器的标准启动方式。配置模块 Module 是 Guice 的基本配置单元，准备各种 Guice 配置模块 Module，组成 `List<Module>` 集合，然后将链表集合包装成 sisu.inject 的 `WireModule` 模块或 `MergedModule` 模块，最后将经过包装的 `WireModule` 或 `MergedModule` 模块传递给 Guice 框架的静态方法 **Guice.createInjector(Module... modules)** 创建 Injector 对象，完成 sisu.plexus 容器的初始化工作。但没有将返回的 Injector 对象保留起来。

其中 WireModule 的特性是能自动解决缺失的依赖，MergedModule 的特性是能去掉重复或断开的绑定（即缺失依赖的绑定）。如果配置容器的 isAutoWiringEnabled 为 true 值，那么 **sisu.plexus 容器同时支持 "META-INF/plexus/components.xml" 配置组件和类路径扫描配置组件，还支持缺失依赖的自动连接特性**。后面两个特性正是 sisu.inject 容器提供的天然支持。



<br/><br/>
<a id="31"></a>

## 2.3.1 DefaultPlexusContainer 的核心组件 ##





<br/><br/>

#### <font size=4 color=green><b>DefaultPlexusContainer 静态常量</b></font> ####

`DefaultPlexusContainer` 定义了两个静态常量，如下所示：

```java
    private static final String DEFAULT_REALM_NAME = "plexus.core";

    private static final Module[] NO_CUSTOM_MODULES = {};
```

`DEFAULT_REALM_NAME` 是默认的领域类加载器的名字，`NO_CUSTOM_MODULES` 是一个空的 `Module[]` 数组，用于为没有提供 `Module[]` 数组参数的构造器提供默认值。`Module` 接口是 Guice 配置逻辑的基本单元，所有组件必须通过 `Module` 的配置才能托管给 Guice 容器管理，我们才能从容器中获取到所需的组件对象。sisu.inject 容器和 sisu.plexus 容器的底层仍然是 Guice 容器，因此，也必须通过这个基本配置单元配置组件。


<br/><br/>

#### <font size=4 color=green><b>Map<ClassRealm, List<ComponentDescriptor<?>>> descriptorMap =new IdentityHashMap<ClassRealm, List<ComponentDescriptor<?>>>()</b></font> ####

首先，这个 Map 保存了每一个 ClassRealm 和它所对应的组件描述符 `ComponentDescriptor<?>` 列表，也就是通过这个对应关系，确立了某一个具体的 ClassRealm 和它所管理的组件描述符的集合的对应关系，这个集合中每一个描述符所描述的组件，都由该 ClassRealm 加载。组件描述符 `ComponentDescriptor` 仍然是 plexus 容器的传统概念，用于描述组件的配置信息，这里直接拿来使用，只是 sisu.plexus 在自己的模块中进行了重新定义。

第二，这个 Map 使用的是 `IdentityHashMap` 实现，而 `IdentityHashMap` 使用 key 的地址作为 key 之间的比较操作，这正符合以 ClassRealm 的实例对象作为 key 的 Map 定义。


<br/><br/>

#### <font size=4 color=green><b>LoggerManagerProvider loggerManagerProvider = new LoggerManagerProvider()</b></font> ####

**LoggerManagerProvider** 是 `LoggerManager` 的提供者。再向容器请求时，容器通过该提供者返回容器字段 `LoggerManager loggerManager`，即 `new ConsoleLoggerManager()` 的实例。


<br/><br/>

#### <font size=4 color=green><b>MutableBeanLocator qualifiedBeanLocator = new DefaultBeanLocator()</b></font> ####

由 sisu.inject 提供的 `BeanLocator` 的默认实现。`BeanLocator`、`MutableBeanLocator` 和 `DefaultBeanLocator` 都来自于 sisu.inject 容器，其中 `MutableBeanLocator` 是 `BeanLocator` 的子接口，`DefaultBeanLocator` 实现了 `MutableBeanLocator` 接口。`BeanLocator` 用于查找和跟踪带有 `@Qualifier` 注解的 bean 实现。




<br/><br/>

#### <font size=4 color=green><b>PlexusBeanLocator plexusBeanLocator</b></font> ####

`PlexusBeanLocator` 是 sisu.plexus 容器定义的接口，并由 `DefaultPlexusBeanLocator` 提供默认实现。`PlexusBeanLocator` 用于定位各种类型的 bean 实现，使用传统 plexus 特定的 role 和可选的 hint 作为查找和跟踪依据。在思想上与 `BeanLocator` 类似，都是查找和跟踪组件实现，只不过所使用的依据不同。`BeanLocator` 使用 `@Qualifier` 注解作为查找和跟踪的定位依据，而 `PlexusBeanLocator` 使用 plexus 传统的 role 和可选的 hint 作为查找和跟踪组件实现的依据。但 DefaultPlexusBeanLocator 内部通过构造器引用了一个 BeanLocator 实例，在 `DefaultPlexusContainer` 容器中就是 qualifiedBeanLocator 字段，而 DefaultPlexusBeanLocator 的定位 `locate()` 方法的实现其实是通过 role 和 hint 创建出对应的 Guice 的 Key 对象，然后借助这个引用的 BeanLocator 实例的 locate() 方法实现的。

`PlexusBeanLocator` 定位到的组件是 `PlexusBean` 的集合 `Iterable<PlexusBean<T>>`，由 `DefaultPlexusBeans<T>` 或 `HintedPlexusBeans<T>` 实现。




<br/><br/>

#### <font size=4 color=green><b>BeanManager plexusBeanManager</b></font> ####

`BeanManager` 是 sisu.inject 提供的 Bean 生命周期管理服务。而 sisu.plexus 提供了具有 plexus 特征的实现 PlexusLifecycleManager 类。




<br/><br/>

#### <font size=4 color=green><b>Module containerModule = new ContainerModule()</b></font> ####

内部类 `ContainerModule` 是 Guice 的 `Module` 纯实现，将 `DefaultPlexusContainer` 容器内的组件，以及 `DefaultPlexusContainer` 实例自身配置到 Guice 容器中管理，如下代码所示：

```java
    final class ContainerModule
        implements Module
    {
        public void configure( final Binder binder )
        {
            binder.bind( Context.class ).toInstance( context );
            binder.bind( ParameterKeys.PROPERTIES ).toInstance( context.getContextData() );

            binder.bind( MutableBeanLocator.class ).toInstance( qualifiedBeanLocator );
            binder.bind( PlexusBeanLocator.class ).toInstance( plexusBeanLocator );
            binder.bind( BeanManager.class ).toInstance( plexusBeanManager );

            binder.bind( PlexusContainer.class ).to( MutablePlexusContainer.class );
            binder.bind( MutablePlexusContainer.class ).to( DefaultPlexusContainer.class );

            // use provider wrapper to avoid repeated injections later on when configuring plugin injectors
            binder.bind( DefaultPlexusContainer.class ).toProvider( Providers.of( DefaultPlexusContainer.this ) );
        }
    }
```




<br/><br/>

#### <font size=4 color=green><b>Module defaultsModule = new DefaultsModule()</b></font> ####

内部类 `DefaultsModule` 也是 Guice 的 `Module` 纯实现，它将一些 `DefaultPlexusContainer` 容器的默认组件配置到 Guice 容器中管理，如下代码所示：

```java
    final class DefaultsModule
        implements Module
    {
        private final LoggerProvider loggerProvider = new LoggerProvider();

        private final PlexusDateTypeConverter dateConverter = new PlexusDateTypeConverter();

        public void configure( final Binder binder )
        {
            binder.bind( LoggerManager.class ).toProvider( loggerManagerProvider );
            binder.bind( Logger.class ).toProvider( loggerProvider );

            // allow plugins to override the default ranking function so we can support component profiles
            final Key<RankingFunction> plexusRankingKey = Key.get( RankingFunction.class, Names.named( "plexus" ) );
            binder.bind( plexusRankingKey ).toInstance( new DefaultRankingFunction( plexusRank.incrementAndGet() ) );
            binder.bind( RankingFunction.class ).to( plexusRankingKey );

            binder.install( dateConverter );

            binder.bind( PlexusBeanConverter.class ).to( PlexusXmlBeanConverter.class );
        }
    }

```

其中，`PlexusBeanConverter` 绑定到 `PlexusXmlBeanConverter` 类，用于将 Plexus XML 配置转换成 bean 组件描述对象。


<br/><br/>
<a id="32"></a>

## 2.3.2 DefaultPlexusContainer 公共方法阐释 ##

sisu.plexus 容器的公共方法与传统 Plexus 容器的公共方法具有相同的语义，只是比传统的 PlexusContainer 接口少了部分方法，但不影响容器的正常使用，因为保留下来的都是使用频率最高的主要方法。而实现却有着完全的不同，这是可以理解的，sisu.plexus 容器基于成熟的 sisu.inject 容器和 Guice 容器，底层都是 Guice 提供的容器框架。


<br/><br/>
#### <font size=4 color=green><b>将现有组件加入容器管理</b></font> ####

将现有组件加入到容器管理：是将其它方式创建的组件加入到 sisu.plexus 容器管理。

- **<T> void addComponent(T component, Class\<?\> role, String roleHint)**：通过 qualifiedBeanLocator 将角色 role 绑定到实例 component，如果 roleHint 不为默认的 "default" 值，则通过下面代码绑定：
  ```java
  binder.bind( (Class) role ).annotatedWith( Names.named( hint ) ).toInstance( component );
  ```
  否则通过下面方法绑定：

  ```java
  binder.bind( (Class) role ).toInstance( component );
  ```

  并递增 plexusRank.incrementAndGet() 值，这里实现了一个匿名的 Module，并通过 Guice.createInjector() 方法创建 injector，把这个 injector 和 plexusRank 一起作为参数调用 `qualifiedBeanLocator.add()` 方法。

<br />

- **void addComponent(Object component, String role)**：使用 hint 的默认值 "default" 调用第一个方法，其中通过 `component.getClass().getClassLoader().loadClass( role )` 方法加载 role 的 Class 实例。
<br />

- **void addComponentDescriptor(ComponentDescriptor<?> componentDescriptor)**：将组件描述符加入到容器组件注册中心管理。
  componentDescriptor：为组件描述符对象，如果该描述符内的领域类加载器 ClassRealm 为 null，则将容器的领域类加载器 containerRealm 设置为该组件描述符的领域类加载器。组件描述符的管理在容器的 `Map<ClassRealm, List<ComponentDescriptor<?>>> descriptorMap` 中，前面讨论或该 Map 集合。



<br/><br/>
#### <font size=4 color=green><b>组件查找 Lookup</b></font> ####

通过 Lookup() 系列方法获取组件实例：


sisu.plexus 容器的所有 Lookup() 方法最终都由 DefaultPlexusContainer 的私有 `locate()` 方法实现，代码如下：

```java
    private <T> Iterable<PlexusBean<T>> locate( final String role, final Class<T> type, final String... hints )
    {
        if ( disposing )
        {
            return Collections.EMPTY_SET;
        }
        final String[] canonicalHints = Hints.canonicalHints( hints );
        if ( null == role || null != type && type.getName().equals( role ) )
        {
            return plexusBeanLocator.locate( TypeLiteral.get( type ), canonicalHints );
        }
        final Set<Class> candidates = new HashSet<Class>();
        for ( final ClassRealm realm : getVisibleRealms() )
        {
            try
            {
                final Class clazz = realm.loadClass( role );
                if ( candidates.add( clazz ) )
                {
                    final Iterable beans = plexusBeanLocator.locate( TypeLiteral.get( clazz ), canonicalHints );
                    if ( hasPlexusBeans( beans ) )
                    {
                        return beans;
                    }
                }
            }
            catch ( final Exception e )
            {
                // drop through...
            }
            catch ( final LinkageError e )
            {
                // drop through...
            }
        }
        return Collections.EMPTY_SET;
    }
```

看得出，这个方法的核心是对 `plexusBeanLocator.locate()` 方法的调用。前面提到过，DefaultPlexusBeanLocator 内部通过构造器引用了一个 BeanLocator 实例，在 `DefaultPlexusContainer` 容器中就是是 qualifiedBeanLocator 字段，而 DefaultPlexusBeanLocator 的定位 `locate()` 方法的实现其实是通过 role 和 hint 创建出对应的 Guice 的 Key 对象，然后借助这个引用的 BeanLocator 实例的 locate() 方法实现的。如下所示：

```java
    public <T> Iterable<PlexusBean<T>> locate( final TypeLiteral<T> role, final String... hints )
    {
        final Key<T> key = hints.length == 1 ? Key.get( role, Names.named( hints[0] ) ) : Key.get( role, Named.class );
        Iterable<BeanEntry<Named, T>> beans = (Iterable<BeanEntry<Named, T>>) beanLocator.<Named, T> locate( key );
        if ( PlexusConstants.REALM_VISIBILITY.equalsIgnoreCase( visibility ) )
        {
            beans = new RealmFilteredBeans<T>( beans );
        }
        return hints.length <= 1 ? new DefaultPlexusBeans<T>( beans ) : new HintedPlexusBeans<T>( beans, role, hints );
    }
```

这里的关键是，通过 Guice 的 `Key<T>` 类的静态方法，将 role 和 hints 转换为 Guice 可理解的 Key 对象：

```java
final Key<T> key = hints.length == 1 ? Key.get( role, Names.named( hints[0] ) ) : Key.get( role, Named.class );
```

然后通过这个 Key 对象调用内部引用的 `beanLocator.locate( key )` 方法，返回 `BeanEntry<Named, T>` 的集合。最后通过 `DefaultPlexusBeans` 或 `HintedPlexusBeans` 的封装，返回给调用者。

这就是 sisu.plexus 容器查找组件的核心逻辑。

下面是各个 lookup() 方法的使用说明：

- **Object 	lookup(String role)**：
  role ：为组件角色，一般使用该组件接口或基类的全限定类名
<br />

- **Object 	lookup(String role, String roleHint)**：
  role ：为组件角色，一般使用该组件接口或基类的全限定类名
  roleHint ：为角色区别标识，当多个类实现同一接口时，使用该字符串识别不同的实现类
<br />

- **\<T\> T 	lookup(Class\<T\> type)**：
  type ：要查找或创建组件的接口或基类类型

<br />

- **\<T\> T 	lookup(Class\<T\> type, String roleHint)**：
  type ：要查找或创建组件的接口或基类类型
  roleHint ：为角色区别标识，当多个类实现同一接口时，使用该字符串识别不同的实现类
<br />

- **\<T\> T 	lookup(Class\<T\> type, String role, String roleHint)**：
  type ：要查找或创建组件的接口或基类类型
  role ：为组件角色，一般使用该组件接口或基类的全限定类名。在查找时，通过判断 **if ( isAssignableFrom( type, implClass ) || Object.class == implClass && role.equals( type.getName() ) )** 是否为 true，表明 type 与 role 的相关性
  roleHint ：角色区别标识，当多个类实现同一 role 指明的接口，或多个子类继承同一 role 指明的基类，使用该字符串识别不同的实现类
<br />


- **\<T\> List\<T\> 	lookupList(Class\<T\> type)**：获取或创建全部实现 type 接口的组件对象集合。
  type ：要查找或创建组件的接口或基类类型
<br />


- **List\<Object\> 	lookupList(String role)**：获取或创建全部实现或继承自 role 表明的组件对象集合。
  role ：为组件角色，一般使用该组件接口或基类的全限定类名
<br />


- **\<T\> Map\<String,T\> 	lookupMap(Class\<T\> type)**：获取或创建全部实现 type 接口的组件对象集合。
  type ：要查找或创建组件的接口或基类类型
  返回值：类型为 Map\<String,T\>，其中 key 为 roleHint，值 T 为与 roleHint 对应的组件对象
<br />

- **Map\<String,Object\> 	lookupMap(String role)**：获取或创建全部实现 role 指明接口或基类的组件对象集合。
  role ：为组件角色，一般使用该组件接口或基类的全限定类名
  返回值：类型为 Map\<String,T\>，  其中 key 为 roleHint，值 T 为与 roleHint 对应的组件对象

<br />


<br/><br/>
#### <font size=4 color=green><b>组件描述符查找 Component Descriptor Lookup</b></font> ####


- **ComponentDescriptor\<?\> getComponentDescriptor( String role, String roleHint )**：根据给定的组件 role 和 roleHint 查找组件描述符。
  role ：组件角色，一般使用该组件接口或基类的全限定类名
  roleHint ：角色区别标识，当多个类实现同一 role 指明的接口，或多个子类继承同一 role 指明的基类，使用该字符串识别不同的实现类
<br />

- **\<T\> ComponentDescriptor\<T\> getComponentDescriptor( Class\<T\> type, String role, String roleHint )**：根据给定的组件 type、role 和 roleHint 查找组件描述符。
  type ：要查找或创建组件的接口或基类类型
  role ：为组件角色，一般使用该组件接口或基类的全限定类名。在查找时，通过判断 **if ( isAssignableFrom( type, implClass ) || Object.class == implClass && role.equals( type.getName() ) )** 是否为 true，表明 type 与 role 的相关性
  roleHint ：角色区别标识，当多个类实现同一 role 指明的接口，或多个子类继承同一 role 指明的基类，使用该字符串识别不同的实现类


它的实现也是通过私有的 locate() 方法:

```java
    public <T> ComponentDescriptor<T> getComponentDescriptor( final Class<T> type, final String role, final String hint )
    {
        final Iterator<PlexusBean<T>> i = locate( role, type, hint ).iterator();
        if ( i.hasNext() )
        {
            final PlexusBean<T> bean = i.next();
            if ( bean.getImplementationClass() != null )
            {
                return newComponentDescriptor( role, bean );
            }
        }
        return null;
    }
```

<br />

- **Map\<String, ComponentDescriptor\<?\>\> getComponentDescriptorMap( String role )**：在容器中查找全部符合 role 的组件描述符。
  role ：组件角色，一般使用该组件接口或基类的全限定类名
  返回值：类型为 Map\<String, ComponentDescriptor\<?\>\>，其中 key 为 roleHint，值 ComponentDescriptor 为与 roleHint 对应的组件描述符对象
  返回值：类型为 List\<ComponentDescriptor\<T\>\>，表示所有找到符合条件的组件描述符。如果没有找到，返回空 Map
<br />

- **\<T\> Map<String, ComponentDescriptor\<T\>\> getComponentDescriptorMap( Class\<T\> type, String role )**：在容器中查找全部符合 type 和 role 的组件描述符。
  type ：要查找或创建组件的接口或基类类型
  role ：为组件角色，一般使用该组件接口或基类的全限定类名。在查找时，通过判断 **if ( isAssignableFrom( type, implClass ) || Object.class == implClass && role.equals( type.getName() ) )** 是否为 true，表明 type 与 role 的相关性
  返回值：类型为 List\<ComponentDescriptor\<T\>\>，表示所有找到符合条件的组件描述符。如果没有找到，返回空 Map

  通过私有的 locate() 方法实现：

  ```java

    public <T> Map<String, ComponentDescriptor<T>> getComponentDescriptorMap( final Class<T> type, final String role )
    {
        final Map<String, ComponentDescriptor<T>> tempMap = new LinkedHashMap<String, ComponentDescriptor<T>>();
        for ( final PlexusBean<T> bean : locate( role, type ) )
        {
            tempMap.put( bean.getKey(), newComponentDescriptor( role, bean ) );
        }
        return tempMap;
    }

  ```
<br /> 

- **List\<ComponentDescriptor\<?\>\> getComponentDescriptorList( String role )**：在容器中查找全部符合 role 的组件描述符。
  role ：组件角色，一般使用该组件接口或基类的全限定类名
  返回值：类型为 List\<ComponentDescriptor\<T\>\>，表示所有找到符合条件的组件描述符。如果没有找到，返回空列表
<br /> 

- **<T> List\<ComponentDescriptor\<T\>\> getComponentDescriptorList( Class\<T\> type,  String role )**：在容器中查找全部符合 type 和 role 的组件描述符。
  type ：要查找组件的接口或基类类型
  role ：为组件角色，一般使用该组件接口或基类的全限定类名。在查找时，通过判断 **if ( isAssignableFrom( type, implClass ) || Object.class == implClass && role.equals( type.getName() ) )** 是否为 true，表明 type 与 role 的相关性
  返回值：类型为 List\<ComponentDescriptor\<T\>\>，表示所有找到符合条件的组件描述符。如果没有找到，返回空列表

  通过私有的 locate() 方法实现：

  ```java

    public <T> List<ComponentDescriptor<T>> getComponentDescriptorList( final Class<T> type, final String role )
    {
        final List<ComponentDescriptor<T>> tempList = new ArrayList<ComponentDescriptor<T>>();
        for ( final PlexusBean<T> bean : locate( role, type ) )
        {
            tempList.add( newComponentDescriptor( role, bean ) );
        }
        return tempList;
    }

  ```



<br/><br/>
#### <font size=4 color=green><b>判断容器中是否含有某个符合指定条件的组件</b></font> ####

- **boolean hasComponent( String role )**：如果容器中含有 role 指定的组件实现，返回 true 值
  role ：组件角色。一般使用该组件接口或基类的全限定类名
<br />

- **boolean hasComponent( String role, String roleHint )**：如果容器中含有 role 和 roleHint 指定的组件实现，返回 true 值
  role ：组件角色，一般使用该组件接口或基类的全限定类名
  roleHint ：角色区别标识。当多个类实现同一 role 指明的接口，或多个子类继承同一 role 指明的基类，使用该字符串识别不同的实现类
<br />

- **boolean hasComponent( Class\<?\> type )**：如果容器中含有 type 指定的组件实现，返回 true 值。
  type ：要判断组件的接口或基类类型
<br />

- **boolean hasComponent( Class\<?\> type, String roleHint )**：如果容器中含有 roleHint 标识的 type 组件实现，返回 true 值。
  type ：要判断组件的接口或基类类型
  roleHint ：角色区别标识。当多个类实现同一 type 指明的接口，或多个子类继承同一 type 指明的基类，使用该字符串识别不同的实现类

<br />

- **boolean hasComponent( Class\<?\> type, String role, String roleHint )**：如果容器中含有 roleHint 标识的 type 组件实现，返回 true 值。
  type ：要判断组件的接口或基类类型
  role ：为组件角色，一般指明该组件接口或基类的全限定类名。在查找时，通过判断 **if ( isAssignableFrom( type, implClass ) || Object.class == implClass && role.equals( type.getName() ) )** 是否为 true，表明 type 与 role 的相关性
  roleHint ：角色区别标识。当多个类实现同一 type 指明的接口，或多个子类继承同一 type 指明的基类，使用该字符串识别不同的实现类



<br/><br/>
#### <font size=4 color=green><b> 组件发现 Discover</b></font> ####

- **List\<ComponentDescriptor\<?\>\> discoverComponents( ClassRealm childRealm )**：使用给定的 ClassRealm 发现为容器配置的组件，即通过 META-INF/plexus/components.xml 文件配置的组件。
  childRealm：是加载组件实现类的领域类加载器 ClassRealm，容器默认使用容器的 containerRealm 加载发现的组件
<br/>

- **List\<ComponentDescriptor\<?\>\> discoverComponents( final ClassRealm realm, final Module... customModules )**：使用给定的 ClassRealm 发现为容器配置的组件，即通过 META-INF/plexus/components.xml 文件配置的组件。
  childRealm：是加载组件实现类的领域类加载器 ClassRealm，容器默认使用容器的 containerRealm 加载发现的组件
  customModules：自定义模块配置，通过 Guice.Inject() 方法创建新的注入器，并加入到 BeanLocator 中，并把发现的 bean 加入到 classRealmManager 管理中。实现如下：

  ```java
      public List<ComponentDescriptor<?>> discoverComponents( final ClassRealm realm, final Module... customModules )
    {
        try
        {
            final List<PlexusBeanModule> beanModules = new ArrayList<PlexusBeanModule>();
            synchronized ( descriptorMap )
            {
                final ClassSpace space = new URLClassSpace( realm );
                final List<ComponentDescriptor<?>> descriptors = descriptorMap.remove( realm );
                if ( null != descriptors )
                {
                    beanModules.add( new ComponentDescriptorBeanModule( space, descriptors ) );
                }
                if ( containerRealm != realm && !classRealmManager.isManaged( realm ) )
                {
                    beanModules.add( new PlexusXmlBeanModule( space, variables ) );
                    final BeanScanning local = BeanScanning.GLOBAL_INDEX == scanning ? BeanScanning.INDEX : scanning;
                    beanModules.add( new PlexusAnnotatedBeanModule( space, variables, local ) );
                }
            }
            if ( !beanModules.isEmpty() )
            {
                classRealmManager.manage( realm, addPlexusInjector( beanModules, customModules ) );
            }
        }
        catch ( final RuntimeException e )
        {
            getLogger().warn( realm.toString(), e );
        }

        return null; // no-one actually seems to use or check the returned component list!
    }
  ```

注意，实际上该方法返回 null 值，它的返回值并没有在哪个调用中用到。


<br/><br/>
#### <font size=4 color=green><b>释放组件 Releases the component from the container</b></font> ####

- **void release( Object component )**：将某个组件从容器中释放掉。释放之后，该组件不再是可访问的，除非再创建一个。
  component：要释放的组件对象
<br/>

- **void releaseAll( Map\<String, ?\> components )**：将 components.values() 中每一个组件对象从容器中释放掉。
  components：要释放的组件对象集合
<br/>

- **void releaseAll( List\<?\> components )**：components 中每一个组件对象从容器中释放掉。
  components：要释放的组件对象集合

下面两个方法都是通过循环迭代调用第一个释放方法。



<br/><br/>
#### <font size=4 color=green><b>释放容器</b></font> ####

- **void dispose()**：释放组件注册中心内的全部组件，然后将自己从多容器的层次结构中移除。




<br/><br/>
#### <font size=4 color=green><b>ClassRealm 管理</b></font> ####


- **ClassRealm getContainerRealm()**：返回容器的领域类加载器 ClassRealm，这个 ClassRealm 被容器用作所有容器包含组件的默认父级领域类加载器。

<br/>

// ----------------------------------------------------------------------------
// Component/Plugin ClassRealm creation
// ----------------------------------------------------------------------------

<br/>

- **ClassRealm createChildRealm( String id )**：使用容器领域类加载器 containerRealm，按指定的 id 创建子级领域类加载器，新建的 childRealm 自动被容器的 ClassWord 管理，并设置 childRealm 的父级 ClassRealm 为 containerRealm。 

<br/>


- **void removeComponentRealm( ClassRealm componentRealm )**：将 componentRealm 指定的领域类加载器 ClassRealm 从组件注册中心解除关联关系，这会使容器删除组件注册中心中所有通过 componentRealm 创建的组件。

<br/>

- **ClassRealm getLookupRealm()**：返回线程本地存储 lookupRealm 中的 ClassRealm 对象，用于调用 lookup() 方法查找组件对象，没有提供 ClassRealm 参数时，使用该 ClassRealm 对象。

<br/>

- **ClassRealm setLookupRealm(ClassRealm realm)**：将 realm 指定的 ClassRealm 对象存储到线程本地存储中，用于调用 lookup() 方法查找组件对象，没有提供 ClassRealm 参数时，使用该 ClassRealm 对象。
<br/>

- **ClassRealm getLookupRealm( Object component )**：一个工具方法，用于获取某具体组件实例的 ClassRealm，如果该组件不是由 ClassRealm 加载的，就返回线程本地存储中的容器 ClassRealm。代码实现如下所示：

```java
    /**
     * Utility method to get a default lookup realm for a component.
     */
    public ClassRealm getLookupRealm( Object component )
    {
        if ( component.getClass().getClassLoader() instanceof ClassRealm )
        {
            return ( (ClassRealm) component.getClass().getClassLoader() );
        }
        else
        {
            return getLookupRealm();
        }

    }
```



<br/><br/>
#### <font size=4 color=green><b>Context 管理</b></font> ####

- **Context getContext()**：获取容器的 Context 对象，Context 类似于 Map 结构的数据对象，用于持有对容器影响较大的上下文对象。在容器初始化时，容器将自身加入到 Context 中：containerContext.put( PlexusConstants.PLEXUS_KEY, this )。













