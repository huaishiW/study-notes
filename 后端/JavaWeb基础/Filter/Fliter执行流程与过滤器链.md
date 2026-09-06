# 1. Filter 为什么需要“放行”

Filter 拦截到请求之后，请求并不会自动继续访问后面的 Web 资源。

如果希望请求继续向后执行，就需要进行：

**放行**

材料中使用：

```java
filterChain.doFilter(request, response);
```

完成放行。

因此可以理解为：

> Filter 先获得请求的控制权，再决定是否允许请求继续访问后面的资源。

# 2. FilterChain

Filter 中：

```java
FilterChain
```

表示过滤器链。

在 `doFilter()` 方法中，可以通过：

```java
filterChain.doFilter(request, response);
```

继续执行后续过滤器或者最终的 Web 资源。

这也是 Filter 请求处理过程中最重要的方法之一。

# 3. filterChain.doFilter() 的作用

例如：

```java
@Override
public void doFilter(
        ServletRequest request,
        ServletResponse response,
        FilterChain filterChain)
        throws IOException, ServletException {

    System.out.println("放行前");

    filterChain.doFilter(request, response);

    System.out.println("放行后");
}
```

其中：

```java
filterChain.doFilter(request, response);
```

可以看作请求处理过程中的一个分界点。

它之前的代码属于：

**放行前逻辑**

它之后的代码属于：

**放行后逻辑**。

# 4. 放行前逻辑

写在：

```java
filterChain.doFilter(request, response);
```

之前的代码，会在请求访问后续 Web 资源之前执行。

例如：

```java
System.out.println("DemoFilter 放行前逻辑");

filterChain.doFilter(request, response);
```

这里：

```java
System.out.println("DemoFilter 放行前逻辑");
```

就属于放行前逻辑。

材料中的示例也是先执行放行前代码，再执行 `filterChain.doFilter()`。

# 5. 放行后逻辑

当后续 Web 资源执行完成后，程序会重新回到当前 Filter。

此时：

```java
filterChain.doFilter(request, response);
```

后面的代码开始执行。

例如：

```java
filterChain.doFilter(request, response);

System.out.println("DemoFilter 放行后逻辑");
```

这里：

```java
System.out.println("DemoFilter 放行后逻辑");
```

就是放行后逻辑。

# 6. Filter 的完整执行特点

对于一个 Filter，可以理解为：

```text
放行前逻辑

后续过滤器或 Web 资源执行

放行后逻辑
```

也就是说，Filter 不仅可以在 Web 资源执行之前处理请求，也可以在 Web 资源执行完成之后继续执行自己的代码。

这一点是理解 Filter 执行流程的关键。

# 7. 不调用 filterChain.doFilter() 会发生什么

材料强调：

> 如果希望继续访问后面的 Web 资源，就必须执行放行操作。

因此，如果 Filter 中没有调用：

```java
filterChain.doFilter(request, response);
```

那么请求就不会继续访问后面的资源。

这也是 Filter 能够用于登录校验的原因之一。

例如：

```text
Token 合法
→ 放行

Token 非法
→ 不放行
```

当前这一篇只理解 Filter 的机制，具体 JWT 登录校验会在后面的综合实现中整理。

# 8. Filter 的拦截路径

Filter 不一定必须拦截所有请求。

可以通过：

```java
@WebFilter(urlPatterns = "...")
```

配置当前过滤器需要拦截哪些资源。

材料中列出了三种常见方式：

- 拦截具体路径
    
- 目录拦截
    
- 拦截所有请求
    

# 9. 拦截具体路径

例如：

```java
@WebFilter(urlPatterns = "/login")
```

表示：

> 只有访问 `/login` 时，当前 Filter 才会执行。

例如：

```java
@WebFilter(urlPatterns = "/login")
public class DemoFilter implements Filter {
}
```

材料中也专门通过 `/login` 对具体路径拦截进行了测试。

# 10. 目录拦截

例如：

```text
/emps/*
```

表示：

> 访问 `/emps` 路径下的所有资源时都会被拦截。

也就是说，如果请求属于：

```text
/emps/...
```

范围，就会经过当前 Filter。

材料中将这种方式称为：

**目录拦截**。

# 11. 拦截所有请求

例如：

```java
@WebFilter(urlPatterns = "/*")
```

表示：

> 当前 Web 应用中的所有请求都会经过这个 Filter。

材料中的 Filter 快速入门和执行流程示例都使用了：

```java
@WebFilter(urlPatterns = "/*")
```

# 12. 三种拦截路径对比

|`urlPatterns`|含义|
|---|---|
|`/login`|只拦截 `/login`|
|`/emps/*`|拦截 `/emps` 下的所有资源|
|`/*`|拦截所有资源|

具体使用哪一种方式，要根据 Filter 的功能决定。

例如：

> 如果一个 Filter 需要统一处理所有请求，就可以使用 `/*`。

# 13. 什么是过滤器链

一个 Web 应用中可以配置：

> 多个 Filter。

当多个 Filter 都能够拦截当前请求时，这些 Filter 会按照一定顺序依次执行。

多个 Filter 组合起来，就称为：

**过滤器链**

也就是：

```text
FilterChain
```

材料中明确指出，一个 Web 应用中的多个过滤器会共同形成过滤器链。

# 14. 多个 Filter 的放行过程

