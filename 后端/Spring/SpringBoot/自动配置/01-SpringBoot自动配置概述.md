# 1. 为什么需要自动配置

直接使用 Spring Framework 进行项目开发时，除了需要管理项目依赖之外，还需要完成大量与框架相关的配置。

例如开发某项功能时，开发者可能需要：

- 声明框架需要使用的 Bean。
    
- 创建对应的配置类。
    
- 配置框架运行需要的相关对象。
    
- 将这些对象注册到 Spring IOC 容器。
    

随着项目使用的框架和组件越来越多，需要手动完成的配置也会不断增加。

Spring Boot 为了进一步简化 Spring 应用开发，提供了两个非常重要的功能：

```text
起步依赖
自动配置
```

其中：

|机制|主要解决的问题|
|---|---|
|Starter 起步依赖|简化项目依赖配置|
|自动配置|简化 Bean 声明和框架配置|

因此，自动配置重点解决的是：

> **项目引入某些功能以后，如何减少开发者手动声明和配置 Bean 的工作。**

---

# 2. 什么是 Spring Boot 自动配置

Spring Boot 自动配置可以理解为：

> **Spring 容器启动后，一些配置类和 Bean 会自动进入 IOC 容器，不需要开发者逐个手动声明。**

例如原本可能需要自己编写：

```java
@Configuration
public class GsonConfig {

    @Bean
    public Gson gson() {
        return new Gson();
    }
}
```

将 `Gson` 注册到 IOC 容器。

而在满足 Spring Boot 对应自动配置条件的情况下，即使项目中没有手动编写这样的 Bean 配置，也可能直接从 IOC 容器中获得：

```java
@Autowired
private Gson gson;
```

因此，自动配置带来的核心效果就是：

```text
减少手动配置
↓
常用配置类和 Bean 自动进入 IOC 容器
↓
开发者可以直接使用
```

自动配置并没有改变 IOC 容器管理 Bean 这件事。

最终这些对象仍然是：

> **Spring IOC 容器中的 Bean。**

区别只在于，以前可能需要开发者自己声明，而现在其中一部分配置工作由 Spring Boot 自动完成。

---

# 3. 自动配置解决了什么问题

## 3.1 减少 Bean 的手动声明

如果没有自动配置，使用某个组件时，开发者可能需要主动创建对象并注册 Bean。

例如：

```java
@Configuration
public class Config {

    @Bean
    public SomeComponent someComponent() {
        return new SomeComponent();
    }
}
```

当需要配置的组件越来越多时，这些配置代码也会越来越多。

自动配置可以帮助 Spring Boot 在适当情况下自动完成这类 Bean 注册。

因此开发者不需要为所有常用组件都重复编写：

```java
@Bean
```

配置。

## 3.2 减少框架配置工作

自动配置不仅仅是简单地创建某一个 Bean。

在实际框架使用过程中，一个功能可能需要多个：

```text
配置类
Bean
相关运行配置
```

Spring Boot 可以提前提供这些常见配置。

因此开发者在引入需要的功能后，可以直接使用其中许多已经准备好的能力，而不需要从头完成所有框架配置。

所以自动配置的核心目的可以概括为：

> **减少框架使用过程中重复、常见的配置工作，让开发者更加集中地编写业务代码。**

---

# 4. Gson 自动配置案例

理解自动配置最直观的方法，就是观察一个：

> **自己没有声明，却能够直接使用的 Bean。**

例如项目中需要把 Java 对象转换成 JSON，可以使用：

```java
Gson
```

正常情况下，如果需要由 Spring 管理这个对象，可以手动声明：

```java
@Bean
public Gson gson() {
    return new Gson();
}
```

但是在示例中，项目并没有自己声明 `Gson` Bean，却仍然可以直接注入：

```java
@Autowired
private Gson gson;
```

并正常使用。

出现这种现象的原因是：

> **Spring Boot 已经通过自动配置机制帮助项目配置了对应的 Bean。**

因此这里真正需要思考的问题不是：

> “Gson 为什么可以被 `@Autowired`？”

因为 `@Autowired` 本身只是负责从 IOC 容器中寻找 Bean。

