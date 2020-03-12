**JVM知识总结**

# **JVM架构图**

​      **灰色-线程私有，独占；**

​      **橙黄色-所有线程共享，存在垃圾回收**

![](images/JVM架构图.png)



 

# **JVM的类加载器**

![](images/ClassLoader.png)

1.    Bootstrap ClassLoader（根加载器）
2.    Extension ClassLoader（扩展类加载器）
3.    System ClassLoader（应用程序类加载器）

# **双亲委派机制**

1. 当AppClassLoader加载一个class时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器ExtClassLoader去完成。
2. 当ExtClassLoader加载一个class时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给BootStrapClassLoader去完成。
3. 如果BootStrapClassLoader加载失败（例如在$JAVA_HOME/jre/lib里未查找到该class），会使用ExtClassLoader来尝试加载；
4. 若ExtClassLoader也加载失败，则会使用AppClassLoader来加载，如果AppClassLoader也加载失败，则会报出异常ClassNotFoundException。

# **沙箱安全机制**

​      沙箱安全机制是由基于双亲委派机制上 采取的一种JVM的自我保护机制,假设你要写一个java.lang.String 的类,由于双亲委派机制的原理,此请求会先交给Bootstrap试图进行加载,但是Bootstrap在加载类时首先通过包和类名查找rt.jar中有没有该类,有则优先加载rt.jar包中的类,因此就**保证了java的运行机制不会被破坏**

 

# **Native-本地方法**

​      Native是一个关键字，JVM是依赖于操作系统的，声明为native的方法就是本地方法，只有声明，没有方法体，表示java的范围只到这里了，native的方法会装载到JVM的**本地方法栈（Native Method Stack）**里面，接下来的实现就是需要调用C/C++的第三方函数库去实现。

 

# **PC寄存器（Program Counter Register）**

​      记录了方法之间的调用和执行情况，类似于排班表，用来**存储指向下一条指令的地址**，也即将要执行的指令代码，它是当前线程所执行的字节码的行号记录器。

 

# **Method Area方法区**

供各线程共享的运行时内存区域。**它存储了每一个类的结构信息**，例如运行时常量池（Runtime Constant Pool）、字段和方法数据、构造函数和普通方法的字节码内容，上面讲的是规范，在不同虚拟机里头实现是不一样的，最典型的就是永生代（PermGen Space）和元空间（Metaspace）。

 

# Stack 栈

栈管运行，堆管内存

栈也叫栈内存，主管Java程序的运行，是在线程创建时创建，它的生命周期时跟随线程的生命期，线程结束栈内存也就释放了，对于栈来说不存在垃圾回收问题。只要线程一结束该栈就over，生命周期和线程是一致的，是线程私有的。8种基本数据类型的变量+对象的引用变量+实例方法都是在函数的栈内存中分配



栈帧中主要保存3类数据

本地变量：输入参数和输出参数以及方法内的变量

栈操作：记录出栈、入栈操作

栈帧数据：包括类文件、方法等



**每个方法执行的同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出口等信息**，每一个方法从调用直至执行完毕的过程，就对应着一个栈帧在虚拟机中入栈到出栈的过程。**栈的大小和具体JVM的实现有关，通常在256K~756K之间,与等于1Mb左右。** 



# Heap 堆

一个JVM实例只存在一个堆内存，堆内存的大小是可以调节的。类加载器读取了类文件后，需要把类、方法、常变量放到堆内存中，保存所有引用类型的真实信息，以方便执行器执行，堆内存分为三部分 ：

- Young Generation Space  	新生区		Young/New 

- Tenure generation space  	养老区		Old/ Tenure 

- Permanent Space              	永久区		Perm



![](images/heap.png)

新生区是类的诞生、成长、消亡的区域，一个类在这里产生，应用，最后被垃圾回收器收集，结束生命。

新生区又分为两部分： 伊甸区（Eden space）和幸存者区（Survivor pace） ，所有的类都是在伊甸区被new出来的。

幸存区有两个： 0区（Survivor 0 space）和1区（Survivor 1 space）。

