# 1. 什么是 Filter

`Filter` 中文称为：

**过滤器**

材料中明确指出：

> Filter 是 JavaWeb 三大组件之一。

JavaWeb 三大组件包括：

```text
Servlet
Filter
Listener
```

Filter 的主要作用是：

> **在请求访问 Web 资源之前，对请求进行统一拦截和处理。**

也就是说，当浏览器请求服务器中的某个资源时，请求可以先经过 Filter，再决定是否继续访问后面的资源。

# 2. Filter 的核心作用

使用 Filter 后，请求在真正访问目标资源之前，需要先经过过滤器。

因此 Filter 很适合处理一些多个接口都会使用的公共逻辑。

材料中列举的典型场景包括：

- 登录校验
    
- 统一编码处理
    
- 敏感字符处理
    

这些功能都有一个共同特点：

> 不属于某一个具体业务方法，而是多个请求都可能需要执行。

# 3. 为什么登录校验适合使用 Filter

前面的 JWT 登录认证已经完成：

```text
登录成功
→ 生成 JWT
→ 前端保存 JWT
→ 后续请求携带 JWT
```

接下来服务端还需要：

> 在每一个受保护接口执行之前校验 JWT。

如果在每个 Controller 中重复编写 Token 校验代码，会产生大量重复逻辑。

Filter 可以统一拦截请求，因此可以把登录校验集中放在 Filter 中完成。

这也是材料从 JWT 登录认证继续引出 Filter 的原因。

# 4. Filter 的基本开发步骤

材料将 Filter 的基本使用概括成两个步骤：

1. 定义过滤器。
    
2. 配置过滤器。
    

# 5. 定义 Filter

定义过滤器时，需要创建一个类并实现：

```java
Filter
```

接口。

例如：

```java
public class DemoFilter implements Filter {

}
```

实现接口后，需要重写 Filter 中的相关方法。

# 6. Filter 的三个核心方法

材料中的示例实现了：

```java
init()
doFilter()
destroy()
```

这三个方法。

# 7. init()

```java
public void init(FilterConfig filterConfig)
        throws ServletException {

}
```

`init()` 是：

**过滤器初始化方法**

材料中说明：

> Web 服务器启动并创建 Filter 实例时，会调用 `init()`。

它的特点是：

```text
服务器启动时调用
只调用一次
```

因此，它适合做过滤器初始化相关工作。

# 8. doFilter()

```java
public void doFilter(
        ServletRequest request,
        ServletResponse response,
        FilterChain chain)
        throws IOException, ServletException {

}
```

`doFilter()` 是 Filter 中最核心的方法。

材料中指出：

> 每当过滤器拦截到一次请求，就会调用一次 `doFilter()`。

因此它的特点是：

```text
每拦截一次请求
→ 调用一次 doFilter()
```

登录校验等请求级逻辑通常就写在这个方法中。

# 9. destroy()

```java
public void destroy() {

}
```

`destroy()` 是：

**过滤器销毁方法**

材料中说明：

> Web 服务器关闭时会调用 `destroy()`。

它的特点是：

```text
服务器关闭时调用
只调用一次
```

因此它和 `init()` 分别对应 Filter 生命周期的初始化和销毁阶段。

# 10. Filter 三个方法的执行时机

可以简单整理为：

|方法|调用时机|调用次数|
|---|---|---|
|`init()`|Filter 创建时|一次|
|`doFilter()`|每次拦截请求时|多次|
|`destroy()`|Filter 销毁时|一次|

其中实际开发中最重要的是：

```java
doFilter()
```

因为真正的请求拦截逻辑都主要写在这里。

# 11. Filter 的基础示例

材料中的快速入门代码如下：

```java
public class DemoFilter implements Filter {

    public void init(FilterConfig filterConfig)
            throws ServletException {

        System.out.println("init ...");
    }

    public void doFilter(
            ServletRequest servletRequest,
            ServletResponse servletResponse,
            FilterChain chain)
            throws IOException, ServletException {

        System.out.println("拦截到了请求...");
    }

    public void destroy() {

        System.out.println("destroy ...");
    }
}
```

这段代码可以观察 Filter 在服务器运行过程中的生命周期。

