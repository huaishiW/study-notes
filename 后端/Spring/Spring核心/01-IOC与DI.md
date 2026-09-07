# 1. 为什么需要 IOC 和 DI

在程序中，如果一个类需要使用另一个类的对象，最直接的方式就是自己创建对象。

例如：

```java
private UserService userService = new UserServiceImpl();
```

这种方式虽然简单，但会让当前类直接依赖具体的实现类。

如果后续业务发生变化，需要把：

```java
UserServiceImpl
```

替换成另一个实现类，例如：

```java
UserServiceImpl2
```

那么原来创建对象的代码也必须修改。

类似的问题同样会出现在 Service 调用 Dao 的过程中。

当一个类直接创建并依赖另一个具体类时，类与类之间的依赖关系就会比较紧密，这种现象称为**耦合**。材料中也正是通过 Controller 直接 `new UserServiceImpl()` 的问题，引出了后续的解耦思想。

# 2. 传统对象创建方式的问题

假设 Controller 中直接创建 Service：

```java
private UserService userService = new UserServiceImpl();
```

这里 Controller 不仅需要知道：

- 自己需要 `UserService`
    
- 还需要知道应该使用 `UserServiceImpl`
    

也就是说，Controller 不仅依赖接口，还依赖了具体实现。

如果 Service 的实现发生变化，Controller 也要跟着修改。

因此，真正需要解决的问题是：

> **让使用对象的一方不再负责创建这个对象。**

对象的创建和对象的使用应该尽可能分离。

# 3. Spring 的解耦思路

Spring 的解决思路是提供一个专门的容器来管理对象。

程序中需要使用的对象，不再由业务代码自己通过 `new` 创建，而是统一交给容器创建和管理。

当某个类需要使用一个对象时，再由容器将这个对象提供给它。

例如，Controller 需要使用 `UserService`，那么 Controller 不再主动：

```java
new UserServiceImpl();
```

而是由 Spring 容器负责创建 `UserServiceImpl` 对象，并在 Controller 运行时把它提供给 Controller。

这就是 IoC 和 DI 要解决的核心问题。

# 4. IOC 控制反转

## 4.1 什么是 IOC

IOC 的全称是：

```text
Inversion Of Control
```

中文称为：

**控制反转**

它的核心含义是：

> **对象的创建控制权由程序自身转移给外部容器。**

传统方式下，对象由程序员主动创建：

```java
UserService userService = new UserServiceImpl();
```

使用 IoC 后，这个对象由 Spring 容器负责创建和管理。

材料中将这种负责创建和管理对象的容器称为：

- IOC 容器
    
- Spring 容器
    

## 4.2 “控制”指的是什么

这里的“控制”，主要指的是：

> **对象创建的控制权。**

原来是程序自己决定：

- 什么时候创建对象
    
- 创建哪个实现类
    
- 如何保存这个对象
    

使用 IoC 后，这些工作交给 Spring 容器完成。

因此叫作“控制反转”。

## 4.3 IOC 的核心作用

IOC 的核心作用是：

**把对象的创建和管理从业务代码中分离出来。**

这样，业务代码更加关注对象的使用，而不用关心对象具体如何创建。

# 5. IOC 容器

IOC 容器就是 Spring 中负责创建和管理对象的容器。

例如项目中存在：

```java
@Component
public class UserServiceImpl implements UserService {
}
```

通过相应注解，可以将这个类产生的对象交给 Spring 容器管理。

之后，程序需要使用 `UserService` 时，就可以由 Spring 容器提供对应对象。

所以可以把 IoC 容器简单理解为：

> **负责统一创建、保存和管理应用程序对象的容器。**

具体如何声明 Bean、Spring 如何扫描这些类，会在后续：

```text
Bean的声明.md
组件扫描.md
```

中单独整理。

# 6. Bean 对象

由 IOC 容器创建和管理的对象，称为：

```text
Bean
```

例如：

```java
@Component
public class UserServiceImpl implements UserService {
}
```

当这个类被 Spring 容器管理后，由 Spring 容器创建出的 `UserServiceImpl` 对象就是一个 Bean 对象。

因此：

> **Bean 本质上仍然是 Java 对象，只不过它的创建和管理工作交给了 Spring 容器。**

材料中也明确将“IOC 容器中创建、管理的对象”称为 Bean 对象。

# 7. DI 依赖注入

## 7.1 什么是 DI

DI 的全称是：

```text
Dependency Injection
```

中文称为：

**依赖注入**

它表示：

> **Spring 容器在程序运行时，为程序提供它所依赖的对象。**

例如 `UserController` 运行时需要：

```java
UserService
```

那么 Spring 容器就会把一个对应的 `UserService` 对象提供给它。

这就是依赖注入。

## 7.2 什么是“依赖”

