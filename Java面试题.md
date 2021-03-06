## Java面试题

### 1. 分代回收算法,对象如何进入老年代?

​	新生代: 对象朝生夕死,存活时间短,创建较频繁,采用"复制"回收算法,新生代内存分为"Eden"+"Survivor From"+"Survivor To",对象分配在eden区,满了之后进行MinorGC,将Eden和Survivor中存活的对象复制到Survivor To区,然后清除Eden和Survivor From

​	经过一次MinorGC,存活的对象年龄+1,当达到一定的阈值之后(默认15),进入老年代;

​	老年代:  

​	"标记-清除":	首先标记出所有需要回收的对象，这一过程在可达性分析过程中进行。在标记完之后统一回收所有被标记的对象。清除之后会产生大量不连续的内存碎片，内存碎片太多会导致以后的程序运行中无法分配出较大的内存，从内不得不触发另外的垃圾回收。

​	"标记-整理":	标记过程仍与”标记-清除”过程一致，但后续步骤不是直接对可回收对象进行清理，而是让所有存活对象都向一端移动，然后直接清理掉端边界以外的内存。



### 2. lock ，sychronized，volatile的区别

​	**volatile:** 

​	轻量级同步机制,主要是保证变量对多线程的**"可见性"**,多线程共享进程的内存资源,也有线程私有的部分,称之为本地内存与共享内存,也就是说,在对volatile变量进行写操作之后,jvm会在该操作后面加上一条指令,将变量立即写回共享内存,该操作也会导致其他cpu中对应的缓存行无效;而在线程读取volatile变量的时候,即时本地内存有该变量的缓存,也会去共享内存中重新获取;这也就保证了"可见性";	

​	除了可见性,volatile关键字还有另一层内存语义--**"有序性"**,lock前缀指令,就是禁止指令重排,相当于"内存屏障",屏障后面的指令不会重排到屏障之前,同理,之前的也不会重排到之后,

​	**Synchronized:**

​	类似于可重入的互斥锁,底层使用的是**MonitorEnter和MonitorExit**,编译时会在同步块前后分别形成MonitorEnter和MonitorExit指令,执行MonitorEnter指令时,首先尝试获取对象锁,如果对象没被锁或者当前线程已经拥有这个对象锁,则把锁的计数器+1,相应的,在执行MonitorExit的时候,将锁的计数器-1,当计数器为0的时候,锁就被释放了,如果获取对象锁失败,那么当前线程就要阻塞,知道对象锁被释放为止;

​	**Lock:**

​	Lock是接口,Synchronized在发生异常时,会自动释放占有的锁资源,因此不会发生死锁现象,而Lock,如果没有主动unlock()去释放锁,则很可能造成死锁,

​	Synchronized不需要手动释放锁,而Lock加锁和释放锁都需要手动指定,并且为了确保锁能被成功释放,锁的释放语句一般是写在finally代码块里;

​	Lock可以让等待锁的线程能够**响应中断,** 并且Lock可以知道有没有成功获取锁资源,也可以tryLock非阻塞式的尝试获取锁;

​	Lock如果是读写锁的话能提高多线程进行读写操作的效率;

### 3. 乐观锁，悲观锁?乐观锁使用场景，使用乐观锁的产品或中间件

​	**悲观锁**：传统的**关系型数据库，行锁,表锁,读写锁等**，**Synchronized**

​	总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。传统的**关系型数据库**里边就用到了很多这种锁机制，比如**行锁，表锁等，读锁，写锁**等，都是在做操作之前先上锁。再比如Java里面的同步原语**synchronized**关键字的实现也是悲观锁。

　　**乐观锁**：**ElasticSearch，原子变量类**

​	顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用**版本号或时间戳**机制。乐观锁**适用于多读**的应用类型，这样可以**提高吞吐量**，像数据库提供的类似于write_condition机制，**ElasticSearch**其实都是提供的乐观锁。在Java中java.util.concurrent.atomic包下面的**原子变量类**就是使用了乐观锁的一种实现方式**CAS**实现的。

​		

### **4. Innodb中的行锁与表锁**

​	在Innodb引擎中既支持行锁也支持表锁，那么什么时候会锁住整张表，什么时候或只锁住一行呢？
只有通过**索引**条件检索数据，InnoDB才使用行级锁，否则，InnoDB将使用表锁！

​	在实际应用中，要特别注意InnoDB行锁的这一特性，不然的话，可能导致大量的锁冲突，从而影响并发性能。

​	行级锁都是基于索引的，**如果一条SQL语句用不到索引是不会使用行级锁的**，会使用表级锁。行级锁的缺点是：由于需要请求大量的锁资源，所以速度慢，内存消耗大。



### 5. 分布式锁的几种实现方式

​	分布式的**CAP理论**告诉我们“任何一个分布式系统都无法同时满足**一致性**（Consistency）、**可用性**（Availability）和**分区容错性**（Partition tolerance），最多只能同时满足两项。所以，很多系统在设计之初就要对这三者做出取舍。在互联网领域的绝大多数的场景中，都需要**牺牲强一致性来换取系统的高可用性**，系统往往只需要保证“最终一致性”，只要这个最终时间是在用户可以接受的范围内即可。

