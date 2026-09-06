> [!abstract] 本节核心  
> Part 01 解决了“**Maven 是什么**”。
> 
> Part 02 解决的是“**Maven 如何进入我们的开发环境，并成为 Java 项目的构建工具**”。
> 
> 核心可以概括为：
> 
> **Maven 安装 → 配置 Maven 环境 → 配置 Repository → IDEA 关联 Maven → 创建 Maven 项目 → 通过 `pom.xml` 描述项目**

---

# 1. Maven 的安装本质

Maven 本身是一个 Java 开发工具，它并不需要传统意义上的安装程序。

解压后的 Maven 目录主要包含：

```text
Maven
├── bin
├── conf
└── lib
```

|目录|作用|
|---|---|
|`bin`|Maven 可执行命令，其中最重要的是 `mvn`|
|`conf`|Maven 配置文件，其中重点是 `settings.xml`|
|`lib`|Maven 自身运行所依赖的 JAR 包|

因此，Maven 的安装本质上并不是“安装一个复杂的软件”，而是：

> **准备 Maven 运行环境，并通过配置文件告诉 Maven 如何工作。**

---

# 2. Maven 的核心配置：`settings.xml`

Maven 的很多全局行为都可以通过 `settings.xml` 配置。

本节主要涉及三个配置：

```text
settings.xml
├── localRepository —— 本地仓库位置
├── mirrors          —— Maven 镜像仓库
└── profiles         —— Maven/JDK 等环境配置
```

其中最重要的是：

> **`pom.xml` 描述“项目”，`settings.xml` 描述“当前 Maven 环境”。**

这是理解 Maven 配置层次时非常重要的一组概念。

---

# 3. 本地仓库配置

## 3.1 什么是本地仓库？

本地仓库就是当前计算机上的一个目录，用于保存 Maven 下载的依赖和其他资源。

例如：

```text
本地仓库
├── 第三方 JAR
├── Maven Plugin
└── 其他 Maven 资源
```

Maven 可以通过 `settings.xml` 中的：

```xml
<localRepository>...</localRepository>
```

指定本地仓库的位置。

## 3.2 为什么需要本地仓库？

Part 01 中已经知道 Maven 会从仓库获取依赖。

本地仓库实际上承担了**本地缓存**的作用：

> 远程获取的 Maven 资源保存到本地，后续项目可以直接复用。

因此可以把它和 Part 01 的 Repository 概念联系起来：

```text
远程仓库
   ↓ 下载
本地仓库
   ↓
项目使用
```

---

# 4. Maven 镜像仓库

Maven 默认可能需要从中央仓库获取依赖。

接下来进一步介绍了通过 `settings.xml` 配置阿里云 Maven 镜像，以改善依赖下载速度。

核心配置位于：

```xml
<mirrors>
    <mirror>
        ...
    </mirror>
</mirrors>
```

其中：

```xml
<mirrorOf>central</mirrorOf>
```

表示这个镜像用于替代中央仓库。

因此：

> **镜像（Mirror）本质上是 Maven 获取远程资源时使用的一个替代仓库地址。**

## 注意概念区分

这里的“镜像”不要和 Part 01 中的“私服”混为一谈。

本节中的阿里云仓库主要是为了：

> **提高公共 Maven 依赖的获取效率。**

而企业私服主要解决：

> **企业内部 Maven 资源管理、共享和发布。**

私服会在后面的 Maven 私服章节进一步学习。

---

# 5. Maven 环境变量

为了能够在命令行直接执行：

```bash
mvn -v
```

需要配置 Maven 环境变量。

这里采用：

```text
MAVEN_HOME = Maven 安装目录
```

然后在 `Path` 中加入：

```text
%MAVEN_HOME%\bin
```

这样系统就能够找到 Maven 的可执行程序。

可以理解为：

```text
MAVEN_HOME
    ↓
Maven 安装目录
    ↓
bin
    ↓
mvn 命令
```

验证：

```bash
mvn -v
```

