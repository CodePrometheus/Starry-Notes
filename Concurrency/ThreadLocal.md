# ThreadLocal

[TOC]

ThreadLocal可以理解为TreadLocalMap的封装，ThreadLocal中存了ThreadLocalMap

> 和HashMap的不同点：
>
> hash计算方式不同，另外ThreadLocalMap解决冲突的方法是**开放地址法**。


其中key是ThreadLocal对象引用，value是当前线程存储值

ThreadLocal在父子线程之间是**不可以被继承**的



ThreadLocalMap是一个Entry数组。**Entry内是一个弱引用key和强引用value**

~~~java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}

// 静态内部类
static class ThreadLocalMap {
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
        table = new Entry[INITIAL_CAPACITY];
        int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
        table[i] = new Entry(firstKey, firstValue);
        size = 1;
        setThreshold(INITIAL_CAPACITY);
    }
}
~~~

ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。



## ThreadLocal的key为什么用弱引用

如果 Key使用强引用：也就是上述说的情况，引用ThreadLocal的对象被回收了，ThreadLocal的引用ThreadLocalMap的Key为强引用并没有被回收，**如果不手动回收的话，ThreadLocal将不会回收那么将导致内存泄漏**。

Key使用弱引用：引用的ThreadLocal的对象被回收了，ThreadLocal的引用ThreadLocalMap的Key为弱引用，如果内存回收，那么将ThreadLocalMap的Key将会被回收，ThreadLocal也将被回收。value在ThreadLocalMap调用get、set、remove的时候就会被清除。



## 在threadLocalMap调用get方法时，发生了gc，弱引用对象threadLocal是否会因为被回收掉，而get不到

不会，因为堆中有一个强引用在，只有在不在用threadLocal了，把这个强引用设为null后，ThreadLocalMap中的弱引用才会被回收





## 内存泄漏问题

> 不再会被使用的对象或者变量占用的内存不能被回收，就是内存泄露。

首先来说，如果把ThreadLocal置为null，那么意味着Heap中的ThreadLocal实例不在有强引用指向，只有弱引用存在，因此GC是可以回收这部分空间的，也就是key是可以回收的。但是**value却存在一条从Current Thread过来的强引用链**。因此只有当**Current Thread销毁时，value才能得到释放**

因此，只要这个线程对象被gc回收，就不会出现内存泄露，**但在threadLocal设为null和线程结束这段时间内不会被回收的**，比如key为null时，就没有办法访问这些key为null的Entry的value，如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value永远无法回收，造成内存泄漏。

![img](images/68747470733a2f2f67697465652e636f6d2f73757065722d6a696d77616e672f696d672f7261772f6d61737465722f696d672f32303231303331343136303734312e706e67)



> 如何避免内存泄漏

每次使用完ThreadLocal，都调用它的remove()方法，清除数据。





## remove

~~~java
private void remove(ThreadLocal<?> key) {
    // 使用hash方式，计算当前ThreadLocal变量所在table数组位置
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    // 再次循环判断是否在为ThreadLocal变量所在table数组位置
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            // 调用WeakReference的clear方法清除对ThreadLocal的弱引用
            // 将entry的对threadlocal的引用赋值为null
            e.clear();
            // 清理key为null的元素
            // 将 entry的value赋值为null
            expungeStaleEntry(i);
            return;
        }
    }
}
~~~



~~~java
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // 根据强引用的取消强引用关联规则，将value显式地设置成null，去除引用
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    // 重新hash，并对table中key为null进行处理
    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        // 对table中key为null进行处理,将value设置为null，清除value的引用
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
            int h = k.threadLocalHashCode & (len - 1);
            if (h != i) {
                tab[i] = null;
                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
~~~







## 线程的状态

六大状态

~~~java
public enum State {
        /**
         * Thread state for a thread which has not yet started.
         */
        NEW,
 
        /**
         * Thread state for a runnable thread.  A thread in the runnable
         * state is executing in the Java virtual machine but it may
         * be waiting for other resources from the operating system
         * such as processor.
         */
        RUNNABLE,
 
        /**
         * Thread state for a thread blocked waiting for a monitor lock.
         * A thread in the blocked state is waiting for a monitor lock
         * to enter a synchronized block/method or
         * reenter a synchronized block/method after calling
         * {@link Object#wait() Object.wait}.
         */
        BLOCKED,
 
        /**
         * Thread state for a waiting thread.
         * A thread is in the waiting state due to calling one of the
         * following methods:
         * <ul>
         *   <li>{@link Object#wait() Object.wait} with no timeout</li>
         *   <li>{@link #join() Thread.join} with no timeout</li>
         *   <li>{@link LockSupport#park() LockSupport.park}</li>
         * </ul>
         *
         * <p>A thread in the waiting state is waiting for another thread to
         * perform a particular action.
         *
         * For example, a thread that has called <tt>Object.wait()</tt>
         * on an object is waiting for another thread to call
         * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on
         * that object. A thread that has called <tt>Thread.join()</tt>
         * is waiting for a specified thread to terminate.
         */
        WAITING,
 
        /**
         * Thread state for a waiting thread with a specified waiting time.
         * A thread is in the timed waiting state due to calling one of
         * the following methods with a specified positive waiting time:
         * <ul>
         *   <li>{@link #sleep Thread.sleep}</li>
         *   <li>{@link Object#wait(long) Object.wait} with timeout</li>
         *   <li>{@link #join(long) Thread.join} with timeout</li>
         *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>
         *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li>
         * </ul>
         */
        TIMED_WAITING,
 
        /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         */
        TERMINATED;
    }
~~~

1. **NEW**：尚未启动的线程的线程状态。
2. **RUNNABLE**：可运行线程的线程状态。线程正在运行或者正在获取CPU时间片。
3. **BLOCKED**：线程的线程状态被阻止，正在等待监视器锁。
4. **WAITING**：等待线程的线程状态。调用某个线程的wait()、join()、 locksupport park()处于等待状态的线程。
5. **TIMED_WAITING**：**具有指定等待时间**的等待线程的线程状态。调用某个线程的具有指定正等待时间的方法sleep(long)、wait(long)、join(long)、LockSupport Parknanos()、LockSupport ParkUntil()处于指定等待时间的等待线程的线程状态。
6. **TERMINATED**：终止线程的线程状态。

