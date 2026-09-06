# 1. 为什么需要自定义 Starter

Spring Boot 已经提供了大量 Starter，用来简化不同开发场景下的依赖配置。

但是实际项目中，还会使用很多：

- 第三方 SDK
    
- 公司内部公共组件
    
- 多个项目重复使用的工具
    
- Spring Boot 官方没有提供 Starter 的技术
    

如果每一个项目都需要重复完成：

```text
引入依赖
配置属性
创建配置类
声明Bean
编写工具类
```

就会产生大量重复配置。

因此，可以把这些公共能力进一步封装成：

```text
自定义 Spring Boot Starter
```

这样其他项目使用时，只需要引入对应 Starter，就能够获得预先配置好的功能。

---

# 2. 自定义 Starter 的目标

当前内容以：

```text
阿里云 OSS
```

为例。

如果不进行 Starter 封装，项目通常需要自己完成：

- 引入 OSS SDK 相关依赖；
    
- 编写 OSS 相关配置；
    
- 读取 `endpoint`、`bucketName` 等参数；
    
- 创建 OSS 操作工具类；
    
- 将相关对象交给 Spring 管理。
    

如果其他项目也需要使用 OSS，就会重复相同操作。

因此，自定义 Starter 的目标是：

> **把公共依赖和自动配置提前封装好，让使用项目只负责引入 Starter 和提供必要配置。**

最终希望达到：

```java
@Autowired
private AliyunOSSOperator aliyunOSSOperator;
```

直接注入后即可使用。

材料中对这个案例的目标描述也是：引入 `aliyun-oss-spring-boot-starter` 后，直接注入 `AliyunOSSOperator` 使用。

---

# 3. 自定义 Starter 的模块划分

当前材料按照常见 Starter 结构，将自定义 Starter 拆分成两个模块：

```text
aliyun-oss-spring-boot-starter
aliyun-oss-spring-boot-autoconfigure
```

两者职责不同。

|模块|主要职责|
|---|---|
|`xxx-spring-boot-starter`|依赖管理|
|`xxx-spring-boot-autoconfigure`|自动配置|

材料明确指出，Starter 模块主要负责依赖管理，而 autoconfigure 模块负责自动配置，自动配置核心代码编写在 autoconfigure 中。

---

# 4. starter 模块

以：

```text
aliyun-oss-spring-boot-starter
```

为例。

这个模块主要负责：

> **统一管理使用当前技术所需要的依赖。**

因此 Starter 本身通常不会承载大量业务代码，它更像一个：

```text
依赖入口
```

使用项目只需要：

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-oss-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

就可以间接获得 Starter 内部组织的相关依赖。

---

# 5. autoconfigure 模块

第二个模块是：

```text
aliyun-oss-spring-boot-autoconfigure
```

它负责：

> **编写自动配置相关代码。**

其中通常需要放置：

- 配置属性类；
    
- 自动配置类；
    
- 需要自动注册的 Bean；
    
- 对应第三方技术所需要的依赖；
    
- 自动配置类声明文件。
    

因此，可以简单区分：

```text
starter
→ 管依赖

autoconfigure
→ 管自动配置
```

---

# 6. starter 为什么依赖 autoconfigure

Starter 模块还需要依赖：

```text
autoconfigure
```

例如：

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-oss-spring-boot-autoconfigure</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

这样使用项目只需要引入：

```text
aliyun-oss-spring-boot-starter
```

Maven 就会通过依赖传递，把：

```text
aliyun-oss-spring-boot-autoconfigure
```

一并引入。

材料也明确说明，将来使用时只需要引入 Starter，因为 autoconfigure 会通过依赖传递一起进入项目。

因此最终依赖关系可以理解为：

```text
业务项目
↓
aliyun-oss-spring-boot-starter
↓
aliyun-oss-spring-boot-autoconfigure
↓
OSS相关依赖和自动配置代码
```

---

# 7. 自定义 Starter 的实现步骤

当前材料将整个实现过程概括为三步：

1. 创建 Starter 模块；
    
2. 创建 autoconfigure 模块；
    
3. 在 autoconfigure 中完成自动配置。
    

具体包括：

