# 《Java并发编程的艺术》笔记
## 第一章 并发编程的挑战
* 略
## 第二章 Java并发机制的底层实现原理
* volatile的两条实现原则：
    1. Lock前缀指令会引起处理器缓存回写到内存
    2. 一个处理器的缓存回写到内存会导致其他处理器的缓存无效。
* volatile的使用优化：共享变量会被频繁读写时，可以通过追加为64字节以提高并发编程的效率。因为目前主流处理器高速缓存行是64个字节宽，不支持部分填充缓存行，通过追加到64字节的方式填满高速缓冲区的缓存行，避免各元素加载到同一缓存行而互相锁定。（**Java7后可能不生效，因为Java7更智能，会淘汰或重新排列无用字段，需要使用其他追加字节的方式**）
* Java对象头

|长度|内容|说明|
|-|-|-|
|32/64bit|Mark Word|存储对象的hashCode或锁信息等|
|32/64bit|Class Metadata Address|存储到对象类型数据的指针|
|32/64bit|Array Length|数组的长度(仅当当前对象为数组时存在)|
* CAS操作，即Compare And Swap，比较并交换。CAS操作需要输入两个数值，一个旧值(期望操作前的值)和一个新值，在操作期间先比较旧值有没有发生变化，如果没有则交换成新值，否则不交换。
* 锁有4种状态，级别从低到高依次是：无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态。这几种状态会随着竞争情况逐渐升级，只升不降。
* 偏向锁
    * 当一个线程访问同步块并获取锁时，会在对象头和栈帧中的所记录里存储锁偏向的线程ID，以后该线程再次加锁解锁时不需要进行CAS操作，只需简单地测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁。
    * 偏向锁使用了一种等到竞争出现才释放锁的机制，所以当其他线程常识竞争偏向锁时，豉油偏向锁的线程才会释放锁。
    * 偏向锁的撤销，需要等待全局安全点（在这个时间点上没有正在执行的字节码）。它会首先暂停拥有偏向锁的线程，然后检查豉油偏向锁的线程是否活着，如果线程不处于活动状态，则将对象头设置成无锁状态；如果线程仍然活着，拥有偏向锁的栈会被执行，遍历偏向对象的锁记录，栈中的所记录和对象头的Mark Word要么重新偏向于其他线程，要么恢复到无锁或者标记对象不适合作为偏向锁，最后唤醒暂停的线程。
    * 偏向锁默认启动，但是它在应用程序启动几秒钟之后才激活，如有必要可以使用JVM参数来关闭延迟：-XX:BiasedLockingStartupDelay=0。如果确定应用程序里所有的锁通常情况下处于竞争状态，可以通过JVM参数来关闭偏向锁：-XX:-UseBiasedLocking=false，那么程序默认会进入轻量级锁状态。
* 轻量级锁
    * 线程在执行同步块之前，JVM会现在当前线程的栈帧中创建用于存储锁记录的空间，并将对象头中的Mark Word复制到锁记录中(Displaced Mark Word)。然后线程尝试使用CAS将对象头中的Mark Word奇幻为指向锁记录的指针。如果成功则当前线程获得锁，否则说明其他线程竞争锁，当前线程尝试使用自旋来获取锁。
    * 轻量级锁解锁时，会使用原子的CAS操作将Displaced Mark Word替换回到对象头，如果成功则表示没有竞争发生，否则表示当前锁存在竞争，锁会升级为重量级锁。
* 重量级锁
    * 其他线程视图获取锁时都被阻塞，持有锁的线程释放锁之后唤醒这些线程，被唤醒的线程再抢。
* 锁的优缺点对比

