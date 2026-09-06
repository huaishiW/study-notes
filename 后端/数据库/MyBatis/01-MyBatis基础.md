# 1. MyBatis 是什么

MyBatis 是一款优秀的**持久层框架**，主要用于简化 JDBC 开发。

使用原生 JDBC 操作数据库时，开发者通常需要自己完成很多重复工作，例如：

- 获取数据库连接。
- 创建 `PreparedStatement`。
- 设置 SQL 参数。
- 执行 SQL。
- 获取 `ResultSet`。
- 解析查询结果。
- 将查询结果封装成 Java 对象。
- 关闭数据库连接和其他资源。

而使用 MyBatis 后，大量重复的 JDBC 代码都可以由框架自动完成。

因此可以将 MyBatis 理解为：

> MyBatis 是建立在 JDBC 基础之上的持久层框架，它通过封装 JDBC 中重复、繁琐的数据库操作，提高数据库访问代码的开发效率。

例如，如果使用 JDBC 查询所有用户，通常需要编写较多数据库连接、SQL 执行和结果解析代码。

而使用 MyBatis 时，可以将数据库操作定义为一个 Mapper 方法：

```java
@Select("select * from user")
List<User> findAll();
```

业务代码只需要调用：

```java
List<User> userList = userMapper.findAll();
```

即可获取查询结果。

所以 MyBatis 的核心作用并不是改变数据库访问的底层机制，而是：

> 对 JDBC 进行封装，使开发者可以更加简单地完成数据库操作。

---

# 2. 为什么需要 MyBatis

JDBC 是 Java 操作关系型数据库最基础的技术。

它能够完成：

- 数据库连接。
- SQL 执行。
- 参数绑定。
- 查询结果获取。
- 数据库资源管理。

但是直接使用 JDBC 开发业务代码时，会存在一些比较明显的问题。

## 2.1 数据库连接信息容易硬编码

原生 JDBC 中可能直接编写：

```java
Connection conn = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/web",
    "root",
    "1234"
);
```

数据库的：

- URL
- 用户名
- 密码

直接写在 Java 代码中。

这样会导致数据库配置与程序代码耦合，不利于后续修改和维护。

使用 Spring Boot + MyBatis 后，这些数据库连接信息可以统一放在配置文件中管理。

例如：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/web
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=1234
```

这样数据库配置和 Java 代码就可以分离。

---

## 2.2 查询结果需要手动解析

原生 JDBC 查询数据库后，会得到：

```java
ResultSet rs
```

之后需要开发者手动读取数据库字段：

```java
while (rs.next()) {
    int id = rs.getInt("id");
    String username = rs.getString("username");
    String password = rs.getString("password");
    String name = rs.getString("name");
    int age = rs.getInt("age");
}
```

如果还需要封装成 Java 对象，则需要继续编写对象赋值代码。

MyBatis 可以自动完成查询结果与 Java 对象之间的映射和封装。

因此开发者通常不需要自己操作：

```java
ResultSet
```

来逐个读取数据库字段。

---

## 2.3 数据库连接需要频繁创建和销毁

使用原生 JDBC 时，一次数据库操作通常需要：

1. 获取 `Connection`。
2. 执行 SQL。
3. 关闭 `Connection`。

如果程序频繁访问数据库，就会不断创建和销毁数据库连接。

而创建数据库连接本身需要消耗一定的系统资源。

MyBatis 在实际项目中通常配合数据库连接池使用。

数据库连接可以被重复利用，从而避免频繁创建和销毁连接所带来的资源浪费。

---

# 3. 持久层是什么

MyBatis 被称为：

> 持久层框架。

因此理解 MyBatis 之前，需要先理解什么是**持久层**。

持久层也可以称为：

- 数据访问层。
- DAO 层。

其中 DAO 是：

```text
Data Access Object
```

即数据访问对象。

持久层的主要职责是：

> 负责程序与数据库之间的数据访问操作。

例如：

- 查询用户。
- 添加用户。
- 修改用户。
- 删除用户。
- 查询员工。
- 添加部门。

这些直接与数据库进行交互的操作，通常都属于持久层的职责。

---

# 4. 持久化是什么意思

“持久层”中的“持久”来源于**数据持久化**。

程序运行时，Java 对象通常存在于内存中。

例如：

```java
User user = new User();
user.setUsername("admin");
```

这个 `user` 对象存在于程序运行时的内存中。

如果程序停止运行，内存中的这些数据通常也会消失。

而数据库中的数据可以长期保存。

例如将用户数据写入：

```text
MySQL
```

之后，即使 Java 程序重新启动，这些数据依然存在。

因此可以简单理解为：

> 持久化就是将程序运行过程中的数据保存到可以长期存储的位置。

数据库就是应用程序中最常见的数据持久化方式之一。

---

# 5. 持久层主要负责什么

持久层主要负责与数据库进行交互。

典型操作包括：

## 5.1 查询数据

例如：

```sql
select * from user;
```

对应程序中的：

```java
List<User> findAll();
```

---

## 5.2 新增数据

例如：

```sql
insert into user(username, password)
values('admin', '123456');
```

对应程序中的：

```java
void insert(User user);
```

---

## 5.3 修改数据

例如：

```sql
update user
set username = 'admin'
where id = 1;
```

对应程序中的：

```java
void update(User user);
```

---

## 5.4 删除数据

例如：

```sql
delete from user
where id = 1;
```

对应程序中的：

```java
void deleteById(Integer id);
```

因此可以将持久层简单理解为：

> 应用程序中专门负责操作数据库的一层。

---

# 6. Mapper 是 MyBatis 中的持久层接口

使用 MyBatis 开发时，持久层接口通常被称为：

```text
Mapper
```

例如操作 `User` 用户数据时，可以创建：

```java
UserMapper
```

例如：

```java
@Mapper
public interface UserMapper {

