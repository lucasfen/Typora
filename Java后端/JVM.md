## JVM主要组成

**类加载**：根据给定的全限定类名来装载class文件到运行时数据区中的方法区

**执行引擎：**执行class文件中的指令

**本地库接口：**与本地库交互，是其他变成语言交互的接口

**运行时数据区：**即JVM内存

首先通过编译器把 Java 代码转换成字节码，类加载器（ClassLoader）再把字节码加载到内存中，将其放在运行时数据区（Runtime data area）的方法区内，而字节码文件只是 JVM 的一套指令集规范，并不能直接交给底层操作系统去执行，因此需要特定的命令解析器执行引擎（Execution Engine），将字节码翻译成底层系统指令，再交由 CPU 去执行，而这个过程中需要调用其他语言的本地库接口（Native Interface）来实现整个程序的功能。

![image-20220415090923437](C:\Users\fqh0722\AppData\Roaming\Typora\typora-user-images\image-20220415090923437.png)

## JVM运行时数据区

程序计数器：指示下一条需要执行的指令

虚拟机栈：用于存储局部变量，操作数栈，动态链接，方法出口等信息

本地方法栈：与虚拟机栈作用相同，只不过虚拟机栈是服务 Java 方法的，而本地方法栈是为虚拟机调用 Native 方法服务的

堆：存放对象实例

方法区：存储被虚拟机加载的类信息，常量，静态变量，即使编译后的代码等数据

## 堆和栈的区别

1. 堆的物理地址分配是不连续的，性能较慢；栈的物理地址分配是连续的，性能较快
2. 堆存放对象的实例和数组；栈存放局部变量，操作数栈，返回结果等
3. 堆是线程共享的；栈是线程私有的
4. 堆因为是不连续的，所以分配的内存是在运行期确认的，因此大小不固定。一般堆大小远远大于栈；栈是连续的，所以分配的内存大小要在编译期就确认，大小是固定的。

## 类加载

### 概念

- 虚拟机把描述类的数据从Class文件加载到内存，并对数据进行校验，解析和初始化，最终形成可以被虚拟机直接使用的java类型。
- 类的加载指的是将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个 java.lang.Class对象，用来封装类在方法区内的数据结构。

### 类加载过程

1. **加载：**

   1. 通过一个类的全限定名来获取定义此类的二进制字节流
   2. 将这个字节流所代表的静态存储结构转化为方法的运行时数据结构
   3. 在堆内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口

2. **验证**：

   检查加载的class文件的正确性

3. **准备**：

   给类中的静态变量分配内存并设置类变量初始值

4. **解析：**

   虚拟机将常量池中的符号引用替换成直接引用的过程。符号引用就理解为 

   一个标示，而在直接引用直接指向内存中的地址；

5. **初始化**：

   对静态变量和静态代码块执行初始化工作。

### 类加载的双亲委派机制

- 类加载器从上至下：启动类加载器，扩展类类加载器，应用程序类加载器，自定义类加载器
- 当类加载器收到一个类加载请求时，将请求委派给父类加载器去完成，只有当父类加载器无法完成加载请求时，子类加载器才会尝试自己去完成类加载请求
  1. 检查类是否加载过，如果没有，则委托父类加载器去加载
  2. 父类加载器加载失败，抛出异常，则尝试自己去加载
- 双亲委派机制的必要性
  1. 保证被加载类的唯一性
  2. 避免核心API库被篡改

## 对象的创建

### Java中对象创建的方式

- 使用new关键字，调用了构造函数
- 使用Class的newInstance方法，调用了构造函数
- 使用Constructor类的newInstance方法，调用了构造函数
- 使用clone方法，没有调用了构造函数
- 使用反序列化，没有调用构造函数

### 对象创建流程

1. 检查方法区中是否已经加载相应的类，如果没有，则先执行相应的类加载。
2. 为对象**分配内存**
3. 将对象初始化零值
4. 设置**对象头**
5. 执行初始化方法，即按照程序员意愿进行初始化，类似属性赋值。并且执行构造函数

### 对象分配内存

在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类 加载完成后便可完全确定，为对象分配空间的任务等同于把 一块确定大小的内存从Java堆中划分出来。 

这个步骤有**两个问题：** 

1.如何划分内存。 

