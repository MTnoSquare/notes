# Spring

Spring 框架包含众多模块，如 Core、Testing、Data Access、Web Servlet 等，其中 Core 是整个 Spring 框架的核心模块。Core 模块提供了 IoC 容器、AOP 功能、数据绑定、类型转换等一系列的基础功能，而这些功能以及其他模块的功能都是建立在 IoC 和 AOP 之上的，所以**IoC 和 AOP 是 Spring 框架的核心。**

## IOC

控制反转，属于一种面向对象编程的设计思想，维护对象直接的依赖关系，降低对象之间的耦合度。

IOC 实现的方式为 DI(依赖注入)

在具体的实现中，主要由三种注入方式：

1. **构造方法注入**

就是被注入对象可以在它的构造方法中声明依赖对象的参数列表，让外部知道它需要哪些依赖对象。然后，IoC Service Provider 会检查被注入的对象的构造方法，取得它所需要的依赖对象列表，进而为其注入相应的对象。构造方法注入方式比较直观，对象被构造完成后，即进入就绪状态，可以马上使用。

2. **setter 方法注入**

通过 setter 方法，可以更改相应的对象属性。所以，当前对象只要为其依赖对象所对应的属性添加 setter 方法，就可以通过 setter 方法将相应的依赖对象设置到被注入对象中。setter 方法注入虽不像构造方法注入那样，让对象构造完成后即可使用，但相对来说更宽松一些，
可以在对象构造完成后再注入。

3. **接口注入**(抛弃)

相对于前两种注入方式来说，接口注入没有那么简单明了。被注入对象如果想要 IoC Service Provider 为其注入依赖对象，就必须实现某个接口。这个接口提供一个方法，用来为其注入依赖对象。IoC Service Provider 最终通过这些接口来了解应该为被注入对象注入什么依赖对象。**相对于前两种依赖注入方式，接口注入比较死板和烦琐。**

## BeanFactory 和 ApplicationContext

|     BeanFactory      |   ApplicationContext   |
| :------------------: | :--------------------: |
|        懒加载        |        即时加载        |
|   显示提供资源对象   | 自己创建和管理资源对象 |
|     不支持国际化     |       支持国际化       |
| 不支持基于依赖的注解 |   支持基于依赖的注解   |

BeanFactory 优点：启动时占用资源少  
缺点：运行速度慢，可能出现空指针异常的错误

ApplicationContext 优点：所以 Bean 在启动时加载，运行速度快；在系统启动时，可以发现系统中的配置问题  
缺点：把费时的操作放到系统启动中完成，所有对象可以预加载，内存占用大

## Spring bean

> Spring 中默认提供的单例不是线程安全的，如果单例的 Bean 是一个有状态的 Bean，则可以采用 ThreadLocal 对状态数据做线程隔离，来保证线程安全。

### 管理 bean 常用注解

1. **@Component、@Repository、@Service、@Controller**

   用于声明 Bean，它们的作用一样，但是语义不同。@Component 用于声明通用的 Bean，@Repository 用于声明 DAO 层的 Bean，@Service 用于声明业务层的 Bean，@Controller 用于声明视图层的控制器 Bean，被这些注解声明的类就可以被容器扫描并创建。

2. **@Autowired、@Qualifier**

   用于注入 Bean，即告诉容器应该为当前属性注入哪个 Bean。其中，@Autowired 是按照 Bean 的类型进行匹配的，如果这个属性的类型具有多个 Bean，就可以通过@Qualifier 指定 Bean 的名称，以消除歧义。

3. **@Scope**

   用于声明 Bean 的作用域，默认情况下 Bean 是单例的，即在整个容器中这个类型只有一个实例。

4. **@PostConstruct、@PreDestroy**

   用于声明 Bean 的生命周期。其中，被@PostConstruct 修饰的方法将在 Bean 实例化后被调用，@PreDestroy 修饰的方法将在容器销毁前被调用。

