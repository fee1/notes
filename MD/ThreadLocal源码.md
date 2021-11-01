# ThreadLocal源码解析
## ThreadLocal简介
```text
    ThreadLocal是一个本地线程副本变量工具类，在每一个线程中都创建一个ThreadLocalMap对象，简单来说ThreadLocal就是一种以空间换取时间的做法，
每个线程都可以访问自己内部的ThreadLocalMap对象的value，通过这种方式避免资源在多线程之间共享。
    常用于解决共享变量属性的线程安全问题。
```
## ThreadLocal的set方法
```java
    public void set(T value) {
        //获取当前线程
        Thread t = Thread.currentThread();
        //获取当前线程的ThreadLocalMap
        ThreadLocalMap map = getMap(t);
        if (map != null)
        map.set(this, value);
        else
            //当前线程ThreadLocalMap如果为null就创建一个，并且赋值
        createMap(t, value);
    }
```
```text
    getMap和createMap都挺简单的，所以不详细说，下面介绍ThreadLocalMap的set方法。
```
### ThreadLocalMap的set方法
```java
public class ThreadLocal<T> {
    private final int threadLocalHashCode = nextHashCode();

    private static AtomicInteger nextHashCode =
            new AtomicInteger();

    private static final int HASH_INCREMENT = 0x61c88647;
    
    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }

    //注意这个类型，这个类是一个弱引用类型的子类（WeakReference类做的弱引用，下次GC就会被回收）
    static class Entry extends WeakReference<ThreadLocal<?>> {
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }

    static class ThreadLocalMap {
        private Entry[] table;

        private void set(ThreadLocal<?> key, Object value) {
            Entry[] tab = table;
            int len = tab.length;
            //每次new出来的ThreadLocal后的threadLocalHashCode都会不一样，多个ThreadLocal共享一个AtomicInteger线程安全的类去生成此数字
            //这样计算出来的值保证数据均匀分散在数组里，并且每次同一个ThreadLocal计算出来的位置都是一样的
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();
                //同一个ThreadLocal就替换值
                if (k == key) {
                    e.value = value;
                    return;
                }
                //k==null，说明有可能因为key是弱引用被GC回收了，这里需要做一下处理，去除掉那些相同的ThreadLocal
                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }
            //如果这个ThreadLocal从类没有出现在tab数组里，直接不会走循环，在计算出来的位置添加这个ThreadLocal
            tab[i] = new Entry(key, value);
            int sz = ++size;
            //到达临界点扩容，原理和HashMap扩容类似
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }

        private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                                       int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;
            Entry e;
            
            //往上寻找到最新的一个key为null的
            int slotToExpunge = staleSlot;
            for (int i = prevIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = prevIndex(i, len))
                if (e.get() == null)
                    slotToExpunge = i;
                
            for (int i = nextIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = nextIndex(i, len)) {
                ThreadLocal<?> k = e.get();
                
                //如果同一个ThreadLocal对象，两个位置对象互换
                if (k == key) {
                    e.value = value;

                    tab[i] = tab[staleSlot];
                    tab[staleSlot] = e;
                    
                    if (slotToExpunge == staleSlot)
                        slotToExpunge = i;
                    //删除旧的数据
                    cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
                    return;
                }
                //来到进入方法时的位置，记录位置
                if (k == null && slotToExpunge == staleSlot)
                    slotToExpunge = i;
            }
            
            //给这个位置设置新的Entry对象
            tab[staleSlot].value = null;
            tab[staleSlot] = new Entry(key, value);
            
            if (slotToExpunge != staleSlot)
                cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
        }
    }
}
```
## 总结
```text
    每个线程都有一个自己的ThreadLocalMap，每个ThreadLocalMap里边都维护了一个属于自己的Entry数组，填充这个数组的方式和填充HashMap的方式
差不多(它相同值时替换，HashMap是添加到链表或者树)。同一个ThreadLocal的set会把上一次的值覆盖掉，也就是说，同一个线程的同一个ThreadLocal
进行set操作，会把之前的值覆盖掉。同一个Thread的不同ThreadLocal会均匀的分布在Entry数组里。
```
![Image](../aimage/Threadlocal.png)