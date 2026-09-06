# 1. 什么是连接点

在前面的 AOP 核心概念中，已经知道：

> **连接点表示可以被 AOP 控制的方法。**

而在 Spring AOP 中，连接点更具体地指：

> **方法的执行过程。**

也就是说，当某个目标方法正在运行时，Spring 可以把这次方法执行抽象成一个连接点对象。

材料中明确指出，在 Spring AOP 中，连接点特指方法的执行。

# 2. JoinPoint 的作用

Spring 使用：

```java
JoinPoint
```

来抽象连接点。

通过 `JoinPoint`，可以获取目标方法执行时的一些相关信息。

材料中明确列举了：

- 目标类名
    
- 方法名
    
- 方法参数
    

这些信息在操作日志、方法监控等 AOP 场景中非常常用。

# 3. JoinPoint 可以获取哪些信息

常见的连接点信息包括：

## 3.1 获取目标对象

可以通过连接点获取当前正在被增强的目标对象。

例如：

```java
joinPoint.getTarget()
```

可以拿到目标对象本身。

如果需要获取目标类名，可以继续调用：

```java
joinPoint.getTarget().getClass().getName()
```

这在记录操作日志时很常见。

---

## 3.2 获取方法信息

可以通过：

```java
joinPoint.getSignature()
```

获取当前方法的签名信息。

例如：

```java
joinPoint.getSignature().getName()
```

可以获得当前执行的方法名。

---

## 3.3 获取方法参数

可以通过：

```java
joinPoint.getArgs()
```

获取当前方法执行时传入的参数。

它返回的是一个参数数组。

例如可以用于记录：

```text
用户提交的数据
方法调用参数
接口请求对应的业务参数
```

这些信息。

# 4. ProceedingJoinPoint

对于环绕通知：

```java
@Around
```

Spring 使用的是：

```java
ProceedingJoinPoint
```

材料中明确指出：

> `@Around` 通知获取连接点信息时，需要使用 `ProceedingJoinPoint`。

# 5. ProceedingJoinPoint 的特殊之处

`ProceedingJoinPoint` 除了可以获取连接点信息之外，还可以控制原始方法是否继续执行。

最关键的方法就是：

```java
proceed()
```

例如：

```java
Object result = proceedingJoinPoint.proceed();
```

它表示：

> 执行目标对象中的原始方法。

这也是 `ProceedingJoinPoint` 和普通 `JoinPoint` 最重要的区别。

# 6. JoinPoint 与 ProceedingJoinPoint 的关系

材料中说明：

```java
ProceedingJoinPoint
```

是：

```java
JoinPoint
```

的子类型。

因此，`ProceedingJoinPoint` 也具备获取目标类、方法名、参数等连接点信息的能力。

但它额外增加了：

```java
proceed()
```

这种控制原始方法执行的能力。

# 7. 不同通知使用哪一种连接点对象

材料中给出了明确区分。

## 7.1 @Around

对于：

```java
@Around
```

使用：

```java
ProceedingJoinPoint
```

例如：

```java
@Around("...")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    Object result = joinPoint.proceed();
    return result;
}
```

因为环绕通知需要自己控制原始方法执行，所以必须使用 `ProceedingJoinPoint`。

## 7.2 其他四种通知

对于：

```java
@Before
@After
@AfterReturning
@AfterThrowing
```

材料中说明使用：

```java
JoinPoint
```

例如：

```java
@Before("...")
public void before(JoinPoint joinPoint) {
}
```

这些通知不需要主动调用原始方法，因此使用普通 `JoinPoint` 即可。

# 8. JoinPoint 在操作日志中的用途

在操作日志场景中，经常需要记录：

- 执行的是哪个类
    
- 执行的是哪个方法
    
- 方法传入了哪些参数
    

这些信息都可以通过连接点对象获取。

例如：

```java
joinPoint.getTarget().getClass().getName()
```

获取目标类名。

```java
joinPoint.getSignature().getName()
```

获取方法名。

```java
joinPoint.getArgs()
```

获取方法参数。

材料后续的操作日志案例正是通过这些连接点信息来构建日志数据。

# 9. JoinPoint 与切入点的区别

这两个概念容易混淆。

## 9.1 JoinPoint

表示：

> 实际的方法执行连接点，以及这次执行过程中携带的信息。

它关注的是：

```text
当前到底是哪个方法在执行
执行时有哪些参数
目标对象是谁
```

## 9.2 Pointcut

表示：

> 哪些连接点需要被匹配和增强。

它关注的是：

```text
哪些方法应该应用通知
```

因此可以简单理解为：

```text
Pointcut 决定“选哪些方法”
JoinPoint 表示“当前这个方法执行”
```

# 10. 在环绕通知中的典型使用方式

例如：

```java
@Around("@annotation(logOperation)")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {

    String className =
            joinPoint.getTarget().getClass().getName();

    String methodName =
            joinPoint.getSignature().getName();

    Object[] args =
            joinPoint.getArgs();

    Object result =
            joinPoint.proceed();

    return result;
}
```

这段代码体现了 `ProceedingJoinPoint` 的两个核心能力：

1. 获取当前连接点的运行时信息。
    
2. 通过 `proceed()` 执行原始方法。
    

# 11. 当前阶段需要掌握的核心内容

1. 在 Spring AOP 中，连接点特指方法的执行。
    
2. Spring 使用 `JoinPoint` 抽象连接点。
    
3. `JoinPoint` 可以获取目标对象、方法信息和方法参数等运行时信息。
    
4. `getTarget()` 可以获取目标对象。
    
5. `getSignature()` 可以获取方法签名信息。
    
6. `getArgs()` 可以获取方法参数。
    
7. `@Around` 通知需要使用 `ProceedingJoinPoint`。
    
8. 其他四种通知使用 `JoinPoint`。
    
9. `ProceedingJoinPoint` 是 `JoinPoint` 的子类型。
    
10. `ProceedingJoinPoint` 可以通过 `proceed()` 控制原始方法执行。
    
11. 操作日志中记录类名、方法名、参数等信息时，经常需要使用连接点对象。
    

> [!tip] 一句话总结
> 
> `JoinPoint 用于表示 Spring AOP 中一次具体的方法执行并获取目标类、方法名和参数等信息，而 @Around 通知使用 ProceedingJoinPoint，它除了具备这些能力外，还可以通过 proceed() 执行原始方法。`