# 1. 为什么需要条件装配

前面已经学习了 Spring Boot 自动配置原理。

Spring Boot 会通过自动配置机制找到大量自动配置类，而这些自动配置类内部又可能声明很多：

```java
@Bean
```

方法。

但是这并不意味着：

> **所有自动配置类中的所有 Bean 都会无条件注册到 IOC 容器。**

因为不同项目的运行环境不同。

例如：

- 有些项目引入了某个依赖；
    
- 有些项目没有引入；
    
- 有些项目已经自己声明了某个 Bean；
    
- 有些项目希望通过配置文件开启某项功能；
    
- 有些项目并不需要这项功能。
    

如果 Spring Boot 不进行任何判断，直接把所有可能的 Bean 全部注册到 IOC 容器，就无法根据当前项目环境灵活完成配置。

因此 Spring Boot 提供了：

> **条件装配。**

---

# 2. 什么是条件装配

条件装配可以理解为：

> **在注册 Bean 之前先进行条件判断，只有满足指定条件时，才将对应 Bean 注册到 Spring IOC 容器。**

材料中将这一类注解称为：

```java
@Conditional
```

相关注解。

它们可以作用在：

- 类上
    
- 方法上
    

当前内容重点介绍了三种常见条件注解：

```java
@ConditionalOnClass
@ConditionalOnMissingBean
@ConditionalOnProperty
```

它们分别从不同角度判断当前环境是否满足 Bean 注册条件。

---

# 3. 条件装配与自动配置

自动配置可以分成两个阶段理解。

前一个阶段解决：

> **有哪些自动配置类可以参与当前项目的配置？**

后一个阶段则进一步判断：

> **这些配置类中的 Bean 到底要不要真正创建？**

因此可以理解为：

```text
自动配置找到候选配置类
↓
读取配置类中的 Bean 定义
↓
检查条件装配注解
↓
条件满足
→ 注册 Bean

条件不满足
→ 不注册 Bean
```

所以：

> **自动配置负责提供候选配置，而条件装配负责根据当前项目环境决定哪些配置真正生效。**

---

# 4. @ConditionalOnClass

`@ConditionalOnClass` 用于判断：

> **当前运行环境中是否存在指定类的字节码文件。**

只有指定类存在时，对应 Bean 才会注册到 IOC 容器。

例如：

```java
@Configuration
public class HeaderConfig {

    @Bean
    @ConditionalOnClass(name = "io.jsonwebtoken.Jwts")
    public HeaderParser headerParser() {
        return new HeaderParser();
    }
}
```

这里的条件是：

```text
当前环境中存在
io.jsonwebtoken.Jwts
```

如果存在，就创建：

```text
HeaderParser
```

并注册到 IOC 容器。

材料中配合 JWT 依赖进行了演示：

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

当 `io.jsonwebtoken.Jwts` 字节码文件存在时，`HeaderParser` 对象就会创建并进入 IOC 容器。

因此可以简单理解为：

```text
指定类存在
↓
条件成立
↓
创建 Bean
```

---

# 5. @ConditionalOnClass 的作用

`@ConditionalOnClass` 很适合表达一种：

> **“只有项目具备某项技术或依赖时，才启用相关配置”**

的逻辑。

例如材料中的：

```java
@ConditionalOnClass(name = "io.jsonwebtoken.Jwts")
```

本质是在判断：

> 当前项目环境中有没有 JWT 相关的类。

只有存在时，相关 Bean 才有创建的前提。

因此：

```java
@ConditionalOnClass
```

关注的是：

> **类是否存在。**

---

# 6. @ConditionalOnMissingBean

`@ConditionalOnMissingBean` 用于判断：

> **当前 IOC 容器中是否缺少指定的 Bean。**

只有不存在对应 Bean 时，Spring 才会继续创建当前 Bean。

例如：

```java
@Configuration
public class HeaderConfig {

    @Bean
    @ConditionalOnMissingBean
    public HeaderParser headerParser() {
        return new HeaderParser();
    }
}
```

在这个例子中，如果 IOC 容器中还没有：

```text
HeaderParser
```

类型的 Bean，那么：

```java
headerParser()
```

方法就会正常执行，并把返回的对象注册到 IOC 容器。

因此：

```text
IOC容器中没有对应Bean
↓
条件成立
↓
创建默认Bean
```

---

# 7. 为什么需要 @ConditionalOnMissingBean

它解决的是一个非常重要的问题：

> **如果项目自己已经提供了某个 Bean，自动配置是否还要再提供一个？**

按照当前材料的条件逻辑：

```text
不存在对应Bean
→ 自动配置创建

已经存在对应Bean
→ 条件不成立
```

因此可以理解为：

> `@ConditionalOnMissingBean` 可以让自动配置只在“当前容器还没有对应 Bean”时生效。

这样自动配置就不会简单地无条件创建 Bean。

材料也明确说明，它可以根据 Bean 的类型或名称判断是否存在对应 Bean。

---

# 8. @ConditionalOnProperty

除了判断类和 Bean 是否存在，还可以根据配置文件中的属性决定是否注册 Bean。

这就是：

```java
@ConditionalOnProperty
```

它用于判断：

> **配置文件中是否存在指定属性以及对应值。**

例如：

```yaml
name: itheima
```

