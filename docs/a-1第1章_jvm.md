---

# 第一章 JVM

### 1. JVM运行机制

### 1.1.1 什么是JVM？

**JVM定义**?

Java Virtual Machine（Java虚拟机）的缩写，JVM是运行Java字节码的虚拟机。

**JVM包括：**

- 类加载子系统 Class Loader SubSystem
- 运行时数据区 Runtime Data Area
- 执行引擎
- 本地接口库 Native Interface Library （本地接口库通过调用本地方法库与OS交互

**Java运行过程：**

- **Java源文件**(.java)被**编译器**编译成**字节码**(.class)文件
- **JVM**将**字节码**编译成对应操作系统的**机器码**
- **机器码**调用对应系统的**本地方法库**执行相应方法

**JVM的特点**

- 实现跨平台的最核心的部分
- .class 文件会在 JVM 上执行，JVM 会解释给操作系统执行
- 有自己的指令集，解释自己的指令集到 CPU 指令集和系统资源的调用
- JVM 只关注被编译的 .class 文件，不关心 .java 源文件



![img](v2-9a8a3331f4d7eb3a4fa240f8625d9969_720w.jpg)

![](3IvMQ.jpg)

**上图中：**

- **类加载器子系统**将.Class文件加载到JVM中

- **运行时数据区**存储JVM运行时产生的数据

- **执行引擎**包括

- - **即时编译器**：将.Class编译成机器码
  - **垃圾回收器**：回收不再使用的对象

- **本地方法库**调用OS本地方法库完成指令操作

### 2.JVM线程

**JVM后台运行线程：**

- **虚拟机线程 JVM Thread**

  在JVM到达安全点（SafePoint）时出现


- **周期性任务线程**

  通过 **定时器** 来调度 线程，实现周期性任务执行


- **GC线程**

  用于垃圾回收


- **编译器线程**

  将.Class动态编译成机器码


- **信号分发线程**

  接收发送给JVM的信号并调用JVM


### **3.Java 内存区域(运行时数据区)**

Java 虚拟机在执行 Java 程序的过程中会把它管理的内存划分成若干个不同的数据区域。JDK. 1.8 和之前的版本略有不同，下面会介绍到。

**JDK 1.8之前：**

![](JVM运行时数据区域.png)

**JDK 1.8 ：**

![](2019-3Java运行时数据区域JDK1.8.png)

**线程私有的：**

- 程序计数器
- 虚拟机栈
- 本地方法栈

**线程共享的：**

- 堆
- 方法区
- 直接内存(非运行时数据区的一部分)

上图中：

- **线程私有区域：**

  生命周期与线程相同，同生共死。


- **线程共享区域：**

  与JVM同生共死。


- **直接内存：**

  又称堆外内存。JDK的NIO就是基于堆外内存实现。Java进程可以通过堆外内存技术避免在Java堆与Native来回复制数据带来资源浪费与性能损耗。因此在高并发场景广泛使用（Netty、Flink、HBase、Hadoop)




![](Java虚拟机运行时数据区.jpg)

#### 3.1程序计数器

程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。**字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完。**

另外，**为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。**

**从上面的介绍中我们知道程序计数器主要有两个作用：**

1. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

**注意：程序计数器是唯一一个不会出现 OutOfMemoryError 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。**



#### 3.2.Java 虚拟机栈

**与程序计数器一样，Java虚拟机栈也是线程私有的，它的生命周期和线程相同，描述的是 Java 方法执行的内存模型，每次方法调用的数据都是通过栈传递的。**

**Java 内存可以粗糙的区分为堆内存（Heap）和栈内存(Stack),其中栈就是现在说的虚拟机栈，或者说是虚拟机栈中局部变量表部分。** （实际上，Java虚拟机栈是由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法出口信息。）

**局部变量表主要存放了编译器可知的各种数据类型**（boolean、byte、char、short、int、float、long、double）、**对象引用**（reference类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

**Java 虚拟机栈会出现两种异常：StackOverFlowError 和 OutOfMemoryError。**

- **StackOverFlowError：** 若Java虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前Java虚拟机栈的最大深度的时候，就抛出StackOverFlowError异常。
- **OutOfMemoryError：** 若 Java 虚拟机栈的内存大小允许动态扩展，且当线程请求栈时内存用完了，无法再动态扩展了，此时抛出OutOfMemoryError异常。

Java 虚拟机栈也是线程私有的，每个线程都有各自的Java虚拟机栈，而且随着线程的创建而创建，随着线程的死亡而死亡。

**栈帧：**

- 存储部分运行数据及数据结构
- 处理**动态链接**方法**返回值**和**异常分派**
- 记录方法执行过程。
- 方法执行时JVM会创建对应的栈帧。
- 方法执行和返回对应栈帧在JVM栈中的入栈与出栈。

**扩展：那么方法/函数如何调用？**

Java 栈可用类比数据结构中栈，Java 栈中保存的主要内容是栈帧，每一次函数调用都会有一个对应的栈帧被压入Java栈，每一个函数调用结束后，都会有一个栈帧被弹出。

Java方法有两种返回方式：

1. return 语句。
2. 抛出异常。

不管哪种返回方式都会导致栈帧被弹出。

![img](v2-70412cbe6a1f36ae405d2a8a38ce3ed1_720w.jpg)



上图：

- 线程1在CPU1执行
- 线程2在CPU2执行
- 线程N在等待获取CPU时间片
- 每个方法的执行对应一个栈帧
- 每个线程只能存在一个活动的栈帧



