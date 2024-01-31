# JavaCollection

## Java集合 <a href="#usercontent-yin-yan" id="usercontent-yin-yan"></a>

**1、说说有哪些常见集合？**

集合相关类和接口都在`java.util`中，主要分为3种：List(列表)、Map(映射)、Set(集合)

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption><p>集合</p></figcaption></figure>

其中`Collection`是集合`List`、`Set`的父接口，它主要有两个子接口：

* `List`：存储的元素有序，可重复
* `Set`：存储的元素不无序，不可重复
* `Map`：是另外的接口，是键值对映射结构的集合

***

## List <a href="#user-content-list" id="user-content-list"></a>

### **1、ArrayList、LinkedList、Vector有什么区别？**

**数据结构不同**

* ArrayList基于数组实现
* LinkedList基于双向链表实现

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption><p>ArrayList、LinkedList</p></figcaption></figure>

**ArrayList更利于查找，LinkedList更利于增删**

* ArrayList基于数组实现，`get(int index)`可以直接通过数组下标获取，时间复杂度O(1)；LinkedList基于链表实现，`get(int index)`需要遍历链表，时间复杂度是O(n)；当然，get(E element)这种查找，两种集合都需要遍历，时间复杂度都是O(n)
* ArrayList增删如果是数组末尾的位置，直接插入或者删除就可以了，但是如果插入中间的位置，就需要把插入位置后的元素都向前或者向后移动，甚至还有可能触发扩容；双向链表的插入和删除只需要改变前驱节点、后继节点和插入节点的指向就行了，不需要移动元素

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption><p>插入</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>删除</p></figcaption></figure>

注意：这个地方放可能会出陷阱，LinkedList更利于增删更多是体现在平均步长上。不是体现在是时间复杂度上，二者增删的时间复杂度都是O(n)

**是否支持随机访问**

* ArrayList基于数组，所以它可以根据下标查找，支持随机访问，当然它也实现了RandmoAccess接口，这个接口只是用来标识是否支持随机访问
* LinkedList基于链表，所以它没法根据序号直接获取元素，它没有实现RandmoAccess接口，标记不支持随机访问

**内存占用**

* ArrayList基于数组，是一块连续的内存空间
* LinkedList基于链表，内存空间不连续，它们在空间占用上都有一些额外的消耗
* ArrayList是预先定义好的数组，可能会有空的内存空间，存在一定空间浪费
* LinkedList每个节点，需要存储前驱和后继，所以每个节点会占用更多的空间

**Vector的特点**

Vector向量势最早出现的集合容器，与ArrayList一样，也是通过数组实现的；vector是一个线程安全的容器，它的每个方法都粗暴的加上了`synchronized`锁，所以它的增删、遍历效率都很低；Vector在扩容时，它的容量会增加一倍

### **2、ArrayList的扩容机制了解吗？**

ArrayList是基于数组的集合，数组的容量是在定义的时候确定的，如果数组满了，在插入，就会数组溢出；所以在插入时候，会先检查是否需要扩容，如果当前容量+1超过数组长度，就会进行扩容

ArrayList的扩容是创建一个1.5倍的新数组，然后把原数组的值拷贝过去

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption><p>扩容</p></figcaption></figure>

```java
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

### **3、ArrayList怎么序列化的知道吗，为什么用transient修饰数组？**

ArrayList的序列化不太一样，它使用`transient`修饰存储元素的elementData的数组，`transient`关键字的作用是让被修饰的成员属性不被序列化

**为什么ArrayList不直接序列化元素数组呢？**

出于效率的考虑，数组可能长度100，但实际只用了50，剩下的50不用其实不用序列化，这样可以提高序列化和反序列化的效率，还可以节省内存空间

**那ArrayList怎么序列化呢？**

ArrayList通过两个方法readObject、writeObject自定义序列化和反序列化策略，实际直接使用两个流`ObjectOutputStream`和`ObjectInputStream`来进行序列化和反序列化

```java
/**
* ArrayList自定义序列化
*/
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException{
    // fail-fast 后续判断是否有并发处理
    int expectedModCount = modCount;
    // 序列化没有标记为 static、transient的字段，包括size等
    s.defaultWriteObject();

    s.writeInt(size);

    // 序列化数组的前size个元素
    for (int i=0; i<size; i++) {
        s.writeObject(elementData[i]);
    }

    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}

