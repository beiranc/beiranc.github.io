---
title: Spring
date: 2020-01-21 03:54:04
tags: Spring
categories: Spring
comments: false
---

##### Spring 特点<!-- more -->

- 轻量级：Spring 是非侵入性的，即基于 Spring 开发的应用中的对象可以不依赖于 Spring 的 API
- 依赖注入
- 面向切面编程
- 容器：Spring 是一个容器，因为它包含并且管理应用对象的生命周期
- 框架：Spring 实现了使用简单的组件配置组合成一个复杂的应用，在 Spring 中可以使用 XML 和 Java 注解组合这些对象
- 一站式：在 IOC 和 AOP 的基础上可以整合各种企业应用的开源框架和优秀的第三方类库（实际上 Spring 自身也提供了展现层的 SpringMVC 和持久层的 Spring JDBC）

----

##### IOC & DI 概述

- IOC（ Inversion of Control ）：其思想是反转资源获取的方向。传统的资源查找方式要求组件向容器发起请求查找资源，作为响应，容器适时的返回资源，而应用了 IOC 之后，则是容器主动地将资源推送给它所管理的组件，组件要做的事情仅仅是选择一种合适的方式来接受资源，这种行为也被称为查找的被动形式
- DI（ Dependency Injection ）：IOC 的另一种表达方式。即组件以一些预定好的方式，如 Setter 方法，接受来自容器的资源注入，相对 IOC 而言，这种表述更直接

----

##### 配置 Bean

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

        - 组件扫描：Spring 能够从 classpath 下自动扫描，侦测和实例化具有特定注解的组件，特定组件包括：

            - @Component：基本注解，标识一个受 Spring 管理的组件
            - @Repository：标识持久层组件
            - @Service：标识业务层组件
            - @Controller：标识控制层组件

        - 对于扫描到的组件，Spring 有默认的命名策略：使用非限定类名，第一个字母小写。也可以在注解中通过 value 属性标识组件的名称

        - 在组件类上使用了特定注解后，还需要在 Spring 的配置文件中声明 \<context:component-scan\>

            - base-package 属性指定一个需要扫描的基类包，Spring 容器将会扫描这个基类包中及其子包中的所有类

            - 当需要扫描多个包时，可以使用逗号分隔

            - 如果仅希望扫描特定的类而非基类包下所有类，可使用 resource-pattern 属性过滤特定的类，如：

                ```XML
                <context:component-scan base-package="beans" resource-pattern="autowire/*.class" />
                ```

            - \<context:include-filter\>：子节点表示要包含的目标类（与 \<context:component-scan\> 的 use-default-filters 属性配合使用）

            - \<context:exclude-filter\>：子节点表示要排除在外的目标类

            - \<context:component-scan\> 下可拥有若干个 \<context:include-filter\> 与 \<context:exclude-filter\> 子节点

            - \<context:include-filter\> 和 \<context:exclude-filter\> 支持多种类型的过滤表达式：

                |    类别    |          示例           |                             说明                             |
                | :--------: | :---------------------: | :----------------------------------------------------------: |
                | annotation |    com.XxxAnnotation    | 所有标注了 XxxAnnotation 的类，该类型采用目标类是否标注了某个注解进行过滤 |
                | assignable |     com.XxxService      | 所有继承或扩展 XxxService 的类，该类型采用目标类是否继承或扩展某个特定类进行过滤 |
                |  aspectj   | com.aaa.bbb..*Service+  | 所有类名以 Service 结束的类及继承或扩展它们的类，该类型采用 AspectJ 表达式进行过滤 |
                |   regex    | com.\\aaa\\\.anno\\\.\* | 所有 com.aaa.anno 包下的类，该类型采用正则表达式根据类的类名进行过滤 |
                |   custom   |    com.XxxTypeFilter    | 采用 XxxTypeFilter 通过代码的方式定义过滤规则，该类必须实现 org.springframework.core.type.TypeFilter 接口 |
            
        - 组件装配：\<context:component-scan\> 元素还会自动注册 AutowiredAnnotationBeanPostProcessor 实例，该实例可以自动装配具有 @Autowired 和 @Resource 以及 @Inject 注解的属性
        
            - @Autowired 注解自动装配具有兼容类型的单个 Bean 属性
                - 构造器，普通字段（即使是非 public），一切具有参数的方法都可以应用 @Autowired 注解
                - 默认情况下，所有使用 @Autowired 注解的属性都需要被设置，当 Spring 找不到匹配的 Bean 装配属性时，会抛出异常，若某一属性允许不被设置，可以设置 @Autowired 注解的 required 属性为 false
                - 默认情况下，当 IOC 容器里存在多个类型兼容的 Bean 时，通过类型的自动装配将无法工作，此时可以在 @Qualifier 注解里提供 Bean 的名称，Spring 允许对方法的入参标注 @Qualifier 以指定注入 Bean 的名称
                - @Autowired 注解也可以应用在数组类型的属性上，此时 Spring 将会把所有匹配的 Bean 进行自动装配
                - @Autowired 注解也可以应用在集合属性上，此时 Spring 读取该集合的类型信息，然后自动装配所有与之兼容的 Bean
                - @Autowired 注解用在 java.util.Map 上时，若该 Map 的键值为 String，那么 Spring 将自动装配与之 Map 值类型兼容的 Bean，此时 Bean 的名称作为键值
            - Spring 还支持 @Resource 和 @Inject 注解，这两个注解和 @Autowired 注解的功能类似
                - @Resource 注解要求提供一个 Bean 名称的属性，若该属性为空，则自动采用标注处的变量或方法名作为 Bean 的名称
                - @Inject 注解 和 @Autowired 注解一样也是按类型匹配注入 Bean，但是没有 required 属性（建议使用 @Autowired）

