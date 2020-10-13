---
title: HashMap源码解析
date: 2019-04-04 09:16:18
tags: [Java,源码]
categories: Java
---

基于JDK1.8分析HashMap的底层实现。

<!--more-->

## 概述

HashMap线程不安全，允许key为null，value为null，遍历无序。

其底层实现有三种数据结构：数组，链表，红黑树。在JDK8中，一个桶存储的链表结构大于8时会转化为红黑树，在红黑树中查找时间复杂度为O(logn)。

数组用来存储键值对，每一个键值对别成为**Entry**，这是HashMap的主干。其中的米一个元素初始值都为null。

![](2.png)



## 链表节点Node

Java8中使用Node替代7中的Entry，适用于链表。

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        // 每一个节点的hashCode由key的hashcode和value的hashCode得到
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    // 重写equals
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

链表中的每一个节点的hashCode由key的hashcode和value的hashCode得到。

## 构造函数

```java
// 默认capacity，table的容量大小，默认16
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

static final int MAXIMUM_CAPACITY = 1 << 30;
// 默认装载因子，table能够使用的比例
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// size 的临界值，当 size 大于等于 threshold 就必须进行扩容操作。
int threshold;

// 哈希桶
transient Node<K,V>[] table;

// 默认构造函数，指定loadFactor为默认0.75f
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; 
}

// 指定初始容量的构造函数
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 指定初始容量和装载因子的构造函数
public HashMap(int initialCapacity, float loadFactor) {
    // 边界处理
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    // 初始容量不能大于2的30次方
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // 装载因子的边界处理
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    
    this.threshold = tableSizeFor(initialCapacity);
}

// 新建hash表
public HashMap(Map<? extends K, ? extends V> m) {
    // 指定加载因子为默认0.75f
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    // 
    putMapEntries(m, false);
}
```

### 重要参数：

* capacity：table 容量大小，默认为16，且必须为2的n次方
* size：键值对数量
* threshold（阈值）：size临界值，size大于等于threshold时需要进行扩容
* loadfactor：装载因子，threshold = capacity * loadfactor

增大负载因子可以减少 Hash 表（就是那个 Entry 数组）所占用的内存空间，但会增加查询数据的时间开销，而查询是最频繁的的操作（HashMap 的 get() 与 put() 方法都要用到查询）；减小负载因子会提高数据查询的性能，但会增加 Hash 表所占用的内存空间。

由于在计算中位运算比取模运算效率高的多，所以 HashMap 规定数组的长度为 `2^n` 。这样用 `2^n - 1` 做位运算与取模效果一致，并且效率还要高出许多。

### tableSizeFor，计算数组容量

```java
/**
     * Returns a power of two size for the given target capacity.
     
     根据期望容量，返回2的n次方实，数组实际容量 length
     */
static final int tableSizeFor(int cap) {
    //经过下面的 或 和位移 运算， n最终各位都是1。
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    // 判断n是否越界，小于0赋值1，大于最大容量设为最大容量
    // 返回 2的n次方作为 table（哈希桶）的阈值
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

### putMapEntries()

将另一个Map的所有元素加入表中，参数evict初始化时为false，其他情况为true

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        if (table == null) { // pre-size
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                     (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        else if (s > threshold)
            resize();
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

## put操作

### put()

往哈希表里插入一个节点的`putVal`函数,如果参数`onlyIfAbsent`是true，那么不会覆盖相同key的值value。如果`evict`是false。那么表示是在初始化时调用的

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
// onlyIfAbsent为true时，只有不存在key是进行put操作
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    // tab当前hash桶，p临时链表节点
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 第一次放入值，table为空，执行初始化resize()
    if ((tab = table) == null || (n = tab.length) == 0)
        // 扩容hash表，并将长度赋给n
        n = (tab = resize()).length;
    // 当前节index节点为空，没有hash冲突，直接构建新节点，挂载在index处
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        // 发生了hash冲突
        Node<K,V> e; K k;
        // 如果该index处的第一个数据与插入数据hash相同，key也相等，当前节点赋值给e
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // p节点是红黑树的节点，调用红黑树的插入
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 不是覆盖操作
            // 遍历链表
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    // 遍历到尾部，添加新节点（Java 7是插入到链表前面）
                    p.next = newNode(hash, key, value, null);
                    // 添加新节点后链表长度大于8，转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 找到了要覆盖的节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // e不为空，存在旧值的key与要插入的key相等 执行覆盖，返回旧值
        if (e != null) { // existing mapping for key
            // 覆盖节点，返回oldva
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    // 以上完成插入新节点的操作
    ++modCount;
    // 判断size是否大于临界值，执行resize操作
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

* 桶下标 index=(n - 1) & hash

HashMap不是在new时申请空间，而是在第一次put时申请空间。没有发生hash冲突，直接构建新节点挂在到index处。

概述：执行put操作时，如果是第一次put，执行resize() 方法；根据key计算hash，hash & (n - 1) 得到在数组中的index位置，如果index位置为空，创建Node节点，挂载在index位置处；index不为空，发生了hash冲突，此时判断index处的第一个节点的key与hash与放入的key与hash是否相等，相等执行覆盖。如果第一个节点是红黑树节点，调用红黑树插入方法，否则是链表节点。遍历链表节点，将要存的节点插入到链表最后，如果存在相等的key，执行覆盖。插入完成后，判断size是否大于临界值，大于则执行resize();

在JDK8中，插入链表是插在链表最后，即尾插入，而JDK7中是头插入。头插入的一个缺点是在并发情况下插入扩容可能出现链表环发生死循环。

### hash()



```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