|锁|优点|缺点|适用场景|
|-|-|-|-|
|偏向锁|加锁和解锁无需额外的消耗，和执行非同步方法相比仅存在纳秒级的差距|如果线程间存在锁竞争，会带来额外的锁撤销的消耗|适用于只有一个线程访问同步块场景|
|轻量级锁|竞争的线程不会阻塞，提高了程序的响应速度|如果始终得不到锁竞争的线程，使用自旋会消耗CPU|追求响应时间，同步块执行速度非常快|
|重量级锁|线程竞争不适用自旋，不会消耗CPU|线程阻塞，响应时间缓慢|追求吞吐量，同步块执行速度较慢|
* 使用循环CAS实现原子操作，JDK的并发包提供了一些类支持原子操作，如AtomicInteger等。
* CAS实现原子操作的三大问题
    1. ABA问题。一个值从A变成B又变成A，使用CAS检查时以为没有变化，但实际上却变化了。解决思路是使用版本号。
    2. 循环时长长，开销大。
    3. 只能保证一个共享变量的原子操作。（Java1.5以后，JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作）
## 第三章 Java内存模型
* 并发编程模型的两个关键问题：线程之间如何通信及线程之间如何同步
* 在命令式编程中，线程之间的通信机制有两种：共享内存和消息传递。
* 内存屏障类型表

|屏障类型|说明|
|-|-|
|LoadLoad Barriers|确保Load1数据的装载先于Load2及所有后序装置指令的装载|
|StoreStore Barriers|确保Store1数据对其他处理器可见(刷新到内存)先于Store2及所有后序存储指令的存储|
|LoadStore Barriers|确保Load1数据装载先于Store2及所有后序的存储指令刷新到内存|
|StoreLoad Barriers|确保Store1数据对其他处理器变得可见(指刷新到内存)先于Load2及所有后序装载指令的装载。StoreLoad Barriers会使改屏障之前的所有内存访问指令(存储和装载指令)完成之后，才执行该屏障之后的内存访问指令|
* 编译器、runtime和处理器都必须遵守as-if-serial语义，即不管怎么重排序，(单线程)程序的执行结果不能被改变。
### volatile的内存语义
* volatile变量自身具有以下特性：
    * 可见性：对一个volatile变量的读，总是能看到(任意线程)对这个volatile变量最后的写入。
    * 原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。
* volatile写-读的内存语义：
    * 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存。
    * 当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。
* JMM内存屏障插入策略：
    * 在每个volatile写操作**前后**分别插入StoreStore、StoreLoad屏障。
    * 在每个volatile读操作**后面**插入LoadLoad、LoadStore屏障。
### 锁的内存语义
* 当线程获取锁时，JMM会把该线程对应的本地内存置为无效。
* 当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中。
* 公平锁和非公平锁的内存语义：
    * 公平锁和非公平锁释放时，最后都要写一个volatile变量state。
    * 公平锁获取时，首先会去读volatile变量。
    * 非公平锁获取时，首先会用CAS操作更新volatile变量，这个操作同时具有volatile读/写的内存语义。
### final域的内存语义
* 对于final域，编译器和处理器要遵循两个重排序规则。
    * 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
    * 初次读一个包含final域的对象的应用，与随后初次读这个final域，这两个操作之间不能重排序。
* 读final域的重排序规则可以确保：在读一个对象的final域之前，一定会先读包含这个final域的对象的引用。
* 写final域的重排序规则可以确保：在对象引用为任意线程可见之前，对象的final域已经被正确初始化。（**前提是对象引用不能在构造函数中逸出**）
## 第四章 Java并发编程基础
* 线程的优先级仅仅是一部分决定因素，因为线程的切换具有随机性，而且针对不同的系统而言，优先级这个概念可能就不存在，其仅仅是决定程序设计的衡量的一个标准。
* 等待/通知的经典范式
    * 等待方遵循如下原则：
        1. 获取对象的锁
        2. 如果条件不满足，那么调用对象的wait()方法，被通知后仍要检查条件。
        3. 条件满足则执行对应的逻辑。
    * 通知方遵循如下原则：
        1. 获得对象的锁。
        2. 改变条件。
        3. 通知所有等待在对象上的线程。
## 第五章 Java中的锁
* Lock接口提供的synchronized关键字不具备的主要特性

