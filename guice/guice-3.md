## 第3章 Guice 的生命范围 Scope ##

默认情况下，Guice 每次提供值的时候返回一个新的实例。这个行为可以通过 scope 来配置。Scope 允许在一定范围内重用实例：应用程序生命周期（lifetime of an application）、会话生命周期（a session）、或者一个请求（a request）。



<br/><br/>
<a id="1"></a>

## 3.1 内置的 scope（Built-in scopes） ##


<br/><br/>

#### <font size=5 color=green><b>Singleton</b></font> ####

Guice 自带的内置 @Singleton scope，在应用程序整个生命周期期间，重用一个单独的注入器 injector 内的同一个实例。Guice 同时支持 `javax.inject.Singleton` 和 `com.google.inject.Singleton` 注解，但最好使用 javax.inject.Singleton，因为这样也可以被其它框架支持。

<br/><br/>

#### <font size=5 color=green><b>SingRequestScopedleton</b></font> ####

[servlet extension](https://github.com/google/guice/wiki/Servlets) 为 web app 提供了另外的 scope，例如 @RequestScoped。



<br/><br/>
<a id="2"></a>

## 3.2 应用 Scope（Applying Scope） ##



