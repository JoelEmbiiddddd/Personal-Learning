## Spring的基础



Spring 是一款开源的轻量级 Java 开发框架，旨在提高开发人员的开发效率以及系统的可维护性。

Spring 最核心的思想就是不重新造轮子，开箱即用，提高开发效率。



### Spring的模块都有哪些？

1. spring-core：Spring 框架的核心模块，也可以说是基础模块，Spring 最核心的思想就是不重新造轮子，开箱即用，提高开发效率。
2. AOP：提供AOP的编程实现
3. Data Access/Integration：提供了ORM的实现，对事物的支持，和对数据库访问的抽象JDBC
4. Spring Web：对Web功能、SpringMVC、Websocket、WebFlux提供了支持。
5. Test：进行测试。



### Spring、Spring MVC、Spring Boot之间的关系

spring包含了很多组件，其中最重要的就是spring-core。使用 Spring 进行开发各种配置过于麻烦比如开启某些 Spring 特性时，需要用 XML 或 Java 进行显式配置，于是，Spring Boot 诞生了。

Spring 旨在简化 J2EE 企业应用程序开发。Spring Boot 旨在简化 Spring 开发（减少配置文件，开箱即用！）



Spring Boot 只是简化了配置，如果你需要构建 MVC 架构的 Web 程序，你还是需要使用 Spring MVC 作为 MVC 框架，只是说 Spring Boot 帮你简化了 Spring MVC 的很多配置，真正做到开箱即用！





## Spring IOC

IOC是一种设计思想，而不是具体的技术体现。IoC 的思想就是将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理。



### 为什么叫控制反转

1. 控制：对象创建、管理、实例化的权利
2. 反转：控制权交给外部环境（Spring框架、IOC容器）



### 为什么我们需要控制反转，可以举一个例子吗？

在一个项目中，有一个sevices类依赖了很多其他的类，如果我们想实现这个类，就要把它依赖的所有类的构造函数都了解。如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。



### Spring Bean是什么？

Bean 代指的就是那些被 IoC 容器所管理的对象。



### 将一个类声明为Bean的注解都有哪些？

1. `@Component`：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。

2. `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。

3. `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。

