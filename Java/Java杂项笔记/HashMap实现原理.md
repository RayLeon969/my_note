首先写一点我看过的比较早的关于HashMap的理解的博客作为切入点，这篇文章虽然写的比较早，HashMap的底层代码可能和现在的不一样，但是比较容易理解底层发生了什么

# Java中的hashCode和equals

1. 关于hashCode

- hashCode的存在主要是用于查找的快性，如Hashtable，HashMap等，**hashCode是用来在散列存储结构中确定对象的存储地址的**。
- 如果两个对象相同，就是适用于equals（java.lang.Object）方法，那么这两个对象的hashCode一定要相同
- 如果对象的equals方法被重写，那么对象的hashCode也尽量重写，并且产生hashCode使用的对象，一定要和equals方法中使用的一致，否则就会违反上面提到的第二点。
- 两个对象的hashCode相同并不表示两个对象一定相同，也就是不一定适用于equals方法，只能够说明这两个对象在散列存储结构中，如Hashtable，他们“存放在同一个篮子里”。

**hashCode适用于查找的，而equals是用来比较两个对象是否相等**。

摘自别人对hashCode的解读：

```
1.hashcode是用来查找的，如果你学过数据结构就应该知道，在查找和排序这一章有
例如内存中有这样的位置
0  1  2  3  4  5  6  7
而我有个类，这个类有个字段叫ID,我要把这个类存放在以上8个位置之一，如果不用hashcode而任意存放，那么当查找时就需要到这八个位置里挨个去找，或者用二分法一类的算法。
但如果用hashcode那就会使效率提高很多。
我们这个类中有个字段叫ID,那么我们就定义我们的hashcode为ID％8，然后把我们的类存放在取得得余数那个位置。比如我们的ID为9，9除8的余数为1，那么我们就把该类存在1这个位置，如果ID是13，求得的余数是5，那么我们就把该类放在5这个位置。这样，以后在查找该类时就可以通过ID除 8求余数直接找到存放的位置了。
2.但是如果两个类有相同的hashcode怎么办那（我们假设上面的类的ID不是唯一的），例如9除以8和17除以8的余数都是1，那么这是不是合法的，回答是：可以这样。那么如何判断呢？在这个时候就需要定义 equals了。
也就是说，我们先通过 hashcode来判断两个类是否存放某个桶里，但这个桶里可能有很多类，那么我们就需要再通过 equals 来在这个桶里找到我们要的类。
那么。重写了equals()，为什么还要重写hashCode()呢？
想想，你要在一个桶里找东西，你必须先要找到这个桶啊，你不通过重写hashcode()来找到桶，光重写equals()有什么用啊
```

2. 关于equals

- equals和==
  - ==用于比较引用和比较基本数据类型时具有不同的功能：
    - 比较基本数据类型时，如果两个值相同，则结果为true
    - 比较引用数据类型时，如果引用指向内存中的同一对象，结果为true
  - equals作为方法，实现对象的比较。由于==运算符不允许我们进行覆盖，也就是说它限制了我表达，因此我们重写equals方法，达到比较对象内容是否相同的目的。
- object类的equals方法的比较规则是：**如果两个对象的类型一致，并且内容一致，就返回true**。



# HashMap的实现原理



## HashMap的概述

HashMap是基于哈希表的Map接口的非同步实现，此实现提供所有可选的映射操作，并允许使用null值和null键。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。

在Java中，最基本的结构就是两种，一个是数组，另外一个是模拟指针（引用），所有的数据结构都可以用这两个基本结构来构造，**HashMap的底层实际上是一个数组，而数组的每一项是一个链表，所以HashMap底层实际上是一个数组+链表的数据结构。**

![image-20231116152756541](assets\image-20231116152756541.png)

每当new一个HashMap的时候，就会初始化一个Entry数组

```java
/**
* The table, resized as necessary. Length MUST Always be a power of two.
*/transient Entry[] table;

static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    final int hash;
    ……
}
```

Map.Entry其实就是一个key-value对，它持有指向下一个元素的引用，这就构成了一个链表。



## HashMap实现存储和读取



### 存储put方法

​     源码：

