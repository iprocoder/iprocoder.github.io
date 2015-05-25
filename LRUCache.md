LRU Cache 设计与实现
=================

*LRU* 是 Least Recently Used 的缩写，即最近最少使用算法。是操作系统中常用的一种页面替换算法，在系统发生缺页中断时，置换未使用时间最长的页面。也常常用在 Cache 中，缓存最近使用过的数据。

## 数据结构

高效的实现 LRU 算法一般是使用哈希表 + 双链表，其中双链表用于缓存数据节点，并且其中的结点是按照最近被使用的时间排序的，每当一个数据节点被使用时或者一个新的数据节点加入时，需要将其放至表头位置。并且，当 Cache 已经满了时，此时如果一个新的数据结点加入到链表中时，需要替换掉最近最久使用的表尾位置的数据节点。

哈希表的作用是为了可以达到迅速找到某个结点，在没有哈希表的情况下，需要从链表中逐个遍历比较，时间复杂度为 O(n)，在使用哈希表的情况下，可以使得我们可以在 O(1) 的时间找到需要访问的结点，或者返回未找到。

##接口

Cache 的主要接口有

```java
LRUCache(int capacity)
int get(int key)
void put(int key, int value)
```

LRU Cache 的主要接口有 构造函数 `LRUCache` 用于初始化一个容量为capacity 的缓存。`get` 用于从缓存中读出数据，并且需要将其放置开头位置，当缓存中没有所需的数据时，返回特殊值 -1。`put` 用于在缓存中加入一个新的数据。当加入一个新数据时，需要先检查缓存中是否已经存在与插入数据键值相同的数据以及容量是否已经满了。当缓存已存在键值相同的数据时，可以将其取出，修改节点的值，将其放置链表的开头并将其重新加入哈希表中。若缓存中不存在键值相同的数据时，先检查容量是否已满，在缓存容量已满的情况下，需要先去掉链表尾结点。并重新根据键值和数据生成一个新的结点放到链表的开头位置并加入哈希表中。

## 代码实现

完整的 Java 实现代码如下：

```java
import java.util.*;

/**
 * 用带头节点和尾节点的双链表 + 哈希表 实现 LRU 缓存
 * Created by iprocoder on 15-5-24.
 */
public class LRUCache {

    /**
     * 缓存节点
     */
    private class CacheNode {

        int key;
        int value;
        CacheNode prev;
        CacheNode next;

        public CacheNode(int key, int value) {
            this.key = key;
            this.value = value;
        }

        public CacheNode() {
        }
    }

    private int capacity;
    private int num;
    private Map<Integer, CacheNode> map;
    private CacheNode head;
    private CacheNode tail;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.num = 0;
        this.map = new HashMap<>(capacity);
        this.head = new CacheNode();
        this.tail = new CacheNode();
        this.head.prev = null;
        this.head.next = tail;
        this.tail.prev = head;
        this.tail.next = null;
    }

    public int get(int key) {
        CacheNode cacheNode = map.get(key);

        if (cacheNode == null)
            return -1;
        detach(cacheNode);
        attach(cacheNode);
        return cacheNode.value;
    }

    public void set(int key, int value) {
        CacheNode cacheNode = map.get(key);
        if (cacheNode != null) {
            detach(cacheNode);
            cacheNode.value = value;
            attach(cacheNode);
        } else {
            if (num == capacity) {
                cacheNode = tail.prev;
                detach(cacheNode);
                map.remove(cacheNode.key);
            } else {
                num++;
                cacheNode = new CacheNode();
            }
            cacheNode.key = key;
            cacheNode.value = value;
            map.put(key, cacheNode);
            attach(cacheNode);
        }
    }

    /**
     * 分离节点
     * @param cacheNode
     */
    private void detach(CacheNode cacheNode) {
        cacheNode.prev.next = cacheNode.next;
        cacheNode.next.prev = cacheNode.prev;
    }

    /**
     * 将节点插入头部
     * @param cacheNode
     */
    private void attach(CacheNode cacheNode) {
        cacheNode.prev = head;
        cacheNode.next = head.next;
        head.next = cacheNode;
        cacheNode.next.prev = cacheNode;
    }

    public static void main(String[] args) {
        LRUCache lruCache = new LRUCache(1);
        lruCache.set(1, 3);
        System.out.println(lruCache.get(1));
        lruCache.set(3, 2);
        System.out.println(lruCache.get(2));
        System.out.println(lruCache.get(1));
    }
}
```

## 参考链接

http://www.hawstein.com/posts/lru-cache-impl.html
> Written with [StackEdit](https://stackedit.io/).