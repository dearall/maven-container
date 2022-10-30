## 第3章 sisu.plexus 容器配置接口及其默认实现 ##

Plexus 容器的默认实现 DefaultPlexusContainer 类通过容器配置接口 ContainerConfiguration 创建实例。

<br/><br/>
<a id="1"></a>

## 3.1 ContainerConfiguration 接口 ##

sisu.plexus 容器配置接口 ContainerConfiguration 定义如下：

```java

package org.codehaus.plexus;

public interface ContainerConfiguration
{
    ContainerConfiguration setName( String name );
    //
    // Configuration
    //
    ContainerConfiguration setContainerConfiguration( String configurationPath );

    String getContainerConfiguration();

    ContainerConfiguration setContainerConfigurationURL( URL configurationUrl );

    URL getContainerConfigurationURL();

    ContainerConfiguration setClassWorld( ClassWorld classWorld );

    ClassWorld getClassWorld();

    ContainerConfiguration setRealm( ClassRealm classRealm );

    ClassRealm getRealm();

    ContainerConfiguration setContext( Map<Object, Object> context );

    Map<Object, Object> getContext();

    ContainerConfiguration setComponentVisibility( String visibility );

    String getComponentVisibility();

    ContainerConfiguration setAutoWiring( boolean on );

    boolean getAutoWiring();

    ContainerConfiguration setClassPathScanning( String scanning );

    String getClassPathScanning();

    ContainerConfiguration setContextComponent( Context context );

    Context getContextComponent();

    ContainerConfiguration setJSR250Lifecycle( boolean on );

    boolean getJSR250Lifecycle();
}

```

对比传统 Plexus container 的 ContainerConfiguration 定义，可以看出：

1. sisu.plexus 定义的 ContainerConfiguration 接口移除了 **String getName()** 方法
<br/>

2. sisu.plexus 定义的 ContainerConfiguration 接口移除了 **ContainerInitializationPhase[] getInitializationPhases()** 方法
<br/>

3. sisu.plexus 定义的 ContainerConfiguration 接口移除了 **ContainerConfiguration setComponentDiscovererManager( ComponentDiscovererManager componentDiscovererManager )** 和 **ComponentDiscovererManager getComponentDiscovererManager()** 方法
<br/>

4. sisu.plexus 定义的 ContainerConfiguration 接口移除了 **ContainerConfiguration setComponentFactoryManager( ComponentFactoryManager componentFactoryManager )** 和 **ComponentFactoryManager getComponentFactoryManager()** 方法
<br/>

5. sisu.plexus 定义的 ContainerConfiguration 接口移除了 **ContainerConfiguration setComponentRepository( ComponentRepository componentRepository )** 和 **ComponentRepository getComponentRepository()** 方法
<br/>

6. sisu.plexus 定义的 ContainerConfiguration 接口移除了 **ContainerConfiguration addLifecycleHandler( LifecycleHandler lifecycleHandler )** 方法
<br/>

7. sisu.plexus 定义的 ContainerConfiguration 接口移除了 **ContainerConfiguration setLifecycleHandlerManager( LifecycleHandlerManager lifecycleHandlerManager )** 和 **LifecycleHandlerManager getLifecycleHandlerManager()** 方法
<br/>

8. sisu.plexus 定义的 ContainerConfiguration 接口移除了 **ContainerConfiguration setConfigurationSource( ConfigurationSource configurationSource )** 和 **ConfigurationSource getConfigurationSource()** 方法

9. sisu.plexus 定义的 ContainerConfiguration 接口移除了 **ContainerConfiguration addComponentDiscoverer( ComponentDiscoverer componentDiscoverer )** 和 **ContainerConfiguration addComponentDiscoveryListener( ComponentDiscoveryListener componentDiscoveryListener )** 方法
<br/>

10. sisu.plexus 定义的 ContainerConfiguration 接口移除了 **List\<Class\> getComponentDiscoverers() 和 **List\<Class\> getComponentDiscoveryListeners()** 方法
<br/><br/>



另外，sisu.plexus 定义的 ContainerConfiguration 接口增加了如下方法：

1. sisu.plexus 定义的 ContainerConfiguration 接口增加了 **setComponentVisibility( String visibility )** 方法和 **String getComponentVisibility()** 方法
<br/>

1. sisu.plexus 定义的 ContainerConfiguration 接口增加了 **setAutoWiring( boolean on )** 和 **boolean getAutoWiring()** 方法
<br/>

1. sisu.plexus 定义的 ContainerConfiguration 接口增加了 **setClassPathScanning( String scanning )** 和 **String getClassPathScanning()** 方法
<br/>

1. sisu.plexus 定义的 ContainerConfiguration 接口增加了 **setContextComponent( Context context )** 和 **Context getContextComponent()** 方法
<br/>

5. sisu.plexus 定义的 ContainerConfiguration 接口增加了 **ContainerConfiguration setJSR250Lifecycle( boolean on )** 和 **boolean getJSR250Lifecycle()** 方法


