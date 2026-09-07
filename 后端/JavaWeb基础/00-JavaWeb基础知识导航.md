# JavaWeb 基础知识导航

[返回后端知识图谱总览](<../后端知识图谱总览.md>)

JavaWeb 基础部分围绕两个问题展开：请求之间如何保持身份状态，以及请求如何在进入业务代码前后被统一处理。

## 会话技术

| 顺序 | 笔记 | 作用 | 关联 |
| ---: | --- | --- | --- |
| 1 | [会话与会话跟踪](<会话技术/01-会话与会话跟踪.md>) | 说明 HTTP 无状态、会话跟踪的本质和 Cookie、Session、Token 三类方案。 | 是 Cookie、Session、Token 三篇笔记的共同入口，也连接[登录认证概述](<../工程基础/登录认证/01-登录认证概述.md>)。 |
| 2 | [Cookie 会话跟踪](<会话技术/02-Cookie会话跟踪.md>) | 解释浏览器自动保存和携带 Cookie 的过程。 | 为理解 Session 的 JSESSIONID 机制提供基础。 |
| 3 | [Session 会话跟踪](<会话技术/03-Session会话跟踪.md>) | 解释服务器端会话数据和 JSESSIONID 的关系。 | 与 Cookie 相连，并引出集群环境下 Token 的优势。 |
| 4 | [Token 令牌认证](<会话技术/04-Token令牌认证.md>) | 解释无状态令牌认证的生成、保存、携带和校验过程。 | 连接[JWT 令牌](<../工程基础/登录认证/03-JWT令牌.md>)和登录校验。 |

## Filter 请求处理

| 顺序 | 笔记 | 作用 | 关联 |
| ---: | --- | --- | --- |
| 1 | [Filter 过滤器](<Filter/Filter过滤器.md>) | 说明 Filter 的作用、三个生命周期方法、注册方式和拦截位置。 | 是过滤器链和登录校验实现的基础。 |
| 2 | [Filter 执行流程与过滤器链](<Filter/Fliter执行流程与过滤器链.md>) | 解释 `FilterChain`、放行、返回顺序、拦截路径和多 Filter 组合。 | 连接[登录校验实现](<../工程基础/登录认证/06-登录校验实现.md>)以及 Spring MVC 的 Interceptor。 |

## 请求处理关系

```text
会话与会话跟踪
 ├─→ Cookie
 ├─→ Session ─→ 集群问题
 └─→ Token ───→ JWT 登录认证

Filter 过滤器
      ↓
Filter 执行流程与过滤器链
      ↓
统一读取 Token、放行或拦截请求
```

## 跨领域连接

- [会话与会话跟踪](<会话技术/01-会话与会话跟踪.md>)回答“身份状态如何保存”，[登录认证概述](<../工程基础/登录认证/01-登录认证概述.md>)进一步回答“登录和后续校验如何组成完整流程”。
- [Token 令牌认证](<会话技术/04-Token令牌认证.md>)是概念层，[JWT 令牌](<../工程基础/登录认证/03-JWT令牌.md>)、[JWT 生成与校验](<../工程基础/登录认证/04-JWT生成与校验.md>)是具体实现层。
- [Filter 过滤器](<Filter/Filter过滤器.md>)属于 Servlet 请求链；[Interceptor 拦截器](<../Spring/SpringMVC/Interceptor/Interceptor拦截器.md>)属于 Spring MVC 请求链。两者在[登录校验实现](<../工程基础/登录认证/06-登录校验实现.md>)中形成方案对比。
- [Filter 执行流程与过滤器链](<Filter/Fliter执行流程与过滤器链.md>)中的放行和返回顺序，可以与[Interceptor 执行流程](<../Spring/SpringMVC/Interceptor/Interceptor执行流程.md>)中的前后执行顺序一起复习。

## 复习抓手

1. 先区分“保存身份状态”和“统一拦截请求”两个问题。
2. 再比较 Cookie、Session、Token 的状态存放位置和请求携带方式。
3. 最后把 Filter、Interceptor 和 Controller 放入一条请求链中理解。
