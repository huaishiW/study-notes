> [!abstract] 核心定义  
> **Maven 是一个用于管理和构建 Java 项目的工具。**
> 
> 它以 **POM（Project Object Model，项目对象模型）** 为核心，通过标准化的项目描述、依赖管理和构建生命周期，使 Java 项目的依赖管理、项目结构和构建过程得到统一。

---

# 1. Maven 解决什么问题？

在没有 Maven 时，Java 项目通常需要手动完成：

- 下载第三方 JAR 包
- 将 JAR 包加入项目
- 管理不同 JAR 包之间的依赖
- 编译源代码
- 执行测试
- 打包项目
- 发布项目
- 约定项目目录结构

当项目规模变大后，这些工作会产生大量重复操作，并且容易出现依赖版本冲突、项目结构不统一等问题。

Maven 主要解决三个问题：

|问题|Maven 的解决方式|
|---|---|
|第三方 JAR 包管理困难|**依赖管理**|
|不同项目结构不统一|**标准项目结构**|
|编译、测试、打包等操作繁琐|**标准化构建流程**|

因此，可以把 Maven 理解为：

> **用统一的方式描述 Java 项目，并统一管理项目依赖和项目构建过程。**

---

# 2. Maven 的核心模型

Maven 的核心可以抽象成三个模型：

```text
Maven
├── POM —— 描述“项目是什么”
├── Dependency —— 描述“项目依赖什么”
└── Lifecycle / Phase —— 描述“项目如何构建”
```

| 核心模型                  | 解决的问题               |
| --------------------- | ------------------- |
| **POM**               | 如何描述一个 Maven 项目     |
| **Dependency**        | 项目需要哪些外部资源          |
| **Lifecycle / Phase** | 项目应该如何进行编译、测试、打包等构建 |

这三个模型共同构成 Maven 的基本工作方式。

---

# 3. POM：项目对象模型

## 3.1 什么是 POM？

POM 全称：

> **Project Object Model —— 项目对象模型**

Maven 会把一个 Java 项目抽象成一个“项目对象”，项目中的各种信息通过 `pom.xml` 进行描述。

例如：

```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>1.0</version>

    <dependencies>
        ...
    </dependencies>
</project>
```

因此：

> **`pom.xml` 是 Maven 描述和管理项目的核心配置文件。**

---

# 4. 坐标：Maven 如何唯一定位一个资源？

Maven 中的 JAR 包等资源需要能够被准确找到。

因此 Maven 为资源建立了 **坐标（Coordinates）**。

核心组成：

```text
groupId
artifactId
version
```

|坐标|含义|
|---|---|
|`groupId`|组织/项目所属的组织标识|
|`artifactId`|模块名称|
|`version`|版本号|

例如：

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>demo-utils</artifactId>
    <version>1.0.0</version>
</dependency>
```

可以把它理解成：

> **坐标 = Maven 世界中一个资源的身份标识。**

因此 Maven 不再需要通过“某个 JAR 文件放在哪里”来识别依赖，而是通过坐标描述：

> **我要哪个组织的哪个模块的哪个版本。**

---

## 5. Dependency：依赖管理模型

Java 项目通常会依赖大量第三方 JAR 包。

传统方式：

```text
下载 JAR
   ↓
复制到项目 lib
   ↓
项目引用
```

Maven 则使用坐标描述依赖：

```text
pom.xml
   ↓
声明依赖坐标
   ↓
Maven 根据坐标寻找资源
   ↓
获得对应 JAR
```

因此：

> **Dependency 模型本质上解决的是“项目需要哪些外部资源，以及这些资源如何被找到和管理”。**

例如：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>...</version>
</dependency>
```

项目只需要声明依赖，而不需要自己把 JAR 文件复制到项目中。

---

# 6. Repository：依赖存在哪里？

Maven 的依赖管理离不开 **仓库（Repository）**。

> **仓库本质上就是一个用于存储 JAR 包、插件等 Maven 资源的目录。**

Maven 中主要涉及：

|仓库|作用|
|---|---|
|**本地仓库**|当前计算机保存的 Maven 资源|
|**中央仓库**|Maven 生态提供的公共资源仓库|
|**远程仓库 / 私服**|企业或团队自己搭建的 Maven 仓库|

## 6.1 Maven 为什么需要仓库？

因为：

```text
pom.xml
   ↓
依赖坐标
   ↓
Repository
   ↓
找到对应 JAR
```

Maven 不直接把所有 JAR 放进项目，而是通过仓库统一管理这些资源。

---

# 7. Maven 的依赖查找思想

项目声明一个依赖后，Maven 首先会检查本地是否已经存在对应资源。

基本逻辑是：

