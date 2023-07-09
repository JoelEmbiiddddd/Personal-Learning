## SpringBoot相关问题



### SpringBoot和Spring的区别是什么

1. SpringBoot是基于Spring开发的，Spirng Boot 本身并不提供 Spring 框架的核心特性以及扩展功能，只是用于快速、敏捷地开发新一代基于 Spring 框架的应用程序。
2. 它并不是用来替代 Spring 的解决方案，而是和 Spring 框架紧密结合用于提升 Spring 开发者体验的工具。

SpringBoot是以 **约定大于配置**的核心思想展开工作。它和spring的区别在于

1. SpringBoot内嵌了Tomcat，jetty这些容器，可以直接跑起来而不用做部署工作了。
2. SpringBoot不需要像Spring一样配置繁琐的XML文件。Spring Boot 可以自动配置(核心)Spring。SpringBoot 将原有的 XML 配置改为 Java 配置，将 bean 注入改为使用注解注入的方式(@Autowire)，并将多个 xml、properties 配置浓缩在一个 appliaction.yml 配置文件中。



### springboot依赖注入冲突怎么解决

依赖冲突：依赖冲突是指项目依赖的某一个jar包，有多个不同的版本，因而造成类包版本冲突。

**使用 dependencyManagement 锁定依赖库的版本号**

总的父项目去写所有的jar包括版面，然后通过modules标签将 子模块导入，子模块只需要导入jar名就行，而不用导入版本号，这样整个项目的jar都集中在父项目的pom文件去管理，子模块也不用担心依赖冲突，因为需要什么导什么但不管版本。



### SpringBoot常用注解

1. `@springbootApplication` *这个注解是 Spring Boot 项目的基石，创建 SpringBoot 项目之后会默认在主类加上。*
2. `Spring Bean`相关的： `@autowire`，`@Component`，`@Repository`，`@Service`，`@Configuration`申明配置类。
3. 读取配置信息的 ： `@ConfigurationProperties`（通过`@ConfigurationProperties`读取配置信息并与 bean 绑定）、`@Value`（读取比较简单的配置信息）。
4. 事务：`@Transactional`在要开启事务的方法上使用`@Transactional`注解即可



### 为什么用SpringBoot

1. 核心设计思想是∶约定优于配置，Spring Boot所有开发细节都是依据此思想进行实现的。在我们实际在做项目的过程中，减少软件开发人员需做决定的数量、获得简单的好处，而又不失灵活性。
   - Spring Boot能根据当前类路径下的类、jar包来自动配置bean，如添加一个spring-boot-starter-web启动器就能拥有web的功能，无需其他配置。比如说我们封装网关算力的Starters，只需要简单配置就可以立即使用，无需再去配置繁重的XML文件。
