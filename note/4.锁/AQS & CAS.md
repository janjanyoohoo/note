# AQS & CAS

\#并发编程/AQS(AbstractQueuedSynchronizer)#

# synchronized

 关键字调用底层的native方法,最终在1.6之前的实现方式是直接添加重量级锁,调用OS资源进行线程操作,性能较差;

 在1.6之前时,加锁的对象实际操作为底层的Object monitor对象进行加锁;在1.6之后通过对象的对象头标志位进行存储锁信息;

## 分析对象头

 java的对象头在对象的不同状态下会有不同的表现形式，

- 无锁状态   0 01
- 偏向锁 1 01
- 轻量级锁 00
- 重量级索 10
- gc标记状态 11

![img](wiz:/088b68f0-2099-11e9-8f40-8f77bbfda04f/088ffcd0-2099-11e9-8f40-8f77bbfda04f/1e559d72-ac10-4019-8c69-ca3dcbf811e6/index_files/bff8a016-7c86-4c9a-be37-b95e31b00605.png)

通过openjdk.jol可以打印对象头的信息

```xml
<dependency> 
    <groupId>org.openjdk.jol</groupId>
    <artifactId>jol‐core</artifactId>
    <version>0.9</version>
</dependency>
```

## 轻量级锁

 轻量级锁尝试在应用层面解决线程同步问题，而不触发操作系统的互斥操 作，轻量级锁减少多线程进入互斥的几率，不能代替互斥



## ReentrantLock

```java
    public ReentrantLock() {
        sync = new NonfairSync();	//默认非公平锁
    }

    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();   // 是否使用公平锁
    }
```

```java
public void lock() {
        sync.lock();  //根据ReentrantLock的Sync实现绝对调用公平与非公平锁的方法
    }
```

## Node

```java
       Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }
```



## 公平锁FairSync

### acquire 获取

```java
//此处进入Sync的实现类,调用lock方法
final void lock() {
        acquire(1); // 获取锁
    }
//调用父类 AQS 的方法
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&   //尝试加锁, ->> 1.1
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))  //尝试入队 ->> 1.2 1.3
        selfInterrupt();
}
```

### tryAcquire   获取锁

```java
	//尝试加锁1.1
    protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
    }
	//公平锁进行加锁逻辑,此处参数acquires并没有实际意义
	protected final boolean tryAcquire(int acquires) {
        	//定义当前线程变量
            final Thread current = Thread.currentThread();
        	//获取锁状态
        	// 情况1: 线程1 无竞争 此处state为默认值0 
        	// 情况2: 线程2 且线程1已经持有锁,此处状态为-1, 判断是否可重入
            int c = getState();
        	// 情况1: 锁状态为0 则锁为free状态,可获取
            if (c == 0) {
                //hasQueuedPredecessors 自己是否需要排队? 
                //情况1 : 返回fasle取反true 当前队列没有人排队,自己不需要排队 尝试CAS获取锁->0,1
                //	CAS 成功-> setExclusiveOwnerThread(current) 设置当前线程拥有锁
                //	CAS 失败-> 返回false
                //情况2 : 返回false取反true 当前队列自己就是排队在最前面的线程, 尝试CAS获取锁->0,1
                //	CAS 成功-> setExclusiveOwnerThread(current) 设置当前线程拥有锁
                //	CAS 失败-> 返回false
                //情况3 : 返回 true取反fasle,自己需要去排队 && 短路返回fasle
                //情况4 : 特殊情况返回true取反false, 队列已经初始化但是没有next元素说明另一个队列可能正在进行入队或出队操作 TODO
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    //成功获取到锁 返回true
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
        	// 情况2 : 锁状态为 -1,已经被线程占有,判断拥有锁的线程是不是自身,是的话直接重入锁,并且重入次数+1
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                //成功获取到锁 返回true
                return true;
            }
        	// 自己需要排队(不管state==0 / == -1) 或 CAS获取锁失败, 返回false
            return false;
        }

```

### 	hasQueuedPredecessors   是否需要排队