```text
本地仓库
   ↓ 没有
远程/中央仓库
   ↓
下载
   ↓
保存到本地仓库
```

如果项目配置了企业私服，则查找顺序为：

```text
本地仓库 → 远程仓库 → 中央仓库
```

因此，本地仓库实际上承担了一个重要角色：

> **缓存已经下载过的 Maven 资源，避免每次构建都重新从远程仓库获取。**

---

# 8. Lifecycle / Phase：Maven 如何构建项目？

Maven 不只是管理依赖，它还定义了一套标准化的项目构建流程。

典型操作包括：

```text
编译 → 测试 → 打包 → 发布
```

Maven 为这些构建过程提供统一的生命周期和阶段（Lifecycle / Phase），并通过插件完成具体工作。

例如：

|构建任务|Maven 中对应的概念|
|---|---|
|编译|`compile`|
|测试|`test`|
|打包|`package`|
|发布|`deploy`|

因此：

> **Lifecycle / Phase 负责规定构建过程，而 Plugin 负责具体执行构建任务。**

例如执行：

```bash
mvn package
```

Maven 会按照生命周期的规则执行相应阶段，而不是让开发者自己编写一系列命令完成编译、测试和打包。

---

# 9. Maven 标准项目结构

Maven 同时规定了统一的 Java 项目目录结构。

核心结构：

```text
项目
├── src
│   ├── main
│   │   ├── java
│   │   └── resources
│   │
│   └── test
│       ├── java
│       └── resources
└── pom.xml
```

其中：

- `main/java`：项目源代码
- `main/resources`：项目运行所需资源/配置
- `test/java`：测试代码
- `test/resources`：测试相关资源

这种统一结构使不同 IDE 创建和使用 Maven 项目时具有一致的组织方式。

---

# 10. 核心概念之间的关系

这一部分最重要的不是分别记住 POM、Dependency、Repository，而是理解它们之间的关系：

```text
                 Maven
                   │
             ┌─────┴─────┐
             │           │
            POM      Lifecycle
             │
      ┌──────┴──────┐
      │             │
  Dependency     Project
      │
      ↓
   Coordinates
      │
      ↓
 Repository
```

可以用一句话串起来：

> **Maven 通过 POM 描述项目，通过坐标描述项目和依赖资源，通过 Repository 获取依赖，通过 Dependency 管理项目依赖，并通过 Lifecycle/Phase 组织项目构建过程。**

---

# 11. 知识依赖关系

建议在 Obsidian 中建立如下知识关系：

```text
Maven
├── POM
│   └── 坐标
│
├── Dependency
│   ├── 坐标
│   └── Repository
│
├── Repository
│   ├── 本地仓库
│   ├── 中央仓库
│   └── 私服
│
└── Lifecycle
    ├── Phase
    └── Plugin
```

其中最值得记住的是：

**POM → Dependency → Coordinates → Repository**

以及：

**Lifecycle → Phase → Plugin**

前一条解释 **“依赖从哪里来”**，后一条解释 **“项目如何构建”**。

---

# 12. 一个实际工程中的 Maven 项目

假设我们开发一个 Spring Boot 项目，需要使用某个第三方库。

传统方式可能需要：

```text
下载 JAR
↓
复制到项目
↓
配置项目依赖
```

使用 Maven 后：

```text
pom.xml
↓
声明 groupId + artifactId + version
↓
Maven 查找 Repository
↓
获得依赖
↓
项目使用
```

当项目需要发布时：

```text
mvn package
```

即可按照 Maven 的标准构建流程完成项目打包。

因此 Maven 实际上把：

> **项目描述 + 依赖管理 + 资源获取 + 项目构建**

统一到了一个模型中。

---

# 13. 本节核心结论

如果只保留 Part 01 的几个核心知识点：

1. **Maven 是 Java 项目的管理和构建工具。**
2. Maven 主要解决 **依赖管理、统一项目结构、标准化构建** 三类问题。
3. **POM 是 Maven 描述项目的核心模型。**
4. **坐标是 Maven 定位资源的重要方式：`groupId + artifactId + version`。**
5. **Dependency 描述项目需要哪些外部资源。**
6. **Repository 负责存储和提供 Maven 资源。**
7. **Lifecycle / Phase 定义标准化的项目构建过程。**
8. **Plugin 负责具体执行构建任务。**
9. Maven 的核心思想不是“帮你下载 JAR”，而是建立了一套**标准化的项目描述、依赖管理和构建模型**。

> [!tip] 学习重点  
> Part 01 不需要急着记大量 Maven 命令。
> 
> 现在真正需要建立的是一个整体模型：
> 
> **POM 描述项目 → Dependency 描述依赖 → 坐标定位资源 → Repository 提供资源 → Lifecycle 组织构建 → Plugin 执行具体任务。**