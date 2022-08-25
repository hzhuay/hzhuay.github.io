---
title: java-multithread
date: 2022-08-19 14:01:08
tags: Java
Categories: Java
---

创建多线程的方式：

1. 继承Thread类，重写run()，调用start()启动。
2. 实现Runnable接口（JDK8+），函数式接口可以用lambda来简化。
3. Callable
4. FutureTask

以上两种方法都是调用run方法，是没有返回值的。用Callable接口和Future类可以解决，也就是异步模型。

Callable接口与Runnable类似，但是是有返回值的，而且支持**泛型**。

```Java
@FunctionalInterface
public interface Callable<V> {
    V call() throws Exception;
}
```

`Callable`一般是配合线程池工具`ExecutorService`来使用的。`ExecutorService`可以使用`submit`方法来让一个`Callable`接口执行。它会返回一个`Future`，我们后续的程序可以通过这个`Future`的`get`方法得到结果，如果还没返回值则阻塞。

`Future`接口只有几个比较简单的方法：

```Java
public abstract interface Future<V> {
    public abstract boolean cancel(boolean paramBoolean);
    public abstract boolean isCancelled();
    public abstract boolean isDone();
    public abstract V get() throws InterruptedException, ExecutionException;
    public abstract V get(long paramLong, TimeUnit paramTimeUnit)
            throws InterruptedException, ExecutionException, TimeoutException;
}
```

如果不需要返回值，可以声明 `Future<?>`形式类型、并返回 `null`作为底层任务的结果。

FutureTask实现RunableFuture，RunableFuture继承Runnable和Future接口。FutureTask提供了Future接口中方法的默认实现。

与Callable的区别在于，submit之后Callable回得到一个Future，再用get获取结果。FutureTask在submit后没有返回值，直接用GET获取。在高并发环境下，Callable和FutureTask可能被多次创建，但FutureTask保证任务只执行一次。

### Thread类

Thread类的几个常用的方法：

- currentThread()：静态方法，返回对当前正在执行的线程对象的引用；
- start()：开始执行线程的方法，java虚拟机会调用线程内的run()方法；
- yield()：yield在英语里有放弃的意思，同样，这里的yield()指的是当前线程愿意让出对当前处理器的占用。这里需要注意的是，就算当前线程调用了yield()方法，程序在调度的时候，也还有可能继续运行这个线程的；
- sleep()：静态方法，使当前正在运行的线程睡眠一段时间；
- join()：使当前线程等待另一个线程执行完毕之后再继续执行，内部调用的是Object类的wait方法实现的；

ThreadGroup线程组。每个Thread一定属于一个线程组，如果new时没有指定，默认加入父线程线程组。线程组也可以包含其他线程组。

Java中线程可以指定优先级，从1-10，默认为5，但只是参考值，操作系统不一定支持10档划分，真正决定的是操作系统的线程调度算法。通常高优先级线程会有更高几率得到执行。

守护线程（Daemon）默认的优先级比较低。如果所有非守护线程结束，那么守护线程自动结束。应用场景为：当所有非守护线程结束时，结束其余的子线程（守护线程）自动关闭，就免去了还要继续关闭子线程的麻烦。

线程优先级不会超过所在**线程组的最大优先级**。

Java线程定义了6个状态:

```Java
// Thread.State 源码
public enum State {
    NEW,//还没调用start()。Thread内部有一个成员记录是否start，所以不能start两次
    RUNNABLE,//其实包含了运行和就绪两种状态
    BLOCKED,//等待锁
    WAITING,
    TIMED_WAITING,//超时等待，比起waiting多了超时自动唤醒
    TERMINATED;
}
```

调用如下3个方法会使线程进入waiting状态：

- Object.wait()：使当前线程处于等待状态直到另一个线程唤醒它；
- Thread.join()：等待调用的线程执行完毕，底层调用的是Object实例的wait方法；
- LockSupport.park()：除非获得调用许可，否则禁用当前线程进行线程调度。

