# 1. 什么是切入点表达式

切入点表达式用于描述：

> **哪些方法需要应用 AOP 通知。**

也就是说，它主要负责确定：

> 当前通知应该作用到项目中的哪些方法。

Spring AOP 中常见的切入点表达式形式包括：

```text
execution(...)
@annotation(...)
```

其中：

```text
execution(...)
```

是最常用的一种方式，它主要根据方法本身的签名信息进行匹配。

# 2. execution 的作用

`execution` 主要根据方法的相关信息进行匹配，例如：

- 返回值
    
- 包名
    
- 类名
    
- 方法名
    
- 方法参数
    
- 方法声明的异常
    

因此，`execution` 适合用于：

> 根据方法签名特征批量匹配一组方法。

# 3. execution 的基本语法

材料中给出的基本语法是：

```text
execution(访问修饰符? 返回值 包名.类名.?方法名(方法参数) throws 异常?)
```

其中带有：

```text
?
```

的部分表示可以省略。

也就是说：

- 访问修饰符可以省略
    
- 包名和类名可以省略
    
- `throws` 异常可以省略
    

而返回值、方法名和参数部分需要按照具体规则进行描述。

# 4. 一个完整示例

例如：

```java
@Before("execution(void com.huaishi.service.impl.DeptServiceImpl.delete(java.lang.Integer))")
```

这个表达式表示匹配：

```java
void com.huaishi.service.impl.DeptServiceImpl.delete(Integer)
```

这个具体方法。

# 5. execution 中的通配符

为了避免每次都写非常完整的方法描述，`execution` 支持通配符。

主要有：

```text
*
..
```

这两个符号。

# 6. * 通配符

`*` 表示：

> **单个独立的任意内容。**

它可以用于匹配：

- 任意返回值
    
- 任意单层包名
    
- 任意类名
    
- 任意方法名
    
- 任意类型的一个参数
    
- 名称中的一部分
    

例如：

```text
execution(* com.huaishi.service.impl.DeptServiceImpl.delete(java.lang.Integer))
```

这里：

```text
*
```

表示任意返回值类型。

# 7. .. 通配符

`..` 表示：

> **多个连续的任意内容。**

它主要有两种常见用途。

## 7.1 匹配任意层级包

例如：

```text
execution(* com..DeptServiceImpl.delete(java.lang.Integer))
```

表示 `com` 下面可以存在任意层级的包。

## 7.2 匹配任意数量参数

例如：

```text
execution(* com..*.*(..))
```

这里参数位置的：

```text
..
```

表示：

> 任意类型、任意数量的参数。

# 8. 访问修饰符可以省略

例如方法原本可能是：

```java
public void delete(Integer id)
```

在切入点表达式中，可以省略：

```text
public
```

直接写：

```text
execution(void com.huaishi.service.impl.DeptServiceImpl.delete(java.lang.Integer))
```

材料中明确指出，方法访问修饰符可以省略。

# 9. 使用 * 匹配任意返回值

例如：

```text
execution(* com.huaishi.service.impl.DeptServiceImpl.delete(java.lang.Integer))
```

这里的：

```text
*
```

表示：

> 不限制方法返回值类型。

无论方法返回：

```text
void
int
String
List
```

只要其他条件符合，就可以被匹配。

# 10. 使用 * 匹配包名

`*` 还可以匹配一层包名。

例如：

```text
execution(* com.huaishi.*.*.DeptServiceImpl.delete(java.lang.Integer))
```

这里每一个：

```text
*
```

只能匹配一层包。

因此：

```text
*
```

和：

```text
..
```

在包匹配上的含义不同。

`*`：

> 匹配一层。

`..`：

> 匹配任意层级。

# 11. 使用 * 匹配类名

例如：

```text
execution(* com..*.delete(java.lang.Integer))
```

这里：

```text
*
```

用于表示任意类。

因此，只要对应包范围中的某个类存在符合条件的 `delete(Integer)` 方法，就可能被匹配。

# 12. 使用 * 匹配方法名

例如：

```text
execution(* com..*.*(java.lang.Integer))
```

方法名位置使用：

```text
*
```

表示：

> 任意方法名。

也就是说，只限制：

- 方法所在的大致包范围
    
- 参数是一个 `Integer`
    

而不限制具体方法名。

# 13. 使用 * 匹配一个参数

例如：

```text
execution(* com.huaishi.service.impl.DeptServiceImpl.delete(*))
```

参数位置的：

```text
*
```

表示：

> 任意类型的一个参数。

