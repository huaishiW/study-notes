# 1. 为什么需要 @Import

Spring Boot 默认会通过组件扫描发现启动类所在包及其子包中的组件。

例如：

```text
com.huaishi
├── Application.java
├── controller
└── service
```

如果启动类位于：

```text
com.huaishi
```

那么 Spring Boot 默认能够扫描：

```text
com.huaishi
```

及其子包。

但是如果引入的第三方依赖中的类位于：

```text
com.example
```

由于：

```text
com.example
```

并不是：

```text
com.huaishi
```

的子包，因此默认组件扫描无法发现其中的组件。

即使第三方类已经使用：

```java
@Component
```

声明，仍然可能无法进入当前项目的 IOC 容器。

因此出现了一个新的问题：

> **不依赖默认组件扫描，如何把指定的类交给 Spring IOC 容器管理？**

Spring 提供的一个重要方式就是：

```java
@Import
```

---

# 2. 为什么不直接扩大 @ComponentScan

对于扫描不到的第三方包，可以显式配置：

```java
@SpringBootApplication
@ComponentScan({
        "com.huaishi",
        "com.example"
})
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

这样确实可以让 Spring 同时扫描：

```text
com.huaishi
com.example
```

从而发现第三方包中的组件。

但是这种方式存在明显问题。

假设项目不断引入第三方依赖：

```text
com.example.a
com.example.b
com.example.c
...
```

那么开发者就可能需要不断修改：

```java
@ComponentScan
```

把这些包全部加入扫描范围。

这样会造成：

- 配置越来越繁琐。
    
- 开发者必须知道第三方组件位于哪些包。
    
- 扫描范围不断扩大。
    

因此，当前材料明确指出：

> Spring Boot 并没有把“不断扩大组件扫描范围”作为自动配置的核心解决方案。

更合适的方式是：

```java
@Import
```

直接导入需要交给 Spring 管理的类。

---

# 3. 什么是 @Import

`@Import` 的作用可以理解为：

> **将指定的类导入 Spring IOC 容器。**

它不要求目标类一定处于当前项目默认的组件扫描范围内。

例如第三方类：

```java
public class TokenParser {

    public void parse() {
        System.out.println("TokenParser ... parse ...");
    }
}
```

即使它位于：

```text
com.example
```

也可以在当前项目中主动导入：

```java
@Import(TokenParser.class)
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

这样：

```java
TokenParser
```

就可以被 Spring 加载并交给 IOC 容器管理。

因此：

```text
@ComponentScan
→ 到指定范围主动寻找组件

@Import
→ 明确告诉 Spring 要导入哪个类
```

二者解决问题的方式不同。

---

# 4. @Import 导入普通类

最直接的使用方式是导入一个普通类。

例如：

```java
@Import(TokenParser.class)
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

这里：

```java
TokenParser.class
```

被显式导入。

即使 `TokenParser` 不在默认组件扫描范围中，也可以通过 `@Import` 被 Spring 加载。

这种方式适合：

> **明确知道需要导入某一个具体类的场景。**

但是当第三方依赖中包含很多 Bean 时，如果一个一个导入：

```java
@Import(A.class)
@Import(B.class)
@Import(C.class)
```

仍然会比较繁琐。

---

# 5. @Import 导入配置类

除了普通类以外，`@Import` 还可以导入一个：

```java
@Configuration
```

配置类。

例如第三方依赖中存在：

```java
@Configuration
public class HeaderConfig {

    @Bean
    public HeaderParser headerParser() {
        return new HeaderParser();
    }

    @Bean
    public HeaderGenerator headerGenerator() {
        return new HeaderGenerator();
    }
}
```

可以直接：

```java
@Import(HeaderConfig.class)
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

这样就不需要分别导入：

```text
HeaderParser
HeaderGenerator
```

而是直接导入：

```text
HeaderConfig
```

由配置类负责声明其中的 Bean。

