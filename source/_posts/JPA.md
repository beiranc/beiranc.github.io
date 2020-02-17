---
title: JPA
date: 2020-02-15 15:20:17
tags: JPA
categories: JPA
comments: false
---

##### JPA 概述<!-- more -->

1. JPA（Java Persistence API）：用于对象持久化的 API，JavaEE 5.0 平台标准的 ORM 规范，使得应用程序以统一的方式访问持久层
2. JPA 与 Hibernate 的关系
    - JPA 是 Hibernate 的一个抽象（就像 JDBC 和 JDBC 驱动的关系）
    - JPA 本质上是一种 ORM 规范，不是 ORM 框架，因为 JPA 并未提供 ORM 实现，它只是制定了一些规范，提供了一些编程的 API 接口，但具体实现要由 ORM 厂商来实现
    - Hibernate 除了作为 ORM 框架以外，它也是一种 JPA 实现
    - 从功能上来说，JPA 是 Hibernate 功能的一个子集
3. JPA 的目标之一是制定一个可以由很多供应商实现的 API，例如 Hibernate、OpenJPA、TopLink 等
4. JPA 优势
    - 标准化
        - 提供相同的 API，这保证了基于 JPA 开发的企业应用能够经过少量的修改就能够在不同的 JPA 框架下运行
    - 简单易用，集成方便
        - JPA 的主要目标之一就是提供更加简单的编程模型，在 JPA 框架下创建实体和创建 Java 类一样简单，只需要使用 javax.persistence.Entity 进行注释，JPA 的框架和接口也都非常简单
    - 可媲美 JDBC 的查询能力
        - JPA 的查询语言是面向对象的，JPA 定义了独特的 JPQL，而且能够支持批量更新和修改、JOIN、GROUP BY、HAVING 等通常只有 SQL 才能提供的高级查询特性，甚至还能够支持子查询
    - 支持面向对象的高级特性
        - JPA 中能够支持面向对象的高级特性，例如类之间的继承、多态和类之间的复杂关系，最大限度的使用面向对象的模型
5. JPA 包括三方面的技术
    - ORM 映射元数据
        - JPA 支持 XML 和 JDK5.0 注解两种元数据的形式，元数据描述对象和表之间的映射关系，框架据此将实体对象持久化到数据库表中
    - JPA 的 API
        - 用来操作实体对象，执行 CRUD 操作，框架在后台完成所有的事情，开发者可以不用编写繁琐的 JDBC 和 SQL 代码
    - 查询语言 JPQL
        - 通过面向对象而非面向数据库的查询语言查询数据，避免程序和具体的 SQL 紧密结合

----

##### 使用 JPA 持久化对象的步骤

1. 导入所需 JAR 包（以 Hibernate 实现为例）

    - hibernate\\lib\\required\\\*\.jar
    - hibernate\\lib\\jpa\\\*\.jar
    - 数据库驱动

