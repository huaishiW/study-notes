# 1. PreparedStatement 是什么

`PreparedStatement` 是 JDBC 中用于执行 SQL 的对象。

它的一个重要特点是：

> 可以使用 `?` 作为 SQL 参数的占位符，再通过 Java 代码动态设置参数值。

例如：

```java
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM user WHERE username = ? AND password = ?"
);

pstmt.setString(1, "daqiao");
pstmt.setString(2, "123456");

ResultSet resultSet = pstmt.executeQuery();
```

在这段代码中：

- 第一个 `?` 表示 `username` 的参数。
- 第二个 `?` 表示 `password` 的参数。
- `setString()` 用于给对应的占位符设置参数值

这种 SQL 编写方式称为**预编译 SQL**。

---

# 2. SQL 的两种参数处理方式

在 JDBC 中，可以使用不同方式向 SQL 传递参数。

主要可以分为：

1. 参数直接写入 SQL。
2. 使用占位符动态传递参数。

## 2.1 参数直接写入 SQL

例如：

```java
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM user WHERE username = 'daqiao' AND password = '123456'"
);

ResultSet resultSet = pstmt.executeQuery();
```

这里的用户名和密码直接写在 SQL 语句中。

也就是说：

- `username = 'daqiao'`
- `password = '123456'`

参数已经被固定在 SQL 中。

这种方式的问题是：

> 当用户名和密码来自前端请求时，SQL 中的参数应该是动态变化的，而不应该直接写死在 SQL 中。

## 2.2 使用占位符动态传递参数

使用 `PreparedStatement` 时，可以把 SQL 改成：

```java
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM user WHERE username = ? AND password = ?"
);
```

然后单独设置参数：

```java
pstmt.setString(1, "daqiao");
pstmt.setString(2, "123456");
```

这样 SQL 和参数值就被分开处理。

SQL 本身负责描述查询逻辑：

```sql
SELECT * FROM user
WHERE username = ?
AND password = ?
```

参数值则通过 Java 代码单独传递：

```java
pstmt.setString(1, "daqiao");
pstmt.setString(2, "123456");
```

因此，实际开发中更加推荐这种方式。

---

# 3. `?` 参数占位符

预编译 SQL 使用 `?` 表示参数占位符。

例如：

```sql
SELECT *
FROM user
WHERE username = ?
AND password = ?
```

这里一共有两个占位符。

它们按照从左到右的顺序进行编号：

- 第一个 `?` 的编号是 `1`。
- 第二个 `?` 的编号是 `2`。

> JDBC 中参数占位符的编号从 `1` 开始，而不是从 `0` 开始。

因此：

```java
pstmt.setString(1, username);
pstmt.setString(2, password);
```

表示：

- 将 `username` 设置给第一个 `?`。
- 将 `password` 设置给第二个 `?`。

---

# 4. 参数绑定

创建 `PreparedStatement` 后，需要为 SQL 中的占位符绑定参数。

例如：

```java
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM user WHERE username = ? AND password = ?"
);

pstmt.setString(1, username);
pstmt.setString(2, password);
```

`setString()` 的基本形式为：

```java
pstmt.setString(参数位置, 参数值);
```

例如：

```java
pstmt.setString(1, "daqiao");
```

表示：

> 将字符串 `"daqiao"` 设置给 SQL 中的第一个 `?`。

第二个参数：

```java
pstmt.setString(2, "123456");
```

表示：

> 将字符串 `"123456"` 设置给 SQL 中的第二个 `?`。

---

# 5. 动态参数示例

实际项目中，用户名和密码通常来自用户请求，因此不会直接写死在 SQL 中。

例如：

```java
@ParameterizedTest
@CsvSource({"daqiao,123456"})
public void testJdbc(String username, String password) throws Exception {

    Connection conn = DriverManager.getConnection(
        "jdbc:mysql://localhost:3306/web",
        "root",
        "1234"
    );

    PreparedStatement pstmt = conn.prepareStatement(
        "SELECT * FROM user WHERE username = ? AND password = ?"
    );

    pstmt.setString(1, username);
    pstmt.setString(2, password);

    ResultSet rs = pstmt.executeQuery();

    while (rs.next()) {
        int id = rs.getInt("id");
        String uName = rs.getString("username");
        String pwd = rs.getString("password");
        String name = rs.getString("name");
        int age = rs.getInt("age");

        System.out.println(
            "ID: " + id +
            ", Username: " + uName +
            ", Password: " + pwd +
            ", Name: " + name +
            ", Age: " + age
        );
    }

    rs.close();
    pstmt.close();
    conn.close();
}
```

这里 SQL 的结构始终保持不变：

```sql
SELECT *
FROM user
WHERE username = ?
AND password = ?
```

发生变化的只是 `username` 和 `password` 两个参数的值。

---

# 6. 为什么推荐使用预编译 SQL

这里给出了预编译 SQL 的两个主要优点：

1. 防止 SQL 注入。
2. 性能更高。

其中，SQL 注入是理解 `PreparedStatement` 时非常重要的问题。

---

# 7. SQL 注入

## 7.1 什么是 SQL 注入

SQL 注入是指：

> 通过控制输入内容，修改程序原本定义好的 SQL 语句，从而改变 SQL 的执行逻辑。

SQL 注入比较典型的场景是用户登录。

例如登录 SQL 原本用于判断：

- 用户名是否正确。
- 密码是否正确。

正常情况下，只有用户名和密码同时正确，才能查询到用户数据。

## 7.2 SQL 注入产生的原因

如果程序通过字符串拼接的方式构造 SQL，用户输入的内容可能成为 SQL 语句结构的一部分。

例如用户输入一个特殊的密码：

