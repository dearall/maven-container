## 第2章 Plexus IOC 容器接口及默认实现 ##

plexus-container-default 模块在其 org.codehaus.plexus 包定义了 **PlexusContainer** 接口，它是Plexus IOC 容器的核心接口，而 **MutablePlexusContainer** 接口继承自 PlexusContainer，并增加了改变容器配置的功能。Plexus 提供的默认实现是 **DefaultPlexusContainer** 类，它同样位于 org.codehaus.plexus 包，同时实现 PlexusContainer 和 MutablePlexusContainer 接口，并且还实现了一个简单的 **LogEnabled** 日志控制接口。


<br/><br/>
<a id="1"></a>

## 2.1 PlexusContainer 接口 ##

PlexusContainer 接口是加载和访问其它组件的入口点。

>JavaDoc: PlexusContainer is the entry-point for loading and accessing other components.

接口定义如下：

```java

package org.codehaus.plexus;

public interface PlexusContainer
{
    String ROLE = PlexusContainer.class.getName();

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
    Object lookup( String role, String roleHint )
        throws ComponentLookupException;

    /**
     * Looks up and returns a component object with the given unique key or role.
     * @param type the unique type of the component within the container
     * @param <T> The type.
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> T lookup( Class<T> type )
        throws ComponentLookupException;

    /**
     * Looks up and returns a component object with the given unique role/role-hint combination.
     * @param type the non-unique type of the component
     * @param roleHint a hint for the desired component implementation
     * @param <T> The type.
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> T lookup( Class<T> type, String roleHint )
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
    <T> T lookup( Class<T> type, String role, String roleHint )
        throws ComponentLookupException;

    /**
     * Looks up and returns a component object matching the given component descriptor.
     * @param componentDescriptor the descriptor of the component
     * @param <T> The type.
     * @return a Plexus component object
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> T lookup( ComponentDescriptor<T> componentDescriptor )
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
     * @param role a non-unique key for the desired components
     * @param roleHints the list of hints.
     * @return a List of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    List<Object> lookupList( String role, List<String> roleHints )
        throws ComponentLookupException;

    /**
     * Looks up and returns a List of component objects with the given role.
     * @param type the non-unique type of the components
     * @param <T> The type.
     * @return a List of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> List<T> lookupList( Class<T> type )
        throws ComponentLookupException;

    /**
     * Looks up and returns a List of component objects with the given role.
     * @param type the non-unique type of the components
     * @param roleHints the list of hints.
     * @param <T> The type.
     * @return a List of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> List<T> lookupList( Class<T> type, List<String> roleHints )
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
     * @param role a non-unique key for the desired components
     * @param roleHints the list of hints.
     * @return a Map of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    Map<String, Object> lookupMap( String role, List<String> roleHints )
        throws ComponentLookupException;

    /**
     * Looks up and returns a Map of component objects with the given role, keyed by all available role-hints.
     * @param type the non-unique type of the components
     * @param <T> The type.
     * @return a Map of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> Map<String, T> lookupMap( Class<T> type )
        throws ComponentLookupException;

    /**
     * Looks up and returns a Map of component objects with the given role, keyed by all available role-hints.
     * @param type the non-unique type of the components
     * @param roleHints the list of hints.
     * @param <T> The type.
     * @return a Map of component objects
     * @throws ComponentLookupException in case of lookup error.
     */
    <T> Map<String, T> lookupMap( Class<T> type, List<String> roleHints )
        throws ComponentLookupException;

    // ----------------------------------------------------------------------
    // Component Descriptor Lookup
    // ----------------------------------------------------------------------

    /**
     * Returns the ComponentDescriptor with the given component role and the default role hint.
     * Searches up the hierarchy until one is found, null if none is found.
     * @param role a unique role for the desired component's descriptor
     * @return the ComponentDescriptor with the given component role
     */
    ComponentDescriptor<?> getComponentDescriptor( String role );

    /**
     * Returns the ComponentDescriptor with the given component role and hint.
     * Searches up the hierarchy until one is found, null if none is found.
     * @param role a unique role for the desired component's descriptor
     * @param roleHint a hint showing which implementation should be used
     * @return the ComponentDescriptor with the given component role
     */
    ComponentDescriptor<?> getComponentDescriptor( String role, String roleHint );

    /**
     * Returns the ComponentDescriptor with the given component role and hint.
     * Searches up the hierarchy until one is found, null if none is found.
     * @param type the Java type of the desired component
     * @param role a unique role for the desired component's descriptor
     * @param roleHint a hint showing which implementation should be used
     * @param <T> The type.
     * @return the ComponentDescriptor with the given component role
     */
    <T> ComponentDescriptor<T> getComponentDescriptor( Class<T> type, String role, String roleHint );

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
    <T> List<ComponentDescriptor<T>> getComponentDescriptorList( Class<T> type,  String role );

    /**
     * Adds a component descriptor to this container. componentDescriptor should have realmId set.
     * @param componentDescriptor {@link ComponentDescriptor}
     * @throws CycleDetectedInComponentGraphException In case of an error.
     */
    void addComponentDescriptor( ComponentDescriptor<?> componentDescriptor )
        throws CycleDetectedInComponentGraphException;

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
    boolean hasComponent( String role, String roleHint );

    /**
     * Returns true if this container has a component with the given role/role-hint.
     * @param type the non-unique type of the component
     * @return true if this container has a component with the given role/role-hint
     */
    boolean hasComponent( Class<?> type );

    /**
     * Returns true if this container has a component with the given role/role-hint.
     * @param type the non-unique type of the component
     * @param roleHint a hint for the desired component implementation
     * @return true if this container has a component with the given role/role-hint
     */
    boolean hasComponent( Class<?> type, String roleHint );

    /**
     * Returns true if this container has a component with the given role/role-hint.
     * @param type the non-unique type of the component
     * @param role a non-unique key for the desired component
     * @param roleHint a hint for the desired component implementation
     * @return true if this container has a component with the given role/role-hint
     */
    boolean hasComponent( Class<?> type, String role, String roleHint );

    /**
     * Disposes of this container, which in turn disposes all of it's components. This container should also remove
     * itself from the container hierarchy.
     */
    void dispose();

    // ----------------------------------------------------------------------
    // Context
    // ----------------------------------------------------------------------

    /**
     * Add a key/value pair to this container's Context.
     * @param key any unique object valid to the Context's implementation
     * @param value any object valid to the Context's implementation
     */
    void addContextValue( Object key, Object value );

    /**
     * Returns this container's context. A Context is a simple data store used to hold values which may alter the
     * execution of the Container.
     * @return this container's context.
     */
    Context getContext();

    /**
     * Returns the Classworld's ClassRealm of this Container, which acts as the default parent for all contained
     * components.
     * @return the ClassRealm of this Container
     */
    ClassRealm getContainerRealm();

    // ----------------------------------------------------------------------
    // Discovery
    // ----------------------------------------------------------------------

    /**
     * Adds the listener to this container. ComponentDiscoveryListeners have the ability to respond to various
     * ComponentDiscoverer events.
     * @param listener A listener which responds to different ComponentDiscoveryEvents
     */
    void registerComponentDiscoveryListener( ComponentDiscoveryListener listener );

    /**
     * Removes the listener from this container.
     * @param listener A listener to remove
     */
    void removeComponentDiscoveryListener( ComponentDiscoveryListener listener );

    /**
     * Discovers components in the given realm.
     * @param childRealm {@link ClassRealm}
     * @return list {@link ComponentDescriptor}
     * @throws PlexusConfigurationException in case of an error.
     * @throws CycleDetectedInComponentGraphException in case of an error.
     */
    List<ComponentDescriptor<?>> discoverComponents( ClassRealm childRealm )
        throws PlexusConfigurationException, CycleDetectedInComponentGraphException;

    /**
     * Discovers components in the given realm.
     * @param realm the {@link ClassRealm}.
     * @param data The data.
     * @return list {@link ComponentDescriptor}
     * @throws PlexusConfigurationException in case of an error.
     * @throws CycleDetectedInComponentGraphException in case of an error.
     */
    List<ComponentDescriptor<?>> discoverComponents( ClassRealm realm, Object data )
        throws PlexusConfigurationException, CycleDetectedInComponentGraphException;

    // ----------------------------------------------------------------------------
    // Component/Plugin ClassRealm creation
    // ----------------------------------------------------------------------------

    ClassRealm createChildRealm( String id );

    ClassRealm getComponentRealm( String realmId );

    /**
     * Dissociate the realm with the specified id from the container. This will
     * remove all components contained in the realm from the component repository.
     *
     * @param componentRealm Realm to remove from the container.
     * @throws PlexusContainerException {@link PlexusContainerException}.
     */
    void removeComponentRealm( ClassRealm componentRealm )
        throws PlexusContainerException;

    /**
     * Returns the lookup realm for this container, which is either
     * the container realm or the realm set by {@link MutablePlexusContainer#setLookupRealm(ClassRealm)}.
     * @return {@link ClassRealm}
     */
    ClassRealm getLookupRealm();

    /**
     * Sets the lookup realm to use for lookup calls that don't have a ClassRealm parameter.
     * @param realm the new realm to use.
     * @return The previous lookup realm. It is advised to set it back once the old-style lookups have completed.
     */
    ClassRealm setLookupRealm(ClassRealm realm);

    /**
     * XXX ideally i'd like to place this in a plexus container specific utility class.
     *
     * Utility method to retrieve the lookup realm for a component instance.
     * If the component's classloader is a ClassRealm, that realm is returned,
     * otherwise the result of getLookupRealm is returned.
     * @param component The component.
     * @return {@link ClassRealm}
     */
    ClassRealm getLookupRealm( Object component );

    void addComponent( Object component, String role )
        throws CycleDetectedInComponentGraphException;

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
    <T> void addComponent( T component, Class<?> role, String roleHint );
}
```

