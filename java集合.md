## ArrayList

### ArrayList和LinkedList最大的区别就是 能否保证线程安全，

- ArrayList可以保证线程安全而LinkedList不行。
- 底层数据结构中，ArrayList是object而linkedList是链表
- ArrayList支持随机访问而linkedlist不行
- ArrayList的空间浪费体现在list列表的结尾都会预留一定的容量空间，而linkedList则是每一个元素都会消耗比Arraylist更多的空间。



### ArrayList插入时扩容

1. 判断长度充足： `ensureCapacityInternal(size+1)`
2. 当长度不足后，就通过扩容函数进行扩容： grow(int minCapacity)
3. 扩容的计算长度是原来的1.5倍
4. 当扩容完以后，就需要进行把数组中的数据拷贝到新数组中，这个过程会用到`Arrays.copyOf(elementData, newCapacity);`，但他的底层用到的是；`System.arraycopy`



### ArrayList 指定位置插入

比如说`list.add(2, "1")`直接插入就会报错

- 指定位置插入首先要判断`rangeCheckForAdd`，size的长度。
- 通过上面的元素插入我们知道，每插入一个元素，size自增一次`size++`。
- 所以即使我们申请了10个容量长度的ArrayList，但是指定位置插入会依赖于size进行判断，所以会抛出`IndexOutOfBoundsException`异常



## Map



### HashTable和HashMap的区别

1. **线程是否安全：** HashTable是线程安全的，这是因为它的内部方法都是经过`synchronized`进行修饰。而HashMap是非线程安全。
2. **对Null key 和 Null value的支持：** `HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个；Hashtable 不允许有 null 键和 null 值，否则会抛出 `NullPointerException`。
3. **初始容量大小和每次扩容大小的不同：** Hashtable默认是11，每次扩容是2n+1。HashMap默认是16，每次扩容是原来的两倍。  如果是给定值，而 `HashMap` 会将其扩充为 2 的幂次方大小。
4. **HashMap的底层结构：** 当链表长度大于阈值的时候，将链表转化为红黑树。而hashtable却不没有。



### HashSet和HashMap的区别

HashSet的底层结构就是HashMap。



### HashMap和TreeMap的区别

二者都继承`AbstractMap`，但是需要注意的是`TreeMap`它还实现了`NavigableMap`接口和`SortedMap` 接口。前者让TreeMap具备了对集合内元素搜索，后者具备根据键排序的能力。

**综上，相比于`HashMap`来说 `TreeMap` 主要多了对集合中的元素根据键排序的能力以及对集合内元素的搜索的能力。**



**如何决定使用 HashMap 还是 TreeMap？**

如果你需要得到一个有序的结果时就应该使用TreeMap（因为HashMap中元素的排列顺序是不固定的）。除此之外，由于HashMap有更好的性能，所以大多不需要排序的时候我们会使用HashMap。



### 为什么HashMap在扩容后不需要重新算Hash位置

这个实际上是在JDK1.8中的优化，当hashmap扩容后，位置肯定也会发生变化，n也会发生变化，但并不需要直接全部计算哈希值，而是通过将数据的hash与扩容前的长度进行与操作，根据结果为0还是不为0来做处理。



分析如下：

每次扩容都是2的倍数，计算位置的时候是和数组的长度-1做与操作，那么影响位置的数据只有最高的一位。因此可以根据`e.hash & oldCap`的结果来判断，如果是0，说明位置没有发生变化，如果不为0，说明位置发生了变化，而且新的位置=老的位置+老的数组长度



### HashMap的底层

JDK1.8以前，底层的数据结构使用的是 **数组和链表**，也就是链表散列。

JDK1,.8后，为了解决哈希冲突，进行了修改：

1. 当链表长度大于阈值的时候（默认为8），就会调用`treeifBin()`命令，根据HashMap数组来决定是否变成红黑树。 当数组大小大于或等于64，就会转化为红黑树。
2. **loadFactor：** 负载因子用来控制数组存放数据的疏密程度。一般是0.75f是最好的。
3. **threshold：**threshold = capacity \* loadFactor，**当 Size>threshold**的时候，那么就要考虑对数组的扩增了，也就是说，这个的意思就是 **衡量数组是否需要扩增的一个标准**。



### HashMap多线程下容易导致死循环问题

在jdk1.7之前的版本中，Hashmap在多线程环境下扩容操作死循环的原因是多个元素进行扩容，同时对链表进行操作，头插法容易造成链表指向发生错误，造成死循环。



jdk1.8之后，采用了尾插法。但是还是不建议在多线程下使用 `HashMap`，因为多线程下使用 `HashMap` 还是会存在数据覆盖的问题。并发环境下，推荐使用 `ConcurrentHashMap` 。



### HashMap 为什么线程不安全？

在多线程环境下，`HashMap`扩容的时候容易造成死循环和数据丢失问题。



## ConcurrentHashMap



`concurrentHashMap`底层使用的就是数组+链表/红黑二叉树。

线程安全：

- JDK1.7的时候使用的是Segment方法来避免并发冲突。
- JDK1.8后，直接使用Node数组+链表+红黑树的数据结构来实现，并发控制使用synchronized。
