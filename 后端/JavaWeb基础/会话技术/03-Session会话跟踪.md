# 1. 什么是 Session

Session 是一种：

> **服务端会话跟踪技术。**

与 Cookie 主要把数据保存在浏览器客户端不同，Session 的数据主要保存在服务器端。

材料中明确指出：

> Session 存储在服务器端，并且 Session 的底层实际上是基于 Cookie 实现的。

# 2. Session 的基本作用

Session 可以在同一次会话的多次请求之间共享数据。

例如第一次请求时保存：

```text
loginUser = tom
```

之后同一个浏览器再次发起请求时，就可以从 Session 中读取之前保存的数据。

因此，Session 主要解决的是：

> **多次 HTTP 请求之间的数据共享问题。**

# 3. Session 为什么需要唯一标识

服务器中可能同时存在很多个 Session。

例如：

```text
浏览器 A → Session A
浏览器 B → Session B
浏览器 C → Session C
```

服务器必须知道：

> 当前请求到底应该使用哪一个 Session。

因此，每一个 Session 都会有一个唯一标识：

```text
Session ID
```

材料中也明确说明，每一个 Session 对象都有自己的 ID。

# 4. JSESSIONID

在 Java Web 中，Session ID 通常通过一个名为：

```text
JSESSIONID
```

的 Cookie 保存。

服务器创建 Session 后，会把 Session ID 通过响应头：

```text
Set-Cookie
```

返回给浏览器。

这个 Cookie 的名称就是：

```text
JSESSIONID
```

浏览器收到之后会自动保存。

# 5. Session 的工作过程

Session 会话跟踪可以分成几个阶段。

## 5.1 第一次请求服务器

浏览器第一次请求服务器时，如果需要使用 Session，服务器会获取 Session 对象。

如果当前还没有 Session：

> 服务器会创建一个新的 Session。

这个 Session 会拥有自己的 Session ID。

## 5.2 服务器返回 JSESSIONID

服务器响应数据时，会通过：

```text
Set-Cookie
```

把：

```text
JSESSIONID = Session ID
```

发送给浏览器。

浏览器随后自动保存这个 Cookie。

## 5.3 后续请求携带 JSESSIONID

浏览器后续再次访问服务器时，会自动携带：

```text
JSESSIONID
```

服务器拿到 Session ID 后，就可以根据这个 ID 找到对应的 Session 对象。

# 6. Session 与 Cookie 的关系

Session 虽然把真正的数据保存在服务端，但它仍然需要 Cookie 帮助识别客户端对应的 Session。

也就是说：

```text
Session 数据
保存在服务器

Session ID
通过 Cookie 保存在浏览器
```

因此，Session 的会话跟踪过程实际上依赖：

```text
JSESSIONID Cookie
```

来完成。

# 7. 为什么说 Session 底层依赖 Cookie

如果没有 Cookie，浏览器后续请求时就无法自动携带：

```text
JSESSIONID
```

服务器也就无法知道：

> 当前请求应该关联哪一个 Session。

因此材料明确指出：

> Session 底层基于 Cookie 实现，如果 Cookie 不可用，那么这种 Session 会话跟踪方案也会失效。

# 8. Java 中获取 Session

可以直接在 Controller 方法参数中获取：

```java
HttpSession session
```

例如：

```java
@GetMapping("/s1")
public Result session1(HttpSession session) {

    log.info("HttpSession-s1: {}", session.hashCode());

    session.setAttribute("loginUser", "tom");

    return Result.success();
}
```

这里 Spring 会把当前请求对应的 Session 对象注入进来。

# 9. 向 Session 中保存数据

可以使用：

```java
session.setAttribute("loginUser", "tom");
```

向当前 Session 中保存数据。

其中：

```text
loginUser
```

是属性名。

```text
tom
```

是保存的值。

保存之后，后续属于同一个 Session 的请求就可以继续读取。

# 10. 从 Session 中获取数据

材料中通过：

```java
HttpSession session = request.getSession();
```

获取当前请求对应的 Session。

然后调用：

```java
Object loginUser = session.getAttribute("loginUser");
```

读取之前保存的数据。

示例：

```java
@GetMapping("/s2")
public Result session2(HttpServletRequest request) {

    HttpSession session = request.getSession();

    Object loginUser =
            session.getAttribute("loginUser");

    return Result.success(loginUser);
}
```

# 11. 同一个浏览器为什么能获取同一个 Session