    @Select("select * from user")
    List<User> findAll();
}
```

这里的：

```java
UserMapper
```

就是一个 MyBatis 持久层接口。

它负责定义：

> 应用程序需要执行哪些数据库操作。

例如：

```java
findAll()
```

表示查询所有用户。

随着后续功能增加，还可能出现：

```java
deleteById()
insert()
update()
findByUsernameAndPassword()
```

这些方法都属于数据库访问操作，因此统一定义在 Mapper 中。

---

# 7. 什么是框架

MyBatis 不仅是持久层技术，同时还是一个：

> 框架。

框架可以理解为：

> 一套已经提供了大量通用功能的软件基础代码。

框架就是一种：

> 半成品软件。

“半成品”并不是说框架功能不完整，而是表示：

> 框架已经完成了大量通用代码，开发者只需要在框架提供的基础上实现自己的业务功能。

例如数据库开发中，很多项目都会重复处理：

- 数据库连接。
- SQL 参数绑定。
- 查询结果解析。
- Java 对象映射。
- 数据库资源管理。

如果每个项目都重新编写这些代码，会产生大量重复工作。

MyBatis 将这些通用功能提前实现。

开发者只需要重点关注：

- SQL 写什么。
- Mapper 方法如何定义。
- 参数是什么。
- 查询结果需要封装成什么对象。

---

# 8. 使用框架有什么意义

框架的主要价值在于减少重复开发。

框架拥有以下几个特点：

- 可重用。
- 通用。
- 高效。
- 规范。
- 可扩展。

这些特点可以分别理解。

## 8.1 可重用

数据库访问中的很多底层代码，每个项目都可能使用。

框架将这些代码统一实现之后，可以在不同项目中重复使用。

开发者不需要每次重新编写。

---

## 8.2 通用

框架解决的是一类项目中的共同问题。

例如 MyBatis 解决的是：

> Java 应用程序访问关系型数据库时存在的大量通用问题。

因此 MyBatis 并不是只能用于某一个项目，而是可以用于大量 Java 项目。

---

## 8.3 提高开发效率

使用原生 JDBC 时，需要编写大量底层代码。

MyBatis 将这些重复代码封装后，开发者可以把更多精力放在：

- SQL。
- 数据模型。
- 业务功能。

---

## 8.4 规范开发方式

框架通常会规定一套推荐的开发结构。

例如 MyBatis 中通常使用：

```text
XxxMapper
```

作为持久层接口。

例如：

```java
UserMapper
EmpMapper
DeptMapper
```

这样不同开发人员编写的代码结构更加统一。

---

## 8.5 提高扩展性

框架已经建立好基础运行机制。

开发者可以在此基础上不断增加新的数据库操作，而不需要修改大量底层代码。

---

# 9. MyBatis 与 JDBC 的关系

理解 MyBatis 时，最重要的一点是：

> MyBatis 并没有替代 JDBC。

MyBatis 的底层仍然需要 JDBC 来真正完成数据库操作。

因此两者的关系可以理解为：

1. JDBC 提供 Java 操作关系型数据库的基础能力。
2. MyBatis 建立在 JDBC 之上。
3. MyBatis 对 JDBC 中重复、繁琐的操作进行了封装。
4. 开发者通过 MyBatis 编写数据库访问程序。
5. MyBatis 底层最终仍通过 JDBC 与数据库通信。

所以：

> JDBC 是基础，MyBatis 是建立在 JDBC 基础上的高级封装。

---

# 10. JDBC 与 MyBatis 的代码差异

使用 JDBC 查询所有用户时，通常需要：

```java
Connection conn = DriverManager.getConnection(...);

PreparedStatement pstmt =
    conn.prepareStatement("select * from user");

ResultSet rs = pstmt.executeQuery();

while (rs.next()) {
    User user = new User();

    user.setId(rs.getInt("id"));
    user.setUsername(rs.getString("username"));
    user.setPassword(rs.getString("password"));
    user.setName(rs.getString("name"));
    user.setAge(rs.getInt("age"));
}

rs.close();
pstmt.close();
conn.close();
```

可以看到，开发者需要处理很多底层细节。

使用 MyBatis 后，可以先定义 Mapper：

```java
@Mapper
public interface UserMapper {

