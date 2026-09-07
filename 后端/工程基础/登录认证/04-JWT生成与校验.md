# 1. 使用 JWT 前需要引入依赖

材料中使用的是：

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

引入之后，就可以使用 `jjwt` 提供的 API 来完成 JWT 的生成和解析。

# 2. JWT 的核心工具类

材料中主要使用：

```java
Jwts
```

这个工具类。

它主要用于两个操作：

- 生成 JWT
    
- 解析和校验 JWT
    

因此可以先记住两个核心入口：

```java
Jwts.builder()
```

用于生成令牌。

```java
Jwts.parser()
```

用于解析令牌。

# 3. Claims 是什么

在生成 JWT 时，需要准备需要放入 Payload 中的数据。

材料中使用：

```java
Map<String, Object> claims = new HashMap<>();
```

保存这些数据。

例如：

```java
claims.put("id", 10);
claims.put("username", "huaishi");
```

这里的：

```text
id
username
```

都会作为 JWT Payload 中携带的数据。

这些 JWT 中携带的数据通常称为：

**Claims**

也就是令牌中保存的信息。

# 4. 生成 JWT

材料中的生成代码如下：

```java
@Test
public void testGenJwt() {

    Map<String, Object> claims = new HashMap<>();

    claims.put("id", 10);
    claims.put("username", "huaishi");

    String jwt = Jwts.builder()
            .signWith(SignatureAlgorithm.HS256, "aXRjYXN0")
            .addClaims(claims)
            .setExpiration(
                    new Date(
                            System.currentTimeMillis()
                            + 12 * 3600 * 1000
                    )
            )
            .compact();

    System.out.println(jwt);
}
```

这段代码完成了 JWT 生成过程中的几个核心配置。

# 5. Jwts.builder()

生成 JWT 首先调用：

```java
Jwts.builder()
```

用于创建 JWT 构建对象。

后续的：

```java
signWith()
addClaims()
setExpiration()
compact()
```

都是在这个构建过程中继续配置 JWT。

# 6. 设置签名算法和密钥

材料中使用：

```java
.signWith(
    SignatureAlgorithm.HS256,
    "aXRjYXN0"
)
```

其中：

```java
SignatureAlgorithm.HS256
```

表示使用：

```text
HS256
```

签名算法。

而：

```text
aXRjYXN0
```

表示当前 JWT 使用的签名密钥。

这个密钥后续在解析 JWT 时还需要再次使用。

# 7. 添加自定义 Claims

通过：

```java
.addClaims(claims)
```

可以把之前准备的数据加入 JWT。

例如：

```java
claims.put("id", 10);
claims.put("username", "huaishi");
```

最终这些信息会进入 JWT 的 Payload。

# 8. 设置 JWT 过期时间

材料中使用：

```java
.setExpiration(
    new Date(
        System.currentTimeMillis()
        + 12 * 3600 * 1000
    )
)
```

设置 JWT 的有效期。

这里表示：

> 从当前时间开始，12 小时后过期。

JWT 中对应的过期时间通常会以：

```text
exp
```

字段存在于 Payload 中。

# 9. compact() 生成最终 Token

完成相关配置后，通过：

```java
.compact()
```

生成最终 JWT 字符串。

最终结果类似：

```text
eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MSwiZXhwIjoxNjcyNzI5NzMwfQ.fHi0Ub8npbyt71UqLXDdLyipptLgxBUg_mSuGJtXtBk
```

这个字符串就是最终可以发送给客户端的 JWT。

# 10. 生成结果中的信息

JWT 生成后，可以解析其中前两部分的信息。

材料中指出：

第一部分可以看到：

```text
HS256
```

签名算法。

第二部分可以看到：

```text
id
exp
```

等 Payload 数据。

由于前两部分经过 Base64 编码，因此可以直接被解码出来。

# 11. 解析 JWT

材料中的 JWT 解析代码如下：

```java
@Test
public void testParseJwt() {

    Claims claims = Jwts.parser()
            .setSigningKey("aXRjYXN0")
            .parseClaimsJws(
                "eyJhbGciOiJIUzI1NiJ9..."
            )
            .getBody();

    System.out.println(claims);
}
```

如果解析成功，就可以获取 JWT Payload 中的数据。

# 12. Jwts.parser()

解析 JWT 时首先调用：

```java
Jwts.parser()
```

用于创建 JWT 解析器。

然后继续配置：

```java
setSigningKey()
```

并调用：

```java
parseClaimsJws()
```

完成解析。

# 13. setSigningKey()

解析 JWT 时需要设置：

```java
.setSigningKey("aXRjYXN0")
```

这里使用的签名密钥必须与生成 JWT 时：

```java
.signWith(
    SignatureAlgorithm.HS256,
    "aXRjYXN0"
)
```

使用的密钥一致。

材料最后也特别强调：