注意到接口中定义了一个字符串型的 `String ROLE = PlexusContainer.class.getName()` 静态 final 变量，值为接口类名。这是 Plexus 中组件接口的约定，它实际上是 Plexus 组件角色的标识符。在上一章定义 Cheese 接口时，也以接口类名定义了这个字符串变量，如下所示：

```java
public interface Cheese
{
    /** The Plexus role identifier. */
    String ROLE = Cheese.class.getName();

    ...
}
```

PlexusContainer 容器是可嵌入的。也就是说，容器可以将其它容器作为组件加载到容器中，子容器也可以再加载孙容器，形成了容器树。因此容器本身也可以被其它容器当做组件进行管理，这就是 PlexusContainer 接口内定义 `String ROLE` 静态字段的原因，并且所有实现 PlexusContainer 接口的类都会自动继承这个静态字段的值。


PlexusContainer 接口定义了 Plexus 容器最核心的功能，可以将它的核心功能划分为如下几个方面：

- **组件查找 Lookup**
- **组件描述符查找 Component Descriptor Lookup**
- **释放组件 Releases the component from the container**
- **按查找条件判断容器是否能找到某个组件**
- **释放容器本身**
- **容器监听器管理**
- **组件发现 discoverComponents**
- **ClassRealm 管理**
- **将活动组件加入到容器管理**






<br/><br/>
<a id="2"></a>

## 2.2 MutablePlexusContainer 接口 ##

MutablePlexusContainer 是 PlexusContainer 的子接口，并增加了改变容器配置的功能。接口定义如下：

