# 1. 为什么需要分页

在实际项目中，查询的数据量可能非常大。

例如：

- 用户列表。
    
- 员工列表。
    
- 订单列表。
    
- 商品列表。
    
- 支付记录。
    

如果一次性查询并返回全部数据：

> 数据量越大，查询、传输和前端展示的压力就越大。

因此通常会采用分页查询。

分页查询的核心目标是：

> 每次只查询指定页的数据，而不是一次性查询全部记录。

例如：

```text
第 1 页，每页 10 条
```

或者：

```text
第 3 页，每页 20 条
```

---

# 2. 原始分页查询

在不使用 PageHelper 时，一个分页功能通常需要查询两部分数据：

1. 数据库中的总记录数。
    
2. 当前页的数据列表。
    

因此 Mapper 层通常需要分别执行两条 SQL。

---

# 3. 查询总记录数

第一条 SQL 用于查询：

> 当前数据一共有多少条记录。

例如：

```sql
select count(*)
from emp;
```

查询结果可能是：

```text
102
```

表示总共有：

```text
102 条员工记录
```

这个值通常用于前端计算：

- 总页数。
    
- 分页导航。
    
- 是否还有下一页。
    

---

# 4. 查询当前页的数据

另一条 SQL 用于：

> 查询指定页真正需要展示的数据。

MySQL 中通常需要使用：

```sql
limit
```

例如：

```sql
select *
from emp
limit 0, 10;
```

表示查询：

> 从第 0 条开始的 10 条数据。

---

# 5. 原始分页存在的问题

原始分页虽然可以正常实现，但是整体步骤非常固定。

通常需要：

1. 查询总记录数。
    
2. 根据页码计算起始索引。
    
3. 查询当前页的数据。
    
4. 将总记录数和数据列表进行封装。
    

因此每次实现用户分页、订单分页、商品分页时，都会重复类似代码。

这种方式存在两个比较明显的问题：

- 开发步骤固定。
    
- 重复代码较多。
    

因此分页查询非常适合通过通用插件统一处理。

---

# 6. PageHelper 是什么

PageHelper 是：

> MyBatis 中常用的第三方分页插件。

它可以帮助开发者自动处理分页查询。

当前内容中将其描述为功能强大、使用方便的 MyBatis 分页插件，并支持单表和多表分页查询。

它的核心价值是：

> 开发者只需要写普通查询 SQL，由 PageHelper 自动完成分页相关处理。

---

# 7. PageHelper 解决了什么问题

使用 PageHelper 后，不需要再手动：

- 编写统计总数的 SQL。
    
- 编写分页 `limit` SQL。
    
- 计算分页起始索引。
    

开发者主要需要提供：

```text
查询第几页
```

以及：

```text
每页多少条
```

PageHelper 会帮助完成分页查询。

---

# 8. 原始分页与 PageHelper 的区别

## 8.1 Mapper 层

原始分页：

> 通常需要编写两条查询。

分别用于：

- 查询总数。
    
- 查询数据列表。
    

使用 PageHelper 后：

> Mapper 中只需要编写正常的列表查询。

例如：

```java
@Select("""
    select e.*, d.name deptName
    from emp e
    left join dept d
        on e.dept_id = d.id
""")
List<Emp> list();
```

Mapper 不需要自己考虑：

```sql
count(...)
```

也不需要自己写：

```sql
limit
```

---

## 8.2 Service 层

原始分页通常需要自己计算起始索引。

例如：

```text
起始索引 = (page - 1) × pageSize
```

而 PageHelper 不需要手动计算。

只需要告诉 PageHelper：

```text
page
pageSize
```

即可。

---

# 9. 引入 PageHelper

在 Spring Boot 项目中，可以通过 Maven 引入 PageHelper Starter：

```xml
<!-- PageHelper分页插件 -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.4.7</version>
</dependency>
```

引入依赖后，就可以在项目中使用 PageHelper 提供的分页功能。

---

# 10. Mapper 不再负责分页

使用 PageHelper 后，Mapper 中只需要：

> 正常查询数据。

例如：

```java
@Select("""
    select e.*, d.name deptName
    from emp as e
    left join dept as d
        on e.dept_id = d.id
""")
List<Emp> list();
```

这里没有：

