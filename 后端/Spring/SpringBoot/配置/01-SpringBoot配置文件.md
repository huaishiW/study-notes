# 1. Spring Boot 配置文件的作用

Spring Boot 项目中，经常需要配置一些程序运行所需的信息，例如：

- 数据库连接地址。
    
- 数据库用户名。
    
- 数据库密码。
    
- 数据库驱动。
    
- MyBatis 配置。
    
- 其他框架或应用程序配置。
    

这些信息通常不会直接写死在 Java 代码中，而是统一放在配置文件中。

这样可以做到：

> 将程序配置与 Java 业务代码分离，便于修改和维护。

例如数据库连接信息可以统一放在 Spring Boot 配置文件中，而不是直接写在：

```java
DriverManager.getConnection(...);
```

这样的 Java 代码里。

---

# 2. Spring Boot 常见配置文件

Spring Boot 支持多种配置文件格式。

当前主要使用两类：

- `application.properties`
    
- `application.yml`
    

另外：

```text
application.yaml
```

与：

```text
application.yml
```

属于同一种 YAML 配置格式，只是文件后缀不同。

也就是说：

```text
application.yml
application.yaml
```

配置内容的写法是相同的。

---

# 3. application.properties

`application.properties` 使用：

```text
key=value
```

的形式配置属性。

例如数据库连接：

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/web
spring.datasource.username=root
spring.datasource.password=1234
```

每一个配置项都是一个完整的：

```text
key=value
```

结构。

例如：

```properties
spring.datasource.username=root
```

其中：

```text
spring.datasource.username
```

是配置项名称。

```text
root
```

是配置值。

---

# 4. properties 配置方式的问题

当配置数量比较少时：

```properties
spring.datasource.url=...
spring.datasource.username=...
spring.datasource.password=...
```

这种形式比较简单。

但是当项目中的配置越来越多时，`properties` 的问题就会逐渐明显。

例如：

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/web
spring.datasource.username=root
spring.datasource.password=1234

mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

可以发现：

```text
spring.datasource
```

以及：

```text
mybatis.configuration
```

这样的前缀会被不断重复。

配置较多时，会导致：

- 配置文件比较臃肿。
    
- 层级关系不够直观。
    
- 相同前缀不断重复。
    

因此 Spring Boot 还支持使用 YAML 格式来组织配置。

---

# 5. application.yml

YAML 配置使用：

> 缩进表示层级关系。

例如数据库配置：

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/web
    username: root
    password: 1234
```

这里可以很清楚地看到：

- `datasource` 属于 `spring`。
    
- `url`、`username`、`password` 等属于 `datasource`。
    

相比 properties 中：

```properties
spring.datasource.url=...
spring.datasource.username=...
spring.datasource.password=...
```

YAML 的层级关系更加明显。

因此在配置较多时，YAML 的可读性通常更好。

---

# 6. properties 与 yml 的对应关系

例如下面的 properties：

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/web
spring.datasource.username=root
spring.datasource.password=1234
```

改写成 YAML：

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/web
    username: root
    password: 1234
```

二者表达的配置含义是相同的。

区别主要在配置的组织方式。

properties 使用：

```text
.
```

来体现层级：

```text
spring.datasource.username
```

YAML 使用：

> 缩进

来体现层级：

```yaml
spring:
  datasource:
    username: root
```

---

# 7. YAML 的核心特点

YAML 的核心特点是：

> 通过缩进组织数据层级。

例如：

```yaml
spring:
  datasource:
    username: root
```

可以理解为：

- 第一层：`spring`
    
- 第二层：`datasource`
    
- 第三层：`username`
    

因此 YAML 中最重要的是：

> 层级和缩进。

而不是像 properties 一样重复写完整配置路径。

---

# 8. YAML 基本语法

使用 YAML 时，需要遵守几个基本规则。

## 8.1 大小写敏感

YAML 中配置名称区分大小写。

例如：

```yaml
username: root
```

和：

```yaml
Username: root
```

并不是同一个配置项。

因此配置名称需要保持正确的大小写。

---

# 9. 冒号后需要空格

YAML 中配置通常写成：

```yaml
key: value
```

例如：

```yaml
username: root
```

需要注意：

> `:` 后面需要有空格。

正确：

```yaml
username: root
```

不要写成：

```yaml
username:root
```

---

# 10. 使用缩进表示层级

YAML 使用缩进表示配置项之间的层级关系。

例如：

```yaml
spring:
  datasource:
    username: root
    password: 1234
```

这里：