2. 创建 persistence.xml，在这个文件中配置持久化单元（ JPA 规范要求放在类路径的 META-INF 目录中且文件名固定）

    - 需要指定与哪个数据库进行交互

    - 需要指定 JPA 使用哪个持久化的框架以及配置该框架的基本属性

    - persistence.xml 文件中元素含义

        ```XML
        <persistence>
            <!-- name 属性用于定于持久化单元的名字 -->
            <!-- 
        		transaction-type 用于指定 JPA 的事务处理策略
        		RESOURCE_LOCAL: 默认值，数据库级别的事务，只能针对一种数据库，不支持分布式事务
        		如需支持分布式事务，应使用 JTA: transaction-type="JTA"
         	-->
            <persistence-unit name="JPA" transaction-type="RESOURCE_LOCAL">
                <!-- 
        			配置使用什么 ORM 框架来作为 JPA 的实现
        			实际上配置的是 javax.persistence.spi.PersistenceProvider 接口的实现类
        			若 JPA 项目中只有一个 JPA 的实现框架, 则也可以不配置该节点 
        		-->
                <provider>org.hibernate.ejb.HibernatePersistence</provider>
                
                <!-- 添加持久化类 -->
                <class>beiran.bean.Customer</class>
                
                <!-- 
                    配置二级缓存的策略 
                    ALL：所有的实体类都被缓存
                    NONE：所有的实体类都不被缓存. 
                    ENABLE_SELECTIVE：标识 @Cacheable(true) 注解的实体类将被缓存
                    DISABLE_SELECTIVE：缓存除标识 @Cacheable(false) 以外的所有实体类
                    UNSPECIFIED：默认值，JPA 实现框架的默认值将被使用
        		-->
                <shared-cache-mode>ENABLE_SELECTIVE</shared-cache-mode>
                
                <properties>
                    <!-- 连接数据库的基本信息 -->
                    <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver"/>
                    <property name="javax.persistence.jdbc.url" value="jdbc:mysql:///jpa"/>
                    <property name="javax.persistence.jdbc.user" value="root"/>
                    <property name="javax.persistence.jdbc.password" value="123"/>
                    
                    <!-- 配置 JPA 实现框架的基本属性. 配置 Hibernate 的基本属性 -->
                    <property name="hibernate.format_sql" value="true"/>
                    <property name="hibernate.show_sql" value="true"/>
                    <property name="hibernate.hbm2ddl.auto" value="update"/>
                    
                    <!-- 二级缓存相关 -->
                    <property name="hibernate.cache.use_second_level_cache" value="true"/>
                    <property name="hibernate.cache.region.factory_class" value="org.hibernate.cache.ehcache.EhCacheRegionFactory"/>
                    <property name="hibernate.cache.use_query_cache" value="true"/>
                </properties>
            </persistence-unit>
        </persistence>
        ```

3. 创建实体类，使用注解来描述实体类跟数据库表之间的映射关系

4. 使用 JPA API 完成 CRUD 操作

    - 创建 EntityManagerFactory（对应 Hibernate 中的 SessionFactory ）
    - 创建 EntityManager（对应 Hibernate 中的 Session ）

    ```java
    // 创建 EntityManagerFactory 对象
    EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("JPA");
    
    // 创建 EntityManager 对象
    EntityManager entityManager = entityManagerFactory.createEntityManager();
    
    // 开启事务
    EntityTransaction entityTransaction = entityManger.getTransaction();
    entityTransaction.begin();
    
    // 执行持久化操作
    entityManager.persist(object);
    
    // 提交事务
    entityTransaction.commit();
    
    // 关闭 EntityManager
    entityManager.close();
    
    // 关闭 EntityManagerFactory
    entityManagerFactory.close();
    ```

----

##### JPA 基本注解

1. @Entity

    - @Entity 注解用于实体类声明语句前，指出该 Java 类是一个实体类，将映射到指定的数据库表

2. @Table

    - 当实体类与其映射的数据库表名不同名时，需要使用 @Table 注解指定数据表名，该注解与 @Entity 注解并列使用
    - @Table 注解常用的属性是 name，用于指定数据表名
    - @Table 注解其他属性
        - catalog 属性用于指定数据库目录，通常无需指定
        - schema 属性用于指定数据库模式，通常无需指定
        - uniqueConstraints 属性用于设置约束条件，通常无需设置

3. @Id

    - @Id 注解用于声明一个实体类的属性映射为数据表的主键列，该属性通常置于属性声明语句上（也可置于属性的 getter 方法之前）

4. @GeneratedValue

    - @GeneratedValue 注解用于标注主键的生成策略，通过 strategy 属性指定。默认情况下，JPA 自动选择一个最适合底层数据库的主键生成策略（ SQLServer 对应 identity，MySQL 对应 auto ）
    - 在 javax.persistence.GenerationType 中定义了以下几种可供选择的策略
        - IDENTITY：采用数据库 ID 自增长的方式来自增主键字段，Oracle 不支持此种方式
        - AUTO（默认值）：JPA 自动选择合适的策略
        - SEQUENCE：通过序列产生主键，通过 @SequenceGenerator 注解指定序列名，MySQL 不支持此种方式
        - TABLE：通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植

5. @Basic

    - @Basic 注解表示一个简单的属性到数据表的字段的映射，对于没有任何标注 @Basic 注解的 getter 方法，默认即为 @Basic
    - @Basic 注解的 fetch 属性表示该字段的读取策略，有 EAGER 和 LAZY 两种，分别表示主支抓取和延迟加载，默认为 EAGER。@Basic 注解的 optional 属性表示该字段是否允许为 null，默认为 true