```java

package org.codehaus.plexus;

public interface MutablePlexusContainer
    extends PlexusContainer
{
    // Core Components

    ComponentRegistry getComponentRegistry();

    void setComponentRegistry( ComponentRegistry componentRegistry );

    ComponentDiscovererManager getComponentDiscovererManager();

    void setComponentDiscovererManager( ComponentDiscovererManager componentDiscovererManager );

    ComponentFactoryManager getComponentFactoryManager();

    void setComponentFactoryManager( ComponentFactoryManager componentFactoryManager );

    LoggerManager getLoggerManager();

    void setLoggerManager( LoggerManager loggerManager );

    Logger getLogger();
    
    void setConfigurationSource( ConfigurationSource configurationSource );

    ConfigurationSource getConfigurationSource();
    
    // Configuration

    void setConfiguration( PlexusConfiguration configuration );

    PlexusConfiguration getConfiguration();

    ClassRealm getComponentRealm( String realmId );

    ClassWorld getClassWorld();
}

```

MutablePlexusContainer 接口定义的功能就是对 Plexus 容器的几个重要管理组件进行配置和获取，通过这些方法可动态改变容器的某些方面的管理，使容器本身具有易变性。





<br/><br/>
<a id="3"></a>

## 2.3 LogEnabled 接口 ##

LogEnabled 接口非常简单，只有一个方法，如下所示：

```java

package org.codehaus.plexus.logging;

public interface LogEnabled
{
    void enableLogging( Logger logger );
}

```

给容器配置日志实例。




<br/><br/>
<a id="4"></a>

## 2.4 Plexus 容器的默认实现 DefaultPlexusContainer 类 ##

DefaultPlexusContainer 是 Plexus 为容器接口提供的默认实现，并且是唯一的实现。Plexus 中所有容器的操作都从创建 DefaultPlexusContainer 的实例开始，例如上一章 App 类的 main 方法：

```java
public class App {
    public static void main(String args[]) throws Exception {
        // 1  定义一个容器，容器会去加载 classpath 下的 META-INF/Plexus/component.xml 中的组件
        PlexusContainer container= new DefaultPlexusContainer();

        ...
    }
```

DefaultPlexusContainer 类从 AbstractLogEnabled 类继承，因此是通过 AbstractLogEnabled 类实现 LogEnabled 接口的。它同时实现 PlexusContainer 和 MutablePlexusContainer 接口。定义如下：

```java
public class DefaultPlexusContainer
    extends AbstractLogEnabled
    implements MutablePlexusContainer
{
    protected static final String DEFAULT_CONTAINER_NAME = "default";

    protected static final String DEFAULT_REALM_NAME = "plexus.core";

    /**
     * Arbitrary data associated with the container.  Data in the container has highest precedence when configuring
     * a component to create.
     */
    protected Context containerContext;

    protected PlexusConfiguration configuration;

    // todo: don't use a reader
    protected Reader configurationReader;

    protected ClassWorld classWorld;

    protected ClassRealm containerRealm;

    // ----------------------------------------------------------------------------
    // Core components
    // ----------------------------------------------------------------------------

    private ComponentRegistry componentRegistry;

    /**
     * Simple index (registry) of ComponentDiscovers and ComponentDiscoveryListener.
     */
    protected ComponentDiscovererManager componentDiscovererManager;

    /**
     * Trivial class to look-up ComponentFactory instances in this container.
     */
    protected ComponentFactoryManager componentFactoryManager;

    /**
     * Generic logger interface.
     */
    protected LoggerManager loggerManager;

    /**
     * Converts a ComponentDescriptor into PlexusConfiguration.
     */
    protected ConfigurationSource configurationSource;

    // ----------------------------------------------------------------------------
    //
    // ----------------------------------------------------------------------------

    // TODO: Is there a more threadpool-friendly way to do this?
    private ThreadLocal<ClassRealm> lookupRealm = new ThreadLocal<ClassRealm>();


    // ----------------------------------------------------------------------
    // Constructors
    // ----------------------------------------------------------------------

    public DefaultPlexusContainer()
        throws PlexusContainerException
    {
        construct( new DefaultContainerConfiguration() );
    }

    public DefaultPlexusContainer( ContainerConfiguration c )
        throws PlexusContainerException
    {
        construct( c );
    }

    // methods
    ...

    ...
}

```

DefaultPlexusContainer 内部通过几个组件来管理其它组件的创建和管理工作，如代码所示。


<br/><br/>

#### <font size=4 color=green><b>DefaultPlexusContainer 构造器</b></font> ####

DefaultPlexusContainer 提供了两个构造器：

- **DefaultPlexusContainer(ContainerConfiguration c)**：通过提供的自定义配置 ContainerConfiguration 实例创建容器。
- **DefaultPlexusContainer()** ：通过 ContainerConfiguration 的默认实现 DefaultContainerConfiguration 创建容器。


创建 DefaultPlexusContainer 容器实例，需要一个 ContainerConfiguration 实例，通过该实例完成容器的初始化。容器并没有一直持有 PlexusConfiguration 实例，初始化结束之后，该实例在容器中就不存在了。

PlexusConfiguration 由 DefaultContainerConfiguration 提供默认的实现。其中提供了为容器配置内部组件的设置和获取信息的方法，与 MutablePlexusContainer 接口中新增的单个配置功能差不多。PlexusConfiguration 一次性配置容器内所有组件，而 MutablePlexusContainer 接口中新增的方法可以进行单项配置。默认的 DefaultContainerConfiguration 实现提供了容器的默认配置信息，参见《Plexus IOC 容器配置接口及其默认实现》的阐述。容器的初始化过程由 DefaultPlexusContainer 内部的 `construct( ContainerConfiguration c )` 方法完成，如下代码清单所示：

