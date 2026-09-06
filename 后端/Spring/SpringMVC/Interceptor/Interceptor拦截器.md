# 1. 什么是 Interceptor

`Interceptor` 中文称为：

**拦截器**

材料中指出：

> Interceptor 是 Spring 框架提供的一种动态拦截方法调用的机制，可以在 Controller 方法执行前后，根据业务需要执行预先设定的代码。

它的典型用途包括：

- 登录校验
    
- 权限判断
    
- 请求前后的统一处理
    

# 2. Interceptor 的核心作用

Interceptor 主要用于：

> **拦截进入 Spring MVC Controller 的请求。**

当请求准备执行某个 Controller 方法时，可以先经过拦截器。

拦截器可以决定：

- 是否允许继续执行 Controller
    
- Controller 执行完成后是否继续处理
    
- 整个请求结束后是否执行额外逻辑
    

因此它非常适合处理：

> 多个 Controller 都需要执行的通用逻辑。

# 3. 为什么登录校验适合使用 Interceptor

材料中指出，可以把登录校验统一写在拦截器中。

当请求进入系统后：

- 如果携带合法 JWT，则放行
    
- 如果没有登录或 JWT 非法，则拒绝请求
    

这样就不需要在每一个 Controller 方法中重复编写登录校验逻辑。

# 4. Interceptor 的基本使用步骤

材料将拦截器的使用分成两个步骤：

1. 定义拦截器
    
2. 注册配置拦截器
    

# 5. 定义自定义拦截器

自定义拦截器需要实现：

```java
HandlerInterceptor
```

接口。

例如：

```java
@Component
public class DemoInterceptor
        implements HandlerInterceptor {

}
```

材料中还将拦截器交给 Spring IOC 容器管理，因此使用：

```java
@Component
```

进行标记。

# 6. HandlerInterceptor 的三个核心方法

`HandlerInterceptor` 中常用的三个方法是：

```java
preHandle()
postHandle()
afterCompletion()
```

它们分别对应请求处理过程中的不同阶段。

# 7. preHandle()

```java
@Override
public boolean preHandle(
        HttpServletRequest request,
        HttpServletResponse response,
        Object handler)
        throws Exception {

    return true;
}
```

`preHandle()` 会在：

> **目标 Controller 方法执行之前**

调用。

它也是登录校验中最重要的方法。

# 8. preHandle() 的返回值

`preHandle()` 返回：

```java
boolean
```

这个返回值直接决定请求是否继续执行。

如果返回：

```java
true
```

表示：

> 放行，请求继续访问目标资源。

如果返回：

```java
false
```

表示：

> 不放行，请求被拦截。

材料中专门测试了将返回值修改为 `false`，此时请求不会继续访问后面的资源。

# 9. preHandle() 为什么最适合登录校验

登录校验必须发生在：

> Controller 业务方法执行之前。

因此可以在：

```java
preHandle()
```

中判断：

- 当前请求是否携带 Token
    
- Token 是否有效
    

如果合法：

```java
return true;
```

如果非法：

```java
return false;
```

这样就可以在业务代码执行前控制请求是否继续。

# 10. postHandle()

```java
@Override
public void postHandle(
        HttpServletRequest request,
        HttpServletResponse response,
        Object handler,
        ModelAndView modelAndView)
        throws Exception {

}
```

`postHandle()` 会在：

> **目标资源方法执行之后**

调用。

因此，它适合处理目标方法已经执行完成后的相关逻辑。

# 11. afterCompletion()

```java
@Override
public void afterCompletion(
        HttpServletRequest request,
        HttpServletResponse response,
        Object handler,
        Exception ex)
        throws Exception {

}
```

`afterCompletion()` 会在：

> **视图渲染完成之后执行，并且最后执行。**

材料中明确将它描述为整个拦截器执行流程中最后调用的方法。

# 12. 三个方法的执行时机

可以先整理成：

|方法|执行时机|
|---|---|
|`preHandle()`|Controller 方法执行之前|
|`postHandle()`|Controller 方法执行之后|
|`afterCompletion()`|视图渲染完成后，最后执行|

其中：

```java
preHandle()
```

还额外负责控制：

> 请求是否放行。

# 13. 自定义 Interceptor 示例

材料中的基础代码如下：

```java
@Component
public class DemoInterceptor
        implements HandlerInterceptor {

    @Override
    public boolean preHandle(
            HttpServletRequest request,
            HttpServletResponse response,
            Object handler)
            throws Exception {

        System.out.println("preHandle .... ");

        return true;
    }

    @Override
    public void postHandle(
            HttpServletRequest request,
            HttpServletResponse response,
            Object handler,
            ModelAndView modelAndView)
            throws Exception {

        System.out.println("postHandle ... ");
    }

    @Override
    public void afterCompletion(
            HttpServletRequest request,
            HttpServletResponse response,
            Object handler,
            Exception ex)
            throws Exception {

        System.out.println(
                "afterCompletion .... "
        );
    }
}
```

# 14. 为什么定义完 Interceptor 还不能直接使用

仅仅创建：

```java
DemoInterceptor
```

并实现：

```java
HandlerInterceptor
```

还不够。

还需要告诉 Spring MVC：

> 这个拦截器需要作用于哪些请求。

因此还需要：

**注册拦截器。**

# 15. WebMvcConfigurer

材料中通过：

```java
WebMvcConfigurer
```

