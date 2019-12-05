---
title: Thinking in Java 笔记
date: 2019-11-05 16:00:34
tags: 复习
categories: Java
comments: false
---

##### 对象导论

1. 访问控制的第一个存在原因就是让客户端程序员无法接触到他们不该接触的部分，这些部分对数据类型的内部操作来说是必须的，但并不是用户解决特定问题所需的接口的一部分。<!-- more -->
2. 访问控制的第二个存在原因就是允许库设计者可以改变内部的工作方式而不用担心会影响到客户端程序员。
3. 因为是在使用现有的类合成新的类，所以这种概念被称为 “组合” ，如果组合是动态发生的，那么它通常被称为 “聚合” 。组合通常被视为 “拥有（ has a ）” 关系。
4. 继承：简单的可以理解为青出于蓝而胜于蓝
5. 两种方法使子类与父类具有差异性：① 新增父类没有的方法（需要考虑是否父类也有需要这些新增的方法的可能性）② 覆盖父类中已有的方法
6. 纯粹替代（替代原则）
7. 前期绑定、后期绑定
    - 前期绑定：在程序执行前根据编译时类型绑定
    - 后期绑定：当向对象发送消息时，被调用的代码直到运行时才能确定，编译器确保被调用方法的存在，并对调用参数和返回值执行类型检查（无法提供此类保证的语言被称为是弱类型的），但是并不知道将被执行的确切代码
8. 把将子类看作是他的父类的过程称为 “向上转型”
9. 并发需要考虑共享资源的问题
10. 事务处理：保证一个客户插入的新数据不会覆盖另一个客户插入的新数据，也不会在将其添加到数据库的过程中丢失

----

##### 操作符

1. “==” 和 “!=” 比较的是对象的引用。`equals()` 的默认行为也是比较引用。
2. 窄化转化（高精度转换为低精度，例如 long 转为 int ）与扩展转换（低精度转换为高精度，例如 int 转换为 long ）。
3. 在进行窄化转换时，Java 默认是不会进行四舍五入操作的，也就是说直接舍弃掉小数部分只保留整数部分。如果需要四舍五入，则要使用 `Math.round()` 方法对数字进行操作。
4. 低位数值（如 int ）在与高位数值（如 long ）进行运算时，结果会按高位来进行输入（输入 long 类型的结果）。
5. 在进行运算时要注意溢出。

----

##### 初始化和清理

1. 只能在构造器中使用 `this` 来调用其他构造器，并且这个调用在构造器中只能调用一次，并且这个调用一定是在起始处，并且禁止在非构造方法中使用 `this` 调用构造器。
2. 引用计数模式、停止-复制模式、标记-清扫模式。自适应清理。
3. 在类的内部，变量定义的先后顺序决定了初始化的顺序，即使变量定义分散在方法定义之间，他们仍旧会在任何方法（包括构造器）被调用之前初始化。
4. 可变参数（Object... args）本质上是一个数组，如若传 0 个参数给可变参数列表，这是可行的。在 JDK1.5 之前没有可变参数特性时，会使用（Object[] args）这样的形式来实现可变参数。
5. 可变参数列表使得重载过程变得复杂了，可以通过增加一个非可变参数来解决这个问题。

---

##### 访问权限控制

1. `package` 关键字和 `import` 关键字允许做的是将单一的全局命名空间分割开，使得无论多少人使用，都不会出现名称冲突的问题。
2. 访问权限的控制通常被称为是具体实现的隐藏，把数据和方法包装进类当中，以及具体实现的隐藏，共同被称为封装。其结果是一个同时带有特征和行为的数据类型。

----

##### 复用类

1. 初始化引用有四种方法：
    - 在定义对象的地方，这意味着他们总是能够在构造器被调用之前被初始化（也就是定义了就直接初始化）
    - 在类的构造器中（也就是在构造方法中对本类所拥有的属性或其他的进行初始化）
    - 在正要使用对象之前，这种方法被称为惰性初始化（即延迟初始化 Delayed initialization ），在生成对象不值得及不必每次都生成对象的情况下，这种方式可以减少额外的负担
    - 使用实例初始化（ new 一个对象）