2.在并发情况下， 可能出现正在给对象A分配内存，指针还没来得及修改，对象B又同时使用了原来的指针来分配内存的情况。 

**划分内存的方法：** 

- 指针碰撞（Bump the Pointer）(默认用指针碰撞) ：如果Java堆中内存是绝对规整的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间那边挪动一段与对象大小相等的距离
- 空闲列表（Free List）：如果Java堆中的内存并不是规整的，已使用的内存和空 闲的内存相互交错，那就没有办法简单地进行指针碰撞了，**虚拟机就必须维护一个列表，记录上哪些内存块是可用的**，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录 

**解决并发问题的方法：** 

CAS（compare and swap）：虚拟机采用CAS配上失败重试的方式保证更新操作的原子性来对分配内存空间的动作进行同步处理。 

本地线程分配缓冲（Thread Local Allocation Buffer,TLAB）把内存分配的动作按照线程划分在不同的空间之中进行，即**每个线程在Java堆中预先分配一小块内存**。通过**­XX:+/­** **UseTLAB**参数来设定虚拟机是否使用TLAB(JVM会默认开启**­XX:+****UseTLAB**)，­XX:TLABSize 指定TLAB大小。

### 内存分配策略

#### 堆上分配

1. 大量对象被分配在eden区， eden区满后会触发minor gc,可能99%以上的对象成为垃圾被回收掉。
2. 剩余存活对象被挪到为空的那块survivor区，下一次eden区满后优惠触发minor gc, 把eden区和survivor区垃圾对象回收，把剩余存活对象一次性挪动到另外一块为空的survivor区

#### 大对象直接进入老年代

大对象就是需要大量连续内存空间的对象（比如：字符串、数组）。JVM参数 -XX:PretenureSizeThreshold 可以设置大对象的大小，如果对象超过设置大小会直接进入老年代，不会进入年轻代，这个参数只在 Serial 和ParNew两个收集器下有效。为了避免为大对象分配内存时的复制操作而降低效率。

#### 长期存活的对象进入老年代

虚拟机采用了分代收集的思想来管理内存，那么内存回收时就必须能识别哪些对象应放在新生代，哪些对象应放在老年代中。为了做到这一点，虚拟机给每个对象一个对象年龄（Age）计数器。如果对象在 Eden 出生并经过第一次 Minor GC 后仍然能够存活，并且能被Survivor 容纳的话，将被移动到 Survivor空间中，并将对象年龄设为1。对象在 Survivor 中每熬过一次 MinorGC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁，CMS收集器默认6岁，不同的垃圾收集器会略微有点不同），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 -XX:MaxTenuringThreshold 来设置

#### 对象动态年龄判断

当前放对象的Survivor区域里(其中一块区域，放对象的那块s区)，一批对象的总大小大于这块Survivor区域内存大小的 50%(-XX:TargetSurvivorRatio可以指定)，那么此时**大于等于**这批对象年龄最大值的对象，就可以直接进入老年代了，例如Survivor区域里现在有一批对象，年龄1+年龄2+年龄n的多个年龄对象总和超过了Survivor区域的50%，此时就会把年龄n(含)以上的对象都放入老年代。这个规则其实是希望那些可能是长期存活的对象，尽早进入老年代。对象动态年龄判断机制一般是在minor gc之后触发的。

#### 老年代空间分配担保机制

年轻代每次**minor gc**之前JVM都会计算下老年代**剩余可用空间** ，如果这个可用空间小于年轻代里现有的所有对象大小之和(**包括垃圾对象**) 

就会看一个“-XX:-HandlePromotionFailure”(jdk1.8默认就设置了)的参数是否设置了，如果有这个参数，就会看看老年代的可用内存大小，是否大于之前每一次minor gc后进入老年代的对象的**平均大小**。如果上一步结果是小于或者之前说的参数没有设置，那么就会触发一次Full gc，对老年代和年轻代一起回收一次垃圾，如果回收完还是没有足够空间存放新的对象就会发生"OOM"。当然，如果minor gc之后剩余存活的需要挪动到老年代的对象大小还是大于老年代可用空间，那么也会触发full gc，full gc完之后如果还是没有空间放minor gc之后的存活对象，则也会发生“OOM”

#### 栈上分配

