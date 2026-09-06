# 1. JDBC 是什么

JDBC（Java Database Connectivity）是 Java 提供的一套用于操作关系型数据库的 API。

它解决的核心问题是：

> Java 程序如何以统一的方式访问不同的关系型数据库。

Java 程序可能需要访问不同类型的数据库，例如：

- MySQL
- Oracle
- SQL Server
- PostgreSQL

如果 Java 程序直接针对每一种数据库编写完全不同的代码，那么应用程序就会和具体数据库产生较强耦合。

JDBC 的作用，就是在 Java 程序和不同数据库之间提供一套统一的访问规范。

JDBC 的基本工作关系可以理解为：

1. Java 程序调用 JDBC API。
2. JDBC API 调用对应数据库厂商提供的数据库驱动。
3. 数据库驱动负责和具体数据库进行通信。
4. 数据库执行 SQL，并将结果返回给 Java 程序。

因此可以把 JDBC 理解为：

> JDBC 是 Java 程序访问关系型数据库的一套标准接口。

---

# 2. JDBC 的本质

JDBC 本身并不是一个具体的数据库实现，而是一套由 Java 定义的数据库访问规范和接口。

Java 官方负责定义 JDBC 中应该具备哪些接口，以及这些接口应该具有什么功能。

例如 JDBC 中常见的接口和对象包括：

- `Connection`
- `PreparedStatement`
- `ResultSet`

不同的数据库厂商则负责实现这些 JDBC 规范。

例如：

- MySQL 提供 MySQL JDBC Driver。
- Oracle 提供 Oracle JDBC Driver。
- PostgreSQL 提供 PostgreSQL JDBC Driver。

因此，JDBC 和数据库驱动并不是同一个概念。

JDBC 负责定义统一规范，而数据库驱动负责实现这套规范，并完成 Java 程序与具体数据库之间的通信。

可以理解为：

- JDBC 负责规定“数据库应该怎样被 Java 操作”。
- 数据库驱动负责实现“这些操作具体怎样在某种数据库中完成”。

---

# 3. 数据库驱动

数据库驱动是数据库厂商根据 JDBC 规范提供的具体实现。

当 Java 程序需要连接 MySQL 时，需要引入 MySQL 对应的 JDBC 驱动。

例如在 Maven 项目中可以引入 MySQL 驱动依赖：

```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

引入数据库驱动后，Java 程序就可以通过 JDBC API 操作对应数据库。

因此三者之间的关系是：

- JDBC：定义数据库访问规范。
- 数据库驱动：实现 JDBC 规范。
- 数据库：真正执行 SQL 和存储数据。

---

# 4. JDBC 在 Java 数据库访问体系中的位置

JDBC 属于 Java 数据库访问技术中比较基础的一层。

在实际 Java 项目中，开发者通常不会一直直接手写 JDBC 代码，而是使用 MyBatis、Hibernate、Spring Data JPA 等更高层框架。

这些框架的底层仍然需要通过 JDBC 与数据库进行交互。

例如使用 MyBatis 操作 MySQL 时，可以理解为：

1. Java 业务代码调用 MyBatis。
2. MyBatis 对数据库操作进行封装。
3. MyBatis 底层通过 JDBC 执行 SQL。
4. JDBC 通过 MySQL Driver 与 MySQL 数据库通信。
5. MySQL 执行 SQL，并返回结果。

因此：

> MyBatis 并不是 JDBC 的替代品，而是建立在 JDBC 之上的进一步封装。

学习 JDBC 的意义也不仅仅是为了以后直接手写 JDBC，而是为了理解：

- Java 程序是如何连接数据库的。
- SQL 是如何从 Java 程序发送到数据库的。
- 查询结果是如何返回到 Java 程序的。
- MyBatis 到底帮助我们封装了哪些重复操作。

---

# 5. JDBC 的基本执行流程

一次典型的 JDBC 数据库操作通常包括以下几个步骤：

1. 建立数据库连接。
2. 创建 SQL 执行对象。
3. 设置 SQL 参数。
4. 执行 SQL。
5. 获取并处理执行结果。
6. 释放数据库资源。

这些步骤分别对应 JDBC 中不同的对象和方法。

例如：

- `Connection`：表示数据库连接。
- `PreparedStatement`：负责准备并执行 SQL。
- `ResultSet`：保存查询返回的结果。
- `executeQuery()`：执行查询语句。
- `executeUpdate()`：执行增、删、改语句。

---

# 6. JDBC 基本查询示例

下面是一个使用 JDBC 查询用户信息的基本示例：

```java
Connection conn = DriverManager.getConnection(
        "jdbc:mysql://localhost:3306/web",
        "root",
        "1234"
);

