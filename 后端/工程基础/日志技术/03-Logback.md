# 1. Logback 是什么

Logback 是 Java 中常用的日志框架。

它主要负责：

> 真正完成日志的记录、格式化和输出。

在当前日志体系中，可以这样区分：

- SLF4J：提供统一日志操作接口。
    
- Logback：负责具体日志实现和输出。
    

程序中通常通过：

```java
log.info("查询部门列表");
```

记录日志。

而日志最终：

- 输出到哪里。
    
- 使用什么格式。
    
- 是否输出。
    
- 如何生成日志文件。
    

则可以由 Logback 的配置进行控制。

---

# 2. Spring Boot 中的 Logback

在 Spring Boot 项目中，Logback 相关依赖已经通过依赖关系提供。

因此一般不需要额外手动引入基础 Logback 依赖。

如果单独使用 Logback，对应依赖形式为：

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.4.11</version>
</dependency>
```

在 Spring Boot 项目中，重点通常不是手动添加这个依赖，而是：

> 如何使用和配置 Logback。

---

# 3. logback.xml

Logback 的配置文件通常叫：

```text
logback.xml
```

配置文件用于控制日志输出行为。

例如可以配置：

- 日志输出格式。
    
- 日志输出位置。
    
- 日志开关。
    
- 日志级别。
    
- 日志文件滚动规则。
    

因此可以将 `logback.xml` 理解为：

> Logback 的核心配置文件。

当前内容中明确指出，`logback.xml` 用于控制日志输出格式、位置和日志开关。

---

# 4. logback.xml 放在哪里

在 Spring Boot 项目中，`logback.xml` 通常放在：

```text
src/main/resources
```

目录下。

例如：

```text
src/
└── main/
    └── resources/
        └── logback.xml
```

这样应用程序启动时就可以加载日志配置。

---

# 5. Logback 配置文件基本结构

一个基础配置可以写成：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration>

    <!-- 日志输出配置 -->

</configuration>
```

最外层：

```xml
<configuration>
```

表示：

> 当前文件是 Logback 的配置文件。

具体日志输出方式则写在其中。

---

# 6. Appender 是什么

Logback 中一个非常重要的概念是：

```text
Appender
```

Appender 可以理解为：

> 日志输出目标。

也就是决定日志：

> 输出到哪里。

当前内容主要涉及两种输出位置：

1. 控制台。
    
2. 系统文件。
    

---

# 7. 控制台输出

如果希望日志显示在控制台中，可以配置：

```xml
<appender name="STDOUT"
          class="ch.qos.logback.core.ConsoleAppender">

</appender>
```

其中：

```text
ConsoleAppender
```

表示：

> 将日志输出到控制台。

而：

```text
STDOUT
```

是这个 Appender 的名称。

---

# 8. 控制台 Appender 完整配置

示例：

```xml
<!-- 控制台输出 -->
<appender name="STDOUT"
          class="ch.qos.logback.core.ConsoleAppender">

    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">

        <pattern>
            %d{yyyy-MM-dd HH:mm:ss.SSS}
            [%thread]
            %-5level
            %logger{50}
            -%msg%n
        </pattern>

    </encoder>

</appender>
```

这段配置主要完成两件事情：

1. 指定日志输出到控制台。
    
2. 指定日志显示格式。
    

---

# 9. Encoder

Appender 中：

```xml
<encoder>
```

用于：

> 定义日志输出时如何编码和格式化。

例如：

```xml
<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
```

内部通过：

```xml
<pattern>
```

指定日志格式。

---

# 10. Pattern 日志格式

示例日志格式：

```xml
<pattern>
    %d{yyyy-MM-dd HH:mm:ss.SSS}
    [%thread]
    %-5level
    %logger{50}
    -%msg%n
</pattern>
```

其中每一个符号都有不同作用。

---

# 11. %d

```text
%d
```

表示：

> 日志时间。

例如：

