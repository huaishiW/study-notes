> [!abstract] **核心问题**  
> Maven 如何把项目运行所需要的 Java 依赖引入项目，并管理依赖之间的关系？

---

# 1. 依赖是什么？

**依赖（Dependency）**：当前项目运行或开发所需要的其他 Java 资源，通常表现为 `jar` 包。

例如项目需要使用 Spring、Logback 等框架，就需要在 Maven 中声明对应依赖。

Maven 通过 `pom.xml` 中的 `<dependency>` 描述项目需要哪些依赖。

---

# 2. 基本依赖配置

依赖配置位于：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>6.1.4</version>
    </dependency>
</dependencies>
```

核心结构：

```text
pom.xml
└── dependencies
    └── dependency
        ├── groupId
        ├── artifactId
        └── version
```

其中：

- `groupId`：组织/项目所属组织
- `artifactId`：依赖名称
- `version`：依赖版本

这三个信息共同构成依赖的 **Maven 坐标**。

因此可以把依赖理解为：

> **依赖声明 = 告诉 Maven：“我要哪个坐标对应的资源”。**

---

# 3. 添加依赖之后发生了什么？

当在 `pom.xml` 中添加依赖并刷新 Maven 后，Maven 会尝试获取对应资源。

基本过程：

```text
pom.xml
  ↓
读取 dependency
  ↓
根据坐标查找资源
  ↓
本地仓库是否存在？
  ├─ 是 → 直接使用
  └─ 否 → 从远程仓库获取
             ↓
          下载到本地仓库
             ↓
          项目使用
```

因此，**本地仓库不仅仅是存储 Maven 资源的地方，同时也是依赖下载后的本地缓存。**

如果依赖在本地仓库不存在，Maven 会访问远程仓库下载，因此第一次引入某些依赖时可能需要等待。

---

# 4. 如何查找依赖坐标？

当不知道某个依赖的 Maven 坐标时，可以通过：

1. Maven 中央仓库搜索
2. IDEA 中搜索 Maven 依赖
3. 已经熟悉 Maven 后直接根据已知坐标配置

例如需要使用 `logback-classic`，可以先搜索其：

```text
groupId
artifactId
version
```

然后将坐标写入 `pom.xml`。

> **本质：** 搜索依赖不是在寻找 jar 文件本身，而是在寻找这个 Maven 资源的**坐标**。

---

# 5. Maven依赖传递

这是 Maven 依赖管理中非常重要的概念。

假设：

```text
A → B → C → D
```

表示：

- A 依赖 B
- B 依赖 C
- C 依赖 D

那么 A 在引入 B 后，也会间接获得 C、D。

这就是：

> **依赖传递（Transitive Dependency）**

例如：

```text
项目 A
  ↓
spring-context
  ↓
其他 Spring 依赖
  ↓
其他底层依赖
```

所以在 IDEA 的 Maven 依赖树中，经常会发现：

> 自己明明只配置了一个依赖，为什么项目中出现了很多其他依赖？

原因就是 **Maven 自动处理了依赖传递**。

---

# 6. 为什么需要依赖传递？

依赖传递解决了一个很重要的问题：

> **项目不需要手动声明每一个底层依赖。**

例如：

```text
项目
 ↓
框架 A
 ↓
工具库 B
 ↓
基础库 C
```

项目只需要声明自己直接需要的 **A**，Maven 可以根据 A 的依赖关系自动获取 B、C。

因此：

```text
直接依赖
   ↓
传递依赖
   ↓
底层依赖
```

形成了一棵**依赖关系树**。

---

# 7. 排除依赖

依赖传递虽然方便，但有时候某些传递进来的依赖并不是当前项目需要的。

这时可以使用：

> **排除依赖（Exclusion）**

它的作用是：

> 主动切断某个传递依赖，使指定资源不再通过当前依赖传递到项目中。

例如：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.1.4</version>

    <exclusions>
        <exclusion>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-observation</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

这里的含义不是：

> “删除 `spring-context`。”

而是：

> “正常引入 `spring-context`，但是不要让它传递 `micrometer-observation`。”

---

# 8. 排除依赖的关系

可以把它理解成：

```text
项目
 ↓
 A
 ↓
 B
 ↓
 C
```

默认情况下：

```text
项目 → A → B → C
```

如果项目不需要 C：

```text
项目 → A → B
              ✕ C
```

通过 `<exclusions>` 主动断开：

```text
A
└── exclusions
    └── C
```

**注意：被排除的依赖只需要提供 `groupId` 和 `artifactId`，不需要指定 `version`。**

---

# 9. 核心关系

这一部分最值得记住的是下面这张关系：

```text
Maven项目
   │
   └── pom.xml
         │
         └── dependencies
               │
               └── dependency
                     │
                     ├── 坐标
                     │    ├── groupId
                     │    ├── artifactId
                     │    └── version
                     │
                     └── 依赖关系
                          │
                          ├── 直接依赖
                          │
                          └── 传递依赖
                               │
                               └── exclusions
```

---

# 10. 与前面知识的联系

Part 01 中已经学习了：

```text
POM
 ↓
Dependency
 ↓
Coordinates
 ↓
Repository
```

本节进一步把这条关系具体化：

> **POM 中声明依赖 → Maven 根据坐标定位资源 → 从本地/远程仓库获取 → 根据依赖关系处理传递依赖。**

因此 Maven 的依赖管理本质上不是简单的：

> “帮我下载几个 jar。”

而是：

> **通过 POM 描述依赖关系，再根据 Maven 坐标和仓库体系解析整个依赖关系。**

---

# 11. 基础 / 进阶划分

## 基础

- 什么是 Maven 依赖
- `<dependencies>`
- `<dependency>`
- `groupId`
- `artifactId`
- `version`
- 本地仓库
- 远程仓库
- 依赖刷新

## 进阶

- 依赖传递
- 依赖关系树
- 排除依赖
- 复杂项目中的依赖冲突与版本管理

> **当前重点掌握：**  
> **依赖配置 → 依赖传递 → 排除依赖。**

---

# 12. 一个实际工程场景

假设一个 Spring 项目：

```text
项目
├── Spring
├── MyBatis
└── MySQL Driver
```

项目只需要在 `pom.xml` 中声明直接需要的依赖。

其中某个框架又依赖其他基础库：

```text
项目
 └── Spring
      ├── 依赖 A
      ├── 依赖 B
      └── 依赖 C
```

Maven 会自动处理这些传递依赖。

如果发现某个传递依赖与项目中的其他依赖产生问题，或者项目明确不需要它，就可以通过：

```xml
<exclusions>
    ...
</exclusions>
```

将其排除。

---

# 13. 一句话总结

> [!tip]
**Maven 依赖管理就是通过 POM 声明项目依赖，利用 Maven 坐标从仓库获取资源，并自动处理依赖传递；当某个传递依赖不需要时，可以使用 exclusions 主动排除。**

## 相关笔记

- [Maven 基础与核心模型](<01-Maven基础与核心模型.md>)：回顾项目坐标、POM 和 Maven 的基本模型。
- [JUnit 单元测试](<05-JUnit单元测试.md>)：查看如何把测试依赖接入 Maven 生命周期。
- [Starter 与内嵌 Tomcat](<../Spring/SpringBoot/基础知识/03-Starter与内嵌Tomcat.md>)：理解 Starter 如何利用依赖传递组织 Web 运行环境。
- [MyBatis 快速入门](<../数据库/MyBatis/02-MyBatis快速入门.md>)：查看 Maven 依赖在数据库项目中的实际使用。