5. **@Autowired 和@Resource**

   1. @Autowired 是 Spring 提供的注解，@Resource 是 JDK 提供的注解。
   2. @Autowired 是只能按类型注入，@Resource 默认按名称注入，也支持按类型注入。
   3. @Autowired 按类型装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许 null 值，可以设置它 required 属性为 false，如果我们想使用按名称装配，可以结合@Qualifier 注解一起使用。@Resource 有两个中重要的属性：name 和 type。name 属性指定 byName，如果没有指定 name 属性，当注解标注在字段上，即默认取字段的名称作为 bean 名称寻找依赖对象，当注解标注在属性的 setter 方法上，即默认取属性名作为 bean 名称寻找依赖对象。需要注意的是，@Resource 如果没有指定 name 属性，并且按照默认的名称仍然找不到依赖对象时， @Resource 注解会回退到按类型装配。但一旦指定了 name 属性，就只能按名称装配了。

### bean 的作用域

|     类型      |                                   说明                                   |
| :-----------: | :----------------------------------------------------------------------: |
|   singleton   |        在 Spring 容器中仅存在一个实例，即 Bean 以单例的形式存在。        |
|   prototype   |       每次调用 getBean()时，都会执行 new 操作，返回一个新的实例。        |
|    request    |                  每次 HTTP 请求都会创建一个新的 Bean。                   |
|    session    | 同一个 HTTP Session 共享一个 Bean，不同的 HTTP Session 使用不同的 Bean。 |
| globalSession |       同一个全局的 Session 共享一个 Bean，一般用于 Portlet 环境。        |

### bean 的生命周期

Spring 容器管理 Bean，涉及对 Bean 的创建、初始化、调用、销毁等一系列的流程，这个流程就是 Bean 的生命周期。整个流程参考下图：

