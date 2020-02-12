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

##### 整合多个配置文件

- Spring 允许通过 \<import\> 将多个配置文件引入到一个文件中，进行配置文件的集成，这样在启动 Spring 容器时，仅需要指定这个合并好的配置文件即可

- \<import\> 元素的 resource 属性支持 Spring 标准的路径资源

    |  地址前缀  |                 示例                 |                      对应资源类型                      |
    | :--------: | :----------------------------------: | :----------------------------------------------------: |
    | classpath: |       classpath:spring-mvc.xml       | 从类路径下加载资源，classpath: 和 classpath:/ 是等价的 |
    |   file:    | file:/conf/security/spring-shiro.xml |     从文件系统目录中加载资源，可采用绝对或相对路径     |
    |  http://   |  http://xxx.xxx/resources/beans.xml  |                从 Web 服务器中加载资源                 |
    |   ftp://   |  ftp://xxx.xxx/resources/beans.xml   |                从 FTP 服务器中加载资源                 |

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
    - 使用 AspectJ 注解声明切面
        - 在 Spring 中声明 AspectJ 切面，需要在 IOC 容器中将切面声明为 Bean 实例。当在 Spring IOC 容器中初始化 AspectJ 切面之后，Spring IOC 容器就会为那些与 AspectJ 切面相匹配的 Bean 创建代理
        - 在 AspectJ 注解中，切面只是一个带有 @Aspect 注解的 Java 类
        - 通知是标注有某种注解的简单的 Java 方法
        - AspectJ 支持 5 种类型的通知注解
            - @Before：前置通知，在目标方法执行之前执行
            - @After：后置通知，在目标方法执行之后执行（无论是否出现异常）。注：在后置通知中不能访问目标方法的执行结果
            - @AfterReturning：返回通知，在目标方法返回结果之后执行。注：返回通知是可以访问到目标方法的执行结果的（注解中的 returning 属性，这个属性的值即为用来传入返回值的参数名称。原始的切点表达式需要出现在 pointcut 属性中）
            - @AfterThrowing：异常通知，在目标方法抛出异常之后执行。注：可以访问到异常对象，且可以指定在出现特定异常时再执行通知（将 throwing 属性添加到注解中）
            - @Around：环绕通知，围绕着方法执行
                - 环绕通知是所有通知类型中功能最全的，能够全面控制连接点，甚至可以控制是否执行连接点
                - 对于环绕通知来说，连接点的参数类型必须是 ProceedingJoinPoint，它是 JoinPoint 的子接口，允许控制何时执行，是否执行连接点
                - 在环绕通知中需要明确调用 ProceedingJoinPoint 的 proceed() 方法来执行被代理的方法，如果忘记这样做就会导致通知被执行了，但是目标方法没有被执行
                - 注意：环绕通知的方法需要返回目标方法执行之后的结果，即调用 joinPoint\.proceed(); 的返回值，否则会出现空指针异常
        - 利用方法签名编写 AspectJ 切入点表达式
            - execution \* com\.aaa\.Test\.\*(\.\.)：匹配 Test 中声明的所有方法，第一个 \* 表示任意修饰符及任意返回值，第二个 \* 表示任意方法，\.\. 表示匹配任意数量的参数。若目标类和接口与该切面在同一个包内，可以省略包名
            - execution public \* Test\.\*(\.\.)：匹配 Test 接口的所有 public 方法
            - execution public double Test\.*(\.\.)：匹配 Test 中返回 double 类型数值的方法
            - execution public double Test\.\*(double, \.\.)：匹配第一个参数为 double 类型的方法，\.\. 匹配任意数量任意类型的参数
            - execution public double Test\.\*(double, double)：匹配任意参数类型为 double, double 类型的方法
        - 在 AspectJ 中，切入点表达式可以通过操作符 \&\&，\|\|，\! 等结合在一起
            - execution \* \*\.add(int, \.\.) || execution \* \*\.sub(int, \.\.)
        - 可以在通知方法中声明一个类型为 JoinPoint 的参数，然后就能访问连接细节，比如方法名称和参数值等
        - 指定切面的优先级
            - 在同一个连接点上应用不止一个切面时，除非明确指定，否则它们的优先级是不确定的
            - 切面的优先级可以通过实现 org\.springframework\.core\.Ordered 接口或利用 @Order 注解指定
            - 实现 org\.springframework\.core\.Ordered 接口，getOrder() 方法的返回值越小，优先级越高。若使用 @Order 注解，则需要在注解中加入优先级，例如 @Order(1)、@Order(2) 等
        - 重用切入点定义
            - 在编写 AspectJ 切面时，可以直接在通知注解中写切入点表达式，但是同一个切入点表达式可能会出现在多个通知中
            - 在 AspectJ 中，可以通过 @Pointcut 注解将一个切入点声明成简单的方法，切入点的方法体通常是空的，因为将切入点定义与代码逻辑混在一起是不合理的
            - 切入点方法的访问控制符同时也控制着这个切入点的可见性，如果切入点需要在多个切面中共用，最好将它们集中在一个公共的类中，在这种情况下，它们必须被声明为 public，在引入这个切入点时，必须将类名也包括在内，如果类没有与这个切面放在同一个包里，还需要包含其包名。其他通知利用通过该方法引入该切点
        - 引入通知
            - 引入通知允许动态的往目标类中插入新的目标方法
            - 在切面中，通过为任意字段添加 @DeclareParents 注解来引入声明
            - 注解类型的 value 属性表示哪些类是当前引入通知的目标，value 属性也可以是一个 AspectJ 类型的表达式，以将一个接口引入到多个类中，defaultImpl 属性中指定这个接口使用的实现类
