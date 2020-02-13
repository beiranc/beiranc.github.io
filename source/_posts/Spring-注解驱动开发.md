---
title: Spring 注解驱动开发
date: 2020-02-13 11:16:08
tags: Spring
categories: Spring
comments: false
---

##### 容器<!-- more -->

1. ###### AnnotationConfigApplicationContext

    - 配置类
        - 使用 @Configuration 注解修饰的类，作用等同于 Spring 的 Bean 配置文件
    - 包扫描
        - 使用 @ComponentScan 注解修饰配置类

2. ###### 组件添加

    - @ComponentScan
        - 作用：扫描给定包中使用 @Controller、@Repository、@Service、@Component 等注解修饰的类，并将其注入 IOC 容器中（适用于自己编写的类）
        - value 属性和 basepackages 属性作用相同，都是指定要扫描的包
        - excludeFilters 属性指定一个 Filter 数组，这个 Filter 数组中指定的为按照什么规则排除哪些组件
        - includeFilters 属性指定一个 Filter 数组，这个 Filter 数组中指定的为只需要包含什么组件
        - useDefaultFilters 属性可以禁用默认的 Filter 来使 includeFilters 生效
        - 过滤类型
            - FilterType.ANNOTATION（默认）：按照注解
            - FilterType.ASSIGNABLE_TYPE：按照给定类型
            - FilterType.ASPECTJ：按照 AspectJ 表达式
            - FilterType.REGEX：按照正则表达式
            - FilterType.CUSTOM：按照自定义规则
                - 自定义规则需要使用 TypeFilter 接口并重写其中的 boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) 方法。match 方法的 metadataReader 参数为读取到的当前正在扫描的类的信息，metadataReaderFactory 参数为一个可以获取其他类信息的工厂
    - @Bean
        - 作用：向 IOC 容器中注册一个 Bean，类型为返回值的类型，id 默认使用方法名作为 id。若需要修改 id，可以修改方法名或者使用 @Bean 注解中的 value 属性（可用于导入第三方包内的类）
        - 可以自定义初始化和销毁方法
        - 初始化其他方式
            - InitializingBean（初始化设置值之后）
            - DisposableBean（销毁）
            - JSR250
                - @PostConstruct
                - @PreDestroy
        - BeanPostProcessor
    - @Configuration
        - 作用：指定配置类
    - @Component
        - 作用：声明一个基本组件
        - @Service、@Controller、@Repository 实际上与 @Component 注解没有区别，只是为了区分才将它们分离出来
    - @Service
        - 作用：声明一个 Service 层组件
    - @Controller
        - 作用：声明一个 Controller 层组件
    - @Repository
        - 作用：声明一个 DAO 层组件
    - @Conditional
        - 作用：按照一定的条件进行判断，满足条件才将 Bean 注入到 IOC 容器中
        - 自定义的规则需要实现 org.springframework.context.annotation.Condition 接口。其中 match() 方法中的 ConditionContext 参数为判断条件能使用的上下文（环境），AnnotatedTypeMetadata 参数为注解信息
        - 若 @Conditional 注解修饰类，则只有满足条件时这个类中配置的所有 Bean 才会注入到 IOC 容器中
    - @Primary
    - @Lazy
        - 懒加载：不在 IOC 容器启动时创建对象，而是在第一次从 IOC 容器中获取 Bean 的时候才去创建对象并初始化
        - singleton 作用域的组件默认是在 IOC 容器启动时就创建对象，为了节省资源，可以使用懒加载的方式创建 singleton 作用域的 Bean
    - @Scope
        - 作用：设置组件的作用域
        - 取值
            - prototype：多实例。只有在从 IOC 容器中获取对象时才创建
            - singleton（默认）：单实例。IOC 容器一启动就将对象注入至 IOC 容器中
            - request：同一次请求创建一个实例
            - session：同一个 session 创建一个实例
    - @Import
        - 作用：快速地向 IOC 容器中导入组件，id 默认为组件的全类名
        - 也可以通过实现 ImportSelector 或 ImportBeanDefinitionRegistrar 接口来导入组件
        - ImportBeanDefinitionRegistrar 接口中的 registerBeanDefinitions 方法中的 AnnotationMetadata 参数为当前类的注解信息，BeanDefinitionRegistry 则为 Bean 定义的注册类，可用于自定义注册 Bean
    - ImportSelector
        - 实现 ImportSelector 接口可以快速的导入多个组件，其中的 selectImports 方法的返回值为要导入到 IOC 容器中的组件的全类名，AnnotationMetadata 参数为当前标注 @Import 注解的类的所有注解信息
    - 工厂模式
        - FactoryBean
            - 实现 FactoryBean 接口可以自定义的创建一个 Bean，其中 FactoryBean 向 IOC 容器中注入的默认为 getObject() 方法返回的实例
            - 使用 &beanName 获取 Factory 本身

3. ###### 组件赋值

    - @Value
    - @Autowired
        - @Qualifier
        - 其他方式
            - @Resources（ JSR250 ）
            - @Inject（ JSR330，需要导入 javax.inject ）
    - @PropertySource
    - @PropertySources
    - @Profile
        - Environment
        - -Dspring.profiles.active=test

4. ###### 组件注入

    - 方法参数
    - 构造器注入
    - ApplicationContextAware
        - ApplicationContextAwareProcessor
    - xxxAware

5. ###### AOP

    - @EnableAspectJAutoProxy
    - @Before/@After/@AfterReturning/@AfterThrowing/@Around
    - @Pointcut

6. ###### 声明式事务

    - @EnableTransactionManagement
    - @Transactional

----

##### 扩展原理

1. ###### BeanFactoryPostProcessor

    - Spring 容器标准初始化之后执行（ BeanPostProcessor 之前），此时 Bean 还未创建
    - Spring 容器初始化两大步
        - 加载保存和读取所有 Bean 配置
        - 按照之前的配置创建 Bean

2. ###### BeanDefinitionRegistryPostProcessor

    - BeanFactoryPostProcessor 子类，可自定义添加 Bean 定义
    - BeanDefinetionRegistry
        - BeanDefinetionBuilder

3. ###### ApplicationListner

    - @EventListener

4. ###### Spring 容器创建过程

    

----

##### Web

1. ###### Servlet 3.0

    - ServletContainerInitializer
    - Registration
        - ServletRegistration
        - FilterRegistration

2. ###### 异步请求

    - Servlet 3.0 异步处理
        - 在Servlet 3.0之前，Servlet采用Thread-Per-Request的方式处理请求。
            即每一次Http请求都由某一个线程从头到尾负责处理。如果一个请求需要进行IO操作，比如访问数据库、调用第三方服务接口等，那么其所对应的线程将同步地等待IO操作完成， 而IO操作是非常慢的，所以此时的线程并不能及时地释放回线程池以供后续使用，在并发量越来越大的情况下，这将带来严重的性能问题。即便是像Spring、Struts这样的高层框架也脱离不了这样的桎梏，因为他们都是建立在Servlet之上的。为了解决这样的问题，Servlet 3.0引入了异步处理，然后在Servlet 3.1中又引入了非阻塞IO来进一步增强异步处理的性能。
    - 返回 Callable
    - 返回 DeferredResult