# 12. Filter 定义后为什么还不能直接生效

仅仅创建：

```java
DemoFilter
```

并实现：

```java
Filter
```

接口，还不能让过滤器真正参与请求处理。

还需要告诉 Web 服务器：

> 当前类是一个过滤器，并且应该拦截哪些请求。

因此还需要进行 Filter 配置。

# 13. @WebFilter

材料中使用：

```java
@WebFilter
```

配置过滤器。

例如：

```java
@WebFilter(urlPatterns = "/*")
public class DemoFilter implements Filter {

}
```

其中：

```java
urlPatterns
```

用于指定：

> 当前 Filter 要拦截哪些请求路径。

# 14. urlPatterns = "/\*"

示例中使用：

```java
@WebFilter(urlPatterns = "/*")
```

其中：

```text
/*
```

表示：

> 拦截浏览器发送到当前 Web 应用中的所有请求。

因此任何请求进入服务器时，都会先经过当前 Filter。

具体不同拦截路径的写法，会在后续：

```text
Filter执行流程与过滤器链.md
```

中继续整理。

# 15. @ServletComponentScan

材料中指出，在 Spring Boot 项目中使用：

```java
@WebFilter
```

定义 Servlet 组件后，还需要在启动类上添加：

```java
@ServletComponentScan
```

开启 Servlet 组件支持。

例如：

```java
@ServletComponentScan
@SpringBootApplication
public class TliasWebManagementApplication {

    public static void main(String[] args) {

        SpringApplication.run(
                TliasWebManagementApplication.class,
                args
        );
    }
}
```

这样 Spring Boot 才会扫描并识别：

```java
@WebFilter
```

标记的过滤器。

# 16. Filter 基础配置需要两个关键注解

在当前 Spring Boot 项目中，可以记住：

## 16.1 Filter 类

使用：

```java
@WebFilter
```

声明过滤器以及拦截路径。

## 16.2 启动类

使用：

```java
@ServletComponentScan
```

开启 Servlet 组件扫描。

两者配合后，Filter 才能够生效。

# 17. Filter 的位置

Filter 属于：

**JavaWeb 层面的统一拦截机制**

它拦截的是 Web 请求。

因此按照我们之前确定的知识库结构，把它放在：

```text
Java后端
└── JavaWeb基础
    └── Filter
        └── Filter过滤器.md
```

会比放在登录认证目录下更合适。

登录认证只是 Filter 的一个具体使用场景。

# 18. Filter 与业务代码的关系

Filter 的主要价值在于：

> 把通用请求处理逻辑从具体 Controller 业务代码中抽离。

例如登录校验不应该重复写在：

```java
DeptController
EmpController
ClazzController
```

等每一个 Controller 中。

而可以统一放到 Filter 中。

这样可以：

- 减少重复代码
    
- 集中维护公共逻辑
    
- 在请求进入业务层之前统一处理
    

# 19. 当前阶段需要掌握的核心内容

1. Filter 是 JavaWeb 三大组件之一。
    
2. Filter 可以在请求访问 Web 资源之前统一拦截请求。
    
3. Filter 常用于登录校验、统一编码、敏感字符处理等公共功能。
    
4. 自定义 Filter 需要实现 `Filter` 接口。
    
5. `init()` 在 Filter 初始化时执行，一般只调用一次。
    
6. `doFilter()` 每次拦截到请求都会执行，是 Filter 的核心方法。
    
7. `destroy()` 在 Filter 销毁时执行，一般只调用一次。
    
8. `@WebFilter` 用于配置过滤器。
    
9. `urlPatterns` 用于指定 Filter 的拦截路径。
    
10. `/*` 表示拦截所有请求。
    
11. Spring Boot 中需要通过 `@ServletComponentScan` 开启 Servlet 组件扫描。
    
12. Filter 本质上属于 JavaWeb 的通用请求拦截机制，登录校验只是其典型应用之一。
    

> [!tip] 一句话总结
> 
> `Filter 是 JavaWeb 提供的统一请求过滤机制，自定义过滤器需要实现 Filter 接口，并通过 doFilter() 处理每一次被拦截的请求，再结合 @WebFilter 和 @ServletComponentScan 完成过滤器配置。`