完成 Spring MVC 相关配置。

首先创建配置类：

```java
@Configuration
public class WebConfig
        implements WebMvcConfigurer {

}
```

然后重写：

```java
addInterceptors()
```

注册自定义拦截器。

# 16. @Configuration

配置类使用：

```java
@Configuration
```

进行标记。

表示：

> 当前类是一个 Spring 配置类。

例如：

```java
@Configuration
public class WebConfig
        implements WebMvcConfigurer {

}
```

# 17. 注入自定义 Interceptor

因为前面：

```java
DemoInterceptor
```

已经通过：

```java
@Component
```

交给 Spring IOC 容器管理。

所以配置类中可以注入：

```java
@Autowired
private DemoInterceptor demoInterceptor;
```

然后将它注册到 Spring MVC 中。

# 18. addInterceptors()

注册拦截器需要重写：

```java
@Override
public void addInterceptors(
        InterceptorRegistry registry) {

}
```

其中：

```java
InterceptorRegistry
```

用于管理项目中的拦截器注册信息。

# 19. addInterceptor()

通过：

```java
registry.addInterceptor(demoInterceptor)
```

把：

```java
DemoInterceptor
```

注册到 Spring MVC。

例如：

```java
@Override
public void addInterceptors(
        InterceptorRegistry registry) {

    registry.addInterceptor(
            demoInterceptor
    );
}
```

# 20. addPathPatterns()

注册拦截器后，还需要指定：

> 当前拦截器拦截哪些请求。

材料中使用：

```java
.addPathPatterns("/**")
```

完整代码：

```java
registry.addInterceptor(demoInterceptor)
        .addPathPatterns("/**");
```

其中：

```text
/**
```

表示：

> 拦截所有请求。

# 21. 完整注册配置

材料中的配置代码如下：

```java
@Configuration
public class WebConfig
        implements WebMvcConfigurer {

    @Autowired
    private DemoInterceptor demoInterceptor;

    @Override
    public void addInterceptors(
            InterceptorRegistry registry) {

        registry.addInterceptor(
                demoInterceptor
        ).addPathPatterns("/**");
    }
}
```

这样，自定义 Interceptor 才会真正参与 Spring MVC 的请求处理。

# 22. Interceptor 中的“放行”

Interceptor 和 Filter 都存在：

> 放行

这个概念。

但是它们的写法不同。

Filter 中使用：

```java
filterChain.doFilter(...)
```

进行放行。

Interceptor 中则通过：

```java
return true;
```

表示放行。

因此：

```text
Filter
通过调用 doFilter() 放行

Interceptor
通过 preHandle() 返回 true 放行
```

这是两者使用时非常容易混淆的地方。

# 23. Interceptor 中的“不放行”

如果：

```java
preHandle()
```

返回：

```java
false
```

请求就不会继续执行目标 Controller 方法。

例如：

```java
@Override
public boolean preHandle(...) {

    return false;
}
```

材料测试后发现，请求被直接拦截，没有继续访问目标资源。

# 24. Interceptor 属于 Spring MVC

材料明确指出：

> 拦截器是 Spring 框架提供的，用于动态拦截控制器方法执行。

因此按照我们之前确定的目录，它应该放在：

```text
Java后端
└── Spring
    └── SpringMVC
        └── Interceptor
            └── Interceptor拦截器.md
```

而不是放在：

```text
JavaWeb基础
```

下面。

# 25. Interceptor 与 Filter 的定位区别

根据当前材料，可以先建立一个基础区别。

## 25.1 Filter

属于：

```text
JavaWeb
```

主要负责过滤 Web 请求。

## 25.2 Interceptor

属于：

```text
Spring MVC
```

主要负责拦截 Controller 方法执行。

因此虽然两者都能实现登录校验，但它们所属的技术体系不同。

具体的执行流程区别会在下一篇：

```text
Interceptor执行流程.md
```

中继续整理。

# 26. 当前阶段需要掌握的核心内容

1. Interceptor 是 Spring 提供的拦截机制。
    
2. Interceptor 可以在 Controller 方法执行前后执行统一逻辑。
    
3. 自定义拦截器需要实现 `HandlerInterceptor`。
    
4. `preHandle()` 在目标方法执行之前执行。
    
5. `preHandle()` 返回 `true` 表示放行。
    
6. `preHandle()` 返回 `false` 表示不放行。
    
7. `postHandle()` 在目标资源方法执行后执行。
    
8. `afterCompletion()` 在视图渲染完成后执行，并且最后执行。
    
9. 自定义 Interceptor 还需要注册后才能生效。
    
10. 可以通过实现 `WebMvcConfigurer` 完成 Spring MVC 配置。
    
11. `addInterceptors()` 用于注册拦截器。
    
12. `addInterceptor()` 用于添加自定义 Interceptor。
    
13. `addPathPatterns()` 用于配置需要拦截的请求。
    
14. `/**` 表示拦截所有请求。
    
15. Interceptor 属于 Spring MVC，而 Filter 属于 JavaWeb。
    
16. 登录校验通常可以放在 `preHandle()` 中完成。
    

> [!tip] 一句话总结
> 
> `Interceptor 是 Spring MVC 提供的请求拦截机制，自定义拦截器通过实现 HandlerInterceptor 在 Controller 执行前后进行处理，并通过 preHandle() 的返回值控制请求是否放行，再由 WebMvcConfigurer 完成注册配置。`