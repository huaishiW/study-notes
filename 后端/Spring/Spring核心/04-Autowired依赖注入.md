# 1. @Autowired 是什么

`@Autowired` 是 Spring 提供的依赖注入注解。

它的作用是：

> 让 Spring IOC 容器为当前类自动提供其运行时所依赖的对象。

例如：

```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;
}
```

这里 `UserController` 依赖 `UserService`，Spring 会从 IOC 容器中找到对应的 Bean，并注入到 `userService` 中。

因此，开发者不再需要手动编写：

```java
private UserService userService = new UserServiceImpl();
```

---

# 2. @Autowired 的默认注入规则

`@Autowired` 默认按照：

**类型**

进行自动装配。

也就是说，Spring 会根据变量或参数的类型，到 IOC 容器中寻找对应类型的 Bean。

例如：

```java
@Autowired
private UserService userService;
```

Spring 会查找 IOC 容器中类型为：

```java
UserService
```

的 Bean。

如果容器中存在一个符合条件的 Bean，就会完成依赖注入。材料中也明确指出，`@Autowired` 默认按照类型进行自动装配。

# 3. 属性注入

## 3.1 基本写法

属性注入是最直接的一种依赖注入方式。

例如：

```java
@RestController
public class UserController {

    @Autowired
    private UserService userService;
}
```

只需要在成员变量上添加：

```java
@Autowired
```

Spring 就会自动为这个属性注入对应对象。

## 3.2 优点

属性注入的优点是：

- 代码简洁
    
- 使用方便
    
- 开发速度快
    

材料中将它的主要优点概括为代码简洁、方便快速开发。

## 3.3 缺点

属性注入也存在一些问题：

- 类之间的依赖关系不够明显
    
- 可能会影响类本身的封装性
    

从代码结构上看，如果只观察构造函数，很难直接看出这个类运行时依赖哪些对象。

因此，属性注入虽然简洁，但规范性相对较弱。

# 4. 构造器注入

## 4.1 基本写法

构造器注入是通过构造方法接收依赖对象。

例如：

```java
@RestController
public class UserController {

    private final UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

Spring 在创建 `UserController` 对象时，会将需要的 `UserService` Bean 作为构造参数传入。

## 4.2 单构造函数时可以省略 @Autowired

如果当前类中只有一个构造函数，那么：

```java
@Autowired
```

可以省略。

例如：

```java
@RestController
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```

Spring 仍然可以完成依赖注入。

材料中特别强调，如果当前类只有一个构造函数，`@Autowired` 可以省略。

## 4.3 优点

构造器注入的主要优点是：

- 类的依赖关系更加清晰
    
- 可以提高代码的安全性
    
- 依赖对象可以声明为 `final`
    

通过构造函数，可以直接看出当前类运行所需要的依赖。

例如：

```java
public UserController(UserService userService)
```

看到这个构造函数，就能明确知道 `UserController` 依赖 `UserService`。

## 4.4 缺点

构造器注入的缺点是：

- 代码量相对更多
    
- 如果依赖对象过多，构造函数参数会比较多
    

如果一个类需要非常多的依赖对象，构造函数可能会变得比较臃肿。

# 5. Setter 注入

## 5.1 基本写法

Setter 注入是通过 Setter 方法完成依赖注入。

例如：

```java
@RestController
public class UserController {

    private UserService userService;

    @Autowired
    public void setUserService(UserService userService) {
        this.userService = userService;
    }
}
```

Spring 会调用对应的 Setter 方法，将 `UserService` Bean 注入进来。

## 5.2 优点

Setter 注入的优点是：

- 保持了类的封装性
    
- 依赖关系比较清晰
    

属性本身仍然可以保持为私有，通过方法完成赋值。

## 5.3 缺点

Setter 注入需要额外编写 Setter 方法，因此代码量会增加。

材料中也指出，它虽然能保持封装性，但会增加额外代码。

# 6. 三种注入方式对比

|注入方式|特点|优点|缺点|
|---|---|---|---|
|属性注入|在成员变量上使用 `@Autowired`|简洁、开发快|依赖关系不明显|
|构造器注入|通过构造函数传入依赖|依赖清晰、更加规范|参数过多时构造函数较长|
|Setter 注入|通过 Setter 方法注入|保持封装性|需要额外编写 Setter|

在实际开发中，属性注入和构造器注入使用得比较多。

材料中提到，官方更推荐构造器注入，因为这种方式更加规范；但很多项目中也会使用属性注入，因为写法更加简洁。

# 7. 属性注入与构造器注入的区别

属性注入更加简洁：

```java
@Autowired
private UserService userService;
```

而构造器注入会明确把依赖写在构造函数中：

```java
private final UserService userService;

public UserController(UserService userService) {
    this.userService = userService;
}
```

两种方式最终都可以完成依赖注入。

主要区别在于：

- 属性注入更偏向简洁
    
- 构造器注入更偏向规范和依赖关系明确
    

当前学习阶段，需要能够识别并理解这两种方式。

# 8. @Autowired 与 IOC 的关系

`@Autowired` 本身并不会创建对象。

它做的是：

> 从 IOC 容器中获取已经存在的 Bean，并将其注入到需要的位置。

因此，使用 `@Autowired` 的前提是：

需要注入的对象已经被 Spring IOC 容器管理。

例如：

```java
@Service
public class UserServiceImpl implements UserService {
}
```

先通过 `@Service` 将 `UserServiceImpl` 声明为 Bean。

然后：

```java
@Autowired
private UserService userService;
```

Spring 才能从 IOC 容器中找到对应 Bean 并完成注入。

所以可以理解为：

- Bean 声明负责把对象交给 IOC 容器
    
- `@Autowired` 负责把 Bean 注入到需要它的地方
    

# 9. 当前阶段需要掌握的核心内容

1. `@Autowired` 是 Spring 提供的依赖注入注解。
    
2. `@Autowired` 默认按照类型进行自动装配。
    
3. 属性注入直接在成员变量上使用 `@Autowired`。
    
4. 构造器注入通过构造方法接收依赖对象。
    
5. 如果类中只有一个构造函数，构造器上的 `@Autowired` 可以省略。
    
6. Setter 注入通过 Setter 方法完成依赖注入。
    
7. 属性注入写法简洁，构造器注入的依赖关系更加清晰。
    
8. 材料中更推荐构造器注入这种更加规范的方式。
    
9. `@Autowired` 不负责创建 Bean，只负责从 IOC 容器中寻找并注入已有 Bean。
    

> [!tip] 一句话总结
> 
> `@Autowired 用于从 Spring IOC 容器中自动注入依赖对象，默认按照类型匹配，常见方式包括属性注入、构造器注入和 Setter 注入，其中构造器注入的依赖关系更加清晰规范。`