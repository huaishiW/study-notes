# 1. 自动配置原理要解决什么问题

前面已经知道，Spring Boot 自动配置的作用是：

> **在 Spring 容器启动过程中，将一些配置类和 Bean 自动加载到 IOC 容器中，从而减少开发者手动配置。**

真正需要继续解决的问题是：

> **Spring Boot 到底是怎么知道应该加载哪些自动配置类的？**

因为第三方依赖中的配置类通常不在当前项目启动类所在包中，因此不能简单依赖默认组件扫描完成。

所以自动配置原理的核心问题可以进一步描述为：

> **Spring Boot 如何找到依赖中的自动配置类，并将这些配置类交给 IOC 容器管理。**

---

# 2. 自动配置的源码入口

分析 Spring Boot 自动配置时，入口是：

```java
@SpringBootApplication
```

当前材料重点关注它包含的三个核心注解：

```java
@SpringBootConfiguration
@ComponentScan
@EnableAutoConfiguration
```

其中：

```java
@EnableAutoConfiguration
```

才是自动配置机制的核心入口。

因此，分析主线可以从：

```text
@SpringBootApplication
↓
@EnableAutoConfiguration
```

继续向下。

---

# 3. @EnableAutoConfiguration

`@EnableAutoConfiguration` 的作用是：

> **开启 Spring Boot 自动配置。**

当前材料指出，`@EnableAutoConfiguration` 内部使用了：

```java
@Import
```

而 `@Import` 导入的是一个实现：

```java
ImportSelector
```

接口的类。

这个实现类就是：

```java
AutoConfigurationImportSelector
```

因此可以形成：

```text
@EnableAutoConfiguration
↓
@Import
↓
AutoConfigurationImportSelector
```

这里的设计和前面学习的：

```text
@Import
+
ImportSelector
```

是一致的。

区别只是 Spring Boot 不再使用我们自己编写的 `MyImportSelector`，而是使用框架内部提供的：

```java
AutoConfigurationImportSelector
```

当前材料明确指出 `AutoConfigurationImportSelector` 是 `ImportSelector` 的实现类。

---

# 4. AutoConfigurationImportSelector

`AutoConfigurationImportSelector` 的职责可以理解为：

> **决定 Spring Boot 需要导入哪些自动配置类。**

由于它实现了：

```java
ImportSelector
```

所以其中需要处理：

```java
selectImports()
```

方法。

前面已经知道：

> `ImportSelector` 的 `selectImports()` 返回哪些类，就代表希望 Spring 导入哪些类。

因此，Spring Boot 自动配置的关键问题就变成：

> **AutoConfigurationImportSelector 的 selectImports() 是如何得到自动配置类集合的？**

---

# 5. selectImports()

当前材料对源码的跟踪从：

```java
selectImports()
```

继续向下。

`AutoConfigurationImportSelector` 中重写了：

```java
selectImports()
```

该方法会进一步调用：

```java
getAutoConfigurationEntry()
```

获取自动配置相关信息。

因此当前源码主线可以继续整理为：

```text
AutoConfigurationImportSelector
↓
selectImports()
↓
getAutoConfigurationEntry()
```

当前材料明确指出，`selectImports()` 底层会调用 `getAutoConfigurationEntry()` 获取可用于自动配置的配置类信息。

---

# 6. getAutoConfigurationEntry()

`getAutoConfigurationEntry()` 会继续调用：

```java
getCandidateConfigurations()
```

获取自动配置候选类。

这里的“候选类”可以理解为：

> **Spring Boot 当前能够找到、后续可能参与自动配置的一批配置类。**

因此主线进一步变成：

```text
selectImports()
↓
getAutoConfigurationEntry()
↓
getCandidateConfigurations()
```

当前材料明确指出：

```text
getAutoConfigurationEntry()
```

会调用：

```text
getCandidateConfigurations()
```

获取配置文件中定义的自动配置类集合。

---

