# 1. 案例需求

当前案例要求：

> 对系统中的新增、删除、修改相关接口记录操作日志，并将日志保存到数据库中。

需要记录的信息包括：

- 操作人
    
- 操作时间
    
- 执行方法的全类名
    
- 执行方法名
    
- 方法运行参数
    
- 方法返回值
    
- 方法执行耗时
    

这些信息可以帮助后续追踪系统中的具体操作。

# 2. 为什么适合使用 AOP

系统中需要记录日志的增、删、改方法通常很多。

如果直接在每个接口方法中重复编写：

```text
获取操作人
记录操作时间
获取类名和方法名
获取方法参数
统计方法耗时
保存日志
```

会产生大量重复代码。

而这些操作日志逻辑本质上属于多个接口共同需要执行的公共功能。

因此，可以把它们统一抽取到 AOP 通知中，在不修改原有业务逻辑的情况下完成增强。

材料也正是基于这一点选择使用 AOP 实现操作日志。

# 3. 为什么使用 @Around

操作日志中有两个信息比较特殊：

```text
方法返回值
方法执行耗时
```

方法返回值只有在原始方法执行完成后才能获取。

而统计方法执行耗时，则需要：

```text
原始方法执行前记录开始时间
原始方法执行后记录结束时间
```

因此，需要同时控制原始方法执行前后的逻辑。

材料最终选择：

```java
@Around
```

环绕通知。

# 4. 为什么使用 @annotation

需要记录操作日志的是系统中的：

- 新增方法
    
- 删除方法
    
- 修改方法
    

但是这些方法的名称并不一定具有统一规律。

如果单纯使用：

```text
execution(...)
```

逐个描述这些方法，会比较繁琐。

因此，材料选择定义一个自定义注解：

```java
@LogOperation
```

然后通过：

```text
@annotation(...)
```

匹配所有添加了该注解的方法。

这样需要记录日志的方法只需要主动添加：

```java
@LogOperation
```

即可。

# 5. 引入 AOP 依赖

首先需要在项目中引入：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

用于提供 Spring AOP 相关功能。

# 6. 操作日志数据结构

材料中定义了：

```text
operate_log
```

操作日志表。

主要字段包括：

```text
id
operate_emp_id
operate_time
class_name
method_name
method_params
return_value
cost_time
```

分别用于保存：

- 日志 ID
    
- 操作员工 ID
    
- 操作时间
    
- 操作类名
    
- 操作方法名
    
- 方法参数
    
- 方法返回值
    
- 方法执行耗时
    

对应实体类为：

```java
OperateLog
```

# 7. 保存操作日志

材料已经提供：

```java
OperateLogMapper
```

用于将日志插入：

```text
operate_log
```

表。

其中定义：

```java
public void insert(OperateLog log);
```

因此，AOP 程序最终只需要构建一个：

```java
OperateLog
```

对象，再调用 Mapper 保存即可。

# 8. 定义 @LogOperation 注解

首先定义自定义注解：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogOperation {
}
```

其中：

```java
@Target(ElementType.METHOD)
```

表示该注解用于方法。

```java
@Retention(RetentionPolicy.RUNTIME)
```

表示这个注解在运行时仍然保留，便于 AOP 在运行阶段识别。

它的作用就是：

> 标识哪些方法需要记录操作日志。

# 9. 在目标方法上添加 @LogOperation

对于需要记录操作日志的方法，添加：

```java
@LogOperation
```

例如：

```java
@LogOperation
@PostMapping
public Result save(@RequestBody Clazz clazz) {
    clazzService.save(clazz);
    return Result.success();
}
```

这样，该方法就可以被：

```text
@annotation(...)
```

切入点表达式匹配。

没有添加该注解的方法，则不会被当前操作日志切面匹配。

# 10. 定义操作日志切面类

材料中定义了：

```java
OperationLogAspect
```

切面类。

基本结构如下：

```java
@Aspect
@Component
public class OperationLogAspect {

