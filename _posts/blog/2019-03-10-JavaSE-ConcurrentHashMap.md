---
layout: post
title: java8 ConcurrentHashMap源码分析
categories: JavaSE
description: 源码分析系列之Java8 ConcurrentHashMap
keywords: ConcurrentHashMap
---

## put方法
直接进入put方法，同其他集合类，主要内容都在putVal方法中。
putVal方法主要思路如下：
- 计算Hash值
- 判断当前的table是否为空，如果为空则进行初始化操作。
- table不为空则根据Hash值找到对应下标的节点
- - 下标节点为空则通过cas将新节点放入，失败进入循环
- - 如果为ForwardingNode类型，则表示当前其他线程正在扩容，则进入helpTransfer()协助扩容
- - 如果不为空且是普通节点，则对节点上锁，往链表或者红黑树添加。
- cas更新baseCount，并判断是否需要扩容
```
//put方法
public V put(K key, V value) {
  return putVal(key, value, false);
}
//第三个参数若为true, 只有在不存在key的时候才进行put
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        //计算hash值
        int hash = spread(key.hashCode());
        int binCount = 0; //记录链表的长度
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable(); //数组为空进行初始化
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) { //找到hash值对应下标
                if (casTabAt(tab, i, null,  //该位置若为空则通过cas将该值放入，失败则再进入循环
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED) //说明是ForwardingNode类型，需要进行协助扩容
                tab = helpTransfer(tab, f);
            else { //f为该位置的头节点，并且不为空
                V oldVal = null;
                synchronized (f) { //获取头节点的锁
                    if (tabAt(tab, i) == f) { 
                        if (fh >= 0) { //头节点的hash值大于0，说明是链表
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) { //遍历链表
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) { //key相同则覆盖
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) { //到了最末端则直接放到最后面
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) { //红黑树
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
                    if (binCount >= TREEIFY_THRESHOLD) //判断是否转为红黑树 阈值为8
                        treeifyBin(tab, i);  // 不仅仅是红黑树转换，如果数组长度小于64则进行数组扩容
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount); // cas更新baseCount 并判断是否需要扩容
        return null;
    }
```
## initTable() 表初始化
initTable() 初始化代码如下，通过对sizeCtl进行cas操作判断是否抢到锁，如果成功将sizeCtl设置为-1则成功抢到。
```
private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
            if ((sc = sizeCtl) < 0)  //主要是通过cas对 sizeCtl进行赋值  若为-1则表示已经被其它线程抢到
                Thread.yield(); // 让出cpu等待系统调度
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) { // cas将线程设置为-1，成功进入以下代码，失败则进入循环
                try {
                    if ((tab = table) == null || tab.length == 0) { 
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY; //默认为16
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n]; //创建数组
                        table = tab = nt; //赋值给table
                        sc = n - (n >>> 2); //sc 为0.75*n 也就是12
                    }
                } finally {
                    sizeCtl = sc; //设置sizeCtl为sc  12
                }
                break;
            }
        }
        return tab;
    }
```
## helpTransfer()协助扩容
其中扩容状态的sizeCtl的高16位为标识符，低16位为正在扩容的线程数。
```
final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
        Node<K,V>[] nextTab; int sc;
        if (tab != null && (f instanceof ForwardingNode) && // table不为空 且 f为ForwardingNode类型 且f.nextTable不为空，尝试帮助扩容。
            (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
            int rs = resizeStamp(tab.length); // 返回一个16位长度的扩容校验标识
            while (nextTab == nextTable && table == tab && // 说明还在扩容
                   (sc = sizeCtl) < 0) {
                //sizeCtl 如果处于扩容状态的话
                //前 16 位是数据校验标识，后 16 位是当前正在扩容的线程总数
                //这里判断校验标识是否相等，如果校验符不等或者扩容操作已经完成了（sc == rs+1）或者扩容线程数已经满了（sc == rs + MAX_RESIZERS）或者不需要帮忙（transferIndex <= 0），直接退出循环，不用协助它们扩容了
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) { // 帮助扩容的线程数加一
                    transfer(tab, nextTab);
                    break;
                }
            }
            return nextTab;
        }
        return table;
    }
```
## transfer()扩容方法
该方法主要是对原数组进行分段，供线程处理。其中transferIndex为转移的下标，一开始为原数组的末尾。即每段为[transferIndex-stride, transferIndex]。
```
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE) // 将 length/8 然后除以 CPU核心数。如果得到的结果小于 16，那么就使用 16。
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        if (nextTab == null) {            // initiating 新table初始化
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1]; // 两倍扩容
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n; // 更新转移下标，为老的tab的length 指向最后一个桶
        }
        int nextn = nextTab.length;
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab); // 用于标记迁移完成的桶
        boolean advance = true;
        boolean finishing = false; // to ensure sweep before committing nextTab
        for (int i = 0, bound = 0;;) { //i 指向当前桶，bound 指向当前线程需要处理的桶结点的区间下限
            Node<K,V> f; int fh;
            while (advance) {   // 这个循环用来分配任务区间  以及--i往下推进
                int nextIndex, nextBound;
                if (--i >= bound || finishing) 
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {   // 判断是否扩容是否结束
                int sc;
                if (finishing) { 
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) { // 如果没完成则将自己的帮助线程数减一
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT) // 相等则说明没有帮忙的线程了，则扩容结束
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit  重新检查一次
                }
            }
            else if ((f = tabAt(tab, i)) == null)  // 节点为空则放入fwd占位
                advance = casTabAt(tab, i, null, fwd); // 进入任务分配往下推进
            else if ((fh = f.hash) == MOVED) // 说明已经已经处理过了
                advance = true; // already processed
            else {  // 有实际值，并且不是占位符
                synchronized (f) {  上锁
                    if (tabAt(tab, i) == f) { 
                        Node<K,V> ln, hn;
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```