- 基于 XML 配置的 AOP
    - 声明切面
        - 在 \<beans\> 根元素中导入 aop Schema
        - 在 Bean 配置文件中，所有的 Spring AOP 配置都必须定义在 \<aop:config\> 元素内部，对于每个切面而言，都要创建一个 \<aop:config\> 元素来为具体的切面实现引用后的 Bean 实例
        - 切面 Bean 必须有一个标示符，供 \<aop:aspect\> 元素引用
    - 声明切入点
        - 切入点使用 \<aop:pointcut\> 元素声明
        - 切入点必须定义在 \<aop:aspect\> 元素下，或者直接定义在 \<aop:config\> 元素下
            - 定义在 \<aop:aspect\> 元素下：只对当前切面有效
            - 定义在 \<aop:config\> 元素下：对所有切面都有效
        - 基于 XML 的 AOP 配置不允许在切入点表达式中用名称引用其他的切入点
    - 声明通知
        - 在 aop Schema 中，每种通知类型都对应一个特定的 XML 元素
        - 通知元素需要使用 \<pointcut-ref\> 来引用切入点，或用 \<pointcut\> 直接嵌入切入点表达式，method 属性指定切面类中通知方法的名称
    - 声明引入
        - 可以使用 \<aop:declare-parents\> 元素在切面内部声明引入

----

##### Spring JDBC