​	在很多场景中，我们为了保证数据的最终一致性，需要很多的技术方案来支持，比如分布式事务、分布式锁等。有的时候，我们需要保证一个方法在同一时间内只能被同一个线程执行。

**分布式锁应该具备哪些条件**

在分析分布式锁的三种实现方式之前，先了解一下分布式锁应该具备哪些条件：

​	1、在分布式系统环境下，一个方法在同一时间只能被一个机器的一个线程执行； 
​	2、**高可用**的获取锁与释放锁； 
​	3、**高性能**的获取锁与释放锁； 
​	4、具备**可重入**特性； 
​	5、具备**锁失效机制**，防止死锁； 
​	6、具备**非阻塞锁**特性，即没有获取到锁将直接返回获取锁失败。

分布式锁的几种实现方式:

​	1. 基于**数据库**实现分布式锁； 
​	2. 基于**缓存（Redis等）**实现分布式锁； 
​	3. 基于**Zookeeper**实现分布式锁；

#### 5.1 基于**数据库**实现分布式锁

​	基于数据库的实现方式的核心思想是：在数据库中创建一个表，表中包含**方法名**等字段，并在**方法名字段上创建唯一索引**，想要执行某个方法，就使用这个方法名向表中插入数据，成功插入则获取锁，执行完成后删除对应的行数据释放锁。

​	使用基于数据库的这种实现方式很简单，但是对于分布式锁应该具备的条件来说，它有一些问题需要解决及优化：

```
1、因为是基于数据库实现的，数据库的可用性和性能将直接影响分布式锁的可用性及性能，所以，数据库需要双机部署、数据同步、主备切换；

2、不具备可重入的特性，因为同一个线程在释放锁之前，行数据一直存在，无法再次成功插入数据，所以，需要在表中新增一列，用于记录当前获取到锁的机器和线程信息，在再次获取锁的时候，先查询表中机器和线程信息是否和当前机器和线程相同，若相同则直接获取锁；

3、没有锁失效机制，因为有可能出现成功插入数据后，服务器宕机了，对应的数据没有被删除，当服务恢复后一直获取不到锁，所以，需要在表中新增一列，用于记录失效时间，并且需要有定时任务清除这些失效的数据；

4、不具备阻塞锁特性，获取不到锁直接返回失败，所以需要优化获取逻辑，循环多次去获取。

5、在实施的过程中会遇到各种不同的问题，为了解决这些问题，实现方式将会越来越复杂；依赖数据库需要一定的资源开销，性能问题需要考虑。
```



#### 5.2 基于**缓存（Redis等）**实现分布式锁 

​	**1、选用Redis实现分布式锁原因：**

​	（1）Redis有很高的性能； 
​	（2）Redis命令对此支持较好，实现起来比较方便

​	**2、使用命令介绍：**

​	（1）SETNX：SETNX key val：当且仅当key不存在时，set一个key为val的字符串，返回1；若key存在，则什么都不做，返回0。

​	（2）expire：expire key timeout：为key设置一个超时时间，单位为second，超过这个时间锁会自动释放，避免死锁。

​	（3）delete：delete key：删除key

在使用Redis实现分布式锁的时候，主要就会使用到这三个命令。

​	**3、实现思想：**

​	（1）获取锁的时候，使用setnx加锁，并使用expire命令为锁添加一个超时时间，超过该时间则自动释放锁，锁的value值为一个随机生成的UUID，通过此在释放锁的时候进行判断。

​	（2）获取锁的时候还设置一个获取的超时时间，若超过这个时间则放弃获取锁。

​	（3）释放锁的时候，通过UUID判断是不是该锁，若是该锁，则执行delete进行锁释放。



#### 5.3 基于**Zookeeper**实现分布式锁

​	ZooKeeper是一个为分布式应用提供**一致性服务**的开源组件，它内部是一个**分层的文件系统目录树**结构，规定同一个目录下只能有一个唯一文件名。基于ZooKeeper实现分布式锁的步骤如下：

（1）创建一个目录mylock； 
（2）线程A想获取锁就在mylock目录下创建临时顺序节点； 
（3）获取mylock目录下所有的子节点，然后获取比自己小的兄弟节点，如果不存在，则说明当前线程顺序号最小，获得锁； 
（4）线程B获取所有节点，判断自己不是最小节点，设置监听比自己次小的节点； 
（5）线程A处理完，删除自己的节点，线程B监听到变更事件，判断自己是不是最小的节点，如果是则获得锁。

​	这里推荐一个Apache的开源库Curator，它是一个ZooKeeper客户端，Curator提供的InterProcessMutex是分布式锁的实现，acquire方法用于获取锁，release方法用于释放锁。

**优点：具备高可用、可重入、阻塞锁特性，可解决失效死锁问题。**

**缺点：因为需要频繁的创建和删除节点，性能上不如Redis方式。**



​	**总结**

​	上面的三种实现方式，没有在所有场合都是完美的，所以，应根据不同的应用场景选择最适合的实现方式。