#### 3.3.本地方法栈

和虚拟机栈所发挥的作用非常相似，区别是： **虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。

方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 StackOverFlowError 和 OutOfMemoryError 两种异常。



#### 3.4.堆

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

Java 堆是垃圾收集器管理的主要区域，因此也被称作**GC堆（Garbage Collected Heap）**.从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以Java堆还可以细分为：新生代和老年代：再细致一点有：Eden空间、From Survivor、To Survivor空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存。**

![](2019-3堆结构.png)


上图所示的 eden区、s0区、s1区都属于新生代，tentired 区属于老年代。大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会加 1(Eden区->Survivor 区后对象的初始年龄变为1)，当它的年龄增加到一定程度（默认为15岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。



#### 3.5.方法区

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。虽然Java虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做 **Non-Heap（非堆）**，目的应该是与 Java 堆区分开来。

方法区也被称为永久代。很多人都会分不清方法区和永久代的关系，为此我也查阅了文献。

##### 方法区和永久代的关系

> 《Java虚拟机规范》只是规定了有方法区这么个概念和它的作用，并没有规定如何去实现它。那么，在不同的 JVM 上方法区的实现肯定是不同的了。  **方法区和永久代的关系很像Java中接口和类的关系，类实现了接口，而永久代就是HotSpot虚拟机对虚拟机规范中方法区的一种实现方式。** 也就是说，永久代是HotSpot的概念，方法区是Java虚拟机规范中的定义，是一种规范，而永久代是一种实现，一个是标准一个是实现，其他的虚拟机实现并没有永久带这一说法。

##### 常用参数

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小