/**
* ArrayList自定义反序列化
*/
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // 反序列化没有标记为 static、transient的字段，包括size等
    s.defaultReadObject();

    s.readInt(); // ignored

    if (size > 0) {
        // 数组扩容
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // 反序列化元素并填充到数组中
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```

### **4、快速失败(fail-fast)和安全失败(fail-safe)了解吗？**

**快速失败(fail-fast)**：快速失败是Java集合的一种错误检测机制

* 在用迭代器遍历一个集合对象时，如果线程A遍历过程中，线程B对集合对象的内容进行了修改(增加、删除、修改)，则会抛出`Concurrent Modification Exception`
* 原理：迭代器在遍历时直接访问集合中的内容，并且在便利过程中使用一个`modCount`变量；集合在被遍历期间如果内容发生变化，就会改变`modCount的值`；每当迭代器使用`hashNext()/next()`遍历下一个元素之前，都会检测`modCount`变量是否为`expectedmodCount`值，是的话就返回遍历；否则抛出异常，终止遍历
* 注意：这里异常的抛出条件是检测到`modCount!=exoectedmodCount`这个条件；如果集合发生变化时修改`modCount`值刚好又设置为了`expectedmodCount`值，则异常不会抛出；因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常之建议用与检测并发修改的bug
* 场景：`java.util`包下的集合类都是快速失败的，不能再多线程下发生并发修改(迭代过程中被修改)比如ArrayList类

**安全失败(fail-safe)**：

* 采用安全失败机制的集合容器，在遍历时不直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历
* 原理：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发`Concurrent Modification Exception`
* 缺点：基于拷贝内容的有点是避免了`Concurrent Modification Exception`，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的
* 场景：`java.util.concurrent`包下的容器都是安全失败，可以在多线程下并发使用，并发修改，比如CopyOnWriteArrayList类

### **5、有哪集中实现ArrayList线程安全的方法？**

fail-fast是一种可能触发的机制，实际上ArrayList的线程安全任然没有保证，一般，保证ArrayList的线程安全可以通过以下方案：

* 使用Vector代替ArrayList(不推荐，Vector是一个历史遗留类)
* 使用Collections.synchronizedList包装ArrayList，然后操作包装后的List
* 使用CopyOnWriteArrayList代替ArrayList
* 在使用ArrayList时，应用程序通过同步机制去控制ArrayList的读写

### **6、CopyOnWriteArrayList了解多少？**

CopyOnWriteArrayList就是线程安全版本的ArrayList

它的名字叫`CopyOnWrite`—写时复制，已经明示了它的原理

CopyOnWriteArrayList采用了一种读写分离的并发策略；CopyOnWriteArrayList容器允许并发读，读操作是无锁的，性能较高；至于写操作，比如向容器中添加一个元素，则首先将当前容器复制一份，然后再新的副本上执行写操作，结束之后再将原容器的引用指向新容器

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption><p>CopyOnWriteArrayList</p></figcaption></figure>

### **7、Arrays.asList获得的List应该注意什么？**

Arrays.asList是Array种的一个静态方法，它能够实现把数组转换成List序列，需要注意下面几点

*   Arrays.asList转换完成后的List不能再进行结构化的修改，什么是结构化的修改？就是不能再进行任何List元素的增加或减少的操作

    ```java
    public static void main(String[] args) {
      Integer[] integer = new Integer[] { 1, 2, 3, 4 };
      List integetList = Arrays.asList(integer);
      integetList.add(5);
    }

    Exception in thread "main" java.lang.UnsupportedOperationException
    ```

    Arrays.asList返回的List不是java.util.ArrayList，而是Arrays的内部类ArrayList

    源码中继承了AbstractList中对add、remove、set方法是直接抛出异常的，也就是说如果继承的子类没有去重写这些方法，那么子类的实例去调用这些方法是会直接抛异常

    ```java
    // 这是 java.util.Arrays 的内部类，而不是 java.util.ArrayList 
    private static class ArrayList<E> extends AbstractList<E>
            implements RandomAccess, java.io.Serializable
        
    public void add(int index, E element) {
      throw new UnsupportedOperationException();
    }
    public E remove(int index) {
      throw new UnsupportedOperationException();
    }
    public E set(int index, E element) {
      throw new UnsupportedOperationException();
    }
    ```

    虽然set方法也抛出异常了，但是由于内部类ArrayList重写了set方法，所以支持其可以对元素进行修改

    ![Set](https://gitee.com/yifanstudy/interview-javacollection/raw/master/Java%E9%9B%86%E5%90%88.assets/1657440977885.png)

    * `解决`：将Arrays.asList返回List用java.util.ArrayList包起来`new ArrayList(Arrays.asList(arr))`
*   Arrays.asList不支持基础类型的转换

    Java中的基础数据类型(byte、char、short、int、long、float、double、boolean)

    * `原因`：Arrays.asList(arr)传入是一个泛型，相当把int\[]类型传入，实际将数组作为了list的元素
    * `解决`：使用基本数据类型的包装类；使用Stream流的方式`Arrays.stream()`

### **8、Collection和Cellections的区别**

Collection和Cellections都是位于java.util包下的类

Collection是集合类的父类，它是一个顶级接口，大部分抽象必须说`AbstractList`、`AbstractSet`都继承了Collection类，Collection类只定义一些标准方法比如说add、remove、set、equals等，具体的方法由抽象类或者实现类去实现

Collections是集合类的工具类，Collections提供了一些工具类的基本使用

* sort方法，对当前集合进行排序，实现了Comparable接口的类，只能使用一种排序方案，这种方案叫做自然比较
* 比如实现线程安全的容器`Collections.synchronizedList`、`Collections.synchronizedMap`等
* reverse反转，使用reverse方法可以根据元素的自然顺序对指定列表按降序进行排序
* fill，使用指定元素代替指定列表中的所有元素

有很多方法，可以点击Collections源码查看，Collections不能被实例化，所以Collections中的方法都是由`Collections.方法`直接调用

***

## Map <a href="#user-content-map" id="user-content-map"></a>

### **1、能说一下HashMap的数据结构吗？**

JDK1.7的数据结构是`数组+链表`

JDK1.8的数据结构是`数组+链表+红黑树`

<figure><img src="../.gitbook/assets/image (31).png" alt=""><figcaption><p>HashMap数据结构</p></figcaption></figure>

其中，桶数组是用来存储数据元素，链表是用来解决冲突，红黑树是为了提高查询的效率

* 数据元素通过映射关系，也就是散列函数，映射到桶数组对应索引的位置
* 如果发生冲突，从冲突的位置拉一个链表，插入冲突的元素
* 如果`链表长度>8 & 数组大小>=64`，链表转为红黑树
* 如果`红黑树节点个数<6`，转为链表

### **2、对红黑树了解多少，为什么不用二叉树/平衡树呢？**

红黑树本质上是一种二叉查找树，为了保持平衡，它又在二叉查找树的基础上增加了一些规则：

* 每个节点要么是红色，要么是黑色
* 根节点永远是黑色的
* 所有的叶子节点都是黑色的(注意这里说叶子节点其实是图中的NULL节点)
* 每个红色节点的两个子节点一定都是黑色
* 从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption><p>红黑树</p></figcaption></figure>

> 之所以不用二叉树

红黑树是一种平衡的二叉树，插入、删除、查找的最坏时间复杂度都为O(logn)，避免了二叉树最坏情况下的O(n)时间复杂度

> 之所以不用平衡二叉树

平衡二叉树是比红黑树更严格的平衡树，为了保持平衡，需要旋转的次数更多，也就是说平衡二叉树保持平衡的效率更低，所以平衡二叉树插入和删除的效率比红黑树要低

**3、红黑树怎么保持平衡的知道吗？**

红黑树有两种方式保持平衡：`旋转和染色`

* 旋转：旋转分为两种，左旋和右旋

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption><p>旋转</p></figcaption></figure>

* 染色

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>染色</p></figcaption></figure>

### **4、HashMap的put流程知道吗？**

流程图：

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption><p>HashMap.put</p></figcaption></figure>

* 首先进行哈希值的扰动，获取一个新的哈希值；`(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)`
*   判断tab数组是否为空或者长度为0，如果是则进行扩容操作

    ```
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    ```
* 根据哈希值计算下标，如果对应下标正好没有存放数据，则直接插入即可否则需要覆盖；`tab[i = (n - 1) & hash]`
* 判断tab\[i]是否为树节点，否则向链表中插入数据，是则向树中插入节点
* 如果链表中插入节点的时候，链表长度大于等于8，则需要把链表转换为红黑树；`treeifyBin(tab，hash)`
* 最后所有元素处理完成后，判断是否超过阈值；`threshold`，超过则扩容

### **5、HashMap怎么查找元素的呢？**

流程图：

<figure><img src="../.gitbook/assets/image (4).png" alt=""><figcaption><p>HashMap.get</p></figcaption></figure>

* 使用扰动函数，获取新的哈希值
* 计算数组下标，获取节点
* 当前节点和key匹配，直接返回
* 否则，当前节点是否为树节点，查找红黑树
* 否则，遍历链表查找

### **6、HashMap的哈希/扰动函数是怎么设计的？**

HashMap的哈希函数是先拿到Key的hashcode，是一个32位的int类型的数值，然后让hashcode的高16位和低16位进行异或操作

```java
static final int hash(Object key) {
    int h;
    // key的hashCode和key的hashCode右移16位做异或运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

这么设计是为了降低哈希碰撞的概率

### **7、为什么哈希/扰动函数能降低hash碰撞？**

因为key.hashCode()函数调用的是key键值类型自带的hash函数，返回int型散列值；int值范围为-2^31\~2^31-1，加起来大概40亿的映射空间

只要哈希函数映射的比较均匀松散，一般应用是很难出现碰撞的；但问题是一个40亿长度的数组，内存是放不下的

例如HashMap数组的初始大小才16，在用之前需要对数组的长度取模运算，得到的余数才能用来访问数组下标

源码中模运算就是把散列值和数组长度-1做一个`与&`操作，位运算比取余%运算要快

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    ...
    // (n - 1) & hash 数组长度-1 & hash
	if ((tab = table) == null || (n = tab.length) == 0)
    	n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
    ...
}
```

这也正好解释了为什么HashMap的数组长度要取2的整数幂；因为这样(数组长度-1)正好相当于一个"低位掩码"；`与`操作的结果就是散列值的高位全部归零，只保留低位值，用来做数组下标访问；以初始长度16为例，16-1=15；2进制表示为`0000 0000 0000 0000 0000 0000 0000 1111`；和某个散列值做`与`操作如下，结果就是截取了最低的四位值

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption><p>数组长度减一取模</p></figcaption></figure>

这样是要快捷一些，但是新的问题来了，就算散列值分布再松散，要是只取最后几位的话，碰撞也会很严重；如果散列本身做得不好，分布上成等差数列的漏洞，如果正好让最后几个低位呈现规律性重复，那就更难搞了

这时候`扰动函数`的价值就体现出来了，看一下扰动函数的示意图：

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption><p>扰动函数</p></figcaption></figure>

右移16位，正好是32bit的一半，自己的高半区和低半区做异或，就是为了混合原始哈希码的高位和低位，以此来加大低位的随机性；而且混合后的低位掺杂了高位的部分特征，这样高位的信息也被变相保留下来

### **8、为什么HashMap的容量是2的倍数？**

*   **方便哈希取余**

    将元素放在table数组上面，使用hash值%数组大小定位位置，而HashMap使用hash值&(数组大小-1)，却能和前面达到一样的效果，这就得益于HashMap的大小是2的倍数，2的倍数意味着该数的二进制位只有一位为1，而该数-1就可以得到二进制位上1变成0，后面的0变成1，再通过&运算，就可以得到和%一样的效果，并且位运算比%的效率高得多

    HashMap的容量是2的n次幂时，(n-1)的2进制也就是11111...111这样形式的，这样与添加元素的hash值进行位运算时，能够充分的散列，使得添加的元素均匀分布在HashMap的每个位置上，减少hash碰撞
*   第二个方面是在扩容是，利用扩容后的大小也是2的倍数，将已经产生hash碰撞的元素完美的转移到新的table中去

    可以简单看看HashMap的扩容机制，Hash Map中的元素在超过`负载因子*HashMap的大小`时就会产生扩容

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption><p>HashMap扩容</p></figcaption></figure>

### **9、如果初始化HashMap，传入一个17的值，他会怎么处理？**

简单来说，就是初始化时，传的不是2的倍数时，HashMap会向上寻找`离得最近的2的倍数`，所以传入17，但HashMap的实际容量是32

在HashMap的初始化中，有这样一段方法：

```java
public HashMap(int initialCapacity, float loadFactor) {
    ...
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

* 阈值threshold，通过方法`tableSizeFor`进行计算，是根据初始化传的参数来计算的
* 同时，这个方法也要寻找初始值大的，最小的那个2进制值；比如传了17，应该找到的是32

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

* MAXIMUM\_CAPACITY = 1 << 30，这个是临界范围，也就是最大的Map集合
* 计算过程是向右移位1、2、4、8、16，和原来的数做`|`或运算，这主要是为了把二进制的各个位置都填上1，当二进制的哥哥位置都是1以后，就是一个标准的2的倍数减一了，最后把结果加1再返回即可

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption><p>初始化过程</p></figcaption></figure>

### **10、还知道哪些哈希函数的构造方法呢？**

HashMap里哈希构造函数的方法叫：

*   **除留取余法**

    H (key)=key%p (p<=N)，关键字除以一个不大于哈希表长度的正整数p，所得余数为地址，当然HashMap里面进行了优化改造，效率更高，散列也更均衡
*   **直接定址法**

    直接根据key来映射到对应的数组位置，例如1232放到下标1232的位置
*   **数字分析法**

    取key的某些数字(例如十位和百位)作为映射的位置
*   **平方取中法**

    取key平方的中间几位作为映射的位置
*   **折叠法**

    将key分割成位数相同的几段，然后把它们的叠加和作为映射位置

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption><p>hash方法</p></figcaption></figure>

### **11、解决哈希冲突有哪些方法？**

HashMap中使用链表的原因就是为了处理哈希冲突，这种方法就是所谓的：

*   **链地址法**：在冲突的位置拉一个链表，把冲突的元素放进去

    除此之外，还有一些常见的解决冲突的办法
*   **开放定址法**：开放定址法就是从冲突的位置再接着往下找，给冲突元素找个空位

    找到空闲位置的方法也有很多种：

    * 线性探测法：从冲突的位置开始，依次判断下一个位置是否空闲，直至找到空闲位置
    * 平方探测法：从冲突的位置x开始，第一次增加`1^2`个位置，第二次增加`2^2`...，直到找到空闲的位置

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption><p>链地址、开放定址</p></figcaption></figure>

* **再哈希法**：换种哈希函数，重新计算冲突元素的地址
* **建立公共溢出区**：再建一个数组，把冲突的元素放进去

**12、为什么HashMap链表转红黑树的阈值为8呢？**

数化发生在table数组的长度大于64，且链表的长度大于8的时候

为什么是8呢？源码的注释也给出了答案

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption><p>转红黑树的阈值注释</p></figcaption></figure>

红黑树节点的大小大概是普通节点大小的两倍，所以转红黑树，牺牲了空间换时间，更多的是一种兜底的策略，保证极端情况下的查找效率

阈值为什么要选8呢？和统计学有关；理想情况下，使用随机哈希码，链表里的节点符合泊松分布，出现节点个数的概率是递减的，节点个数为8的情况，发生概率仅为`0.00000006`

至于红黑树转回链表的阈值为什么是6，而不是8？是因为如果这个阈值也设置成8，假如发生碰撞，节点增减刚好在8附近，会发生链表和红黑树的不断转换，导致资源浪费

### **13、扩容在什么时候，为什么扩容因子是0.75？**

为了减少哈希冲突发生的概率，当前HashMap的元素个数达到一个临界值的时候，就会触发扩容，把所有元素rehash之后再放在扩容后的容器中，这是一个相当耗时的操作

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption><p>扩容</p></figcaption></figure>

而这个`临界值threshold`就是由加载因子和当前容器的容量大小来确定的，假如采用默认的构造方法：

> 临界值（threshold）= 默认容量（DEFAULT\_INITIAL\_CAPACITY）\* 默认加载因子（DEFAULT\_LOAD\_FACTOR）

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption><p>临界值</p></figcaption></figure>

那就是大于`16x0.75=12`时，就会触发扩容操作

> 那为什么选择了0.75作为HashMap的默认加载因子呢？

简单来说，就是对`空间`成本和`时间`成本平衡的考虑

在HashMap中有这样一段注释：

<figure><img src="../.gitbook/assets/image (14).png" alt=""><figcaption><p>HashMap加载因子</p></figcaption></figure>

HashMap的散列构造方式是Hash取余，负载因子决定元素个数达到多少时扩容

假如设的比较大，元素比较多，空位比较少的时候才扩容，那么发生哈希冲突的概率就增加了，查找的时间成本就增加了

设的比较小的话，元素比较少，空位比较多的时候就扩容，发生哈希碰撞的概率就降低了，查找时间成本降低，但是就需要更多的空间存储元素，空间成本就增加了

### **14、那扩容机制了解吗？**

HashMap是基于数组+链表+红黑树实现的，但用于存放key值得桶数组的长度是固定的，由初始化参数确定

那么，随着数据的插入数量增加以及负载因子的作用下，就需要扩容来存放更多的数据；而扩容中有一个非常重要的点，就是JDK1.8中的优化操作，可以不需要再重新计算每一个元素的哈希值

因为HashMap的初始容量是2的次幂，扩容之后的长度是原来的两倍，新的容量也是2的次幂，所以，元素，要么在原位置，要么在原位置再移动2的次幂

看下图，n为table的长度，图`a`表示扩容前的key1和key2两种key确定索引的位置，图`b`表示扩容后key1和key2两种key确定索引位置

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption><p>扩容后索引位置</p></figcaption></figure>

元素在重新计算hash之后，因为n变成2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption><p>索引变化</p></figcaption></figure>

所以在扩容时，只需要看原来的hash值新增的那一位是0还是1就行了，是0的话索引没变，是1的话变成`原索引+oldCap`，看看如16扩容为32的示意图：

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption><p>16扩容32</p></figcaption></figure>

扩容节点迁移主要逻辑：

```java
// final Node<K,V>[] resize()
if (e.next == null)
    // 未形成链表，直接分配到新tbale
    newTab[e.hash & (newCap - 1)] = e;
else if (e instanceof TreeNode)
    // 数节点分割
    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
else { // preserve order
    // 链表拆成两个链表lo、hi，两个链表的头、尾节点
    Node<K,V> loHead = null, loTail = null;
    Node<K,V> hiHead = null, hiTail = null;
    Node<K,V> next;
    do {
        // 遍历链表，将节点放到相应的新链表
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
    // lo链表放到新table原位置
    if (loTail != null) {
        loTail.next = null;
        newTab[j] = loHead;
    }
    // hi链表放到j+oldCap位置
    if (hiTail != null) {
        hiTail.next = null;
        newTab[j + oldCap] = hiHead;
    }
}
```

### **15、JDK1.8对HashMap主要做了哪些优化呢，为什么？**

JDK1.8的HashMap主要有五个优化：

*   **数据结构**：数组+链表改成了数组+链表+红黑树

    `原因`：发生hash冲突，元素会存入链表，链表过长转为红黑树。将时间复杂度由O(n)降为O(logn)
*   **链表插入方式**：链表的插入方式从头插改成了尾插法

    简单来说就是插入时，如果数组位置上已经有元素，1.7将新元素放到数组中，原始节点作为新节点的后继节点，1.8遍历链表，将元素放置到链表的最后

    `原因`：因为1.7头插法扩容时，头插法会使链表发生反转，多线程环境下会产生环
*   **扩容rehash**：扩容的时候1.7需要对原数组的元素进行重新hash定位在新数组的位置，1.8采用更简单的判断逻辑，不需要重新通过哈希函数计算位置，新的位置不变或索引+新增容量大小

    `原因`：提高扩容的效率，跟快的扩容
* **扩容时机**：在插入时，1.7先判断是否需要扩容，再插入；1.8先进行插入，插入完成在判断是否需要扩容
*   **散列函数**：1.7做了四次位移和四次异或，1.8只做了一次

    `原因`：做四次的话，边际效用也不大，改为一次，提升效率

### **16、你能自己设计实现一个HashMap吗？**

这道题**快手**常考

不要慌，红黑树版多半是写不出来，但是数组+链表版还是问题不大的，详细可见：

[**手写HashMap，快手面试官直呼内行**](https://gitee.com/link?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FZ9yoRZW5itrtgbS-cj0bUg)

整体的设计：

* 散列函数：hashCode()+除留取余法
* 冲突解决：链地址法
* 扩容：节点重新hash获取位置

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption><p>HashMap手写的设计点</p></figcaption></figure>

```java
/**
 * @description:
 * @author: 刈凡
 * @create: 2022/7/9 23:50
 */
public class ThirdHashMap<K, V> {
    // 默认容量
    final int DEFAULT_CAPACITY = 1 << 4;
    // 负载因子
    final float LOAD_FACTOR = 0.75f;
    // HashMap的大小
    int size;
    // 桶数组
    Node<K, V>[] buckets;

    /**
     * 无参构造器，设置桶数组默认容量
     */
    public ThirdHashMap() {
        buckets = new Node[DEFAULT_CAPACITY];
        size = 0;
    }

    /**
     * 有参构造器，指定桶数组容量
     * @param capacity
     */
    public ThirdHashMap(int capacity) {
        buckets = new Node[capacity];
        size = 0;
    }

    /**
     * 哈希函数，获取地址
     * @param key
     * @param length
     * @return
     */
    private int getIndex(K key, int length) {
        // 获取hash code
        int hashCode = key.hashCode();
        // 和桶数组长度取余
        int index = hashCode % length;
        return Math.abs(index);
    }

    /**
     * put方法
     * @param key
     * @param value
     */
    public void put(K key, V value) {
        // 判断是否需要进行扩容
        if (size >= buckets.length * LOAD_FACTOR) {
            resize();
        }
        putVal(key, value, buckets);
    }

    /**
     * 将元素存入指定的node数组
     * @param key
     * @param value
     * @param table
     */
    private void putVal(K key, V value, Node<K, V>[] table) {
        // 获取位置
        int index = getIndex(key, table.length);
        Node node = table[index];
        // 插入的位置为空
        if (node == null) {
            table[index] = new Node<>(key, value);
            size++;
            return;
        }
        // 插入位置不为空，说明发生冲突，使用连地址，遍历链表
        while (node != null) {
            // 如果key相同，就覆盖
            if ((node.key.hashCode() == key.hashCode()
                    && (node.key == key || node.key.equals(key)))) {
                node.value = value;
                return;
            }
            node = node.next;
        }
        // 当前key不存在链表中，插入链表头部
        Node<K, V> newNode = new Node<>(key, value, table[index]);
        table[index] = newNode;
        size++;
    }

    /**
     * 扩容
     */
    private void resize() {
        // 创建一个两倍容量的桶数组
        Node[] newBuckets = new Node[buckets.length * 2];
        // 将当前元素重新散列到新的桶数组中
        rehash(newBuckets);
        buckets = newBuckets;
    }

    /**
     * 重新散列当前元素
     * @param newBuckets
     */
    private void rehash(Node<K, V>[] newBuckets) {
        // map大小重新计算
        size = 0;
        // 将旧的桶数组的元素全部刷到新的桶数组组里
        for (int i = 0; i < buckets.length; i++) {
            // 为空，跳过
            if (buckets[i] == null) {
                continue;
            }
            Node<K, V> node = buckets[i];
            while (node != null) {
                // 将元素放入新数组
                putVal(node.key, node.value, newBuckets);
                node = node.next;
            }
        }
    }

    /**
     * 获取元素
     * @param key
     * @return
     */
    public V get(K key) {
        // 获取key对应的地址
        int index = getIndex(key, buckets.length);
        if (buckets[index] == null) {
            return null;
        }
        Node<K, V> node = buckets[index];
        // 查找链表
        while (node != null) {
            if ((node.key.hashCode() == key.hashCode()
                    && (node.key == key || node.key.equals(key)))) {
                return node.value;
            }
            node = node.next;
        }
        return null;
    }
}

class Node<K, V> {
    // 键值对
    K key;
    V value;

    // 链表，后继
    Node<K, V> next;

    public Node(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public Node(K key, V value, Node<K, V> next) {
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```

```java
/**
 * @description:
 * @author: 刈凡
 * @create: 2022/7/10 0:27
 */
public class TestHashMap {
    public static void main(String[] args) {
        ThirdHashMap map = new ThirdHashMap();
        for (int i = 0; i < 100; i++) {
            map.put("刈凡" + i, "你这瓜保熟吗？" + i);
        }        
        System.out.println(map.size);
        System.out.println(map.buckets.length);
        
        for (int i = 0; i < 100; i++) {
            System.out.println(map.get("刈凡" + i));
        }        
        
        map.put("刈凡1","哥们，你这瓜保熟吗？");
        map.put("刈凡1","你这瓜熟我肯定要啊！");
        System.out.println(map.get("刈凡1"));
    }
}
```

### **17、HashMap是线程安全的吗，多线程下会有什么问题？**

HashMap不是线程安全的，可能会发生这些问题：

* 多线程下扩容死循环；JDK7中的HashMap使用头插法插入元素，在多线程的环境下，扩容的时候有可能导致环形链表的出现，形成死循环(头插法会将原来的链表倒序，在多线程下就有可能出现环)；因此，JDK8使用尾插法插入元素，在扩容时会保持链表元素原本的顺序，不会出现环形链表的问题
* 多线程的put可能导致元素的丢失；多线程同时执行put操作，如果计算出来的索引位置是相同的，那会造成前一个key被后一个key覆盖，从而导致元素的丢失；此问题在JDK7和JDK8中都存在
* put和get并发时，可能导致get为null；线程1执行put时，因为元素个数超出threshold而导致rehash，线程2此时执行get，有可能导致这个问题；这个问题在JDK7和8中都存在

### **18、有什么办法能解决HashMap线程不安全的问题呢？**

Java中有HashTable、Collections.synchronizedMap、以及ConcurrentHashMap可以实现线程安全的Map

* HashTable是直接在操作方法上加synchronized关键字，锁住整个table数组，粒度比较大
* Collections.synchronizedMap是使用Collections集合工具的内部类，通过传入Map封装出一个SyschronizedMap对象，内部定义了一个对象锁，方法内通过对象锁实现
* ConcurrentHashMap在JDK7中使用分段锁，在JDK8中使用CAS+synchronized；它也是高并发场景下的首选数据结构

### **19、能具体说下ConcurrentHashMap的实现吗？**

ConcurrentHashMap线程安全在JDK7版本是基于`分段锁`实现；在JDK8是基于`CAS+synchronized`实现

**JDK1.7分段锁**

从结构上说，1.7版本的ConcurrentHashMap采用分段锁机制，里面包含一个Segment数组，Segment继承ReentrantLock，Segement则包含HashEntry数组，HashEntry本身就是一个链表的结构，具有保存key，value的能力和能指向一个节点的指针

实际上就是相当于每个Segment都是一个HashMap，默认的Segement长度是16，也就是支持16线程的并发写，Segment之间相互不会受到影响

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption><p>JDK7分段锁</p></figcaption></figure>

**Put流程**

整个流程的HashMap非常相似，只不过是先定位到具体的Segement，然后通过ReentrantLock去操作而已，后面的流程，就和HashMap基本上是一样的

* 计算hash，定位segment，segment如果是空就先初始化
* 使用ReentrantLock加锁，如果获取锁失败则尝试自旋，自旋超过次数就阻塞获取，保证一定获取锁成功
* 遍历HashEntry，就是和HashMap一样，数组中key和hash一样就直接替换，不存在就再插入链表，链表同样操作

**Get流程**

* Get也很简单，key通过hash定位到segment，再遍历链表定位到具体的元素上，需要注意的是value是volatile的，所以Get是不需要加锁的

**JDK1.8 CAS+synchronized**

JDK1.8实现线程安全不是在数据结构上下功夫，它的数据结构和HashMap是一样的，数字+链表+红黑树；它实现线程安全的关键点在于Put流程

**Put流程**

*   首先计算hash，便利node数组，如果node是空的话，就通过CAS+自旋的方式初始化

    ```java
    tab = initTable();
    ```

    node数组初始化

    ```java
    private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
            //如果正在初始化或者扩容
            if ((sc = sizeCtl) < 0)
                //等待
                Thread.yield(); // lost initialization race; just spin
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {   //CAS操作
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
    ```
*   如果当前数组位置是空则直接通过CAS自旋写入数据

    ```java
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }
    ```
*   如果hash==MOVED，说明需要扩容，执行扩容

    ```java
    else if ((fh = f.hash) == MOVED)
        tab = helpTransfer(tab, f);
    ```

    ```java
    final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
        Node<K,V>[] nextTab; int sc;
        if (tab != null && (f instanceof ForwardingNode) &&
            (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
            int rs = resizeStamp(tab.length);
            while (nextTab == nextTable && table == tab &&
                   (sc = sizeCtl) < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                    transfer(tab, nextTab);
                    break;
                }
            }
            return nextTab;
        }
        return table;
    }
    ```
*   如果都不满足，就是用synchronized写入数据，写入数据同样判断链表、红黑树、链表写入和HashMap的方式一样，key的hash一样就覆盖，反之就尾插法，链表长度超过8就转换成红黑树

    ```java
    synchronized (f){
        ……
    }
    ```

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption><p>JDK8Put流程</p></figcaption></figure>

**Get流程**

Get很简单，和HashMap基本相同，通过key计算位置，table该位置key相同就返回，如果是红黑树按照红黑树获取，否则就链表遍历获取

### **20、HashMap内部节点是有序的吗？**

HashMap是无序的，根据hash值随即插入；如果想使用有序的Map，可以使用LinkedHashMap或者TreeMap

### **21、讲讲LinkedHashMap怎么实现有序的？**

LinkedHashMap维护了一个双向链表，有头尾节点，同时LinkedHashMap节点Entry内部除了继承HashMap的Node属性，还有before和after用于标识前置节点和后置节点

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>LinkedHashMap</p></figcaption></figure>

可以实现按插入的顺序(先得到的肯定是先插入的)或者访问顺序排序(在构造时带参数，按照访问次序排序)

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption><p>排序顺序</p></figcaption></figure>

### **22、讲讲TreeMap怎么实现有序的？**

TreeMap是按照Key的自然顺序或者Comprator的顺序进行排序，内部是通过红黑树来实现；所以要么key所属的类实现Comparable接口，或者自定义一个实现了Comparator接口的比较器，传给TreeMap用于key的比较

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>TreeMap</p></figcaption></figure>

### **23、讲讲HashTable？**

Hashtable的初始容量是11，加载因子是0.75，每次扩容的大小为大小的两倍加1，线程安全，每个方法都被synchronized关键字修饰，不能存放空值，判断相等的条件是hash相等同时equals相等

Hashtable是遗留类，很多映射的常用功能与HashMap类似，不同的是它继承自Dictionary类，并且是线程安全的，任一时间只有一个线程能写Hashtable，并发性不如ConcurrentHashMap，因为ConcurrentHashMap引入分段锁；Hashtable不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换

***

## Set <a href="#user-content-set" id="user-content-set"></a>

### **1、讲讲HashSet的底层实现？**

HashSet底层就是基于HashMap实现的（HashSet的源码非常少，因为除了clone()、writeObject()、readObject()是HashSet自己不得不实现之外，其他方法都是直接调用HashMap中的方法）

HashSet的add方法，直接调用HashMap的put方法，将添加的元素作为key，new一个Object作为value，直接调用HashMap的put方法，它会根据返回值是否为空判断是否插入元素成功

```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption><p>HashSet</p></figcaption></figure>

而在HashMap中的putVal方法中，进行了一系列判断，最后的结果是，只有key在table数组不存在的时候，才会返回插入的值

```java
if (e != null) { // existing mapping for key
    V oldValue = e.value;
    if (!onlyIfAbsent || oldValue == null)
        e.value = value;
    afterNodeAccess(e);
    return oldValue;
}
```

### **2、Set的特点？**

Set注重独一无二的性质，该体系集合用于存储无序(存入和取出的顺序不一定相同)元素，值不能重复；对象的相等性本质是对象hashCode值(Java是依据对象的内存地址计算出的此序号)判断的，如果想要让两个不同的对象视为相等的，就必须覆盖Object的hashCode方法和equals方法

### **3、TreeSet(二叉树)？**

* TreeSet()是使用二叉树的原理对新add()的对象按照指定的顺序排序(升序、降序)，没增加一个对象都会进行排序，将对象插入二叉树指定的位置
* Integer和String对象都可以进行默认的TreeSet的排序，而自定义类的对象是不可以的，自定义的类必须实现Comparable接口，并且覆写相应的compareTo()函数，才可以正常使用，或者自定义Comparator接口传入构造函数
* 在覆写compare函数时，要返回相应的值才能使TreeSet按照一定的规则来排序
* 比较此对象与指定对象的顺序；如果该对象小于、等于或大于指定对象，则分别返回负整数、零或正整数

### **4、聊聊LinkedHashSet(HashSet+LinkedHashMap)？**

对于LinkedHashSet而言，它继承与HashSet、又基于LinkedHashMap来实现的；LinkedHashSet底层使用LinkedHashMap来保存所有元素，它继承与HashSet，其所有的方法操作上又与HashSet相同，因此LinkedHashSet的实现上非常简单，只提供了四个构造方法，并通过传递一个标识参数，调用父类的构造器，底层构造一个LinkedHashMap来实现，在相关操作上与父类HashSet的操作相同，直接调用父类HashSet的方法即可

### **5、分析HashSet和TreeSet分别如何实现去重的？**

> HashSet如何实现去重？

HashSet的去重机制：hashCode()+equals()，底层先通过存入对象，进行位运算得到一个hash值，通过hash值得到对应的索引，如果发现table索引所在的位置没有数据，就直接存入；如果有数据，就进行equals()比较\[遍历比较，因为可能是一个链表]，如果比较后，不相同就加入，否则就不加入

> TreeSet如何实现去重？

TreeSet的去重机制：如果你传入了一个Comparator匿名对象，就是用实现的compare去重，如果方法返回0，就认为是相同的元素/数据，就不添加；如果没有传入一个Comparator匿名对象，则以你添加的对象实现的Comparable接口的compareTo方法去重；(两者都没有会报类型转换异常)
