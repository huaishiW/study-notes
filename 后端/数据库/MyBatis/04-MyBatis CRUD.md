# 1. MyBatis 中的 CRUD

CRUD 是数据库开发中最常见的四类操作：

- Create：新增
    
- Read：查询
    
- Update：修改
    
- Delete：删除
    

在 MyBatis 中，可以通过 Mapper 接口配合注解来完成这些操作。

常用注解包括：

```java
@Insert
@Select
@Update
@Delete
```

它们分别对应：

|操作|SQL|MyBatis 注解|
|---|---|---|
|新增|`INSERT`|`@Insert`|
|查询|`SELECT`|`@Select`|
|修改|`UPDATE`|`@Update`|
|删除|`DELETE`|`@Delete`|

这些注解的核心作用是：

> 将 Mapper 方法和对应的 SQL 语句绑定起来。

例如：

```java
@Select("select * from user")
List<User> findAll();
```

表示调用：

```java
findAll();
```

时，MyBatis 会执行：

```sql
select * from user;
```

---

# 2. @Delete 删除数据

假设需求是：

> 根据用户 ID 删除用户信息。

对应 SQL：

```sql
delete from user where id = 5;
```

如果直接写成：

```java
@Delete("delete from user where id = 5")
void deleteById();
```

这种写法存在明显问题：

> ID 被直接写死在 SQL 中。

这样无论调用多少次：

```java
deleteById();
```

都只能删除：

```text
id = 5
```

的用户。

因此实际开发中需要使用动态参数。

---

# 3. 使用 #{} 传递删除参数

可以将 SQL 修改为：

```java
@Delete("delete from user where id = #{id}")
void deleteById(Integer id);
```

调用时：

```java
userMapper.deleteById(36);
```

此时方法参数：

```java
36
```

会传递给：

```text
#{id}
```

对应的位置。

因此最终执行的是针对指定用户 ID 的删除操作。

`#{}` 是 MyBatis 中非常重要的参数占位方式。

---

# 4. #{} 是什么

`#{}` 是 MyBatis 提供的：

> 参数占位符。

例如：

```java
@Delete("delete from user where id = #{id}")
void deleteById(Integer id);
```

其中：

```text
#{id}
```

用于获取 Mapper 方法中传入的参数。

调用：

```java
userMapper.deleteById(36);
```

之后，MyBatis 会将参数值：

```text
36
```

绑定到 SQL 中。

---

# 5. #{} 底层使用预编译 SQL

`#{}` 最重要的特点之一是：

> 在实际执行 SQL 时，会被转换成 JDBC 中的 `?` 占位符。

例如：

```java
@Delete("delete from user where id = #{id}")
```

最终类似于：

```sql
delete from user where id = ?;
```

然后再单独传递参数：

```text
36
```

因此 `#{}` 本质上使用的是：

> 预编译 SQL。

这和 JDBC 中的：

```java
PreparedStatement
```

参数绑定机制是一致的。

也正因为如此，`#{}` 是 MyBatis 中推荐使用的参数传递方式。

---

# 6. DML 操作的返回值

删除、修改、新增都属于 DML。

DML SQL 执行完成后，会有一个：

> 受影响的记录数量。

因此 Mapper 方法可以定义返回值：

```java
@Delete("delete from user where id = #{id}")
Integer deleteById(Integer id);
```

例如：

```java
Integer rows = userMapper.deleteById(36);
```

如果：

```text
rows = 1
```

表示本次删除操作影响了一条数据库记录。

这个返回值与 JDBC 中：

```java
executeUpdate();
```

返回受影响记录数的机制一致。

---

# 7. @Insert 新增数据

假设需要添加一个用户。

对应 SQL：

```sql
insert into user(username, password, name, age)
values('zhouyu', '123456', '周瑜', 20);
```

如果每个数据都直接写死在 SQL 中，就无法动态新增不同用户。

因此可以使用：

```java
@Insert("""
    insert into user(username, password, name, age)
    values(#{username}, #{password}, #{name}, #{age})
""")
void insert(User user);
```

这里的：

```text
#{username}
#{password}
#{name}
#{age}
```

用于获取 `User` 对象中的对应属性值。

---

# 8. 对象参数如何传入 SQL

如果 Mapper 方法接收一个对象：

```java
void insert(User user);
```

则 MyBatis 可以通过：

```text
#{属性名}
```

获取对象中的属性。

例如：

```java
User user = new User();

user.setUsername("admin");
user.setPassword("123456");
user.setName("管理员");
user.setAge(30);

userMapper.insert(user);
```

