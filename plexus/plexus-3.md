## 第3章 Plexus IOC 容器配置接口及其默认实现 ##

Plexus 容器的默认实现 DefaultPlexusContainer 类通过容器配置接口 ContainerConfiguration 创建实例。




<br/><br/>
<a id="1"></a>

## 3.1 ContainerConfiguration 接口 ##

容器配置接口 ContainerConfiguration 定义如下：

```java

package org.codehaus.plexus;

public interface ContainerConfiguration
{
    ContainerConfiguration setName( String name );

    String getName();

    ContainerConfiguration setContext( Map<Object, Object> context );

    Map<Object, Object> getContext();

    ContainerConfiguration setClassWorld( ClassWorld classWorld );

    ClassWorld getClassWorld();

    ContainerConfiguration setRealm( ClassRealm realm );

    ClassRealm getRealm();
    
    //
    // Configuration
    //
    ContainerConfiguration setContainerConfiguration( String configuration );

    String getContainerConfiguration();

    ContainerConfiguration setContainerConfigurationURL( URL configuration );

    URL getContainerConfigurationURL();

    // Programmatic Container Initialization and Setup

    // Much of this setup and initialization can be completely hidden. It's probably not likely
    // someone will need to change these core components, but rather adding things like different
    // factories, and component managers.

    // Container initialization phases

    ContainerInitializationPhase[] getInitializationPhases();

    // Component lookup manager

    // Component discoverer manager

    ContainerConfiguration addComponentDiscoverer( ComponentDiscoverer componentDiscoverer );

    ContainerConfiguration addComponentDiscoveryListener( ComponentDiscoveryListener componentDiscoveryListener );

    ContainerConfiguration setComponentDiscovererManager( ComponentDiscovererManager componentDiscovererManager );

    ComponentDiscovererManager getComponentDiscovererManager();

    // Component factory manager

    ContainerConfiguration setComponentFactoryManager( ComponentFactoryManager componentFactoryManager );

    ComponentFactoryManager getComponentFactoryManager();

    // Component manager manager

    // Component repository

    ContainerConfiguration setComponentRepository( ComponentRepository componentRepository );

    ComponentRepository getComponentRepository();

    // Component composer

    // Lifecycle handler manager

    ContainerConfiguration addLifecycleHandler( LifecycleHandler lifecycleHandler );

    ContainerConfiguration setLifecycleHandlerManager( LifecycleHandlerManager lifecycleHandlerManager );

    LifecycleHandlerManager getLifecycleHandlerManager();

    // Configuration Sources

    ContainerConfiguration setConfigurationSource( ConfigurationSource configurationSource );

    ConfigurationSource getConfigurationSource();

    ContainerConfiguration addComponentDiscoverer( Class<?> componentDiscoverer );
    
    ContainerConfiguration addComponentDiscoveryListener( Class<?> componentDiscoveryListener );

    List<Class> getComponentDiscoverers();

    List<Class> getComponentDiscoveryListeners();    
}
```




<br/><br/>
<a id="2"></a>

## 3.2 ContainerConfiguration 的默认实现 DefaultContainerConfiguration 类 ##

Plexus 通过 DefaultContainerConfiguration 类为 ContainerConfiguration 接口提供了默认实现。下面分析它的几个重要方法实现。


<br/><br/>

#### <font size=4 color=green><b>ContainerInitializationPhase[] getInitializationPhases() 方法</b></font> ####

DefaultPlexusContainer 将自己内部组件的初始化过程分离到不同的工具类中去完成，这些初始化工具类都实现一个公共的 ContainerInitializationPhase 接口，定义如下：

```java
public interface ContainerInitializationPhase
{
    String ROLE = ContainerInitializationPhase.class.getName();

    void execute( ContainerInitializationContext context )
        throws ContainerInitializationException;
}
```

在 DefaultPlexusContainer 内部进行初始化容器时，通过调用同一的接口方法 `ContainerInitializationPhase.execute( ContainerInitializationContext context )` 来实现初始化内部组件的目的。

getInitializationPhases() 方法的实现是返回 DefaultContainerConfiguration 内部一个数组字段：

```java
    public ContainerInitializationPhase[] getInitializationPhases()
    {
        return initializationPhases;
    }
```

而 initializationPhases 是已经初始化的 ContainerInitializationPhase 数组定义：

```java
    private ContainerInitializationPhase[] initializationPhases =
        {
            new InitializeComponentRegistryPhase(),
            new InitializeComponentFactoryManagerPhase(),
            new InitializeContainerConfigurationSourcePhase(),
            new InitializeLoggerManagerPhase(),
            new InitializeSystemPropertiesPhase(),
            new InitializeComponentDiscovererManagerPhase(),
            new InitializeUserConfigurationSourcePhase()

        };
```

而 DefaultPlexusContainer 通过如下统一的 `phase.execute( initializationContext )` 方法完成对其内部几个大组件的初始化：

