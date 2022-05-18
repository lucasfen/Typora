# Java集合框架

![image-20220508104345462](C:\Users\fqh0722\AppData\Roaming\Typora\typora-user-images\image-20220508104345462.png)

### List和数组的区别

- 数组是一种数据结构，初始化时固定大小；List是一个集合，大小不固定
- 数组中可以放对象和基本数据类型；集合中只能放对象

## HashMap

### 基本数据结构

JDK1.8之前，HashMap底层是数组 + 链表

JDK1.8，HashMap底层是数组 + 链表 + 红黑树

### **HashMap的重要成员变量**

HashMap默认初始容量：16

HashMap最大容量：2的30次方

HashMap默认加载因子：0.75 即如果当前HashMap已用容量占初始容量的四分之三，则HashMap会进行扩容。加载因子决定了扩容的阈值

HashMap链表转红黑树阈值：当链表长度大于8，数组大小大于64时，链表转化为红黑树，否则，即使链表长度大于8，优先扩容

### HashMap的put方法

1. 如果table没有初始化就进行初始化
2. 计算key的索引
3. 判断索引处有没有元素，没有则直接插入
4. 如果有元素，
   - 判断key,hash是否相同，如果相同，则覆盖
   - 判断是否是红黑树，如果是，则在红黑树中插入节点
   - 遍历链表，如果链表长度大于8，则转换为红黑树，在红黑树中插入，如果链表长度小于8，则尾插法插入，如果链表中存在相同的节点，则覆盖
5. 插入成功后，检查是否需要扩容

```java
 public V put(K key, V value) {
 2     // 对key的hashCode()做hash
 3     return putVal(hash(key), key, value, false, true);
 4 }
 5 
 6 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
 7                boolean evict) {
 8     Node<K,V>[] tab; Node<K,V> p; int n, i;
 9     // 步骤①：tab为空则创建
10     if ((tab = table) == null || (n = tab.length) == 0)
11         n = (tab = resize()).length;
12     // 步骤②：计算index，并对null做处理 
13     if ((p = tab[i = (n - 1) & hash]) == null) 
14         tab[i] = newNode(hash, key, value, null);
15     else {
16         Node<K,V> e; K k;
17         // 步骤③：节点key存在，直接覆盖value
18         if (p.hash == hash &&
19             ((k = p.key) == key || (key != null && key.equals(k))))
20             e = p;
21         // 步骤④：判断该链为红黑树
22         else if (p instanceof TreeNode)
23             e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
24         // 步骤⑤：该链为链表
25         else {
26             for (int binCount = 0; ; ++binCount) {
27                 if ((e = p.next) == null) {
28                     p.next = newNode(hash, key,value,null);
                        //链表长度大于8转换为红黑树进行处理
29                     if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
30                         treeifyBin(tab, hash);
31                     break;
32                 }
                    // key已经存在直接覆盖value
33                 if (e.hash == hash &&
34                     ((k = e.key) == key || (key != null && key.equals(k)))) 
35							break;
36                 p = e;
37             }
38         }
39         
40         if (e != null) { // existing mapping for key
41             V oldValue = e.value;
42             if (!onlyIfAbsent || oldValue == null)
43                 e.value = value;
44             afterNodeAccess(e);
45             return oldValue;
46         }
47     }

48     ++modCount;
49     // 步骤⑥：超过最大容量 就扩容
50     if (++size > threshold) //非原子操作，有并发问题
51         resize();
52     afterNodeInsertion(evict);
53     return null;
54 }
```

### HashMap计算Key的索引

1. 将key的hashcode的高16位与hashcode进行异或，得到key的hash值
2. 将key的hash值通过寻址算法，得到key的索引,寻址算法：hash & (table.length - 1)

- 将key的hashcode的高16位与hashcode进行异或，可以使hashcode的高16位也参与到key的索引的计算中，减少碰撞的几率

### HashMap扩容

HashMap扩容是将原来hashmap的容量 * 2

JDK1.8之前扩容，转移元素时，需要重新计算key的索引

JDK1.8扩容时，只需要判断原来hash值的高位的数值。

- 如果元素原来hash值的高位数是0，则元素还是放在数组相同的位置上
- 如果元素原来的hash值的高位数是1，则元素放在原来的数组位置 + 原来数组容量的位置上

### HashMap的容量为什么是2的幂次方

- 根据寻址算法，将hashmap的容量设置为2的幂次方，使hash的低位都能参与计算，有利于使key的索引分布更均匀
- 便于使用位运算进行寻址算法，更高效

### Hash碰撞

1. 开放定址法：发生hash冲突时，以当前地址为基准，进行再寻址的方法。例如：发生hash冲突时，顺序查找下一个位置，直到找到一个空位置。
2. 再哈希法：发生hash冲突时，使用另一个Hash算法，对当前hash值进行再hash
3. 链地址法：使用链表保存冲突的key

### HashMap的并发问题

JDK1.8之前，HashMap在链表上插入新节点是头插法，在多线程扩容过程中，可能形成链表环

JDK1.8，HashMap在链表上插入新节点是尾插法，不会形成链表环，但hashmap在put方法中没有考虑多线程问题，还是会有并发问题，例如在插入节点后检测是否需要扩容，就有并发问题

## ConcurrentHashMap

#### ConcurrentHashMap解决并发问题

JDK1.8之前，ConcurrentHashMap基于ReetrantLock实现分段锁,将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。

- segment继承ReetrantLock充当锁的角色
- 每一个segment维护了若干个桶

![image-20220422175138723](C:\Users\fqh0722\AppData\Roaming\Typora\typora-user-images\image-20220422175138723.png)

JDK1.8,ConcurrentHashMap采用CAS和Synchronized保证线程安全。

synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升N倍

例如插入元素过程：

- 如果相应位置的Node还没有初始化，则调用CAS插入相应的数据；（插入null节点，无需加锁）

- 如果相应位置的Node不为空，且当前该节点不处于移动状态，则对该节点加synchronized锁，遍历链表更新节点或插入新 

  节点

![image-20220422181901035](C:\Users\fqh0722\AppData\Roaming\Typora\typora-user-images\image-20220422181901035.png)

#### ConcurrentHashMap的迭代

## CopyOnWriteArrayList

核心思想：读写分离，空间换时间，避免为保证并发安全导致的激烈的锁竞争。 

1、CopyOnWrite适用于读多写少的情况，最大程度的提高读的效率； 

2、CopyOnWrite是最终一致性，在写的过程中，原有的读的数据是不会发生更新的，只有新的读才能读到最新数据； 

3、如何使其他线程能够及时读到新的数据，需要使用volatile变量； 

4、写的时候不能并发写，需要对写操作进行加锁； 

```java
/*添加元素api */ 
public boolean add(E e) { 
    final ReentrantLock lock = this.lock; 
    lock.lock();
    try { 
        Object[] elements = getArray(); 
        int len = elements.length; 
        Object[] newElements = Arrays.copyOf(elements, len + 1); //复制 一个array副本 
        newElements[len] = e; //往副本里写入 
        setArray(newElements); //副本替换原本，成为新的原本 
        return true; 
        } finally { 
        	lock.unlock(); 
        } 
    } 
}
//读api 
public E get(int index) { 
   return get(getArray(), index); //无锁 
}
```

