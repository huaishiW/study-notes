# 1. 什么是 Bean

在 Spring 中，由 IOC 容器负责创建和管理的对象称为：

```text
Bean
```

Bean 本质上仍然是一个普通的 Java 对象，只不过它不再完全由程序员手动创建和管理，而是交给 Spring IOC 容器统一管理。

例如：

```java
@Component
public class UserServiceImpl implements UserService {
}
```

当这个类被 Spring 识别并注册到 IOC 容器后，对应的 `UserServiceImpl` 对象就是一个 Bean。

因此，可以把 Bean 理解为：

> **由 Spring IOC 容器负责管理的 Java 对象。**

---

# 2. 为什么需要声明 Bean

Spring 要想帮助我们管理一个对象，首先需要知道：

> **哪些对象需要进入 IOC 容器。**

如果只是定义一个普通 Java 类：

```java
public class UserServiceImpl {
}
```

Spring 默认并不会直接把它作为 Bean 管理。

因此，需要通过相应的方式告诉 Spring：

> 当前对象需要交给 IOC 容器管理。

目前主要接触两类 Bean 声明方式：

```text
基于组件注解声明 Bean
基于 @Bean 声明 Bean
```

它们解决的是同一个问题：

> **如何把对象交给 Spring IOC 容器管理。**

但是它们适合的使用场景不同。

---

# 3. 基于组件注解声明 Bean

对于自己编写的类，最常见的方式是在类上添加组件注解。

主要包括：

```java
@Component
@Controller
@Service
@Repository
```

Spring 通过组件扫描发现这些类，并将对应对象注册到 IOC 容器。

## 3.1 @Component

`@Component` 是最基础、最通用的组件注解。

例如：

```java
@Component
public class UserServiceImpl {
}
```

表示当前类需要交给 Spring IOC 容器管理。

如果一个类不属于明确的 Controller、Service 或 Dao 层，可以直接使用：

```java
@Component
```

声明。

## 3.2 @Controller

`@Controller` 主要用于：

> **控制层类**

例如：

```java
@Controller
public class UserController {
}
```

表示当前类属于 Controller 层，同时由 Spring IOC 容器管理。

控制层通常负责：

- 接收请求
    
- 调用业务逻辑
    
- 返回响应结果
    

在前后端分离项目中也经常使用：

```java
@RestController
```

它同样会使当前类成为 Spring 管理的 Bean。

## 3.3 @Service

`@Service` 主要用于：

> **业务逻辑层类**

例如：

```java
@Service
public class UserServiceImpl implements UserService {
}
```

虽然也可以使用：

```java
@Component
```

但 `@Service` 能够更加清楚地表达：

> 当前类属于业务逻辑层。

因此业务实现类通常优先使用 `@Service`。

## 3.4 @Repository

`@Repository` 主要用于：

> **数据访问层类**

例如：

```java
@Repository
public class UserDaoImpl implements UserDao {
}
```

表示当前类属于数据访问层，同时需要由 Spring 管理。

在使用 MyBatis 时，Mapper 接口通常会使用：

```java
@Mapper
```

因此在当前 MyBatis 项目中，`@Repository` 的直接使用相对较少。

## 3.5 四种组件注解的区别

|注解|主要作用|
|---|---|
|`@Component`|通用组件|
|`@Controller`|控制层|
|`@Service`|业务逻辑层|
|`@Repository`|数据访问层|

它们都能够使对应类进入 Spring IOC 容器。

主要区别在于：

> **语义和使用场景不同。**

如果一个类属于明确的程序层次，应优先使用对应的语义化注解。

---

# 4. 基于 @Bean 声明 Bean

组件注解适合声明我们自己能够修改的类。

但是实际项目中，还会使用很多第三方依赖提供的类。

例如某个第三方类：

```java
public class AliyunOSSOperator {

}
```

这个类的源码并不是当前项目直接维护的，因此通常无法直接在它上面添加：

```java
@Component
```

此时就可以使用：

```java
@Bean
```

将对应对象注册到 IOC 容器。

## 4.1 @Bean 的基本作用

例如：

