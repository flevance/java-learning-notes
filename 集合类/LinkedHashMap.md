# LinkedHashMap

LinkedHashMap是有序的HashMap。它保留插入的顺序，可以保证输出的顺序和输入时的相同.内部原理是通过维护了一个双向链表（就是两个链表）完成了顺序.
使用场景：有序的，k-v结构的。

## 类图

![LinkedHashMap](image/LinkedHashMap.jpg)

## 属性

~~~ java
// 双向链表的头
transient LinkedHashMap.Entry<K,V> head;
// 双向链表的尾
transient LinkedHashMap.Entry<K,V> tail;
// 此链接的哈希映射的迭代排序方法：对于访问顺序为true ，对于插入顺序为false 
// 设置访问顺序，true代表只要访问就对链表进行排序。false是只有发生插入才对链表进行排序
final boolean accessOrder;
~~~

## 内部类Entry

继承自HashMap的Node，和HashMap的区别是多了一个before和after

~~~ java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
~~~

## 构造方法

~~~ java
public LinkedHashMap() {
    // 使用了父类HashMap的无参构造方法
    super();
    accessOrder = false;
}

// 其余的构造方法都类似，都是走的HashMap的构造方法
~~~

## put()方法

put方法是继承自HashMap，具体的逻辑和HashMap类似。  
区别主要是LinkedHashMap重写了HashMap的afterNodeAccess和afterNodeInsertion()方法（HashMap的这两个方法为空实现)  
这里着重讲这两个方法  
afterNodeAccess是对访问后，将节点移动到尾部节点，需要设置accessOrder为true（这个值默认为false）  
afterNodeInsertion是在插入后，将原先的Node节点进行移除（如果需要移除，需要手动重写removeEldestEntry方法）

~~~ java
/**
 * 节点插入后执行该方法
 * @param evict 标志是否处于创建模式,如果为false，则处于创建模式
 */
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    // 声明一个first的Entry节点
    LinkedHashMap.Entry<K,V> first;
    // 如果处于创建模式，则不会进入判断（因为创建模式也没有需要移除的节点）
    // 获取到head节点，指向first
    // 且需要删除旧的Node
    // removeEldestEntry默认为false，如果需要移除就需要重写这个方法
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        // 获取到链表头部节点的key
        K key = first.key;
        // 继承自HashMap的remove方法
        removeNode(hash(key), key, null, false, true);
    }
}
~~~

~~~ java
// 节点访问后执行该方法，将访问的节点移到队尾
void afterNodeAccess(Node<K,V> e) { // move node to last
    // 声明一个last节点
    LinkedHashMap.Entry<K,V> last;
    // 如果是访问排序且当前节点不是尾部节点的时候
    if (accessOrder && (last = tail) != e) {
        // 获取当前需要排序的节点为P节点，b为p节点的上一个，a为p节点的下一个
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
        // 将p节点指向尾部节点
        tail = p;
        ++modCount;
    }
}
~~~

## remove()方法

remove方法主要讲解LinkedHashMap所重写的afterNodeRemoval方法

~~~ java
// 移除之后需要将数据从双向链表中进行移除
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
~~~