```text
Starter负责依赖管理
Autoconfigure负责自动配置
自动配置类注册到AutoConfiguration.imports
```

材料对此给出了明确的三步实现方案。

---

# 8. 配置属性类

为了让使用者可以在：

```yaml
application.yml
```

中配置 OSS 参数，需要定义一个配置属性类。

例如：

```java
@Data
@ConfigurationProperties(prefix = "aliyun.oss")
public class AliyunOSSProperties {

    private String endpoint;
    private String bucketName;
}
```

其中：

```java
@ConfigurationProperties(prefix = "aliyun.oss")
```

表示：

> 将配置文件中以 `aliyun.oss` 开头的属性绑定到当前 Java 对象。

对应配置可以写成：

```yaml
aliyun:
  oss:
    endpoint: xxx
    bucket-name: xxx
```

当前材料使用 `AliyunOSSProperties` 保存 `endpoint`、`bucketName` 等配置。

---

# 9. 自动配置类

接下来需要定义：

```text
AliyunOSSAutoConfiguration
```

例如：

```java
@Configuration
@EnableConfigurationProperties(AliyunOSSProperties.class)
public class AliyunOSSAutoConfiguration {

    @Bean
    public AliyunOSSOperator aliyunOSSOperator(
            AliyunOSSProperties aliyunOSSProperties) {

        return new AliyunOSSOperator(aliyunOSSProperties);
    }
}
```

这里包含两个重要部分。

## 9.1 EnableConfigurationProperties

```java
@EnableConfigurationProperties(AliyunOSSProperties.class)
```

用于让：

```text
AliyunOSSProperties
```

配置属性类参与当前自动配置。

这样配置文件中的 OSS 属性就可以绑定到：

```java
AliyunOSSProperties
```

对象中。

## 9.2 @Bean

```java
@Bean
public AliyunOSSOperator aliyunOSSOperator(
        AliyunOSSProperties aliyunOSSProperties) {

    return new AliyunOSSOperator(aliyunOSSProperties);
}
```

负责创建：

```text
AliyunOSSOperator
```

并注册到 IOC 容器。

当前材料就是通过 `@EnableConfigurationProperties + @Bean` 完成这个自动配置类。

---

# 10. 配置属性如何传给工具类

`AliyunOSSOperator` 在工作时需要使用：

```text
endpoint
bucketName
```

等配置。

因此自动配置类通过：

```java
AliyunOSSProperties aliyunOSSProperties
```

方法参数获得配置属性 Bean。

然后：

```java
return new AliyunOSSOperator(aliyunOSSProperties);
```

将配置传递给工具类。

这正好复用了前面学习过的：

> `@Bean` 方法参数可以由 Spring IOC 容器自动注入。

所以自定义 Starter 并没有引入新的 Bean 管理体系，它仍然建立在：

```text
IOC
@Bean
配置属性
自动配置
```

这些已有机制之上。

---

# 11. 注册自动配置类

仅仅创建：

```java
AliyunOSSAutoConfiguration
```

还不够。

Spring Boot 还需要知道：

> **这个类是需要参与自动配置的类。**

因此材料要求在：

```text
src/main/resources
```

下创建：

```text
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

并写入自动配置类的全限定名：

```text
com.aliyun.oss.AliyunOSSAutoConfiguration
```

材料明确说明，Spring Boot 启动时会读取这个文件并加载其中配置的自动配置类。

---

# 12. AutoConfiguration.imports 的作用

前面学习 Spring Boot 自动配置原理时已经知道：

```text
AutoConfiguration.imports
```

用于记录候选自动配置类。

在自定义 Starter 中，我们现在从：

> **使用者**

变成了：

> **Starter 的提供者。**

因此需要主动告诉 Spring Boot：

```text
我的依赖中提供了一个自动配置类
```

具体就是在：

```text
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

中登记：

```text
AliyunOSSAutoConfiguration
```

这样 Spring Boot 才能在启动过程中发现它。

---

# 13. 自定义 Starter 的整体工作过程

自定义 Starter 完成以后，业务项目只需要引入：

```xml
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-oss-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

然后配置：

```yaml
aliyun:
  oss:
    endpoint: xxx
    bucket-name: xxx