我们通过JVM内存分配可以知道JAVA中的对象都是在堆上进行分配，当对象没有被引用的时候，需要依靠GC进行回收内存，如果对象数量较多的时候，会给GC带来较大压力，也间接影响了应用的性能。为了减少临时对象在堆内分配的数量，JVM通过**逃逸分析**确定该对象不会被外部访问。如果不会逃逸可以将该对象在**栈上分配**内存，这样该对象所占用的内存空间就可以随栈帧出栈而销毁，就减轻了垃圾回收的压力。 

**对象逃逸分析**：就是分析对象动态作用域，当一个对象在方法中被定义后，它可能被外部方法所引用，例如作为调用参数传递到其他地方中

**标量替换：**通过逃逸分析确定该对象不会被外部访问，并且对象可以被进一步分解时，**JVM不会创建该对象**，而是将该对象成员变量分解若干个被这个方法使用的成员变量所代替，这些代替的成员变量在栈帧或寄存器上分配空间，这样就 不会因为没有一大块连续空间导致对象内存不够分配。开启标量替换参数(-XX:+EliminateAllocations)，JDK7之后默认 开启。 

**标量与聚合量：**标量即不可被进一步分解的量，而JAVA的基本数据类型就是标量（如：int，long等基本数据类型以及reference类等），标量的对立就是可以被进一步分解的量，而这种量称之为聚合量。而在JAVA中对象就是可以被进一步分解的聚合量。

### 对象头

在HotSpot虚拟机中，对象在内存中存储的布局可以分为3块区域：对象头（Header）、 实例数据（Instance Data）和对齐填充（Padding）。 HotSpot虚拟机的对象头包括两部分信息，第一部分用于存储对象自身的运行时数据即Mark Word， 如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时 间戳等。对象头的另外一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例

![image-20220415110751856](C:\Users\fqh0722\AppData\Roaming\Typora\typora-user-images\image-20220415110751856.png)

JVM一般是这样使用锁和Mark Word的：

1. 当没有被当成锁时，这就是一个普通的对象，Mark Word记录对象的HashCode，锁标志位是01，是否偏向锁那一位是0。
2. 当对象被当做同步锁并有一个线程A抢到了锁时，锁标志位还是01，但是否偏向锁那一位改成1，前23bit记录抢到锁的线程id，表示进入偏向锁状态。
3. 当线程A再次试图来获得锁时，JVM发现同步锁对象的标志位是01，是否偏向锁是1，也就是偏向状态，Mark Word中记录的线程id就是线程A自己的id，表示线程A已经获得了这个偏向锁，可以执行同步锁的代码。
4. 当线程B试图获得这个锁时，JVM发现同步锁处于偏向状态，但是Mark Word中的线程id记录的不是B，那么线程B会先用CAS操作试图获得锁，这里的获得锁操作是有可能成功的，因为线程A一般不会自动释放偏向锁。如果抢锁成功，就把Mark Word里的线程id改为线程B的id，代表线程B获得了这个偏向锁，可以执行同步锁代码。如果抢锁失败，则继续执行步骤
5. 偏向锁状态抢锁失败，代表当前锁有一定的竞争，偏向锁将升级为轻量级锁。JVM会在当前线程的线程栈中开辟一块单独的空间，里面保存指向对象锁Mark Word的指针，同时在对象锁Mark Word中保存指向这片空间的指针。上述两个保存操作都是CAS操作，如果保存成功，代表线程抢到了同步锁，就把Mark Word中的锁标志位改成00，可以执行同步锁代码。如果保存失败，表示抢锁失败，竞争太激烈，继续执行步骤
6. 轻量级锁抢锁失败，JVM会使用自旋锁，自旋锁不是一个锁状态，只是代表不断的重试，尝试抢锁。从JDK1.7开始，自旋锁默认启用，自旋次数由JVM决定。如果抢锁成功则执行同步锁代码，如果失败则继续执行步骤
7. 自旋锁重试之后如果抢锁依然失败，同步锁会升级至重量级锁，锁标志位改为10。在这个状态下，未抢到锁的线程都会被阻塞。
   

## 垃圾收集器

### 判断对象是否存活

1. 引用计数法
2. 可达性分析法

### 可以作为GC ROOT的对象

