# 1. ResultSet 是什么

`ResultSet` 是 JDBC 中的**结果集对象**，用于封装 DQL 查询语句返回的数据。

当使用 `PreparedStatement` 执行查询时：

```java
ResultSet rs = pstmt.executeQuery();
```

`executeQuery()` 会将数据库查询得到的结果返回，并封装到 `ResultSet` 对象中。

例如执行：

```sql
SELECT * FROM user;
```

如果数据库查询出多条用户数据，这些查询结果就可以通过 `ResultSet` 逐行读取。

因此可以理解为：

> `ResultSet` 是 JDBC 用来接收和读取数据库查询结果的对象。

---

# 2. ResultSet 的产生

`ResultSet` 通常来自：

```java
pstmt.executeQuery();
```

完整写法为：

```java
ResultSet rs = pstmt.executeQuery();
```

其中：

- `pstmt` 是 `PreparedStatement` 对象。
- `executeQuery()` 用于执行 DQL 查询语句。
- `ResultSet` 保存查询得到的数据。

例如：

```java
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM user WHERE username = ? AND password = ?"
);

pstmt.setString(1, "daqiao");
pstmt.setString(2, "123456");

ResultSet rs = pstmt.executeQuery();
```

执行过程可以用文字描述为：

1. `PreparedStatement` 准备需要执行的 SQL。
2. 设置 SQL 中的参数。
3. 调用 `executeQuery()` 执行查询。
4. 数据库返回查询结果。
5. JDBC 将查询结果封装成 `ResultSet`。
6. Java 程序通过 `ResultSet` 读取这些数据。

---

# 3. ResultSet 中的光标

理解 `ResultSet` 时，一个非常重要的概念是：**光标**。

`ResultSet` 中可能包含多条数据库记录。

Java 程序并不是一次把所有记录直接取出来，而是通过光标逐行读取数据。

在读取某一条记录之前，需要先让光标移动到这一行。

这个操作通过：

```java
rs.next();
```

完成。

---

# 4. next() 方法

`next()` 用于：

> 将 ResultSet 的光标从当前位置向前移动一行，并判断移动后的当前行是否是一条有效数据。

基本写法：

```java
rs.next();
```

返回值类型为：

```java
boolean
```

它有两种结果。

## 4.1 返回 true

如果：

```java
rs.next()
```

返回：

```text
true
```

表示当前光标所在位置是一条有效数据。

此时可以继续通过：

```java
rs.getInt(...)
rs.getString(...)
```

等方法读取当前记录。

---

## 4.2 返回 false

如果：

```java
rs.next()
```

返回：

```text
false
```

表示已经没有下一条有效数据。

也就是说，结果集中的数据已经读取完毕。

---

# 5. 为什么通常使用 while 遍历 ResultSet

由于数据库查询可能返回多条记录，所以通常会使用：

```java
while (rs.next()) {

}
```

遍历查询结果。

例如：

```java
while (rs.next()) {
    int id = rs.getInt("id");
    String username = rs.getString("username");
    String password = rs.getString("password");
    String name = rs.getString("name");
    int age = rs.getInt("age");
}
```

这里：

```java
rs.next()
```

每执行一次，就尝试让光标移动到下一条记录。

如果当前记录存在：

```text
next() → true
```

则执行 `while` 循环中的代码。

如果已经没有数据：

```text
next() → false
```

则结束循环。

因此：

```java
while (rs.next()) {
    // 读取当前记录
}
```

可以理解为：

> 只要结果集中还有下一条有效记录，就继续读取。

---

# 6. ResultSet 的遍历过程

假设数据库查询结果共有三条数据。

第一次执行：

```java
rs.next();
```

如果存在第一条记录，则返回：

```text
true
```

此时可以读取第一条记录。

第二次执行：

```java
rs.next();
```

光标继续移动到第二条记录，并返回：

```text
true
```

第三次执行时，同样读取第三条记录。

当第四次执行：

```java
rs.next();
```

由于已经不存在第四条数据，因此返回：

```text
false
```

循环结束。

所以一个完整的遍历过程可以描述为：

1. 调用 `next()`。
2. 光标移动到下一条记录。
3. 判断当前记录是否有效。
4. 如果有效，读取当前行的数据。
5. 再次调用 `next()`。
6. 重复以上过程。
7. 当 `next()` 返回 `false` 时，遍历结束。

