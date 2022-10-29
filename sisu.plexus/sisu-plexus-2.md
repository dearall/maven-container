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

2. sisu.plexus 定义的 PlexusContainer 接口移除了 **\<T\> T lookup( ComponentDescriptor\<T\> componentDescriptor )** 方法

3. sisu.plexus 定义的 PlexusContainer 接口移除了 **List\<Object\> lookupList( String role, List\<String\> roleHints )** 方法

4. sisu.plexus 定义的 PlexusContainer 接口移除了 **\<T\> List\<T\> lookupList( Class\<T\> type, List\<String\> roleHints )** 方法

5. sisu.plexus 定义的 PlexusContainer 接口移除了 **Map\<String, Object\> lookupMap( String role, List\<String\> roleHints )** 方法

6. sisu.plexus 定义的 PlexusContainer 接口移除了 **\<T\> Map\<String, T\> lookupMap( Class\<T\> type, List\<String\> roleHints )** 方法

7. sisu.plexus 定义的 PlexusContainer 接口移除了 **ComponentDescriptor\<?\> getComponentDescriptor( String role )** 方法

8. sisu.plexus 定义的 PlexusContainer 接口移除了 **List<ComponentDescriptor\<?\>\> discoverComponents( ClassRealm realm, Object data )** 方法

9. sisu.plexus 定义的 PlexusContainer 接口移除了 **ClassRealm getComponentRealm( String realmId )** 方法


看得出，sisu.plexus 的 PlexusContainer 接口定义，相当于传统 Plexus PlexusContainer 接口定义的一个子集，没有增加额外的方法。







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
}

```

