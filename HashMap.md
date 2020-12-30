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