---

# 7. getXxx() 方法

当光标已经移动到一条有效记录后，需要读取当前记录中的字段。

`ResultSet` 提供了一系列：

```text
getXxx()
```

方法获取数据。

例如：

```java
rs.getInt("id");
rs.getString("username");
```

其中 `Xxx` 通常对应需要读取的数据类型。

---

# 8. getInt()

如果数据库字段需要以整数形式读取，可以使用：

```java
rs.getInt(...)
```

例如：

```java
int id = rs.getInt("id");
```

表示获取当前记录中：

```text
id
```

字段的值，并保存到 Java 的 `int` 变量中。

例如：

```java
int age = rs.getInt("age");
```

表示读取当前记录中的年龄。

---

# 9. getString()

如果数据库字段需要以字符串形式读取，可以使用：

```java
rs.getString(...)
```

例如：

```java
String username = rs.getString("username");
```

表示读取当前记录中的：

```text
username
```

字段。

同样：

```java
String password = rs.getString("password");
String name = rs.getString("name");
```

分别读取： `password`、 `name`字段。

---

# 10. 根据列名获取数据

`ResultSet` 可以根据数据库字段名称获取数据。

例如：

```java
int id = rs.getInt("id");

String username = rs.getString("username");

String password = rs.getString("password");

String name = rs.getString("name");

int age = rs.getInt("age");
```

这里传入的：

```text
id
username
password
name
age
```

都是查询结果中的列名。

推荐使用这种方式获取数据。

原因在于这种写法能够直接看出代码正在读取哪个数据库字段，代码含义更加明确。

例如：

```java
rs.getString("username");
```

能够非常直接地表示：

> 获取当前记录的 `username` 字段。

---

# 11. 根据列编号获取数据

除了使用列名之外，`ResultSet` 还可以根据列的编号获取数据。

也就是说，`getXxx()` 可以使用：

- 列编号
- 列名

两种方式定位数据。

例如，如果查询结果中的第一列是 `id`，理论上可以根据对应的列位置获取数据。

不过还是推荐：

> 优先根据列名获取数据。

因此学习和实际编写代码时，可以重点掌握：

```java
rs.getInt("id");
rs.getString("username");
```

这种形式。

---

# 12. ResultSet 数据解析

假设数据库中的 `user` 表包含：

- `id`
- `username`
- `password`
- `name`
- `age`

执行查询：

```java
ResultSet rs = pstmt.executeQuery();
```

之后，可以通过：

```java
while (rs.next()) {
    int id = rs.getInt("id");
    String username = rs.getString("username");
    String password = rs.getString("password");
    String name = rs.getString("name");
    int age = rs.getInt("age");

    System.out.println(
        "ID: " + id +
        ", Username: " + username +
        ", Password: " + password +
        ", Name: " + name +
        ", Age: " + age
    );
}
```

完成查询结果的解析。

这段代码实际上完成了两个核心步骤。

## 12.1 定位当前记录

通过：

```java
rs.next();
```

移动到下一条查询结果。

## 12.2 获取当前记录中的字段

通过：

```java
rs.getInt(...)
rs.getString(...)
```

分别读取当前记录中的不同字段。

因此，ResultSet 的结果解析过程可以概括为：

> 先使用 `next()` 定位一条记录，再使用 `getXxx()` 获取当前记录中的各个字段。

---

# 13. 完整查询示例

```java
Connection conn = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/web",
    "root",
    "1234"
);

PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM user WHERE username = ? AND password = ?"
);

pstmt.setString(1, "daqiao");
pstmt.setString(2, "123456");

ResultSet rs = pstmt.executeQuery();

while (rs.next()) {

    int id = rs.getInt("id");

    String username = rs.getString("username");

    String password = rs.getString("password");

    String name = rs.getString("name");

    int age = rs.getInt("age");

    System.out.println(
        "ID: " + id +
        ", Username: " + username +
        ", Password: " + password +
        ", Name: " + name +
        ", Age: " + age
    );
}

rs.close();
pstmt.close();
conn.close();
```

这段代码中和 `ResultSet` 直接相关的核心部分是：

