# 	java并发编程

## 一. 锁(Lock)

### **1. ReentrantLock介绍**

​	ReentrantLock是一个可重入的互斥锁，又被称为“独占锁”。

​	顾名思义，ReentrantLock锁在同一个时间点只能被一个线程锁持有；而可重入的意思是，ReentrantLock锁，可以被单个线程多次获取。
​	ReentrantLock分为“**公平锁**”和“**非公平锁**”。它们的区别体现在获取锁的机制上是否公平。“锁”是为了保护竞争资源，防止多个线程同时操作线程而出错，ReentrantLock在同一个时间点只能被一个线程获取(当某线程获取到“锁”时，其它线程就必须等待)；ReentraantLock是通过一个FIFO的等待队列来管理获取该锁所有线程的。在“公平锁”的机制下，线程依次排队获取锁；而“非公平锁”在锁是可获取状态时，不管自己是不是在队列的开头都会获取锁。

```java
class Depot { 
    private int size;        // 仓库的实际数量
    private Lock lock;        // 独占锁

    public Depot() {
        this.size = 0;
        this.lock = new ReentrantLock();
    }
    public void produce(int val) {
        lock.lock();
        try {
            size += val;
            System.out.printf("%s produce(%d) --> size=%d\n", 
                    Thread.currentThread().getName(), val, size);
        } finally {
            lock.unlock();
        }
    }
}
```

```java
	private int capacity;    // 仓库的容量
	private int size;   	// 当前资源数量
	private Lock lock;        // 独占锁
	private Condition fullCondtion;            // 生产条件
	private Condition emptyCondtion;        // 消费条件

public void produce(int val) {
        lock.lock();
        try {
             // left 表示“想要生产的数量”(有可能生产量太多，需多此生产)
            int left = val;
            while (left > 0) {
                // 库存已满时，等待“消费者”消费产品。
                while (size >= capacity)
                    fullCondtion.await();
                // 获取“实际生产的数量”(即库存中新增的数量)
                // 如果“库存”+“想要生产的数量”>“总的容量”，则“实际增量”=“总的容量”-“当前容量”。(此时填满仓库)
                // 否则“实际增量”=“想要生产的数量”
                int inc = (size+left)>capacity ? (capacity-size) : left;
                size += inc;
                left -= inc;
                System.out.printf("%s produce(%3d) --> left=%3d, inc=%3d, size=%3d\n", 
                        Thread.currentThread().getName(), val, left, inc, size);
                // 通知“消费者”可以消费了。
                emptyCondtion.signal();
            }
        } catch (InterruptedException e) {
        } finally {
            lock.unlock();
        }
    }

```



### 2. 公平锁

#### **基本概念**

1. **AQS **-- 指AbstractQueuedSynchronizer类。

​    AQS是java中管理“锁”的抽象类，锁的许多公共方法都是在这个类中实现。AQS是独占锁(例如，ReentrantLock)和共享锁(例如，Semaphore)的公共父类。

2. **AQS**锁的类别 -- 分为“**独占锁**”和“**共享锁**”两种。

​    (01) **独占锁** -- 锁在一个时间点只能被一个线程锁占有。根据锁的获取机制，它又划分为“**公平锁**”和“**非公平锁**”。公平锁，是按照通过CLH等待线程按照先来先得的规则，公平的获取锁；而非公平锁，则当线程要获取锁时，它会无视CLH等待队列而直接获取锁。独占锁的典型实例子是ReentrantLock，此外，ReentrantReadWriteLock.WriteLock也是独占锁。
​    (02) **共享锁** -- 能被多个线程同时拥有，能被共享的锁。JUC包中的ReentrantReadWriteLock.ReadLock，CyclicBarrier， CountDownLatch和Semaphore都是共享锁。这些锁的用途和原理，在以后的章节再详细介绍。

3. **CLH队列** -- Craig, Landin, and Hagersten lock queue

​    CLH队列是AQS中“**等待锁”的线程队列**。在多线程中，为了保护竞争资源不被多个线程同时操作而起来错误，我们常常需要通过锁来保护这些资源。在独占锁中，竞争资源在一个时间点只能被一个线程锁访问；而其它线程则需要等待。CLH就是管理这些“等待锁”的线程的队列。
​    CLH是一个**非阻塞**的 FIFO 队列。也就是说往里面插入或移除一个节点的时候，在并发条件下不会阻塞，而是通过**自旋锁**和 **CAS** 保证节点插入和移除的原子性。

4. **CAS函数** -- Compare And Swap 