- Bean 的配置方式：

    - 通过全类名（反射）
    - 通过工厂方法（静态工厂方法 & 实例工厂方法）
        - 调用静态工厂方法创建 Bean 是将对象创建的过程封装到静态方法中，当客户端需要对象时，只需要简单的调用静态方法，而不需要关心创建对象的细节
        - 要声明通过静态方法创建的 Bean，需要在 Bean 的 class 属性里指定拥有该工厂的方法的类，同时在 factory-method 属性里指定工厂方法的名称，最后，使用 \<constrctor-arg\> 元素为该方法传递方法参数
        - 实例工厂方法：将对象的创建过程封装到另外一个对象实例的方法里。当客户端需要请求对象时，只需要简单的调用该实例方法而不需要关系对象的创建细节
        - 要声明通过实例工厂方法创建的 Bean，需要在 Bean 的 factory-bean 属性里指定拥有该工厂方法的 Bean，然后在 factory-method 属性里指定该工厂方法的名称，最后使用 \<constructor-arg\> 元素为工厂方法传递方法参数
    - FactoryBean
        - 自定义的 FactoryBean 需要实现 org.springframework.beans.factory.FactoryBean\<T\> 接口，然后在配置文件中定义一个 Bean，class 为 自定义的 FactoryBean 的全类名，这个 Bean 返回的实例为自定义 FactoryBean 中实现的 getObject() 方法的返回值

----

##### IOC 容器 BeanFactory & ApplicationContext 概述

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

----

##### 依赖注入的方式

- Setter 注入

    - 通过 setter 方法注入 Bean 的属性值或依赖的对象

    - 属性注入使用 \<property\> 元素，使用 name 属性指定 Bean 的属性名称，value 属性或 \<value\> 子节点指定属性值

    - 属性注入是实际应用中最常用的注入方式

        ```XML
        <!-- 通过全类名的方式来配置 Bean -->
        <bean id="human" class="beans.Human">
            <property name="name" value="Spring First"></property>
        </bean>
        ```

