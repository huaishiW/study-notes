>[!abstract] Maven 从单模块项目进入大型项目开发后，需要解决三个核心问题：
> 
> 1. 如何拆分项目，降低模块之间的耦合？
>     
> 2. 如何统一管理多个模块的公共配置和依赖版本？
>     
> 3. 如何一次性构建整个项目？
>     

因此，多模块 Maven 项目主要涉及：

- **分模块设计**：解决项目规模和模块复用问题
- **继承**：解决公共 Maven 配置重复问题
- **`dependencyManagement`**：解决依赖版本统一管理问题
- **聚合**：解决多模块项目统一构建问题

---

# 1. 分模块设计

## 1.1 为什么需要分模块

如果所有业务代码都放在一个 Maven 项目中，随着项目规模扩大，会出现两个主要问题：

- 项目代码集中，维护和管理困难  
- 项目中的公共组件难以独立复用

例如一个大型项目可能同时包含：

- 商品
- 搜索
- 购物车
- 订单
- 用户中心
- 公共实体类
- 工具类
- 通用组件

如果这些内容全部位于同一个工程中，项目边界不清晰，团队协作和代码复用都会受到影响。

## 1.2 什么是分模块设计

**分模块设计**：按照项目的功能或结构，将一个大型项目拆分为多个相对独立的 Maven 模块。

例如：

```text
项目
├── 公共组件
├── 商品模块
├── 搜索模块
├── 购物车模块
└── 订单模块
```

模块之间通过 Maven 依赖进行资源共享。

例如订单模块需要使用公共实体类时，只需要依赖公共模块，而不需要把整个项目都引入进来。

## 1.3 分模块设计的核心价值

|问题|分模块后的解决方式|
|---|---|
|项目过于庞大|按功能或结构拆分|
|团队协作困难|不同团队/成员负责不同模块|
|公共代码难以复用|将公共代码独立成模块|
|模块边界不清晰|通过 Maven 依赖明确模块关系|
|修改某一功能影响范围过大|降低模块之间的耦合|

因此：

> **分模块的本质不是“把代码分到几个文件夹”，而是建立清晰的模块边界，并通过 Maven 依赖管理模块之间的关系。**

## 1.4 常见拆分策略

这里给出了三种策略：

|策略|说明|
|---|---|
|按功能模块拆分|商品、订单、搜索、购物车等|
|按层拆分|Controller、Service、DAO、Entity 等|
|功能 + 层|先按业务划分，再在模块内部进行分层|

实际项目中需要根据系统规模和团队组织方式选择。

---

# 2. 模块依赖与公共模块

分模块之后，一个重要问题是：

> **模块之间如何共享代码？**

答案是：**通过 Maven dependency 引入其他模块。**

例如：

```text
tlias-pojo
    ↑
    │ dependency
    │
tlias-web-management
```

`web-management` 不需要复制 `pojo` 中的代码，只需要在 `pom.xml` 中声明：

```xml
<dependency>
    <groupId>com.huaishi</groupId>
    <artifactId>tlias-pojo</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

因此，模块本身也是 Maven 可以管理的资源，同样通过：

> **坐标 + 依赖**

进行引用。

## 2.1 模块拆分

原项目中的公共代码包括：

- `pojo`：实体类
- `utils`：工具类
- `web-management`：业务代码

拆分之后：

```text
tlias-pojo
tlias-utils
tlias-web-management
```

其中：

- `tlias-pojo` → 提供实体类
- `tlias-utils` → 提供公共工具
- `tlias-web-management` → 使用上述公共模块并承载业务代码

特别强调：

> **实际开发中应该先设计模块，再进行编码，而不是项目开发完成后再进行拆分。**

---

# 3. Maven 继承

## 3.1 为什么需要继承

模块拆分之后，又出现了新的问题：

多个模块往往存在大量公共 Maven 配置。

例如：

```text
tlias-pojo
tlias-utils
tlias-web-management
```

都可能使用：

- Spring Boot
- Lombok
- 编译器配置
- 字符编码配置
- 其他公共依赖

如果每个模块都单独配置，会造成大量重复。

因此 Maven 提供了**继承机制**。

---

## 3.2 什么是 Maven 继承

Maven 继承描述的是**父工程与子工程之间的配置继承关系**。

子工程可以继承父工程中的 Maven 配置。

常见用途：

- 统一公共依赖
- 统一项目属性
- 统一插件配置
- 减少子模块中的重复配置

基本形式：

```xml
<parent>
    <groupId>...</groupId>
    <artifactId>...</artifactId>
    <version>...</version>
    <relativePath>...</relativePath>