调用如下方法会使线程进入timed_waiting状态：

- Thread.sleep(long millis)：使当前线程睡眠指定时间；
- Object.wait(long timeout)：线程休眠指定时间，等待期间可以通过notify()/notifyAll()唤醒；
- Thread.join(long millis)：等待当前线程最多执行millis毫秒，如果millis为0，则会一直执行；
- LockSupport.parkNanos(long nanos)： 除非获得调用许可，否则禁用当前线程进行线程调度指定时间；
- LockSupport.parkUntil(long deadline)：同上，也是禁止线程进行调度指定时间；

注意：只有获得锁的进程才能进入2种waiting状态。

join()方法是Thread类的一个实例方法。它的作用是让当前线程陷入“等待”状态，等join的这个线程执行完成后，再继续执行当前线程。比如我在B线程的代码中加入了A.join()，意思就是B需要等待A执行完再继续执行。应用场景：子线程进行耗时运算，而父线程需要等待运算结果进行后续处理，就可以在父线程中调用子线程的join。

Java线程间通信：

1. 锁，用synchronized给对象或者类上锁
2. 等待和通知机制。wait和notify是Object的方法，wait会让调用者释放对象锁，notify会唤醒等待对象锁的线程。由于需要操作对象锁，所以必须放在同步代码块或者同步方法中。
3. 信号量
4. 管道。管道是基于“管道流”的通信方式。JDK提供了`PipedWriter`、 `PipedReader`、 `PipedOutputStream`、 `PipedInputStream`。其中，前面两个是基于字符的，后面两个是基于字节流的。

ThreadLocal是一个本地线程副本变量工具类。内部是一个**弱引用**的Map来维护。在new Thread传入同一个ThreadLocal对象，但是各线程内的ThreadLocal实际上互相独立。

最常见的ThreadLocal使用场景为用来解决数据库连接、Session管理等。数据库连接和Session管理涉及多个复杂对象的初始化和关闭。如果在每个线程中声明一些私有变量来进行操作，那这个线程就变得不那么“轻量”了，需要频繁的创建和关闭连接。

InheritableThreadLocal类与ThreadLocal类稍有不同，Inheritable是继承的意思。它不仅仅是当前线程可以存取副本值，而且它的子线程也可以存取这个副本值。