这里仍然要求：

> 参数数量必须是一个。

# 14. 使用 .. 匹配任意参数

如果写：

```text
execution(* com..*.*(..))
```

参数位置的：

```text
..
```

表示：

> 任意个、任意类型的参数。

因此，它既可以匹配：

```java
list()
```

也可以匹配：

```java
delete(Integer id)
```

还可以匹配：

```java
update(Integer id, String name)
```

只要其他条件符合即可。

# 15. 方法名的一部分也可以使用 *

`*` 不仅可以完全代替方法名，还可以用于匹配方法名的一部分。

例如：

```text
execution(* com.huaishi.service.impl.DeptServiceImpl.find*(..))
```

表示匹配：

```text
DeptServiceImpl
```

中所有以：

```text
find
```

开头的方法。

例如：

```java
findAllDept()
findDeptById()
```

都可以被匹配。

# 16. 使用逻辑运算符组合表达式

如果一个表达式无法完整描述需求，可以使用逻辑运算符进行组合。

材料中提到：

```text
&&
||
!
```

分别表示：

```text
且
或
非
```

例如：

```text
execution(* com.huaishi.service.DeptService.list(..))
||
execution(* com.huaishi.service.DeptService.delete(..))
```

表示：

> `list()` 或 `delete()` 方法都可以匹配。

# 17. execution 表达式的书写建议

材料中给出了几个比较重要的书写建议。

## 17.1 方法命名尽量规范

业务方法命名应该尽量具有规律。

例如：

```java
findAllDept()
findDeptById()

updateDeptById()
updateDeptByMoreCondition()
```

这样可以通过：

```text
find*
```

快速匹配查询方法，通过：

```text
update*
```

快速匹配更新方法。

## 17.2 优先基于接口描述切入点

材料建议：

> 描述切入点方法时，通常基于接口，而不是直接基于实现类。

例如：

```text
execution(* com.huaishi.service.DeptService.*(..))
```

相比直接写：

```text
DeptServiceImpl
```

这种方式扩展性更好。

## 17.3 尽量缩小匹配范围

在满足业务需求的前提下，切入点表达式的范围应该尽量精确。

例如：

```text
execution(* com.huaishi.*.*.DeptServiceImpl.find*(..))
```

相比：

```text
execution(* com..*.*(..))
```

匹配范围更明确。

材料建议：

> 包名匹配时尽量不要滥用 `..`，可以使用 `*` 来限制具体层级。

# 18. * 与 .. 的区别

可以简单总结为：

|符号|含义|
|---|---|
|`*`|单个任意内容|
|`..`|多个连续的任意内容|

在包名中：

```text
*
```

匹配一层包。

```text
..
```

匹配任意层级包。

在参数中：

```text
*
```

匹配一个任意类型参数。

```text
..
```

匹配任意数量、任意类型参数。

# 19. execution 的适用场景

`execution` 最适合：

> 方法名称、包结构、类名或者参数存在一定规律的场景。

例如所有查询方法统一以：

```text
find
```

开头，那么可以通过：

```text
find*
```

直接匹配。

如果目标方法之间没有明显规律，那么继续使用 `execution` 可能会让表达式变得比较复杂。

这种情况下，材料后续引出了：

```text
@annotation(...)
```

这种基于注解匹配的方法。

# 20. 当前阶段需要掌握的核心内容

1. `execution` 是 Spring AOP 中最常用的切入点表达式之一。
    
2. `execution` 可以根据返回值、包名、类名、方法名和参数等信息匹配方法。
    
3. 访问修饰符、部分包和类信息以及 `throws` 可以省略。
    
4. `*` 表示单个任意内容。
    
5. `*` 可以匹配任意返回值、单层包、类名、方法名或一个参数。
    
6. `..` 可以匹配任意层级包。
    
7. 参数位置的 `..` 表示任意类型、任意数量参数。
    
8. `*` 可以用于匹配方法名的一部分，例如 `find*`。
    
9. 多个表达式可以通过 `&&`、`||`、`!` 进行组合。
    
10. 方法命名应尽量规范，方便通过表达式批量匹配。
    
11. 描述切入点时，材料建议尽量基于接口而不是实现类。
    
12. 在满足需求的前提下，应尽量缩小切入点匹配范围。
    

> [!tip] 一句话总结
> 
> `execution 切入点表达式通过返回值、包名、类名、方法名和参数等方法签名信息匹配目标方法，其中 * 用于匹配单个任意内容，.. 用于匹配任意层级包或任意数量参数。`