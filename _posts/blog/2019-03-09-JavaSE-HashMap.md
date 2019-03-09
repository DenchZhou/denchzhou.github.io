---
layout: post
title: Java8 HashMap源码分析
categories: JavaSE
description: 源码分析系列之Java8 HashMap
keywords: HashMap
---

直接打上断点开始分析
![image.png](https://upload-images.jianshu.io/upload_images/14607771-9db1e7021c421b92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
先看看构造方法   另外两个构造方法只是可以自己设置初始容器大小和loadfactor  感兴趣的可以自己看一看
```
/**
     * Constructs an empty {@code HashMap} with the default initial capacity
     * (16) and the default load factor (0.75).
       DEFAULT_LOAD_FACTOR默认为0.75f   初始容量默认为16
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```
接下来进入put方法  put方法核心是putVal 
```
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
}

//hash方法为计算哈希值  暂时跳过
//接下来是putVal方法
//olnyIfAbsent为false  则改变现有值
//evict为true 则不为确定的模式
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        //创建一个Node数组   P节点  
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //一开始为空  resize()方法重新分配内存
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // (n-1)&hash为计算数组中的位置  如果未空直接创建一个新的节点
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //不为空则存在冲突  往该位置的链表或者红黑树添加
         else {
            Node<K,V> e; K k;
            //如果当前值的哈希与要加入的哈希相等 并且key也相等  则直接覆盖
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //否则判断是否是红黑树  是的话 往红黑树添加
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //此时不是红黑树 则是链表往链表尾部添加  同时判断是否大于阈值8 大于则转换 
            //成红黑树 即调用treeifyBin  
            //如果没到队尾就发现有哈希值相同 则跳出循环 直接覆盖
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //当数量大于thredshold则扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
putVal方法还算容易理解  就是往一个数组加元素  如果冲突后该节点为链表就往链表添加  如果为红黑树则往红黑树添加  哈希相同且key相同则进行替换
接下来看一下putVal中的 resize()方法
```
/**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        //一开始默认为空
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        //大于0则已经初始化
        if (oldCap > 0) {
            //当前已经达到最大 无需扩充 则threshold为最大  直接返回
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //容量和阈值都翻倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        // 初始容量已存在threshold中
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        //默认构造函数  oldThr为空  则进行初始化  newCap为16   newThr为12
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //计算阈值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        //threshold在这里进行赋值
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        //之前已经初始化  现在开始扩容
        if (oldTab != null) {
            //复制元素，重新进行hash
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
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
由此可见，扩容较为复杂  扩容机制为原来的两倍   同时会遍历每个元素重新hash找位置 比较耗时，应尽量避免。get方法较为简单，就不分析了  大致就是hash  然后找数组上是否存在  存在则判断是否有红黑树  链表等等 

总结：数组大小n总是2的整数次幂，因此计算下标时直接( hash & n-1)，这样的好处就是可以直接取代取模运算，提高计算速度。分配内存初始化通过resize()方法 ，主要时初始化的时候和put方法超过阈值的时候扩容，因此最好是在构造函数就进行初始化。哈希冲突时 转为链表存储，如果链表的长度大于8则转化为红黑树。