|特性|描述|
|-|-|
|尝试非阻塞地获取锁|当前线程尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取并持有锁|
|能被中断地获取锁|获取到锁的线程能够相应中断，当获取到锁的线程被中断时，中断异常将会被抛出，同时锁会被释放|
|超时获取锁|在指定的截止时间之前获取锁，如果截止时间到了仍旧无法获取锁，则返回|
### 队列同步器(AbstractQueuedSynchronizer)
* 同步器提供如下3个方法来访问或修改同步状态(int成员变量)：
    * getState()：获取当前同步状态
    * setState(int newState)：设置当前同步状态
    * compareAndSetState(int expect, int update)：使用CAS设置当前状态，该方法能够保证状态设置的原子性。
* 同步器可重写的方法

|方法名称|描述|
|-|-|
|protected boolean tryAcquire(int arg)|独占式获取同步状态，实现该方法需要查询当前状态并判断同步状态是否符合预期，然后再进行CAS设置同步状态|
|protected boolean tryRelease(int arg)|独占式释放同步状态，等待获取同步状态的线程将有机会获取同步状态|
|protected int tryAcquireShared(int arg)|共享式获取同步状态，返回大于等于0的值，表示获取成功，反之失败|
|protected boolean tryReleaseShared(int arg)|共享式释放同步状态|
|protected boolean isHeldExclusively()|当前同步器是否在独占模式下被线程占用，一般该方法表示是否被当前线程所独占|
* 同步器提供的模板方法

