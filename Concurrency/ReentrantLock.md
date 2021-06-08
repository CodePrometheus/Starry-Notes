# ReentrantLock

[TOC]

## 简介

java.util.concurrent包下提供的一套互斥锁，相比Synchronized，ReentrantLock类提供了一些高级功能

 基于API层面的互斥锁，需要lock()和unlock()方法配合try/finally语句块来完成



## 底层实现

 ReenTrantLock的实现是一种**自旋锁**，通过循环调用CAS操作来实现加锁。

它的性能比较好也是因为**避免了使线程进入内核态的阻塞状态**。想尽办法避免线程进入内核的阻塞状态是我们去分析和理解锁设计的关键钥匙。





## 区别

**和synchronized区别：**

 1、**底层实现**：synchronized 是**JVM**层面的锁，是**Java关键字**，通过monitor对象来完成（monitorenter与monitorexit），ReentrantLock 是从jdk1.5以来（java.util.concurrent.locks.Lock）提供的**API层面**的锁。

 2、**实现原理****：synchronized 的实现涉及到**锁的升级**，具体为无锁、偏向锁、自旋锁、向OS申请重量级锁；ReentrantLock实现则是通过利用**CAS**（CompareAndSwap）自旋机制保证线程操作的原子性和volatile保证数据可见性以实现锁的功能。

 3、**是否可手动释放：synchronized 不需要用户去手动释放锁，synchronized 代码执行完后系统会自动让线程释放对锁的占用； ReentrantLock则需要用户去手动释放锁，如果没有手动释放锁，就可能导致死锁现象**。

 4、**是否可中断**synchronized是不可中断类型的锁，除非加锁的代码中出现异常或正常执行完成； ReentrantLock则可以中断，可通过trylock(long timeout,TimeUnit unit)设置超时方法或者将lockInterruptibly()放到代码块中，调用interrupt方法进行中断。

 5、**是否公平锁**synchronized为非公平锁 ReentrantLock则即可以选公平锁也可以选非公平锁，通过构造方法new ReentrantLock时传入boolean值进行选择，为空默认false非公平锁，true为公平锁,公平锁性能非常低





## 公平锁

公平锁自然是遵循**FIFO**（先进先出）原则的，先到的线程会优先获取资源，后到的会进行排队等待

 **优点：**所有的线程都能得到资源，不会饿死在队列中。适合大任务

 **缺点：**吞吐量会下降，队列里面除了第一个线程，其他的线程都会阻塞，cpu唤醒阻塞线程的开销大



**公平锁效率低原因：**

 公平锁要维护一个队列，后来的线程要加锁，即使锁空闲，也要先检查有没有其他线程在 wait，如果有自己要挂起，加到队列后面，然后唤醒队列最前面线程。这种情况下相比较非公平锁多了一次**挂起和唤醒**。

 **线程切换的开销**，其实就是非公平锁效率高于公平锁的原因，因为**非公平锁减少了线程挂起的几率**，后来的线程有一定几率逃离被挂起的开销。



## 非公平锁

多个线程去获取锁的时候，会直接去尝试获取，获取不到，再去进入等待队列，如果能获取到，就直接获取到锁。

 **优点：**可以减少CPU唤醒线程的开销，整体的吞吐效率会高点，CPU也不必取唤醒所有线程，会减少唤起线程的数量。

 **缺点：** 这样可能导致队列中间的线程一直获取不到锁或者长时间获取不到锁