```java
@Bean
public AliyunOSSOperator aliyunOSSOperator(
        AliyunOSSProperties ossProperties) {

    return new AliyunOSSOperator(ossProperties);
}
```

这里方法返回：

```java
AliyunOSSOperator
```

对象。

`@Bean` 表示：

> **将当前方法返回的对象注册到 Spring IOC 容器中。**

因此：

```java
return new AliyunOSSOperator(ossProperties);
```

虽然对象本身是通过 `new` 创建的，但是创建完成后会交给 Spring IOC 容器统一管理。

---

# 5. 为什么第三方类适合使用 @Bean

对于自己编写的类，可以直接修改源码：

```java
@Service
public class UserServiceImpl {
}
```

但是对于第三方依赖中的类：

```java
public class AliyunOSSOperator {
}
```

通常无法直接修改它的源码添加：

```java
@Component
```

此时可以在自己的配置代码中创建对象：

```java
@Bean
public AliyunOSSOperator aliyunOSSOperator(
        AliyunOSSProperties ossProperties) {

    return new AliyunOSSOperator(ossProperties);
}
```

这样即使第三方类本身没有任何 Spring 注解，也仍然可以成为 IOC 容器中的 Bean。

因此：

> **第三方 Bean 是 `@Bean` 非常典型的使用场景。**

---

# 6. 使用 @Configuration 集中管理 Bean

`@Bean` 方法可以写在 Spring 能够识别的配置位置中。

实际项目中，如果存在多个需要手动声明的 Bean，通常不建议全部堆放在启动类中。

可以专门创建配置类：

```java
@Configuration
public class OSSConfig {

    @Bean
    public AliyunOSSOperator aliyunOSSOperator(
            AliyunOSSProperties ossProperties) {

        return new AliyunOSSOperator(ossProperties);
    }
}
```

其中：

```java
@Configuration
```

表示：

> 当前类是一个 Spring 配置类。

而：

```java
@Bean
```

负责声明具体 Bean。

因此可以形成这样的职责划分：

```text
@Configuration
负责声明配置类

@Bean
负责声明配置类中的 Bean
```

对于第三方 Bean，将它们集中放在配置类中管理，可以让配置代码更加清晰。

---

# 7. @Bean 方法中的依赖注入

通过 `@Bean` 创建对象时，这个对象本身也可能依赖 IOC 容器中的其他 Bean。

例如：

```java
@Bean
public AliyunOSSOperator aliyunOSSOperator(
        AliyunOSSProperties ossProperties) {

    return new AliyunOSSOperator(ossProperties);
}
```

这里：

```java
AliyunOSSProperties ossProperties
```

并不是我们在调用方法时手动传入的。

Spring 会从 IOC 容器中寻找符合条件的 Bean，并注入到方法参数中。当前材料明确说明，如果第三方 Bean 依赖其他 Bean，可以直接在 Bean 定义方法中声明形参，容器会根据类型完成自动装配。

因此：

```text
@Bean 方法需要其他 Bean
        ↓
在方法参数中声明依赖
        ↓
Spring 从 IOC 容器中查找
        ↓
自动传入对应 Bean
```

例如：

```java
@Bean
public OrderService orderService(UserMapper userMapper) {
    return new OrderService(userMapper);
}
```

这里 `UserMapper` 如果已经存在于 IOC 容器中，Spring 就可以将它注入到：

```java
userMapper
```

参数中。

---

# 8. Bean 的名称规则

Bean 除了类型之外，还拥有自己的名称。

不同 Bean 声明方式，其默认名称规则也有所不同。

## 8.1 组件注解声明的 Bean 名称

例如：

```java
@Service
public class UserServiceImpl {
}
```

如果没有显式指定名称，默认 Bean 名称通常为：

```text
userServiceImpl
```

即：

> **类名首字母小写。**

也可以显式指定：

```java
@Service("userService")
public class UserServiceImpl {
}
```

此时 Bean 名称就是：

```text
userService
```

## 8.2 @Bean 声明的 Bean 名称

例如：

```java
@Bean
public AliyunOSSOperator aliyunOSSOperator(
        AliyunOSSProperties ossProperties) {

    return new AliyunOSSOperator(ossProperties);
}
```