<br/><br/>
<a id="2"></a>

## 3.2 ContainerConfiguration 的默认实现 DefaultContainerConfiguration 类 ##

sisu.plexus 通过 DefaultContainerConfiguration 类为 ContainerConfiguration 接口提供了默认实现。实现代码如下：

```java

package org.codehaus.plexus;

public final class DefaultContainerConfiguration
    implements ContainerConfiguration
{
    // ----------------------------------------------------------------------
    // Implementation fields
    // ----------------------------------------------------------------------

    private String configurationPath;

    private URL configurationUrl;

    private ClassWorld classWorld;

    private ClassRealm classRealm;

    private Map<Object, Object> contextData;

    private String componentVisibility = PlexusConstants.REALM_VISIBILITY;

    private String classPathScanning = PlexusConstants.SCANNING_OFF;

    private boolean autoWiring;

    private Context contextComponent;

    private boolean jsr250Lifecycle;

    // ----------------------------------------------------------------------
    // Public methods
    // ----------------------------------------------------------------------

    public ContainerConfiguration setName( final String name )
    {
        return this;
    }

    public ContainerConfiguration setContainerConfiguration( final String configurationPath )
    {
        this.configurationPath = configurationPath;
        return this;
    }

    public String getContainerConfiguration()
    {
        return configurationPath;
    }

    public ContainerConfiguration setContainerConfigurationURL( final URL configurationUrl )
    {
        this.configurationUrl = configurationUrl;
        return this;
    }

    public URL getContainerConfigurationURL()
    {
        return configurationUrl;
    }

    public ContainerConfiguration setClassWorld( final ClassWorld classWorld )
    {
        this.classWorld = classWorld;
        return this;
    }

    public ClassWorld getClassWorld()
    {
        return classWorld;
    }

    public ContainerConfiguration setRealm( final ClassRealm classRealm )
    {
        this.classRealm = classRealm;
        return this;
    }

    public ClassRealm getRealm()
    {
        return classRealm;
    }

    public ContainerConfiguration setContext( final Map<Object, Object> contextData )
    {
        this.contextData = contextData;
        return this;
    }

    public Map<Object, Object> getContext()
    {
        return contextData;
    }

    public ContainerConfiguration setComponentVisibility( final String componentVisibility )
    {
        this.componentVisibility = componentVisibility;
        return this;
    }

    public String getComponentVisibility()
    {
        return componentVisibility;
    }

    public ContainerConfiguration setClassPathScanning( final String classPathScanning )
    {
        this.classPathScanning = classPathScanning;
        if ( !PlexusConstants.SCANNING_OFF.equalsIgnoreCase( classPathScanning ) )
        {
            autoWiring = true;
        }
        return this;
    }

    public String getClassPathScanning()
    {
        return classPathScanning;
    }

    public ContainerConfiguration setAutoWiring( final boolean autoWiring )
    {
        this.autoWiring = autoWiring;
        return this;
    }

    public boolean getAutoWiring()
    {
        return autoWiring;
    }

    public ContainerConfiguration setContextComponent( final Context contextComponent )
    {
        this.contextComponent = contextComponent;
        return this;
    }

    public Context getContextComponent()
    {
        return contextComponent;
    }

    public ContainerConfiguration setJSR250Lifecycle( final boolean jsr250Lifecycle )
    {
        this.jsr250Lifecycle = jsr250Lifecycle;
        return this;
    }

    public boolean getJSR250Lifecycle()
    {
        return jsr250Lifecycle;
    }
}

```

DefaultContainerConfiguration 的实现为一个个简单的 Java Bean 类。其中上下文 context 对象直接使用一个 Map\<Object, Object\> 类型保存数据，而 contextComponent 使用 Context 对象保存数据。**setClassPathScanning( final String classPathScanning )** 和 **setAutoWiring( final boolean autoWiring )** 控制着由 sisu.inject 项目提供的类路径扫描（classpath scanning），自动绑定（auto-binding）和动态装配（dynamic wiring）三大特性。**setClassPathScanning( final String classPathScanning )** 通过如下逻辑直接影响到 autoWiring 的值：

```java
    public ContainerConfiguration setClassPathScanning( final String classPathScanning )
    {
        this.classPathScanning = classPathScanning;
        if ( !PlexusConstants.SCANNING_OFF.equalsIgnoreCase( classPathScanning ) )
        {
            autoWiring = true;
        }
        return this;
    }
```

其中，字符串常量 PlexusConstants.SCANNING_OFF 的值为 "off"。只要设置的 classPathScanning 值不为 "off"，就自动开启自动装配特性。但 classPathScanning 默认是关闭的，如下所示：

```java
private String classPathScanning = PlexusConstants.SCANNING_OFF;
```

因此，使用默认构造的 DefaultContainerConfiguration 实例创建的 sisu.plexus 容器，不具有类路径扫描和自动装配特性。要通过 **setClassPathScanning( final String classPathScanning )** 配置之后，才能具有这两个特性。