2. 当创建了一个子类的对象，该对象包含了一个父类的子对象，这个对象与用父类直接创建的对象是一样的。所以在子类的构造方法中第一件应该做的事就是调用父类的构造方法（即 `super.constructor()` ）
3. 复用类的三种方法：组合、继承、代理
    - 代理：在类中放置一个成员对象，并暴露该成员对象的所有方法（调用）
4. 到底是使用组合还是继承，一个最清晰的办法是问一问自己是否需要从新类向基类进行向上转型；如果需要向上转型，那么继承是必要的；如果不需要，则应当好好考虑是否需要继承
5. 一个既是 `static` 又是 `final` 的域只占据一段不能改变的存储空间
6. 给引用添加 `final` 时，意味着这个引用无法再指向另一个新的对象，但是已经指向的对象内部的成员变量是允许修改的
7. 空白 `final` ：声明为 `final` 但是又未给定初值的域。（如 `public final String s;` ）空白 `final` 必须在使用之前被初始化（意味着必须在构造器中初始化）
8. `final` 参数：在参数列表中以声明方式将参数指明为 `final` ，这一特性主要用来向匿名内部类传递数据
9. `final` 类中的所有方法都隐式的指定为 `final`，因为无法覆盖他们。所以在 `final` 类中可以给方法添加 `final` 修饰词，但是这样没意义
10. 构造器也是 `static` 方法，尽管 `static` 关键字并没有显式地写出来，因此更准确的讲，类是在其任何 `static` 成员被访问时加载的

----

##### 多态

1. 将一个方法同一个方法主体关联起来称为 “绑定”

    - 若在程序执行前进行绑定（如果有的话，由编译器和连接程序实现），叫做**前期绑定**
    - 在运行时根据对象类型进行绑定叫做**后期绑定**，也叫做**动态绑定**或者**运行时绑定**
    - Java 中除了 `static` 方法和 `final` 方法（ `private` 方法属于 `final` 方法）之外，其他所有方法都是后期绑定

2. 我们所做的修改，不会对程序中其他不应受影响的部分造成破坏

3. :star:注意：调用构造器的顺序：

    1. 在其他任何事物发生之前，将分配给对象的存储空间初始化成二进制的零
    2. 调用基类构造器。这个步骤会不断地反复递归下去，首先是构造这种层次结构的根，然后是下一层导入类，依此类推，直到最底层的导出类
    3. 按声明顺序调用成员变量的初始化方法
    4. 调用导出类构造器的主体

4. 销毁对象的顺序应与声明对象的顺序相反。若某个类中的成员对象也存在于其他一个或多个类中（共享的情况），那就不能简单的进行销毁，这种情况应当使用 “引用计数” 来跟踪仍旧访问着共享对象的数量

5. 由于在构造器中调用某个动态绑定的方法会造成所有数据可能为零的情况（原因参考调用构造器顺序中的第一点），所以在编写构造器时，要尽可能的遵循这一条规则：“**用尽可能简单的方法使对象进入正常状态，如果可以的话，避免调用其他方法**”。在构造器内部唯一能够安全调用的方法是基类中的 `final` 方法（也适用于 `private` 方法，因为 `private` 方法自动属于 `final` 方法），因为这些方法不能被覆盖，也就不存在在基类构造器中调用导出类中对基类的某个方法的覆盖

6. 协变返回类型：在导出类中的被覆盖方法可以返回基类方法的返回类型的某种导出类型

    ```Java
    class Grain {
        public String toString() {
            return "Grain";
        }
    }
    
    class Wheat extends Grain {
        public String toString() {
            return "Wheat";
        }
    }
    
    class Mill {
        Grain process() {
            return new Grain();
        }
    }
    
    class WheatMill {
        // 覆盖父类的 process() 方法
        // 并将返回值修改成 Grain 类的子类 Wheat
        Wheat process() {
            return new Wheat();
        }
    }
    ```

7. 用继承表达行为间的差异，并用字段表达状态上的变化（状态模式）
8. 运行时类型识别（RTTI : Runtime type infomation ）
   
    - 向下转型对类型进行检查时，如果类型不对，会抛出一个 `ClassCastException` （类转型异常）

----

##### 接口