配置类：

```java
@Configuration
public class HeaderConfig {

    @Bean
    @ConditionalOnProperty(
        name = "name",
        havingValue = "itheima"
    )
    public HeaderParser headerParser() {
        return new HeaderParser();
    }
}
```

这里要求配置文件中的：

```text
name
```

值必须满足：

```text
itheima
```

条件满足时，`HeaderParser` 才会注册到 IOC 容器。

---

# 9. @ConditionalOnProperty 条件不满足

如果把条件修改为：

```java
@Bean
@ConditionalOnProperty(
    name = "name",
    havingValue = "itheima2"
)
public HeaderParser headerParser() {
    return new HeaderParser();
}
```

但是配置文件仍然只有：

```yaml
name: itheima
```

那么：

```text
itheima
≠
itheima2
```

条件不成立。

因此：

```text
HeaderParser
```

不会注册到 IOC 容器。

材料中的测试结果也说明，当配置文件不存在满足条件的：

```text
name: itheima2
```

时，IOC 容器中就不存在 `HeaderParser` 对象。

因此：

```java
@ConditionalOnProperty
```

关注的是：

> **配置属性是否满足要求。**

---

# 10. 三种条件装配方式对比

当前阶段可以重点掌握这三个条件：

|注解|判断内容|条件成立时|
|---|---|---|
|`@ConditionalOnClass`|指定类是否存在|注册 Bean|
|`@ConditionalOnMissingBean`|IOC 容器中是否缺少对应 Bean|注册 Bean|
|`@ConditionalOnProperty`|配置文件中的属性和值是否满足要求|注册 Bean|

可以分别记成：

```text
@ConditionalOnClass
→ 看“类”

@ConditionalOnMissingBean
→ 看“Bean”

@ConditionalOnProperty
→ 看“配置”
```

三者虽然判断依据不同，但最终解决的是同一个问题：

> **当前 Bean 是否应该注册到 IOC 容器。**

---

# 11. 条件装配让自动配置具有选择性

如果自动配置只有：

```java
@Bean
```

而没有条件判断，那么自动配置类一旦被加载，其中的 Bean 就可能直接参与注册。

加入条件装配以后，就可以根据：

```text
当前是否存在某个类
当前是否已经存在某个Bean
当前配置文件是否满足指定条件
```

选择性地进行 Bean 注册。

因此自动配置不再只是：

```text
发现配置类
→ 创建Bean
```

而是：

```text
发现配置类
↓
判断当前环境
↓
满足条件
↓
创建对应Bean
```

这就是 Spring Boot 自动配置能够根据不同项目环境进行选择性配置的重要原因。

---

# 12. @Bean 与条件装配的配合

条件装配经常和：

```java
@Bean
```

一起出现。

例如：

```java
@Bean
@ConditionalOnClass(name = "io.jsonwebtoken.Jwts")
public HeaderParser headerParser() {
    return new HeaderParser();
}
```

这里：

```java
@Bean
```

解决的是：

> 方法返回的对象需要注册为 Bean。

而：

```java
@ConditionalOnClass
```

解决的是：

> 这个 Bean 在当前环境下到底应不应该注册。

因此可以理解为：

```text
@Bean
→ 定义“要注册什么”

@Conditional...
→ 判断“现在要不要注册”
```

二者职责不同。

---

# 13. 条件装配与 IOC 容器

无论使用哪一种条件注解，最终结果仍然回到 IOC 容器。

条件成立：

```text
Bean注册到IOC容器
```

条件不成立：

```text
Bean不进入IOC容器
```

因此条件装配并不是独立于 IOC 的一套机制，而是：

> **控制 Bean 是否进入 IOC 容器的一种条件判断机制。**

这也说明 Spring Boot 自动配置最终仍然建立在 Spring IOC 和 Bean 管理体系之上。

---

# 14. 当前阶段需要掌握的核心内容

1. Spring Boot 找到自动配置类之后，并不会无条件创建其中的所有 Bean。
    
2. 条件装配用于在 Bean 注册之前判断当前环境是否满足要求。
    
3. 条件满足时，Bean 才会注册到 IOC 容器。
    
4. 当前内容重点介绍 `@ConditionalOnClass`、`@ConditionalOnMissingBean`、`@ConditionalOnProperty`。
    
5. `@ConditionalOnClass` 根据指定类是否存在决定是否注册 Bean。
    
6. `@ConditionalOnMissingBean` 根据 IOC 容器中是否缺少对应 Bean 决定是否注册。
    
7. `@ConditionalOnProperty` 根据配置文件中的属性和值决定是否注册 Bean。
    
8. `@Bean` 负责声明 Bean，Conditional 类注解负责判断当前 Bean 是否应该创建。
    
9. 条件装配使 Spring Boot 自动配置能够根据不同项目环境选择性地注册 Bean。
    
10. 自动配置最终仍然是围绕 Spring IOC 容器完成 Bean 管理。
    

> [!tip] 一句话总结
> 
> `Spring Boot条件装配会在Bean注册前根据当前环境进行判断，@ConditionalOnClass检查类是否存在，@ConditionalOnMissingBean检查容器中是否缺少对应Bean，@ConditionalOnProperty检查配置属性是否满足要求，只有条件成立时对应Bean才会注册到IOC容器。`