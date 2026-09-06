# 1. MyBatis 快速入门目标

MyBatis 快速入门阶段的核心目标是：

> 使用 MyBatis 查询数据库中的全部用户数据。

为了完成这个功能，需要准备几个基本组成部分：

1. 创建 Spring Boot 工程。
2. 引入 MyBatis、MySQL 驱动等依赖。
3. 准备数据库表。
4. 创建与数据库表对应的实体类。
5. 配置数据库连接信息。
6. 创建 Mapper 接口。
7. 使用 `@Mapper` 标记 Mapper 接口。
8. 使用 `@Select` 编写查询 SQL。
9. 通过测试类调用 Mapper 方法。

整个过程的重点不是记住项目创建步骤，而是理解：

> 一个最基础的 MyBatis 程序需要哪些组成部分，以及这些组成部分分别负责什么。

---

# 2. 创建 Spring Boot 工程

创建项目时需要导入：

- Spring Boot Web 启动依赖。
- MyBatis 起步依赖。
- MySQL 数据库驱动。
- Lombok。

Spring Boot 与 MyBatis 结合后，可以帮助我们完成很多自动配置工作。

例如：

- 创建和管理 MyBatis 相关对象。
- 根据数据库配置创建数据源。
- 将 Mapper 代理对象交给 Spring IOC 容器管理。

因此在后续代码中，我们可以直接通过 Spring 使用 Mapper 对象。

---

# 3. MyBatis 依赖

使用 MyBatis 时，需要在项目中引入对应依赖。

创建 Spring Boot 工程后，会自动在 `pom.xml` 中加入 MyBatis 相关依赖和 MySQL 驱动依赖。

其中 MyBatis 起步依赖的作用可以理解为：

> 将 MyBatis 与 Spring Boot 进行整合，使 MyBatis 可以方便地运行在 Spring Boot 项目中。

MySQL 驱动则负责：

> 让 Java 程序能够通过 JDBC 与 MySQL 数据库进行通信。

---

# 4. 为什么还需要 MySQL 驱动

虽然我们使用的是 MyBatis，但 MyBatis 底层仍然通过 JDBC 操作数据库。

因此仍然需要数据库厂商提供的 JDBC 驱动。

也就是说：

- MyBatis 负责封装 JDBC。

- JDBC 提供数据库访问规范。

- MySQL Driver 提供 JDBC 的具体实现。

- MySQL 数据库负责真正执行 SQL。

所以使用 MyBatis 并不意味着不再需要数据库驱动。

---

# 5. 准备数据库表

这里使用一个简单的 `user` 表。

示例表结构如下：

```sql
create table user(
    id int unsigned primary key auto_increment comment 'ID,主键',
    username varchar(20) comment '用户名',
    password varchar(32) comment '密码',
    name varchar(10) comment '姓名',
    age tinyint unsigned comment '年龄'
) comment '用户表';
```

并准备了几条测试数据：

```sql
insert into user(id, username, password, name, age)
values
(1, 'daqiao', '123456', '大乔', 22),
(2, 'xiaoqiao', '123456', '小乔', 18),
(3, 'diaochan', '123456', '貂蝉', 24),
(4, 'lvbu', '123456', '吕布', 28),
(5, 'zhaoyun', '12345678', '赵云', 27);
```

这个表是后续 MyBatis 查询的基础数据来源。

---

# 6. 创建实体类

数据库表中的数据最终需要在 Java 程序中表示。

因此需要创建实体类：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {

    private Integer id;

    private String username;

    private String password;

    private String name;

    private Integer age;
}
```

这里必须强调：

> 实体类属性名与数据库表字段名一一对应。

例如：

|数据库字段|Java 属性|
|---|---|
|`id`|`id`|
|`username`|`username`|
|`password`|`password`|
|`name`|`name`|
|`age`|`age`|

这样 MyBatis 在查询数据时，就可以将数据库中的字段值映射到对应的 Java 属性中。

---

# 7. 实体类的作用

实体类主要用于：

> 在 Java 程序中表示数据库中的一条记录。

例如数据库中存在一条用户记录：

```text
id = 1
username = daqiao
password = 123456
name = 大乔
age = 22
```

MyBatis 查询后，可以将这条数据封装成：

```java
User user;
```

其中：

```java
user.getId();
user.getUsername();
user.getPassword();
user.getName();
user.getAge();
```

分别对应数据库中的各个字段。

如果查询多条用户数据，则可以封装成：

```java
List<User>
```

---

# 8. Lombok 在实体类中的作用

在很多实体类上基本都会以下几个注解：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
```

这些注解来自 Lombok。

它们主要用于减少实体类中的重复代码。

例如：

```java
@Data
```

可以帮助生成常用方法，例如：