​	在分布式环境中，对资源进行上锁有时候是很重要的，比如抢购某一资源，这时候使用分布式锁就可以很好地控制资源。 
​	当然，在具体使用中，还需要考虑很多因素，比如超时时间的选取，获取锁时间的选取对并发量都有很大的影响，上述实现的分布式锁也只是一种简单的实现，主要是一种思想.



### 6. 进程,线程之间的通信方式

(1)进程间的通信方式：

```
  管道(pipe)
  有名管道(named pipe)
  信号量(semophore)
  消息队列(message queue)
  信号(signal)
  共享内存(shared memory)
  套接字(socket)；
```

(2)线程程间的通信方式：

​	**1、锁机制**  如:**Object-->wait/notify  Lock.Condition--> await/signal**
​		1.1 互斥锁：提供了以排它方式阻止数据结构被并发修改的方法。 
​		1.2 读写锁：允许多个线程同时读共享数据，而对写操作互斥。 
​		1.3 条件变量：可以以原子的方式阻塞进程，直到某个特定条件为真为止。

对条件测试是在互斥锁的保护下进行的。条件变量始终与互斥锁一起使用。

​	**2、信号量机制**：包括无名线程信号量与有名线程信号量 
​	**3、信号机制**：类似于进程间的信号处理。

线程间通信的主要目的是用于线程同步，所以线程没有像进程通信中用于数据交换的通信机制。



### 7. ThreadLocal原理

​	

```
ThreadLocal.get(){
	// 流程
	// 1 使用当前线程 获取当前线程的成员变量 ThreadLocalMap
		// THreadLocalMap 是Entry<ThreadLocal,value>数组,类似HashMap,因为一个线程可能有多个ThreadLocal变量
	// 2 使用ThreadLocal作为键(key) 从ThreadLocalMap中获取到 Entry<ThreadLocal,value>
	// 3 return Entry<ThreadLocal,value>的value

	Thread t ==> ThreadLocalMap( Entry<ThreadLocal,value>[] table) ==> Entry<ThreadLocal,value> ==> value
} 

```

​	在ThreadLocal类中有一个**static声明的Map**，用于存储每一个线程的变量副本，**Map中元素的键为线程对象，而值对应线程的变量副本。**

​	首先，在每个线程Thread内部有一个ThreadLocal.ThreadLocalMap类型的成员变量**threadLocals**，这个threadLocals就是用来存储实际的变量副本的，键值为当前**ThreadLocal**变量，value为变量副本（即T类型的变量）。

　　初始时，在Thread里面，threadLocals为空，当通过ThreadLocal变量调用get()方法或者set()方法，就会对Thread类中的threadLocals进行初始化，并且以当前ThreadLocal变量为键值，以ThreadLocal要保存的副本变量为value，存到threadLocals。

　　然后在当前线程里面，如果要使用副本变量，就可以通过get方法在threadLocals里面查找。

```java
public T get() {
  Thread t = Thread.currentThread();
  ThreadLocalMap map = getMap(t);
  if (map != null) {
    ThreadLocalMap.Entry e = map.getEntry(this);
    if (e != null) {
      @SuppressWarnings("unchecked")
      T result = (T)e.value;
      return result;
    }
  }
  return setInitialValue();
}
ThreadLocalMap getMap(Thread t) {
  return t.threadLocals;
}
void createMap(Thread t, T firstValue) {
  t.threadLocals = new ThreadLocalMap(this, firstValue);
}
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
  table = new Entry[INITIAL_CAPACITY];
  int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
  table[i] = new Entry(firstKey, firstValue);
  size = 1;
  setThreshold(INITIAL_CAPACITY);
}
```



```
总结一下：

　　1）实际的通过ThreadLocal创建的副本是存储在每个线程自己的threadLocals中的；

　　2）为何threadLocals的类型ThreadLocalMap的键值为ThreadLocal对象，因为每个线程中可有多个threadLocal变量，就像上面代码中的longLocal和stringLocal；

　　3）在进行get之前，必须先set，否则会报空指针异常；

　　    如果想在get之前不需要调用set就能正常访问的话，必须重写initialValue()方法。

　　　 因为在上面的代码分析过程中，我们发现如果没有先set的话，即在map中查找不到对应的存储，则会通过调用setInitialValue方法返回i，而在setInitialValue方法中，有一个语句是T value = initialValue()， 而默认情况下，initialValue方法返回的是null。
```



### 8.  无锁,偏向锁,轻量级锁,重量级锁

#### 8.1 偏向锁

**1、偏向锁核心思想**

​	偏向锁是一种针对加锁操作的优化手段，他的核心思想是：**如果一个线程获得了锁，那么锁就进入了偏向模式。当这个线程再次请求锁时，无需再做任何同步操作**，这样就节省了大量有关锁申请的操作，从而提高了程序的性能。

**2、偏向锁设计初衷**

​	为什么会出现这种设计的方式那？这是因为根据HotSpot的作者研究，他发现锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低而引入了的偏向锁这个概念。

**3、偏向锁获取锁流程**

偏向锁获取锁流程如下：

​	（1）当一个线程访问同步块并获取锁时，会在**对象头和栈帧中的锁记录里存储锁偏向的线程ID**，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需简单地测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁；