1. 创建一个能够根据所传递的参数对象的不同而具有不同行为的方法，叫做 “策略模式”
2. 适配器中的代码将接收你所拥有的接口，并产生你所需要的接口（适配器模式）
3. 使用接口的核心原因：为了能够向上转型为多个基类
    - 使用接口的第二个原因：防止客户端程序员创建该类的对象（抽象类也能做到）
    - 如果要创建一个不带任何方法定义以及成员变量的基类，那么就应该使用接口而不是抽象类
4. 恰当的原则是应当优先选择类而不是接口，从类开始，如果接口的必需性变得非常明确，那么就进行重构
5. 尽量避免在组合的不同接口中使用相同的方法名
6. 使用工厂方法模式的常见原因是创建框架

----

##### 内部类

1. 在内部类中生成外部类的对象的引用需要使用 `OuterClass.this` ；要想创建某个内部类的对象需要使用外部类对象才能创建

    - 在拥有外部类对象之前是不可能创建内部类对象的，因为内部类对象会连接到创建他的外部类对象上。如果创建的是**静态内部类**，那么就不需要对外部类对象的引用

    ```Java
    public class Outer {
        public class Inner {}
        public static void main(String[] args) {
            Outer outer = new Outer();
            // 使用外部类对象创建内部类对象
            Inner inner = outer.new Inner();
        }
    }
    ```

2. 定义一个匿名内部类并且希望他使用一个在其外部定义的对象，那么编译器会要求其参数引用是 `final` 的

3. 匿名内部类中的实例初始化（构造器效果）：使用非静态代码块来实现（因为每次创建对象都会执行）

    - 匿名内部类中的实例初始化方法无法重载，所有有且仅有一个构造器
    - 基类需要的是有参构造器：将基类需要的参数传递给基类的构造器即可

    ```Java
    interface Product {
        public String readName();
    }
    
    class Test {
        public Product getProduct(final float widget) {
            return new Product() {
                private int cost;
                // 非静态代码块（相当于匿名内部类的构造器）
                {
                    cost = Math.round(widget);
                    if (cost > 100) {
                        System.out.println("超重！");
                    }
                }
                
                public String readName() {
                    return "Product!";
                }
            }
        }
    }
    ```

4. 不需要内部类与外部类有联系时，可以使用关键字 `static` 来声明内部类，通常称为嵌套类

    - 普通内部类对象隐式地保存了一个引用，指向创建他的外部类对象
    - 当内部类是 `static` 时意味着：
        - 要创建嵌套类的对象，并不需要外部类的对象
        - 不能从嵌套类的对象中访问非静态的外部类对象
        - 普通内部类的字段与方法，只能放在类的外部层次上，所以普通内部类中不能有 `static` 数据和 `static` 字段，也不能包含嵌套类。但是嵌套类中可以包含上述所有

5. 接口内部的类由于是自动 `public` 和 `static` 的，所以甚至可以在接口内部实现接口本身

6. 一个内部类被嵌套多少层不重要，反正都能访问所有在他外面的外部类的所有成员

7. 使用内部类最重要的原因是：每个内部类都能独立的继承自一个（接口的）实现，所以无论外部类是否已经继承了某个（接口的）实现，对于内部类都没有影响

8. 如果使用内部类，可以获得一些如下所示的特性：

    - 内部类可以有多个实例，每个实例都有自己的状态信息，并且与其外部类对象的信息相互独立
    - 在单个外部类中，可以让多个内部类以不同的方式实现同一个接口，或继承同一个类
    - 创建内部类对象的时刻不依赖于外部类对象的实现
    - 内部类没有 “is-a” 关系，他就是一个独立的实体

9. 闭包（ closure ）：是一个可调用的对象，他记录了一些信息（持有上下文中某部分信息），这些信息来自于创建它的作用域

    - 内部类是面向对象的闭包，因为他不仅包含了外部类对象（创建内部类作用域）的信息，还持有一个指向外部类对象的引用，在此作用域内，内部类有权操作所有的成员（包括 `private` 成员）
    - **回调（ callback ）：通过回调，对象能够携带一些信息，这些信息允许他在稍后的某个时刻调用初始的对象**
    - 回调的价值在于他的灵活性，可以在运行时动态决定需要调用什么方法

