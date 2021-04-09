# LinkedList

LinkedList通过Node节点形成链表结构。  
每个节点维护着当前节点的对象，和上一个节点及下一个节点的内存地址
LinkedList内部维护一个first的节点和last节点，通过这两个可以获取到第一个和最后一个几点，以进行操作

## 类图

![LinkedList](image/LinkedList.jpg)

## 内部类Node

~~~ java

private static class Node<E> {
    // 当前节点的对象
    E item;
    // 下一个节点的内存地址
    Node<E> next;
    // 上一个节点的内存地址
    Node<E> prev;
    // 构造方法。没有空参构造。所有的Node节点都需要声明prev和next节点的内存地址
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}

~~~

## 常量

~~~ java
// 标志LinkedList的长度
transient int size = 0;

// LinkedList的第一个节点
transient Node<E> first;

// LinkedList的最后一个节点
transient Node<E> last;

~~~

## 常用方法

这里主要介绍一个常用的方法，addFirst()

### addFirst()方法

addFirst()实现自Deque接口。在链表的首部添加一个元素

~~~ java
// 在首部添加一个元素
public void addFirst(E e) {
    linkFirst(e);
}

private void linkFirst(E e) {
    // 获取到first节点
    final Node<E> f = first;
    // 将参数的对象创建一个新的节点newNode。next指向原先的first节点
    final Node<E> newNode = new Node<>(null, e, f);
    // 将新的节点作为LinkedList的first节点
    first = newNode;
    // 如果原先的first节点是null
    if (f == null)
        // 那么新的节点同时也是最后一个节点（这种情况是LinkedList为空的时候）
        last = newNode;
    else
        // 如果不是，那么就讲原先的首节点的上一个节点指向新的节点
        f.prev = newNode;
    // LinkedList的大小自增1
    size++;
    // 修改计数自增1
    modCount++;
}

~~~

### addLast()方法

addLast()方法实现自Deque接口。在链表尾部添加一个元素

~~~ java
// 在尾部添加一个元素
public void addLast(E e) {
    linkLast(e);
}

void linkLast(E e) {
    // 获取原先的尾部节点
    final Node<E> l = last;
    // 这里将尾部节点，参数对象构建一个新的Node节点
    final Node<E> newNode = new Node<>(l, e, null);
    // 将尾部节点指向该node
    last = newNode;
    // 如果原先的尾部节点为null
    if (l == null)
        // 那么头部节点也是新节点
        first = newNode;
    else
        // 如果不是，那么原先的尾部节点的next节点为新节点
        l.next = newNode;
    // LinkedList的大小自增1
    size++;
    // 修改计数自增1
    modCount++;
}
~~~
