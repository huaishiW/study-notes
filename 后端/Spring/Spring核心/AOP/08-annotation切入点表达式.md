# 1. 为什么需要 @annotation

前面学习的：

```text
execution(...)
```

主要是根据方法的：

- 包名
    
- 类名
    
- 方法名
    
- 参数
    
- 返回值
    

等方法签名信息进行匹配。

如果需要增强的方法本身具有明显规律，那么使用 `execution` 会比较方便。

例如：

```java
findAll()
findById()
findDept()
```

这些方法都以：

```text
find
```

开头，就可以通过方法名规律进行统一匹配。

但是，如果需要增强的方法名称没有明显规律，例如：

```java
list()
delete()
```

两个方法之间没有统一的方法名前缀或后缀，那么通过 `execution` 描述时就会比较繁琐。

这种情况下，可以使用：

```text
@annotation(...)
```

根据方法上的注解进行匹配。

# 2. 什么是 @annotation

`@annotation` 是 Spring AOP 中的一种切入点表达式。

它的作用是：

> **根据方法上是否存在指定注解来匹配目标方法。**

也就是说，不再根据方法名称、包结构等信息判断是否需要增强，而是通过一个注解明确标识：

> 这个方法需要被 AOP 处理。

例如，可以自定义：

```java
@LogOperation
```

然后将它添加到需要增强的方法上。

切面中再通过：

```java
@annotation(com.huaishi.anno.LogOperation)
```

匹配所有带有 `@LogOperation` 的方法。

# 3. @annotation 的基本使用思路

使用 `@annotation` 通常需要完成两个核心步骤：

1. 定义一个自定义注解。
    
2. 将自定义注解添加到需要被 AOP 增强的方法上。
    

随后，在切面类中通过：

```text
@annotation(...)
```

匹配这些方法。

材料中也是按照这一方式进行实现的。

# 4. 定义自定义注解

材料中定义了一个：

```java
@LogOperation
```

注解。

代码如下：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogOperation {
}
```

这个注解的作用是：

> 标识哪些方法需要被 AOP 程序匹配。

# 5. @Target

自定义注解中使用：

```java
@Target(ElementType.METHOD)
```

表示：

> 当前注解可以标注在方法上。

因为当前需求是通过注解标识需要被 AOP 增强的方法，所以这里指定：

```java
ElementType.METHOD
```

例如：

```java
@LogOperation
public void delete(Integer id) {
}
```

就是将自定义注解直接标注到方法上。

# 6. @Retention

自定义注解中还使用：

```java
@Retention(RetentionPolicy.RUNTIME)
```

表示：

> 当前注解在程序运行期间仍然保留。

Spring AOP 需要在程序运行过程中识别方法上的这个注解，因此这里使用：

```java
RetentionPolicy.RUNTIME
```

当前阶段只需要理解：

> 要让 Spring AOP 在运行时根据注解匹配方法，这个注解需要能够保留到运行阶段。

# 7. 在目标方法上添加自定义注解

定义完：

```java
@LogOperation
```

之后，就可以把它添加到真正需要增强的方法上。

例如：

```java
@Override
@LogOperation
public List<Dept> list() {
    List<Dept> deptList = deptMapper.list();
    return deptList;
}
```

以及：

```java
@Override
@LogOperation
public void delete(Integer id) {
    deptMapper.delete(id);
}
```

这里：

```java
list()
delete()
```

虽然方法名没有统一规律，但是因为它们都添加了：

```java
@LogOperation
```

所以可以通过注解统一匹配。

# 8. 没有添加注解的方法不会被匹配

例如同一个业务类中还有：

```java
public void save(Dept dept) {
}
```

```java
public Dept getById(Integer id) {
}
```

```java
public void update(Dept dept) {
}
```

如果这些方法没有添加：

```java
@LogOperation
```

那么使用：

```java
@annotation(com.huaishi.anno.LogOperation)
```

时，它们就不会被当前切入点匹配。

因此，`@annotation` 可以实现非常明确的控制：

> 需要增强哪个方法，就给哪个方法添加对应注解。

# 9. 在切面中使用 @annotation

定义好注解，并给目标方法添加注解后，就可以在切面类中进行匹配。

例如：

```java
@Slf4j
@Component
@Aspect
public class MyAspect {

    @Before("@annotation(com.huaishi.anno.LogOperation)")
    public void before() {
        log.info("before ...");
    }

