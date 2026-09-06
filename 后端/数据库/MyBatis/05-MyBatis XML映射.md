# 1. MyBatis 的两种 SQL 配置方式

MyBatis 中编写 SQL 主要有两种方式：

1. 注解方式。
    
2. XML 映射方式。
    

注解方式例如：

```java
@Select("select * from user")
List<User> findAll();
```

XML 方式则是：

> 将 SQL 从 Mapper 接口中移出，单独写到 XML 映射文件中。

这两种方式本质上都是为了完成：

> Mapper 方法和 SQL 语句之间的映射。

区别主要在于 SQL 写在哪里。

---

# 2. 为什么需要 XML 映射

注解方式比较适合简单的增删改查。

例如：

```java
@Select("select * from user")
List<User> findAll();
```

这种 SQL 比较短，直接写在注解中比较方便。

但随着 SQL 逐渐复杂，例如：

- SQL 语句变长。
    
- 查询条件变多。
    
- SQL 结构更加复杂。
    

如果继续全部写在 Java 注解中，代码会逐渐变得不够清晰。

因此 MyBatis 还提供 XML 映射文件方式：

> 将复杂 SQL 独立写在 XML 文件中，使 Mapper 接口和 SQL 配置分离。

所以可以简单理解为：

- 简单 SQL：可以使用注解。
    
- 复杂 SQL：更适合使用 XML。
    

---

# 3. Mapper 接口与 XML 映射文件

使用 XML 方式后，Mapper 接口中通常只保留方法定义。

例如：

```java
@Mapper
public interface UserMapper {

    List<User> findAll();
}
```

这里没有：

```java
@Select
```

也没有直接写 SQL。

SQL 会放在对应的 XML 文件中：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">

    <select id="findAll"
            resultType="com.huaishi.pojo.User">
        select * from user
    </select>

</mapper>
```

这样：

```java
List<User> findAll();
```

就会和 XML 中：

```xml
<select id="findAll">
```

建立对应关系。

---

# 4. XML 映射文件的核心作用

XML 映射文件主要负责：

> 描述 Mapper 接口中的方法应该执行什么 SQL，以及查询结果应该如何封装。

例如：

```xml
<select id="findAll"
        resultType="com.huaishi.pojo.User">
    select * from user
</select>
```

这里包含三个重要信息：

- `<select>`：这是一条查询 SQL。
    
- `id="findAll"`：对应 Mapper 中的 `findAll()` 方法。
    
- `resultType="com.huaishi.pojo.User"`：每条查询结果封装成 `User` 对象。
    

因此 XML 并不是一个独立运行的文件。

它需要与：

> Mapper 接口

配合使用。

---

# 5. XML 映射文件的基本结构

一个基础的 MyBatis XML 映射文件可以写成：

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.huaishi.mapper.UserMapper">

    <select id="findAll"
            resultType="com.huaishi.pojo.User">
        select * from user
    </select>

</mapper>
```

其中最核心的是：

```xml
<mapper namespace="...">
```

以及里面的 SQL 映射标签：

```xml
<select>
```

---

# 6. XML 映射文件必须遵守的规范

使用 XML 映射方式时，需要保证 Mapper 接口和 XML 文件能够正确对应。

主要有三个重要规范。

## 6.1 文件名与 Mapper 接口名一致

例如 Mapper 接口：

```text
UserMapper.java
```

对应 XML 文件：

```text
UserMapper.xml
```

也就是：

> 同名。

---

## 6.2 Mapper 接口与 XML 放在对应位置

XML 映射文件需要和 Mapper 接口保持相应的包结构。

例如 Mapper 接口位于：

```text
com.huaishi.mapper
```

那么 XML 文件也需要按照相应目录结构进行放置。

核心目的只有一个：

> 让 MyBatis 能够正确找到这个 Mapper 对应的 XML 映射文件。

---

## 6.3 namespace 必须对应 Mapper 接口

XML 中：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">
```

其中：

```text
com.huaishi.mapper.UserMapper
```

必须对应 Mapper 接口的：

> 全限定类名。

例如接口：

```java
package com.huaishi.mapper;

public interface UserMapper {
}
```

那么它的全限定名就是：

```text
com.huaishi.mapper.UserMapper
```

因此 XML 中必须写：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">
```

---

# 7. 什么是全限定类名

全限定类名可以理解为：

> 包名 + 类名。

例如：

```java
package com.huaishi.mapper;

public interface UserMapper {
}
```

类名是：

```text
UserMapper
```

包名是：

```text
com.huaishi.mapper
```

完整的全限定类名就是：

