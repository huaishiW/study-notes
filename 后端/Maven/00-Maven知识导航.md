# Maven 知识导航

[返回后端知识图谱总览](<../后端知识图谱总览.md>)

Maven 部分的主线是“项目模型 → 依赖 → 构建 → 测试 → 模块化与发布”。

## 学习顺序

| 顺序 | 笔记 | 作用 | 关联 |
| ---: | --- | --- | --- |
| 1 | [Maven 基础与核心模型](<01-Maven基础与核心模型.md>) | 建立项目对象模型、坐标和生命周期的总体认识。 | 连接到[安装与 IDEA 集成](<02-Maven安装与IDEA集成.md>)和[依赖管理](<03-Maven依赖管理.md>)。 |
| 2 | [Maven 安装与 IDEA 集成](<02-Maven安装与IDEA集成.md>) | 让 Maven 进入实际开发环境。 | 依赖[基础与核心模型](<01-Maven基础与核心模型.md>)；服务于后续所有构建操作。 |
| 3 | [Maven 依赖管理](<03-Maven依赖管理.md>) | 理解坐标、依赖范围、传递依赖和版本管理。 | 直接连接 Spring Boot Starter、MyBatis 和 JUnit。 |
| 4 | [Maven 生命周期与构建](<04-Maven生命周期与构建.md>) | 理解 clean、compile、test、package 等阶段。 | 连接[JUnit 单元测试](<05-JUnit单元测试.md>)和多模块构建。 |
| 5 | [JUnit 单元测试](<05-JUnit单元测试.md>) | 将测试纳入 Maven 的标准生命周期。 | 与[Spring Boot 快速入门](<../Spring/SpringBoot/基础知识/02-SpringBoot快速入门.md>)、[MyBatis 快速入门](<../数据库/MyBatis/02-MyBatis快速入门.md>)中的测试实践相连。 |
| 6 | [Maven 多模块设计](<06-Maven多模块设计.md>) | 将大型工程拆成可独立管理又可统一构建的模块。 | 依赖依赖管理和生命周期；可用于组织 Web、业务和持久层。 |
| 7 | [Maven 私服](<07-Maven私服.md>) | 理解企业内部依赖发布和共享。 | 是多模块和依赖管理向团队协作环境的延伸。 |

## 关系主线

```text
基础与核心模型
        ↓
安装与 IDEA 集成
        ↓
依赖管理 ───────────────→ Spring Boot Starter / MyBatis / JUnit
        ↓
生命周期与构建
        ↓
单元测试
        ↓
多模块设计 ─────────────→ 私服
```

## 跨领域连接

- [Maven 依赖管理](<03-Maven依赖管理.md>)是[Spring Boot 概述](<../Spring/SpringBoot/基础知识/01-SpringBoot概述.md>)、[Starter 与内嵌 Tomcat](<../Spring/SpringBoot/基础知识/03-Starter与内嵌Tomcat.md>)和[MyBatis 快速入门](<../数据库/MyBatis/02-MyBatis快速入门.md>)的共同基础。
- [Maven 生命周期与构建](<04-Maven生命周期与构建.md>)解释了测试和打包如何被统一执行。
- [Maven 多模块设计](<06-Maven多模块设计.md>)可以进一步连接 Spring Boot 应用中的分层模块和数据库访问模块。
- [Maven 私服](<07-Maven私服.md>)是依赖管理从个人工程扩展到团队协作的结果。

## 复习抓手

1. 先回答 Maven 如何描述一个项目，再回答它如何管理依赖。
2. 把依赖解析和生命周期执行分开理解：前者回答“需要什么”，后者回答“按什么顺序完成构建”。
3. 多模块和私服属于工程组织能力，建立在前面两条主线之上。
