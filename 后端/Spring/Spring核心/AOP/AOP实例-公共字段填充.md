# 1. 问题背景

在实际项目中，很多数据表都会存在一些含义相同的公共字段，例如：

|字段|含义|类型|
|---|---|---|
|`create_time`|创建时间|`datetime`|
|`create_user`|创建人 ID|`bigint`|
|`update_time`|修改时间|`datetime`|
|`update_user`|修改人 ID|`bigint`|

这些字段需要在不同的数据库操作中自动设置。

## 字段填充规则

|字段|INSERT|UPDATE|
|---|:-:|:-:|
|`create_time`|✓||
|`create_user`|✓||
|`update_time`|✓|✓|
|`update_user`|✓|✓|

新增数据时，`createTime`、`updateTime` 设置为当前时间，`createUser`、`updateUser` 设置为当前登录用户 ID。

更新数据时，只需要设置 `updateTime` 和 `updateUser`。

---

# 2. 原始实现方式

最直接的实现方式是在每个业务方法中手动设置公共字段。

例如新增员工：

```java
/**
 * 新增员工
 * @param employeeDTO
 */
public void save(EmployeeDTO employeeDTO) {
    // 设置当前记录的创建时间和修改时间
    employee.setCreateTime(LocalDateTime.now());
    employee.setUpdateTime(LocalDateTime.now());

    // 设置当前记录创建人id和修改人id
    employee.setCreateUser(BaseContext.getCurrentId());
    employee.setUpdateUser(BaseContext.getCurrentId());

    employeeMapper.insert(employee);
}
```

更新员工：

```java
/**
 * 编辑员工信息
 * @param employeeDTO
 */
public void update(EmployeeDTO employeeDTO) {
    employee.setUpdateTime(LocalDateTime.now());
    employee.setUpdateUser(BaseContext.getCurrentId());

    employeeMapper.update(employee);
}
```

分类等其他业务同样需要重复编写这些代码。

## 存在的问题

这种方式虽然简单，但公共字段的处理逻辑会分散在大量业务方法中，导致：

1. **代码重复**
2. **业务代码中混入公共逻辑**
3. **后续维护成本增加**
4. **容易出现遗漏或处理不一致**

因此，需要将公共字段的处理逻辑统一抽离出来。

---

# 3. 核心解决方案：AOP + 自定义注解

使用 **AOP 切面编程**实现公共字段自动填充。

核心思想是：

> 使用 `@AutoFill` 标记需要自动填充的方法，AOP 切面统一拦截这些方法，并根据操作类型自动完成公共字段的赋值。

这样，Service 层只需要处理业务逻辑，不再需要手动设置创建人、创建时间、修改人和修改时间。

整个机制可以概括为：

> **Mapper 方法通过 `@AutoFill` 声明需要自动填充，`AutoFillAspect` 在方法执行前完成字段赋值。**

---

# 4. 实现思路

实现公共字段自动填充主要分为三个步骤：

## ① 自定义 `AutoFill` 注解

用于标识需要进行公共字段自动填充的 Mapper 方法，同时通过注解参数指定数据库操作类型。

## ② 创建 `AutoFillAspect` 切面

统一拦截添加了 `@AutoFill` 注解的方法，根据操作类型完成公共字段赋值。

## ③ 在 Mapper 方法上添加 `@AutoFill`

明确告诉 AOP 哪些 Mapper 方法需要进行自动填充，以及当前方法属于新增还是修改操作。

整个功能涉及的主要技术包括：

- 枚举
- 自定义注解
- AOP
- 反射

---

# 5. OperationType：表示数据库操作类型

使用枚举区分 `INSERT` 和 `UPDATE`：

```java
/**
 * 数据库操作类型
 */
public enum OperationType {

    /**
     * 更新操作
     */
    UPDATE,

    /**
     * 插入操作
     */
    INSERT
}
```

这样 `AutoFill` 就可以通过 `OperationType` 表示当前 Mapper 方法需要按照哪一种规则进行公共字段填充。

