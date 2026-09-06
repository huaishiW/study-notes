# 1. @SpringBootApplication 的作用

Spring Boot 应用的启动类通常会使用：

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

`@SpringBootApplication` 是 Spring Boot 启动类上的核心注解。

在分析 Spring Boot 自动配置原理时，可以把它作为一个重要入口。

当前材料中重点关注它包含的三个核心注解：

```java
@SpringBootConfiguration
@ComponentScan
@EnableAutoConfiguration
```

这三个注解分别承担不同职责：

|注解|主要职责|
|---|---|
|`@SpringBootConfiguration`|表明启动类是 Spring 配置类|
|`@ComponentScan`|执行组件扫描|
|`@EnableAutoConfiguration`|开启 Spring Boot 自动配置|

因此可以先建立整体认识：

> `@SpringBootApplication` 并不是只完成一件事情，而是同时整合了配置类声明、组件扫描和自动配置等能力。

---

# 2. @SpringBootConfiguration

`@SpringBootApplication` 中包含：

```java
@SpringBootConfiguration
```

当前材料进一步指出，`@SpringBootConfiguration` 上使用了：

```java
@Configuration
```

因此：

> **Spring Boot 启动类本身也是一个 Spring 配置类。**

例如：

```java
@SpringBootApplication
public class Application {
}
```

因为 `@SpringBootApplication` 中包含 `@SpringBootConfiguration`，而 `@SpringBootConfiguration` 又具有 `@Configuration` 的作用，所以当前启动类具备配置类的身份。

这也解释了为什么前面可以直接在启动类中编写：

```java
@Bean
public AliyunOSSOperator aliyunOSSOperator(
        AliyunOSSProperties ossProperties) {

    return new AliyunOSSOperator(ossProperties);
}
```

因为启动类本身就是一个配置类。

不过在实际项目中，如果 Bean 配置较多，通常仍然更适合单独创建配置类集中管理，而不是把大量 `@Bean` 方法全部放到启动类中。

---

# 3. @ComponentScan

`@SpringBootApplication` 中还包含：

```java
@ComponentScan
```

它负责：

> **组件扫描。**

Spring Boot 启动后，会扫描启动类所在包及其子包中的组件类。

例如：

```text
com.huaishi
├── Application.java
├── controller
├── service
└── mapper
```

如果启动类：

```text
Application.java
```

位于：

```text
com.huaishi
```

那么默认扫描范围会覆盖：

```text
com.huaishi
com.huaishi.controller
com.huaishi.service
com.huaishi.mapper
```

因此这些包中使用：

```java
@Component
@Controller
@Service
@Repository
```

等组件注解声明的类，可以被 Spring 扫描发现并注册到 IOC 容器。

---

# 4. 为什么 Spring Boot 通常不用手动写 @ComponentScan

普通 Spring 配置中，可以显式使用：

```java
@ComponentScan
```

指定组件扫描范围。

但是在 Spring Boot 项目中，启动类通常只需要：

```java
@SpringBootApplication
```

就已经具备默认的组件扫描能力。

原因就是：

> `@SpringBootApplication` 本身已经包含 `@ComponentScan`。

因此，Spring Boot 应用运行时能够自动扫描启动类所在包及其子包。

这一点也是启动类通常放在项目根包位置的重要原因。

---

# 5. @ComponentScan 不能解释完整的自动配置

虽然 `@ComponentScan` 可以扫描当前项目中的组件，但它存在一个明显的范围限制：

> 默认只扫描启动类所在包以及其子包。

例如：

```text
com.itheima
└── Application.java
```

而第三方依赖中的类位于：

```text
com.example
```

那么：

```text
com.example
```

并不是：

```text
com.itheima
```

的子包，因此默认组件扫描无法发现其中的组件。

这说明：

> **Spring Boot 自动配置不能仅仅依赖普通的组件扫描。**

虽然可以通过：

```java
@ComponentScan({
        "com.itheima",
        "com.example"
})
```

手动扩大扫描范围，但如果项目引入大量第三方依赖，就需要不断配置额外扫描包。

这种方式会变得繁琐，而且材料中明确指出 Spring Boot 并没有采用这种方式作为自动配置的核心方案。

因此，`@ComponentScan` 主要负责：

> **当前应用自身组件的自动发现。**

而真正负责自动配置的是：

```java
@EnableAutoConfiguration
```

---

# 6. @EnableAutoConfiguration

`@SpringBootApplication` 中第三个非常重要的注解是：

```java
@EnableAutoConfiguration
```

它是当前材料中明确指出的：

> **Spring Boot 自动配置的核心注解。**

它负责开启 Spring Boot 的自动配置能力。