6. @Column

    - 当实体的属性与其映射的数据表的列不同名时需要使用 @Column 注解说明，该属性通常置于实体的属性声明语句之前（也可以置于 getter 方法之前），还可以与 @Id 注解一起使用
    - @Column 注解常用的属性是 name，用于设置映射数据表的列名，此外，该注解还包含其他多个属性，例如 unique、nullable、length 等
    - @Column 注解的 columnDefinition 属性表示该字段在数据表中的实际类型，通常 ORM 框架可以根据属性类型自动判断数据表中字段的类型，但是对于 Date 类型仍然无法确定数据表中字段类型究竟是 DATE、TIME 还是 TIMESTAMP。此外，String 类型的默认映射类型为 VARCHAR，如果要将 String 类型映射到特定数据表的 BLOB 或者 TEXT，可以通过 columnDefinition 属性进行设置

7. @Transient

    - @Transient 注解表示该属性并非一个到数据表的字段的映射，ORM 框架将忽略该属性
    - 如果一个属性并非数据表的字段映射，就务必为其标注 @Transient 注解，否则，ORM 框架默认其注解为 @Basic

8. @Temporal

    - 在核心的 Java API 中并没有定义 Date 类型的精度，而在数据库中，表示 Date 类型的数据有 DATE、TIME 和 TIMESTAMP 三种精度，在进行属性映射时可以使用 @Temporal 注解来调整精度

9. 用数据表来生成主键

    - 将当前主键的值单独保存到一个数据表中，主键的值每次都是从指定的表中查询来获得
    - 这种方法生成主键的策略可以适用于任何数据库，不必担心不同数据库不兼容造成的问题

    ```Java
    // name 属性表示该主键生成策略的名称，它被引用在 @GeneratedValue 中设置的 generator 值中
    // table 属性表示存储生成的主键值的表名
    // pkColumnName 属性表示在实体类对应的表中，该主键生成策略所对应键值的名称
    // pkColumnValue 属性表示在实体类对应的表中，该生成策略所对应的主键
    // valueColumnName 属性表示在实体类对应的表中，该主键当前所生成的值，它的值会随着每次创建而累加
    // initialValue 属性表示生成的主键的初始值，默认为 0
    // allocationSize 属性表示每次主键值增加的大小，默认为 50
    @TableGenerator(name = "ID_GENERATOR", table = "JPA_ID_GENERATOR",
                   allocationSize = 1, initialValue = 1, 
                    pkColumnName = "PK_NAME", pkColumnValue = "PERSON_ID", 
                   valueColumnName = "ID_VAL")
    @GeneratedValue(strategy = GenerationType.TABLE, generator="ID_GENERATOR")
    @Id
    private Integer id;
    ```

    ![用数据表来生成主键](image-20200217000332301.png)

----

##### JPA API

1. Persistence
    - Persistence 类用于获取 EntityManagerFactory 实例，该类包含一个 createEntityManagerFactory 的静态方法
    - createEntityManagerFactory 方法有两个重载版本
        - 带有一个参数的方法以 JPA 配置文件 persistence.xml 中的持久化单元名（ persistence-unit ）作为参数
        - 带有两个参数的方法：前一个参数的含义相同，后一个参数为 Map 类型，用于设置 JPA 的相关属性，这时将忽略其他地方设置的属性。Map 对象的属性名必须是 JPA 实现框架提供商的命名空间约定的属性名
    
2. EntityManagerFactory
    - EntityManagerFactory 接口主要用于创建 EntityManager 实例
        - createEntityManager()：用于创建 EntityManager 对象实例
        - createEntityManager(Map map)：用于创建 EntityManager 对象实例的重载方法，Map 参数用于提供 EntityManager 的属性
        - isOpen()：检查 EntityManagerFactory 是否处于打开状态，EntityManagerFactory 创建之后就一直处于打开状态，除非调用 close() 方法将其关闭
        - close()：关闭 EntityManagerFactory，EntityManagerFactory 关闭后将释放所有资源，isOpen() 方法将返回 false，其他方法则不能调用，否则将抛出 IllegalStateException
    
