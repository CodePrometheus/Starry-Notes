# ConcurrentHashMap常见问题

[TOC]



### 如何用CAS实现加锁

原理:使用CAS技术与同步锁技术相结合

1.不允许插入null键或null值
2.putAll源码中,不直接使用Java层次级别的数组更新,而是使用CAS的方式直接从系统底层内存空间中更新,所以并发安全

~~~java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            //通过cas进行添加
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                    
        }
        //....

~~~







### 如何实现线程安全，是绝对安全吗

ConcurrentHashMap是线程安全的，那是在他们的内部操作，其外部操作还是需要自己来保证其同步的，特别是静态的ConcurrentHashMap,其有更新和查询的过程，要保证其线程安全，需要syn一个不可变的参数才能保证其原子性



### JDK1.8 中为什么使用内置锁 synchronized替换 可重入锁 ReentrantLock

- 在 JDK1.6 中，对 synchronized 锁的实现引入了大量的优化，并且 synchronized 有多种锁状态，会从无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁一步步转换。
- 减少内存开销 。假设使用可重入锁来获得同步支持，那么每个节点**都需要通过继承 AQS 来获得同步支持**。但并不是每个节点都需要获得同步支持的，只有链表的头节点（红黑树的根节点）需要同步，这无疑带来了巨大内存浪费。



### 1.8做了哪些优化

1、整体结构
1.7：Segment + HashEntry + Unsafe

1.8: 移除Segment，使锁的粒度更小，Synchronized + CAS + Node + Unsafe

2、put（）
1.7：先定位Segment，再定位桶，put全程加锁，没有获取锁的线程提前找桶的位置，并最多自旋64次获取锁，超过则挂起。

1.8：由于移除了Segment，类似HashMap，可以直接定位到桶，拿到first节点后进行判断，1、为空则CAS插入；2、为-1则说明在扩容，则跟着一起扩容；3、else则加锁put（类似1.7）

3、get（）
基本类似，由于value声明为volatile，保证了修改的可见性，因此不需要加锁。

4、resize（）
1.7：跟HashMap步骤一样，只不过是搬到单线程中执行，避免了HashMap在1.7中扩容时死循环的问题，保证线程安全。

1.8：支持并发扩容，HashMap扩容在1.8中由头插改为尾插（为了避免死循环问题），ConcurrentHashmap也是，迁移也是从尾部开始，扩容前在桶的头部放置一个hash值为-1的节点，这样别的线程访问时就能判断是否该桶已经被其他线程处理过了。

5、size（）
1.7：很经典的思路：计算两次，如果不变则返回计算结果，若不一致，则锁住所有的Segment求和。

1.8：用baseCount来存储当前的节点个数





### 如何在很短的时间内将大量数据插入到ConcurrentHashMap

将大批量数据保存到map中有两个地方的消耗将会是比较大的：第一个是扩容操作，第二个是锁资源的争夺。第一个扩容的问题，主要还是要通过配置合理的容量大小和扩容因子，尽可能减少扩容事件的发生；第二个锁资源的争夺，在put方法中会使用synchonized对头节点进行加锁，而锁本身也是分等级的，主要思路就是尽可能的避免锁升级。所以，针对第二点，我们可以将数据通过通过ConcurrentHashMap的spread方法进行预处理，这样我们可以将存在hash冲突的数据放在一个组里面，每个组都使用单线程进行put操作，这样的话可以保证锁仅停留在偏向锁这个级别，不会升级，从而提升效率

快速插入可以考虑布隆过滤器