- Getter
    
- Setter
    
- `toString()`
    

而：

```java
@NoArgsConstructor
```

表示生成无参构造方法。

```java
@AllArgsConstructor
```

表示生成全参数构造方法。

在快速入门阶段，不需要把重点放在 Lombok 上，只需要知道：

> Lombok 用于简化实体类代码编写。

---

# 9. 配置数据库连接信息

创建实体类后，还需要告诉 Spring Boot：

> 应该连接哪个数据库，以及使用什么账号密码。

这里在 application.properties 中配置数据库连接信息：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/web
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=root@1234
```

这些配置都以：

```text
spring.datasource
```

开头。

---

# 10. spring.datasource.url

配置：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/web
```

用于指定数据库连接地址。

可以简单理解为：

> Java 程序需要连接哪个 MySQL 数据库。

其中：

```text
jdbc:mysql://localhost:3306/web
```

表示连接本机 MySQL 服务中的 `web` 数据库。

---

# 11. spring.datasource.driver-class-name

配置：

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

用于指定数据库驱动类。

这里使用的是：

```text
com.mysql.cj.jdbc.Driver
```

即 MySQL JDBC Driver。

这个驱动负责真正完成 Java 与 MySQL 之间的通信。

---

# 12. spring.datasource.username

配置：

```properties
spring.datasource.username=root
```

表示连接数据库时使用的用户名。

例如：

```text
root
```

---

# 13. spring.datasource.password

配置：

```properties
spring.datasource.password=root@1234
```

表示连接数据库时使用的密码。

因此数据库连接的四个核心信息可以概括为：

- 驱动。
    
- URL。
    
- 用户名。
    
- 密码。
    

---

# 14. 为什么要把数据库配置写在配置文件中

相比原生 JDBC：

```java
DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/web",
    "root",
    "1234"
);
```

MyBatis 配合 Spring Boot 后，把数据库连接信息统一写入：

```text
application.properties
```

这样做的好处是：

> 数据库配置与 Java 业务代码分离。

以后数据库地址、账号或者密码发生变化时，只需要修改配置文件，而不需要修改 Mapper 或业务代码。

---

# 15. Mapper 是什么

配置好数据库之后，需要编写 MyBatis 的持久层接口。

首先创建这个文件：

```java
UserMapper
```

作为用户数据的 Mapper 接口。

基本代码如下：

```java
@Mapper
public interface UserMapper {

    @Select("select * from user")
    List<User> findAll();
}
```

在 MyBatis 中，持久层接口通常写为：

```text
XxxMapper
```

例如：

```text
UserMapper
EmpMapper
DeptMapper
```

Mapper 主要负责：

> 定义数据库操作方法以及对应的 SQL。

---

# 16. Mapper 为什么是接口

`UserMapper` 定义成：

```java
public interface UserMapper
```

而不是普通类。

例如：

```java
@Mapper
public interface UserMapper {

    @Select("select * from user")
    List<User> findAll();
}
```

这里并没有自己编写：

```java
UserMapperImpl
```

这样的实现类。

原因是：

> MyBatis 在程序运行时会自动为 Mapper 接口生成实现对象。

这个实现对象本质上是一个代理对象。

开发者只需要：

- 定义 Mapper 接口。
    
- 定义方法。
    
- 指定 SQL。
    

真正执行 SQL 的实现逻辑由 MyBatis 自动生成。

---

# 17. @Mapper

`@Mapper` 用于标记：

> 当前接口是一个 MyBatis Mapper 接口。

例如：

```java
@Mapper
public interface UserMapper {

}
```

程序运行时，MyBatis 会为该接口生成代理对象。

> 在框架中会自动生成接口的实现类对象，也就是代理对象，并交给 Spring IOC 容器管理。

因此我们后面可以直接使用 Spring 的：

```java
@Autowired
```

将 `UserMapper` 注入到其他对象中。

---

# 18. Mapper 代理对象

当定义：

```java
@Mapper
public interface UserMapper {

    @Select("select * from user")
    List<User> findAll();
}
```

时，开发者只定义了接口，并没有写具体实现。

但是程序运行后仍然可以调用：

```java
userMapper.findAll();
```

原因就在于：

> MyBatis 自动创建了 `UserMapper` 的代理对象。

这个代理对象会根据 Mapper 方法上的 SQL 注解完成实际数据库访问。

所以：

```java
userMapper.findAll();
```

虽然表面上只是调用一个接口方法，但背后会触发：

1. MyBatis 获取对应 SQL。
    
2. 获取数据库连接。
    
3. 执行 SQL。
    
4. 获取查询结果。
    
5. 将结果映射成 `User`。
    
6. 返回 `List<User>`。
    

---

# 19. @Select

