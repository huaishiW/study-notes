# 1. Spring 生态

在学习 Spring Boot 之前，需要先明确 Spring Boot 在整个 Spring 体系中的位置。

Spring 经过长期发展，已经形成了一套包含多个项目的开发生态。不同的 Spring 项目负责解决不同领域的问题，这一整套技术通常被称为：

```text
Spring 全家桶
```

在整个 Spring 生态中，最基础、最核心的是：

```text
Spring Framework
```

Spring Framework 提供了 Java 后端开发中大量基础能力，例如：

- 依赖注入
    
- 事务管理
    
- Web 开发支持
    
- 数据访问
    
- 消息服务
    

可以先建立一个基本认识：

```text
Spring 生态
    │
    ├── Spring Framework
    │      └── 提供核心基础能力
    │
    ├── Spring Boot
    │
    └── 其他 Spring 项目
```

Spring Boot 并不是独立于 Spring Framework 存在的，而是建立在 Spring 生态基础之上的。

# 2. 为什么会出现 Spring Boot

如果直接基于 Spring Framework 进行项目开发，会存在两个比较明显的问题：

- **配置繁琐**
    
- **入门难度较大**
    

也就是说，Spring Framework 虽然提供了丰富的功能，但是开发者在真正使用这些功能时，需要进行较多的配置。

因此，Spring 官方提供了 Spring Boot，用于进一步简化 Spring 应用的开发过程。

# 3. 什么是 Spring Boot

Spring Boot 是 Spring 生态中的一个项目。

它的主要作用是帮助开发者：

```text
快速构建 Spring 应用
```

Spring Boot 最主要的两个特点是：

- **简化配置**
    
- **快速开发**
    

可以简单理解为：

```text
Spring Framework
        ↓
提供 Spring 的核心功能

Spring Boot
        ↓
降低这些功能的配置和使用成本
```

因此，Spring Boot 并不是替代 Spring，而是让 Spring 应用的开发变得更加简单。

# 4. Spring Boot 解决的核心问题

Spring Boot 主要解决的是 Spring 应用开发过程中配置繁琐的问题。

使用 Spring Boot 后，开发者可以减少大量配置工作，把更多精力放在具体业务功能上。

可以简单理解为：

```text
创建 Spring Boot 应用
      ↓
引入对应功能
      ↓
Spring Boot 简化大量配置
      ↓
编写业务代码
```

# 5. Spring 与 Spring Boot 的关系

## 5.1 Spring

Spring 可以理解为一个完整的开发生态，其中包含多个项目。

```text
Spring
├── Spring Framework
├── Spring Boot
└── ...
```

## 5.2 Spring Framework

Spring Framework 是整个 Spring 体系最核心的基础框架。

它提供了很多基础能力，例如：

```text
依赖注入
事务管理
Web 开发支持
数据访问
消息服务
```

## 5.3 Spring Boot

Spring Boot 的重点是帮助开发者更加快速地构建 Spring 应用。

三者之间可以建立这样的认识：

```text
Spring
│
├── Spring Framework
│      │
│      └── 提供核心基础能力
│
└── Spring Boot
       │
       └── 简化 Spring 应用的构建和开发
```

# 6. Spring Boot 的核心特点

## 6.1 简化配置

Spring Boot 可以减少直接使用 Spring Framework 时大量繁琐的配置工作。

## 6.2 快速开发

Spring Boot 可以帮助开发者更加快速地创建和开发 Spring 应用，提高开发效率。

可以概括为：

```text
Spring Boot
    │
    ├── 简化配置
    │
    └── 快速开发
```

# 7. 当前阶段需要掌握的核心认识

1. **Spring Boot 属于 Spring 生态中的一个项目。**
    
2. **Spring Framework 是 Spring 体系中最基础、最核心的部分。**
    
3. **Spring Boot 的主要作用是简化 Spring 应用的构建和开发。**
    
4. **Spring Boot 最核心的特点是简化配置和快速开发。**
    

> [!tip] 一句话总结
> 
> `Spring Boot 是 Spring 生态中用于快速构建 Spring 应用的项目，它通过简化配置降低开发难度并提高开发效率。`