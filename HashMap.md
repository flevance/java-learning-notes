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

可以参考链接：
https://blog.csdn.net/lukabruce/article/details/98033819

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
        // 如果不是，那么就设定新的大小为旧的大小的两倍。
        // 校验是否超过了最大容量并且校验就旧的大小是否超过了初始容量16
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
            // 如果新的容量小于最大容量 且 旧的容量大于初始值。则设定新的阈值为旧的阈值的两倍
            // 这里的两倍就是相当于 新的容量 * 负载因子。因为左移的效率更高
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
        // 如果是，则新的阈值是ft（即新的容量 * 负载因子），如果不是，则是Integer的最大值
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
    }
    // 将新的阈值进行赋值操作
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        // 定义一个新的数组，（新的数组其实就是扩容的数组后的数组）
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    // 将新定义的数组赋值给数组
    table = newTab;
    // 如果就得数组不为null，那还需要将原数组copy过来。如果是null，那么就直接将新数组就返回了
    if (oldTab != null) {
        // 遍历
        for (int j = 0; j < oldCap; ++j) {
            // 声明一个节点e
            Node<K,V> e;
            // 获取旧数组的对应节点位置，判断是否为null
            if ((e = oldTab[j]) != null) {
                // 如果不为null，则将旧的数组的这个节点置为null（置为null的目的是方便GC？）
                oldTab[j] = null;
                // 校验下一个节点是不是null
                if (e.next == null)
                    // e.hash是算出hash值。然后与(newCap -1)进行与操作。然后确定这个Node的新位置
                    // 进行与操作的原因是： 算出的hash值可能超过索引的最大值。在进行与操作之后，就不会超过最大索引了
                    // 例如：容量为16的情况下,e.hash为17，直接放置就会索引越界异常。单将17&(16-1)=10001&1111=1
                    // &是与操作符，一假为假，全真才是真：1&0=0,1&1=1,0&0=0,0&1=0;有一个0就是假，全是1才是1
                    newTab[e.hash & (newCap - 1)] = e;
                // 如果当前节点是树形节点
                else if (e instanceof TreeNode)
                    // 那么就树进行分割操作
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                // 既不是树又不是null,则代表next有值得情况下
                else { // preserve order
                    // 定义低位的头和低位的尾
                    Node<K,V> loHead = null, loTail = null;
                    // 定义高位的头和高位的尾
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    // 重要！！！
                    // 将同一桶中的元素根据(e.hash & oldCap)是否为0进行分割，分成两个不同的链表，完成rehash
                    // (最开始的时候是与old-1进行与操作，决定了原来的位置，现在与old进行与操作。所以根据结构来算，要不就是原先的数不动，要不就是变成了原先的二倍)
                    // 举个例子：
                    // 00101 和 10101 与 (old-1)即 1111 = 101
                    // 但是：当 00101 和 10101 与 old （即10000） 进行与操作时：
                    // 00101 & 10000 = 00000 = 0
                    // 10101 & 10000 = 10000 != 0
                    // 通过这个操作，就完成了分割（就是下面1.2.两个操作）
                    do {
                        // 获取e的next节点
                        next = e.next;
                        // 当e.hash与oldCap进行与操作
                        // (1.)与操作的结果=0就是索引不变的链表（lo）
                        if ((e.hash & oldCap) == 0) {
                            // 如果低位链表的尾是null(即代表现在的这个链表是不是空的)
                            if (loTail == null){
                                 // 如果为空的，就把这个节点添加在头的位置
                                loHead = e;
                            }   
                            // 如果不是空的
                            else{
                                // 那么就讲这个节点添加在尾部节点的下一个位置
                                loTail.next = e;
                            }

                            // 然后就将loTail节点后移一个位置，指向了原先节点的loTail.next
                            // 可以理解为：loTail = loTail.next
                            loTail = e;
                        }
                        // （2.）与操作的结果!=0就是索引变化的链表(hi)
                        else {
                            // 如果高位的头是null
                            if (hiTail == null)
                                // 设定e是高位的头
                                hiHead = e;
                            // 否则
                            else
                                // 设定高位尾的下一个节点是e
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