![](https://static.nowcoder.com/images/activity/2021jxy/java/img/spring-1.jpg)

这个过程是由 Spring 容器自动管理的，其中有两个环节我们可以进行干预。

我们可以自定义初始化方法，并在该方法前增加@PostConstruct 注解，届时 Spring 容器将在调用 SetBeanFactory 方法之后调用该方法。
我们可以自定义销毁方法，并在该方法前增加@PreDestroy 注解，届时 Spring 容器将在自身销毁前，调用这个方法。

## Spring 解决循环依赖

spring 对循环依赖的处理有三种情况：

1. 构造器的循环依赖：这种依赖 spring 是处理不了的，直接抛出 BeanCurrentlylnCreationException 异常。
2. 单例模式下的 setter 循环依赖：通过“三级缓存”处理循环依赖。
3. 非单例循环依赖：无法处理。

Spring 单例对象的初始化大略分为三步：

1. createBeanInstance：实例化，其实也就是调用对象的构造方法实例化对象；
2. populateBean：填充属性，这一步主要是多 bean 的依赖属性进行填充；
3. initializeBean：调用 spring xml 中的 init 方法。

循环依赖主要发生在第一二步，也就是构造器循环依赖和 field 循环依赖。 Spring 为了解决单例的循环依赖问题，使用了三级缓存。

```java
/** Cache of singleton objects: bean name –> bean instance */
private final Map singletonObjects = new ConcurrentHashMap(256);
/** Cache of singleton factories: bean name –> ObjectFactory */
private final Map> singletonFactories = new HashMap>(16);
/** Cache of early singleton objects: bean name –> bean instance */
private final Map earlySingletonObjects = new HashMap(16);
```

这三级缓存的作用分别是：

- singletonFactories ： 进入实例化阶段的单例对象工厂的 cache （三级缓存）；**(Spring 解决循环依赖的核心)**
- earlySingletonObjects ：完成实例化但是尚未初始化的，提前暴光的单例对象的 Cache （二级缓存）；
- singletonObjects：完成初始化的单例对象的 cache（一级缓存）。

通过“A 的某个 field 或者 setter 依赖了 B 的实例对象，同时 B 的某个 field 或者 setter 依赖了 A 的实例对象”这种循环依赖的情况进行分析。

1. A 首先完成了初始化的第一步，并且将自己提前曝光到 singletonFactories(三级缓存)中，此时进行初始化的第二步，发现自己依赖对象 B，此时就尝试去 get(B)，发现 B 还没有被 create，所以走 create 流程
2. B 在初始化第一步的时候发现自己依赖了对象 A，于是尝试 get(A)，尝试一级缓存 singletonObjects(肯定没有，因为 A 还没初始化完全)，尝试二级缓存 earlySingletonObjects（也没有），尝试三级缓存 singletonFactories，由于 A 通过 ObjectFactory 将自己提前曝光了，所以 B 能够通过 ObjectFactory.getObject 拿到 A 对象(虽然 A 还没有初始化完全，但是总比没有好呀)
3. B 拿到 A 对象后顺利完成了初始化阶段 1、2、3，完全初始化之后将自己放入到一级缓存 singletonObjects 中。此时返回 A 中，A 此时能拿到 B 的对象顺利完成自己的初始化阶段 2、3，最终 A 也完成了初始化，进去了一级缓存 singletonObjects 中，而且更加幸运的是，由于 B 拿到了 A 的对象引用，所以 B 现在 hold 住的 A 对象完成了初始化。

## AOP

AOP（Aspect Oriented Programming）是面向切面编程，它是一种编程思想，是面向对象编程（OOP）的一种补充。面向对象编程将程序抽象成各个层次的对象，而面向切面编程是将程序抽象成各个切面。

### 实现

基于[代理模式](../../设计模式/代理模式.md)实现

Spring Aop 基于动态代理实现；Aspectj 基于静态代理方式实现。

> **Q:既然有没有接口都可以用 CGLIB，为什么 Spring 还要使用 JDK 动态代理？**
> A:在性能方面，CGLib 创建的代理对象比 JDK 动态代理创建的代理对象高很多。但是，CGLib 在创建代理对象时所花费的时间比 JDK 动态代理多很多。所以，对于单例的对象因为无需频繁创建代理对象，采用 CGLib 动态代理比较合适。反之，对于多例的对象因为需要频繁的创建代理对象，则 JDK 动态代理更合适。

### 应用场景

1. 事务管理
2. 日志
3. ......

## Spring 事务

Spring 支持两种事务编程模型：

1. 编程式事务

Spring 提供了 TransactionTemplate 模板，利用该模板我们可以通过编程的方式实现事务管理，而无需关注资源获取、复用、释放、事务同步及异常处理等操作。相对于声明式事务来说，这种方式相对麻烦一些，但是好在更为灵活，我们可以将事务管理的范围控制的更为精确。

2. 声明式事务

Spring 事务管理的亮点在于声明式事务管理，它允许我们通过声明的方式，在 IoC 配置中指定事务的边界和事务属性，Spring 会自动在指定的事务边界上应用事务属性。相对于编程式事务来说，这种方式十分的方便，只需要在需要做事务管理的方法上，增加@Transactional 注解，以声明事务特征即可。

### Spring 事务传播

当我们调用一个业务方法时，它的内部可能会调用其他的业务方法，以完成一个完整的业务操作。这种业务方法嵌套调用的时候，如果这两个方法都是要保证事务的，那么就要通过 Spring 的事务传播机制控制当前事务如何传播到被嵌套调用的业务方法中。

Spring 在 TransactionDefinition 接口中规定了 7 种类型的事务传播行为，它们规定了事务方法和事务方法发生嵌套调用时如何进行传播，如下表：

|       事务传播类型        |                                                说明                                                |
| :-----------------------: | :------------------------------------------------------------------------------------------------: |
|   PROPAGATION_REQUIRED    |     如果当前没有事务，则新建一个事务；如果已存在一个事务则加入到这个事务中。这是最常见的选择。     |
|   PROPAGATION_SUPPORTS    |                        支持当前事务，如果当前没有事务，则以非事务方式执行。                        |
|   PROPAGATION_MANDATORY   |                           使用当前的事务，如果当前没有事务，则抛出异常。                           |
| PROPAGATION_REQUIRES_NEW  |                           新建事务，如果当前存在事务，则把当前事务挂起。                           |
| PROPAGATION_NOT_SUPPORTED |                     以非事务方式执行操作，如果当前存在事务，则把当前事务挂起。                     |
|     PROPAGATION_NEVER     |                        以非事务方式执行操作，如果当前存在事务，则抛出异常。                        |
|    PROPAGATION_NESTED     | 如果当前存在事务，则在嵌套事务内执行；如果当前没有事务，则执行与 PROPAGATION_REQUIRED 类似的操作。 |

### 事务的坑

![](https://community-header-1306990603.cos.ap-guangzhou.myqcloud.com/20220905160207.png)

## Spring 设计模式

工厂模式：Spring通过BeanFactory，ApplicationContext创建bean对象

代理模式：Spring aop

单例模式：Spring中的Bean默认单例

模板方法模式：Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。

观察者模式：Spring事件驱动模型

适配器模式：Spring aop的增强或通知使用到了适配器模式，spring MVC 中也是用到了适配器模式适配Controller。

装饰者模式：我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源

# SpringMVC

## 工作原理

![](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg-blog.csdnimg.cn%2F20201108145728249.png%3Fx-oss-process%26%2361%3Bimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dobDg2MTRqb2hu%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_ce&refer=http%3A%2F%2Fimg-blog.csdnimg.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1648282726&t=b9892229ac8c6ac9cf7abec4b37b795a)

1. 整个过程开始于客户端发出的一个HTTP请求，Web应用服务器接收到这个请求。如果匹配DispatcherServlet的请求映射路径，则Web容器将该请求转交给DispatcherServlet处理。
2. DispatcherServlet接收到这个请求后，将根据请求的信息（包括URL、HTTP方法、请求报文头、请求参数、Cookie等）及HandlerMapping的配置找到处理请求的处理器（Handler）。可将HandlerMapping看做路由控制器，将Handler看做目标主机。值得注意的是，在Spring MVC中并没有定义一个Handler接口，实际上任何一个Object都可以成为请求处理器。
3. 当DispatcherServlet根据HandlerMapping得到对应当前请求的Handler后，通过HandlerAdapter对Handler进行封装，再以统一的适配器接口调用Handler。HandlerAdapter是Spring MVC框架级接口，顾名思义，HandlerAdapter是一个适配器，它用统一的接口对各种Handler方法进行调用。
4. 处理器完成业务逻 辑的处理后，将返回一个ModelAndView给DispatcherServlet，ModelAndView包含了视图逻辑名和模型数据信息。
5. ModelAndView中包含的是“逻辑视图名”而非真正的视图对象，DispatcherServlet借由ViewResolver完成逻辑视图名到真实视图对象的解析工作。
6. 当得到真实的视图对象View后，DispatcherServlet就使用这个View对象对ModelAndView中的模型数据进行视图渲染。
7. 最终客户端得到的响应消息可能是一个普通的HTML页面，也可能是一个XML或JSON串，甚至是一张图片或一个PDF文档等不同的媒体形式。
## 注解

1. **@RequestBody**

用于读取 Request 请求（可能是 POST,PUT,DELETE,GET 请求）的 body 部分并且Content-Type 为 application/json 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去。系统会使用HttpMessageConverter或者自定义的HttpMessageConverter将请求的 body 中的 json 字符串转换为 java 对象。


2. **@PathVariable**

作用：该注解是用于绑定url中的占位符，但是注意，spring3.0以后，url才开始支持占位符的，它是Spring MVC支持的rest风格url的一个重要的标志。

3. **@RequestParam**

作用：是将请求参数绑定到你的控制器的方法参数上，是Spring MVC中的接收普通参数的注解。

属性：

value是请求参数中的名称。
required是请求参数是否必须提供参数，它的默认是true，意思是表示必须提供。

4. **@RequestMapping**

作用：该注解的作用就是用来处理请求地址映射的，也就是说将其中的处理器方法映射到url路径上。

属性：

* method：是让你指定请求的method的类型，比如常用的有get和post。
* value：是指请求的实际地址，如果是多个地址就用{}来指定就可以啦。
* produces：指定返回的内容类型，当request请求头中的Accept类型中包含指定的类型才可以返回的。
* consumes：指定处理请求的提交内容类型，比如一些json、html、text等的类型。
* headers：指定request中必须包含那些的headed值时，它才会用该方法处理请求的。
* params：指定request中一定要有的参数值，它才会使用该方法处理请求。
## 拦截器

拦截器会对处理器进行拦截，这样通过拦截器就可以增强处理器的功能。Spring MVC中，所有的拦截器都需要实现HandlerInterceptor接口，该接口包含如下三个方法：preHandle()、postHandle()、afterCompletion()。

这些方法的执行流程如下图：

![](https://static.nowcoder.com/images/activity/2021jxy/java/img/springmvc-2.png)

Spring MVC拦截器的开发步骤如下：

1. 开发拦截器：

实现handlerInterceptor接口，从三个方法中选择合适的方法，实现拦截时要执行的具体业务逻辑。

2. 注册拦截器：

定义配置类，并让它实现WebMvcConfigurer接口，在接口的addInterceptors方法中，注册拦截器，并定义该拦截器匹配哪些请求路径。


如果是对Controller记性拦截，则可以使用Spring MVC的拦截器。

如果是对所有的请求（如访问静态资源的请求）进行拦截，则可以使用Filter。

如果是对除了Controller之外的其他Bean的请求进行拦截，则可以使用Spring AOP。
# Springboot

## Springboot 自动装配流程

使用 Spring Boot 时，我们只需引入对应的 Starters，Spring Boot 启动时便会自动加载相关依赖，配置相应的初始化参数，以最快捷、简单的形式对第三方软件进行集成，这便是 Spring Boot 的自动配置功能。Spring Boot 实现该运作机制锁涉及的核心部分如下图所示：

![](https://static.nowcoder.com/images/activity/2021jxy/java/img/springboot-2.jpg)

整个自动装配的过程是：

1. 通过@EnableAutoConfiguration 注解开启自动配置，加载`spring.factories`中注册的各种`AutoConfiguration`类
2. 当某个`AutoConfiguration`类满足其注解@Conditional 指定的生效条件（Starters 提供的依赖、配置或 Spring 容器中是否存在某个 Bean 等）时，实例化该 AutoConfiguration 类中定义的 Bean（组件等），并注入 Spring 容器，就可以完成依赖框架的自动配置。

## SpringBoot 注解

### @SpringBootApplication

在 Spring Boot 入口类中，唯一的一个注解就是@SpringBootApplication。它是 Spring Boot 项目的核心注解，用于开启自动配置，准确说是通过该注解内组合的@EnableAutoConfiguration 开启了自动配置。

### @EnableAutoConfiguration

@EnableAutoConfiguration 的主要功能是启动 Spring 应用程序上下文时进行自动配置，它会尝试猜测并配置项目可能需要的 Bean。自动配置通常是基于项目 classpath 中引入的类和已定义的 Bean 来实现的。在此过程中，被自动配置的组件来自项目自身和项目依赖的 jar 包中。

### @Import

**@EnableAutoConfiguration 的关键功能是通过@Import 注解导入的 ImportSelector 来完成的**。从源代码得知@Import(AutoConfigurationImportSelector.class)是@EnableAutoConfiguration 注解的组成部分，也是自动配置功能的核心实现者。

### Conditional

@Conditional 注解是由 Spring 4.0 版本引入的新特性，可根据是否满足指定的条件来决定是否进行 Bean 的实例化及装配，比如，设定当类路径下包含某个 jar 包的时候才会对注解的类进行实例化操作。总之，就是根据一些特定条件来控制 Bean 实例化的行为。


# MyBatis

## $和#

使用#设置参数时，MyBatis会创建预编译的SQL语句，然后在执行SQL时MyBatis会为预编译SQL中的占位符（?）赋值。预编译的SQL语句执行效率高，并且可以防止注入攻击。

使用$设置参数时，MyBatis只是创建普通的SQL语句，然后在执行SQL语句时MyBatis将参数直接拼入到SQL里。这种方式在效率、安全性上均不如前者，但是可以解决一些特殊情况下的问题。例如，在一些动态表格（根据不同的条件产生不同的动态列）中，我们要传递SQL的列名，根据某些列进行排序，或者传递列名给SQL都是比较常见的场景，这就无法使用预编译的方式了。
## MyBatis缓存

## 