如果没有显式指定名称，默认 Bean 名称是：

```text
aliyunOSSOperator
```

也就是：

> **`@Bean` 所标注的方法名。**

当前材料明确指出，可以通过 `@Bean` 的 `name` 或 `value` 属性指定 Bean 名称；如果没有指定，则默认使用方法名。

例如：

```java
@Bean("ossOperator")
public AliyunOSSOperator aliyunOSSOperator(
        AliyunOSSProperties ossProperties) {

    return new AliyunOSSOperator(ossProperties);
}
```

此时 Bean 名称为：

```text
ossOperator
```

因此可以整理为：

|声明方式|默认 Bean 名称|
|---|---|
|`@Component` 及其衍生注解|类名首字母小写|
|`@Bean`|方法名|

---

# 9. 两种 Bean 声明方式的区别

目前接触到的两种主要 Bean 声明方式可以这样理解：

|对比项|组件注解|`@Bean`|
|---|---|---|
|典型注解|`@Component`、`@Service` 等|`@Bean`|
|注解位置|类上|方法上|
|Bean 来源|通常是自己编写的类|常用于第三方类或手动创建对象|
|对象创建|Spring 根据组件类创建|Bean 方法中创建并返回|
|默认 Bean 名称|类名首字母小写|方法名|
|是否依赖组件扫描发现 Bean 类|是|Bean 类本身不需要通过组件扫描发现|

需要注意：

> `@Bean` 并不是“第三方类专用注解”。

第三方 Bean 只是它非常典型的使用场景。

`@Bean` 的本质是：

> **将方法返回的对象注册到 IOC 容器。**

---

# 10. 组件扫描与 Bean 声明

对于：

```java
@Component
@Controller
@Service
@Repository
```

这一类组件注解，仅仅声明还不够。

对应的类还需要：

> **被 Spring 的组件扫描发现。**

例如：

```java
@Service
public class UserServiceImpl {
}
```

如果 `UserServiceImpl` 不在 Spring 的组件扫描范围内，Spring 默认可能无法发现这个类，也就无法按照这种方式将它注册成 Bean。

因此，对于基于组件注解声明的 Bean，通常需要同时满足：

1. 类上存在组件注解；
    
2. 类位于组件扫描范围内。
    

组件扫描的具体规则由：

```text
03-组件扫描.md
```

单独整理。

而通过 `@Bean` 声明对象时，目标对象所属的类本身不需要添加 `@Component` 等组件注解。

---

# 11. 当前阶段需要掌握的核心内容

1. Bean 是由 Spring IOC 容器管理的 Java 对象。
    
2. Spring 必须先知道哪些对象需要交给 IOC 容器管理。
    
3. 自己编写的组件类通常可以通过 `@Component` 及其衍生注解声明 Bean。
    
4. `@Controller`、`@Service`、`@Repository` 都具有组件声明能力，但语义不同。
    
5. 对于无法直接修改源码的第三方类，可以使用 `@Bean` 注册对象。
    
6. `@Bean` 的作用是将方法返回的对象注册到 IOC 容器。
    
7. 第三方 Bean 是 `@Bean` 的典型应用场景，但 `@Bean` 并不只用于第三方类。
    
8. 可以通过 `@Configuration` 创建配置类，将需要手动声明的 Bean 集中管理。
    
9. `@Bean` 方法需要依赖其他 Bean 时，可以通过方法参数声明依赖，由 Spring 自动注入。
    
10. 组件注解声明的 Bean 默认名称通常是类名首字母小写。
    
11. `@Bean` 声明的 Bean 默认名称是方法名，也可以通过 `name` 或 `value` 指定名称。
    
12. 基于组件注解声明的类，还需要被组件扫描发现后才能正常注册。
    
13. `@Bean` 所返回对象的类本身不需要通过组件扫描来声明。
    

> [!tip] 一句话总结
> 
> `Spring可以通过@Component及其衍生注解或@Bean将对象注册到IOC容器，其中组件注解通常用于自己编写的类，而@Bean常用于无法直接添加组件注解的第三方类，并可配合@Configuration集中管理这些Bean。`