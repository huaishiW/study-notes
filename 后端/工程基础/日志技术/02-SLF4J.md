# 1. SLF4J 是什么

SLF4J 的全称是：

```text
Simple Logging Facade for Java
```

中文通常称为：

> 简单日志门面。

SLF4J 提供了一套统一的日志操作接口和抽象，使 Java 程序可以使用统一方式记录日志，而不必直接依赖某一种具体日志实现。

因此可以将 SLF4J 理解为：

> Java 程序记录日志时使用的一套统一入口。

当前内容中，SLF4J 的主要作用是允许应用程序使用不同的底层日志框架。

---

# 2. SLF4J 为什么叫“日志门面”

“门面”可以理解为：

> 对外提供统一的使用方式。

应用程序记录日志时，可以统一使用：

```java
log.info("查询部门列表");
```

或者：

```java
log.debug("开始计算...");
```

Java 代码主要面对的是统一日志 API。

至于日志最终：

- 如何输出。
    
- 输出到哪里。
    
- 使用什么格式。
    
- 是否输出。
    

则由底层日志实现和配置决定。

因此 SLF4J 的核心职责不是直接定义日志文件保存规则，而是：

> 给应用程序提供统一的日志操作方式。

---

# 3. SLF4J 与 Logback 的区别

SLF4J 和 Logback 都与日志有关，但职责不同。

## 3.1 SLF4J

SLF4J 主要负责：

> 提供统一日志接口。

例如程序中使用：

```java
log.info(...);
log.debug(...);
log.warn(...);
log.error(...);
```

---

## 3.2 Logback

Logback 主要负责：

> 真正完成日志输出以及相关日志配置。

例如通过：

```text
logback.xml
```

控制：

- 日志输出格式。
    
- 日志输出位置。
    
- 日志级别。
    
- 日志开关。
    

所以可以简单区分：

|技术|主要作用|
|---|---|
|SLF4J|提供统一日志 API|
|Logback|具体完成日志输出|

---

# 4. Logger

使用 SLF4J 记录日志时，需要一个日志记录对象：

```java
Logger
```

例如：

```java
private static final Logger log =
        LoggerFactory.getLogger(LogTest.class);
```

这里的：

```java
log
```

就是后续真正用来调用：

```java
log.debug(...)
log.info(...)
```

等方法的对象。

因此可以理解为：

> `Logger` 是程序中直接用于记录日志的对象。

---

# 5. LoggerFactory

`Logger` 对象可以通过：

```java
LoggerFactory
```

获取。

例如：

```java
LoggerFactory.getLogger(LogTest.class);
```

完整写法：

```java
private static final Logger log =
        LoggerFactory.getLogger(LogTest.class);
```

其中：

```java
LogTest.class
```

表示当前日志对象与 `LogTest` 类关联。

这样在日志输出中，就可以包含日志来源类的信息。当前示例也展示了日志输出除了消息本身，还可以包含输出时间、线程名和日志所在类等信息。

---

# 6. Logger 的基本定义方式

一个常见写法是：

```java
public class LogTest {

    private static final Logger log =
            LoggerFactory.getLogger(LogTest.class);

}
```

这里通常将日志对象定义为：

```java
private static final
```

然后在整个类中统一通过：

```java
log
```

记录日志。

例如：

```java
log.info("程序开始执行");
```

---

# 7. 使用 Logger 记录日志

定义 Logger 后，可以直接调用对应方法。

例如：

```java
log.debug("开始计算...");
```

表示记录一条调试日志。

计算结束后：

```java
log.info("计算结果为: " + sum);
```

记录普通运行信息。

最后：

```java
log.debug("结束计算...");
```

示例：

```java
public class LogTest {

    private static final Logger log =
            LoggerFactory.getLogger(LogTest.class);

    @Test
    public void testLog() {

        log.debug("开始计算...");

        int sum = 0;

        int[] nums = {
            1, 5, 3, 2, 1, 4, 5,
            4, 6, 7, 4, 34, 2, 23
        };

        for (int i = 0; i < nums.length; i++) {
            sum += nums[i];
        }

        log.info("计算结果为: " + sum);

        log.debug("结束计算...");
    }
}
```

---

# 8. 常见日志记录方法

当前内容中涉及的常见日志方法包括：

```java
log.trace(...)
log.debug(...)
log.info(...)
log.warn(...)
log.error(...)
```

它们分别对应不同日志级别。