- Constructor 注入

    - 通过构造方法注入 Bean 的属性值或者依赖的对象，它保证了 Bean 实例在实例化之后即可使用
    - 构造器注入在 \<constructor-arg\> 元素里声明属性，\<constructor-arg\> 中没有 name 属性
    - 使用构造器注入属性值可以指定参数的位置（ index ）和参数的类型 （ type ），以区分重载的构造器

- 注入属性值细节

    - 字面值：可用字符串表示的值，可以通过 \<value\> 元素标签或者 value 属性进行注入。基本数据类型及其封装类、String 等类型都可以采取字面值注入的方式。若字面值中包含特殊字符，可以使用 \<![CDATA[]]\> 把字面值包裹起来
    - 在 Bean 的配置文件中，可以通过 \<ref\> 元素或者 ref 属性为 Bean 的属性或者构造器参数指定对 Bean 的引用，也可以在属性或构造器里包含 Bean 的声明，这样的 Bean 称为内部 Bean
    - 可以使用专用的 \<null/\> 元素标签为 Bean 的字符串或其他对象类型的属性注入 null 值。Spring 支持级联属性的配置（需要先初始化属性再为级联属性赋值）
    - 在 Spring 中可以通过一组内置的 xml 标签（如 \<list\> 、\<set\> 、\<map\>）来配置集合属性。
        - 配置 java.util.List 类型的属性，需要指定 \<list\> 标签，在标签里包含一些元素，这些标签可以通过 \<value\> 指定简单的常量值，也可以通过 \<ref\> 指定对其他 Bean 的引用，也可以通过 \<bean\> 指定内置 Bean 定义，还可以通过 \<null/\> 指定空元素，甚至可以内嵌其他集合。
        - 数组的定义和 List 一样，都使用 \<list\>。
        - java.util.Set 需要使用 \<set\> 标签，定义元素的方法与 \<list\> 一致。
        - java.util.Map 通过 \<map\> 标签定义，\<map\> 标签里可以使用多个 \<entry\> 作为子标签，每个条目包含一个键和一个值。必须在 \<key\> 标签里定义键。因为键和值的类型没有限制，所以可以自由地为它们指定 \<value\>、\<ref\>、\<bean\>、\<null\> 元素。可以将 Map 的键和值作为 \<entry\> 的属性定义：简单常量使用 key 和 value 来定义，Bean 引用通过 key-ref 和 value-ref 属性定义。
        - 使用 \<props\> 定义 java.util.Properties，该标签使用多个 \<prop\> 作为子标签，每个 \<prop\> 标签必须定义 key 属性
        - 使用基本的集合标签定义集合时，不能将集合作为独立的 Bean 定义，导致其他的 Bean 无法引用该集合，所以无法在不同 Bean 之间共享集合。可以使用 util schema 里的集合标签定义独立的集合 Bean，需要注意的是必须在 \<beans\> 根元素里添加 util schema 定义
        - Spring 从 2.5 开始引入了一个 p 命名空间，可以通过 \<bean\> 元素属性的方式配置 Bean 的属性（需要先添加 p schema 定义）

- 自动装配

    - XML 配置中的 Bean 自动装配
        - Spring IOC 容器可以自动装配 Bean，需要做的仅仅是在 \<bean\> 的 autowire 属性里指定自动装配的模式
        - byType（根据类型自动装配）：若 IOC 容器中有多个与目标 Bean 类型一致的 Bean，在这种情况下，Spring 将无法判定哪个 Bean 最适合该属性，所以不能执行自动装配
        - byName（根据名称自动装配）：必须将目标 Bean 的名称和属性名设置得完全相同
        - constructor（通过构造器自动装配）：当 Bean 中存在多个构造器时，这种自动装配方式将会很复杂，不建议使用

----

##### Bean 之间关系