```java
-XX:PermSize=N //方法区(永久代)初始大小
-XX:MaxPermSize=N //方法区(永久代)最大大小,超过这个值将会抛出OutOfMemoryError异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。**

JDK 1.8 的时候，方法区（HotSpot的永久代）被彻底移除了（JDK1.7就已经开始了），取而代之是元空间，元空间使用的是直接内存。

下面是一些常用参数：

```java
-XX:MetaspaceSize=N //设置Metaspace的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置Metaspace的最大大小
```

与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。

##### 为什么要将永久代(PermGen)替换为元空间(MetaSpace)呢?

整个永久代有一个 JVM 本身设置固定大小上线，无法进行调整，而元空间使用的是直接内存，受本机可用内存的限制，并且永远不会得到java.lang.OutOfMemoryError。你可以使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制。`-XX：MetaspaceSize` 调整标志定义元空间的初始大小如果未指定此标志，则 Metaspace 将根据运行时的应用程序需求动态地重新调整大小。

当然这只是其中一个原因，还有很多底层的原因，这里就不提了。



![img](v2-b8d6f65c34e24e24dbacacd684e8c6ee_720w.jpg)

#### 运行时常量池

运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池信息（用于存放编译期生成的各种字面量和符号引用）

既然运行时常量池时方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 异常。

**JDK1.7及之后版本的 JVM 已经将运行时常量池从方法区中移了出来，在 Java 堆（Heap）中开辟了一块区域存放运行时常量池。** 

![](02276bcccbe181985ac50af2668940c9.jpg)
——图片来源：https://blog.csdn.net/wangbiao007/article/details/78545189

#### 直接内存

**直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 OutOfMemoryError 异常出现。**

JDK1.4 中新加入的 **NIO(New Input/Output) 类**，引入了一种基于**通道（Channel）** 与**缓存区（Buffer）** 的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为**避免了在 Java 堆和 Native 堆之间来回复制数据**。

本机直接内存的分配不会收到 Java 堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。



#### 3.6 Java对象的创建过程

下图便是 Java 对象的创建过程(掌握每一步在做什么)。

![](73f30855e4cdfd6f2e944398e97981a2.png)

**①类加载检查：** 虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

**②分配内存：** 在**类加载检查**通过后，接下来虚拟机将为新生对象**分配内存**。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。**分配方式**有 **“指针碰撞”** 和 **“空闲列表”** 两种，**选择那种分配方式由 Java 堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定**。

**内存分配的两种方式：（掌握）**

选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 收集器的算法是"标记-清除"，还是"标记-整理"（也称作"标记-压缩"），值得注意的是，复制算法内存也是规整的

![](72d8.jpg)

**内存分配并发问题（补充内容，需要掌握）**

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全：

- **CAS+失败重试：** CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。**
- **TLAB：** 为每一个线程预先在Eden区分配一块儿内存，JVM在给线程中的对象分配内存时，首先在TLAB分配，当对象大于TLAB中的剩余内存或TLAB的内存已用尽时，再采用上述的CAS进行内存分配

**③初始化零值：** 内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

**④设置对象头：** 初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是那个类的实例、如何才能找到类的元数据信息、对象的哈希吗、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

**⑤执行 init 方法：** 在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

#### 3.7 对象的访问定位有哪两种方式?

建立对象就是为了使用对象，我们的Java程序通过栈上的 reference 数据来操作堆上的具体对象。对象的访问方式有虚拟机实现而定，目前主流的访问方式有**①使用句柄**和**②直接指针**两种：

1. **句柄：** 如果使用句柄的话，那么Java堆中将会划分出一块内存来作为句柄池，reference 中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息；

![使用句柄](https://images.xiaozhuanlan.com/photo/2019/eea6f944dd5d522a079b43d409f620d7.png)

2. **直接指针：**  如果使用直接指针访问，那么 Java 堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而reference 中存储的直接就是对象的地址。

![使用直接指针](https://images.xiaozhuanlan.com/photo/2019/242ab9b89650e22c816ebdeea7311ac5.png)

**这两种对象访问方式各有优势。使用句柄来访问的最大好处是 reference 中存储的是稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。使用直接指针访问方式最大的好处就是速度快，它节省了一次指针定位的时间开销。**



### **4.JVM运行时内存**

- 也叫JVM堆
- GC角度分为新生代（占1/3）、老年代（占2/3）和永久代（占很少）。新生代又分为3个区。



![img](v2-4a1826a5bfa42aad8174547754fd72d0_720w.jpg)

#### 4.1 说一下堆内存中对象的分配的基本策略

**堆空间的基本结构：**

<div align="center">  
<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3堆结构.png" width="400px"/>
</div>


上图所示的 eden区、s0区、s1区都属于新生代，tentired 区属于老年代。大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会加 1(Eden区->Survivor 区后对象的初始年龄变为1)，当它的年龄增加到一定程度（默认为15岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

另外，大对象和长期存活的对象会直接进入老年代。

![堆内存常见分配策略](8b15e0786919315de03da1f92d493b8d.jpg)

#### 4.2.新生代/老年代/永久代特点

##### ① 新生代

- 新建对象存放区域（大对象除外）
- 由于JVM频繁创建对象，新生代会频繁GC
- 新生代GC算法叫做**复制算法**

**Eden区**

新建对象首先存放区，如果是大对象，会放在老年代。

**SurvivorFrom区**

将上次GC幸存者作为这次GC扫描对象

**SurvivorTo区**

保留上次GC幸存者



**新生代GC过程：**

- 将Eden区和SurvivorFrom区存活对象复制到SurvivorTo区中。如果对象年龄太大或SurvivorTo区内存不够或是大对象，则直接复制到老年代中。
- 清空Eden区和SurvivorFrom区对象
- 将SurvivorTo区与SurvivorFrom区互换，原先的SurvivorTo区成为下次GC的SurvivorFrom区

##### ②老年代

- 存储长生命周期对象与大对象

- 对象比较稳定，不会频繁GC

- 老年代GC算法叫做标记清除算法

  首先扫描所有对象并标记存活对象，然后回收未标记对象。


- 老年代GC时间长
- 老年代没有内存可分配时，会抛出OOM

##### ③ 永久代

- 意味内存永久保留区域
- 主要存放Class和元数据
- 运行时不会被GC，因此永久代内存随着Class文件增加而增加，加载过多Class文件会抛出OOM。比如Tomcat引用Jar文件过多导致无法启动
- Java8中，永久代被元数据区取代。

**永久代与元数据区区别**

元数据区没有使用JVM内存，而是使用OS内存。因此元数据内存不再被JVM内存限制，只和OS内存有关

#### 4.3 Minor Gc和Full GC 有什么不同呢？

大多数情况下，对象在新生代中 eden 区分配。当 eden 区没有足够空间进行分配时，虚拟机将发起一次Minor GC。

- **新生代GC（Minor GC）**:指发生新生代的的垃圾收集动作，Minor GC非常频繁，回收速度一般也比较快。
- **老年代GC（Major GC/Full GC）**:指发生在老年代的GC，出现了Major GC经常会伴随至少一次的Minor GC（并非绝对），Major GC的速度一般会比Minor GC的慢10倍以上。



### 5.垃圾回收算法

#### 5.1如何确定垃圾(如何判断对象是否死亡?)

堆中几乎放着所有的对象实例，对堆垃圾回收前的第一步就是要判断哪些对象已经死亡（即不能再被任何途径使用的对象）。

通过**引用计数法**和**可达性分析**来判断对象是否可回收。



![img](v2-f3476a19bad816d9d715a64df6aa9052_720w.jpg)



**引用计数法**

- Java如果操作一个对象，就要获得对象的引用。
- 给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加1；当引用失效，计数器就减1；任何时候计数器为0的对象就是不可能再被使用的。对象没有被引用，可被回收。
- 引用计数法会导致循环引用问题，当两个对象相互引用时，就不能被回收。





![img](v2-aaf0cada4546a8ee80185746c3e3f1a1_720w.jpg)



**可达性分析算法**

- 为解决引用计数的循环引用问题而设计

- 这个算法的基本思想就是通过一系列的称为 **“GC Roots”** 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连的话，则证明此对象是不可用的。

- 不可达对象被两次标记后回收。

  ![可达性分析算法](7ba1179cfa1bba68f85cd7538256a5a1.jpg)



#### 5.2 简单的介绍一下强引用,软引用,弱引用,虚引用

无论是通过引用计数法判断对象引用数量，还是通过可达性分析法判断对象的引用链是否可达，判定对象的存活都与“引用”有关。

JDK1.2之前，Java中引用的定义很传统：如果reference类型的数据存储的数值代表的是另一块内存的起始地址，就称这块内存代表一个引用。

JDK1.2以后，Java对引用的概念进行了扩充，将引用分为强引用、软引用、弱引用、虚引用四种（引用强度逐渐减弱）

##### 1. 强引用(StrongReference):

**将一个对象赋值给另一个对象时。强引用一定会为可达性，因此不会被GC。强引用也是造成内存泄漏的主要原因。**

以前我们使用的大部分引用实际上都是强引用，这是使用最普遍的引用。如果一个对象具有强引用，那就类似于**必不可少的生活用品**，垃圾回收器绝不会回收它。当内存空 间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足问题。

##### 2. 软引用(SoftReference)

**通过SoftReference类实现。系统内存不足时会被回收。**

如果一个对象只具有软引用，那就类似于**可有可无的生活用品**。如果内存空间足够，垃圾回收器就不会回收它，如果内存空间不足了，就会回收这些对象的内存。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收，JAVA虚拟机就会把这个软引用加入到与之关联的引用队列中。

##### 3. 弱引用(WeakReference)

**通过WeakReference类实现。GC过程中一定会被回收。**

如果一个对象只具有弱引用，那就类似于**可有可无的生活用品**。弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它 所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程， 因此不一定会很快发现那些只具有弱引用的对象。 

弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。

##### 4. 虚引用（PhantomReference）

**通过PhantomReference类实现，虚引用和引用队列一起使用，用于追踪对象的垃圾回收状态。**

"虚引用"顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收。

**虚引用主要用来跟踪对象被垃圾回收的活动**。

**虚引用与软引用和弱引用的一个区别在于：** 虚引用必须和引用队列（ReferenceQueue）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是 否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。程序如果发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动。 

特别注意，在程序设计中一般很少使用弱引用与虚引用，使用软引用的情况较多，这是因为**软引用可以加速JVM对垃圾内存的回收速度，可以维护系统的运行安全，防止内存溢出（OutOfMemory）等问题的产生**。

#### 如何判断一个常量是废弃常量?

运行时常量池主要回收的是废弃的常量。那么，我们如何判断一个常量是废弃常量呢？

假如在常量池中存在字符串 "abc"，如果当前没有任何String对象引用该字符串常量的话，就说明常量 "abc" 就是废弃常量，如果这时发生内存回收的话而且有必要的话，"abc" 就会被系统清理出常量池。

#### 如何判断一个类是无用的类?

方法区主要回收的是无用的类，那么如何判断一个类是无用的类的呢？

判定一个常量是否是“废弃常量”比较简单，而要判定一个类是否是“无用的类”的条件则相对苛刻许多。类需要同时满足下面3个条件才能算是 **“无用的类”** ：

- 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类的任何实例。
- 加载该类的 ClassLoader 已经被回收。
- 该类对应的 java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

虚拟机可以对满足上述3个条件的无用类进行回收，这里说的仅仅是“可以”，而并不是和对象一样不使用了就会必然被回收。

#### 5.3 Java中垃圾收集有哪些算法，各自的特点？

标记清除法、复制法、标记整理法、分代收集法



![img](v2-e3f944e2973ee808b2ca9abc841a860c_720w.jpg)



##### 标记-清除算法

- 基础GC算法
- 分为标记和清除两阶段
- 标记阶段标记，清除阶段回收

<img src="63707281.jpg" alt="公众号" width="500px">

算法分为“标记”和“清除”阶段：首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。它是最基础的收集算法，后续的算法都是对其不足进行改进得到。这种垃圾收集算法会带来两个明显的问题：

1. **效率问题**
2. **空间问题（标记清除后会产生大量不连续的碎片）**如果回收小对象过多，会引起内存碎片化，导致大对象无法获取连续可用内存问题。



##### **复制算法**

为解决标记算法内存碎片问题而设计。它可以将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。这样就使每次的内存回收都是对内存区间的一半进行回收。

<img src="90984624.jpg" alt="公众号" width="500px">

![img](v2-23c42a3cd4415bdce9ba97179fdb5301_720w.jpg)



复制算法问题：

每次只用一半内存，浪费空间。

区域1与区域2来回复制影响性能。



##### 标记-整理算法

- 结合上面两个算法优点。
- 标记阶段和标记清除算法一样。
- 清除阶段和复制算法一样。不是直接对可回收对象回收，而是将存活对象移到内存另一个区域，然后直接清理掉边界以外的内存。



![img](v2-3f0c0d1ae2a482dca6d4b153c29750f3_720w.jpg)





##### 分代收集算法

- 上面3个算法都无法对所有类型对象回收（长生命对象、短生命对象、大对象、小对象）。
- 因此针对不同对象，设计了分代收集算法。当前虚拟机的垃圾收集都采用分代收集算法，这种算法没有什么新的思想，只是根据对象存活周期的不同将内存分为几块。一般将java堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。
- 新生代存放新对象，数量多，生命短，每次GC都大量被回收。采用复制算法。
- 老年代存放大对象，生命长，被GC数量少。采用标记清除算法。

**比如在新生代中，每次收集都会有大量对象死去，所以可以选择复制算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们必须选择“标记-清除”或“标记-整理”算法进行垃圾收集。**



##### 分代收集算法和分区收集算法

- 分代收集算法

  新生代用复制算法

  老年代用标记整理算法


- 分区收集算法

  将堆内存划分为连续大小不等的小区域，每个小区域单独运行与GC。

  好处是根据每个小区域灵活使用使用和释放内存。


##### HotSpot为什么要分为新生代和老年代？

主要是为了提升GC效率。上面提到的分代收集算法已经很好的解释了这个问题。

### 6.常见的垃圾收集器

Java堆内存分为新生代与老年代。新生代用复制算法进行GC。老年代用标记整理进行GC。

因此JVM为新生代和老年代提供了不同的GC器。

![垃圾收集算法分类](da5619d747fb4f9207e7359463b05491.jpg)



![img](v2-8c89f27bd3579edda9f16c04e30d5f9a_720w.jpg)

**如果说收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。**

虽然我们对各个收集器进行比较，但并非要挑选出一个最好的收集器。因为知道现在为止还没有最好的垃圾收集器出现，更加没有万能的垃圾收集器，**我们能做的就是根据具体应用场景选择适合自己的垃圾收集器**。试想一下：如果有一种四海之内、任何场景下都适用的完美收集器存在，那么我们的HotSpot虚拟机就不会实现那么多不同的垃圾收集器了。

##### 1.Serial收集器

**特点: 新生代，单线程，复制算法。**

Serial（串行）收集器收集器是最基本、历史最悠久的垃圾收集器了。大家看名字就知道这个收集器是一个单线程收集器了。它的 **“单线程”** 的意义不仅仅意味着它只会使用一条垃圾收集线程去完成垃圾收集工作，更重要的是它在进行垃圾收集工作的时候必须暂停其他所有的工作线程（ **"Stop The World"** ），直到它收集结束。

 **新生代采用复制算法，老年代采用标记-整理算法。**

![ Serial收集器](c881f630a77ac5e87ba4c7ae3e0fc877.jpg)

虚拟机的设计者们当然知道Stop The World带来的不良用户体验，所以在后续的垃圾收集器设计中停顿时间在不断缩短（仍然还有停顿，寻找最优秀的垃圾收集器的过程仍然在继续）。

但是Serial收集器有没有优于其他垃圾收集器的地方呢？当然有，它**简单而高效（与其他收集器的单线程相比）**。Serial收集器由于没有线程交互的开销，自然可以获得很高的单线程收集效率。Serial收集器对于运行在Client模式下的虚拟机来说是个不错的选择。

##### 2.ParNew收集器

**特点: 新生代，多线程，复制算法。**

**ParNew收集器其实就是Serial收集器的多线程版本，除了使用多线程进行垃圾收集外，其余行为（控制参数、收集算法、回收策略等等）和Serial收集器完全一样。**

 **新生代采用复制算法，老年代采用标记-整理算法。**

![ParNew收集器](10395bc0572402b99293ff6cfec4931c.jpg)

它是许多运行在Server模式下的虚拟机的首要选择，除了Serial收集器外，只有它能与CMS收集器（真正意义上的并发收集器，后面会介绍到）配合工作。

**并行和并发概念补充：**

- **并行（Parallel）** ：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。
- **并发（Concurrent）**：指用户线程与垃圾收集线程同时执行（但不一定是并行，可能会交替执行），用户程序在继续运行，而垃圾收集器运行在另一个CPU上。

##### 3.Parallel Scavenge收集器

**特点: 新生代，多线程，复制算法**

Parallel Scavenge 收集器类似于ParNew 收集器。 **那么它有什么特别之处呢？**

```
-XX:+UseParallelGC 
    使用Parallel收集器+ 老年代串行

