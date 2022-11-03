# Summary

* [前言](README.md)

## 第一部分 Plexus
* [第1章 Plexus 项目](plexus/plexus-1.md)
    * [1.1 Plexus 项目的由来](plexus/plexus-1.md#1)
    * [1.2 Plexus 项目概述](plexus/plexus-1.md#2)
    * [1.3 Plexus 项目现状](plexus/plexus-1.md#3)
    * [1.4 Plexus IOC 容器初体验](plexus/plexus-1.md#4)
* [第2章 Plexus IOC 容器接口及默认实现](plexus/plexus-2.md)
    * [2.1 PlexusContainer 接口](plexus/plexus-2.md#1)
    * [2.2 MutablePlexusContainer 接口](plexus/plexus-2.md#2)
    * [2.3 LogEnabled 接口](plexus/plexus-2.md#3)
    * [2.4 Plexus 容器的默认实现 DefaultPlexusContainer 类](plexus/plexus-2.md#4)
        * [2.4.1 DefaultPlexusContainer 的核心组件](plexus/plexus-2.md#41)
        * [2.4.2 DefaultPlexusContainer 公共方法阐释](plexus/plexus-2.md#42)
* [第3章 Plexus IOC 容器配置接口及其默认实现](plexus/plexus-3.md)
    * [3.1 ContainerConfiguration 接口](plexus/plexus-3.md#1)
    * [3.2 ContainerConfiguration 的默认实现 DefaultContainerConfiguration 类](plexus/plexus-3.md#2)
* [第4章 Plexus IOC 容器的组件配置文件](plexus/plexus-4.md)
    * [4.1 Plexus 组件描述文件](plexus/plexus-4.md#1)
        * [4.1.1 component-set 元素](plexus/plexus-4.md#11)
        * [4.1.2 component 元素](plexus/plexus-4.md#12)
        * [4.1.3 requirement 元素](plexus/plexus-4.md#13)
        * [4.1.4 dependency 元素](plexus/plexus-4.md#14)
    * [4.2 Plexus 容器描述文件](plexus/plexus-4.md#2)
        * [4.2.1 plexus 元素](plexus/plexus-4.md#21)
        * [4.2.2 component-discoverer-manager 元素](plexus/plexus-4.md#22)
        * [4.2.3 listener 元素](plexus/plexus-4.md#23)
        * [4.2.4 component-discoverer 元素](plexus/plexus-4.md#24)

## 第二部分 Classworld 项目
* [第1章 Plexus Classworld 项目](classworld/classworld-1.md)
    * [1.1 Classworld API 详解](classworld/classworld-1.md#1)
        * [1.1.1 ClassRealm 类](classworld/classworld-1.md#11)
        * [1.1.2 ClassWorld 类](classworld/classworld-1.md#12)
    * [1.2 使用 Classworlds API](classworld/classworld-1.md#2)

## 第三部分 sisu.plexus
* [第1章 org.eclipse.sisu.plexus 项目](sisu.plexus/sisu-plexus-1.md)
    * [1.1 sisu.plexus IOC 容器初体验](sisu.plexus/sisu-plexus-1.md#1)
* [第2章 sisu.plexus 容器接口及默认实现](sisu.plexus/sisu-plexus-2.md)
  * [2.1 sisu.plexus 的 PlexusContainer 接口](sisu.plexus/sisu-plexus-2.md#1)
  * [2.2 sisu.plexus 的 MutablePlexusContainer 接口](sisu.plexus/sisu-plexus-2.md#2)
  * [2.3 sisu.plexus 提供的 Plexus 容器的默认实现 DefaultPlexusContainer 类](sisu.plexus/sisu-plexus-2.md#3)
* [第3章 sisu.plexus 容器配置接口及其默认实现](sisu.plexus/sisu-plexus-3.md)
  * [3.1 ContainerConfiguration 接口](sisu.plexus/sisu-plexus-3.md#1)
  * [3.2 ContainerConfiguration 的默认实现 DefaultContainerConfiguration 类](sisu.plexus/sisu-plexus-3.md#2)





## 第四部分 sisu.inject









## 第五部分 Google Guice
* [第1章 Google Guice 入门 Getting Started](guice/guice-1.md)
  * [1.1 Google Guice 概述 Overview](guice/guice-1.md#1)
  * [1.2 什么是依赖注入 What is dependency injection](guice/guice-1.md#2)
  * [1.3 Guice 核心概念](guice/guice-1.md#3)
* [第2章 Guice 的内部模型 Mental Model](guice/guice-2.md)
  * [2.1 Guice 是一个 map](guice/guice-2.md#1)
  * [2.2 使用 Guice](guice/guice-2.md#2)
  * [2.3 依赖构成一个有向图 Dependencies form a graph](guice/guice-2.md#3)
* [第3章 Guice 的生命范围 Scope](guice/guice-3.md)
  * [3.1 Guice 内置的 scope（Built-in scopes）](guice/guice-3.md#1)
  * [3.2 应用 Scope（Applying Scope）](guice/guice-3.md#2)
  * [3.3 渴望型单例 Eager Singleton](guice/guice-3.md#3)
  * [3.4 scope 的选择](guice/guice-3.md#4)
  * [3.5 scope 与并发性](guice/guice-3.md#5)
  * [3.6 在测试中使用 NO_SCOPE](guice/guice-3.md#6)
* [第4章 Guice 的绑定 Bindings](guice/guice-4.md)
  * [4.1 链式绑定 Linked Binding](guice/guice-4.md#1)
  * [4.2 绑定注解 Binding Annotations](guice/guice-4.md#2)
  * [4.3 实例绑定 Instance Bindings](guice/guice-4.md#3)
  * [4.4 使用 @Provides 注解方法](guice/guice-4.md#4)
  * [4.5 使用 Provider 绑定](guice/guice-4.md#5)
  * [4.6 无目标绑定 Untargeted Binding](guice/guice-4.md#6)
  * [4.7 构造器绑定 Constructor Binding](guice/guice-4.md#7)
  * [4.8 内置的绑定 Built-in Binding](guice/guice-4.md#8)
  * [4.9 Just-in-time Binding](guice/guice-4.md#9)
  * [4.10 多绑定 Multibindings](guice/guice-4.md#10)
    * [4.10.1 多绑定 Multibinding](guice/guice-4.md#101)
    * [4.10.2 多绑定 MapBinder 类](guice/guice-4.md#102)
    * [4.10.3 多绑定 OptionalBinder 类](guice/guice-4.md#103)
    * [4.10.4 使用 @Provides-like 方法](guice/guice-4.md#104)
    * [4.10.5 限制](guice/guice-4.md#105)
    * [4.10.6 观察多绑定内部信息 Inspecting Multibindings](guice/guice-4.md#106)
  * [第5章 限制绑定源 Restricting the Binding Source](guice/guice-5.md)
  * [第6章 注入 Injections](guice/guice-6.md)
    * [6.1 构造器注入 Constructor Injection](guice/guice-6.md#1)
    * [6.2 方法注入 Method Injection](guice/guice-6.md#2)
    * [6.3 字段注入 Field Injection](guice/guice-6.md#3)
    * [6.4 可选注入 Optional Injections](guice/guice-6.md#4)
    * [6.5 按需注入 On-demand Injection](guice/guice-6.md#5)
    * [6.6 静态注入 Static Injections](guice/guice-6.md#6)
    * [6.7 自动注入 Automatic  Injections](guice/guice-6.md#7)
    * [6.8 注入点 Injection Points](guice/guice-6.md#8)
  * [第7章 注入 Provider](guice/guice-7.md)
    * [7.1 多实例提供者 provider](guice/guice-7.md#1)
    * [7.2 延迟加载的 provider](guice/guice-7.md#2)
    * [7.3 混合 scope 的 provider](guice/guice-7.md#3)
  * [第8章 Aspect Oriented Programming, AOP](guice/guice-8.md)
    * [8.1 实例：拒绝在周末进行方法调用](guice/guice-8.md#1)
    * [8.2 约束](guice/guice-8.md#2)
    * [8.3 注入拦截器 Injecting Interceptors](guice/guice-8.md#3)
    * [8.4 AOP 联盟 AOP Alliance](guice/guice-8.md#4)
  * [第9章 与 JSR-330 集成 JSR-330 Integration](guice/guice-9.md)
  * [第10章 扩展 Guice](guice/guice-10.md)
  * [第11章 自定义 Scope](guice/guice-11.md)
  * [第12章 自定义注入](guice/guice-12.md)
    * [12.1 示例：注入一个 Log4J Logger](guice/guice-12.md#1)