- JDBC Template

    - 为了使 JDBC 更加易于使用，Spring 在 JDBC API 上定义了一个抽象层

    - 作为 Spring JDBC 的核心，JDBC Template 的设计目的是为不同类型的 JDBC 操作提供模板方法，每个模板方法都能控制整个过程，并允许覆盖过程中的特定任务，通过这种方式，可以在尽可能保留灵活性的情况下，将数据库存取的工作量降到最低

    - 常用方法

        - 用 SQL 语句和参数更新数据库：`public int update(String sql, Object... args) throws DataAccessException`
        - 批量更新数据库：`public int[] batchUpdate(String sql, List<Object[]> batchArgs)`
        - 查询单行：`public <T> T queryForObject(String sql, ParameterizeRiwMapper<T> rm, Object... args) throws DataAccessException`
        - 便利的 BeanPropertyRowMapper 实现：org.springframework.jdbc.core.BeanPropertyRowMapper\<T\>
        - 查询多行：`public <T> List<T> query(String sql, ParameterizeRiwMapper<T> rm, Object... args) throws DataAccessException`
        - 单值查询：`public <T> T queryForObject(String sql, Class<T> requiredType, Object... args) throws DataAccessException`

    - 简化 JDBC 模板查询

        - JdbcTemplate 类被设计成线程安全的，所以可以在 IOC 容器中声明它的单个实例，并将这个实例注入到所有的 DAO 实例中
        - JDBC Template 也利用了 JDK1.5 的特性（自动装箱、泛型、可变长参数等）来简化开发
        - Spring JDBC 还提供了一个 JdbcDaoSupport 类来简化 DAO 实现，该类声明了 jdbcTemplate 属性，它可以从 IOC 容器中注入，或者自动从数据源中创建

        ```XML
        <!-- 注入 JDBC 模板示例 -->
        <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
            <property name="dataSource" ref="dataSource"></property>
        </bean>
        
        <bean id="personDAO" class="xxx.xxx.PersonDAO">
            <property name="jdbcTemplate" ref="jdbcTemplate"></property>
        </bean>
        
        <!-- 扩展 JdbcDaoSupport 示例 -->
        <!-- public class PersonDAO extends JdbcDaoSupport { } -->
        <bean id="personDAO" class="xxx.xxx.PersonDAO">
            <property name="dataSource" ref="dataSource"></property>
        </bean>
        ```

    - 在 JDBC Template 中使用命名参数

        - 在标准的 JDBC 用法中 SQL 参数是用占位符 "?" 来表示的，并且受到位置的限制。定位参数的问题在于，一旦参数的顺序发生改变，就必须改变参数绑定
        - 在 Spring JDBC 中，绑定 SQL 参数的另一种选择是使用命名参数（ named parameter）
        - 命名参数：SQL 按名称（以冒号开头）而不是按位置进行指定，命名参数更易维护，也提升了可读性，命名参数由框架类在运行时用占位符替代
        - 命名参数只在 NamedParameterJdbcTemplate 中得到支持
        - 在 SQL 语句中使用命名参数时，可以在一个 Map 中提供参数值，参数名为键。也可以使用 SqlParameterSource 参数
        - 批量更新时可以提供 Map 或者 SqlParameterSource 的数组

----

##### Spring 的事务管理

- 事务管理用来确保数据的完整性和一致性

- 事务就是一系列的动作，它们被当作一个单独的工作单元，这些动作要么全部完成，要么全部都不起作用

- 事务的四个关键属性（ ACID ）

    - 原子性（ atomicity ）：事务是一个原子操作，由一系列动作组成，事务的原子性确保动作要么全部完成要么完全不起作用
    - 一致性（ consistency ）：一旦所有的事务动作完成，事务就被提交，数据和资源就处于一种满足业务规则的一致性状态中
    - 隔离性（ isolation ）：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏
    - 持久性（ durability ）：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，通常情况下，事务的结果被写到持久化存储器中

