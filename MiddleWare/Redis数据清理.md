# Redis数据清理

[TOC]



## 过期键清理策略

redis采用定期删除+惰性删除

- 过期键的删除策略
  - 定时删除，为每个过期键建立一个timer，缺点占用CPU
  - 惰性删除，**只会在取出key的时候**判断是否过期，对内存不友好。
  - 定期删除，即**根据设定执行时长和操作频率清理**，缺点难以确定。
    - Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对CPU时间的影响，默认100ms就随机抽一些设置了过期时间的key，不会扫描全部的过期键，因为开销过大。
  - Redis使用惰性删除和定期删除结合的方式配合使用。
- redis在内存空间不足的时候，为了保证命中率，就会选择一定的数据淘汰策略——内存淘汰机制（过期键的补充措施）





## 内存淘汰策略

- 内存淘汰机制：八种大体上可以分为4种，**lru**（最近最少使用）、**lfu**（最少使用频率）、**random**（随机）、**ttl**（根据生存时间，快过期）。

1. volatile-lru：从已设置过期时间的数据集中挑选最近最少使用的数据淘汰。
2. volatile-ttl：从已设置过期时间的数据集中挑选将要过期的数据淘汰。
3. volatile-random：从已设置过期时间的数据集中任意选择数据淘汰。
4. volatile-lfu：从已设置过期时间的数据集挑选使用频率最低的数据淘汰。
5. allkeys-lru：从数据集中挑选最近最少使用的数据淘汰。（缓存常用）
6. allkeys-lfu：从数据集中挑选使用频率最低的数据淘汰。
7. allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
8. no-enviction（驱逐）：禁止驱逐数据，这也是默认策略。意思是当内存不足以容纳新入数据时，新写入操作就会报错，请求可以继续进行，线上任务也不能持续进行，采用no-enviction策略可以保证数据不被丢失。（字典库常用）



## LRU实现原理

常规的LRU算法会维护一个**双向链表**，用来表示访问关系，且需要额外的存储存放 next 和 prev 指针，牺牲比较大的存储空间。

Redis的实现LRU会维护一个全局的LRU时钟，并且每个键中也有一个时钟，每次访问键的时候更新时钟值。

默认的LRU时钟的分辨率是1秒，可以通过改变
REDIS_LRU_CLOCK_RESOLUTION宏的值来改变

~~~c
#define REDIS_LRU_BITS 24
unsigned lruclock:REDIS_LRU_BITS; /* Clock for LRU eviction */
~~~



淘汰过程：Redis会基于server.maxmemory_samples配置选取固定数目的key，然后比较它们的lru访问时间，然后淘汰最近最久没有访问的key，maxmemory_samples的值越大，Redis的近似LRU算法就越接近于严格LRU算法，但是相应消耗也变高，对性能有一定影响，样本值默认为5。

~~~java
public class LRU<K, V> implements Iterable<K> {

    private Node head;
    private Node tail;
    private HashMap<K, Node> map;
    private int maxSize;

    private class Node {

        Node pre;
        Node next;
        K k;
        V v;

        public Node(K k, V v) {
            this.k = k;
            this.v = v;
        }
    }


    public LRU(int maxSize) {

        this.maxSize = maxSize;
        this.map = new HashMap<>(maxSize * 4 / 3);

        head = new Node(null, null);
        tail = new Node(null, null);

        head.next = tail;
        tail.pre = head;
    }


    public V get(K key) {

        if (!map.containsKey(key)) {
            return null;
        }

        Node node = map.get(key);
        unlink(node);
        appendHead(node);

        return node.v;
    }


    public void put(K key, V value) {

        if (map.containsKey(key)) {
            Node node = map.get(key);
            unlink(node);
        }

        Node node = new Node(key, value);
        map.put(key, node);
        appendHead(node);

        if (map.size() > maxSize) {
            Node toRemove = removeTail();
            map.remove(toRemove.k);
        }
    }


    private void unlink(Node node) {

        Node pre = node.pre;
        Node next = node.next;

        pre.next = next;
        next.pre = pre;

        node.pre = null;
        node.next = null;
    }


    private void appendHead(Node node) {
        Node next = head.next;
        node.next = next;
        next.pre = node;
        node.pre = head;
        head.next = node;
    }


    private Node removeTail() {

        Node node = tail.pre;

        Node pre = node.pre;
        tail.pre = pre;
        pre.next = tail;

        node.pre = null;
        node.next = null;

        return node;
    }


    @Override
    public Iterator<K> iterator() {

        return new Iterator<K>() {
            private Node cur = head.next;

            @Override
            public boolean hasNext() {
                return cur != tail;
            }

            @Override
            public K next() {
                Node node = cur;
                cur = cur.next;
                return node.k;
            }
        };
    }
}
~~~

