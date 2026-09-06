# 1. 多 Bean 注入为什么会产生冲突

`@Autowired` 默认按照**类型**进行依赖注入。

例如：

```java
@Autowired
private UserService userService;
```

Spring 会到 IOC 容器中寻找类型为 `UserService` 的 Bean。

如果容器中只有一个符合条件的 Bean，那么 Spring 可以直接完成注入。

但是，如果存在多个相同类型的 Bean，例如：

```java
@Service
public class UserServiceImpl implements UserService {
}
```

和：

```java
@Service
public class UserServiceImpl2 implements UserService {
}
```

这两个类都实现了：

```java
UserService
```

那么 IOC 容器中就会存在多个 `UserService` 类型的 Bean。

此时 Spring 无法判断到底应该注入哪一个对象，因此会出现依赖注入冲突并导致程序启动失败。

# 2. 解决多 Bean 冲突的方式

当 IOC 容器中存在多个相同类型的 Bean 时，Spring 提供了三种常见解决方案：

- `@Primary`
    
- `@Qualifier`
    
- `@Resource`
    

它们的目的都是：

> 在多个候选 Bean 中明确指定最终要注入的对象。

# 3. @Primary

## 3.1 基本作用

`@Primary` 用于指定：

> 当存在多个相同类型的 Bean 时，优先使用当前 Bean。

例如：

```java
@Primary
@Service
public class UserServiceImpl implements UserService {
}
```

如果 IOC 容器中同时存在多个 `UserService` 类型的 Bean，那么 Spring 会优先选择带有：

```java
@Primary
```

的这个 Bean。

材料中将它定义为：当存在多个相同类型的 Bean 时，通过 `@Primary` 确定默认实现。

## 3.2 使用特点

`@Primary` 标注在 Bean 的实现类上。

它解决的是：

> 哪个 Bean 应该作为默认选择。

因此，如果某一个实现类在大多数情况下都应该被优先使用，可以为它添加 `@Primary`。

# 4. @Qualifier

## 4.1 基本作用

`@Qualifier` 用于：

> 指定当前需要注入哪一个具体名称的 Bean。

例如：

```java
@RestController
public class UserController {

    @Autowired
    @Qualifier("userServiceImpl")
    private UserService userService;
}
```

这里明确指定需要注入名称为：

```text
userServiceImpl
```

的 Bean。

## 4.2 必须配合 @Autowired 使用

材料中明确说明：

```java
@Qualifier
```

不能单独完成依赖注入，需要配合：

```java
@Autowired
```

一起使用。

例如：

```java
@Autowired
@Qualifier("userServiceImpl")
private UserService userService;
```

其中：

- `@Autowired` 负责执行依赖注入。
    
- `@Qualifier` 负责进一步指定要注入的 Bean 名称。
    

## 4.3 Bean 名称

如果没有显式指定 Bean 名称，Spring 默认通常使用：

```text
类名首字母小写
```

例如：

```java
public class UserServiceImpl
```

默认 Bean 名称为：

```text
userServiceImpl
```

因此：

```java
@Qualifier("userServiceImpl")
```

就可以指定这个 Bean。

# 5. @Resource

## 5.1 基本作用

`@Resource` 也可以完成依赖注入。

它主要按照：

**Bean 名称**

进行注入。

例如：

```java
@RestController
public class UserController {

    @Resource(name = "userServiceImpl")
    private UserService userService;
}
```

这里通过：

```java
name = "userServiceImpl"
```

指定需要注入的 Bean。

## 5.2 与 @Qualifier 的区别

使用 `@Qualifier` 时，需要配合：

```java
@Autowired
```

例如：

```java
@Autowired
@Qualifier("userServiceImpl")
private UserService userService;
```

而使用 `@Resource` 时：

```java
@Resource(name = "userServiceImpl")
private UserService userService;
```

可以直接完成注入，不需要再额外添加 `@Autowired`。

# 6. @Autowired 与 @Resource 的区别

材料中总结了两个主要区别。

## 6.1 注解来源不同

`@Autowired` 是：

```text
Spring 框架提供的注解
```

而 `@Resource` 是：

```text
JDK 提供的注解
```

## 6.2 默认匹配方式不同

`@Autowired` 默认：

```text
按照类型注入
```

`@Resource` 默认：

```text
按照名称注入
```

因此可以概括为：

|注解|默认匹配方式|
|---|---|
|`@Autowired`|按类型|
|`@Resource`|按名称|

# 7. 三种解决方案的区别

## 7.1 @Primary

`@Primary` 的思路是：

> 从多个 Bean 中指定一个默认优先 Bean。

例如：

```java
@Primary
@Service
public class UserServiceImpl implements UserService {
}
```

适合希望某个实现类成为默认实现的情况。

## 7.2 @Qualifier

`@Qualifier` 的思路是：

> 在注入位置明确指定 Bean 名称。

例如：

```java
@Autowired
@Qualifier("userServiceImpl")
private UserService userService;
```

适合不同位置需要使用不同实现类的情况。

## 7.3 @Resource

`@Resource` 也可以：

> 通过 Bean 名称直接指定需要注入的对象。

例如：

```java
@Resource(name = "userServiceImpl")
private UserService userService;
```

它不需要再配合 `@Autowired` 使用。

# 8. 三种方式对比

|方式|标注位置|主要作用|
|---|---|---|
|`@Primary`|Bean 实现类|指定默认优先 Bean|
|`@Qualifier`|注入位置|指定需要注入的 Bean 名称|
|`@Resource`|注入位置|按名称指定需要注入的 Bean|

其中：

```java
@Primary
```

更偏向于在 Bean 声明阶段确定默认实现。

而：

```java
@Qualifier
@Resource
```

更偏向于在具体注入位置决定到底使用哪个 Bean。

# 9. 当前阶段需要掌握的核心内容

1. `@Autowired` 默认按照类型进行依赖注入。
    
2. 当 IOC 容器中只有一个对应类型的 Bean 时，可以正常完成注入。
    
3. 当存在多个相同类型的 Bean 时，Spring 无法判断应该注入哪个对象，会出现冲突。
    
4. `@Primary` 可以指定某个 Bean 作为默认优先实现。
    
5. `@Qualifier` 可以按照 Bean 名称指定要注入的对象。
    
6. `@Qualifier` 必须配合 `@Autowired` 使用。
    
7. `@Resource` 可以通过名称直接指定需要注入的 Bean。
    
8. `@Autowired` 是 Spring 提供的注解，默认按类型注入。
    
9. `@Resource` 是 JDK 提供的注解，默认按名称注入。
    

> [!tip] 一句话总结
> 
> `当 IOC 容器中存在多个相同类型的 Bean 时，可以使用 @Primary 指定默认实现，或通过 @Qualifier、@Resource 按 Bean 名称明确指定要注入的对象。`