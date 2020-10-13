---
title: ConcurrentHashMap源码分析
date: 2019-04-10 09:06:35
tags: [Java, 源码]
categories: Java
---

ConcurrentHashMap是HashMap的线程安全版，基于JDK1.8分析其源码。

<!--more-->

# 概述

![](image.jpg)





经典的开源框架Spring的底层数据结构就是使用ConcurrentHashMap实现的。

hash冲突的处理方式也与HashMap类似，冲突的记录被存储到同一位置。

JDK8中实现线程安全是使用`CAS`算法，而不是以前的`Segment`概念。

key和value都不允许为空。

# 初始化

```java
/**
     * Creates a new, empty map with the default initial table size (16).
     */
public ConcurrentHashMap() {
}

public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    this.sizeCtl = cap;
}

```

通过提供初始容量，计算了 sizeCtl，sizeCtl = 【 (1.5 * initialCapacity + 1)，然后向上取最近的 2 的 n 次方】



```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            Thread.yield(); // lost initialization race; just spin
        // cas 将sizeCtl设置为-1，代表抢到了锁
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    // 默认初始容量为16
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    // 数组赋值给table table是volatile的
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

## sizeCtl

```java
/**
     * Table initialization and resizing control.  When negative, the
     * table is being initialized or resized: -1 for initialization,
     * else -(1 + the number of active resizing threads).  Otherwise,
     * when table is null, holds the initial table size to use upon
     * creation, or 0 for default. After initialization, holds the
     * next element count value upon which to resize the table.
     */
private transient volatile int sizeCtl;
```



控制标识符，不同的值有不同的属性。

* 负数代表正在初始化或正在扩容
* -1代表正在初始化
* -N标识有N-1个线程正在进行扩容操作
* 正数或0表示hash表还没被初始化，数值表示初始化或下一次扩容的大小。类似于HashMap中loadfactor的概念。它的值始终是当前ConcurrentHashMap容量的0.75倍，这与loadfactor是对应的。

它是多线程共享的。

## Node

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;

    Node(int hash, K key, V val, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.val = val;
        this.next = next;
    }

    public final K getKey()       { return key; }
    public final V getValue()     { return val; }
    public final int hashCode()   { return key.hashCode() ^ val.hashCode(); }
    public final String toString(){ return key + "=" + val; }
    public final V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    public final boolean equals(Object o) {
        Object k, v, u; Map.Entry<?,?> e;
        return ((o instanceof Map.Entry) &&
                (k = (e = (Map.Entry<?,?>)o).getKey()) != null &&
                (v = e.getValue()) != null &&
                (k == key || k.equals(key)) &&
                (v == (u = val) || v.equals(u)));
    }

    /**
         * Virtualized support for map.get(); overridden in subclasses.
         */
    Node<K,V> find(int h, Object k) {
        Node<K,V> e = this;
        if (k != null) {
            do {
                K ek;
                if (e.hash == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
            } while ((e = e.next) != null);
        }
        return null;
    }
}
```

这里的Node和HashMap中实现类似，但有不同，

* **它对val和next设置了volatile同步锁**
* 不允许调用`setValue`方法直接改变属性值
* 增加了find方法辅助map.get()方法

# Unsafe与CAS

大量使用`U.compareAndSwap`方法，**利用CAS算法实现无锁修改值**。基本思想是不断的比较当前内存中的变量值与你指定的变量值是否相等，相等则接受，否则拒绝，因此当前线程中的值并不是最新值，修改可能覆盖其他线程的修改结果。**类似乐观锁，SVN的思想**。

## 三个核心方法

```java
// 获得i位置上的节点
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}
// 使用cas算法设置i节点的值，实现并发是因为指定i节点原来的值
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
// 使用volatile方法设置节点的值
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
    U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
}
```

# put方法

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    // 计算hash
    int hash = spread(key.hashCode());
    // 相应链表长度
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        // table为空，初始化
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        // 根据hash计算节点的index
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 用一次CAS操作技能新值放入其中，put操作结束
            // 如果cas失败，说明有并发操作，进入下一个循环
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        // 遇到表连接点，帮助进行整合表
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else { // f是该位置的头节点且不为空
            V oldVal = null;
            // 对table的index位置加锁，只锁住当前index位置
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    // 说明是链表节点
                    if (fh >= 0) {
                        // 记录链表长度
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            // hash与key相同，执行覆盖操作
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            // 遍历到了最后一个节点，新建节点插到最后
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    // 红黑树节点
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                              value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                // 链表长度大于8，转化为红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    // 不一定会进行红黑树转换，如果当前数组长度小于64，
                    // 选择进行数组扩容，而不是转换红黑树
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

**当key或value为null 的时候，抛出异常，因此ConcurrentHashMap的key与value都不能为空。**

流程：计算key的hashCode，然后计算table的index（(n-1) & hash），如果index位置为null，使用casTabAt方法插入，否则使用synchronized关键字对index位置加锁，仅仅锁住index位置因此其他线程可以安全地获取其他位置，提高了并发。然后判断index位置上第一个节点的hashCode值，小于0为红黑树根节点。如果是链表节点，遍历，当hash与key相同时，修改节点值，否则添加新节点。是红黑树节点的情况暂不分析。

# get

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    // 计算hash
    int h = spread(key.hashCode());
    // hash对应index位置不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        if ((eh = e.hash) == h) {
            // 链表头节点的key与传入的key相同且不为null，返回该节点的值
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        // 节点在树上
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        // 遍历链表查找节点
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

1. 计算hash 
2. 根据hash得到index ：hash & (len - 1)
3. 根据该位置的节点性质查找
   1. null，返回null
   2. 该位置节点恰好就是，返回该节点的值
   3. 该位置节点hash < 0 ，正在扩容，或者是红黑树
   4. 链表，遍历查找

# 1.7中的ConcurrentHashMap

1.7中主要使用Segment实现线程安全。

整个ConcurrentHashMap有一个个Segment组成，即ConcurrentHashMap是一个Segment数组，Segment通过集成ReentrantLock进行加锁，每次锁住一个segment。

Segment数组不能扩容，扩容是Segment数组某个位置内部数组HashEntry进行扩容。

![](https://www.javadoop.com/blogimages/map/3.png)



# 参考

* [CSDN-ConcurrentHashMap源码分析（JDK8版本）](<https://blog.csdn.net/u010723709/article/details/48007881>)
* [掘金-Java 8 ConcurrentHashMap源码分析](<https://juejin.im/entry/59fc786d518825297f3fa968>)
* [Javadoop - Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析](<https://www.javadoop.com/post/hashmap>)





