如果能够正常输出 Maven 版本等信息，则说明 Maven 命令已经能够被系统识别。

---

# 6. Maven 与 JDK 的关系

Maven 本身使用 Java 开发，因此 Maven 的运行依赖 JDK。

同时，**Maven 构建项目时还需要知道项目应该使用哪个 JDK 版本进行编译。** 这里引出了一个重要的关系

> **Maven 是构建工具，而 JDK 是 Maven 构建 Java 项目时所依赖的 Java 开发环境。**

---

# 7. IDEA 集成 Maven

安装 Maven 后，还需要让 IDEA 知道：

> **应该使用哪个 Maven。**

因此 IDEA 中需要配置 Maven。

可以通过在 Setting 中的：

```text
Build, Execution, Deployment
    ↓
Build Tools
    ↓
Maven
```

进行 Maven 相关配置。

同时可以设置工程的编译版本，例如 JDK 17。

## 全局配置 vs 项目配置

这里需要建立一个很重要的认识：

> **IDEA 中的 Maven 配置可以作为全局配置存在，也可以在具体项目中使用。**

全局 Maven 配置意味着：

> 当前没有指定某个具体 Project，因此配置会作为默认 Maven 环境，之后创建的项目可以直接使用。

---

# 8. Maven 项目的标准目录结构

创建 Maven 项目后，会得到标准的项目结构：

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
│
├── target
└── pom.xml
```

|目录|作用|
|---|---|
|`src/main/java`|项目源代码|
|`src/main/resources`|项目配置文件及其他资源|
|`src/test/java`|测试代码|
|`src/test/resources`|测试相关资源|
|`target`|编译、打包等构建产生的文件|
|`pom.xml`|Maven 项目描述文件|

这里可以与 Part 01 的“统一项目结构”对应起来。

> **Maven 不仅管理项目的依赖和构建过程，还通过约定统一了项目的目录组织方式。**

---

# 9. `pom.xml`：项目的核心描述文件

POM 全称：

> **Project Object Model —— 项目对象模型**

Maven 使用：

```text
pom.xml
```

描述当前项目。

基础 `pom.xml` 包含：

```xml
<project>

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.huaishi</groupId>
    <artifactId>maven-project01</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

</project>
```

---

# 10. `pom.xml` 中的核心配置

## 10.1 `modelVersion`

```xml
<modelVersion>4.0.0</modelVersion>
```

表示当前 POM 所遵循的模型版本。

按照示例来说目前使用的是 `4.0.0`。

---

## 10.2 项目坐标

```xml
<groupId>com.huaishi</groupId>
<artifactId>maven-project01</artifactId>
<version>1.0-SNAPSHOT</version>
```

这三个元素共同构成项目坐标。

也就是说：

> **一个 Maven 项目本身也是一个 Maven 资源，因此它同样需要坐标。**

这与 Part 01 的坐标概念形成了直接联系。

---

## 10.3 `properties`

`properties` 可以定义项目构建过程中使用的属性。

主要用于定义以下几项：

```xml
<maven.compiler.source>17</maven.compiler.source>
<maven.compiler.target>17</maven.compiler.target>
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
```

分别描述：

- Java 编译版本
- Java 运行目标版本
- 项目字符编码

---

# 11. Maven 坐标

Maven 坐标是 Part 01 中已经出现的核心概念，本节进行了具体展开。

> **Maven 坐标是 Maven 资源的唯一标识，可以通过坐标唯一定位一个资源。**

一个基本坐标由：

```text
groupId
artifactId
version
```

组成。

## 11.1 `groupId`

表示项目所属的组织。

通常采用域名反写：

```text
com.huaishi
org.springframework
com.example
```

---

## 11.2 `artifactId`

表示项目或模块名称。

例如：

```text
order-service
goods-service
maven-project01
```

---

## 11.3 `version`

表示资源版本。

这里介绍了两类常见版本：

|版本|含义|
|---|---|
|`SNAPSHOT`|功能尚处于开发阶段的快照版本|
|`RELEASE`|功能趋于稳定，可以发行的版本|

例如：

```text
1.0-SNAPSHOT
```

表示一个开发中的版本。

---

# 12. 坐标的真正作用

坐标不仅用于第三方 JAR。坐标还可以用于标识**插件、依赖以及当前项目**。

而一个 Maven 项目如果被其他项目依赖，同样需要通过坐标进行引入。

因此可以形成一个非常重要的认识：

```text
Maven Resource
├── 当前项目
├── Dependency
└── Plugin
        ↓
      Coordinates
