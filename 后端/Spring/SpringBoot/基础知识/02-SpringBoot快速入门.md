# 1. Spring Boot Web 应用的基本开发流程

使用 Spring Boot 开发一个最基础的 Web 应用时，通常需要先创建 Spring Boot 工程，并引入 Web 开发所需要的依赖。随后编写 Controller 类，在其中定义用于处理请求的方法，并通过注解配置请求路径。完成代码编写后，运行 Spring Boot 启动类即可启动应用，浏览器随后可以通过对应的地址访问服务，由 Controller 接收请求并返回响应结果。

例如实现这样一个功能：

```text
浏览器请求：

/hello?name=huaishi
```

服务器返回：

```text
Hello huaishi
```

---

# 2. 创建 Spring Boot 工程

## 2.1 创建项目

Spring Boot 项目通常可以通过 Spring 官方提供的项目脚手架快速创建。

创建工程时，需要填写项目的基本信息，例如：

- 项目名称
    
- Group
    
- Artifact
    
- Java 版本
    
- Spring Boot 版本
    

然后根据项目需要选择对应的依赖。

---

## 2.2 引入 Web 开发依赖

开发 Web 应用时，需要选择：

```text
Spring Web
```

创建完成后，项目的 `pom.xml` 中会包含对应的 Web 开发依赖。

这个依赖为 Spring Boot 项目提供了 Web 开发所需要的相关能力，因此在创建 Web 项目时，需要将它引入到工程中。

当前阶段只需要先理解：**Spring Boot 项目如果要开发 Web 功能，就需要引入对应的 Web 开发依赖。**

至于这个依赖为什么能够提供 Spring MVC、Tomcat 等相关能力，会在 `Starter与内嵌Tomcat.md` 中单独整理。

---

# 3. 编写 Controller

在 Spring Boot Web 项目中，需要定义一个类来接收和处理浏览器发送的请求。

例如：

```java
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello(String name) {
        System.out.println("HelloController ... hello: " + name);
        return "Hello " + name;
    }
}
```

这个类就是一个最基本的 Controller。

Controller 的主要职责是接收客户端发送的请求，调用对应的方法进行处理，并将结果返回给客户端。

---

# 4. @RestController

## 4.1 基本作用

```java
@RestController
```

用于标识当前类是一个 Web 请求处理类。

例如：

```java
@RestController
public class HelloController {
}
```

Spring Boot 应用启动后，Spring 会识别这个类，并将它作为请求处理组件进行管理。

因此，定义 Controller 时，需要使用相应的注解告诉 Spring：**这个类中的方法用于处理 Web 请求。**

---

## 4.2 当前阶段的理解

当前阶段可以先将 `@RestController` 理解为：

> 用于声明当前类是一个能够接收请求并返回响应数据的 Controller。

`@RestController` 更具体的组成以及它为什么可以直接返回响应数据，会在：

```text
SpringMVC
└── ResponseBody与RestController.md
```

中继续整理。

---

# 5. @RequestMapping

## 5.1 基本作用

```java
@RequestMapping("/hello")
```

用于配置请求路径，并将这个请求路径与某个 Controller 方法建立对应关系。

例如：

```java
@RequestMapping("/hello")
public String hello(String name) {
    return "Hello " + name;
}
```

当客户端访问 `/hello` 时，Spring 会根据请求路径找到对应的 `hello()` 方法，并执行这个方法。

因此，`@RequestMapping` 可以简单理解为：

> 告诉 Spring，当收到指定路径的请求时，应该由哪个方法负责处理。

---

## 5.2 请求示例

浏览器访问：

```text
http://localhost:8080/hello?name=huaishi
```

这个地址中：

- `http://localhost:8080` 表示要访问当前计算机上运行的 Web 服务。
    
- `/hello` 表示本次请求的路径。
    
- `name=huaishi` 表示请求中携带的参数。
    

由于 `hello()` 方法配置了：

```java
@RequestMapping("/hello")
```

所以 Spring 收到这个请求后，会找到并执行对应的 `hello()` 方法。

---

# 6. Controller 方法接收请求参数

Controller 方法可以直接声明用于接收请求参数的形参，例如：

```java
public String hello(String name)
```

当浏览器发送：

```text
/hello?name=huaishi
```

请求中包含一个名为 `name` 的参数，其值为：

```text
huaishi
```

Spring 在处理请求时，会将这个参数的值传递给方法中的：