```text
com.huaishi.mapper.UserMapper
```

因此：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">
```

实际上是在告诉 MyBatis：

> 当前 XML 映射文件属于 `UserMapper` 接口。

---

# 8. namespace 的作用

`namespace` 的核心作用是：

> 将一个 XML 映射文件和某一个 Mapper 接口建立对应关系。

例如：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">
```

表示：

> 当前 XML 中定义的 SQL 都属于 `UserMapper`。

如果还有：

```java
EmpMapper
```

则应该有对应的：

```xml
<mapper namespace="com.huaishi.mapper.EmpMapper">
```

因此不同 Mapper 的 SQL 可以通过 `namespace` 区分开。

---

# 9. statement id

XML 中每一条 SQL 都需要一个：

```text
id
```

例如：

```xml
<select id="findAll"
        resultType="com.huaishi.pojo.User">
    select * from user
</select>
```

其中：

```text
findAll
```

就是这条 SQL 的 `id`。

这个 `id` 必须与 Mapper 接口中的：

> 方法名一致。

例如 Mapper：

```java
List<User> findAll();
```

那么 XML 中就需要：

```xml
<select id="findAll">
```

如果 Mapper 方法是：

```java
User findById(Integer id);
```

那么对应 SQL 的 `id` 就应该是：

```xml
<select id="findById">
```

---

# 10. statement id 的作用

`namespace` 负责确定：

> 属于哪个 Mapper。

而 `id` 负责确定：

> 对应 Mapper 中的哪个方法。

例如：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">

    <select id="findAll">
        ...
    </select>

</mapper>
```

可以理解为：

- `namespace` 找到 `UserMapper`。
    
- `id` 找到 `findAll()`。
    

因此 MyBatis 可以准确找到：

```java
UserMapper.findAll()
```

对应的 SQL。

---

# 11. Mapper 方法与 XML SQL 如何对应

假设 Mapper 中有：

```java
@Mapper
public interface UserMapper {

    List<User> findAll();
}
```

XML：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">

    <select id="findAll"
            resultType="com.huaishi.pojo.User">
        select * from user
    </select>

</mapper>
```

当程序调用：

```java
userMapper.findAll();
```

时，MyBatis 可以根据两个信息找到 SQL。

第一步，通过：

```text
namespace
```

找到：

```text
UserMapper
```

第二步，通过：

```text
id
```

找到：

```text
findAll
```

最终定位到：

```sql
select * from user;
```

因此：

> `namespace + id` 共同确定了一条 Mapper SQL 映射。

---

# 12. select 标签

查询 SQL 使用：

```xml
<select>
```

标签。

例如：

```xml
<select id="findAll"
        resultType="com.huaishi.pojo.User">

    select * from user

</select>
```

`<select>` 表示：

> 当前映射的是一条 SELECT 查询语句。

它内部直接编写 SQL：

```sql
select * from user;
```

---

# 13. resultType

查询数据之后，数据库返回的是查询结果。

MyBatis 需要知道：

> 每一条查询结果应该封装成什么 Java 类型。

这就是：

```xml
resultType
```

的作用。

例如：

```xml
<select id="findAll"
        resultType="com.huaishi.pojo.User">
```

表示：

> 查询返回的每一条数据库记录，都封装成 `User` 对象。

因此：

```text
resultType
```

表示的是：

> 查询返回的单条记录所对应的 Java 类型。

---

# 14. resultType 与 List 的关系

这里有一个容易混淆的问题。

Mapper 方法：

```java
List<User> findAll();
```

返回的是：

```java
List<User>
```

但是 XML 中写的是：

```xml
resultType="com.huaishi.pojo.User"
```

而不是：

```text
List<User>
```

原因是：

> `resultType` 描述的是单条查询记录封装成什么类型。

例如数据库查询返回 5 条用户记录。

每一条记录分别被封装成：

```java
User
```

最终 MyBatis 再将这些对象组成：

```java
List<User>
```

因此：

```java
List<User> findAll();
```

对应：

```xml
resultType="com.huaishi.pojo.User"
```

是正确的。

---

# 15. 查询结果的封装过程

假设 SQL：

```sql
select * from user;
```

查询出三条记录。

Mapper：

```java
List<User> findAll();
```

XML：

```xml
<select id="findAll"
        resultType="com.huaishi.pojo.User">
    select * from user
</select>
```

MyBatis 会：

1. 执行 SQL。
    
2. 获取数据库返回的三条记录。
    
3. 将第一条记录封装成一个 `User`。
    
4. 将第二条记录封装成一个 `User`。
    
5. 将第三条记录封装成一个 `User`。
    
