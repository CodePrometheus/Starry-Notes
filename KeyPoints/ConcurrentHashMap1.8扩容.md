# ConcurrentHashMap1.8扩容

[TOC]

## transfer任务

对于一个大任务拆分成多个小任务供多线程执行，一般都要求这些小任务具有相似性，流程一致，并且很重要的一点，任务之间的相互影响尽量少。那么在扩容之中，是怎么划分这个任务的呢？

一般我们说的扩容，都包含两个步骤。第一，新建一个2倍大小的数组，这个过程要求单线程完成（多线程创建几个数组没有意义，容易出错），这个步骤每个HashMap、ConcurrentHashMap基本是都是同一套，前面我说过很多了，到这里没什么好说的了。第二步，执行节点迁移，说白了就是rehash，相当于把旧数组中所有的节点重新“put”到新数组中。在1.8的HashMap中，用了一个技巧，避免了重新根据hash值定位。根据这个，我们可以知道，进行 n -> 2n 的扩容时，扩容前节点所在的hash桶的索引为index，这个节点迁移到新数组中只会有两种情况：要么在还是在新数组的索引为index处，要么迁移到新数组的索引为 index + n 的地方。所以旧的table数组上各个hash桶中的节点的迁移是不会互相影响的，这一点对多线程扩容非常有利。根据这一点，可以知道，**每个hash桶的迁移都可以作为一个线程在扩容时的一个transfer任务**。

另外，每个线程要任务都不应该规模太小，因为扩容并不是IO型操作，节点迁移的执行速度本身很快，太多的线程来执行节点迁移，线程调度开销占比变大，反而降低了吞吐量。ConcurrentHashMap这里，会根据CPU的核心数目，来算出一个transfer任务包含的hash桶的数量。

~~~java
// 下面的代码用于计算每个transfer中要迁移多少个hash桶，一个transfer任务完成后，可以再次申请
int n = tab.length, stride;
if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
    stride = MIN_TRANSFER_STRIDE; // subdivide range
~~~

下面统一假定1个线程的一次transfer任务要负责迁移16个hash桶



## transfer任务申请

在经典的生成者消费者模式中，会有一个任务队列。消费者向任务队列申请任务，申请到任务时，队列中元素数量减1，当任务队列中元素数量为0时，不能再申请任务。ConcurrentHashMap这里并没有使用额外的任务队列，因为table数组本身就可以当作是一个队列。第一点中说了，一个transfer任务中要负责迁移stride个hash桶，最简单的设计当然是16个hash桶都是连续的。为了记录还有多少个任务，使用了一个类变量transferIndex，可以把这个看成是任务队列的size，每一次申请任务，这个size减1。

另外，迭代操作的下标是从小往大，也就是正向，为了减少扩容时的transfer和迭代的冲突，transfer使用反向，也就是下标从大到小。顺序相反时，二者相遇过后，迭代没处理的都是已经transfer的hash桶，transfer没处理的，都是已经迭代的hash桶，冲突会变少。

所以，任务队列的size减1，翻译过来就是，在table数组中的索引减stride。这个索引就是 transferIndex，用于标记整体的transfer进行到了哪里。因为transfer个数，从1开始，因此transferIndex也是从1开始，下标在[transferIndex - 实际的stride（下界要 >= 0）, transferIndex - 1]内的hash桶，就是每个transfer的任务区间。transferIndex <= 0 时，代表没有任务可以申请，此时无法帮助扩容。注意，NCPU不一定是2^n，因此最后一个任务中的hash桶的数量可能不足stride个，此时只执行余下的数量。

为了保证每个任务只被领取一次，transferIndex递减是用CAS操作完成的。

特殊情况下，会出现多线程扩容重叠，此时某个transfer任务虽然被领取了，但是却不能被执行，会被作废。这是根据transfer方法的代码理解得带的，因为transfer方法的代码中有考虑任务作废的情况。但是根据下面第3点的分析，扩容重叠这种特殊情况是有一个机制来避免的，代码中有处理扩容作废，但是实际不会发生。

![img](images/20170305232906639)





## 执行逻辑