```text
username
password
```

属于：

```text
datasource
```

而：

```text
datasource
```

又属于：

```text
spring
```

所以缩进实际上表达的是：

> 谁属于谁。

---

# 11. 同一层级必须对齐

例如：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/web
    username: root
    password: 1234
```

这里：

```text
url
username
password
```

属于同一个层级，因此必须左侧对齐。

如果缩进关系不一致，配置结构就可能发生变化。

所以 YAML 中：

> 缩进代表结构，相同层级必须保持对齐。

---

# 12. 缩进不能使用 Tab

YAML 的缩进需要使用：

> 空格。

不应该直接使用 Tab 字符进行层级缩进。

在 IDEA 中，编辑器通常会帮助将 Tab 转换为空格。

至于具体缩进几个空格并不是最重要的。

重点是：

> 相同层级保持一致。

---

# 13. YAML 注释

YAML 使用：

```text
#
```

表示注释。

例如：

```yaml
# 数据库连接配置
spring:
  datasource:
    username: root
```

从：

```text
#
```

开始到当前行结束的内容会被作为注释处理。

例如：

```yaml
username: root # 数据库用户名
```

其中：

```text
# 数据库用户名
```

属于注释。

---

# 14. YAML 中定义对象

YAML 可以表示一个对象或 Map 集合。

例如：

```yaml
user:
  name: zhangsan
  age: 18
  password: 123456
```

这里可以理解为存在一个：

```text
user
```

对象。

它内部包含三个属性：

- `name`
    
- `age`
    
- `password`
    

也就是说：

```text
user
```

下面的配置通过缩进组成一个整体。

---

# 15. YAML 中定义 Map

对象和 Map 在 YAML 中的基本表现形式相似。

例如：

```yaml
user:
  name: zhangsan
  age: 18
```

可以理解为：

```text
user
```

对应一个包含：

```text
name
age
```

两个键值对的数据结构。

核心写法仍然是：

```yaml
key:
  childKey: value
```

---

# 16. YAML 中定义数组或集合

YAML 还可以表示：

- 数组
    
- List
    
- Set
    

例如：

```yaml
hobby:
  - java
  - game
  - sport
```

这里：

```text
hobby
```

包含三个元素：

- `java`
    
- `game`
    
- `sport`
    

集合中的每一个元素前面使用：

```text
-
```

表示。

因此可以记住：

> YAML 中集合元素通常使用 `-` 表示。

---

# 17. YAML 对象和集合的区别

对象通常写成：

```yaml
user:
  name: zhangsan
  age: 18
```

它表示：

> 一个对象下面包含多个不同属性。

集合通常写成：

```yaml
hobby:
  - java
  - game
  - sport
```

它表示：

> 一个配置项下面包含多个并列元素。

所以从形式上可以区分：

对象：

```yaml
key:
  childKey: value
```

集合：

```yaml
key:
  - value1
  - value2
```

---

# 18. 以 0 开头的值

YAML 中，如果某个配置值以：

```text
0
```

开头，需要特别注意。

当前内容中指出，以 `0` 开头的值应使用引号包裹，例如：

```yaml
code: '012345'
```

这样可以明确告诉 YAML：

> 这是一个普通的字符串值。

因此，对于手机号、编号等可能以 `0` 开头且本质上不是数学数值的数据，可以使用字符串形式保存。

---

# 19. 数据库配置从 properties 改写为 yml

假设原来的配置文件为：

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/web01
spring.datasource.username=root
spring.datasource.password=root@1234

mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

改写为：

```yaml
# 数据源配置
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/web01
    username: root
    password: root@1234

# MyBatis 配置
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

两种配置表达的是相同的信息。

区别只是：

> properties 使用完整 key，而 YAML 通过缩进把公共前缀提取出来。

---

# 20. 数据源配置的层级

例如：

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/web01
    username: root
    password: root@1234
```

可以逐层理解。

第一层：

```text
spring
```

第二层：

```text
datasource
```

第三层：

- `driver-class-name`
    
- `url`
    
- `username`
    
- `password`
    

因此原来的：

```properties
spring.datasource.username=root
```

实际上就是：

```text
spring
  → datasource
      → username
```

YAML 只是把这种层级直接表现出来了。

---

# 21. MyBatis 配置的层级

例如：

```yaml
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

对应 properties：