​	（2）如果测试成功，表示线程已经获得了锁。如果测试失败，则需要再测试一下Mark Word中偏向锁的标识是否设置成1（表示当前是偏向锁）；

​	（3）如果没有设置，则使用CAS竞争锁；

​	（4）如果设置了，则尝试使用CAS将对象头的偏向锁指向当前线程。

4、**偏向锁升级为轻量锁**

​	对于只有一个线程访问的同步资源场景，锁的竞争不是很激烈，这时候使用偏向锁是一种很好的选择，因为连续多次极有可能是同一个线程请求相同的锁。

​	但是在锁竞争比较激烈的场景，最有可能的情况是每次不同的线程来请求相同的锁，这样的话偏向锁就会失效，倒不如不开启这种模式，幸运的是Java虚拟机提供了参数可以让我们有选择的设置是否开启偏向锁。

​	如果偏向锁失败，虚拟机并不会立即挂起线程，而是使用轻量级锁进行操作。



#### **8.2 轻量级锁**

​	如果偏向锁失败，虚拟机并不会立即挂起线程，而是使用轻量级锁进行操作。**轻量级锁他只是简单的将对象头部作为指针，指向持有锁的线程堆栈的内部，来判断一个线程是否持有对象锁**。如果线程获得轻量级锁成功，则可以顺利进入临界区。如果轻量级锁加锁失败，则表示其他线程抢先夺到锁，那么当前线程的轻量级锁就会膨胀为重量级锁。



#### **8.3 自旋锁**

​	轻量级锁就会膨胀为重量级锁后，虚拟机为了避免线程真实的在操作系统层面挂起，虚拟机还会在做最后的努力–**自旋锁**。

​	由于当前线程暂时无法获得锁，但是什么时候可以获得锁时一个未知数。也许在几个CPU时钟周期之后，就可以获得锁。如果是这样的话，直接把线程挂起肯定是一种得不偿失的选择，因此系统会在进行一次努力：他会假设在不就的将来，限额和从那个可以得到这把锁，因此虚拟机会让当前线程**做几个空循环**（这也就是自旋锁的意义），若经过几个空循环可以获取到锁则进入临界区，如果还是获取不到则系统会真正的挂起线程。

那么为什么锁的升级无法逆向那？

​	这是因为，自旋锁无法预知到底会空循环几个时钟周期，并且会很消耗CPU，为了避免这种无用的自旋操作，一旦锁升级为重量锁，就不会再恢复到轻量级锁，这也是为什么一旦升级无法降级的原因所在。

#### **8.4 Java虚拟机对锁优化所做的努力**

​	从Java虚拟机在优化synchronized的时候引入了：偏向锁、轻量级锁、重量级锁以及自旋锁，都可以看出Java虚拟机通过各种方式，尽量减少获取所和释放锁所带来的性能消耗。

​	但这还不全是Java虚拟机锁做的努力，另外还有：**锁消除** 、 **CAS**等等，更重要的还有一个**无锁**的概念，包括上文中提到的自旋锁，这些内容会在后续的文章中继续学习。



### 9. 线程池参数

```
当一个任务通过execute(Runnable)方法欲添加到线程池时：
1、 如果此时线程池中的数量小于corePoolSize，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。
2、 如果此时线程池中的数量等于 corePoolSize，但是缓冲队列 workQueue未满，那么任务被放入缓冲队列。
3、如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，再有新的线程，开始增加线程池的线程数量处理新的线程，直到maximumPoolSize；
4、 如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量等于maximumPoolSize，那么通过 handler所指定的策略来处理此任务。也就是：处理任务的优先级为：核心线程corePoolSize、任务队列workQueue、最大线程 maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。
5、 当线程池中的线程数量大于 corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止。这样，线程池可以动态的调整池中的线程数。
```

​	**当线程数小于corePoolSize时,提交一个任务创建一个线程(即使这时有空闲线程)来执行该任务。**
​	**当线程数大于等于corePoolSize，首选将任务添加等待队列workQueue中（这里的workQueue是上面的BlockingQueue），等有空闲线程时，让空闲线程从队列中取任务。**
​	**当等待队列满时，如果线程数量小于maximumPoolSize则创建新的线程，否则使用拒绝线程处理器来处理提交的任务。**



Java通过Executors提供四种线程池，分别为：
​	**newCachedThreadPool:**创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
​	**newFixedThreadPool:** 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
​	**newScheduledThreadPool:** 创建一个定长线程池，支持定时及周期性任务执行。
​	**newSingleThreadExecutor:** 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。



