# 1. 什么是 Cookie

Cookie 是一种：

> **客户端会话跟踪技术。**

它的数据主要存储在：

**客户端浏览器**

中。

当浏览器第一次访问服务器时，服务器可以通过响应向浏览器设置一个 Cookie。

之后浏览器会保存这个 Cookie，并在后续请求中自动携带给服务器。

这样，服务器就可以利用 Cookie 在同一次会话的多次请求之间共享数据。

# 2. Cookie 如何实现会话跟踪

材料中的基本过程是：

用户第一次访问服务器，例如请求登录接口。

登录成功以后，服务端可以设置 Cookie，例如保存：

```text
用户名
用户 ID
```

服务器响应后，浏览器会自动保存 Cookie。

以后浏览器再次访问服务器时，又会自动把这个 Cookie 携带到请求中。

服务器收到 Cookie 后，就可以根据其中的数据判断：

> 当前请求是否和之前的请求存在关联。

这就是 Cookie 实现会话跟踪的基本思路。

# 3. Cookie 中可以保存什么

材料中举出的例子包括：

```text
当前登录用户名
当前用户 ID
```

例如：

```text
login_username = huaishi
```

以后浏览器访问其他接口时，就可以继续携带这个值。

服务端再通过 Cookie 获取之前保存的数据。

# 4. Cookie 的三个自动过程

材料中特别强调了 Cookie 使用过程中存在三个“自动”。

## 4.1 服务器自动将 Cookie 响应给浏览器

服务端设置 Cookie 后，会通过 HTTP 响应把 Cookie 发送给浏览器。

## 4.2 浏览器自动保存 Cookie

浏览器接收到 Cookie 后，会自动把它保存到浏览器本地。

## 4.3 浏览器自动携带 Cookie

以后再次请求对应服务器时，浏览器会自动把 Cookie 放到请求中发送给服务器。

因此，开发者不需要每次都手动完成 Cookie 的保存和发送。

# 5. 为什么 Cookie 可以自动传递

Cookie 是 HTTP 协议支持的一种机制。

HTTP 中提供了两个与 Cookie 直接相关的头：

```text
Set-Cookie
Cookie
```

分别承担 Cookie 的设置和携带。

# 6. Set-Cookie 响应头

服务器向浏览器设置 Cookie 时，会使用：

```text
Set-Cookie
```

响应头。

例如服务端设置：

```text
login_username = huaishi
```

服务器响应中会携带对应的：

```text
Set-Cookie
```

浏览器识别这个响应头后，会自动保存其中的 Cookie 数据。

# 7. Cookie 请求头

浏览器后续再次发送请求时，会通过：

```text
Cookie
```

请求头携带已经保存的 Cookie。

因此可以简单区分：

```text
Set-Cookie
服务器 → 浏览器

Cookie
浏览器 → 服务器
```

前者负责设置数据，后者负责把已有 Cookie 带回服务器。

# 8. Java 中设置 Cookie

材料中的示例代码如下：

```java
@GetMapping("/c1")
public Result cookie1(HttpServletResponse response) {

    response.addCookie(
            new Cookie("login_username", "huaishi")
    );

    return Result.success();
}
```

这里首先创建：

```java
new Cookie("login_username", "huaishi")
```

然后通过：

```java
response.addCookie(...)
```

将 Cookie 加入响应。

最终浏览器会通过响应中的：

```text
Set-Cookie
```

接收到这个 Cookie。

# 9. Java 中获取 Cookie

后续请求到达服务器时，可以通过：

```java
request.getCookies()
```

获取浏览器携带的所有 Cookie。

材料中的示例：

```java
@GetMapping("/c2")
public Result cookie2(HttpServletRequest request) {

    Cookie[] cookies = request.getCookies();

    for (Cookie cookie : cookies) {

        if (cookie.getName().equals("login_username")) {

            System.out.println(
                    "login_username: " + cookie.getValue()
            );
        }
    }

    return Result.success();
}
```

其中：

```java
cookie.getName()
```

用于获取 Cookie 名称。