```xml
%d{yyyy-MM-dd HH:mm:ss.SSS}
```

可以按照指定格式输出时间。

例如：

```text
2026-09-06 13:30:20.123
```

---

# 12. %thread

```text
%thread
```

表示：

> 当前执行日志代码的线程名称。

例如：

```text
main
```

或者 Web 应用中的请求处理线程。

线程信息可以帮助判断：

> 当前日志由哪个线程产生。

---

# 13. %level

```text
%level
```

表示：

> 当前日志级别。

例如：

```text
INFO
DEBUG
WARN
ERROR
```

配置中的：

```text
%-5level
```

表示按照指定宽度显示日志级别。

---

# 14. %logger

```text
%logger
```

表示：

> 产生日志的 Logger，也可以帮助定位日志来自哪个类。

例如：

```text
com.huaishi.controller.DeptController
```

配置：

```text
%logger{50}
```

用于限制显示长度。

---

# 15. %msg

```text
%msg
```

表示：

> 程序实际记录的日志消息。

例如代码：

```java
log.info("查询部门列表");
```

那么 `%msg` 对应的就是：

```text
查询部门列表
```

---

# 16. %n

```text
%n
```

表示：

> 换行。

每一条日志输出完成后换到下一行。

---

# 17. 一条完整日志包含什么

如果配置：

```xml
<pattern>
    %d{yyyy-MM-dd HH:mm:ss.SSS}
    [%thread]
    %-5level
    %logger{50}
    -%msg%n
</pattern>
```

那么一条日志通常可以包含：

- 时间。
    
- 线程。
    
- 日志级别。
    
- Logger / 类信息。
    
- 日志正文。
    

这也是专业日志相比：

```java
System.out.println(...)
```

更有价值的地方之一。

当前示例中也明确展示了日志输出包含时间、线程名和具体类等附加信息。

---

# 18. 文件输出

除了输出到控制台之外，Logback 还可以将日志写入文件。

例如：

```xml
<appender name="FILE"
          class="ch.qos.logback.core.rolling.RollingFileAppender">

</appender>
```

这里使用的是：

```text
RollingFileAppender
```

表示：

> 支持滚动管理的文件日志输出。

---

# 19. 为什么日志要输出到文件

控制台日志适合开发和调试时直接查看。

但是程序正式运行后，如果关闭控制台，历史信息就不方便查看。

日志文件可以保存：

- 过去发生的请求。
    
- 程序运行状态。
    
- 错误信息。
    
- 异常信息。
    

因此文件日志更适合：

> 保存长期运行记录，方便后续排查问题。

---

# 20. RollingFileAppender

示例：

```xml
<appender name="FILE"
          class="ch.qos.logback.core.rolling.RollingFileAppender">

</appender>
```

普通文件如果持续写入，可能越来越大。

`RollingFileAppender` 可以配合滚动策略：

> 按照一定规则生成新的日志文件。

例如：

- 按日期。
    
- 按文件大小。
    

这样可以避免所有日志长期写在一个巨大文件中。

---

# 21. rollingPolicy

文件 Appender 中可以配置：

```xml
<rollingPolicy
    class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
```

表示：

> 根据时间和文件大小管理日志滚动。

也就是说，可以结合：

- 日期。
    
- 文件大小。
    

决定什么时候创建新的日志文件。

---

# 22. FileNamePattern

例如：

```xml
<FileNamePattern>
    D:/tlias-%d{yyyy-MM-dd}-%i.log
</FileNamePattern>
```

它用于定义：

> 日志文件生成时的文件名规则。

其中：

```text
%d{yyyy-MM-dd}
```

表示日期。

```text
%i
```

表示序号。

所以可能生成：

```text
tlias-2026-09-06-0.log
tlias-2026-09-06-1.log
```

这样的日志文件。

---

# 23. MaxHistory

配置：

```xml
<MaxHistory>30</MaxHistory>
```

表示：

