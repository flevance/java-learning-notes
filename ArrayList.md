# ArrayList源码学习

## 类图

首先放上类图

![ArrayList](image/ArrayList.jpg)

## 常量

~~~ java

// 初始默认大小为10
private static final int DEFAULT_CAPACITY = 10;

// 空的数组（共享的空数组，对于那种创建）
private static final Object[] EMPTY_ELEMENTDATA = {};

// 无参构造函数使用，代表一个空的数组
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

// ArrayList的元素存放区，只要不是0，就是使用这个，默认的大小是DEFAULT_CAPACITY，即10
transient Object[] elementData;

private int size;

~~~

## 构造方法

~~~ java
// 无参构造函数，这里使用了DEFAULTCAPACITY_EMPTY_ELEMENTDATA，刚创建的时候长度是DEFAULTCAPACITY_EMPTY_ELEMENTDATA，即0。如果往其中放置了任何元素，默认大小都会变为10
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

// 有参构造函数：声明了初始容量的构造函数
public ArrayList(int initialCapacity) {
    
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                            initialCapacity);
    }
}
~~~