10. 模板方法模式：模板方法包括算法的基本结构，并且会调用一个或多个可覆盖方法，以完成算法的动作。设计模式总是将变化的事物与不变的事物分离开，在模板方法模式中，模板方法是保持不变的事物，而那些可覆盖的方法就是变化的事物

11. 主要用来响应事件的系统叫做 “事件驱动系统”

12. 内部类允许：

    - 控制框架的完整实现是由单个的类创建的，从而使得实现的细节被封装了起来，内部类用来表示解决问题所必须的各种动作
    - 内部类能够很容易的访问外部类的任意成员，所以可以避免这种实现变得笨拙

13. 内部类的继承：

    - 内部类的构造器必须连接到指向外部类对象的引用
    - 所以在继承内部类时必须要有一个已经初始化了的外部类的对象，也就是要在内部类的子类的构造器中调用包含内部类的外部类对象的初始化方法 `enclosingClassReference.super()`

    ```Java
    // 外部类和内部类原型
    class Outer {
        class Inner {}
    }
    
    // 继承内部类
    class ExtendInnerClass extends Outer.Inner {
        // 构造方法中传递一个外部类的引用
        // 并且调用这个引用生成一个可连接的外部类对象
        public ExtendInnerClass(Outer outer) {
            outer.super();
        }
        
        @Test
        public void testExtendInnerClass() {
            Outer outer = new Outer();
            ExtendInnerClass extendInnerClass = new ExtendInnerClass(outer);
        }
    }
    ```

14. 创建一个新类继承自外部类时，重新定义外部类里面的内部类（也就是覆盖他们），实际上是覆盖不了的，除非在新类中创建一个内部类并指定他继承自外部类里面的内部类

15. 局部内部类不能有访问说明符，因为他不是外部类的一部分，但是他可以访问当前代码块里的常量以及外部类的所有成员。局部内部类允许有以类名命名的构造器

    - 使用局部内部类而不是匿名内部类的原因是**需要一个已命名的构造器，或者需要重载构造器**，而匿名内部类只能用于实例初始化
    - 使用局部内部类而不是匿名内部类的另一个原因就是**需要不止一个该内部类的对象**

16. 内部类编译文件命名规则（`.class` 文件）：`外部类类名$内部类类名.class`

    - 匿名内部类编译文件只会简单地产生一个数字作为标识符，例如 `外部类类名$1.class`、`外部类类名$2.class` 。如果是嵌套的内部类，只需用 `$` 将其连接在其外部类后面既可

----

##### 通过异常处理错误

1. 异常情形：是指阻止当前方法或作用域继续执行的问题

2. 监控区域：他是一段可能产生异常的代码，并且后面跟着处理这些异常的代码

3. 重抛异常会将异常抛给上一级环境中的异常处理程序，同一个 `try` 块的后续 `catch` 子句将被忽略。如果只是将当前异常对象重新抛出，那么 `printStackTrace()` 方法显示的将是原来异常抛出点的调用栈的信息，而不是重新抛出点的信息，若想更新这个信息，可以调用 `fillInStackTrace()` 方法，这样就将返回一个新的 `Throwable` 对象，他是通过把当前调用栈信息填入原来那个异常对象而建立的（原有的调用栈信息将会丢失）

4. 由于异常对象都是在堆上建立的，故垃圾回收器会自动将其回收

5. `Exception` 是可以被抛出的基本类型，在 Java 类库、用户方法以及运行时故障中都有可能抛出 `Exception` 型异常；`Error` 用来表示编译时和系统错误

6. 如果不捕获 `RuntimeException` 类型的异常而直达 `main()` 方法，那么在程序退出前会调用异常的 `printStackTrace()` 方法

7. 只能在代码中忽略 `RuntimeException` 类型的异常，因为 `RuntimeException` 代表的是编程错误：

    - 无法预料的错误（比如使用 null 引用）
    - 应该在代码中检查的错误（比如数组越界）

8. 派生类构造器不能捕获基类构造器抛出的异常

9. 异常限制对构造器不起作用

10. 对于在构造阶段可能会抛出异常，并且要求清理的类，最安全的使用方式是使用嵌套的 `try-catch` 语句。基本原则是：**在创建需要清理的对象后，立即进入一个 `try-finally` 语句块**

