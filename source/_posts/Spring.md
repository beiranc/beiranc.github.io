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

6. Bean 之间关系：

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

7. Bean 的作用域：

    - singleton（默认值）：单例，在实例化 IOC 容器时创建，并且整个 IOC 容器的生命周期内只初始化一次
    - prototype：原型，实例化 IOC 容器时不创建，而是在每次获取这个 bean 时才实例化
    - Web 环境作用域

8. 使用外部属性文件

    - 在配置文件里配置 bean 时，有时需要在 bean 的配置里混入系统部署的细节信息（比如文件路径、数据源配置信息等），而这些部署细节实际上需要和 bean 配置分离

    - Spring 提供了一个 PropertyPlaceholderConfigurer 的 BeanFactory 后置处理器，这个处理器允许用户将 bean 配置的部分内容移到属性文件中，可以在 bean 配置文件里使用形式为 \${var} 的变量，PropertyPlaceholderConfigurer 从属性文件中加载属性，并使用这些属性来替换变量

    - Spring 还允许在属性文件中使用 \${propName}，以实现属性之间的相互引用

    - 注册 PropertyPlaceholderConfigurer ：

        - Spring 2.5 之后通过 \<context:property-placeholder\> 元素导入，例如：

            ```XML
            <!-- 需要添加 context Schema 定义 -->
            
            <context:property-placeholder location="classpath:db.properties" />
            ```

9. SpEL

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

10. IOC 容器中 Bean 的生命周期

11. Spring 4.x 新特性：泛型依赖注入