```

> **坐标实际上是 Maven 资源管理体系中的统一身份标识。**

---

# 13. IDEA 导入 Maven 项目

如果已有一个 Maven 项目，需要让 IDEA 识别它，可以通过两种方式导入：

### 方式一

```text
File
→ Project Structure
→ Modules
→ Import Module
→ pom.xml
```

### 方式二

```text
Maven 面板
→ Add Maven Projects
→ pom.xml
```

这里最值得理解的并不是具体菜单路径，而是：

> **IDEA 通过识别项目的 `pom.xml`，获得 Maven 项目的结构、坐标以及构建和依赖信息。**

---

# 14. 本节核心概念关系

Part 02 最重要的关系可以整理成：

```text
Maven
│
├── 安装目录
│   ├── bin —— mvn
│   ├── conf —— settings.xml
│   └── lib
│
├── settings.xml
│   ├── localRepository
│   ├── mirrors
│   └── profiles
│
└── Maven Project
    ├── pom.xml
    │   ├── 坐标
    │   ├── properties
    │   └── 后续依赖配置
    │
    ├── src
    │   ├── main
    │   └── test
    │
    └── target
```

其中需要特别区分：

|配置|关注对象|
|---|---|
|`settings.xml`|**Maven 环境本身**|
|`pom.xml`|**具体 Maven 项目**|

---

# 15. 与 Part 01 的知识连接

Part 01 建立了：

> Maven → POM → Dependency → Coordinates → Repository → Lifecycle

Part 02 则把其中的 **POM、Coordinates、Repository** 落到了实际项目中。

因此目前的知识网络可以扩展为：

```text
Maven
│
├── 环境
│   ├── Maven 安装
│   ├── settings.xml
│   └── IDEA
│
├── 项目
│   ├── pom.xml
│   ├── 标准目录结构
│   └── Coordinates
│
├── 依赖
│   └── Repository
│
└── 构建
    └── Lifecycle
```

---

# 16. 本节核心结论

如果只保留 Part 02，需要记住：

1. **Maven 本质上是解压即可使用的 Java 构建工具。**
2. `bin` 存放 Maven 命令，`conf` 存放配置，`lib` 存放 Maven 自身依赖。
3. **`settings.xml` 用于配置 Maven 环境。**
4. **本地仓库用于保存 Maven 获取到的依赖和其他资源。**
5. `mirrors` 可以配置远程仓库镜像。
6. Maven 可以通过环境变量让系统直接识别 `mvn` 命令。
7. IDEA 需要关联 Maven，才能使用 Maven 进行项目构建。
8. Maven 项目具有统一的目录结构。
9. **`pom.xml` 是描述 Maven 项目的核心文件。**
10. Maven 项目本身也是一个 Maven 资源，因此也拥有自己的坐标。
11. **坐标 = `groupId + artifactId + version`。**
12. 坐标不仅可以标识依赖，也可以标识项目和插件。
13. `settings.xml` 和 `pom.xml` 的核心区别是：
    - `settings.xml` → **Maven 环境**
    - `pom.xml` → **具体项目**

> [!tip] 本节最值得建立的认知  
> **Maven 的“环境”和“项目”是两层不同的东西。**
> 
> `settings.xml` 管 Maven 怎么工作；
> 
> `pom.xml` 管项目是什么、依赖什么以及如何构建。
> 
> 而 Maven 坐标则成为连接项目、依赖、仓库的重要身份标识。