    @Autowired
    private OperateLogMapper operateLogMapper;

}
```

其中：

```java
@Aspect
```

表示当前类是切面类。

```java
@Component
```

表示将当前切面类交给 Spring IOC 容器管理。

# 11. 定义环绕通知

操作日志使用：

```java
@Around
```

定义环绕通知：

```java
@Around("@annotation(log)")
public Object around(
        ProceedingJoinPoint joinPoint,
        LogOperation log) throws Throwable {

}
```

这里的：

```java
@annotation(log)
```

表示匹配带有 `@LogOperation` 注解的方法。

参数：

```java
ProceedingJoinPoint joinPoint
```

则用于：

- 获取连接点信息
    
- 执行原始方法
    

# 12. 统计方法执行耗时

首先记录开始时间：

```java
long startTime = System.currentTimeMillis();
```

然后执行原始方法：

```java
Object result = joinPoint.proceed();
```

原始方法执行完成后记录结束时间：

```java
long endTime = System.currentTimeMillis();
```

最终计算：

```java
long costTime = endTime - startTime;
```

得到当前方法执行耗时。

# 13. 获取操作类名

通过：

```java
joinPoint.getTarget()
```

获取当前目标对象。

继续调用：

```java
joinPoint.getTarget().getClass().getName()
```

即可获得目标类完整类名。

然后保存到：

```java
operateLog.setClassName(...);
```

中。

材料中的代码正是通过连接点目标对象获取操作类名。

# 14. 获取方法名

可以通过：

```java
joinPoint.getSignature().getName()
```

获取当前执行的方法名称。

然后保存到：

```java
operateLog.setMethodName(...);
```

中。

例如执行：

```java
save()
```

就可以获得当前方法名并记录到操作日志中。

# 15. 获取方法参数

通过：

```java
joinPoint.getArgs()
```

可以获得方法调用时实际传入的参数。

材料中将参数转换成字符串：

```java
Arrays.toString(joinPoint.getArgs())
```

然后保存到：

```java
methodParams
```

字段中。

# 16. 获取方法返回值

执行：

```java
Object result = joinPoint.proceed();
```

之后：

```java
result
```

就是原始方法执行后的返回结果。

材料中使用：

```java
operateLog.setReturnValue(result.toString());
```

将返回结果记录到操作日志中。

# 17. 构建 OperateLog

获得相关信息后，可以构建：

```java
OperateLog operateLog = new OperateLog();
```

并依次设置：

```java
operateLog.setOperateEmpId(...);
operateLog.setOperateTime(LocalDateTime.now());
operateLog.setClassName(...);
operateLog.setMethodName(...);
operateLog.setMethodParams(...);
operateLog.setReturnValue(...);
operateLog.setCostTime(...);
```

这样就获得了一条完整的操作日志数据。

# 18. 将日志保存到数据库

构建完成之后，通过：

```java
operateLogMapper.insert(operateLog);
```

将日志保存到：

```text
operate_log
```

表。

最后：

```java
return result;
```

继续返回原始方法执行结果。

# 19. 获取当前登录员工 ID

操作日志还需要保存：

```text
操作人 ID
```

材料中当前登录员工的信息来源于 JWT。

`TokenFilter` 在解析 JWT 后可以获得当前登录员工 ID。

但是问题是：

> Filter 中解析出来的员工 ID，如何传递给后面的 AOP 程序？

材料采用：

```java
ThreadLocal
```

在同一个请求线程中共享当前登录员工 ID。

# 20. 使用 CurrentHolder 保存当前用户

材料定义了：

```java
CurrentHolder
```

工具类：

```java
public class CurrentHolder {

    private static final ThreadLocal<Integer> CURRENT_LOCAL
            = new ThreadLocal<>();

    public static void setCurrentId(Integer employeeId) {
        CURRENT_LOCAL.set(employeeId);
    }