如果一个类需要使用另一个对象才能完成自己的功能，就可以认为它依赖这个对象。

例如：

```java
public class UserController {

    private UserService userService;
}
```

`UserController` 中需要使用 `UserService`，因此：

```text
UserController 依赖 UserService
```

类似地：

```java
public class UserServiceImpl {

    private UserDao userDao;
}
```

则表示：

```text
UserServiceImpl 依赖 UserDao
```

## 7.3 什么是“注入”

“注入”指的是：

> 原本需要当前类自己创建的依赖对象，现在由 Spring 容器提供并赋值给当前类。

例如：

```java
@Autowired
private UserService userService;
```

这里 `UserController` 没有自己：

```java
new UserServiceImpl();
```

而是由 Spring 容器找到对应的 `UserService` Bean，并将它提供给 `userService`。

这就是依赖注入。

# 8. IOC 与 DI 的关系

IOC 和 DI 是两个紧密相关但含义不同的概念。

## 8.1 IOC 关注对象由谁创建

IOC 解决的是：

> **对象应该由谁创建和管理？**

答案是：

```text
Spring IOC 容器
```

也就是说，对象创建权从程序自身转移到了容器。

## 8.2 DI 关注对象如何被使用

DI 解决的是：

> **一个类需要另一个对象时，这个对象如何提供给它？**

答案是：

```text
由 Spring 容器注入
```

因此可以这样理解：

- IOC 负责**管理对象**
    
- DI 负责**提供对象**
    

两者共同实现对象创建和对象使用之间的解耦。

# 9. IOC 与 DI 的基本使用

## 9.1 将对象交给 IOC 容器

在材料中的入门示例中，通过：

```java
@Component
```

将实现类交给 IOC 容器管理。

例如：

```java
@Component
public class UserDaoImpl implements UserDao {
}
```

以及：

```java
@Component
public class UserServiceImpl implements UserService {
}
```

这样 Spring 就可以负责创建和管理这些对象。

## 9.2 注入需要的对象

当 Service 需要 Dao 时，可以使用：

```java
@Autowired
private UserDao userDao;
```

当 Controller 需要 Service 时，可以使用：

```java
@Autowired
private UserService userService;
```

这样，Controller 和 Service 都不需要自己通过 `new` 创建依赖对象，而是由 Spring 容器提供。

# 10. 使用 IOC 和 DI 后的变化

在没有 IoC 和 DI 时，Controller 可能需要这样编写：

```java
private UserService userService = new UserServiceImpl();
```

Service 中也可能直接创建 Dao：

```java
private UserDao userDao = new UserDaoImpl();
```

使用 IoC 和 DI 后，可以改成：

```java
@Autowired
private UserService userService;
```

以及：

```java
@Autowired
private UserDao userDao;
```

此时，业务类只需要声明自己需要什么类型的对象，不再主动负责创建具体实现对象。

对象的创建由 Spring 容器负责，对象之间的依赖由 Spring 容器负责注入。

这样可以降低不同层之间因直接创建具体实现类而产生的耦合。

# 11. 当前阶段需要掌握的核心内容

1. 直接通过 `new` 创建依赖对象，会让类与具体实现之间产生较强的耦合。
    
2. Spring 通过 IOC 容器统一创建和管理对象。
    
3. IOC 的全称是 `Inversion Of Control`，表示控制反转。
    
4. IOC 中“反转”的核心是将对象创建权从程序转移给容器。
    
5. IOC 容器中创建和管理的对象称为 Bean。
    
6. DI 的全称是 `Dependency Injection`，表示依赖注入。
    
7. DI 是由 Spring 容器向程序提供其运行时所需要的对象。
    
8. IOC 负责对象的创建和管理，DI 负责将所需对象提供给使用者。
    
9. IOC 和 DI 的目的都是降低对象之间的耦合，使程序更加容易维护和扩展。
    

> [!tip] 一句话总结
> 
> `IOC 将对象的创建和管理交给 Spring 容器，DI 再由容器把程序所依赖的对象提供给使用者，从而降低类与具体实现之间的耦合。`

## 相关笔记

- [Bean 的声明](<02-Bean的声明.md>)：继续学习如何把对象注册为 Bean。
- [组件扫描](<03-组件扫描.md>)：理解组件注解声明的 Bean 如何被 Spring 发现。
- [Autowired 依赖注入](<04-Autowired依赖注入.md>)：深入学习容器如何将依赖注入使用者。
- [Bean 作用域与创建时机](<06-Bean作用域与创建时机.md>)：补充 Bean 在容器中的作用范围和创建时机。
- [Spring Boot 自动配置概述](<../SpringBoot/自动配置/01-SpringBoot自动配置概述.md>)：了解自动配置的 Bean 最终仍由 IOC 容器管理。