</parent>
```

这里设计示例是：

```text
tlias-parent
├── tlias-pojo
├── tlias-utils
└── tlias-web-management
```

其中三个子模块继承 `tlias-parent`。

---

## 3.3 Maven 的单继承

Maven 一个项目只能直接继承一个父工程。

而 Spring Boot 项目通常已经继承：

```text
spring-boot-starter-parent
```

因此不能再让业务模块同时直接继承：

```text
tlias-parent
```

这里采用的是**多重继承**：

```text
spring-boot-starter-parent
            ↑
      tlias-parent
            ↑
    ┌───────┼────────┐
tlias-pojo tlias-utils tlias-web-management
```

也就是说：

- `tlias-parent` 继承 Spring Boot 父工程
- 各业务模块继承 `tlias-parent`

这样既可以获得 Spring Boot 父工程的配置，又可以使用自己的项目父工程。

---

# 4. `packaging`：模块如何被打包

父工程通常不会产生实际业务代码，因此其 `pom.xml` 一般使用：

```xml
<packaging>pom</packaging>
```

这里介绍了三种主要类型：

| packaging | 用途                  |
| --------- | ------------------- |
| `jar`     | 普通 Java 模块          |
| `war`     | Web 项目，部署到外部 Tomcat |
| `pom`     | 父工程或聚合工程            |

其中：

> **`pom` 类型工程主要用于组织和管理其他 Maven 模块，本身通常不承载业务代码。**

---

# 5. 公共依赖与 `dependencies`

如果某个依赖是**所有子模块都需要使用的**，可以直接配置在父工程：

```xml
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.34</version>
    </dependency>
</dependencies>
```

子工程继承父工程之后，可以直接获得这个依赖。

因此：

> `<dependencies>` 中声明的是**实际依赖**。

父工程中的直接依赖可以被子工程继承。

---

# 6. 依赖版本统一管理

## 6.1 为什么需要版本统一

并不是所有依赖都是所有模块都需要的。

例如：

```text
tlias-pojo       不需要 JWT
tlias-utils      需要 JWT
tlias-web        需要 JWT
tlias-report     需要 JWT
```

因此 JWT 不能简单地放入所有模块都会继承的 `<dependencies>`。

但多个模块如果都使用 JWT，又应该保持版本一致。

否则可能出现：

```text
模块 A → jjwt 0.9.1
模块 B → jjwt 0.9.2
模块 C → jjwt 0.9.1
```

版本分散在多个 `pom.xml` 中，升级和维护都比较困难。

---

## 6.2 `dependencyManagement`

Maven 使用：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

统一管理依赖版本。

子模块只需要：

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
</dependency>
```

无需再次填写 `<version>`。

### 核心区别

这是 Maven 多模块中非常重要的一组概念：

| 配置                       | 是否引入依赖 | 是否管理版本 |
| ------------------------ | -----: | -----: |
| `<dependencies>`         |      ✅ |      ✅ |
| `<dependencyManagement>` |      ❌ |      ✅ |

因此：

> **`dependencies` 决定“我要使用什么依赖”，`dependencyManagement` 决定“这个依赖统一使用什么版本”。**

主要强调：`dependencyManagement` 本身不会把依赖引入子工程，子工程仍然需要声明自己需要的依赖。

---

# 7. 使用 Maven 属性集中管理版本

当项目依赖较多时，即使版本集中在 `dependencyManagement` 中，也可能出现大量重复版本号。

可以进一步使用 `<properties>`：

```xml
<properties>
    <spring-boot.version>3.2.8</spring-boot.version>
    <lombok.version>1.18.34</lombok.version>
    <jwt.version>0.9.1</jwt.version>
</properties>
```

然后引用：

```xml
<version>${jwt.version}</version>
```

这样升级版本时，只需要修改属性：

```xml
<jwt.version>0.9.2</jwt.version>
```

而不需要搜索整个项目中的所有 `pom.xml`。

## 推荐理解

多模块项目中可以形成三层管理：

```text
properties
    ↓
统一维护版本号

dependencyManagement
    ↓
统一规定依赖版本

dependencies
    ↓
具体模块声明自己需要的依赖
```

这个关系比单纯记忆 XML 标签更重要。

---

# 8. Maven 聚合

## 8.1 为什么需要聚合

多模块项目还有一个实际问题：

**如何构建整个项目？**

假设：

```text
tlias-parent
tlias-pojo
tlias-utils
tlias-web-management
```

其中 `web-management` 又依赖 `pojo` 和 `utils`。

