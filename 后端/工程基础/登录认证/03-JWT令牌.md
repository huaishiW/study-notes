# 1. 什么是 JWT

JWT 全称：

```text
JSON Web Token
```

它是一种令牌格式。

材料中对 JWT 的描述是：

> JWT 定义了一种简洁、自包含的格式，用于在通信双方之间以 JSON 数据格式安全传输信息。

在当前登录认证场景中，JWT 主要用于：

> **表示已经登录成功的用户身份。**

登录成功后，服务器生成 JWT 并返回给前端。

后续请求再携带 JWT，服务端通过校验 JWT 判断用户身份是否合法。

# 2. JWT 的两个核心特点

材料中重点强调了 JWT 的两个特点：

```text
简洁
自包含
```

## 2.1 简洁

所谓简洁，是指：

> JWT 最终就是一个字符串。

因此它很容易在客户端和服务端之间传递。

例如可以放在：

- 请求头
    
- 请求参数
    

等位置进行传输。

材料中明确指出，JWT 是一个简单字符串，可以直接在请求参数或请求头中传递。

## 2.2 自包含

所谓自包含，是指：

> JWT 本身可以携带自定义数据。

例如可以在 JWT 中保存：

```text
用户 ID
用户名
```

材料中的例子就是：

```json
{
  "id": "1",
  "username": "Tom"
}
```

因此，JWT 不只是一个随机字符串，它内部还可以包含业务所需要的信息。

# 3. JWT 的组成

一个 JWT 由三个部分组成。

三个部分之间使用英文：

```text
.
```

进行分隔。

即：

```text
Header.Payload.Signature
```

材料中明确列出了这三部分：

1. Header
    
2. Payload
    
3. Signature
    

# 4. Header 头部

JWT 的第一部分是：

```text
Header
```

主要用于记录令牌本身的一些信息，例如：

- Token 类型
    
- 签名算法
    

材料中的示例：

```json
{
  "alg": "HS256",
  "type": "JWT"
}
```

其中：

```text
alg
```

表示签名算法。

```text
type
```

表示令牌类型。

# 5. Payload 有效载荷

JWT 的第二部分是：

```text
Payload
```

中文通常称为：

**有效载荷**

它主要用来保存：

> JWT 中需要携带的数据。

例如：

```json
{
  "id": "1",
  "username": "Tom"
}
```

这些数据可以是：

- 用户 ID
    
- 用户名
    
- 其他需要携带的信息
    

材料中也指出，Payload 可以保存自定义信息以及默认信息。

# 6. Signature 签名

JWT 的第三部分是：

```text
Signature
```

也就是：

**签名**

它的核心作用是：

> **防止 Token 被篡改。**

材料中说明，Signature 是根据：

- Header
    
- Payload
    
- 指定密钥
    
- 指定签名算法
    

计算得到的。

# 7. Signature 为什么重要

如果 JWT 中：

```text
Header
Payload
```

任意内容被修改，那么原来生成的 Signature 就无法再与修改后的内容匹配。

在服务端校验时就会失败。

因此，材料把 Signature 的作用总结为：

> 防止 JWT 被篡改，保证令牌可靠。

# 8. JWT 三部分各自负责什么

可以简单理解为：

```text
Header
描述“这个 Token 怎么签名”

Payload
描述“这个 Token 携带什么数据”

Signature
证明“前面的内容没有被篡改”
```

三者共同组成完整 JWT。

# 9. JWT 为什么能携带 JSON 数据

材料中指出，在生成 JWT 时，会对原始 JSON 数据进行：

```text
Base64
```

编码。

因此原始 JSON 数据最终会转成适合在字符串中传输的形式。

# 10. 什么是 Base64

材料中对 Base64 的描述是：

> 一种基于 64 个可打印字符表示二进制数据的编码方式。

也就是说，Base64 的作用主要是：

> 把原始数据转换成便于传输的文本形式。

# 11. Base64 不是加密

这一点非常重要。

材料明确指出：

> **Base64 是编码方式，不是加密方式。**

因为 Base64 编码的数据：

> 可以被重新解码出来。

因此 JWT 的 Header 和 Payload 并不是“看不到内容”。

只要进行 Base64 解码，就可以看到其中的数据。

# 12. JWT 中哪些部分可以被解析

材料指出：

- Header 可以被解析
    
- Payload 可以被解析
    
- Signature 不是简单的 Base64 编码结果
    

例如生成 JWT 后，可以看到：

Header 中的签名算法。

Payload 中的：

```text
id
exp
```

等信息。

而 Signature 是通过签名算法计算出来的，并不是简单的 Base64 编码。

# 13. JWT 的安全性来自哪里

JWT 的安全性并不是来自：

```text
Base64
```

因为 Base64 可以解码。

真正起到防篡改作用的是：

```text
Signature
```

也就是说：

```text
Base64
负责编码和传输

Signature
负责防止篡改
```

这是理解 JWT 时非常重要的区别。

# 14. Payload 中的数据是否保密

根据当前材料，可以明确知道：

> Payload 可以被解析出来。

因此 Payload 中的数据并不是依靠 JWT 本身进行隐藏。

材料中的示例也直接展示了：

```text
id
username
exp
```

等内容可以被解析。

所以当前阶段需要特别记住：

> JWT 的签名主要保证数据没有被篡改，并不等于 Payload 内容不可见。

# 15. JWT 与普通 Token 的关系

前面已经知道：

> Token 是用户身份令牌这一类方案的统称。

而 JWT 是：

> 当前材料采用的一种具体 Token 格式。

因此可以理解为：

```text
Token
表示令牌这一类认证机制

JWT
是其中一种具体实现
```

JWT 相比一个完全随机的普通字符串，多了：

- 标准结构
    
- 自包含数据
    
- 签名校验
    

# 16. JWT 在登录认证中的作用

在当前登录认证场景中，JWT 主要承担：

> 用户身份凭证

的作用。

登录成功后，可以把：

```text
用户 ID
用户名
```

等信息放到 Payload 中。

然后生成完整 JWT 返回给前端。

后续请求中，前端携带这个 JWT。

服务端对 JWT 进行校验和解析后，就可以判断：

> 当前请求对应的是哪个用户。

具体如何生成和校验 JWT，会在：

```text
JWT生成与校验.md
```

中继续整理。

# 17. 当前阶段需要掌握的核心内容

1. JWT 全称是 `JSON Web Token`。
    
2. JWT 是当前材料采用的一种 Token 实现形式。
    
3. JWT 最终表现为一个字符串。
    
4. JWT 具有简洁和自包含的特点。
    
5. JWT 中可以携带自定义数据。
    
6. JWT 由 `Header`、`Payload`、`Signature` 三部分组成。
    
7. 三部分之间使用 `.` 分隔。
    
8. Header 主要保存 Token 类型和签名算法。
    
9. Payload 主要保存需要携带的数据。
    
10. Signature 主要用于防止 Token 内容被篡改。
    
11. JWT 的 Header 和 Payload 会经过 Base64 编码。
    
12. Base64 是编码方式，不是加密方式。
    
13. Header 和 Payload 中的数据可以被解析出来。
    
14. JWT 的防篡改能力主要来自 Signature，而不是 Base64。
    

> [!tip] 一句话总结
> 
> `JWT 是一种由 Header、Payload 和 Signature 三部分组成的自包含令牌，其中 Header 和 Payload 通过 Base64 编码便于传输，而 Signature 用于防止令牌内容被篡改。`