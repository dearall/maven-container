## 第1章 Sisu 项目 ##

sisu 是一系列的 Guice 模块，这些模块加入了类路径扫描（classpath scanning）、自动绑定（auto-binding）、以及动态自动装配（dynamic auto-wiring）特性。这些模块一起组成了 Sisu 容器，但这些模块也可以单独使用。

sisu 是基于 JSR330 规范的模块化容器，使用 Google 的 Guice 容器执行依赖注入，并提供核心 JSR330 规范的支持，但移除了想 Guice 模块中编写显式绑定的需要：

```java
Guice.createInjector(
  new WireModule(                       // auto-wires unresolved dependencies
    new SpaceModule(                     // scans and binds @Named components
      new URLClassSpace( classloader )    // abstracts class/resource finding
) ) );
```

上述代码中，`SpaceModule` 使用`ClassSpace`抽象扫描类路径。任何通过`@Qualifier`注解的注解，例如`@Named`，注解的类被绑定为组件，使它们对 [BeanLocator](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/inject/package-summary.html) 可见，并由[WireModule](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/wire/package-summary.html) 包围（围绕模块集合构成应用程序，本例中只有一个，即 `SpaceModule`）。

`WireModule` 使用 Guice 的内省 SPI 来分析应用中的所有绑定，并应用各种规则，以完成连接（wiring）配置。不满足 `@Inject` 的要求通过 `BeanLocator` 操纵，提供一个跨越它所知道的全部 injector 的查找服务，给出插件风格的解决方案。同样作为单个的组件，`WireModule`可以满足组件集合的请求，例如 List 和 Map（使用 `@Named` 的值作为 key）。它也清理重复的和被覆盖的绑定，并合并 [Parameters](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/Parameters.html) 绑定。




<br/><br/>
<a id="1"></a>

## 1.1 sisu 能做什么 ##

sisu 查找使用 `@Named` 注解的类型，绑定它们，然后为它们装配任何缺失的依赖。

将下面的代码粘贴到 'src/main/java/example/Example.java' 文件中：

```java
package example;

import java.awt.*;
import java.util.*;
import javax.swing.*;

// Standard JSR330 annotations
import javax.inject.Inject;
import javax.inject.Named;

import com.google.inject.Guice;

// Sisu modules for scanning and wiring
import org.eclipse.sisu.space.SpaceModule;
import org.eclipse.sisu.space.URLClassSpace;
import org.eclipse.sisu.wire.WireModule;

// let's create some simple Swing tabs...

abstract class AbstractTab extends JPanel {}

// Sisu will spot any @Named components

@Named("One")
class ButtonTab extends AbstractTab {
  ButtonTab() {
    add(new JButton("Button"));
  }
}

@Named("Two")
class CheckBoxTab extends AbstractTab {
  CheckBoxTab() {
    add(new JCheckBox("CheckBox"));
  }
}

@Named("Three")
class RadioButtonTab extends AbstractTab {
  RadioButtonTab() {
    ButtonGroup group = new ButtonGroup();
    group.add(new JRadioButton("+1"));
    group.add(new JRadioButton("0"));
    group.add(new JRadioButton("-1"));
    Enumeration<AbstractButton> e = group.getElements();
    while (e.hasMoreElements()) {
      add(e.nextElement());
    }
  }
}

// Sisu will launch @EagerSingleton's on startup

@Named
@org.eclipse.sisu.EagerSingleton
class Example implements Runnable {
  JTabbedPane pane = new JTabbedPane();

  // Sisu spots we want a map of tabs and wires it up for us

  @Inject
  Example(Map<String, AbstractTab> tabs) {

    for (Map.Entry<String, AbstractTab> t:tabs.entrySet()) {
      pane.addTab(t.getKey(), t.getValue());
    }

    SwingUtilities.invokeLater(this);
  }

  public void run() {
    JFrame frame = new JFrame("Sisu 5-minute demo");
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    frame.add(pane, BorderLayout.CENTER);
    frame.setLocation(100, 50);
    frame.setSize(300, 200);
    frame.setVisible(true);
  }

  public static void main(String[] args) {
    ClassLoader classloader = Example.class.getClassLoader();

    Guice.createInjector(
      new WireModule(                       // auto-wires unresolved dependencies
        new SpaceModule(                     // scans and binds @Named components
          new URLClassSpace( classloader )    // abstracts class/resource finding
    )));
  }
}

```

<br/><br/>
<a id="2"></a>

## 1.2 sisu 如何工作 ##

sisu 提供一个 [SpaceModule](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/space/package-summary.html) 类用于扫描，一个[WireModule](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/wire/package-summary.html) 类用于将请求装配到它的动态[BeanLocator](https://eclipse.github.io/sisu.inject/apidocs/reference/org/eclipse/sisu/inject/package-summary.html) 上。

使用下面的 maven 命令运行这个 demo：

```shell
mvn compile exec:java -Dexec.mainClass=example.Example
```