如果单独构建 `web-management`，可能需要提前将相关模块安装到本地 Maven 仓库。

当模块数量增加以后，手动构建每个模块会非常繁琐。

因此 Maven 提供了**聚合**。

---

## 8.2 什么是聚合

**聚合**：将多个 Maven 模块组织成一个整体，使 Maven 可以统一对这些模块执行构建操作。

聚合工程通常是一个：pom 类型的空工程。

通过：

```xml
<modules>
    <module>../tlias-pojo</module>
    <module>../tlias-utils</module>
    <module>../tlias-web-management</module>
</modules>
```

声明参与聚合的模块。

之后在聚合工程执行：

```bash
mvn package
```

Maven 就可以统一构建聚合范围内的模块。

这里将其概括为项目的“一键构建”，包括 `clean`、`compile`、`test`、`package`、`install` 等操作。

---

# 9. 继承与聚合不是一回事

这是本章最容易混淆的概念。

|对比|继承|聚合|
|---|---|---|
|解决的问题|公共配置管理|多模块统一构建|
|关注点|配置复用|项目构建|
|关系配置位置|子工程 `<parent>`|聚合工程 `<modules>`|
|父工程是否知道子模块|不知道|聚合工程知道|
|常见配置|`<parent>`|`<modules>`|
|packaging|通常为 `pom`|通常为 `pom`|

## 核心区别

> **继承解决“配置怎么复用”，聚合解决“模块怎么一起构建”。**

二者可以独立存在，也经常放在同一个 `pom.xml` 中。

因此真实项目中经常会看到：

```text
tlias-parent
├── 负责继承关系
├── 负责公共依赖管理
└── 负责聚合子模块
```

但要注意：

**“父工程”和“聚合工程”是两个概念，只是实际项目中经常由同一个 Maven 工程同时承担。**

---

# 10. 多模块 Maven 的核心模型

到这里，可以把 Maven 多模块体系压缩成四个核心概念：

|概念|核心职责|
|---|---|
|**模块**|划分项目边界|
|**继承**|复用父工程配置|
|**`dependencyManagement`**|统一依赖版本|
|**聚合**|统一构建多个模块|

它们分别解决不同层面的问题：

```text
项目结构 → 分模块

配置复用 → 继承

依赖版本治理 → dependencyManagement

统一构建 → 聚合
```

这里的四行不是需要记忆的流程，而是**四个独立的设计问题与对应机制**。

---

# 11. 与前面 Maven 知识的联系

本章实际上是在前面 Maven 基础模型上的扩展。

## POM

每一个 Maven 模块都有自己的：

```text
pom.xml
```

因此多模块项目本质上仍然是多个 Maven 项目的组合。

## 坐标

每一个模块都有自己的 Maven 坐标：

```text
groupId
artifactId
version
```

模块之间正是通过坐标建立依赖关系。

## Dependency

模块之间可以通过 `<dependency>` 相互依赖。

因此：

> **Maven 模块本身也是一种可以被其他 Maven 项目依赖的资源。**

## Repository

如果模块没有参与当前项目的统一构建，那么 Maven 可能需要从本地仓库获取已经安装的模块。

聚合则可以减少开发阶段手动安装模块的麻烦。

---

# 12. 本章知识定位

## 基础知识

- 分模块设计
- Maven 模块依赖
- Maven 继承
- `packaging`
- 聚合
- `<dependencies>`
- `<dependencyManagement>`

## 进阶知识

- 多模块项目的依赖结构设计
- 父工程配置设计
- 依赖版本治理
- 聚合构建
- 多重继承关系

## 后续可继续扩展

本章建立的是 Maven 多模块的基础模型，后续可以继续扩展：

- Maven 插件统一管理
- BOM
- 更复杂的模块依赖关系
- 依赖冲突与版本仲裁
- 多模块发布
- CI/CD 中的 Maven 多模块构建

---

# 13. 核心记忆

>[!tips]
> **分模块**：把大型项目拆成多个独立模块，建立清晰的模块边界。
> **继承**：让多个模块共享父工程配置，减少重复。
> **`dependencies`**：声明实际需要使用的依赖。
> **`dependencyManagement`**：统一管理依赖版本，但不会自动引入依赖。
> **属性**：把版本号集中到 `<properties>` 中维护。
> **聚合**：把多个 Maven 模块组织起来，实现统一构建。

最重要的一句话：

> **Maven 多模块设计 = 用模块划分项目边界，用继承复用配置，用 `dependencyManagement` 治理依赖版本，用聚合统一构建项目。**