> JWT 校验时使用的签名密钥，必须和生成 JWT 时使用的密钥配套。

# 14. parseClaimsJws()

通过：

```java
.parseClaimsJws(jwt)
```

可以对 JWT 进行解析和校验。

如果 JWT 合法：

> 解析成功。

如果 JWT 被篡改、已经过期或者其他校验失败：

> 解析过程会报错。

因此这个方法不仅是在“读取数据”，同时也承担 JWT 校验作用。

# 15. getBody()

JWT 解析成功后：

```java
.getBody()
```

可以获取 Payload 中的 Claims。

例如材料中的解析结果：

```text
{id=10, username=huaishi, exp=1701909015}
```

其中包含：

- `id`
    
- `username`
    
- `exp`
    

# 16. 如何判断 JWT 是否有效

材料中的判断方式非常直接：

> 如果解析 JWT 的过程中没有报错，说明 JWT 解析成功。

也就是说：

```text
解析成功
→ JWT 合法

解析报错
→ JWT 非法或已经失效
```

# 17. JWT 被篡改会发生什么

材料进行了一个测试：

修改 JWT Header 中的一个字符，然后再次解析。

结果：

> JWT 解析报错。

材料据此得出结论：

> 如果 JWT 中的内容被篡改，解析校验就会失败。

这也说明了 Signature 的作用：

> 校验 JWT 内容是否仍然与原始签名一致。

# 18. JWT 为什么能发现内容被篡改

生成 JWT 时，Signature 是根据：

- Header
    
- Payload
    
- 签名密钥
    
- 签名算法
    

共同计算得到的。

如果修改了 Header 或 Payload 中的内容：

> 原始 Signature 就无法再与修改后的数据对应。

因此解析校验会失败。

这一点和前面的：

```text
JWT令牌.md
```

中的 Signature 防篡改机制对应。

# 19. JWT 过期后会发生什么

材料还专门测试了 Token 过期。

首先将有效期设置为：

```java
.setExpiration(
    new Date(
        System.currentTimeMillis()
        + 60 * 1000
    )
)
```

表示：

> JWT 只在 60 秒内有效。

等待超过有效期后，再进行解析时：

> 程序会报错。

因此材料得出的结论是：

> JWT 过期之后会失效，解析时会被认为是非法 Token。

# 20. JWT 校验失败的常见情况

根据当前材料，可以归纳出几种校验失败情况。

## 20.1 Token 被篡改

JWT 中的内容被修改。

结果：

```text
解析失败
```

## 20.2 Token 已经过期

当前时间超过：

```text
exp
```

表示的有效期。

结果：

```text
解析失败
```

## 20.3 签名密钥不匹配

生成 JWT 和解析 JWT 使用了不同的密钥。

结果：

```text
无法通过签名校验
```

材料最后将这些注意事项总结为：

- 生成和校验 JWT 必须使用匹配的签名密钥。
    
- JWT 解析时报错，说明 Token 被篡改或者已经失效。
    

# 21. JWT 生成与校验的核心对应关系

可以把两部分对应起来理解：

|生成 JWT|校验 JWT|
|---|---|
|`Jwts.builder()`|`Jwts.parser()`|
|`signWith()`|`setSigningKey()`|
|`addClaims()`|`getBody()`|
|`setExpiration()`|自动检查是否过期|
|`compact()`|`parseClaimsJws()`|

这两组操作共同完成 JWT 的完整生命周期。

# 22. 当前阶段需要掌握的核心内容

1. 当前材料使用 `jjwt` 操作 JWT。
    
2. `Jwts.builder()` 用于生成 JWT。
    
3. Claims 用于保存 JWT Payload 中携带的数据。
    
4. `addClaims()` 用于添加自定义 Claims。
    
5. `signWith()` 用于设置签名算法和签名密钥。
    
6. 当前材料使用 `HS256` 签名算法。
    
7. `setExpiration()` 用于设置 JWT 过期时间。
    
8. `compact()` 用于生成最终 JWT 字符串。
    
9. `Jwts.parser()` 用于解析 JWT。
    
10. `setSigningKey()` 设置解析 JWT 使用的签名密钥。
    
11. 生成和解析 JWT 必须使用匹配的签名密钥。
    
12. `parseClaimsJws()` 用于解析并校验 JWT。
    
13. `getBody()` 可以获得 Payload 中的 Claims。
    
14. JWT 被篡改时解析会失败。
    
15. JWT 过期后解析也会失败。
    
16. 如果 JWT 解析过程中没有报错，可以认为当前 Token 校验成功。
    

> [!tip] 一句话总结
> 
> `JWT 可以通过 Jwts.builder() 配置 Claims、签名密钥和过期时间后生成，并通过 Jwts.parser() 使用相同密钥进行解析校验，Token 被篡改、过期或签名不匹配时都会导致解析失败。`