SQL：

```java
@Insert("""
    insert into user(username, password, name, age)
    values(#{username}, #{password}, #{name}, #{age})
""")
```

MyBatis 会读取：

```java
user.getUsername();
user.getPassword();
user.getName();
user.getAge();
```

对应的属性值。

因此：

```text
#{username}
```

表示获取：

```java
user.username
```

对应的数据。

这种方式特别适合：

> SQL 中需要同时传递多个相关参数的场景。

---

# 9. 为什么多个参数适合封装成对象

如果新增用户需要：

- username
    
- password
    
- name
    
- age
    

可以设计成：

```java
void insert(
    String username,
    String password,
    String name,
    Integer age
);
```

但随着字段越来越多，方法参数会变得很长。

因此更常见的方式是：

```java
void insert(User user);
```

然后在 SQL 中通过：

```text
#{username}
#{password}
#{name}
#{age}
```

读取对象属性。

这样 Mapper 方法会更加简洁。

---

# 10. @Update 修改数据

假设需要：

> 根据用户 ID 修改用户信息。

SQL：

```sql
update user
set username = 'zhouyu',
    password = '123456',
    name = '周瑜',
    age = 20
where id = 1;
```

在 MyBatis 中可以写成：

```java
@Update("""
    update user
    set username = #{username},
        password = #{password},
        name = #{name},
        age = #{age}
    where id = #{id}
""")
void update(User user);
```

这里同样通过：

```text
#{属性名}
```

获取 `User` 对象中的数据。

---

# 11. 修改操作示例

调用时：

```java
User user = new User();

user.setId(6);
user.setUsername("admin666");
user.setPassword("123456");
user.setName("管理员");
user.setAge(30);

userMapper.update(user);
```

SQL 中：

```text
#{id}
```

获取：

```java
user.getId();
```

而：

```text
#{username}
```

获取：

```java
user.getUsername();
```

其余属性也是同样的机制。

因此，当一个 SQL 涉及一个完整对象的多个属性时，直接传递对象是一种非常常见的写法。

---

# 12. @Select 查询数据

假设需求是：

> 根据用户名和密码查询用户信息。

对应 SQL：

```sql
select *
from user
where username = 'zhouyu'
  and password = '123456';
```

在 MyBatis 中可以定义：

```java
@Select("""
    select *
    from user
    where username = #{username}
      and password = #{password}
""")
User findByUsernameAndPassword(
    @Param("username") String username,
    @Param("password") String password
);
```

这里 Mapper 方法有两个参数：

```java
String username
String password
```

因此需要解决一个问题：

> SQL 中的 `#{username}` 和 `#{password}` 分别应该对应哪个方法参数？

这就涉及：

```java
@Param
```

注解。

---

# 13. @Param 是什么

`@Param` 的作用是：

> 给 Mapper 方法的参数起一个明确的名字。

例如：

```java
User findByUsernameAndPassword(
    @Param("username") String username,
    @Param("password") String password
);
```

这里：

```java
@Param("username")
```

表示将第一个方法参数命名为：

```text
username
```

所以 SQL 中可以写：

```text
#{username}
```

同理：

```java
@Param("password")
```

对应：

```text
#{password}
```

---

# 14. @Param 的参数绑定过程

例如：

```java
userMapper.findByUsernameAndPassword(
    "admin666",
    "123456"
);
```

Mapper：

```java
@Select("""
    select *
    from user
    where username = #{username}
      and password = #{password}
""")
User findByUsernameAndPassword(
    @Param("username") String username,
    @Param("password") String password
);
```

可以这样理解：

第一个参数：

```text
admin666
```

通过：

```java
@Param("username")
```

绑定给：

```text
#{username}
```

第二个参数：

```text
123456
```

通过：

```java
@Param("password")
```

绑定给：

```text
#{password}
```

这样 MyBatis 就可以明确知道每个 SQL 参数应该从哪里获取。

---

# 15. @Param 是否必须使用

在当前 Spring Boot 项目中，如果编译后能够保留方法参数名，则：

```java
@Param
```

可以省略。

例如：

```java
@Select("""
    select *
    from user
    where username = #{username}
      and password = #{password}
""")
User findByUsernameAndPassword(
    String username,
    String password
);
```

MyBatis 可以直接通过：

```text
username
password
```

识别方法参数。

因此：

> `@Param` 的核心作用是明确指定参数名称，而不是所有情况下都必须使用。

不过在理解参数绑定机制时，需要掌握 `@Param` 的作用。