    @Select("select * from user")
    List<User> findAll();
}
```

之后直接调用：

```java
List<User> userList = userMapper.findAll();
```

开发者不再需要自己编写：

- `Connection`
- `PreparedStatement`
- `ResultSet`
- 数据结果逐字段解析
- JDBC 资源释放

等大量重复代码。

---

# 11. MyBatis 对 JDBC 做了哪些封装

对比一下 JDBC 与 MyBatis ，可以将 MyBatis 的主要封装总结为三个方面。

## 11.1 数据库连接配置

JDBC 中数据库连接参数可能直接写在 Java 代码中：

```java
DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/web",
    "root",
    "1234"
);
```

MyBatis 配合 Spring Boot 后，可以将这些配置统一写在：

```text
application.properties
```

例如：

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/web
spring.datasource.username=root
spring.datasource.password=1234
```

这样数据库配置与 Java 代码实现分离。

---

## 11.2 查询结果自动映射

JDBC 中需要手动读取：

```java
rs.getInt("id");
rs.getString("username");
rs.getString("password");
```

然后再手动封装到 Java 对象。

MyBatis 可以帮助开发者完成：

> 数据库查询结果到 Java 对象的映射和封装。

例如 Mapper 返回：

```java
List<User>
```

MyBatis 可以将查询到的用户数据直接封装成多个 `User` 对象。

因此开发者通常不再直接操作 `ResultSet`。

---

## 11.3 数据库连接管理

JDBC 中开发者需要不断：

- 获取连接。
- 使用连接。
- 释放连接。

MyBatis 配合数据库连接池后，可以统一管理数据库连接。

数据库连接可以重复利用，而不是每次数据库操作都重新创建。

这样可以减少资源浪费并提高数据库访问效率。

---

# 12. MyBatis 主要帮助开发者关注什么

开发者在编写持久层程序时，通常更加关注两个方面。

## 12.1 数据库连接配置

例如：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/web
spring.datasource.username=root
spring.datasource.password=1234
```

告诉程序：

> 应该连接哪个数据库。

---

## 12.2 Mapper 接口和 SQL

例如：

```java
@Mapper
public interface UserMapper {

    @Select("select * from user")
    List<User> findAll();
}
```

告诉 MyBatis：

> 需要执行什么数据库操作。

因此可以简单理解为：

> MyBatis 将大量数据库访问的底层流程隐藏起来，让开发者更多地关注数据库配置和 SQL 本身。

---

# 13. MyBatis 的基本工作方式

假设有：

```java
@Mapper
public interface UserMapper {

    @Select("select * from user")
    List<User> findAll();
}
```

代码中调用：

```java
List<User> userList = userMapper.findAll();
```

整体执行过程可以理解为：

1. Java 程序调用 `UserMapper.findAll()`。
2. MyBatis 获取该方法对应的 SQL。
3. MyBatis 获取数据库连接。
4. MyBatis 底层通过 JDBC 执行 SQL。
5. 数据库执行 `select * from user`。
6. 数据库返回查询结果。
7. MyBatis 解析查询结果。
8. MyBatis 将每条用户数据封装成 `User` 对象。
9. 最终返回 `List<User>`。

开发者只需要：

```java
userMapper.findAll();
```

但框架内部帮助完成了大量 JDBC 操作。

---

# 14. 学习 MyBatis 时应该关注什么

MyBatis 内容很多，但刚开始学习时，不应该把重点放在记忆大量注解和配置上。

应该先理解几个核心问题。

## 14.1 MyBatis 是做什么的

需要理解：

> MyBatis 是用于简化 JDBC 数据库开发的持久层框架。

---

## 14.2 MyBatis 为什么属于持久层

因为 MyBatis 主要解决的是：

> Java 程序访问数据库的问题。

因此通常应用在 DAO / Mapper 这一层。

---

## 14.3 MyBatis 为什么叫框架

因为它提前提供了大量通用数据库访问功能。

开发者只需要在框架提供的基础上完成具体数据库操作。

---

## 14.4 MyBatis 与 JDBC 是什么关系

需要明确：

> MyBatis 并不是 JDBC 的替代品，而是对 JDBC 的进一步封装。

MyBatis 底层仍然通过 JDBC 操作数据库。

---

# 15. 学习重点

## 15.1 基础

需要掌握：

- MyBatis 是什么。
- 什么是持久层。
- 持久层主要负责什么。
- 什么是框架。
- Mapper 是什么。
- MyBatis 为什么可以简化数据库开发。

## 15.2 重点

需要重点理解：

- MyBatis 与 JDBC 的关系。
- JDBC 原生开发存在哪些重复工作。
- MyBatis 对 JDBC 做了哪些封装。
- 为什么使用 MyBatis 后不需要自己解析 `ResultSet`。
- 为什么 MyBatis 可以减少数据库连接管理代码。

---

> [!tip] 一句话总结
> 
> `MyBatis` 是建立在 JDBC 基础上的持久层框架，它通过封装数据库连接、SQL 执行、结果映射和资源管理等重复操作，大幅简化 Java 程序访问数据库的过程，使开发者能够更加专注于 Mapper、SQL 和具体的数据访问需求。