2. Spring Boot而且内嵌了各种[servlet](https://so.csdn.net/so/search?q=servlet&spm=1001.2101.3001.7020)容器，Tomcat、Jetty等，现在不再需要打成war包部署到容器中，Spring Boot只要打成一个可执行的jar包就能独立运行，所有的依赖包都在一个jar包内。
3. 应用监控：Spring Boot提供一系列端点可以监控服务及应用，做健康检测。





### SpringBoot和SpringMVC的区别

二者的目的是不相同的。

1. Spring Boot：Spring Boot使快速引导和开始开发基于Spring的应用程序变得容易。 它避免了很多样板代码。 它在幕后隐藏了很多复杂性，因此开发人员可以快速上手并轻松开发基于Spring的应用程序。
2. Spring MVC：Spring MVC是用于构建Web应用程序的Web MVC框架。 它包含许多用于各种功能的配置文件。 这是一个面向HTTP的Web应用程序开发框架。 它是Spring的一个模块，是一个web框架





## Springboot 自动装配

自动装配就是**通过注解或者一些简单的配置就能在 Spring Boot 的帮助下实现某块功能**。

在没有spring boot的情况下，如果我们需要引入第三方依赖，就需要手动配置。而Spring Boot 项目，我们只需要添加相关依赖，无需配置，通过启动下面的 `main` 方法即可，然后通过少量注解和简单的配置就可以使用第三方提供的功能。



### 原理

**SpringBoot的核心注解：SpringBootApplication**

`@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。根据 SpringBoot 官网，这三个注解的作用分别是：

1. `@EnableAutoConfiguration: ` 启动SpringBoot的自动装配机制
2. `@SpringConfiguration：`允许在上下文中注册额外的bean或导入其他配置类
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







### 自动装配和自动配置的区别

1. 自动配置是一种机制，可以根据类路径来自动配置spring应用程序的功能。自动配置是Spring Boot提供的特殊功能，其目的是简化Spring应用程序的配置并提高开发效率。
2. 自动装配是Spring框架的一项核心功能，它允许Spring容器自动将相互依赖的bean对象注入到其他bean对象中，以实现对象之间的协作。通过自动装配，您可以避免手动编写大量的配置代码来管理bean之间的依赖关系。





## Spring事件监听机制

`Spring` 的事件驱动模型基于 `ApplicationEvent` 和 `ApplicationListener` ，通过事件驱动的方式来实现业务模块之间的交互，交互的方式也有同步和异步两种。它是一种观察者模式的监听方式。

`观察者模式`：观察者和主题两个不同的角色，多个观察者同时观察一个主题，两者只通过抽象接口保持松耦合状态，这样双方可以相对独立的进行扩展和变化



### 事件

Spring为容器内事件定义了一个抽象类**ApplicationEvent**，该类继承了JDK中的事件基类**EventObject**。只需要继承 **ApplicationEvent**就可以发布自己的事件了。



### 事件监听器

Spring定义了一个**ApplicationListener**接口作为事件监听器的抽象，该接口继承了JDK中表示事件监听器的标记接口EventListener，内部只定义了一个抽象方法onApplicationEvent(evnt)，当监听的事件在容器中被发布，该方法将被调用。



基于Spring框架来实现对自定义事件的监听流程十分简单，只需要三步：

- ① 自定义事件类
- ② 自定义事件监听器并向容器注册（通过@Component注解）
- ③ 发布事件。通过实现ApplicationContextAware接口获取ApplicationContext实例，然后调用其发布事件方法



监听容器中发布的事件

步骤：

1. 写一个监听器监听某个事件（ApplicationEvent及其子类）

2. 把监听器加入到容器；

3. 只要容器中有相关事件的发布，我们就能监听到这个事件

   - ContextRefreshedEvent:容器刷新完成（所有bean都完全创建）会发布这个事件；
   - ContextClosedEvent：关闭容器会发布这个事件



### 事件发布器

ApplicationContext接口继承了ApplicationEventPublisher接口，从而提供了对外发布事件的能力



### ApplicationContext的作用是什么

ApplicationContext相当于Spring的一个与IOC容器连接的桥梁，通过getBean();方法，我们可以轻松的从IOC容器中获取Bean对象。



### ApplicationContextAware

在某些类中我们经常需要通过ApplicationContext来获取需要的bean,但每一次使用new ClassPathXmlApplicationContext()都会重新装配文件并实例化上下文bean，这样肯定是很麻烦的。

此时ApplicationContextAware接口的作用就体现出来了——spring会给实现此接口的类注入ApplicationContext对象。实现了这个接口的bean，当spring容器初始化的时候，会自动的将ApplicationContext注入进来。



Spring容器会检测容器中的所有Bean，如果发现某个Bean实现了ApplicationContextAware接口，Spring容器会在创建该Bean之后，自动调用该Bean的setApplicationContextAware()方法，调用该方法时。







## SpringBoot Starter原理

**Starter的原理就是自SpringBoot自动装配的原理**



自定义一个Starter的流程

1. 创建一个项目，命名是Starter，引入Springboot相关依赖

2. 使用`@ConfigurationProperties`，并定义属性配置的前缀

   （定义属性配置的前缀是为了避免属性名称的冲突和组织属性的层次结构。通过为属性配置添加前缀，可以在属性名上添加一个命名空间，使其在全局配置文件中具有唯一性，避免不同starter之间的属性冲突。）

3. 创建自动配置类吗，使用`@Configuration`和`EnableConfigurationProperties(限定名.class)`

4. 在`Spring.factories`文件中添加自动配置类路径



## Springboot 启动流程

这个问题也可以是

`Spring Boot 的SpringApplication.run(xxx.class,args)  是怎么运行的`



1. 加载并且启动监听器
2. 创建项目运行环境，加载配置
3. 初始化 Spring 容器，准备环境变量，比如说系统变量，环境变量，命令行参数，默认变量。
4. 执行 Spring 容器前置处理器，根据不同类型环境创建不同类型的applicationcontext容器
5. 刷新 Spring 容器，比如调用bean factory的后置处理器，注册BeanPostProcessor后置处理器。
6. 执行 Spring 后置处理器
7. 发布事件
8. 执行自定义执行器：调用ApplicationRunner和CommandLineRunner的run方法，我们实现这两个接口可以在spring容器启动后需要的一些东西比如加载一些业务数据等;
9. 返回容器，如果没有什么异常的话。