1. 虚拟机栈中引用的对象
2. 本地方法栈中Native方法引用的对象
3. 方法区中类静态属性引用的对象
4. 方法区中常量引用的对象

### Minor GC和Full GC

- minor gc 会回收新生代，因为新生代对象存活时间很短，因此minor gc执行会很频繁，执行速度一般也很快；full gc会回收老年代和新生代和方法区， 很少执行，会比较慢

### 什么时候触发Full GC

1. 显示调用System.gc()时，可能会触发Full GC
2. 老年代空间不足，例如大对象直接进入老年代，或者长期存活的对象进入老年代，为了避免这种情况，可以尽量避免创建大对象，或者调高进入老年代的年龄阈值，或者调大新生代空间
3. 空间分配担保失败

### 垃圾回收算法

- 标记-清除算法：先标记出可以回收的对象，然后回收被标记的对象占用的空间

  优点：实现简单，不需要对象进行移动

  缺点：标记清除过程效率低，产生大量不连续的内存碎片

- 复制算法：把内存空间划分为两个相等的区域，每次只使用其中一个区域，垃圾收集时，遍历当前使用的区域，把存活的对象复制到另一个区域，最后将当前使用的区域的可回收对象进行回收

  优点：不用考虑内存碎片

  缺点：可用内存大小缩小为原来的一般

- 标记-整理算法：先标记出可以回收的对象，然后将所有存活的对象压缩到内存的一端，使他们紧凑的排列在一起，然后对段边界以外的内存进行回收

  优点：解决内存碎片问题

  缺点：需要进行局部对象移动，降低了效率

### 分代收集算法

当前商业虚拟机都采用**分代收集**的垃圾收集算法。分代收集算法，顾名思义是根据对象的**存活周期**将内存划分为几块。一般包括**年轻代**、**老年代**和 **永久代**，如图所示： 

![image-20220416001212499](C:\Users\fqh0722\AppData\Roaming\Typora\typora-user-images\image-20220416001212499.png)

### JVM的垃圾回收器

用于回收新生代的收集器 包括Serial、PraNew、Parallel Scavenge，回收老年代的收集器包括Serial Old、Parallel Old、CMS，还有用于回收整个Java堆的G1收集器。不同收集器之间的连线表示它们可以搭配使用。

Serial收集器（复制算法）：新生代单线程收集器，标记和清理都是单线程，优点是简单高效

ParNew收集器（复制算法）：新生代并行收集器，是Serial收集器的多线程版本

Parallel Scavenge收集器（复制算法）：新生代并行收集器，追求高吞吐量

Serial Old收集器 (标记-整理算法): 老年代单线程收集器，Serial收集器的老年代版本；

Parallel Old收集器 (标记-整理算法)： 老年代并行收集器，吞吐量优先，Parallel Scavenge收集器的老年代版本； 

CMS(Concurrent Mark Sweep)收集器（标记-清除算法）： 老年代并行收集器，以获取最短回收停顿时间为目标的收集器，具有高并发、低停顿的特点，追求最短GC回收停顿时间。 

G1(Garbage First)收集器 (标记-整理算法)： Java堆并行收集器，G1收集器是JDK1.7提供的一个新收集器，G1收集器基于“标记-整理”算法实现，也就是说不会产生内存碎片。此外，G1收集器不同于之前的收集器的一个重要特点是：G1回收的范围是整个Java堆(包括新生代，老年代)，而前六种收集器回收的范围仅限于新生代或老年代。 

![image-20220416002757934](C:\Users\fqh0722\AppData\Roaming\Typora\typora-user-images\image-20220416002757934.png)

### CMS垃圾收集器

CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。它非常符合在注重用户体验的应用上使用，它是HotSpot虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。工作流程如下：

1. **初始标记：** 暂停所有的其他线程(STW)，并记录下gc roots**直接能引用的对象**，**速度很快**。
2. **并发标记：** 并发标记阶段就是从GC Roots的直接关联对象开始遍历整个对象图的过程， 这个过程耗时较长但是不需要停顿用户线程， 可以与垃圾收集线程一起并发运行。因为用户程序继续运行，可能会有导致已经标记过的对象状态发生改变。 
3. **重新标记：** 重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短。主要用到三色标记里的增量更新算法(见下面详解)做重新标记。
4. **并发清理：**开启用户线程，同时GC线程开始对未标记的区域做清扫。这个阶段如果有新增对象会被标记为黑色不做任何处理，即浮动垃圾，这次GC无法处理。 
5. **并发重置：**重置本次GC过程中的标记数据。