    public static Integer getCurrentId() {
        return CURRENT_LOCAL.get();
    }

    public static void remove() {
        CURRENT_LOCAL.remove();
    }
}
```

主要提供三个操作：

```text
setCurrentId()
getCurrentId()
remove()
```

分别负责：

- 保存当前员工 ID
    
- 获取当前员工 ID
    
- 清除当前线程中的员工 ID
    

# 21. TokenFilter 保存当前登录员工 ID

在 `TokenFilter` 中解析 JWT 后：

```java
Claims claims = JwtUtils.parseJWT(token);
Integer empId =
        Integer.valueOf(claims.get("id").toString());
```

再调用：

```java
CurrentHolder.setCurrentId(empId);
```

把当前登录员工 ID 保存到 ThreadLocal。

请求执行完成后再调用：

```java
CurrentHolder.remove();
```

清除线程中的数据。

# 22. AOP 获取当前登录员工

在切面中可以直接调用：

```java
CurrentHolder.getCurrentId()
```

获取当前请求对应的员工 ID。

然后保存到：

```java
operateLog.setOperateEmpId(...)
```

中。

这样，操作日志中就能够记录当前操作是谁执行的。

# 23. ThreadLocal 在这个案例中的作用

在这个案例中：

```text
TokenFilter
```

负责解析 JWT 并获取当前员工 ID。

而：

```text
OperationLogAspect
```

需要使用这个员工 ID 记录日志。

两者处于同一个请求线程中，因此可以通过 `ThreadLocal` 完成数据共享。

材料最后也明确总结：

> 在同一个线程、同一个请求中进行数据共享，可以使用 ThreadLocal。

ThreadLocal 的完整原理不属于 AOP 本身，可以在单独的 `ThreadLocal.md` 中继续整理。

# 24. 案例中使用到的 AOP 知识

这个案例基本串联了前面学习到的主要 AOP 内容。

使用：

```java
@Aspect
```

定义切面类。

使用：

```java
@Around
```

定义环绕通知。

使用：

```text
@annotation(...)
```

匹配带有自定义注解的方法。

使用：

```java
@LogOperation
```

标记需要记录日志的目标方法。

使用：

```java
ProceedingJoinPoint
```

获取连接点信息并执行原始方法。

通过：

```java
joinPoint.proceed()
```

执行原始业务方法。

最终把获得的操作信息保存到数据库。

# 25. 当前阶段需要掌握的核心内容

1. 操作日志属于多个接口共同需要的公共功能，适合使用 AOP 实现。
    
2. 操作日志需要获取方法执行前后的信息，因此材料选择 `@Around` 环绕通知。
    
3. 增、删、改方法命名不一定有统一规律，因此材料选择 `@annotation` 匹配。
    
4. `@LogOperation` 用于标识哪些方法需要记录操作日志。
    
5. `ProceedingJoinPoint.proceed()` 用于执行原始方法并获得返回结果。
    
6. `getTarget()` 可以获取目标对象。
    
7. `getSignature().getName()` 可以获取方法名称。
    
8. `getArgs()` 可以获取方法实际参数。
    
9. 原始方法执行前后记录时间，可以计算方法执行耗时。
    
10. 获取到完整操作信息后，可以构建 `OperateLog` 并通过 Mapper 保存到数据库。
    
11. 当前登录员工 ID 来自 JWT，材料通过 ThreadLocal 在同一个请求线程中传递给 AOP。
    
12. 请求处理完成后，需要通过 `CurrentHolder.remove()` 清理当前线程中保存的数据。
    

> [!tip] 一句话总结
> 
> `AOP操作日志通过 @LogOperation 标记目标方法，使用 @Around 和 ProceedingJoinPoint 获取类名、方法名、参数、返回值及执行耗时，并结合当前登录用户信息将完整操作记录保存到数据库。`