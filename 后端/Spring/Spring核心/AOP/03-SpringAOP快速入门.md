# 1. Spring AOP 入门案例

学习 Spring AOP 时，可以先通过一个简单案例理解它的基本开发方式。

当前案例的需求是：

> **统计部门管理中各个业务层方法的执行耗时。**

如果使用普通方式实现，需要在每一个业务方法执行前记录开始时间，在方法执行结束后记录结束时间，再计算方法执行耗时。

这种方式会导致相同的耗时统计代码重复出现在多个业务方法中。

Spring AOP 可以把这部分重复逻辑统一抽取出来，并在不修改原始业务方法的情况下完成增强。

# 2. 引入 Spring AOP 依赖

在 Spring Boot 项目中使用 AOP，需要先引入：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

这个依赖用于为 Spring Boot 项目提供 AOP 相关支持。

# 3. 定义切面类

AOP 程序通常需要单独定义一个切面类。

例如：

```java
@Component
@Aspect
@Slf4j
public class RecordTimeAspect {

}
```

这里使用了两个关键注解。

## 3.1 @Component

```java
@Component
```

用于将当前类交给 Spring IOC 容器管理。

只有交给 Spring 管理后，Spring 才能识别并使用这个切面类。

## 3.2 @Aspect

```java
@Aspect
```

用于标识：

> 当前类是一个切面类。

在前面的核心概念中已经知道，切面用于描述通知与切入点之间的关系。

因此，使用 `@Aspect` 后，Spring 会将这个类作为 AOP 切面进行处理。

# 4. 使用 @Around 定义环绕通知

当前案例使用的是：

```java
@Around
```

也就是环绕通知。

示例：

```java
@Around("execution(* com.huaishi.service.impl.DeptServiceImpl.*(..))")
public Object recordTime(ProceedingJoinPoint pjp) throws Throwable {

}
```

这里：

```java
@Around(...)
```

表示当前方法是一个环绕通知。

而括号中的：

```java
execution(* com.huaishi.service.impl.DeptServiceImpl.*(..))
```

用于描述这个通知应该作用到哪些方法上。

具体的 `execution` 表达式写法会在后续 `execution切入点表达式.md` 中单独整理。

当前阶段只需要理解：

> `@Around` 用于定义一个可以在目标方法执行前后都执行逻辑的通知。

# 5. ProceedingJoinPoint

在环绕通知中，方法参数通常会使用：

```java
ProceedingJoinPoint
```

例如：

```java
public Object recordTime(ProceedingJoinPoint pjp) throws Throwable
```

它用于表示当前正在被 AOP 控制的方法执行。

在当前案例中，最关键的操作是：

```java
pjp.proceed();
```

它的作用是：

> **继续执行原始目标方法。**

如果没有调用：

```java
pjp.proceed();
```

那么原始业务方法就不会继续执行。

# 6. 使用 AOP 统计方法执行耗时

完整代码如下：

```java
@Component
@Aspect
@Slf4j
public class RecordTimeAspect {

    @Around("execution(* com.huaishi.service.impl.DeptServiceImpl.*(..))")
    public Object recordTime(ProceedingJoinPoint pjp) throws Throwable {

        // 记录方法执行开始时间
        long begin = System.currentTimeMillis();

        // 执行原始方法
        Object result = pjp.proceed();

        // 记录方法执行结束时间
        long end = System.currentTimeMillis();

        // 计算方法执行耗时
        log.info("方法执行耗时: {}毫秒", end - begin);

        return result;
    }
}
```

这个通知方法中主要完成了几件事情。

## 6.1 记录开始时间

```java
long begin = System.currentTimeMillis();
```

在原始方法执行之前记录当前时间。

## 6.2 执行原始方法

```java
Object result = pjp.proceed();
```

调用原始业务方法，并接收其返回值。

## 6.3 记录结束时间

```java
long end = System.currentTimeMillis();
```

在原始方法执行完成后记录结束时间。

## 6.4 计算执行耗时

```java
log.info("方法执行耗时: {}毫秒", end - begin);
```

使用结束时间减去开始时间，即可得到方法执行耗时。

## 6.5 返回原始方法结果

```java
return result;
```

环绕通知执行完额外逻辑后，还需要将原始方法的返回结果继续返回。

# 7. 为什么 AOP 可以减少重复代码

如果不使用 AOP，每一个业务方法中都需要重复编写：

```java
long begin = System.currentTimeMillis();

// 原始业务逻辑

long end = System.currentTimeMillis();

log.info("方法执行耗时: {}毫秒", end - begin);
```

而使用 AOP 后，只需要把这部分代码集中写在一个通知方法中。

符合切入点条件的业务方法执行时，都会统一应用这段耗时统计逻辑。

这样就不需要修改每一个业务方法。

# 8. Spring AOP 的基本开发步骤

根据当前案例，可以把 Spring AOP 的基本开发方式概括为几个步骤。

首先，在项目中引入：

```xml
spring-boot-starter-aop
```

然后定义一个由 Spring 管理的切面类：

```java
@Component
@Aspect
```

接着定义通知方法，并使用：

```java
@Around(...)
```

指定通知类型和目标方法范围。

在环绕通知中，通过：

```java
ProceedingJoinPoint.proceed()
```

执行原始方法。

最后，在原始方法执行前后加入需要的公共逻辑，例如统计执行耗时。

# 9. AOP 入门案例体现出的特点

这个案例体现了 AOP 的几个典型特点。

## 9.1 不修改原始业务方法

耗时统计逻辑全部定义在切面类中。

原来的 Service 方法不需要修改。

## 9.2 公共逻辑集中管理

所有方法耗时统计逻辑都统一放在：

```java
RecordTimeAspect
```

中。

如果以后修改统计方式，只需要修改这一处代码。

## 9.3 一个通知可以增强多个方法

通过切入点表达式：

```java
execution(* com.huaishi.service.impl.DeptServiceImpl.*(..))
```

可以让通知同时作用于多个符合条件的方法。

# 10. AOP 的常见应用场景

除了统计方法执行耗时，材料中还列举了几个典型的 AOP 应用场景：

- 记录系统操作日志
    
- 权限控制
    
- 事务管理
    

例如 Spring 的事务管理中，使用：

```java
@Transactional
```

后，事务相关逻辑可以在原始业务方法执行前后自动完成。

这些场景都具有一个共同特点：

> 多个方法需要执行相同或类似的公共逻辑。

# 11. 当前阶段需要掌握的核心内容

1. Spring Boot 项目使用 AOP 时，需要引入 `spring-boot-starter-aop`。
    
2. `@Aspect` 用于标识当前类是一个切面类。
    
3. 切面类还需要通过 `@Component` 交给 Spring IOC 容器管理。
    
4. `@Around` 用于定义环绕通知。
    
5. 环绕通知可以在原始方法执行前后执行额外逻辑。
    
6. `ProceedingJoinPoint` 用于表示当前连接点。
    
7. `ProceedingJoinPoint.proceed()` 用于执行原始目标方法。
    
8. 环绕通知可以接收原始方法的返回值，并在增强逻辑执行完成后继续返回。
    
9. AOP 可以把多个方法中的公共逻辑统一抽取到切面中。
    
10. 方法耗时统计、操作日志、权限控制和事务管理都是 AOP 的典型应用场景。
    

> [!tip] 一句话总结
> 
> `Spring AOP 的基本使用方式是引入 AOP 依赖，定义由 Spring 管理的 @Aspect 切面类，再通过 @Around 和 ProceedingJoinPoint.proceed() 在不修改原始业务代码的情况下对目标方法进行功能增强。`