`@Select` 是 MyBatis 提供的查询注解。

它用于：

> 在 Mapper 方法上定义 `SELECT` 查询语句。

例如：

```java
@Select("select * from user")
List<User> findAll();
```

其中：

```java
@Select("select * from user")
```

表示调用：

```java
findAll();
```

时，需要执行：

```sql
select * from user;
```

> `@Select` 表示 select 查询，用于书写 select 查询语句。

---

# 20. Mapper 方法返回值

Mapper 中的方法：

```java
List<User> findAll();
```

返回：

```java
List<User>
```

因为：

```sql
select * from user;
```

可能查询出多条用户记录。

MyBatis 会将每一条记录封装成：

```java
User
```

对象。

然后将多个 `User` 对象组成：

```java
List<User>
```

最终返回。

因此：

```java
List<User> findAll();
```

表示：

> 查询多条用户数据，并将结果封装成一个用户集合。

---

# 21. 一个最基础的 Mapper

完整的 Mapper 可以写成：

```java
import com.huaishi.pojo.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

import java.util.List;

@Mapper
public interface UserMapper {

    /**
     * 查询全部用户
     */
    @Select("select * from user")
    List<User> findAll();
}
```

这个接口中主要包含三个核心部分。

## 21.1 @Mapper

```java
@Mapper
```

告诉 MyBatis：

> 这是一个 Mapper 接口。

---

## 21.2 @Select

```java
@Select("select * from user")
```

告诉 MyBatis：

> 调用下面的方法时需要执行这条查询 SQL。

---

## 21.3 Mapper 方法

```java
List<User> findAll();
```

定义：

> Java 程序应该通过什么方法调用这次数据库查询，以及查询结果应该返回什么类型。

---

# 22. 编写单元测试

Mapper 编写完成之后，可以通过 Spring Boot 测试类测试。

```java
@SpringBootTest
class SpringbootMybatisQuickstartApplicationTests {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testFindAll() {

        List<User> userList = userMapper.findAll();

        for (User user : userList) {
            System.out.println(user);
        }
    }
}
```

这里主要涉及：

- `@SpringBootTest`
    
- `@Autowired`
    
- Mapper 方法调用
    

三个部分。

---

# 23. @SpringBootTest

测试类上：

```java
@SpringBootTest
```

表示：

> 当前测试类需要加载 Spring Boot 环境。

运行测试时，Spring Boot 会启动相关 Spring 环境，包括 IOC 容器。

因此测试类中可以直接获取由 Spring 管理的 Bean。

---

# 24. @Autowired 注入 Mapper

由于 `UserMapper` 的代理对象已经交给 Spring IOC 容器管理，因此测试类中可以直接写：

```java
@Autowired
private UserMapper userMapper;
```

Spring 会从 IOC 容器中找到：

```java
UserMapper
```

对应的代理对象，并赋值给变量：

```java
userMapper
```

之后就可以直接调用：

```java
userMapper.findAll();
```

---

# 25. 调用 Mapper 方法

调用：

```java
List<User> userList = userMapper.findAll();
```

之后，MyBatis 会执行：

```sql
select * from user;
```

并将查询结果自动封装为：

```java
List<User>
```

因此后续可以直接遍历：

```java
for (User user : userList) {
    System.out.println(user);
}
```

开发者不需要自己操作：

```java
ResultSet
```

完成字段读取。

---

# 26. 快速入门程序的完整执行过程

一个最基础的 MyBatis 查询可以用以下流程理解。

第一步：

Spring Boot 根据：

```properties
spring.datasource.*
```

读取数据库连接配置。

第二步：

Spring Boot 建立数据库访问环境。

第三步：

MyBatis 识别：

```java
@Mapper
```

标记的 Mapper 接口。

第四步：

MyBatis 为 Mapper 接口生成代理对象。

第五步：

代理对象被放入 Spring IOC 容器。

第六步：

程序通过：

```java
@Autowired
```

获取 Mapper 对象。

第七步：

调用：

```java
userMapper.findAll();
```

第八步：

MyBatis 获取：

```java
@Select("select * from user")
```

中的 SQL。

第九步：

MyBatis 底层执行 SQL。

第十步：

数据库返回查询结果。

第十一步：

MyBatis 将数据库结果映射成：

```java
User
```

对象。

第十二步：

最终返回：

```java
List<User>
```

给调用者。

---

# 27. 快速入门中各部分的职责

整个快速入门程序中，不同部分有不同职责。

## 27.1 数据库表

负责：

> 真正存储用户数据。

---

## 27.2 User 实体类

负责：

> 在 Java 程序中表示用户数据。

---

## 27.3 application.properties

负责：

> 保存数据库连接信息。

---

## 27.4 UserMapper

负责：

> 定义需要执行的数据库操作。