### 10. 数据库事务

 1.  什么是数据库事务

     ​	事务（Transaction）是并发控制的基本单位。所谓的事务，它是一个操作序列，这些操作要么都执行，要么都不执行，它是一个不可分割的工作单位。例如，银行转账工作：从一个账号扣款并使另一个账号增款，这两个操作要么都执行，要么都不执行,在关系数据库中,一个事务可以是一条SQL语句、一组SQL语句或整个程序。 。所以，应该把它们看成一个事务。事务是数据库维护数据一致性的单位，在每个事务结束时，都能保持数据一致性。

     2. 数据库事务的性质，隔离级别

       事务的性质:A : 原子性	C: 一致性	I: 隔离性		D: 持久性

    隔离级别:

    ​	**读未提交**: 可以读取到其他事务未提交的数据

    ​	**读已提交**: 可以读取其他事务已提交的数据,不能重复读; **读取不加锁,增删改加锁**

    ​	**可重复读**:  一次事务里对数据的读取不会不同,**对第一次读取的行加锁,**可能出现幻读(虚读),InnoDB解决了幻读的问题;

    ​	**串行化**:	  解决幻读的问题,对整张表加锁,**读写锁**,且读写锁互斥,写写也互斥;

	3. 数据库如何实现串行化

### 11. 缓存淘汰算法

​	常见类型包括LRU、LFU、ARC、FIFO、MRU。

#### **1. LRU**（LeastRecently User）最近最少使用算法

​	这个缓存算法将最近使用的条目存放到靠近缓存顶部的位置。当一个新条目被访问时，LRU将它放置到缓存的顶部。当缓存达到极限时，较早之前访问的条目将从缓存底部开始被移除。这里会使用到昂贵的算法，而且它需要记录“年龄位”来精确显示条目是何时被访问的。此外，当一个LRU缓存算法删除某个条目后，“年龄位”将随其他条目发生改变。