---

# 16. #{} 与 ${}

MyBatis 提供了两种常见参数写法：

```text
#{}
```

和：

```text
${}
```

它们看起来相似，但工作机制完全不同。

这是 MyBatis 参数处理里非常重要的一组区别。

---

# 17. #{} 的工作方式

`#{}` 是：

> 参数占位符。

例如：

```java
@Delete("delete from user where id = #{id}")
```

执行时，MyBatis 会将：

```text
#{id}
```

转换成：

```text
?
```

最终形成预编译 SQL：

```sql
delete from user where id = ?;
```

然后单独传递参数值。

因此：

```text
#{}
```

具有两个主要特点：

- 安全性更高。
    
- 使用预编译 SQL。
    

所以实际开发中：

> 推荐优先使用 `#{}`。

---

# 18. ${} 的工作方式

`${}` 是：

> SQL 字符串拼接符。

它不会转换成：

```text
?
```

而是直接把参数内容拼接进 SQL。

例如：

```text
${tableName}
```

参数的值会直接成为 SQL 字符串的一部分。

因此 `${}` 与 `#{}` 最大的区别在于：

- `#{}`：参数绑定。
    
- `${}`：字符串拼接。
    

---

# 19. #{} 与 ${} 的核心区别

|对比项|`#{}`|`${}`|
|---|---|---|
|类型|参数占位符|字符串拼接|
|最终 SQL|转换为 `?`|直接拼接参数|
|是否预编译|是|否|
|SQL 注入风险|较低|存在风险|
|性能|更好|相对较低|
|推荐程度|推荐|特殊情况使用|

这一部分中最重要的原则是：

> 只要参数能够使用 `#{}`，就应该优先使用 `#{}`。

---

# 20. 为什么 ${} 存在 SQL 注入风险

`${}` 会把参数值直接拼接到 SQL 中。

也就是说：

> 参数内容可能成为 SQL 结构的一部分。

这和 JDBC 中直接通过字符串拼接构造 SQL 的问题类似。

因此，如果 `${}` 的值来自不可信的外部输入，就可能产生 SQL 注入风险。

这也是为什么实际开发中不应该随意使用 `${}`。

---

# 21. ${} 什么时候使用

`${}` 并不是完全不能使用。

它主要用于：

> SQL 结构本身需要动态变化的场景。

例如：

- 动态表名。
    
- 动态字段名。
    

因为：

```text
#{}
```

主要用于表示：

> SQL 中的数据值。

而不能直接代表 SQL 结构中的表名、字段名等内容。

所以两者可以简单区分为：

> 数据值优先使用 `#{}`，SQL 结构动态变化时才考虑 `${}`。

由于 `${}` 属于字符串拼接，因此使用时需要格外注意参数安全。

---

# 22. 参数是单个值时

如果 Mapper 方法只有一个简单参数：

```java
void deleteById(Integer id);
```

SQL 可以直接使用：

```java
@Delete("delete from user where id = #{id}")
```

此时参数绑定比较简单。

调用：

```java
userMapper.deleteById(36);
```

即可。

---

# 23. 参数是对象时

如果 SQL 需要多个属于同一个对象的数据：

```java
void insert(User user);
```

可以直接通过对象属性名访问：

```java
@Insert("""
    insert into user(username, password, name, age)
    values(#{username}, #{password}, #{name}, #{age})
""")
```

这种方式适用于：

- 新增对象。
    
- 修改对象。
    

---

# 24. 方法有多个独立参数时

如果参数不是封装在一个对象中，而是多个独立参数：

```java
User findByUsernameAndPassword(
    String username,
    String password
);
```

可以通过：

```java
@Param
```

明确参数名称：

```java
User findByUsernameAndPassword(
    @Param("username") String username,
    @Param("password") String password
);
```

然后 SQL 中使用：

```text
#{username}
#{password}
```

因此可以根据参数形式选择不同的绑定方式。

---

# 25. CRUD Mapper 完整示例

一个基础的 `UserMapper` 可以包含：

```java
@Mapper
public interface UserMapper {

    /**
     * 根据ID删除用户
     */
    @Delete("delete from user where id = #{id}")
    Integer deleteById(Integer id);

    /**
     * 新增用户
     */
    @Insert("""
        insert into user(username, password, name, age)
        values(#{username}, #{password}, #{name}, #{age})
    """)
    void insert(User user);

    /**
     * 修改用户
     */
    @Update("""
        update user
        set username = #{username},
            password = #{password},
            name = #{name},
            age = #{age}
        where id = #{id}
    """)
    void update(User user);

    /**
     * 根据用户名和密码查询用户
     */
    @Select("""
        select *
        from user
        where username = #{username}
          and password = #{password}
    """)
    User findByUsernameAndPassword(
        @Param("username") String username,
        @Param("password") String password
    );
}
```

