# 1. 为什么需要配置优先级

Spring Boot 支持通过多种方式配置应用程序。

常见的配置来源包括：

- `application.properties`
    
- `application.yml`
    
- `application.yaml`
    
- Java 系统属性
    
- 命令行参数
    

当不同配置来源中没有重复配置项时，它们可以正常共同使用。

但是如果多个地方同时配置了相同的属性，例如：

```text
server.port
```

Spring Boot 就需要决定：

> **最终应该使用哪一个配置值。**

这就是配置优先级需要解决的问题。

---

# 2. Spring Boot 配置文件之间的优先级

Spring Boot 支持三种常见的配置文件：

```text
application.properties
application.yml
application.yaml
```

当它们同时存在，并且配置了相同属性时，不同配置文件之间存在优先级。

## 2.1 properties 与 YAML 同时配置

例如：

`application.properties`：

```properties
server.port=8081
```

`application.yml`：

```yaml
server:
  port: 8082
```

如果两者同时配置 `server.port`，当前材料中的结果是：

```text
application.properties
优先于
application.yml
```

因此最终采用 `application.properties` 中的配置。

## 2.2 yml 与 yaml 同时配置

例如项目中同时存在：

```text
application.yml
application.yaml
```

并配置相同属性时：

```text
application.yml
优先于
application.yaml
```

## 2.3 三种配置文件的优先级

按照当前材料中的比较结果，从高到低为：

```text
application.properties
>
application.yml
>
application.yaml
```

也就是说，同一个属性同时存在于三个文件中时：

> 优先级高的配置会覆盖优先级低的配置。

虽然 Spring Boot 支持多种配置文件格式，但是实际开发中通常应该统一使用一种配置格式，避免多个配置文件同时维护相同属性而增加理解和维护成本。

---

# 3. Java 系统属性

除了配置文件以外，Spring Boot 还支持通过 Java 系统属性提供配置。

基本格式：

```text
-Dkey=value
```

例如配置 Web 服务端口：

```text
-Dserver.port=9000
```

运行 Jar 包时可以写成：

```bash
java -Dserver.port=9000 -jar app.jar
```

这里：

```text
-Dserver.port=9000
```

表示在启动 JVM 时设置：

```text
server.port=9000
```

Java 系统属性的优先级高于前面的 Spring Boot 配置文件。

因此，如果：

`application.properties`：

```properties
server.port=8080
```

同时启动程序时指定：

```text
-Dserver.port=9000
```

那么最终生效的是：

```text
9000
```

---

# 4. 命令行参数

Spring Boot 还支持通过命令行参数提供配置。

基本格式：

```text
--key=value
```

例如：

```text
--server.port=10010
```

运行 Jar 包时：

```bash
java -jar app.jar --server.port=10010
```

表示启动 Spring Boot 应用时，将：

```text
server.port
```

配置为：

```text
10010
```

命令行参数具有比 Java 系统属性更高的优先级。

例如同时执行：

```bash
java -Dserver.port=9000 -jar app.jar --server.port=10010
```

这里同时存在：

```text
Java 系统属性：
server.port=9000

命令行参数：
server.port=10010
```

最终生效的是：

```text
server.port=10010
```

因为命令行参数优先级更高。

---

# 5. 常见配置方式的完整优先级

当前阶段接触到的五种配置方式，按照优先级从高到低可以整理为：

|优先级|配置方式|示例|
|---|---|---|
|1|命令行参数|`--server.port=10010`|
|2|Java 系统属性|`-Dserver.port=9000`|
|3|`application.properties`|`server.port=8081`|
|4|`application.yml`|`server.port: 8082`|
|5|`application.yaml`|`server.port: 8083`|

因此可以记为：

命令行参数\>Java 系统属性\>application.properties\>application.yml\>application.yaml

这里所谓的“优先级高”，可以理解为：

> **当不同配置来源中出现相同配置项时，优先级更高的配置值最终生效。**

并不是说优先级低的配置文件完全不会被读取。

例如：

`application.yml`：

```yaml
spring:
  datasource:
    username: root
```

命令行参数：

```text
--server.port=10010
```

两者配置的是不同属性，因此并不存在覆盖关系，可以同时生效。

只有配置项相同时，配置优先级才会直接影响最终结果。

---

# 6. 外部配置的实际作用

Java 系统属性和命令行参数的重要价值在于：

> 可以在不修改项目内部配置文件的情况下，在程序启动时改变部分运行配置。

例如项目已经完成打包：

```text
app.jar
```

配置文件中默认端口：

```properties
server.port=8080
```

部署时如果希望临时使用：

```text
9000
```

可以直接：

```bash
java -Dserver.port=9000 -jar app.jar
```

也可以：

```bash
java -jar app.jar --server.port=9000
```

这样不需要重新修改：

```text
application.properties
application.yml
```

也不需要重新编译代码。

因此，配置优先级不仅仅是一个需要记忆的顺序，它还使应用能够：

- 保留项目中的默认配置
    
- 在不同运行环境中覆盖部分配置
    
- 在启动程序时动态调整运行参数
    

---

# 7. 当前阶段需要掌握的核心内容

1. Spring Boot 可以从多个配置来源读取属性。
    
2. 多个配置来源配置不同属性时，可以同时生效。
    
3. 只有相同配置项发生重复时，才需要根据配置优先级决定最终值。
    
4. `application.properties` 的优先级高于 `application.yml`。
    
5. `application.yml` 的优先级高于 `application.yaml`。
    
6. Java 系统属性使用 `-Dkey=value` 的形式配置。
    
7. 命令行参数使用 `--key=value` 的形式配置。
    
8. Java 系统属性的优先级高于 Spring Boot 配置文件。
    
9. 命令行参数的优先级高于 Java 系统属性。
    
10. 当前学习范围内的优先级可以记为：命令行参数 > Java 系统属性 > `application.properties` > `application.yml` > `application.yaml`。
    
11. 外部配置可以在不修改项目内部配置文件的情况下覆盖部分运行参数。
    
12. 实际项目中通常应统一使用一种主要配置文件格式，避免多个配置文件同时维护相同属性。
    

> [!tip] 一句话总结
> 
> `Spring Boot支持从配置文件、Java系统属性和命令行参数等多个来源读取配置，当同一属性重复出现时会按照优先级决定最终值，当前学习范围内可记为：命令行参数 > Java系统属性 > application.properties > application.yml > application.yaml。`