### G1垃圾收集器

G1收集器是 JDK1.7提供的一个新收集器，G1收集器基于“标记-整理”算法实现，也就是说不会产生内存碎片。此外，G1收集器不同于之前的收集器的一个重要特点是：G1回收的范围是整个Java堆(包括新生代，老年代)，而其他收集器回收的范围仅限于新生代或老年代

- **并行与并发**：G1能充分利用CPU、多核环境下的硬件优势，使用多个CPU（CPU或者CPU核心）来缩短Stop-The-World停顿时间。部分其他收集器原本需要停顿Java线程来执行GC动作，G1收集器仍然可以通过并发的方式让java程序继续执行。 
- **分代收集**：虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但是还是保留了分代的概念。 
- **空间整合**：与CMS的“标记--清理”算法不同，G1从整体来看是基于“**标记整理**”算法实现的收集器；从局部上来看是基于“复制”算法实现的。 
- **可预测的停顿**：这是G1相对于CMS的另一个大优势，降低停顿时间是G1 和 CMS 共同的关注点，但G1 除了追求低停顿外，还能建立**可预测的停顿时间模型**，能让使用者明确指定在一个长度为M毫秒的时间片段(通过参数"**-** **XX:MaxGCPauseMillis**"指定)内完成垃圾收集。

### **分代垃圾回收器**工作过程

分代回收器有两个分区：老生代和新生代，新生代默认的空间占比总空间的1/3，老生代的默认占比是 2/3。 

新生代使用的是复制算法，新生代里有 3 个分区：Eden、To Survivor、From Survivor，它们的默认占比是 8:1:1，它的执行流程如下： 

1. 把 Eden + From Survivor 存活的对象放入 To Survivor 区；
2. 清空 Eden 和 From Survivor 分区； 
3. From Survivor 和 To Survivor 分区交换，From Survivor 变 To Survivor，To Survivor 变 From Survivor。 
4. 每次在 From Survivor 到 To Survivor 移动时都存活的对象，年龄就 +1，当年龄到达 15（默认配置是 15）时，升级为老生代。大对象也会直接进入老生代。老生代当空间占用到达某个值之后就会触发全局垃圾收回，一般使用标记整理的执行算法。以上这些循环往复就构成了整个分代垃圾回收的整体执行流程。

## 深拷贝与浅拷贝

浅拷贝（shallowCopy）只是增加了一个指针指向已存在的内存地址， 

深拷贝（deepCopy）是增加了一个指针并且申请了一个新的内存，使这个增加 的指针指向这个新的内存， 

使用深拷贝的情况下，释放内存的时候不会因为出现浅拷贝时释放同一个内存的错误

## JVM调优

###  JVM 调优的工具

JDK 自带了很多监控工具，都位于 JDK 的 bin 目录下，其中最常用的是jconsole 和 jvisualvm 这两款视图监控工具。 

- jconsole：用于对 JVM 中的内存、线程和类等进行监控； 

- jvisualvm：JDK 自带的全能分析工具，可以分析：内存快照、线程快照、程序死锁、监控内存的变化、gc 变化等。 

### 常用的 JVM 调优的参数 

- -Xms2g：初始化推大小为 2g； 
- -Xmx2g：堆最大内存为 2g； 
- -XX:NewRatio=4：设置年轻的和老年代的内存比例为 1:4； 
- -XX:SurvivorRatio=8：设置新生代 Eden 和 Survivor 比例为 8:2； 
- –XX:+UseParNewGC：指定使用 ParNew + Serial Old 垃圾回收器组合； 
- -XX:+UseParallelOldGC：指定使用 ParNew + ParNew Old 垃圾回收器组合；
- -XX:+UseConcMarkSweepGC：指定使用 CMS + Serial Old 垃圾回收器组合；
- -XX:+PrintGC：开启打印 gc 信息； 
- -XX:+PrintGCDetails：打印 gc 详细信息。

### JVM常用命令

**jps:**显示正在运行的虚拟机进程