> 最多保留一定数量的历史日志。

当前示例中配置的是：

```text
30
```

用于限制历史日志文件的保存范围。

---

# 24. maxFileSize

配置：

```xml
<maxFileSize>10MB</maxFileSize>
```

表示：

> 单个日志文件达到指定大小后触发滚动。

当前示例设置：

```text
10MB
```

也就是说，当日志文件超过该大小时，可以继续生成新的日志文件。

---

# 25. 文件日志完整配置

可以整理为：

```xml
<!-- 文件日志 -->
<appender name="FILE"
          class="ch.qos.logback.core.rolling.RollingFileAppender">

    <rollingPolicy
        class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">

        <FileNamePattern>
            D:/tlias-%d{yyyy-MM-dd}-%i.log
        </FileNamePattern>

        <MaxHistory>30</MaxHistory>

        <maxFileSize>10MB</maxFileSize>

    </rollingPolicy>

    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">

        <pattern>
            %d{yyyy-MM-dd HH:mm:ss.SSS}
            [%thread]
            %-5level
            %logger{50}
            -%msg%n
        </pattern>

    </encoder>

</appender>
```

这段配置完成：

- 文件输出。
    
- 日志滚动。
    
- 历史文件限制。
    
- 单文件大小限制。
    
- 日志输出格式。
    

---

# 26. root

定义完 Appender 后，还需要告诉 Logback：

> 哪些 Appender 真正启用。

可以通过：

```xml
<root>
```

进行配置。

例如：

```xml
<root level="ALL">

    <appender-ref ref="STDOUT"/>

    <appender-ref ref="FILE"/>

</root>
```

这里表示：

- 使用 `STDOUT` 输出到控制台。
    
- 使用 `FILE` 输出到文件。
    

---

# 27. appender-ref

配置：

```xml
<appender-ref ref="STDOUT"/>
```

表示：

> 启用名为 `STDOUT` 的 Appender。

例如前面定义：

```xml
<appender name="STDOUT" ...>
```

这里就通过：

```text
ref="STDOUT"
```

引用它。

同理：

```xml
<appender-ref ref="FILE"/>
```

就是引用文件 Appender。

---

# 28. 同时输出到控制台和文件

可以配置：

```xml
<root level="ALL">

    <appender-ref ref="STDOUT"/>

    <appender-ref ref="FILE"/>

</root>
```

这样一条日志产生之后：

> 可以同时显示在控制台，并保存到日志文件。

这体现了日志框架相比普通控制台打印的灵活性。

---

# 29. 日志开关

Logback 可以通过配置控制日志是否开启。

例如：

```xml
<root level="ALL">
```

表示开启日志输出。

而：

```xml
<root level="OFF">
```

可以关闭日志输出。

当前内容中将其概括为：

- `ALL`：开启日志。
    
- `OFF`：关闭日志。
    

---

# 30. 为什么日志开关比删除代码更好

如果使用：

```java
System.out.println(...)
```

不希望继续输出时，通常需要删除或注释代码。

而日志框架可以保留：

```java
log.info(...);
```

代码不动，只调整配置。

例如：

```xml
<root level="OFF">
```

即可关闭输出。

因此：

> 日志行为可以通过配置控制，而不需要修改业务代码。

---

# 31. Logback 配置的核心结构

可以把当前 Logback 配置理解为三个主要部分。

## 31.1 Appender

负责：

> 日志输出到哪里。

例如：

```text
ConsoleAppender
RollingFileAppender
```

---

## 31.2 Encoder / Pattern

负责：

> 日志按照什么格式输出。

例如：

```text
时间 + 线程 + 级别 + 类 + 消息
```

---

## 31.3 root

负责：

> 启用哪些日志输出目标，以及控制日志级别。

例如：

```xml
<root level="ALL">
```

---

# 32. 一个完整的 Logback 配置示例

