---
title: Spring
date: 2020-01-21 03:54:04
tags: Spring
categories: Spring
comments: false
---

1. Spring 具有以下特点：<!-- more -->

    - 轻量级：Spring 是非侵入性的，即基于 Spring 开发的应用中的对象可以不依赖于 Spring 的 API
    - 依赖注入
    - 面向切面编程
    - 容器：Spring 是一个容器，因为它包含并且管理应用对象的生命周期
    - 框架：Spring 实现了使用简单的组件配置组合成一个复杂的应用，在 Spring 中可以使用 XML 和 Java 注解组合这些对象
    - 一站式：在 IOC 和 AOP 的基础上可以整合各种企业应用的开源框架和优秀的第三方类库（实际上 Spring 自身也提供了展现层的 SpringMVC 和持久层的 Spring JDBC）

2. IOC & DI 概述

    - IOC（ Inversion of Control ）：其思想是反转资源获取的方向。传统的资源查找方式要求组件向容器发起请求查找资源，作为响应，容器适时的返回资源，而应用了 IOC 之后，则是容器主动地将资源推送给它所管理的组件，组件要做的事情仅仅是选择一种合适的方式来接受资源，这种行为也被称为查找的被动形式
    - DI（ Dependency Injection ）：IOC 的另一种表达方式。即组件以一些预定好的方式，如 Setter 方法，接受来自容器的资源注入，相对 IOC 而言，这种表述更直接

3. 配置 Bean

    - 配置形式：

        - 基于 XML 文件的形式

            - 在 XML 文件中通过 bean 节点的来配置 bean

                ```XML
                <!-- 通过全类名的方式来配置 Bean -->
                <bean id="human" class="beans.Human"></bean>
                ```

            - id：Bean 的名称。在 IOC 容器中必须是唯一的。若 id 没有指定，Spring 自动将全限定类名作为 Bean 的名字。id 可以指定多个名字，名字之间可用逗号、分号或空格分隔

            - class：Bean 的全类名，通过反射的方式在 IOC 容器中创建 Bean，所以要求 Bean 中必须要有一个无参构造器

        - 基于注解的方式

    - Bean 的配置方式：

        - 通过全类名（反射）
        - 通过工厂方法（静态工厂方法 & 实例工厂方法）
        - FactoryBean

4. IOC 容器 BeanFactory & ApplicationContext 概述

    - 在 Spring IOC 容器读取 Bean 配置创建 Bean 实例之前，必须对它进行实例化，只有在容器实例化后，才能从 IOC 容器中获取 Bean 实例并使用
    - Spring 提供了两种类型的 IOC 容器实现：
        - BeanFactory：IOC 容器的基本实现
        - ApplicationContext：提供了更多的高级特性，是 BeanFactory 的子接口
        - BeanFactory 是 Spring 框架的基础设施，面向 Spring 本身。ApplicationContext 面向使用 Spring 框架的开发者，几乎所有的应用场合都直接使用 ApplicationContext 而非底层的 BeanFactory
        - 无论使用何种方式，配置文件是相同的
    - ApplicationContext 主要实现类：
        - ClassPathXmlApplicationContext：从类路径下加载配置文件
        - FileSystemXmlApplicationContext：从文件系统中加载配置文件
    - ConfigurableApplicationContext 扩展于 ApplicationContext，新增加两个主要方法：refresh() 和 close()，让 ApplicationContext 具有启动、刷新和关闭上下文的能力
    - ApplicationContext 在初始化上下文时就实例化所有单例的 Bean
    - WebApplicationContext 是专门为 Web 应用准备的，它允许从相对于 Web 根目录的路径中完成初始化工作

5. 依赖注入的方式：

    - Set 注入
    - Constructor 注入

    - 注入属性值细节
    - 自动装配

6. Bean 之间关系：

    - 继承
    - 依赖

7. Bean 的作用域：

    - singleton
    - prototype
    - Web 环境作用域

8. 使用外部属性文件

9. SpEL

10. IOC 容器中 Bean 的生命周期

11. Spring 4.x 新特性：泛型依赖注入