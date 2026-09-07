# Spring 知识导航

[返回后端知识图谱总览](<../后端知识图谱总览.md>)

Spring 部分可以按“容器基础 → AOP → Spring Boot → Spring MVC”阅读。Spring Boot 的自动配置建立在 Spring 容器能力之上，Spring MVC 则把这些能力放入 Web 请求处理流程。

## Spring 核心：IOC、DI 与 Bean

| 顺序 | 笔记 | 作用 | 关联 |
| ---: | --- | --- | --- |
| 1 | [IOC 与 DI](<Spring核心/01-IOC与DI.md>) | 解释控制反转、IOC 容器、Bean 和依赖注入。 | 是 Bean 声明、扫描和注入的总入口。 |
| 2 | [Bean 的声明](<Spring核心/02-Bean的声明.md>) | 解释组件注解、`@Bean`、`@Configuration` 和 Bean 名称。 | 连接组件扫描、自动配置和第三方对象注册。 |
| 3 | [组件扫描](<Spring核心/03-组件扫描.md>) | 解释 `@ComponentScan`、默认扫描范围和启动类位置。 | 连接 Spring Boot 的 `@SpringBootApplication`。 |
| 4 | [Autowired 依赖注入](<Spring核心/04-Autowired依赖注入.md>) | 比较属性、构造器和 Setter 注入。 | 建立在 Bean 已进入 IOC 容器的前提上。 |
| 5 | [多 Bean 依赖注入冲突](<Spring核心/05-多Bean依赖注入冲突.md>) | 解释 `@Primary`、`@Qualifier` 和 `@Resource`。 | 是自动注入规则在多个候选 Bean 场景下的延伸。 |
| 6 | [Bean 作用域与创建时机](<Spring核心/06-Bean作用域与创建时机.md>) | 解释 singleton、prototype、`@Scope`、`@Lazy` 和创建时机。 | 补充 Bean 的生命周期使用特征。 |

## Spring 核心：AOP

| 顺序 | 笔记 | 作用 | 关联 |
| ---: | --- | --- | --- |
| 1 | [AOP 概述](<Spring核心/AOP/01-AOP概述.md>) | 说明横切关注点、减少重复代码和常见应用场景。 | 连接操作日志、权限控制和事务等案例。 |
| 2 | [AOP 核心概念](<Spring核心/AOP/02-AOP核心概念.md>) | 解释 JoinPoint、Advice、Pointcut、Aspect、Target。 | 是所有通知和切入点表达式笔记的概念基础。 |
| 3 | [Spring AOP 快速入门](<Spring核心/AOP/03-SpringAOP快速入门.md>) | 用环绕通知完成方法耗时统计。 | 连接通知类型和 JoinPoint。 |
| 4 | [AOP 通知类型](<Spring核心/AOP/04-AOP通知类型.md>) | 比较前置、后置、返回后、异常后和环绕通知。 | 连接通知执行顺序和具体案例。 |
| 5 | [Pointcut 公共切入点](<Spring核心/AOP/05-Pointcut公共切入点.md>) | 解释如何抽取和复用公共切入点。 | 连接 execution、`@annotation` 和切面复用。 |
| 6 | [execution 切入点表达式](<Spring核心/AOP/06-execution切入点表达式.md>) | 说明按方法签名匹配目标方法的语法。 | 与 `@annotation` 形成两种匹配思路对比。 |
| 7 | [AOP 通知执行顺序](<Spring核心/AOP/07-AOP通知执行顺序.md>) | 解释多个切面叠加时的默认顺序和 `@Order`。 | 建立在通知类型之上。 |
| 8 | [annotation 切入点表达式](<Spring核心/AOP/08-annotation切入点表达式.md>) | 说明通过自定义注解精确匹配目标方法。 | 直接连接公共字段填充和操作日志案例。 |
| 9 | [JoinPoint 连接点信息](<Spring核心/AOP/09-JoinPoint连接点信息.md>) | 说明如何获取目标对象、方法、参数和返回过程中的信息。 | 是操作日志案例获取上下文的基础。 |
| 10 | [AOP 实例：公共字段填充](<Spring核心/AOP/AOP实例-公共字段填充.md>) | 用 AOP、自定义注解和反射完成 INSERT/UPDATE 字段填充。 | 连接 `@Before`、`@annotation`、JoinPoint 和 MyBatis Mapper。 |
| 11 | [AOP 实例：数据库记录操作日志](<Spring核心/AOP/AOP实例-数据库记录操作日志.md>) | 用 AOP 记录方法、参数、返回值、耗时和当前用户。 | 汇合 AOP、日志、登录校验、ThreadLocal 和数据库持久化。 |

