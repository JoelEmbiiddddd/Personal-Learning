## ArrayList

### ArrayList和LinkedList的区别

- ArrayLis和LinkedList都无法保证线程安全。
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



### HashMap如何进行扩容

HashMap扩容分为两步：

- **扩容：创建一个新的Entry空数组，长度是原数组的2倍。**
- **ReHash：遍历原Entry数组，把所有的Entry重新Hash到新数组。**





### 为什么HashMap在扩容后不需要重新算Hash位置

这个实际上是在JDK1.8中的优化，当hashmap扩容后，位置肯定也会发生变化，n也会发生变化，但并不需要直接全部计算哈希值，而是通过将数据的hash与扩容前的长度进行与操作，根据结果为0还是不为0来做处理。

分析如下：

每次扩容都是2的倍数，计算位置的时候是和数组的长度-1做与操作，那么影响位置的数据只有最高的一位。因此可以根据`e.hash & oldCap`的结果来判断，如果是0，说明位置没有发生变化，如果不为0，说明位置发生了变化，而且新的位置=老的位置+老的数组长度



### HashMap的底层

JDK1.8以前，底层的数据结构使用的是 **数组和链表**，也就是链表散列。

JDK1,.8后，为了解决哈希冲突，进行了修改：

1. 当链表长度大于阈值的时候（默认为8），就会调用`treeifBin()`命令，根据HashMap数组来决定是否变成红黑树。 当数组大小大于或等于64，就会转化为红黑树。
2. **loadFactor：** 负载因子用来控制数组存放数据的疏密程度。一般是0.75f是最好的。**loadFactor 太大导致查找元素效率低，太小导致数组的利用率低，存放的数据会很分散。loadFactor 的默认值为 0.75f 是官方给出的一个比较好的临界值**。
3. **threshold：**threshold = capacity \* loadFactor，**当 Size>threshold**的时候，那么就要考虑对数组的扩增了，也就是说，这个的意思就是 **衡量数组是否需要扩增的一个标准**。







### HashMap多线程下容易导致死循环问题

在jdk1.7之前的版本中，Hashmap在多线程环境下扩容操作死循环的原因是多个元素进行扩容，同时对链表进行操作，头插法容易造成链表指向发生错误，造成死循环。



jdk1.8之后，采用了尾插法。但是还是不建议在多线程下使用 `HashMap`，因为多线程下使用 `HashMap` 还是会存在数据覆盖的问题。并发环境下，推荐使用 `ConcurrentHashMap` 。



### HashMap 为什么线程不安全？

在多线程环境下，`HashMap`扩容的时候容易造成死循环和数据丢失问题。

假设有两个线程A,B，都在进行put操作，并且hash函数计算出的插入下标是相同的，当线程A执行完第六行代码后由于时间片耗尽导致被挂起，而线程B得到时间片后在该下标处插入了元素，完成了正常的插入，然后线程A获得时间片，由于之前已经进行了hash碰撞的判断，所有此时不会再进行判断，而是直接进行插入，这就导致了线程B插入的数据被线程A覆盖了，从而线程不安全。



## ConcurrentHashMap



### 为什么HashTable它的速度比较慢

主要是因为HashMap使用了Synchronized关键字对put等操作进行加表，也就是说在put等修改Hash表的时候，就会锁住整个表，使得其表现很差。



### JDK1.7和JDK1.8的变化

在JDK1.7中，JAVA使用的是分段锁机制来实现ConcurrentHashMap，相当于ConcurrentHashMap在对象中保存了一个segment数组，即将整个hash表划分为多个分段，而每个Segment元素都类型于一个Hashtable。这样，在执行put操作时首先根据hash算法定位到元素属于哪个Segment，然后对该Segment加锁即可。

Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。



在JDK1.8中，ConcurrentHashMap的实现原理摒弃了通过分段锁机制来实现的，所以其最大并发度受Segment的个数限制，而是选择了与HashMap类似的数组+链表 或数组+红黑树的方式实现，而加锁则采用CAS和synchronized实现。



### ConcurrentHashMap的结构

和HashMap相似的结构，也就是数组+链表+红黑树。这里边有几个很重要的属性

1. `sizeCtl`：它是一个控制标识符：
   - 负数代表正在进行初始化或扩容操作
   - 正数表示初始化或下次要进行扩容的大小，一般为0.75