```java
String name
```

因此在方法内部，`name` 的值就是 `huaishi`。

随后执行：

```java
return "Hello " + name;
```

最终返回：

```text
Hello huaishi
```

当前阶段可以先理解为：**Spring 能够将请求中携带的参数自动传递给 Controller 方法中名称对应的参数。**

更完整的请求参数接收规则可以在后续学习 Spring MVC 参数接收时继续补充。

---

# 7. Spring Boot 启动类

创建 Spring Boot 工程后，会自动生成一个启动类。

通常结构类似：

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Spring Boot 启动类最明显的特征是：

```java
@SpringBootApplication
```

应用开发完成后，只需要运行这个类中的 `main()` 方法，就可以启动 Spring Boot 应用。

当前阶段先掌握启动方式即可，至于为什么一个 `main()` 方法就能够启动 Web 应用，会在后续的 `Starter与内嵌Tomcat.md` 中继续分析。

---

# 8. 启动并测试程序

## 8.1 启动应用

完成 Controller 编写后，运行带有：

```java
@SpringBootApplication
```

注解的启动类。

当 Spring Boot 应用成功启动后，就可以开始接收客户端发送的请求。

---

## 8.2 浏览器访问

例如在浏览器中访问：

```text
http://localhost:8080/hello?name=huaishi
```

浏览器会向当前运行的 Spring Boot 应用发送请求。

Spring 收到请求后，会根据 `/hello` 找到 `HelloController` 中对应的 `hello()` 方法，同时将请求参数 `name=huaishi` 传递给方法中的 `name` 参数。

方法处理完成后返回：

```text
Hello huaishi
```

Spring 再将这个结果作为响应返回给浏览器。

这样就完成了一次最基本的 Web 请求处理。

---

# 9. localhost 和 8080

## 9.1 localhost

```text
localhost
```

表示当前计算机。

因此：

```text
http://localhost
```

表示访问当前电脑上运行的 Web 服务。

在本地开发和测试 Spring Boot 项目时，经常会使用 `localhost` 访问自己电脑上的应用。

---

## 9.2 8080

```text
8080
```

是当前 Spring Boot Web 应用使用的端口。

因此：

```text
http://localhost:8080
```

表示访问当前计算机上运行在 `8080` 端口的 Web 服务。

在没有额外修改配置的情况下，Spring Boot Web 项目通常会使用 `8080` 端口启动。

---

# 10. Spring Boot Web 入门程序的执行过程

一个最简单的 Spring Boot Web 程序，可以从请求处理的角度理解。

首先，开发者创建 Controller，并使用 `@RestController` 将这个类声明为 Web 请求处理类。随后通过 `@RequestMapping` 给 Controller 方法配置请求路径。

应用启动后，浏览器可以向这个路径发送请求。Spring 收到请求后，会查找负责处理该路径的方法，并将请求中携带的数据传递给方法。Controller 方法完成处理后，将结果返回，最终由 Spring 将这个结果响应给浏览器。

因此，最基础的 Spring Boot Web 开发，本质上就是完成：

- 请求路径的配置
    
- 请求数据的接收
    
- Controller 方法的处理
    
- 响应结果的返回
    

---

# 11. 项目创建失败时的处理方式

创建 Spring Boot 项目时，需要联网访问项目脚手架。

如果因为网络原因无法正常使用 Spring 官方提供的创建方式，也可以使用其他可用的 Spring Boot 项目初始化工具完成工程创建。

无论通过哪种脚手架创建，只要最终得到的是标准的 Spring Boot 项目，其基本开发方式并不会发生变化。

---

# 12. 当前阶段需要掌握的核心内容

1. Spring Boot 工程可以通过项目脚手架快速创建。
    
2. 开发 Web 项目时，需要引入 Web 相关依赖。
    
3. `@RestController` 用于声明 Web 请求处理类。
    
4. `@RequestMapping` 用于配置请求路径。
    
5. Controller 方法可以接收请求参数并返回处理结果。
    
6. 运行带有 `@SpringBootApplication` 的启动类可以启动 Spring Boot 应用。
    
7. Spring Boot 应用启动后，可以通过 `localhost:8080` 在本地进行访问和测试。
    

> [!tip] 一句话总结
> 
> `Spring Boot Web 应用的基本开发方式是创建工程并引入 Web 依赖，通过 Controller 接收和处理请求，再运行启动类启动应用并向客户端返回响应结果。`