# 7. AutoConfiguration.imports

`getCandidateConfigurations()` 需要知道：

> 到底有哪些类属于自动配置类？

当前材料重点跟踪到的配置文件是：

```text
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

这个文件中记录了大量自动配置类的全限定类名。

可以理解为：

```text
AutoConfiguration.imports
↓
记录自动配置类
↓
Spring Boot读取这些类名
↓
得到候选自动配置类集合
```

因此：

```java
AutoConfigurationImportSelector
```

并不是自己硬编码所有自动配置类。

而是通过读取：

```text
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

获得需要进一步处理的自动配置类。

当前材料明确指出 `getCandidateConfigurations()` 会从该文件中获取配置类集合。

---

# 8. GsonAutoConfiguration 示例

例如：

```text
GsonAutoConfiguration
```

就是一个自动配置类。

当前材料通过搜索：

```text
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

发现其中配置了：

```text
GsonAutoConfiguration
```

然后继续查看：

```java
GsonAutoConfiguration
```

本身。

材料指出，这个类属于一个配置类，其中会通过：

```java
@Bean
```

声明需要注册到 IOC 容器中的对象。

因此前面出现：

```java
@Autowired
private Gson gson;
```

即使自己没有编写：

```java
@Bean
public Gson gson() {
    return new Gson();
}
```

仍然能够获取 Gson Bean。

原因就在于：

> Spring Boot 找到了 `GsonAutoConfiguration`，并通过其中的 Bean 配置完成了对象注册。

---

# 9. 自动配置类最终如何产生 Bean

自动配置类本身仍然属于：

> **Spring 配置类。**

配置类内部仍然可以使用：

```java
@Bean
```

声明 Bean。

例如可以抽象成：

```java
@Configuration
public class XxxAutoConfiguration {

    @Bean
    public Xxx xxx() {
        return new Xxx();
    }
}
```

当这个自动配置类被 Spring Boot 导入以后，其中的：

```java
@Bean
```

方法就会参与 Bean 注册。

因此，自动配置并没有发明一套完全不同的 Bean 管理机制。

最终还是回到了前面学习过的：

```text
配置类
↓
@Bean
↓
方法返回对象
↓
注册到IOC容器
```

自动配置真正新增的是：

> **配置类不再完全由开发者手动导入，而是由 Spring Boot 自动找到并导入。**

当前材料也明确把自动配置归结为：配置类中定义 `@Bean` 方法，Spring 处理这些配置类并将对应对象注册到 IOC 容器。

---

# 10. 自动配置完整主线

到这里，可以把当前阶段的自动配置主线整理为：

```text
@SpringBootApplication
↓
@EnableAutoConfiguration
↓
@Import
↓
AutoConfigurationImportSelector
↓
selectImports()
↓
getAutoConfigurationEntry()
↓
getCandidateConfigurations()
↓
读取 AutoConfiguration.imports
↓
获得自动配置类
↓
导入自动配置类
↓
处理配置类中的 @Bean
↓
Bean注册到IOC容器
```

这条主线是理解 Spring Boot 自动配置最重要的内容。

---

# 11. 为什么自动配置不依赖默认组件扫描

前面已经知道：

```java
@ComponentScan
```

默认只能扫描启动类所在包以及其子包。

而 Spring Boot 提供的自动配置类可能位于其他 Jar 包中。

因此自动配置并不是简单地：

> 把所有依赖包全部扫描一遍。

而是通过：

```text
@EnableAutoConfiguration
+
@Import
+
AutoConfigurationImportSelector
```

主动找到并导入自动配置类。

因此可以区分：

|机制|主要作用|
|---|---|
|`@ComponentScan`|在指定包范围内发现组件|
|自动配置导入|根据自动配置机制导入配置类|

所以：

> **组件扫描和自动配置虽然最终都可能让 Bean 进入 IOC 容器，但它们找到 Bean 配置来源的方式不同。**

---

# 12. 为什么需要 AutoConfiguration.imports

如果没有一个统一位置记录自动配置类，那么：

```java
AutoConfigurationImportSelector
```

就需要自己知道所有：

```text
Web相关配置类
JSON相关配置类
数据库相关配置类
其他配置类
```

这种方式不利于扩展。

而通过：

```text
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