```xml
<?xml version="1.0" encoding="UTF-8"?>

<configuration>

    <!-- 控制台输出 -->
    <appender name="STDOUT"
              class="ch.qos.logback.core.ConsoleAppender">

        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">

            <pattern>
                %d{yyyy-MM-dd HH:mm:ss.SSS}
                [%thread]
                %-5level
                %logger{50}
                -%msg%n
            </pattern>

        </encoder>

    </appender>

    <!-- 文件输出 -->
    <appender name="FILE"
              class="ch.qos.logback.core.rolling.RollingFileAppender">

        <rollingPolicy
            class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">

            <FileNamePattern>
                D:/tlias-%d{yyyy-MM-dd}-%i.log
            </FileNamePattern>

            <MaxHistory>30</MaxHistory>

            <maxFileSize>10MB</maxFileSize>

        </rollingPolicy>

        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">

            <pattern>
                %d{yyyy-MM-dd HH:mm:ss.SSS}
                [%thread]
                %-5level
                %logger{50}
                -%msg%n
            </pattern>

        </encoder>

    </appender>

    <!-- 日志输出 -->
    <root level="INFO">

        <appender-ref ref="STDOUT"/>

        <appender-ref ref="FILE"/>

    </root>

</configuration>
```

---

# 33. 配置执行过程

当代码执行：

```java
log.info("查询部门列表");
```

可以这样理解整个过程。

1. 程序通过 SLF4J 的 Logger 记录日志。

2. 底层 Logback 接收到日志。

3. Logback 根据：

```text
logback.xml
```

读取日志配置。

4. 根据日志级别判断是否需要输出。

5.  `<pattern>` 格式化日志。

6. 根据 Appender 输出到指定位置。

---

# 34. Logback 真正负责什么

学习到这里，可以把 Logback 的主要职责概括为：

1. 接收日志记录请求。
    
2. 判断日志是否应该输出。
    
3. 按照指定格式组织日志。
    
4. 将日志输出到控制台。
    
5. 将日志保存到文件。
    
6. 管理日志文件滚动。
    

因此 Logback 不仅仅是：

> “把日志打印出来”。

而是负责完整的日志输出管理。

---

# 35. 常见概念区分

## 35.1 Logger

Java 代码中记录日志的对象。

例如：

```java
log.info(...);
```

---

## 35.2 SLF4J

统一的日志门面。

---

## 35.3 Logback

具体日志实现。

---

## 35.4 logback.xml

Logback 配置文件。

---

## 35.5 Appender

决定日志输出位置。

---

## 35.6 Pattern

决定日志输出格式。

---

## 35.7 rollingPolicy

决定文件日志如何滚动管理。

---

# 36. 学习重点

## 36.1 基础

需要掌握：

- Logback 是什么。
    
- `logback.xml` 的作用。
    
- Appender 是什么。
    
- 如何输出日志到控制台。
    
- 如何输出日志到文件。
    
- `root` 的作用。
    

## 36.2 配置

需要掌握：

- `ConsoleAppender`
    
- `RollingFileAppender`
    
- `PatternLayoutEncoder`
    
- `FileNamePattern`
    
- `MaxHistory`
    
- `maxFileSize`
    
- `appender-ref`
    

## 36.3 重点

需要重点理解：

- SLF4J 和 Logback 的职责区别。
    
- Appender 为什么代表输出目标。
    
- Pattern 为什么可以控制日志格式。
    
- 为什么日志文件需要滚动。
    
- `root` 如何决定启用哪些日志输出位置。
    
- 为什么日志行为可以通过配置修改，而不用修改业务代码。
    

---

> [!tip] 一句话总结
> 
> `Logback` 是具体的日志实现框架，通过 `logback.xml` 可以配置日志格式、输出位置和日志开关；Appender 决定日志输出到控制台还是文件，Pattern 决定日志显示格式，而 `RollingFileAppender` 和滚动策略可以控制日志文件的生成、大小和历史保留。