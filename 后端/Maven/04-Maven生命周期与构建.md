> [!abstract] **核心问题**
> Maven 如何统一项目的构建过程？生命周期、阶段（Phase）和插件（Plugin）分别承担什么职责？

---

# 1. Maven生命周期

Maven 生命周期（Lifecycle）是 Maven 对**项目构建过程的抽象和统一**。

软件项目通常都需要经历清理、编译、测试、打包、部署等构建工作。Maven 将这些常见构建步骤进行标准化，并形成生命周期体系。

需要注意：

> **生命周期本身不负责执行具体任务，它只是规定构建过程由哪些阶段组成。**

真正执行编译、测试、打包等工作的，是 **Maven Plugin（插件）**。

---

# 2. 三套生命周期

Maven 将项目构建划分为三套**相互独立**的生命周期：

|生命周期|主要职责|
|---|---|
|`clean`|清理构建产生的文件|
|`default`|项目的核心构建过程|
|`site`|生成项目报告、站点等|

其中日常开发最常接触的是 `default` 生命周期。

## clean

负责清理之前构建产生的文件，例如删除 `target` 目录中的构建结果。

## default

负责项目的核心构建工作，包括：

- 编译
- 测试
- 打包
- 安装
- 部署

## site

用于生成项目相关的报告和站点文档。

---

# 3. Phase：生命周期中的构建阶段

每套生命周期由多个 **Phase（阶段）** 组成。

Phase 是 Maven 生命周期真正对外提供的构建节点，例如：

```text
clean
compile
test
package
install
```

常用 Phase：

|Phase|作用|
|---|---|
|`clean`|清理上一次构建产生的文件|
|`compile`|编译项目源代码|
|`test`|执行单元测试|
|`package`|将项目打包成 `jar`、`war` 等文件|
|`install`|将项目安装到本地 Maven 仓库|

## Phase 的顺序关系

**同一套生命周期中的 Phase 具有顺序性。**

执行某个较后的 Phase 时，该生命周期中位于它之前的 Phase 也会执行。

例如：

```bash
mvn package
```

执行 `package` 时，会先完成 `compile`、`test` 等前置阶段。

但是：

```bash
mvn package
```

**不会执行 `clean`**。

原因是：

> `clean` 属于 `clean` 生命周期，而 `package` 属于 `default` 生命周期；不同生命周期之间没有这种前后执行关系。

---

# 4. Lifecycle 与 Phase 的关系

可以把两者理解为：

- **Lifecycle**：一套完整的构建规范
- **Phase**：这套规范中的具体构建阶段

例如 `default` 生命周期包含大量 Phase，日常开发主要关注：

```text
compile → test → package → install
```

而 `clean` 属于另一套独立生命周期。

这里不需要记住所有 Phase，实际开发中首先掌握常用阶段即可。

---

# 5. Plugin：真正执行构建任务的组件

Maven 生命周期是**抽象的**。

例如：

> `compile` 只表示“进入编译阶段”，它本身并不负责编译 Java 源代码。

真正完成编译工作的，是 **Plugin（插件）**。

因此 Maven 的职责可以理解为：

|概念|职责|
|---|---|
|Lifecycle|定义构建过程的整体规范|
|Phase|定义具体构建阶段|
|Plugin|执行具体构建任务|

这是 Maven 构建体系中非常重要的一组概念。

## 为什么要这样设计？

因为 Maven 将：

> **“什么时候做什么”**

和：

> **“具体怎么做”**

进行了分离。

生命周期负责描述构建过程，插件负责提供具体实现。

这也是 Maven 能够扩展构建能力的重要原因。

---

# 6. Maven构建的核心模型

Maven 构建体系可以浓缩成三个概念：

```text
Lifecycle
  └─ Phase
       └─ Plugin执行具体任务
```

与其记大量流程，更应该记住它们之间的**职责关系**：

> **Lifecycle 组织构建过程，Phase 划分构建阶段，Plugin 执行阶段中的具体任务。**

---

# 7. 如何执行生命周期

日常开发中主要有两种方式。

## IDEA

在 IDEA 右侧的 Maven 工具窗口中，可以直接选择对应的生命周期阶段执行。

例如：

- `clean`
- `compile`
- `test`
- `package`
- `install`

## 命令行

在 Maven 项目目录下执行：

```bash
mvn compile
mvn test
mvn package
mvn install
```

命令的基本形式是：

```bash
mvn <phase>
```

例如：

```bash
mvn package
```

表示执行 `package` 阶段。

---

# 8. 最容易混淆的三个概念

## Lifecycle ≠ Phase

Lifecycle 是一套完整的生命周期，例如：

```text
default
```

Phase 是生命周期中的具体阶段，例如：

```text
compile
test
package
```

---

## Phase ≠ Plugin

Phase 只是一个构建阶段，并不直接完成实际工作。

Plugin 才是真正执行任务的组件。

例如：

> `compile` 表示进入编译阶段，而具体的 Java 编译工作由相应插件完成。

---

## clean ≠ compile

二者属于不同生命周期：

- `clean` → `clean` 生命周期
- `compile` → `default` 生命周期

所以执行：

```bash
mvn package
```

不会自动执行 `clean`。

---

# 9. 与前面知识的联系

Part 01 中我们已经知道：

> Maven 的核心模型包括 **POM、Dependency、Lifecycle**。

Part 03 又学习了：

> POM 可以声明项目依赖。

本节进一步明确 Lifecycle：

> **POM 描述项目需要什么，Lifecycle 描述项目如何进行标准化构建。**

因此 Maven 的核心职责可以从两个方向理解：

|领域|Maven解决的问题|
|---|---|
|依赖管理|项目需要哪些外部资源|
|生命周期|项目如何进行清理、编译、测试、打包等构建工作|

而 Plugin 则是生命周期能够真正落地执行的基础。

---

# 10. 基础 / 进阶

## 基础

- Maven 生命周期
- `clean` / `default` / `site`
- Phase
- `compile`
- `test`
- `package`
- `install`
- Plugin
- `mvn <phase>`

## 进阶

- Lifecycle 与 Phase 的内部机制
- Phase 与 Plugin Goal 的对应关系
- Maven Plugin 配置
- 自定义 Plugin
- 生命周期绑定机制

> 当前阶段最重要的是建立正确的概念模型，不需要急着深入 Plugin 的内部实现。

---

# 11. 核心结论

> **Maven Lifecycle 是对项目构建过程的抽象；Lifecycle 由多个有序 Phase 组成；执行 Phase 时，由 Plugin 完成具体构建任务。**

最重要的三个概念：

|概念|记忆方式|
|---|---|
|Lifecycle|一套构建规范|
|Phase|一个具体构建阶段|
|Plugin|阶段对应的实际执行者|

**记住职责，而不是死记完整流程。**