第一次请求时，服务器创建 Session，并返回：

```text
JSESSIONID
```

浏览器保存之后，第二次请求会自动携带相同的 `JSESSIONID`。

服务器根据这个 ID 找到同一个 Session 对象。

因此材料测试中，两次请求打印出的 Session：

```text
hashCode
```

是相同的。

同时，第一次请求中保存的数据，在第二次请求中也成功读取到了。

# 12. Session 的优点

材料中给出的主要优点是：

> **Session 数据存储在服务端，相对更安全。**

因为实际共享的数据不直接保存在客户端浏览器中。

# 13. Session 的缺点

材料中列出了几个主要问题：

- 服务器集群环境下无法直接使用 Session
    
- 移动端 APP 中无法直接使用 Cookie
    
- 用户可以禁用 Cookie
    
- Cookie 不能跨域
    

由于 Session 依赖 Cookie，因此 Cookie 存在的问题也会影响 Session。

# 14. 为什么集群环境下 Session 会出现问题

这是 Session 最重要的限制之一。

现代项目通常不会只部署一份，而是会部署在多台服务器上。

例如：

```text
Tomcat 1
Tomcat 2
Tomcat 3
```

前面再通过负载均衡服务器分发请求。

# 15. Session 在集群中的问题

假设用户第一次登录时，请求被负载均衡分发到了：

```text
Tomcat 1
```

于是 Session 被创建在：

```text
Tomcat 1
```

服务器上。

浏览器随后拿到了对应的：

```text
JSESSIONID
```

下一次请求时，负载均衡可能把请求分发到：

```text
Tomcat 2
```

此时浏览器虽然携带了相同的 `JSESSIONID`，但：

> Tomcat 2 中并不存在 Tomcat 1 创建的那个 Session。

因此服务器无法找到原来的会话数据。

# 16. Session 的根本问题

Session 数据默认保存在：

> 当前服务器自身。

在单机环境中，这通常没有问题。

但在集群环境中：

```text
不同服务器
拥有各自独立的 Session 数据
```

请求如果被分发到不同服务器，就可能无法共享原来的 Session。

因此材料指出：

> Session 在服务器集群环境下无法直接使用。

# 17. Cookie 与 Session 的区别

可以先做一个基础对比：

|对比项|Cookie|Session|
|---|---|---|
|数据主要存储位置|浏览器客户端|服务器端|
|浏览器是否保存数据|保存 Cookie 数据|主要保存 JSESSIONID|
|是否依赖 Cookie|本身就是 Cookie|底层依赖 Cookie|
|服务端存储压力|较小|需要保存 Session 数据|
|集群环境|不直接依赖服务端 Session|默认存在 Session 共享问题|

# 18. Session 为什么逐渐被 Token 替代

材料在比较 Cookie 和 Session 后指出，这两种传统会话技术在现代企业项目中存在较多问题。

尤其是：

```text
前后端分离
移动端
服务器集群
```

这些场景下，Session 使用起来存在明显限制。

因此材料后续继续引出了：

```text
Token 令牌技术
```

作为当前案例最终采用的会话跟踪方式。

# 19. 当前阶段需要掌握的核心内容

1. Session 是服务端会话跟踪技术。
    
2. Session 数据主要保存在服务器端。
    
3. 每一个 Session 都有自己的 Session ID。
    
4. Java Web 中通常使用 `JSESSIONID` 表示 Session ID。
    
5. `JSESSIONID` 通过 Cookie 保存在浏览器中。
    
6. 浏览器后续请求会自动携带 `JSESSIONID`。
    
7. 服务器根据 `JSESSIONID` 找到对应 Session。
    
8. `session.setAttribute()` 可以向 Session 中保存数据。
    
9. `session.getAttribute()` 可以读取 Session 中的数据。
    
10. Session 底层依赖 Cookie。
    
11. Session 的优势是实际会话数据保存在服务端。
    
12. Session 在服务器集群环境下存在会话数据无法直接共享的问题。
    
13. Cookie 不可用时，基于 Cookie 的 Session 会话跟踪也会受到影响。
    
14. 正因为这些问题，材料后续选择了 Token 令牌技术。
    

> [!tip] 一句话总结
> 
> `Session 是一种服务端会话跟踪技术，服务器保存实际会话数据，并通过浏览器 Cookie 中的 JSESSIONID 识别对应 Session，但由于它依赖 Cookie 且默认数据存储在单台服务器上，因此在集群环境中存在明显限制。`