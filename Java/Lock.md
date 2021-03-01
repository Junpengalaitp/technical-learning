# Java并发包中的锁原理解析
## 6.1 LockSupport工具类
* 用于挂起和唤醒线程，该工具累是创建锁和其他同步类的基础。
* LockSupport类与每个使用它的线程都会关联一个许可证，在默认情况下调用它的方法的线程是不持有许可证的。
### 1. void park() 方法
* 如果调用park方法的线程已经拿到了许可证，则会马上返回，否则调用线程会被禁止参与线程的调度，也就是会被阻塞挂起。
* 默认情况下调用线程不持有许可证。
* 其他线程调用unpark(Thread thread)方法并将当前线程作为参数时，调用park方法而被阻塞的线程会返回。
* 如果其他线程调用了阻塞线程的interrupt()方法，设置了中断标志或者线程被虚假唤醒，则阻塞线程也会返回。
* 因调用park()方法而被阻塞的线程被其他线程中断而返回时并不会抛出InterruptedException
<ul>
<li>Some other thread invokes with the
current thread as the target; or
<li>Some other thread
the current thread; or
<li>The call spuriously (that is, for no reason) returns.
</ul>

### 2. void unpark(Thread thread)方法
* 当一个线程调用unpark时，如果参数thread线程没有持有与LockSupport类关联的许可证，则让thread线程持有。
* 如果thread之前因park而被挂起，则unpark后，该线程会被唤醒。
* 如果thread没有被park过，调用unpark会立即返回。

### 3. void parkNanos(long timeout)
* 和park类似，但没有许可证的情况下也会在timout后返回

### 4. park(Object blocker)方法
* 在park时记录下blocker

### 5. void parkNanos(Object blocker, long timeout)
* 在parkNanos的基础上记录blocker

### 6. parkUntil(long time)方法
* park直到epoch的time

## 6.2 抽象同步队列AQS概述
### 6.2.1 AQS: 锁的底层支持
* AQS是一个FIFO的双向队列，其内部通过节点head和tail记录首尾，队列元素的类型为Node。
* Node中的Thread变量用来存放进入AQS队列里面的线程。
  * SHARED用来标记该线程是获取共享资源时被阻塞挂起后放入AQS队列的。
  * EXCLUSIVE用来标记线程是获取独占资源时被挂起后放入AQS队列的。
* waitStatus
  * CANCELED: thread has cancelled
  * SIGNAL: indicates successor's thread needs unparking
  * CONDITION: indicates thread is waiting on condition
  * PROPAGATE: indicate the next acquireShared should unconditionally propagate.
* state
  * ReentrantLock: 可以用state表示当前线程获取锁的可重入次数
  * ReentrantReadWriteLock: state高16位表示获取读锁的次数，低16位表示获取到写锁的线程可重入次数
  * Semaphore: 表示当前可用信号的个数
  * CountdownLatch: 表示当前计数器的值
* ConditionObject
  * 用于结合锁来实现线程同步。
  * ConditionObject可以直接访问AQS对象内部的变量，比如status状态值和AQS队列。
  * ConditionObject是条件变量，每个条件变量对应一个条件队列(单向链表队列)，其用来存放调用条件变量的await方法后被阻塞的线程。
* 对于AQS来说，线程同步的关键是对状态值state进行操作。根据state是否属于一个线程，操作state的方式分为独占方式和共享方式。
  * 独占: acquire, acquireInterruptibly, release
  * 共享: acquireShared, acquireSharedInterruptibly, releaseShared
  * 使用独占方式获取的资源是与具体线程绑定的，就是说如果一个线程获取到了资源， 就会标记是这个线程获取到了，其他线程再尝试操作state获取资源时会发现当前资源不是自己持有的，就会在获取失败后被阻塞。
    * 在ReenrantLock中，当一个线程获取到锁后，在AQS内部会用CAS操作把state状态从0变成1，然后设置当前锁的持有者为当前线程，当该持有者线程再次获取它时，state会自增1。非持有者线程发现自己不是持有者是就会被放入AQS中挂起。 
    * 共享方式的锁的资源与具体线程是不相关的，当多个线程请求资源时通过CAS方式竞争获取资源，当一个资源被一个线程获取后，另一个线程再去获取时如果当前资源还能满足它的需要，则该线程只需要用CAS方式进行获取即可。
      * Semaphore: 当一个线程通过acquire()方法获取信号量时，会首先看当前信号量个数是否满足需要，不满足则把当前线程放入阻塞队列，如果满足则通过CAS获取。
