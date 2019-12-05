---
title: Hibernate
date: 2019-12-03 15:33:42
tags: 复习
categories: Hibernate
comments: false
---

##### ORM 的思想

1. 将关系数据库中记录映射为对象，以对象的形式展现，可以把对数据库的操作转化为对对象的操作<!-- more -->

----

##### Hibernate 开发步骤

1. 创建 Hibernate 配置文件
2. 创建持久化类
3. 创建对象-关系映射文件
4. 通过 Hibernate API 编写访问数据库的代码

----

##### HibernateTool For Eclipse 安装

1. [Install Software 链接](http://download.jboss.org/jbosstools/oxygen/stable/updates/)
2. [JBoss 官网](https://tools.jboss.org/downloads/)
3. [Hibernate Tools 官网](http://hibernate.org/tools/)

----

##### Configuration 类

1. Configuration 类负责管理 Hibernate 的配置信息，包括以下内容：

    - Hibernate 运行的底层信息：数据库的 URL、用户名、密码、JDBC 驱动类、数据库 Dialect（方言）、数据库连接池等（对应 `hibernate.cfg.xml` 文件）
    - 持久化类与数据表的映射关系（ `*.hbm.xml` 文件）

2. 创建 Configuration 的两种方式

    - 属性文件（ `hibernate.properties` ）

        ```Java
        Configuration configuration = new Configuration();
        ```

    - XML 文件（ `hibernate.cfg.xml` ）

        ```Java
        Configuration configuration  = new Configuration().configure();
        ```

    - 注：Configuration 的 `configure()` 方法还支持带参数的访问

        ```Java
        File file = new File("example.xml");
        Configuration configuration  = new Configuration().configure(file);
        ```

----

##### SessionFactory 接口

1. 针对单个数据库映射关系经过编译后的内存镜像，是线程安全的

2. SessionFactory 对象一旦构造完毕，即被赋予特定的配置信息

3. SessionFactory 是生成 Session 的工厂

4. 构造 SessionFactory 很消耗资源，一般情况下一个应用中只初始化一个 SessionFactory 对象

5. Hibernate4 新增了一个 ServiceRegistry 接口，所有基于 Hibernate 的配置或者服务都必须统一向这个 ServiceRegistry 注册后才能生效

6. Hibernate4 中创建 SessionFactory 的步骤

    ```Java
    // 创建Configuration对象
    Configuration configuration = new Configuration().configure();
    
    // 获取ServiceRegistry对象
    ServiceRegistry serviceRegistry = new ServiceRegistryBuilder().applySettings(configuration.getProperties()).buildServiceRegistry();
    
    // 创建SessionFactory对象
    SessionFactory sessionFactory = configuration.buildSessionFactory(serviceRegistry);
    ```

----

##### Session 接口

1. Session 是应用程序与数据库之间交互操作的一个**单线程对象**，是 Hibernate 运作的中心，所有持久化对象必须在 Session 的管理下才可以进行持久化操作。此对象的生命周期很短。Session 对象有一个一级缓存，显式执行 `flush()` 之前，所有的持久层操作的数据都缓存在 Session 对象处。相当于 JDBC 中的 Connection，实际上是对 Connection 的封装
2. Session 接口是 Hibernate 向应用程序提供的操纵数据库的最主要的接口，他提供了基本的**保存，更新，删除和加载 Java 对象**的方法。Session 具有一个一级缓存，位于缓存中的对象称为持久化对象，他和数据库中的相关记录对应。Session 能够在某些时间点，按照缓存中对象的变化来执行相关的 SQL 语句，来同步更新数据库，这一过程被称为刷新缓存（ flush ）。站在持久化的角度，Hibernate 把对象分为四种状态，分别是**持久化状态、临时状态、游离状态、删除状态**等。Session 的特定方法能够将对象从一个状态转换到另外一个状态

----

##### hibernate.cfg.xml 配置文件解释

1. hbm2ddl.auto
    - 有四个取值：
        - create：会根据 `*.hbm.xml` 文件来生成数据表，但是每次运行都会删除上一次的表，重新生成表，哪怕第二次没有任何改变
        - create-drop：会根据 `*.hbm.xml` 文件来生成数据表，但是 SessionFactory 一关闭，生成的数据表就自动删除
        - update（最常用）：会根据 `*.hbm.xml` 文件来生成数据表，但是若 `*.hbm.xml` 文件中对应的数据表与数据库中对应的数据表结构不同才会更新数据表，不会删除已有的行和列
        - validate：会和数据库中的表进行比较，若 `*.hbm.xml` 文件中的列在数据表中不存在，则抛出异常，但不会做任何修改
2. format_sql
    - 是否将 SQL 转化为格式良好的 SQL （即格式化 SQL ），取值 true/false
3. show_sql
    - 是否在控制台将 SQL 语句打印出来，取值 true/false
4. dialect
    - 使用怎样的数据库方言，取值见 `hibernate\project\etc\hibernate.properties` 文件中
5. connection.autocommit
    - 是否自动提交事务，取值 true/false

----

##### flush 缓存

1. flush：Session 按照缓存中对象的属性变化来同步更新数据库

2. 默认情况下 Session 在以下时间点刷新缓存：

    - 显式调用 Session 的 `flush()` 方法
    - 当应用程序调用 Transaction 的 `commit()` 方法时，该方法会先刷新缓存，然后再向数据库提交事务
    - 当应用程序执行一些查询（ HQL，Criteria ）操作时，如果缓存中持久化对象的属性已经发生了变化，会先刷新缓存，以保证查询结果能够反映持久化对象的最新状态

3. flush 缓存的例外情况：如果对象使用 native 生成器生成 OID，那么当调用 Session 的 `save()` 方法保存对象时，会立即执行向该数据库插入该实体的 INSERT 语句

4. `commit()` 方法与 `flush()` 方法的区别：`flush()` 方法执行一系列 SQL 语句，但不提交事务；`commit()` 方法先调用 `flush()` 方法，然后提交事务，提交事务意味着对数据库的操作将永久保存下来

5. 若希望改变 `flush()` 方法的默认刷新时间点，可以通过 Session 的 `setFlushMode()` 方法显式设定刷新的时间点

    | 模式                     | 各种查询方法 | Transaction 的 commit() 方法 | Session 的 flush() 方法 |
    | ------------------------ | ------------ | ---------------------------- | ----------------------- |
    | FlushMode.AUTO(默认模式) | 刷新缓存     | 刷新缓存                     | 刷新缓存                |
    | FlushMode.COMMIT         | 不刷新缓存   | 刷新缓存                     | 刷新缓存                |
    | FlushMode.NEVER          | 不刷新缓存   | 不刷新缓存                   | 刷新缓存                |

----

##### 对象的三种状态

1. Transient 瞬时状态
    - 在数据库中没数据，跟 Session 不相关，没存过
    - 使用 new 操作符初始化的对象不是立刻就持久的。他们的状态是瞬时的，也就是说他们没有任何跟数据库表相关联的行为，只要应用不再去引用这些对象（不再被任何其他对象所引用），他们的状态就会丢失，并由垃圾回收机制回收
2. Persistenet 持久化状态
    - 在数据库中有记录，Session 中也有记录，自动更新
    - 持久化实例时任何具有数据库标识的实例，他由持久化管理器 Session 统一管理，持久化实例是在事务中进行操作的，他们的状态是在事务结束时同数据库进行同步，当事务提交时，通过执行 SQL 的 INSERT、UPDATE、DELETE 语句把内存中的状态同步到数据库中
3. Detached 游离态
    - 在数据库中有记录，但是在 Session 中没有，需要手动同步
    - Session 关闭后，持久化对象就变成了游离对象，游离表示这个对象不能再和数据库保持同步，他们不再受 Hibernate 管理
4. 三种状态的区分
    - Transient 对象：随时可能会被垃圾回收器回收（在数据库中没有与之对应的记录，因为是 new 初始化），没有纳入 Session 管理，而执行 `save()` 方法后，就变为 Persistent 对象。**内存中一个对象，没有 ID，缓存中也没有**
    - Persistent 对象：在数据库中存在对应记录，纳入 Session 管理，在清理缓存（脏数据检查）时，会和数据库同步。**内存中有，缓存中也有，数据库中有（ID）**
    - Detached 对象：可能被垃圾回收器回收掉（数据库中存在对应的记录，只是没有任何对象引用他是指 Session 引用），没有纳入 Session 管理。**内存中有，缓存中没有，数据库中有（ID）**

----

##### Hibernate 主键生成策略

1. increment：适用于代理主键，由 Hibernate 自动以递增方式生成
2. identity：适用于代理主键，由底层数据库生成标识符
3. sequence：适用于代理主键，Hibernate 根据底层数据库的序列生成标识符，这要求底层数据库支持序列
4. hilo：适用于代理主键，Hibernate 使用 high/low 算法生成标识符
5. seqhilo：适用于代理主键，使用一个 high/low 算法来高效的生成 long，short 或者 int 类型的标识符
6. native：适用于代理主键，根据底层数据库自动生成标识符，自动选择 identity，sequence 或者 hilo
7. uuid.hex：适用于代理主键，Hibernate 采用 128 位的 UUID 算法生成标识符
8. uuid.string：适用于代理主键，UUID 被编码成一个 16 位字符长的字符串
9. assigned：适用于自然主键，由 Java 应用程序负责生成标识符
10. foreign：适用于代理主键，使用另外一个相关联的对象的标识符（外键）
11. 注：常用 identity 和 native 方式

----

##### refresh() 方法与 clear() 方法

1. `refresh()` 方法是强制将 Session 中的对象与数据表中的保持一致（发送一条 SELECT 语句）
2. `clear()` 方法会将 Session 中缓存的所有的对象清空

----

##### Hibernate 的 get() 方法和 load() 方法的区别

1. 执行 `get()` 方法，会立即加载对象（即立即查询）
2. 执行 `load()` 方法，若不使用该对象，则不会立即执行查询，而是先返回一个代理对象（即延迟加载）
3. 若数据表中没有对应的记录，Session 没关，`get()` 方法返回 Null，`load()` 方法抛出异常（需要使用对象时，不使用对象不抛异常）
4. `load()` 方法可能会抛出懒加载异常（`LazyInitializationException`），在需要初始化代理对象之前已经关闭了 Session 对象时会出现这种情况

----

##### Hibernate 的 update() 方法

1. 更新一个游离对象时需要显式调用 `update()` 方法（会将一个游离对象变为持久化对象），无论游离对象和数据表中是否一致都会发送 UPDATE 语句
2. 在 `*.hbm.xml` 文件中 class 节点设置 `select-before-update` 属性为 true（通常不需要设置，除非有特殊的触发器）
3. 若数据表中没有对应的记录，但也调用了 `update()` 方法，则会抛出异常

----

##### Hibernate 的 saveOrUpdate() 方法

1. 判定为临时对象的标准
    - Java 对象的 OID 为 Null
    - `*.hbm.xml` 文件中为 id 设置了 `unsaved-value` 属性，并且 Java 对象的 OID 取值和这个 `unsaved-value` 属性相匹配

----

##### Hibernate 的 delete() 方法

1. 计划执行一条 DELETE 语句
2. 将对象从 Session 缓存中删除，该对象进入删除状态
3. `hibernate.cfg.xml` 文件中有一个 `hibernate.user_identifier_rollback` 属性，默认为 false，可以将其置为 true 从而使 `delete()` 方法在删除对象时将对象的 OID 置为 Null，使其变为临时对象

----

##### Hibernate 的 evict() 方法

1. 从 Session 缓存中将指定的持久化对象移除

----

##### Hibernate 配置文件

1. 每个 Hibernate 配置文件都对应一个 Configuration 对象
2. Hibernate 配置文件有两种选项
    - `hibernate.properties`
    - `hibernate.cfg.xml`

----

##### JDBC 连接属性

1. connection.url：数据库 URL
2. connection.name：数据库用户名
3. connection.password：数据库用户密码
4. connection.driver_class：数据库 JDBC 驱动
5. dialect：配置数据库的方言，根据底层的数据库不同产生不同的 SQL 语句，Hibernate 会针对数据库的特性在访问时进行优化

----

##### C3P0 数据库连接池属性

1. hibernate.c3p0.max_size：数据库连接池的最大连接数
2. hibernate.c3p0.min_size：数据库连接池的最小连接数
3. hibernate.c3p0.timeout：数据库连接池中连接对象在多长时间没有使用过后，就应该被销毁
4. hibernate.c3p0.max_statements：缓存 Statement 对象的数量
5. hibernate.c3p0.idle_text_period：表示连接池检测线程多长时间检测一次池内的所有连接对象是否超时，连接池本身不会把自己从连接池中移除，而是专门有一个线程按照一定的时间间隔来做这个事，这个线程通过比较连接对象最后一次使用时间和当前时间的时间差来和 timeout 做对比，进而决定是否销毁这个连接对象
6. hibernate.c3p0.acquire_increment：当数据库连接池中的连接耗尽时，同一时刻获取多少个数据库连接

----

##### 在 Hibernate 中使用 C3P0 数据源

1. 导入 jar 包（位于 `hibernate\lib\optional\c3p0` 路径下）
2. 加入配置

----

##### hibernate-mapping

1. hibernate-mapping 是 Hibernate 映射文件的根元素，他有以下属性：
    - schema：指定所映射的数据 schema 的名称，若指定该属性，则表明会自动添加该 schema 前缀
    - catalog：指定所映射的数据库的 catalog 名称
    - default-cascade（默认为 none）：设置 Hibernate 默认的级联风格，若配置 Java 属性，集合映射时没有指定 cascade 属性，则 Hibernate 将采用此处指定的级联风格
    - default-access（默认为 property）：指定 Hibernate 的默认的属性访问策略，默认值为 property，即使用 getter/setter 方法来访问属性；若指定 access，则 Hibernate 会忽略 getter/setter 方法，而通过反射访问成员变量
    - default-lazy（默认为 true）：设置 Hibernate 默认的延迟加载策略，该属性的默认值为 true，即启用延迟加载策略，若配置 Java 属性映射，集合映射时没有指定 lazy 属性，则 Hibernate 将采用此处指定的延迟加载策略
    - auto-import（默认为 true）：指定是否可以在查询语言中使用非全限定的类型（仅限于本映射文件中的类）
    - package（可选）：指定一个包前缀，如果在映射文档中没有指定一个限定的类名，则使用这个作为包名

----

##### class

1. class 元素用于指定类和表的映射，他有以下属性：
    - name：指定该持久化类映射的持久化类的类名
    - table：指定该持久化类映射的表名，Hibernate 默认以持久化类的类名作为表名
    - dynamic-insert：若设置为 true，则表示当保存一个对象时，会动态生成 INSERT 语句，INSERT 语句中仅包含所有取值不为 Null 的字段，默认为 false
    - dynamic-update：若设置为 true，则表示当更新一个对象时，会动态生成 UPDATE 语句，UPDATE 语句中仅包含所有取值需要更新的字段，默认为 false
    - select-before-update：设置 Hibernate 在更新某个持久化对象之前是否需要先执行一次查询，默认值为 false
    - batch-size：指定根据 OID 来抓取实例时每批抓取的实例数
    - lazy：指定是否使用延迟加载
    - mutable：若设置为 true，则表示所有的 \<property\> 元素的 update 属性为 false，表示整个实例都不能被更新，默认为 true
    - discriminator-value：指定区分不同子类的值，当使用 \<subclass\> 元素来定义持久化类的继承关系时需要使用该属性

----

##### 映射对象标识符

1. Hibernate 使用对象标识符（ OID ）来建立内存中的对象和数据表中记录的对应关系，对象的 OID 和数据表中的主键对应，Hibernate 通过标识符生成器来为主键赋值
2. Hibernate 推荐在数据表中使用代理主键（即不具备业务含义的字段），代理主键通常为整数类型，因为能节省空间
3. Hibernate 提供了标识符生成器接口：IdentifierGenerator，并提供了各种内置实现

----

##### id

1. 设定持久化类的 OID 和表的主键映射，有以下属性：
    - name：标识持久化类 OID 的属性名
    - column：设置标识属性所映射的数据表的列名（主键字段的名字）
    - unsaved-value：若设定了该属性，Hibernate 会通过比较持久化类的 OID 值和该属性值来区分当前持久化类对象是否为临时对象
    - type：指定 Hibernate 映射类型，Hibernate 映射类型是 Java 类型与 SQL 类型的桥梁，如果没有为某个属性显式设定映射类型，HIbernate 会运用反射机制先识别出持久化类的特定属性的 Java 类型，然后自动使用与之对应的默认的 Hibernate 映射类型
    - Java 的基本数据类型和包装类型对应相同的 Hibernate 映射类型，基本数据类型无法表达 Null，所以对于持久化类的 OID 推荐使用包装类型

----

##### generator

1. 设定持久化类的标识符生成器，有一个属性：
    - class：指定使用的标识符生成器全限定类名或其缩写名（一般为缩写名）

----

##### 主键生成策略

1. increment

    - increment 生成器由 Hibernate 以递增的方式为代理主键赋值
    - Hibernate 会先读取数据表中主键的最大值，然后在接下来向表中插入新纪录的时候，就在读取到的这个最大值的基础上递增，增量为 1
    - 适用范围
        - 由于 increment 生成标识符机制不依赖于底层数据库系统，因此他适用于所有的数据库系统
        - 适用于只有单个 Hibernate 应用进程访问同一个数据库的场合，在集群环境下不推荐使用他（在多线程环境下可能会出现脏数据）
        - OID 必须为 long、int 或者 short 类型，如果把 OID 定义为 byte 类型，在运行时会抛出异常

2. identity

    - identity 标识符生成器由底层数据库来负责生成标识符，他要求底层数据库把主键定义为自动增长字段类型
    - 适用范围
        - 由于 identity 生成标识符的机制依赖于底层数据库系统，因此，要求底层数据库系统必须支持自动增长字段类型，支持自动增长字段类型的数据库包括：DB2、MySQL、MSSQL Server、Sybase 等
        - OID 必须为 long、int 或者 short 类型，如果把 OID 定义为 byte 类型，在运行时会抛出异常

3. sequence

    - sequence 标识符生成器利用底层数据库提供的序列来生成标识符

    - 要在 \<generator\> 标签中注明使用的序列是谁

        ```XML
        <id name="id">
            <generator class="sequence">
                <!-- 指定的序列名 -->
                <param name="sequence">test_seq</param>
            </generator>
        </id>
        ```

    - Hibernate 在持久化一个对象时，先从底层数据库中获得一个唯一的标识号，再把他作为主键值

    - 适用范围

        - 由于 sequence 生成标识符的机制依赖于底层数据库系统的序列，因此，要求底层数据库必须支持序列，支持序列的数据库包括：DB2、Oracle 等
        - OID 必须为 long、int 或者 short 类型，如果把 OID 定义为 byte 类型，在运行时会抛出异常

4. hilo

    - hilo 标识符生成器是由 Hibernate 按照一种 high/low 算法来生成标识符，他从数据库的特定表的字段中获取 high 的值

        ```XML
        <id name="id">
            <generator class="hilo">
                <param name="table">HI_TABLE</param>
                <param name="column">NEXT_VALUE</param>
                <param name="max_lo">10</param>
            </generator>
        </id>
        ```

    - Hibernate 在持久化一个对象时，由 HIbernate 负责生成主键值，hilo 标识符生成器在生成标识符时，需要读取并修改 HI_TABLE 表中的 NEXT_VALUE 值

    - 适用范围

        - 由于 hilo 生成标识符机制不依赖于底层数据库系统，因此他适合于所有的数据库系统
        - OID 必须为 long、int 或者 short 类型，如果把 OID 定义为 byte 类型，在运行时会抛出异常

5. native

    - native 标识符生成器依据底层数据库对自动生成标识符的支持能力，来选择使用 identity、sequence 或者 hilo 标识符生成器
    - 适用范围
        - 由于 native 能根据底层数据库系统的类型，自动选择合适的标识符生成器，因此很适合于跨数据库平台开发
        - OID 必须为 long、int 或者 short 类型，如果把 OID 定义为 byte 类型，在运行时会抛出异常

----

##### property

1. property 元素用于指定类的属性和表的字段映射，他有以下属性：

    - name：指定该持久化类的属性的名字

    - column：指定与类的属性映射的表的字段名，如果没有设置该属性，Hibernate 则会直接使用类的属性名作为字段名

    - type：指定 Hibernate 映射类型，Hibernate 映射类型是 Java 类型与 SQL 类型的桥梁，如果没有为某个属性显式设定映射类型，Hibernate 会运用反射机制先识别出持久化类的特定属性的 Java 类型，然后自动使用与之对应的默认的 Hibernate 映射类型

    - not-null：若该属性值为 true，则表明不允许为 Null，默认为 false

    - access：指定 Hibernate 默认的属性访问策略，默认值为 property，即使用 getter/setter 方法来访问属性，若指定为 field，则 Hibernate 会忽略 getter/setter 方法，而通过反射机制来访问成员变量

    - unique：设置是否为该属性所映射的数据列添加 unique 约束

    - update：设置该列值是否可以被修改，默认为 true

    - index：指定一个字符串的索引名称，当系统需要 Hibernate 自动建表时，用于为该属性所映射的数据列创建索引，从而加快该数据列的查询

    - length：指定该属性所映射数据列的字段的长度

    - scale：指定该属性所映射数据列的小数位数，对 double、float、decimal 等类型的数据列有效

    - formula：设置一个 SQL 表达式，Hibernate 将根据他来计算出派生属性的值

        - 派生属性：并不是持久化类的的所有属性都直接和表的字段匹配，持久化类的有些属性的值必须在运行时通过计算才能得出来，这类属性称为派生属性

        - formula = “( SQL 表达式)” 的英文括号不能少

        - 如果需要在 formula 属性中使用参数，就直接使用 WHERE cur.id = id 形式，其中 id 就是参数，和当前持久化对象的 id 属性对应的列的 id 值将作为参数传入

            ```XML
            <!-- 字段名为 test, 表名为 News, test 字段由 title 字段和 author 字段拼接 -->
            <property name="test" formula="(SELECT concat(title, ': ' author) FROM News news WHERE news.id = id)"></property>
            ```

----

##### Java 时间和日期类型的 Hibernate 映射

1. 持久化类中将成员变量类型设置为 java.util.Date 类型
2. `*.hbm.xml` 文件中在 property 中设置 type 为 Hibernate 映射类型即可

| Hibernate 映射类型 | Java 类型                         | 标准 SQL 类型 | 描述           |
| ------------------ | --------------------------------- | ------------- | -------------- |
| date               | java.util.Date/java.sql.Date      | DATE          | yyyy-MM-dd     |
| time               | java.util.Date/java.sql.Time      | TIME          | hh:mm:ss       |
| timestamp          | java.util.Date/java.sql.Timestamp | TIMESTAMP     | yyyymmddhhmmss |
| calendar           | java.util.Calendar                | TIMESTAMP     | yyyymmddhhmmss |
| calendar_date      | java.util.Calendar                | DATE          | yyyy-MM-dd     |

----

##### Java 大对象类型的 Hibernate 映射

1. 长字符串可直接使用 java.lang.String 类型表示

2. 字节数组 byte[] 可用于存放图片或文件的二进制数据

3. JDBC API 中还提供了 java.sql.Clob 和 java.sql.Blob 类型，他们分别表示标准 SQL 中的 CLOB 和 BLOB 类型

4. MySQL 不支持标准 SQL 的 CLOB 类型，在 MySQL 中，用 TEXT、MEDIUMTEXT 和 LONGTEXT 类型来表示长度超过 255 的长文本数据

5. 在持久化类中，二进制大对象可以声明为 byte[] 类型数组或者 java.sql.Blob 类型，字符串可以声明为 java.lang.String 或者 java.sql.Clob 类型

6. 如果想精确映射 SQL 类型，可以在 \<column\> 标签中使用 sql-type 属性，sql-type 属性的取值为当前数据库所支持的 SQL 类型

7. 保存一个二进制大对象需要一个 InputStream 来读取图片或文件

    ```Java
    InputStream inputStream = new FileInputStream("图片/文件路径");
    Blob image = Hibernate.getLobCreator(session).createBlob(inputStream, inputStream.available());
    ```

----

##### 映射组成关系

1. Hibernate 把持久化类的属性分为两种

    - value type（值类型）：没有 OID，不能被单独持久化，生命周期依赖于所属的持久化类的对象的生命周期
    - entity type（实体类型）：有 OID，可以被单独持久化，有独立的生命周期

2. Hibernate 使用 \<component\> 元素来映射组成关系

    ```XML
    <!-- 以 Worker 和 Pay 为例， 工人有工资，工资为工人的一部分 -->
    <!-- 在这里表示 Pay 是 Worker 类的一个组成部分 -->
    <!-- 在 Hibernate 中称为组件 -->
    <component name="pay" class="Pay">
        <!-- parent 元素指定组件属性所属的整体类 -->
        <!-- 使用 parent 元素的前提是在 Pay 中有一个属性为 Worker 的引用 -->
        <parent name="Worker" />
        <property name="payId" column="pay_id" type="java.lang.Integer"></property>
        <property name="payName" column="pay_name" type="java.lang.String"></property>
    </component>
    ```

----

##### 四个地方会出现懒加载

1. Session.load() 方法
    - 如果在 Session 关闭之后再查询此对象会出现懒加载异常，可以在 Session 关闭之前初始化一下查询出来的代理对象：`Hibernate.initialize(object);` 来解决这个问题
2. one-to-one 关联关系
3. many-to-one 关联关系
    - n 对 1 时无论哪一端都是默认懒加载的查询，如果不需要懒加载，则要修改映射文件
4. one-to-many 关联关系

----

##### 单向 n 对 1 关联关系

1. 使用 many-to-one 来映射多对一的关联关系，以添加外键的形式来描述多对一的关系

    ```XML
    <!-- n 端映射文件中需要加入 1 端的配置 -->
    <many-to-one name="customer" class="Customer" column="customer_id"></many-to-one>
    ```

    - name：n 端关联 1 端的属性的名字
    - class：1 端的属性对应的类名
    - column：1 端在 n 端的外键的名字

---

##### 双向 n 对 1 关联关系

1. 在单向 n 对 1 的基础上在 1 端的类中加上 n 端的集合作为一个属性

    ```Java
    // 以 Customer 和 Order 为例
    // Customer 为 1 端类
    class Customer {
        // Order 为 n 端类
        private Set<Order> orders = new HashSet<>();
    }
    ```

2. 在 1 端的映射文件中加入上面那个集合的映射

    ```XML
    <!-- name 为集合名, table 为 n 端的表名 -->
    <set name="orders" table="Orders">
        <!-- key 为 外键 -->
        <key column="customer_id"></key>
        <!-- 映射类型 -->
        <one-to-many class="Order" />
    </set>
    ```

3. \<set\> 元素的 inverse 属性

    - 在 Hibernate 中通过 inverse 属性来决定由双向关联的哪一端来维护表和表之间的关系，`inverse = false` 的为主动方，`inverse = true` 的为被动方，由主动方负责维护关联关系
    - 在没有设置 inverse 属性的情况下（即默认情况），两边都维护关联关系
    - 在 1 对 n 关系中，将 n 方设为主动方有助于改善性能

4. \<set\> 元素的 cascade 属性

    - 在对象-关系映射文件中，用于映射持久化类之间关联关系的元素，\<set\> 、\<many-to-one\> 、\<one-to-one\> 等标签都有一个 cascade 属性，他用于指定如何操纵与当前对象关联的其他对象

        | cascade 属性值    | 描述                                                         |
        | ----------------- | ------------------------------------------------------------ |
        | none              | 当 Session 操纵当前对象时，忽略其他关联的对象（cascade 属性默认值） |
        | save-update       | 当通过 Session 的 save() 、update() 、saveOrUpdate() 等方法来保存或更新对象时，级联保存所有关联的新建的临时对象，并且级联更新所有的游离对象 |
        | persist           | 当通过 Session 的 persist() 方法来保存当前对象时，会级联保存所有关联的新建的临时对象 |
        | delete            | 当通过 Session 的 delete() 方法删除当前对象时，会级联删除所有关联的对象 |
        | lock              | 当通过 Session 的 lock() 方法把当前游离对象添加到 Session 缓存中时，会把所有关联的游离对象也添加到 Session 缓存中 |
        | replicate         | 当通过 Session 的 replicate() 方法复制当前对象时，会级联复制所有关联的对象 |
        | evict             | 当通过 Session 的 evict() 方法从 Session 缓存中清除当前对象时，会级联清除所有关联的对象 |
        | refresh           | 当通过 Session 的 refresh() 方法刷新当前对象时，会级联刷新所有关联的对象，所谓刷新是指读取数据库中相应数据，然后根据数据库中的最新数据去同步更新 Session 缓存中的相应对象 |
        | all               | 包含 save-update、persist、merge、delete、lock、replicate、evict、refresh 等行为 |
        | delete-orphan     | 删除所有和当前对象解除关联关系的对象                         |
        | merge             | 当通过 Session 的 merge() 方法来保存当前对象时，会级联融合所有关联的游离对象 |
        | all-delete-orphan | 包含 all 和 delete-orphan 的行为                             |

5. \<set\> 元素的 order-by 属性

    - \<set\> 元素有一个 order-by 属性，如果设置了该属性，当 Hibernate 通过 SELECT 语句到数据库中检索集合对象时，利用 order-by 子句进行排序

    - order-by 属性还可以加入 SQL 函数

        ```XML
        <!-- order-by 中放入的是列名 -->
        <set name="orders" inverse="true" cascade="save-update" order-by="ORDER_DATE DESC">
            <key column="customer_id"></key>
            <one-to-many class="Order"></one-to-many>
        </set>
        ```

----

##### 双向 1 对 1 关联关系

1. 按照外键映射（需要使用 unique 约束标记外键）

    - 对于基于外键映射的 1 对 1 关联，其外键可以存放至任意一边，在需要存放外键的一端，增加 \<many-to-one\> 元素，为 \<many-to-one\> 元素增加 `unique = “true”` 属性来表示为 1 对 1 关联

    - 另一端需要使用 \<one-to-one\> 元素，该元素使用 `property-ref` 属性指定使用被关联实体主键以外的字段作为关联字段（即指定引用的外键），如果不使用这个属性指定，则会默认使用该表的主键来作为外键进行查询

        ```XML
        <one-to-one name="dept" class="Department" property-ref="manager"></one-to-one>
        ```

2. 按照主键映射

    - 一端的主键生成器使用 foreign 策略，表明根据 “对方” 的主键来生成自己的主键，自己并不能独立生成主键，\<param\> 子元素表示指定使用当前持久化类的哪个属性作为 “对方”

        ```XML
        <id name="id" column="ID" type="java.lang.Integer">
            <!-- 使用外键方式生成当前表的主键 -->
            <generator class="foreign">
                <!-- property 属性指定使用当前持久化类的哪一个属性的主键来作为外键 -->
                <param name="property">manager</param>
            </generator>
        </id>
        ```

    - 采用 foreign 主键生成器策略的一端增加 \<one-to-one\> 元素映射关联属性，其 \<one-to-one\> 属性还应增加 `constrained = “true”` 属性，另一端增加 \<one-to-one\> 元素映射关联属性

    - constrained（约束）：指定为当前持久化类对应的数据库表的主键添加一个外键约束，引用被关联的对象（ “对方” ）所对应的数据库表主键

        ```XML
        <one-to-one name="manager" class="Manager" constrained="true"></one-to-one>
        ```

----

##### 单向 n 对 n 关联关系

1. （n 对 n 关联必须使用连接表）需要一张存放有两个或多个持久化类主键的索引表

2. 与 1 对 n 映射类似，必须为 \<set\> 集合元素添加 \<key\> 子元素，指定 categories_items 表中参照 categories 表的外键为 category_id；与 1 对 n 关联映射不同的是，建立 n 对 n 关联时，集合中的元素使用 \<many-to-many\> ，\<many-to-many\> 子元素的 class 属性指定 items 集合中存放的是 Item 对象，column 属性指定 categories_items 表中参照 items 表的外键为 item_id

    ```XML
    <!-- table 属性用来指定中间表 -->
    <set name="items" table="categories_items" cascade="save-update">
        <key column="category_id"></key>
        <!-- 指定 n 对 n 的关联关系 -->
        <!-- column 属性用来指定 Set 集合中的持久化类在中间表的外键列名称 -->
        <many-to-many class="Item" column="item_id"></many-to-many>
    </set>
    ```

    注：categories 表和 items 表分别表示商品的分类和商品两个实体

----

##### 双向 n 对 n 关联关系

1. 双向 n 对 n 关联需要两端都使用集合属性
2. 双向 n 对 n 关联必须使用连接表
3. 集合属性应增加 key 子元素用以映射外键列，集合元素里还应增加 \<many-to-many\> 子元素关联实体类
4. 在双向 n 对 n 关联的两边都需要指定连接表的表名以及外键列的列名，两个集合元素 \<set\> 的 table 元素的值必须指定，而且必须相同。\<set\> 元素的两个子元素：\<key\> 和 \<many-to-many\> 都必须指定 column 属性，其中，\<key\> 和 \<many-to-many\> 分别指定本持久化类和关联类在连接表中的外键列名，因此两边的 \<key\> 与 \<many-to-many\> 的 column 属性交叉相同，也就是说，一边的 \<set\> 元素的 \<key\> 的 column 值为 a，\<many-to-many\> 的 column 为 b；则另一边的 \<set\> 元素的 \<key\> 的 column 值为 b，\<many-to-many\> 的 column 值为 a
5. 对于双向 n 对 n 关联，必须把其中一端的 inverse 设置为 true，否则两端都维护关联关系可能会造成主键冲突 

----

##### 映射继承关系

1. Hibernate 支持三种继承映射策略
    - 使用 subclass 进行映射
        - 将域模型中的每一个实体对象映射到一个独立的表中，也就是说不用在关系数据模型中考虑域模型中的继承关系和多态
    - 使用 joined-subclass 进行映射
        - 对于继承关系中的子类使用同一张表，这就需要在数据库中增加额外的区分子类类型的字段
    - 使用 union-subclass 进行映射
        - 域模型中的每个类映射到一张表，通过关系数据模型中的外键来描述表之间的继承关系。相当于按照域模型的结构来建立数据库中的表，并通过外键来建立表之间的继承关系

----

##### 使用 subclass 元素的继承映射

1. 采用 subclass 的继承映射可以实现对于继承关系中父类和子类使用同一张表

2. 因为父类和子类的实例全部保存在同一张表中，因此需要在该表增加一列，使用该列来区分每行记录到底是哪个类的实例，这一列被称为辨别者列（ discriminator ）

3. 在这种映射策略下，使用 subclass 来映射子类，使用 class 或 subclass 的 `discriminator-value` 属性来指定辨别者列的值

4. 所有子类定义的字段都不能有非空约束。如果为那些字段添加非空约束，那么父类的实例在那些列其实并没有值，这将引起数据库完整性冲突，导致父类的实例无法保存到数据库中

    ```XML
    <!-- 父类的配置文件 -->
    <!-- 鉴别者列 column为该列列名 type为该列的数据类型 -->
    <discriminator column="TYPE" type="java.lang.String"></discriminator>
    ```

5. 由于父类与子类都使用同一张表，所以无论查询父类记录还是查询子类记录，都只需要查询一张表即可

6. 缺点：

    - 多出一列鉴别者列
    - 子类独有的字段无法添加非空约束
    - 数据表的字段会随着继承的层次加深而增多

----

##### 使用 joined-subclass 元素的继承映射

1. 使用 joined-subclass 元素的继承映射可以实现每个子类一张表
2. 使用这种映射策略时，父类实例保存在父类表中，子类实例由父类表和子类表共同存储。因为子类实例也是一个特殊的父类实例，因此也必然包含了父类实例的属性，于是将子类和父类共有的属性保存在父类表中，子类中增加的属性，保存在子类表中
3. 在这种映射策略下，无须使用鉴别者列，但需要为每个子类使用 key 元素映射共有主键
4. 子类增加的属性可以添加非空约束（因为父类和子类并不在同一张表中）
5. 由于是每个子类一张表，所以在插入数据时会插入多张表，性能会降低
6. 查询父类记录，使用一个左外连接；查询子类记录，使用一个内连接
7. 优点：
    - 不需要使用鉴别者列
    - 子类独有的字段可以添加非空约束
    - 没有冗余的字段

----

##### 使用 union-subclass 元素的继承映射

1. 使用 union-subclass 元素可以实现将每一个实体对象映射到一个独立的表中
2. 子类增加的属性可以有非空约束，即父类实例的数据保存在父类数据表中，而子类实例的数据保存在子类数据表中
3. 子类实例的数据仅保存在子类表中，而在父类表中没有任何记录
4. 在这种映射策略下，子类表的字段会比父类表的映射字段要多，因为字段表的字段等于父类表的字段与子类增加属性的综合
5. 在这种映射策略下，既不需要使用鉴别者列也不需要使用 key 元素来映射共有主键
6. 使用 union-subclass 映射策略是不可以使用 identity 的主键生成策略的，因为同一类继承层次中所有实体类都需要使用同一个主键种子，即多个持久化实体对应的记录的主键应该是连续的，受此影响，也不该使用 native 生成策略，因为 native 策略会根据数据库来选择使用 identity 或者 sequence 来生成主键

----

##### Hibernate 的检索策略

1. 原则
    - 不浪费内存（懒加载）
    - 更高的查询效率（发送尽可能少的 SQL 语句）
    
2. 类级别的检索策略
    - 类级别可选的检索策略包括**立即检索**和**延迟检索**，默认为延迟检索
        - 立即检索：立即加载检索方法指定的对象
        - 延迟检索：延迟加载检索方法指定的对象
    - 类级别的检索策略可通过 \<class\> 元素的 lazy 属性进行设置
    - 如果程序加载一个对象的目的是为了访问他的属性，可以采取立即检索；如果程序加载一个持久化对象仅仅是为了获取他的引用，那么可以采用延迟检索
    - 无论 \<class\> 元素的 lazy 属性是 true 还是 false，Session 的 `get()` 方法及 Query 的 `list()` 方法在类级别总是使用立即检索策略
    - 若 \<class\> 元素的 lazy 属性为 true 或取默认值，Session 的 `load()` 方法不会执行查询数据表的 SELECT 语句，仅返回代理类对象的实例，该代理类实例有如下特征：
        - 由 Hibernate 在运行时采用 CGLIB 工具动态生成
        - Hibernate 创建代理类实例时，仅初始化其 OID 属性
        - 在应用程序第一次访问代理类实例的非 OID 属性时，Hibernate 会初始化代理类实例
    
3. 1 对 1 和 n 对 n 的检索策略
    - 在映射文件中，用 \<set\> 元素来配置 1 对 n 关联及 n 对 n 关联关系，\<set\> 元素有 lazy 和 fetch 属性
        - lazy：主要决定 orders 集合被初始化的时机，即到底是在加载 Customer 对象时就被初始化还是在程序访问 orders 集合时被初始化
        - fetch：取值为 select 或 subselect 时，决定初始化 orders 集合的查询语句的形式；若取值为 join，则决定 orders 集合被初始化的时机，默认值为 select
        - 当 fetch 属性为 subselect 时：
            - 假定 Session 缓存中有 n 个 orders 集合代理类实例没有被初始化，Hibernate 能够通过带子查询的 SELECT 语句，来批量初始化 n 个 orders 集合代理类实例
            - batch-size 属性将被忽略
            - 子查询中的 SELECT 语句为最初查询 Customers 表的 OID 的 SELECT 语句
        - 当 fetch 属性为 join 时：
            - 检索 Customer 对象时，会采用迫切左外连接（通过左外连接加载与检索指定的对象关联的对象，也就是说使用左外连接进行查询并且把集合进行初始化）策略来检索所有关联的 Order 对象
            - lazy 属性会被忽略
            - Query 的 `list()` 方法会忽略映射文件中配置的迫切左外连接检索策略，而依旧采用延迟加载策略
        - \<set\> 元素的 batch-size 属性：用来为延迟检索策略或立即检索策略设定批量检索的数量，批量检索能减少 SELECT 语句的数目，提高延迟检索或立即检索的运行性能（代表一次可以初始化多少个集合）
    
4. 延迟检索和增强延迟检索
    - 在延迟检索（lazy 属性设为 true 时）集合属性时，Hibernate 在以下情况下初始化集合代理类实例：
        - 应用程序第一次访问集合属性：`iterator()` 、`size()` 、`contains()` 等方法
        - 通过 `Hibernate.initialize()` 静态方法显式初始化
    - 增强延迟检索（lazy 属性为 extra）：与 `lazy = "true"` 类似，主要区别是增强延迟检索策略能进一步延迟 Customer 对象的 orders 集合代理实例的初始化时机
        - 当程序第一次访问 orders 属性的 `iterator()` 方法时，会导致 orders 集合代理类实例的初始化
        - 当程序第一次访问 orders 属性的 `size()` 、`contains()` 和 `isEmpty()` 方法时，Hibernate 不会初始化 orders 集合类的实例，仅通过特定的 SELECT 语句查询必要的信息，不会检索所有的 Order 对象
    
5. n 对 1 和 1 对 1 关联的检索策略

    - 和 \<set\> 一样，\<many-to-one\> 元素也有一个 lazy 属性和 fetch 属性

        | lazy 属性（默认为 proxy） | fetch 属性（默认为 select） | 检索 Order 对象时对关联的 Customer 对象使用的检索策略 |
        | ------------------------- | --------------------------- | ----------------------------------------------------- |
        | proxy                     | 未显式设置（取默认值）      | 延迟检索                                              |
        | no-proxy                  | 未显式设置（取默认值）      | 无代理延迟检索                                        |
        | FALSE                     | 未显式设置（取默认值）      | 立即检索                                              |
        | 未显式设置（取默认值）    | join                        | 迫切左外连接策略                                      |

    - 若 fetch 属性设为 join，那么 lazy 属性会被忽略

    - 迫切左外连接检索策略的优点在于比立即检索策略使用的 SELECT 语句更少

    - 无代理延迟检索需要增加持久化类的字节码才能实现

    - Query 的 `list()` 方法会忽略映射文件配置的迫切左外连接检索策略，而采用延迟检索策略

    - 如果在关联级别使用了延迟加载或立即加载检索策略，可以设定批量检索大小，以帮助提高延迟检索或立即检索的运行性能

    - Hibernate 允许在应用程序中覆盖映射文件中设定的检索策略

    - 1 端 \<class\> 元素可以通过设置 batch-size 属性值来设定一次初始化 1 端代理对象的个数

----

##### Hibernate 检索方式

1. Hibernate 提供了以下几种检索对象的方式
    - 导航对象图检索方式：根据已经加载的对象导航到其他对象
    - OID 检索方式：按照对象的 OID 来检索对象（主要指 Session 的 `get()` 方法和 `load()` 方法）
    - HQL 检索方式：使用面向对象的 HQL 查询语言（ HQL：Hibernate Query Language）
    - QBC 检索方式：使用 QBC（ Query By Criteria ）API 来检索对象，这种 API 封装了基于字符串形式的查询语句，提供了更加面向对象的查询接口
    - 本地 SQL 检索方式：使用本地数据库的 SQL 查询语句

----

##### HQL 检索方式

1. HQL 是面向对象的查询语言，与 SQL 语言类似，在 Hibernate 提供的各种检索方式中，HQL 是使用最广的一种检索方式，有如下功能：

    - 在查询语句中设定各种查询条件
    - 支持投影查询，即仅检索出对象的部分属性
    - 支持分页查询
    - 支持连接查询
    - 支持分组查询，允许使用 HAVING 和 GROUP BY 关键字
    - 提供内置聚集函数，如 `sum()` 、`min()`、`max()` 
    - 支持子查询
    - 支持动态绑定参数
    - 能够调用用户定义的 SQL 函数或标准的 SQL 函数

2. HQL 检索方式包括以下步骤：

    - 通过 Session 的 `createQuery()` 方法创建一个 Query 对象，他包括一个 HQL 查询语句，HQL 查询语句中可以包含命名参数
    - 动态绑定参数
    - 调用 Query 相关方法执行查询语句

3. Query 接口支持**方法链编程风格**，他的 `setXxx()` 方法返回自身实例而不是 void 类型

4. HQL 与 SQL 对比：

    - HQL 是面向对象的，Hibernate 负责解析 HQL 查询语句，然后根据 ORM 映射文件中的映射信息将 HQL 查询语句翻译成相应的 SQL 语句，HQL 查询语句中的主体是域模型中的类及类的属性
    - SQL 查询语句是与关系数据库绑定在一起的，SQL 查询语句中的主体是数据库表及表的字段

5. 绑定参数：

    - Hibernate 的参数绑定机制依赖于 JDBC API 中的 PreparedStatement 的预定义 SQL 语句功能
    - HQL 的参数绑定有两种形式：
        - 按参数名字绑定：在 HQL 查询语句中定义命名参数，命名参数以 “:” 开头
        - 按参数位置绑定：在 HQL 查询语句中使用 “?” 来定义参数位置
    - 相关方法：
        - `setEntity()` ：把参数与一个持久化类绑定
        - `setParameter()` ：绑定任意类型的参数，该方法的第三个参数显式指定 Hibernate 映射类型

6. HQL 采用 ORDER BY 关键字对查询结果进行排序

7. 分页查询

    - `setFirstResult(int firstResult)` ：设定从哪一个对象开始检索，参数 firstResult 表示这个对象在查询结果中的索引位置，索引位置的起始值为 0，默认情况下，Query 从查询结果中的第一个对象开始检索
    - `setMaxResult(int maxResult)` ：设定一次最多检索出的对象的数目，在默认情况下，Query 和 Criteria 接口检索出查询结果中所有的对象

8. 在映射文件中定义命名查询语句

    - Hibernate 允许在映射文件中定义字符串形式的查询语句

    - \<query\> 元素用于定义一个 HQL 查询语句，他和 \<class\> 元素并列

        ```XML
        <query name="findNewsByTitle">
            <![CDATA[
            	FROM News n WHERE n.title LIKE :title
            ]]>
        </query>
        ```

    - 在程序中通过 Session 的 `getNameQuery()` 方法获取查询语句对应的 Query 对象

9. 投影查询

    - 查询结果仅包含实体的部分属性，通过 SELECT 关键字实现

    - Query 的 `list()` 方法返回的集合中包含的是数组类型的元素，每个对象数组代表查询结果的一条记录

    - 可以在持久化类中定义一个对象的构造器来包装投影查询返回的记录，使程序能完全运用面向对象的语义来访问查询结果集

    - 可以通过 DISTINCT 关键字来保证查询结果不会返回重复元素

        ```SQL
        /* 返回一个包含 Object[] 的 List */
        SELECT e.email, e.salary, e.dept FROM Employee e WHERE e.dept = :dept;
        /* 返回一个包含 Employee 对象的 List (前提是 Employee 类中要有这个构造器) */
        SELECT new Employee(e.email, e.salary, e.dept) FROM Employee e WHERE e.dept = :dept;
        ```

10. 报表查询

    - 报表查询用于对数据分组和统计，与 SQL 一样，HQL 利用 GROUP BY 关键字对数据分组，用 HAVING 关键字对数据设定约束条件
    - 在 HQL 查询语句中可以调用以下聚集函数：
        - `count()`
        - `min()`
        - `max()`
        - `sum()`
        - `avg()`

----

##### HQL 的迫切左外连接

1. 迫切左外连接

    - LEFT JOIN FETCH 关键字表示迫切左外连接检索策略

    - `list()` 方法返回的集合中存放实体对象的引用，每个 Department 对象关联的 Employee 集合都被初始化，存放所有关联的 Employee 的实体对象

    - 查询结果中可能会包含重复元素，可以通过一个 HashSet 来过滤重复元素（或者 DISTINCT 关键字）

        ```SQL
        /* DISTINCT 关键字位置 */
        SELECT DISTINCT d FROM Department d LEFT JOIN FETCH d.emps;
        ```

2. 左外连接

    - LEFT JOIN 关键字表示左外连接查询
    - `list()` 方法返回的集合中存放的是对象数组类型
    - 根据配置文件来决定 Employee 集合的检索策略
    - 如果希望 `list()` 方法返回的集合仅包含 Department 对象，可以在 HQL 查询语句中使用 SELECT 关键字

----

##### HQL 的迫切内连接

1. 迫切内连接
    - INNER JOIN FETCH 关键字表示迫切内连接，也可以省略 INNER 关键字
    - `list()` 方法返回的集合中存放 Department 对象的引用，每个 Department 对象的 Employee 集合都被初始化，存放所有关联的 Employee 对象
2. 内连接
    - INNER JOIN 关键字表示内连接，也可以省略 INNER 关键字
    - `list()` 方法返回的集合中存放的每个元素对应查询结果的一条记录，每个元素都是对象数组类型
    - 如果希望 `list()` 方法返回的集合仅包含 Department 对象，可以在 HQL 查询语句中使用 SELECT 关键字
3. 与左外连接的区别：不返回左表不符合条件的记录

----

##### 关联级别运行时的检索策略

1. 如果在 HQL 中没有显式指定检索策略，将使用映射文件配置的检索策略
2. HQL 会忽略映射文件中设置的迫切左外连接策略，如果希望 HQL 采用迫切左外连接策略，就必须在 HQL 查询语句中显式的指定
3. 若在 HQL 代码中显式指定了检索策略，就会覆盖映射文件中配置的检索策略

----

##### QBC 检索

1. QBC 查询就是通过使用 Hibernate 提供的 Query By Criteria API 来查询对象，这种 API 封装了 SQL 语句的动态拼装，对查询提供了更加面向对象的功能接口

2. 本地 SQL 查询来完善 HQL 不能涵盖所有的查询特性

3. 使用步骤

    ```Java
    // 通过 Session 创建一个 Criteria 对象
    Criteria criteria = session.createCriteria(Employee.class);
    
    // 添加查询条件
    // 在 QBC 中查询条件使用 Criterion 来表示
    // Criterion 可以通过 Restrictions 的静态方法获取
    criteria.add(Restrictions.eq("email", "SKUMAR"));
    criteria.add(Restrictions.gt("salary", 5000F));
    
    // 执行查询
    Employee employee = (Employee) criteria.uniqueResult();
    ```

4. SQL 中的 AND 和 OR

    ```Java
    // AND 条件使用 Conjunction 类表示, Conjunction 本身又是一个 Criterion 对象
    // 而且 Conjunction 中还可以添加 Criterion
    Conjunction conjunction = Restrictions.conjunction();
    conjunction.add(Restrictions.like("name", "a", MatchMode.ANYWHERE));
    
    // OR 条件使用 Disjunction 类表示, 与 Conjunction 类似
    Disjunction disjunction = Restrictions.disjunction();
    disjunction.add(Restrictions.ge("salary", 6000F));
    ```

5. 统计查询

    ```Java
    // 统计查询使用 Projection 类来表示
    criteria.setProjection(Projections.max("salary"));
    ```

6. 排序查询

    ```Java
    // 排序使用 Criteria 的 addOrder()方法
    criteria.addOrder(Order.asc("salary"));
    ```

7. 分页查询

    ```Java
    // 分页查询与 HQL 相同, 都是使用 setFirstResult() 方法与 setMaxResults() 方法
    int pageSize = 5;
    int pageNum = 4;
    criteria.setFirstResult((pageNum - 1) * pageSize)
        	.setMaxResults(pageSize)
        	.list();
    ```

---

##### 本地 SQL 检索

```Java
// 与 HQL 类似, 都是使用 Session 来获取一个 Query 对象来进行
String sql = "INSERT INTO department VALUES(?, ?)";
Query query = session.createSQLQuery(sql);
query.setInteger(0, 80)
     .setString(1, "name")
     .executeUpdate();
```

----

##### Hibernate 缓存

1. Hibernate 提供了两个级别的缓存

    - 第一级别的缓存是 Session 级别的缓存，他是属于事务范围的缓存，这一级别的缓存是由 Hibernate 管理的
    - 第二级别的缓存是 SessionFactory 级别的缓存，他是属于进程范围的缓存

2. SessionFactory 的缓存可以分为两类

    - 内置缓存：Hibernate 自带的，不可卸载的，通常在 Hibernate 的初始化阶段，Hibernate 会把映射元数据和预定义的 SQL 语句放到 SessionFactory 的缓存中，映射元数据是映射文件中数据（`*.hbm.xml` 中数据）的复制，该内置缓存是只读的
    - 外置缓存（二级缓存）：一个可配置的缓存插件，在默认情况下，SessionFactory 不会启用这个缓存插件，外置缓存中的数据是数据库数据的复制，外置缓存的物理介质可以是内存也可以是硬盘

3. 适合放入二级缓存中的数据：很少被修改的，不是很重要的数据，允许出现偶尔的并发问题的数据

4. 不适合放入二级缓存中的数据：经常被修改的，财务数据（不允许出现并发问题），与其他应用程序共享的数据

5. 二级缓存的并发访问策略

    - 两个并发的事务同时访问持久层的缓存的相同数据时，也有可能出现各类并发问题
    - 二级缓存可以设定如下四种并发访问策略，每一种访问策略对应一种事务隔离级别：
        - 非严格读写（ Nonstrict-read-write ）：不保证缓存与数据库数据的一致性，提供 Read Uncommited 事务隔离级别，对于极少被修改，而且允许脏读的数据，可以采用本策略
        - 读写型（ Read-write ）：提供 Read Commited 数据隔离级别，对于经常读但是很少被修改的数据，可以采用这种策略（因为可以防止脏读）
        - 事务型（ Transactional ）：仅在受管理环境下使用，提供 Repeatable Read 事务隔离级别，对于经常读但很少被修改的数据，可以采用这种策略（因为可以防止脏读和重复读）
        - 只读型（ Read-Only ）：提供 Serializable 数据隔离级别，对于从来不会被修改的数据，可以采用这种策略

6. 管理 Hibernate 的二级缓存

    - Hibernate 的二级缓存是进程或集群范围内的缓存

    - 二级缓存是可配置的插件，Hibernate 允许使用以下缓存插件：

        - EHCache：可作为进程范围内的缓存，存放数据的物理介质可以使用内存或者硬盘，对 Hibernate 的查询缓存提供了支持

        - OpenSymphony OSCache：可作为进程范围内的缓存，存放数据的物理介质可以使用内存或硬盘，提供了丰富的缓存数据过期策略，对 Hibernate 的查询缓存提供了支持

        - SwarmCache：可作为集群范围内的缓存，但不支持 Hibernate 的查询缓存

        - JBossCache：可作为集群范围内的缓存，支持 Hibernate 的查询缓存

        - 四种缓存插件支持的并发访问策略

            |                      | Read-Only | Nonstrict-read-write | Read-Write | Transactional |
            | -------------------- | --------- | -------------------- | ---------- | ------------- |
            | EHCache              | ✓         | ✓                    | ✓          | ✗             |
            | OpenSymphony OSCache | ✓         | ✓                    | ✓          | ✗             |
            | SwarmCache           | ✓         | ✓                    | ✗          | ✗             |
            | JBossCache           | ✓         | ✗                    | ✗          | ✓             |

7. 配置进程范围内的二级缓存

    - 步骤（以 EHCache 为例）

        - 选择合适的缓存插件：EHCache（ jar 包和配置文件），并编辑配置文件（ jar 包路径：`hibernate\lib\optional\ehcache\*.jar`，配置文件路径：`hibernate\project\etc\ehcache.xml`）

        - 在 Hibernate 的配置文件中启用二级缓存并指定和 EHCache 对应的缓存适配器

            ```XML
            <!-- 启用二级缓存 -->
            <property name="cache.use_second_level_cache">true</property>
            
            <!-- 指定对应的缓存适配器, 如果这个值不对就需要去 jar 包里找正确的 -->
            <property name="hibernate.cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</property>
            ```

        - 选择需要使用二级缓存的持久类，设置他的二级缓存并发访问策略

            - \<class\> 元素的 cache 子元素表示 Hibernate 会缓存对象的简单属性，但不会缓存集合属性，若希望缓存集合属性中的元素，必须在 \<set\> 元素中加入 \<cache\> 子元素

            - 在 Hibernate 配置文件中通过 \<class-cache /\> 节点配置使用缓存

                ```XML
                <class-cache usage="read-write" class="完整类路径" />
                ```

        - 设置持久化类中集合的二级缓存（注意要同时将集合中元素的类也使用二级缓存）

            ```XML
            <collection-cache usage="read-write" class="完整类路径.集合属性名" />
            ```

8. ehcache.xml 文件解释

    - \<diskStore\>：指定一个目录（当 EHCache 把数据写到硬盘上时，就写到这个目录下）
    - \<defaultCache\>：设置缓存的默认数据过期策略
    - \<cache\>：设定具体的命名缓存的数据过期策略，每个命名缓存代表一个缓存区域
    - 缓存区域（ region ）：一个具体名称的缓存块，可以给每一个缓存块设置不同的缓存策略，如果没有设置任何的缓存区域，则所有被缓存的对象，都将使用默认的缓存策略
    - Hibernate 在不同的缓存区域保存不同的类/集合
        - 对于类而言，区域的名称是类名，如 com.beiran.Customer
        - 对于集合而言，区域的名称是类名加属性名，如 com.beiran.Customer.orders

9. 查询缓存

    - 默认情况下设置的二级缓存对 HQL 和 QBC 查询是无效的

    - 对于经常使用的查询语句，如果启用了查询缓存，当第一次执行查询语句时，Hibernate 会把查询结果存放在查询缓存中，以后再次执行该查询语句时，只需从缓存中获得查询结果，从而提高查询性能

    - 查询缓存适用于以下场合：

        - 应用程序运行时经常使用查询语句
        - 很少对与查询语句检索到的数据进行插入、删除和更新操作

    - 启用查询缓存的步骤：

        - 配置二级缓存（因为查询缓存依赖于二级缓存）

        - 在 Hibernate 配置文件中启用查询缓存

            ```XML
            <!-- 配置文件中启用查询缓存 -->
            <property name="cache.use_query_cache">true</property>
            ```

        - 对于希望启用查询缓存的查询语句调用 Query 的 `setCacheable()` 方法

10. 时间戳缓存区域

    - 时间戳缓存区域存放了对于查询结果相关的表进行插入、更新或者删除操作的时间戳，Hibernate 通过时间戳缓存区域来判断缓存的查询结果是否过期
        - T1 时刻执行查询操作，把查询结果存放在 QueryCache 区域，记录该区域时间戳为 T1
        - T2 时刻对查询结果相关的表进行更新，Hibernate 把 T2 时刻存放在 UpdateTimestampCache 区域
        - T3 时刻执行查询结果前，先比较 QueryCache 区域的时间戳和 UpdateTimestampCache 区域的时间戳，若 T2 \> T1，那么就丢弃原先存放在 QueryCache 区域的查询结果，重新到数据库中查询数据，再把结果存放到 QueryCache 区域；若 T2 \< T1，则直接从 QueryCache 中获取查询结果

----

##### Query 接口的 iterate() 方法

1. 同 `list()` 方法一样也能执行查询操作
2. `list()` 方法执行的 SQL 语句包含实体类对应的数据表的所有字段
3. `iterate()` 方法执行的 SQL 语句仅包含实体类对应的数据表的 ID 字段
4. 当遍历访问结果集时，该方法先到 Session 缓存及二级缓存中查看是否存在特定的 OID 的对象，如果存在，就直接返回该对象，如果不存在就通过相应的 SELECT 语句去数据库中加载特定的实体对象
5. 大多数情况下应该使用 `list()` 方法来执行查询，`iterate()` 方法仅适用于以下场合可以稍微提升一点查询的性能：
    - 要查询的数据表中包含大量字段
    - 启用了二级缓存，并且二级缓存中可能已经包含了待查询的对象

----

##### 管理 Session

1. Hibernate 自身提供了三种管理 Session 对象的方法
    - Session 对象的生命周期与本地线程绑定
    - Session 对象的生命周期与 JTA 事务绑定
    - Hibernate 委托程序管理 Session 对象的生命周期
2. 在 Hibernate 的配置文件中，`hibernate.current_seesion_context_class` 属性用于指定 Session 管理方式，可选值有：
    - thread：Session 对象的生命周期与本地线程绑定
    - jta：Session 对象的生命周期与 JTA 事务绑定
    - managed：Hibernate 委托程序来管理 Session 对象的生命周期
3. Hibernate 按以下规则将 Session 与本地线程绑定
    - 当一个线程（ threadA ）第一次调用 SessionFactory 对象的 `getCurrentSession()` 方法时，该方法会创建一个新的 Session 对象（ SessionA ），把该对象与 threadA 绑定，然后将 SessionA 返回
    - 当 threadA 再次调用 SessionFactory 对象的 `getCurrentSession()` 方法时，该方法会将 SessionA 对象返回
    - 当 threadA 提交 SessionA 对象关联的事务时，Hibernate 会自动 flush SessionA 对象的缓存，然后提交事务，关闭 SessionA 对象，当 threadA 撤销 SessionA 对象关联的事务时，也会自动关闭 SessionA 对象
    - 当 threadA 再次调用 SessionFactory 对象的 `getCurrentSession()` 方法时，该方法又会创建一个新的 Session 对象（ SessionB ），并把该对象与 threadA 绑定，然后将 SessionB 返回

----

##### 批量处理数据

1. 通常指在一个事务中处理大量的插入、更新、删除操作

2. 通过 Session

    - Session 的 `save()` 以及 `update()` 方法都会把处理的对象存放在自己的缓存中，如果通过一个 Session 对象来处理大量持久化对象，应该及时从缓存中清空已处理完毕并且不会再访问的对象，具体做法通常是再处理完一个对象或小批量对象后，立即调用 `flush()` 方法刷新缓存，然后再调用 `clear()` 方法清空缓存

    - 有以下约束：

        - 需要在 Hibernate 配置文件中设置 JDBC 单次批处理的数目，应保证每次向数据库发送批量的 SQL 语句数目与 batch_size 属性一致
        - 若对象采用 ”identity“ 标识符生成器，则 Hibernate 无法在 JDBC 层进行批量插入操作
        - 进行批量操作时，建议关闭二级缓存

    - 使用可滚动的结果集 `org.hibernate.ScrollableResults`，该对象实际上并不包含任何对象，只包含用于在线定位记录的游标，只有当程序遍历访问 ScrollableResults 对象的特定元素时才会到数据库中加载相应的对象

    - `org.hibernate.ScrollableResults` 对象由 Query 的 `scroll()` 方法返回

        ```Java
        ScrollableResults sr = session.createQuery("FROM Department").scroll();
        ```

3. 通过 HQL

    - HQL 只支持 INSERT INTO ... SELECT （子查询）形式的插入语句，但不支持 INSERT INTO ... VALUES 形式的插入语句，所以不能用 HQL 进行批量插入操作

4. 通过 StatelessSession（无状态 Session）

    - StatelessSession 与 Session 的区别：
        - StatelessSession 没有缓存，通过 StatelessSession 来加载、保存或更新后的对象处于游离态
        - StatelessSession 不会与 Hibernate 的二级缓存交互
        - 当调用 StatelessSession 的 `save()` 、`update()`、`delete()` 方法时，这些方法会立即执行相应的 SQL 语句，而不会仅计划执行一条 SQL 语句
        - StatelessSession 不会进行脏数据检查，因此修改了对象的属性后，还需要调用 StatelessSession 的 `update()` 方法来更新数据库中数据
        - StatelessSession 不会对关联的对象进行任何级联操作
        - 通过同一个 StatelessSession 对象两次加载的 OID 相同的对象，得到的两个对象内存地址不同
        - StatelessSession 所做的操作可以被 Interceptor 拦截到，但是会被 Hibernate 的事件处理系统忽略掉

5. 通过 JDBC API