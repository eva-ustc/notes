# 《Java多线程编程核心技术》笔记

## 第一章 Java多线程技能
* 使用多线程时，代码的运行结果于代码执行顺序或调用顺序无关。
* `interrupted()`为`Thread`的静态方法，用于测试当前线程（即执行该方法的线程）是否已经是中断状态，执行后将清除中断状态的标志。
* `isInterrupted()`为非静态方法，用于测试线程对象是否已经是中断状态，但不清楚状态标志。
* 线程在`sleep`状态下`interrupt`的话，会在`run()`中抛出`InterruptedException`，并且清除中断状态标志。
* 线程在`interrupt`后`sleep`的话，会在`run()`中抛出`InterruptedException`，并且清除中断状态标志。
* `yield()`方法的作用是放弃当前的CPU资源，将它让给其他的任务去占用CPU执行时间。
* 线程的优先级是继承的。
* 当最后一个非守护线程结束时，守护线程才随着JVM一同结束工作。

## 第二章 对象及变量的并发访问
* `synchronized`声明方法时，获得的是对象锁，即相当于`synchronized(this)`。
* `synchronized(this)`没有获得该对象的成员实例变量的锁，因此调用成员变量时不一定同步。
* `synchronized(string)`时要注意常量池带来的问题
* `synchronized(obj)`语句块中，若obj对象发生改变，则其他线程可以进入`synchronized`语句块。
* `volatile`要求线程每次都要从共享内存中读写变量，而不是在线程的工作内存中，因此保证了变量的可见性，但并不保证原子性，因此一般只能用于保证读取的时候数据必然是最新的。
* `synchronized`可以用于线程间的互斥，还可以确保变量在内存的可见性（**关于其保证可见性还需细看**）

## 第三章 线程间通信
* `wait()`方法是使当前执行代码的线程进行等待，将当前线程置入“预执行队列”，并且在`wait()`所在的代码行出停止执行，知道接到通知或被中断为止。在调用`wait()`之前，线程必须获得该对象的对象级别锁，即只能在同步方法或同步块中调用`wait()`方法。
* `notify()`也要在同步方法或同步块中调用。执行`notify()`方法后，当前线程不会马上释放该锁对象锁，要等到执行`notify()`方法的线程将程序执行完，也就是退出`synchronized`代码块后。
* P<sub>142</sub>（**配合第七章P<sub>280</sub>一起看**）
* `wait()`状态下的线程被interrupt的话会报`InterruptedException`。
* `wait(long)`方法的功能是超过这个时间限制后自动唤醒。
* `join()`方法的作用是使父线程等待子线程执行完毕后再执行，本质上是通过`wait`方法来阻塞父线程。（需要注意的是`join`只调用了`wait`，却没有对应的`notify`，因为在`Thread`的`start`方法中做了相应的处理，所以当join线程执行完成后会自动唤醒主线程继续往下执行）
* `join(long)`方法的作用同样是使父线程等待子线程执行完毕后再执行，但如果等待时间超出指定时间则父线程继续执行。**需要注意的是，由于`join`本质上是通过`wait`来阻塞父线程的，所以`wait`以后如果子线程长期霸占着锁不释放或者没抢到锁的话，即使超出指定时间也无法使父线程继续往下执行。** 例如如下代码：
```java
class MyThread extends Thread {
    
    private synchronized void work() {
        try {
            System.out.println("work begin!");
            Thread.sleep(5000);
            System.out.println("work end!");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void run() {
        try {
            Thread.sleep(1000); // 确保join方法先运行
            work();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
public class Test {
    
    public static void main(String[] args) {
        MyThread t = new MyThread();
        t.start();
        try {
            System.out.println("join begin!");
            t.join(2000);
            System.out.println("join end!");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

程序输出结果为：
join begin!
work begin!
work end!
join end!
```
* `ThreadLocal`类可以通过`get/set`存取每个线程的私有数据。通过重写`initialValue`方法设置`get`的默认值。
* `InheritableThreadLocal`类可以在子线程中取得父线程继承下来的值。**但子线程必须在父线程set数据后创建（注意是`new`而不是`start`）才能够继承；若要继承默认值的话，则必须等父类执行过`get`以后方可。子线程在创建后就会立刻继承父线程的数据，以后即使父线程修改了数据，子线程的数据还是旧的，除非传递的是可变对象。但若传递的是可变对象，则子线程修改对象后又会影响父线程的。**

## 第四章 Lock的使用
* `ReentrantLock`类默认使用非公平锁（**公平锁与非公平锁还需细看**）
* **`lock`与`synchronized`的区别和使用还需细看**
* `ReentrantReadWriteLock`类为读写锁，它有两个锁，一个是读操作相关的锁，也成为共享锁；一个是写操作相关的锁，也叫排他锁。读读不互斥，读写互斥，写写互斥。

## 第五章 定时器 Timer
* `Timer`类主要负责计划任务，也就是在指定的时间开始执行某一个任务。默认情况下不是守护线程。通过`schedule(TimerTask taks, Date firstTime)`方法计划任务。**若计划时间早于当前时间，则立即执行。**
* `TimerTask`是以队列的方式一个个地被顺序执行，因此执行的时间有可能和预期的时间不一致。因为前面的任务有可能消耗的时间较长，则后面的任务运行的时间也会被延误。被延误的任务在上一个任务完成后立即执行。
* `schedule(TimeTask task, Date firstTime, long period)`方法的作用是在指定的日期之后，按指定的时间间隔周期性地无限循环执行某一任务。
* `scheduleAtFixedRate`和`schedule`延时区别没看懂？（**回头细看**）
* `scheduleAtFixedRate`若计划时间早于当前时间，会把这个时间段之间应做的任务“补充性”执行回来。

## 第六章 单例模式与多线程
* 略

## 第七章 拾遗增补
* 线程有六种状态：
    * `new`，线程实例化后还从未执行start()方法
    * `runnable`，线程进入可运行状态（`ready`, `running`）
    * `blocked`，等待锁
    * `waiting`，执行了`wait()`后
    * `timed_wating`，线程执行了`sleep()`方法，呈等待状态，等待时间到达，继续向下运行
    * `terminated`，线程被销毁时的状态
* 线程组的作用是，可以批量的管理线程或线程组对象，有效地对线程或线程组对象进行组织。
* `SimpleDateFormat`不是线程安全的。
* 线程的异常处理（**有空再总结**）
