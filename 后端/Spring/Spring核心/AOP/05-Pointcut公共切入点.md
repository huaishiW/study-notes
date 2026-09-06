# 1. 为什么需要抽取公共切入点

在一个切面类中，可能会定义多个通知。

例如：

```java
@Before("execution(* com.huaishi.service.*.*(..))")
public void before() {
}

@After("execution(* com.huaishi.service.*.*(..))")
public void after() {
}

@Around("execution(* com.huaishi.service.*.*(..))")
public Object around(ProceedingJoinPoint pjp) throws Throwable {
    return pjp.proceed();
}
```

可以看到，这几个通知使用的是同一个切入点表达式：

```java
execution(* com.huaishi.service.*.*(..))
```

如果每一个通知都重复书写相同的切入点表达式，会产生大量重复代码。

当以后需要修改切入点范围时，也需要逐个修改每一个通知中的表达式，维护起来比较麻烦。

因此，可以把重复的切入点表达式统一抽取出来。

# 2. @Pointcut

Spring 提供了：

```java
@Pointcut
```

用于抽取公共切入点表达式。

例如：

```java
@Pointcut("execution(* com.huaishi.service.*.*(..))")
private void pt() {
}
```

这里：

```java
execution(* com.huaishi.service.*.*(..))
```

是真正的切入点表达式。

而：

```java
pt()
```

则作为这个公共切入点的引用名称。

以后其他通知就可以直接引用：

```java
pt()
```

而不需要重复编写完整表达式。

# 3. 定义公共切入点

一个公共切入点通常可以这样定义：

```java
@Pointcut("execution(* com.huaishi.service.*.*(..))")
private void pt() {
}
```

这个方法本身不需要编写业务逻辑。

它的主要作用是：

> 为一个公共的切入点表达式提供引用名称。

因此，这个方法通常被称为：

**切入点方法**

# 4. 在同一个切面类中引用

定义公共切入点以后，同一个切面类中的通知可以直接通过方法名进行引用。

例如：

```java
@Slf4j
@Component
@Aspect
public class MyAspect {

    @Pointcut("execution(* com.huaishi.service.*.*(..))")
    private void pt() {
    }

    @Before("pt()")
    public void before() {
        log.info("before ...");
    }

    @After("pt()")
    public void after() {
        log.info("after ...");
    }
}
```

这里：

```java
@Before("pt()")
```

以及：

```java
@After("pt()")
```

都引用了同一个公共切入点。

这样就避免了重复书写完整的切入点表达式。

# 5. 抽取公共切入点的好处

把切入点表达式抽取出来以后，可以减少重复代码。

例如原来多个通知都写：

```java
execution(* com.huaishi.service.*.*(..))
```

抽取后只需要统一定义一次：

```java
@Pointcut("execution(* com.huaishi.service.*.*(..))")
private void pt() {
}
```

其他通知直接引用：

```java
@Before("pt()")
@After("pt()")
@Around("pt()")
```

这样做主要有两个好处：

- 减少重复代码
    
- 修改切入点时只需要修改一处
    

因此，更方便维护。

# 6. private 切入点方法

如果切入点方法使用：

```java
private
```

修饰，例如：

```java
@Pointcut("execution(* com.huaishi.service.*.*(..))")
private void pt() {
}
```

那么这个切入点只能在：

**当前切面类内部**

进行引用。

例如：

```java
@Before("pt()")
```

这种写法只能在定义 `pt()` 的同一个切面类中使用。

材料中明确指出，`private` 修饰的切入点方法仅能在当前切面类中引用。

# 7. public 切入点方法

如果其他切面类也需要使用这个公共切入点，就需要把：

```java
private
```

修改为：

```java
public
```

例如：

```java
@Pointcut("execution(* com.huaishi.service.*.*(..))")
public void pt() {
}
```

这样其他切面类才能引用这个切入点。

# 8. 不同切面类之间引用切入点

假设公共切入点定义在：

```java
com.huaishi.aspect.MyAspect1
```

中：

```java
@Aspect
@Component
public class MyAspect1 {

    @Pointcut("execution(* com.huaishi.service.*.*(..))")
    public void pt() {
    }
}
```

另一个切面类如果想使用这个切入点，需要使用完整路径进行引用：

```java
@Before("com.huaishi.aspect.MyAspect1.pt()")
public void before() {
    log.info("MyAspect2 -> before ...");
}
```

也就是说，跨切面类引用时通常需要写：

```text
包名.切面类名.切入点方法名()
```

材料中的示例就是通过这种方式引用其他切面类中的公共切入点。

# 9. 同类引用与跨类引用的区别

如果是在同一个切面类中引用：

```java
@Before("pt()")
```

直接写切入点方法名即可。

如果是在其他切面类中引用：

```java
@Before("com.huaishi.aspect.MyAspect1.pt()")
```

则需要使用完整的限定路径。

因此可以简单区分为：

```text
同一个切面类
→ 直接使用 pt()

其他切面类
→ 使用 包名.类名.pt()
```

# 10. @Pointcut 的本质作用

`@Pointcut` 并不会单独执行通知逻辑。

它的核心作用只是：

> 把重复使用的切入点表达式提取出来，并提供一个可以重复引用的名称。

真正决定“什么时候执行什么逻辑”的，仍然是：

```java
@Before
@After
@Around
@AfterReturning
@AfterThrowing
```

这些通知注解。

`@Pointcut` 只是帮助这些通知统一复用切入点表达式。

# 11. 当前阶段需要掌握的核心内容

1. 多个通知重复使用相同切入点表达式时，会产生大量重复代码。
    
2. `@Pointcut` 可以将公共切入点表达式抽取出来。
    
3. `@Pointcut` 通常标注在一个没有实际业务逻辑的方法上。
    
4. 其他通知可以通过切入点方法名引用公共切入点。
    
5. 同一个切面类中可以直接使用 `pt()` 引用。
    
6. `private` 修饰的切入点方法只能在当前切面类内部使用。
    
7. 如果其他切面类也需要引用，应将切入点方法声明为 `public`。
    
8. 跨切面类引用时，需要使用完整的 `包名.类名.方法名()`。
    
9. `@Pointcut` 的主要作用是复用切入点表达式，减少重复代码并提高维护性。
    

> [!tip] 一句话总结
> 
> `@Pointcut 用于抽取并复用公共切入点表达式，同一切面类可以直接通过切入点方法名引用，而跨切面类引用时需要使用公开的切入点方法及其完整限定路径。`