- Spring 中的事务管理

    - Spring 在不同的事务管理 API 之上定义了一个抽象层，而开发者不必了解底层的事务管理 API，就可以使用 Spring 的事务管理机制

    - Spring 既支持编程式事务管理也支持声明式事务管理

        - 编程式事务管理：将事务管理代码嵌入到业务方法中来控制事务的提交和回滚，在使用编程式事务管理时，必须在每个事务操作中包含额外的事务管理代码
        - 声明式事务管理：它将事务管理代码从业务方法中分离出来，以声明的方式来实现事务管理。Spring 通过 Spring AOP 来进行声明式事务管理

    - Spring 的核心事务管理抽象是 org.springframework.transaction.PlatformTransactionManager

        - org.springframework.jdbc.datasource.DataSourceTransactionManager：在应用程序中只需要处理一个数据源，而通过 JDBC 存取
        - org.springframework.transaction.jta.JtaTransactionManager：在 JavaEE 应用服务器上用 JTA（ Java Transaction API ）进行事务管理
        - HibernateTransactionManager：用 Hibernate 框架存取数据库
        - 注：事务管理器以普通的 Bean 形式声明在 Spring IOC 容器中

    - 用事务通知声明式地管理事务

        - 启用声明式事务管理可以通过 tx Schema 中定义的 \<tx:advice\> 元素声明事务通知，为此需要将 tx Schema 定义添加到 \<beans\>
        - 声明了事务通知后，需要将其与切入点关联，由于事务通知是在 \<aop:config\> 元素外声明的，所以它无法直接与切入点关联，必须在 \<aop:config\> 元素中声明一个增强器通知与切入点关联
        - 由于 Spring AOP 是基于代理的方法，故只能增强 public 方法。故只有 public 方法才能通过 Spring AOP 进行事务管理

        ```XML
        <!-- 事务通知声明式管理事务示例 -->
        <bean id="bookShopService" class="xx.xx.BookShopServiceImpl">
            <property name="bookShopDAO" ref="bookDAO"></property>
        </bean>
        
        <!-- 声明事务管理器 -->
        <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource"></property>
        </bean>
        
        <!-- 声明事务通知 -->
        <tx:advice id="bookShopTxAdvice" transaction-manager="transactionManager"></tx:advice>
        
        <!-- 声明事务通知需要通知的方法（即需要进行事务管理的方法） -->
        <aop:config>
            <aop:pointcut expression="execution(* *.BookShopService.*(..))" id="bookShopOperation"></aop:pointcut>
            <aop:advisor advice-ref="bookShopTxAdvice" pointcut-ref="bookShopOperation"></aop:advisor>
        </aop:config>
        ```

    - 用 @Transactional 注解声明式地管理事务

        - 除了在带有切入点，通知和增强器的 Bean 配置文件中声明事务以外，Spring 还允许简单的使用 @Transactional 注解来标注事务方法
        - 为了将方法定义为支持事务处理的，可以为方法添加 @Transactional 注解，根据 Spring AOP 的机制，只能标注 public 方法
        - 可以在类级别上添加 @Transactional 注解，应用到类上时，这个类中所有的 public 方法都将支持事务处理
        - 在 Bean 配置文件中启用 \<tx:annotation-driven\> 元素，并为之指定事务管理器即可
        - 若事务管理器的名称为 transactionManager，则可以在 \<tx:annotation-driven\> 元素中省略 transaction-manager 属性，因为它会自动检测该名称的事务管理器

        ```XML
        <!-- @Transactional 注解声明式管理事务的配置文件示例 -->
        <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
            <property name="dataSource" ref="dataSource"></property>
        </bean>
        
        <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource"></property>
        </bean>
        
        <context:component-scan base-package="xxx.xxx.transaction"></context:component-scan>
        
        <tx:annotation-driven></tx:annotation-driven>
        ```

    - 事务传播属性

        - 当事务方法被另一个事务方法调用时，必须指定事务应该如何传播，Spring 定义了 7 种类型的传播行为

        |    传播属性    |                             描述                             |
        | :------------: | :----------------------------------------------------------: |
        | REQUIRED(默认) | 若有事务在运行，当前的方法就在这个事务内运行，否则就启动一个新的事务，并在自己的事务内运行 |
        |  REQUIRED_NEW  | 当前的方法必须启动新事务，并在它自己的事务内运行，如果有事务正在运行，应该将它挂起 |
        |    SUPPORTS    | 如果有事务在运行，当前的方法就在这个事务内运行，否则它可以不运行在事务中 |
        |  NOT_SUPPORTS  |   当前的方法不应该运行在事务中，如果有运行的事务，将它挂起   |
        |   MANDATORY    | 当前的方法必须运行在事务内部，如果没有正在运行的事务，就抛出异常 |
        |     NEVER      |  当前的方法不应该运行在事务中，如果有运行的事务，就抛出异常  |
        |     NESTED     | 如果有事务在运行，当前的方法就应该在这个事务的嵌套事务内运行，否则就启动一个新的事务，并在它自己的事务内运行 |

        - 事务传播属性可以在 @Transactional 注解的 propagation 属性中定义。也可以在 \<tx:advice\> 元素中添加 \<tx:attributes\> 元素，然后在 \<tx:attributes\> 元素中添加 \<tx:method\> 元素来设定传播属性

            ```XML
            <!-- 事务的传播属性 XML 配置示例 -->
            <tx:advice id="bookShopTxAdvice" transaction-manager="transactionManager">
                <tx:attributes>
                    <tx:method name="methodName" propagation="REQUIRES_NEW"></tx:method>
                </tx:attributes>
            </tx:advice>
            ```

    - 并发事务导致的问题

        - 脏读
        - 不可重复读
        - 幻读

    - 事务的隔离级别

        - Spring 支持的事务隔离级别

            |    隔离级别     |                             描述                             |
            | :-------------: | :----------------------------------------------------------: |
            |     DEFAULT     | 使用底层数据库的默认隔离级别，对于大多数数据库来说，默认隔离级别都是 READ_COMMITED |
            | READ_UNCOMMITED | 允许事务读取未被其他事务提交的变更，脏读，不可重复读和幻读的问题都会出现 |
            |  READ_COMMITED  | 只允许事务读取已经被其他事务提交的变更，可以避免脏读，但不可重复读和幻读的问题仍然可能出现 |
            | REPEATABLE_READ | 确保事务可以多次从一个字段中读取相同的值，在这个事务持续期间，禁止其他事务对这个字段进行更新，可以避免脏读和不可重复读，但幻读的问题仍然存在 |
            |  SERIALIZEABLE  | 确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其他事务对该表执行插入，更新和删除操作，所有并发问题都可以避免，但性能非常低 |

        - 事务的隔离级别是需要底层数据库引擎库支持，而不是应用程序或框架的支持

        - Oracle 支持的两种事务隔离级别：READ_COMMITED、SERIALIZEABLE

        - Mysql 支持四种事务隔离级别

        - 设置隔离事务属性

            - 用 @Transactional 注解声明式地管理事务时可以在 @Transactional 的 isolation 属性中设置隔离级别
            - 也可以在配置文件中指定（同传播属性配置方式）

    - 设置回滚事务属性

        - 默认情况下只有未检查异常（ RuntimeException 和 Error 类型的异常）会导致事务回滚，而受检查异常不会
        - 事务的回滚规则可以通过 @Transactional 注解的 rollbackFor 和 nonRollbackFor 属性来定义，这两个属性被声明为 Class[] 类型的，因此可以为这两个属性指定多个异常类
            - rollbackFor：遇到时必须进行回滚
            - nonRollbakcFor：遇到时必须不回滚
        - 在配置文件中指定的方式与传播属性配置方法相同，如果有不止一种异常，用逗号分隔

    - 设置超时和只读属性

        - 超时事务属性：事务在强制回滚之前可以保持多久，这样可以防止长期运行的事务占用资源
        - 只读事务属性：表示这个事务只读取数据但不更新数据，这样可以帮助数据库引擎优化事务
        - 超时（ timeout ）和只读（ readOnly ）属性可以在 @Transactional 注解中定义，超时属性以秒为单位
        - 在配置文件中指定的方式与传播属性配置方法相同