4. `@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 `Service` 层返回数据给前端页面。

   

### @Component和@Bean的区别是什么

1. `@Component` 注解作用于类，而`@Bean`注解作用于方法。
2. `@Component`通常是通过类路径扫描来自动侦测以及自动装配到 Spring 容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了 Spring 这是某个类的实例，当我需要用它的时候还给我。
3. 很多第三方库中的类需要装配到`Spring`容器时，则只能通过@Bean来实现



### 注入Bean的注解有哪些？

1. AutoWired
2. Resource
3. Inject



### @AutoWired和@Resource的区别是什么？

- `@Autowired` 是 Spring 提供的注解，`@Resource` 是 JDK 提供的注解。

- `Autowired` 默认的注入方式为`byType`（根据类型进行匹配），`@Resource`默认注入方式为 `byName`（根据名称进行匹配）。

- 当一个接口存在多个实现类的情况下，`@Autowired` 和`@Resource`都需要通过名称才能正确匹配到对应的 Bean。`Autowired` 可以通过 `@Qualifier` 注解来显式指定名称，`@Resource`可以通过 `name` 属性来显式指定名称。



### 单例 Bean 的线程安全问题了解吗？

单例 Bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候是存在资源竞争的。

1. 在 Bean 中尽量避免定义可变的成员变量。
2. 在类中定义一个 `ThreadLocal` 成员变量，将需要的可变成员变量保存在 `ThreadLocal` 中（推荐的一种方式）。

不过，大部分 Bean 实际都是无状态（没有实例变量）的（比如 Dao、Service），这种情况下， Bean 是线程安全的。





### Bean的生命周期

Bean的生命周期主要包括：Bean定义，Bean初始化，Bean生存期，Bean销毁。 具体流程如下：

1. spring启动，查找并加载所需的Bean，然后初始化。
2. 进行Bean的属性依赖注入。
3. 如果Bean实现了BeanNameAware接口（@Resource，@Qualifier），spring会将Bean的Id传入SetBeanName方法去执行。
4. 如果Bean实现了BeanFactoryAware接口，spring会调用Bean的setBeanFactory方法将BeanFactory的ioc容器传入。
5. 如果Bean实现的是ApplicationContextAware接口的话，Spring会调用Bean的setApplicationContext将Bean应用的上下文传入进来。
6. 还一些其他的设定例如使用@PostContruct来定义Bean在初始化时执行的方法，或者使用@PreDestory来定义Ioc容器被销毁时执行的方法等。



## String AOP

Spring AOP 就是基于动态代理的，AOP能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。



### Spring AOP 和 AspectJ AOP有什么区别？

1. Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。
2. Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。
3. Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单
4. 当切面太多的话，最好选择 AspectJ 。



### 多个切面的顺序如何确定

1. 通常使用`@Order` 注解直接定义切面顺序（值越小优先级越高）。

2. **实现`Ordered` 接口重写 `getOrder` 方法。**



## Spring事务



### Spring支持两种方式的事务

1. 通过TransactionTemplate或TransactionManager手动管理事务。但用得不多。（编程式声明）
2. 通过AOP实现，即基于@Transactional来保证。（声明式事务管理）



### @Transactional注释的使用说明

它的作用范围：

1. 方法：该注解只能用到`public`方法上，否则就不会生效。
2. 类：如果这个注解使用在类上的话，就表明该注解对该类中所有public方法都生效。
3. 接口：不推荐使用在接口上。



### @Transactional 事务注解原理

我们知道，**`@Transactional` 的工作机制是基于 AOP 实现的，AOP 又是使用动态代理实现的。如果目标对象实现了接口，默认情况下会采用 JDK 的动态代理，如果目标对象没有实现了接口,会使用 CGLIB 动态代理。**

1. 如果一个类或者一个类中的 public 方法上被标注`@Transactional` 注解的话，Spring 容器就会在启动的时候为其创建一个代理类，
2. 在调用被`@Transactional` 注解的 public 方法的时候，实际调用的是，`TransactionInterceptor` 类中的 `invoke()`方法。
3. 这个方法的作用就是在目标方法之前开启事务，方法执行过程中如果遇到异常的时候回滚事务，方法调用完成之后提交事务。



### Spring AOP自调用方法（会使得事务失效）

**若同一类中的其他没有 `@Transactional` 注解的方法内部调用有 `@Transactional` 注解的方法，有`@Transactional` 注解的方法的事务会失效。**

这是由于`Spring AOP`代理的原因造成的，因为只有当 `@Transactional` 注解的方法在类以外被调用的时候，Spring 事务管理才生效。

所以我们要避免同一类中自调用或者使用 AspectJ 取代 Spring AOP 代理



### 使用@Transactional的注意事项：

1. `@Transactional` 注解只有作用到 public 方法上事务才生效，不推荐在接口上使用；

2. 避免同一个类中调用 `@Transactional` 注解的方法，这样会导致事务失效；

3. 正确的设置 `@Transactional` 的 `rollbackFor` 和 `propagation` 属性，否则事务可能会回滚失败;

4. 被 `@Transactional` 注解的方法所在的类必须被 Spring 管理，否则不生效；

5. 底层使用的数据库必须支持事务机制，否则不生效；



## Spring中的设计模式



**工厂设计模式** : Spring 使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。

**代理设计模式** : Spring AOP 功能的实现。

**单例设计模式** : Spring 中的 Bean 默认都是单例的。

**模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。

**包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。

**观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。

**适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`



### 工厂模式

Spring 使用工厂模式可以通过 `BeanFactory` 或 `ApplicationContext` 创建 bean 对象。

二者的区别：

1. `beanFactory:` 延迟注入，即使用到的时候才会注入。 相较于ApplicationContext，占用更少的内存。
2. `ApplicationContext:` 不管用没用，都一次性注入。



### 单例设计模式

**为什么使用单例设计模式**

很多时候只需要一个，比如线程池，缓存等等。如果制造出多个实例就可能会产生一些问题，比如程序的行为异常，资源使用过量。



**使用单例模式的好处** :

- 对于频繁使用的对象，可以省略创建对象所花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销；
- 由于 new 操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻 GC 压力，缩短 GC 停顿时间。

Spring 通过 `ConcurrentHashMap` 实现单例注册表的特殊方式实现单例模式。



### 代理模式

代理模式主要是在AOP中使用。

Spring AOP就是基于动态代理的。



### 模板模式

它定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。

