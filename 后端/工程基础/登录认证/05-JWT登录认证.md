# 1. JWT 登录认证要解决什么问题

前面的基础登录已经能够完成：

```text
用户名正确
密码正确
→ 登录成功
```

但是登录成功之后，后续请求仍然需要一个能够证明用户身份的凭证。

因此，JWT 登录认证需要完成两个核心动作：

```text
登录成功后生成 JWT
后续请求携带 JWT
```

材料中也明确将 JWT 登录认证分成两个阶段：

1. 登录成功后生成令牌并返回前端。
    
2. 后续请求中携带令牌，由服务端进行校验。
    

# 2. 登录成功后生成 JWT

在当前实现中，JWT 的生成发生在：

```java
EmpServiceImpl.login()
```

方法中。

基础登录时，代码只是根据用户名和密码查询员工。

现在需要在：

```text
登录成功
```

之后增加 JWT 生成逻辑。

# 3. JwtUtils 工具类

为了避免每次都重复编写 JWT 生成和解析代码，材料中单独定义了：

```java
JwtUtils
```

工具类。

它主要封装两个方法：

```java
generateJwt()
parseJWT()
```

分别负责：

- 生成 JWT
    
- 解析 JWT
    

# 4. JwtUtils 的基本结构

材料中的工具类如下：

```java
public class JwtUtils {

    private static String signKey = "SVRIRUlNQQ==";

    private static Long expire = 43200000L;

    public static String generateJwt(
            Map<String, Object> claims) {

        String jwt = Jwts.builder()
                .addClaims(claims)
                .signWith(
                    SignatureAlgorithm.HS256,
                    signKey
                )
                .setExpiration(
                    new Date(
                        System.currentTimeMillis()
                        + expire
                    )
                )
                .compact();

        return jwt;
    }

    public static Claims parseJWT(String jwt) {

        Claims claims = Jwts.parser()
                .setSigningKey(signKey)
                .parseClaimsJws(jwt)
                .getBody();

        return claims;
    }
}
```

# 5. signKey

工具类中定义：

```java
private static String signKey = "SVRIRUlNQQ==";
```

它表示：

> JWT 的签名密钥。

这个密钥会同时用于：

```text
生成 JWT
解析 JWT
```

前面已经知道，生成和解析 JWT 时使用的签名密钥必须匹配。

# 6. expire

工具类中还定义：

```java
private static Long expire = 43200000L;
```

用于表示：

> JWT 的有效时间。

生成 JWT 时，会根据当前时间加上这个有效期设置 Token 的过期时间。

# 7. generateJwt()

生成 JWT 的方法是：

```java
public static String generateJwt(
        Map<String, Object> claims)
```

调用：

```java
Jwts.builder()
```

构建 JWT。

然后：

```java
.addClaims(claims)
```

添加 Payload 数据。

通过：

```java
.signWith(
    SignatureAlgorithm.HS256,
    signKey
)
```

设置签名。

最后设置过期时间并调用：

```java
.compact()
```

得到最终 JWT 字符串。

# 8. 登录成功后准备 Claims

在：

```java
EmpServiceImpl.login()
```

中，如果已经根据用户名和密码查询到了员工：

```java
empLogin != null
```

就说明登录成功。

此时创建：

```java
Map<String, Object> dataMap =
        new HashMap<>();
```

并存入：

```java
dataMap.put(
    "id",
    empLogin.getId()
);

dataMap.put(
    "username",
    empLogin.getUsername()
);
```

这里保存的：

```text
id
username
```

最终会进入 JWT 的 Payload。

# 9. 调用 JwtUtils 生成 Token

准备好 Claims 后：

```java
String jwt =
        JwtUtils.generateJwt(dataMap);
```

即可得到当前登录用户对应的 JWT。

这个 JWT 就是后续用于证明用户身份的：

> Token。

# 10. 把 JWT 放入 LoginInfo

生成 JWT 后，需要将它返回给前端。

材料中通过：

```java
LoginInfo
```

进行封装：

```java
LoginInfo loginInfo =
        new LoginInfo(
            empLogin.getId(),
            empLogin.getUsername(),
            empLogin.getName(),
            jwt
        );
```

与基础登录最大的区别就是：

之前：

```java
token = null
```

现在：

```java
token = jwt
```

# 11. 完整登录方法

材料中的完整逻辑可以整理为：

```java
@Override
public LoginInfo login(Emp emp) {

    Emp empLogin =
            empMapper.getUsernameAndPassword(emp);

    if (empLogin != null) {

        Map<String, Object> dataMap =
                new HashMap<>();

        dataMap.put(
            "id",
            empLogin.getId()
        );

        dataMap.put(
            "username",
            empLogin.getUsername()
        );

        String jwt =
                JwtUtils.generateJwt(dataMap);

        return new LoginInfo(
                empLogin.getId(),
                empLogin.getUsername(),
                empLogin.getName(),
                jwt
        );
    }

    return null;
}
```