真正的问题是：

> **我们没有手动声明 Gson，Gson Bean 是怎么进入 IOC 容器的？**

答案就是：

```text
Spring Boot 自动配置
```

这也是后续研究自动配置原理时需要解决的核心问题。

---

# 5. 自动配置与 Starter 的区别

Starter 和自动配置都是 Spring Boot 简化开发的重要机制，但它们解决的问题并不相同。

## 5.1 Starter

Starter 主要解决：

> **依赖如何方便地进入项目。**

例如：

```xml
spring-boot-starter-web
```

会利用 Maven 的依赖传递机制，将 Web 开发常见依赖引入当前项目。

因此 Starter 主要解决的是：

```text
依赖配置问题
```

## 5.2 自动配置

自动配置主要解决：

> **相关依赖进入项目以后，其中需要使用的配置类和 Bean 如何进入 IOC 容器。**

因此可以这样区分：

|机制|主要职责|
|---|---|
|Starter|将需要的依赖引入项目|
|自动配置|将符合条件的配置类和 Bean 注册到 IOC 容器|

二者虽然经常配合出现，但并不是同一个概念。

可以建立这样的认识：

```text
引入所需依赖
↓
项目具备相关类和组件
↓
Spring Boot 自动完成部分常用配置
↓
相关 Bean 进入 IOC 容器
↓
开发者直接使用
```

因此：

> **Starter 更关注“依赖”，自动配置更关注“配置类和 Bean”。**

---

# 6. 自动配置后续需要解决的核心问题

理解自动配置概念以后，接下来真正需要研究的是：

> **Spring Boot 到底如何把其他依赖中的配置类和 Bean 自动加载到当前项目的 IOC 容器中？**

因为前面已经知道，Spring Boot 默认组件扫描主要扫描：

```text
启动类所在包
+
启动类所在包的子包
```

但是第三方依赖中的类通常并不位于当前项目的默认组件扫描范围内。

因此，仅仅依靠普通的：

```java
@ComponentScan
```

并不能完整解释 Spring Boot 的自动配置机制。

后续需要继续分析：

```text
@SpringBootApplication
@EnableAutoConfiguration
@Import
ImportSelector
AutoConfigurationImportSelector
```

以及 Spring Boot 如何找到需要加载的自动配置类。

这一篇只需要先建立一个整体认识：

> **自动配置的最终目标，是让符合条件的配置类和 Bean 自动进入 Spring IOC 容器。**

具体如何实现，将在后续原理笔记中继续分析。

---

# 7. 当前阶段需要掌握的核心内容

1. Spring Boot 的两个重要功能是 Starter 起步依赖和自动配置。
    
2. Starter 主要简化依赖配置，自动配置主要简化 Bean 声明和框架配置。
    
3. 自动配置发生在 Spring 容器启动过程中。
    
4. 自动配置会让一些配置类和 Bean 自动进入 IOC 容器。
    
5. 自动配置后的对象仍然属于普通的 Spring Bean，仍然由 IOC 容器统一管理。
    
6. Gson 是理解自动配置的典型案例：开发者没有手动声明 Gson Bean，却仍然可以从 IOC 容器中注入使用。
    
7. `@Autowired` 只负责获取 Bean，真正需要研究的是这个 Bean 为什么会自动进入 IOC 容器。
    
8. Starter 和自动配置不是同一个概念，前者主要解决依赖问题，后者主要解决配置问题。
    
9. 自动配置的核心原理问题是：Spring Boot 如何将依赖中的配置类和 Bean 自动加载到当前项目的 IOC 容器。
    
10. `@SpringBootApplication`、`@EnableAutoConfiguration`、`@Import` 等属于后续自动配置实现原理，不需要在概述阶段展开。
    

> [!tip] 一句话总结
> 
> `Spring Boot自动配置是在Spring容器启动过程中自动将符合条件的配置类和Bean注册到IOC容器，从而减少开发者手动声明Bean和配置框架的工作；Starter主要解决依赖引入问题，而自动配置主要解决依赖引入之后的Bean和配置问题。`