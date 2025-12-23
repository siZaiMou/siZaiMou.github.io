---
layout: post
title: "Java 集合：八股学习记录和补充"
categories: [学习记录]
---

# 集合

# 常见集合

两大接口：Collection、Map

![](/assets/H55wbiFhzoniIZxKV1fcSTysnfh.png)

![](/assets/Q9N6bL7I0oOj2RxWPlIc3A9Cnwg.png)

* List是有序的Collection，使用此接口能够精确的控制每个元素的插入位置，用户能根据索引访问List中元素。常用的实现List的类有LinkedList，ArrayList，Vector，Stack。
  * ArrayList是容量可变的非线程安全列表，其底层使用数组实现，允许添加空值。当集合扩容时，会创建更大的数组，并把原数组复制到新数组。ArrayList支持对元素的快速随机访问，但插入与删除速度很慢。
  * LinkedList本质是一个双向链表，与ArrayList相比，其插入和删除速度更快，但随机访问速度更慢。
  * Vector 是 Java 早期提供的线程安全的动态数组，如果不需要线程安全，并不建议选择，毕竟同步是有额外开销的。Vector 内部是使用对象数组来保存数据，可以根据需要自动的增加容量，当数组已满时，会创建新的数组，并拷贝原有数组数据。
* Set不允许存在重复的元素，与List不同，set中的元素是无序的。常用的实现有HashSet，LinkedHashSet和TreeSet。
  * HashSet通过HashMap实现，HashMap的Key即HashSet存储的元素，所有Key都是用相同的Value，一个名为PRESENT的Object类型常量。使用Key保证元素唯一性，但不保证有序性。由于HashSet是HashMap实现的，因此线程不安全。
    * <span style="color: #E0E1E4">为什么value不能是null：add方法的底层是调用了HashMap的put方法，我们知道HashMap的put方法的返回值为null(失败)或者value(成功)。如果底层hashmap的value保存的是null的话，那么put成功或者失败都会返回null，那么HashSet的add（）方法每次返回的布尔值都是true了，也就不能够确认add成功还是失败了。</span>
  * LinkedHashSet继承自HashSet，通过LinkedHashMap实现，使用双向链表维护元素插入顺序，保证存储和读取的顺序一致。
  * TreeSet通过TreeMap实现，添加元素到集合时按照比较规则将其插入合适的位置，保证插入后的集合仍然有序。
* Map 是一个键值对集合，存储键、值和之间的映射。Key 无序，唯一；value 不要求有序，允许重复。Map 没有继承于 Collection 接口，从 Map 集合中检索元素时，只要给出键对象，就会返回对应的值对象。主要实现有TreeMap、HashMap、HashTable、LinkedHashMap、ConcurrentHashMap
  * HashMap：JDK1.8 之前 HashMap 由数组+链表组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突），JDK1.8 以后当链表长度大于阈值（默认为 8）且数组大小低于64（默认）时，将链表转化为红黑树，以减少搜索时间
  * LinkedHashMap：LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的entry可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。
  ![](/assets/D9Pxbnrl0oUxCixWuvycH7P0nLg.png)

  * HashTable：数组+链表组成的，数组是 HashTable 的主体，链表则是主要为了解决哈希冲突而存在的。线程安全。
  * TreeMap：红黑树（自平衡的排序二叉树）。它可以对键进行排序，默认按照自然顺序排序，也可以通过指定的比较器进行排序。也是kv对。
  ![](/assets/KtlkbCgJUoH73qxiGLNcihFEnAh.png)

  * ConcurrentHashMap：Node数组+链表+红黑树实现，线程安全的（jdk1.8以前Segment锁，1.8以后volatile + CAS 或者 synchronized）
