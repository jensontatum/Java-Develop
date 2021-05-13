一.三个主流Java虚拟机

Hotspot、 JRockit、J9 三大虚拟机

###### 1. Hotspot虚拟机

- SUN公司Hotspot虚拟机，现被Oracle公司收购

- 名称中的Hotspot指的就是它的热点代码探测技术
  - 通过计数器找到最具编译价值代码，触发即时编译或栈上替换
  
  - 通过JIT即时编译器和解释器协同工作，在最优化的程序响应时间与最佳执行性能中取得平衡
  
  - 内存布局如下：
  
    ![image-20210323155605491](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323155605491.png)
    ![image-20210323155605491](https://raw.githubusercontent.com/jensontatum/Java-Develop/master/images/image-20210323155605491.png)

###### 2. JRockit虚拟机

- EBA公司 JRockit虚拟机，现被Oracle公司收购

- 专注于服务器端应用，不太关注程序启动速度。（移动端、客户端更看中用户体验，需要程序相应速度快速）因此=JRockit内部不包含解释器，全部代码都靠JIT即时编译器编译后执行

- JRockit  JVM是世界上最快的JVM

  

###### 3.  J9 虚拟机

- 广泛应用于IBM公司的各种Java产品，IBM公司产品

  

## 二、类加载子系统

###### 2.1类的加载过程

![image-20201223141211620](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201223141211620.png)

![image-20201223141358658](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201223141358658.png)

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201223141559000.png" alt="image-20201223141559000" style="zoom:80%;" />

- 加载（虚拟机自带的类加载器）

  - ①通过一个类的全限定名（包名+类名）获取定义此类的二进制字节流

  - ②将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构

  - ③在内存（指堆内存）中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据访问的入口

    

    补充：

    ​		①堆是java虚拟机所管理的内存中最大的一块内存区域，也是被各个线程共享的内存区域，在JVM启动时创建。该内存区域存放了对象实例及数组(所有new的对象)。

    ​		②方法区是用于存储虚拟机加载的类信息、常量、静态变量、是各个线程共享的内存区域。方法区还包含一部分是运行时常量池，其中的主要内容来自于JVM对Class的加载。

![image-20201223160632929](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201223160632929.png)

- 验证

  - 目的：确保Class文件的字节流包含的信息符合当前虚拟机的要求，确保这些信息被当作代码运行后不会危害虚拟机的安全。
  - 验证方式：文件格式验证、元数据验证、字节码验证、符号引用验证

- 准备

  - 为类变量分配内存并且设置设置该类变量的初始值，即零值
  - 这里不包含用final修饰的static，final修饰的变量在准备阶段显示初始化
  - 这里不会为实例变量分配初始化，类变量会分配在方法区，而实例变量会随着对象一起分配到Java堆内存中。

- 解析

  - 是Java虚拟机将符号引用转换为直接引用
  - 解析操作常常伴随着JVM在执行完初始化之后再执行
  - 符号引用就是一组符号来描述所引用的目标。符号引用的字面量形式明确定义在《java虚拟机规范》的Class文件格式中。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

  ![image-20201223162848291](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201223162848291.png)

- 初始化

  - 说明： 类加载过程中除了第一阶段用户使用自定义类加载器加载Class文件，其余动作都是Java虚拟机主导控制，直到初始化阶段，JVM真正执行类中编写的Java程序代码，将主导权交由应用程序
  - 初始化阶段就是执行类构造器方法<clinit>（）的过程。
  - 此方法不需定义，是Javac编译器自动生成的，自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来。（若类中没有静态代码块，也没有变量赋值操作，则不会生成该方法。
  - 构造器方法中指令按语句在源文件中出现的顺序执行。
  - <clinit>不同于类的构造器。（关联：构造器是虚拟机视角下的<init>（））
  - 若该类具有父类，JVM会保证子类的<clinit>（）执行前，父类的<clinit>（）已经执行完毕。
  - 虚拟机必须保证一个类的<clinit>（）方法在多线程下被同步加锁。

###### 2.2 类加载子系统的作用

![image-20201222151431274](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201222151431274.png)

###### 2.3 类加载器的分类

- 引导类加载器：   BootStrap ClassLoader(又叫启动类加载器)

  ![image-20201223140712889](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201223140712889.png)

- 自定义类加载器：将所有派生于抽象类ClassLoader的类加载器都划分为自定义类加载器，包含：

  - 扩展类加载器    Extention ClassLoader

    ![image-20201223140736405](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201223140736405.png)

  - 系统类加载器    Applicaton ClassLoader （又叫应用类加载器）

    ![image-20201223140819540](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201223140819540.png)

  - 用户自定义加载器

    ​    在Java的日常应用程序开发中，类的加载几乎是由上述3种类加载器相互配合执行的，在必要时，我们还可以自定义类加载器，来定制类的加载方式

###### 2.4 双亲委派机制

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201223165256658.png" alt="image-20201223165256658" style="zoom:50%;" />

   		 说明：这里的四者之间的关系是包含关系，不是上下层，也不是子父类的继承关系

- 工作原理
  - 1）如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行
  - 2）如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器；
  - 3）如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式。
- 优势
  - 避免了类的重复加载
  - 保护程序的安全，防止核心API被随意篡改
- 沙箱安全机制
  - 自定义java.lang.String，实现main方法，实际中String中没有main方法，(现实中不能这么做），但是在加载自定义string类的时候会率先使用引导类加载器加载，而引导类加载器在加载的过程中会先加载jdk自带的文件（rt.jar包中java\lang\string.Class），报错信息说没有main方法，就是因为加载的是rt.jar包中的string类。这样可以保证对java核心源代码的保护，这就是沙箱安全机制。

## 三、运行时数据区概述及线程

![image-20210323155838146](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323155838146.png)

Java和C++之间有一堵由内存动态分配和垃圾回收技术所围成的高墙，墙外面的人想进去，墙里面的人想出来；

每个线程：独立包含程序设计器、虚拟机栈、本地方法栈；

线程之间共享：堆、堆外内存（永久代或元空间，即方法区）；

每个JVM只有一个Runtime实例，即为运行时环境。

#### 3.1概述

###### 3.1.1 内存的概念

- 内存是非常重要的系统资源，是硬盘和CPU的中间仓库及桥梁，承载着操作系统和应用程序的实时运行。JVM内存布局规定了Java在运行过程中内存申请、分配、管理的策略，保证了JVM的高效稳定运行。不同的JVM对于内存的划分方式和管理机制存在着部分差异。这里讲的是Hotspot虚拟机。J9和JRockit虚拟机没有方法区

###### 3.1.2 线程的概念

- 线程是一个程序里的运行单元。JVM允许一个应用有多个程并行的执行。
- 在Hotspot JVM里，每个线程都与操作系统的本地线程直接映射。当一个Java线程准备好执行以后，此时一个操作系统的本地线程也同时创建。Java线程执行终止后，本地线程也会回收。
- 操作系统负责所有线程的安排调度到任何一个可用的CPU上。一旦本地线程初始化成功，它就会调用Java线程中的run（）方法。

#### 3.2 程序计数器（PC寄存器）

- 介绍：JVM中的程序计数器并非广义上所指的物理寄存器，不同于CPU的寄存器，它是对物理PC寄存器的一种抽象模拟。

  - 它是一块很小的内存空间，几乎可以忽略不记。也是运行速度最快的存储区域。

  - 在JVM规范中，每个线程都有它自己的程序计数器，是线程私有的，生命周期与线程的生命周期保持一致。

  - 任何时间一个线程都只有一个方法在执行，也就是所谓的当前方法。程序计数器会存储当前线程正在执行的Java方法的JVM指令地址；或者，如果是在执行native方法，则是未指定值（undefned）。说明：native方法指的是 java调用非java代码的接口，即该方法的实现由非java语言实现

    

- **作用**：PC寄存器用来存储指向下一条指令的地址，也就是将要执行的指令代码。由执行引擎读取下一条指令。

  ​		                                     <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323174614893.png" alt="image-20210323174614893" style="zoom:50%;" />

  - 它是程序控制流的指示器，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。
  - 字节码解释器（执行引擎）工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令。
  - **它是唯一一个在Java虚拟机规范中没有规定任何OutofMemoryError情况的区域，也不存在GC垃圾回收问题。**

- 面试题1：使用PC寄存器存储字节码指令地址有什么用呢？为什么使用PC寄存器记录当前线程的执行地址呢？（属于同一个问题）
  - 因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪开始继续执行。
  - JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令

- 面试题2：PC寄存器为什么会被设定为线程私有？
  
  - 多线程在一个特定的时间段内只会执行其中某一个线程的方法，CPU会不停地做任务切换，这样必然导致经常中断或恢复，为了能够准确地记录各个线程正在执行的当前字节码指令地址，给每一个线程都分配一个PC寄存器，这样一来各个线程之间便可以进行独立计算，从而不会出现相互干扰的情况。

#### 3.3 虚拟机栈

##### 1.虚拟机栈的概述

- Java虚拟机栈是什么
  - Java虚拟机栈（Java virtual Machine stack），早期也叫Java栈。每个线程在创建时都会创建一个虚拟机栈，其内部保存一个个的栈帧（stack Frame），对应着一次次的Java方法调用。
  - 是线程私有的，生命周期和线程一致

- **作用**：主管Java程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。

- 栈的特点：
  - 栈是一种快速有效的分配存储方式，访问速度仅次于程序计数器。
  - JVM直接对Java栈的操作只有两个：
    - 每个方法执行，伴随着进栈（入栈、压栈）
    - 执行结束后的出栈工作
  - **对于栈来说不存在垃圾回收问题**


- **设置栈内存大小**：我们可以使用参数-Xss选项来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度。

###### 面：开发中虚拟机栈遇到的异常有哪些？栈中可能出现的异常

- Java虚拟机规范允许Java栈的大小是动态的或者是固定不变的。
  - 如果采用固定大小的Java虚拟机栈，那每一个线程的Java虚拟机栈容量可以在线程创建的时候独立选定。如果线程请求分配的栈容量超过Java虚拟机栈允许的最大容量，Java虚拟机将会抛出一个**StackoverflowError异常**。
  - 如果Java虚拟机栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的虚拟机栈，那Java虚拟机将会**抛出一个OutofMemoryError异常**。

##### 2.栈的存储单位

###### ①栈中存储什么

- 每个线稈都有自己的栈，栈中的数据都是**以栈帧（stack Frame）的格式存在**。
- 在这个线程上正在执行的**每个方法都各自对应一个栈帧**（stack Frame）
- 栈帧是一个内存区块，是一个数据集，维系着方法执行过程中的各种数据信息。

###### ②栈运行原理

- JVM直接对Java栈的操作只有两个，就是对栈帧的压栈和出栈，遵循“先进后出”/“后进先出”原则。
- 在一条活动线程中，一个时间点上，只会有一个活动的栈帧。即只有当前正在执行的方法的栈帧（栈顶栈帧）是有效的，这个栈帧被称为当前栈帧（current Frame），与当前栈帧相对应的方法就是当前方法（current Method），定义这个方法的类就是当前类（current class）
- 执行引擎运行的所有字节码指令只针对当前栈帧进行操作。
- 如果在该方法中调用了其他方法，对应的新的栈帧会被创建出来，放在栈的顶端，成为新的当前帧。
- **不同线程中所包含的栈帧是不允许存在相互引用**的，即不可能在一个栈帧之中引用另外一个线程的栈帧。
- 如果当前方法调用了其他方法，方法返回之际，当前栈帧会传回此方法的执行结果给前一个栈帧，接着，虚拟机会丢弃当前栈帧，使得前一个栈帧重新成为当前栈帧。
- Java方法有两种返回函数的方式，**一种是正常的函数返回，使用return指令；另外一种是抛出异常。不管使用哪种方式，都会导致栈帧被弹出**。

###### ③栈帧的内部结构

   每个栈帧中存储着：

- **局部变量表**（Local variables）

- **操作数栈**（operand stack）（或表达式栈）

- 动态链接（Dynamic Linking）（或指向运行时常量池的方法引用）

- 方法返回地址（Return Address）（或方法正常退出或者异常退出的定义）

- 一些附加信息

  <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323163359348.png" alt="image-20210323163359348" style="zoom: 50%;" />

##### 3.局部变量表

- 局部变量表也被称之为局部变量数组或本地变量表

- **定义为一个数字数组，主要用于存储方法参数和定义在方法体内的局部变量**，这些数据类型包括各类基本数据类型、对象引用（reference），以及returnAddress类型。

- 由于局部变量表是建立在线程的栈上，是线程的私有数据，因此**不存在数据安全问题**

- **局部变量表所需的容量大小是在编译期确定下来的**，并保存在方法的Code属性的maximum local variables数据项中。在方法运行期间是不会改变局部变量表的大小的。

- **方法嵌套调用的次数由栈的大小决定**。一般来说，**栈越大，方法嵌套调用次数越多**。对一个函数而言，它的参数和局部变量越多，使得局部变量表膨胀，它的栈帧就越大，以满足方法调用所需传递的信息增大的需求。进而函数调用就会占用更多的栈空间，导致其嵌套调用次数就会减少。

- **局部变量表中的变量只在当前方法调用中有效**。在方法执行时，虚拟机通过使用局部变量表完成参数值到参数变量列表的传递过程。**当方法调用结束后，随着方法栈帧的销毁，局部变量表也会随之销毁**。

- **局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收**。

- 关于Slot的理解

  - 参数值的存放总是在局部变量数组的index0开始，到数组长度-1的索引结束。

  - 局部变量表，最基本的存储单元是Slot（变量槽）

  - 局部变量表中存放编译期可知的各种基本数据类型（8种），引用类型
    （reference），returnAddress类型的变量。

  - 在局部变量表里，32位以内的类型只占用一个slot（包括returnAddress类型），64位的类型（long和double）占用两个slot.

    - byte、short、char在存储前被转换为int，boolean也被转换为int，0表示false，非0表示true.
    - long和double则占据两个slot.

  - JVM会为局部变量表中的每一个S1ot都分配一个访问素引，通过这个素引即可成功访问到局部变量表中指定的局部变量值

  - 当一个实例方法被调用的时候，它的方法参数和方法体内部定义的局部变量将会按照顺序被复制到局部变量表中的每一个slot上

  - 如果需要访问局部变量表中一个64bit的局部变量值时，只需要使用前一个索引即可。（比如：访问long或double类型变量）

  - 如果当前帧是由构造方法或者实例方法创建的，那么该对象引用this将会存放在index为0的slot处，其余的参数按照参数表顺序继续排列。

    <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323173011029.png" alt="image-20210323173011029" style="zoom: 50%;" />

  - **栈帧中的局部变量表中的槽位是可以重用的**，如果一个局部变量过了其作用域，那么在其作用域之后申明的新的局部变量就很有可能会复用过期局部变量的槽位，从而**达到节省资源的目的**。

    <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323173636580.png" alt="image-20210323173636580" style="zoom:50%;" />

  - 静态变量和局部变量的对比：

    - 我们知道类变量表有两次初始化的机会，第一次是在“**准备阶段**”，执行系统初始化，对类变量设置零值，另一次则是在“**初始化**”阶段，赋予程序员在代码中定义的初始值。
    - 和类变量初始化不同的是，局部变量表不存在系统初始化的过程，这意味着一旦定义了局部变量则必须**人为的初始化**，否则无法使用。


##### 4.操作数栈

- 操作数栈，**主要用于保存计算过程的中间结果，同时作为计算过程中变量临时的存储空间**。
- 操作数栈就是JVM执行引擎的一个工作区，当一个方法刚开始执行的时候，一个新的栈帧也会随之被创建出来，这个方法的操作数栈是空的。
- 每一个操作数栈都会拥有一个明确的栈深度用于存储数值，其所需的最大深度在编译期就定义好了，保存在方法的Code属性中，为max_stack的值。
- 栈中的任何一个元素都是可以任意的Java数据类型。
  - 32bit的类型占用一个栈单位深度
  - 64bit的类型占用两个栈单位深度
- 操作数栈**并非采用访问索引的方式来进行数据访问**的，而是只能通过标准的入栈（push）和出栈（pop）操作来完成一次数据访问。
- **如果被调用的方法带有返回值的话，其返回值将会被压入当前栈帧的操作数栈中**，并更新PC寄存器中下一条需要执行的字节码指令。
- 另外，我们说Java虚拟机的**执行引擎是基于栈的执行引擎，其中的栈指的就是操作数栈。**

##### 5.动态链接

- 每一个栈帧内部都包含一个指向**运行时常量池中该栈帧所属方法的引用**。包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接
  （Dynamic Linking）。比如：invokedynamic指令
- 在Java源文件被编译到**字节码文件中时，所有的变量和方法引用都作为符号引用（Symbolic Reference）保存在class文件的常量池里**。比如：描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示的，那么**动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用**。
- 为什么需要常量池？
  **常量池的作用，就是为了提供一些符号和常量，便于指令的识别**。

##### 6.方法返回地址

- 存放调用该方法的pc寄存器的值。（个人理解：A调用B，A的PC寄存器的值保存到B的方法返回地址中，当B方法结束时，返回该地址值，执行引擎根据这个找到A方法继续执行的位置）
- 个方法的结束，有两种方式：
  - 正常执行完成
  - 出现未处理的异常，非正常退出

- 无论通过哪种方式退出，在方法退出后都**返回到该方法被调用的位置**。方法正常退出时，**调用者的pc计数器的值作为返回地址，即调用该方法的指令的下一条指令的地址**。而通过异常退出的，返回地址是要通过异常表来确定，栈帧中一般不会保存这部分信息。
- 本质上，方法的退出就是当前栈帧出栈的过程。此时，需要恢复上层方法的局部变量表、操作数栈、将返回值压入调用者栈帧的操作数栈、设置PC寄存器值等，让调用者方法继续执行下去。
- **正常完成出口和异常完成出口的区别在于：通过异常完成出口退出的不会给他的上层调用者产生任何的返回值**。

#### 3.4 本地方法接口

- 什么是本地方法接口

  ​    一个Native Method就是一个Java调用非Java代码的接口。**本地接口的作用是融合不同的编程语言为Java所用**。

- 为什么使用本地方法接口？

  ​      Java使用起来非常方便，然而有些层次的任务用Java实现起来不容易，或者使用Java执行效率较低，尤其是与操作系统或底层硬件交互时，使用本地方法接口就可以使得Java与底层结构高效的进行信息交流。

#### 3.5 本地方法栈

- **Java虚拟机栈用于管理Java方法的调用，而本地方法栈用于管理本地方法的调用**。
- 本地方法栈，是**线程私有**的。
- 允许被实现成固定或者是可动态扩展的内存大小。（在**内存溢出方面是相同的**）
  - 如果线程请求分配的栈容量超过本地方法栈允许的最大容量，Java虚拟机将会抛出一个stackoverflowError异常。
  - 如果本地方法栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的本地方法栈，那么Java虚拟机将会抛出一个OutofMemoryError异常。
- 本地方法是使用c语言实现的。
- 它的具体做法是Native Method Stack中登记native方法，在Execution Engine执行时加载本地方法库。
- 在Hotspot JVM中，直接将本地方法栈和虚拟机栈合二为一。

#### 3.6 堆

##### 1.堆的核心概述

- 一个JVM家例只存在一个堆内存，堆也是Java内存管理的核心区域。

- **Java堆区在JVM启动的时候即被创建**，其空间大小也就确定了。是JVM管理的最大一块内存空间。

  - 堆内存的大小是可以调节的。

- 《Java虚拟机规范》规定，堆可以处于物理上不连续的内存空间中，但在逻辑上它应该被视为连续的。

- 所有的线程共享Java堆，在这里还可以划分**线程私有的缓冲区（Thread Local Allocation Buffer，TLAB）**

- 《Java虚拟机规范》中对Java堆的描述是：**所有的对象实例以及数组**都应当在运行时分配在堆上。

- 在方法结束后，堆中的对象不会马上被移除，仅仅在垃圾收集的时候才会被移除。

- 堆，是GC（Garbage Collection，垃圾收集器）执行垃圾回收的重点区域。

  ![image-20210328143251816](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210328143251816.png)

##### 2.设置堆内存大小与OOM

- Java堆区用于存储Java对象实例，那么堆的大小在JVM启动时就已经设定好了，大家可以通过选项"-Xmx"和"-Xms"来进行设置。
  - "-Xms"用于表示堆区的起始内存，等价于-XX：InitialHeapsize
  - "-Xmx"则用于表示堆区的最大内存，等价于-XX:MaxHeapSize一旦堆区中的内存大小超过"-Xmx"所指定的最大内存时，将会抛出OutOfMemoryError异常。
- 通常会将-Xms和-Xmx两个参数配置相同的值，其**目的是为了能够在java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小，从而提高性能。**
- 默认情况下，
  - 初始内存大小：物理电脑内存大小/64
  - 最大内存大小：物理电脑内存大小/4

##### 3.年轻代与老年代

- 存储在yVM中的Java对象可以被划分为两类：

  - 一类是生命周期较短的瞬时对象，这类对象的创建和消亡都非常迅速
  - 另外一类对象的生命周期却非常长，在某些极端的情况下还能够与JVM的生命周期保持一致。

- Java堆区进一步细分的话，可以划分为年轻代（YoungGen）和老年代（O1dGen）

- 其中年轻代又可以划分为Eden空间、Survivor0空间和Survivor1空间（有时也叫做from区、to区，其中两个Survivor区角色总是互换，谁空谁是to区）。

  ![image-20210328144429996](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210328144429996.png)

- 配置新生代与老年代在堆结构的占比。
  - 默认-XX：NewRatio=2，表示新生代占1，老年代占2，新生代占整个堆的1/3
  - 可以修改-XX：NewRatio=4，表示新生代占1，老年代占4，新生代占整个堆的1/5

- 在HotSpot中，Eden空间和另外两个Survivor空间缺省所占的比例是8：1：1
- 开发人员可以通过选项"-XX：SurvivorRatio"调整这个空间比例。比如-xx：SurvivorRatio=8
- **几乎所有的**Java对象都是在Eden区被new出来的。
- 绝大部分的Java对象的销毁都在新生代进行了
  
  - IBM公司的专门研究表明，新生代中80%的对象都是“朝生夕死”的。
- 可以使用选项"-Xmn"设置新生代最大内存大小
  
  - 这个参数一般使用默认值就可以了。

##### 4.图解对象分配过程

- 1.new的对象先放伊甸园区。此区有大小限制。
- 2.当伊甸园的空间填满时，程序又需要创建对象，JVM的垃圾回收器将对伊甸园区进行垃圾回收（Minor GC），将伊甸园区中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到伊甸园区。
- 3.然后将伊甸园中的剩余对象移动到幸存者0区。
- 4.如果再次触发垃圾回收，此时上次幸存下来的放到幸存者0区的对象，如果没有回收，就会放到幸存者1区。
- 5·如果再次经历垃圾回收，此时会重新放回幸存者0区，接着再去幸存者1区。
- 6．啥时候能去养老区呢？可以设置次数。默认是15次。
  - 可以设置参数：-XX:MaxTenuringThreshold-<N>进行设置

- 7.在养老区，相对悠闲。当养老区内存不足时，再次触发GC：Major GC，进行养老区的内存清理。
- 8.若养老区执行了Major GC之后发现依然无法进行对象的保存，就会产生00M异常
- 9.当老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小就会进行Minor GC，否则将进行Full GC.

##### 5.Minor GC、Major GC、Full GC

​		JVM在进行GC时，并非每次都对上面三个内存（新生代、老年代；方法区）区域一起回收的，大部分时候回收的都是指新生代。
​		针对HotSpot VM的实现，它里面的GC按照回收区域又分为两大种类型：一种是部分收集Partial GC），一种是整堆收集（Full GC）

- 部分收集：不是完整收集整个Java堆的垃圾收集。其中又分为：
  - 新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集
  - 老年代收集（Major GC/01d GC）：只是老年代的垃圾收集。
    - 目前，只有CMS GC会有单独收集老年代的行为。
    - 注意，很多时候Major Gc会和Ful1 GC混淆使用，需要具体分辨是老年代回收还是整堆回收。
  - 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集。
    - 目前，只有G1 GC会有这种行为
- 整堆收集（Full GC）：收集整个java堆和方法区的垃圾收集。

- **年轻代GC（Minor GC）触发机制**：
  - 当年轻代空间不足时，就会触发Minor GC，这里的年轻代满指的是Eden代满，**Survivor满不会引发GC**。（每次Minor GC会清理年轻代的内存。
  - 因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。
  - Minor GC会引发STW，暂停其它用户的线程，等垃圾回收结束，用户线程才恢复运行。

- **老年代GC**（Major GC/Ful1 GC）**触发机制**：
  - 指发生在老年代的GC，对象从老年代消失时，我们说"Major GC"或"Full GC"发生了
  - 出现了Major GC，经常会伴随至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）。
    - 也就是在老年代空间不足时，会先尝试触发Minor GC。如果之后空间还不足则触发Major GC
  - Major GC的速度一般会比Minor GC慢10倍以上，STW的时间更长。
  - 如果Major GC后，内存还不足，就报OOM了。

- **Full GC触发机制**：
  触发Ful1 Gc执行的情况有如下五种：
  - （1）调用System.gc（）时，系统建议执行Ful1 GC，但不是必然执行的
  - （2）老年代空间不足
  - （3）方法区空间不足
  - （4）通过Minor GC后进入老年代的平均大小大于老年代的可用内存
  - （5）由Eden区、Survivor spacee（From Space）区向survivor space1（To Space）区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小。
  - 说明：full gc是开发或调优中尽量要避免的。这样暂时时间会短一些。

##### 6.堆空间的分代思想

- 为什么需要把Java堆区分代，不分代就不能正常工作了吗？

  ​		因为Java堆中的对象大部分都是朝生夕死的对象，生命周期短。如果不进行分代，所有的对象都放在一起，每次进行GC的时候，就要进行全堆扫描，耗费大量时间。如果进行分代，将生命周期短的对象放在新生代，将生命周期长的对象放在老年代，进行分代垃圾回收。GC性能得到很大的优化。

##### 7.内存分配策略

- 如果对象在Eden出生并经过第一次MinorGC后仍然存活，并且能被survivor容纳的话，将被移动到Survivor空间中，并将对象年龄设为1。对象在survivor区中每熬过一次MinorGC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁，其实每个JVM、每个GC都有所不同）时，就会被晋升到老年代中。
- 对象晋升老年代的年龄阈值，可以通过选项-XX：MaxTenuringThreshold来设置。

##### 8.为对象分配内存：TLAB

###### 1.为什么有TLAB（Thread Local Allocation Buffer）？

- 堆区是线程共享区域，任何线程都可以访问到堆区中的共享数据
- 由于对象实例的创建在JVM中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的
- 为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度。

###### 2.什么是TLAB？

- 对Eden区域继续进行划分，JVM为每个线程分配了一个私有缓存区域，它包含在Eden空间内。
- 多线程同时分配内存时，使用TLAB可以避免一系列的非线程安全问题，同时还能够提升内存分配的吞吐量，因此我们可以将这种内存分配方式称之为**快速分配策略**。
  

###### 3.关于TLAB的说明：

- 尽管不是所有的对象实例都能够在TLAB中成功分配内存，但JVM确实是将**TLAB作为内存分配的首选**。
- 在程序中，开发人员可以通过选项"-XX:UseTLAB"设置是否开启TLAB空间。默认情况下，TLAB空间的内存非常小，仅占有整个Eden空间的1%，
- 当然我们可以通过选项"-XX：TLABWasteTargetPercent"设置TLAB空间所占用Eden空间的百分比大小。
- 一旦对象在TLAB空间分配内存失败时，JVM就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在Eden空间中分配内存。

###### 4.TLAB的作用

- TLAB中的对象并不是线程私有的，这个技术只是为了解决在给对象分配内存时，不需要加锁，普通的Eden区分配内存是需要加锁的；TLAB在Eden中，是可以被其他线程访问的，GC的时候也会被回收；至于为什么内存分配需要加锁，因为多线程可能竞争同一块内存空间，而那一块内存空间只能分配给一个对象，加锁是乐观锁，CAS+失败重试，提高性能，由此可见TLAB对性能提高很大

##### 9.堆空间的参数设置

- -XX：+PrintFlagsInitial：查看所有的参数的默认初始值
- -XX：+PrintFlagsFinal：查看所有的参数的最终值（可能会存在修改，不再是初始值）
- -Xms：初始堆空间内存（默认为物理内存的1/64）
- -Xmx：最大堆空间内存（默认为物理内存的1/4）
- -Xmn：设置新生代的大小。（初始值及最大值）
- -XX：NewRatio：配置新生代与老年代在堆结构的占比
- -XX：SurvivorRatio：设置新生代中Eden和s0/S1空间的比例
- -XX：MaxTenuringThreshold：设置新生代垃圾的最大年龄
- -XX：+PrintGCDetails：输出详细的GC处理日志
  - 打印qc简要信息：①-XX：+PrintGC      ②-verbose：gc
    

##### 10.堆是分配对象的唯一选择嘛

###### 1.什么是逃逸分析

- 逃逸分析的实现是基于分析对象动态的作用域：

  - 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸。
  - 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸。例如作为调用参数传递到其他地方中。

  ![image-20210329092021827](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329092021827.png)

  上述对象没有发生逃逸的对象，则可以分配到栈上，随着方法执行的结束，栈空间就被移除。

  ![image-20210329092114343](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329092114343.png)

###### 2.逃逸分析：代码优化

- 一、**栈上分配**。将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配。

  ​		JIT编译器在编译期间根据逃逸分析的结果，发现如果一个对象并没有逃逸出方法的话，就可能被优化成栈上分配。分配完成后，继续在调用栈内执行，最后线程结束，栈空间被回收，局部变量对象也被回收。这样就无须进行垃圾回收了。

- 二、**同步省略**。如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步。

  ​		线程同步的代价是相当高的，同步的后果是降低并发性和性能。
  ​		在动态编译同步块的时候，JIT编译器可以借助逃逸分析来**判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程**。如果没有，那么JIT编译器在编译这个同步块的时候就会取消对这部分代码的同步。这样就能大大提高并发性和性能。这个取消同步的过程就叫同步省略，也叫**锁消除**。

- 三、**分离对象或标量替换**。有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。

  ​		在JIT阶段，如果经过逃逸分析，发现一个对象不会被外界访问的话，那么经过JIT优化，就会把这个对象拆解成若干个其中包含的若干个成员变量来代替。这个过程就是**标量替换**。

###### 3.结论

- 注意到有一些观点，认为通过逃逸分析，JVM会在栈上分配那些不会逃逸的对象，这在理论上是可行的，但是取决于JVM设计者的选择。据我所知，Oracle Hotspot JVM中并未这么做，这一点在逃逸分析相关的文档里已经说明，所以可以明确**所有的对象实例都是创建在堆上**。
- 目前很多书籍还是基于JDK 7以前的版本，JDK已经发生了很大变化，intern字符串的缓存和静态变量曾经都被分配在永久代上，而永久代已经被元数据区取代。但是，intern字符串缓存和静态变量并不是被转移到元数据区，而是直接在堆上分配，所以这一点同样符合前面一点的结论：**对象实例都是分配在堆上**。

##### 11.本章小结

- 年轻代是对象的诞生、成长、消亡的区域，一个对象在这里产生、应用，最后被垃圾回收器收集、结束生命。
- 老年代放置长生命周期的对象，通常都是从Survivor区域筛选拷贝过来的Java对象。当然，也有特殊情况，我们知道普通的对象会被分配在TLAB上；如果对象较大，JVM会试图直接分配在Eden其他位置上；如果对象太大，完全无法在新生代找到足够长的连续空闲空间，JVM就会直接分配到老年代。
- 当GC只发生在年轻代中，回收年轻代对象的行为被称为MinorGC。当GC发生在老年代时则被称为MajorGC或者FullGC。一般的，MinorGC的发生频率要比MajorGC高很多，即老年代中垃圾回收发生的频率将大大低于年轻代。

#### 3.7 方法区

##### 1.堆、栈、方法区的交互关系

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329093953452.png" alt="image-20210329093953452" style="zoom: 50%;" />

##### 2.方法区的理解

- 方法区（Method Area）与Java堆一样，是各个**线程共享**的内存区域。**方法区看作是独立于Java堆的内存空间。**

  <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329094631833.png" alt="image-20210329094631833" style="zoom: 50%;" />![image-20210329094654027](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329094654027.png)

  <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329094722019.png" alt="image-20210329094722019" style="zoom:50%;" />

- **方法区在JVM启动的时候被创建**，并且它的实际的物理内存空间中和Java堆区一样都可以是不连续的。

- 方法区的大小，跟堆空间一样，可以选择固定大小或者可扩展。

- 方法区的大小决定了系统可以保存多少个类，如果系统定义了太多的类，导致方法区溢出，虚拟机同样会抛出内存溢出错误：**java.lang.OutofMemoryError：PermGen space**或者**java.lang.OutOfMemoryError：Metaspace**

  - **加载大量的第三方的jar包；Tomcat部署的工程过多（30-50个），大量动态的生成反射类**

- **关闭JVM就会释放这个区域的内存**。

- 元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代最大的**区别**在于：**元空间不在虚拟机设置的内存中，而是使用本地内存。**

##### 3.设置方法区大小与OOM

- 设置方法区大小：方法区大小不是固定的，JVM根据应用的需要动态调整

  - ![image-20210329101312950](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329101312950.png)

  - **jdk8及以后**：
    - 默认值依赖于平台。windows下，**-XX:Metaspacesize是21M，-XX:MaxMetaspaceSize的值是-1，即没有限制。**
    - 与永久代不同，如果不指定大小，**默认情况下，虚拟机会耗尽所有的可用系统内存**。如果元数据区发生溢出，虚拟机一样会抛出异常OutofMemoryError：Metaspace
    - -XX:Metaspacesize：设置初始的元空间大小。对于一个64位的服务器端JVM来说，其默认的-XX:Metaspacesize值为21MB，这就是初始的高水位线，**一旦触及这个水位线，Full GC将会被触发并卸载没用的类（即这些类对应的类加载器不再存活）**，然后这个高水位线将会重置。新的高水位线的值取决于GC后释放了多少元空间。如果释放的空间不足，那么在不超过MaxMetaspacesize时，适当提高该值。如果释放空间过多，则适当降低该值。
    - 如果初始化的高水位线设置过低，上述高水位线调整情况会发生很多次。通过垃圾回收器的日志可以观察到Full GC多次调用。为了避免频繁地GC，建议将-XX:Metaspacesize设置为一个相对较高的值。

  

- 如何解决这些OOM
  - 1、要解决O0M异常或heap space的异常，一般的手段是首先通过内存映像分析工具如Eclipse Memory Analyzer）对dump出来的堆转储快照进行分析，**重点是确认内存中的对象是否是必要的，也就是要先分清楚到底是出现了内存泄漏（Memory Leak）还是内存溢出（Memory overflow）**
  - 2、如果是内存泄漏，可进一步通过工具查看泄漏对象到Gc Roots的引用链。于是就能找到泄漏对象是通过怎样的路径与GC Roots相关联并导致垃圾收集器无法自动回收它们的。掌握了泄漏对象的类型信息，以及GC Roots引用链的信息，就可以比较准确地定位出泄漏代码的位置。
  - 3、如果不存在内存泄漏，换句话说就是内存中的对象确实都还必须存活着，那就应当检查虚拟机的堆参数（-Xmx与-Xms），与机器物理内存对比看是否还可以调大，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗。让在不没有难学的

##### 4.方法区的内部结构

- 了解变量在方法区、栈内存、堆内存中的分配要了解两部分内容，一个是“**变量在内存中的分配**”，另一个是“**变量所引用的数据在内存中的分配**”。以下简称为“变量分配”与“数据分配”。

  - 原始数据类型变量：
    原始数据类型变量的“变量分配”与“数据分配”是在一起的（**都在方法区或栈内存或堆内存**）

  - 引用数据类型变量：
    引用数据类型变量的“变量分配”与“数据分配”不一定是在一起的，（**变量所引用的数据都在堆中，而变量在各自内存中分配**）

- 方法区用于存储已被虚拟机加载的**类型信息、方法信息、属性信息、运行时常量池、JIT及时编译器编译后的代码缓存、常量**等，注意：**静态变量和字符串常量池在堆中**。

  <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329114225369.png" alt="image-20210329114225369" style="zoom:50%;" />

  - **类型信息**
    对每个加载的类型（类clajs、接口interface、枚举enum、注解annotation），JVM必须在方法区中存储以下类型信息：
    - ①这个类型的完整有效名称（全类名=包名.类名）
    - ②这个类型直接父类的完整有效名（对于interface或是java.lang.object，都没有父类）
    - ③这个类型的修饰符（public，abstract，final的某个子集）
    - ④这个类型直接接口的一个有序列表

  - **域（Field）信息**
    - JVM必须在方法区中保存类型的所有域的相关信息以及域的声明顺序。
    - 域的相关信息包括：域名称、域类型、域修饰符（public，private，protected，static，final，volatile，transient的某个子集）

  - **方法（Method）信息**
    JVM必须保存所有方法的以下信息，同域信息一样包括声明顺序方法名称
    - ①方法的返回类型（或void）
    - ②方法参数的数量和类型（按顺序）
    - ③方法的修饰符（public，private，protected，static，final，synchronized，native，abstract的一个子集）
    - ④方法的字节码（bytecodes）、操作数栈、局部变量表及大小（abstract和native方法除外）
    - ⑤异常表（abstract和native方法除外）
      - 每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引

  - **non-final的类变量**
    - 静态变量和类关联在一起，随着类的加载而加载，它们成为类数据在逻辑上的一部分。类变量被类的所有实例共享，即使没有类实例时你也可以访问它。
    - 补充说明：**全局常量：static fina**l   被声明为final的类变量的处理方法则不同，每个全局常量在编译的时候就会被分配了。

  - **运行时常量池** VS **常量池**

    ​           说明：一个有效的字节码文件中包含类的版本信息、字段、方法以及接口等描述信息，（**通过类的加载器加载到方法区的类型信息、域信息、方法信息等**）、此外还包含一项信息那就是常量池表（Constant Pool Table），包括各种字面量和对类型、域和方法的符号引用（**通过类加载器加载为方法区的运行时常量池**）。

    - 面：常量池中有什么
      - 数量值、字符串值、类引用、方法引用、字段引用
    - 面：为什么需要常量池
      - 一个java源文件中的类、接口，编译后产生一个字节码文件。而Java中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，而是存到常量池，这个字节码包含了指向常量池的引用。在动态链接的时候会用到运行时常量池相关引用。

    - 运行时常量池（Runtime Constant Pool）**是方法区的一部分**。
    - 常量池表（Constant Pool Table）是Class文件的一部分，**用于存放编译期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中**。
    - 运行时常量池，在加载类和接口到虚拟机后，就会创建对应类和接口的运行时常量池。
    - JVM为每个已加载的类型（类或接口）都维护一个常量池。池中的数据项像数组项一样，是通过**索引访问**的，例如“#4”。
    - 运行时常量池中包含多种不同的常量，包括编译期就已经明确的数值字面量，也包括到运行期解析后才能够获得的方法或者字段引用。**此时不再是常量池中的符号地址了，这里换为真实地址**。
    - 运行时常量池，相对于Class文件常量池的另一重要特征是：具**备动态性**。执行string.intern（），就会往字符串常量池中添加字符串
    - 当创建类或接口的运行时常量池时，如果构造运行时常量池所需的内存空间超过了方法区所能提供的最大值，则JVM会抛outofMemoryError异常。

##### 5.方法区的使用举例（忽略）

##### 6.方法区的演进细节

###### ①.Hotspot中方法区的演进细节

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329111922421.png" alt="image-20210329111922421" style="zoom:50%;" />

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329112042984.png" alt="image-20210329112042984" style="zoom:50%;" />

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329112105801.png" alt="image-20210329112105801" style="zoom:50%;" />

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329112149951.png" alt="image-20210329112149951" style="zoom:50%;" />

###### ②.面：永久代为什么要被元空间替换

- 因为永久代设置空间大小是很难确定的。在某些场景下，如果动态加载类过多，容易产生Perm区的OOM。元空间的最大可分配空间就是系统可用内存空间。因此，默认情况下，元空间的大小仅受本地内存限制。不容易产生OOM。

###### ③.面：String Table为什么要调整

- jdk7中将stringTable从运行时常量池放到了堆空间中。因为永久代的回收效率很低，在Full GC的时候才会触发。而Full GC是老年代的空间不足、永久代不足时才会触发。这就导致StringTable回收效率不高。而我们开发中会有大量的字符串被创建，回收效率低，导致永久代内存不足。放到堆里，能及时回收内存。

###### ④.静态变量指向的对象实体

​		    只要是对象实例必然会分配在堆中

##### 7.方法区的垃圾回收

- 方法区的垃圾收集主要回收两部分内容：**常量池中废弃的常量和不再使用的类型**。
- HotSpot虚拟机对常量池的回收策略是很明确的，**只要常量池中的常量没有被任何地方引用，就可以被回收**。
  回收废弃常量与回收Java堆中的对象非常**类似**。
- 判定一个类型是否属于“不再被使用的类”的条件就比较苛刻了。需要同时满足下面三个条件：
  - 该类所有的实例都已经被回收，也就是Java堆中不存在该类及其任何派生子类的实例。
  - 加载该类的类加载器已经被回收，这个条件除非是经过精心设计的可替换类加载器的场景，否则通常是很难达成的。
  - 该类对应的java.1ang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。
  - 就算同时满足上述条件也仅仅是允许被回收，还得看具体实现

##### 8.总结

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329104907270.png" alt="image-20210329104907270" style="zoom: 50%;" />

##### 9.常见面试题

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329104930195.png" alt="image-20210329104930195" style="zoom:50%;" />

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329104944781.png" alt="image-20210329104944781" style="zoom:50%;" />

#### 3.8 运行时数据区各区域的OOM、GC情况

- 程序计数器     不存在OOM；不存在GC垃圾回收器（内存最小，运行最快）
- 虚拟机栈         存在StackOverflowError、OOM；不存在GC垃圾回收器，因为栈只有出栈、入栈操作
- 本地方法栈     存在StackOverflowError、OOM；不存在GC垃圾回收器，因为栈只有出栈、入栈操作
- 堆                     存在OOM；存在GC垃圾回收器（垃圾回收重点区域）
- 方法区             存在OOM；存在GC垃圾回收器

## 四. 执行引擎

#### 4.1 对象的实例化、内存布局与访问定位（面试常考）

##### 1.对象的实例化

- 创建对象的方式

  <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329151245923.png" alt="image-20210329151245923" style="zoom: 67%;" />

- 创建对象的步骤

  <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329152139806.png" alt="image-20210329152139806" style="zoom: 50%;" />

  

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210118113514947.png" alt="image-20210118113514947" style="zoom: 67%;" />

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210118113332148.png" alt="image-20210118113332148" style="zoom: 67%;" />

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210118113405075.png" alt="image-20210118113405075" style="zoom: 67%;" />

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210118113258848.png" alt="image-20210118113258848" style="zoom: 67%;" />

- 补充说明：给对象的属性赋值的操作：
  ①属性的默认初始化--->  ②显式初始化/③代码块中初始化----> ④构造器中初始化

  其中①对应上述第4步；②③④对应上述第6步

##### 2.对象的内存布局

 <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329154806149.png" alt="image-20210329154806149" style="zoom:67%;" />

![image-20210329155243670](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329155243670.png)

##### 3.对象的访问定位

对象访问的方式主要有两种：

- 使用句柄访问

  - 使用句柄访问的话，Java堆中将划分一块内存作为句柄池，reference中存储的就是对象的句柄池的地址，而句柄池包含了对象实例数据与类型数据的各自具体的地址信息，如下图所示：

    <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329163620995.png" alt="image-20210329163620995" style="zoom:50%;" />

  - 优点：reference中存储的是稳定的句柄地址，在对象被移动（垃圾回收的标记压缩算法经常需要移动对象）时只需改变句柄中的实例数据指针，而reference本身不需要被修改

  

- 直接指针访问

  - 使用直接指针访问的话，Java堆中的对象的对象头汇总需包含指向对象类型数据的指针，，reference中存储的时对象地址。**HotSpot默认使用的对象访问定位**。如下图所示：

    <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329164158902.png" alt="image-20210329164158902" style="zoom:50%;" />

  - 优点：访问速度更快，节省了一次指针定位的时间开销。此外不要开辟额外的内存用于存放句柄池，节省了内存开销。

#### 4.2 直接内存

#### 4.3 执行引擎

<img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329170625379.png" alt="image-20210329170625379" style="zoom: 67%;" />

##### 1.执行引擎概述

- 执行引擎是Java虚拟机核心的组成部分之一。

  - “虚拟机”是一个相对于“物理机”的概念，这两种机器都有代码执行能力，其区别是**物理机的执行引擎是直接建立在处理器、缓存、指令集和操作系统层面上的**，而**虚拟机的执行引擎则是由软件自行实现的**，因此可以不受物理条件制约地定制指令集与执行引擎的结构体系，**能够执行那些不被硬件直接支持的指令集格式。**

- 执行引擎的作用

  - JVM的主要任务是负责**装载字节码到其内部**，但字节码并不能够直接运行在操作系统之上，执行引擎（Execution Engine）的**任务就是将字节码指令解释/编译为对应平台上的本地机器指令**。简单来说，JVM中的执行引擎充当了将高级语言翻译为机器语言的译者。

- 执行引擎的工作过程

  <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329171422079.png" alt="image-20210329171422079" style="zoom: 50%;" />

##### 2.Java代码编译和执行过程

- 什么是解释器（Interpreter），么是JIT编译器？
  - 解释器：当Java虚拟机启动时会根据预定义的规范对字节码采用逐行解释的方式执行，将每条字节码文件中的内容“翻译”为对应平台的本地机器指令执行。
  - JIT（Just In Time Compiler）编译器：就是虚拟机将源代码直接编泽成和本地机器平台相关的机器语言。

##### 3.机器码、指令、汇编语言

##### 4.解释器

##### 5.JIT编译器

#### 4.4 String Table（堆空间）

##### 4.41 String的基本特性

###### 1. 基本特性

- string：字符串使用一对""引起来表示。

- string声明为final的，不可被继承

- string实现了serializable接口：表示字符串是支持序列化的。实现了comparable接口：表示string可以比较大小

- string在jdk8及以前内部定义了final char[] value用于存储字符串数据。jdk9时改为byte[]，原因如下截图显示：

  ![image-20210226184426262](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210226184426262.png)

- 结论：String再也不用char[]来存储啦，改成了byte[]加上编码标记，节约了一些空间。此外，StringBuilder和StringBuffer底层也跟着改变

###### 2. String的不可变性

- 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值。

- 当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。

- 当调用string的replace（）方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。

- 通过字面量的方式（区别于new）给一个字符串赋值，此时的字符串值声明在字符串常量池中。

- ```java
  代码举例
  String s1 = "abc";//字面量的定义方式
  String s2 = "abc";
s1 = "hello";
  
  ```

System.out.println(s1 == s2);//比较s1和s2的地址值，false

  System.out.println(s1);//hello
System.out.println(s2);//abc

System.out.println("*****************");

  String s3 = "abc";
  s3 += "def";
  System.out.println(s3);//abcdef
System.out.println(s2);//abc

System.out.println("*****************");

  String s4 = "abc";
  String s5 = s4.replace('a', 'm');
  System.out.println(s4);//abc
  System.out.println(s5);//mbc
  ```
  
  

###### 3. String Table底层特性

- 字符串常量池中是不会存储相同 的字符串的。(使用String类的equals()比较，返回true)

- string的string Poo1是一个固定大小的Hashtable，默认值大小长度是1009（jdk6,jdk7之后维60013），如果放进string Pool的string非常多，就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用string.intern时性能会大幅下降。

- 使用-xx：stringTablesize可设置stringTable的长度在jdk6中stringTable是固定的，就是1009的长度，所以如果常量池中的字符串过多就会导致效率下降很快。stringTablesize设置没有要求 

- 在jdk7中，stringTable的长度默认值是60013，stringrablesize设置没有要求

- 从Jdk8开始，stringTable的长度默认值是60013，设置stringTable的长度的话，1009是可设置的最小值。

  

##### 4.4.2 String的内存分配

- 常量池就类似一个Java系统级别提供的缓存。8种基本数据类型的常量池都是系统协调的，**string类型的常量池比较特殊。它的主要使用方法有两种**。
  - 直接使用双引号声明出来的string对象会直接存储在字符串常量池中。比如：string info ="atguigu.com"；

  - 如果不是用双引号声明的String对象，可以使用string提供的intern（）方法。这个后面重点谈

- Java 6及以前，字符串常量池存放在永久代。
- Java 7中Oracle的工程师对字符串池的逻辑做了很大的改变，即**字符串常量池的位置调整到Java堆内**。
  - 所有的字符串都保存在堆（Heap）中，和其他普通对象一样，这样可以让你在进行调优应用时仅需要调整堆大小就可以了。

- Java8方法区又叫元空间，字符串常量池放在堆空间中

- 如何保证变量s指向的是字符串常量池中的数据呢？

  - 方式一： String s = "shkstart";//字面量定义的方式

  - 方式二： 调用intern()
    - String s = new String("shkstart").intern();
    - String s = new StringBuilder("shkstart").toString().intern();

##### 4.4.3 String的基本操作

###### 1. String实例化的不同方式

- 方式说明
  方式一：通过字面量定义的方式
  方式二：通过new + 构造器的方式

- ```java
  代码举例
  //通过字面量定义的方式：此时的s1和s2的数据javaEE声明在方法区中的字符串常量池中。
  String s1 = "javaEE";
  String s2 = "javaEE";
  //通过new + 构造器的方式:此时的s3和s4保存的地址值，是数据在堆空间中开辟空间以后对应的地址值。
  String s3 = new String("javaEE");
String s4 = new String("javaEE");
  
  System.out.println(s1 == s2);//true
  System.out.println(s1 == s3);//false
  System.out.println(s1 == s4);//false
  System.out.println(s3 == s4);//false
  ```

  

###### 2.String与其他结构的转化

- 1 . 与基本数据类型、包装类之间的转换

  - String --> 基本数据类型、包装类：调用包装类的静态方法：parseXxx(str)

  -  基本数据类型、包装类 --> String:调用String重载的valueOf(xxx)

  ```java
  @Test
  public void test1(){
      String str1 = "123";
  
   //int num = (int)str1;//错误的
      int num = Integer.parseInt(str1);
  
      String str2 = String.valueOf(num);//"123"
      String str3 = num + "";
  
      System.out.println(str1 == str3);
  }
  ```

  

- 2.与字符数组之间的转换

  - String --> char[]:调用String的toCharArray()
  - char[] --> String:调用String的构造器

   

  ```java
  @Test
  public void test2(){
  String str1 = "abc123";  //题目： a21cb3
  
  char[] charArray = str1.toCharArray();
  for (int i = 0; i < charArray.length; i++) {
      System.out.println(charArray[i]);
  }
  
  char[] arr = new char[]{'h','e','l','l','o'};
  String str2 = new String(arr);
  System.out.println(str2);
  
  }
  ```

  

- 3 .与字节数组之间的转换

  - 编码：String --> byte[]:调用String的getBytes()
  - 解码：byte[] --> String:调用String的构造器

  

  编码：字符串 -->字节  (看得懂 --->看不懂的二进制数据)
  解码：编码的逆过程，字节 --> 字符串 （看不懂的二进制数据 ---> 看得懂

  说明：解码时，要求解码使用的字符集必须与编码时使用的字符集一致，否则会出现乱码。

  ```java
  @Test
  public void test3() throws UnsupportedEncodingException {
  String str1 = "abc123中国";
  byte[] bytes = str1.getBytes();//使用默认的字符集，进行编码。
System.out.println(Arrays.toString(bytes));
  
  byte[] gbks = str1.getBytes("gbk");//使用gbk字符集进行编码。
System.out.println(Arrays.toString(gbks));
  
  ```

System.out.println("******************");

  String str2 = new String(bytes);//使用默认的字符集，进行解码。
System.out.println(str2);

  String str3 = new String(gbks);
System.out.println(str3);//出现乱码。原因：编码集和解码集不一致！

  String str4 = new String(gbks, "gbk");
  System.out.println(str4);//没出现乱码。原因：编码集和解码集一致！
  ```
  
  


  }

- 4 .与StringBuffer、StringBuilder之间的转换
  String -->StringBuffer、StringBuilder:调用StringBuffer、StringBuilder构造器
  StringBuffer、StringBuilder -->String:

  ①调用String构造器；②StringBuffer、StringBuilder的toString()

###### 3.常用方法

- ```java
  - int length()：返回字符串的长度： return value.length
  
  - char charAt(int index)： 返回某索引处的字符return value[index]
  
  - boolean isEmpty()：判断是否是空字符串：return value.length == 0
  
  - String toLowerCase()：使用默认语言环境，将 String 中的所字符转换为小写
  
  - String toUpperCase()：使用默认语言环境，将 String 中的所字符转换为大写
  
  - String trim()：返回字符串的副本，忽略前导空白和尾部空白
  
  - boolean equals(Object obj)：比较字符串的内容是否相同
  
  - boolean equalsIgnoreCase(String anotherString)：与equals方法类似，忽略大小写
  
  - String concat(String str)：将指定字符串连接到此字符串的结尾。 等价于用“+”
  
  - int compareTo(String anotherString)：比较两个字符串的大小
  
  - String substring(int beginIndex)：返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串。
  
  - String substring(int beginIndex, int endIndex) ：返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。
  
  - boolean endsWith(String suffix)：测试此字符串是否以指定的后缀结束
  
  - boolean startsWith(String prefix)：测试此字符串是否以指定的前缀开始
  
  - boolean startsWith(String prefix, int toffset)：测试此字符串从指定索引开始的子字符串是否以指定前缀开始
  
  - boolean contains(CharSequence s)：当且仅当此字符串包含指定的 char 值序列时，返回 true
  
  - 索引
  
    int indexOf(String str)：返回指定子字符串在此字符串中第一次出现处的索引
  
    int indexOf(String str, int fromIndex)：返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始
  
    int lastIndexOf(String str)：返回指定子字符串在此字符串中最右边出现处的索引
  
    int lastIndexOf(String str, int fromIndex)：返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索
      注：indexOf和lastIndexOf方法如果未找到都是返回-1
  
  - 替换：
    String replace(char oldChar, char newChar)：返回一个新的字符串，它是通过用 newChar 替换此字符串中出现的所 oldChar 得到的。
    String replace(CharSequence target, CharSequence replacement)：使用指定的字面值替换序列替换此字符串所匹配字面值目标序列的子字符串。
    String replaceAll(String regex, String replacement)：使用给定的 replacement 替换此字符串所匹配给定的正则表达式的子字符串。
    String replaceFirst(String regex, String replacement)：使用给定的 replacement 替换此字符串匹配给定的正则表达式的第一个子字符串。
  - 匹配:
    boolean matches(String regex)：告知此字符串是否匹配给定的正则表达式。
  - 切片：
    String[] split(String regex)：根据给定正则表达式的匹配拆分此字符串。
    String[] split(String regex, int limit)：根据匹配给定的正则表达式来拆分此字符串，最多不超过limit个，如果超过了，剩下的全部都放到最后一个元素中。
  ```

  

##### 4.4.4 字符串拼接操作

- 常量与常量的拼接结果在常量池，原理是编译期优化

- 常量池中不会存在相同内容的常量。

- 只要其中有一个是变量，结果就在堆中。变量拼接的原理是stringBuilder

- 如果拼接的结果调用intern（）方法，则主动将常量池中还没有的字符串对象放入池中，并返回此对象地址。

- 代码举例
  
  ```java
String s1 = "javaEE";
  String s2 = "hadoop";
  
  String s3 = "javaEEhadoop";
  String s4 = "javaEE" + "hadoop";
  String s5 = s1 + "hadoop";
String s6 = "javaEE" + s2;
  String s7 = s1 + s2;
  
  System.out.println(s3 == s4);//true
  System.out.println(s3 == s5);//false
  System.out.println(s3 == s6);//false
  System.out.println(s3 == s7);//false
  System.out.println(s5 == s6);//false
System.out.println(s5 == s7);//false
  System.out.println(s6 == s7);//false
  
  ```

String s8 = s6.intern();//返回值得到的s8使用的常量池中已经存在的“javaEEhadoop”
  System.out.println(s3 == s8);//true

  String s1 = "javaEEhadoop";
  String s2 = "javaEE";
String s3 = s2 + "hadoop";
  System.out.println(s1 == s3);//false

  final String s4 = "javaEE";//s4:常量
String s5 = s4 + "hadoop";
  System.out.println(s1 == s5);//true
  ```
  
  
  
- 底层原理：

  ![image-20210227105156689](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210227105156689.png)

- 总结
  - 字符串拼接操作不一定使用的是StringBuilder!
        如果拼接符号左右两边都是字符串常量或常量引用，则仍然使用编译期优化，即非StringBuilder的方式。
  - 通过StringBuilder的append()的方式添加字符串的效率要远高于使用String的字符串拼接方式！详情：
    - ① StringBuilder的append()的方式：自始至终中只创建过一个StringBuilder的对象
    - ② 使用String的字符串拼接方式：创建过多个StringBuilder和String的对象         
    - ③ 使用String的字符串拼接方式：内存中由于创建了较多的StringBuilder和String的对象，内存占用更大；如果进行GC，需要花费额外的时间。

##### 4.4.5   intern()的使用（重点！！！！）

如果不是用双引号声明的string对象，可以使用string提供的intern方法：intern方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。
·比如：string myInfo = new string（"I love study"）.intern（）； 

###### 1. jdk1.6，将这个字符串对象尝试放入串池。

- 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
- 如果没有，会把此对象复制一份，放入串池，并返回串池中的对象地址

###### 2. Jdk1.7起，将这个字符串对象尝试放入串池。

- 如果串池中有，则并不会放入。返回已有的串池中的对象的地址
- 如果没有，则会把对象的引用地址复制一份，放入串池，并返回串池中的引用地址

###### 3. 代码示例

- ```java
  - String s = new String("1");
    s.intern();//调用此方法之前，字符串常量池中已经存在了"1"
    String s2 = "1";
  System.out.println(s == s2);//jdk6：false   jdk7/8：false
  
  
  
    String s3 = new String("1") + new String("1");//s3变量记录的地址为：new String("11")  ,执行完上一行代码以后，字符串常量池中，是否存在"11"呢？答案：不存在！！
     s3.intern();//在字符串常量池中生成"11"。如何理解：**jdk6:创建了一个新的对象"11",也就有新的地址。jdk7:此时常量中并没有创建"11",而是创建一个指向堆空间中new String("11")的地址**
    String s4 = "11";//s4变量记录的地址：使用的是上一行代码代码执行时，在常量池中生成的"11"的地址 
  System.out.println(s3 == s4);//jdk6：false  jdk7/8：true
  
  - String s3 = new String("1") + new String("1");//new String("11")
    //执行完上一行代码以后，字符串常量池中，是否存在"11"呢？答案：不存在！！
    String s4 = "11";//在字符串常量池中生成对象"11"
    String s5 = s3.intern();
    System.out.println(s3 == s4);//false
    System.out.println(s5 == s4);//true
  
  - `String s = new String("a") + new String("b");`//new String("ab")
        //在上一行代码执行完以后，字符串常量池中并没有"ab"
  
    `String s2 = s.intern();`//jdk6中：在串池中创建一个字符串"ab"//jdk8中：串池中没有创建字符串"ab",而是创建一个引用，指向new String("ab")，将此引用返回
  
    `System.out.println(s2 == "ab")`;//jdk6:true  jdk8:true
    `System.out.println(s == "ab");`//jdk6:false  jdk8:true 
  
  - String x = "ab";
    String s = new String("a") + new String("b");//new String("ab")
     //在上一行代码执行完以后，字符串常量池中并没有"ab"
  
    String s2 = s.intern();//先在串池中找"ab"，发现已有，则s2指向和x指向的地址一样，都指向串池中的"ab"
  
    System.out.println(s2 == x);//jdk6:true  jdk8:true
    System.out.println(s == x);//jdk6:false  jdk8:false
  
  - String s1 = new String("ab");//执行完以后，会在字符串常量池中会生成"ab"         //false
    //        String s1 = new String("a") + new String("b");////执行完以后，不会在字符串常量池中会生成"ab"        //true
    s1.intern();
    String s2 = "ab";
    System.out.println(s1 == s2);
  ```

###### 4. 应用场景

- 使用intern()执行效率、空间使用上结论：对于程序中大量存在存在的字符串，尤其其中存在很多重复字符串时，使用intern()可以节省内存空间。原因：使用intern()方法就不会在堆空间维护大量的String对象，而是直接引用串池中的对象，又因为串池中字符串常量不重复，所以节省了内存

- 大的网站平台，需要内存中存储大量的字符串。比如社交网站，很多人都存储：北京市、海淀区等信息。这时候如果字符串都调用intern（）方法，就会明显降低内存的大小。

4.46 面试题：new String("aa")的底层会创建几个对象？

- new String("ab")会创建几个对象？看字节码，就知道是两个。

  - 一个对象是：new关键字在堆空间创建的

  - 另一个对象是：字符串常量池中的对象"ab"。 字节码指令：ldc

 - new String("a") + new String("b")会创建几个对象？

   - 对象1：new StringBuilder()
   - 对象2： new String("a")
   - 对象3： 常量池中的"a"
   - 对象4： new String("b")
   - 对象5： 常量池中的"b"
   - 深入剖析： StringBuilder的toString():
     - 对象6 ：new String("ab")
     - 强调一下，toString()的调用，在字符串常量池中，没有生成"ab"

##### 4.4.7 String Table的垃圾回收

-XX:+PrintStringTableStatistics          打印字符串常量池信息

-XX:+PrintGCDetials                              打印垃圾回收信息

##### 4.4.8 G1中的String去重操作

- 许多大规模的Java应用的瓶颈在于内存，测试表明，在这些类型的应用里面，Java堆中存活的数据集合差不多25%是string对象。更进一步，这里面差不多一半string对象是重复的，重复的意思是说：stringl.equals（string2）=true。堆上存在重复的string对象必然是一种内存的浪费。这个项目将在G1垃圾收集器中实现自动持续对重复的string对象进行去重，这样就能避免浪费内存。

## 五、垃圾回收

#### 5.1 垃圾回收概述

###### 5.11 什么是垃圾

- 垃圾是指**在运行程序中没有任何指针指向的对象**，这个对象就是需要被回收的垃圾。

###### 5.12 为什么需要GC

- 如果不及时对内存中的垃圾进行清理，那么，这些垃圾对象所占的内存空间会一直保留到应用程序结束，被保留的空间无法被其他对象使用。甚至可能***导致内存溢出***。
- 除了释放没用的对象，垃圾回收也可以清除内存里的记录碎片。碎片整理将所占用的堆内存移到堆的一端，***以便JVM将整理出的内存分配给新的对象。***
- 随着应用程序所应付的业务越来越庞大、复杂，用户越来越多，***没有GC就不能保证应用程序的正常进行***。而经常造成STW的GC又跟不上实际的需求，所以才会不断地尝试对GC进行优化。

###### 5.13 早期的垃圾回收

- 在早期的C/C++时代，垃圾回收基本上是手工进行酌。开发人员可以使用new关键字进行内存申请，并使用delete关键字进行内存释放。这种方式可以灵活控制内存释放的时间，但是会给开发人员带来***频繁申请和释放内存的管理负担***。倘若有一处内存区间由于程序员编码的问题忘记被回收，那么就会产生***内存泄漏***，垃圾对象永远无法被清除，随着系统运行时间的不断增长，垃圾对象所耗内存可能持续上升，直到出现***内存溢出并造成应用程序崩溃***。

###### 5.14 应该关心哪些区域的垃圾回收

- 垃圾回收器可以对年轻代回收，也可以对老年代回收，甚至是全堆和方法区的回收。

  其中，***Java堆是垃圾收集器的工作重点***。

  - 从次数上讲：

    ***频繁收集Young区***
    ***较少收集old区***
    ***基本不动Perm（永久代、元空间）区***

#### 5.2 垃圾回收相关算法

##### 5.21 垃圾标记阶段算法之***引用计数算法***

- 垃圾标记阶段：对象存活判断
- 在堆里存放着几乎所有的Java对象实例，在GC执行垃圾回收之前，首先**需要区分出内存中哪些是存活对象，哪些是已经死亡的对象**。只有被标记为己经死亡的对象，GC才会在执行垃圾回收时，释放掉其所占用的内存空间，因此这个过程我们可以称为**垃圾标记阶段**。
- 判断对象存活一般有两种方式：**引用计数算法**和**可达性分析算法**。

- **引用计数算法**（Reference Counting）：比较简单，**对每个对象保存一个整型的引用计数器属性。用于记录对象被引用的情况**。
  - 对于一个对象A，只要有任何一个对象引用了A，则A的引用计数器就加1；当引用失效时，引用计数器就减1。只要对象A的引用计数器的值为0，即表示对象A不可能再被使用，可进行回收。
  - 优点：**实现简单，垃圾对象便于辨识；判定效率高，回收没有延迟性**。
  - 缺点：
    - 它需要单独的字段存储计数器，这样的做法增加了存储空间的开销。
    - 每次赋值都需要更新计数器，伴随着加法和减法操作，这增加了时间开销。
    - 引用计数器有一个严重的问题，即无法处理循环引用的情况。这是一条致命缺陷，导致在Java的垃圾回收器中没有使用这类算法。

- 引用计数算法小结（了解即可）
  - 引用计数算法，是很多语言的资源回收选择，例如因人工智能而更加火热的Python，它更是同时支持引用计数和垃圾收集机制。
  - 具体哪种最优是要看场景的，业界有大规模实践中仅保留引用计数机制，以提高吞吐量的尝试。
  - Java并没有选择引用计数，是因为其存在一个基本的难题，也就是很难处理循环引用关系。
  - Python如何解决循环引用？
    - 手动解除：很好理解，就是在合适的时机，解除引用关系。
    - 使用弱引用weakref，weakref是Python提供的标准库，旨在解决循环引用。

##### 5.22 垃圾标记阶段算法之***可达性分析算法***（GC Roots重点）

- **可达性分析算法**（又称**跟搜索算法、追踪性垃圾收集**）：相对于引用计数算法而言，可达性分析算法不仅同样具备**实现简单和执行高效**等特点，更重要的是该算法可以**有效地解决在引用计数算法中循环引用的问题，防止内存泄漏的发生**。
- **基本思路**：
  - 可达性分析算法是以根对象集合（GC Roots）为起始点，按照从上至下的方式**搜索被根对象集合所连接的目标对象是否可达**。
  - 使用可达性分析算法后，内存中的存活对象都会被根对象集合直接或间接连接着，搜索所走过的路径称为**引用链**（Reference Chain）
  - 如果目标对象没有任何引用链相连，则是不可达的，就意味着该对象己经死亡，可以标记为垃圾对象。
  - 在可达性分析算法中，只有能够被根对象集合直接或者间接连接的对象才是存活对象。

- **GC Roots**根集合就是一组必须活跃的引用
  - 虚拟机栈中引用的对象
    - 比如：各个线程被调用的方法中使用到的参数、局部变量等。
  - 本地方法栈内JNI（通常说的本地方法）引用的对象
  - 方法区中类静态属性引用的对象，
    - 比如：Java类的引用类型静态变量
  - 方法区中常量引用的对象
    - 比如：字符串常量池（String Table）里的引用
  - 所有被同步锁synchronized持有的对象
  - Java虚拟机内部的引用。
    - 基本数据类型对应的Class对象，一些常驻的异常对象（如：NullPointerException，OutOfMemoryError），系统类加载器。
  - 反映java虚拟机内部情况的JMXBean，JVMT1中注册的回调、本地代码缓存等。
  - 除了这些固定的GC Roots集合以外，根据用户所选用的垃圾收集器以及当前回收的内存区域不同，还可以有其他对象“临时性”地加入，共同构成完整GC Roots集合。比如：分代收集和局部回收（Partial Gc）
    - 如果只针对Java堆中的某一块区域进行垃圾回收（比如：典型的只针对新生代），必须考虑到内存区域是虚拟机自己的实现细节，更不是孤立封闭的，这个区域的对象完全有可能被其他区域的对象所引用，这时候就需要一并将关联的区域对象也加入GC Roots集合中去考虑，才能保证可达性分析的准确性。
  - **小技巧**：由于Root采用栈方式存放变量和指针，所以如果一个指针，它保存了堆内存里面的对象，但是自己又不存放在堆内存里面，那它就是一个Root。

- **注意**：

  - 如果要使用可达惜分析算法来判断内存是否可回收，那么分析工作必须在一个能保障一致性的快照中进行。这点不满足的话分析结果的准确性就无法保证。

  - 这点也是导致GC进行时必须"stop The World"的一个重要原因。

    即使是号称（几乎）不会发生停顿的CMS收集器中，**枚举根节点时也是必须要停顿的**。

##### 5.23 对象的finalization机制

- Java语言提供了对象终止（finalization）机制来允许开发人员提供对象被销毁之前的自定义处理逻辑。

- 当垃圾回收器发现没有引用指向一个对象，即：垃圾回收此对象之前，总会先调用这个对象的finalize（）方法。

- 由于finalize（）方法的存在，**虚拟机中的对象一般处于三种可能的状态**。如果从所有的根节点都无法访问到某个对象，说明对象已经不再使用了。一般来说，此对象需要被回收。但事实上，也并非是“非死不可”的，这时候它们暂时处于“缓刑”阶段。**一个无法触及的对象有可能在某一个条件下“复活”自己**，如果这样，那么对它的回收就是不合理的，为此，定义虚拟机中的对象可能的三种状态（面试题）。如下：

  - **可触及的**：从根节点开始，可以到达这个对象。
  - **可复活的**：对象的所有引用都被释放，但是对象有可能在finalize（）中复活。
  - **不可触及的**：对象的finalize（）被调用，并且没有复活，那么就会进入不可触及状态。不可触及的对象不可能被复活，因为finalize（）只会被调用一次。

  ​      以上3种状态中，是由于finalize（）方法的存在，进行的区分。只有在对象不可触及时才可以被回收。

- 具体过程：

  判定一个对象obja是否可回收，至少要经历两次标记过程：

  - 1.如果对象objA到GC Roots没有引用链，则进行第一次标记。
  - 2·进行筛选，判断此对象是否有必要执行finalize（）方法
    - ①如果对象obja没有重写finalize（）方法，或者finalize（）方法已经被虚拟机调用过，则虚拟机视为“没有必要执行”，objA被判定为不可触及的
    - ②如果对象objA重写了finalize（）方法，且还未执行过，那么objA会被插入到F-Queue队列中，由一个虚拟机自动创建的、低优先级的Finalizer线程触发其finalize（）方法执行。
    - ③ **finalize（）方法是对象逃脱死亡的最后机会**，稍后GC会对F-Queue队列中的对象进行第二次标记。如果objA在finalize（）方法中与引用链上的任何一个对象建立了联系，那么在第二次标记时，objA会被移出“即将回收”集合。之后，如果对象再次出现没有引用存在的情况。在这个情况下，finalize方法不会被再次调用，对象会直接变成不可触及的状态，也就是说，一个对象的finalize方法只会被调用一次。

##### 5.24 GC Roots

**GC Roots**根集合就是一组必须活跃的引用

- 虚拟机栈中引用的对象
  - 比如：各个线程被调用的方法中使用到的参数、局部变量等。
- 本地方法栈内JNI（通常说的本地方法）引用的对象
- 方法区中类静态属性引用的对象，
  - 比如：Java类的引用类型静态变量
- 方法区中常量引用的对象
  - 比如：字符串常量池（String Table）里的引用
- 所有被同步锁synchronized持有的对象
- Java虚拟机内部的引用。
  - 基本数据类型对应的Class对象，一些常驻的异常对象（如：NullPointerException，OutOfMemoryError），系统类加载器。
- 反映java虚拟机内部情况的JMXBean，JVMT1中注册的回调、本地代码缓存等。
- 除了这些固定的GC Roots集合以外，根据用户所选用的垃圾收集器以及当前回收的内存区域不同，还可以有其他对象“临时性”地加入，共同构成完整GC Roots集合。比如：分代收集和局部回收（Partial Gc）
  - 如果只针对Java堆中的某一块区域进行垃圾回收（比如：典型的只针对新生代），必须考虑到内存区域是虚拟机自己的实现细节，更不是孤立封闭的，这个区域的对象完全有可能被其他区域的对象所引用，这时候就需要一并将关联的区域对象也加入GC Roots集合中去考虑，才能保证可达性分析的准确性。
- **小技巧**：由于Root采用栈方式存放变量和指针，所以如果一个指针，它保存了堆内存里面的对象，但是自己又不存放在堆内存里面，那它就是一个Root。



**概述**：当成功区分出内存中存活对象和死亡对象后，GC接下来的任务就是执行垃圾回收，释放掉无用对象所占用的内存空间，以便有足够的可用内存空间为新对象分配内存。目前在JVM中比较常见的三种垃圾收集算法是标记一清除算法（Mark-Sweep）、复制算法（Copying）、标记-压缩算法（Mark-Compact)

##### 5.25 垃圾清除阶段算法之***标记-清除算法***

- **执行过程**：当堆中的有效内存空间（available memory）被耗尽的时候，就会停止整个程序（也被称为stop the world），然后进行两项工作，第一项则是标记，第二项则是清除。
  - 标记：Collector从引用根节点开始遍历，标记所有被引用的对象。一般是在对象的Header中记录为可达对象。
  - 清除：Collector对堆内存从头到尾进行线性的遍历，如果发现某个对象在其Header中没有标记为可达对象，则将其回收。

- **缺点**：
  - 效率不算高
  - 在进行GC的时候，需要停止整个应用程序，导致用户体验差
  - 这种方式清理出来的空闲内存是不连续的，产生内存碎片。需要维护一个空闲列表。

- **注意**：何为清除？
           这里所谓的清除并不是真的置空，而是把需要清除的对象地址保存在空闲的地址列表里。下次有新对象需要加载时，判断垃圾的位置空间是否够，如果够，就存放。

##### 5.26 垃圾清除阶段算法之***复制算法***

- **核心思想**：将活着的内存空间分为两块，每次只使用其中一块，在垃圾回收时将正在使用的内存中的存活对象复制到未被使用的内存块中，之后清除正在使用的内存块中的所有对象，交换两个内存的角色，最后完成垃圾回收。
- **优点**：
  - 没有标记和清除过程，实现简单，运行高效
  - 复制过去以后保证空间的连续性，不会出现“碎片”问题。
- **缺点**：
  - 此算法的缺点也是很明显的，就是需要两倍的内存空间。
  - 对于G1这种分拆成为大量region的GC，复制而不是移动，意味着GC需要维护region之间对象引用关系，不管是内存占用或者时间开销也不小。

- **适用场景**：适合系统中产生大量的垃圾对象，复制算法需要复制的存活对象数量并不会太多的场景，

  ​			例如：在新生代，对常规应用的垃圾回收，一次通常可以回收70% - 99% 的内存空间。
  回收性价比很高。所以现在的商业虚拟机都是用这种收集算法回收新生代。

##### 5.27 垃圾清除阶段算法之***标记-压缩（整理）算法***

- **背景**：复制算法的高效性是建立在**存活对象少、垃圾对象多**的前提下的。这种情况在新生代经常发生，但是在老年代，更常见的情况是大部分对象都是存活对象。如果依然使用复制算法，由于存活对象较多，复制的成本也将很高。因此，**基于老年代垃圾回收的特性，需要使用其他的算法**。标记一清除算法的确可以应用在老年代中，但是该算法不仅执行效率低下，而且在执行完内存回收后还会产生内存碎片。
- **执行过程**：
  - 第一阶段和标记清除算法一样，从根节点开始标记所有被引用对象。
  - 第二阶段将所有的存活对象压缩到内存的一端，按顺序排放。之后，清理边界外所有的空间。
- **优点**：
  - 消除了标记-清除算法当中，内存区域分散的缺点，我们需要给新对象分配内存时，JVM只需要持有一个内存的起始地址即可（**指针碰撞**）。这比维护一个空闲列表显然少了许多开销。
  - 消除了复制算法当中，内存减半的高额代价。
- **缺点**：
  - 从效率上来说，标记-整理算法要低于复制算法。
  - 移动对象的同时，如果对象被其他对象引用，则还需要调整引用的地址。
  - 移动过程中，需要全程暂停用户应用程序。即：STW

- **指针碰撞**（Bump the Pointer）：
          如果内存空间以规整和有序的方式分布，即已用和未用的内存都各自一边，彼此之间维系着一个记录下一次分配起始点的标记指针，当为新对象分配内存时，只需要通过修改指针的偏移量将新对象分配在第一个空闲内存位置上，这种分配方式就叫做指针碰撞（Bump tHe Pointer）

##### 5.28 小结

三种算法的对比：

![image-20210304204410419](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210304204410419.png)

##### 5.29 分代算法

- **背景**：
  - 分代收集算法，是基于这样一个事实：不同的对象的生命周期是不一样的。因此，**不同生命周期的对象可以采取不同的收集方式，以便提高回收效率**。一般是把Java堆分为新生代和老年代，这样就可以根据各个年代的特点使用不同的回收算法，以提高垃圾回收的效率。
  - 在Java程序运行的过程中，会产生大量的对象，其中有些对象是与业务信息相关，比如**Http请求中的Session对象、线程、Socket连接**，这类对象跟业务直接挂钩，因此生命周期比较长。但是还有一些对象，主要是程序运行过程中生成的临时变量，这些对象生命周期会比较短，比如：**String对象**，由于其不变类的特性，系统会产生大量的这些对象，有些对象甚至只用一次即可回收。

- **核心思想**：

  - 在HotSpot中，基于分代的概念，GC所使用的内存回收算法必须结合年轻代和老年代各自的特点。

  - 年轻代（Young Gen）
    年轻代特点：区域相对老年代较小，对象生命周期短、存活率低，回收频繁。
    这种情况复制算法的回收整理，速度是最快的。复制算法的效率只和当前存活对象大小有关，因此很适用于年轻代的回收。而复制算法内存利用率不高的问题，通过hotspot中的两个survivor的设计得到缓解。

  - 老年代（Tenured Gen）
    老年代特点：区域较大，对象生命周期长、存活率高，回收不及年轻代频繁。
    这种情况存在大量存活率高的对象，复制算法明显变得不合适。一般是由标记-清除或者是标记-清除与标记-整理的混合实现。

    Mark阶段的开销与存活对象的数量成正比。
    sweep阶段的开销与所管理区域的大小成正相关。
    Compact阶段的开销与存活对象的数据成正比。

- **举例**：

  ​           以HotSpot中的CMS回收器为例，CMS是基于Mark-Sweep实现的，对于对象的回收效率很高。而对于碎片问题，CMS采用基于Mark-Compact算法的Serial Old回收器作为补偿措施：当内存回收不佳（碎片导致的concurrent Mode Failure时），将采用Serial Old执行Full GC以达到对老年代内存的整理。
  ​           分代的思想被现有的虚拟机广泛使用。几乎所有的垃圾回收器都区分新生代和老年代。

##### 5.30 增量收集算法、分区算法

- **背景**：

  ​         上述现有的算法，在垃圾回收过程中，应用软件将处于一种Stop The World的状态。在Stop The World状态下，应用程序所有的线程都会挂起，暂停一切正常的工作，等待垃圾回收的完成。如果垃圾回收时间过长，应用程序会被挂起很久，将严重影响用户体验或者系统的稳定性。为了解决这个问题，增量收集（Incremental Collecting）算法被提出。

- **基本思想**：

  ​        如果一次性将所有的垃圾进行处理，需要造成系统长时间的停顿，那么就可以让垃圾收集线程和应用程序线程交替执行。每次，**垃圾收集线程只收集一小片区域的内存空间，接着切换到应用程序线程。依次反复，直到垃圾收集完成**。总的来说，增量收集算法的基础仍是传统的标记-清除算法和复制算法。增量收集算法通过**对线程间冲突的妥善处理，允许垃圾收集线程以分阶段的方式完成标记、清理或复制工作**。

- **缺点**：

  ​          使用这种方式，由于在垃圾回收过程中，间断性地还执行了应用程序代码，所以能减少系统的停顿时间。但是，因为线程切换和上下文转换的消耗，会使得垃圾回收的总体成本上升，**造成系统吞吐量的下降**。

##### 5.31 分区算法

- **算法思想**：
  - 一般来说，在相同条件下，堆空间越大，一次GC时所需要的时间就越长，有关GC产生的停顿也越长。为了更好地控制GC产生的停顿时间，将一块大的内存区域分割成多个小块，根据目标的停顿时间，每次合理地回收若干个小区间，而不是整个堆空间，从而减少一次GC所产生的停顿。
  - 分代算法将按照对象的生命周期长短划分成两个部分，分区算法将整个堆空间划分成连续的不同小区间。
  - 每一个小区间都独立使用，独立回收。这种算法的好处是可以控制一次回收多少个小区间。

#### 5.3 垃圾回收相关概念

###### 5.31 System.gc()的理解

- 在默认情况下，通过System.（）或者Runtime.getRuntime（）.gc（）的调用，会显式触发Full GC，同时对老年代和新生代进行回收，尝试释放被丢弃对象占用的内存。
  然而System.gc（）调用附带一个免责声明，无法保证对垃圾收集器的调用。且不建议自己调用，因为JVM有自己的GC策略

  ![image-20210322095905012](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210322095905012.png)

###### 5.32 内存溢出与内存泄漏

- 内存溢出

  - 定义：没有空闲内存，并且垃圾收器也无法提供更多内存
  - 原因：
    - 1）Java虚拟机的堆内存设置不够。可能是堆的大小不合理，比如我们要处理比较可观的数据量，但是没有显式指定JVM堆大小或者指定数值偏小。我们可以通过参数-Xms、-Xmx来调整。
    - 2）代码中创建了大量大对象，并且长耐间不能被垃圾收集器收集（存在被引用），就会导致内存溢出

- 内存泄漏

  - 定义：严格来说，**只有对象不会再被程序用到了，但是GC又不能回收他们的情况，才叫内存泄漏**。但实际情况很多时候一些不太好的实践（或疏忽）会导致对象的生命周期变得很长甚至导致OOM，也可以叫做**宽泛意义上的“内存泄漏”**

  - 举例：类中一个变量定义成局部变量时，出了方法就要被回收了，如果被定义成成员变量，生命周期就会长一些，如果被定义成静态成员变量，生命周期更长，随着类的加载而加载，随着类的消亡而消亡。本来生命周期不需要定义成怎么长，被我们误定义了。当程序中存在大量的生命周期很长的对象，可以理解为宽泛意义上的内存泄漏。
  
    ![image-20210322091147533](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210322091147533.png)

###### 5.32 Stop The World

- Stop-The-World，简称STW，指的是GC事件发生过程中，会产生应用程序的停顿。停顿产生时整个应用程序线程都会被暂停，没有任何响应，有点像卡死的感觉，这个停顿称为STW.

  - 可达性分析算法中枚举根节点（GC Roots）会导致所有Java执行线程停顿。

    - 分析工作必须在一个能确保一致性的快照中进行

    - 一致性指整个分析期间整个执行系统看起来像被冻结在某个时间点上

    - 如果出现分析过程中对象引用关系还在不断变化，则分析结果的准确性无法保证

      

- 被STW中断的应用程序线程会在完成GC之后恢复，频繁中断会让用户感觉像是网速不快造成电影卡带一样，所以我们需要减少STW的发生。
- STW事件和采用哪款GC无关，所有的GC都有这个事件。
- 即时是G1也不能完全避免Stop-The-World情况发生，只能说垃圾回收器越来越优秀，回收效率越来越高，尽可能地缩短了暂停时间。
- STW是JVM在**后台自动发起和自动完成**的。在用户不可见的情况下，把用户正常的工作线程全部停掉。
- 开发中不要用System.gc（），会导致stop-the-world的发生。

###### 5.33 垃圾回收的并行与并发

- 操纵系统中的并行与并发：
  - 并发：在操作系统中，是指一个时间段中有几个程序都处于已启动运行到运行完毕之间，且这几个程序都是在同一个处理器上运行。并发不是真正意义上的“同时进行”，只是CPU把一个时间段划分成几个时间片段（时间区间），然后在这几个时间区间之间来回切换，由于CPU处理的速度非常快，只要时间间隔处理得当，即可让用户感觉是多个应用程序同时在进行。
  - 并行：当系统有一个以上CPU或者为多核CPU时，当一个CPU执行一个进程时，另一个CPU可以执行另一个进程，两个进程互不抢占CPU资源，可以同时进行，我们称之为并行（Parallel）。
  - **二者对比**：
    - 并发，指的是多个事情，在**同一时间段内同时发生**了；并行，指的是多个事情，在**同一时间点上同时发生了**。
    - 并发的多个任务之间是互相抢占资源的。并行的多个任务之间是不互相抢占资源的。
    - 只有在多CPU或者一个CPU多核的情况中，才会发生并行。否则，看似同时发生的事情，其实都是并发执行的。

- 垃圾回收的并行与并发：

  - 并行（Parallel）：指多条垃圾收集线程并行工作，但此时用户线程仍处于等待状态。如ParNew、Parallel Scavenge、Parallel old；

  - 串行（Serial）

    - 相较于并行的概念，单线程执行。
    - 如果内存不够，则程序暂停，启动JVM垃圾回收器进行垃圾回收。回收完，再启动程序的线程。

  - 并发（concurrent）：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），垃圾回收线程在执行时不会停顿用户程序的运行。

    - 用户程序在继续运行，而垃圾收集程序线程运行于另一个CPU上；

    - 如：CMS、G1垃圾回收器

###### 5.34 安全点与安全区域

###### 面：强引用、软引用、弱引用、虚引用有什么区别？具体使用场景是什么？

​           除强引用外，其他3种引用均可以在java.lang.ref包中找到它们的身影。Reference子类中只有终结器引用是包内可见的，其他3种引用类型均为public，可以在应用程序中直接使用 

- **强引用**（strongReference）：最传统的“引用”的定义，是指在程序代码之中普遍存在的引用赋值，即类似"object obj-new object（）"这种引用关系。无论任何情况下，只要强引用关系还存在，垃圾收集器就永远不会回收掉被引用的对象。
- **软引用**（SoftReference）：在系统将要发生内存溢出之前，将会把这些对象列入回收范围之中进行第二次回收。如果这次回收后还没有足够的内存，才会抛出内存溢出异常。
- **弱引用**（WeakReference）：被弱引用关联的对象只能生存到下一次垃圾收集之前。当垃圾收集器工作时，无论内存空间是否足够，都会回收掉被弱引用关联的对象。
- **虚引用**（PhantomReference）：一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来获得一个对象的实例。为一个对象设置虚引用关联的**唯一目的就是能在这个对象被收集器回收时收到一个系统通知**。

###### 5.35 强引用

- 概念

  ​           当在Java语言中使用new操作符创建一个新的对象，并将其赋值给一个变量的时候，这个变量就成为指向该对象的一个强引用。

- 特点

  - 强引用可以直接访问目标对象。
  - 强引用所指向的对象在任何时候都不会被系统回收，虚拟机宁愿抛出O0M异常，也不会回收强引用所指向对象。
  - 强引用可能导致内存泄漏（主要原因之一）。

###### 5.36 软引用

- 软引用是用来描述一些还有用，但非必需的对象。**只被软引用关联着的对象，在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收**，如果这次回收还没有足够的内存，才会抛出内存溢出异常。

- **应用场景**：

  ​          软引用通常用来**实现内存敏感的缓存**。比如：**高速缓存**就有用到软引用。如果还有空闲内存，就可以暂时保留缓存，当内存不足时清理掉，这样就保证了使用缓存的同时，不会耗尽内存。

- 垃圾回收器在某个时刻决定回收软可达的对象的时候，会清理软引用，并可选地把引用存放到一个引用队列（Reference Queue）

- 用法：

  ![image-20210322163943209](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210322163943209.png)

###### 5.37 弱引用

- 弱引用也是用来描述那些非必需对象，**只被弱引用关联的对象只能生存到下一次垃圾收集发生为止**。在系统GC时，只要发现弱引用，不管系统堆空间使用是否充足，都会回收掉只被弱引用关联的对象。

- 但是，由于垃圾回收器的线程通常优先级很低，因此，并不一定能很快地发现持有弱引用的对象。在这种情况下，弱引用对象可以存在较长的时间。

- 弱引用和软引用一样，在构造弱引用时，也可以指定一个引用队列，当弱引用对象被回收时，就会加入指定的引用队列，通过这个队列可以跟踪对象的回收情况。

- **适用场景**：**软引用、弱引用都非常适合来保存那些可有可无的缓存数据**。如果这么做，当系统内存不足时，这些缓存数据会被回收，不会导致内存溢出。而当内存资源充足时，这些缓存数据又可以存在相当长的时间，从而起到加速系统的作用。

- **用法**：

  ![image-20210322170545706](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210322170545706.png)

###### 面：WeakHashMap底层原理

- WeakHashMap中，对于一个给定的键，其映射的存在并不阻止垃圾回收器对该键的丢弃，这就使该键成为可终止的，被终止，然后被回收。某个键被终止时，它对应的键值对也就从映射中有效地移除了。这边“弱键"的实现和清除，是通过WeakReference和ReferenceQueue实现的。
- WeakHashMap的key是“弱键”，即是WeakReference类型的。ReferenceQueue是一个队列，它会保存被GC回收的“弱键”。实现步骤是：
  - 1，新建WeakHashMap，将"键值对"添加到WeakHashMap中。实际上，WeakHashMap是通过数组table保存Entry（键值对）：每一个Entry实际上是一个单向链表，即Entry是键值对链表
  - 2，当其"弱键"不再被其它对象引用，并被GC回收时。在GC回收该"弱键"时，这个"弱键"也同时会被添加到Referenceoueue（aueue）队列中
  - 3，当下一次我们需要操作WeakHashMap时，会先同步table和queue，table中保存了全部的键值对，而queue中保存被GC回收的键值对；同步它们，就是删除table中被GC回收的键值对。
  - 这就是“弱键"如何被自动从WeakHashMap中删除的步骤了。和HashMap一样，WeakHashMap是不同步的。可以使用Collections.synchronizedMap方法来构造同步的WeakHashMap

###### 5.38 虚引用

- **为一个对象设置虚引用关联的唯一目的在于跟踪垃圾回收过程**。比如：能在这个对象被收集器回收时收到一个系统通知。

- **用法**：

  ![image-20210322171452042](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210322171452042.png)

###### 5.39 终结器引用

#### 5.4 垃圾回收器

###### 5.41 GC分类与性能指标

- GC分类：
  - 按线程数分，可以分为串行垃圾回收器和并行垃圾回收器。
  - 按照工作模式分，可以分为并发式垃圾回收器和独占式垃圾回收器。
  - 按碎片处理方式分，可分为压缩式垃圾回收器和非压缩式垃圾回收器。
  - 按工作的内存区间分，又可分为年轻代垃圾回收器和老年代垃圾回收器。

- 性能指标：
  - **吞吐量**：运行用户代码的时间占总运行时间的比例（总运行时间：程序的运行时间+内存回收的时间）
  - **暂停时间**：执行垃圾收集时，程序的工作线程被暂停的时间。

- ”**高吞吐量**”和”**低暂停时间**”是一对相互竞争的目标（矛盾）。
  - 因为如果选择以吞吐量优先，那么**必然需要降低内存回收的执行频率**，但是这样会导致Gc需要更长的暂停时间来执行内存回收。
  - 相反的，如果选择以低延迟优先为原则，那么为了降低每次执行内存回收时的暂停时间，也**只能频繁地执行内存回收**，但这又引起了年轻代内存的缩减和导致程序吞吐量的下降。

- 现在标准：**在最大吞吐量优先的情况下，降低停顿时间**。

###### 5.42 不同的垃圾回收器概述

7款经典的垃圾回收器：

- 串行回收器：Serial、Serial Old
- 并行回收器：ParNew、Parallel Scavenge、Parallel old
- 并发回收器：CMS、G1

垃圾回收器的经典组合：

![image-20210322172921260](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210322172921260.png)

- 1  两个收集器间有连线，表明它们可以搭配使用：Serial/Serial old，Serial/CMS，ParNew/Serial old，ParNew/CMS、Parallel Scavenge/Serial old，Parallel Scavenge/Parallel old，G1；
- 2   其中Serial old作为CMS出现"Concurrent Mode Failure"失败的后备预案。
- 3（红色虚线）由于维护和兼容性测试的成本，在JDK 8时将Serial+CMS ParNew+Serial old这两个组合声明为废弃，并在JDK 9中完全取消了这些组合的支持，即：移除。
- 4（绿色虚线）JDK 14中：弃用Parallel Scavenge和Serialold Gc组合
- 5（青色虚线）JDK 14中：删除CMS回收器

**总结**：为什么要有很多收集器，一个不够吗？因为Java的使用场景很多，移动端，服务器等。所以就需要针对不同的场景，提供不同的垃圾收集器，提高垃圾收集的性能。所以**我们选择的只是对具体应用最合适的收集器**。

###### 5.43 Serial回收器：串行回收

- 原理
  - Serial收集器，用于年轻代，采用**复制算法、串行回收和"Stop-The-World"机制**的方式执行内存回收。
  - Serial old收集器，用于老年代。采用了**串行回收和"Stop The World"机制**，只不过内存回收算法使用的是**标记-压缩算法**。
    - Serial old是运行在Client模式下默认的老年代的垃圾回收器
    - Serial old在Server模式下主要有两个用途：①与新生代的Parallel Scavenge配合使用；②作为老年代CMS收集器的后备垃圾收集方案

- 原理图：

  <img src="C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210322173943148.png" alt="image-20210322173943148" style="zoom:67%;" />

- 总结：
  - 优势：**简单而高效**（与其他收集器的单线程比），对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。
  - 这种垃圾收集器大家了解，现在已经不用串行的了。而且在限定单核cpu才可以用。现在都不是单核的了。对于交互较强的应用而言，这种垃圾收集器是不能接受的。一般在Java web应用程序中是不会采用串行垃圾收集器的。

###### 5.44 ParNew回收器：并行回收

- 原理：

  ​         ParNew收集器在年轻代中采用**复制算法、"stop-the-World"机制，并行回收**的方式执行内存回收。

  Par是Parallel的缩写，New：只能处理的是新生代

- 原理图：

  ![image-20210322174534764](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210322174534764.png)

- 总结
  - ParNew和Serial收集器的对比？
    - ParNew收集器运行在多CPU的环境下，由于可以充分利用多CPU、多核心等物理硬件资源优势，可以**更快速地完成垃圾收集，提升程序的吞吐量**。
    - 但是在单个CPU的环境下，ParNew收集器不比serial收集器更高效。虽然Serial收集器是基于串行回收，但是由于CPU不需要频繁地做任务切换，因此**可以有效避免多线程交互过程中产生的一些额外开销**。
  - 除Serial外，目前只有ParNew GC能与CMS收集器配合工作

###### 5.45 Parallel回收器：吞吐量优先

- 原理：
  -  Parallel scavenge收集器同样也采用了**复制算法、并行回收和"stop the world"机制**。
  - Parallel old收集器采用了**标记-压缩**算法，但同样也是基于**并行回收和"Stop-the-World"机制**。

- 原理图：

  ![image-20210322175454520](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210322175454520.png)

- 总结

  - Parallel收集器和ParNew收集器对比：

    - 和ParNew收集器不同，Parallel scavenge收集器的目标则是达至一个**可控制的吞吐量**（Throughput），它也被称为吞吐量优先的垃圾收集器。

    - 自适应调节策略也是Parallel scavenge与ParNew一个重要区别。

  - 高吞吐量则可以高效率地利用CPU时间，尽快完成程序的运算任务，主要**适合在后台运算而不需要太多交互的任务**。因此，常见在服务器环境中使用。**例如，那些执行批量处理、订单处理、工资支付、科学计算的应用程序。**

  - 在**Java8中，默认是此垃圾收集器**。

  - 参数配置：

    - **-XX：+UseparallelGC** 手动指定年轻代使用Parallel并行收集器执行内存回收任务。

    - **-XX：+UseParalleloldGC** 手动指定老年代都是使用并行回收收集器。

      - 分别适用于新生代和老年代。默认jdk8是开启的。
      - 上面两个参数，默认开启一个，另一个也会被开启。（互相激活）

    - **-XX：ParallelGCThreads**设置年轻代并行收集器的线程数。

      ​    最好与CPU数量相等，以避免过多的线程数影响垃圾收集性能。

      - 在默认情况下，当CPU数量小于8个，ParallelGCThreads的值等于CPU数量。

      - 当CPU数量大于8个，ParallelGCThreads的值等于3+[5*CPU Count]/8]

    - **-XX：MaxGCPayseMi1lis**设置垃圾收集器最大停顿时间（即STw的时间）。单位是毫秒。

      - 为了尽可能地把停顿时间控制在MaxGCPauseMil1s以内，收集器在工作时会调整Java堆大小或者其他一些参数。
      - 对于用户来讲，停顿时间越短体验越好。但是在服务器端，我们注重高并发，整体的吞吐量。所以服务器端适合Parallel，进行控制。
      - 该参数**使用需谨慎**。

    - -XX:GCTimeRatio垃圾收集时间占总时间的比例（= 1/（N + 1））。用于衡量吞吐量的大小

      - 取值范围（0，100）。默认值99，也就是垃圾回收时间不超过18。
      - 与前一个-XX：MaxGCPauseMillis参数有一定矛盾性。暂停时间越长，Radio参数就容易超过设定的比例。

    - -XX：+UseAdaptivesizePolicy 设置Parallel scavenge收集器具有**自适应调节策略**。

      - 在这种模式下，年轻代的大小、Eden和survivor的比例、晋升老年代的对象年龄等参数会被自动调整，已达到在堆大小、吞吐量和停顿时间之间的平衡点。
      - 在手动调优比较困难的场合，可以直接使用这种自适应的方式，仅指定虚拟机的最大堆、目标（GCTimeRatio）和停顿时间MaxGCPauseMills），让虚拟机自己完成调优工作。

###### 5.46 CMS回收器：低延迟 （Concurrent-Mark-Sweep）

- 概述：CMS垃圾收集器是HotSpot虚拟机中第一款真正意义上的**并发收集器**，它第一次实现了让**垃圾收集线程与用户线程同时工作**。目前很大一部分的Java应用集中在互联网站或者B/S系统的服务端上，这类应用尤**其重视服务的响应速度，希望系统停顿时间最短**，以给用户带来较好的体验。CMS收集器就非常符合这类应用的需求。与ParNew的年轻代垃圾回收器配合使用

- 原理：CMS的垃圾收集，应用于老年代垃圾回收，算法采用**标记-清除算法，并且也会"Stop-The-World"，并发执行**。

- CMS的**工作原理**图

  ![image-20210323093853999](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323093853999.png)

  CMS整个过程分为4个主要阶段，即初始标记阶段、并发标记阶段、重新标记阶段和并发清除阶段。

  - 初始标记（Initial-Mark）阶段：在这个阶段中，程序中所有的工作线程都将会因为"Stop-the-World"机制而出现短暂的暂停，这个阶段的主要任务**仅仅只是标记出GC Roots能直接关联到的对象**。一旦标记完成之后就会恢复之前被暂停的所有应用线程。由于直接关联对象比较小，所以这里的**速度非常快**。
  - 并发标记（Concurrent-Mark）阶段：从GC Roots的**直接关联对象开始遍历整个对象图的过程**，这个过程**耗时较长**但是**不需要停顿用户线程**，可以与垃圾收集线程一起并发运行。
  - 重新标记（Remark）阶段：由于在并发标记阶段中，程序的工作线程会和垃圾收集线程同时运行或者交叉运行，因此为了**修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录**，这个阶段的停顿时间通常会比初始标记阶段稍长一些，但也**远比并发标记阶段的时间短**。
  - 并发清除（Concurrent-Sweep）阶段：此阶段清**理删除掉标记阶段判断的已经死亡的对象，释放内存空间**。由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的

- CMS垃圾回收器**低延迟**的原因：

  ​    尽管CMS收集器采用的是并发回收（非独占式），但是在其**初始化标记和再次标记这两个阶段中仍然需要执行"stop-the-World"机制暂停程序中的工作线程**，不过**暂停时间并不会太长**，因此可以说明目前所有的垃圾收集器都做不到完全不需要"Stop-The-World"，只是尽可能地缩短暂停时间。**由于最耗费时间的并发标记与并发清除阶段都不需要暂停工作，所以整体的回收是低停顿的**。

- CMS垃圾回收器的Mark Sweep会造成内存碎片，为什么不把算法换成Mark Compact？
  - CMS收集器的垃圾收集算法采用的是**标记-清除算法**，这意味着每次执行完内存回收后，由于被执行内存回收的无用对象所占用的内存空间极有可能是不连续的一些内存块，不可避免地将会**产生一些内存碎片**。那么CMS在为新对象分配内存空间时，将无法使用指针碰撞（Bump the Pointer）技术，而只能够选择空闲列表（Free List）执行内存分配。
  - 之所以不采用Mark Compact算法，因为当并发清除的时候，用Compact整理内存的话，原来的用户线程使用的内存无法使用，要保证用户线程能继续执行，前提的它运行的资源不受影响。Mark compact更适合"Stop The World"这种场景下使用。

- 说明：由于在垃圾收集阶段用户线程没有中断，所以**在CMS回收过程中，还应该确保应用程序用户线程有足够的内存可用**。因此，CMS收集器不能像其他收集器那样等到老年代几乎完全被填满了再进行收集，而是**当堆内存使用率达到某一阈值时，便开始进行回收**，以确保应用程序在CMS工作过程中依然有足够的空间支持应用程序运行。要是CMS运行期间预留的内存无法满足程序需要，就会**出现一次"Concurrent Mode Failure失败，这时虚拟机将启动后备预案：临时启用Serial old**收集器来重新进行老年代的垃圾收集，这样停顿时间就很长了。
- CMS回收器的优缺点：
  - 优点：并发收集、低延迟
  - 弊端：
    - 1）**会产生内存碎片**，导致并发清除后，用户线程可用的空间不足。在无法分配大对象的情况下，不得不提前触发Full GC.
    - 2）**CMS收集器对CPU资源非常敏感**。在并发阶段，它虽然不会导致用户停顿，但是会因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低。
    - 3）**CMS收集器无法处理浮动垃圾**。可能出现"Concurrent Mode Failure"失败而导致另一次Full GC的产生。在并发标记阶段由于程序的工作线程和垃圾收集线程是同时运行或者交叉运行的，那么**在并发标记阶段如果产生新的垃圾对象，CMS将无法对这些垃圾对象进行标记，最终会导致这些新产生的垃圾对象没有被及时回收**，从而只能在下一次执行GC时释放这些之前未被回收的内存空间。

- 参数设置

  - **-XX：+UseConcMarkSweepGC**手动指定使用CMS收集器执行内存回收任务。

    ​           开启该参数后会自动将-XX：+UseParNewGC打开。即：ParNew（Young区用）+CMS（old区用）+Serial old的组合。

  - **-XX:CMSInitiatingoccupanyFraction**设置堆内存使用率的阈值，一旦达到该阈值，便开始进行回收。

    - JDK5及以前版本的默认值为68，即当老年代的空间使用率达到68%时，会执行一次CMS回收。JDK6及以上版本默认值为92%
    - 如果内存增长缓慢，则可以设置一个稍大的值，大的阈值可以有效降低CMS的触发频率，减少老年代回收的次数可以较为明显地改善应用程序性能。反之，如果应用程序内存使用率增长很快，则应该降低这个阈值，以避免频繁触发老年代串行收集器。因此**通过该选项便可以有效降低Full GC的执行次数**。

  - **-XX：+UseCMSCompactAtFul1Collection**用于指定在执行完Full GC后对内存空间进行压缩整理，以此避免内存碎片的产生。不过由于内存压缩整理过程无法并发执行，所带来的问题就是停顿时间变得更长了。

  - **-XX：CMSFullGCsBeforeCompaction**设置在执行多少次Full GC后对内存空间进行压缩整理。1

  - **-XX：ParallelCMSThreads**设置CMS的线程数量。
    CMS默认启动的线程数是（ParallelGCThreads+3）/4，ParallelGCThreads是年轻代并行收集器的线程数。当CPU资源比较紧张时，受到CMS收集器线程的影响，应用程序的性能在垃圾回收阶段可能会非常糟糕。

- 小结

  - HotSpot有这么多的垃圾回收器，那么如果有人问，Serial GC、Parallel GC、Concurrent Mark Sweep GC这三个GC有什么不同呢？

    请记住以下口令：

    - 如果你想要最小化地使用内存和并行开销，请选serial GC；
    - 如果你想要最大化应用程序的吞吐量，请选Parallel GC；
    - 如果你想要最小化Gc的中断或停顿时间，请选CMS GC

- 说明：
  - JDK9新特性：CMS被标记为Deprecate了，过时
  - JDK14新特性：删除CMS垃圾回收器

###### 5.47 G1回收器：区域化分代式

- 为什么叫Garbage First（G1）
  - 因为G1是一个并行回收器，它把堆内存分割为很多不相关的区域（Region）（物理上不连续的）。使用不同的Region来表示Eden、幸存者0区，幸存者1区，老年代等。
  - G1 GC有计划地避免在整个Java堆中进行全区域的垃圾收集。G1跟踪各个Region里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，**每次根据允许的收集时间，优先回收价值最大的Region**
  - 由于这种方式的侧重点在于回收垃圾最大量的区间（Region），所以我们给G1一个名字：垃圾优先（Garbage First）。

- 概述
  - G1（Garbage-First）是一款面向服务端应用的垃圾收集器，**主要针对配备多核CPU及大容量内存的机器**，以极高概率满足Gc停顿时间的同时，还兼具高吞吐量的性能特征。
  - 是JDK9及以后的默认垃圾回收器

- G1回收器的优势

  - **并行与并发**

    - 并行性：G1在回收期间，可以有多个GC线程同时工作，有效利用多核计算能力。此时用户线程STW
    - 并发性：G1拥有与应用程序交替执行的能力，部分工作可以和应用程序同时执行，因此，一般来说，不会在整个回收阶段发生完全阻塞应用程序的情况

  - **分代收集**

    - 从分代上看，**G1依然属于分代型垃圾回收器**，它会区分年轻代和老年代，年轻代依然有Eden区和Survivor区。但从堆的结构上看，它**不要求整个Eden区、年轻代或者老年代都是连续的，也不再坚持固定大小和固定数量。**
    - 将**堆空间分为若干个区域（Region），这些区域中包含了逻辑上的年轻代和老年代**。
    - 和之前的各类回收器不同，它同**时兼顾年轻代和老年代**。对比其他回收器，或者工作在年轻代，或者工作在老年代

  - **空间整合**

    - CMS：“标记-清除”算法、内存碎片、若干次GC后进行一次碎片整理。
    - G1将内存划分为一个个的region。内存的回收是以region作为基本单位的。**Region之间是复制算法**，但**整体上实际可看作是标记-压缩（Mark-Compact）算法**，两种算法都可以避免内存碎片。这种特性有利于程序长时间运行，分配大对象时不会因为无法找到连续内存空间而提前触发下一次Gc.尤其是当Java堆非常大的时候，G1的优势更加明显。

  - **可预测的停顿时间模型**（即：软实时soft real-time）

    ​       这是G1相对于CMS的另一大优势，G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。

    - 由于分区的原因，G1可以只选取部分区域进行内存回收，这样缩小了回收的范围，因此对于全局停顿情况的发生也能得到较好的控制。
    - G1跟踪各个Region里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），在后台维护一个优先列表，**每次根据允许的收集时间，优先回收价值最大的Region**。保证了G1收集器在有限的时间内可以获取尽可能高的收集效率。

- 参数设置
  - **-XX：+UseG1GC** 手动指定使用G1收集器执行内存回收任务。
  - **-XX:G1HeapRegionsize** 设置每个Region的大小。值是2的幂，范围是1MB到32MB之间，目标是根据最小的Java堆大小划分出约2048个区域。默认是堆内存的1/2000.
  - **-XX：MaxGCPauseMillis** 设置期望达到的最大GC停顿时间指标（JVM会尽力实现，但不保证达到）。默认值是200ms
  - **-XX：ParallelGCThread** 设置STW工作线程数的值。最多设置为8
  - **-XX：ConcGCThreads**设置并发标记的线程数。将n设置为并行垃圾回收线程
    数（ParallelGCThreads）的1/4左右。
  - **-XX：InitiatingHeapoccupancyPercent** 设置触发并发Gc周期的Java堆占用率阈值。超过此值，就触发Gc。默认值是45.

- 分区Region（化整为零）G1创新点

  - 使用G1收集器时，它将整个Java堆划分成约2048个大小相同的独立Region块，每个Region块大小根据堆空间的实际大小而定，整体被控制在1MB到32MB之，且为2X，即1MB，2MB，4MB，8MB，16MB，32MB。可以通过-XX:G1HeapRegionsize设定。**所有的Region大小相同，且在JVM生命周期内不会被改变**。虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了，它们都是一部分Region（不需要连续）的集合。通过Region的动态分配方式实现逻辑上的连续。

    ![image-20210323113551137](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323113551137.png)

  - 设置H的原因：

    ​           对于堆中的大对象，默认直接会被分配到老年代，但是如果它是一个短期存在的大对象，就会对垃圾收集器造成负面影响。为了解决这个问题，G1划分了一个Humongous区，它用来专门存放大对象。**如果一个H区装不下一个大对象，那么G1会寻找连续的H区来存储**。为了能找到连续的H区，有时候不得不启动Full GC。G1的大多数行为都把H区作为老年代的一部分来看待。

- G1垃圾回收器垃圾回收过程

  ​      G1 GC的垃圾回收过程主要包括如下三个环节：

  - 年轻代GC（Young GC）

  - 老年代并发标记过程（Concurrent Marking.

  - 混合回收（Mixed GC）

  - 如果需要，单线程、独占式、高强度的Full GC还是继续存在的。它针对Gc的评估失败提供了一种失败保护机制，即强力回收。

    ![image-20210323114150218](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323114150218.png)

  - 应用程序分配内存，**当年轻代的Eden区用尽时开始年轻代回收过程**；G1的年轻代收集阶段是一个**并行的独占式**收集器。在年轻代回收期，G1GC暂停所有应用程序程，启动多线程执行年轻代回收。然后**从年轻代区间移动存活对象到Survivor区间或者老年区间，也有可能是两个区间都会涉及**

  - 当堆内存使用达到一定值（默认45%）时，开始老年代并发标记过程。

  - 标记完成马上开始混合回收过程。对于一个混合回收期，G1 GC从老年区间移动存活对象到空闲区间，这些空闲区间也就成为了老年代的一部分。老年代的G1回收器和其他GC不同，G1的老年代回收器不需要整个老年代被回收，一次只需要扫描/回收一小部分老年代的Region就可以了。同时，这个老年代Region是和年轻代一起被回收的。

    ![image-20210323144454898](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323144454898.png)

    ![image-20210323144511741](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323144511741.png)

    ![image-20210323144525360](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323144525360.png)

    ![image-20210323144536945](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323144536945.png)

    ![image-20210323144552968](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323144552968.png)

    ![image-20210323144727079](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210323144727079.png)

###### 5.48 垃圾回收器总结

###### 5.49 GC日志分析

###### 5.10 垃圾回收器的新发展

##### 5.5 大厂面试题

![image-20210301135811532](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210301135811532.png)

![image-20210301135822404](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210301135822404.png)

![image-20210329101538347](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210329101538347.png)

![image-20201230160626685](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20201230160626685.png)
