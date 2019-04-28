---
layout: post
title: LinkedHashMap源码分析及实现LRU
categories: JavaSE
description: 源码分析系列之LinkedHashMap
keywords: LinkedHashMap LRU
---

### 概述
从名字上看LinkedHashMap相比于HashMap，显然多了链表的实现。从功能上看，LinkedHashMap有序，HashMap无序。这里的顺序指的是添加顺序或者访问顺序。

### 基本使用
```
@Test
    public void test1(){
        LinkedHashMap<Integer, Integer> map = new LinkedHashMap<>(16, 0.75f, true);

        map.put(1,1);
        map.put(3,3);
        map.put(4,4);
        map.put(2,2);

        for (Map.Entry entry : map.entrySet()) {
            System.out.println(entry.getKey() + ":" + entry.getValue());
        }
        map.get(1);
        map.get(2);
        System.out.println();
        for (Map.Entry entry : map.entrySet()) {
            System.out.println(entry.getKey() + ":" + entry.getValue());
        }
    }
```
我们通过构造函数的第三个参数accessOrder设为true（默认为false），该参数为true，则根据访问顺序排序。如下，当我们调用get（）方法的时候，再次遍历发现数据被放到了最后面。
```
1:1
3:3
4:4
2:2

3:3
4:4
1:1
2:2
```
### 源码分析
首先LinkedHashMap继承了HashMap，因此其基本使用与HashMap几乎完全一致。
可以看到内部实现了双向链表  head指向最久访问，tail指向最新访问。
```
/**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;

    /**
     * HashMap.Node subclass for normal LinkedHashMap entries.
     */
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;  // Entry类维护了双线链表所需的前后节点指向
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
 ```
既然有了双向链表的基本定义，那么他是怎么实现双向链表的呢？找了这个类，发现并没有重写put方法，说明直接使用父类的put。那么直接看HashMap的putVal（）方法。HashMap的源码就不再详细讲了，感兴趣的朋友可以看看我的另一篇文章。
观察HashMap的putVal的时候我们发现创建节点会频繁的使用到newNode（）方法。
```
if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null); //添加节点的时候使用newNode
        else {
           ......省略
```
我们观察LinkedHashMap，果然，在这里重写了，将每个节点添加到双向链表中。
```
   Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
            new LinkedHashMap.Entry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }
   private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
```
前面已经看到了put操作，LinkedHashMap在put方法执行的时候维护了一个双向链表。接下来我们看一下get操作。
```
  public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null) // 调用父类的getNode
            return null;
        if (accessOrder) // accessOrder为true则调整双向链表顺序
            afterNodeAccess(e);
        return e.value;
    }
    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) { // accessOrder为true且 尾节点不为当前元素
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
```
当我们将accessOrder设为true的时候，LinkedHashMap会将当前元素调整到双向链表末尾。
### LRU缓存的实现
LRU（Least Recently Used，最近最少使用）算法根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”。

根据定义，为了实现LRU，LinkedHashMap我们需要将accessOrder设为true。以保证最新使用的可以调整到链表的末尾。那么另一个比较重要的就是当缓存满的时候，需要淘汰最久没有使用的数据，及链表的头。

因此，该操作应该是存在put方法执行的过程中，由于LinkedHashMap并没有实现put方法，那么继续看HashMap里面的putVal(); 在方法快结束的时候发现了一个有趣的类，从方法名就感觉可以从这里入手。
```
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true); //最后一个参数即下面的evict为true
    }
   afterNodeInsertion(evict);  // HashMap并没有对这个类进行实现。
```
查看LinkedHashMap的afterNodeInsertion
```
    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) { // 我们只需要在这里重写移除最久没有使用的节点的判断条件，就可以移除节点了
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
```
我们假设LRU里最大容量为16，只需要将构造方法里的accessOrder设为true，并且重写removeEldestEntry的判断条件就可以了。
```
public class LruLinkedHashMap<K, V> extends LinkedHashMap<K, V> {

    private Integer capacity;

    public LruLinkedHashMap(int capacity) {
        super(capacity, 0.75f, true);
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }

    public static void main(String[] args) {
        LruLinkedHashMap<Integer, Integer> lru = new LruLinkedHashMap(16);
        for (int i = 0; i < 16; i++) {
            lru.put(i, i);
        }
        System.out.println("当前lru缓存的元素为");
        for (Map.Entry entry : lru.entrySet()) {
            System.out.println(entry.getKey() + ":" + entry.getValue());
        }
        lru.get(0);
        lru.get(1);
        System.out.println("当前lru缓存的元素为");
        for (Map.Entry entry : lru.entrySet()) {
            System.out.println(entry.getKey() + ":" + entry.getValue());
        }
        lru.put(16, 16);
        lru.put(17, 17);
        System.out.println("当前lru缓存的元素为");
        for (Map.Entry entry : lru.entrySet()) {
            System.out.println(entry.getKey() + ":" + entry.getValue());
        }
    }
}
```
测试结果如下，最左边的为最近最久未使用。
```
当前lru缓存的元素为
0:0 1:1 2:2 3:3 4:4 5:5 6:6 7:7 8:8 9:9 10:10 11:11 12:12 13:13 14:14 15:15 
当前lru缓存的元素为
2:2 3:3 4:4 5:5 6:6 7:7 8:8 9:9 10:10 11:11 12:12 13:13 14:14 15:15 0:0 1:1 
当前lru缓存的元素为
4:4 5:5 6:6 7:7 8:8 9:9 10:10 11:11 12:12 13:13 14:14 15:15 0:0 1:1 16:16 17:17 
```