----

##### Spring 整合 Hibernate

- Spring 整合 Hibernate 是为了让 IOC 容器来管理 Hibernate 的 SessionFactory 以及让 Hibernate 使用 Spring 的声明式事务（注：不推荐使用 HibernateTemplate 和 HibernateDaoSupport，因为使用它们会导致 Hibernate 与 Spring 的耦合）

- 在 Spring 中配置 SessionFactory

    - 可以使用 LocalSessionFactoryBean 工厂 Bean 来声明一个使用 XML 映射文件的 SessionFactory 实例
    - 需要为该工厂 Bean 指定 configLocation 属性来加载 Hibernate 配置文件或者直接使用 hibernateProperties 属性来设定 Hibernate 的配置
    - 如果在 Spring IOC 容器中配置数据源，可以将该数据源注入到 LocalSessionFactoryBean 的 dataSource 属性中，该属性指定的数据源会覆盖掉 Hibernate 配置文件中的数据库配置
    - 可以在 LocalSessionFactoryBean 的 mappingResources 属性中指定 XML 映射文件的位置，该属性为 String[] 类型，因此可以指定一组映射文件

- 使用 Spring 的 ORM 模板持久化对象

    - 同 JDBC 一样，Spring 采取了相同的方法，即定义模板类和 DAO 支持类来简化 ORM 框架的使用，而且 Spring 在不同的事务管理 API 之上定义了一个事务抽象层，对于不同的 ORM 框架，只需要选择相应的事务管理器实现即可

