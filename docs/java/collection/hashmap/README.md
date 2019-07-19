## HashMap相关问题

### HashMap有了解吗？简单讲一下
`HashMap`顶层是`Map`接口，以`key-value`键值对的方式存储数据。  
`HashMap`底层基于**数组+链表**形式存在，所以：  
- `HashMap`在内存分布上是一个连续的空间，并且如果空间达到设定的阈值（默认负载因子为0.75）需要进行扩容，每次扩容容量都到2的n次方；
- 它通过`key`的hash来确定`value`在数组中的位置，在出现hash冲突时，比较key的值（`equals()`方法）是否相等，如果相等则覆盖原值，如果不相等采用**拉链法**解决冲突
- 在JDK1.8开始对解决冲突做了优化，当链表长度超过8时，将链表转化为红黑树。

#### 构造方法源码：
HashMap有四个构造方法：
```java
// 一、默认的无参构造方法，初始化了负载因子为默认的0.75
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
// 二、指定数组初始大小的构造方法，实际调用下面的构造方法
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
// 三、使用另一个Map的实例构造HashMap实例，比如将TreeMap快速转化为HashMap
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
// 四、同时指定初始容量和负载因子的构造方法
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        // 初始容量小于0，抛出异常
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        // 初始容量大于最大值（1 << 30相当于2^30），设置为最大值
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        // 负载因子小于等于0或者非浮点数，抛异常
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    // 设置负载因子
    this.loadFactor = loadFactor;
    // 设置初始大小，需要经过tableSizeFor()方法处理为2^n
    this.threshold = tableSizeFor(initialCapacity);
}
```
#### tableSizeFor源码：
这个方法保证了HashMap总是使用2的幂作为哈希表的大小。
```java
static final int tableSizeFor(int cap) {
    // 将期望设置的容量先-1，是为了让计算后的值大于或等于原值
    int n = cap - 1;
    // 将n无符号右移1位（相当于n/2^1），再按位或n，结果给n。
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    // 最后一次右移16位结束，我的理解是：
    // 这个方法目的是为了使容量大小保持在2的n次方，所以移位或操作的目的是为了得到一个全1的二进制，然后再+1就可以得到2的n次方
    // 因此，右移16位相当于n/2^16，而int型占用32位，所以右移16位就一定可以达成目的。
    n |= n >>> 16;
    // 计算之后，n小于0就返回1，大于等于0时需要判断是否到达最大值，够的话直接使用最大值，不够的话就+1。这样返回的容量范围就是[1, 2^30]
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ?  MAXIMUM_CAPACITY : n + 1;
}
```
没看懂移位那块的注释没关系，来个图理解下当入参为10时的运算过程：
>![](./images/tableSizeFor.png)  
[图片来源](https://www.jianshu.com/p/cbe3f22793be)

#### put()源码(JDK1.8):
以下为HashMap在JDK1.8的源码，JDK1.7中没有红黑树的相关操作
```java
public V put(K key, V value) {
    // 直接调用了putVal方法
    return putVal(hash(key), key, value, false, true);
}
// 注意final关键字，意味着这个方法不能被重写
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // 判断table有没有被初始化或者容量为0时，调用resize()扩容方法进行初始化数组
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 1.使用(n - 1) & hash计算出这个key在数组中的位置
    // 2.如果此位置是null的，则创建一个新的节点放入该位置就可以；如果不为null，进入else
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        // 与第一个节点比较，如果hash相等并且key的引用或值也相等，那就记下来一会替换掉就可以了
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 与首节点不相等，且此位置存的是一个树节点
        else if (p instanceof TreeNode)
            // 调用putTreeVal方法，放入树中，方法内也会比较hash的
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            // 这里只开始遍历链表
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    // 到达链表末尾，创建新的节点，追加进链表
                    p.next = newNode(hash, key, value, null);
                    // 加入之后判断链表长度是否到达了8，如果是，将链表转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 比较每个节点，如果重复就跳出循环了，要把这个值给覆盖了
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // e不为null，说明要覆盖原值了（为null的时候说明已经追加上了）
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            // 还要考虑禁止覆盖的情况，允许覆盖了才会真正覆盖值，key好像并没有换
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // 回调方法，实际什么也没干。自行重写
            afterNodeAccess(e);
            return oldValue;
        }
    }
    // 记录操作数
    ++modCount;
    // 判断下大小有没有超过阈值，超过了就扩容
    if (++size > threshold)
        resize();
    // 回调方法，实际什么也没干。自行重写
    afterNodeInsertion(evict);
    // 不知道这个方法设置返回值的意义何在，求大佬解答！
    return null;
}
```
上面除了表面上的逻辑，我还有另外的理解：
- HashMap底层的数组是懒加载的机制，也就是说链表数组在创建HashMap实例时并没有开辟内存，而是在放入键值对时进行了判断，然后再使用设置的容量去申请的内存
- 这个方法内部是没有锁机制的，所以线程不安全
- hash冲突时的解决逻辑是充分考虑大部分场景的使用习惯的，因为它没有直接遍历树或者链表去放值而是先比较第一个节点，在平时多用String类型作为key，在hash冲突时必然会覆盖之前的值，减少了put操作的复杂度

### 为什么链表到达一定程度时，要转化成红黑树呢？二叉查找树或者平衡二叉查找树可以吗？
- 链表是线性结构，在链表中查找节点时要从头节点开始遍历链表，随着链表的增加，查找时间也会增加
- 红黑树是一种自平衡二叉查找树，查找的最坏时间复杂度为O(logn)，因为其特性所以它的插入时自平衡的效率要高于平衡二叉查找树（*这块我也没有深入研究，还是因为菜*）
- 二叉查找树更不可以了，因为当新加节点一直比根节点大或者一直小的话，它就退化成了一个链表

附：红黑树的特性:
>
- 节点是红色或黑色。
- 根是黑色。
- 所有叶子都是黑色（叶子是NIL节点）。
- 每个红色节点必须有两个黑色的子节点。（从每个叶子到根的所有路径上不能有两个连续的红色节点。）
- 从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点。  
![](./images/Red-black_tree_example.png)  
----来自[维基百科](https://zh.wikipedia.org/wiki/%E7%BA%A2%E9%BB%91%E6%A0%91)

### HashMap是线程安全的吗？
通过HashMap源码可知，不是。要想线程安全，使用[ConcurrentHashMap](#ConcurrentHashMap%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E7%9A%84%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%8D%E7%94%A8HashTable)。

### ConcurrentHashMap如何保证线程安全的？为什么不用HashTable？
`ConcurrentHashMap`在JDK1.8中也被优化了，我们分开看：  
1. 在JDK1.8之前，采用分段数组的方式，每段数组单独加锁，如图：
>![](./images/segment.png)  
来源：[ConcurrentHashMap实现原理及源码分析](https://www.cnblogs.com/chengxiao/p/6842045.html)

`Segment`继承了`ReentrantLock`，所以它作为了一个**可重入锁**。在`ConcurrentHashMap`，维护了一个`Segment`数组，每个`Segment`维护一个`HashEntry`数组，并发环境下，对于不同`Segment`的数据进行操作是不用考虑锁竞争的，所以效率会高一些。

2. 从JDK1.8开始，`ConcurrentHashMap`取消了`Segment`分段锁，采用CAS和`synchronized`来保证并发安全。数据结构跟`HashMap`的结构类似(数组+链表/红黑二叉树)，如图：
>![](./images/JDK1.8-ConcurrentHashMap-Structure.jpg)

`synchronized`只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升N倍。

3. 至于现在为什么不用HashTable，看过源码一下就明白了：HashTable内部是直接在方法上加入`synchronized`来处理并发的，所以不管会不会发生并发问题，都要进行锁竞争，因此会很慢，一张图作对比：  
>![](./images/HashTable.png)

### 为什么数组大小一定是2的n次方（每次扩容都要到2的n次方）？

通过`tableSizeFor()`方法可以看到，`HashMap`并不是每次都用我们制定的大小去初始化数组的，而是将我们指定大小向上计算为最接近的2的n次方，如我们指定13，那么实际初始化的大小就是16，指定17就是32。
我是这么个思路：
1. 先看下`putVal()`方法里计算key位置的算法：
```java
// n为数组大小，hash为hash(Object key)方法计算结果
// 在数组大小为2的幂次方的前提下，这里等同于hash%n，但是位运算效率更高
i = (n - 1) & hash
```

2. 可以发现，当n为2的幂次方时，n-1的二进制就全为1，比如默认n是16，二进制为`10000`，那n-1的二进制就是`1111`，保持了相对稳定，在与hash进行&运算时，运算结果是由key的hash值决定的。  
试想：如果n可以是任意值（比如17，二进制10001），那计算结果很有可能计算结果大量相同（结果集中在`10001/10000/00000/00001`），导致**数组分布不均匀**

>更详细的解释请看[这篇博客](https://blog.csdn.net/zjcjava/article/details/78495416)

### 一百万个key-value键值对要放到HashMap里，需要注意什么？