这个接口已经包含 MyBatis 中最基础的四种数据库操作。

---

# 26. Mapper 方法与 SQL 的对应关系

MyBatis 中可以把 Mapper 方法理解为：

> Java 代码访问数据库的入口。

例如：

```java
userMapper.deleteById(1);
```

对应：

```sql
delete from user where id = ?;
```

调用：

```java
userMapper.insert(user);
```

对应：

```sql
insert into user(...);
```

调用：

```java
userMapper.update(user);
```

对应：

```sql
update user set ...;
```

调用：

```java
userMapper.findByUsernameAndPassword(...);
```

对应：

```sql
select * from user where ...;
```

因此 Mapper 负责在：

> Java 方法

和：

> SQL 操作

之间建立对应关系。

---

# 27. MyBatis CRUD 与 JDBC 的区别

在 JDBC 中执行删除，需要：

```java
PreparedStatement pstmt =
    conn.prepareStatement(
        "delete from user where id = ?"
    );

pstmt.setInt(1, id);

int rows = pstmt.executeUpdate();
```

而 MyBatis 中只需要：

```java
@Delete("delete from user where id = #{id}")
Integer deleteById(Integer id);
```

可以看到，MyBatis 帮助我们省略了：

- 创建 `PreparedStatement`
    
- 手动设置参数
    
- 调用 `executeUpdate()`
    
- 管理 JDBC 执行过程
    

开发者主要关注：

> Mapper 方法和 SQL。

---

# 28. #{} 与 JDBC 中 ? 的关系

这是理解 MyBatis 参数绑定时非常重要的一点。

MyBatis 中：

```text
#{id}
```

最终会被转换为：

```text
?
```

因此：

```java
@Delete("delete from user where id = #{id}")
```

最终执行逻辑类似 JDBC：

```java
PreparedStatement pstmt =
    conn.prepareStatement(
        "delete from user where id = ?"
    );

pstmt.setInt(1, id);
```

所以：

> `#{}` 是 MyBatis 对 JDBC 预编译参数绑定机制的一层封装。

这也是为什么 `#{}` 具有较高安全性。

---

# 29. CRUD 中真正需要掌握的核心

学习这一部分时，不应该只记：

```java
@Insert
@Delete
@Update
@Select
```

更重要的是理解三个问题。

## 29.1 Mapper 方法如何对应 SQL

通过：

```java
@Insert
@Delete
@Update
@Select
```

将 Mapper 方法与 SQL 绑定。

---

## 29.2 Java 参数如何传入 SQL

主要通过：

```text
#{}
```

完成参数绑定。

参数可以来自：

- 单个方法参数。
    
- Java 对象属性。
    
- `@Param` 指定的参数名称。
    

---

## 29.3 #{} 和 ${} 为什么不同

因为：

```text
#{}
```

使用的是参数绑定和预编译 SQL。

而：

```text
${}
```

使用的是字符串拼接。

这是二者最本质的区别。

---

# 30. 学习重点

## 30.1 基础

需要掌握：

- `@Insert` 的作用。
    
- `@Delete` 的作用。
    
- `@Update` 的作用。
    
- `@Select` 的作用。
    
- `#{}` 的作用。
    
- `${}` 的作用。
    
- `@Param` 的作用。
    

## 30.2 重点

需要重点理解：

- `#{}` 为什么会转换成 `?`。
    
- 为什么推荐使用 `#{}`。
    
- `#{}` 与 `${}` 的本质区别。
    
- 为什么 `${}` 存在 SQL 注入风险。
    
- 对象参数如何通过 `#{属性名}` 获取数据。
    
- 多个独立方法参数如何使用 `@Param`。
    
- DML Mapper 方法为什么可以返回受影响记录数。
    

---

> [!tip] 一句话总结
> 
> MyBatis 可以通过 `@Insert`、`@Delete`、`@Update`、`@Select` 将 Mapper 方法与 CRUD SQL 绑定；参数通常使用 `#{}` 传递，它会被转换成 JDBC 的 `?` 并使用预编译 SQL，而 `${}` 属于字符串拼接，存在 SQL 注入风险；当 Mapper 方法包含多个独立参数时，可以使用 `@Param` 明确参数名称。