这个方法已经完成：

```text
验证用户名密码
生成 JWT
返回登录用户信息和 Token
```

# 12. 登录成功后的响应结果

登录接口执行成功后，返回的数据中会包含：

```text
id
username
name
token
```

其中：

```text
token
```

就是服务器生成的 JWT。

前端拿到这个 JWT 后，需要保存起来，以便后续请求继续使用。

# 13. 前端保存 JWT

材料中明确指出，当前案例把 JWT 保存到了浏览器：

```text
localStorage
```

中。

因此登录成功后，可以理解为：

> 前端获得 Token，并把它保存到浏览器本地存储中。

后续只要浏览器还保存这个 Token，就可以继续在请求中使用。

# 14. localStorage 在当前案例中的作用

在当前材料中，`localStorage` 用来：

> 保存服务器返回的 JWT。

这样就不需要每次请求前重新登录。

后续请求时，前端可以从：

```text
localStorage
```

读取 Token，然后放到请求中发送给服务端。

# 15. 后续请求携带 Token

材料中通过浏览器开发者工具观察到：

> 后续请求都会在请求头中携带 JWT。

当前案例中的请求头名称为：

```text
token
```

例如访问部门查询接口时，请求头中会携带：

```text
token: JWT令牌
```

服务端之后就可以从请求头获取这个 Token。

# 16. JWT 登录认证中的身份数据

当前 JWT Payload 中保存的是：

```text
id
username
```

也就是说，JWT 不只是证明：

> 当前用户已经登录。

还可以在后续解析 Token 后重新获得：

```text
员工 ID
用户名
```

等身份信息。

这也体现了 JWT：

**自包含**

的特点。

# 17. 为什么把用户 ID 放进 JWT

当前材料中把：

```java
empLogin.getId()
```

保存到了 JWT 中。

这样后续服务端解析 JWT 后，可以获得：

> 当前请求对应的员工 ID。

这个信息以后可以继续用于：

- 确定当前登录用户
    
- 记录操作人
    
- 其他需要用户身份的业务逻辑
    

当前材料后面的登录校验主要关注 Token 是否合法，而员工 ID 的进一步使用会在具体业务场景中体现。

# 18. JWT 登录认证目前完成到什么程度

到当前阶段，已经完成：

## 18.1 登录时

```text
用户提交用户名密码

服务端验证身份

登录成功

生成 JWT

返回 JWT 给前端
```

## 18.2 客户端

```text
接收 JWT

保存到 localStorage

后续请求携带 JWT
```

但是服务端目前还缺少最后一步：

> **对后续请求携带的 JWT 进行统一校验。**

这就是后面 Filter 和 Interceptor 要解决的问题。

# 19. JWT 登录认证与基础登录的区别

基础登录只完成：

```text
用户名 + 密码
→ 查询数据库
→ 判断是否登录成功
```

加入 JWT 后则变成：

```text
用户名 + 密码
→ 验证身份
→ 生成 JWT
→ 返回给前端
→ 前端保存
→ 后续请求携带
```

因此 JWT 让：

> 一次登录成功

变成了：

> 后续请求可以继续携带身份凭证。

# 20. 当前阶段需要掌握的核心内容

1. JWT 登录认证建立在基础用户名密码登录之上。
    
2. 登录成功后需要生成 JWT。
    
3. 当前材料通过 `JwtUtils` 封装 JWT 的生成和解析逻辑。
    
4. `generateJwt()` 用于生成 JWT。
    
5. JWT Payload 中保存了用户 ID 和用户名。
    
6. 生成的 JWT 被封装到 `LoginInfo.token` 中返回前端。
    
7. 当前案例将 JWT 保存到浏览器 `localStorage`。
    
8. 后续请求会把 JWT 放到请求头中携带到服务端。
    
9. 当前案例使用的请求头名称是 `token`。
    
10. JWT 可以在后续解析时重新获得用户身份信息。
    
11. 当前阶段已经完成 Token 的生成、返回、保存和传递。
    
12. 后续还需要通过 Filter 或 Interceptor 对请求中的 JWT 进行统一校验。
    

> [!tip] 一句话总结
> 
> `JWT登录认证是在用户名密码验证成功后生成包含用户身份信息的 JWT，将其通过 LoginInfo 返回给前端并保存到 localStorage，后续请求再通过请求头携带该令牌完成身份状态传递。`