11. 异常处理的一个重要原则是：**只有在你知道如何处理的情况下才去捕获异常**。**异常处理的一个重要目标就是把错误处理的代码同错误发生的地点相分离**

12. 非检查异常通常指的是 `RuntimeException` ，也就是运行时异常

13. 可以将被检查的异常包装进 `RuntimeException` 中

     ```Java
     // WrapCheckedException 中 throwRuntimeException() 方法可以抛出各种异常
     // 但是都被包装在 RuntimeException 中
     // 所以在调用 throwRuntimeException() 方法时并不会要求捕获异常
     class WrapCheckedException {
         public void throwRuntimeException(int type) {
             try {
                 switch (type) {
                     case 0: throw new FileNotFoundException();
                     case 1: throw new IOException();
                     case 2: throw new RuntimeException("WTF?");
                     default: return;
                 }
             } catch (Exception e) {
                 throw new RuntimeException(e);
             }
         }
     }
     
     class OtherException extends Exception {}
     
     public class TurnOffChecking {
         public static void main(String[] args) {
             WrapCheckedException wce = new WrapCheckedException();
             wce.throwRuntimeException(3);
             
             for (int i = 0; i < 4; i++) {
                 try {
                     if (i < 3) {
                         wce.throwRuntimeException(i);
                     } else {
                         throw new OtherException();
                     }
                 } catch (OtherException e) {
                     System.out.println("OtherException " + e);
                 } catch (RuntimeException re) {
                     try {
                         throw re.getCause();
                     } catch (FileNotFoundException e) {
                         System.out.println("FileNotFoundException " + e);
                     } catch (IOException e) {
                         System.out.println("IOException " + e);
                     } catch (Throwable e) {
                         System.out.println("Throwable " + e);
                     }
                 }
             }
         }
     }
     ```

14. 应当在以下情况使用异常：

     - 在恰当的级别处理问题（在知道该如何处理的情况下采取捕获这个异常）
     - 解决问题并且重新调用产生异常的方法
     - 进行少许修补，然后绕过异常发生的地方继续执行
     - 用别的数据进行计算，以代替方法预计会返回的值
     - 把当前运行环境下能做的事情尽量做完，然后把相同的异常重抛到更高层
     - 把当前运行环境下能做的事情尽量做完，然后把不同的异常重抛到更高层
     - 终止程序
     - 进行简化
     - 让类库和程序更安全

----

##### 字符串

1. 使用 “+” 操作符连接字符串原理：构建一个 StringBuilder 对象，使用 StringBuilder 对象的 `append()` 方法来连接 “+” 操作符的每一个部分，最后将 StringBuilder 对象连接好的字符串通过 `toString()` 方法返回。所以如果要连接的字符串较长，则应该直接使用 StringBuilder 来进行连接（尤其是在循环中，因为如果直接使用 “+” 操作符来连接，每一次循环都会创建一个 StringBuilder 对象），并且 StringBuilder 还支持直接预先为其指定大小

2. 如果希望 `toString()` 方法打印出对象的内存地址，应该使用 Object 类的 `toString()` 方法，即 `super.toString()` 。因为如果使用 this 关键字来打印对象的内存地址的话，可能会发生自动类型转换（通常是因为 “+” 操作符），而自动类型转换会调用 this 的 `toString()` 方法，这样就会导致无意识的递归而出错

3. Formatter 类可用于格式化字符串与翻译数据，格式化修改语法：
    `%[argument_index$][flags][width][.precision]conversion`

    - width 用来控制域的长度
    - 可以用 “-” 符号来改变对齐方向（默认右对齐）
    - precision 用来指定最大尺寸，例如小数点后的位数，字符串的长度等

4. `String.format()` 内部也是创建一个 Formatter 对象来格式化字符串的

    ```Java
    public static String format(String format, Object... args) {
        return new Formatter().format(format, args).toString();
    }
    ```

----

##### 类型信息

