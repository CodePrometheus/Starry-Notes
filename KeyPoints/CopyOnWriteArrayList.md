# CopyOnWriteArrayList

[TOC]



## 简介

> 解决高并发下ArrayList出现的问题
>
> 像我们的String一样，不在原来的对象上直接进行操作，而是复制一份对其进行修改，另外此处的修改操作是利用Lock锁进行上锁的，所以保证了线程安全问题

### 底层

和ArrayList一样，其底层数据结构也是数组，加上transient不让其被序列化，加上volatile修饰来保证多线程下的其可见性和有序性

~~~java
/** The lock protecting all mutators */
final transient ReentrantLock lock = new ReentrantLock();

/** The array, accessed only via getArray/setArray. */
private transient volatile Object[] array;
~~~







### 读写分离

写操作在一个**复制的数组上进行**，读操作还是在**原始数组中进行**，读写分离，互不影响。

**写操作需要加锁**，防止并发写入时导致写入数据丢失。

写操作结束之后需要把**原始数组指向新的复制数组**。

~~~java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock(); // 加锁，避免多线程写时copy多个副本
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        // t
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}

final void setArray(Object[] a) {
    array = a;
}

// 读取操作没有任何同步控制和锁操作，因为内部数组不会发生修改，只会被另外一个array替换，因此可以保证数据安全
@SuppressWarnings("unchecked")
private E get(Object[] a, int index) {
    return (E) a[index];
}
~~~



### 适用场景

CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合**读多写少**的应用场景。

但是 CopyOnWriteArrayList 有其缺陷：

- **内存占用**：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
- 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。**只能保证数据的最终一致性**。
- 在原数组的内容比较多时，可能会导致**Young GC或者Full GC**

所以 CopyOnWriteArrayList **不适合**内存敏感以及对实时性要求很高的场景



## 分析

**Fail-Safe**

基于容器的一个克隆，因此，对容器内容的修改不影响遍历。java.util.concurrent包下的容器都是安全失败的,可以在多线程下并发使用，并发修改。常见的的使用Fail-Safe方式遍历的容器有ConcerrentHashMap和CopyOnWriteArrayList等。

原理：采用安全失败机制的集合容器，在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发Concurrent Modification Exception。

缺点：基于拷贝内容的优点是避免了Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。



## 区别

虽然是采用了读写分离的思想，但是却有很大不同，不同之处在于copy。

### 读写锁

读线程具有实时性，写线程会阻塞。解决了数据不一致的问题。**但是读写锁依然会出现读线程阻塞等待的情况**

### CopyOnWriteArrayList

读线程具有实时性，写线程会阻塞。不能解决数据不一致的问题。**但是CopyOnWriteArrayList 不会出现读线程阻塞等待的情况**