```properties
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

可以逐层拆分为：

第一层：

```text
mybatis
```

第二层：

```text
configuration
```

第三层：

```text
log-impl
```

因此可以理解：

> properties 中的 `.`，在 YAML 中通常体现为层级缩进。

---

# 22. 如何把 properties 转换成 YAML

把 properties 转换成 YAML 时，可以按以下方式思考。

假设：

```properties
spring.datasource.username=root
```

先按照：

```text
.
```

拆分：

```text
spring
datasource
username
```

然后按照层级改写：

```yaml
spring:
  datasource:
    username: root
```

再例如：

```properties
mybatis.configuration.log-impl=xxx
```

按照 `.` 拆分后：

```text
mybatis
configuration
log-impl
```

YAML：

```yaml
mybatis:
  configuration:
    log-impl: xxx
```

因此转换时不需要死记格式，只需要理解：

> properties 的路径结构，转换成 YAML 后就是缩进层级结构。

---

# 23. 公共前缀只需要写一次

properties：

```properties
spring.datasource.url=...
spring.datasource.username=...
spring.datasource.password=...
```

存在大量：

```text
spring.datasource
```

重复。

YAML 可以写成：

```yaml
spring:
  datasource:
    url: ...
    username: ...
    password: ...
```

公共的：

```text
spring
datasource
```

都只需要写一次。

所以当配置项较多、层级较深时，YAML 的结构通常更加清晰。

---

# 24. 修改配置文件时需要注意什么

从：

```text
application.properties
```

切换到：

```text
application.yml
```

时，需要确保程序最终加载的是新的配置文件。

示例操作中，将原来的：

```text
application.properties
```

修改为其他名称，使 Spring Boot 不再加载它，然后创建：

```text
application.yml
```

并写入新的 YAML 配置。

核心目的并不是一定要加下划线，而是：

> 避免旧配置继续被加载，从而确保使用新的配置内容。

---

# 25. properties 与 YAML 如何选择

在当前学习阶段，可以这样理解。

## 25.1 properties

特点：

- 使用 `key=value`。
    
- 写法直观。
    
- 配置较少时比较简单。
    
- 配置较多时公共前缀重复较多。
    
- 层级结构不够明显。
    

---

## 25.2 YAML

特点：

- 使用 `key: value`。
    
- 使用缩进表示层级。
    
- 公共前缀无需不断重复。
    
- 配置结构更加直观。
    
- 配置项较多时更加清晰。
    

因此当前项目更推荐使用：

```text
application.yml
```

来组织配置。

---

# 26. 常见错误

## 26.1 冒号后没有空格

错误：

```yaml
username:root
```

正确：

```yaml
username: root
```

---

## 26.2 层级缩进错误

正确：

```yaml
spring:
  datasource:
    username: root
    password: 1234
```

如果：

```text
username
password
```

属于同一层级，就应该保持相同缩进。

---

## 26.3 使用 Tab 控制层级

YAML 中应该使用空格缩进。

不要依赖真正的 Tab 字符表示层级。

---

## 26.4 同一层级没有对齐

例如：

```yaml
spring:
  datasource:
    username: root
     password: 1234
```

这里：

```text
username
password
```

没有保持相同缩进，就不能正确表达原本希望的层级关系。

---

## 26.5 把层级关系理解成普通格式美化

YAML 中的缩进不是为了让代码更好看。

它本身就是：

> 配置结构的一部分。

因此缩进发生变化，配置的层级含义也可能随之变化。

---

# 27. 学习重点

## 27.1 基础

需要掌握：

- Spring Boot 配置文件有什么作用。
    
- `application.properties` 的基本形式。
    
- `application.yml` 的基本形式。
    
- `application.yml` 和 `application.yaml` 的关系。
    
- properties 与 YAML 的主要区别。
    

## 27.2 YAML 语法

需要掌握：

- YAML 大小写敏感。
    
- `:` 后需要空格。
    
- 使用空格缩进表示层级。
    
- 相同层级保持左侧对齐。
    
- `#` 表示注释。
    
- 如何表示对象或 Map。
    
- 如何表示数组、List 或 Set。
    

## 27.3 重点

需要重点理解：

- YAML 中缩进为什么重要。
    
- properties 中的配置层级如何转换成 YAML。
    
- 为什么 YAML 可以减少公共配置前缀的重复。
    
- 为什么配置较多时 YAML 的结构更加直观。
    

---

> [!tip] 一句话总结
> 
> Spring Boot 可以使用 `application.properties` 或 `application.yml` 管理程序配置；properties 使用 `key=value` 表示完整配置路径，而 YAML 使用缩进表示层级关系，能够减少公共前缀重复，使大量配置的结构更加清晰。