```java

private void construct( ContainerConfiguration c )
    throws PlexusContainerException
{
    configurationSource = c.getConfigurationSource();

    // ----------------------------------------------------------------------------
    // ClassWorld
    // ----------------------------------------------------------------------------

    classWorld = c.getClassWorld();

    // Make sure we have a valid ClassWorld
    if ( classWorld == null )
    {
        classWorld = new ClassWorld( DEFAULT_REALM_NAME, Thread.currentThread().getContextClassLoader() );
    }

    containerRealm = c.getRealm();

    if ( containerRealm == null )
    {
        try
        {
            containerRealm = classWorld.getRealm( DEFAULT_REALM_NAME );
        }
        catch ( NoSuchRealmException e )
        {
            containerRealm = (ClassRealm) classWorld.getRealms().iterator().next();

            if ( containerRealm == null )
            {
                System.err.println( "No container realm! Expect errors." );

                new Throwable().printStackTrace();
            }
        }
    }

    setLookupRealm( containerRealm );

    // ----------------------------------------------------------------------------
    // Context
    // ----------------------------------------------------------------------------

    if ( c.getContext() != null )
    {
        containerContext = new DefaultContext( c.getContext() );            
    }
    else
    {                
        containerContext = new DefaultContext();
    }

    // ----------------------------------------------------------------------------
    // Configuration
    // ----------------------------------------------------------------------------

    InputStream in = null;

    if ( c.getContainerConfiguration() != null )
    {
        in = toStream( c.getContainerConfiguration() );
    }

    try
    {
        if ( c.getContainerConfigurationURL() != null )
        {
            in = c.getContainerConfigurationURL().openStream();
        }
    }
    catch ( IOException e )
    {
        throw new PlexusContainerException( "Error reading configuration URL", e );
    }

    try
    {
        configurationReader = in == null ? null : ReaderFactory.newXmlReader( in );
    }
    catch ( IOException e )
    {
        throw new PlexusContainerException( "Error reading configuration file", e );
    }

    try
    {
        initialize( c );

        start();
    }
    finally
    {
        IOUtil.close( configurationReader );
    }
    
    for( Class clazz : c.getComponentDiscoverers() )
    {
        try
        {
            ComponentDiscoverer cd = (ComponentDiscoverer) lookup( clazz );
            componentDiscovererManager.addComponentDiscoverer( cd );
        }
        catch ( ComponentLookupException e )
        {
        }
    }
    
    for( Class clazz : c.getComponentDiscoveryListeners() )
    {
        try
        {
            ComponentDiscoveryListener cdl = (ComponentDiscoveryListener) lookup( clazz );
            componentDiscovererManager.registerComponentDiscoveryListener( cdl );
        }
        catch ( ComponentLookupException e )
        {
        }
    }                
}
```




<br/><br/>

#### <font size=4 color=green><b>DefaultPlexusContainer 静态字段定义</b></font> ####

DefaultPlexusContainer 通过静态 String 型字段 DEFAULT_CONTAINER_NAME 定义了容器的默认名称为 "default"，通过静态 String 字段 DEFAULT_REALM_NAME 定义了默认的领域类加载器 realmId 为 "plexus.core"，如下所示：

```java
    protected static final String DEFAULT_CONTAINER_NAME = "default";
    protected static final String DEFAULT_REALM_NAME = "plexus.core";
```


<br/><br/>

#### <font size=4 color=green><b>Context containerContext</b></font> ####

Context 是一个接口，由 DefaultContext 提供默认实现。表示为一个 Map 型的数据结构，用于存储与容器相关联的任意的数据（arbitrary data）。在配置一个组件的创建时，容器中的数据具有最高的优先级。

Context 通过 `put(Object key, Object value)` 方法向 Context 中存入数据，并通过 `get(Object key)` 方法从其中取回数据。DefaultPlexusContainer 在其内部初始化时，通过 `initialize( ContainerConfiguration containerConfiguration )` 方法，将容器本身实例加入到 Context 上下文环境中：

```java
containerContext.put( PlexusConstants.PLEXUS_KEY, this );
```

其中，PlexusConstants.PLEXUS_KEY 常量字符串值为 "plexus"。


<br/><br/>

#### <font size=4 color=green><b>PlexusConfiguration configuration</b></font> ####

PlexusConfiguration 是一个接口，为 plexus 配置层次结构数据。DefaultPlexusContainer 使用 XmlPlexusConfiguration 作为该配置接口的实例。通过 **META-INF/plexus/plexus.xml** 文件对 plexus 容器本身的数据进行配置，与 ContainerConfiguration 表示的内部管理组件的配置不同。 

configuration 通过 `initializeConfiguration( ContainerConfiguration c )` 方法初始化，并一直由容器持有该实例：

```java
protected void initializeConfiguration( ContainerConfiguration c )
    throws PlexusConfigurationException, ContextException, IOException
{
    // We need an empty plexus configuration for merging. This is a function of removing the
    // plexus-boostrap.xml file.
    configuration = new XmlPlexusConfiguration( "plexus" );

    if ( configurationReader != null )
    {
        // User userConfiguration

        PlexusConfiguration userConfiguration = PlexusTools.buildConfiguration( "<User Specified Configuration Reader>", getInterpolationConfigurationReader( configurationReader ) );

        // Merger of bootstrapConfiguration and user userConfiguration

        configuration = PlexusConfigurationMerger.merge( userConfiguration, configuration );
    }
}
```




<br/><br/>
#### <font size=4 color=green><b>Reader configurationReader</b></font> ####

configurationReader 是 DefaultPlexusContainer 容器读取配置文件的读取器。从 `construct( ContainerConfiguration c )` 代码中可以看到，默认容器可以从 ContainerConfiguration 的资源配置文件或 URL 中读取组件配置信息。默认配置的 DefaultPlexusContainer 并没有真正用到这个 Reader，而是通过 DefaultComponentDiscoverer 的 `String getComponentDescriptorLocation()` 方法获得默认的组件配置文件，以读取组件配置信息：