---

# 6. AutoFill：自定义注解

```java
/**
 * 自定义注解，用于标识某个方法需要进行功能字段自动填充处理
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AutoFill {

    // 数据库操作类型：UPDATE INSERT
    OperationType value();
}
```

## 6.1 `@Target`

```java
@Target(ElementType.METHOD)
```

表示 `@AutoFill` 只能作用于**方法**。

例如：

```java
@AutoFill(value = OperationType.INSERT)
void insert(Category category);
```

---

## 6.2 `@Retention`

```java
@Retention(RetentionPolicy.RUNTIME)
```

表示该注解在**运行时仍然存在**。

这是因为 AOP 切面需要在程序运行过程中通过反射获取 Mapper 方法上的 `@AutoFill` 注解。

---

## 6.3 `value()`

```java
OperationType value();
```

用于指定当前方法对应的数据库操作类型：

```java
@AutoFill(value = OperationType.INSERT)
```

表示新增操作。

```java
@AutoFill(value = OperationType.UPDATE)
```

表示更新操作。

因此，`AutoFill` 实际上给 Mapper 方法增加了一项描述信息：

> **这个方法需要自动填充，并且应该按照哪种操作规则进行填充。**

---

# 7. AutoFillAspect：自动填充切面

首先定义切面：

```java
/**
 * 自定义切面，实现公共字段自动填充处理逻辑
 */
@Aspect
@Component
@Slf4j
public class AutoFillAspect {

    /**
     * 切入点
     */
    @Pointcut("execution(* com.sky.mapper.*.*(..)) && @annotation(com.sky.annotation.AutoFill)")
    public void autoFillPointCut(){}

    /**
     * 前置通知，在通知中进行公共字段的赋值
     */
    @Before("autoFillPointCut()")
    public void autoFill(JoinPoint joinPoint){
        log.info("开始进行公共字段自动填充...");
    }
}
```

这里需要重点理解三个注解：

### `@Aspect`

```java
@Aspect
```

表示当前类是一个 **AOP 切面类**。

### `@Component`

```java
@Component
```

将切面交给 Spring 容器管理。

### `@Before`

```java
@Before("autoFillPointCut()")
```

表示在目标 Mapper 方法执行**之前**执行 `autoFill()`。

---

# 8. Pointcut：确定哪些方法需要拦截

切入点定义：

```java
@Pointcut("execution(* com.sky.mapper.*.*(..)) && @annotation(com.sky.annotation.AutoFill)")
public void autoFillPointCut(){}
```

它的核心作用是：

> **拦截 `com.sky.mapper` 包下，并且添加了 `@AutoFill` 注解的方法。**

因此，并不是所有 Mapper 方法都会触发公共字段自动填充。

只有明确添加：

```java
@AutoFill(...)
```

的方法才会被切面处理。

---

# 9. AutoFillAspect 的完整实现