1. 所有的类都是在对其第一次使用时，动态加载到 JVM 中的，当程序创建第一个对类的静态成员的引用时，就会加载这个类，这个证明构造器也是类的静态方法，即使在构造器之前并没有使用 `static` 关键字。所以使用 new 操作符创建类的新对象也会被当作对类的静态成员的引用
2. 如果一个类还没被加载，那么可以使用 `Class.forName("类名")` 来加载他
3. 使用 `.class` 来创建对 Class 对象的引用时，不会自动初始化该 Class 对象
4. 为了使用类而做的准备工作实际上包含三个步骤：
    1. 加载。这是由类加载器执行的，该步骤将查找字节码（通常在 classpath 所指定的路径中查找，但这并非是必需的），并从这些字节码中创建一个 Class 对象
    2. 链接。在链接阶段将验证类中的字节码，为静态域分配存储空间，并且如果必需的话，将解析这个类创建的对其他类的所有引用
    3. 初始化。如果该类具有超类，将对其初始化，执行静态初始化和静态初始化块
5. 初始化被延迟代理对静态方法（构造器隐式的是静态的）或者非常数静态域进行首次引用时才进行
6. 如果一个 `static final` 值是编译器常量，那么这个值不需要对类进行初始化就可以被读取。但是，如果只是将一个域设为 `static final` ，就不足以确保这种行为。如果一个 `static` 域不是 `final` 的，那么在对他进行访问时，总是要求在被读取前，要先进行链接（为这个域分配存储空间）和初始化（初始化该存储空间）
7. 已知的 RTTI （ Runtime Type Identification ）形式有：
    1. 传统的类型转换，由 RTTI 确保类型转换的正确性，如果执行了一个错误的类型转化，就会抛出一个 ClassCastException 异常
    2. 代表对象的类型的 Class 对象，通过查询 Class 对象可以获取运行时所需的信息
    3. `instanceof` 关键字
8. `instanceof` 与 “\==” 区别：`instanceof` 保持了类型的概念，他指的是 ”你是这个类吗？或者你是这个类的派生类吗？“；而 “\==” 就没有考虑继承，他或者是这个确切的类型，或者不是
9. RMI（远程方法调用）
10. RTTI 和反射的区别：对 RTTI 来说，编译器在编译时打开和检查 `.class` 文件（可以普通的调用对象的所有方法）；对于反射机制来说，`.class` 文件在编译时是不可获取的，所以是在运行时打开和检查 `.class` 文件
11. 有时引入空对象的思想将会很有用，他可以接收传递给他的所代表的对象的消息，但是将返回表示为实际上并不存在任何 “真实对象” 的值。空对象最有用之处在于他更靠近数据，因为对象表示的是问题空间内的实体
12. 在任何时刻，只要你想要将额外的操作从 “实际” 对象中分离到不同的地方，特别是当你希望能够很容易地做出修改，从没有使用额外操作转为这些操作，或者反过来时，代理就显得很有用（设计模式的关键就是封装修改————因此你需要修改事物以证明这种模式的正确性）
13. 动态代理可以将所有调用重定向到调用处理器，因此通常会调用处理器的构造器传递一个 “实际“ 对象的引用，从而使得调用处理器在执行其中介任务时，可以将请求转发
14. 空对象可以接受传递给他的所代表的对象的消息，但是将返回表示为实际上并不存在任何 ”真实“ 对象的值。用这种方式的好处是可以假定所有的对象都是有效的，从而不需要去检查 `null` 。空对象可以看作是策略模式的一种特例，空对象的一种变体称为空迭代器模式，他使得在组合层次结构中遍历各个节点中的操作对客户端透明（客户端可以使用相同的逻辑来遍历组合和叶子节点）
15. 空对象的逻辑变体是模拟对象和桩。模拟对象和桩都只是假扮可以传递实际信息的存活对象，而不是像空对象一样可以称为 `null` 的替代物
16. 模拟对象和桩之间的差异在于程度不同。模拟对象往往是轻量级和自测试的，通常很多模拟对象被创建出来是为了处理各种不同的测试情况，桩只是返回桩数据，他通常是重量级的，并且经常在测试之间被复用，桩可以根据他们被调用的方式，通过配置进行修改，因此桩是一种复合对象，他要做很多事情，而对于模拟对象来说，如果需要做很多事情，通常会创建大量小而简单的模拟对象
17. `interface` 关键字的一个重要目标是允许下隔离构件，从而降低耦合，但是通过 RTTI 是可以调用接口实现类中并不在接口中声明的方法的。解决这个问题的方法：① 直接将这个不在接口中声明的方法进行声明 ② 使用包访问权限，这样可以防止在包外部的客户端使用，但是使用反射仍然能够调用包访问权限的方法，甚至是 `private` 权限的方法，只需要在 Method 对象上调用 `setAccessible(true)` 就能使用所有方法
18. 使用 `javap -private 类名` 会显示这个类的所有信息
19. 即使是私有内部类和匿名内部类也依旧会被反射获取到信息，而 `final` 域实际上在遭遇修改时是安全的，运行时系统会在不抛出异常的情况下接受任何修改尝试，但实际上不会发生任何修改
20. 多态与 RTTI：RTTI 允许通过匿名基类的引用来发现类型信息，很有可能会导致多个 `switch` 语句的产生，而使用多态机制则仅调用一次既可以。面向对象编程语言的目的是让我们在凡是可以使用的地方都使用多态机制，只有在必需的地方使用 RTTI