也就是说：

```text
@SpringBootApplication
        ↓
@EnableAutoConfiguration
        ↓
开启自动配置
```

前面已经知道，自动配置的目标是：

> 将符合条件的配置类和 Bean 自动注册到 Spring IOC 容器。

而 `@EnableAutoConfiguration` 就是这套机制的重要入口。

---

# 7. @EnableAutoConfiguration 与 @Import

当前材料继续指出：

```java
@EnableAutoConfiguration
```

内部使用了：

```java
@Import
```

并通过 `@Import` 导入一个实现了：

```java
ImportSelector
```

接口的类。

其中涉及到：

```java
AutoConfigurationImportSelector
```

它是：

```java
ImportSelector
```

的实现类。

因此可以先建立这样的认识：

```text
@EnableAutoConfiguration
        ↓
@Import
        ↓
ImportSelector
        ↓
AutoConfigurationImportSelector
```

不过，这一篇只需要知道：

> `@EnableAutoConfiguration` 会继续借助 `@Import` 和 `ImportSelector` 去完成自动配置类的导入。

至于：

- `@Import` 可以导入哪些内容；
    
- `ImportSelector` 是什么；
    
- `selectImports()` 如何工作；
    
- `AutoConfigurationImportSelector` 如何找到自动配置类；
    

这些将在后续笔记中单独展开。

---

# 8. 三个核心注解的职责划分

现在可以把 `@SpringBootApplication` 的三个核心组成整理为：

|注解|核心作用|当前负责的范围|
|---|---|---|
|`@SpringBootConfiguration`|声明配置类|让启动类具有 Spring 配置类身份|
|`@ComponentScan`|组件扫描|扫描启动类所在包及其子包|
|`@EnableAutoConfiguration`|开启自动配置|导入 Spring Boot 提供的自动配置|

它们分别解决不同问题。

## 8.1 @SpringBootConfiguration

解决：

> 当前启动类是不是一个配置类。

## 8.2 @ComponentScan

解决：

> 当前项目中的组件如何被 Spring 自动发现。

## 8.3 @EnableAutoConfiguration

解决：

> Spring Boot 如何开启自动配置能力。

因此不能把：

```java
@ComponentScan
```

和：

```java
@EnableAutoConfiguration
```

理解成同一种机制。

前者主要负责扫描当前应用组件，后者负责自动配置。

---

# 9. @SpringBootApplication 与自动配置的整体关系

Spring Boot 自动配置的源码分析通常从：

```java
@SpringBootApplication
```

开始。

但真正需要继续深入跟踪的是：

```java
@EnableAutoConfiguration
```

因为当前材料明确指出：

> `@EnableAutoConfiguration` 才是自动配置的核心。

所以后续分析可以形成这样的入口：

```text
@SpringBootApplication
        ↓
找到 @EnableAutoConfiguration
        ↓
继续分析 @Import
        ↓
继续分析 ImportSelector
        ↓
继续分析自动配置类如何被找到并导入
```

而：

```java
@SpringBootConfiguration
```

和：

```java
@ComponentScan
```

虽然同样非常重要，但它们并不是自动配置类导入机制的核心部分。

---

# 10. 当前阶段需要掌握的核心内容

1. `@SpringBootApplication` 是 Spring Boot 启动类上的核心注解。
    
2. 当前材料重点关注它包含的三个注解：`@SpringBootConfiguration`、`@ComponentScan`、`@EnableAutoConfiguration`。
    
3. `@SpringBootConfiguration` 上包含 `@Configuration`，因此 Spring Boot 启动类本身也是配置类。
    
4. `@ComponentScan` 负责扫描启动类所在包及其子包中的组件。
    
5. Spring Boot 默认组件扫描无法直接扫描与启动类包无父子关系的第三方依赖包。
    
6. 手动扩大 `@ComponentScan` 范围可以解决部分问题，但不是 Spring Boot 自动配置采用的核心方式。
    
7. `@EnableAutoConfiguration` 是自动配置的核心注解。
    
8. `@EnableAutoConfiguration` 内部继续使用 `@Import`。
    
9. `@Import` 会涉及 `ImportSelector` 和 `AutoConfigurationImportSelector`。
    
10. 当前只需要掌握三个核心注解的职责，不需要在这一篇深入源码细节。
    

> [!tip] 一句话总结
> 
> `@SpringBootApplication整合了@SpringBootConfiguration、@ComponentScan和@EnableAutoConfiguration三项核心能力，分别负责声明配置类、扫描应用组件和开启自动配置，其中@EnableAutoConfiguration是Spring Boot自动配置机制的核心入口。`