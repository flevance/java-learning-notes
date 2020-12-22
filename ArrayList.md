# ArrayList源码学习

## 类图

首先放上类图

![ArrayList](image/ArrayList.jpg)

## 常量

~~~ java

// 初始默认大小为10
private static final int DEFAULT_CAPACITY = 10;

// 空的数组（在构造方法为给定大小且=0的时候使用）
private static final Object[] EMPTY_ELEMENTDATA = {};

// 默认无参构造函数使用，代表一个空的数组
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
    // 如果参数的容量大于0
    if (initialCapacity > 0) {
        // 则创建一个给定容量大小的Object数组，赋值给elementData
        this.elementData = new Object[initialCapacity];
    // 如果参数的容量大小=0
    } else if (initialCapacity == 0) {
        // 则将EMPTY_ELEMENTDATA赋值给elementData（就是给定一个空的数组）
        this.elementData = EMPTY_ELEMENTDATA;
    // 如果参数小于0
    } else {
        // 抛出异常，给定的参数不能为initialCapacity异常
        throw new IllegalArgumentException("Illegal Capacity: "+
                                            initialCapacity);
    }
}

// 有参构造函数：参数为一个给定的collection集合（）
public ArrayList(Collection<? extends E> c) {
    // 首先将给定的参数转换为数组，并复制给elementData（toArray是定义在Collection的方法，所有子类必须实现）
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
~~~