```

最终就可以：

```java
@Autowired
private AliyunOSSOperator aliyunOSSOperator;
```

背后的过程可以整理为：

```text
业务项目引入Starter
↓
Maven传递引入autoconfigure
↓
Spring Boot读取AutoConfiguration.imports
↓
发现AliyunOSSAutoConfiguration
↓
加载AliyunOSSProperties
↓
读取application.yml配置
↓
执行@Bean方法
↓
创建AliyunOSSOperator
↓
注册到IOC容器
↓
业务代码直接注入使用
```

这就是自定义 Starter 的完整核心逻辑。

---

# 14. 自定义 Starter 的本质

从表面上看，自定义 Starter 是：

> 自己制作一个依赖。

但从实现原理来看，它实际上同时组合了两部分能力。

## 14.1 依赖封装

通过：

```text
xxx-spring-boot-starter
```

统一组织开发所需依赖。

解决：

> **项目需要引入哪些依赖。**

## 14.2 自动配置

通过：

```text
xxx-spring-boot-autoconfigure
```

提前完成 Bean 和框架配置。

解决：

> **依赖进入项目以后，相关对象怎么自动进入 IOC 容器。**

因此可以理解为：

```text
自定义Starter
=
依赖管理
+
自动配置
```

---

# 15. Starter 与 autoconfigure 为什么分离

如果把所有东西全部放进一个模块，也可以实现功能。

但是当前材料采用：

```text
starter
+
autoconfigure
```

两个模块进行职责拆分。

这样：

```text
Starter
负责作为用户引入的依赖入口

Autoconfigure
负责真正的自动配置实现
```

使用者只需要关注：

```text
Starter
```

而内部自动配置细节则由：

```text
autoconfigure
```

模块维护。

这种结构让“使用入口”和“实现细节”分离得更加清楚。

---

# 16. 自定义 Starter 的价值

自定义 Starter 最适合解决：

> **多个项目重复接入同一种公共技术的问题。**

例如公司内部有多个项目都要使用 OSS。

如果没有 Starter：

```text
项目A配置一次
项目B配置一次
项目C配置一次
```

大量代码和配置重复。

封装成 Starter 后：

```text
公共组件统一维护
↓
各项目引入Starter
↓
提供少量业务配置
↓
直接使用
```

这样能够：

- 减少重复代码；
    
- 统一技术接入方式；
    
- 集中维护公共配置；
    
- 降低其他项目使用公共组件的成本。
    

---

# 17. 当前阶段需要掌握的核心内容

1. 当某个第三方技术没有现成 Starter，而且多个项目都会重复使用时，可以考虑自定义 Starter。
    
2. 自定义 Starter 的目标是封装依赖和自动配置，让其他项目可以低成本使用公共功能。
    
3. 当前材料将 Starter 拆成 `starter` 和 `autoconfigure` 两个模块。
    
4. `xxx-spring-boot-starter` 主要负责依赖管理。
    
5. `xxx-spring-boot-autoconfigure` 主要负责自动配置。
    
6. Starter 需要依赖 autoconfigure，这样使用者只需要引入 Starter。
    
7. 配置属性可以通过 `@ConfigurationProperties` 封装。
    
8. 自动配置类可以通过 `@EnableConfigurationProperties` 启用配置属性类。
    
9. 自动配置类通过 `@Bean` 创建需要自动注册的对象。
    
10. 自动配置类需要登记到 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`。
    
11. Spring Boot 启动时读取 `.imports` 文件，从而发现自定义自动配置类。
    
12. 自定义 Starter 本质上是“依赖管理 + 自动配置”的组合。
    
13. 使用项目最终只需要引入 Starter、提供必要配置，然后直接注入自动配置好的 Bean。
    

> [!tip] 一句话总结
> 
> `自定义Spring Boot Starter就是把公共技术所需的依赖和自动配置封装起来，由starter模块负责依赖管理、autoconfigure模块负责配置属性和Bean自动注册，并通过AutoConfiguration.imports让Spring Boot发现自动配置类，从而让业务项目只需引入Starter并提供少量配置即可直接使用。`