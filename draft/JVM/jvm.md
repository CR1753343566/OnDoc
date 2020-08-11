# JVM

[JDK,JRE,JVM](#jrejdkjvm)

[JVM基本概念](#jvm)

- [Java代码运行](#javarun) 
- [JVM内存区域](#jvmmemory) 
    - [线程私有区域](#ownmemory)
    - [线程共享区域](#allmemory)

[JVM运行时内存](#runtimememory)
- [新生代](#new)
- [MinorGC](#minor)
- [老年代](#old)
- [MajorGC](#major)
- [永久代](#permanent)

[元数据](#metadata)

## <span id="jrejdkjvm">JDK JRE JVM  之间的联系</span>

#### JRE 

JavaRuntimeEnvironment Java运行环境，也就是Java平台。所有Java程序都需要在JRE下才能运行，普通用户只运行已开发好的Java程序，安装JRE即可

#### JDK

JavaDevelopment Kit 程序开发者用来编译，调试Java程序用的开发工具包。JDK的工具也是Java程序也需要JRE才能运行。

#### JVM

JVM(JavaVirtualMachine，Java虚拟机)是JRE的一部分


## <span id="jvm">基本概念</span>

JVM是可运行java代码的假想计算机，包含一套字节码指令集，一组寄存器，一个栈，垃圾回收，堆和存储方法域。JVM运行在操作系统之上，与硬件没有直接交互

一个运行的Java虚拟机实例负责运行一个Java程序。当启动一个Java程序时，一个虚拟机实例就诞生了，当该程序关闭退出，这个虚拟机实例也就随之消亡。如果在同一台计算机上同时运行三个Java程序，将得到3个虚拟机实例。每个Java程序都运行于它自己的Java虚拟机实例中

#### <span id="javarun"> Java代码的执行</span>

1. 将java代码编译为class文件 javac 
2. 装载class ClassLoader
3. 执行class

    - 解释执行
    - 编译执行
Java源文件，通过编译器，产生对应的.class文件，也就是字节码文件，字节码通过Java虚拟机(JVM)的解释器，编译成特定机器上的机器码

1. Java源文件 ——> 编译器 ——> 字节码文件
2. 字节码文件 ——> JVM ——> 机器码

每个平台的解释器是不同的，但是实现的虚拟机相同，所以JAVA可以跨平台，当一个程序开始运行，虚拟机就开始实例化了，多个程序启动就会存在多个虚拟机实例。程序退出或者关闭，则虚拟机实例消亡，多个虚拟机实例之间数据不能共享

## <span id="jvmmemory">JVM内存区域</span>

#### <span id="ownmemory">线程私有区域</spam>

1. 程序计数器

    较小的内存空间，是当前线程执行的字节码的行号指示器，每个线程都有一个独立的程序计数器，执行Java方法计数器记录的是虚拟机字节码指令的地址

    程序计数器内存区域不会出现OutOfMemoryError的情况（内存溢出）

2. 虚拟机栈

    描述Java方法执行的内存模型，每个方法在执行的同时会创建一个栈帧(Stack Frame),用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每个方法从调用直至完成的过程，就对应一个栈帧在虚拟机栈中从入栈到出栈的过程

    栈帧用来存储数据和部分过程结果的数据结构，，同时也被用来处理动态链接 (Dynamic Linking)、 方法返回值和异常分派（ Dispatch Exception）。栈帧随着方法调用而创
    建，随着方法结束而销毁——无论方法是正常完成还是异常完成（抛出了在方法内未被捕获的异 常）都算作方法结束。 
 
 
3. 本地方法区

    本地方法栈服务的对象是JVM执行的native方法，其就是一个java调用非java代码的接口，作用是与操作系统和外部环境交互

#### <span id="allmemory">线程共享区域</span>

1. JAVA堆

堆是被线程共享的一块内存区域，创建的对象和数组都保存在Java堆内存中，也是垃圾收集器进行垃圾回收的最重要的内存区域。由于现代JVM采用的分代收集算法，因此Java堆从GC的角度还可以细分为
        
- 新生代 (Eden区、From Survivor 区、To Survivor区)
- 老年代

2. 方法区/永久代

用于存储被JVM加载的类信息、常量、静态变量、即时编译器编译后的代码等数据

类型信息：

1. 这个类的全限定名
2. 这个类的直接超类的全限定名
3. 这个类是类类型还是接口类型
4. 这个类型的访问修饰符
5. 任何直接超接口的全限定名的有序列表

运行时常量池（Runtime Constant Pool）是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述等信息外，还有一项信息是常量池 （Constant Pool Table），用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。 Java 虚拟机对Class文件的每一部分（自然也包括常量池）的格式都有严格的规定，每一个字节用于存储哪种数据都必须符合规范上的要求，这样才会 被虚拟机认可、装载和执行。 

HotSpot JVM 把GC分代收集扩展至方法区，即使用Java堆的永久代实现方法区， 这样HotSpot的垃圾收集器就可以像管理Java堆一样管理这部分内存, 而不必为方法区开发专门的内存管理器(永久代的内存回收的主要目标是针对常量池的回收和类型的卸载, 因此收益一般很小)。 


## <span id="runtimememory">JVM运行时内存</span>

Java堆从GC的角度可分为新生代(Eden区、From Survivor区和To Survivor区)和老年代

#### <span id="new">新生代</span>

新生代用来存放新生的对象，一般占据堆的1/3空间，由于频繁的创建对象，所以新生代会频繁触发MinorGC进行垃圾回收

新生代又分为Eden 区、ServivorFrom、ServivorTo三个区

1. Eden 区

    Java新对象的出生地(如果创建的对象占用内存很大，则直接分配到老年代)。当Eden区内存不够的时候会触发MinorGC，对新生代区进行一次垃圾回收

2. ServivorFrom 区

    上一次GC的幸存者，作为这一次GC的被扫描者

3. ServivorTo 区

    保留了一次MinorGC过程中的幸存者

#### <span id="minor">MinorGC 的过程 (复制 清空 互换)</span>

MinorGC采用复制清除算法

1. Eden、ServivorFrom 复制到 ServivorTo ,年龄 +1 

    首先把Eden区和ServivorFrom区域中存活的对象复制到ServicorTo区(检查年龄是否达到了老年的标准，将达到标准的对象复制到老年代区），同时把这些对象的年龄+1（如果 ServicorTo 不够位置了就放到老年区）

2. 清空 Eden、ServivorFrom

3. ServivorTo ServivorFrom 互换，原ServivorTo成为下一次GC时的ServivorFrom区

#### <span id="old">老年代</span>

存放应用程序中生命周期长的内存对象

老年代对象比较稳定，不会经常进行MajorGC，在进行MinorGC前一般都先进行了一次MinorGC，使得新生代有新的对象晋升入老年代，导致空间不够才触发。当无法找到足够大连续空间分配在给新创建的较大对象时也会触发一次MajorGC进行垃圾回收腾出空间

#### <span id="major">MajorGC 的过程</span> 

MajorGC采用标记清除算法

扫描一次所有老年代，标记出存活的对象，回收没有标记的对象,MajorGC的耗时比较长，因为要扫描再回收。MajorGC 会产生内存碎片，为了减 少内存损耗，我们一般需要进行合并或者标记出来方便下次直接分配。当老年代也满了装不下的时候，就会抛出OOM（Out of Memory）异常


#### <span id="permanent">永久代</span>

指内存的永久保存区域，主要存放Class和Meta（元数据）的信息，Class在被加载的时候被放入永久区域，它和存放实例的区域不同，GC不会在主程序运行期对永久区域进行清理，所以这也导致了永久代的区域会随着加载的Class的增多而胀满，最终导致OOM异常

## <span id="metadata">Java8与元数据</span>

在Java8中，永久代已经被删除，被一个称为"元数据区"（元空间）的区域所取代

元空间的本质和永久代类似，元空间与永久代之间大的区别在于：元空间并不在虚拟机中，而是使用本地内存。

因此，默认情况下，元空间的大小仅受本地内存限制。类的元数据放入 native memory, 字符串池和类的静态变量放入 java 堆中，这样可以加载多少类的元数据就不再由 MaxPermSize控制, 而由系统的实际可用空间来控制

JDK1.6 有永久代的说法，当永久代内存满了会抛出 PermGen Space 的内存溢出

JDK1.7和JDK1.8会出现堆内存溢出，并且JDK1.8中的PermSize和MaxPermGen已经失效，JDK 1.7 和 1.8 将字符串常量由永久代转移到堆中，并且 JDK 1.8 中已经不存在永久代

元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。因此，默认情况下，元空间的大小仅受本地内存限制，但可以通过以下参数来指定元空间的大小：

    -XX:MetaspaceSize 初始空间大小，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间就适当降低该值；如果释放的空间较少，那么在不超过MaxMetaspaceSize时，适当提高该值
    -XX:MaxMetaspaceSize 最大空间，默认没有限制

#### 为什么JDK 8中去除了永久代引入了元空间

1. 字符串存在永久代中，容易出现性能问题和内存溢出
2. 类及方法的信息等比较难确定其大小，因此对于永久代的大小指定比较困难，太小容易出现永久代溢出
3. 永久代会为GC带来不必要的复杂度，并且回收效率偏低
4. 对永久代进行调优是很困难的

Java虚拟机的永久代(JDK1.7之前)，主要用来存储class类，常量，方法描述等。对永久代的回收主要包括废弃常量和无用的类











