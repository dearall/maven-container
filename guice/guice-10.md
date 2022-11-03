## 第10章 扩展 Guice ##

Guice 的 [service provider interface](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/package-summary.html)，SPI 包提供 Guice 的内部模型，以帮助在工具、扩展、以及插件方面的开发。

<br/><br/>

#### <font size=5 color=green><b>核心抽象</b></font> ####

- [InjectionPoint](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/InjectionPoint.html)：一个构造器、字段、或方法可以接收注入。通常情况是一个带有 `@Inject` 注解的成员。对于非 private, 无参数的构造器、成员可以忽略注解。每一个注入的有一个依赖的集合。
 
- [Key](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Key.html)：一个类型，加上一个可选的绑定注解。

- [Dependency](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/Dependency.html)：一个 key，可选地与一个注入点关联。它们的存在是为了可注入字段、可注入方法和构造器的参数。

- [Element](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/Element.html)：是可配置的单元，例如 `bind` 或 `requestInjection` 语句。Element 是可视的 [visitable](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/ElementVisitor.html) 的。

- [Module](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Module.html)：是一个配置元素的集合。模块的 [Extracting the elements](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/Elements.html#getElements(java.lang.Iterable)) 可以进行静态分析和代码重写。可以检查、重写、验证这些元素，并使用它们构建新的模块。

- [Injector](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/Injector.html)：管理应用程序由模块指定的对象图，SPI 访问 injector 的工作类似于反射。它可用于获取应用程序的绑定和依赖图。

[Elements SPI](https://github.com/google/guice/wiki/InspectingModules) 提供使用这些类的更多信息。


#### <font size=5 color=green><b>对扩展作者提供的抽象</b></font> ####

- [@Toolable](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/Toolable.html)：这是一个用于同时也被 @Inject 注解的方法的注解。这个注解指示 Guice 即使是在 Stage.TOOL 阶段也注入方法。通常用于扩展需要收集信息来实现 `HasDependencies`，或者验证要求并过早失败的测试。

- [ProviderWithExtensionVisitor](http://google.github.io/guice/api-docs/latest/javadoc/com/google/inject/spi/ProviderWithExtensionVisitor.html)：一个提供实例实现以允许扩展来访问自定义 BindingTargetVisitor 子接口的接口。

[Extensions SPI](https://github.com/google/guice/wiki/ExtensionSPI) 页面提供更多信息。



<br/><br/>

#### <font size=5 color=green><b>示例</b></font> ####

在模块中为每一个静态注入记录一个 warning 日志：

```java
  public void warnOfStaticInjections(Module... modules) {
    for (Element element : Elements.getElements(modules)) {
      element.acceptVisitor(new DefaultElementVisitor<Void>() {
        @Override
        public Void visit(StaticInjectionRequest element) {
          logger.warning("Static injection is fragile! Please fix "
              + element.getType().getName() + " at " + element.getSource());
          return null;
        }
      });
    }
  }
```





