```java
//自己是否需要排队
public final boolean hasQueuedPredecessors() {
    // The correctness of this depends on head being initialized
    // before tail and on head.next being accurate if the current
    // thread is first in queue.
    //队列头
    Node t = tail; // Read fields in reverse initialization order
    //队列尾
    Node h = head;
    //空节点变量
    Node s;
    // h != t 头节点等于尾节点?    ->
    // (s = h.next) == null 头节点的next为空队列中只有一个节点 ?    ->
    // s.thread != Thread.currentThread() 头节点的next节点不是当前节点,就是不是排队最先的那个节点
    // 此处 通过 && 与 || 的短路特性能够计算出一共会发生4种情况
    // 情况1 : h != t -> fasle : 队列没有初始化,因为null==null,当前线程前面没人 + 锁可获取state=0,此时短路返回 fasle
    // 情况2 : h != t(true)  && ((s = h.next) == null(false) || s.thread != Thread.currentThread() (fasle) : 队列已经初始化,队列不止一个节点,自己就是队列中的第二个节点, 返回 false
    // 情况3 : h != t(true)  && ((s = h.next) == null(false) || s.thread != Thread.currentThread() (true) : 队列已经初始化,队列中不止一个元素,并且自己不是排队在第二个的元素, 返回 true
    // 情况4 : h != t(true)  && ((s = h.next) == null(true) : : 队列已经初始化,但是头结点的next节点为空,这种情况比较特殊返回true,TODO标记此处为Q1
    return h != t  &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

### setExclusiveOwnerThread

```java
    //成功获取到了锁, 将拥有锁的线程变为当前线程
	protected final void setExclusiveOwnerThread(Thread thread) {
        exclusiveOwnerThread = thread;
    }
```

### addWaiter   构建等待节点

```java
// 入队操作分为两个方法 acquireQueued(addWaiter (Node.EXCLUSIVE), arg))
// 方法 1: addWaiter  
//Creates and enqueues node for current thread and given mode.为当前线程和给定模式创建并排队节点。
//Params: 参数
//mode – Node.EXCLUSIVE for exclusive, Node.SHARED for shared模式– Node.EXCLUSIVE用于独占，Node.SHARED用于共享
//Returns: 返回值
//the new node 新节点
//此方法参数 mode = Node.EXCLUSIVE,独占锁的模式
//进入到此方法 说明当前线程需要来排队获取锁
    private Node addWaiter(Node mode) {
        //创建一个当前线程的独占模式节点 node
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        //定义一个变量,为尾节点,队列未初始化时尾节点为null
        Node pred = tail;
        //如果尾节点不是null,说明有上一个节点,已经初始化过了队列
        if (pred != null) {
            //那么当前线程的上一个节点就是当前的尾节点
            node.prev = pred;
            //然后CAS设置尾节点为当前线程节点,因为队列已经初始化,所以此处只需要CAS设置尾节点即可
            if (compareAndSetTail(pred, node)) {
                //维护链表,讲上一个节点的next指向当前节点
                pred.next = node;
                //返回当前节点
                return node;
            }
        }
        //队列没有进行初始化,传递当前节点到enq方法
        enq(node);
        //返回当前节点
        return node;
    }

```

### enq

```java
    //队列尚未初始化,进行队列初始化.参数为当前需要入队的node线程节点
	private Node enq(final Node node) {
        //此处死循环 
        //执行第一次 tail=null - > 执行if分支
        //执行第二次 if分支执行完 tail!=null 执行else分支
        for (;;) {
            //定义尾节点,第一次为null, 第二次为头结点信息
            Node t = tail;
            //执行第一次 : 尾节点为null  必须进行初始化
            if (t == null) { // Must initialize 
                //初始化队列 CAS设置队列 头节点 为一个 null 节点, 作为哨兵节点,此节点没有线程,其他信息都有
                //哨兵节点存在此处的一个很重要因素是存在此处的一个释放锁的状态waitStatus,此状态在当前线程执行完释放锁时会判断,如果等于-1,则需要叫醒下一个线程,如果为0则说明没有下一个在排队的线程不要要unpack()
                if (compareAndSetHead(new Node()))
                    //此处并不是一个原子操作,在锁已经被线程1占有的情况下,线程2初始化队列执行到此,失去CPU执行权,然后线程1释放了锁,此时新进入一个线程3执行到标记Q1时,便会形成特殊情况state==0, h != t(true)  && ((s = h.next) == null(true) : : 队列已经初始化,但是头结点的next节点为空,这种情况比较特殊返回true, 这种情况下线程3仍然会返回true告知需要排队;
          //尾节点为头节点,并不会结束循环,再次判断
                    tail = head;
            // 第二次循环 进入else分支
            } else {
                //定义node节点的上一个节点为头结点
                node.prev = t;
                //CAS设置尾节点为当前节点
                if (compareAndSetTail(t, node)) {
                    //头节点null的next节点为当前节点
                    t.next = node;
                    //结束循环,返回尾节点
                    return t;
                }
            }
        }
    }

