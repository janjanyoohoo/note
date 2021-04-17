# JUC

## volatile关键字与内存可见性

**volatile只能保证数据的**

- **具有可见性**
- **无法保证数据的原子性**
- **不具备互斥性**

当多个线程操作共享数据时，可以保证内存中的数据可见。用这个关键字修饰共享数据，就会**及时的把线程缓存中的数据刷新到主存中去**，也可以理解为，就是直接操作主存中的数据。所以在不使用锁的情况下，可以使用volatile。

​	**volatile通过命令cpu寄存器不要对数据进行缓存,并且对数据的操作都会通知到使用该数据的线程保证数据的可见性;**

​	其他线程再读取该数据时,不会从缓存中读取而是从内存中读取最新的值;

![image-20210203115845487](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210203115845.png)

![image-20210203115856949](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210203115857.png)



## synchronized

​	关键字调用底层的native方法,最终在1.6之前的实现方式是直接添加重量级锁,调用OS资源进行线程操作,性能较差;

 	在1.6之前时,加锁的对象实际操作为底层的**Object monitor**对象进行加锁;在1.6之后通过对象的对象头标志位进行存储锁信息;

**synchronized 可以保证数据的 **

- **原子性**
- **可见性**

- **具备互斥性**

## 分析对象头

 java的对象头在对象的不同状态下会有不同的表现形式，

- 无锁状态   0 01
- 偏向锁      1 01
- 轻量级锁     00
- 重量级索     10
- gc标记状态 11

![image-20210203114151877](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210203114151.png)

通过**openjdk.jol**可以打印对象头的信息

```xml
<dependency> 
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol‐core</artifactId>
    <version>0.9</version>
</dependency>
```

## 偏向锁

​	在jdk1.6后,当只有一个线程访问同步资源时,会先使用偏向锁而不是直接重量级锁;

## 轻量级锁

 轻量级锁尝试在应用层面解决线程同步问题，而不触发操作系统的互斥操 作，轻量级锁减少多线程进入互斥的几率，不能代替互斥

当有多个线程争抢共享资源时,偏向锁就会是升级为轻量级锁; 通过CAS自旋方式进行同步资源访问

## 重量级锁

​	直接调度系统资源进行线程操作, 当轻量级锁自旋达到一定次数时, 轻量级锁就会升级为重量级锁; JDK也在逐渐优化重量级锁的性能问题;



## Condition