PreparedStatement pstmt =
        conn.prepareStatement(
                "SELECT * FROM user WHERE username = ? AND password = ?"
        );

pstmt.setString(1, "daqiao");
pstmt.setString(2, "123456");

ResultSet rs = pstmt.executeQuery();

while (rs.next()) {
    int id = rs.getInt("id");
    String username = rs.getString("username");
    String name = rs.getString("name");
}

rs.close();
pstmt.close();
conn.close();
```

学习这段代码时，不需要先死记每一行代码，而应该先理解不同对象的职责。

---

# 7. Connection

`Connection` 表示 Java 程序与数据库之间建立的一次连接。

例如：

```java
Connection conn = DriverManager.getConnection(
        url,
        username,
        password
);
```

这里的 `Connection` 可以理解为 Java 程序与数据库之间建立的通信通道。

只有建立数据库连接之后，Java 程序才能继续执行 SQL。

后续创建 `PreparedStatement`、执行 SQL 等操作，都需要基于 `Connection` 完成。

---

# 8. PreparedStatement

`PreparedStatement` 用于创建和执行带参数的 SQL。

例如：

```java
PreparedStatement pstmt =
        conn.prepareStatement(
                "SELECT * FROM user WHERE username = ?"
        );
```

其中的 `?` 表示参数占位符。

之后再通过：

```java
pstmt.setString(1, "daqiao");
```

为 SQL 中的占位符设置具体参数。

因此，`PreparedStatement` 主要负责两件事：

1. 保存准备执行的 SQL。
2. 为 SQL 中的参数占位符设置具体值并执行 SQL。

`PreparedStatement` 还与预编译 SQL、SQL 注入防护等知识密切相关。

详细请看[02-PreparedStatement](02-PreparedStatement.md)

---

# 9. ResultSet

当 Java 程序执行查询 SQL 后，数据库返回的查询结果会被封装到 `ResultSet` 中。

例如：

```java
ResultSet rs = pstmt.executeQuery();
```

`ResultSet` 可以理解为数据库返回给 Java 程序的查询结果集合。

例如数据库返回多条用户数据时，需要逐行读取：

```java
while (rs.next()) {
    int id = rs.getInt("id");
    String username = rs.getString("username");
}
```

其中：

```java
rs.next();
```

用于将游标移动到下一条记录。

然后可以通过不同方法读取当前记录中的字段，例如：

```java
rs.getInt("id");
rs.getString("username");
```

详情请看[03-ResultSet](03-ResultSet.md)

---

# 10. executeQuery() 和 executeUpdate()

JDBC 中执行 SQL 时，通常根据 SQL 类型使用不同的方法。

## 10.1 executeQuery()

`executeQuery()` 主要用于执行查询语句。

例如：

```sql
SELECT * FROM user;
```

对应 Java 代码：

```java
ResultSet resultSet = pstmt.executeQuery();
```

执行后会返回一个 `ResultSet`。

因此：

> `executeQuery()` 主要用于查询数据，并返回查询结果集。

---

## 10.2 executeUpdate()

`executeUpdate()` 主要用于执行：

- `INSERT`
- `UPDATE`
- `DELETE`

例如：

```java
int rows = pstmt.executeUpdate();
```

返回值 `rows` 表示受到影响的数据行数。

例如返回：

```text
1
```

表示此次 SQL 操作影响了一条数据库记录。

因此：

> `executeUpdate()` 主要用于执行增、删、改操作，并返回受影响的记录数量。

---

# 11. JDBC 查询与增删改的区别

查询操作和增删改操作最主要的区别在于返回结果不同。

查询操作通常使用：

```java
executeQuery();
```

因为查询需要获取数据库返回的数据，所以返回：

```java
ResultSet
```

而增、删、改操作通常使用：

```java
executeUpdate();
```

因为这类操作主要关心修改了多少条数据，所以返回：

```java
int
```

用于表示受影响的记录数。

---

# 12. 原生 JDBC 存在的问题

虽然 JDBC 可以完成数据库操作，但是直接使用原生 JDBC 编写业务代码时，会存在较多重复和繁琐操作。

一次完整的 JDBC 数据库操作通常需要开发者手动完成：

1. 获取数据库连接。
2. 创建 SQL 执行对象。
3. 设置 SQL 参数。
4. 执行 SQL。
5. 获取执行结果。
6. 解析查询结果。
7. 将数据库数据封装成 Java 对象。
8. 关闭 `ResultSet`。
9. 关闭 `PreparedStatement`。
10. 关闭 `Connection`。

随着业务越来越复杂，这些重复代码会越来越多。

---

## 12.1 问题一：数据库配置容易硬编码

原生 JDBC 中可能会直接写：

```java
DriverManager.getConnection(
        "jdbc:mysql://localhost:3306/web",
        "root",
        "1234"
);
```

这里直接包含：

- 数据库地址
- 数据库名称
- 用户名
- 密码

如果这些信息直接写在 Java 代码中，就属于硬编码。

这样会导致数据库配置发生变化时，需要修改程序代码，不利于项目维护。

实际项目通常会将数据库配置放在配置文件中。

---

## 12.2 问题二：查询结果需要手动解析

使用 JDBC 查询数据时，需要手动从 `ResultSet` 中读取数据库字段。

例如：

```java
while (rs.next()) {
    int id = rs.getInt("id");
    String username = rs.getString("username");
    String name = rs.getString("name");
}
```

如果最终需要封装成 Java 对象，还需要手动完成：

```java
User user = new User();