```sql
limit
```

也没有：

```sql
count(*)
```

说明 Mapper 并没有自己编写分页逻辑。

分页操作由 PageHelper 自动处理。

---

# 11. PageHelper.startPage()

分页查询前，需要设置分页参数：

```java
PageHelper.startPage(page, pageSize);
```

例如：

```java
PageHelper.startPage(1, 10);
```

表示：

> 查询第 1 页，每页展示 10 条数据。

其中：

```text
page
```

表示页码。

```text
pageSize
```

表示每页显示的记录数量。

---

# 12. startPage() 的作用

`PageHelper.startPage()` 的作用可以理解为：

> 告诉 PageHelper，紧接着执行的查询应该按照什么分页条件执行。

例如：

```java
PageHelper.startPage(page, pageSize);

List<Emp> empList = empMapper.list();
```

PageHelper 会根据：

```java
page
```

和：

```java
pageSize
```

自动对后面的查询进行分页处理。

---

# 13. startPage() 必须在查询前调用

正确顺序：

```java
PageHelper.startPage(page, pageSize);

List<Emp> empList = empMapper.list();
```

也就是：

> 先设置分页参数，再执行 Mapper 查询。

如果顺序反过来：

```java
List<Emp> empList = empMapper.list();

PageHelper.startPage(page, pageSize);
```

前面的查询已经执行完成，就不能再对它进行分页。

因此基本使用模式必须是：

```text
设置分页参数
↓
执行查询
↓
获取分页结果
```

---

# 14. PageHelper 只处理紧跟着的查询

使用：

```java
PageHelper.startPage(page, pageSize);
```

之后，PageHelper：

> 只会对紧跟在后面的第一条 SQL 进行分页处理。

因此：

```java
PageHelper.startPage(page, pageSize);

List<Emp> empList = empMapper.list();
```

这里真正受到分页影响的是：

```java
empMapper.list();
```

这一点在使用时非常重要。

---

# 15. Service 中的完整分页代码

一个基础的分页实现：

```java
@Override
public PageResult page(Integer page, Integer pageSize) {

    // 1. 设置分页参数
    PageHelper.startPage(page, pageSize);

    // 2. 执行查询
    List<Emp> empList = empMapper.list();

    Page<Emp> p = (Page<Emp>) empList;

    // 3. 封装分页结果
    return new PageResult(
        p.getTotal(),
        p.getResult()
    );
}
```

这个过程可以划分为三个步骤：

1. 设置分页参数。
    
2. 执行正常查询。
    
3. 获取并封装分页结果。
    

---

# 16. Page

执行分页查询后：

```java
List<Emp> empList = empMapper.list();
```

可以转换为：

```java
Page<Emp> p = (Page<Emp>) empList;
```

这里：

```java
Page<Emp>
```

包含的不只是当前页的数据。

还包含分页相关信息。

例如：

```java
p.getTotal();
```

可以获得总记录数。

```java
p.getResult();
```

可以获得当前页的数据列表。

---

# 17. getTotal()

```java
p.getTotal();
```

表示：

> 获取满足查询条件的总记录数量。

例如：

```text
total = 100
```

表示数据库中一共有：

```text
100 条符合条件的数据
```

这个值并不是：

> 当前页有多少条。

而是：

> 所有分页数据一共有多少条。

---

# 18. getResult()

```java
p.getResult();
```

用于：

> 获取当前页真正查询出来的数据列表。

例如设置：

```java
PageHelper.startPage(2, 10);
```

那么：

```java
p.getResult();
```

得到的就是：

> 第 2 页对应的数据。

因此可以区分：

```text
getTotal()
```

负责总数量，

```text
getResult()
```

负责当前页数据。

---

# 19. PageResult

分页接口一般不仅需要返回当前页数据，还需要返回总记录数。

因此可以将分页结果封装成：

```java
PageResult
```

例如：

```java
return new PageResult(
    p.getTotal(),
    p.getResult()
);
```

也就是说：

```text
PageResult
```

主要包含：

- 总记录数。
    
- 当前页数据列表。
    

---

# 20. 为什么需要总记录数

前端分页组件通常不仅要显示当前页的数据，还需要知道：

> 数据总共有多少条。

例如：

```text
总记录数：53
每页：10 条
```

