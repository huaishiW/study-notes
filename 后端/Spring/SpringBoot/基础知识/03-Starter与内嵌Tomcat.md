# 1. 为什么 main 方法可以启动 Web 应用

在普通 Java 程序中，执行 `main()` 方法本身并不会自动启动一个 Web 服务器。

但是在 Spring Boot Web 项目中，只需要运行启动类中的 `main()` 方法，应用就可以启动，并且浏览器能够通过对应端口访问 Web 服务。

出现这种现象的关键原因，是项目中引入了 Web 开发所需要的起步依赖：

```xml
spring-boot-starter-web
```

这个依赖不仅提供了 Web 开发所需要的相关组件，还会进一步引入 Tomcat 相关依赖。

因此，Spring Boot Web 项目本身已经具备运行 Web 服务器的能力。

---

# 2. Starter 起步依赖

## 2.1 Starter 是什么

Starter 通常被称为：

**起步依赖**

它是 Spring Boot 提供的一种依赖组织机制。

一个 Starter 本质上是一组预先组织好的依赖集合，用于提供某个特定开发场景所需要的常见依赖。

例如开发 Web 应用时，可以引入：

```xml
spring-boot-starter-web
```

它会帮助项目引入 Web 开发中常用的相关依赖。

因此，开发者不需要自己逐个寻找和添加多个依赖。

---

## 2.2 Starter 的作用

如果没有 Starter，开发者在实现某一种功能时，可能需要自己判断：

- 这个功能需要哪些依赖
    
- 各个依赖之间是否存在关联
    
- 是否还需要额外引入其他组件
    

Starter 将某个开发场景中经常需要一起使用的依赖预先进行了组织。

因此，开发者只需要根据当前开发场景选择对应的 Starter，就可以获得一组常用依赖。

Starter 的主要作用可以概括为：

**简化依赖配置，让开发者更方便地搭建对应的开发环境。**

---

# 3. 常见的 Starter

材料中介绍了两个比较基础的 Starter。

## 3.1 spring-boot-starter-web

```xml
spring-boot-starter-web
```

主要用于 Web 应用开发。

其中包含了 Web 应用开发所需要的一些常见依赖。

在创建 Spring Boot Web 项目时选择 `Spring Web`，本质上就是为项目引入 Web 开发相关的 Starter。

---

## 3.2 spring-boot-starter-test

```xml
spring-boot-starter-test
```

主要用于项目测试。

其中包含了单元测试所需要的一些常见依赖。

因此，不同 Starter 通常对应不同的开发场景。

---

# 4. spring-boot-starter-web 与 Tomcat

## 4.1 Web Starter 会引入 Tomcat

`spring-boot-starter-web` 本身还依赖：

```xml
spring-boot-starter-tomcat
```

因此，项目在引入 `spring-boot-starter-web` 后，也会间接拥有 Tomcat 相关依赖。

这是因为 Maven 支持依赖传递。

例如，项目直接依赖：

```xml
spring-boot-starter-web
```

而 `spring-boot-starter-web` 又依赖：

```xml
spring-boot-starter-tomcat
```

那么最终当前项目中也会存在 Tomcat 相关依赖。

---

## 4.2 Maven 的依赖传递

Maven 的依赖传递是指：

如果当前项目依赖了某个组件，而这个组件本身又依赖其他组件，那么这些间接依赖也可以被当前项目使用。

在 Spring Boot Web 项目中：

```text
项目直接依赖：spring-boot-starter-web

spring-boot-starter-web 又依赖：
spring-boot-starter-tomcat
```

因此，即使没有在项目中手动单独添加 Tomcat 依赖，项目中依然会存在 Tomcat。

这也是 Spring Boot 能够简化依赖配置的重要原因之一。

---

# 5. 什么是内嵌 Tomcat

## 5.1 基本概念

由于 Tomcat 的相关依赖已经包含在 Spring Boot Web 项目中，因此 Tomcat 不再是一个必须单独部署和启动的外部服务器。

这种直接集成在应用中的 Tomcat，被称为：

**内嵌 Tomcat**

也就是说，Tomcat 已经成为当前 Spring Boot 应用的一部分。

---

## 5.2 Spring Boot 如何使用内嵌 Tomcat

当运行 Spring Boot 启动类中的：

```java
public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
}
```

Spring Boot 应用会启动，同时其中的内嵌 Tomcat 服务器也会被启动。

随后，当前 Spring Boot 项目会运行在这个 Tomcat 服务器中。

因此，开发者不需要像传统 Web 项目那样，先单独安装和启动 Tomcat，再将项目部署进去。

---

# 6. 默认端口 8080

Spring Boot Web 应用启动后，默认会使用：

```text
8080
```

端口。

因此本地访问应用时，通常使用：

```text
http://localhost:8080
```

例如前面的 Controller 定义了：

```java
@RequestMapping("/hello")
```

那么可以通过：

```text
http://localhost:8080/hello
```

访问对应接口。

材料中说明，运行 Spring Boot 启动类后，实际上启动了其中内嵌的 Tomcat，而当前项目也会自动运行在这个 Tomcat 中，并默认占用 `8080` 端口。

---

# 7. Starter 与内嵌 Tomcat 的区别

Starter 和内嵌 Tomcat 是两个不同的概念。

## 7.1 Starter

Starter 主要解决的是：

**依赖配置问题**

它将某一种开发场景下常用的依赖组织到一起，让开发者不需要逐个配置。

例如：

```xml
spring-boot-starter-web
```

就是 Web 开发场景对应的 Starter。

---

## 7.2 内嵌 Tomcat

内嵌 Tomcat 主要负责：

**提供 Web 服务器运行环境**

Spring Boot Web 项目之所以可以直接启动并接收浏览器请求，是因为项目中已经包含了 Tomcat。

因此，可以这样理解：

- Starter 是一种依赖组织机制。
    
- Tomcat 是实际运行 Web 应用的服务器。
    
- `spring-boot-starter-web` 会间接帮助项目引入 Tomcat。
    

---

# 8. Starter 如何体现 Spring Boot 的简化开发

Starter 是 Spring Boot 实现“简化配置、快速开发”的重要方式之一。

开发者在开发某种功能时，不再需要手动寻找大量相关依赖，而是可以直接选择对应的 Starter。

例如：

```xml
spring-boot-starter-web
```

用于 Web 开发。

```xml
spring-boot-starter-test
```

用于测试。

这种方式降低了项目依赖配置的复杂度，也使 Spring Boot 项目的创建和开发更加方便。

---

# 9. 当前阶段需要掌握的核心内容

1. Starter 是 Spring Boot 提供的一种起步依赖机制。
    
2. Starter 本质上是一组针对特定开发场景预先组织好的依赖。
    
3. `spring-boot-starter-web` 提供 Web 开发所需要的常见依赖。
    
4. `spring-boot-starter-web` 会依赖 `spring-boot-starter-tomcat`。
    
5. Maven 的依赖传递机制使项目可以间接获得 Tomcat 依赖。
    
6. Spring Boot Web 项目中的 Tomcat 属于内嵌 Tomcat。
    
7. 运行 Spring Boot 的 `main()` 方法时，内嵌 Tomcat 会随应用一起启动。
    
8. Spring Boot Web 应用默认使用 `8080` 端口。
    

> [!tip] 一句话总结
> 
> `Starter 通过预先组织特定开发场景所需的依赖来简化项目配置，而 spring-boot-starter-web 会间接引入内嵌 Tomcat，使 Spring Boot Web 应用可以直接通过 main 方法启动并提供 Web 服务。`