# 面试必问ConcurrentHashMap

## 1.JDK1.7底层实现

### 图：

### 1.底层采用Segment+HashEntry的方式进行实现

### 2.是一种数组和链表的数据结构，每一个Segment包含一个HashEntry数组

### 3.Segment用来充当锁的角色(并且是一个可重入锁)，HashEntry用来封装映射表的键值对

## 2.JDK1.8底层实现

### 1.采用Node+CAS+Synchronized来保证并发安全，synchronized只锁链表或者红黑树的首节点，只要不发生hash冲突，就不会产生并发

- 图：

## 3.源码

### 1.put数据

    public V put(K key, V value) {
        return putVal(key, value, false);
    }
 
    final V putVal(K key, V value, boolean onlyIfAbsent) {
 
        //1.校验参数是否合法
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
 
        //2.遍历Node
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            
            //2.1.Node为空，初始化Node
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
 
            //2.2.CAS对指定位置的节点进行原子操作
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
 
            //2.3.如果Node的hash值等于-1,map进行扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
 
            //2.4.如果Node有值，锁定该Node。
            //如果key的hash值大于0，key的hash值和key值都相等，则替换，否则new一个新的后继Node节点存放数据。
            //如果key的hash小于0，则考虑节点是否为TreeBin实例，替换节点还是额外添加节点。
 
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
 
        //3.计算map的size
        addCount(1L, binCount);
        return null;
    }


### 2.初始化table

   
    private transient volatile int sizeCtl;
 
    private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
 
            //如果当前Node正在进行初始化或者resize(),则线程从运行状态变成可执行状态，cpu会从可运行状态的线程中选择
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
 
            //如果当前Node没有初始化或者resize()操作，那么创建新Node节点，并给sizeCtl重新赋值
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
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


### 3.Unsafe提供了三个原子操作

    @SuppressWarnings("unchecked")
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
 
        //获取obj对象中offset偏移地址对应的object型field的值,支持volatile load语义
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }
 
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
 
        //在obj的offset位置比较object field和期望的值，如果相同则更新。
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }
 
    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
 
        //设置obj对象中offset偏移地址对应的object型field的值为指定值。
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
    }