3. EntityManager
    - 在 JPA 规范中，EntityManager 是完成持久化操作的核心对象，实体作为普通 Java 对象，只有在调用 EntityManager 将其持久化后才会变为持久化对象。EntityManager 对象在一组实体类与底层数据源之间进行 O/R 映射的管理，它可以用管理和更新 Entity Bean，根据主键查找 Entity Bean，还可以通过 JPQL 语句查询实体
    
    - 实体的状态
        - 新建状态：新创建的对象，尚未拥有持久性主键
        - 持久化状态：已经拥有持久性主键并和持久化建立了上下文环境
        - 游离状态：拥有持久化主键，但是没有与持久化建立上下文环境
        - 删除状态：拥有持久化主键，已经和持久话建立了上下文环境，但是已经从数据库中删除了
        
    - find(Class\<T\> entityClass, Object primaryKey)：返回指定的 OID 对应的实体类对象，如果这个实体存在于当前的持久化环境，则返回一个被缓存的对象，否则会创建一个新的 Entity，并加载数据库中相关信息。若 OID 不存在于数据库中，则返回一个 null。第一个参数为被查询的实体类类型，第二个参数为待查找实体的主键值
    
    - getReference(Class\<T\> entityClass, Object primaryKey)：与 find() 方法类似，不同的是如果缓存中不存在指定的 Entity，EntityManager 会创建一个 Entity 类的代理，但是不会立即加载数据库中的信息，只有第一次真正使用此 Entity 的属性才加载，所以如果此 OID 在数据库中不存在，getReference() 方法不会返回 null 值，而是抛出 EntityNotFoundException
    
    - persist(Object entity)：用于将新创建的 Entity 纳入到 EntityManager 的管理，该方法执行后，传入 persist() 方法的 Entity 对象将转换成持久化状态
    
        - 如果传入 persist() 方法的 Entity 对象已经处于持久化状态，则 persist() 方法什么都不做
        - 如果对删除状态的 Entity 对象进行 persist() 操作，会转换为持久化状态
        - 如果对游离态的 Entity 对象进行 persist() 操作，可能会在 persist() 方法抛出 EntityExistException（也有可能是在 flush 或事务提交后抛出）
    
    - remove(Object entity)：删除实例，如果实例是被管理的（即与数据库实体记录关联），则同时会删除关联的数据库记录
    
    - merge(T entity)：用于处理 Entity 的同步（即数据库的插入和更新操作）
    
        ![JPA merge() 流程](JPA%20merge()%20%E6%B5%81%E7%A8%8B.png)
    
    - flush()：同步持久化上下文环境（即将持久化上下文环境中的所有未保存实体的状态信息保存到数据库中）
    
    - setFlushMode(FlushModeType flushMode)：设置持久化上下文环境的 Flush 模式，参数可以取两个枚举值
    
        - FlushModeType.AUTO 为自动更新数据库实体
        - FlushModeType.COMMIT 为直到提交事务时才更新数据库记录
    
    - getFlushMode()：获取持久化上下文环境的 Flush 模式，返回 FlushModeType 类型的枚举值
    
    - refresh(Object entity)：用数据库实体记录的值更新实体对象的状态（即更新实例的属性值）
    
    - clear()：清除持久化上下文环境，断开所有关联的实体，如果这时候还有未提交的更新则会被撤销
    
    - contains(Object entity)：判断一个实例是否是当前持久化上下文环境管理的实体
    
    - isOpen()：判断当前的 EntityManager 是否为打开状态
    
    - getTransaction()：获取资源层的事务对象，EntityTransaction 实例可用于开始和提交多个事务
    
    - close()：关闭 EntityManager，之后若调用 EntityManager 实例的方法或其派生的查询对象的方法都将抛出 IllegalStateException，除了 getTransaction() 和 isOpen() 方法（返回 false ）。但是，当与 EntityManager 关联的事务处于活动状态时，调用 close() 方法后持久化上下文环境将仍处于被管理状态直到事务完成为止
    
    - createQuery(String qlString)：创建一个查询对象
    
    - createNamedQuery(String name)：根据命名的查询语句块创建查询对象，参数为命名的查询语句
    
        - 使用 @NamedQuery 注解（标注在类上，且可标注多个），该注解的 name 属性指定 JPQL 语句名称，query 属性指定 JPQL 语句
    
    - createNativeQuery(String sqlString)：使用标准 SQL 语句创建查询对象，参数为标准 SQL 语句字符串
    
    - createNativeQuery(String sqls, String resultSetMapping)：使用标准 SQL 语句创建查询对象，并指定返回结果集 Map 的名称
    