```

### acquireQueued  入队

```java
//入队操作1.3 以排他的不间断模式获取已在队列中的线程。用于条件等待方法以及获取。 此处还会自旋一次获取锁,前一个线程可能已经释放锁了(交替执行)
//参数为当前入队的线程节点,和参数1
 final boolean acquireQueued(final Node node, int arg) {
     	//失败标记
        boolean failed = true;
        try {
            //线程是否被打断 默认fasle, 且lock方法打断是无效的
            boolean interrupted = false;
            //死循环,第一次修改pred的status,第二次进行pack
            for (;;) {
                //获取当前入队节点的上一个节点
                final Node p = node.predecessor();
                //循环1 : 如果上一个节点是头节点, 则尝试自旋一次获取锁  (此处是第1次入队前自旋) 如果获取成功则执行逻辑, 失败则执行第二个if
                //循环2 : 已经修改了pred的status,因为不是原子操作,再次尝试自旋CAS获取锁(第二次入队前自旋),成功则进入逻辑,否则进入第二个if分支
                //循环3 : 休眠的线程2被唤醒,再次循环执行到此处,CAS获取锁(唤醒后获取锁)
                if (p == head && tryAcquire(arg)) {
                    //获取到锁,维护链表,设置当前节点为头结点,并将头节点引用全部清空方便GC
                    //当线程2被唤醒后,从方法返回便会获取到锁到此维护队列,将当前node与head关联取消,并将当前节点的线程置为null设置为新的head哨兵节点
                    setHead(node);
                    p.next = null; // help GC
                    //标记位,成功获取到锁后修改不执行cancelAcquire()
                    failed = false;
                    //返回fasle
                    return interrupted;
                }
                //获取线程失败,或者不是排在第二个的节点时 应该入队了
                //循环1->情况1 shouldParkAfterFailedAcquire()返回false,当前线程已经修改了pred线程的waitStatus, 逻辑&&短路
                //循环2->情况2 shouldParkAfterFailedAcquire()返回if分支1的true,执行parkAndCheckInterrupt()->pack 正式阻塞
                if (shouldParkAfterFailedAcquire(p, node) && 
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
            
        } finally {
            if (failed) //未出现错误不会执行到此处, failed=false 方法返回 线程返回开始执行逻辑
                //如果没有成功获取到锁却发生异常跳出了循环,执行此处代码
                cancelAcquire(node);
        }
    }
```

### shouldParkAfterFailedAcquire   修改唤醒状态

```java
//入队操作1.4
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus; // pred.waitStatus默认值 =0
        if (ws == Node.SIGNAL) // Node.SIGNAL=-1, 当前线程的上一个节点已经设置状态了(一般循环调用此方法第二次时执行到此)
            /*
             * This node has already set status asking a release 该节点已设置状态，要求发布
             * to signal it, so it can safely park.发出信号，以便可以安全停车。
             */
            return true;
        if (ws > 0) {
            /* 前一个节点是被取消的节点,跳过前一个节点进行排队,并且进行重试, 此时前任节点的status = 1 = Node.CANCELLED
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry. 重试
             */
            do {
                //不断尝试前面最近的一个未被取消的排队线程node,并将其next关联到此节点
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            //关联 最近的未被取消节点到当前节点
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we 
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            //当前线程CAS修改 pred节点的waitStatus, 告知上个节点释放锁需要叫醒我
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
    	//执行了第二种或第三种情况,返回fasle
        return false;
    }

```

### parkAndCheckInterrupt   开始pack

```java
    //线程开始pack,醒来之后会跳回acquireQueue维护队列
	private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```



### cancelAcquire

```java
//异常跳出死循环   TODO
private void cancelAcquire(Node node) {
    // Ignore if node doesn't exist
    //非空判断
    if (node == null)
        return;
	//讲当前节点线程置为null
    node.thread = null;

    // Skip cancelled predecessors
    Node pred = node.prev;
    //如果当前node的pred.status>0 说明前任节点是取消排队节点,需要跳过前任节点,循环寻找最近的未取消的前任节点关联其.next = node
    while (pred.waitStatus > 0)
        //pred = pred.pred
        //node.pred = pred
        node.prev = pred = pred.prev;

    // predNext is the apparent node to unsplice. CASes below will
    // fail if not, in which case, we lost race vs another cancel
    // or signal, so no further action is necessary.
    Node predNext = pred.next;

    // Can use unconditional write instead of CAS here.
    // After this atomic step, other Nodes can skip past us.
    // Before, we are free of interference from other threads.
    // 当前节点的状态=取消=1
    node.waitStatus = Node.CANCELLED;

    // If we are the tail, remove ourselves. 如果当前节点是尾节点则移除当前节点,将尾节点设置尾最近未取消的pred
    if (node == tail && compareAndSetTail(node, pred)) {
        //将尾节点的next设置为null
        compareAndSetNext(pred, predNext, null);
    } else {
        // If successor needs signal, try to set pred's next-link
        // so it will get one. Otherwise wake it up to propagate.
        int ws;
        if (pred != head &&
            ((ws = pred.waitStatus) == Node.SIGNAL ||
             (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
            pred.thread != null) {
            Node next = node.next;
            if (next != null && next.waitStatus <= 0)
                compareAndSetNext(pred, predNext, next);
        } else {
            unparkSuccessor(node);
        }

        node.next = node; // help GC
    }
}
```

### 释放锁

### release

```java
//释放锁,此方法ReentrantLock调用sync的release实现
public void unlock() {
    sync.release(1);
}
//释放锁
public final boolean release(int arg) {
    //尝试释放锁, 
    //情况1: 释放成功返回true, 情况2:释放锁的线程没有锁,会抛出异常结束程序
    if (tryRelease(arg)) {  
        //维护链表,定义变量
        Node h = head;
        //如果头节点不是null(说明存在链表), 并且头节点的waitStatus不是0(说明有人在排队,需要被unpack)
        if (h != null && h.waitStatus != 0)
            //unpack队列中的下一个线程,传递头结点
            unparkSuccessor(h);
        //返回值对方法的调用者没有影响
        //返回true 
        return true;
    }
    //返回false
    return false;
}
```

### tryRelease

```java
//实现类ReentrantLock
//参数 1
protected final boolean tryRelease(int releases) {
    		//state 锁状态,被持有为1,未持有为0
            int c = getState() - releases;
    		//如果当前线程,不是持有锁的线程 抛出异常,无法释放锁
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
    	 	//当前锁状态为已持有
            boolean free = false;
    		//如果c==0 说明锁被拥有 可以被释放
            if (c == 0) {
                //锁状态未持有
                free = true;
                //设置拥有锁线程为null
                setExclusiveOwnerThread(null);
            }
    		//c=0 , 设置锁装维为0
            setState(c);
    		//返回true,释放完成
            return free;
        }

//设置锁状态未0
protected final void setState(int newState) {
    state = newState;
}
```

### unparkSuccessor

```java
    //唤醒下个等待锁的线程
	//参数 头节点node
	private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        //头结点的状态 < 0 说明有线程在休眠
        int ws = node.waitStatus;
        if (ws < 0)
            //CAS 修改头节点的waitStatus为0
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        //取出头节点的next节点
        Node s = node.next;
        //情况1 : 如果next节点是null,说明头节点的下一个节点取消了
        //情况2 : 其waitStatus>0 也说明下一个节点取消了
        if (s == null || s.waitStatus > 0) {
            //把next节点置空
            s = null;
            //从尾节点开始向上找,找到离head最近的一个waitStatus<=0 即在排队的线程节点进行唤醒
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    //把最近的节点赋值给变量head.next
                    s = t;
        }
        //如果存在这个head最近的这个等待被唤醒节点,则唤醒该节点
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```



# CAS 

**CAS (Compare-And-Swap)** 是一种硬件对并发的支持，针对多处理器操作而设计的处理器中的一种特殊指令，用于管理对共享数据的并发访问

CAS 是一种**无锁的非阻塞算法**的实现

CAS 包含了3 个操作数：

1. 需要读写的内存值V
2. 进行比较的值A 
3. 拟写入的新值B

当且**仅当V 的值等于A 时，CAS 通过原子方式用新值B 来更新V 的值，否则不会执行任何操作**

### 原子操作问题

​	通过jdk 1.5提供的**Atomic包提供的工具类**可以实现 comparaAndSwap的原子操作即可

### ABA问题

​	通过增加一个**version版本号**来控制数据的版本,即可区分ABA问题是否发生从而解决, 但是如果数据的ABA对结果并无影响也可以不用处理;

## ReentrantLock

### 简单Atomic+自旋实现

```
//简单实现
private Integer status = 0 ;
AtomicInteger atomic = new AtomicInteger()
public boolean lock(){

	while(!atomic.comparaAndSet(0,1)){  //如果修改不成功 !false 进入死循环无法继续执行 调用lock方法处以下的代码; 这种方法cpu大量空转浪费资源;
	}
	return true;
}
```

### pack+自旋实现

```

```