```text
' or '1' = '1
```

如果程序直接把用户输入拼接到 SQL 中，就可能改变 SQL 原本的判断逻辑。

其中：

```sql
'1' = '1'
```

始终成立。

而 `OR` 表示多个条件之间只需要满足其中一个条件。

这样就可能导致：

> 即使用户名或密码本身是错误的，最终 SQL 仍然能够查询到数据。

如果程序把“查询到用户数据”作为登录成功的依据，就可能出现绕过正常身份验证的情况。

---

# 8. SQL 注入的本质

SQL 注入真正的问题并不只是“用户输入了一段特殊字符串”。

核心问题是：

> 用户输入的数据参与了 SQL 语句结构的构造。

正常情况下，用户输入应该只是一个**参数值**。

但是采用字符串拼接时，用户输入会直接参与 SQL 字符串的构造，这样用户输入就可能改变原有 SQL 的语义。

因此：

> 如果外部输入可以直接参与 SQL 字符串的拼接，就可能产生 SQL 注入风险。

---

# 9. PreparedStatement 如何解决 SQL 注入

使用预编译 SQL 时：

```java
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM user WHERE username = ? AND password = ?"
);
```

SQL 的结构已经确定。

随后再单独设置参数：

```java
pstmt.setString(1, username);
pstmt.setString(2, password);
```

即使用户输入：

```text
' or '1' = '1
```

这个内容也会作为第二个 `?` 对应的一个完整字符串参数进行处理。

也就是说，它的作用只是作为：

> `password` 参数的值。

而不会作为 SQL 语句结构的一部分。

因此，它不会改变原本 SQL 的判断结构。

原本的 SQL 逻辑仍然是：

```sql
SELECT *
FROM user
WHERE username = ?
AND password = ?
```

程序仍然是在数据库中寻找：

> `password` 的值等于用户输入字符串的记录。

如果数据库中不存在这样的密码，就查询不到对应数据。

因此：

> 使用预编译 SQL，可以避免用户输入的数据直接改变 SQL 语句的结构，从而防止这类 SQL 注入问题。

---

# 10. 普通参数和 SQL 结构要区分

使用 `PreparedStatement` 时需要理解一个重要概念：

`?` 主要用于表示**数据参数**。

例如：

```sql
SELECT *
FROM user
WHERE username = ?
```

这里的 `username` 的具体值属于数据，可以通过：

```java
pstmt.setString(1, username);
```

进行传递。

因此可以把 SQL 理解成两个部分。

## 10.1 SQL 结构

例如：

```sql
SELECT *
FROM user
WHERE username = ?
AND password = ?
```

用于确定：

- 查询哪张表。
- 查询哪些字段。
- 使用什么查询条件。

## 10.2 参数值

例如：

```text
daqiao
123456
```

用于确定具体查询的数据。

使用 `PreparedStatement` 时，SQL 结构和参数值被分开处理。

---

# 11. PreparedStatement 的基本使用步骤

使用 `PreparedStatement` 执行查询，一般包含以下步骤。

## 11.1 创建 PreparedStatement

```java
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM user WHERE username = ? AND password = ?"
);
```

## 11.2 设置参数

```java
pstmt.setString(1, username);
pstmt.setString(2, password);
```

参数位置按照 SQL 中 `?` 从左到右的顺序确定。

## 11.3 执行 SQL

查询操作可以使用：

```java
ResultSet rs = pstmt.executeQuery();
```

执行完成后获得查询结果。

## 11.4 处理查询结果

```java
while (rs.next()) {
    String username = rs.getString("username");
}
```

## 11.5 释放资源

```java
rs.close();
pstmt.close();
conn.close();
```

---

# 12. PreparedStatement 的核心特点

可以把 `PreparedStatement` 的核心特点总结为以下几点。

## 12.1 使用占位符

SQL 中通过 `?` 表示动态参数。

## 12.2 SQL 与参数分离

SQL：

```sql
SELECT *
FROM user
WHERE username = ?
```

参数：

```java
pstmt.setString(1, username);
```

两部分分开处理。

## 12.3 参数动态传递

同一条 SQL 可以接收不同的参数值，而不需要把参数直接写死在 SQL 中。

## 12.4 防止 SQL 注入

用户传入的普通参数会作为参数值处理，而不是直接参与 SQL 结构的拼接。

---

# 13. 开发建议

在实际项目中，应优先使用预编译 SQL。

推荐：

```java
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM user WHERE username = ? AND password = ?"
);

pstmt.setString(1, username);
pstmt.setString(2, password);
```

而不是将外部参数直接拼接进 SQL。

核心原则是：

> SQL 负责定义查询结构，外部输入负责提供参数值，不应该让普通用户输入直接改变 SQL 的结构。

---

# 14. 学习重点

## 14.1 基础

需要掌握：

- `PreparedStatement` 的作用。
- 什么是预编译 SQL。
- `?` 占位符的作用。
- JDBC 参数编号从 `1` 开始。
- 如何使用 `setString()` 设置参数。
- 如何使用 `executeQuery()` 执行查询。

## 14.2 重点

需要进一步理解：

- 参数硬编码和动态参数有什么区别。
- 为什么不应该直接拼接用户输入。
- 什么是 SQL 注入。
- SQL 注入为什么会改变原 SQL 的逻辑。
- 为什么使用预编译 SQL 可以防止 SQL 注入。

---

>[!tip] 一句话总结
> `PreparedStatement` 通过 `?` 占位符将 SQL 结构与参数值分开处理，使参数可以动态传递，同时避免普通用户输入直接改变 SQL 语句的结构，因此能够有效防止 SQL 注入，也是 JDBC 开发中推荐使用的 SQL 执行方式。