## Spring Boot：基础、配置与自动配置

| 分组 | 笔记 | 作用 | 关联 |
| --- | --- | --- | --- |
| 基础 | [Spring Boot 概述](<SpringBoot/基础知识/01-SpringBoot概述.md>) | 说明 Spring 生态、Spring Framework 与 Spring Boot 的关系。 | 是快速入门和自动配置的总入口。 |
| 基础 | [Spring Boot 快速入门](<SpringBoot/基础知识/02-SpringBoot快速入门.md>) | 完成 Web 工程、Controller、启动和访问。 | 连接 MVC、Maven、IOC 和 REST 响应。 |
| 基础 | [Starter 与内嵌 Tomcat](<SpringBoot/基础知识/03-Starter与内嵌Tomcat.md>) | 解释 Starter、依赖传递、内嵌 Tomcat 和默认端口。 | 连接 Maven 依赖管理与 Web 运行环境。 |
| 配置 | [Spring Boot 配置文件](<SpringBoot/配置/01-SpringBoot配置文件.md>) | 解释 properties、YAML、层级结构和数据库/MyBatis 配置。 | 连接 MyBatis 快速入门和数据库连接池。 |
| 配置 | [Spring Boot 配置优先级](<SpringBoot/配置/02-SpringBoot配置优先级.md>) | 解释配置文件、系统属性和命令行参数的优先级。 | 是配置文件在部署环境中的延伸。 |
| 自动配置 | [Spring Boot 自动配置概述](<SpringBoot/自动配置/01-SpringBoot自动配置概述.md>) | 说明自动配置为什么存在，以及它和 Starter 的区别。 | 引出核心注解、导入和条件装配。 |
| 自动配置 | [SpringBootApplication 核心注解](<SpringBoot/自动配置/02-SpringBootApplication核心注解.md>) | 拆解 `@SpringBootConfiguration`、`@ComponentScan` 和 `@EnableAutoConfiguration`。 | 连接组件扫描和自动配置原理。 |
| 自动配置 | [Import 与 ImportSelector](<SpringBoot/自动配置/03-Import与ImportSelector.md>) | 解释 `@Import`、ImportSelector 和 `@EnableXxx`。 | 是理解自动配置导入机制的中间层。 |
| 自动配置 | [Spring Boot 自动配置原理](<SpringBoot/自动配置/04-SpringBoot自动配置原理.md>) | 跟踪 AutoConfigurationImportSelector、imports 文件和 Bean 注册。 | 连接条件装配和自定义 Starter。 |
| 自动配置 | [Conditional 条件装配](<SpringBoot/自动配置/05-Conditional条件装配.md>) | 解释按类、Bean、属性决定是否装配。 | 是自动配置实现选择性的关键。 |
| Starter | [自定义 Spring Boot Starter](<SpringBoot/Starter/自定义SpringBootStarter.md>) | 综合配置属性、自动配置类和 `AutoConfiguration.imports`。 | 是 Starter、自动配置和 Maven 依赖管理的综合案例。 |
| Web | [全局异常处理](<SpringBoot/web/全局异常处理.md>) | 解释异常传播、`@RestControllerAdvice` 和统一响应。 | 连接 Controller、REST 响应和日志。 |

## Spring MVC：请求与响应

