# 1. 测试基础

## 1.1 什么是测试

测试是用于检验软件**正确性、完整性、安全性和质量**的过程。

从测试对象和测试阶段来看，常见测试可以划分为：

|测试类型|主要对象|核心目的|主要参与者|
|---|---|---|---|
|单元测试|基本组成单位|验证单个功能单元是否正确|开发人员|
|集成测试|多个已测试单元|验证单元之间协作是否正确|开发人员|
|系统测试|完整软件系统|验证系统功能、性能等是否满足要求|测试人员|
|验收测试|面向用户需求的完整系统|验证是否满足验收标准|客户/需求方|

**核心区别：**

- 单元测试关注“一个功能单元是否正确”
- 集成测试关注“多个单元组合后是否正确”
- 系统测试关注“整个系统是否正确”
- 验收测试关注“系统是否满足用户和业务要求”

---

# 2. 测试方法

根据测试人员对程序内部实现的了解程度，可以分为：

|测试方法|是否关注内部实现|主要关注点|
|---|---|---|
|白盒测试|是|代码结构、执行逻辑|
|黑盒测试|否|功能、输入输出、兼容性|
|灰盒测试|部分关注|内部结构 + 外部行为|

## 白盒测试

测试人员了解程序内部结构和代码逻辑，主要用于验证代码和逻辑的正确性。

## 黑盒测试

测试人员不关注程序内部实现，而是从外部观察输入和输出，主要用于功能、兼容性以及验收等测试。

## 灰盒测试

介于白盒和黑盒之间，同时考虑程序内部结构和外部功能表现。

> **注意：测试阶段和测试方法是两个不同维度。**
> 
> 例如：单元测试属于测试阶段，而白盒测试属于测试方法，两者不是同一层面的概念。

---

# 3. JUnit

## 3.1 什么是单元测试

单元测试针对程序中的**最小功能单元**进行测试。在本课程中主要以**方法**作为测试单元。

例如：

```java
public Integer getAge(String idCard) {
    // ...
}
```

可以针对 `getAge()` 编写单独的测试代码，验证该方法是否得到预期结果。

---

## 3.2 什么是 JUnit

JUnit 是 Java 中常用的单元测试框架，用于辅助开发人员编写和执行单元测试。

相比直接使用 `main()` 方法测试，JUnit 将**测试代码与业务代码分离**，并提供测试生命周期管理、断言、参数化测试以及测试结果分析等能力。

### 使用 main 方法测试的问题

传统方式通常直接在 `main()` 中调用业务代码：

- 测试代码和源代码混在一起，不利于维护
- 一个测试失败可能影响后续测试
- 缺乏规范化的自动化测试机制
- 不方便统一分析测试结果

JUnit 解决的核心问题是：

> **让测试成为独立、规范、可重复执行的代码。**

---

# 4. JUnit 基本使用

Maven 项目中通常将测试代码放在：

```text
src
├── main
│   └── java
└── test
    └── java
```

`test/java` 用于存放测试代码，`main/java` 用于存放正式程序代码。

JUnit 测试依赖通常声明在 `pom.xml` 中：

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.9.1</version>
    <scope>test</scope>