- 继承
    - Spring 允许继承 bean 的配置，被继承的 bean 称为父 bean，继承这个父 bean 的 bean 称为子 bean
    - 子 bean 从父 bean 中继承配置，包括 bean 的属性配置
    - 子 bean 也可以覆盖从父 bean 继承过来的配置
    - 父 bean 可以作为配置模板，也可以作为 bean 实例，若只想把父 bean 作为模板，可以设置 \<bean\> 的 abstract 属性为 true，这样 Spring 将不会实例化这个 bean
    - 并不是 \<bean\> 元素里的所有属性都会被继承，比如 autowire 、abstract 等
    - 也可以忽略父 bean 的 class 属性，让子 bean 指定自己的类，而共享相同的属性配置，但此时 abstract 必须设为 true
- 依赖
    - Spring 允许通过 depends-on 属性设定 bean 前置依赖的 bean，前置依赖的 bean 会在本 bean 实例化之前创建好
    - 如果前置依赖多个 bean，则可以通过逗号、空格的方式配置 bean 的名称

----

##### Bean 的作用域

- singleton（默认值）：单例，在实例化 IOC 容器时创建，并且整个 IOC 容器的生命周期内只初始化一次
- prototype：原型，实例化 IOC 容器时不创建，而是在每次获取这个 bean 时才实例化
- Web 环境作用域

----

##### 使用外部属性文件

- 在配置文件里配置 bean 时，有时需要在 bean 的配置里混入系统部署的细节信息（比如文件路径、数据源配置信息等），而这些部署细节实际上需要和 bean 配置分离

- Spring 提供了一个 PropertyPlaceholderConfigurer 的 BeanFactory 后置处理器，这个处理器允许用户将 bean 配置的部分内容移到属性文件中，可以在 bean 配置文件里使用形式为 \${var} 的变量，PropertyPlaceholderConfigurer 从属性文件中加载属性，并使用这些属性来替换变量

- Spring 还允许在属性文件中使用 \${propName}，以实现属性之间的相互引用

- 注册 PropertyPlaceholderConfigurer ：

    - Spring 2.5 之后通过 \<context:property-placeholder\> 元素导入，例如：

        ```XML
        <!-- 需要添加 context Schema 定义 -->
        
        <context:property-placeholder location="classpath:db.properties" />
        ```

----

##### SpEL

- Spring 表达式语言，是一个支持运行时查询和操作对象图的表达式语言

- 语法类似 EL：SpEL 使用 \#{...} 作为定界符

- SpEL 为 bean 的属性进行动态赋值提供了便利

- 通过 SpEL 可以实现：

    - 通过 bean 的 id 对 bean 进行引用
    - 调用方法以及引用对象中的属性
    - 计算表达式的值
    - 正则表达式的匹配

- 字面量：

    - 整数：

        ```XML
        <property name="count" value="#{5}" />
        ```

    - 小数：

        ```XML
        <property name="frequency" value="#{89.7}" />
        ```

    - 科学计数法：

        ```XML
        <property name="capacity" value="#{1e4}" />
        ```

    - String 可以使用单引号或者双引号作为字符串的定界符号：

        ```XML
        <property name="name" value="#{'Chunk'}" />
        
        <property name="name" value='#{"Chunk"}' />
        ```

    - Boolean：

        ```XML
        <property name="enabled" value="#{fasle}" />
        ```

