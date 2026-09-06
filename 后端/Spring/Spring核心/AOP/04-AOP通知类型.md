# 1. 什么是通知类型

在 Spring AOP 中，通知表示：

> 对目标方法进行增强时，需要额外执行的公共逻辑。

不同通知的主要区别在于：

> **通知在目标方法执行的什么时机被触发。**

Spring AOP 中常见的通知类型有五种：

```text
@Before
@After
@AfterReturning
@AfterThrowing
@Around
```

材料中对这五种通知的执行时机进行了明确说明。

# 2. @Before 前置通知

`@Before` 表示：

**前置通知**

它会在目标方法执行之前执行。

例如：

```java
@Before("execution(* com.huaishi.service.*.*(..))")
public void before(JoinPoint joinPoint) {
    log.info("before ...");
}
```

当符合切入点条件的目标方法准备执行时，会先执行 `before()` 通知方法。

因此，`@Before` 适合处理：

- 方法执行前的准备工作
    
- 前置校验
    
- 前置日志记录
    

当前阶段只需要记住：

> `@Before` 在目标方法之前执行。

# 3. @After 后置通知

`@After` 表示：

**后置通知**

它会在目标方法执行之后执行。

例如：

```java
@After("execution(* com.huaishi.service.*.*(..))")
public void after(JoinPoint joinPoint) {
    log.info("after ...");
}
```

它有一个非常重要的特点：

> **无论目标方法是否发生异常，`@After` 都会执行。**

也就是说：

```text
目标方法正常执行完成
→ @After 执行

目标方法发生异常
→ @After 仍然执行
```

材料中明确说明，`@After` 在目标方法之后执行，并且无论是否发生异常都会执行。

# 4. @AfterReturning 返回后通知

`@AfterReturning` 表示：

**返回后通知**

它会在目标方法正常执行完成之后执行。

例如：

```java
@AfterReturning("execution(* com.huaishi.service.*.*(..))")
public void afterReturning(JoinPoint joinPoint) {
    log.info("afterReturning ...");
}
```

它和 `@After` 最大的区别是：

> `@AfterReturning` 只有在目标方法正常执行完成时才会执行。

如果目标方法发生异常：

```text
@AfterReturning 不执行
```

材料在异常测试中也明确说明，程序出现异常时，`@AfterReturning` 不会执行。

# 5. @AfterThrowing 异常后通知

`@AfterThrowing` 表示：

**异常后通知**

它只会在目标方法执行过程中发生异常时触发。

例如：

```java
@AfterThrowing("execution(* com.huaishi.service.*.*(..))")
public void afterThrowing(JoinPoint joinPoint) {
    log.info("afterThrowing ...");
}
```

如果目标方法正常执行完成：

```text
@AfterThrowing 不执行
```

如果目标方法发生异常：

```text
@AfterThrowing 执行
```

材料的正常执行测试中指出，程序没有发生异常时，`@AfterThrowing` 不执行；而发生异常时，它会执行。

# 6. @Around 环绕通知

`@Around` 表示：

**环绕通知**

它是五种通知中功能最强的一种。

例如：

```java
@Around("execution(* com.huaishi.service.*.*(..))")
public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {

    log.info("around before ...");

    Object result = proceedingJoinPoint.proceed();

    log.info("around after ...");

    return result;
}
```

它可以在：

- 目标方法执行前
    
- 目标方法执行后
    

都执行额外逻辑。

因此叫作“环绕通知”。

# 7. @Around 为什么比较特殊

## 7.1 需要手动调用原始方法

和其他通知不同，`@Around` 中需要手动调用：

```java
ProceedingJoinPoint.proceed()
```

才能让目标方法继续执行。

例如：

```java
Object result = proceedingJoinPoint.proceed();
```

如果不调用 `proceed()`，目标方法就不会执行。

材料对此有明确说明：`@Around` 需要自己调用 `ProceedingJoinPoint.proceed()`，其他通知不需要考虑目标方法的执行。

## 7.2 需要接收原始方法的返回值

环绕通知的方法返回值需要使用：

```java
Object
```

例如：

```java
public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable
```

然后接收：

```java
Object result = proceedingJoinPoint.proceed();
```

最后返回：

```java
return result;
```

材料中也明确指出，`@Around` 通知方法的返回值必须使用 `Object` 接收原始方法返回结果，否则无法正常获取原始方法返回值。

# 8. @Around 遇到异常时的表现

考虑下面的环绕通知：

```java
@Around("execution(* com.huaishi.service.*.*(..))")
public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {

    log.info("around before ...");

    Object result = proceedingJoinPoint.proceed();

    log.info("around after ...");

    return result;
}
```

如果：

```java
proceedingJoinPoint.proceed();
```

执行目标方法时发生异常，那么后面的：

```java
log.info("around after ...");
```

不会继续执行。

原因是程序在执行原始方法时已经发生异常，后续代码无法继续执行。

材料中的异常测试也明确说明了这一点。

# 9. 正常执行时各通知的表现

当目标方法正常执行，没有发生异常时：

```text
@Before
执行

@Around
执行目标方法之前的逻辑
执行原始方法
执行目标方法之后的逻辑

@After
执行

@AfterReturning
执行

@AfterThrowing
不执行
```

材料中明确指出，在没有异常的情况下，`@AfterThrowing` 不会执行。

# 10. 发生异常时各通知的表现

当目标方法执行过程中发生异常时：

```text
@Before
执行

@Around
执行目标方法之前的逻辑
执行原始方法时发生异常
后续环绕代码不再继续执行

@After
执行

@AfterReturning
不执行

@AfterThrowing
执行
```

材料对此总结得很明确：

- `@AfterReturning` 不执行
    
- `@AfterThrowing` 执行
    
- `@Around` 中原始方法之后的代码不再执行
    

# 11. 五种通知对比

|通知|执行时机|发生异常时|
|---|---|---|
|`@Before`|目标方法执行前|会执行|
|`@After`|目标方法执行后|仍然执行|
|`@AfterReturning`|目标方法正常返回后|不执行|
|`@AfterThrowing`|目标方法发生异常后|执行|
|`@Around`|目标方法执行前后|`proceed()` 抛异常后，后续代码不再继续|

# 12. 当前阶段需要掌握的核心内容

1. `@Before` 是前置通知，在目标方法执行前运行。
    
2. `@After` 是后置通知，无论目标方法是否发生异常都会执行。
    
3. `@AfterReturning` 只有在目标方法正常执行完成后才会执行。
    
4. `@AfterThrowing` 只在目标方法发生异常时执行。
    
5. `@Around` 可以在目标方法执行前后都执行增强逻辑。
    
6. `@Around` 需要主动调用 `ProceedingJoinPoint.proceed()` 执行原始方法。
    
7. 环绕通知需要接收并返回原始方法的返回值。
    
8. 如果目标方法在 `proceed()` 时发生异常，环绕通知中后续代码不会继续执行。
    
9. 五种通知最核心的区别是它们相对于目标方法的执行时机不同。
    

> [!tip] 一句话总结
> 
> `Spring AOP 提供 @Before、@After、@AfterReturning、@AfterThrowing 和 @Around 五种通知，它们主要区别在于相对于目标方法的执行时机，其中 @Around 功能最强，需要通过 ProceedingJoinPoint.proceed() 主动执行原始方法。`