```java
cookie.getValue()
```

用于获取 Cookie 的值。

# 10. Cookie 会话跟踪示例

第一次访问：

```text
/c1
```

服务端执行：

```java
response.addCookie(
        new Cookie("login_username", "huaishi")
);
```

浏览器收到响应后保存 Cookie。

之后再访问：

```text
/c2
```

浏览器会自动携带 Cookie。

服务端通过：

```java
request.getCookies()
```

就可以读取第一次请求中保存的数据。

因此实现了：

> 在两次 HTTP 请求之间共享数据。

# 11. Cookie 的优点

材料中给出的主要优点是：

> Cookie 是 HTTP 协议原生支持的技术。

很多过程浏览器都会自动完成。

例如：

- 自动解析 `Set-Cookie`
    
- 自动保存 Cookie
    
- 后续请求自动携带 Cookie
    

因此 Cookie 使用起来比较方便。

# 12. Cookie 的缺点

材料中列出了几个问题：

- 移动端 APP 中无法直接使用 Cookie
    
- 用户可以禁用 Cookie
    
- Cookie 不能跨域
    
- 安全性存在一定问题
    

# 13. 什么是跨域

材料中通过前后端分离部署介绍了跨域问题。

例如前端部署在：

```text
http://192.168.150.200
```

后端部署在：

```text
http://192.168.150.100:8080
```

浏览器访问前端页面后，再从前端页面请求后端接口。

由于前端和后端地址不同，就会形成跨域请求。

# 14. 如何判断是否跨域

材料中给出的判断维度有三个：

- 协议
    
- IP
    
- 端口
    

这三个维度中，只要有一个不同，就属于跨域。

例如：

```text
http://192.168.150.200
```

访问：

```text
https://192.168.150.200
```

协议不同，属于跨域。

---

```text
http://192.168.150.200
```

访问：

```text
http://192.168.150.100
```

IP 不同，属于跨域。

---

```text
http://192.168.150.200
```

访问：

```text
http://192.168.150.200:8080
```

端口不同，也属于跨域。

# 15. Cookie 与跨域问题

材料中指出，在前后端分离部署的场景下：

> Cookie 不能直接跨域使用。

如果前端页面和后端接口属于不同域，那么基于 Cookie 进行会话跟踪就会受到限制。

这也是当前材料后面继续比较 Session 和 Token 的原因之一。

# 16. Cookie 与会话跟踪的核心关系

Cookie 本身只是：

> 浏览器保存的一组数据。

但在会话跟踪场景中，可以利用它：

- 登录成功后保存身份相关信息
    
- 后续请求自动携带身份信息
    
- 服务端据此判断请求状态
    

因此，Cookie 可以作为多次请求之间共享状态的一种方式。

# 17. 当前阶段需要掌握的核心内容

1. Cookie 是客户端会话跟踪技术。
    
2. Cookie 数据主要保存在浏览器客户端。
    
3. 服务器可以通过响应向浏览器设置 Cookie。
    
4. 浏览器会自动保存服务器返回的 Cookie。
    
5. 后续请求中浏览器会自动携带 Cookie。
    
6. `Set-Cookie` 是服务器向浏览器设置 Cookie 的响应头。
    
7. `Cookie` 是浏览器向服务器携带 Cookie 的请求头。
    
8. Java 中可以通过 `response.addCookie()` 设置 Cookie。
    
9. 可以通过 `request.getCookies()` 获取浏览器携带的 Cookie。
    
10. Cookie 的优势是 HTTP 和浏览器原生支持，很多操作可以自动完成。
    
11. Cookie 存在移动端使用受限、用户可禁用以及跨域限制等问题。
    
12. 材料中通过协议、IP 和端口三个维度判断是否存在跨域。
    

> [!tip] 一句话总结
> 
> `Cookie 是一种客户端会话跟踪技术，服务器通过 Set-Cookie 将数据发送给浏览器，浏览器保存后会在后续请求中通过 Cookie 请求头自动携带这些数据，从而实现多次请求之间的状态共享。`