    @After("@annotation(com.huaishi.anno.LogOperation)")
    public void after() {
        log.info("after ...");
    }
}
```

这里：

```java
@annotation(com.huaishi.anno.LogOperation)
```

表示：

> 匹配所有标注了 `@LogOperation` 注解的方法。

因此，只要目标方法上存在这个注解，对应通知就可以被应用。

# 10. @annotation 中为什么写完整类名

示例中的表达式是：

```java
@annotation(com.huaishi.anno.LogOperation)
```

其中：

```text
com.huaishi.anno.LogOperation
```

是自定义注解的完整类名。

也就是：

```text
包名 + 注解类名
```

这样 Spring 就能够明确知道当前切入点需要匹配的是哪个注解。

# 11. execution 与 @annotation 的区别

这两种切入点表达式的核心区别在于：

## 11.1 execution

```text
execution(...)
```

主要根据：

> 方法本身的描述信息进行匹配。

例如：

- 包名
    
- 类名
    
- 方法名
    
- 参数
    
- 返回值
    

适合方法存在明显规律的情况。

---

## 11.2 @annotation

```text
@annotation(...)
```

主要根据：

> 方法上是否存在指定注解进行匹配。

适合：

- 方法名没有明显规律
    
- 只想增强某些指定方法
    
- 希望通过注解显式标识目标方法
    

材料也明确指出，如果切入点方法名称不规则，使用 `execution` 会比较繁琐，而通过自定义注解进行匹配会更加灵活。

# 12. 两种方式的使用对比

|切入点表达式|匹配依据|更适合的场景|
|---|---|---|
|`execution(...)`|方法签名信息|方法具有明显命名或包结构规律|
|`@annotation(...)`|方法上的注解|方法没有明显规律，需要精确指定|

例如，如果所有查询方法都是：

```text
findXxx()
```

可以考虑使用：

```text
execution(...)
```

如果需要增强的是：

```text
list()
delete()
save()
```

这些没有明显统一规律的方法，则可以通过：

```java
@LogOperation
```

逐个标记。

# 13. @annotation 的主要优势

## 13.1 匹配方式更加直观

看到：

```java
@LogOperation
public void delete(Integer id) {
}
```

就可以直接知道：

> 当前方法属于需要进行日志相关增强的方法。

---

## 13.2 不依赖方法名称规律

方法可以叫：

```java
list()
delete()
save()
update()
```

只要添加了相同注解，就可以统一匹配。

---

## 13.3 控制粒度更加明确

需要哪个方法被增强，就在对应方法上添加注解。

不需要增强的方法，不添加即可。

因此，目标方法范围比较容易控制。

# 14. @annotation 的代价

相比直接使用：

```text
execution(...)
```

`@annotation` 需要额外定义一个自定义注解。

并且还需要在目标方法上主动添加这个注解。

材料中也指出，这种方式虽然多了一步自定义注解的操作，但整体更加灵活。

# 15. @annotation 的典型使用思路

在实际使用时，可以把某种需要统一处理的功能定义成一个具有明确语义的注解。

例如材料中的：

```java
@LogOperation
```

表示：

> 当前方法需要进行操作日志相关处理。

以后切面只需要匹配：

```java
@annotation(com.huaishi.anno.LogOperation)
```

即可统一处理所有带有该注解的方法。

这种方式非常适合材料后续的：

```text
AOP操作日志案例
```

因为增、删、改方法名称并不一定具有统一规律，所以可以通过 `@LogOperation` 明确标记需要记录操作日志的方法。

# 16. 切入点表达式也可以组合

材料中说明，根据业务需要，可以使用：

```text
&&
||
!
```

组合复杂的切入点表达式。

因此：

```text
execution(...)
```

和：

```text
@annotation(...)
```

并不是完全互斥的两套机制。

在更复杂的业务场景中，可以根据实际需要组合不同条件。

当前阶段重点掌握两种表达式各自最常见的使用方式即可。

# 17. 当前阶段需要掌握的核心内容

1. `@annotation` 是 Spring AOP 中一种基于注解匹配方法的切入点表达式。
    
2. 当目标方法名称没有明显规律时，使用 `@annotation` 会更加灵活。
    
3. 使用 `@annotation` 前通常需要先定义自定义注解。
    
4. `@Target(ElementType.METHOD)` 表示注解可以作用在方法上。
    
5. `@Retention(RetentionPolicy.RUNTIME)` 表示注解在运行时仍然保留。
    
6. 哪个方法需要被增强，就可以在该方法上添加对应自定义注解。
    
7. `@annotation(注解完整类名)` 可以匹配所有标注了该注解的方法。
    
8. `execution` 主要根据方法签名匹配，而 `@annotation` 主要根据方法上的注解匹配。
    
9. `execution` 更适合具有明显规律的方法，`@annotation` 更适合没有明显命名规律、需要明确指定的方法。
    
10. `execution`、`@annotation` 等条件还可以根据需要通过逻辑运算符进行组合。
    

> [!tip] 一句话总结
> 
> `@annotation 切入点表达式通过方法上的自定义注解匹配目标方法，适合方法名称没有统一规律但需要精确指定增强范围的场景。`