user.setId(rs.getInt("id"));
user.setUsername(rs.getString("username"));
user.setName(rs.getString("name"));
```

也就是说，开发者需要手动完成数据库字段和 Java 对象属性之间的数据转换。

这也是 MyBatis 后来重点封装的一部分。

---

## 12.3 问题三：数据库连接需要手动管理

数据库连接属于比较重要的系统资源。

传统 JDBC 操作通常需要：

1. 创建数据库连接。
2. 使用数据库连接执行 SQL。
3. 操作完成后关闭数据库连接。

如果每次请求都不断创建和销毁数据库连接，会产生额外性能开销。

因此实际项目中通常会使用：

[[03-数据库连接池]]

来统一管理和复用数据库连接。

---

# 13. JDBC 与 MyBatis 的关系

JDBC 和 MyBatis 的关系是学习数据库持久层技术时非常重要的一组关系。

原生 JDBC 需要开发者自己完成很多操作，例如：

- 建立数据库连接。
- 创建 SQL 执行对象。
- 设置 SQL 参数。
- 执行 SQL。
- 处理 `ResultSet`。
- 将查询结果封装成 Java 对象。
- 管理数据库资源。

MyBatis 对其中大量重复操作进行了封装。

例如使用 JDBC 查询用户时，可能需要编写：

```java
Connection conn = ...;
PreparedStatement pstmt = ...;
ResultSet rs = ...;

while (rs.next()) {
    // 解析数据
}