1. 通过计算 CPU 核心数和 Map 数组的长度得到每个线程（CPU）要帮助处理多少个桶，并且这里每个线程处理都是平均的。默认每个线程处理 16 个桶。因此，如果长度是 16 的时候，扩容的时候只会有一个线程扩容。

2. 初始化临时变量 nextTable。将其在原有基础上**扩容两倍**。

3. 死循环开始转移。多线程并发转移就是在这个死循环中，根据一个 finishing 变量来判断，该变量为 true 表示扩容结束，否则继续扩容。

   3.1 进入一个 while 循环，分配数组中一个桶的区间给线程，默认是 16. 从大到小进行分配。当拿到分配值后，进行 i-- 递减。这个 i 就是数组下标。（`其中有一个 bound 参数，这个参数指的是该线程此次可以处理的区间的最小下标，超过这个下标，就需要重新领取区间或者结束扩容，还有一个 advance 参数，该参数指的是是否继续递减转移下一个桶，如果为 true，表示可以继续向后推进，反之，说明还没有处理好当前桶，不能推进`)
   3.2 出 while 循环，进 if 判断，判断扩容是否结束，如果扩容结束，**清空临死变量，更新 table 变量，更新扩容阈值**。如果没完成，但已经无法领取区间（没了），该线程退出该方法，并将 sizeCtl 减一，表示扩容的线程少一个了。如果减完这个数以后，sizeCtl 回归了初始状态，表示没有线程再扩容了，该方法所有的线程扩容结束了。（`这里主要是判断扩容任务是否结束，如果结束了就让线程退出该方法，并更新相关变量`）。然后检查所有的桶，防止遗漏。
   3.3 如果没有完成任务，且 i 对应的槽位是空，尝试 CAS 插入占位符，让 putVal 方法的线程感知。
   3.4 如果 i 对应的槽位不是空，且有了占位符，那么该线程跳过这个槽位，处理下一个槽位。
   3.5 如果以上都是不是，说明这个槽位有一个实际的值。开始同步处理这个桶。
   3.6 到这里，都还没有对桶内数据进行转移，只是计算了下标和处理区间，然后一些完成状态判断。同时，如果对应下标内没有数据或已经被占位了，就跳过了。

4. 处理每个桶的行为都是同步的。防止 putVal 的时候向链表插入数据。
   4.1 如果这个桶是链表，那么就将这个链表根据 length 取于拆成两份，取于结果是 0 的放在新表的低位，取于结果是 1 放在新表的高位。
   4.2 如果这个桶是红黑数，那么也拆成 2 份，方式和链表的方式一样，然后，判断拆分过的树的节点数量，如果数量小于等于 6，改造成链表。反之，继续使用红黑树结构。
   4.3 到这里，就完成了一个桶从旧表转移到新表的过程。

精简，分成 3 步骤：

1. 计算每个线程可以处理的桶区间。默认 16.
2. 初始化临时变量 nextTable，扩容 2 倍。
3. 死循环，计算下标。完成总体判断。
4. 1 如果桶内有数据，同步转移数据。通常会像链表拆成 2 份。



## 源码

