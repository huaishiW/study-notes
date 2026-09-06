# 1. 什么是 Bean 作用域

Bean 被 Spring IOC 容器管理之后，还需要确定一个问题：

> **同一种 Bean 在容器中应该存在多少个实例，以及什么时候创建新的实例。**

这就是 Bean 的**作用域（Scope）**。

作用域决定了 Bean 实例的使用范围和创建方式。

当前内容中指出，Spring 支持多种 Bean 作用域，其中默认使用的是：

```text
singleton
```

即单例作用域。

当前材料还说明 Spring 一共支持五种作用域，其中后三种需要在 Web 环境中才能生效，但当前材料中的作用域表格没有完整显示，因此这一阶段重点掌握实际演示的：

```text
singleton
prototype
```

即可。

# 2. singleton 单例作用域

## 2.1 singleton 是默认作用域

Spring IOC 容器中的 Bean 默认采用：

```text
singleton
```

作用域。

所谓 singleton，可以理解为：

> **同一个 Bean 在 Spring IOC 容器中默认只有一个实例对象。**

因此，如果多次从 IOC 容器中获取同一个 singleton Bean，通常得到的都是同一个对象。

例如：

```java
@RestController
@RequestMapping("/depts")
public class DeptController {

    @Autowired
    private DeptService deptService;

    public DeptController() {
        System.out.println("DeptController constructor ....");
    }
}
```

没有显式配置 `@Scope` 时，`DeptController` 默认就是 singleton Bean。

## 2.2 多次获取 singleton Bean

可以通过 `ApplicationContext` 多次获取同一种 Bean：

```java
@SpringBootTest
class ApplicationTests {

    @Autowired
    private ApplicationContext applicationContext;

    @Test
    public void testScope() {
        for (int i = 0; i < 10; i++) {
            DeptController deptController =
                    applicationContext.getBean(DeptController.class);

            System.out.println(deptController);
        }
    }
}
```

对于默认的 singleton Bean，多次获取时复用的是容器中已经存在的同一个 Bean 实例，而不是每次重新创建一个新的对象。

因此 singleton 的核心特点是：

```text
一个 Bean
+
一个 IOC 容器
→
通常维护一个实例
```

# 3. prototype 多实例作用域

如果不希望 Bean 使用默认的 singleton 作用域，可以通过：

```java
@Scope
```

修改 Bean 的作用域。

例如：

```java
@Scope("prototype")
@RestController
@RequestMapping("/depts")
public class DeptController {

    @Autowired
    private DeptService deptService;

    public DeptController() {
        System.out.println("DeptController constructor ....");
    }
}
```

这里：

```java
@Scope("prototype")
```

表示将 `DeptController` 设置为：

```text
prototype
```

作用域。

## 3.1 prototype 的特点

prototype 与 singleton 最大的区别在于：

> **每次使用该 Bean 时，都会创建一个新的 Bean 实例。**

因此，多次从 IOC 容器中获取 prototype Bean 时，不再一直复用同一个对象。

可以简单对比：

|作用域|Bean 实例特点|
|---|---|
|`singleton`|默认情况下，同一个 Bean 复用同一个实例|
|`prototype`|每次获取时创建新的实例|

# 4. 使用 @Scope 配置 Bean 作用域

Spring 可以通过：

```java
@Scope
```

指定 Bean 的作用域。

例如：

```java
@Scope("prototype")
@Service
public class UserServiceImpl {

}
```

表示：

> `UserServiceImpl` 不再使用默认 singleton，而是使用 prototype。

如果没有显式配置：

```java
@Scope
```

则默认按照：

```text
singleton
```

管理 Bean。

因此实际开发中，不需要为了使用 singleton 而专门编写：

```java
@Scope("singleton")
```

因为 singleton 本身就是默认值。

当前内容也明确指出，实际开发中的绝大部分 Bean 都采用 singleton，因此大多数情况下并不需要额外配置 `@Scope`。

# 5. Bean 的创建时机

Bean 作用域不仅影响实例数量，还会影响当前内容中观察到的 Bean 创建时机。

## 5.1 singleton 默认在容器启动时创建