</dependency>
```

其中：

```xml
<scope>test</scope>
```

表示该依赖只用于测试环境。

---

## 4.1 测试类与测试方法

测试类通常按照：

```text
XxxxTest
```

进行命名。

测试方法使用 `@Test` 标记：

```java
@Test
public void testGetAge() {
    Integer age = new UserService().getAge("110002200505091218");
}
```

`@Test` 的作用是告诉 JUnit：

> 这个方法是一个测试方法，可以被测试框架执行。

因此，JUnit 测试代码的基本结构可以理解为：

```text
测试类
├── 测试方法
├── 测试方法
└── 测试方法
```

不需要使用大量流程图描述，因为 JUnit 的核心并不是某个固定执行流程，而是**通过注解定义测试代码的角色**。

---

# 5. 断言 Assertions

## 5.1 为什么需要断言

单纯执行方法并打印结果：

```java
System.out.println(age);
```

只能看到结果，无法让测试框架自动判断：

> **实际结果是否符合预期？**

JUnit 提供 **Assertions（断言）** 来解决这个问题。

断言本质上是在测试中定义一个**预期条件**：

> 如果实际结果不满足预期，测试失败。

---

## 5.2 常见断言

|断言|作用|
|---|---|
|`assertEquals(expected, actual)`|判断两个值是否相等|
|`assertNotEquals(unexpected, actual)`|判断两个值是否不相等|
|`assertNull(actual)`|判断对象是否为 `null`|
|`assertNotNull(actual)`|判断对象是否不为 `null`|
|`assertTrue(condition)`|判断条件是否为 `true`|
|`assertFalse(condition)`|判断条件是否为 `false`|
|`assertSame(expected, actual)`|判断两个对象引用是否相同|

例如：

```java
@Test
public void testGetGender() {
    String gender = new UserService()
            .getGender("612429198904201611");

    Assertions.assertEquals("男", gender);
}
```

这里真正具有测试意义的是：

```java
Assertions.assertEquals("男", gender);
```

因为它明确规定了：

**预期结果 = `"男"`**

而不是简单地观察控制台输出。

---

## 5.3 `assertEquals` 与 `assertSame`

这两个断言容易混淆：

- `assertEquals`：比较两个值是否相等
- `assertSame`：比较两个对象是否是**同一个引用**

例如：

```java
String s1 = new String("Hello");
String s2 = "Hello";
```

两个字符串的内容可能相同，但它们不一定是同一个对象。

因此：

```java
assertEquals(s1, s2);
```

关注的是**值**。

而：

```java
assertSame(s1, s2);
```

关注的是**对象引用**。

---

# 6. JUnit 常用注解

JUnit 的重要特征之一是通过**注解描述测试代码的生命周期和用途**。

|注解|作用|执行特点|
|---|---|---|
|`@Test`|声明测试方法|测试方法的基本注解|
|`@BeforeEach`|测试前置操作|每个测试方法执行前执行一次|
|`@AfterEach`|测试后置操作|每个测试方法执行后执行一次|
|`@BeforeAll`|全局前置操作|所有测试方法执行前执行一次|
|`@AfterAll`|全局后置操作|所有测试方法执行后执行一次|
|`@ParameterizedTest`|参数化测试|同一个测试逻辑使用不同参数执行|
|`@ValueSource`|参数来源|为参数化测试提供参数|
|`@DisplayName`|自定义显示名称|改善测试结果的可读性|

---

## 6.1 `@BeforeEach` / `@AfterEach`

适合放置**每个测试都需要执行的准备和清理工作**。

```java
@BeforeEach
public void testBefore() {
    System.out.println("before...");
}

@AfterEach
public void testAfter() {
    System.out.println("after...");
}
```

典型用途：

- 初始化测试数据
- 创建测试对象
- 清理测试环境
- 释放测试资源

---

## 6.2 `@BeforeAll` / `@AfterAll`

适合放置**整个测试类只需要执行一次**的初始化和清理工作。

```java
@BeforeAll
public static void testBeforeAll() {
    System.out.println("before all...");
}