假设当前存在两个过滤器：

```text
Filter1
Filter2
```

请求首先进入：

```text
Filter1
```

Filter1 执行放行前逻辑后调用：

```java
filterChain.doFilter(...)
```

请求继续进入：

```text
Filter2
```

Filter2 再执行自己的放行前逻辑。

如果 Filter2 继续放行，最终才会访问 Web 资源。

材料中明确说明：

> Filter 会一个一个执行，前一个 Filter 放行之后才会进入下一个 Filter，最后一个 Filter 放行后才会访问 Web 资源。

# 15. 多个 Filter 的返回过程

Web 资源执行完成后，程序开始返回。

这时候 Filter 的执行顺序会反过来。

如果放行阶段是：

```text
Filter1
Filter2
```

那么放行后的逻辑执行顺序就是：

```text
Filter2
Filter1
```

材料中明确指出：

> 放行后的逻辑按照相反顺序执行。

# 16. 为什么过滤器链是反向返回

可以把多个 Filter 理解成：

> 一个 Filter 包着另一个 Filter。

Filter1 调用：

```java
filterChain.doFilter(...)
```

后暂时停在那里。

然后 Filter2 执行。

Filter2 再放行给 Web 资源。

Web 资源执行完成后，先回到 Filter2。

Filter2 执行完放行后逻辑，再回到 Filter1。

因此形成：

```text
进入顺序：
Filter1
Filter2

返回顺序：
Filter2
Filter1
```

# 17. 两个 Filter 的代码理解

假设 Filter1：

```java
System.out.println("Filter1 前");

filterChain.doFilter(request, response);

System.out.println("Filter1 后");
```

Filter2：

```java
System.out.println("Filter2 前");

filterChain.doFilter(request, response);

System.out.println("Filter2 后");
```

那么整体效果可以理解为：

```text
Filter1 前
Filter2 前

Web资源执行

Filter2 后
Filter1 后
```

这就是过滤器链的基本执行特点。

# 18. 注解方式配置 Filter 的默认顺序

材料中指出：

> 使用注解方式配置 Filter 时，执行优先级按照过滤器类名字符串的自然排序。

例如有两个 Filter：

```text
AbcFilter
DemoFilter
```

由于：

```text
AbcFilter
```

在自然排序中更靠前，因此它会先执行。

也就是：

```text
AbcFilter
DemoFilter
```

的顺序。

# 19. Filter 类名顺序对过滤器链的影响

例如：

```java
@WebFilter(urlPatterns = "/*")
public class AbcFilter implements Filter {
}
```

以及：

```java
@WebFilter(urlPatterns = "/*")
public class DemoFilter implements Filter {
}
```

按照材料中的规则：

放行前：

```text
AbcFilter
DemoFilter
```

放行后：

```text
DemoFilter
AbcFilter
```

因此，要理解两个规则：

1. 注解方式 Filter 的进入顺序按照类名自然排序。
    
2. 放行后的逻辑按照进入顺序反向执行。
    

# 20. Filter 执行流程的核心规律

Filter 的执行流程可以归纳成：

## 20.1 单个 Filter

```text
放行前逻辑
Web资源
放行后逻辑
```

## 20.2 多个 Filter

如果进入顺序是：

```text
Filter1
Filter2
Filter3
```

那么返回顺序就是：

```text
Filter3
Filter2
Filter1
```

其本质都是：

```java
filterChain.doFilter(...)
```

形成的嵌套调用关系。

# 21. Filter 执行流程在实际开发中的意义

理解 Filter 的执行流程后，就能知道不同逻辑应该写在哪里。

例如：

放行前可以执行：

```text
登录校验
请求预处理
```

而放行后可以执行：

```text
请求结束后的处理逻辑
```

多个 Filter 之间还可以分别负责不同的公共功能，共同组成过滤器链。

# 22. 当前阶段需要掌握的核心内容

1. Filter 拦截请求后，需要调用 `filterChain.doFilter()` 才能继续访问后续资源。
    
2. `filterChain.doFilter()` 表示放行请求。
    
3. `filterChain.doFilter()` 之前的代码属于放行前逻辑。
    
4. 后续 Web 资源执行完成后，会重新回到 Filter。
    
5. `filterChain.doFilter()` 后面的代码属于放行后逻辑。
    
6. 不执行 `filterChain.doFilter()`，请求就不会继续访问后面的资源。
    
7. `@WebFilter` 的 `urlPatterns` 可以配置 Filter 的拦截路径。
    
8. `/login` 表示拦截具体路径。
    
9. `/emps/*` 表示目录拦截。
    
10. `/*` 表示拦截所有资源。
    
11. 一个 Web 应用中可以配置多个 Filter。
    
12. 多个 Filter 会组成过滤器链。
    
13. Filter 在放行阶段按照顺序依次执行。
    
14. Web 资源执行完成后，Filter 的放行后逻辑按照相反顺序执行。
    
15. 材料中使用注解配置 Filter 时，其默认执行顺序按照过滤器类名的自然排序。
    

> [!tip] 一句话总结
> 
> `Filter 通过 filterChain.doFilter() 控制请求放行，放行前后可以分别执行处理逻辑；多个 Filter 会组成过滤器链，进入时依次执行，资源处理完成后再按相反顺序执行放行后的逻辑。`