材料正是通过 `HeaderConfig` 演示了这一方式。

因此，导入配置类比逐个导入普通类更适合：

> **一组相关 Bean 需要一起注册的场景。**

---

# 6. 为什么还需要 ImportSelector

虽然导入配置类已经比逐个导入 Bean 更方便，但仍然存在一个问题：

> **使用第三方依赖的开发者，仍然需要知道应该导入哪个配置类。**

例如：

```java
@Import(HeaderConfig.class)
```

这意味着使用者必须知道：

```text
HeaderConfig
```

这个类的存在。

但是第三方依赖内部到底包含：

- 哪些配置类；
    
- 哪些 Bean；
    
- 哪些类需要被导入；
    

最清楚的其实不是使用者，而是：

> **第三方依赖自身。**

因此，更合理的设计是：

> 让第三方依赖自己决定需要导入哪些类。

这就可以使用：

```java
ImportSelector
```

---

# 7. ImportSelector

`ImportSelector` 是一个可以决定：

> **需要向 Spring 导入哪些类**

的接口。

可以编写一个实现类：

```java
public class MyImportSelector implements ImportSelector {

    @Override
    public String[] selectImports(
            AnnotationMetadata importingClassMetadata) {

        return new String[]{
                "com.example.HeaderConfig"
        };
    }
}
```

这里最重要的是：

```java
selectImports()
```

方法。

它返回：

```java
String[]
```

数组。

数组中保存需要导入类的：

> **全限定类名。**

例如：

```java
return new String[]{
        "com.example.HeaderConfig"
};
```

表示希望 Spring 导入：

```text
com.example.HeaderConfig
```

材料明确演示了 `ImportSelector` 实现类通过 `selectImports()` 返回配置类全限定名的方式。

---

# 8. 通过 @Import 导入 ImportSelector

有了：

```java
MyImportSelector
```

之后，就不再直接：

```java
@Import(HeaderConfig.class)
```

而是：

```java
@Import(MyImportSelector.class)
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

此时可以理解为：

```text
@Import
↓
MyImportSelector
↓
selectImports()
↓
返回需要导入类的全限定名
↓
Spring加载这些类
```

这样需要导入哪些配置类，就由：

```java
MyImportSelector
```

统一决定。

当前项目不需要直接知道所有配置类。

---

# 9. ImportSelector 的价值

如果没有 `ImportSelector`，可能需要：

```java
@Import(ConfigA.class)
@Import(ConfigB.class)
@Import(ConfigC.class)
```

使用者必须知道：

```text
ConfigA
ConfigB
ConfigC
```

而使用 `ImportSelector` 后，可以变成：

```java
@Import(MyImportSelector.class)
```

由：

```text
MyImportSelector
```

内部统一决定：

```text
需要导入 ConfigA
需要导入 ConfigB
需要导入 ConfigC
```

所以它解决的核心问题是：

> **把“应该导入哪些类”的决定权集中起来。**

这样第三方依赖就可以自己维护自己的导入规则，而不是要求每一个使用者都了解它内部的配置结构。

---

# 10. @EnableXxx 的出现

虽然：

```java
@Import(MyImportSelector.class)
```

已经能够把导入规则交给第三方依赖管理，但对于使用者来说，仍然需要直接接触：

```java
MyImportSelector
```

这个内部实现类。

为了进一步降低使用成本，可以再封装一层自定义注解。

例如：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import(MyImportSelector.class)
public @interface EnableHeaderConfig {
}
```

这里：

```java
@EnableHeaderConfig
```

内部已经封装：

```java
@Import(MyImportSelector.class)
```

材料也明确指出，常见做法是由第三方依赖提供一个 `@EnableXxx` 注解，并在其中封装 `@Import`。

---

# 11. 使用 @EnableXxx

此时使用第三方功能时，只需要：