* 独占(Node.EXCLUSIVE)方式下的资源获取与释放
  * 获取: 尝试acquire(int arg)获取资源，设置state的值，成功直接返回，失败则将当前线程封装为类型为Node.EXCLUSIVE的Node节点后插入到AQS阻塞队列的尾部，并调用LockSupport.park(this)挂起自己
  * 释放: 尝试release(int arg)释放资源，设置state的值，然后调用LockSupport.unpark(thread)激活一个队列里的线程。被激活的线程使用tryAcquire尝试，看当前状态变量state的值是否能满足自己的需要，满足则该线程被激活，然后继续向下运行，否则还是会被放入AQS队列并被挂起。


* 共享(Node.SHARED)方式下的资源获取与释放
  * 获取: 当线程调用acquiredShared(int arg)获取共享资源时，会首先使用tryAcquireShared尝试获取资源，具体是设置状态变量state的值，成功则直接返回，失败则将当前线程封装成Node.SHARED加入AQS队列，并用LockSupport.park(this)方法挂起自己
  * 释放: 当线程调用releaseShared(int arg)释放共享资源时，会首先使用tryReleaseShared尝试获取资源，具体是设置状态变量state的值。然后使用LockSupport.unpark(thread)激活AQS队列里被阻塞的一个线程。被激活的线程使用tryAcquire尝试，看当前状态变量state的值是否能满足自己的需要，满足则该线程被激活，然后继续向下运行，否则还是会被放入AQS队列并被挂起。

* AQS并没有提供tryAcquire，tryRelease，tryAcquireShared，tryReleaseShared方法的实现，需要具体子类来实现。state具体的意义也需要子类来定义。
* 实现子类还需要重新isHeldExclusively方法，来判断锁是被当前线程独占还是被共享。

### 6.2.2 AQS: 条件变量支持
* 和notify与wait来配合synchronized内置锁实现线程同步类似，条件变量的signal和await方法也是用来配合锁实现线程同步的基础设施。
* AQS的锁可以对于多个条件变量，synchronized只有一个
* ConditionObject是AQS的内部类，可以访问AQS的内部变量和方法。在每个条件变量内部都维护了一个条件队列，用来存放条件变量的await()方法时被阻塞的线程。
* 当线程调用条件变量的await()方法时（必须先获得锁），在内部会构造一个类型为Node.CONDITION的节点，然后将其插入条件队列尾部，之后线程会释放获取的锁，并被阻塞挂起。这时候如果有其他线程获取到锁，如果获取到锁的线程调用了条件变量的await方法，该线程也会被阻塞挂起，放入条件变量的阻塞队列，释放获取到的锁，在await()方法处阻塞
* 当另外一个线程调用条件变量的signal方法时，在内部就会把条件队列的头节点放入AQS队列，然后激活这个线程。

## 6.3 独占锁ReenrantLock的原理
### 6.3.1 类图结构
* 可重入的独占锁，同时只有一个线程可以获取该锁，其他尝试获取该锁的线程会被阻塞而放入AQS的阻塞队列里面。
* AQS的state状态值表示线程获取该锁的可重入次数。
  * 为0时不被任何线程持有
  * 被持有和重入后自增1
  * 释放时自减1，为0时释放锁(可能需要释放多次)
### 6.3.2 获取锁
1. lock()
2. lockInterruptibly()
3. tryLock()
4. tryLock(long timeout, TimeUnit unit)
### 6.3.3 释放锁
1. unlock()

## 6.4 读写锁ReentrantReadWriteLock原理
### 6.4.1 类图结构
* 读写各一个锁，ReadLock and WriteLock
* state高16位表示读状态，低16位表示写状态
### 6.4.2 写锁的获取与释放
1. void lock()
   * 独占锁，同时只有一个线程可以获取，没有线程持有写锁和读锁时可以获取
   * 可重入锁

### 6.4.3 读锁的获取与释放
* 没有线程持有读锁就可以获取，是共享锁
* 持有写锁的线程可以获取

## 6.5 StampedLock
* 提供了三种模式的读写控制，当调用获取锁的函数时，会返回一个long类型的变量(stamp)，表示了锁的状态
* 写锁
* 悲观读锁
* 乐观读锁
* 都是不可重入锁