rs.close();
pstmt.close();
conn.close();
```

而使用 MyBatis 后，业务代码中可能只需要：

```java
List<User> users = userMapper.findAll();
```

因此可以理解为：

> JDBC 提供 Java 访问数据库的底层标准能力，而 MyBatis 在 JDBC 基础上进一步封装了数据库访问过程，从而减少重复代码并提高开发效率。

---

# 14. JDBC 核心对象之间的关系

JDBC 中几个重要对象之间具有明确的职责关系。

## Connection

`Connection` 负责表示 Java 程序和数据库之间的连接。

## PreparedStatement

`PreparedStatement` 基于 `Connection` 创建，负责准备 SQL、绑定参数和执行 SQL。

## ResultSet

`ResultSet` 是查询 SQL 执行之后得到的结果集合。

因此，一次查询操作通常可以理解为：

1. 先通过 `Connection` 建立数据库连接。
2. 再通过 `Connection` 创建 `PreparedStatement`。
3. 使用 `PreparedStatement` 设置 SQL 参数。
4. 使用 `PreparedStatement` 执行 SQL。
5. 如果执行的是查询，则得到 `ResultSet`。
6. Java 程序通过 `ResultSet` 读取查询结果。
7. 最后释放相关数据库资源。

---

# 15. JDBC、数据库驱动和数据库之间的关系

这三个概念需要区分清楚。

## JDBC

JDBC 是 Java 定义的一套数据库访问规范。它规定了 Java 程序应该通过什么接口完成数据库操作。

## 数据库驱动

数据库驱动由具体数据库厂商提供。数据库驱动负责实现 JDBC 规范，并将 JDBC 调用转换成具体数据库能够理解的操作。

## 数据库

数据库负责真正执行 SQL、存储数据并返回结果。

因此整个过程可以理解为：

1. Java 程序调用 JDBC API。
2. JDBC API 调用数据库驱动。
3. 数据库驱动与数据库进行通信。
4. 数据库执行 SQL。
5. 数据库将执行结果返回给驱动。
6. 驱动再通过 JDBC 接口将结果返回给 Java 程序。

---

# 16. JDBC 为什么重要

虽然现代 Java 项目通常使用 MyBatis 等框架，但 JDBC 仍然非常重要。

因为 JDBC 是理解 Java 数据库访问机制的基础。

掌握 JDBC 后，可以更容易理解：

- MyBatis 为什么需要 Mapper。
- MyBatis 为什么可以自动完成结果映射。
- `#{}` 参数绑定是如何工作的。
- 数据库连接池为什么存在。
- SQL 最终是如何执行的。
- Java 对象与数据库数据之间是如何转换的。

因此，JDBC 更重要的价值是帮助我们理解持久层框架的底层原理。

---

# 17. 学习重点

## 基础知识

需要掌握：

- JDBC 是什么。
- JDBC 为什么是一套规范。
- JDBC 和数据库驱动有什么区别。
- Java 程序如何通过 JDBC 操作数据库。
- `Connection` 的作用。
- `PreparedStatement` 的作用。
- `ResultSet` 的作用。
- `executeQuery()` 和 `executeUpdate()` 的区别。

## 重点知识

需要进一步理解：

- 为什么推荐使用 `PreparedStatement`。
- SQL 参数是如何绑定的。
- `ResultSet` 是如何遍历的。
- JDBC 为什么需要手动释放资源。

## 后续进阶知识

之后可以继续学习：

- 预编译 SQL。
- SQL 注入。
- 数据库连接池。
- MyBatis 对 JDBC 的封装。
- MyBatis 参数绑定。
- MyBatis 结果映射。

---

> [!tip] 一句话总结
>JDBC 是 Java 操作关系型数据库的一套标准 API。Java 定义 JDBC 规范，数据库厂商通过数据库驱动实现这套规范，而 MyBatis 等持久层框架则在 JDBC 基础上进一步封装数据库访问过程，从而减少重复代码并提高开发效率。