记录自动配置类之后：

> 自动配置类的声明和自动配置选择逻辑可以相对分离。

Spring Boot 只需要按照既定机制读取配置类信息，就能够获得自动配置候选类。

因此这个文件承担的核心作用是：

> **告诉 Spring Boot 当前依赖中有哪些自动配置类可以参与自动配置。**

---

# 13. 所有自动配置 Bean 都会创建吗

读取：

```text
AutoConfiguration.imports
```

以后，会得到大量自动配置类。

但是这并不意味着：

> 所有自动配置类中的所有 Bean 都一定会被创建。

当前材料明确指出：

> 自动配置类中的 Bean 通常还会受到 `@Conditional` 一类条件装配注解的控制，只有满足条件时才会真正注册到 IOC 容器。

因此，当前可以先建立：

```text
发现自动配置类
≠
其中所有Bean一定创建
```

真正是否创建 Bean，还需要判断：

```text
当前环境是否满足条件
```

这就是后续：

```java
@Conditional
```

条件装配需要解决的问题。

这一篇先不展开具体条件注解。

---

# 14. 自动配置的两个阶段

为了更容易理解，可以把当前材料中的自动配置过程拆成两个阶段。

## 14.1 找到自动配置类

这一阶段主要包括：

```text
@EnableAutoConfiguration
↓
AutoConfigurationImportSelector
↓
AutoConfiguration.imports
↓
获得自动配置类
```

解决的问题是：

> **有哪些配置类可以参与自动配置？**

## 14.2 根据配置类注册 Bean

自动配置类进入 Spring 后：

```text
自动配置类
↓
@Bean
↓
根据条件决定是否创建
↓
Bean进入IOC容器
```

解决的问题是：

> **自动配置类中的哪些 Bean 最终真正生效？**

因此：

> **自动配置不是简单地“找到 Bean”，而是先找到候选自动配置类，再根据具体配置决定 Bean 是否注册。**

---

# 15. 当前阶段需要掌握的核心内容

1. Spring Boot 自动配置源码分析从 `@SpringBootApplication` 开始。
    
2. `@EnableAutoConfiguration` 是自动配置的核心入口。
    
3. `@EnableAutoConfiguration` 内部使用 `@Import`。
    
4. `@Import` 导入了 `AutoConfigurationImportSelector`。
    
5. `AutoConfigurationImportSelector` 实现了 `ImportSelector`。
    
6. `AutoConfigurationImportSelector` 重写了 `selectImports()`。
    
7. `selectImports()` 会继续调用 `getAutoConfigurationEntry()`。
    
8. `getAutoConfigurationEntry()` 会调用 `getCandidateConfigurations()` 获取候选自动配置类。
    
9. 当前材料重点通过 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 获取自动配置类。
    
10. `AutoConfiguration.imports` 中记录的是自动配置类信息。
    
11. 自动配置类本质上仍然是配置类，可以通过 `@Bean` 声明 Bean。
    
12. 自动配置最终仍然依赖 Spring IOC 容器完成 Bean 管理。
    
13. Spring Boot 自动配置不是通过扩大默认组件扫描范围实现的。
    
14. 找到自动配置类并不代表其中所有 Bean 都一定创建。
    
15. Bean 最终是否注册，还可能受到 `@Conditional` 条件装配机制控制。
    

> [!tip] 一句话总结
> 
> `Spring Boot自动配置以@EnableAutoConfiguration为核心入口，通过@Import导入AutoConfigurationImportSelector，再由其读取AutoConfiguration.imports获取候选自动配置类，最终处理这些配置类中的@Bean并将满足条件的对象注册到IOC容器。`