4. EntityTransaction

    - EntityTransaction 接口用来管理资源层 EntityManager 的事务操作，通过调用 EntityManager 的 getTransaction() 方法获取其实例
    - begin()：用于启动一个事务，此后的多个数据库操作将作为整体被提交或回滚，若这时事务已经启动则会抛出 IllegalStateException
    - commit()：用于提交当前事务（即将事务启动以后的所有数据库更新操作持久化到数据库中）
    - rollback()：回滚当前事务（即回滚事务启动后的所有数据库更新操作，从而不对数据库产生影响）
    - isActive()：查询当前事务是否是活动的。如果返回 true 则不能调用 begin() 方法，否则将抛出 IllegalStateException。如果返回 false 则不能调用 commit()、rollback()、setRollbackOnly() 及 getRollbackOnly() 方法，否则将抛出 IllegalStateException

----

##### 映射关联关系

1. 映射单向多对一的关联关系

    - 使用 @ManyToOne 注解来映射单向多对一的关联关系
    - 使用 @JoinColumn 注解来映射外键列的名称
    - 可以使用 @ManyToOne 注解的 fetch 属性来修改默认的关联属性的加载策略。默认情况下，使用左外连接的方式来获取 n 的一端的对象和其关联的 1 的一端的对象
    - 保存多对一时，建议先保存 1 的一端，再保存 n 的一端，这样不会多出额外的 UPDATE 语句
    - 不能直接删除 1 的一端，因为有外键约束

2. 映射单向一对多的关联关系

    - 使用 @OneToMany 注解来映射一对多的关联关系
    - 使用 @JoinColumn 注解来映射外键列的名称
    - 可以使用 @OneToMany 注解的 fetch 属性来修改默认的加载策略。可以通过 @OneToMany 注解的 cascade 属性来修改默认的删除策略（默认情况下，若删除 1 的一端，会先把关联的 n 的一端的外键置为 null，然后再进行删除）
    - 单向一对多关联关系执行保存时，一定会多出 UPDATE 语句，因为 n 的一端在插入时不会同时插入外键列

3. 映射双向多对一的关联关系

    - 双向一对多关系中，必须存在一个关系维护端，在 JPA 规范中，要求 n 的一端作为关系的维护端（ owner side ），1 的一端作为被维护端（ inverse side )
    - 可以在 1 的一端指定 @OneToMany 注解并设置 mappedBy 属性，以指定它是这一关联中的被维护端，n 的一端为维护端。若在 1 的一端的 @OneToMany 中使用 mappedBy 属性，则 @OneToMany 端就不能再使用 @JoinColumn 属性了
    - 在 n 的一端指定 @ManyToOne 注解，并使用 @JoinColumn 指定外键名称

4. 映射双向一对一的关联关系

    - 基于外键的一对一关联关系，在双向一对一关联中，需要在关系被维护端（ inverse side ）中的 @OneToOne 注解中指定 mappedBy，以指定是这个关联中的被维护端，同时需要在关系维护端（ owner side ）建立外键列指向关系被维护端的主键列（需要指定 unique 属性）
    - 对于双向一对一关联关系，建议先保存不维护关联关系的一端（即没有外键的一端），这样不会多出 UPDATE 语句
    - 默认情况下，若获取维护关联关系的一端，则会通过左外连接获取其关联的对象，但可以通过 @OneToOne 的 fetch 属性来修改加载策略
    - 双向一对一无法使用延迟加载