```java
public class DefaultComponentDiscoverer
    extends AbstractResourceBasedComponentDiscoverer
{
    public String getComponentDescriptorLocation()
    {
        return "META-INF/plexus/components.xml";
    }
}
```

这就是 Plexus 容器默认实现用到的组件配置文件的位置。


<br/><br/>
#### <font size=4 color=green><b>ClassWorld classWorld 和 ClassRealm containerRealm</b></font> ####

ClassWorld 来自于 Plexus 项目的另一个子项目 plexus-classworlds，详细信息参考第四章的论述。

classWorld 字段是容器的领域类加载器 ClassRealm 的管理器，它管理容器中所有的领域类加载器，包括构造 DefaultPlexusContainer 时，创建了以 DEFAULT_REALM_NAME 为 id，Thread.currentThread().getContextClassLoader() 为基本类加载器的领域类加载器，并把该实例作为容器的领域类加载器 containerRealm 的值。也把它存储到线程本地变量 ThreadLocal<ClassRealm> lookupRealm 中。



<br/><br/>
<a id="41"></a>

## 2.4.1 DefaultPlexusContainer 的核心组件 ##


<br/><br/>
#### <font size=4 color=green><b>ComponentRegistry componentRegistry</b></font> ####

组件注册中心 ComponentRegistry 是 DefaultPlexusContainer 的组件管理中心，负责组件添加、移除以及查找等组件管理操作。Plexus 提供的默认实现是 DefaultComponentRegistry 类，其内部持有对容器 DefaultPlexusContainer 的引用，从 DefaultContainerConfiguration 获取的组件仓库 ComponentRepository 和生命周期处理器管理器 LifecycleHandlerManager 实例。

组件中心 componentRegistry 由 InitializeComponentRegistryPhase 类执行初始化。如下所示：

```java
public class InitializeComponentRegistryPhase 
    implements ContainerInitializationPhase
{
    public void execute( ContainerInitializationContext context )
        throws ContainerInitializationException
    {
        ComponentRepository repository = getComponentRepository( context );

        LifecycleHandlerManager lifecycleHandlerManager = getLifecycleHandlerManager( context );

        ComponentRegistry componentRegistry = new DefaultComponentRegistry( context.getContainer(),
            repository,
            lifecycleHandlerManager );

        componentRegistry.registerComponentManagerFactory( new PerLookupComponentManagerFactory() );

        componentRegistry.registerComponentManagerFactory( new SingletonComponentManagerFactory() );

        context.getContainer().setComponentRegistry( componentRegistry );
    }
```




<br/><br/>
#### <font size=4 color=green><b>ComponentDiscovererManager componentDiscovererManager</b></font> ####

ComponentDiscovererManager 对组件发现器 ComponentDiscoverer 和发现监听器的简单管理，默认实现类DefaultComponentDiscovererManager 类。主要在容器启动时，执行组件发现操作，具体发现操作由其所管理的 ComponentDiscoverer 对象执行 `findComponents( Context context, ClassRealm classRealm )`。ComponentDiscovererManager 由 InitializeComponentDiscovererManagerPhase 执行初始化，如下所示：

```java
public class InitializeComponentDiscovererManagerPhase
    extends AbstractCoreComponentInitializationPhase
{
    public void initializeCoreComponent( ContainerInitializationContext context )
        throws ContainerInitializationException
    {
        ComponentDiscovererManager componentDiscovererManager = context.getContainerConfiguration().getComponentDiscovererManager();

        context.getContainer().setComponentDiscovererManager( componentDiscovererManager );

        for ( ComponentDiscoveryListener listener : componentDiscovererManager.getComponentDiscoveryListeners().keySet() )
        {
            try
            {
                // This is a hack until we have completely live components

                context.getContainer().addComponent( listener, listener.getClass().getName() );

                context.getContainer().getComponentDiscovererManager().removeComponentDiscoveryListener( listener );

                ComponentDiscoveryListener cdl = context.getContainer().lookup( listener.getClass() );

                context.getContainer().getComponentDiscovererManager().registerComponentDiscoveryListener( cdl );
            }
            catch ( Exception e )
            {
                throw new ContainerInitializationException( "Error initializing component discovery listener.", e );
            }
        }
    }
}

```

其中 `ComponentDiscovererManager componentDiscovererManager = context.getContainerConfiguration().getComponentDiscovererManager()` 已由 DefaultContainerConfiguration 将两个初始的组件发现器实例：**DefaultComponentDiscoverer** 对象和 **PlexusXmlComponentDiscoverer** 对象加入到返回的 componentDiscovererManager 管理器，如下所示：

```java
    public ComponentDiscovererManager getComponentDiscovererManager()
    {
        if ( componentDiscovererManager == null )
        {
            componentDiscovererManager = new DefaultComponentDiscovererManager();

            ((DefaultComponentDiscovererManager)componentDiscovererManager).addComponentDiscoverer( new DefaultComponentDiscoverer() );

            ((DefaultComponentDiscovererManager)componentDiscovererManager).addComponentDiscoverer( new PlexusXmlComponentDiscoverer() );
        }

        return componentDiscovererManager;
    }
```

随后的初始化操作，会调用 `discoverComponents( getContainerRealm() );` 通过 ComponentDiscovererManager 发现应用配置的组件。而 DefaultComponentDiscoverer 和 PlexusXmlComponentDiscoverer 实例会读取 **META-INF/plexus/components.xml** 组件配置文件，以创建所有配置组件的组件描述符 ComponentDescriptor 实例，并保存到组件管理中心 componentRegistry 的组件仓库 repository 中，以备后用。




<br/><br/>
#### <font size=4 color=green><b>ComponentFactoryManager componentFactoryManager</b></font> ####