- 引用 Bean、属性和方法

    - 引用其他对象：

        ```XML
        <!-- 通过 value 属性和 SpEL 配置 Bean 之间的引用关系 -->
        <property name="prefix" value="#{prefixGenerator}"></property>
        ```

    - 引用其他对象的属性：

        ```XML
        <!-- 通过 value 属性和 SpEL 配置 suffix 属性值为另一个 Bean 的 suffix 属性值 -->
        <property name="suffix" value="#{sequenceGenerator2.suffix}"></property>
        ```

    - 调用其他方法，还可以链式操作：

        ```XML
        <!-- 通过 value 属性和 SpEL 配置 suffix 属性值为另一个 Bean 的方法的返回值 -->
        <property name="suffix" value="${sequenceGenerator2.toString()}"></property>
        
        <!-- 方法的连缀 -->
        <property name="suffix" value="${sequenceGenerator2.toString().toUpperCase()}"></property>
        ```

    - 调用静态方法或静态属性：通过 T() 调用一个类的静态方法，它将返回一个 Class Object，然后再调用相应方法或属性

        ```XML
        <property name="initValue" value="#{T(java.lang.Math).PI}"></property>
        ```

- SpEL 支持的运算符号：

    - 算数运算符：\+,\-,\*,/,^
    - 加号也能用作字符串的连接
    - 比较运算符：\<,\>,==,\<=,\>=,lt,gt,eg,le,ge 
    - 逻辑运算符：and,or,not,\|
    - 三目运算符
    - 正则表达式：matches

----

##### IOC 容器中 Bean 的生命周期

- Spring IOC 容器可以管理 Bean 的生命周期，Spring 允许在 Bean 生命周期的特定点执行定制的任务
- Spring IOC 容器对 Bean 的生命周期进行管理的过程：
    - 通过构造器或工厂方法创建 Bean 实例
    - 为 Bean 的属性设置值和对其他 Bean 的引用
    - 调用 Bean 的初始化方法
    - Bean 可用
    - 当容器关闭时，调用 Bean 的销毁方法
- 在 Bean 的声明里设置 init-method 和 destroy-method 属性，为 Bean 指定初始化方法和销毁方法
- Bean 后置处理器允许在调用初始化方法前后对 Bean 进行额外的处理
- Bean 后置处理器对 IOC 容器里的所有 Bean 实例逐一处理，而非单一实例，其典型应用是检查 Bean 属性的正确性或根据特定的标准更改 Bean 的属性
- 对 Bean 后置处理器而言，需要实现 org.springframework.beans.factory.config.BeanPostProcessor 接口，在初始化方法被调用前后，Spring 将把每个 Bean 实例分别传递给上述接口的 postProcessAfterInitialization(Object, String) 与 postProcessBeforeInitialization(Object, String) 这两个方法
- 添加了 Bean 后置处理器后 Bean 的生命周期：
    - 通过构造器或工厂方法创建 Bean 实例
    - 为 Bean 的属性设置值和对其他 Bean 的引用
    - 将 Bean 实例传递给 Bean 后置处理器的 postProcessBeforeInitialization 方法
    - 调用 Bean 的初始化方法
    - 将 Bean 实例传递给 Bean 后置处理器的 postProcessAfterInitialization 方法
    - Bean 可用
    - 当容器关闭时，调用 Bean 的销毁方法

----

##### Spring 4.x 新特性：泛型依赖注入

- Spring 4.x 中可以为子类注入子类对应的泛型类型的成员变量的引用

----

##### Spring AOP