5. 映射双向多对多的关联关系

    - 在双向多对多关系中，必须指定一个关系维护端（ owner side ），可以通过 @ManyToMany 注解中 mappedBy 属性来标识其为关系维护

        ```Java
        // @JoinTable 来映射中间表
        // joinColumns 映射当前类所在的表在中间表中的外键
        // inverseJoinColumns 映射关联的类所在中间表的外键
        @ManyToMany
        @JoinTable(name = "中间表名称",
                  joinColumns = @JoinColumn(name = "本类的外键",
                                           referencedColumnName = "本类与外键对应的主键"),
                  inverseJoinColumns = { @JoinColumn(name = "对方类的外键", 
                                                    referencedColumnName = "对方类与外键对应的主键") })
        ```

    - 对于关联的集合对象，默认使用懒加载。无论使用维护关系的一端还是不维护关系的一端获取，发送的 SQL 语句数量相同

----

##### 使用二级缓存

1. \<shared\-cache\-mode\> 节点
    - 若 JPA 实现框架支持二级缓存，该节点可以配置在当前的持久化单元中是否启用二级缓存
        - ALL：所有实体类都被缓存
        - NONE：所有的实体类都不被缓存
        - ENABLE_SELECTIVE：标识 @Cacheable(true) 注解的实体类将被缓存
        - DISABLE_SELECTIVE：缓存除标识 @Cacheable(false) 注解以外的所有实体类
        - UPSPECIFIED：默认值，使用 JPA 实现框架的默认值

----

##### JPQL

1. Java Persistence Query Language 简称 JPQL，是一种和 SQL 非常类似的中间性和对象化的查询语言，它最终会被编译成针对不同底层数据库的 SQL 查询，从而屏蔽不同数据库的差异

2. JPQL 语言的语句可以是 SELECT 语句、UPDATE 语句或 DELETE 语句，它们都通过 javax.persistence.Query 接口封装执行

3. javax.persistence.Query 接口封装了执行数据库查询的相关方法，调用 EntityManager 的 createQuery()、createNamedQuery()、createNativeQuery() 方法即可获取查询对象，进而可以调用 javax.persistence.Query 接口的相关方法来执行查询操作

4. javax.persistence.Query 接口的主要方法

    - int executeUpdate()：用于执行 UPDATE 语句或 DELETE 语句
    - List getResultList()：用于执行 SELECT 语句并返回结果集实体集合
    - Object getSingleResult()：用于执行只返回单个结果实体的 SELECT 语句
    - Query setFirstResult(int startPosition)：用于设置从哪个实体记录开始返回查询结果
    - Query setMaxResults(int maxResult)：用于设置返回结果实体的最大值，可以 setFirstResult() 方法结合使用实现分页查询
    - Query setFlushMode(FlushModeType flushMode)：设置查询对象的 Flush 模式，参数同 EntityManager 的 setFlushMode() 方法
    - setHint(String hintName, Object value)：设置与查询对象相关的特定供应商参数或提示信息，参数名及其取值需要参考特定 JPA 实现框架提供商的文档。如果第二个参数无效将抛出 IllegalArgumentException
    - setParameter(int position, Object value)：为查询语句的指定位置参数赋值，position 指定参数序号（从 1 开始），value 为赋给参数的值
    - setParameter(int position, Date value, TemporalType type)：为查询语句的指定位置参数赋 Date 值，position 指定参数序号，value 为赋给参数的值，type 取 TemporalType 的枚举常量，包括 DATE、TIME、TIMESTAMP 三种，用于将 Java 的 Date 类型的值临时转换为数据库支持的日期时间类型（java.sql.Date、java.sql.Time、java.sql.Timestamp）
    - setParameter(int position, Calendar value, TemporalType type)：为查询语句的指定位置参数赋 Calendar 值，position 指定参数序号，value 为赋给参数的值，type 取值同上
    - setParameter(String name, Object value)：为查询语句的指定名称参数赋值
    - setParameter(String name, Date value, TemporalType type)：为查询语句的指定名称参数赋 Date 值，用法同上
    - setParameter(String name, Calendar value, TemporalType type)：为查询语句的指定名称参数赋 Calendar 值，用法同上。该方法调用时如果参数位置或参数名不对，或者所赋参数值类型不匹配，则抛出 IllegalArgumentException

5. SELECT\-FROM 语句

    - SELECT 用来指定查询返回的结果实体或实体的某些属性
    - FROM 子句声明查询源实体类，并指定标识符变量（相当于 SQL 的别名）
    - 如果不希望返回重复实体，可使用 DISTINCT 关键字修饰

