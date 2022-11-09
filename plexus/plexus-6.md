## 第6章 Plexus 组件描述符 Plexus Component Descriptor ##


一个组件描述符，描述容器管理该组件所要求的组件属性。看一个最简单的组件描述符：

```xml
    <component-sets>
      <components>
        <component>
          <role>foo.Foo</role>
          <implementation>foo.DefaultFoo</implementation>
        </component>
      </components>
    </component-sets>
```

以 `component` 标签开始，其内部由 `role` 和 `implementation` 标签定义的值。其中 `role` 标签定义了这个组件提供的接口，它一般是 Java 接口的名字，类名（包括抽象类名）也是可以的，不允许使用任意的字符串。`implementation` 标签告诉 Plexus 所要使用的实现指定 `role` 的组件名，通常情况是指定实现给定接口（role）的类名。

如果有多个的组件为给定的 role 提供实现，可以通过 `role-hint` 的帮助以示区别：

```xml
<component>
  <role>foo.SomeComponent</role>
  <role-hint>componentA</role-hint>
  <implementation>foo.FooA</implementation>
<component>
 
<component>
  <role>foo.SomeComponent</role>
  <role-hint>componentB</role-hint>
  <implementation>foo.FooB</implementation>
<component>
...

```

`role` 和 `role-hint` 一起定义了组件标识符（component identity），并使用它们来从 Plexus 容器中查找组件。`role-hint` 是可选的，它作为额外的 id 限定器用于区分同一个类型（role）的不同组件实现。可以使用任意的字符串作为 `role-hint` 的值。利用不同的 `role-hint`，可以多次部署组件的同一个实现，例如：

```xml
    ...
    <component>
      <role>foo.SomeComponent</role>
      <role-hint>instance-one</role-hint>
      <implementation>foo.FooA</implementation>
    <component>
     
    <component>
      <role>foo.SomeComponent</role>
      <role-hint>instance-two</role-hint>
      <implementation>foo.FooA</implementation>
    <component>
    ...
```

<br/>

#### <font size=3><b>Requirements</b></font> ####

组件作为隔离的单元没有什么太大的用处，它们也不经常单独使用。当把组件与其它组件连接在一起并为它们提供配置设置时，就变得非常实用了。有几种不同的方法来使用组件来组装应用系统。

默认的，也是最简单的方法是，使用**字段注入 field injection**将组件连接起来（wiring components），意思是由 Plexus 选择对象并把它们赋值给组件对象内的字段。

那么 Plexus 如何能知道要注入哪些要求的字段以及要求的是什么对象呢？在组件描述符中有一个 `requirements` 小节可以声明该组件所依赖的哪些组件，如下所示：

```xml
    ...
    <component>
      ...
      <requirements>
        <requirement>
          ...
        </requirement>
        <requirement>
          ...
        </requirement>
      </requirements>
    </component>
    ...
```

简单情况下，一个依赖的声明类似这样：

```xml
      ...
      <requirement>
        <role>org.codehaus.plexus.ComponentA</role>
      </requirement>
      ...
```

想要声明一个同时指定 role 和 role-hint 的依赖组件时，可以：

```xml
      ...
      <requirement>
        <role>org.codehaus.plexus.ComponentB</role>
        <role-hint>foo</role-hint>
      </requirement>
      ...
```

在任何情况下，组件合成器（Component Composer）会对具体的要求（the given requirements）试图找到一个匹配的字段和属性。标准情况下，组件合成器会试图找到组件类中的一个字段（通常为私有字段 private field），它有一个类型匹配 `requirement` 标签的 `role` 元素的值。

例如下面这个 Java 类的情况：

```java
    package foo;
    public class SomeComponentImpl
    {
         // this is a "requirement" of this component
         org.codehaus.plexus.ComponentA  a;
     
         // this is ordinary field
         int  b;
    }
```

需要准备下面组件描述符，将 ComponentA 作为一个 requirement 列在 `requirements` 小节内：

```xml
    ...
    <component>
      <role>foo.SomeComponent</role>
      <implementation>foo.SomeComponentImpl</implementation>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.ComponentA</role>
        </requirement>
      </requirements>
    <component>
    ...
```


<br/>

#### <font size=3><b>集合 Collections</b></font> ####

Plexus 可以注入依赖组件的 Map, List 或者数组的集合类型。组件数组的情形与单个依赖的情况使用相同的方法。唯一不同的是对一个给定 role 使用所有可见的实现。例如：

```java
    package foo;
    public class SomeComponentImpl
    {
         org.codehaus.plexus.ComponentA  a[];
    }
```

xml 描述符：

```xml
    ...
    <component>
      <role>foo.SomeComponent</role>
      <implementation>foo.SomeComponentImpl</implementation>
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.ComponentA</role>
        </requirement>
      </requirements>
    </component>
     
    ...
     
    <component>
      <role>org.codehaus.plexus.ComponentA</role>
      <role-hint>A</role-hint>
      ...
    </component>
    <component>
      <role>org.codehaus.plexus.ComponentA</role>
      <role-hint>B</role-hint>
      ...
    </component>
    ...
```

对于 List 和 Map 情况，必须显式定义依赖应注入的地方。可以通过 `field-name` 标签的帮助来实现：

XML 描述符：

```xml
    ...
    <component>
      ...
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.ComponentB</role>
          <role-hint>foo</role-hint>
          <field-name>mapA</field-name>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.ComponentB</role>
          <role-hint>bar</role-hint>
          <field-name>listB</field-name>
        </requirement>
      </requirements>
    </component>
    ...
```

对应的 Java 代码如下：

```java
    package foo;
    public class SomeComponentImpl
    {
           private Map mapA;
           private List listB;
    }
```

注意，在使用 Map 时，组件的 `role-hint` 用作 key，而组件的实例用作 value。

也可以使用 `field` 标签指明一个单独的（"singular"）组件要求，如下所示：

```xml
    ...
    <component>
      ...
      <requirements>
        <requirement>
          <role>org.codehaus.plexus.ComponentB</role>
          <role-hint>foo</role-hint>
          <field>b1</field>
        </requirement>
        <requirement>
          <role>org.codehaus.plexus.ComponentB</role>
          <role-hint>bar</role-hint>
          <field>b2</field>
        </requirement>
      </requirements>
    <component>
    ...
```

对应的 Java 代码：

```java
    package foo;
    public class SomeComponentImpl
    {
        //(component with role-hint = "foo" will be injected here)
        org.codehaus.plexus.ComponentB  b1;
     
        //(component with role-hint = "bar" will be injected here)
        org.codehaus.plexus.ComponentB  b2;
    }
```

显式指定要注入哪个字段的依赖被认为是一个好的实践。


<br/>

#### <font size=3><b>Configuration</b></font> ####

最后，有一个可选的配置小节可用于配置组件：

```xml
    ...
    <component>
      ...
      <configuration>
        <a>bleh</a>
        <b>
          <x>1</x>
          <y>2.0f</y>
        </b>
      </configuration>
    <component>
    ...
```

关于配置的问题，参考 [第7章 Plexus 自动组件配置 Auto Configuration](plexus/plexus-7.md)。