前端就可以进一步计算：

```text
总页数：6 页
```

因此分页接口通常需要同时返回：

```text
total
```

和：

```text
rows
```

之类的数据。

---

# 21. 分页请求参数

例如客户端请求：

```text
GET /emps?page=1&pageSize=5
```

其中：

```text
page=1
```

表示：

> 查询第 1 页。

```text
pageSize=5
```

表示：

> 每页查询 5 条记录。

然后 Service 中可以将这两个参数传递给：

```java
PageHelper.startPage(page, pageSize);
```

当前示例也是使用 `page=1&pageSize=5` 验证分页查询。

---

# 22. PageHelper 的实现机制

使用 PageHelper 后，Mapper 中虽然只写了一条普通 SQL：

```sql
select e.*, d.name deptName
from emp e
left join dept d
    on e.dept_id = d.id
```

但是分页查询执行时，实际上会产生：

> 两条 SQL。

分别负责：

1. 查询总记录数。
    
2. 查询当前页的数据。
    

---

# 23. 第一条 SQL：查询总记录数

PageHelper 会对原始查询 SQL 进行改造。

原来的查询：

```sql
select e.*, d.name deptName
from emp e
left join dept d
    on e.dept_id = d.id;
```

分页时会生成用于统计总数的查询。

核心思想类似：

```sql
select count(0)
from ...
```

也就是说：

> PageHelper 自动把原来的查询增强成统计总记录数的 SQL。

当前内容明确说明，它会将查询返回字段改造成 `count(0)` 来获取总记录数。

---

# 24. 第二条 SQL：查询分页数据

第二条 SQL 用于：

> 获取当前页真正需要的数据。

PageHelper 会在原有查询 SQL 的基础上增加：

```sql
limit
```

进行分页。

也就是说，开发者没有手动写：

```sql
limit
```

但是 PageHelper 会帮助生成对应的分页 SQL。

---

# 25. PageHelper 本质上做了什么

从当前实现机制来看，PageHelper 的核心作用可以理解为：

> 对 Mapper 原本要执行的查询 SQL 进行分页增强。

开发者编写：

```sql
select ...
from ...
```

PageHelper 会进一步生成：

```text
统计总记录数的 SQL
```

以及：

```text
带 limit 的分页查询 SQL
```

然后把执行结果封装到分页对象中。

---

# 26. PageHelper 并不是不需要两条 SQL

这是一个容易产生的误解。

使用 PageHelper 后：

> 开发者不需要自己写两条 SQL。

但 PageHelper 底层仍然会执行：

1. 总记录数查询。
    
2. 分页数据查询。
    

区别在于：

> 这两条分页 SQL 由 PageHelper 根据原始 SQL 自动生成，而不是由开发者手动维护。

因此它减少的是：

> 重复的分页开发代码。

---

# 27. PageHelper 与 Mapper 的职责

使用 PageHelper 后，可以进一步区分职责。

## 27.1 Mapper

负责：

> 描述真正要查询什么业务数据。

例如：

```java
List<Emp> list();
```

SQL：

```sql
select e.*, d.name deptName
from emp e
left join dept d
    on e.dept_id = d.id
```

---

## 27.2 PageHelper

负责：

> 给正常查询增加分页能力。

包括：

- 统计总记录数。
    
- 添加分页限制。
    
- 封装分页相关结果。
    

所以 Mapper 不再需要负责分页计算。

---

# 28. PageHelper 与 Service 的职责

Service 主要负责：

1. 接收分页参数。
    
2. 调用 `PageHelper.startPage()`。
    
3. 调用 Mapper 查询数据。
    
4. 获取分页结果。
    
5. 封装成项目统一的分页返回对象。
    

例如：

```java
public PageResult page(Integer page, Integer pageSize) {

    PageHelper.startPage(page, pageSize);

    List<Emp> empList = empMapper.list();

    Page<Emp> p = (Page<Emp>) empList;

    return new PageResult(
        p.getTotal(),
        p.getResult()
    );
}
```

---

# 29. PageHelper 的基本使用流程

可以把 PageHelper 的使用总结为四步。