> 当伊甸园的空间用完时，程序又需要创建对象，JVM的垃圾回收器将对伊甸园区进行垃圾回收(**Minor GC**)，将伊甸园区中的不再被其他对象所引用的对象进行销毁。然后将伊甸园中的剩余对象移动到幸存 0区。若幸存 0区也满了，再对该区进行垃圾回收，然后移动到 1 区。那如果1 区也满了呢？再移动到养老区。若养老区也满了，那么这个时候将产生**MajorGC（FullGC）**，进行养老区的内存清理。若养老区执行了Full GC之后发现依然无法进行对象的保存，就会产生OOM异常“OutOfMemoryError”。

**如果出现java.lang.OutOfMemoryError: Java heap space异常，说明Java虚拟机的堆内存不够。原因有二：**

**（1）Java虚拟机的堆内存设置不够，可以通过参数-Xms、-Xmx来调整。**

**（2）代码中创建了大量大对象，并且长时间不能被垃圾收集器收集（存在被引用）。**

------

![](images/heap_space.png)

​							    **MinorGC的过程（复制->清空->互换）**  

**1：eden、SurvivorFrom 复制到 SurvivorTo，年龄+1** 

首先，当Eden区满的时候会触发第一次GC,把还活着的对象拷贝到SurvivorFrom区，当Eden区再次触发GC的时候会扫描Eden区和From区域,对这两个区域进行垃圾回收，经过这次回收后还存活的对象,则直接复制到To区域（如果有对象的年龄已经达到了老年的标准，则赋值到老年代区），同时把这些对象的年龄+1

**2：清空 eden、SurvivorFrom** 

然后，清空Eden和SurvivorFrom中的对象，也即**复制之后有交换，谁空谁是to**

**3：SurvivorTo和 SurvivorFrom 互换** 

最后，SurvivorTo和SurvivorFrom互换，原SurvivorTo成为下一次GC时的SurvivorFrom区。**部分对象会在From和To区域中复制来复制去,如此交换15次(由JVM参数MaxTenuringThreshold决定,这个参数默认是15)**,最终如果还是存活,`就存入到老年代`



实际而言，方法区（Method Area）和堆一样，是各个线程共享的内存区域，它用于存储虚拟机加载的：类信息+普通常量+静态常量+编译器编译后的代码等等，**虽然JVM规范将方法区描述为堆的一个逻辑部分，但它却还有一个别名叫做Non-Heap(非堆)，目的就是要和堆分开**。 



在Java8中，永久代已经被移除，被一个称为元空间的区域所取代。元空间的本质和永久代类似。

> 元空间与永久代之间最大的区别在于：
>
> 永久带使用的JVM的堆内存，但是java8以后的**元空间并不在虚拟机中而是使用本机物理内存**。
>
> 因此，默认情况下，元空间的大小仅受本地内存限制。类的元数据放入 native memory, 字符串池和类的静态变量放入 java 堆中，这样可以加载多少类的元数据就不再由MaxPermSize 控制, 而由系统的实际可用空间来控制。



## 堆内存调优



| **-Xms**               | **设置初始分配大小，默认为物理内存的1/64** |
| ---------------------- | ------------------------------------------ |
| **-Xmx**               | **最大分配内存，默认为物理内存的1/4**      |
| **-XX:PrintGCDetails** | **输出详细的GC处理日志**                   |



```java
public static void main(String[] args){
	long maxMemory = Runtime.getRuntime().maxMemory() ;//返回 Java 虚拟机试图使用的最大内存量。
	long totalMemory = Runtime.getRuntime().totalMemory() ;//返回 Java 虚拟机中的内存总量。
	System.out.println("MAX_MEMORY = " + maxMemory + "（字节）、" + (maxMemory / (double)1024 / 1024) + "MB");
	System.out.println("TOTAL_MEMORY = " + totalMemory + "（字节）、" + (totalMemory / (double)1024 / 1024) + "MB");
}
```

输出结果：

```java
-Xmx:MAX_MEMORY:	1883242496	字节,	1796.0	MB
-Xms:TOTAL_MEMORY:	128974848	字节,	123.0	MB
```

![](images/heap_space_code_result.png)



GC的打印结果：

![](images/GC_result.png)