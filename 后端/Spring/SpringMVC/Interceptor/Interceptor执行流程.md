# 1. Interceptor 的拦截路径配置

在注册 Interceptor 时，需要指定：

> 当前拦截器到底拦截哪些请求。

材料中通过：

```java
addPathPatterns(...)
```

配置需要拦截的路径。

例如：

```java
registry.addInterceptor(demoInterceptor)
        .addPathPatterns("/**");
```

这里：

```text
/**
```

表示拦截所有 Spring MVC 资源。

# 2. addPathPatterns()

`addPathPatterns()` 用于：

> 指定需要被 Interceptor 拦截的请求路径。

例如：

```java
registry.addInterceptor(demoInterceptor)
        .addPathPatterns("/**");
```

表示：

> 所有符合 `/**` 的请求都会经过当前 Interceptor。

# 3. excludePathPatterns()

除了指定：

> 哪些请求需要拦截。

还可以指定：

> 哪些请求不需要拦截。

材料中使用：

```java
excludePathPatterns(...)
```

完成配置。

例如：

```java
registry.addInterceptor(demoInterceptor)
        .addPathPatterns("/**")
        .excludePathPatterns("/login");
```

表示：

- 默认拦截所有请求
    
- `/login` 请求除外
    

# 4. 为什么登录接口通常需要排除

登录校验 Interceptor 的职责是：

> 判断当前用户是否已经登录。

但：

```text
/login
```

本身就是用户用来完成登录的接口。

如果 `/login` 也必须先通过登录校验，就会产生逻辑矛盾。

因此材料中的配置：

```java
.excludePathPatterns("/login")
```

就是明确让登录接口不经过当前拦截器。

# 5. 常见路径规则

材料中列出了几种常见的 Interceptor 路径匹配方式。

# 6. /* 一级路径

```text
/*
```

表示匹配：

> 一级路径。

例如可以匹配：

```text
/depts
/emps
/login
```

但是不能匹配：

```text
/depts/1
```

因为：

```text
/depts/1
```

已经包含了更深一级路径。

# 7. /** 任意级路径

```text
/**
```

表示：

> 任意层级路径。

例如可以匹配：

```text
/depts
/depts/1
/depts/1/2
```

因此：

```text
/**
```

通常用于表示：

> 拦截所有请求。

# 8. /depts/*

```text
/depts/*
```

表示：

> 匹配 `/depts` 下一级的路径。

例如可以匹配：

```text
/depts/1
```

但是不能匹配：

```text
/depts/1/2
```

# 9. /depts/**

```text
/depts/**
```

表示：

> 匹配 `/depts` 下任意层级的路径。

例如可以匹配：

```text
/depts
/depts/1
/depts/1/2
```

但不能匹配：

```text
/emps/1
```

因为它已经不属于 `/depts` 路径范围。

# 10. 常见路径规则对比

|路径模式|含义|
|---|---|
|`/*`|一级路径|
|`/**`|任意层级路径|
|`/depts/*`|`/depts` 下一级路径|
|`/depts/**`|`/depts` 下任意层级路径|

# 11. Interceptor 的完整执行位置

Interceptor 并不是最先接收到请求的组件。

材料中通过执行流程说明了：

> 请求会先经过 Filter，然后进入 Spring MVC 环境，再经过 Interceptor，最后执行 Controller。

# 12. Filter 先于 Interceptor

浏览器请求 Web 应用时：

> Filter 会先拦截请求。

Filter 首先执行自己的：

**放行前逻辑**

然后调用：

```java
filterChain.doFilter(...)
```

进行放行。

放行之后，请求才继续进入 Spring 环境。

# 13. DispatcherServlet

材料中指出，Tomcat 并不能直接识别 Spring MVC 中编写的 Controller。

Tomcat 能识别的是：

```text
Servlet
```

因此 Spring Web 环境中提供了一个非常核心的 Servlet：

```java
DispatcherServlet
```

它也称为：

**前端控制器**

所有进入 Spring MVC 的请求都会先进入：

```text
DispatcherServlet
```

再由它将请求转交给对应 Controller。

# 14. Interceptor 在 Controller 前执行

当请求进入 Spring MVC 并准备执行 Controller 时：

> Interceptor 会先拦截这次请求。

首先执行：

```java
preHandle()
```

# 15. preHandle() 决定是否继续执行 Controller

`preHandle()` 返回：

```java
true
```

表示：

> 放行。

于是请求继续执行 Controller。

如果返回：

```java
false
```