* JUC(java.util.concurrent)
  * java.util.concurrent 包提供的都是线程安全的集合
  * 并发Map：ConcurrentHashMap、ConcurrentSkipListMap
    * ConcurrentHashMap是基于数组 ＋ 单向链表 ＋ [红黑树](https://zhida.zhihu.com/search?content_id=381763588&content_type=Answer&match_order=1&q=%E7%BA%A2%E9%BB%91%E6%A0%91&zhida_source=entity)的结构，ConcurrentHashMap只是对key做了hash找到存储的位置，插入、读取数据时复杂度是O(1).
    * ConcurrentSkipListMap 是SkipList（跳表）结构实现，基于不同的实现我们会发现，ConcurrentSkipListMap 的key的存储是有序的，插入、读取数据时复杂度是O（logn）
  * 并发Set：ConcurrentSkipListSet、CopyOnWriteArraySet
    * ConcurrentHashSet可由ConcurrentHashMap::keySet得到
  * 并发List：CopyOnWriteArrayList(当对象进行写操作时，使用了Lock锁做同步处理，内部拷贝了原数组，并在新数组上进行添加操作，最后将新数组替换掉旧数组；若进行的读操作，则直接返回结果，操作过程中不需要进行同步。)适用于读操作远多于写操作的场景，
  * 并发Queue：ConcurrentLinkedQueue(CAS实现)、BlockingQueue(读写阻塞等待)
  * Vector、HashTable也是线程安全的，它们的方法都加了锁

# 遍历

for、增强for(迭代器)、Iterator(遍历时删除元素)、ListIterator(双向访问)

forEach

![](/assets/DDJub7Sjkogi7Bxi045c0xntnmg.png)

Stream API

![](/assets/AkLNbztDDomiAJxzmIucRc30nUh.png)

# List

![](/assets/HaFDbDFmmojtktx7hticnkj5nXc.png)

## 边遍历边修改元素

* 最原始的for(i=0;i<len;i++)、直接用iterator可以边遍历边修改(增删改)，使用for(e:arr)、forEach会报错
* 使用迭代器/foreach：需要使用迭代器的set或remove方法，而不能使用List的相关方法，否则可能会抛出`ConcurrentModificationException`异常。原理：
  * 在AbstractList中，有一个全局变量modCount，记录了结构性改变的次数(结构性改变指的是那些修改了列表大小的操作，在迭代过程中可能会造成错误的结果)。此外还有一个expectedModCount变量，在iterator调用迭代器的方法时，会判断modCount是否和expectedModCount相等。
  * 在 ArrayList 中就继承 AbstractList 并在每个结构性改变的方法(如add、addAll、remove、clear)中让 madCount 变量自增 1。但不会改变expectedModCount。
  * modCount变量由迭代器（Iterator）和列表迭代器（ListIterator）使用，当进行next()、remove()、previous()、set()、add()等操作时，如果modCount的值意外改变，那么迭代器或者列表迭代器就会抛出此异常。
  * 在 next() 方法中，判断了当前的 modCount 跟初始化 Itr 时的 expectedModCount 是否相同，不同则说明列表数据进行了结构性改变，此时就会抛出 ConcurrentModificationException。
  * 通过迭代器修改集合时，会同步更新`expectedModCount`（使其等于当前`modCount`）。
* 线程安全的集合不会抛出异常，可能导致读到旧数据。并非所有集合都采用以上机制（如CopyOnWriteArrayList使用fail-safe策略，迭代时不会抛出该异常，但可能读取到旧数据）。

## List删除指定下标元素时间复杂度

* ArrayList：O(n)，删除某位置后需要移动其它元素。尾部元素是O(1)。
* LinkedList：O(n)，遍历后再删除。头尾节点是O(1)。
* CopyOnWriteArrayList：O(n)，在写操作时会创建一个新的数组，通常为 O (n)。但在并发环境下，它的删除操作不会影响读操作，具有较好的并发性能。

## ArrayList如何变为线程安全

1. 使用Collections类的synchronizedList方法将ArrayList包装成线程安全的List。
    1. synchronizedList会给list的所有方法加锁。锁对象是里面的object对象。

![](/assets/SYrrbSHnioS206xNtHpcq5s8nsb.png)

1. 使用用CopyOnWriteArrayList或Vector代替。

## ArrayList为什么线程不安全

### ArrayList add步骤

![](/assets/AQ0sbl5ZqorHDKxlRzrcaj2pn6d.png)

ensureCapacityInternal()这个方法的作用就是判断如果将当前的新元素加到列表后面，列表的elementData数组的大小是否满足，如果size + 1的这个需求长度大于了elementData这个数组的长度，那么就要对这个数组进行扩容(grow方法)。

![](/assets/QW3CbzQneoWrjdxjdQ5cAJG9n9b.png)

大体可以分为三步：

* 判断数组需不需要扩容，如果需要的话，调用grow方法进行扩容
* 将数组的size位置设置值（因为数组的下标是从0开始的）
* 将当前集合的大小加1
* 除此之外，还可以指定index进行add：
  * 指定位置插入 (add(int index, E element))：这需要移动元素，时间复杂度为 O(n)
  * 检查索引越界、检查容量是否要扩容
  * 使用 `System.arraycopy`将 `index`及其之后的元素向后移动一位，为新元素腾出位置。将新元素放入 `index`。将 `size`加 1。

### 删步骤

可以按照索引 (remove(int index))或值(remove(Object o))删除数据，计算需要移动的元素数量 (`numMoved = size - index - 1`)。如果数量大于 0，使用 `System.arraycopy`将 `index+1`之后的元素向前移动一位，覆盖被删除元素的位置。将数组末尾位置 (`--size`) 的元素置为 `null`​，​帮助垃圾回收​，防止内存泄漏。

### 可能出现的情况

* 部分值为null（并没有add null进去）：当线程1走到了扩容那里发现当前size是9，而数组容量是10，所以不用扩容，这时候cpu让出执行权，线程2也进来了，发现size是9，而数组容量是10，所以不用扩容，这时候线程1继续执行，将数组下标索引为9的位置set值了，还没有来得及执行size++，这时候线程2也来执行了，又把数组下标索引为9的位置set了一遍，这时候两个先后进行size++，导致下标索引10的地方就为null了。
* 索引越界：线程1走到扩容那里发现当前size是9，数组容量是10不用扩容，cpu让出执行权，线程2也发现不用扩容，这时候数组的容量就是10，而线程1 set完之后size++，这时候线程2再进来size就是10，数组的大小只有10，而你要设置下标索引为10的就会越界（数组的下标索引从0开始）；
* size与add的数量不符：这个基本上每次都会发生，这个理解起来也很简单，因为size++本身就不是原子操作，可以分为三步：获取size的值，将size的值加1，将新的size值覆盖掉原来的，线程1和线程2拿到一样的size值加完了同时覆盖，就会导致一次没有加上，所以肯定不会与我们add的数量保持一致的；

## ArrayList扩容步骤

默认初次构造大小为0，第一次add后扩容到10

ArrayList在添加元素时，如果当前元素个数已经达到了内部数组的容量上限，就会触发扩容操作。ArrayList的扩容操作主要包括以下几个步骤：

1. 计算新的容量：一般情况下，新的容量会扩大为原容量的1.5倍（在JDK 10之后，扩容策略做了调整），然后检查是否超过了最大容量限制。
    1. 如果二倍扩容的话，那释放的内存连接起来的大小（如果拼接的话），都小于即将要分配的内存大小。浪费空间，而1.5倍的话，可以重用之前的释放的内存
    ![](/assets/WJT4b6CwLooGWAxX4x7cj9Isnpf.png)

    3. 此外1.5 可以充分利用移位操作，减少浮点数或者运算时间和运算次数。
2. 创建新的数组：根据计算得到的新容量，创建一个新的更大的数组。
3. 将元素复制：将原来数组中的元素逐个复制到新数组中。
4. 更新引用：将ArrayList内部指向原数组的引用指向新数组。
5. 完成扩容：扩容完成后，可以继续添加新元素。

## CopyOnWriteArrayList如何确保安全

CopyOnWriteArrayList底层也是通过一个数组保存数据，使用volatile关键字修饰数组，保证当前线程对数组对象重新赋值后，其他线程可以及时感知到。

假设只加锁保护修改操作，而不复制数组（即直接修改原数组），会导致读操作可能读取到 “中间状态” 的数据。

CopyOnWriteArrayList 的设计初衷是**优化读多写少场景的性能**。如果读操作需要加锁（或因修改操作锁而阻塞），会丧失并发读的性能优势。👇

![](/assets/LrbBb5OrUolCEqxZdhvcFOixnOh.png)

通过 “写时复制”，读操作可以直接访问当前数组（旧数组），无需加锁或阻塞，即使此时有写操作在进行（写操作在新数组上，add、remove、set）。这种 “读写分离” 使得读操作的性能接近单线程访问，这是仅靠锁无法实现的（锁会导致读操作等待写操作释放锁）。

CopyOnWriteArrayList允许在迭代期间进行修改，而不会抛出ConcurrentModificationException异常。

迭代器在创建时会捕获一个数组的快照，因此迭代过程中对列表的修改不会影响迭代器的遍历。这种特性使得CopyOnWriteArrayList在需要频繁迭代且不希望出现并发修改异常的场景中非常有用。

使用ReentrantLock保证写的线程安全。

![](/assets/C3EwbXOXqoXY2OxzdjWc7DaOnqd.png)

# Map

![](/assets/X2WmbjXLHozGjzxqM26cnxHjnKg.png)

## 快速遍历

* for(Map.Entry<T,T>entry:map.entrySet()){entry.getKey()/entry.getValue()}
* for(T key:map.keySet())
* map.entrySet().iterator()
* map.forEach((k,v)->)
* map.entrySet().forEach(entry->xxx)
* 遍历hashmap，存在阻塞时parallelStream性能最高, 非阻塞时parallelStream性能最低。
* HashMap 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个。

## HashMap的Hash方法与冲突解决

* 在 JDK 1.7 版本之前， HashMap 数据结构是数组和链表，HashMap通过哈希算法将元素的键（Key）映射到数组中的槽位（Bucket）。如果多个键映射到同一个槽位，它们会以链表的形式存储在同一个槽位上，因为链表的查询时间是O(n)，所以冲突很严重，一个索引上的链表非常长，效率就很低了。

![](/assets/K8Ywb0RLboUKANxGWHccH87OnYg.png)

* JDK1.8 版本的时候做了优化，当一个链表的长度超过8且数组长度大于等于64的时候就转换数据结构，不再使用链表存储，而是使用红黑树，查找时使用红黑树，时间复杂度O（log n），可以提高查询性能，但是在数量较少时，当红黑树节点减少到 **≤ 6**，会将红黑树转换回链表。

![](/assets/Q7vZbiU2koRAqwxmSsucggrLnz1.png)

数组下标=hash值&(n-1) 

n为数组长度，hash值由以下扰动函数得到：

jdk1.8扰动函数，h为int值，长4字节，将h值右移16位后与自身进行异或，即自己的高半区和低半区做异或，是为了混合原始哈希码的高位和低位，来加大低位的随机性。而且混合后的低位掺杂了高位的部分特征，使高位的信息也被保留下来


```Java
    static final int hash(Object key) {
      int h;
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

&运算作用为取模

* 冲突解决：链表法、开放寻址法(线性探测、二次探测)、再哈希法(冲突时使用另一个hash函数进行计算)、哈希桶扩容

## HashMap put

### jdk1.8+

![](/assets/VNszbssbWoFkmYxyLl0cypKOnrf.png)

1. 第一步：根据要添加的键的哈希码计算在数组中的位置（索引）。
2. 第二步：检查该位置是否为空（即没有键值对存在）
    1. 如果为空，则直接在该位置创建一个新的Entry对象来存储键值对。将要添加的键值对作为该Entry的键和值，并保存在数组的对应位置。将HashMap的修改次数（modCount）加1，以便在进行迭代时发现并发修改。
3. 第三步：如果该位置已经存在其他键值对，检查该位置的第一个键值对的哈希码和键是否与要添加的键值对相同。
    1. 如果相同，则表示找到了相同的键，直接将新的值替换旧的值，完成更新操作。
4. 第四步：如果第一个键值对的哈希码和键不相同，则需要遍历链表或红黑树来查找是否有相同的键：
    1. 如果键值对集合是链表结构，从链表的头部开始逐个比较键的哈希码和equals()方法，直到找到相同的键或达到链表末尾。
        1. 如果找到了相同的键，则使用新的值取代旧的值，即更新键对应的值。
        2. 如果没有找到相同的键，则将新的键值对添加到链表的尾部。
    2. 如果键值对集合是红黑树结构，在红黑树中使用哈希码和equals()方法进行查找。根据键的哈希码，定位到红黑树中的某个节点，然后逐个比较键，直到找到相同的键或达到红黑树末尾。
        1. 如果找到了相同的键，则使用新的值取代旧的值，即更新键对应的值。
        2. 如果没有找到相同的键，则将新的键值对添加到红黑树中。
5. 第五步：检查链表长度是否达到阈值（默认为8）：
    1. 如果链表长度超过阈值，且HashMap的数组长度大于等于64，则会将链表转换为红黑树，以提高查询效率。如果数组长度不大于64，则扩容。
6. 第六步：检查负载因子是否超过阈值（默认为0.75）：
    1. 如果键值对的数量（size）与数组的长度的比值大于阈值，则需要进行扩容操作。
7. 扩容：
    1. 创建一个新的两倍大小的数组。
    2. 将旧数组中的键值对重新计算哈希码并分配到新数组中的位置。
    3. 更新HashMap的数组引用和阈值参数。
8. 完成

### jdk 1.7

直接遍历链表：

1. 如果定位到的数组位置没有元素 就直接插入。
2. 如果定位到的数组位置有元素，遍历以这个元素为头结点的链表，依次和插入的 key 比较，如果 key 相同就直接覆盖，不同就采用头插法插入元素。

## HashMap扩容

resize方法

hashMap默认的负载因子是0.75，初始容量是16，即如果hashmap中的元素个数超过了总容量75%，则会触发扩容，扩容分为两个步骤：

1. 对哈希表长度的扩展（2倍）
2. 将旧哈希表中的数据放到新的哈希表中。

因为我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置，比如8->16，新位置就是原位置+8。对应到二进制里即加1000。所以新位置不需要重新计算hash，直接原位置+oldCap即可。

举例说明：

至于旧有值移动到新的节点的时候存放于哪个节点，Java 是根据 (e.hash & oldCap) == 0 来判断的：

① 等于0时，则将该节点放到新数组时的索引位置等于其在旧数组时的索引位置,记为低位区链表lo开头-low;

② 不等于0时,则将该节点放到新数组时的索引位置等于其在旧数组时的索引位置再加上旧数组长度，记为高位区链表hi开头high.

在 HashMap 扩容时，节点会根据哈希值重新分配到新表中。首先遍历旧表中的每个 bucket（槽位）：如果某个 bucket 中只有一个节点，则直接计算其在新表中的位置并放入；

如果 bucket 中是红黑树结构，调用 TreeNode 的 split 方法将其分成两部分，一部分保留在原来的索引位置，另一部分移动到新表中的新索引位置（原索引 + oldCap）；


```Java
1. 遍历原红黑树，按 <mark style="background-color: #BBBFC4">hash & bit</mark> 拆分出 <mark style="background-color: #BBBFC4">lo</mark>（原索引）和 <mark style="background-color: #BBBFC4">hi</mark>（新索引）两个链表
2. 断开节点的 <mark style="background-color: #BBBFC4">next</mark> 引用
3. 判断拆分后的节点数是否 ≤ 6（<mark style="background-color: #BBBFC4">UNTREEIFY_THRESHOLD</mark>）
4. 若节点数 > 6，则将链表重新转为红黑树
5. 将拆分后的结构（链表 / 红黑树）挂载到新数组的对应索引（<mark style="background-color: #BBBFC4">index</mark>/<mark style="background-color: #BBBFC4">index+bit</mark>）
```

如果 bucket 中是链表结构，则遍历链表中的每个节点，根据哈希值决定它的新位置，如果节点的新位置与旧索引相同（e.hash & oldCap == 0），则将其保留在原索引处；如果节点的新位置是原索引 + oldCap，则将其移至新表中相应的新索引位置，最终将链表的这两部分分别放入新表中的对应位置。

![](/assets/ZHu8bBMtcozHnHxb5bTcHfNynyi.png)

## HashCode和equal方法

HashMap使用Key对象的hashCode()和equals方法去决定key-value对的索引。

HashMap在比较元素时，会先通过hashCode进行比较（比较下标是否相等，也就是是否冲突），相同的情况下再通过equals进行比较。

所以 equals相等的两个对象，hashCode一定相等。hashCode相等的两个对象，equals不一定相等（比如散列冲突的情况）。

所以正确实现它们非常重要，equals()和hashCode()的实现应该遵循以下规则：

如果o1.equals(o2)，那么o1.hashCode() == o2.hashCode()总是为true的。

如果o1.hashCode() == o2.hashCode()，并不意味着o1.equals(o2)会为true。

重写了equals方法，不重写hashCode方法时，可能会出现equals方法返回为true，而hashCode方法却返回false，这样的一个后果会导致在hashmap等类中存储多个一模一样的对象，导致出现覆盖存储的数据的问题，这与hashmap只能有唯一的key的规范不符合。

## HashMap的线程安全问题

1. JDK1.7中的 HashMap 使用头插法插入元素，在多线程的环境下，扩容的时候有可能导致环形链表的出现，形成死循环。因此，JDK1.8使用尾插法插入元素，在扩容时会保持链表元素原本的顺序，不会出现环形链表的问题。

![](/assets/SdJYb9at3oOxcSxZIgDchAW0nJe.jpeg)

![](/assets/MWMyb9y7vo02VJx65lBcrnCOnRg.jpeg)



1. 多线程同时执行 put 操作，如果计算出来的索引位置是相同的，那会造成前一个 key 被后一个 key 覆盖，从而导致元素的丢失。此问题在JDK 1.7和 JDK 1.8 中都存在。
2. 并发 <mark style="background-color: #BBBFC4">`put`</mark> 与 <mark style="background-color: #BBBFC4">`get`</mark> 导致 <mark style="background-color: #BBBFC4">`get`</mark> 到 null 或旧值
    * <mark style="background-color: #BBBFC4">`**get**`</mark>**到 null**：线程 A 正在插入一个键值对（已计算哈希并定位桶，但尚未完成节点赋值），此时线程 B 对同一键执行 <mark style="background-color: #BBBFC4">`get`</mark>，会发现桶中无该节点，返回 null（但实际线程 A 后续会完成插入）。
    * <mark style="background-color: #BBBFC4">`**get**`</mark>**到旧值**：线程 A 正在更新一个键的 value，线程 B 同时读取该键，可能读到更新前的旧值（因 value 未加 volatile 修饰，无法保证内存可见性）。
3. HashMap 的 <mark style="background-color: #BBBFC4">`size`</mark> 字段记录元素总数，多线程并发 <mark style="background-color: #BBBFC4">`put`</mark>/<mark style="background-color: #BBBFC4">`remove`</mark> 时，对 <mark style="background-color: #BBBFC4">`size`</mark> 的修改是**非原子操作**（如 <mark style="background-color: #BBBFC4">`size++`</mark> 实际是读取 - 修改 - 写入三步）：
    * 线程 A 读取 <mark style="background-color: #BBBFC4">`size=10`</mark>，准备修改为 11；
    * 线程 B 同时读取 <mark style="background-color: #BBBFC4">`size=10`</mark>，准备修改为 11；
    * 最终 <mark style="background-color: #BBBFC4">`size`</mark> 可能变为 11（正确应为 12），导致 size 与实际元素数量不一致。
4. 扩容相关
    * **数据丢失**：线程间扩容 / 插入操作相互覆盖，导致节点丢失；
      * 线程 A：执行扩容，遍历桶<mark style="background-color: #BBBFC4">`j=1`</mark>的链表（节点 A1→A2），拆分出<mark style="background-color: #BBBFC4">`loHead=A1、hiHead=A2`</mark>，但未挂载到新数组；
      * 线程 B：抢占 CPU，完成桶<mark style="background-color: #BBBFC4">`j=1`</mark>的扩容，将<mark style="background-color: #BBBFC4">`A1`</mark>挂载到新数组索引 1，<mark style="background-color: #BBBFC4">`A2`</mark>挂载到索引 5；
      * 线程 A 恢复执行，直接将自己拆分的<mark style="background-color: #BBBFC4">`loHead=A1`</mark>挂载到索引 1、<mark style="background-color: #BBBFC4">`hiHead=A2`</mark>挂载到索引 5（覆盖线程 B 的挂载结果），且线程 B 在扩容中插入的新节点<mark style="background-color: #BBBFC4">`A3`</mark>被完全覆盖。
    * **数据不一致**：读线程读取到扩容的中间状态（如空值、不完整链表）；
    * **引用可见性问题**：<mark style="background-color: #BBBFC4">`table`</mark> 引用替换非原子，线程可能读取到旧数组或空数组。
      * 线程 A：执行扩容，创建新数组<mark style="background-color: #BBBFC4">`newTab`</mark>，执行<mark style="background-color: #BBBFC4">`table = newTab`</mark>（仅替换引用，未迁移任何节点）；
      * 线程 B：执行<mark style="background-color: #BBBFC4">`get`</mark>操作，读取<mark style="background-color: #BBBFC4">`table`</mark>拿到空的新数组，所有查询返回<mark style="background-color: #BBBFC4">`null`</mark>；
      * 线程 A 后续完成所有节点迁移，但因<mark style="background-color: #BBBFC4">`table`</mark>无<mark style="background-color: #BBBFC4">`volatile`</mark>修饰，线程 B 的 CPU 缓存未刷新，始终读取旧的<mark style="background-color: #BBBFC4">`table`</mark>引用（旧数组已被置空）。

# ConcurrentHashMap

## 实现原理

### jdk1.7

在 JDK 1.7 中它使用的是数组加链表的形式实现的，而数组又分为：大数组 Segment 和小数组 HashEntry。 Segment 是一种可重入锁（ReentrantLock），在 ConcurrentHashMap 里扮演锁的角色；HashEntry 则用于存储键值对数据。一个 ConcurrentHashMap 里包含一个 Segment 数组，一个 Segment 里包含一个 HashEntry 数组，每个 HashEntry 是一个链表结构的元素。

JDK 1.7 ConcurrentHashMap 分段锁技术将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问，能够实现真正的并发访问。

![](/assets/GOHjbgG8OoX7s0xp1GNcXuLonMc.png)

### jdk1.8

使用了Node数组 + 链表/红黑树的方式优化了 ConcurrentHashMap 的实现

![](/assets/VoMabaMoSoZGK1xjQPjc6C9lnsc.png)

jdk 1.8 ConcurrentHashMap 主要通过 volatile + CAS 或者 synchronized 来实现的线程安全的。添加元素时首先会判断容器是否为空：

* 如果为空则使用  volatile  加  CAS  来初始化
* 如果容器不为空，则根据存储的元素计算该位置是否为空。
  * 如果根据存储的元素计算结果为空，则利用  CAS  设置该节点；
  * 如果根据存储的元素计算结果不为空，则使用 synchronized  ，然后，遍历桶中的数据，并替换或新增节点到桶中，最后再判断是否需要转为红黑树，这样就能保证并发访问时的线程安全了。

如果把上面的执行用一句话归纳的话，就相当于是ConcurrentHashMap通过对头结点加锁来保证线程安全的，锁的粒度相比 Segment 来说更小了，发生冲突和加锁的频率降低了，并发操作的性能就提高了。

## 详解

### 初始化

putVal之前会调用initTable()方法

成员变量 sizeCtl （sizeControl 的缩写），它的值决定着当前的初始化状态。

* -1 说明正在初始化，其他线程需要自旋等待
* -N 说明 table 正在进行扩容，高 16 位表示扩容的标识戳，低 16 位减 1 为正在进行扩容的线程数
* 0 默认值，代表 ConcurrentHashMap 未初始化，第一次 put 时触发初始化。
* 大于0 表示 table 扩容的阈值，如果 table 已经初始化。


```Java
/**
 * 初始化表格，使用 sizeCtl 中记录的大小。
 */
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; 
    int sc;
    while ((tab = table) == null || tab.length == 0) {
        // 如果 sizeCtl < 0，说明其他线程已经成功执行了 CAS 操作，正在进行初始化。
        if ((sc = sizeCtl) < 0)
            // 让出 CPU 使用权
            Thread.yield(); // 初始化竞争失败；只是循环等待
        
        //通过 CAS 操作，确保只有一个线程可以将 sizeCtl 的值从 sc（通常为一个非负整数）变为 -1，表示该线程已经获得了初始化的权限。
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    //初始化后sizeCtl应该大于0，代表扩容阈值，n-(n>>>2)=n-n/4=0.75*n
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

### put

根据key计算出table位置后：

1. 如果目标位置i为空：尝试使用 casTabAt(`casTabAt(tab, i, null, new Node<K,V>(hash, key, value))`实现方法为`U.compareAndSetReference(tab, ((long)i << ASHIFT) + ABASE, c, v);`) CAS性地插入一个新节点。如果插入成功 (casTabAt 返回 true)，则跳出循环。**期盼值**：<mark style="background-color: #BBBFC4">`null`</mark>（表示当前桶未被其他线程初始化）
2. 如果此位置正在MOVED(扩容)：则调用helpTransfer方法协助扩容。helpTransfer 方法通过检测当前表的扩容状态，如果发现扩容进程正在进行（即 ForwardingNode 和 nextTable 存在），它会尝试增加扩容计数（sizeCtl），并在成功时触发数据迁移（transfer 方法）。
3. 如果插入时指定onlyIfAbsent为true：键已经存在，则不进行插入操作，直接返回已存在的值。
4. 常规情况：synchronized(f)，对table的当前位置加锁+hashmap的put流程

* CAS/Synchronized时机
  * 如果计算出来的hash槽没有存放元素，那么就可以直接使用CAS来进行设置值，这是因为在设置元素的时候，因为hash值经过了各种扰动后，造成hash碰撞的几率较低，那么我们可以预测使用较少的自旋来完成具体的hash落槽操作。(情况1)
  * 当发生了hash碰撞的时候说明容量不够用了或者已经有大量线程访问了，因此这时候使用synchronized来处理hash碰撞比CAS效率要高，因为发生了hash碰撞大概率来说是线程竞争比较强烈。(情况2)



### 扩容

#### 时机

* putVal后会调用addCount方法增加集合元素计数，如果当前集合元素个数到达扩容阈值时就会触发扩容。
* 扩容状态下其他线程对集合进行插入、修改、删除、合并、compute 等操作时遇到 ForwardingNode 节点会触发(协助)扩容 。
* putAll 批量插入或者插入节点后发现存在链表长度达到 8 个或以上，但数组长度为 64 以下时会优先扩容而非树化 。

#### 过程

1. **初始化新数组**
    * 创建一个新的数组，容量是旧数组的两倍（即 `newCapacity = oldCapacity << 1`）。
2. **分配迁移任务**
    * 将旧数组分成多个区间（默认是 16 个），每个线程负责迁移一个区间的数据。
    * 每个线程从数组的末尾开始向前迁移数据，直到所有数据迁移完成。
3. **数据迁移**
    * 对于每个桶（`bucket`），线程会锁定当前桶的头节点（通过 `synchronized` 锁），然后将数据从旧数组迁移到新数组。迁移逻辑与HashMap类似，也会将链表拆分为高低位两组。
      * 如果是链表，直接将链表拆分到新数组的对应位置。
      * 如果是红黑树，会检查树的节点数量，如果节点数量小于等于 `6`，则将树退化为链表。
    * 迁移完成后，将旧数组中的桶标记为已迁移（通常设置为一个特殊的节点，如 `ForwardingNode`，它起到标记和路由的作用：标记该桶已迁移；路由读操作到新数组查询）。
    * 其他线程在进行`put`操作时若发现目标桶是 `ForwardingNode`，则不会阻塞，而是协助进行迁移​（`helpTransfer`方法），加速整个过程。
4. **更新引用**
    * 当所有数据迁移完成后，`ConcurrentHashMap` 会将内部的数组引用指向新的数组。并调整 `sizeCtl`为新的扩容阈值。

# Set

HashSet、LinkedHashSet 和 TreeSet 的主要区别在于底层数据结构不同。HashSet 的底层数据结构是哈希表（基于 HashMap 实现）。LinkedHashSet 的底层数据结构是链表和哈希表，元素的插入和取出顺序满足 FIFO。TreeSet 底层数据结构是红黑树，元素是有序的，排序的方式有自然排序和定制排序。

HashSet 底层就是基于 HashMap 实现的，可以复用其一切逻辑。（HashSet 的源码非常非常少，因为除了 clone()、writeObject()、readObject()是 HashSet 自己不得不实现之外，其他方法都是直接调用 HashMap 中的方法。

# Queue

## **ArrayBlockingQueue**

JUC，基于数组实现，具有固定的容量，并支持阻塞的插入和移除操作。

### 基本特性

* **有界队列**：`ArrayBlockingQueue` 是一个有界队列，容量在创建时固定，无法动态调整。
* **线程安全**：所有操作都是线程安全的，内部通过锁机制（`ReentrantLock`）实现。
* **阻塞操作**：
  * 当队列满时，插入操作会被阻塞，直到队列有空闲空间。
  * 当队列为空时，移除操作会被阻塞，直到队列中有新元素。
* **公平性**：支持公平锁和非公平锁（默认是非公平锁），公平锁可以保证等待时间最长的线程优先获取锁。

### 实现原理

* **环形数组**：`ArrayBlockingQueue` 使用一个环形数组存储元素，通过两个指针（`putIndex` 和 `takeIndex`）分别指向插入和移除的位置。
* **锁机制**：所有操作都需要获取锁（`ReentrantLock`），确保线程安全。
* **条件变量**：
  * `notEmpty`：用于阻塞消费者线程，当队列为空时，消费者线程会等待。
  * `notFull`：用于阻塞生产者线程，当队列满时，生产者线程会等待。
* **条件变量**：
  * 当队列满时，生产者线程会调用 `notFull.await()` 进入等待状态。
  * 当队列空时，消费者线程会调用 `notEmpty.await()` 进入等待状态。
  * 当插入或移除操作完成后，会通过 `notEmpty.signal()` 或 `notFull.signal()` 唤醒等待的线程。

## PriorityQueue

 [praɪˈɒrəti]

Java 集合框架中的一个基于优先级堆的无界优先级队列。它允许元素按照自然顺序（通过 `Comparable` 接口）或自定义顺序（通过 `Comparator` 接口）进行排序，队头的元素总是优先级最高的元素。

### 基本特性

* **无界队列**：`PriorityQueue` 是一个无界队列，容量可以动态增长。
* **优先级排序**：元素按照优先级排序，队头元素总是优先级最高的元素（最小堆或最大堆）。
* **非线程安全**：`PriorityQueue` 不是线程安全的，如果需要线程安全的优先级队列，可以使用 `PriorityBlockingQueue`。
* **不允许 null 元素**：`PriorityQueue` 不允许插入 `null` 元素。
* **基于堆实现**：`PriorityQueue` 内部使用二叉堆（通常是最小堆）实现。

### **实现原理**

* **二叉堆**：`PriorityQueue` 内部使用二叉堆（通常是最小堆）实现。堆是一种完全二叉树，满足以下性质：
  * 父节点的值总是小于或等于子节点的值（最小堆）。
  * 父节点的值总是大于或等于子节点的值（最大堆）。
* **插入操作**：
  * 将新元素插入数组末尾。
  * 从新元素开始向上调整堆结构（`siftUp`），直到满足堆的性质。
* **移除操作**：
  * 移除队头元素（数组的第一个元素）。
  * 将数组末尾元素移动到队头。
  * 从队头开始向下调整堆结构（`siftDown`），直到满足堆的性质。

## DelayQueue

`DelayQueue` 是 Java 并发包 (`java.util.concurrent`) 中的一个无界阻塞队列，用于存放实现了 `Delayed` 接口的元素。只有当元素的延迟时间到期时，才能从队列中取出该元素。

### **基本特性**

* **无界队列**：`DelayQueue` 是一个无界队列，容量可以动态增长。
* **延迟时间**：元素必须实现 `Delayed` 接口，定义延迟时间。
* **阻塞操作**：
  * 当队列为空时，获取元素的操作会被阻塞。
  * 当队列中没有到期的元素时，获取元素的操作会被阻塞，直到有元素到期。
* **线程安全**：`DelayQueue` 是线程安全的，内部通过锁机制实现。

### **实现原理**

* **优先级队列**：`DelayQueue` 内部使用 `PriorityQueue` 存储元素，按照延迟时间排序。
* **延迟时间**：元素必须实现 `Delayed` 接口，定义 `getDelay(TimeUnit unit)` 方法，返回剩余的延迟时间。
* **阻塞机制**：
  * 当队列为空时，获取元素的操作会被阻塞。
  * 当队列中没有到期的元素时，获取元素的操作会被阻塞，直到有元素到期。