```java

    protected void initializePhases( ContainerConfiguration containerConfiguration )
        throws PlexusContainerException
    {
        ContainerInitializationPhase[] initPhases = containerConfiguration.getInitializationPhases();

        ContainerInitializationContext initializationContext = new ContainerInitializationContext(
            this,
            classWorld,
            containerRealm,
            configuration,
            containerConfiguration );

        for ( ContainerInitializationPhase phase : initPhases )
        {
            try
            {
                phase.execute( initializationContext );
            }
            catch ( Exception e )
            {
                throw new PlexusContainerException( "Error initializaing container in " + phase.getClass().getName()
                    + ".", e );
            }
        }
    }
```



<br/><br/>

#### <font size=4 color=green><b>ComponentRepository getComponentRepository() 方法</b></font> ####

该方法一定会返回一个 ComponentRepository 实现类的实例，即 DefaultComponentRepository 的实例，该方法不会返回 null 值。如下所示：

```java
    public ComponentRepository getComponentRepository()
    {
        if ( componentRepository == null )
        {
            componentRepository = new DefaultComponentRepository();
        }

        return componentRepository;
    }
```


<br/><br/>

#### <font size=4 color=green><b>ComponentFactoryManager getComponentFactoryManager() 方法</b></font> ####

返回组件工厂管理器实现类的实例，即 DefaultComponentFactoryManager 类实例。该方法不会返回 null 值。如下所示：

```java
    public ComponentFactoryManager getComponentFactoryManager()
    {
        if ( componentFactoryManager == null )
        {
            componentFactoryManager = new DefaultComponentFactoryManager();                        
        }

        return componentFactoryManager;
    }

```


<br/><br/>

#### <font size=4 color=green><b>ComponentDiscovererManager getComponentDiscovererManager() 方法</b></font> ####

返回组件发现管理器实现类实例，即 DefaultComponentDiscovererManager 实例。该方法不会返回 null 值，并且会向新创建的 DefaultComponentDiscovererManager 实例添加两个初始的组件发现器实例：DefaultComponentDiscoverer 对象和 PlexusXmlComponentDiscoverer 对象。如下所示：

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


<br/><br/>

#### <font size=4 color=green><b>LifecycleHandlerManager getLifecycleHandlerManager() 方法</b></font> ####

返回生命周期处理器管理器 LifecycleHandlerManager 实例，即默认实现 DefaultLifecycleHandlerManager 的实例，该方法不会返回 null 值。如下所示：

```java
public LifecycleHandlerManager getLifecycleHandlerManager()
    {
        if ( lifecycleHandlerManager == null )
        {
            lifecycleHandlerManager = new DefaultLifecycleHandlerManager();

            // Plexus
            LifecycleHandler plexus = new BasicLifecycleHandler( "plexus" );
            // Begin
            plexus.addBeginSegment( new LogEnablePhase() );
            plexus.addBeginSegment( new ContextualizePhase() );
//            plexus.addBeginSegment( new AutoConfigurePhase() );
            plexus.addBeginSegment( new InitializePhase() );
            plexus.addBeginSegment( new StartPhase() );
            // End
            plexus.addEndSegment( new StopPhase() );
            plexus.addEndSegment( new DisposePhase() );
            plexus.addEndSegment( new LogDisablePhase() );
            lifecycleHandlerManager.addLifecycleHandler( plexus );

            // Basic
            LifecycleHandler basic = new BasicLifecycleHandler( "basic" );
            // Begin
            basic.addBeginSegment( new LogEnablePhase() );
            basic.addBeginSegment( new ContextualizePhase() );
//            basic.addBeginSegment( new AutoConfigurePhase() );
            basic.addBeginSegment( new InitializePhase() );
            basic.addBeginSegment( new StartPhase() );
            // End
            basic.addEndSegment( new StopPhase() );
            basic.addEndSegment( new DisposePhase() );
            basic.addEndSegment( new LogDisablePhase() );
            lifecycleHandlerManager.addLifecycleHandler( basic );

            // Plexus configurable
            LifecycleHandler plexusConfigurable = new BasicLifecycleHandler( "plexus-configurable" );
            // Begin
            plexusConfigurable.addBeginSegment( new LogEnablePhase() );
            plexusConfigurable.addBeginSegment( new ContextualizePhase() );
            plexusConfigurable.addBeginSegment( new ConfigurablePhase() );
            plexusConfigurable.addBeginSegment( new InitializePhase() );
            plexusConfigurable.addBeginSegment( new StartPhase() );
            // End
            plexusConfigurable.addEndSegment( new StopPhase() );
            plexusConfigurable.addEndSegment( new DisposePhase() );
            plexusConfigurable.addEndSegment( new LogDisablePhase() );
            lifecycleHandlerManager.addLifecycleHandler( plexusConfigurable );

            // Passive
            LifecycleHandler passive = new BasicLifecycleHandler( "passive" );
            lifecycleHandlerManager.addLifecycleHandler( passive );

            // Bootstrap
            LifecycleHandler bootstrap = new BasicLifecycleHandler( "bootstrap" );
            bootstrap.addBeginSegment( new ContextualizePhase() );
            lifecycleHandlerManager.addLifecycleHandler( bootstrap );
        }

        return lifecycleHandlerManager;
    }

```