2. `Node类`：是最核心的内部类，所有插入ConcurrentHashMap的数据都会包装在里边，和HashMap类似，唯一不同的就是它对value设置了`volatile`关键字，它不允许调用`setValue`方法直接改变`Node`的`value`域。
3. `TreeNode类：`当链表长度过长的时候，会转换为`TreeNode`。但是与`HashMap`不相同的是，它并不是直接转换为红黑树，而是把这些结点包装成`TreeNode`放在`TreeBin`对象中，由`TreeBin`完成对红黑树的包装。
4. `TreeBin: `这个类并不负责包装用户的`key`、`value`信息，而是包装的很多`TreeNode`节点。它代替了`TreeNode`的根节点，也就是说在实际的`ConcurrentHashMap`“数组”中，存放的是`TreeBin`对象，而不是`TreeNode`对象，这是与`HashMap`的区别。另外这个类还带有了读写锁。
5. ForwardNode类：一个用于连接两个`table`的节点类。它包含一个`nextTable`指针，用于指向下一张表。



### Unsafe和CAS

在`ConcurrentHashMap`中，可以看到大量使用CAS算法来实现无锁化的修改值操作，他可以大大降低锁代理的性能消耗。



### Put方法的流程

由于ConcurrentMap涉及到多线程，会涉及到几点问题：

1. 如果一个或多个线程对ConcurrentHashMap进行扩容操作。当前线程也要进入扩容的操作中。这个扩容的操作之所以能被检测到，是因为`transfer`方法中在空结点上插入`forward`节点，如果检测到需要插入的位置被`forward`节点占有，就帮助进行扩容；
2. 如果检测到插入的节点是非空切不是forward节点，就对这个节点加锁，**保证了线程安全**



### 扩容核心方法 Transfer

当`ConcurrentHashMap`容量不足的时候，需要对`table`进行扩容。这个方法的基本思想跟`HashMap`是很像的，但是由于它是支持并发扩容的，所以要复杂的多。原因是它支持多线程进行扩容操作，而并没有加锁。



整个扩容操作主要分为两部分

1. 第一步构建一个NextTable，是原来的1.75倍，单线程来完成。
2. 第二步就是将原来的table中的元素复制到nexttable中，允许多线程操作。





#### 单线程完成

整体的思想就是遍历、复制的过程。

1. 如果这个位置为空，就在原`table`中的`i`位置放入`forwardNode`节点，这个也是触发并发扩容的关键点；
2. 如果这个位置是`Node`节点（fh>=0），如果它是一个链表的头节点，就构造一个反序链表，把他们分别放在`nextTable`的`i`和`i+n`的位置上
3. 如果这个位置是`TreeBin`节点（fh<0），也做一个反序处理，并且判断是否需要`untreefi`，把处理的结果分别放在`nextTable`的`i`和`i+n`的位置上
4. 遍历过所有的节点以后就完成了复制工作，这时让`nextTable`作为新的`table`，并且更新`sizeCtl`为新容量的`0.75`倍，完成扩容。



#### 多线程下完成

在代码中有一个判断语句，如果遍历到的节点是`forward`节点，就向后继续遍历，再加上给节点上锁的机制，就完成了多线程的控制。多线程遍历节点，处理了一个节点，就把对应点的值`set`为`forward`，另一个线程看到`forward`，就向后遍历。这样交叉就完成了复制工作。而且还很好的解决了线程安全的问题。



## CopyOnWriteArrayList

CopyOnWriteArrayList实现了List接口，List接口定义了对列表的基本操作；同时实现了RandomAccess接口，表示可以随机访问(数组具有随机访问的特性)；同时实现了Cloneable接口，表示可克隆；同时也实现了Serializable接口，表示可被序列化。



### 如何保证线程安全？

实际在copyOnWriteArraylist中，有一个可重入锁ReentrantLock，可以保证线程按照。也使用到了反射机制和CAS来保证原子性的修改lock域。

比如说添加锁的流程是

1. 先获取锁，保证多线程访问的安全，获得当前object的数组和长度。
2. 根据获得的object的数组复制一个长度为n+1的数组。
3. 将下标为length的数组元素设置为元素e，再设置当前Object[]为newElements，释放锁。



另外CopyOnwriteArrayList有一个addIfAbsent，该函数用于添加元素(如果数组中不存在，则添加；否则，不添加，直接返回)，可以保证多线程环境下不会重复添加元素。



### 扩容方法

- 由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致young gc或者full gc