![](https://images2017.cnblogs.com/blog/1254811/201712/1254811-20171227103310463-323247292.png)
* 同步队列及等待队列的节点属性类型与名称以及描述

![](https://images2017.cnblogs.com/blog/1254811/201712/1254811-20171227112935041-1271371951.png)
* 同步队列中的结点只有当其前驱结点为头结点时，才尝试获取同步状态。
* 读写锁的同步状态高16位表示读状态、低16位表示写状态。
### LockSupport
* 当需要阻塞或唤醒一个线程的时候，都会使用LockSupport工具类来完成相应工作。LockSupport定义了一组公共静态方法，这些方法提供了最基本的线程阻塞和唤醒功能，而LockSupport也成为了构建同步组建的基础工具。
* LockSupport提供的阻塞和唤醒方法（其中参数blocker是用来标识当前线程在等待的对象，便于问题排查和系统监控）

|方法名称|描述|
|-|-|
|void park(Object blocker)|阻塞当前线程，如果调用unpark方法或者当前线程被终端，才能从park方法返回|
|void parkNanos(Object blocker, long nanos)|阻塞当前线程，最长不超过nanos纳秒，返回条件在park的基础上增加了超时返回|
|void parkUntil(Object blocker, long deadline)|阻塞当前线程，知道deadline时间|
|void unpark(Thread thread)|唤醒处于阻塞状态的线程thread|
## 第六章 Java并发容器和框架
* ConcurrentHashMap使用锁分段技术，将数据非诚一段一段地存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。
* ConcurrentLinkedQueue非阻塞的线程安全队列
### 阻塞队列
* 阻塞队列(BlockingQueue)是一个支持两个附加操作的队列。这两个附加的操作支持阻塞的插入和移除方法。
    1. 支持阻塞的插入方法：当队列满时，队列会阻塞插入元素的线程，知道队列不满。
    2. 支持阻塞的移除方法：当队列空时，获取元素的线程会等待队列变为非空。
* 插入和移除操作的4种处理方式

|方法/处理方式|抛出异常|返回特殊值|一直阻塞|超时退出|
|-|-|-|-|-|
|插入方法|add(e)|offer(e)|put(e)|offer(e, time, unit)|
|移除方法|remove()|poll()|take()|poll(time, unit)|
|检查方法|element()|peek()|不可用|不可用|
* 抛出异常：当队列满时，如果再往队列里插入元素会抛出IllegalStateException("Queue full")异常。当队列空时，从队列里获取元素会抛出NoSuchElementException异常。
* 返回特殊值：当往队列插入元素时，会返回元素是否插入成功，成功返回true。如果是移除方法，则从队列里取出一个元素，如果没有则返回null。
* 一直阻塞：当阻塞队列满时，如果生产者线程往队列里put元素，队列会一直阻塞生产者线程，知道队列可用或者响应中断退出。当队列空时，如果消费者线程从队列列take元素，队列会阻塞消费者线程，知道队列不为空。
* 超时退出：当阻塞队列满时，如果生产者线程往队列里插入元素，队列会阻塞生产者线程一段时间，如果超时则退出。
* JDK7提供了7个阻塞队列：
    1. ArrayBlockingQueue：用数组实现的有界阻塞队列，按FIFO原则对元素进行排序。
    2. LinkedBlockingQueue：用链表实现的有界阻塞队列，默认和最大长度为Integer.MAX_VALUE，按FIFO原则对元素进行排序。
    3. PriorityBlockingQueue：支持优先级的无界阻塞队列，默认情况下元素采取自然顺序升序排列。不保证同优先级元素的顺序。
    4. DelayQueue：支持延时获取元素的无界阻塞队列。队列使用PriorityQueue实现，队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素。可应用于：
        * 缓存系统的设计：用DelayQueue保存缓存元素的有效期，使用一个线程循环查询DelayQueue，一旦能从DelayQueue中获取元素表示缓存有效期到了。
        * 定时任务调度：使用DelayQueue保存当天将会执行的任务和执行时间，一旦从DelayQueue中获取到任务就开始执行，比如TimerQueue就是使用DelayQueue实现的。
    5. SynchronousQueue：不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。
    6. LinkedTransferQueue：链表结构组成的无界阻塞TransferQueue队列。相对于其他阻塞队列，LinkedTransferQueue多了tryTransfer和transfer方法。
        * transfer方法：如果当前有消费者正在等待接收元素，transfer方法可以把生产者传入的元素立刻传给消费者。如果没有消费者在等待接收元素，则将元素存放在队列tail节点并等到钙元素被消费者消费了才返回。
        * tryTransfer方法：如果没有消费者等待接收元素，则立即返回false。
    7. LinkedBlockingDeque：链表结构组成的双向阻塞队列。
### Fork/Join框架
* Fork/Join框架是一个用于并行执行任务的框架，是一个把大任务分隔成若干个小任务，最终汇总每个小任务结果后得到大人物结果的框架。
* 工作窃取算法：假如我们需要做一个比较大的任务，我们可以把这个任务分割为若干互不依赖的子任务，为了减少线程间的竞争，于是把这些子任务分别放到不同的队列里，并为每个队列创建一个单独的线程来执行队列里的任务，线程和队列一一对应，比如A线程负责处理A队列里的任务。但是有的线程会先把自己队列里的任务干完，而其他线程对应的队列里还有任务等待处理。干完活的线程与其等着，不如去帮其他线程干活，于是它就去其他线程的队列里窃取一个任务来执行。而在这时它们会访问同一个队列，所以为了减少窃取任务线程和被窃取任务线程之间的竞争，通常会使用双端队列，被窃取任务线程永远从双端队列的头部拿任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行。工作窃取算法的优点是充分利用线程进行并行计算，并减少了线程间的竞争，其缺点是在某些情况下还是存在竞争，比如双端队列里只有一个任务时。并且消耗了更多的系统资源，比如创建多个线程和多个双端队列。
* ForkJoinTask：我们要使用ForkJoin框架，必须首先创建一个ForkJoin任务。它提供在任务中执行fork()和join()操作的机制，通常情况下我们不需要直接继承ForkJoinTask类，而只需要继承它的子类，Fork/Join框架提供了以下两个子类：
    * RecursiveAction：用于没有返回结果的任务。
    * RecursiveTask ：用于有返回结果的任务。
* ForkJoinPool ：ForkJoinTask需要通过ForkJoinPool来执行，任务分割出的子任务会添加到当前工作线程所维护的双端队列中，进入队列的头部。当一个工作线程的队列里暂时没有任务时，它会随机从其他工作线程的队列的尾部获取一个任务。
* ForkJoinTask在执行的时候可能会抛出异常，但是我们没办法在主线程里直接捕获异常，所以ForkJoinTask提供了isCompletedAbnormally()方法来检查任务是否一件抛出异常或已经被取消了，并且可以通过ForkJoinTask的getException方法获取异常。其中，getException方法返回Throwable对象，如果任务被取消了则返回CancellationException，如果任务没有完成或者没有抛出异常则返回null。
## 第七章 Java中的13个原子操作类（Ps：认真数了一下发现只有12个）
* 原子更新方式
    * 原子更新基本类型
    * 原子更新数组
    * 原子更新引用
    * 原子更新属性（字段）
*原子更新基本类型
    * AtomicBoolean ：原子更新布尔类型
    * AtomicInteger： 原子更新整型
    * AtomicLong: 原子更新长整型
* 原子更新数组
    * AtomicIntegerArray ：原子更新整型数组里的元素
    * AtomicLongArray :原子更新长整型数组里的元素
    * AtomicReferenceArray : 原子更新引用类型数组的元素
* 原子更新引用类型
    * AtomicReference ：原子更新引用类型
    * AtomicReferenceFieldUpdater ：原子更新引用类型里的字段
    * AtomicMarkableReference：原子更新带有标记位的引用类型。可以原子更新一个布尔类型的标记位和应用类型
* 原子更新字段类
    * AtomicIntegerFieldUpdater:原子更新整型的字段的更新器
    * AtomicLongFieldUpdater：原子更新长整型字段的更新器
    * AtomicStampedReference:原子更新带有版本号的引用类型。该类将整型数值与引用关联起来，可用于原子的更新数据和数据的版本号，可以解决使用CAS进行原子更新时可能出现的ABA问题。
## 第八章 Java中的并发工具类
### CountDownLatch
* CountDownLatch允许一个或多个线程等待其他线程完成操作。
* CountDownLatch的构造函数接收一个int类型的参数作为计数器，如果你想等待N个点完成，就传入N。
* 每次调用CountDownLatch的countDown方法时，N就减1，CountDownLatch的await方法会阻塞当前线程，知道N变成0。
### CyclicBarrier
* CyclicBarrier的作用是让一组线程到达一个屏障（也可以称之为同步点）时被阻塞，知道最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才能继续运行。
* CyclicBarrier(int parties)构造函数接收一个int参数用来设置拦截线程的数量，还有一个更高级的构造函数CyclicBarrier(int parties, Runnable barrierAction)设定第一个到达屏障的线程执行barrierAction。
* getNumberWaiting方法可以获得CyclicBarrier阻塞的线程数量，isBroken()方法用来了解阻塞的线程是否被中断。
### Semaphore
* Semaphore(信号量)用来控制同时访问特定资源的线程数量，通过协调各个线程以保证合理的使用公共资源。
* Semaphore(int permits)构造方法接收一个int参数，表示可用的许可证数量。
* 每次线程使用Semaphore的acquire()方法获取一个许可证，用完后调用release()方法归还。
* 其他方法
    * intavailablePermits()：返回此信号量中当前可用的许可证数。
    * intgetQueueLength()：返回正在等待获取许可证的线程数。
    * booleanhasQueuedThreads()：是否有线程正在等待获取许可证。
    * void reducePermits(int reduction)：减少reduction个许可证，是个protected方法。
    * Collection getQueuedThreads()：返回所有等待获取许可证的线程集合，是个protected方法。
### Exchanger
* 用于线程间交换数据，每两次执行Exchanger的exchange(V data)方法，则交换两次执行的数据并返回。可用于校对工作。
## 第九章 Java中的线程池
* 线程池的3个好处
    1. 降低资源消耗：通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
    2. 提高响应速度：当任务到达时，任务可以不需要等到线程创建就能立即执行。
    3. 提高线程的可管理性：线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性。使用线程池可以进行统一分配、调优和监控。
* 当提交一个新任务到线程池时，线程池的流程：
    1. 如果当前运行的线程少于corePoolSize，则创建新线程来执行任务。
    2. 如果运行的线程等于或多于corePollSize，则将任务加入BlockingQueue。
    3. 如果无法将任务加入BlockingQueue（队列已满），则创建新的线程来处理任务。
    4. 如果创建新线程将导致当前运行的线程数超过maximumPoolSize，任务将被拒绝，并调用RejectedExecutionHandler.rejectedExecution()方法。
### 线程池的构造方法
* ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, milliseconds, runnableTaskQueue, handler)，其中：
    1. corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，线程池会创建一个线程来执行任务，即使其他空闲的基本线程能够执行新任务也会创建线程，知道需要执行的任务数大于线程池基本大小。如果调用了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有基本线程。
    2. runnableTaskQueue（任务队列）：用于保存等待执行的任务的阻塞队列。可以选择ArratBlockingQueue, LinkedBlockingQueue, SynchronousQueue, PriorityBlockingQueue。
    3. maximumPoolSize（线程池最大数量）：线程池允许创建的最大线程数。
    4. ThreadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字。
    5. RejectedExecutionHandler（饱和策略）：当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。这个策略默认情况下是AbortPolicy，表示无法处理新任务时抛出异常。策略有下列几种：
        * AbortPolicy：直接抛出异常
        * CallerRunsPolicy：只用调用者所在线程来运行任务。
        * DiscardOldestPolicy：丢弃队列里最近的一个任务，并执行当前任务。
        * DiscardPolicy：不处理，丢弃掉。
        * 其他应用场景需要实现RejectedExecutionHandler接口自定义。
    6. keepAliveTime（线程活动保持时间）：线程池的工作线程空闲后，保持存活的时间。如果任务多且执行时间短，可以调高存活时间提高线程利用率。
    7. TimeUnit（线程活动保持时间的单位）
* 向线程池提交任务有两种方式：
    * execute(Runnable command)：没有返回值
    * submit(Callable<T> task)：返回一个future类型的对象，通过这个对象判断任务是否执行成功，并通过其get()方法来获取返回值，该方法会阻塞当前线程知道任务完成。
* 线程池的关闭有两种方法：
    * shutdown()：将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。
    * shutdownNow()：将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表。
    * 只要调用了两种关闭方法的任一种，isShutdown()方法都会返回true。当且仅当所有任务都关闭，CIA表示线程池关闭成功，这是isTerminaed()方法才会返回true。
### 合理配置线程池（设N为CPU个数）
* CPU密集型任务，应配置尽可能少的线程，如N+1。
* IO密集型任务，应配置尽可能多的线程，如2N。
* 优先级不同的任务可以考虑使用优先级队列priorityBlockingQueue来处理，但优先级低的任务可能永远不被执行。
* 使用有界队列能增加系统的稳定性和预警性，避免队列越来越多撑满内存，导致系统不可用。
### 线程池的监控
* 监控线程池的时候可以使用以下属性：
    * taskCount：线程池需要执行的任务数量。
    * completedTaskCount：线程池在运行过程中已完成的任务数量，小于或等于taskCount。
    * largestPoolSize：线程池里曾经创建过的最大线程数量。通过这个数据可以知道线程池是否曾经满过。
    * getPoolSize：线程池的线程数量。如果线程池不销毁的话，线程池里的线程不会自动销毁，所以这个大小只增不减。
    * getActiveCount：获取活动的线程数。
* 可以通过继承线程池来自定义线程池，重写线程池的beforeExecute, afterExecute和terminated方法，也可以在任务执行前后和线程池关闭前执行一些代码来进行监控。例如，监控任务的平均执行时间、最大执行时间和最小执行时间等。这几个方法在线程池里都是空方法。
## 第十章 Executor框架
* 工厂类Executors可创建3种类型的ThreadPoolExecutor：
    * FixedThreadPool：可重用固定线程数的线程池。`new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>())`
    * SingleThreadExecutor：使用单个worker线程的Executor。`new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>())`
    * CachedThreadPool：会根据需要创建新线程的线程池。`new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnbalbe>())`
* 工厂类Executors可创建2种类型的ScheduledThreadPoolExecutor：
    * ScheduledThreadPoolExecutor：包含若干个线程的ScheduledThreadPoolExecutor。
    * SingleThreadScheduledExecutor：只包含一个线程的ScheduledThreadPoolExecutor。
## 第十一章 Java并发编程实践
* 略