HashMap允许key为null，key为null，hash为0，下标为0

根据key取hash值，扰动函数，使hash均衡，减少hash碰撞的几率。它会综合hash值高位和低位的特征，并存放在低位，因此在与运算时，相当于高低位一起参与了运算，以减少hash碰撞的概率。

JDK8中简化操作。

### resize() 重点

**初始化或加倍哈希桶大小。如果是当前哈希桶是null,分配符合当前阈值的初始容量目标。否则，扩容成以前的两倍。**

```java
final Node<K,V>[] resize() {
	// 当前表的哈希桶
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    // 当前桶容量大于0
    if (oldCap > 0) {
        // 大于最大容量
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 将临界值设为最大
            threshold = Integer.MAX_VALUE;
            // 返回当前哈希桶，不再扩容
            return oldTab;
        }
        // 如过当前容量的两倍小于最大容量并且当前容量大于默认容量
        // 新容量为当前容量的两倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 新的临界值也为当前的两倍
            newThr = oldThr << 1; 
    }
    // //如果当前表是空的，但是有阈值。代表是初始化时指定了容量、阈值的情况
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               
        // zero initial threshold signifies using defaults
        // 容量为0，代表第一次执行put操作，需要初始化
        // 容量和临界值都初始化为默认值
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    //如果新的阈值是0，对应的是  当前表是空的，但是有阈值的情况
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        //进行越界修复
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 当前哈希桶不为空
    if (oldTab != null) {
        // 遍历
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            // 当前节点有元素，值赋给e
            if ((e = oldTab[j]) != null) {
                // 置空，便于GC
                oldTab[j] = null;
                //当前链表就一个元素，没有发生hash冲突
                if (e.next == null)
                    // 直接将元素放在新的桶里
                    // 这里去下标 当前节点的hash 与 桶的长度减一
                    // 桶的长度是2的n次方，这样就是取模
                    newTab[e.hash & (newCap - 1)] = e;
                // 发生过hash碰撞且节点是红黑树节点
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // 没有放生哈希碰撞，节点小于8，依次放入新的哈希桶对应位置
                else { // preserve order
                    //因为扩容是容量翻倍，所以原链表上的每个节点，现在可能存放在原来的下标，即low位， 
                    // 或者扩容后的下标，即high位。 high位=  low位+原哈希桶容量
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        //这里又是一个利用位运算 代替常规运算的高效点： 
                        // 利用哈希值 与 旧的容量，可以得到哈希值去模后，是大于等于oldCap还是小于oldCap，
                        // 等于0代表小于oldCap，应该存放在低位，否则存放在高位
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 高位相同的逻辑
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 低位链表存在源index处
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 高位存在新index
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

* HashMap中用到了许多位运算，更高效
* 扩容时，将旧的数组中引用置null，便于GC
* 取下标 是用 **哈希值 与运算 （桶的长度-1）** `i = (n - 1) & hash`。 由于桶的长度是2的n次方，这么做其实是等于 一个**模运算**。但是**效率更高**
* 因为扩容是容量翻倍，所以原链表上的每个节点，现在可能存放在原来的下标，即low位， 或者扩容后的下标，即high位。 high位= low位+原哈希桶容量

resize是很费时间的操作，因此最好在HashMap初始化时指定大小，尽量减少调用resize方法。

![](hashmap1.jpg)

## get操作

### get()

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

// 传入扰动后的hash 和 key查找
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 哈希表不为空，根据hash计算下标，该下标有节点的话
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 如果链表头结点就是要查找的节点，直接返回
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 链表头结点的下一个节点不为空
        if ((e = first.next) != null) {
            // 是红黑树树节点，使用红黑树的查找
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 是链表，遍历链表
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    // 没有找到，返回null
    return null;
}
```

1. 计算key 的hash，根据hash计算index，hash & (len - 1)
2. 判断数组该位置第一个原始是否恰好为要找的
3. 判断第一个节点元素类型是否为TreeNode，是用红黑树方法取数据，不是继续
4. 便利链表，找到相等的key

以上在O(1)的时间里查找执行完

## 7与8的一些不同

* 7是数组+链表的结构；8是数组+链表+红黑树，链表长度大于8转化为红黑树
* 插入时7是头插法；8是尾插法。一种原因是防止形成链表环
* 7是先计算size后插入；8是先插入后计算

## 参考

* [CS-Notes-HashMap](<https://github.com/CyC2018/CS-Notes/blob/master/docs/notes/Java%20%E5%AE%B9%E5%99%A8.md#hashmap>)
* [掘金-面试必备：HashMap源码解析（JDK8）](<https://juejin.im/post/599652796fb9a0249975a318>)
* [Javadoop - Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析](<https://www.javadoop.com/post/hashmap>)
* [掘金 - HashMap为何从头插入改为尾插入](<https://juejin.im/post/5ba457a25188255c7b168023>)