## 29.1 引入依赖

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.4.7</version>
</dependency>
```

---

## 29.2 Mapper 正常查询

```java
List<Emp> list();
```

不需要手写分页 SQL。

---

## 29.3 查询前设置分页参数

```java
PageHelper.startPage(page, pageSize);
```

---

## 29.4 获取分页结果

```java
List<Emp> empList = empMapper.list();

Page<Emp> p = (Page<Emp>) empList;

long total = p.getTotal();

List<Emp> rows = p.getResult();
```

最后封装成项目需要的返回类型。

---

# 30. PageHelper 中最重要的执行顺序

需要重点记住：

```java
PageHelper.startPage(page, pageSize);

List<Emp> empList = empMapper.list();
```

而不是：

```java
List<Emp> empList = empMapper.list();

PageHelper.startPage(page, pageSize);
```

原因是：

> PageHelper 需要在 SQL 执行之前知道分页参数，并且只处理后面紧跟着执行的第一条 SQL。

---

# 31. SQL 末尾不要添加分号

使用 PageHelper 时，当前内容特别强调：

> SQL 语句结尾不要添加 `;`。

因为 PageHelper 需要对原 SQL 进行进一步改造。

例如在原 SQL 后面添加：

```sql
limit ...
```

如果 SQL 已经提前用：

```text
;
```

结束，就可能影响 PageHelper 对 SQL 的增强处理。

因此 Mapper 中建议写：

```java
@Select("select * from emp")
```

而不要写：

```java
@Select("select * from emp;")
```

---

# 32. PageHelper 只作用于第一条 SQL

例如：

```java
PageHelper.startPage(page, pageSize);

List<Emp> empList = empMapper.list();

List<Dept> deptList = deptMapper.list();
```

PageHelper 的分页处理作用在：

```java
empMapper.list();
```

因为这是：

> `startPage()` 后紧跟着执行的第一条 SQL。

后面的：

```java
deptMapper.list();
```

不会继续使用同一次分页设置。

---

# 33. PageHelper 与手动分页的代码差异

原始分页通常需要类似：

```java
Long total = empMapper.count();

Integer start = (page - 1) * pageSize;

List<Emp> rows =
    empMapper.page(start, pageSize);

return new PageResult(total, rows);
```

而 PageHelper 可以简化为：

```java
PageHelper.startPage(page, pageSize);

List<Emp> empList = empMapper.list();

Page<Emp> p = (Page<Emp>) empList;

return new PageResult(
    p.getTotal(),
    p.getResult()
);
```

最大的变化是：

> 不再手动处理分页 SQL 和起始索引。

---

# 34. PageHelper 的核心优势

## 34.1 减少重复 SQL

不需要反复编写：

```sql
count(...)
```

以及：

```sql
limit ...
```

---

## 34.2 Mapper 更关注业务查询

Mapper 只需要关注：

> 我要查询哪些数据。

而不是：

> 分页应该怎样计算。

---

## 34.3 Service 不需要计算起始索引

不再需要手动：

```java
(page - 1) * pageSize
```

只需要：

```java
PageHelper.startPage(page, pageSize);
```

---

## 34.4 分页实现更加统一

用户、员工、订单、商品等分页查询，都可以采用相似的使用模式。

因此分页逻辑更容易统一管理。

---

# 35. PageHelper 中几个容易混淆的对象

## 35.1 page

```java
Integer page
```

表示：

> 页码。

例如：

```text
1
2
3
```

---

## 35.2 pageSize

```java
Integer pageSize
```

表示：

> 每页显示多少条数据。

例如：

```text
10
20
50
```

---

## 35.3 PageHelper

```java
PageHelper
```

负责：

> 设置并触发分页处理。

核心方法：

```java
PageHelper.startPage(page, pageSize);
```

---

## 35.4 Page

```java
Page<Emp>
```

负责：

> 保存 PageHelper 查询后的分页结果。

其中可以获取：

```java
getTotal()
getResult()
```

---

## 35.5 PageResult

```java
PageResult
```

是项目自己使用的：

> 分页结果返回对象。

它与 PageHelper 提供的 `Page<T>` 不是同一个概念。

---

# 36. Page 与 PageResult 的区别

这是一个容易混淆的地方。

## 36.1 Page

PageHelper 分页查询产生的分页对象。

例如：

```java
Page<Emp> p;
```

可以获取：

```java
p.getTotal();
p.getResult();
```

---

## 36.2 PageResult

项目自主定义的返回对象。

例如：

```java
return new PageResult(
    p.getTotal(),
    p.getResult()
);
```

也就是说：

> `Page<T>` 是分页插件内部使用的结果对象，而 `PageResult` 是项目对外返回分页数据时使用的对象。

---

# 37. PageHelper 的完整执行过程

假设客户端请求：

```text
GET /emps?page=2&pageSize=10
```

Service 获取：

```text
page = 2
pageSize = 10
```

然后执行：

```java
PageHelper.startPage(2, 10);
```

接下来调用：

```java
empMapper.list();
```

PageHelper 对查询进行处理。

第一步：

> 自动执行总记录数查询。

第二步：

> 自动增加分页条件，查询第 2 页的数据。

第三步：

> 将总记录数和当前页数据封装到 `Page<Emp>` 中。

第四步：

Service 获取：

```java
p.getTotal();
p.getResult();
```

第五步：

封装为：

```java
PageResult
```

返回给 Controller。

---

# 38. 学习 PageHelper 时真正需要理解什么

PageHelper 最重要的不是记住依赖版本，而是理解它改变了：

> 分页职责的分配方式。

原来是：

```text
Mapper
负责 count SQL
负责 limit SQL

