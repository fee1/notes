# HashMap

## 1.基本概念

### get、put存取值，基于哈希表的Map接口非同步实现。允许存储null键值。不保证顺序不变。

## 2.实现原理

### 1.基本概念：底层是数组和链表的结合体，基于哈希(hash)算法实现。

### 2.put时先用key的hashCode第二次hash计算对象元素所在数组的下标，如果hash值相同，key不同会添加到链表中(key-value都放进去)，如果key相同会覆盖原来的值。

### 3.get时直接找到hash值对应的下标，遍历找到key值对应的相同的元素

## 3.在JDK1.7和1.8中有什么不同

### 1.resize扩容优化

### 2.引入了红黑树，目的是为了避免单条链表过长而影响效率问题。

- 扩展：红黑树算法

### 3.解决多线程死循环(闭环)问题(仍然是线程不安全的)，多线程可能会造成数据丢失。

- 扩展：死循环是怎么造成的，在多线程情况下扩容引起的，线程A运行到【Entry<K,V> next = e.next;】就被挂起了，此时线程B把扩容的全过程走完，线程A开始执行线程扩容流程，但是扩容的过程将已经扩容好的同一链表相邻元素相互引用，造成读取数据的死循环。

  void transfer(Entry[] newTable, boolean rehash) {
      int newCapacity = newTable.length;
      for (Entry<K,V> e : table) {
          while(null != e) {
              Entry<K,V> next = e.next;
              if (rehash) {
                  e.hash = null == e.key ? 0 : hash(e.key);
              }
              int i = indexFor(e.hash, newCapacity);
              e.next = newTable[i];
              newTable[i] = e;
              e = next;
          }
      }
  }
  

### 图解

## 4.HashMap的put流程

### 关键代码

public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}


### putVal流程图解：

## 5.resize原理

### 1.什么时候开始扩容？

- 扩容是在记录插入元素size的大小大于等于threshold(临界点)的值的时候开始对map扩容。

  if (++size > threshold)
      resize();
  
  

### 2.threshold(临界点)是怎么计算出来的值？

- newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);第一次通过载荷因子0.75*16得出12，之后每次扩容都是上一次临界点的2倍。

  Node<K,V>[] oldTab = table;
  int oldCap = (oldTab == null) ? 0 : oldTab.length;
  int oldThr = threshold;
  int newCap, newThr = 0;
  //旧的数组长度大于0
  if (oldCap > 0) {
      //如果大于10亿，直接设置临界值为20亿，不扩容数组长度了
      if (oldCap >= MAXIMUM_CAPACITY) {
          threshold = Integer.MAX_VALUE;
          return oldTab;
      }
      //新的数组长度为以前的2倍，新的临界值也为以前的2倍
      else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
               oldCap >= DEFAULT_INITIAL_CAPACITY)
          newThr = oldThr << 1; // double threshold
  }
  else if (oldThr > 0) // initial capacity was placed in threshold
      newCap = oldThr;
  else {
      //16长度数组           // zero initial threshold signifies using defaults
      newCap = DEFAULT_INITIAL_CAPACITY;
      //12长度临界值
      newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
  }
  
  

### 3.怎么把旧的数据迁移到新的数组链上？

- 遍历旧的数组，进行扩容，没有形成链表的直接节点的e.hash & (newCap - 1)计算得到新数组的下标。有链表的将链表拆分【(e.hash & oldCap) == 0】为两个链，一个需要移动另一个不需要，减小迁移量。

  // 把当前index对应的链表分成两个链表，减少扩容的迁移量
                          HashMap.Node<K,V> loHead = null, loTail = null;
                          HashMap.Node<K,V> hiHead = null, hiTail = null;
                          HashMap.Node<K,V> next;
                          do {
                              next = e.next;
                              if ((e.hash & oldCap) == 0) {
                                  // 扩容后不需要移动的链表
                                  if (loTail == null)
                                      loHead = e;
                                  else
                                      loTail.next = e;
                                  loTail = e;
                              }
                              else {
                                  // 扩容后需要移动的链表
                                  if (hiTail == null)
                                      hiHead = e;
                                  else
                                      hiTail.next = e;
                                  hiTail = e;
                              }
                          } while ((e = next) != null);
                          if (loTail != null) {
                              // help gc
                              loTail.next = null;
                              newTab[j] = loHead;
                          }
                          if (hiTail != null) {
                              // help gc
                              hiTail.next = null;
                              // 扩容长度为当前index位置+旧的容量
                              newTab[j + oldCap] = hiHead;
                          }
                      }
  
  ————————————————
  版权声明：本文为CSDN博主「青元子」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
  原文链接：https://blog.csdn.net/u010890358/article/details/80496144
  

### 4.可以自定义负载因子吗？

- 可以的，在扩容操作中其中一个if分支里做的就是根据自定义负载因子确定临界值。

  if (newThr == 0) {
      float ft = (float)newCap * loadFactor;
      newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                (int)ft : Integer.MAX_VALUE);
  }
  
  

### 5.需要注意的是：扩容的时候，如果没有形成链表或者树，确定位置的方式是[hashCode&(newLentgh-1)]，形成链表的拆分为两个，要移动的确定位置计算方式是 原位置+旧数组长度 (index+oldLength)

## 6.扩展性问题

### 1.怎么解决hash冲突？

- 1.使用链表和树来将相同hash值数据存储
- 2.hashcode低16位与hashcode的高16位进行异或操作，降低hash冲突概率

### 2.能否使用任何类做map的key？

- 1.不能，如果某个类重写了equals和hashcode方法，可能会导致hash出现很多冲突。变成单链表这些。
- 2.String、Integer类更适合作为k，因为是不可变(final)，并且equals和hashCode方法严格遵守规范，不容易出现hash计算错误的情况。

	- equal的规范是什么？

		- 1.自反性

			- x.equals(x)==true

		- 2.对称性

			- y.equals(x)==true  <---> x.equals(y)==true

		- 3.传递性

			- x.equals(y)==true，y.equals(z)==true  ---> x.equals(z)==true

		- 4.一致性

			- 对于不等于null的x和y，只要对象中，被equals操作使用的信息没有被修改，那么多次调用x.equals(y)，要么一直返回true，要么一直返回false。
			- 要想严格遵循这一条约定，必须保证equals方法里面不依赖外部不可靠的资源，如果equals方法里面依赖了外部的资源，就很难保证具有一致性了。

		- 5.非空性

			- 对于不等于null的x，x.equals(null)必须返回false。

### 3.碰到线程安全问题怎么办？

- 使用ConCurrentHashMap，分段锁形式，既保证线程安全，有保证效率。
- HashTable线程安全但是过时了，没人用
- CollectionsUtils给HashMap上锁，或者Synchronized关键字上锁，这些上锁方式都极大影响性能

### 5.对顺序有要求可以选择TreeMap

## 7.刁专的问题

### 1.为什么是长度2的幂次方？

- 取余(%)操作除数是2的幂次方等价于除数减一的与(&)操作，也就是定位新数组迁移数据的[hash&(length-1)]操作，采用二进制的&操作计算，提高运算效率，这就解释了为什么是2的幂次方。

### 2.为什么是2次扰动，而不是多次扰动？

- 高16与低16位同时参与异或运算，足以减少hash冲突。所以这两次就够了。

### 3.如何用自己的Object作为key？

- 1.重写hashCode()方法，一定要用对象的关键部分参与散列码的计算，这样可以减少hash碰撞。

	- 像String就使用了每一个字符参与hashcode计算

- 2.重写equals()方法，需要遵循自反性、对称性、传递性、一致性，为了保证key在哈希表中的唯一性。