6. 最终将三个 `User` 放入 `List`。
    
7. 返回 `List<User>`。
    

因此 `resultType` 关注的是：

> 单条数据的类型。

Mapper 返回值关注的是：

> 整个查询结果最终返回给 Java 程序的类型。

---

# 16. XML 映射完整示例

Mapper 接口：

```java
@Mapper
public interface UserMapper {

    /**
     * 查询全部用户
     */
    List<User> findAll();
}
```

对应 XML：

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.huaishi.mapper.UserMapper">

    <!-- 查询所有用户 -->
    <select id="findAll"
            resultType="com.huaishi.pojo.User">

        select * from user

    </select>

</mapper>
```

调用：

```java
List<User> users = userMapper.findAll();
```

MyBatis 会找到：

```xml
<select id="findAll">
```

并执行其中的：

```sql
select * from user;
```

之后将每条结果封装为：

```java
User
```

最终返回：

```java
List<User>
```

---

# 17. namespace、id、resultType 的区别

这三个属性非常重要，也很容易混淆。

|配置|作用|
|---|---|
|`namespace`|确定 XML 对应哪个 Mapper 接口|
|`id`|确定 SQL 对应 Mapper 中哪个方法|
|`resultType`|确定查询得到的单条记录封装成什么 Java 类型|

例如：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">

    <select id="findAll"
            resultType="com.huaishi.pojo.User">

        select * from user

    </select>

</mapper>
```

其中：

```text
namespace
```

对应：

```java
UserMapper
```

`id`：

```text
findAll
```

对应：

```java
findAll()
```

`resultType`：

```text
com.huaishi.pojo.User
```

对应：

```java
User
```

对象。

---

# 18. XML 映射的匹配规则

使用 XML 映射时，可以重点记住以下三个对应关系。

## 18.1 Mapper 接口 ↔ XML 文件

例如：

```text
UserMapper.java
```

对应：

```text
UserMapper.xml
```

即：

> 同名。

---

## 18.2 Mapper 全限定名 ↔ namespace

Mapper：

```text
com.huaishi.mapper.UserMapper
```

对应：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">
```

即：

> `namespace` 与 Mapper 接口全限定名一致。

---

## 18.3 Mapper 方法名 ↔ SQL id

Mapper：

```java
List<User> findAll();
```

对应：

```xml
<select id="findAll">
```

即：

> SQL 标签的 `id` 与 Mapper 方法名一致。

这些规则是 XML 映射能够正常工作的基础。

---

# 19. XML 与注解方式的区别

注解方式：

```java
@Select("select * from user")
List<User> findAll();
```

特点是：

> SQL 直接写在 Mapper 接口中。

XML 方式：

```java
List<User> findAll();
```

SQL 放在：

```xml
<select id="findAll">
    select * from user
</select>
```

特点是：

> Mapper 方法与 SQL 分离。

因此可以简单理解为：

|对比|注解|XML|
|---|---|---|
|SQL 位置|Mapper 接口中|XML 文件中|
|简单 SQL|比较方便|可以使用|
|复杂 SQL|容易变长|更适合|
|Java 与 SQL|写在一起|分离|

当前阶段重点理解：

> 两种方式最终完成的是同一件事，只是 SQL 的配置位置不同。

---

# 20. 一个 Mapper 方法不能同时配置两份 SQL

对于同一个 Mapper 方法：

```java
List<User> findAll();
```

不能同时：

```java
@Select("select * from user")
List<User> findAll();
```

又在 XML 中写：

```xml
<select id="findAll"
        resultType="com.huaishi.pojo.User">

    select * from user

</select>
```

也就是说：

> 一个 Mapper 方法对应的 SQL，要么使用注解配置，要么使用 XML 配置，不应该同时配置。

否则同一个方法会出现两份 SQL 映射定义。

---

# 21. XML 中的 DTD

一个 MyBatis XML 文件顶部通常包含：

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
```

这一部分属于 XML 文件的约束声明。

在当前阶段，不需要重点记忆其中每一段内容。

使用时保持正确的 MyBatis Mapper XML 结构即可。

真正需要重点掌握的是后面的：

```xml
<mapper namespace="...">
```

以及其中的 SQL 映射配置。

---

# 22. XML 映射中真正需要关注什么

刚开始学习 XML 映射时，不需要把重点放在记忆整份 XML 模板。

真正需要掌握的是下面四个问题。

## 22.1 这个 XML 属于谁

看：

```xml
namespace
```