```java
public V put(K key, V value) {
     // HashMap允许存放null键和null值。
     // 当key为null时，调用putForNullKey方法，将value放置在数组第一个位置。 4     if (key == null)
         return putForNullKey(value);
     // 根据key的keyCode重新计算hash值。 7     int hash = hash(key.hashCode());
     // 搜索指定hash值在对应table中的索引。 9     int i = indexFor(hash, table.length);
     // 如果 i 索引处的 Entry 不为 null，通过循环不断遍历 e 元素的下一个元素。11     for (Entry<K,V> e = table[i]; e != null; e = e.next) {
         Object k;
         if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
             // 如果发现已有该键值，则存储新的值，并返回原始值15             V oldValue = e.value;
             e.value = value;
             e.recordAccess(this);
             return oldValue;
         }
     }
     // 如果i索引处的Entry为null，表明此处还没有Entry。22     modCount++;
     // 将key、value添加到i索引处。24     addEntry(hash, key, value, i);
     return null;
 }
```

其中的hash（int h）方法：

```java
static int hash(int h) {
     h ^= (h >>> 20) ^ (h >>> 12);
     return h ^ (h >>> 7) ^ (h >>> 4);
 }
```

**根据上面的源码可以看出：当程序试图将一个key-value键值对放入hashmap中时，程序首先根据该key的hashCode返回值决定改Entry的存储位置，如果两个Entry的key的hashCode（）返回值相同，则他们的存储位置相同。如果这两个Entry的key通过equals方法比较返回true，就将新的Entry的value覆盖原来Entry中的value，注意：key不会被覆盖。如果这两个Entry的key通过equals比较返回false，新添加的Entry将于集合中原有Entry形成Entry链，而且新添加的Entry位于Entry链的头部。**

**通过这种方式可以高效解决HashMap的冲突问题**



### 读取get方法

​	源码：

```java
 public V get(Object key) {
     if (key == null)
         return getForNullKey();
     int hash = hash(key.hashCode());
     for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
         Object k;
         if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
             return e.value;
     }
     return null;
 }
```

从HashMap中get元素时，首先计算key的hashcode，找到数组中对应位置的某一元素，然后通过key的equals方法在对应链表中寻找需要的元素。



## HashMap的resize

  当hashmap中的元素越来越多的时候，碰撞的几率也就越来越高（因为数组的长度是固定的），所以为了提高查询的效率，就要对hashmap的数组进行扩容，数组扩容这个操作也会出现在ArrayList中，所以这是一个通用的操作，很多人对它的性能表示过怀疑，不过想想我们的“均摊”原理，就释然了，而在hashmap数组扩容之后，最消耗性能的点就出现了：原数组中的数据必须重新计算其在新数组中的位置，并放进去，这就是resize。

​    那么hashmap什么时候进行扩容呢？当hashmap中的元素个数超过数组大小*loadFactor时，就会进行数组扩容，loadFactor的默认值为0.75，也就是说，默认情况下，数组大小为16，那么当hashmap中元素个数超过16*0.75=12的时候，就把数组的大小扩展为2*16=32，即扩大一倍，然后重新计算每个元素在数组中的位置，而这是一个非常消耗性能的操作，所以如果我们已经预知hashmap中元素的个数，那么预设元素的个数能够有效的提高hashmap的性能。比如说，我们有1000个元素new HashMap(1000), 但是理论上来讲new HashMap(1024)更合适，不过上面annegu已经说过，即使是1000，hashmap也自动会将其设置为1024。 但是new HashMap(1024)还不是更合适的，因为0.75*1000 < 1000, 也就是说为了让0.75 * size > 1000, 我们必须这样new HashMap(2048)才最合适，既考虑了&的问题，也避免了resize的问题。

**小结：**

1. **利用key的hashCode重新hash计算出当前对象的元素在数组中的下标**
2. **存储时，如果出现当前hash值相同的key，此时有两种情况：**
   1. **如果key相同，则覆盖原始值**
   2. **如果key不同，则将当前的key-value放入链表中**
3. **获取时，直接找到hash值对应的下标，再进一步判断key是否相同，从而找到对应值**
4. **HashMap解决hash冲突的核心就是：使用了数组存储的方式，然后将冲突的key的对象放入链表中，一旦发现冲突就在链表中做进一步对比.**