```java
ResultSet rs = pstmt.executeQuery();

while (rs.next()) {
    int id = rs.getInt("id");
    String username = rs.getString("username");
}
```

其中：

- `executeQuery()`：执行查询并返回结果集。
- `ResultSet`：保存查询结果。
- `next()`：移动光标，并判断当前行是否有效。
- `getInt()`：读取整数数据。
- `getString()`：读取字符串数据。

---

# 14. ResultSet 与 executeQuery()

查询语句通常使用：

```java
executeQuery();
```

例如：

```java
ResultSet resultSet = pstmt.executeQuery();
```

`executeQuery()` 的返回值就是：

```java
ResultSet
```

因此：

> DQL 查询的结果通过 `ResultSet` 返回给 Java 程序。

与之不同的是，增、删、改操作一般使用：

```java
executeUpdate();
```

例如：

```java
int rowsUpdated = pstmt.executeUpdate();
```

它返回的是受影响的记录数量，而不是 `ResultSet`。

所以需要区分：

```text
DQL 查询：
executeQuery()
返回 ResultSet

DML 增删改：
executeUpdate()
返回受影响的记录数
```

---

# 15. ResultSet 资源释放

`ResultSet` 使用完成后，需要关闭：

```java
rs.close();
```

在 JDBC 中资源通常按照以下形式关闭：

```java
rs.close();
pstmt.close();
conn.close();
```

也就是在查询结果处理完成后，依次释放使用到的 JDBC 资源。

因此在原生 JDBC 代码中，不仅要关心：

> 如何获取数据。

还要注意：

> 数据读取完成之后，需要关闭相应资源。

---

# 16. 常见理解误区

## 16.1 ResultSet 不是一条数据

`ResultSet` 表示的是整个查询结果集。

例如 SQL 返回十条记录时：

```java
ResultSet rs
```

中包含的是这次查询得到的结果，而不是其中某一条记录。

具体读取哪条记录，需要依靠：

```java
rs.next();
```

移动光标。

---

## 16.2 next() 不是获取字段值

`next()` 的主要作用是：

> 移动光标，并判断当前位置是否是一条有效数据。

真正读取字段需要使用：

```java
getInt()
getString()
```

等 `getXxx()` 方法。

所以：

```java
rs.next();
```

负责定位记录。

而：

```java
rs.getString("username");
```

负责读取当前记录中的数据。

---

## 16.3 不能只写 getXxx() 而忽略 next()

通常需要先通过：

```java
rs.next();
```

让光标移动到有效记录，再读取当前行中的字段。

因此典型写法是：

```java
while (rs.next()) {
    String username = rs.getString("username");
}
```

而不是单独只考虑：

```java
rs.getString("username");
```

---

# 17. ResultSet 的核心使用模式

使用 `ResultSet` 时，可以记住一个非常固定的基本模式：

```java
ResultSet rs = pstmt.executeQuery();

while (rs.next()) {

    数据类型 变量 = rs.getXxx("列名");

}

rs.close();
```

核心实际上只有三个动作：

### 获取结果集

```java
ResultSet rs = pstmt.executeQuery();
```

### 遍历结果

```java
while (rs.next()) {

}
```

### 获取字段

```java
rs.getXxx("列名");
```

因此学习 `ResultSet` 不需要记忆大量代码，重点是掌握：

> `executeQuery()` 获取结果集 → `next()` 定位记录 → `getXxx()` 获取字段。

---

# 18. 学习重点

## 18.1 基础

需要掌握：

- `ResultSet` 是什么。
- `ResultSet` 用于封装什么数据。
- `executeQuery()` 的返回值是什么。
- `next()` 方法有什么作用。
- `next()` 为什么返回 `boolean`。
- `getXxx()` 方法有什么作用。

## 18.2 重点

需要重点理解：

- `ResultSet` 中光标的作用。
- 为什么遍历结果集通常使用 `while (rs.next())`。
- `next()` 与 `getXxx()` 的职责区别。
- 如何根据列名获取数据库字段。
- DQL 查询为什么返回 `ResultSet`。

---

> [!tip]  一句话总结
> `ResultSet` 是 JDBC 用于封装 DQL 查询结果的对象；读取数据时先通过 `next()` 移动光标并判断当前行是否有效，再通过 `getXxx()` 获取当前记录中的字段值。