例如：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">
```

表示属于：

```java
UserMapper
```

---

## 22.2 这条 SQL 对应哪个方法

看：

```xml
id
```

例如：

```xml
<select id="findAll">
```

对应：

```java
findAll()
```

---

## 22.3 这是什么 SQL

看 SQL 标签。

例如：

```xml
<select>
```

表示查询。

---

## 22.4 查询结果封装成什么类型

看：

```xml
resultType
```

例如：

```xml
resultType="com.huaishi.pojo.User"
```

表示：

> 单条查询记录封装成 `User`。

---

# 23. XML 映射的整体执行过程

假设 Mapper：

```java
@Mapper
public interface UserMapper {

    List<User> findAll();
}
```

XML：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">

    <select id="findAll"
            resultType="com.huaishi.pojo.User">

        select * from user

    </select>

</mapper>
```

程序调用：

```java
userMapper.findAll();
```

整个过程可以理解为：

1. 程序调用 `UserMapper.findAll()`。
    
2. MyBatis 根据 Mapper 接口确定对应的 XML。
    
3. 根据 `namespace` 确认 XML 属于 `UserMapper`。
    
4. 根据方法名 `findAll` 查找 `id="findAll"`。
    
5. 找到对应的 `<select>` SQL。
    
6. 执行 `select * from user`。
    
7. 数据库返回查询结果。
    
8. 根据 `resultType`，将每条记录封装成 `User`。
    
9. 最终组成 `List<User>`。
    
10. 返回给调用者。
    

因此 XML 映射的核心目的就是：

> 建立 Mapper 方法、SQL 和查询结果类型之间的对应关系。

---

# 24. XML 映射的核心配置

一个查询映射最核心的代码其实可以浓缩为：

```xml
<mapper namespace="Mapper接口全限定名">

    <select id="Mapper方法名"
            resultType="单条记录对应的Java类型">

        SQL语句

    </select>

</mapper>
```

例如：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">

    <select id="findAll"
            resultType="com.huaishi.pojo.User">

        select * from user

    </select>

</mapper>
```

只要能够理解其中：

```text
namespace
id
resultType
```

三者的职责，就掌握了 XML 映射最基础的工作方式。

---

# 25. 常见错误

## 25.1 namespace 写错

错误：

```xml
<mapper namespace="com.huaishi.pojo.User">
```

但真正的 Mapper 是：

```java
com.huaishi.mapper.UserMapper
```

这样就无法建立正确的 Mapper 对应关系。

应该写：

```xml
<mapper namespace="com.huaishi.mapper.UserMapper">
```

---

## 25.2 id 与方法名不一致

Mapper：

```java
List<User> findAll();
```

XML：

```xml
<select id="list">
```

这里：

```text
findAll
```

和：

```text
list
```

不一致。

应该保持：

```xml
<select id="findAll">
```

---

## 25.3 resultType 理解成集合类型

Mapper 返回：

```java
List<User>
```

不要因此把 `resultType` 理解成 `List<User>`。

应该写单条记录对应的：

```xml
resultType="com.huaishi.pojo.User"
```

因为 `resultType` 描述的是：

> 单条查询记录的封装类型。

---

## 25.4 同一个方法同时使用注解和 XML

不要同时写：

```java
@Select("select * from user")
List<User> findAll();
```

以及：

```xml
<select id="findAll">
    select * from user
</select>
```

同一个方法只选择其中一种 SQL 配置方式。

---

# 26. 学习重点

## 26.1 基础

需要掌握：

- MyBatis 有注解和 XML 两种 SQL 配置方式。
    
- 为什么复杂 SQL 更适合放在 XML 中。
    
- XML 映射文件的作用。
    
- `<mapper>` 标签的作用。
    
- `<select>` 标签的作用。
    
- `namespace` 的作用。
    
- `id` 的作用。
    
- `resultType` 的作用。
    

## 26.2 重点

需要重点理解：

- `namespace` 为什么必须对应 Mapper 接口全限定名。
    
- SQL 标签的 `id` 为什么必须与 Mapper 方法名一致。
    
- `resultType` 为什么表示单条记录的类型，而不是整个集合类型。
    
- Mapper 方法是如何找到 XML 中对应 SQL 的。
    
- 为什么一个 Mapper 方法不能同时使用注解和 XML 配置 SQL。
    

---

> [!tip] 一句话总结
> 
> MyBatis XML 映射通过 `namespace` 将 XML 与 Mapper 接口绑定，通过 SQL 标签的 `id` 将 SQL 与 Mapper 方法绑定，再通过 `resultType` 指定查询返回的单条记录所封装的 Java 类型，从而建立 Mapper 方法、SQL 语句和查询结果之间的映射关系。