```java
@EnableHeaderConfig
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

使用者不再需要直接知道：

```text
HeaderConfig
MyImportSelector
```

这些内部实现。

可以理解为：

```text
@EnableHeaderConfig
        ↓
    @Import
        ↓
MyImportSelector
        ↓
 selectImports()
        ↓
  HeaderConfig
        ↓
   @Bean 方法
        ↓
Bean进入IOC容器
```

对使用者而言，只需要知道：

```java
@EnableHeaderConfig
```

即可开启对应功能。

---

# 12. 四种导入方式的演进

当前材料实际上展示了一条逐步优化的过程。

|方式|使用方式|主要特点|
|---|---|---|
|导入普通类|`@Import(TokenParser.class)`|直接导入单个类|
|导入配置类|`@Import(HeaderConfig.class)`|一次导入一组 Bean 配置|
|导入 `ImportSelector`|`@Import(MyImportSelector.class)`|由选择器决定导入哪些类|
|`@EnableXxx`|`@EnableHeaderConfig`|对 `@Import` 和选择器进一步封装|

可以理解成：

```text
直接指定 Bean
↓
指定配置类
↓
让 ImportSelector 决定配置类
↓
再用 @EnableXxx 隐藏 ImportSelector
```

每向后一步，都减少了使用者需要了解的第三方内部细节。

其中材料认为：

```java
@EnableXxx
```

这一形式更加方便和优雅。

---

# 13. @Import 与 Spring Boot 自动配置的联系

前面已经知道：

```java
@EnableAutoConfiguration
```

是 Spring Boot 自动配置的核心入口。

它内部也使用：

```java
@Import
```

并导入了一个：

```java
ImportSelector
```

的实现类。

因此前面学习的：

```text
@Import
ImportSelector
@EnableXxx
```

并不是独立的小知识。

它们实际上是在为理解 Spring Boot 自动配置做准备。

可以先建立这样的认识：

```text
@EnableAutoConfiguration
↓
@Import
↓
ImportSelector实现类
↓
决定需要导入哪些自动配置类
```

Spring Boot 中具体使用的是：

```java
AutoConfigurationImportSelector
```

但是：

> **`AutoConfigurationImportSelector` 到底如何查找自动配置类，不在这一篇展开。**

它将在后面的自动配置原理中继续分析。

---

# 14. 当前阶段需要掌握的核心内容

1. Spring Boot 默认组件扫描无法发现所有第三方依赖中的组件。
    
2. 可以扩大 `@ComponentScan` 范围，但这种方式比较繁琐，不是 Spring Boot 自动配置的核心方案。
    
3. `@Import` 可以将指定类导入 Spring IOC 容器。
    
4. `@Import` 可以直接导入普通类。
    
5. `@Import` 可以导入 `@Configuration` 配置类，由配置类进一步声明多个 Bean。
    
6. 如果直接导入配置类，使用者仍然需要知道第三方依赖内部有哪些配置类。
    
7. `ImportSelector` 可以通过 `selectImports()` 决定需要导入哪些类。
    
8. `selectImports()` 返回类的全限定名数组。
    
9. 使用 `ImportSelector` 后，可以由第三方依赖自身维护导入规则。
    
10. 可以通过 `@EnableXxx` 注解进一步封装 `@Import` 和 `ImportSelector`。
    
11. `@EnableXxx` 可以让使用者只关注“开启某项功能”，而不需要了解内部具体导入了哪些类。
    
12. Spring Boot 的 `@EnableAutoConfiguration` 也沿用了 `@Import + ImportSelector` 这一类设计思路。
    
13. 下一步需要继续研究的是 `AutoConfigurationImportSelector` 如何找到 Spring Boot 的自动配置类。
    

> [!tip] 一句话总结
> 
> `@Import可以绕过默认组件扫描直接将指定类导入IOC容器，ImportSelector进一步把“导入哪些类”的决定集中到选择器中，而@EnableXxx又对这一过程进行封装，这套设计也为理解Spring Boot的自动配置机制奠定了基础。`