简单的组件工厂管理器，用于在本容器中查找指定 id 的组件工厂。使用默认实现的 DefaultComponentFactoryManager 类。通过 InitializeComponentFactoryManagerPhase 类初始化，如下所示：

```java
public class InitializeComponentFactoryManagerPhase
    extends AbstractCoreComponentInitializationPhase
{
    public void initializeCoreComponent( ContainerInitializationContext context )
        throws ContainerInitializationException
    {
        ComponentFactoryManager componentFactoryManager = context.getContainerConfiguration().getComponentFactoryManager();

        if ( componentFactoryManager instanceof Contextualizable )
        {
            //TODO: this is wrong here jvz.
            context.getContainer().getContext().put( PlexusConstants.PLEXUS_KEY, context.getContainer() );

            try
            {
                ( (Contextualizable) componentFactoryManager).contextualize( context.getContainer().getContext() );
            }
            catch ( ContextException e )
            {
                throw new ContainerInitializationException( "Error contextualization component factory manager.", e );
            }
        }

        context.getContainer().setComponentFactoryManager( componentFactoryManager );
    }
}
```




<br/><br/>
#### <font size=4 color=green><b>LoggerManager loggerManager</b></font> ####

容器的日志管理器接口，由 ConsoleLoggerManager 类提供默认实现。通过 InitializeLoggerManagerPhase 初始化，如下代码所示：

```java
public class InitializeLoggerManagerPhase
    extends AbstractCoreComponentInitializationPhase
{
    public void initializeCoreComponent( ContainerInitializationContext context )
        throws ContainerInitializationException
    {
        LoggerManager loggerManager = context.getContainer().getLoggerManager();

        // ----------------------------------------------------------------------
        // The logger manager may have been set programmatically so we need
        // to check. If it hasn't then we will try to look up a logger manager
        // that may have been specified in the plexus.xml file. If that doesn't
        // work then we'll programmatcially stuff in the console logger.
        // ----------------------------------------------------------------------

        if ( loggerManager == null )
        {
            try
            {
                loggerManager = context.getContainer().lookup( LoggerManager.class );
            }
            catch ( ComponentLookupException e )
            {
                ComponentDescriptor cd = new ComponentDescriptor();

                cd.setRole( LoggerManager.ROLE );

                cd.setRoleHint( PlexusConstants.PLEXUS_DEFAULT_HINT );

                cd.setImplementation( ConsoleLoggerManager.class.getName() );

                try
                {
                    context.getContainer().addComponentDescriptor( cd );
                }
                catch ( CycleDetectedInComponentGraphException cre )
                {
                    throw new ContainerInitializationException( "Error setting up logging manager.", cre );
                }

                loggerManager = new ConsoleLoggerManager( "info" );
            }

            context.getContainer().setLoggerManager( loggerManager );
        }

        Logger logger = loggerManager.getLoggerForComponent( PlexusContainer.class.getName() );

        context.getContainer().enableLogging( logger );
    }
}

```



<br/><br/>
#### <font size=4 color=green><b>ConfigurationSource configurationSource</b></font> ####

一个转换器，可以将一个组件描述符 ComponentDescriptor 转换为 PlexusConfiguration 对象，默认使用 ChainedConfigurationSource 实现。通过 InitializeUserConfigurationSourcePhase 初始化，如下代码所示：

```java
public class InitializeUserConfigurationSourcePhase
    extends AbstractCoreComponentInitializationPhase
{
    public void initializeCoreComponent( ContainerInitializationContext context )
        throws ContainerInitializationException
    {
        ConfigurationSource existingConfigurationSource = context.getContainer().getConfigurationSource();

        try
        {
            // is the user overriding the ConfigurationSource (role-hint: default) or only extending it?
            if ( context.getContainer().hasComponent( ConfigurationSource.class, PlexusConstants.PLEXUS_DEFAULT_HINT ) )
            {
                // overriding

                ConfigurationSource cs = context.getContainer().lookup( ConfigurationSource.class );

                // swap the user provided configuration source with current one
                context.getContainer().setConfigurationSource( cs );
            }
            else
            {
                // extending
                List<ConfigurationSource> userConfigurationSources = context.getContainer().lookupList( ConfigurationSource.class );

                if ( userConfigurationSources.size() > 0 )
                {
                    List<ConfigurationSource> configurationSources =
                        new ArrayList<ConfigurationSource>( userConfigurationSources.size() + 1 );

                    // adding user provied ones to be able to interfere
                    configurationSources.addAll( userConfigurationSources );

                    // at the end adding the container source, to make sure config will be returned
                    // from plexus.xml if no user interference is given
                    configurationSources.add( existingConfigurationSource );

                    ConfigurationSource configurationSource = new ChainedConfigurationSource( configurationSources );

                    context.getContainer().setConfigurationSource( configurationSource );

                }

                // register the default source, either the chained or the existing one as default
                ComponentDescriptor<ConfigurationSource> cd = new ComponentDescriptor<ConfigurationSource>();

                cd.setRole( ConfigurationSource.ROLE );

                cd.setRoleHint( PlexusConstants.PLEXUS_DEFAULT_HINT );

                cd.setImplementationClass( context.getContainer().getConfigurationSource().getClass() );

                try
                {
                    context.getContainer().addComponentDescriptor( cd );
                }
                catch ( CycleDetectedInComponentGraphException cre )
                {
                    throw new ContainerInitializationException( "Error setting up configuration source.", cre );
                }
            }

        }
        catch ( ComponentLookupException e )
        {
            throw new ContainerInitializationException( "Error setting up user configuration source.", e );
        }
    }
}

```





<br/><br/>
<a id="42"></a>

## 2.4.2 DefaultPlexusContainer 公共方法阐释 ##



<br/><br/>
#### <font size=4 color=green><b>将现有组件加入容器管理</b></font> ####

