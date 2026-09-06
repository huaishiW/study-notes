# 1. Controller 如何返回响应数据

在 Spring Boot Web 项目中，Controller 不仅负责接收客户端请求，还需要将处理结果返回给客户端。

例如：

```java
@RestController
public class UserController {

    @RequestMapping("/list")
    public List<User> list() {
        return userList;
    }
}
```

这里 `list()` 方法直接返回了一个 `List<User>` 集合。

这个返回值最终可以直接作为响应数据发送给浏览器。

实现这一功能的关键注解是：

```java
@ResponseBody
```

---

# 2. @ResponseBody

## 2.1 基本作用

`@ResponseBody` 可以标注在 Controller 方法或者 Controller 类上。

它的主要作用是：

> 将 Controller 方法的返回值直接写入 HTTP 响应体中，并返回给客户端。

例如：

```java
@ResponseBody
@RequestMapping("/hello")
public String hello() {
    return "Hello";
}
```

这里方法返回的：

```text
Hello
```

会直接作为响应内容返回给浏览器。

---

## 2.2 返回普通数据

如果 Controller 方法返回的是字符串：

```java
@ResponseBody
@RequestMapping("/hello")
public String hello() {
    return "Hello Spring Boot";
}
```

那么这个字符串会直接作为响应内容发送给浏览器。

也就是说，方法的返回值不会被当作页面名称，而是直接作为响应数据。

---

## 2.3 返回对象或集合

如果 Controller 方法返回的是实体对象或者集合，例如：

```java
@ResponseBody
@RequestMapping("/user")
public User user() {
    return user;
}
```

或者：

```java
@ResponseBody
@RequestMapping("/list")
public List<User> list() {
    return userList;
}
```

Spring 会将对象或集合转换成：

```text
JSON
```

格式，然后再将 JSON 数据返回给浏览器。

因此，在前后端分离项目中，经常会直接返回：

- Java 对象
    
- List 集合
    
- Map
    
- 其他可以转换为 JSON 的数据
    

---

# 3. 为什么 Java 对象可以直接返回 JSON

Controller 方法中返回的本质上仍然是 Java 对象。

例如：

```java
public List<User> list() {
    return userList;
}
```

但是浏览器最终接收到的通常是类似这样的数据：

```json
[
  {
    "id": 1,
    "username": "tom",
    "name": "Tom"
  }
]
```

这是因为 `@ResponseBody` 会让 Spring 对返回值进行处理。

当返回值是实体对象或者集合时，Spring 会将其转换成 JSON 格式，然后写入 HTTP 响应体。

因此，Controller 中不需要手动把 Java 对象转换为 JSON 字符串。

---

# 4. @RestController

前面的 Controller 通常会这样定义：

```java
@RestController
public class UserController {
}
```

但是类中并没有显式添加：

```java
@ResponseBody
```

方法的返回值仍然可以直接响应给浏览器。

原因是：

```java
@RestController
```

本身就是一个组合注解。

它由两个注解组合而成：

```java
@Controller
@ResponseBody
```

因此可以简单理解为：

```java
@RestController = @Controller + @ResponseBody
```

---

# 5. @Controller 与 @ResponseBody

## 5.1 @Controller

`@Controller` 用于标识当前类是一个 Controller 请求处理类。

它主要告诉 Spring：

> 当前类需要作为 Web 控制器进行管理。

---

## 5.2 @ResponseBody

`@ResponseBody` 用于告诉 Spring：

> Controller 方法的返回值需要直接作为响应数据返回给客户端。

因此，两者负责的内容不同。

`@Controller` 负责声明 Controller，而 `@ResponseBody` 负责处理返回值。

---

# 6. @RestController 的作用

由于：

```java
@RestController = @Controller + @ResponseBody
```

所以在类上使用：

```java
@RestController
```

相当于同时完成了两件事情：

1. 将当前类声明为 Controller。
    
2. 让当前类中方法的返回值默认直接作为响应数据返回。
    

例如：

```java
@RestController
public class UserController {

    @RequestMapping("/list")
    public List<User> list() {
        return userList;
    }
}
```

虽然 `list()` 方法上没有：

```java
@ResponseBody
```

但因为类上已经使用了 `@RestController`，所以返回值仍然会直接响应给浏览器。

---

# 7. 类级别的 @ResponseBody

`@ResponseBody` 不仅可以标注在方法上，也可以标注在类上。

如果标注在类上：

```java
@Controller
@ResponseBody
public class UserController {
}
```

那么这个类中的所有 Controller 方法都会具备 `@ResponseBody` 的效果。

因此：

```java
@Controller
@ResponseBody
```

和：

```java
@RestController
```

在当前场景下具有类似的效果。

为了简化代码，通常直接使用：

```java
@RestController
```

---

# 8. 前后端分离项目中的使用方式

在前后端分离项目中，后端 Controller 通常不会直接返回页面，而是向前端返回数据。

常见的返回数据类型包括：

```text
String
Java 对象
List 集合
Map
```

这些数据最终通常以 JSON 的形式返回给前端。

因此，在前后端分离项目中，请求处理类一般直接使用：

```java
@RestController
```

这样就不需要在每个方法上重复添加：

```java
@ResponseBody
```

---

# 9. @RestController 与 @ResponseBody 的区别

虽然两者经常一起出现，但它们并不是同一个概念。

## 9.1 @ResponseBody

主要负责：

> 将 Controller 方法返回值直接写入响应体。

可以作用在：

- 方法上
    
- 类上
    

---

## 9.2 @RestController

主要用于：

> 声明一个以返回响应数据为主的 Controller。

它本身包含：

```java
@Controller
@ResponseBody
```

因此更加适合前后端分离开发。

---

# 10. 当前阶段需要掌握的核心内容

1. Controller 方法的返回值可以作为响应数据返回给客户端。
    
2. `@ResponseBody` 会将方法返回值直接写入 HTTP 响应体。
    
3. 返回普通字符串时，可以直接作为响应内容返回。
    
4. 返回实体对象或集合时，会被转换为 JSON 数据后返回。
    
5. `@RestController` 是一个组合注解。
    
6. `@RestController` 包含 `@Controller` 和 `@ResponseBody`。
    
7. 在类上使用 `@RestController` 后，通常不需要再在每个方法上单独添加 `@ResponseBody`。
    
8. 前后端分离项目中的 Controller 通常使用 `@RestController`。
    

> [!tip] 一句话总结
> 
> `@ResponseBody 用于将 Controller 方法的返回值直接写入响应体，而 @RestController 相当于 @Controller 与 @ResponseBody 的组合，因此非常适合直接向前端返回 JSON 数据的前后端分离项目。`