----

##### 泛型

1. Java 泛型的核心概念：告诉编译器想使用什么类型的对象，然后编译器来处理一切细节

2. 元组：是将一组对象直接打包存储于其中的一个单一对象，这个容器对象允许读取其中元素，但是不允许向其添加新的对象（也就是数据传送对象或者信使）

3. 泛型方法使得该方法能够独立于类而产生变化，所以无论何时，只要可以，就应该尽量使用泛型方法，也就是说，如果使用泛型方法可以取代将整个类泛型化，那么就应该只使用泛型方法。另外，对于一个 `static` 的方法而言，无法访问泛型类的类型参数，所以，若 `static` 方法需要使用泛型参数，就必需使其成为泛型方法

4. 当使用泛型类时，必须在创建对象时就指定其类型参数的值，而使用泛型方法时，通常不必指明参数类型，因为编译器会自己找出具体类型，这称为类型参数推断

5. 类型推断只对赋值操作有效，其他时候并不起作用，如果你将一个泛型方法调用的结果作为参数，传递给另一个方法，这时编译器并不会执行类型推断，在这种情况下，编译器认为：调用泛型方法后，其返回值被赋给一个 Object 类型的变量

6. 在泛型方法中，可以显式地指明类型。显式地指明类型必须在点操作符与方法名之间插入尖括号（菱形操作符），然后把类型置于尖括号中。如果是在定义该方法的类的内部，必须在点操作符之前使用 `this` 关键字；如果是 `static` 方法，必须在点操作符之前加上类名，只有在编写非赋值语句才需要这样的额外说明

7. 泛型的一个重要好处是能够简单而安全的创建复杂的模型

8. 在泛型代码内部，无法获取任何有关泛型参数类型的信息