---

## 27.5 @Mapper

负责：

> 标记 Mapper 接口，让 MyBatis 为接口生成代理对象。

---

## 27.6 @Select

负责：

> 为 Mapper 方法定义查询 SQL。

---

## 27.7 Spring IOC 容器

负责：

> 管理 MyBatis 创建的 Mapper 代理对象。

---

# 28. 一个最小可运行 MyBatis 查询

如果只关注核心代码，可以将 MyBatis 快速入门浓缩为三个部分。

## 28.1 数据库配置

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/web
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=root@1234
```

---

## 28.2 Mapper

```java
@Mapper
public interface UserMapper {

    @Select("select * from user")
    List<User> findAll();
}
```

---

## 28.3 调用 Mapper

```java
@Autowired
private UserMapper userMapper;

@Test
public void testFindAll() {

    List<User> users = userMapper.findAll();

    for (User user : users) {
        System.out.println(user);
    }
}
```

只要理解这三个部分，就已经掌握了 MyBatis 最基础的使用模式。

---

# 29. 测试类的包位置

特别强调：

> 测试类所在的包需要与 Spring Boot 引导类所在包相同。

这是快速入门阶段需要注意的项目结构问题。

如果包结构不正确，可能导致 Spring Boot 没有按照预期扫描到相关组件。

---

# 30. SQL 提示配置

在 Mapper 中编写：

```java
@Select("select * from user")
```

时，IDEA 默认可能无法识别：

- 表名。
    
- 字段名。
    
- SQL 语法上下文。
    

建议可以在 IDEA 中配置 MySQL 数据库连接。

配置完成之后，IDEA 可以为 SQL 提供：

- SQL 关键字提示。
    
- 表名提示。
    
- 字段名提示。
    
- SQL 错误检查。
    

但是需要注意：

> IDEA 中配置数据库连接只是为了增强 SQL 编写提示，不会影响项目实际运行。

即使不配置，MyBatis 程序仍然可以正常运行。

---

# 31. 配置 MyBatis SQL 日志

默认情况下，执行 Mapper 方法时，控制台不一定会直接显示 MyBatis 实际执行的 SQL。

这里提供了一个辅助配置：

```properties
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

加入：

```text
application.properties
```

之后，可以在控制台查看 MyBatis 执行 SQL 的日志。

这个配置在学习阶段比较有用，因为可以帮助观察：

> Mapper 方法最终到底执行了什么 SQL。

例如调用：

```java
userMapper.findAll();
```

时，可以通过控制台日志观察是否真正执行：

```sql
select * from user;
```

---

# 32. 快速入门阶段真正需要记住什么

这一部分虽然包含项目创建、配置和测试等多个步骤，但真正需要掌握的并不是操作流程本身，而是几个核心概念。

## 32.1 数据库配置

通过：

```properties
spring.datasource.*
```

配置数据库连接。

---

## 32.2 实体类

使用：

```java
User
```

表示数据库中的用户数据。

---

## 32.3 Mapper 接口

通过：

```java
UserMapper
```

定义数据库访问操作。

---

## 32.4 @Mapper

告诉 MyBatis：

> 这是需要由框架生成代理对象的 Mapper 接口。

---

## 32.5 @Select

告诉 MyBatis：

> 这个 Mapper 方法需要执行哪一条查询 SQL。

---

## 32.6 Mapper 方法

通过：

```java
List<User> findAll();
```

定义 Java 程序调用数据库操作的入口。

---

# 33. 学习重点

## 33.1 基础

需要掌握：

- MyBatis Spring Boot 项目需要哪些基本依赖。
    
- 为什么仍然需要 MySQL Driver。
    
- 数据源配置应该写在哪里。
    
- 实体类的作用。
    
- Mapper 接口的作用。
    
- `@Mapper` 的作用。
    
- `@Select` 的作用。
    

## 33.2 重点

需要重点理解：

- Mapper 为什么只定义接口而不需要自己写实现类。
    
- Mapper 代理对象是谁创建的。
    
- Mapper 代理对象为什么可以使用 `@Autowired` 注入。
    
- MyBatis 如何根据 Mapper 方法执行对应 SQL。
    
- 查询结果为什么可以直接封装成 `User` 或 `List<User>`。
    
- 一个最基础的 MyBatis 查询从方法调用到结果返回经历了什么过程。
    
---

> [!tip] 一句话总结
> 
> `MyBatis` 快速入门的核心是：通过 `spring.datasource` 配置数据库连接，使用实体类表示数据库数据，通过 `@Mapper` 定义持久层接口、通过 `@Select` 为方法指定查询 SQL，再由 MyBatis 自动生成 Mapper 代理对象、执行 SQL 并将查询结果映射成 Java 对象。