Spring 中 `JdbcTemplate`、`HibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。一般情况下，我们都是使用继承的方式来实现模板模式，但是 Spring 并没有使用这种方式，而是使用 Callback 模式与模板方法模式配合，既达到了代码复用的效果，同时增加了灵活性。



### 观察者模式

当一个对象发生改变的时候，依赖这个对象的所有对象也会做出反应。

Spring 事件驱动模型就是观察者模式很经典的一个应用。Spring 事件驱动模型非常有用，在很多场景都可以解耦我们的代码。比如我们每次添加商品的时候都需要重新更新商品索引，这个时候就可以利用观察者模式来解决这个问题。



**Spring事件驱动模型中的三种角色**： 

1. 事件角色： ApplicationEvent
2. 事件监听者角色：ApplicationListener
3. 事件发布者角色：ApplicationEventPublisher



 **Spring 的事件流程总结**

1. 定义一个事件: 实现一个继承自 `ApplicationEvent`，并且写相应的构造函数；
2. 定义一个事件监听者：实现 `ApplicationListener` 接口，重写 `onApplicationEvent()` 方法；
3. 使用事件发布者发布消息: 可以通过 `ApplicationEventPublisher` 的 `publishEvent()` 方法发布消息。



### 适配器模式

适配器模式(Adapter Pattern) 将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作。



**Spring AOP中的适配器模式**

Spring AOP中的增强或通知中使用到了适配器模式，与之相关的接口是AdvisorAdapter。

在通知中Advice中，常见的类型有：

1. beforeadvice
2. afteradvice
3. afterreturnAdvice

每个类型 Advice（通知）都有对应的拦截器:`MethodBeforeAdviceInterceptor`、`AfterReturningAdviceInterceptor`、`ThrowsAdviceInterceptor` 等等。



### 装饰器模式

装饰者模式可以动态地给对象添加一些额外的属性或行为。在JDK中，`InputStream`家族，`InputStream` 类下有 `FileInputStream` (读取文件)、`BufferedInputStream` (增加缓存,使读取文件速度大大提升)等子类都在不修改`InputStream` 代码的情况下扩展了它的功能。



## SpringBoot 自动装配原理



使用spring的时候，我们要开启某些spring的特性或者引入第三方依赖的时候，还需要用XML或Java进行显式配置。

但是，Spring Boot 项目，我们只需要添加相关依赖，无需配置，通过启动下面的 `main` 方法即可

这就是自动装配的原因。



### 什么是自动装配

在没有spring boot的情况下，如果我们需要引入第三方依赖，就需要手动配置。而springboot，我们只需要引入starter，然后通过少量注解和简单的配置就可以使用第三方提供的功能。



也就是自动装配可以简单理解为：**通过注解或者一些简单的配置就能在 Spring Boot 的帮助下实现某块功能**。





### SpringBoot如何实现自动装配



**SpringBoot的核心注解：SpringBootApplication**

`@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。根据 SpringBoot 官网，这三个注解的作用分别是：

1. `@EnableAutoConfiguration: ` 启动SpringBoot的自动装配机制
2. `@Configuration：`允许在上下文中注册额外的bean或导入其他配置类
3. `@ComponetScan：` 扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean。



**@EnableAutoConfiguration：实现自动装配的核心注解**

自动装配核心功能的实现实际是通过 `AutoConfigurationImportSelector`类。那么这个类干了一件什么事情呢？这个类叫加载自动装配类

`AutoConfigurationImportSelector` 类实现了 `ImportSelector`接口，也就实现了这个接口中的 `selectImports`方法，该方法主要用于**获取所有符合条件的类的全限定类名，这些类需要被加载到 IoC 容器中**。具体流程如下：

1. 判断自动装配开关是否打开，默认是开启的。
2. 获取@EnableAutoConfiguration注解中的exclude(指定要排除的自动配置类)和exludeName（指定要排除的自动配置类的 全限定类名 ）
3. 获取需要自动装配的所有配置类，读取`META-INF/spring.factories` 。注意，此处的spring.factories不是每次启动都全部加载的，而是满足要求才会加载，有一些条件注解。



[6db825b79af04cf1b56b4e677fdb6e42.png (2245×1587) (csdnimg.cn)](https://img-blog.csdnimg.cn/6db825b79af04cf1b56b4e677fdb6e42.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6J-R6J6C5oG26Zy45LmLamF2YeaBtumcuA==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

**总结：**

![img](https://img-blog.csdnimg.cn/6db825b79af04cf1b56b4e677fdb6e42.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6J-R6J6C5oG26Zy45LmLamF2YeaBtumcuA==,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