- 代码混乱与代码分散问题
- AOP（ Aspect-Oriented Programming），是一种新的方法论，是对传统 OOP 的补充
    - 连接点（ join point ）：在程序执行过程中某个特定的点，通常在这些点添加关注点的功能，比如某方法调用的时候或者处理异常的时候，在 Spring AOP 中，一个连接点总是代表一个方法的执行，表示”在什么地方做“。Spring 中只有方法是连接点，属性不能是连接点，也就是说只能切入方法
    - 切入点（ pointcut ）：匹配连接点的断言，通知和一个切入点表达式关联，并在满足这个切入点的连接点上运行，比如在执行某个特定名称的方法时。切入点表达式如何与连接点匹配是 AOP 的核心，Spring 默认使用 AspectJ 切入点语法
    - 通知（ advice ）：在切面的某个特定的连接点上执行的动作。通知有各种类型，其中包括 around、before、after 等通知。以拦截器为通知模型，并维护一个以连接点为中心的拦截器链，表示”具体怎么做“
    - 引入（ introduction ）：也被称为内部类型声明（ inter-type declaration ），为已有的类声明额外的方法或者某个类型的字段。Spring 允许引入新的接口（以及一个对应的实现类）到任何被代理的对象
    - 目标对象（ target object ）：被一个或多个切面所通知的对象，Spring AOP 是通过运行时代理实现的，故这个对象永远是一个被代理对象
    - 织入（ weaving ）：把切面连接到其他的应用程序类型或者对象上，并创建一个被通知的对象的过程。也就是说织入是一个过程，是将切面应用到目标对象从而创建出 AOP 代理对象的过程。这些可以在编译时，类加载时和运行时完成。Spring 在运行时完成织入
    - 切面（ Aspect ）：切面将切入点、引入、目标对象等信息集结在一起，从而定义相应的织入规则，这样一个整体称为切面。一个关注点的模块化，这个关注点可能会横切多个对象，综合表示”在什么地方，要做什么，以及具体如何做“
- 基于 AspectJ 注解方式
    - 启用 AspectJ 注解
        - 在 Spring 应用中使用 AspectJ 注解，必须在 classspath 下包含 AspectJ 类库：aopalliance.jar、aspectj.weaver.jar、spring-aspects.jar
        - 将 aop Schema 添加到 \<beans\> 根元素中
        - 在 Spring IOC 容器中启用 AspectJ 注解支持，只需要在 Bean 配置文件中定义一个空的 XML 元素 \<aop:aspectj-autoproxy\>
        - 当 Spring IOC 容器检测到 Bean 配置文件中的 \<aop:aspectj-autoproxy\> 元素时，会自动为与 AspectJ 切面匹配的 Bean 创建代理
    - 使用 AspectJ 注解声明且面
        - 在 Spring 中声明 AspectJ 切面，需要在 IOC 容器中将切面声明为 Bean 实例。当在 Spring IOC 容器中初始化 AspectJ 切面之后，Spring IOC 容器就会为那些与 AspectJ 切面相匹配的 Bean 创建代理
        - 在 AspectJ 注解中，切面只是一个带有 @Aspect 注解的 Java 类
        - 通知是标注有某种注解的简单的 Java 方法
        - AspectJ 支持 5 种类型的通知注解
            - @Before：前置通知，在方法执行之前执行
            - @After：后置通知，在方法执行之后执行
            - @AfterReturning：返回通知，在方法返回结果之后执行
            - @AfterThrowing：异常通知，在方法抛出异常之后执行
            - @Around：环绕通知，围绕着方法执行
        - 利用方法签名编写 AspectJ 切入点表达式
            - execution \* com\.aaa\.Test\.\*(\.\.)：匹配 Test 中声明的所有方法，第一个 \* 表示任意修饰符及任意返回值，第二个 \* 表示任意方法，\.\. 表示匹配任意数量的参数。若目标类和接口与该切面在同一个包内，可以省略包名
            - execution public \* Test\.\*(\.\.)：匹配 Test 接口的所有 public 方法
            - execution public double Test\.*(\.\.)：匹配 Test 中返回 double 类型数值的方法
            - execution public double Test\.\*(double, \.\.)：匹配第一个参数为 double 类型的方法，\.\. 匹配任意数量任意类型的参数
            - execution public double Test\.\*(double, double)：匹配任意参数类型为 double, double 类型的方法
        - 在 AspectJ 中，切入点表达式可以通过操作符 \&\&，\|\|，\! 等结合在一起
            - execution \* \*\.add(int, \.\.) || execution \* \*\.sub(int, \.\.)
        - 可以在通知方法中声明一个类型为 JoinPoint 的参数，然后就能访问连接细节，比如方法名称和参数值等
- 基于 XML 配置的 AOP

----