`Condition` 将 `Object` 监视器方法（[`wait`](../../../../java/lang/Object.html#wait())、[`notify`](../../../../java/lang/Object.html#notify()) 和 [`notifyAll`](../../../../java/lang/Object.html#notifyAll())）分解成截然不同的对象，以便通过将这些对象与任意  [`Lock`](../../../../java/util/concurrent/locks/Lock.html)  实现组合使用，为每个对象提供多个等待 set（wait-set）。其中，`Lock` 替代了  `synchronized` 方法和语句的使用，`Condition` 替代了 Object 监视器方法的使用。 

- signalAll()  唤醒全部
- signal()    唤醒一个 对应调用了await的线程
- await()     休眠

**与此 Condition 相关的锁以原子方式释放，并且出于线程调度的目的，将禁用当前线程**，且在发生以下四种情况之一 以前，当前线程将一直处于休眠状态： 

- 其他某个线程调用此 Condition 的 signal() 方法，并且碰巧将当前线程选为被唤醒的线程；
- 其他某个线程调用此 Condition 的 signalAll() 方法；
- 其他某个线程中断当前线程，且支持中断线程的挂起；
- 发生“虚假唤醒” 

## CopyOnWriteArrayList

### 写时复制：

​	 CopyOnWrite容器即是写时复制容器,不直接往当前容器中Ovject[]添加数据,而是对当前容器先copy一份,复制一个新的容器newElement,然后往新容器中写入数据,添加完数据后再把新容器指向原容器的引用;

​	这样做的好处是可以对CopyOnWrite容器进行并发的读,而不需要加锁,因为当前容器不会添加任何元素,所以copyonwrite也是一种读写分离的思想,读和写不同的容器;

## CopyOnWriteArraySet

```

```

## CountDownLatch

```

```

## CyclicBarrier

```

```

## Semaphore

信号量获取不到将会进入阻塞

```

```

## ReadWriteLock

如果有一个线程对共享资源进行写,那么就会全部排他,进入阻塞; 如果只是读则可以并发读;

```

```

## LongAdder

在高并发下实现高性能统计的类, 其主要内容在**Striped64**类中;

### add

```java
 public void add(long x) {
        Cell[] as; long b, v; int m; Cell a;
     //如果当前cells数组不是null  并且尝试CAS修改base的值失败
        if ((as = cells) != null || !casBase(b = base, b + x)) {
            // 是否未发生竞争的标记
            boolean uncontended = true;
            // 当前数组null 或 length-1 == 0 或 当前getProbe线程hash计算对应的下标元素为null  或 再次尝试CAS修改修改值发生失败
            if (as == null || (m = as.length - 1) < 0 ||(a = as[getProbe() & m]) == null ||!(uncontended = a.cas(v = a.value, v + x)))
                //尝试在数组中添加值
                longAccumulate(x, null, uncontended);
        }
    }
```

### casBase & cas

```java
    final boolean casBase(long cmp, long val) {
        return UNSAFE.compareAndSwapLong(this, BASE, cmp, val);
    }

    final boolean cas(long cmp, long val) {
        return UNSAFE.compareAndSwapLong(this, valueOffset, cmp, val);
    }
```

### longAccumulate

```java
//x = 需要增加的值  fn=长二进制运算符  wasUncontended=是否未发生竞争
final void longAccumulate(long x, LongBinaryOperator fn,boolean wasUncontended) {
    	//hash值变量
        int h;
    	//当前线程hash==0 说明未分配hash
        if ((h = getProbe()) == 0) {
            //初始化线程的hash
            ThreadLocalRandom.current(); // force initialization
            //保存线程hash
            h = getProbe();
            //因为hash值没有初始化,竞争发生在索引0上导致失败, 所以重置线程的未竞争状态为true, 打在0索引上的竞争忽视
            wasUncontended = true;
        }
    	//扩容意向
        boolean collide = false;                // True if last slot nonempty
    	//重试机制
        for (;;) {
            //变量定义
            Cell[] as; Cell a; int n; long v;
            //分支1 如果当前数组部位null 并且 length>0  说明已经初始化
            if ((as = cells) != null && (n = as.length) > 0) {
                //分支1.1 当前线程对应的下标元素是为null
                if ((a = as[(n - 1) & h]) == null) {
                    //分支1.1.1 锁是否是可获取状态
                    if (cellsBusy == 0) {       // Try to attach new Cell
                        //创建一个新元素 值为需要存储的x
                        Cell r = new Cell(x);   // Optimistically create
                        //分支1.1.1.1 再次确认锁自由状态, 并尝试CAS获取锁
                        if (cellsBusy == 0 && casCellsBusy()) {
                            //创建节点的标记 = false
                            boolean created = false;
                            try {               // Recheck under lock
                                Cell[] rs; int m, j;
                                //分支1.1.1.1.1 如果当前cells不为null 并且 length>0 并且 线程对应的下标中元素为null
                                if ((rs = cells) != null && (m = rs.length) > 0 && rs[j = (m - 1) & h] == null) {
                                    //将新建的节点放入对应的下表中
                                    rs[j] = r;
                                    //新建成功的标记
                                    created = true;
                                }
                            } finally {
                                //释放锁
                                cellsBusy = 0;
                            }
                            //新建成功的标记, 则跳出循环
                            if (created)
                                break;
                            //如果在获取到锁以后尝试添加不满足条件分支1.1.1.1.1 则可能是并发线程修改了当前数组,跳过此次循环再次进行重新判断
                            continue;           // Slot is now non-empty
                        }
                    }
                    //扩容意向false
                    collide = false;
                }
                //分支1.2 如果发生了竞争CAS失败,会到此分支,那么重置一次,再次外部循环
                else if (!wasUncontended)       // CAS already known to fail
                    wasUncontended = true;      // Continue after rehash
                //分支1.3 如果到此则说明当前线程再次竞争失败一次,然后尝试cas修改一次
                else if (a.cas(v = a.value, ((fn == null) ? v + x : fn.applyAsLong(v, x))))
                    //如果修改成功则跳出
                    break;
                //分支1.4 到此则说明竞争失败多次, 判断是否符合扩容的条件,如果当前length>cpu核心数,或者当前数组已经改变了 则将扩容意向变为false,线程再次返回开始重拾
                else if (n >= NCPU || cells != as)
                    collide = false;            // At max size or stale
                //分支1.5 到达此处的线程会是失败多次的线程,所以扩容意向改为true(不是一定会扩容,因为此处返回会经过1.4,不满足条件会再次改为false)
                else if (!collide)
                    collide = true;
                //分支1.6 尝试获取锁,进行扩容
                else if (cellsBusy == 0 && casCellsBusy()) {
                    try {
                        //获取到锁再次校验数组未被并发修改过
                        if (cells == as) {      // Expand table unless stale
                            //扩容一倍
                            Cell[] rs = new Cell[n << 1];
                            //将原数组的元素放到新数组对应下标
                            for (int i = 0; i < n; ++i)
                                rs[i] = as[i];
                            //引用新数组
                            cells = rs;
                        }
                    } finally {
                        //释放锁
                        cellsBusy = 0;
                    }
                    //扩容意向 fasle
                    collide = false;
                    //跳过此次循环后,已经扩容完成,线程返回继续重试
                    continue;                   // Retry with expanded table
                }
                //每次CAS失败执行到此处都会对线程的hash值重置,下次重试时就可能会换一个索引进行尝试竞争
                h = advanceProbe(h);
            }
            //分支2 当前数组未初始化 尝试获取锁 并且获取成功
            else if (cellsBusy == 0 && cells == as && casCellsBusy()) {
                //初始化标记
                boolean init = false;
                try {                           // Initialize table
                    //获取到了锁,校验数组未被并发修改
                    if (cells == as) {
                        //初始化数组 大小2
                        Cell[] rs = new Cell[2];
                        //在当前线程的对应下标处放入一个新元素,值是当前线程任务所携带的数据
                        rs[h & 1] = new Cell(x);
                        //引用新数组
                        cells = rs;
                        //初始化标记 true
                        init = true;
                    }
                } finally {
                    //释放锁
                    cellsBusy = 0;
                }
                //初始化成功? 成功则跳出循环
                if (init)
                    break;
            }
            //分支3 数组未初始化,并且尝试获取锁失败,掉头尝试修改base的值,成功则跳出循环,
            //fn==null?base+1 : 否则执行fn.applyAsLong(v, x)的逻辑,因为参数传的null  此处不会执行到fn.applyAsLong
            else if (casBase(v = base, ((fn == null) ? v + x : fn.applyAsLong(v, x))))
                break;                          // Fall back on using base
        }
    }
```



## ConcurrentHahMap

### 属性与构造

```java
	//哈希表最大值 0111 1111 1111 1111 1111 1111 1111 1111  2的幂
	private static final int MAXIMUM_CAPACITY = 1 << 30;
	//哈希表默认大小 2的幂
    private static final int DEFAULT_CAPACITY = 16;
	//可能的最大的数组大小（非2的幂）。 toArray和相关方法需要。
    static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
	//默认的并发级别,未使用到
    private static final int DEFAULT_CONCURRENCY_LEVEL = 16;
	//加载因子
    private static final float LOAD_FACTOR = 0.75f;
	//树化阈值
    static final int TREEIFY_THRESHOLD = 8;
	//转换为链表阈值6   7作为过渡防止频繁	链表 <=> 红黑树
    static final int UNTREEIFY_THRESHOLD = 6;
	//转换树的前提最小元素数量64, 即使链表长度>8 也需要满足此条件才会树化
    static final int MIN_TREEIFY_CAPACITY = 64;
	//哈希表扩容时,每一个线程默认扩容步长16,最小步长为DEFAULT_CAPACITY
    private static final int MIN_TRANSFER_STRIDE = 16;
	//The number of bits used for generation stamp in sizeCtl. Must be at least 6 for 32bit arrays.
	//sizeCtl中用于生成标记的位数。对于32位阵列，必须至少为6。
    private static int RESIZE_STAMP_BITS = 16;
	// MAX_RESIZERS = 0111 1111 1111 1111 1111 1111 1111 1111 最大扩容阈值
    private static final int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
	// sizeCtl的最大值为16
    private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

	//
    static final int MOVED     = -1; // hash for forwarding nodes
    static final int TREEBIN   = -2; // hash for roots of trees
    static final int RESERVED  = -3; // hash for transient reservations
    static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash
	//当前电脑的cpu核数
    static final int NCPU = Runtime.getRuntime().availableProcessors();
	//用于序列化兼容性。 Segment是jdk1.7 concurrentHashMap所使用的技术
    private static final ObjectStreamField[] serialPersistentFields = {
        new ObjectStreamField("segments", Segment[].class),
        new ObjectStreamField("segmentMask", Integer.TYPE),
        new ObjectStreamField("segmentShift", Integer.TYPE)
       
	//当前的哈希表
    transient volatile Node<K,V>[] table;

    /**
     * 只在扩容时使用
     */
    private transient volatile Node<K,V>[] nextTable;

    /**
     * 基础计数器, 竞争成功时count+1
     */
    private transient volatile long baseCount;

    /**
     * 扩容阈值
     */
    private transient volatile int sizeCtl;

    /**
     * 调整大小时要拆分的下一个表索引,分段扩容时使用
     */
    private transient volatile int transferIndex;

    /**
     * 调整大小和/或创建CounterCell时使用的锁标志位
     */
    private transient volatile int cellsBusy;

    /**
     * baseCount发生竞争后,竞争失败线程尝试将数据保存在此数组,初始值length为2
     */
    private transient volatile CounterCell[] counterCells;

    // views
    private transient KeySetView<K,V> keySet;
    private transient ValuesView<K,V> values;
    private transient EntrySetView<K,V> entrySet;
```

### Node

```java
    //哈希表中存储的元素
	static class Node<K,V> implements Map.Entry<K,V> {
        //当前key的hash值
        final int hash;
        //key value
        final K key;
        volatile V val;
        // 下一个节点,volatile保证可见性
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }
    }
```

### hash算法

```java
 	//HASH_BITS=0x7fffffff 32位的正数最大值
	//通过此方法将一个hash值得高位参与运算,降低hash冲突的几率,并通过& 0111 1111 1111 1111 1111 1111 1111 1111 保证一定是一个正数
	static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
    }

    /**
     * Returns a power of two table size for the given desired capacity.
     * See Hackers Delight, sec 3.2
     */
	//自定义capacity时,计算出最接近该值的2次幂
	//无符号右移,31位,最终结果是从该值得最高位到最低位 全为1, 然后+1, 就是该值最近的一个2次幂整数
    private static final int tableSizeFor(int c) {
        int n = c - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

### putVal

```java
public V put(K key, V value) {
        return putVal(key, value, false);
    }

    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        //k v不允许存储null
        if (key == null || value == null) throw new NullPointerException();
        // 计算哈希,高16位参与运算
        int hash = spread(key.hashCode());
        
        int binCount = 0;
        //重试机制
        for (Node<K,V>[] tab = table;;) {
            
            Node<K,V> f; int n, i, fh;
            //情况1 : 如果当前表==null,或 当前表length==0, 对表进行初始化
            if (tab == null || (n = tab.length) == 0)
                //初始化表,然后继续循环重试
                tab = initTable();
            //情况2 : n=tab.length , i=tab.length-1, hash=spread(key.hashCode());  tabAt找到当前参数key对应索引处元素是否为null,
            //true进入循环,false尝试情况3
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                //情况2.1  下标出元素为null,则CAS操作new一个元素放到指定下标处, true 则break, false说明有竞争且竞争失败继续循环
                if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            //情况3 : 对应下标元素不是null,且对应元素的hash值为-1,说明正在扩容,则尝试帮助其他线程进行扩容
            else if ((fh = f.hash) == MOVED)
                //帮助当前表进行扩容(多线程扩容)
                tab = helpTransfer(tab, f);
            //情况4 : 表已经初始化,且参数key对应元素不是null,表也没有在扩容,开始遍历链表或者红黑树的逻辑
            else {
                V oldVal = null;
                //线程安全同步,此处用作锁的对象是链表存储在hash表中的头元素,如果是红黑树,则是用红黑树对象(与HashMap不同)
                synchronized (f) {
                    //双重检验,防止头元素发生了变化,确认没有变化
                    if (tabAt(tab, i) == f) {
                        //确认头元素的hash值是大于等于0,负数可能是红黑树
                        if (fh >= 0) {
                            //链表长度的标记
                            binCount = 1;
                            //遍历链表
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                //如果当前元素和参数key,hash都相同则更新值
                                if (e.hash == hash &&((ek = e.key) == key ||(ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    //此处是put方法传进来的 false
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                //没有相同的key,执行插入,尾插法
                                Node<K,V> pred = e;
                                //当前的next为null 则new一个节点放到next节点
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,value, null);
                                    //完成插入 结束玄幻
                                    break;
                                }
                            }
                        }
                        //如果当前是红黑树节点,则执行红黑树逻辑
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            //红黑树put值
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                //插入完成后判断binCount值是否需要进行树化
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        //树化的方法
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        //类似于HashMap的size属性, 此处逻辑与LongAdder相似, 注意此处维持的是count的最终一致性,而不是实时的一致性
        addCount(1L, binCount);
        return null;
    }
```

### initTable

```java
    private final Node<K,V>[] initTable() {
        
        Node<K,V>[] tab; int sc;
        //如果当前表==null,或 当前表length==0, 对表进行初始化
        while ((tab = table) == null || tab.length == 0) {
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
            //CAS操作修改标志位
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    //双重校验table是否改变
                    if ((tab = table) == null || tab.length == 0) {
                        //如果sizeCtlfu'ze>0 则初始化表大小为sizeCtl否则为默认值16
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        //sc 扩容阈值 = length-length>>2相当于 1 - 1/4 = 0.75
                        sc = n - (n >>> 2);
                    }
                } finally {
                    //扩容阈值赋值
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```

### addCount

```java
//x=1   check=binCount 链表长度或红黑树的2
private final void addCount(long x, int check) {
        CounterCell[] as; long b, s;
        //as = 当前的counterCells数组
    	//如果当前数组不为null 或 CAS将baseCount+1 失败
    	if ((as = counterCells) != null || !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            
            CounterCell a; long v; int m;
            //没有发生竞争的标记
            boolean uncontended = true;
            //情况1 : 当前cells数组为null 
            //情况2 : 当前cells数组不为null 并且 当前数组的length-1<0
            //情况3 : 当前cells数组不为null 并且 当前数组的length-1不小于0 并且 当前线程hash值对应当前数组的下标中元素为null
            //情况4 : 当前cells数组不为null 并且 当前数组的length-1不小于0 并且 当前线程hash值对应当前数组的下标中元素不为null 并且 尝试CAS将当前线程hash值对应cells数组下标的元素+1 发生了竞争且竞争修改失败了
            if (as == null || (m = as.length - 1) < 0 || (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended = U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                //进入fullAddCount方法尝试修改
                fullAddCount(x, uncontended);
                //从此方法出来则一定是修改成功了(或者其他不定错误原因)所以直接返回
                return;
            }
            //check<=1的情况1 未执行更新或插入的逻辑,没有元素修改,不需要count修改
            if (check <= 1)
                return;
            //统计count数量
            s = sumCount();
        }
    	//如果check>0  说明哈希表有发生元素的变化
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            //重试机制  如果当前哈希表的元素个数>扩容阈值, 并且 当前哈希表已经初始化  并且 当前哈希表的length 小于 扩容的最大值
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null && (n = tab.length) < MAXIMUM_CAPACITY) {
                //rs是一个扩容有关的扩容戳
                int rs = resizeStamp(n);
                //分支1 如果扩容阈值小于0,则应该进行初始化 TODO
                if (sc < 0) {
                     //高16位代表扩容的标记、低16位代表并行扩容的线程数 
        			//高RESIZE_STAMP_BITS位 扩容标记 
        			//低RESIZE_STAMP_SHIFT位 并行扩容线程数
                    //分支1.1 不进行扩容
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 || sc == rs + MAX_RESIZERS || (nt = nextTable) == null 
                        || transferIndex <= 0)
                        break;
                    //尝试CAS将扩容阈值修改为 sc+1  代表当前正在进行初始化逻辑
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                //分支2  尝试扩容CAS将扩容阈值修改为 sc+2  代表当前在进行扩容逻辑
                else if (U.compareAndSwapInt(this, SIZECTL, sc,(rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                s = sumCount();
            }
        }
    }

    static final int resizeStamp(int n) {
        /*
        	numberOfLeadingZeros该方法的作用是返回无符号整型i的最高非零位前面的0的个数，包括符号位在内；
			如果i为负数，这个方法将会返回0，符号位为1.
			比如说，10的二进制表示为 0000 0000 0000 0000 0000 0000 0000 1010
			java的整型长度为32位。那么这个方法返回的就是28
        */
        //(1 << (RESIZE_STAMP_BITS - 1) 返回恒定是 1111 1111 1111 1111, 16位
        return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
    }
```

![image-20210206211911574](C:/Users/jianjian/AppData/Roaming/Typora/typora-user-images/image-20210206211911574.png)

### fullAddCount

```java
	//x=1,wasUncontended是否未发生了竞争
	private final void fullAddCount(long x, boolean wasUncontended) {
        int h;
        //如果当前线程的hash为0,则rehash
        if ((h = ThreadLocalRandom.getProbe()) == 0) {
            ThreadLocalRandom.localInit();      // force initialization
            h = ThreadLocalRandom.getProbe();
            //重置是否未发生竞争的状态为true,因为hash值为0,所以之前发生过的一次竞争忽略
            wasUncontended = true;
        }
        //扩容意向,fasle不扩容, true可能扩容可能不扩容,看具体条件
        boolean collide = false;                // True if last slot nonempty
        //循环重试机制
        for (;;) {
            //变量定义
            CounterCell[] as; CounterCell a; int n; long v;
            //情况 1 : 如果当前counterCells不是null 并且 length>0 说明已经初始化
            if ((as = counterCells) != null && (n = as.length) > 0) {
                //情况1.1 : 如果当前线程的hash值对应的数组下标中元素为null
                if ((a = as[(n - 1) & h]) == null) {
                    //当前cellsBusy锁标记是否是自由状态 0无锁,1锁被占用
                    if (cellsBusy == 0) {            // Try to attach new Cell
                        //创建一个CounterCell对象,参数为需要添加的值 1
                        CounterCell r = new CounterCell(x); // Optimistic create
                        //再次确认锁是否可以获取 然后尝试获取锁 CAS修改CELLSBUSY锁的状态为1
                        //情况1 : 锁可获取 并且获取锁成功
                        //注意此处的两个变量cellsBusy,CELLSBUSY不是同一个变量但是,CELLSBUSY在静态代码块中初始化为cellsBusy,所以CELLSBUSY每次获取到的值是最新的cellsBusy值,修改的同样也是cellsBusy的值; 在下方释放锁的代码中,释放的是cellsBusy的值
                        if (cellsBusy == 0 && U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                            //是否新创建了节点的标记
                            boolean created = false;
                            try {               // Recheck under lock
                                CounterCell[] rs; int m, j;
                                //再次校验当前counterCells数组没有被并发修改掉 因为A线程在cellsBusy == 0时停止, 线程B获取锁执行完了全部流程后线程A可以正常获取锁进入,但是此时数组已经发生变化,所以需要再次校验
                                //情况1 : 数组不为null &&  length>0  && 当前需要插入元素的节点为null
                                if ((rs = counterCells) != null &&(m = rs.length) > 0 && rs[j = (m - 1) & h] == null) {
                                    //更新对应下标的节点为新建的CounterCell对象
                                    rs[j] = r;
                                    //修改成功标记
                                    created = true;
                                }
                            } finally {
                                //释放锁
                                cellsBusy = 0;
                            }
                            //修改成功? 成功则跳出 失败则再次回到情况1开始重试
                            if (created)
                                break;
                            //注意此处continue跳过的进入了cellsBusy == 0 && U.compareAndSwapInt(this, CELLSBUSY, 0, 1)但是插入元素失败的情况或者当前counterCells数组已经变化了的情况,则进行下次重试
                            continue;           // Slot is now non-empty
                        }
                    }
                    //扩容意向? 发生了
                    collide = false;
                }
                //情况1.2 : 如果线程未发生竞争的标记位fasle 说明发生了竞争 则重置标记让线程去rehash更换一个对应索引后再次进行重试
                else if (!wasUncontended)       // CAS already known to fail
                    wasUncontended = true;      // Continue after rehash
                //情况1.3 : 为发生竞争线程,但是对应下标的元素不是null,尝试修改CELLVALUE,成功则结束
                else if (U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))
                    break;
                //情况1.4 : 走到此处的线程 对应下标有元素,并且尝试修改元素的值时又与并发线程CAS竞争了且竞争失败,就要考虑扩容了,但是要数组的length小于cpu的内核数量,因为数组大于内核数的话并不会被并发访问到,浪费资源; 然后rehash后开始新一轮的回到情况1重试
                else if (counterCells != as || n >= NCPU)
                    //扩容意向, 此处false因为在考虑扩容前还需要再重试一次
                    collide = false;            // At max size or stale
                //情况1.5 : 扩容意向fasle取反
                //能走到这里是说明当前线程已经到了一次 情况1.4 又失败了, 然后将扩容意向改为true, 考虑扩容, 然后rehash后开始新一轮的回到情况1重试
                else if (!collide)
                    collide = true;
                //情况1.6 能够到这一步 说明线程已经自旋尝试很多次都失败了, 如果锁是可获取状态,会尝试获取锁,获取成功则进入扩容逻辑
                else if (cellsBusy == 0 && U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                    try {
                        //获取到锁 再次校验数组是否没有改变
                        if (counterCells == as) {// Expand table unless stale
                            //数组扩容1倍
                            CounterCell[] rs = new CounterCell[n << 1];
                            //将对应索引的元素放到新数组同样的索引上
                            for (int i = 0; i < n; ++i)
                                rs[i] = as[i];
                            //更新新数组到成员变量
                            counterCells = rs;
                        }
                    } finally {
                        //释放锁
                        cellsBusy = 0;
                    }
                    //扩容意向重置为false
                    collide = false;
                    //扩容完成,跳过此次循环,回到情况1 开始新一次的重试
                    continue;                   // Retry with expanded table
                }
                //每次进入情况1 但是走到这一步都说明尝试修改值失败了,发生了竞争; 那么重置一个线程的hash值让线程下次重试换一个索引来尝试更新值
                h = ThreadLocalRandom.advanceProbe(h);
            }
            //情况 2 : 当前数组为null未初始化 并且 锁状态可获取 并且 数组未改变 并且 获取锁成功
            else if (cellsBusy == 0 && counterCells == as &&  U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                //初始化标记
                boolean init = false;
                try {                           // Initialize table
                    //尝试初始化数组, 但是会校验一下数组是否已经被并发线程初始化了
                    if (counterCells == as) {
                        //未初始化,则初始化数组, 默认长度为2
                        CounterCell[] rs = new CounterCell[2];
                        //初始化完成后会将自己线程的任务 new一个元素放到数组里, 完成自己的任务跳出循环
                        rs[h & 1] = new CounterCell(x);
                        //赋值当前成员变量counterCells数组为新初始化的数组
                        counterCells = rs;
                        //初始化完成的标记
                        init = true;
                    }
                } finally {
                    //释放锁
                    cellsBusy = 0;
                }
                //初始化成功? 成功则跳出, 如果已经被并发线程初始化了等情况则再次回到情况1开始重试
                if (init)
                    break;
            }
            //情况 3 : 当前数组为null / 锁不可获取或获取失败 / counterCells数组改变了
            //再次尝试CAS 修改baseCount属性的值,成功则跳出,失败则再次回到情况1开始重试
            else if (U.compareAndSwapLong(this, BASECOUNT, v = baseCount, v + x))
                break;                          // Fall back on using base
        }
    }
```

### transfer

```java
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        //定义stride步长
        int n = tab.length, stride;
        //如果不是单核cpu,那么步长stride=length的八分之一 / cpu核心数 (例如length=16, stride = 2/cpu核心数)
        //单核cpu 步长=length
        //如果步长小于默认步长16, 进入此逻辑
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            //步长小于16,则指定默认为16
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        //如果目标nextTab未初始化定义
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                //初始化新哈希表,并指向引用, 扩容大小为 旧数组的1倍
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                //如果发生异常,则指定扩容阈值为一个不可能到达的值不再进行扩容,尝试解决内存溢出
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            //引用nextTab到全局变量
            nextTable = nextTab;
            //新表的开始index=n,  n=旧表的length
            transferIndex = n;
        }
        //新表的length
        int nextn = nextTab.length;
        //
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;
        boolean finishing = false; // to ensure sweep before committing nextTab
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            while (advance) {
                int nextIndex, nextBound;
                if (--i >= bound || finishing)
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```