6. JPQL 支持包含参数的查询

    - 若占位符使用 "?index"（ index = { 1, 2, 3, ... }），则执行查询前必须使用 javax.persistence.Query.setParameter(int position, Object value) 方法给参数赋值
    - 若占位符使用 ":name"（ name 为自定义参数名 ），则执行查询前必须使用 javax.persistence.Query.setParameter(String name, Object value) 方法给参数赋值

7. WHERE 条件表达式中可用的运算符基本上与 SQL 一致

8. 若只须查询实体的部分属性而不需要返回整个实体，则执行 getResultList() 方法返回的不再是实体集合，而是一个 Object[] 类型的集合。也可以在实体类中创建对应的构造器，然后在 JPQL 语句中利用对应的构造器返回实体类的对象（ `SELECT new Object(field1, field2) FROM Object o WHERE condition` ）

9. 使用 Hibernate 的查询缓存

    ```Java
    String jpql = "SELECT c FROM Customer c";
    Query query = entityManager.createQuery(jpql);
    query.setHint(QueryHints.CACHEABLE, true);
    ```

10. ORDER BY 子句用于对查询结果集进行排序

11. GROUP BY 子句用于对查询结果分组统计，通常需要使用聚合函数。没有 GROUP BY 子句的查询是基于整个实体类的，使用聚合函数将返回单个结果值，可使用 getSingleResult() 方法获取查询结果

12. HAVING 子句用于对 GROUP BY 分组设置约束条件

13. 支持关联查询和子查询。在 JPQL 中很多时候都是通过在实体类中配置实体关联的类属性来实现隐含的关联查询（ `SELECT o FROM Orders o WHERE o.address.streetNumber = 2000` ）

14. JPQL 函数

    - 字符串处理函数
        - concat(String s1, String s2)：字符串合并/连接函数
        - substring(String s, int start, int length)：取子串函数
        - trim(\[leading\|trailing|both,\] \[char c,\] String s)：去掉字符串的首/尾/指定的字符或空格
        - lower(String s)：将字符串转换成小写
        - upper(String s)：将字符串转换成大写
        - length(String s)：获取字符串长度
        - locate(String s1, String s2\[, int start\])：从第一个字符串中查找第二个字符串出现的位置，若未找到则返回 0
    - 算术函数
        - abs
        - mod
        - sqrt
        - size（用于求集合的元素个数）
    - 日期函数
        - current_date
        - current_time
        - current_timestamp

----

##### 整合 Spring

1. 三种整合方式

    - LocalEntityManagerFactoryBean：适用于那些仅使用 JPA 进行数据访问的项目，该 FactoryBean 将根据 JPA Persistence Provider 自动检测配置文件进行工作，一般从 META-INF/persistence.xml 文件中读取配置信息。这种方式最简单，但是不能设置 Spring 中定义的 DataSource，且不支持 Spring 管理的全局事务
    - 从 JNDI 获取：用于从 JavaEE 服务器获取制定的 EntityManagerFactory，这种方式在进行 Spring 事务管理一般要使用 JTA 事务管理
    - LocalContainerEntityManagerFactoryBean：适用于所有环境的 FactoryBean，能全面控制 EntityManagerFactory 配置，如指定 Spring 定义的 DataSource 等

2. Spring 配置文件中需要配置的

    ```XML
    <!-- 配置 EntityManagerFactory -->
    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource"></property>
        
        <!-- 配置 JPA 提供商的适配器. 可以通过内部 bean 的方式来配置 -->
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"></bean>
        </property>
        
        <!-- 配置实体类所在的包 -->
        <property name="packagesToScan" value="beiran.bean"></property>
        
        <!-- 配置 JPA 的基本属性. 例如 JPA 实现框架的属性 -->
        <property name="jpaProperties">
            <props>
                <prop key="hibernate.show_sql">true</prop>
                <prop key="hibernate.format_sql">true</prop>
                <prop key="hibernate.hbm2ddl.auto">update</prop>
            </props>
        </property>
    </bean>
    
    <!-- 配置 JPA 使用的事务管理器 -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"></property>
    </bean>
    
    <!-- 配置支持基于注解是事务配置 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>
    ```

3. 在 DAO 中使用 @PersistenceContext 注解来注入 EntityManager 实例