[toc]

## HashMap学习笔记

### hashmap的存储结构

hashmap的存储结构是**数组+链表**，组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的，如果定位到的数组位置不含链表（当前entry的next指向null）,那么查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度为O(n)，首先遍历链表，存在即覆盖，否则新增；对于查找操作来讲，仍需遍历链表，然后通过key对象的equals方法逐一比对查找。所以，性能考虑，**HashMap中的链表出现越少，性能才会越好。**

### 创建一个空的hashmap

```java
//根据resize方法中的这两句
newCap = DEFAULT_INITIAL_CAPACITY;//16
newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
//创建了一个初始容量大小为16的hashmap

```



### put方法流程

![put方法流程](https://img-blog.csdnimg.cn/20181105181728652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvc2hpbWF4aWFvMQ==,size_16,color_FFFFFF,t_70)

总结：在我们new HashMap（）时其实还没有给map分配空间，分配空间是在第一次put时分配的，由于第一次put值，默认会分配16大小的数组空间完成map初始化，接下来是根据存储的值进行hash算法计算得到插入的数组索引。加下来去hash表即table中查询这个值是否存在，不存在直接插入一条key-value。如果存在再查询key是否存在，存在就直接覆盖之前插入的数据；不存在就需要在链表上添加一个节点了，java8中为了提高速度使用的是如果链表长度大于8后，就将链表转换换成红黑树方便之后的查询添加等操作，最后将值插入后会进行map整个长度的校验，如果map的长度达到了该扩容的大小后（默认数量达到了容量的0.75倍后即12就开始数组扩容），将容量扩大为原来的两倍。源码中的方法是`newThr = oldThr << 1; // double threshold`通过移位的方式直接扩大为两倍，比*2运行速度快点。

### hashmap底层原理

hashmap底层最基本的是机制：

* 数据结构为数组+链表，初始化数组长度为16
* 数据进来后，首先算hashcode值：
  * 对应hashcode值索引上为空直接存在数组上；
  * 对应hashcode值索引数据不为空则进行链表重头开始调用equals方法比较；(注：实际使用中需要比较两个对象是否相同并不是比较对象的存储位置，而是对比对象内属性一致即算equals。所以比较两个对象时，应该重写它的equals方法!!!)
    * 比较结果相同则替换对应值
    * 比较结构都不相同则添加在数组头部，

### hashmap的优化

hashmap应该尽量避免hashcode值相同（hash碰撞）；

1. 所以hashmap提供了加载因子（默认0.75），也就是说当数组容量达到数组的75%时，将数组扩容到原来的两倍。扩容时会将每个元素重新计算hashcode值，然后重新分配到新的数组中。

* 为什么不等到100%?
  * 因为元素算出的hashcode值有可能会出现某个索引上为空而其他元素满了的情况，所以不能等到100%的时候再扩容。

2. java8之后数据结构改为数组+链表+红黑树；解决了某一条链表上过长导致进行多次equals方法的情况。

* 当某条链表长度大于8，并且hashmap中总节点大于64时。将链表转化成红黑树。（注：红黑树从小到大）。

3. java8 之后去除了永久区的设计，用MetaSpace元空间（物理内存）取代。