```java
/**
 * 自定义切面，实现公共字段自动填充处理逻辑
 */
@Aspect
@Component
@Slf4j
public class AutoFillAspect {

    /**
     * 切入点
     */
    @Pointcut("execution(* com.sky.mapper.*.*(..)) && @annotation(com.sky.annotation.AutoFill)")
    public void autoFillPointCut(){}

    /**
     * 前置通知，在通知中进行公共字段的赋值
     */
    @Before("autoFillPointCut()")
    public void autoFill(JoinPoint joinPoint){

        log.info("开始进行公共字段自动填充...");

        // 获取当前被拦截的方法上的数据库操作类型
        MethodSignature signature =
                (MethodSignature) joinPoint.getSignature();

        AutoFill autoFill =
                signature.getMethod().getAnnotation(AutoFill.class);

        OperationType operationType = autoFill.value();

        // 获取当前被拦截的方法参数
        Object[] args = joinPoint.getArgs();

        if(args == null || args.length == 0){
            return;
        }

        // 获取实体对象
        Object entity = args[0];

        // 准备需要填充的数据
        LocalDateTime now = LocalDateTime.now();
        Long currentId = BaseContext.getCurrentId();

        // 根据操作类型进行不同的字段填充
        if(operationType == OperationType.INSERT){

            // INSERT：填充4个公共字段
            try {
                Method setCreateTime =
                        entity.getClass().getDeclaredMethod(
                                AutoFillConstant.SET_CREATE_TIME,
                                LocalDateTime.class);

                Method setCreateUser =
                        entity.getClass().getDeclaredMethod(
                                AutoFillConstant.SET_CREATE_USER,
                                Long.class);

                Method setUpdateTime =
                        entity.getClass().getDeclaredMethod(
                                AutoFillConstant.SET_UPDATE_TIME,
                                LocalDateTime.class);

                Method setUpdateUser =
                        entity.getClass().getDeclaredMethod(
                                AutoFillConstant.SET_UPDATE_USER,
                                Long.class);

                // 通过反射为对象属性赋值
                setCreateTime.invoke(entity, now);
                setCreateUser.invoke(entity, currentId);
                setUpdateTime.invoke(entity, now);
                setUpdateUser.invoke(entity, currentId);

            } catch (Exception e) {
                e.printStackTrace();
            }

        } else if(operationType == OperationType.UPDATE){

            // UPDATE：填充2个公共字段
            try {
                Method setUpdateTime =
                        entity.getClass().getDeclaredMethod(
                                AutoFillConstant.SET_UPDATE_TIME,
                                LocalDateTime.class);

                Method setUpdateUser =
                        entity.getClass().getDeclaredMethod(
                                AutoFillConstant.SET_UPDATE_USER,
                                Long.class);

                // 通过反射为对象属性赋值
                setUpdateTime.invoke(entity, now);
                setUpdateUser.invoke(entity, currentId);

            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```

---

# 10. AutoFillAspect 的核心逻辑

这段代码较长，但真正需要理解的是几个关键操作。

## 10.1 获取 `@AutoFill` 信息

```java
MethodSignature signature =
        (MethodSignature) joinPoint.getSignature();

AutoFill autoFill =
        signature.getMethod().getAnnotation(AutoFill.class);

OperationType operationType = autoFill.value();
```

这里通过 `JoinPoint` 获取当前被拦截的方法，再通过反射获取方法上的 `@AutoFill` 注解，最后得到 `INSERT` 或 `UPDATE`。

---

## 10.2 获取方法参数

```java
Object[] args = joinPoint.getArgs();

if(args == null || args.length == 0){
    return;
}

Object entity = args[0];
```

`joinPoint.getArgs()` 可以获取目标方法运行时传入的参数。

例如：

```java
void insert(Category category);
```

执行：

```java
categoryMapper.insert(category);
```

那么 `args[0]` 就是当前的 `Category` 对象，也就是需要进行公共字段填充的实体对象。

---

## 10.3 获取公共字段需要的数据

```java
LocalDateTime now = LocalDateTime.now();
Long currentId = BaseContext.getCurrentId();
```

这里获取：

- `now`：当前时间
- `currentId`：当前登录用户 ID

它们将作为公共字段的填充值。

---

# 11. 为什么需要反射？

自动填充功能需要同时支持不同的实体对象，例如：

```text
Employee
Category
其他业务实体
```

如果直接编写：

```java
employee.setCreateTime(...);
category.setCreateTime(...);
```

那么切面就需要知道每一种实体的具体类型，导致公共逻辑与具体业务对象产生耦合。

反射则可以根据方法名称动态调用 setter：

```java
setCreateTime.invoke(entity, now);
```

切面不需要关心 `entity` 具体是什么类型，只需要它具有对应的方法即可。

因此：

> **反射使公共字段自动填充逻辑能够与具体实体类型解耦。**

这是这个案例中反射最重要的作用。

---

# 12. INSERT 和 UPDATE 的处理逻辑

### INSERT

新增数据时需要填充四个字段：

