## 第4章 Plexus IOC 容器的组件配置文件 ##

默认实现的 DefaultPlexusContainer 类从以下位置读取 XML 配置信息：

- 多个 META-INF/plexus/components.xml 文件声明的组件。
- 一个 META-INF/plexus/plexus.xml 文件用于配置 plexus 容器，和运行时额外声明的组件。

但不局限于这些文件，Plexus 容器天生就是可扩展的，它也能被配置为以编程方式或者被扩展以从任何源读取配置信息。在 Maven 2 中的一个例子是从 META-INF/maven/plugin.xml 文件读取插件的配置信息，并将从 maven 仓库下载的 Mojo 进行实例化。



<br/><br/>
<a id="1"></a>

## 4.1 Plexus 组件描述文件 ##

Plexus 组件配置文件 **META-INF/plexus/components.xml** 文件为 ComponentDescriptor\<T\> 接口提供组件配置信息，通过 [ http://plexus.codehaus.org/xsd/components-1.3.0.xsd](http://plexus.codehaus.org/xsd/components-1.3.0.xsd) XSD 定义。由如下元素组成：

```xml
<component-set>
  <components>
    <component>
      <role/>
      <role-hint/>
      <implementation/>
      <version/>
      <component-type/>
      <instantiation-strategy/>
      <lifecycle-handler/>
      <component-profile/>
      <component-composer/>
      <component-configurator/>
      <component-factory/>
      <description/>
      <alias/>
      <isolated-realm/>
      <configuration/>
  
      <requirements>
        <requirement>
          <role/>
          <field-name/>
          <role-hint/>
          <role-hints/>
          <optional/>
        </requirement>
      </requirements>
    </component>
  </components>
  
  <dependencies>
    <dependency>
      <artifact-id/>
      <group-id/>
      <type/>
      <version/>
    </dependency>
  </dependencies>
</component-set>
```

下面分别对每个元素进行阐述。



<br/><br/>
<a id="11"></a>

## 4.1.1 component-set 元素 ##

component-set 元素是 META-INF/plexus/components.xml 文件的唯一根元素，包含一个 ComponentDescriptor 的集合以及该集合的依赖，对应 Plexus 容器的 ComponentSetDescriptor 类。该元素包含如下子元素：

<br/><br/>
<div>表 4.1.1 component-set 包含的子元素</div>

<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center>子元素</th>
        <th align=center>类型</th>
        <th align=center>说明</th>
    </tr>
    <tr>
      <td>components/component*</td>
      <td>List<ComponentDescriptor></td>
      <td>可以有多个，是包含组件描述符的集合，包含 0 或多个组件描述符</td>
    </tr>
    <tr>
      <td>dependencies/dependency*</td>
      <td>List<ComponentDependency></td>
      <td>可以有多个，是这个组件描述符集合必须的依赖集合，包含 0 或多个依赖项</td>
    </tr>
</table>



<br/><br/>
<a id="12"></a>

## 4.1.2 component 元素 ##

component 元素为一个组件实例化的描述，对应 Plexus 容器的 ComponentDescriptor\<T\> 类。可以包含如下表所示的元素：

<br/><br/>
<div>表 4.1.2 component-set 包含的子元素</div>

<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center>子元素</th>
        <th align=center>类型</th>
        <th align=center>说明</th>
    </tr>
    <tr>
      <td>role</td>
      <td>String</td>
      <td>组件角色。一般指明该组件接口或基类的全限定类名</td>
    </tr>
    <tr>
      <td>role-hint</td>
      <td>String</td>
      <td>组件的 role-hint，角色区别标识。当多个类实现同一个 role 指明的接口，或多个子类继承同一 role 指明的基类，使用该字符串识别不同的实现类</td>
    </tr>
    <tr>
      <td>implementation</td>
      <td>String</td>
      <td>指明组件的实现。实现可以是一个字符串，指明普通 Java 组件的 FQCN（Fully Qualified Class Name），即全限定类名，或者用于其它组件工厂实现的名称或文件</td>
    </tr>
    <tr>
      <td>version</td>
      <td>String</td>
      <td>组件项目时间线的某个具体版本，例如 v1.0，2.1.4</td>
    </tr>
    <tr>
      <td>component-type</td>
      <td>String</td>
      <td>这个组件的类型</td>
    </tr>
    <tr>
      <td>instantiation-strategy</td>
      <td>String</td>
      <td>该组件的实例化策略。取值可以是 "per-lookup", "singleton", "keep-alive" or "poolable" 其中之一。默认值为 <b>"singleton"</b></td>
    </tr>
    <tr>
      <td>lifecycle-handler</td>
      <td>String</td>
      <td>为该组件配置的生命周期。例如 "basic", "passive", "bootstrap"</td>
    </tr>
    <tr>
      <td>component-profile</td>
      <td>String</td>
      <td></td>
    </tr>
    <tr>
      <td>component-composer</td>
      <td>String</td>
      <td>该组件要使用的 composer 的类型 ID。例如，"setter" 或 "field" 为不同的依赖注入类型</td>
    </tr>
    <tr>
      <td>component-configurator</td>
      <td>String</td>
      <td>为这个项目指明组件配置器的类型。例如，使用 "basic" 为普通类型，使用 map-oriented" 表明是面向 map 的组件</td>
    </tr>
    <tr>
      <td>component-factory</td>
      <td>String</td>
      <td>用于创建这个组件的工厂的 id 号。例如，"jruby" 用于 JRuby 工厂</td>
    </tr>
    <tr>
      <td>description</td>
      <td>String</td>
      <td>一段人类可读的组件描述</td>
    </tr>
    <tr>
      <td>alias</td>
      <td>String</td>
      <td>该组件的一个别名。别名是与普通 key 不同的另一个名字</td>
    </tr>
    <tr>
      <td>isolated-realm</td>
      <td>boolean</td>
      <td>如果该组件在一个隔离的领域类加载器中，值为 true。默认值为：<b>false</b></td>
    </tr>
    <tr>
      <td>configuration</td>
      <td>DOM</td>
      <td>为该组件定义的配置值</td>
    </tr>
    <tr>
      <td>requirements/requirement*</td>
      <td>List<ComponentRequirement> 	</td>
      <td>可以多个，该组件的项目要求</td>
    </tr>
</table>


<br/><br/>
<a id="13"></a>

## 4.1.3 requirement 元素 ##

表示一个被另一个组件要求的组件（required by another component），对应 Plexus 容器的 ComponentRequirement 类。

<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center>子元素</th>
        <th align=center>类型</th>
        <th align=center>说明</th>
    </tr>
    <tr>
      <td>role</td>
      <td>String</td>
      <td>被要求组件的 role</td>
    </tr>
    <tr>
      <td>field-name</td>
      <td>String</td>
      <td>该组件字段名，即被要求的组件作为该组件中某个具体字段的值 —— 字段注入</td>
    </tr>
    <tr>
      <td>role-hint</td>
      <td>String</td>
      <td>被要求组件的 role-hint。默认为 <b>default</b></td>
    </tr>
    <tr>
      <td>role-hints/role-hint*</td>
      <td>List<String></td>
      <td>被要求组件的多个 role-hint。可以多个</td>
    </tr>
    <tr>
      <td>optional</td>
      <td>boolean</td>
      <td>控制这个被要求组件的创建失败，是否可以被寄主组件（host component）所容忍，或者寄主组件的创建是否也应失败</td>
    </tr>
</table>



<br/><br/>
<a id="14"></a>

## 4.1.4 dependency 元素 ##

表示依赖的类库，按 maven 的语义定义所依赖的类库。对应 Plexus 容器的 ComponentDependency 类。


<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center>子元素</th>
        <th align=center>类型</th>
        <th align=center>说明</th>
    </tr>
    <tr>
      <td>artifact-id</td>
      <td>String</td>
      <td>依赖库的 artifact ID</td>
    </tr>
    <tr>
      <td>group-id</td>
      <td>String</td>
      <td>依赖库的 group ID</td>
    </tr>
    <tr>
      <td>type</td>
      <td>String</td>
      <td>依赖库的类型，例如 "jar"</td>
    </tr>
    <tr>
      <td>version</td>
      <td>String</td>
      <td>依赖库的版本，即该库项目开发时间线上的某个具体的时间点。例如 2.0，4.5.3</td>
    </tr>
</table>

<br/><br/>
<a id="2"></a>

## 4.2 Plexus 容器描述文件 ##

Plexus 容器描述文件 **META-INF/plexus/plexus.xml** 是对 Plexus 容器的整体配置。通过 [http://plexus.codehaus.org/xsd/plexus-1.3.0.xsd](http://plexus.codehaus.org/xsd/plexus-1.3.0.xsd) 提供 XSD 定义。

容器描述文件 plexus.xml 文件是组件描述文件 components.xml 的超集。其中可以包含 \<components/\> 元素，用于声明组件配置信息，\<components/\> 元素与组件描述文件 components.xml 中的 \<components/\> 元素使用同一语法定义，这里不再重复，参考上一节阐述。

plexus.xml 文件由如下元素组成：

```xml
    <plexus>
      <load-on-start/>
      <system-properties/>
      <configurations-directory/>
      <logging/>
      <component-repository/>
      <resources/>
      <component-manager-manager/>
      <component-discoverer-manager implementation=.. >
        <listeners>
          <listener implementation=.. />
        </listeners>
        <component-discoverers>
          <component-discoverer implementation=.. />
        </component-discoverers>
      </component-discoverer-manager>
      <component-factory-manager/>
      <lifecycle-handler-manager/>
      <component-composer-manager/>
     
      <components/>
    </plexus>
```

下面分别对每个元素进行阐述。



<br/><br/>
<a id="21"></a>

## 4.2.1 plexus 元素 ##

plexus 元素是 Plexus 容器描述文件 META-INF/plexus/plexus.xml 的唯一根元素，用于配置容器和运行时额外声明的组件。对应 org.codehaus.plexus.configuration.PlexusConfiguration 接口和默认实现 DefaultPlexusConfiguration 类，可以包含如下子元素：


<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center>子元素</th>
        <th align=center>类型</th>
        <th align=center>说明</th>
    </tr>
    <tr>
      <td>load-on-start</td>
      <td>DOM</td>
      <td>TBD</td>
    </tr>
    <tr>
      <td>system-properties</td>
      <td>DOM</td>
      <td>TBD</td>
    </tr>
    <tr>
      <td>configurations-directory</td>
      <td>DOM</td>
      <td>TBD</td>
    </tr>
    <tr>
      <td>logging</td>
      <td>DOM</td>
      <td>TBD</td>
    </tr>
    <tr>
      <td>component-repository</td>
      <td>DOM</td>
      <td>TBD</td>
    </tr>
    <tr>
      <td>resources</td>
      <td>DOM</td>
      <td>TBD</td>
    </tr>
    <tr>
      <td>component-manager-manager</td>
      <td>DOM</td>
      <td>TBD</td>
    </tr>
    <tr>
      <td>component-discoverer-manager</td>
      <td>ComponentDiscovererManager</td>
      <td>TBD</td>
    </tr>
    <tr>
      <td>component-factory-manager</td>
      <td>DOM</td>
      <td>TBD</td>
    </tr>
    <tr>
      <td>lifecycle-handler-manager</td>
      <td>DOM</td>
      <td>TBD</td>
    </tr>
    <tr>
      <td>component-composer-manager</td>
      <td>DOM</td>
      <td>TBD</td>
    </tr>
    <tr>
      <td>components</td>
      <td>DOM</td>
      <td>可在本描述文件中配置组件描述符，与 components.xml 文件完全类似</td>
    </tr>
</table>

TBD：即 To Be Determined 待决定。


<br/><br/>
<a id="22"></a>

## 4.2.2 component-discoverer-manager 元素 ##

对应 org.codehaus.plexus.component.discovery.ComponentDiscovererManager 接口及其默认实现 DefaultComponentDiscovererManager 类。对组件发现器和组件发现监听器进行管理，并在发现组件过程中，触发组件发现事件。该元素支持如下属性：

<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center>元素属性 Attribute</th>
        <th align=center>类型</th>
        <th align=center>说明</th>
    </tr>
    <tr>
      <td>implementation</td>
      <td>String</td>
      <td>实现类的全限定类名</td>
    </tr>
</table>

component-discoverer-manager 元素支持如下子元素：

<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center>子元素</th>
        <th align=center>类型</th>
        <th align=center>说明</th>
    </tr>
    <tr>
      <td>listeners/listener*</td>
      <td>List<ComponentDiscoveryListener></td>
      <td>配置 0 或多个组件发现监听器。通过 listener 子元素的 implementation 属性指明监听器的全限定类名</td>
    </tr>
    <tr>
      <td>component-discoverers/component-discoverer*</td>
      <td>List<ComponentDiscoverer></td>
      <td>配置 0 或多个组件发现器。通过 component-discoverer 子元素的 implementation 属性指明监听器的全限定类名</td>
    </tr>
</table>



<br/><br/>
<a id="23"></a>

## 4.2.3 listener 元素 ##

listener 元素对应 org.codehaus.plexus.component.discovery.ComponentDiscoveryListener 接口，支持如下属性：

<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center>元素属性 Attribute</th>
        <th align=center>类型</th>
        <th align=center>说明</th>
    </tr>
    <tr>
      <td>implementation</td>
      <td>String</td>
      <td>ComponentDiscoveryListener 实现类的全限定类名</td>
    </tr>
</table>



<br/><br/>
<a id="24"></a>

## 4.2.4 component-discoverer 元素 ##

component-discoverer 元素对应 org.codehaus.plexus.component.discovery.ComponentDiscoverer 接口，支持如下属性：

<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center>元素属性 Attribute</th>
        <th align=center>类型</th>
        <th align=center>说明</th>
    </tr>
    <tr>
      <td>implementation</td>
      <td>String</td>
      <td>ComponentDiscoverer 实现类的全限定类名</td>
    </tr>
</table>