默认情况下，singleton Bean 会在 IOC 容器启动过程中创建。

例如：

```java
@RestController
public class DeptController {

    public DeptController() {
        System.out.println("DeptController constructor ....");
    }
}
```

即使还没有主动通过：

```java
applicationContext.getBean(...)
```

获取这个 Bean，Spring 在容器启动时就会创建默认的 singleton Bean。

当前材料明确指出：

> 默认 singleton Bean 在容器启动时创建。

## 5.2 prototype 在使用时创建

prototype Bean 与 singleton 不同。

当前内容指出：

> prototype Bean 每一次使用时都会创建一个新的实例。

因此可以建立这样的区别：

|作用域|当前内容中的创建特点|
|---|---|
|`singleton`|默认在 IOC 容器启动时创建|
|`prototype`|每次使用时创建新的实例|

# 6. @Lazy 延迟初始化

虽然 singleton Bean 默认会在容器启动时创建，但 Spring 允许通过：

```java
@Lazy
```

改变这一默认行为。

例如：

```java
@Lazy
@Service
public class UserServiceImpl {

}
```

`@Lazy` 表示：

> **延迟初始化当前 Bean。**

也就是说，Bean 不再按照默认情况随着 IOC 容器启动立即创建，而是延迟到第一次使用时再进行初始化。

因此：

```text
singleton 默认情况
→ 容器启动时创建

singleton + @Lazy
→ 第一次使用时再创建
```

当前材料明确将 `@Lazy` 描述为对默认 singleton Bean 的延迟初始化方式。

需要注意：

> `@Lazy` 改变的是 Bean 的初始化时机，并不意味着 Bean 自动变成 prototype。

也就是说：

```java
@Lazy
@Service
public class UserServiceImpl {
}
```

仍然是 singleton Bean，只是创建时间被推迟。

# 7. singleton 与 prototype 的核心区别

当前阶段可以重点从以下几个方面区分二者：

|对比项|`singleton`|`prototype`|
|---|---|---|
|是否为默认作用域|是|否|
|实例数量特点|通常复用同一个 Bean 实例|每次获取创建新实例|
|默认创建时机|IOC 容器启动时|使用 Bean 时|
|是否需要 `@Scope`|通常不需要|需要配置 `@Scope("prototype")`|
|实际开发使用情况|绝大多数 Bean|相对较少|

实际项目中的 Controller、Service 等 Bean，大多数情况下都直接使用默认的 singleton 作用域即可。

因此：

> **没有明确的多实例需求时，一般不需要主动修改 Bean 的作用域。**

当前材料同样指出，实际开发中的绝大部分 Bean 都是单例 Bean。

# 8. 当前阶段需要掌握的核心内容

1. Bean 作用域决定 Bean 实例的使用范围和创建方式。
    
2. Spring 支持多种 Bean 作用域，当前重点掌握 `singleton` 和 `prototype`。
    
3. Spring IOC 容器中的 Bean 默认使用 `singleton`。
    
4. singleton Bean 默认复用同一个实例。
    
5. 默认 singleton Bean 通常在 IOC 容器启动时创建。
    
6. 可以使用 `@Lazy` 将 singleton Bean 延迟到第一次使用时初始化。
    
7. `@Lazy` 改变的是初始化时机，而不是 Bean 的作用域。
    
8. 可以通过 `@Scope` 修改 Bean 的作用域。
    
9. `@Scope("prototype")` 表示使用 prototype 作用域。
    
10. prototype Bean 每次使用时都会创建新的实例。
    
11. 实际开发中的绝大多数 Bean 都采用默认 singleton，因此通常不需要主动配置 `@Scope`。
    
12. 当前内容并没有系统展开 Bean 的完整初始化、销毁等生命周期过程，应将完整 Bean 生命周期作为后续独立知识继续学习。
    

> [!tip] 一句话总结
> 
> `Spring Bean默认采用singleton作用域并通常在IOC容器启动时创建，可以使用@Lazy延迟初始化；通过@Scope("prototype")可以将Bean设置为多实例模式，使其在每次使用时创建新的实例。`