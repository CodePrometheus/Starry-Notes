# OOM

[TOC]



**什么情况下发生OOM**

**1、Java堆溢出：heap**

Java堆内存主要用来存放运行过程中所以的对象，该区域OOM异常一般会有如下错误信息;
java.lang.OutofMemoryError: Javaheap space
此类错误一般通过Eclipse Memory Analyzer分析OOM时dump的内存快照就能分析出来，到底是由于程序原因导致的内存泄露，还是由于没有估计好JVM内存的大小而导致的内存溢出。

另外，Java堆常用的JVM参数：
-Xms：初始堆大小，默认值为物理内存的1/64(<1GB)，默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制.
-Xmx：最大堆大小，默认值为物理内存的1/4(<1GB)，默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制
-Xmn：年轻代大小(1.4or lator)，此处的大小是（eden + 2 survivor space)，与jmap -heap中显示的New gen是不同的。

 

**2、栈溢出：stack**

栈用来存储线程的局部变量表、操作数栈、动态链接、方法出口等信息。如果请求栈的深度不足时抛出的错误会包含类似下面的信息：
java.lang.StackOverflowError

另外，由于每个线程占的内存大概为1M，因此线程的创建也需要内存空间。操作系统 可用内存-Xmx-MaxPermSize即是栈可用的内存，如果申请创建的线程比较多超过剩余内存的时候，也会抛出如下类似错误(如果虚拟机栈可以动态扩展，并且扩展时无法申请到足够的内存): 

java.lang.OutofMemoryError: unable to create new native thread

相关的JVM参数有：
-Xss: 每个线程的堆栈大小,JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K.
在相同物理内存下,减小这个值能生成更多的线程.但是操作系统对一个进程内的线程数还是有限制的,不能无限生成,经验值在3000~5000左右。



**3、运行时常量溢出  constant**
运行时常量保存在方法区，存放的主要是编译器生成的各种字面量和符号引用，但是运行期间也可能将新的常量放入池中，比如String类的intern方法。
如果该区域OOM，错误结果会包含类似下面的信息：

java.lang.OutofMemoryError: PermGen space

相关的JVM参数有：

-XX:PermSize：设置持久代(perm gen)初始值，默认值为物理内存的1/64

-XX:MaxPermSize：设置持久代最大值，默认为物理内存的1/4

 

**4、方法区溢出  directMemory**
方法区主要存储被虚拟机加载的类信息，如类名、访问修饰符、常量池、字段描述、方法描述等。理论上在JVM启动后该区域大小应该比较稳定，但是目前很多框架，比如Spring和Hibernate等在运行过程中都会动态生成类，因此也存在OOM的风险。
如果该区域OOM，错误结果会包含类似下面的信息：

java.lang.OutofMemoryError: PermGen space

相关的JVM参数可以参考运行时常量。