可以先简单理解为：

|方法|含义|
|---|---|
|`trace()`|记录程序运行轨迹|
|`debug()`|调试信息|
|`info()`|一般运行信息|
|`warn()`|警告信息|
|`error()`|错误信息|

日志级别本身会单独整理，所以这里重点只需要知道：

> `Logger` 通过不同方法记录不同类型的日志。

---

# 9. info() 的基本使用

`info()` 用于记录一般性的程序运行信息。

例如：

```java
log.info("查询部门列表");
```

可以表示：

> 当前程序正在执行查询部门列表功能。

再例如：

```java
log.info("新增部门");
```

用于记录一次新增操作。

实际业务代码中，`info` 是比较常用的日志级别之一。

---

# 10. debug() 的基本使用

`debug()` 主要用于：

> 记录程序调试过程中的信息。

例如：

```java
log.debug("开始计算...");
```

以及：

```java
log.debug("结束计算...");
```

可以帮助开发人员观察：

> 某段代码是否正常开始和结束执行。

当前内容也指出，`debug` 在实际开发中使用较多。

---

# 11. 日志中记录动态数据

实际开发中的日志往往不仅需要固定文字，还需要记录变量。

例如删除部门时，需要记录：

```java
id
```

如果使用字符串拼接，可以写：

```java
log.info("根据id删除部门, id=" + id);
```

但是示例中采用的是：

```java
log.info("根据id删除部门, id: {}", id);
```

这里：

```text
{}
```

作为日志消息中的参数位置。

后面的：

```java
id
```

会填入这个位置。

因此：

```java
log.info("根据id删除部门, id: {}", id);
```

可以理解为记录：

```text
根据id删除部门, id: 实际id值
```

---

# 12. {} 参数占位

日志中可以使用：

```text
{}
```

表示动态参数的位置。

例如：

```java
log.info("新增部门, dept: {}", dept);
```

这里：

```text
{}
```

对应：

```java
dept
```

运行时会将 `dept` 的实际内容加入日志。

例如：

```java
log.info("根据ID查询, id: {}", id);
```

其中动态值就是：

```java
id
```

这种方式可以让固定日志文本和动态数据分开表达。

---

# 13. 业务代码中的日志示例

例如一个部门查询接口：

```java
@GetMapping
public Result list() {

    log.info("查询部门列表");

    List<Dept> deptList = deptService.findAll();

    return Result.success(deptList);
}
```

这里日志记录在业务执行之前：

```java
log.info("查询部门列表");
```

可以帮助开发人员知道：

> 当前接口已经被调用，并开始执行查询操作。

---

# 14. 删除操作中的日志

例如：

```java
@DeleteMapping
public Result delete(Integer id) {

    log.info("根据id删除部门, id: {}", id);

    deptService.deleteById(id);

    return Result.success();
}
```

相比只记录：

```java
log.info("删除部门");
```

这里同时记录：

```java
id
```

可以获得更加具体的运行信息。

因此日志不仅应该记录：

> 做了什么操作。

在需要时还可以记录：

> 操作涉及的关键参数。

---

# 15. 新增操作中的日志

例如：

```java
@PostMapping
public Result save(@RequestBody Dept dept) {

    log.info("新增部门, dept: {}", dept);

    deptService.save(dept);

    return Result.success();
}
```

这里记录了：

- 当前正在新增部门。
    
- 当前接收到的 `dept` 对象。
    

因此如果新增数据出现问题，可以通过日志查看当时接收到的数据内容。

---

# 16. @Slf4j

如果每个类都手动编写：

```java
private static final Logger log =
        LoggerFactory.getLogger(当前类.class);
```

会出现重复代码。

当前业务示例中使用了：

```java
@Slf4j
```

例如：

```java
@Slf4j
@RequestMapping("/depts")
@RestController
public class DeptController {

}
```

添加：

```java
@Slf4j
```

之后，就可以直接在类中使用：

```java
log.info(...)
```

记录日志。

---

# 17. @Slf4j 的作用

`@Slf4j` 可以理解为：

> 帮助当前类生成一个名为 `log` 的日志记录对象。

因此原本需要手动写：

```java
private static final Logger log =
        LoggerFactory.getLogger(DeptController.class);
```

使用：

```java
@Slf4j
```

后，就可以省略这段代码。

之后直接：

```java
log.info("查询部门列表");
```

即可。

---

# 18. 手动 Logger 与 @Slf4j