​    CAS函数，是比较并交换函数，它是**原子操作函数**；即，通过CAS操作的数据都是以**原子方式**进行的。例如，compareAndSetHead(), compareAndSetTail(), compareAndSetNext()等函数。它们共同的特点是，这些函数所执行的动作是以原子的方式进行的。



#### ReentrantLock数据结构

ReentrantLock的UML类图

[![img](https://images0.cnblogs.com/blog/497634/201401/271417467039316.jpg)](https://images0.cnblogs.com/blog/497634/201401/271417467039316.jpg)

从图中可以看出：
(01) ReentrantLock实现了Lock接口。
(02) ReentrantLock与sync是组合关系。ReentrantLock中，包含了Sync对象；而且，Sync是AQS的子类；更重要的是，Sync有两个子类FairSync(公平锁)和NonFairSync(非公平锁)。ReentrantLock是一个独占锁，至于它到底是公平锁还是非公平锁，就取决于sync对象是"FairSync的实例"还是"NonFairSync的实例"。



#### **获取公平锁(基于JDK1.7.0_40)**

##### **1. lock()**

lock()在ReentrantLock.java的FairSync类中实现，它的源码如下：

```
final void lock() {
    acquire(1);
}
```

**说明**：“当前线程”实际上是通过acquire(1)获取锁的。
​        这里说明一下“**1**”的含义，它是设置“锁的状态”的参数。对于“独占锁”而言，锁处于可获取状态时，它的状态值是0；锁被线程初次获取到了，它的状态值就变成了1。
​        由于ReentrantLock(公平锁/非公平锁)是可重入锁，所以“独占锁”可以被单个线程多此获取，每获取1次就将锁的状态+1。也就是说，初次获取锁时，通过acquire(1)将锁的状态值设为1；再次获取锁时，将锁的状态值设为2；依次类推...这就是为什么获取锁时，传入的参数是1的原因了。
​        可重入就是指锁可以被单个线程多次获取。

##### **2. acquire()**

acquire()在AQS中实现的，它的源码如下：

```
public final void acquire(int arg) {
    if (!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

​	(01) “当前线程”首先通过tryAcquire()尝试获取锁。获取成功的话，直接返回；尝试失败的话，进入到等待队列排序等待(前面还有可能有需要线程在等待该锁)。
​	(02) “当前线程”尝试失败的情况下，先通过addWaiter(Node.EXCLUSIVE)来将“当前线程”加入到"CLH队列(非阻塞的FIFO队列)"末尾。CLH队列就是线程等待队列。
​	(03) 再执行完addWaiter(Node.EXCLUSIVE)之后，会调用acquireQueued()来获取锁。由于此时ReentrantLock是公平锁，它会**根据公平性原则**来获取锁。
​	(04) “当前线程”在执行acquireQueued()时，会进入到CLH队列中休眠等待，直到获取锁了才返回！如果“当前线程”在休眠等待过程中被中断过，acquireQueued会返回true，此时"当前线程"会调用selfInterrupt()来自己给自己产生一个中断。至于为什么要自己给自己产生一个中断，后面再介绍。

上面是对acquire()的概括性说明。下面，我们将该函数分为4部分来逐步解析。
**一. tryAcquire()**
**二. addWaiter()**
**三. acquireQueued()**
**四. selfInterrupt()**

**1. tryAcquire()**

公平锁的tryAcquire()在ReentrantLock.java的**FairSync**类中实现，源码如下：

```java
protected final boolean tryAcquire(int acquires) {
    // 获取“当前线程”
    final Thread current = Thread.currentThread();
    // 获取“独占锁”的状态
    int c = getState();
    // c=0意味着“锁没有被任何线程锁拥有”，
    if (c == 0) {
        // 若“锁没有被任何线程锁拥有”，
        // 则判断“当前线程”是不是CLH队列中的第一个线程线程，
        // 若是的话，则获取该锁，设置锁的状态，并切设置锁的拥有者为“当前线程”。
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        // 如果“独占锁”的拥有者已经为“当前线程”，
        // 则将更新锁的状态。
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

**说明**：根据代码，我们可以分析出，tryAcquire()的作用就是尝试去获取锁。注意，这里只是尝试！
​         尝试成功的话，返回true；尝试失败的话，返回false，后续再通过其它办法来获取该锁。后面我们会说明，在尝试失败的情况下，是如何一步步获取锁的。

**2. hasQueuedPredecessors()**

hasQueuedPredecessors()在AQS中实现，源码如下：

```
public final boolean hasQueuedPredecessors() {
    Node t = tail; 
    Node h = head;
    Node s;
    return h != t &&
        ((s = h.next) == null || s.thread != Thread.currentThread());
}
```

**说明**： 通过代码，能分析出，hasQueuedPredecessors() 是通过判断"当前线程"是不是在CLH队列的队首，来返回AQS中是不是有比“当前线程”等待更久的线程。下面对head、tail和Node进行说明。



##### **3. Node的源码**

Node就是CLH队列的节点。Node在AQS中实现，它的数据结构如下：

```java
private transient volatile Node head;    // CLH队列的队首
private transient volatile Node tail;    // CLH队列的队尾

// CLH队列的节点
static final class Node {
    static final Node SHARED = new Node();
    static final Node EXCLUSIVE = null;

    // 线程已被取消，对应的waitStatus的值
    static final int CANCELLED =  1;
    // “当前线程的后继线程需要被unpark(唤醒)”，对应的waitStatus的值。
    // 一般发生情况是：当前线程的后继线程处于阻塞状态，而当前线程被release或cancel掉，因此需要唤醒当前线程的后继线程。
    static final int SIGNAL    = -1;
    // 线程(处在Condition休眠状态)在等待Condition唤醒，对应的waitStatus的值
    static final int CONDITION = -2;
    // (共享锁)其它线程获取到“共享锁”，对应的waitStatus的值
    static final int PROPAGATE = -3;

    // waitStatus为“CANCELLED, SIGNAL, CONDITION, PROPAGATE”时分别表示不同状态，
    // 若waitStatus=0，则意味着当前线程不属于上面的任何一种状态。
    volatile int waitStatus;

    // 前一节点
    volatile Node prev;

    // 后一节点
    volatile Node next;

    // 节点所对应的线程
    volatile Thread thread;

    // nextWaiter是“区别当前CLH队列是 ‘独占锁’队列 还是 ‘共享锁’队列 的标记”
    // 若nextWaiter=SHARED，则CLH队列是“独占锁”队列；
    // 若nextWaiter=EXCLUSIVE，(即nextWaiter=null)，则CLH队列是“共享锁”队列。
    Node nextWaiter;

    // “共享锁”则返回true，“独占锁”则返回false。
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    // 返回前一节点
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {    // Used to establish initial head or SHARED marker
    }

    // 构造函数。thread是节点所对应的线程，mode是用来表示thread的锁是“独占锁”还是“共享锁”。
    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    // 构造函数。thread是节点所对应的线程，waitStatus是线程的等待状态。
    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

**说明**：
Node是CLH队列的节点，代表“等待锁的**线程队列**”。
(01) 每个Node都会一个线程对应。
(02) 每个Node会通过prev和next分别指向上一个节点和下一个节点，这分别代表上一个等待线程和下一个等待线程。
(03) Node通过**waitStatus**保存线程的**等待状态**。
(04) Node通过**nextWaiter**来区分线程是**“独占锁”线程还是“共享锁”线程**。如果是“独占锁”线程，则nextWaiter的值为EXCLUSIVE；如果是“共享锁”线程，则nextWaiter的值是SHARED。



##### **4. compareAndSetState()**

compareAndSetState()在AQS中实现。它的源码如下：

```
protected final boolean compareAndSetState(int expect, int update) {
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

**说明**： compareAndSwapInt() 是sun.misc.Unsafe类中的一个本地方法。对此，我们需要了解的是 compareAndSetState(expect, update) 是以原子的方式操作当前线程；若当前线程的状态为expect，则设置它的状态为update。

##### **5. setExclusiveOwnerThread()**

setExclusiveOwnerThread()在AbstractOwnableSynchronizer.java中实现，它的源码如下：

```
// exclusiveOwnerThread是当前拥有“独占锁”的线程
private transient Thread exclusiveOwnerThread;
protected final void setExclusiveOwnerThread(Thread t) {
    exclusiveOwnerThread = t;
}
```

**说明**：setExclusiveOwnerThread()的作用就是，设置线程t为当前拥有“独占锁”的线程。

##### **6. getState(), setState()**

getState()和setState()都在AQS中实现，源码如下：

```
// 锁的状态
private volatile int state;
// 设置锁的状态
protected final void setState(int newState) {
    state = newState;
}
// 获取锁的状态
protected final int getState() {
    return state;
}
```

**说明**：state表示锁的状态，对于“独占锁”而已，state=0表示锁是可获取状态(即，锁没有被任何线程锁持有)。由于java中的独占锁是可重入的，state的值可以>1。

**小结**：tryAcquire()的作用就是让“当前线程”尝试获取锁。获取成功返回true，失败则返回false。