9. Java 泛型是使用擦除来实现的，这意味着当你在使用泛型时，任何具体的类型信息都被擦除了，唯一知道的是在使用一个对象。使用擦除来实现是因为要兼容以前未泛型化的类库，Java 为了这种兼容性而使用了擦除来实现泛型化，但这也导致了很多问题。（[Neal Gafter](http://gafter.blogspot.com/)）

    ```Java
    // a cheat way to create generic array
    class GenericArrayWithTypeToken<T> {
        private T[] array;
        
        public GenericArrayWithTypeToken(Class<T> type, int size) {
            // 使用强制转换的以及反射的方式创建泛型数组
            array = (T[]) Array.newInstance(type, size);
        }
        
        public void put(int index, T item) {
            array[index] = item;
        }
        
        public T get(int index) {
            return array[index];
        }
        
        // 返回整个数组
        public T[] rep() {
            return array;
        }
    }
    ```

10. 边界 `<T extends Object>` 声明 T 必须具有类型 Object 或者从 Object 导出的类型

11. 泛型类型参数将擦除到他的第一个边界，编译器实际上会把参数类型替换为他的擦除

12. 只有当你希望使用的类型参数比某个具体类型（以及他的所有子类型）更加 ”泛化“ 时，也就是说，当你希望代码能够跨多个类工作时，使用泛型才会有所帮助

13. 擦除的核心动机是他使得泛化的客户端可以使用非泛化的类库，反之亦然，这经常被称为 ”迁移兼容性“

14. 在泛型中所有动作都发生在边界处————对传递进来的值进行额外的编译期检查，并插入对传递出去的值的转型（checkcast）

    ```Java
    interface Generator<T> {
        T next();
    }
    
    // 生成某个类的对象
    // 这个类必须是 public 的并且有一个默认的构造器（无参构造器）
    class BasicGenerator<T> implements Generator<T> {
        private Class<T> type;
        
        private BasicGenerator(Class<T> type) {
            this.type = type;
        }
        
        @Override
        public T next() {
            try {
                return type.newInstance();
            } catch (Exception e) {
                throw new RuntimeException("该类禁止外部初始化！" + e);
            }
        }
        
        public static <T> Generator<T> create(Class<T> type) {
            return new BasicGenerator<T>(type);
        }
    }
    ```

15. 对于在泛型中创建数组，推荐使用 `Array.newInstance()` 方式；泛型方法返回值前的 `<T>` 代表返回值类型可以与参数中的类型不一致

16. 若需要获取泛型的类型信息，则需要引入类型标签来获得（泛型类中加入一个 `Class<T> type` 的属性，然后在构造器中进行初始化）

17. 创建一个 `new T()` 无法实现的一部分原因是因为擦除，另一部分原因是因为编译器无法验证 T 具有默认（无参）构造器。解决这个问题有两种方式，一种是使用显式的工厂，并将限制其类型，使得只能接受实现了这个工厂的类；另一种方法就是模板方法模式

    ```Java
    /**
     * 
     * 使用显式工厂并限制其类型
     *
     */
    
    interface FactoryInterface<T> {
        T create();
    }
    
    class Foo<T> {
        private T x;
        public <F extends FactoryInterface<T>> Foo(F factory) {
            x = factory.create();
        }
    }
    
    // 也可以有其他工厂
    class IntegerFactory implements FactoryInterface<Integer> {
        public Integer create() {
            return new Integer(0);
        }
    }
    
    public class FactoryTest {
        public static void main(String[] args) {
            new Foo<Integer>(new IntegerFactory());
        }
    }
    
    /**
     * 
     * 模板方法模式
     *
     */
    
    abstract class GenericWithCreate<T> {
        final T element;
        
        GenericWithCreate() {
            element = create();
        }
        
        abstract T create();
    }
    
    class Order {}
    
    class Creator extends GenericWithCreate<Order> {
        Order create() {
            return new Order();
        }
        
        void testFunction() {
            System.out.println(element.getClass().getSimpleName());
        }
    }
    
    public class CreatorTest {
        public static void main(String[] args) {
            Creator creator = new Creator();
            creator.testFunction();
        }
    }
    ```

18. PECS 原则

    - 如果要从集合中读取类型 T 的数据，并且不能写入，可以使用 `? extends` （Producer Extends）
    - 如果要向集合中写入类型 T 的数据，并且不需要读取，可以使用 `? super` （Consumer Super）
    - 如果既要存又要取，那么就不要使用任何通配符
    - `extends` 确定的是上界，所以可读；（比如 Fruit 有两个子类：Apple 和 Orange，那么为了防止一个 Apple 容器中插入 Orange 元素，所以拒绝写入）
    - `super` 确定的是下界，所以可写；

19. 捕获转换

    - 向一个使用 `<?>` 的方法传递原生类型，对编译器来说，可能会推断出实际的参数类型，使得这个方法可以回转并调用另一个使用这个确切类型的方法

    ```Java
    // function2 中将实际的类型参数传递给 function1 来使用
    class CaptureConversion {
        static <T> void function1(Holder<T> holder) {
            T t = holder.get();
            System.out.println(t.getClass().getSimpleName());
        }
        
        static void function2(Holder<?> holder) {
            function1(holder);
        }
        
        public static void main(String[] args) {
            Holder raw = new Holder<Integer>(1);  
            function2(raw);
            
            Holder rawBasic = new Holder();
            function2(rawBasic);
            
            Holder<?> wildcarded = new Holder<Double>(1.0);
            function2(wildcarded);
        }
    }
    
    class Holder<T> {
        private T value;
        
        public Holder() {};
        
        public Holder(T val) {
            value = val;
        }
        
        public void set(T val) {
            value = val;
        }
        
        public T get() {
            return value;
        }
        
        public boolean equals(Object obj) {
            return value.equals(obj);
        }
    }
    ```