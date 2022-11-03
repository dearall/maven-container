## 第9章 与 JSR-330 集成 JSR-330 Integration ##

Guice 3.0 加入的特性。

JSR-330 规范为 Java 平台定义了标准化注解，例如 `@Inject` 和 `Provider` 接口。它当前没有指定应用如何配置，因此没有与 Guice 的模块相似的定义。

Guice 实现了一个完全的 JSR-330 注入器，下面表格简要说明 JSR-330 的类型和它们的 Guice 对等定义：

<table width="100%">
    <tr bgcolor=#AA0000>
        <th align=center>JSR-330 javax.inject</th>
        <th align=center>Guice com.google.inject</th>
        <th align=center>说明</th>
    </tr>
    <tr>
      <td>@Inject</td>
      <td>@Inject</td>
      <td>有约束可互换的</td>
    </tr>
    <tr>
      <td>@Named</td>
      <td>@Named</td>
      <td>可互换的</td>
    </tr>
    <tr>
      <td>@Qualifier</td>
      <td>@BindingAnnotation</td>
      <td>可互换的</td>
    </tr>
    <tr>
      <td>@Scope</td>
      <td>@ScopeAnnotation</td>
      <td>可互换的</td>
    </tr>
    <tr>
      <td>@Singleton</td>
      <td>@Singleton</td>
      <td>可互换的</td>
    </tr>
    <tr>
      <td>Provider</td>
      <td>Provider</td>
      <td>Guice 的 Provider 接口扩展自 JSR-330 的 Provider 接口</td>
    </tr>
</table>

Note 1：JSR-330 在注入点上添加了额外的约束。字段必须是非 final 的。不支持可选注入。方法必须是非抽象的，并且本身没有参数化类型。另外，方法覆写有本质的不同：如果一个要被注入的类覆写了某个方法，而超类（superclass）的方法被 `javax.inject.Inject` 注解，而子类的方法没有加注解，那么该方法不会被注入。

Note 2：Guice 的 `Provider` 接口扩展自 JSR-330 的 `Provider` 接口。使用 `Providers.guicify()` 将一个 JSR-330 的 `Provider` 类型转换为 Guice 的 `Provider` 类型。



<br/><br/>

#### <font size=5 color=green><b>最佳实践</b></font> ####

优先使用 JSR-330 的注解和 Provider 接口。