Service
负责计算起始索引
负责组合分页结果
```

使用 PageHelper 后：

```text
Mapper
只负责正常业务查询

PageHelper
负责分页 SQL 增强

Service
负责设置分页参数
负责封装分页结果
```

这才是使用分页插件后最大的变化。

---

# 39. 常见错误

## 39.1 在 Mapper 中继续自己写 limit

既然已经使用：

```java
PageHelper.startPage(...)
```

Mapper 应该保持正常查询。

不要重复手动添加分页逻辑。

---

## 39.2 在查询之后调用 startPage()

错误：

```java
List<Emp> empList = empMapper.list();

PageHelper.startPage(page, pageSize);
```

应该先设置：

```java
PageHelper.startPage(page, pageSize);
```

再执行查询。

---

## 39.3 SQL 末尾添加分号

例如：

```java
@Select("select * from emp;")
```

当前 PageHelper 使用规范中不应该这样写。

应该：

```java
@Select("select * from emp")
```

---

## 39.4 认为 PageHelper 不会执行 count 查询

错误。

PageHelper 只是：

> 不需要开发者自己写 count SQL。

它分页时仍然需要获取总记录数，因此会自动生成并执行相应 SQL。

---

## 39.5 混淆 Page 与 PageResult

```java
Page<Emp>
```

属于分页插件查询结果。

```java
PageResult
```

属于项目自己的分页返回模型。

二者职责不同。

---

# 40. 学习重点

## 40.1 基础

需要掌握：

- PageHelper 是什么。
    
- 为什么需要分页插件。
    
- 原始分页存在哪些重复代码。
    
- 如何引入 PageHelper。
    
- `PageHelper.startPage()` 的作用。
    
- `page` 和 `pageSize` 分别代表什么。
    

## 40.2 分页结果

需要掌握：

- `Page<T>` 的作用。
    
- `getTotal()` 的作用。
    
- `getResult()` 的作用。
    
- `Page<T>` 与 `PageResult` 的区别。
    

## 40.3 实现机制

需要重点理解：

- PageHelper 为什么可以让 Mapper 不写分页 SQL。
    
- PageHelper 会自动执行总数查询和分页数据查询。
    
- PageHelper 如何在原 SQL 基础上生成 `count(0)`。
    
- PageHelper 如何通过 `limit` 实现分页。
    
- 为什么 `startPage()` 必须写在 Mapper 查询之前。
    
- 为什么它只处理后面紧跟的第一条 SQL。
    
- 为什么 SQL 结尾不要添加分号。
    

---

> [!tip] 一句话总结
> 
> `PageHelper` 是 MyBatis 的分页插件，开发者只需要在查询之前调用 `PageHelper.startPage(page, pageSize)` 并执行普通 Mapper 查询，PageHelper 就会自动生成统计总记录数和带 `limit` 的分页 SQL，再通过 `Page<T>` 提供总记录数和当前页数据，从而省去手动编写分页 SQL 和计算起始索引的重复工作。