- Spring 对不同数据存储策略的支持类

    |   支持类   |             JDBC             |          Hibernate          |
    | :--------: | :--------------------------: | :-------------------------: |
    |   模板类   |         JdbcTemplate         |      HibernateTemplate      |
    | DAO 支持类 |        JdbcDaoSupport        |     HibernateDaoSupport     |
    | 事务管理类 | DataSourceTransactionManager | HibernateTransactionManager |

    - HibernateTemplate 确保了 Hibernate Session 能够正确的打开和关闭
    - HibernateTemplate 也会让原生的 Hibernate 事务参与到 Spring 的事务管理中体系中，从而利用 Spring 的声明式事务管理事务
    - HibernateTemplate 中的模板方法管理 Session 和事务，如果在一个支持事务的 DAO 方法中有多个 Hibernate 操作，模板方法可以确保它们会在同一个 Session 和事务中运行，因此没有必要为了 Session 和事务管理去和 Hibernate API 打交道
    - 通过为 DAO 方法添加 @Transactional 注解将其声明为受事务管理的
    - HibernateTemplate 类是线程安全的，因此可以在 Bean 配置文件中只声明一个实例，然后将其注入到所有的 Hibernate DAO 中
    - Hibernate DAO 可以通过继承 HibernateDaoSupport 来继承 setSessionFactory() 和 setHibernateTemplate() 方法，然后，只要在 DAO 方法中调用 getHibernateTemplate() 方法就可以获取到模板实例
    - 如果为 HibernateDaoSupport 实现类注入了 SessionFactory 实例就不需要再为之注入 HibernateTemplate 实例了。因为 HibernateDaoSupport 会根据传入的 SessionFactory 在其构造器内创建 HibernateTemplate 实例并赋给 hibernateTemplate 属性

- 使用 Hibernate 的上下文 Session 持久化对象

    - Spring 的 HibernateTemplate 可以管理 Session 和事务，简化 DAO 的实现，但是如果要使用 HibernateTemplate 就意味着 DAO 必须依赖于 Spring

    - Hibernate 上下文 Session 对象和 Spring 的事务管理结合的很好，但此时需要保证所有的 DAO 方法都支持事务。此时不需要在 Bean 配置文件中配置，因为 Spring 已经开始事务了，只需在 ThreadLoacl 对象中绑定 Session 对象即可

        ```XML
        <prop keys="hibernate.current_session_context_class">thread</prop>
        ```

    - 为了保持一致的异常处理方法，即把 Hibernate 异常转换为 Spring 的 DataAccessException 异常，那么必须为需要转换的 DAO 类添加 @Repository 注解，然后再注册一个 org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor 实例，将原生的 Hibernate 异常转换为 Spring 的 DataAccessException 层次结构中的数据存取异常，这个 Bean 后置处理器只为添加了 @Repository 注解的 Bean 转换异常

    - 从 Hibernate3 开始，SessionFactory 新增了一个 getCurrentSession() 方法，该方法可直接获取上下文相关的 Session

    - Hibernate 通过 CurrentSessionContext 接口的实现类和配置参数 hibernate.current_session_context_class 定义上下文

        - JTASessionContext：根据 JTA 来跟踪和界定 Session 对象
        - ThreadLocalSessionContext：通过当前正在执行的线程来跟踪和界定 Session 对象
        - ManagedSessionContext

    - 若使用的为 ThreadLocalSessionContext 策略，Hibernate 的 Session 会随着 getCurrentSession() 方法自动打开，随着事务提交而自动关闭

----

##### 在 Web 应用中使用 Spring

- 通过注册 Servlet 监听器 ContextLoaderListener，Web 应用程序可以加载 Spring 的 ApplicationContext 对象，这个监听器会将加载好的 ApplicationContext 对象保存到 Web 应用程序的 ServletContext 中，随后，Servlet 或者可以访问 ServletContext 的任意对象就能通过一个辅助方法来使用 Spring IOC 容器中的对象了