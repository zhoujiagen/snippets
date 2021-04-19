# Notes of Google Guice

# 源码阅读笔记模板

|时间|内容|
|:---|:---|
|20210419|kick off.|

## 术语

<!-- 记录阅读过程中出现的关键字及其简单的解释. -->

## 介绍

<!-- 描述软件的来源、特性、解决的关键性问题等. -->

## 动机

<!-- 描述阅读软件源码的动机, 要达到什么目的等. -->

## 系统结构

<!-- 描述软件的系统结构, 核心和辅助组件的结构; 系统较复杂时细分展示. -->

## 使用

<!-- 记录软件如何使用. -->

## 数据结构和算法

<!-- 描述软件中重要的数据结构和算法, 支撑过程部分的记录. -->

### `javax.inject`


|Name|Description|
|:---|:---|
|`Provider<T>`|Provides instances of T.|
|`Inject`|Identifies injectable constructors, methods, and fields.|
|`Named`|String-based qualifier.|
|`Qualifier`|Identifies qualifier annotations.|
|`Scope`|Identifies scope annotations.|
|`Singleton`|Identifies a type that the injector only instantiates once.|

### `com.google.inject`


|<div style="width: 100px;">Name</div>|Description|
|:---|:---|
|`Key`|Guice uses `Key` to identify a dependency that can be resolved.|
|`Provider`|Guice uses `Provider` to represent factories that are capable of creating objects to satisfy dependency requirements.|
|`AbstractModule`|To create bindings, extend `AbstractModule` and override its configure method. In the method body, call `bind()` to specify each binding. These methods are type checked so the compiler can report errors if you use the wrong types. Once you've created your modules, pass them as arguments to `Guice.createInjector()` to build an injector.|
|`Provides`|Annotates methods of a `Module` to create a provider method binding.|
|`Qualifier`|`@Qualifier` is a JSR-330 meta-annotation that tells Guice that this is a binding annotation.<br/>Older code may still use Guice `BindingAnnotation` in place of the standard `@Qualifier` annotation. New code should use `@Qualifier` instead.|
|`name.Named`|Annotates named things.|

## 过程

<!-- 描述软件中重要的过程性内容, 例如服务器的启动、服务器响应客户端请求、服务器背景活动等. -->

### Creating Bindings

- linked bindings
- instance bindings
- `@Provides` methods
- provider bindings
- constructor bindings
- untargetted bindings
- built-in bindings
- just-in-time binding
- Multibindings: `Multibinder`, `MapBinder`, `OptionalBinder`

### Injections

- Constructor Injection
- Method Injection
- Field Injection
- Optional Injections
- On-demand Injection
- Static Injections
- Automatic Injection
- Injection Points

An injection point is a place in the code where Guice has been asked to inject a dependency.

- Injecting Providers

### Extensions

|<div style="width: 150px;">Abstraction</div>|Description|
|:---|:---|
|`spi.InjectionPoint`|A constructor, field or method that can receive injections. Typically this is a member with the `@Inject` annotation. For non-private, no argument constructors, the member may omit the annotation. Each injection point has a collection of dependencies.|
|`Key`|A type, plus an optional binding annotation.|
|`spi.Dependency`|A key, optionally associated to an injection point. These exist for injectable fields, and for the parameters of injectable methods and constructors.|
|`spi.Element`|A configuration unit, such as a `bind` or `requestInjection` statement. Elements are visitable|
|`Module`|A collection of configuration elements. Extracting the elements of a module enables static analysis and code-rewriting. You can inspect, rewrite, and validate these elements, and use them to build new modules.|
|`Injector`|Manages the application's object graph, as specified by modules. SPI access to the injector works like reflection. It can be used to retrieve the application's bindings and dependency graph.|


## 文献引用

<!-- 记录软件相关和进一步阅读资料: 文献、网页链接等. -->

- Guice wiki: https://github.com/google/guice/wiki
- JSR-330 Dependency Injection standard for Java: https://github.com/javax-inject/javax-inject

## 其他备注