将现有组件加入到容器管理：是将以其它方式创建的组件加入到 Plexus 容器管理。组件被加入到组件注册中心的非托管 Map 集合 `Map<Key, Object> unmanagedComponents` 中。非托管组件与容器托管组件的重要区别是，该 Map 中 Key 的领域类加载器 realm 的值为 null。在使用容器的 lookup() 系列方法查找或创建组件对象时，会首先查找非托管的 unmanagedComponents 集合，如果找到直接返回。只有在 unmanagedComponents 中没有找到相应的组件实例时，才继续由组件注册中心查找和创建管理组件。

- **void addComponent(Object component, String role, String roleHint)**：
  component ：为组件实例对象
  role ：为组件角色，一般使用该组件接口或基类的全限定类名
  roleHint ：为角色区别标识，当多个类实现同一接口时，使用该字符串识别不同的实现类
<br />

- **void addComponent(Object component, String role)**：使用常量 PLEXUS_DEFAULT_HINT 作为 roleHint 调用第一个方法，PLEXUS_DEFAULT_HINT 值为 "default"。
<br />

- **<T> void addComponent(T component, Class<?> role, String roleHint)**：通过 role.getName() 获取角色字符串，调用第一个方法。

<br />

- **void addComponentDescriptor(ComponentDescriptor<?> componentDescriptor)**：将组件描述符加入到容器组件注册中心管理。
  componentDescriptor：为组件描述符对象，如果该描述符内的领域类加载器 ClassRealm 为 null，则将容器的领域类加载器 containerRealm 设置为该组件描述符的领域类加载器。
  



<br/><br/>
#### <font size=4 color=green><b>组件查找 Lookup</b></font> ####

通过 Lookup() 系列方法获取组件实例：

Lookup() 系列方法，通过组件注册中心，通过给定的角色 role 以及角色提示 roleHint 的值，首先查找非托管组件集合，即由 addComponent() 方法添加到容器中的非托管组件集合，如果找到直接返回。如果在非托管组件集合中没有找到对应的组件，下一步则找到或创建组件管理器 `ComponentManager<T>`。根据组件的配置不同，确定其是单例组件还是非单例组件，得到不同的组件管理器实现：SingletonComponentManager 或 PerLookupComponentManager 实例。如果组件描述文件中没有为 `<instantiation-strategy>` 元素指定值，使用默认的 DEFAULT_INSTANTIATION_STRATEGY 值为 "singleton"。该元素可以指定为 "per-lookup", "singleton", "keep-alive" 或者 "poolable"。由具体的组件管理器返回或者创建组件实例。如果组件实例还没有创建，则使用 AbstractComponentManager 中的 `ComponentBuilder<T> builder = new XBeanComponentBuilder<T>(this)` 实例，通过 `XBeanComponentBuilder.build(ComponentDescriptor<T> descriptor, ClassRealm realm, ComponentBuildListener listener)` 方法，由 `JavaComponentFactory.newInstance( ComponentDescriptor componentDescriptor, ClassRealm classRealm, PlexusContainer container )` 方法创建。该方法在内部通过调用 `Object instance = implementationClass.newInstance()` 创建组件对象，最后返回给 lookup() 的调用者。

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

- **\<T\> T 	lookup(ComponentDescriptor\<T\> componentDescriptor)**：通过所提供的组件描述符查找或创建组件对象。
  componentDescriptor：查找或创建组件对象的组件描述符
<br />

- **\<T\> List\<T\> 	lookupList(Class\<T\> type)**：获取或创建全部实现 type 接口的组件对象集合。
  type ：要查找或创建组件的接口或基类类型
<br />

- **\<T\> List\<T\> 	lookupList(Class\<T\> type, List\<String\> roleHints)**：获取或创建实现 type 接口，指定 roleHint 的组件对象集合。
  type ：要查找或创建组件的接口或基类类型
  roleHints：指定返回 roleHints 中指定的组件对象
<br />

- **List\<Object\> 	lookupList(String role)**：获取或创建全部实现或继承自 role 表明的组件对象集合。
  role ：为组件角色，一般使用该组件接口或基类的全限定类名
<br />

- **List\<Object\> 	lookupList(String role, List\<String\> roleHints)** ：获取或创建 roleHints 指明的，实现或继承自 role 表明的组件对象集合。
  role ：为组件角色，一般使用该组件接口或基类的全限定类名
  roleHints：指定返回 roleHints 中指定的组件对象

<br />

- **\<T\> Map\<String,T\> 	lookupMap(Class\<T\> type)**：获取或创建全部实现 type 接口的组件对象集合。
  type ：要查找或创建组件的接口或基类类型
  返回值：类型为 Map\<String,T\>，其中 key 为 roleHint，值 T 为与 roleHint 对应的组件对象

<br />

- **\<T\> Map\<String,T\> 	lookupMap(Class\<T\> type, List\<String\> roleHints)**：获取或创建指定 roleHints 的实现 type 接口的组件对象集合。
  type ：要查找或创建组件的接口或基类类型
  roleHints：返回 roleHints 中指定的组件对象
  返回值：类型为 Map\<String,T\>，其中 key 为 roleHint，值 T 为与 roleHint 对应的组件对象

<br />

- **Map\<String,Object\> 	lookupMap(String role)**：获取或创建全部实现 role 指明接口或基类的组件对象集合。
  role ：为组件角色，一般使用该组件接口或基类的全限定类名
  返回值：类型为 Map\<String,T\>，  其中 key 为 roleHint，值 T 为与 roleHint 对应的组件对象

<br />

- **Map\<String,Object\> 	lookupMap(String role, List\<String\> roleHints)**：获取或创建指定 roleHints 的，实现 role 指明接口或基类的组件对象集合。
  role ：为组件角色，一般使用该组件接口或基类的全限定类名
  roleHints：返回 roleHints 中指定的组件对象
  返回值：类型为 Map\<String,T\>，其中 key 为 roleHint，值 T 为与 roleHint 对应的组件对象