```java
[root@VM_247_254_centos ~]#jps -lm
26176 org.apache.zookeeper.server.quorum.QuorumPeerMain /usr/local/zookeeper-3.4.10/bin/../conf/zoo.cfg
25044 /usr/local/apache-activemq-5.14.5//bin/activemq.jar start
23732 sun.tools.jps.Jps -lm
25446 org.apache.catalina.startup.Bootstrap start
```

**jstat:**用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。例如查看垃圾收集信息如下

```java
[root@VM_247_254_centos ~]# jstat -gc 25446
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
6400.0 6400.0  0.0   1601.4 51712.0  50837.0   128808.0   88450.0   67584.0 66167.7 7936.0 7630.5    401    5.939  10      1.247    7.186

```

**jmap:**用于生成headdump即堆转储快照

```java
jmap -dump:format=b,file=dump.dprof 25446
Dumping heap to /home/gem/dump.dprof ...
Heap dump file created
```

**jstack:**jstack用于生成java虚拟机当前时刻的线程快照。 线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等。 线程出现停顿的时候通过jstack来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做什么事情，或者等待什么资源。 如果java程序崩溃生成core文件，jstack工具可以用来获得core文件的java stack和native stack的信息，从而可以轻松地知道java程序是如何崩溃和在程序何处发生问题。另外，jstack工具还可以附属到正在运行的java程序中，看到当时运行的java程序的java stack和native stack的信息, 如果现在运行的java程序呈现hung的状态，jstack是非常有用的。

```java
jstack -l 25446 | more
2018-01-25 21:18:22
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.121-b13 mixed mode):

"main-EventThread" #174 daemon prio=5 os_prio=0 tid=0x00007feb692b7000 nid=0x6502 waiting on condition [0x00007feb32bb1000]
    //等待
   java.lang.Thread.State: WAITING (parking)
	at sun.misc.Unsafe.park(Native Method)
	- parking to wait for  <0x00000000f0a00860> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
	at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
	at java.util.concurrent.LinkedBlockingQueue.take(LinkedBlockingQueue.java:442)
	at org.apache.zookeeper.ClientCnxn$EventThread.run(ClientCnxn.java:501)

   Locked ownable synchronizers:
	- None

"main-SendThread(115.159.192.69:2181)" #173 daemon prio=5 os_prio=0 tid=0x00007feb694bd800 nid=0x6501 runnable [0x00007feb363ce000]
    //运行
   java.lang.Thread.State: RUNNABLE
	at sun.nio.ch.EPollArrayWrapper.epollWait(Native Method)
	at sun.nio.ch.EPollArrayWrapper.poll(EPollArrayWrapper.java:269)
	at sun.nio.ch.EPollSelectorImpl.doSelect(EPollSelectorImpl.java:93)
	at sun.nio.ch.SelectorImpl.lockAndDoSelect(SelectorImpl.java:86)
	- locked <0x00000000f09ef268> (a sun.nio.ch.Util$3)
	- locked <0x00000000f09ef258> (a java.util.Collections$UnmodifiableSet)
	- locked <0x00000000f09ef140> (a sun.nio.ch.EPollSelectorImpl)
	at sun.nio.ch.SelectorImpl.select(SelectorImpl.java:97)
	at org.apache.zookeeper.ClientCnxnSocketNIO.doTransport(ClientCnxnSocketNIO.java:349)
	at org.apache.zookeeper.ClientCnxn$SendThread.run(ClientCnxn.java:1141)

   Locked ownable synchronizers:
	- None

```

**jinfo:**实时查看和调整虚拟机运行参数。 之前的jps -v口令只能查看到显示指定的参数，如果想要查看未被显示指定的参数的值就要使用jinfo命令

```java
[root@VM_247_254_centos ~]# jinfo -flags 25446
Attaching to process ID 25446, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.121-b13
Non-default VM flags: -XX:CICompilerCount=2 -XX:InitialHeapSize=16777216 -XX:MaxHeapSize=262144000 -XX:MaxNewSize=87359488 -XX:MinHeapDeltaBytes=196608 -XX:NewSize=5570560 -XX:OldSize=11206656 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops
Command line:  -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat -Djava.io.tmpdir=/usr/local/tomcat/temp

```