表示：

> 不放行。

此时：

> Controller 方法不会继续执行。

材料中明确指出，`preHandle()` 的布尔返回值直接决定 Controller 是否执行。

# 16. Controller 执行完成后的 Interceptor

如果：

```java
preHandle()
```

返回 `true`，Controller 正常执行。

Controller 方法执行完成之后，会继续执行 Interceptor 中的：

```java
postHandle()
```

以及：

```java
afterCompletion()
```

# 17. 请求最终回到 Filter

Interceptor 和 Controller 相关逻辑执行完成后：

> 请求再次返回 DispatcherServlet。

之后继续回到前面的 Filter。

此时执行 Filter 中：

**放行后的逻辑**

最后再把响应数据返回浏览器。

# 18. 完整请求执行顺序

根据材料，可以整理出完整执行顺序：

```text
Filter 放行前逻辑

DispatcherServlet

Interceptor.preHandle()

Controller

Interceptor.postHandle()

Interceptor.afterCompletion()

DispatcherServlet

Filter 放行后逻辑
```

# 19. Filter 和 Interceptor 的执行层级不同

Filter 属于：

```text
JavaWeb
```

它位于 Spring MVC 外层。

而 Interceptor 属于：

```text
Spring MVC
```

它是在请求进入 Spring 环境之后才工作的。

因此：

> Filter 的执行范围比 Interceptor 更靠外。

# 20. Filter 与 Interceptor 的接口规范不同

材料总结的第一个区别是：

Filter 需要实现：

```java
Filter
```

接口。

Interceptor 需要实现：

```java
HandlerInterceptor
```

接口。

因此两者虽然都可以拦截请求，但属于完全不同的接口规范。

# 21. Filter 与 Interceptor 的拦截范围不同

材料总结的第二个区别是：

> Filter 会拦截 Web 应用中的资源，而 Interceptor 主要拦截 Spring 环境中的资源。

可以简单理解为：

```text
Filter
范围更靠外

Interceptor
主要围绕 Spring MVC Controller
```

# 22. Filter 与 Interceptor 的基本对比

|对比项|Filter|Interceptor|
|---|---|---|
|所属技术|JavaWeb|Spring MVC|
|实现接口|`Filter`|`HandlerInterceptor`|
|执行位置|Spring MVC 外层|Spring MVC 内部|
|放行方式|`filterChain.doFilter()`|`preHandle()` 返回 `true`|
|主要拦截对象|Web 请求资源|Spring Controller 相关请求|

# 23. 登录校验应该使用 Filter 还是 Interceptor

当前材料分别演示了：

```text
Filter 登录校验
Interceptor 登录校验
```

并明确指出：

> 两种方式只需要选择其中一种即可。

也就是说，在当前案例中，它们都是：

> 实现统一 JWT 登录校验的两种技术方案。

并不需要同时启用。

# 24. 当前阶段需要掌握的核心内容

1. Interceptor 通过 `addPathPatterns()` 配置需要拦截的路径。
    
2. `excludePathPatterns()` 可以配置不需要拦截的路径。
    
3. `/*` 表示一级路径。
    
4. `/**` 表示任意层级路径。
    
5. `/depts/*` 表示 `/depts` 下一级路径。
    
6. `/depts/**` 表示 `/depts` 下任意层级路径。
    
7. 请求会先经过 Filter，再进入 Spring MVC。
    
8. Spring MVC 的核心前端控制器是 `DispatcherServlet`。
    
9. Interceptor 在 Controller 执行之前调用 `preHandle()`。
    
10. `preHandle()` 返回 `true` 后才会继续执行 Controller。
    
11. Controller 执行完成后会继续调用 `postHandle()` 和 `afterCompletion()`。
    
12. Spring MVC 处理完成后，请求再次回到 Filter，执行放行后的逻辑。
    
13. Filter 实现 `Filter` 接口，Interceptor 实现 `HandlerInterceptor`。
    
14. Filter 位于 Spring MVC 外层，Interceptor 主要作用于 Spring MVC 环境。
    
15. 当前材料中的 JWT 登录校验可以选择 Filter 或 Interceptor 中的一种实现。
    

> [!tip] 一句话总结
> 
> `Interceptor 位于 Spring MVC 请求处理链路中，请求先经过 Filter 和 DispatcherServlet，再由 preHandle() 决定是否执行 Controller，Controller 完成后继续执行 postHandle() 与 afterCompletion()，最后再返回 Filter 执行放行后的逻辑。`