![image](http://upload-images.jianshu.io/upload_images/1466264-5a472df5ec57137f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### **2. LFU**(Least Frequently Used) 最不经常使用算法

​	这个缓存算法使用一个计数器来记录条目被访问的频率。通过使用LFU缓存算法，最低访问数的条目首先被移除。这个方法并不经常使用，因为它无法对一个拥有**最初高访问率**之后长时间没有被访问的条目缓存负责。

![image](http://xiaorui.cc/wp-content/uploads/2015/04/20150420105345_48639.png)



#### 3. FIFO（First inFirst out）-先进先出算法：

​	FIFO是英文First In First Out 的缩写，是一种先进先出的数据缓存器，他与普通存储器的区别是没有外部读写地址线，这样使用起来非常简单，但缺点就是只能顺序写入数据，顺序的读出数据，其数据地址由内部读写指针自动加1完成，不能像普通存储器那样可以由地址线决定读取或写入某个指定的地址。

![image](http://images.cnitblog.com/i/221914/201407/082202026607080.png)



#### **4. 最近最常使用算法（MRU）：**

​	这个缓存算法最先移除最近最常使用的条目。一个MRU算法擅长处理一个条目越久，越容易被访问的情况。

#### **5. 自适应缓存替换算法(ARC)：**

​	在IBM Almaden研究中心开发，这个缓存算法同时跟踪记录LFU和LRU，以及驱逐缓存条目，来获得可用缓存的最佳使用。





### 12. JUC (Java Util Concurrency) 包

#### 	1. JUC概况

以下是Java JUC包的主体结构：
[![img](http://static.oschina.net/uploads/img/201307/24132945_rpmS.jpg)](http://static.oschina.net/uploads/img/201307/24132945_rpmS.jpg)

- Atomic : AtomicInteger,AtomicBoolean,AtomicLong, AtomicArray... AtomicReference
- Locks : AbstractQueuedSynchronizer(同步器,锁机制的核心)Lock, Condition, ReadWriteLock
- Collections : Queue, ConcurrentMap,ConcurrentHashMap,BlockingQueue
- Executer : Future, Callable, Executor
- Tools : CountDownLatch, CyclicBarrier, Semaphore,Exchanger,ForkJoinPool,ForkJoinTask



### 13 hashmap为何是线程不安全的？ConcurrentHashMap原理

​	hash碰撞和扩容导致:	rehash()的时候多线程put容易形成在链表上形成环,再次访问的时候产生无限循环

​	ConcurrentHashMap:

```
	jdk1.7中是采用Segment + HashEntry + ReentrantLock的方式进行实现的，

	1.8中放弃了Segment臃肿的设计，取而代之的是采用 Node + CAS + Synchronized 来保证并发安全进行实现。
```



- JDK1.8的实现降低锁的粒度，JDK1.7版本锁的粒度是基于Segment的，包含多个HashEntry，而JDK1.8锁的粒度就是HashEntry（首节点）
- JDK1.8版本的数据结构变得更加简单，使得操作也更加清晰流畅，因为已经使用synchronized来进行同步，所以不需要分段锁的概念，也就不需要Segment这种数据结构了，由于粒度的降低，实现的复杂度也增加了
- JDK1.8使用红黑树来优化链表，基于长度很长的链表的遍历是一个很漫长的过程，而红黑树的遍历效率是很快的，代替一定阈值的链表，这样形成一个最佳拍档



#### get没有加锁的话，ConcurrentHashMap是如何保证读到的数据不是脏数据的呢？

- 在1.8中ConcurrentHashMap的**get操作全程不需要加锁**，这也是它比其他并发集合比如hashtable、用Collections.synchronizedMap()包装的hashmap;安全效率高的原因之一。
- get操作全程不需要加锁是因为**Node的成员val是用volatile修饰**的,和数组用volatile修饰没有关系。
- 数组用volatile修饰主要是保证在数组扩容的时候保证可见性。

  ​

  ​get操作可以无锁是由于**Node的元素val和指针next是用volatile修饰的**，在多线程环境下线程A修改结点的val或者新增节点的时候是对线程B可见的。

```java
	/**
     * The array of bins. Lazily initialized upon first insertion.
     * Size is always a power of two. Accessed directly by iterators.
     */
    transient volatile Node<K,V>[] table;
	/**
     * The next table to use; non-null only while resizing.
     * nextTable：扩容时新生成的数据，数组为table的两倍
     */
    private transient volatile Node<K,V>[] nextTable;

static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
  		// Node的元素val和指针next是用volatile修饰的
        volatile V val;
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }
        ...
}
```

既然volatile修饰数组对get操作没有效果那加在数组上的volatile的目的是什么呢？

​	其实就是为了使得Node数组在**扩容的时候对其他线程具有可见性**而加的volatile



### 14. 数据库引擎及比较

​	![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1554026562343&di=9f5f39738ab5366b539553ff60a9986c&imgtype=0&src=http%3A%2F%2Faliyunzixunbucket.oss-cn-beijing.aliyuncs.com%2Fjpg%2F05a2a5d493d1bde2529adf7a95435c7d.jpg%3Fx-oss-process%3Dimage%2Fresize%2Cp_100%2Fauto-orient%2C1%2Fquality%2Cq_90%2Fformat%2Cjpg%2Fwatermark%2Cimage_eXVuY2VzaGk%3D%2Ct_100)

**Mysql各种索引区别：**

​	**普通索引(INDEX)**：	最基本的索引，没有任何限制
​	**唯一索引(UNIQUE)：**	与"普通索引"类似，不同的就是：索引列的值必须唯一，但**允许有空值**。
​	**主键索引(PRIMARY)：**	它 是一种特殊的唯一索引，**不允许有空值**。 
​	**全文索引(FULLTEXT )：仅可用于 MyISAM 表**， 用于在一篇文章中，检索文本信息的, 针对较大的数据，生成全文索引很耗时耗空间。
​	**组合索引：**	为了更多的提高mysql效率可建立组合索引，遵循**”最左前缀“**原则。

#### 1. InnoDB

​	InnoDB是**事务型数据库**的首选引擎，**支持事务安全表（ACID），支持行锁定和外键，**上图也看到了，InnoDB是默认的MySQL引擎。InnoDB主要特性有：

​	Innodb引擎提供了对数据库**ACID事务**的支持，并且实现了SQL标准的**四种隔离级别**。

​	该引擎还提供了**行级锁和外键约束**，它的设计目标是**处理大容量数据库系统**，它本身其实就是基于MySQL后台的完整数据库系统;

​	MySQL运行时Innodb会在内存中建立**缓冲池，用于缓冲数据和索引**。但是该引擎**不支持FULLTEXT类型的索引**，而且它没有保存表的行数，当SELECT COUNT(*) FROM TABLE时需要扫描全表。

​	当需要使用数据库事务时，该引擎当然是首选。由于锁的粒度更小，写操作不会锁定全表，所以**在并发较高时，使用Innodb引擎会提升效率。**但是使用行级锁也不是绝对的，如果在执行一个SQL语句时MySQL不能确定要扫描的范围，InnoDB表同样会锁全表。



#### 2. **MyISAM存储引擎**

​	MyIASM没有提供对数据库事务的支持，也不支持行级锁和外键，因此当INSERT(插入)或UPDATE(更新)数据时即**写操作需要锁定整个表**，效率便会低一些。

​	不过和Innodb不同，**MyIASM中存储了表的行数**，于是SELECT COUNT(*) FROM TABLE时只需要直接读取已经保存好的值而不需要进行全表扫描。

​	如果表的读操作远远多于写操作且不需要数据库事务的支持，那么MyIASM也是很好的选择。



两种引擎的选择

​	大尺寸的数据集趋向于选择InnoDB引擎，因为它支持事务处理和故障恢复。数据库的大小决定了故障恢复的时间长短，**InnoDB可以利用事务日志进行数据恢复**，这会比较快。主键查询在InnoDB引擎下也会相当快，不过需要注意的是如果主键太长也会导致性能问题。

​	大批的INSERT语句(在每个INSERT语句中写入多行，批量插入)在MyISAM下会快一些，但是UPDATE语句在InnoDB下则会更快一些，尤其是在并发量大的时候。

#### 3. MEMORY存储引擎

​	所有的数据都在内存中，数据的处理速度快，但是安全性不高。如果需要很快的读写速度，对数据的安全性要求较低，可以选择MEMOEY。它对表的大小有要求，不能建立太大的表。所以，这类数据库只使用在相对较小的数据库表。

​	注意，**同一个数据库也可以使用多种存储引擎的表**。如果一个表要求比较高的事务处理，可以选择InnoDB。这个数据库中可以将查询要求比较高的表选择MyISAM存储。如果该数据库需要一个用于查询的临时表，可以选择MEMORY存储引擎。

#### B-Tree和B+Tree

​	B+Tree是B-Tree的变种，那么我就先讲B-Tree吧，B-Tree的每个结点做多可以有d个分支（叉），d称为B-Tree的度，如下图所示，它的每个结点可以有4个元素，5个分支，于是它的度为5。B-Tree中的元素是有序的，比如图中元素7左边的指针指向的结点中的元素都小于7，而元素7和16之间的指针指向的结点中的元素都处于7和16之间，正是满足这样的关系，才能高效的查找：

​	首先从根节点进行**二分查找**，找到就返回对应的值，否则就进入相应的区间结点递归的查找，直到找到对应的元素或找到null指针，找到null指针则表示查找失败。这个查找是十分高效的，其时间复杂度为**O(logN)**（以d为底，当d很大时，树的高度就很低），因为每次检索最多只需要检索树高h个结点。



接下来就该讲B+Tree了，它是B-Tree的变种，如图所示：

​	![B+Tree](http://www.2cto.com/uploadfile/Collfiles/20150327/2015032710000561.png)



#### MyISAM引擎的索引结构

​	**MyISAM引擎的索引结构为B+Tree**，其中B+Tree的数据域存储的内容为实际数据的地址，也就是说它的索引和实际的数据是分开的，只不过是用索引指向了实际的数据，这种索引就是所谓的**非聚集索引**。

#### Innodb引擎的索引结构

​	Innodb引擎的索引结构同样也是**B+Tree**，但是**Innodb的索引文件本身就是数据文件**，即B+Tree的数据域存储的就是实际的数据，这种索引就是**聚集索引**。这个**索引的key就是数据表的主键**，因此InnoDB表数据文件本身就是主索引。
​	因为**InnoDB的数据文件本身要按主键聚集**，所以InnoDB要求表**必须有主键**（MyISAM可以没有），如果没有显式指定，则MySQL系统会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整形。

​	并且和MyISAM不同，**InnoDB的辅助索引数据域存储的也是相应记录主键的值而不是地址**，所以当以辅助索引查找时，会先根据辅助索引找到主键，再根据主键索引找到实际的数据。所以**Innodb不建议使用过长的主键**，否则会使辅助索引变得过大。建议使用自增的字段作为主键，这样B+Tree的每一个结点都会被顺序的填满，而不会频繁的分裂调整，会有效的提升插入数据的效率。



#### Mysql的存储引擎和索引

        可以说数据库必须有索引，没有索引则检索过程变成了顺序查找，O(n)的时间复杂度几乎是不能忍受的。我们非常容易想象出一个只有单关键字组成的表如何使用B+树进行索引，只要将关键字存储到树的节点即可。当数据库一条记录里包含多个字段时，一棵B+树就只能存储主键，如果检索的是非主键字段，则主键索引失去作用，又变成顺序查找了。这时应该在第二个要检索的列上建立第二套索引。  这个索引由独立的B+树来组织。

​	有两种常见的方法可以解决多个B+树访问同一套表数据的问题:

​	一种叫做**聚簇索引**（clustered index ），一种叫做**非聚簇索引**（secondary index）。这两个名字虽然都叫做索引，但这并不是一种单独的索引类型，而是一种数据存储方式。

​	对于聚簇索引存储来说，行数据和主键B+树存储在一起，辅助键B+树只存储辅助键和主键，主键和非主键B+树几乎是两种类型的树。

​	对于非聚簇索引存储来说，主键B+树在叶子节点存储指向真正数据行的指针，而非主键。

        **InnoDB使用的是聚簇索引**，将主键组织到一棵B+树中，而行数据就储存在叶子节点上，若使用"where id = 14"这样的条件查找主键，则按照B+树的检索算法即可查找到对应的叶节点，之后获得行数据。**若对Name列进行条件搜索，则需要两个步骤**：第一步在辅助索引B+树中检索Name，到达其叶子节点获取对应的主键。第二步使用主键在主索引B+树种再执行一次B+树检索操作，最终到达叶子节点即可获取整行数据。

        **MyISM使用的是非聚簇索引**，非聚簇索引的两棵B+树看上去没什么不同，节点的结构完全一致只是存储的内容不同而已，主键索引B+树的节点存储了主键，辅助键索引B+树存储了辅助键。表数据存储在独立的地方，这两颗B+树的叶子节点都使用一个地址指向真正的表数据，对于表数据来说，这两个键没有任何差别。由于索引树是独立的，通过辅助键检索无需访问主键的索引树。



**聚簇索引的优势在哪？**

        1 由于行数据和叶子节点存储在一起，这样主键和行数据是一起被载入内存的，找到叶子节点就可以立刻将行数据返回了，如果按照主键Id来组织数据，获得数据更快。

        2 辅助索引使用主键作为"指针" 而不是使用地址值作为指针的好处是，减少了当出现行移动或者数据页分裂时辅助索引的维护工作，使用主键值当作指针会让辅助索引占用更多的空间，换来的好处是InnoDB在移动行时无须更新辅助索引中的这个"指针"。也就是说行的位置（实现中通过16K的Page来定位，后面会涉及）会随着数据库里数据的修改而发生变化（前面的B+树节点分裂以及Page的分裂），使用聚簇索引就可以保证不管这个主键B+树的节点如何变化，辅助索引树都不受影响。





### 15. 操作系统中的锁

​	在硬件层面，CPU提供了**原子操作(test and set)、关中断、锁内存总线**的机制；OS基于这几个CPU硬件机制，就能够实现锁；再基于锁，就能够实现各种各样的同步机制（信号量、消息、Barrier等等等等）。

​	锁是线程同步时的一个重要的工具，然而操作系统中包含了多种不同的锁，各种锁之间有什么不同呢？

1、**信号量（Semaphore）**

​	信号量分为二元信号量和多元信号量，所谓二元信号量就是指该信号量只有两个状态，要么被占用，要么空闲；而多元信号量则允许同时被N个线程占有，超出N个外的占用请求将被阻塞。信号量是“**系统级别**”的，即同一个信号量可以被不同的进程访问。

2、**互斥量 （Mutex）**

​	和二元信号量类似， 唯一不同的是，互斥量的获取和释放必须是在**同一个线程**中进行的。如果一个线程去释放一个并不是它所占有的互斥量是无效的。而信号量是可以由其它线程进行释放的。

3、**临界区（Critical Section）**

​	术语中，把临界区的锁的获取称为进入临界区，而把锁的释放称为离开临界区。临界区是**“进程级别”**的，即它只在本进程的所有线程中可见，其它性质与互斥量相同（即谁获取，谁释放）

4、**读写锁（Read-Write Lock）**

​	适用于一个特定的场合。比如对于一段线程间访问的数据，如果程序大部分时间都是在读取，而只有很少的时间才会写入，那么使用前面几种锁时，每次读取也是同样 要申请锁的，而这时其它的线程就无法再对此段数据进行读取。可是，多个线程同时对一段数据进行读取时，是不存在同步问题的，那么这些读取时设置的锁就影响 了程序的性能。读写锁的出现就是为了解决这个问题的。

​	对于一个读写锁，有两种获取方式：共享（Shared）或独占 （Exclusive）。如果当前读写锁处于空闲状态，那么当多个线程同时以共享方式访问该读写锁时，都可以成功；而此时如果一个线程以独占的方式访问该 读写锁，那么它会等待所有共享访问都结束后才可以成功。在读写锁被独占访问的过程中，再次共享和独占请求访问该锁，都会进行等待状态。

5、**条件变量（Condition Variable）**

​	条件变量相当于一种通知机制。多个线程可以设置等待该条件变量，而一旦另外的线程设置了该条件变量（相当于唤醒条件变量）后，多个等待的线程就可以继续执行了。



### 16. 线程同步的7种方式

 1.  volatile

      	1. volatile变量在多线程之间的"可见性";
      	2. 禁止指令重排;
      	3. 根本原因: lock前缀指令,将修改内容同步到共享内存,并使其他CPU(内核)无效化(Invalidate)其Cache

 2.  synchronized代码块

	3. synchronized

	4. Lock

    	1.   注：关于Lock对象和synchronized关键字的选择：   

        ```
        	a.最好两个都不用，使用一种java.util.concurrent包提供的机制,能够帮助用户处理所有与锁相关的代码。 
        	b.如果synchronized关键字能满足用户的需求，就用synchronized，因为它能简化代码 
        	c.如果需要更高级的功能，就用ReentrantLock类，此时要注意及时释放锁，否则会出现死锁，通常在finally代码释放锁 
        ```

    	2. **sync的核心是AQS，AQS的核心是state和双端队列。**

	5. ThreadLocal

    	1. ```java
        ThreadLocal.get(){
        	// 流程
        	// 1 使用当前线程 获取当前线程的成员变量 ThreadLocalMap
        		// THreadLocalMap 是Entry<ThreadLocal,value>数组,类似HashMap,因为一个线程可能有多个ThreadLocal变量
        	// 2 使用ThreadLocal作为键(key) 从ThreadLocalMap中获取到 Entry<ThreadLocal,value>
        	// 3 return Entry<ThreadLocal,value>的value

        	Thread t ==> ThreadLocalMap( Entry<ThreadLocal,value>[] table) ==> Entry<ThreadLocal,value> ==> value
        } 
        ```

        ​

	6. BlockingQueue

    	1. ```
          LinkedBlockingQueue 类常用方法 
            LinkedBlockingQueue() : 创建一个容量为Integer.MAX_VALUE的LinkedBlockingQueue 
            put(E e) : 在队尾添加一个元素，如果队列满则阻塞 
            size() : 返回队列中的元素个数 
            take() : 移除并返回队头元素，如果队列空则阻塞 
           
           注：BlockingQueue<E>定义了阻塞队列的常用方法，尤其是三种添加元素的方法，我们要多加注意，当队列满或空时：
        　　add()方法会抛出异常		remove()方法会抛出异常
        　　offer(e)方法返回false	   poll(e)方法会返回null
        　　put()方法会阻塞		  take()方法会阻塞
        ```

        ​

	7. AtomicXXX原子变量

注：多线程中的非同步问题主要出现在对域的读写上，如果让域自身避免这个问题，则就不需要修改操作该域的方法。 
​    用final域，有锁保护的域和volatile域可以避免非同步的问题。 





































