![img](https://s2.loli.net/2022/08/20/63kOvCfoNeEXYdw.png)

Java中使用的是**共享内存并发模型**。

尽管堆是共享的，但是堆中还是会有内存不可见的问题，因为现代计算机往往会在高速缓存中缓存共享变量。

Java中每个线程有一个私有的本地内存，保存了该线程已读写的共享变量的副本，这是一个抽象的概念，并不真实存在。

Java线程之间的通信由Java内存模型（简称JMM）控制，从抽象的角度来说，JMM定义了线程和主内存之间的抽象关系。

注意，根据JMM的规定，线程对共享变量的所有操作都必须在自己的本地内存中进行，不能直接从主内存中读取。所以，线程A无法直接访问线程B的工作内存，线程间通信必须经过主内存。JMM通过控制主内存与每个线程的本地内存之间的交互，来提供内存可见性保证。

volatile关键字保证多线程操作共享变量的**可见性**以及**禁止指令重排序**，synchronized关键字不仅保证可见性，同时也保证了原子性（互斥性）。

### 重排序

编译器和处理器为了提高效率，往往会对指令做重排序。指令重排一般分三种情况：

- **编译器优化重排**：编译器在**不改变单线程程序语义**的前提下，可以重新安排语句的执行顺序。
- **指令并行重排**：现代处理器采用了指令级并行技术来将多条指令重叠执行。如果**不存在数据依赖性**(即后一个执行的语句无需依赖前面执行的语句的结果)，处理器可以改变语句对应的机器指令的执行顺序。
- **内存系统重排**：由于处理器使用缓存和读写缓存冲区，这使得加载(load)和存储(store)操作看上去可能是在乱序执行，因为三级缓存的存在，导致内存与缓存的数据同步存在时间差。

**指令重排可以保证串行语义一致，但是没有义务保证多线程间的语义也一致**。所以在多线程下，指令重排序可能会导致一些问题。

### 顺序一致性

顺序一致性模型有两大特性：

- 一个线程中的所有操作必须按照程序的顺序（即Java代码的顺序）来执行。
- 不管程序是否同步，所有线程都只能看到一个单一的操作执行顺序。即在顺序一致性模型中，每个操作必须是**原子性的，且立刻对所有线程可见**。

JMM没有保证顺序一致性，当一个线程吧数据写入本地内存，还没有刷到主内存时，这个写操作只对自己可见，其他线程看到的执行顺序时不一样对的。

在JMM中，临界区内（同步块或同步方法中）的代码可以发生重排序（但不允许临界区内的代码“逃逸”到临界区之外，因为会破坏锁的内存语义）。由此可见，JMM的具体实现方针是：在不改变（正确同步的）程序执行结果的前提下，尽量为编译期和处理器的优化打开方便之门。

对于未同步的多线程程序，JMM只提供**最小安全性**：线程读取到的值，要么是之前某个线程写入的值，要么是默认值，不会无中生有。为了实现这个安全性，JVM在堆上分配对象时，首先会对**内存空间清零**，然后才会在上面分配对象（这两个操作是同步的）。

**JMM没有保证未同步程序的执行结果与该程序在顺序一致性中执行结果一致。因为如果要保证执行结果一致，那么JMM需要禁止大量的优化，对程序的执行性能会产生很大的影响。**

1. 顺序一致性保证单线程内的操作会按程序的顺序执行；JMM不保证单线程内的操作会按程序的顺序执行。（因为重排序，但是JMM保证单线程下的重排序不影响执行结果） 
2. 顺序一致性模型保证所有线程只能看到一致的操作执行顺序，而JMM不保证所有线程能看到一致的操作执行顺序。（因为JMM不保证所有操作立即可见）
3.  JMM不保证对64位的long型和double型变量的写操作具有原子性，而顺序一致性模型保证对所有的内存读写操作都具有原子性。

JMM的规定是：对编译器和处理器来说，**只要不改变程序的执行结果（单线程程序和正确同步了的多线程程序），编译器和处理器怎么优化都行。**因此对程序员提供了**happens-before规则**：如果操作A happens-before操作B，那么操作A在内存上所做的操作对操作B都是可见的，不管它们在不在一个线程。

### 架构图

<img src="https://raw.githubusercontent.com/dunwu/images/dev/cs/java/javacore/concurrent/java-concurrent-basic-mechanism.png" alt="img" style="zoom:67%;" />



### volatile

在Java中，volatile关键字有特殊的内存语义。volatile主要有以下两个功能：

- 保证变量的**内存可见性**
- 禁止volatile变量与普通变量**重排序**（JSR133提出，Java 5 开始才有这个“增强的volatile内存语义”）

JVM保证了volatile变量的可见性，但是普通变量可能还是有问题。

JVM通过**内存屏障**来限制处理器重排。内存屏障分两种：读屏障（Load Barrier）和写屏障（Store Barrier）。内存屏障有两个作用：

1. 阻止屏障两侧的指令重排序；
2. 强制把写缓冲区/高速缓存中的脏数据等写回主内存，或者让缓存中相应的数据失效。

### synchronized和锁

Java中的锁都是基于对象的。类锁也是一种对象锁。上锁一般三种形式：

```Java
// 关键字在实例方法上，锁为当前实例
public synchronized void instanceLock() {
    // code
}

// 关键字在静态方法上，锁为当前Class对象
public static synchronized void classLock() {
    // code
}

// 关键字在代码块上，锁为括号里面的对象。这种与上一种等价
public void blockLock() {
    synchronized (this.getClass()) {
        // code
    }
}

// 关键字在代码块上，锁为括号里面的对象
public void blockLock() {
    Object o = new Object();
    synchronized (o) {
        // code
    }
}
```

Java6之后为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁“。在Java 6 以前，所有的锁都是”重量级“锁。所以在Java 6 及其以后，一个对象其实有四种锁状态，它们级别由低到高依次是：

1. 无锁状态
2. 偏向锁状态
3. 轻量级锁状态
4. 重量级锁状态

这里有一段锁升级和降级的知识

#### 对象头

Java锁基于对象，那么对象里肯定有地方保存了锁的信息。每个Java对象都有对象头。如果是非数组类型，则用2个字宽来存储对象头，如果是数组，则会用3个字宽来存储对象头。子宽取决于操作系统位数，内容为：

- Mark World：存储对象的hashCode或锁信息等
- Class Metadata Adddress：存储到对象类型数据的指针
- Array Length：数组长度（只有数组类型才有）

Mark World格式：

<img src="https://raw.githubusercontent.com/dunwu/images/dev/snap/20200629191250.png" alt="img" style="zoom:67%;" />

#### 偏向锁

Hotspot的作者经过以往的研究发现大多数情况下**锁不仅不存在多线程竞争，而且总是由同一线程多次获得**，于是引入了偏向锁。

偏向锁会偏向于第一个访问锁的线程，如果在接下来的运行过程中，该锁没有被其他的线程访问，则持有偏向锁的线程将永远不需要触发同步。也就是说，**偏向锁在资源无竞争情况下消除了同步语句，连CAS操作都不做了，提高了程序的运行性能。**

大白话就是对锁置个变量，如果发现为true，代表资源无竞争，则无需再走各种加锁/解锁流程。如果为false，代表存在其他线程竞争资源，那么就会走后面的流程。

**实现原理**：一个线程在第一次进入同步块时，会在对象头和栈帧中的**锁记录**里存储锁的偏向的**线程ID**。当下次该线程进入这个同步块时，会去检查锁的Mark Word里面是不是放的自己的线程ID。如果是，表明该线程已经获得了锁，以后该线程在进入和退出同步块时不需要花费CAS操作来加锁和解锁 ；如果不是，就代表有另一个线程来竞争这个偏向锁。这个时候会尝试使用CAS来替换Mark Word里面的线程ID为新线程的ID，这个时候要分两种情况：

- 成功，表示之前的线程不存在了， Mark Word里面的线程ID为新线程的ID，锁不会升级，仍然为偏向锁；
- 失败，表示之前的线程仍然存在，那么暂停之前的线程，设置偏向锁标识为0，并设置锁标志位为00，升级为轻量级锁，会按照轻量级锁的方式进行竞争锁。

偏向锁使用了一种**等到竞争出现才释放锁的机制**，所以当其他线程尝试竞争偏向锁时， 持有偏向锁的线程才会释放锁。偏向锁升级成轻量级锁时，会暂停拥有偏向锁的线程，重置偏向锁标识，**开销很大**。如果应用程序里所有的锁通常出于竞争状态，那么偏向锁就会是一种累赘，可以选择一开始就把偏向锁这个默认功能给关闭。

#### 轻量级锁

多个线程在不同时段获取同一把锁，即不存在锁竞争的情况，也就没有线程阻塞。针对这种情况，JVM采用轻量级锁来避免线程的阻塞与唤醒。

JVM会为每个线程在当前线程的栈帧中创建用于**存储锁记录的空**间，我们称为Displaced Mark Word。如果一个线程获得锁的时候发现是轻量级锁，会把锁的Mark Word复制到自己的Displaced Mark Word里面。

#### 重量级锁

重量级锁依赖于操作系统的互斥量（mutex） 实现的，而操作系统中线程间状态的转换需要相对比较长的时间，所以重量级锁效率很低，但被阻塞的线程不会消耗CPU。

#### 锁消除

锁消除是指对于被检测出不可能存在竞争的共享数据的锁进行消除。JIT 编译器在动态编译同步块的时候，借助了一种被称为逃逸分析的技术，来判断同步块使用的锁对象是否只能够被一个线程访问，而没有被发布到其它线程。

#### 锁粗化

在 JIT 编译器动态编译时，如果发现几个相邻的同步块使用的是同一个锁实例，那么 JIT 编译器将会把这几个同步块合并为一个大的同步块，从而避免一个线程“反复申请、释放同一个锁“所带来的性能开销。

### Lock

synchronized 无法通过**破坏不可抢占条件**来避免死锁。原因是 synchronized 申请资源的时候，如果申请不到，线程直接进入阻塞状态了，而线程进入阻塞状态，啥都干不了，也释放不了线程已经占有的资源。

与内置锁 `synchronized` 不同的是，**`Lock` 提供了一组无条件的、可轮询的、定时的以及可中断的锁操作**，所有获取锁、释放锁的操作都是显式的操作。

接口：

- `lock()` - 获取锁。
- `unlock()` - 释放锁。
- `tryLock()` - 尝试获取锁，仅在调用时锁未被另一个线程持有的情况下，才获取该锁。
- `tryLock(long time, TimeUnit unit)` - 和 `tryLock()` 类似，区别仅在于限定时间，如果限定时间内未获取到锁，视为失败。
- `lockInterruptibly()` - 锁未被另一个线程持有，且线程没有被中断的情况下，才能获取锁。
- `newCondition()` - 返回一个绑定到 `Lock` 对象上的 `Condition` 实例。

### Condition

Condition 实现了管程模型里面的条件变量。

内置锁（`synchronized`）配合内置条件队列（`wait`、`notify`、`notifyAll` ），显式锁（`Lock`）配合显式条件队列（`Condition` ）。

```java
public interface Condition {
    void await() throws InterruptedException;//类似wait
    void awaitUninterruptibly();
    long awaitNanos(long nanosTimeout) throws InterruptedException;
    boolean await(long time, TimeUnit unit) throws InterruptedException;
    boolean awaitUntil(Date deadline) throws InterruptedException;
    void signal();//类似notify
    void signalAll();//类似notifyAll
}
```

### ReentrantLock

获取锁操作 `lock()` 必须在 `try catch` 块中进行，并且将释放锁操作 `unlock()` 放在 `finally` 块中进行，以保证锁一定被被释放，防止死锁的发生。

与无条件获取锁相比，tryLock 有更完善的容错机制。

- `tryLock()` - **可轮询获取锁**。如果成功，则返回 true；如果失败，则返回 false。也就是说，这个方法**无论成败都会立即返回**，获取不到锁（锁已被其他线程获取）时不会一直等待。
- `tryLock(long, TimeUnit)` - **可定时获取锁**。和 `tryLock()` 类似，区别仅在于这个方法在**获取不到锁时会等待一定的时间**，在时间期限之内如果还获取不到锁，就返回 false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回 true。
- `lockInterruptibly()` - **可中断获取锁**。可中断获取锁可以在获得锁的同时保持对中断的响应。

### CAS

CAS: Compare and Swap

> 比较并设置。用于在硬件层面上提供原子性操作。在 Intel 处理器中，比较并交换通过指令cmpxchg实现。 比较是否和给定的数值一致，如果一致则修改，不一致则不修改。

在Java中，如果一个方法是native的，那Java就不负责具体实现它，而是交给底层的JVM使用c或者c++去实现。Java实现CAS使用`Unsafe`类，里面有一些native方法（对于int，long，object等类型的CAS操作，即多线程安全的赋值），是C++实现的，它的具体实现和操作系统、CPU都有关系。

JDK提供了一些用于原子操作的类，在`java.util.concurrent.atomic`包下面。JDK17中有17个类，类名都是AtomicXXX，主要是四中类型：

- 原子更新基本类型
- 原子更新数组
- 原子更新引用
- 原子更新字段（属性）

比如JDK1.8之后`AtomicInteger`类的`getAndAdd(int delta)`方法是调用`Unsafe`类的方法来实现的。

```java
@HotSpotIntrinsicCandidate
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!weakCompareAndSetInt(o, offset, v, v + delta));
    return v;
}
```

CAS有三大问题：

1. ABA问题。就是一个值原来是A，变成了B，又变回了A。解决思路是在变量前面追加上**版本号或者时间戳**。从JDK 1.5开始，JDK的atomic包里提供了一个类`AtomicStampedReference`类来解决ABA问题。这个类的`compareAndSet`方法的作用是首先检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果二者都相等，才使用CAS设置为新的值和标志。

2. 循环时间长开销大。CAS多与自旋结合。如果自旋CAS长时间不成功，会占用大量的CPU资源。解决思路是让JVM支持处理器提供的**pause指令**。pause指令能让自旋失败时cpu睡眠一小段时间再继续自旋，从而使得读操作的频率低很多。
3. 只能保证一个共享变量的原子操作：使用JDK 1.5开始就提供的`AtomicReference`类保证对象之间的原子性，把多个变量放到一个对象里面进行CAS操作；使用锁。锁内的临界区代码可以保证只有当前线程能操作。

### AQS

**AQS**是`AbstractQueuedSynchronizer`的简称，即`抽象队列同步器`。AQS是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的同步器，比如我们提到的ReentrantLock，Semaphore，ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。我们只需要实现几个protected方法也可以实现自定义的同步器。

AQS内部使用了一个volatile的变量state来作为资源的标识。还有几个protected方法来获取和修改state，基于Unsafe的CAS实现的。AQS类本身实现的是一些**排队和阻塞**的机制，比如具体线程等待队列的维护（如获取资源失败入队/唤醒出队等）。它内部使用了一个**先进先出（FIFO）的双端队列**，并使用了两个指针head和tail用于标识队列的头部和尾部。其数据结构如图：

<img src="https://3803541485-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-L_5HvtIhTFW9TQlOF8e%2F-L_5TIKcBFHWPtY3OwUo%2F-L_5TJQjOPACL_iNG1yE%2FAQS数据结构.png?generation=1551665548550906&alt=media" alt="img" style="zoom:67%;" />

队列中并不直接储存线程，而是储存**拥有线程的Node节点**。

#### 资源共享

资源有两种共享模式，或者说两种同步方式：

- 独占模式（Exclusive）：资源是独占的，一次只能一个线程获取。如ReentrantLock。
- 共享模式（Share）：同时可以被多个线程获取，具体的资源个数可以通过参数指定。如Semaphore/CountDownLatch。



## 线程池

线程池顶层接口是`Executor`，`ThreadPoolExecutor`是这个接口的实现类。构造函数有四种，最少5个参数，最多7个参数。

```Java
public ThreadPoolExecutor(int corePoolSize,//核心线程数最大值
                          int maximumPoolSize,//线程总数最大值（核心线程+非核心线程）
                          long keepAliveTime,//非核心线程闲置超时时长。
                          TimeUnit unit,//时间单位
                          BlockingQueue<Runnable> workQueue,//阻塞队列
                          ThreadFactory threadFactory,//线程工厂，用于统一管理创建线程的配置
                          RejectedExecutionHandler handler)//拒绝处理策略
```

线程池中有两类线程，核心线程和非核心线程。

- 核心线程默认情况下会一直存在于线程池中，即使这个核心线程什么都不干（铁饭碗）
- 非核心线程如果长时间的闲置，就会被销毁（临时工）。

**BlockingQueue workQueue**：阻塞队列，维护着**等待执行的Runnable任务对象**。

常用的几个阻塞队列：

1. **LinkedBlockingQueue**链式阻塞队列，底层数据结构是链表，默认大小是`Integer.MAX_VALUE`，也可以指定大小。
2. **ArrayBlockingQueue**数组阻塞队列，底层数据结构是数组，需要指定队列的大小。
3. **SynchronousQueue**同步队列，内部容量为0，每个put操作必须等待一个take操作，反之亦然。
4. **DelayQueue**延迟队列，该队列中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素 。



线程数量大于最大线程数就会采用**拒绝处理策略**，四种拒绝处理的策略为 ：

1. **ThreadPoolExecutor.AbortPolicy**：**默认拒绝处理策略**，丢弃任务并抛出RejectedExecutionException异常。
2. **ThreadPoolExecutor.DiscardPolicy**：丢弃新来的任务，但是不抛出异常。
3. **ThreadPoolExecutor.DiscardOldestPolicy**：丢弃队列头部（最旧的）的任务，然后重新尝试执行程序（如果再次失败，重复此过程）。
4. **ThreadPoolExecutor.CallerRunsPolicy**：由调用线程处理该任务。



#### 线程池状态

线程池本身有一个调度线程，负责创建、销毁、管理任务队列等。

线程池有自己的状态，`ThreadPoolExecutor`类中定义了一个`volatile int`变量**runState**来表示线程池的状态 ，分别为RUNNING、SHURDOWN、STOP、TIDYING 、TERMINATED。

- 线程池创建后处于**RUNNING**状态。
- 调用shutdown()方法后处于**SHUTDOWN**状态，线程池不能接受新的任务，清除一些空闲worker,会等待阻塞队列的任务完成。
- 调用shutdownNow()方法后处于**STOP**状态，线程池不能接受新的任务，中断所有线程，阻塞队列中没有被执行的任务全部丢弃。此时，poolsize=0,阻塞队列的size也为0。
- 当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为**TIDYING**状态。接着会执行terminated()函数。ThreadPoolExecutor中有一个控制状态的属性叫ctl，它是一个AtomicInteger类型的变量。
- 线程池处在TIDYING状态时，**执行完terminated()方法之后**，就会由 **TIDYING -> TERMINATED**， 线程池被设置为TERMINATED状态。

#### 线程如何复用

ThreadPoolExecutor在创建线程时，会将线程封装成**工作线程worker**,并放入**工作线程组**中，然后这个worker反复从阻塞队列中拿任务去执行。

### 阻塞队列

BlockingQueue是Java util.concurrent包下重要的数据结构，区别于普通的队列，BlockingQueue提供了**线程安全的队列访问方式**，并发包下很多高级同步类的实现都是基于BlockingQueue实现的。

比起普通队列的插入和删除方法，阻塞队列提供了抛出异常、返回特殊值、阻塞、超时退出4中版本。

实现类：

- ArrayBlockingQueue：由数组形成的有界阻塞的队列。初始化队列大小后不可改变，构造方法中的fair决定内部锁是否采用公平锁，默认非公平。
- LinkedBlockingQueue：链表结构的有界阻塞队列。默认大小Integer.MAX_VALUE
- DelayQueue：该队列中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素 。
- PriorityBlockingQueue
- SynchronousQueue：没有任何内部内容，对列没有容量，每个put必须等待一个take

阻塞队列的原理是利用的Lock锁的多条件阻塞控制。

在构造时，除了初始化队列的大小和是否是公平锁之外，还对同一个锁（lock）初始化了两个监视器（Condition），分别是notEmpty和notFull。这两个监视器的作用目前可以简单理解为标记分组，当该线程是put操作时，给他加上监视器notFull，标记这个线程是一个生产者；当线程是take操作时，给他加上监视器notEmpty，标记这个线程是消费者。

## 锁的分类

- **可重入锁和非可重入锁**：是否允许线程对一个资源重复加锁。synchronized关键字就是使用的重入锁。比如说，你在一个synchronized实例方法里面调用另一个本实例的synchronized实例方法，它可以重新进入这个锁，不会出现任何异常。不然就会导致**死锁**。

- **公平锁和非公平锁**：获取锁的顺序是否FIFO。一般情况下，**非公平锁能提升一定的效率。但是非公平锁可能会发生线程饥饿（有一些线程长时间得不到锁）的情况**。所以要根据实际的需求来选择非公平锁和公平锁

    ReentrantLock支持非公平锁和公平锁两种。

- **读写锁和排他锁**：Java提供了ReentrantReadWriteLock类作为读写锁的默认实现，内部维护了两个锁：一个读锁，一个写锁。通过分离读锁和写锁，使得在“读多写少”的环境下，大大地提高了性能。

    synchronized和ReentrantLock都是排他锁。

### 接口

juc.locks包下共有三个接口：`Condition`、`Lock`、`ReadWriteLock`。Lock接口里面有一些获取锁和释放锁的方法声明，而ReadWriteLock里面只有两个方法，分别返回“读锁”和“写锁”。

Lock接口中有一个方法是可以获得一个`Condition`。每个对象都可以用继承自`Object`的**wait/notify**方法来实现**等待/通知机制**。而Condition接口也提供了类似Object监视器的方法，通过与**Lock**配合来实现等待/通知模式。

Condition和Object的wait/notify基本相似。其中，Condition的await方法对应的是Object的wait方法，而Condition的**signal/signalAll**方法则对应Object的notify/notifyAll()。但Condition类似于Object的等待/通知机制的加强版。



ReentrantReadWriteLock实现了读写锁，但是在写操作时是独占的，可能导致“写饥饿”。

`StampedLock`类是Java8加入的，没有实现Lock接口和ReadWriteLock接口，但是实现了读写锁的功能，而且性能比ReentrantReadWriteLock更高。StampedLock还把读锁分为了“乐观读锁”和“悲观读锁”两种。

StampedLock可以避免“写饥饿”，核心思想在于，**在读的时候如果发生了写，应该通过重试的方式来获取新的值，而不应该阻塞写操作。这种模式也就是典型的无锁编程思想，和CAS自旋的思想一样**。这种操作方式决定了StampedLock在读线程非常多而写线程非常少的场景下非常适用，同时还避免了写饥饿情况的发生。

乐观读锁的意思就是先假定在这个锁获取期间，共享变量不会被改变，既然假定不会被改变，那就不需要上锁。在获取乐观读锁之后进行了一些操作，然后又调用了validate方法，这个方法就是用来验证tryOptimisticRead之后，是否有写操作执行过，如果有，则获取一个悲观读锁，这里的悲观读锁和ReentrantReadWriteLock中的读锁类似，也是个共享锁。



Vector和HashTable是线程安全的容器，但是同步方法是**对方法加锁**，性能不好。

- ConcurrentMap接口

ConcurrentHashMap类提供了一种与HashTable完全不同的加锁策略提供更高效的并发性和伸缩性。供了一种粒度更细的加锁机制来实现在多线程下更高的性能，这种机制叫**分段锁**(Lock Striping)。

- ConcurrentLinkedDeque和ConcurrentLinkedQueue。没有并发List。
- ConcurrentSkipListSet

### CopyOnWrite容器

CopyOnWrite容器即**写时复制的容器**,当我们往一个容器中添加元素的时候，不直接往容器中添加，而是将当前容器进行copy，复制出来一个新的容器，然后向新容器中添加我们需要的元素，最后将原容器的引用指向新容器。这样在并发场景对读操作不需要加锁，从而达到**读写分离**。

JDK1.5加入了CopyOnWriteArrayList和CopyOnWriteArraySet 

CopyOnWrite容器有**数据一致性**的问题，它只能保证**最终数据一致性**。读操作可能读到旧数据。



**工作窃取算法**指的是在多线程执行不同任务队列的过程中，某个线程执行完自己队列的任务后从其他线程的任务队列里窃取任务来执行。当一个线程窃取另一个线程的时候，为了减少两个任务线程之间的竞争，我们通常使用**双端队列**来存储任务。被窃取的任务线程都从双端队列的**头部**拿任务执行，而窃取其他任务的线程从双端队列的**尾部**执行任务。
