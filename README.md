# 前言 #

<br/><br/>
在研究 maven 及其子项目的时候，发现大量使用了 Plexus Container、Classword、Plexus CLI、Plexus Utils，为了搞清楚这些模块的真实面目，因此决心花些精力研究一下。把研究的过程记录到这个文集中，发布到 github 上，对自己和其他同行都有些益处。

Plexus Container 是 maven 早期版本使用的自研自用项目。进入 maven 3.0 之后，maven 放弃了这个项目的维护，而由 eclipse.sisu.plexus 通过 eclipse.sisu.inject 类库对 Plexus Container 接口进行了重新实现。因此，当前的 Plexus Container 是由 eclipse.sisu.plexus 项目组开发维护，而 maven 项目组直接使用这个实现，继续在 maven 及其子项目中使用 Plexus 容器。 

eclipse.sisu.inject 类库是 Google 的 Guice 项目的扩展项目，它由一些列 Guice 增强模块组成，基于 JSR330 规范，增加了类路径扫描、自动绑定，以及动态装配功能。这些模块组合在一起构成了 sisu 容器，当然，这些模块也可单独使用。

Guice 项目是 Google 开发的小型 IoC 容器，比 Spring 轻量，快捷广泛用于 Google 内部和其它开源项目中。按 mvnrepository.com 统计，目前使用 Guice 的项目有 4889 个之多，包括大名鼎鼎的 Jenkins 和 maven 及其子项目。

这几个项目具有一定的传承或依存关系，因此，我使用剥洋葱的方法，一层一层揭开它们的底层逻辑。

<br/>

Classword、Plexus CLI、Plexus Utils 都是比较小的工具项目，在 maven 生态中，到处可以见到它们的身影，因此会在用到它们时，顺手用几个小章节把它们揭示出来。

<br/>

wagon 是另一个 maven 维护的开源项目，用于 maven 及其子项目中，用以执行网络操作相关的任务，主要是 http 下载工作。本文集也会给它一部分章节，阐释它的用法和内部机制。


文集位置：https://github.com/dearall/maven-container.git

演示代码：https://github.com/dearall/maven-container-demo.git