-XX:+UseParallelOldGC
    使用Parallel收集器+ 老年代并行
```

**Parallel Scavenge收集器关注点是吞吐量（高效率的利用CPU）。CMS等垃圾收集器的关注点更多的是用户线程的停顿时间（提高用户体验）。所谓吞吐量就是CPU中用于运行用户代码的时间与CPU总消耗时间的比值。** Parallel Scavenge收集器提供了很多参数供用户找到最合适的停顿时间或最大吞吐量，如果对于收集器运作不太了解的话，手工优化存在的话可以选择把内存管理优化交给虚拟机去完成也是一个不错的选择。

 **新生代采用复制算法，老年代采用标记-整理算法。**
![ParNew收集器](10395bc0572402b99293ff6cfec4931c.jpg)

##### 4.Serial Old收集器

**特点: 老年代，单线程，标记整理算法。配合Serial使用**

**Serial收集器的老年代版本**，它同样是一个单线程收集器。它主要有两大用途：一种用途是在JDK1.5以及以前的版本中与Parallel Scavenge收集器搭配使用，另一种用途是作为CMS收集器的后备方案。



![img](v2-a46b78603065a902898e47866870310b_720w.jpg)



##### 5.Parallel Old收集器

**特点: 老年代，多线程，标记整理算法。配合Parallel Scavenge使用**



![img](v2-78509613ff642263a094a7eb49a17059_720w.jpg)

 **Parallel Scavenge收集器的老年代版本**。使用多线程和“标记-整理”算法。在注重吞吐量以及CPU资源的场合，都可以优先考虑 Parallel Scavenge收集器和Parallel Old收集器。

##### 6.CMS收集器

**特点: 老年代，多线程，标记清除算法。**

**CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。它而非常符合在注重用户体验的应用上使用。**

**CMS（Concurrent Mark Sweep）收集器是HotSpot虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。**

从名字中的**Mark Sweep**这两个词可以看出，CMS收集器是一种 **“标记-清除”算法**实现的，它的运作过程相比于前面几种垃圾收集器来说更加复杂一些。整个过程分为四个步骤：

- **初始标记：** 暂停所有的其他线程，并标记和GC Roots直接关联对象，速度很快。需要暂停工作线程。
- **并发标记：** 用户线程一起工作，执行GC Roots跟踪标记过程。同时开启GC和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以GC线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方。
- **重新标记：** 在并发标记过程中，用户线程运行导致对象状态变化，需要暂停工作线程并且重新标记。重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短
- **并发清除：** 和用户线程一起工作，执行清除GC Roots不可达对象任务。开启用户线程，同时GC线程开始对为标记的区域做清扫。

![CMS垃圾收集器](https://images.xiaozhuanlan.com/photo/2019/81ccc1600b6a2fdab98115abeea41877.jpg)

从它的名字就可以看出它是一款优秀的垃圾收集器，主要优点：**并发收集、低停顿**。但是它有下面三个明显的缺点：

- **对CPU资源敏感；**
- **无法处理浮动垃圾；**
- **它使用的回收算法-“标记-清除”算法会导致收集结束时会有大量空间碎片产生。**

##### 7.G1收集器

**G1 (Garbage-First)是一款面向服务器的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器. 以极高概率满足GC停顿时间要求的同时,还具备高吞吐量性能特征.**

将堆内存划分为大小固定的独立区域，后台维护优先级表。优先回收垃圾最多区域。

相对于CMS改进：

①基于标记整理算法，不产生内存碎片。

②在不牺牲吞吐量前提下实现最短停顿垃圾回收。

被视为JDK1.7中HotSpot虚拟机的一个重要进化特征。它具备一下特点：

- **并行与并发**：G1能充分利用CPU、多核环境下的硬件优势，使用多个CPU（CPU或者CPU核心）来缩短Stop-The-World停顿时间。部分其他收集器原本需要停顿Java线程执行的GC动作，G1收集器仍然可以通过并发的方式让java程序继续执行。
- **分代收集**：虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但是还是保留了分代的概念。
- **空间整合**：与CMS的“标记--清理”算法不同，G1从整体来看是基于“标记整理”算法实现的收集器；从局部上来看是基于“复制”算法实现的。
- **可预测的停顿**：这是G1相对于CMS的另一个大优势，降低停顿时间是G1 和 CMS 共同的关注点，但G1 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内。

G1收集器的运作大致分为以下几个步骤：

- **初始标记**
- **并发标记**
- **最终标记**
- **筛选回收**

**G1收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的Region(这也就是它的名字Garbage-First的由来)**。这种使用Region划分内存空间以及有优先级的区域回收方式，保证了GF收集器在有限时间内可以尽可能高的收集效率（把内存化整为零）。

### **7.网络编程**

#### 1.阻塞IO

读写数据时会发生阻塞。

工作流程：

用户线程发出IO请求，内核检查数据是否就绪，用户线程阻塞等待数据就绪；

数据就绪后，内核将数据复制到用户线程中，返回IO执行结果到用户线程，用户线程解除阻塞开始处理数据。

#### 2.非阻塞IO

用户线程发出IO请求，无须阻塞就可以得到内核返回结果。内核返回false，没准备好，需要稍后发起IO操作。

一旦数据准备好，并且再次收到用户线程请求，内核就将数据复制到用户线程中并将结果通知用户线程。

在非阻塞IO中，用户线程需要不断询问内核数据是否就绪，在未就绪时可以处理其他任务。

#### 3.多路复用IO

Java NIO模型基于此。会有一个Selector线程不断轮询多个Socket状态，在Socket有读写事件时，才通知用户线程进行IO操作。

多路复用IO可以一个线程管理多个Socket，并且只有真正Socket读写事件时才会使用操作系统的IO资源，节约资源。

#### 4.信号驱动IO

用户线程发起IO操作，系统为该请求注册一个信号函数，然后用户线程处理其他任务；

当内核数据就绪时，系统发送信号给用户线程，用户线程会在信号函数中调用对应IO操作。

#### 5.异步IO

用户线程不关心IO操作，只需要发起请求，接收到内核返回成功或失败信号时，说明IO操作完成，直接使用数据。

#### 6.Java IO

5个类

File、OutputStream、InputStream、Writer、Reader

一个接口

Serializable

#### 7.NIO

三个核心：Selector（选择器）、Channel（通道）、Buffer（缓存区）

Selector用于监听多个Channel事件



NIO与传统IO区别：

1. IO面向流。NIO面向缓冲区，方便在缓存中对数据进行前后移动操作。
2. 传统IO阻塞。NIO非阻塞。



![img](v2-e7113f02333c7ca9a09208900d396205_720w.jpg)



Channel:

和流Stream类似，不同的是Stream单向，Channel双向。

Buffer：

一个容器，读写必须经过Buffer。

![img](v2-8018d89da46799598a986f78e54247fc_720w.jpg)



Selector:

选择器，检测多个注册的Channel是否有IO事件。一个Selector管理多个Channel。

### 8.JVM类加载机制

#### 8.1 类文件结构

##### 介绍一下类文件结构吧！

根据 Java 虚拟机规范，类文件由单个 ClassFile 结构组成：

```java
ClassFile {
    u4             magic; //Class 文件的标志
    u2             minor_version;//Class 的小版本号
    u2             major_version;//Class 的大版本号
    u2             constant_pool_count;//常量池的数量
    cp_info        constant_pool[constant_pool_count-1];//常量池
    u2             access_flags;//Class 的访问标记
    u2             this_class;//当前类
    u2             super_class;//父类
    u2             interfaces_count;//接口
    u2             interfaces[interfaces_count];//一个类可以实现多个接口
    u2             fields_count;//Class 文件的字段属性
    field_info     fields[fields_count];//一个类会可以有个字段
    u2             methods_count;//Class 文件的方法数量
    method_info    methods[methods_count];//一个类可以有个多个方法
    u2             attributes_count;//此类的属性表中的属性数
    attribute_info attributes[attributes_count];//属性表集合
}
```

**Class文件字节码结构组织示意图** ：

![类文件字节码结构组织示意图](b11ec323269c0b4eca43c663eaa42d8f.png)

下面会按照上图结构按顺序详细介绍一下 Class 文件结构涉及到的一些组件。

1.  **魔数:**  确定这个文件是否为一个能被虚拟机接收的 Class 文件。 
2.  **Class 文件版本** ：Class 文件的版本号，保证编译正常执行。
3.  **常量池** ：常量池主要存放两大常量：字面量和符号引用。
4.  **访问标志** ：标志用于识别一些类或者接口层次的访问信息，包括：这个 Class 是类还是接口，是否为 public 或者 abstract 类型，如果是类的话是否声明为 final 等等。
5.  **当前类索引,父类索引** ：类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名，由于 Java 语言的单继承，所以父类索引只有一个，除了 `java.lang.Object` 之外，所有的 java 类都有父类，因此除了 `java.lang.Object` 外，所有 Java 类的父类索引都不为 0。
6.  **接口索引集合** ：接口索引集合用来描述这个类实现了那些接口，这些被实现的接口将按`implents`(如果这个类本身是接口的话则是`extends`) 后的接口顺序从左到右排列在接口索引集合中。
7.  **字段表集合**  ：描述接口或类中声明的变量。字段包括类级变量以及实例变量，但不包括在方法内部声明的局部变量。
8.  **方法表集合** ：类中的方法。
9.  **属性表集合** ： 在 Class 文件，字段表，方法表中都可以携带自己的属性表集合。

**Class文件中的常量池**

在Class文件结构中，最头的4个字节用于存储魔数Magic Number，用于确定一个文件是否能被JVM接受，再接着4个字节用于存储版本号，前2个字节存储次版本号，后2个存储主版本号，再接着是用于存放常量的常量池，由于常量的数量是不固定的，所以常量池的入口放置一个U2类型的数据(constant_pool_count)存储常量池容量计数值。

常量池主要用于存放两大类常量：字面量(Literal)和符号引用量(Symbolic References)，字面量相当于Java语言层面常量的概念，如文本字符串，声明为final的常量值等，符号引用则属于编译原理方面的概念，包括了如下三种类型的常量：

- 类和接口的全限定名
- 字段名称和描述符
- 方法名称和描述符

**静态常量池和动态常量池的关系以及区别**

静态常量池存储的是当class文件被java虚拟机加载进来后存放在方法区的一些字面量和符号引用，字面量包括字符串，基本类型的常量，符号引用其实引用的就是常量池里面的字符串，但符号引用不是直接存储字符串，而是存储字符串在常量池里的索引。

动态常量池是当class文件被加载完成后，java虚拟机会将静态常量池里的内容转移到动态常量池里，在静态常量池的符号引用有一部分是会被转变为直接引用的，比如说类的静态方法或私有方法，实例构造方法，父类方法，这是因为这些方法不能被重写其他版本，所以能在加载的时候就可以将符号引用转变为直接引用，而其他的一些方法是在这个方法被第一次调用的时候才会将符号引用转变为直接引用的。
方法区里存储着class文件的信息和动态常量池,class文件的信息包括类信息和静态常量池。可以将类的信息是对class文件内容的一个框架，里面具体的内容通过常量池来存储。

动态常量池里的内容除了是静态常量池里的内容外，还将静态常量池里的符号引用转变为直接引用，而且动态常量池里的内容是能动态添加的。例如调用String的intern()方法就能将string的值添加到String常量池中，这里String常量池是包含在动态常量池里的，但在jdk1.8后，将String常量池放到了堆中。

**常量池的好处**

常量池是为了避免频繁的创建和销毁对象而影响系统性能，其实现了对象的共享。 
例如字符串常量池，在编译阶段就把所有的字符串文字放到一个常量池中。 
（1）节省内存空间：常量池中所有相同的字符串常量被合并，只占用一个空间。 
（2）节省运行时间：比较字符串时，==比equals()快。对于两个引用变量，只用==判断引用是否相等，也就可以判断实际值是否相等。

#### 8.2 类加载过程

**知道类加载的过程吗**？

类加载过程：**加载->连接->初始化**。连接过程又可分为三步:**验证->准备->解析**。



![img](v2-a7dfe1365af25ff420217f787ee31216_720w.jpg)



加载：

JVM读取Class文件，创建java.lang.Class对象

验证：

只有通过验证的Class文件才能被JVM加载

准备：

为方法区中的类变量分配内存空间并设置初始值

解析：

JVM将常量池中的符号引用替换为直接引用

初始化：

通过执行类构造器的<client>方法为类初始化。

##### 那加载这一步做了什么?

类加载过程的第一步，主要完成下面3件事情：

1. 通过全类名获取定义此类的二进制字节流
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
3. 在内存中生成一个代表该类的 Class 对象,作为方法区这些数据的访问入口

虚拟机规范多上面这3点并不具体，因此是非常灵活的。比如："通过全类名获取定义此类的二进制字节流" 并没有指明具体从哪里获取、怎样获取。比如：比较常见的就是从 ZIP 包中读取（日后出现的JAR、EAR、WAR格式的基础）、其他文件生成（典型应用就是JSP）等等。

**一个非数组类的加载阶段（加载阶段获取类的二进制字节流的动作）是可控性最强的阶段，这一步我们可以去完成还可以自定义类加载器去控制字节流的获取方式（重写一个类加载器的 `loadClass()` 方法）。数组类型不通过类加载器创建，它由 Java 虚拟机直接创建。**

类加载器、双亲委派模型也是非常重要的知识点，这部分内容会在后面的问题中单独介绍到。

加载阶段和连接阶段的部分内容是交叉进行的，加载阶段尚未结束，连接阶段可能就已经开始了。

#### 8.3 类加载器

JVM提供3个：启动类加载器、扩展类加载器、应用程序类加载器



![img](v2-a9193e5b6d47da137b284952c156304f_720w.jpg)

**知道哪些类加载器**?

JVM 中内置了三个重要的 ClassLoader，除了 BootstrapClassLoader 其他类加载器均由 Java 实现且全部继承自`java.lang.ClassLoader`：

1. **BootstrapClassLoader(启动类加载器)** ：最顶层的加载类，由C++实现，负责加载 `%JAVA_HOME%/lib`目录下的jar包和类或者或被 `-Xbootclasspath`参数指定的路径中的所有类。
2. **ExtensionClassLoader(扩展类加载器)** ：主要负责加载目录 `%JRE_HOME%/lib/ext` 目录下的jar包和类，或被 `java.ext.dirs` 系统变量所指定的路径下的jar包。
3. **AppClassLoader(应用程序类加载器)** :面向我们用户的加载器，负责加载当前应用classpath下的所有jar包和类。
4. **自定义的类加载器**：用户通过继承java.lang.ClassLoader自定义类加载器

#### 8.3 双亲委派机制

JVM通过双亲委派对类加载。

##### 双亲委派模型介绍

每一个类都有一个对应它的类加载器。系统中的 ClassLoder 在协同工作的时候会默认使用 **双亲委派模型** 。即在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。加载的时候，自己不会加载，而是首先会把该请求委派该父类加载器的 `loadClass()` 处理，其父类接收到委派请求后，又会委托给他的父类，依此类推。这样所有类的加载请求都委托给了**启动类加载器**`BootstrapClassLoader` 。当父类加载器无法处理时，才由自己来处理。若父类收到委派，发现自己也无法加载（通常由于该类的Class文件在父类的类加载路径不存在），会将该信息反馈给子类，向下委托。直到加载成功，否则报ClassNotFound。

![ClassLoader](80f3af661a8724c4dee84411c166c03d.png)

每个类加载都有一个父类加载器，我们通过下面的程序来验证。

```java
public class ClassLoaderDemo {
    public static void main(String[] args) {
        System.out.println("ClassLodarDemo's ClassLoader is " + ClassLoaderDemo.class.getClassLoader());
        System.out.println("The Parent of ClassLodarDemo's ClassLoader is " + ClassLoaderDemo.class.getClassLoader().getParent());
        System.out.println("The GrandParent of ClassLodarDemo's ClassLoader is " + ClassLoaderDemo.class.getClassLoader().getParent().getParent());
    }
}
```

Output

```
ClassLodarDemo's ClassLoader is sun.misc.Launcher$AppClassLoader@18b4aac2
The Parent of ClassLodarDemo's ClassLoader is sun.misc.Launcher$ExtClassLoader@1b6d3586
The GrandParent of ClassLodarDemo's ClassLoader is null
```

`AppClassLoader`的父类加载器为`ExtClassLoader`
`ExtClassLoader`的父类加载器为null，**null并不代表`ExtClassLoader`没有父类加载器，而是 `Bootstrap ClassLoader`** 。

其实这个双亲翻译的容易让别人误解，我们一般理解的双亲都是父母，这里的双亲更多地表达的是“父母这一辈”的人而已，并不是说真的有一个 Mather ClassLoader 和一个 Father ClassLoader 。另外，类加载器之间的“父子”关系也不是通过继承来体现的，是由“优先级”来决定。官方API文档对这部分的描述如下:

> The Java platform uses a delegation model for loading classes. **The basic idea is that every class loader has a "parent" class loader.** When loading a class, a class loader first "delegates" the search for the class to its parent class loader before attempting to find the class itself.

##### 双亲委派模型实现源码分析

双亲委派模型的实现代码非常简单，逻辑非常清晰，都集中在 `java.lang.ClassLoader` 的 `loadClass()` 中，相关代码如下所示。

```java
private final ClassLoader parent; 
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // 首先，检查请求的类是否已经被加载过
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {//父加载器不为空，调用父加载器loadClass()方法处理
                        c = parent.loadClass(name, false);
                    } else {//父加载器为空，使用启动类加载器 BootstrapClassLoader 加载
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                   //抛出异常说明父类加载器无法完成加载请求
                }
                
                if (c == null) {
                    long t1 = System.nanoTime();
                    //自己尝试加载
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

##### 双亲委派模型带来了什么好处呢？

**保证类的唯一性与安全性。**双亲委派模型保证了Java程序的稳定运行，可以避免类的重复加载（JVM 区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器加载产生的是两个不同的类），也保证了 Java 的核心 API 不被篡改。如果不用没有使用双亲委派模型，而是每个类加载器加载自己的话就会出现一些问题，比如我们编写一个称为 `java.lang.Object` 类的话，那么程序运行的时候，系统就会出现多个不同的 `Object` 类。

##### 如果我们不想用双亲委派模型怎么办？

为了避免双亲委托机制，我们可以自己定义一个类加载器，然后重载 `loadClass()` 即可。

##### 如何自定义类加载器?

除了 `BootstrapClassLoader` 其他类加载器均由 Java 实现且全部继承自`java.lang.ClassLoader`。如果我们要自定义自己的类加载器，很明显需要继承 `ClassLoader`。

**双亲委派流程**



![img](v2-5e3ccbab338bb19fada17cc75f52ebfa_720w.jpg)



### 9.总结

![img](v2-cc8549e92d3f297334b3af985182c45f_720w.jpg)