两种写法最终都是为了获得：

```java
log
```

日志对象。

## 18.1 手动创建

```java
private static final Logger log =
        LoggerFactory.getLogger(DeptController.class);
```

---

## 18.2 使用 @Slf4j

```java
@Slf4j
public class DeptController {

}
```

然后直接：

```java
log.info(...);
```

在实际业务示例中采用的是第二种方式。

---

# 19. @Slf4j 与 Lombok

`@Slf4j` 属于 Lombok 提供的注解。

它的作用主要是：

> 减少手动创建 Logger 对象的重复代码。

因此它和之前实体类中使用的：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
```

类似，都是通过 Lombok 减少一些固定代码的手动编写。

---

# 20. 日志输出不仅包含消息正文

例如代码：

```java
log.info("计算结果为: " + sum);
```

真正输出到控制台时，并不一定只有：

```text
计算结果为: ...
```

还可以包含：

- 日志输出时间。
    
- 当前线程名。
    
- 日志级别。
    
- 日志来源类。
    
- 日志正文。
    

当前示例明确展示了这些附加信息。

因此日志比普通：

```java
System.out.println(...)
```

提供的信息更加完整。

---

# 21. SLF4J 的基本使用模式

手动创建日志对象时，可以记住：

```java
private static final Logger log =
        LoggerFactory.getLogger(当前类.class);
```

然后根据需要记录日志：

```java
log.debug("调试信息");

log.info("普通信息");

log.warn("警告信息");

log.error("错误信息");
```

如果项目中使用 Lombok，则可以：

```java
@Slf4j
public class XxxController {

}
```

直接使用：

```java
log.info(...);
```

---

# 22. 固定文本与动态参数

日志大致可以分成两种形式。

## 22.1 只记录固定信息

例如：

```java
log.info("查询部门列表");
```

适合记录：

> 当前执行了什么操作。

---

## 22.2 同时记录动态数据

例如：

```java
log.info("根据id删除部门, id: {}", id);
```

或者：

```java
log.info("新增部门, dept: {}", dept);
```

适合记录：

> 当前执行了什么操作，以及操作时的重要数据是什么。

---

# 23. 日志应该记录什么

根据当前业务示例，可以看到日志通常会记录：

- 当前正在执行的业务操作。
    
- 重要方法参数。
    
- 关键对象数据。
    

例如：

```java
log.info("查询部门列表");
```

记录操作。

```java
log.info("根据id删除部门, id: {}", id);
```

记录操作和关键参数。

```java
log.info("新增部门, dept: {}", dept);
```

记录操作和对象数据。

这样程序出现问题时，就能够更方便地追踪执行过程。

---

# 24. SLF4J 的核心理解

学习 SLF4J 时，需要重点区分：

## 24.1 Logger

用于：

> 真正调用日志方法。

例如：

```java
log.info(...);
```

---

## 24.2 LoggerFactory

用于：

> 获取 Logger 对象。

例如：

```java
LoggerFactory.getLogger(LogTest.class);
```

---

## 24.3 @Slf4j

用于：

> 简化 Logger 对象的创建。

添加注解后可以直接使用：

```java
log
```

---

## 24.4 Logback

负责：

> 底层日志输出和日志配置。

因此这些概念不是相互替代关系，而是分别承担不同职责。

---

# 25. 学习重点

## 25.1 基础

需要掌握：

- SLF4J 是什么。
    
- 什么是日志门面。
    
- `Logger` 的作用。
    
- `LoggerFactory` 的作用。
    
- 如何使用 `log.info()`。
    
- 如何使用 `log.debug()`。
    
- `@Slf4j` 的作用。
    

## 25.2 重点

需要重点理解：

- SLF4J 为什么不是具体日志输出实现。
    
- SLF4J 与 Logback 的职责区别。
    
- `LoggerFactory.getLogger()` 为什么需要传入当前类。
    
- `@Slf4j` 为什么可以代替手动创建 Logger。
    
- 日志中的 `{}` 如何用于记录动态参数。
    

---

> [!tip] 一句话总结
> `SLF4J` 是 Java 的统一日志门面，程序通过 `Logger` 调用 `debug()`、`info()` 等方法记录日志，可以使用 `LoggerFactory` 手动创建日志对象，也可以通过 Lombok 的 `@Slf4j` 简化创建过程；日志中的 `{}` 可以用于传入动态参数，而真正的日志输出由 Logback 等底层日志框架完成。