<br/><br/>
#### <font size=4 color=green><b>组件描述符查找 Component Descriptor Lookup</b></font> ####


- **ComponentDescriptor\<?\> getComponentDescriptor( String role )**：根据给定的组件 role 和默认 roleHint 查找组件描述符，默认 roleHint 为 ""。
  role ：组件角色，一般使用该组件接口或基类的全限定类名
<br />

- **ComponentDescriptor\<?\> getComponentDescriptor( String role, String roleHint )**：根据给定的组件 role 和 roleHint 查找组件描述符。
  role ：组件角色，一般使用该组件接口或基类的全限定类名
  roleHint ：角色区别标识，当多个类实现同一 role 指明的接口，或多个子类继承同一 role 指明的基类，使用该字符串识别不同的实现类
<br />

- **\<T\> ComponentDescriptor\<T\> getComponentDescriptor( Class\<T\> type, String role, String roleHint )**：根据给定的组件 type、role 和 roleHint 查找组件描述符。
  type ：要查找或创建组件的接口或基类类型
  role ：为组件角色，一般使用该组件接口或基类的全限定类名。在查找时，通过判断 **if ( isAssignableFrom( type, implClass ) || Object.class == implClass && role.equals( type.getName() ) )** 是否为 true，表明 type 与 role 的相关性
  roleHint ：角色区别标识，当多个类实现同一 role 指明的接口，或多个子类继承同一 role 指明的基类，使用该字符串识别不同的实现类
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
<br /> 

- **List\<ComponentDescriptor\<?\>\> getComponentDescriptorList( String role )**：在容器中查找全部符合 role 的组件描述符。
  role ：组件角色，一般使用该组件接口或基类的全限定类名
  返回值：类型为 List\<ComponentDescriptor\<T\>\>，表示所有找到符合条件的组件描述符。如果没有找到，返回空列表
<br /> 

- **<T> List\<ComponentDescriptor\<T\>\> getComponentDescriptorList( Class\<T\> type,  String role )**：在容器中查找全部符合 type 和 role 的组件描述符。
  type ：要查找组件的接口或基类类型
  role ：为组件角色，一般使用该组件接口或基类的全限定类名。在查找时，通过判断 **if ( isAssignableFrom( type, implClass ) || Object.class == implClass && role.equals( type.getName() ) )** 是否为 true，表明 type 与 role 的相关性
  返回值：类型为 List\<ComponentDescriptor\<T\>\>，表示所有找到符合条件的组件描述符。如果没有找到，返回空列表




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

容器的组件发现管理器 ComponentDiscovererManager componentDiscovererManager，是 Plexus 核心组件之一，由 DefaultComponentDiscovererManager 类提供默认实现。初始化时，组件发现管理器管理两个初始的组件发现器：**DefaultComponentDiscoverer** 和 **PlexusXmlComponentDiscoverer**。在容器启动时，组件发现管理器执行组件发现操作。具体发现操作由其所管理的两个 ComponentDiscoverer 对象执行 `findComponents( Context context, ClassRealm classRealm )`方法，PlexusXmlComponentDiscoverer 读取 **META-INF/plexus/plexus.xml** 文件，DefaultComponentDiscoverer 读取 **META-INF/plexus/components.xml** 文件，通过读取两个配置文件中 \<components\> 元素内的组件配置信息，创建全部组件的组件描述符 componentSetDescriptor 对象，保存到组件祖册中心组件 ComponentRegistry componentRegistry 中。在组件发现过程中，组件发现管理器 ComponentDiscovererManager 会为每一个发现的组件描述符，触发组件发现事件，组件侦听器 ComponentDiscoveryListener 用于接收该事件。


- **void registerComponentDiscoveryListener( ComponentDiscoveryListener listener )**：为组件发现管理器安装侦听器
<br/>

- **void removeComponentDiscoveryListener( ComponentDiscoveryListener listener )**：将某个具体的组件发现侦听器从组件发现管理器上移除

监听器接口定义：

```java
public interface ComponentDiscoveryListener
{
    /**
     * Signals to this listener that a component has been discovered.
     * @param event the event that signals what components have been discovered
     */
    void componentDiscovered( ComponentDiscoveryEvent event );
}

```

<br/>

- **List\<ComponentDescriptor\<?\>\> discoverComponents( ClassRealm childRealm )**：使用给定的 ClassRealm 发现为容器配置的组件，即通过 META-INF/plexus/components.xml 文件配置的组件。
  childRealm：是加载组件实现类的领域类加载器 ClassRealm，Plexus 容器默认使用容器的 containerRealm 加载发现的组件

- **List\<ComponentDescriptor\<?\>\> discoverComponents( ClassRealm realm, Object data )**：使用给定的 ClassRealm 发现为容器配置的组件，即通过 META-INF/plexus/components.xml 文件配置的组件。
  childRealm：是加载组件实现类的领域类加载器 ClassRealm，Plexus 容器默认使用容器的 containerRealm 加载发现的组件
  data：附加数据，用于发现组件时，触发 ComponentDiscoveryListener 侦听器的 componentDiscovered( ComponentDiscoveryEvent event ) 调用，该附加数据 data 作为 ComponentDiscoveryEvent 事件对象的一部分，发送给监听器，监听器可以通过 ComponentDiscoveryEvent 对象接收到该 data 数据。
  



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

- **ClassRealm getComponentRealm( String realmId )**：获取指定 realmId 的组件领域类加载器。如果在容器的 classWorld 找到指定 realmId 对应的 ClassRealm，直接返回。否则返回容器本身的 containerRealm。

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


- **void addContextValue( Object key, Object value )**：向容器的 Context 对象添加数据，key-value 对。内部通过 containerContext.put( key, value ) 将输入存储到 Context 对象。




