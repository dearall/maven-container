## 第7章 Plexus 自动组件配置 Auto Configuration ##

假设自动配置机制应用到下面的类：

```java
    class com.MyComponent implements com.SomeInterface
    {
       private String propertyA;
     
       private int propertyB;
     
       ...
    }
```

使用一个 Plexus 配置如下：

```xml
    <component>
      <role>com.SomeInterface</role>
      <implementation>com.MyComponent</implementation>
        <configuration>
            <propertyA>foo</propertyA>
            <propertyB>1</propertyA>
        </configuration>
    </component>
```

组件配置器 ComponentConfigurator 在一个组件实例上执行类似如下的操作：

```java
    com.SomeInterface component = new com.MyComponent();
     
    component.propertyA = "foo";
     
    component.propertyB = 1;
```

自动配置支持如下类型：

- java.lang.Boolean & boolean
- java.lang.Byte & byte
- java.lang.Character & char
- java.lang.Double & double
- java.lang.Float & float
- java.lang.Integer & int
- java.lang.Long & long
- java.lang.Short & short
- java.lang.StringBuffer
- java.lang.String
- java.util.Date (todo: document supported patterns)
- java.math.BigDecimal
- java.math.BigInteger



<br/>

#### <font size=3><b>组合类型 Composite types (Object properties)</b></font> ####

- 带有属性的对象

组件实现：

```java
    class com.MyComponent
    {
       private Person person;
     
       ...
    }
     
    class com.Person
    {
       private String firstname;
     
       private String lastname;
     
       ...
    }
```

组件配置：

```xml
    <configuration>
       <person>
           <firstname>Baltzar<firstname>
           <lastname>Gabka</lastname>
       </person>
    </configuration>
```

组件配置器执行的操作：

```java
    Person person = new Person();
     
    person.firstname = "Baltazar";
    person.lastname = "Gabka";
     
    component.person = person;
```

- java.lang.Properties

组件实现：

```java
    class com.MyComponent
    {
       private Properties propertiesA;
       ...
    }
```

组件配置：

```xml
    <configuration>
       <propertiesA>
          <property>
              <name>key1</name>
              <value>value1</value>
          </property>
          <property>
              <name>key2</name>
              <value>value2</value>
          </property>
       </propertiesA>
    </configuration>
```

组件配置器执行的操作：

```java
    com.MyComponent component;
     
    Properties properties = new Properties();
     
    properties.put( "key1", "name1" )
     
    properties.put( "key2", "name2" )
     
    setFieldValue( component, "propertiesA", properties );
     
    component.propertiesA = properties;
```













