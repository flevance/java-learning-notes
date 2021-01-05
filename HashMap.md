# HashMap源码学习

## 类图

首先上类图

![HashMap](image/HashMap.jpg)

## 常量

~~~ java
// 初始容量大小（16）
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 

// 最大容量 1073741824
static final int MAXIMUM_CAPACITY = 1 << 30;

// 默认负载因子 0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 树化阈值 8 
static final int TREEIFY_THRESHOLD = 8;

// 树退化阈值
static final int UNTREEIFY_THRESHOLD = 6;

// 树的最小容量
static final int MIN_TREEIFY_CAPACITY = 64;

~~~

## PUT方法
~~~ java

public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        // resize()方法是根据已有的元素来重新获tab数组取大小
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
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
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}

/**
重新定义Node数组
**/
final Node<K,V>[] resize() {
    // 定义oldTab位旧的数组
    Node<K,V>[] oldTab = table;
    // 判断旧的数组是否是为null，如果为null，则定义旧的容量为0，否则定义旧的容量为旧的数组的大小
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 从属性中获取阈值。如果旧的数组为null，则threshold就为0。如果不为null,则一般就是旧的大小*负载因子（负载因子0.75）
    int oldThr = threshold;
    // 定义新的数组的大小和阈值为0
    int newCap, newThr = 0;
    // 如果旧的容量大于0
    if (oldCap > 0) {
        // 则判断旧的大小是否超过了最大值（2 ^ 32 次幂）
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 如果是，就给阈值设定为Integer的最大值
            threshold = Integer.MAX_VALUE;
            // 这个时候就不进行扩容，就直接返回原来的数组（扩无可扩了）
            return oldTab;
        }
        // 如果不是，那么就设定新的大小为旧的大小的两倍。并且校验是否超过了最大容量并且校验就旧的大小是否超过了初始容量16
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 并且设定新的阈值是旧的阈值的两倍
            newThr = oldThr << 1; // double threshold
    }
    // 如果旧的容量不大于0且旧的阈值大于0
    else if (oldThr > 0) // initial capacity was placed in threshold
        // 那么就设定新的大小就是旧的阈值
        newCap = oldThr;
    // 如果旧的容量不大于0且旧的阈值不大于0
    else {               // zero initial threshold signifies using defaults
        // 那么就设置新的大小为初始值
        newCap = DEFAULT_INITIAL_CAPACITY;
        // 设置新的阈值为默认负载因子 * 默认初始容量（这种情况一般出现在新new HashMap然后初次put的时候）
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 如果新的容量等于0
    if (newThr == 0) {
        // ft = 新的容量 * 负载因子
        float ft = (float)newCap * loadFactor;
        // 校验新的容量是否小于最大容量 且 ft是否小于最大容量
        // 如果是，则是ft，如果不是，则是Integer的最大值
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
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
~~~