~~~java
/**
 * Moves and/or copies the nodes in each bin to new table. See
 * above for explanation.
 * 
 * transferIndex 表示转移时的下标，初始为扩容前的 length。
 * 
 * 我们假设长度是 32
 */
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    // 将 length / 8 然后除以 CPU核心数。如果得到的结果小于 16，那么就使用 16。
    // 这里的目的是让每个 CPU 处理的桶一样多，避免出现转移任务不均匀的现象，如果桶较少的话，默认一个 CPU（一个线程）处理 16 个桶
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range 细分范围 stridea：TODO
    // 新的 table 尚未初始化
    if (nextTab == null) {            // initiating
        try {
            // 扩容  2 倍
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            // 更新
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            // 扩容失败， sizeCtl 使用 int 最大值。
            sizeCtl = Integer.MAX_VALUE;
            return;// 结束
        }
        // 更新成员变量
        nextTable = nextTab;
        // 更新转移下标，就是 老的 tab 的 length
        transferIndex = n;
    }
    // 新 tab 的 length
    int nextn = nextTab.length;
    // 创建一个 fwd 节点，用于占位。当别的线程发现这个槽位中是 fwd 类型的节点，则跳过这个节点。
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    // 首次推进为 true，如果等于 true，说明需要再次推进一个下标（i--），反之，如果是 false，那么就不能推进下标，需要将当前的下标处理完毕才能继续推进
    boolean advance = true;
    // 完成状态，如果是 true，就结束此方法。
    boolean finishing = false; // to ensure sweep before committing nextTab
    // 死循环,i 表示下标，bound 表示当前线程可以处理的当前桶区间最小下标
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        // 如果当前线程可以向后推进；这个循环就是控制 i 递减。同时，每个线程都会进入这里取得自己需要转移的桶的区间
        while (advance) {
            int nextIndex, nextBound;
            // 对 i 减一，判断是否大于等于 bound （正常情况下，如果大于 bound 不成立，说明该线程上次领取的任务已经完成了。那么，需要在下面继续领取任务）
            // 如果对 i 减一大于等于 bound（还需要继续做任务），或者完成了，修改推进状态为 false，不能推进了。任务成功后修改推进状态为 true。
            // 通常，第一次进入循环，i-- 这个判断会无法通过，从而走下面的 nextIndex 赋值操作（获取最新的转移下标）。其余情况都是：如果可以推进，将 i 减一，然后修改成不可推进。如果 i 对应的桶处理成功了，改成可以推进。
            if (--i >= bound || finishing)
                advance = false;// 这里设置 false，是为了防止在没有成功处理一个桶的情况下却进行了推进
            // 这里的目的是：1. 当一个线程进入时，会选取最新的转移下标。2. 当一个线程处理完自己的区间时，如果还有剩余区间的没有别的线程处理。再次获取区间。
            else if ((nextIndex = transferIndex) <= 0) {
                // 如果小于等于0，说明没有区间了 ，i 改成 -1，推进状态变成 false，不再推进，表示，扩容结束了，当前线程可以退出了
                // 这个 -1 会在下面的 if 块里判断，从而进入完成状态判断
                i = -1;
                advance = false;// 这里设置 false，是为了防止在没有成功处理一个桶的情况下却进行了推进
            }// CAS 修改 transferIndex，即 length - 区间值，留下剩余的区间值供后面的线程使用
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;// 这个值就是当前线程可以处理的最小当前区间最小下标
                i = nextIndex - 1; // 初次对i 赋值，这个就是当前线程可以处理的当前区间的最大下标
                advance = false; // 这里设置 false，是为了防止在没有成功处理一个桶的情况下却进行了推进，这样对导致漏掉某个桶。下面的 if (tabAt(tab, i) == f) 判断会出现这样的情况。
            }
        }// 如果 i 小于0 （不在 tab 下标内，按照上面的判断，领取最后一段区间的线程扩容结束）
        //  如果 i >= tab.length(不知道为什么这么判断)
        //  如果 i + tab.length >= nextTable.length  （不知道为什么这么判断）
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            if (finishing) { // 如果完成了扩容
                nextTable = null;// 删除成员变量
                table = nextTab;// 更新 table
                sizeCtl = (n << 1) - (n >>> 1); // 更新阈值
                return;// 结束方法。
            }// 如果没完成
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {// 尝试将 sc -1. 表示这个线程结束帮助扩容了，将 sc 的低 16 位减一。
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)// 如果 sc - 2 不等于标识符左移 16 位。如果他们相等了，说明没有线程在帮助他们扩容了。也就是说，扩容结束了。
                    return;// 不相等，说明没结束，当前线程结束方法。
                finishing = advance = true;// 如果相等，扩容结束了，更新 finising 变量
                i = n; // 再次循环检查一下整张表
            }
        }
        else if ((f = tabAt(tab, i)) == null) // 获取老 tab i 下标位置的变量，如果是 null，就使用 fwd 占位。
            advance = casTabAt(tab, i, null, fwd);// 如果成功写入 fwd 占位，再次推进一个下标
        else if ((fh = f.hash) == MOVED)// 如果不是 null 且 hash 值是 MOVED。
            advance = true; // already processed // 说明别的线程已经处理过了，再次推进一个下标
        else {// 到这里，说明这个位置有实际值了，且不是占位符。对这个节点上锁。为什么上锁，防止 putVal 的时候向链表插入数据
            synchronized (f) {
                // 判断 i 下标处的桶节点是否和 f 相同
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;// low, height 高位桶，低位桶
                    // 如果 f 的 hash 值大于 0 。TreeBin 的 hash 是 -2
                    if (fh >= 0) {
                        // 对老长度进行与运算（第一个操作数的的第n位于第二个操作数的第n位如果都是1，那么结果的第n为也为1，否则为0）
                        // 由于 Map 的长度都是 2 的次方（000001000 这类的数字），那么取于 length 只有 2 种结果，一种是 0，一种是1
                        //  如果是结果是0 ，Doug Lea 将其放在低位，反之放在高位，目的是将链表重新 hash，放到对应的位置上，让新的取于算法能够击中他。
                        int runBit = fh & n;
                        Node<K,V> lastRun = f; // 尾节点，且和头节点的 hash 值取于不相等
                        // 遍历这个桶
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            // 取于桶中每个节点的 hash 值
                            int b = p.hash & n;
                            // 如果节点的 hash 值和首节点的 hash 值取于结果不同
                            if (b != runBit) {
                                runBit = b; // 更新 runBit，用于下面判断 lastRun 该赋值给 ln 还是 hn。
                                lastRun = p; // 这个 lastRun 保证后面的节点与自己的取于值相同，避免后面没有必要的循环
                            }
                        }
                        if (runBit == 0) {// 如果最后更新的 runBit 是 0 ，设置低位节点
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun; // 如果最后更新的 runBit 是 1， 设置高位节点
                            ln = null;
                        }// 再次循环，生成两个链表，lastRun 作为停止条件，这样就是避免无谓的循环（lastRun 后面都是相同的取于结果）
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            // 如果与运算结果是 0，那么就还在低位
                            if ((ph & n) == 0) // 如果是0 ，那么创建低位节点
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else // 1 则创建高位
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        // 其实这里类似 hashMap 
                        // 设置低位链表放在新链表的 i
                        setTabAt(nextTab, i, ln);
                        // 设置高位链表，在原有长度上加 n
                        setTabAt(nextTab, i + n, hn);
                        // 将旧的链表设置成占位符
                        setTabAt(tab, i, fwd);
                        // 继续向后推进
                        advance = true;
                    }// 如果是红黑树
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        // 遍历
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            // 和链表相同的判断，与运算 == 0 的放在低位
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            } // 不是 0 的放在高位
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        // 如果树的节点数小于等于 6，那么转成链表，反之，创建一个新的树
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                        (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                        (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        // 低位树
                        setTabAt(nextTab, i, ln);
                        // 高位数
                        setTabAt(nextTab, i + n, hn);
                        // 旧的设置成占位符
                        setTabAt(tab, i, fwd);
                        // 继续向后推进
                        advance = true;
                    }
                }
            }
        }
    }
}
~~~





## 图解

Cmap 支持并发扩容，实现方式是，将表拆分，让每个线程处理自己的区间。如下图

![img](images/4236553-9085b57399ff2318.png)



假设总长度是 64 ，每个线程可以分到 16 个桶，各自处理，不会互相影响。

扩容前的状态。

![img](images/4236553-13c7cd70508724c5.png)



当对 4 号桶或者 10 号桶进行转移的时候，会将链表拆成两份，规则是根据节点的 hash 值取于 length，如果结果是 0，放在低位，否则放在高位。

因此，10 号桶的数据，黑色节点会放在新表的 10 号位置，白色节点会放在新桶的 26 号位置。

下图是循环处理桶中数据的逻辑：

![img](images/4236553-9069a2e2dc85ff74.png)

处理完之后，新桶的数据是这样的：

![img](images/4236553-dcc2bb9654a884e0.png)

通过给每个线程分配桶区间，避免线程间的争用，通过为每个桶节点加锁，避免 putVal 方法导致数据不一致。同时，在扩容的时候，也会将链表拆成两份，这点和 HashMap 的 resize 方法类似。

而如果有新的线程想 put 数据时，也会帮助其扩容