| 顺序 | 笔记 | 作用 | 关联 |
| ---: | --- | --- | --- |
| 1 | [Interceptor 拦截器](<SpringMVC/Interceptor/Interceptor拦截器.md>) | 解释 HandlerInterceptor、三个生命周期方法和注册方式。 | 连接登录校验和 Filter 对比。 |
| 2 | [Interceptor 执行流程](<SpringMVC/Interceptor/Interceptor执行流程.md>) | 解释路径匹配、排除路径、DispatcherServlet 和完整执行顺序。 | 补充 Interceptor 的实际请求链位置。 |
| 3 | [ResponseBody 与 RestController](<SpringMVC/ResponseBody与RestController.md>) | 解释对象转 JSON、`@ResponseBody` 和 `@RestController`。 | 连接 Spring Boot 快速入门和全局异常统一响应。 |

## 总体关系

```text
IOC 与 DI
   ↓
Bean 声明 → 组件扫描 → Autowired → 多 Bean 冲突 → Bean 作用域
   ↓
Spring AOP 基础 → 通知 / 切入点 / JoinPoint → 两个 AOP 案例
   ↓
Spring Boot 基础 → 配置 → @SpringBootApplication
                         ↓
                 Import / 自动配置原理 / 条件装配
                         ↓
                 自定义 Starter
   ↓
Spring MVC：Interceptor / Controller 响应 / 全局异常处理
```

## 关键跨域连接

- [Bean 的声明](<Spring核心/02-Bean的声明.md>)和[组件扫描](<Spring核心/03-组件扫描.md>)共同回答“Bean 如何进入容器”；[Autowired 依赖注入](<Spring核心/04-Autowired依赖注入.md>)回答“Bean 如何获得依赖”；[Bean 作用域与创建时机](<Spring核心/06-Bean作用域与创建时机.md>)回答“Bean 以什么范围和时机存在”。
- [组件扫描](<Spring核心/03-组件扫描.md>)与[SpringBootApplication 核心注解](<SpringBoot/自动配置/02-SpringBootApplication核心注解.md>)都涉及 `@ComponentScan`，但前者聚焦扫描，后者还包含自动配置入口。
- [Spring Boot 配置文件](<SpringBoot/配置/01-SpringBoot配置文件.md>)与[MyBatis 快速入门](<../数据库/MyBatis/02-MyBatis快速入门.md>)共同涉及数据库连接和 MyBatis 配置。
- [Spring Boot 自动配置概述](<SpringBoot/自动配置/01-SpringBoot自动配置概述.md>)、[Import 与 ImportSelector](<SpringBoot/自动配置/03-Import与ImportSelector.md>)、[自动配置原理](<SpringBoot/自动配置/04-SpringBoot自动配置原理.md>)和[条件装配](<SpringBoot/自动配置/05-Conditional条件装配.md>)是一条连续的机制链。
- [Spring Boot 快速入门](<SpringBoot/基础知识/02-SpringBoot快速入门.md>)和[ResponseBody 与 RestController](<SpringMVC/ResponseBody与RestController.md>)都涉及 `@RestController`；前者关注快速建立 Web 接口，后者关注响应语义。
- [AOP 实例：公共字段填充](<Spring核心/AOP/AOP实例-公共字段填充.md>)连接[MyBatis CRUD](<../数据库/MyBatis/04-MyBatis CRUD.md>)的持久层操作；[AOP 实例：数据库记录操作日志](<Spring核心/AOP/AOP实例-数据库记录操作日志.md>)连接[登录校验实现](<../工程基础/登录认证/06-登录校验实现.md>)的当前用户信息和[SLF4J](<../工程基础/日志技术/02-SLF4J.md>)的日志调用。

## 复习抓手

1. IOC、Bean、扫描、注入和作用域解决的是容器管理问题。
2. AOP 先学概念，再学通知和切入点，最后看两个完整案例。
3. Spring Boot 先理解简化开发，再追踪自动配置如何导入和筛选 Bean。
4. Spring MVC 结合请求链复习 Interceptor，结合响应结果复习 `@RestController` 和全局异常处理。
