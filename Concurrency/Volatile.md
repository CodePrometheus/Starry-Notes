# Volatile

[TOC]



## 简介

Volatile是一种同步机制，比synchronized或者Lock相关类更轻量，因为使用Volatile并**不会发生上下文切换等开销很大的行为**

   如果一个变量被修饰Volatile，JVM就知道这个变量可能被并发修改

   注意❗ Volatile做不到synchronized那样的原子保护，仅在很有限的场景下才能发挥作用



## 适用场景

![](images/image-20210605103418783.png)



~~~java
volatile int a;
// 用原子整形统计正确的数
AtomicInteger realA = new AtomicInteger();

public static void main(String[] args) throws InterruptedException {
    NoVolatileOne noVolatileOne = new NoVolatileOne();
    Thread thread1 = new Thread(noVolatileOne);
    Thread thread2 = new Thread(noVolatileOne);
    thread1.start();
    thread2.start();
    thread1.join();
    thread2.join();
    System.out.println(noVolatileOne.a);// <20000
    System.out.println(noVolatileOne.realA.get());
}

@Override
public void run() {
    for (int i = 0; i < 10000; i++) {
        // a++是组合操作，b
        a++;
        realA.incrementAndGet();
    }
}
~~~





## 作用

**保证数据的可见性**： 被volatile修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免出现数据脏读的现象，读一个volatile变量之前，**需要先使相应的本地缓存失效，这样就必须到主内存读取最新值**，写一个Volatile属性会**立即刷入到主内存**，任何一个线程对其的修改立马对其他线程可见，Volatile属性不会被线程缓存，始终从主存中读取



**禁止指令重排序**： 在多线程操作情况下，指令重排会导致计算结果不一致，**解决单例双重锁乱序问题**



**Volatile提供了happens-before保证，此外，Volatile能使得long和double的赋值是原子的**



## 和synchronized关系

-> volatile属性的**读写操作都是无锁的**，不能代替synchronized，因为它**没有提供原子性和互斥性**

如果一个共享变量自始自终**只被各个线程赋值**，而没有其他操作，那么就可以用volatile来代替synchronized或者代替原子变量，因为**赋值自身是有原子性的，而volatile又保证了可见性**，所以就足以保证线程安全

   也正因为无锁，不需要花费时间在获取锁和释放锁上，所以说它是低成本的
  并且，**Volatile只能修饰属性**，不像synchronized还可以修饰方法



## 底层实现

观察加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，**会多出一个lock前缀指令**

　　lock前缀指令实际上相当于一个**内存屏障**（也成内存栅栏），内存屏障会提供3个功能：

　　1）它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面

　　2）它会**强制将对缓存的修改操作立即写入主存**；

　　3）如果是**写操作，它会导致其他CPU中对应的缓存行无效**