```java
setCreateTime.invoke(entity, now);
setCreateUser.invoke(entity, currentId);
setUpdateTime.invoke(entity, now);
setUpdateUser.invoke(entity, currentId);
```

对应：

|字段|数据来源|
|---|---|
|`createTime`|当前时间|
|`createUser`|当前登录用户 ID|
|`updateTime`|当前时间|
|`updateUser`|当前登录用户 ID|

### UPDATE

更新数据时只需要修改两个字段：

```java
setUpdateTime.invoke(entity, now);
setUpdateUser.invoke(entity, currentId);
```

对应：

|字段|数据来源|
|---|---|
|`updateTime`|当前时间|
|`updateUser`|当前登录用户 ID|

---

# 13. 在 Mapper 中使用 `@AutoFill`

以 `CategoryMapper` 为例：

```java
@Mapper
public interface CategoryMapper {

    /**
     * 插入数据
     */
    @Insert("insert into category(type, name, sort, status, create_time, update_time, create_user, update_user)" +
            " VALUES" +
            " (#{type}, #{name}, #{sort}, #{status}, #{createTime}, #{updateTime}, #{createUser}, #{updateUser})")
    @AutoFill(value = OperationType.INSERT)
    void insert(Category category);

    /**
     * 根据id修改分类
     */
    @AutoFill(value = OperationType.UPDATE)
    void update(Category category);

}
```

这里的两个注解分别表示：

```java
@AutoFill(value = OperationType.INSERT)
```

`insert()` 执行时自动填充四个公共字段。

```java
@AutoFill(value = OperationType.UPDATE)
```

`update()` 执行时自动填充修改时间和修改人。

EmployeeMapper 也需要进行同样的处理。

---

# 14. 改造后的 Service

完成自动填充之后，Service 中原本手动设置公共字段的代码可以删除。

例如不再需要：

```java
employee.setCreateTime(LocalDateTime.now());
employee.setUpdateTime(LocalDateTime.now());
employee.setCreateUser(BaseContext.getCurrentId());
employee.setUpdateUser(BaseContext.getCurrentId());
```

Service 只负责业务逻辑，而公共字段统一交给切面处理。

这实现了：

> **业务逻辑与公共逻辑的分离。**

---

# 15. 这个案例中的核心设计

这个案例并不是单纯为了实现“自动设置四个字段”，更重要的是理解几个技术之间是如何配合的。

|技术|作用|
|---|---|
|枚举 `OperationType`|区分 INSERT 和 UPDATE|
|`@AutoFill`|标记需要自动填充的 Mapper 方法|
|AOP|统一拦截目标方法|
|`JoinPoint`|获取当前被拦截的方法及其参数|
|反射|动态调用实体对象的 setter|
|`BaseContext`|获取当前登录用户 ID|

可以把整个设计概括为：

> **注解负责声明，AOP 负责拦截，枚举负责区分操作类型，反射负责动态赋值。**

---

# 16. 知识依赖

学习这个案例之前，需要掌握：

### 基础知识

- Java 枚举
- Java 注解
- Java 反射
- Spring Bean
- Spring AOP 基础

### 进一步理解

- 自定义注解与 AOP 的组合使用
- 根据注解信息改变程序行为
- 反射实现对象的动态操作
- 公共逻辑与业务逻辑解耦

---

# 17. 最终总结

### 问题

多个业务模块都需要重复设置：

```text
createTime
createUser
updateTime
updateUser
```

导致业务代码中存在大量重复的公共逻辑。

### 解决方案

使用：

> **自定义注解 + AOP + 反射**

将公共字段处理逻辑统一到切面中。

### 核心过程

> Mapper 方法使用 `@AutoFill` 声明操作类型，AOP 在方法执行前读取注解和方法参数，根据 `INSERT` 或 `UPDATE` 规则，通过反射为实体对象的公共字段赋值。

### 最重要的设计思想

> **不要让每个业务方法重复处理相同的公共逻辑，而应该识别出这类横切关注点，并通过 AOP 等机制统一处理。**