@AfterAll
public static void testAfterAll() {
    System.out.println("after all...");
}
```

十分重要的一点是：

> `@BeforeAll` 和 `@AfterAll` 修饰的方法必须使用 `static`。

---

## 6.3 参数化测试

普通测试往往需要针对多个输入重复编写类似代码。

例如：

```java
@Test
public void testGetGender() {
    // 测试一个身份证号
}
```

如果需要测试多个身份证号，就可以使用参数化测试。

```java
@ParameterizedTest
@ValueSource(strings = {
    "612429198904201611",
    "612429198904201631",
    "612429198904201626"
})
public void testGetGender(String idcard) {
    String gender = new UserService().getGender(idcard);
}
```

这里：

- `@ParameterizedTest`：声明这是参数化测试
- `@ValueSource`：提供测试参数
- 测试方法通过参数接收每次测试的数据

因此参数化测试适合：

> **测试逻辑相同、输入数据不同的场景。**

使用 `@ParameterizedTest` 后，不需要再同时使用 `@Test`。

---

## 6.4 `@DisplayName`

用于给测试类或测试方法指定更加直观的名称：

```java
@DisplayName("测试-获取年龄")
@Test
public void testGetAge() {
    // ...
}
```

它主要改善测试结果的**可读性**，不会改变测试逻辑。

---

# 7. 测试代码的位置

Maven 项目中：

```text
src/main/java
```

用于正式业务代码。

```text
src/test/java
```

用于测试代码。

技术上，测试代码可以写在 `main` 中，但这种做法不规范。

将测试代码放在 `test` 目录的意义不仅是目录分类，更重要的是 Maven 可以根据不同目录和依赖范围区分：

- 正式代码
- 测试代码
- 正式运行环境
- 测试运行环境

因此，`src/test/java` 是 Maven 项目中约定俗成的测试代码位置。

---

# 8. Maven 依赖范围 Scope

JUnit 这一部分同时引出了 Maven 的另一个重要概念：**依赖范围（scope）**。

在 `pom.xml` 中：

```xml
<scope>test</scope>
```

用于限制依赖的使用范围。

依赖范围主要影响三个维度：

1. 主程序是否可以使用
2. 测试程序是否可以使用
3. 是否参与最终运行环境

在 Maven 的 scope 中的常见范围：

|Scope|主程序|测试程序|打包运行|
|---|---|---|---|
|`compile`（默认）|✓|✓|✓|
|`test`|✗|✓|✗|
|`provided`|✓|✓|✗|
|`runtime`|✗|✓|✓|

典型例子：

|Scope|典型依赖|
|---|---|
|`compile`|`log4j`|
|`test`|JUnit|
|`provided`|Servlet API|
|`runtime`|JDBC 驱动|

---

## 8.1 为什么 JUnit 使用 `test`

JUnit 是测试工具，并不是正式业务程序运行所需要的依赖。

因此：

```xml
<scope>test</scope>
```

可以让 JUnit：

- 在测试代码中可用
- 在正式 `main` 代码中不可用
- 不进入正式程序的运行依赖

这体现了 Maven 依赖管理的一个重要思想：

> **依赖不仅要声明“需要什么”，还可以声明“在哪里需要”。**

---

# 9. 本节核心知识关系

本节可以与前面的 Maven 依赖管理和项目结构连接起来：

|概念|解决的问题|
|---|---|
|单元测试|验证最小功能单元是否正确|
|JUnit|提供规范化的 Java 单元测试能力|
|`@Test`|标记测试方法|
|Assertions|自动判断实际结果是否符合预期|
|生命周期注解|管理测试前置、后置资源|
|参数化测试|使用不同输入重复执行同一测试逻辑|
|`@DisplayName`|提升测试结果可读性|
|`src/test/java`|与正式业务代码隔离测试代码|
|`scope=test`|将测试依赖限制在测试环境|

### 与 Maven 前面知识的连接

* **POM**：JUnit 作为 Maven 依赖，需要在 `pom.xml` 中声明。
* **Dependency**：JUnit 本身就是当前项目所依赖的测试资源。
* **Scope**：通过 `scope=test` 限制 JUnit 只服务于测试代码。
* **Lifecycle**：Maven 的 `test` 阶段负责执行项目测试。

因此，JUnit 并不是孤立的知识点，它实际上连接了前面学习的：

> **Maven POM → Dependency → Scope → Test 生命周期**

---

# 10. 基础与进阶

### 基础知识

- 单元测试的概念
- JUnit 的作用
- `@Test`
- Assertions
- `@BeforeEach`
- `@AfterEach`
- `@BeforeAll`
- `@AfterAll`
- `@ParameterizedTest`
- `@ValueSource`
- `@DisplayName`
- `src/test/java`
- Maven `scope=test`

### 进阶方向

本节主要介绍 JUnit 的基础使用，没有深入展开以下内容：

- 测试生命周期的完整实现机制
- 参数化测试的更多参数来源
- 异常测试
- 超时测试
- 测试套件
- Mock / Stub
- Spring 环境下的单元测试
- 单元测试与集成测试的隔离
- 测试覆盖率

这些内容可以在后续学习 Spring Boot、数据库测试、Mock 框架时继续扩展，而不需要全部堆积到本笔记中。

---

# 11. 本节总结

JUnit 单元测试的核心不是记忆各种注解，而是理解三个层次：

* **测试思想**：验证程序实际行为是否符合预期。
* **JUnit**：提供标准化的测试组织和执行能力。
* **断言**：把“预期结果”明确表达出来，让测试框架能够自动判断成功或失败。

而在 Maven 项目中，JUnit 又与 Maven 的依赖管理和项目结构结合：

- 测试代码放在 `src/test/java`
- JUnit 通过 `pom.xml` 声明
- `scope=test` 限制 JUnit 的使用范围
- Maven 的 `test` 阶段负责测试执行

>[!tip] **长期记忆重点**
> 
> **JUnit = 测试框架；`@Test` = 测试方法；Assertion = 判断结果；Scope = 限制依赖使用范围。**