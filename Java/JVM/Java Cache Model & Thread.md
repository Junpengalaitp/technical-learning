# Java内存模型与线程
## 概述
* CPU的运算速度快出存储和通信系统太多，大量的时间浪费在磁盘I/O，网络通信或者数据库访问上。
* 需要将CPU等待其他资源的时间压榨出来来提高效率。多线程并发是一个好的压榨方式。
* Transactions Per Second (TPS)
  * 代表一秒内服务端平均能相应的请求总数

## 硬件的效率和一致性
* 绝大多数的运算任务都不可能只靠CPU计算就能完成。现代计算机都使用读写速度接近与CPU的高速缓存来所为内存与处理器之间的缓冲。
* 缓存一致性(Cache Coherence)
  * 每个处理器都有自己的高速缓存，又共享同一主存(Main Memory)。(共享内存多核系统 Shared Memory Multiprocessors System)。
* 内存模型
  * 在特定的操作协议下，对特定内存或高速缓存进行读写访问的过程抽象。JVM有自己的内存模型。
* CPU对乱序执行的优化(Out-Of-Order Execution)
  * JVM指令重排序(Instruction Reorder)

## Java内存模型
* 主要目的是定义程序中各种变量的访问规则，即JVM把变量值存储到内存和从内存中取出变量值这样的底层细节。

### 主内存与工作内存
* Java内存模型规定了所有的变量都存储在主内存中，每条线程还有自己的工作内存，其中保存了被该线程使用的变量和主内存副本。
* 线程对变量的所有操作都必须在工作内存中进行，而不能直接读写主内存中的数据。
  * volatile变量依然操作的是工作内存，但是每次使用前都要从主内存获取刷新

### 先行发生原则(Happens-Before)
* Java自带的先行发生关系
  * 程序次序规则(Program Order Rule)
    * 单线程内部代码的执行是有序的
  * 管程锁定规则(Monitor Lock Rule)
    * 同一个锁的unlock操作先行发生于后面的lock操作
  * volatile变量规则(Volatile Variable Rule)
    * 同个volatile变量的写操作先行发生与对这个变量的读操作
  * 线程启动规则(Thread Start Rule)
    * Thread.start()方法先行发生于此线程的其他所有操作
  * 线程终止规则(Thread Termination Rule)
    * 此线程的其他所有操作先行发生于Thread.stop()方法
  * 线程中断规则(Thread Interruption Rule)
    * Thread.interrupt()方法先行发生于线程对中断检测的方法
  * 对象终结规则(Finalizer Rule)
    * 对象的初始化先行发生于它的finalize()方法
  * 传递性(Transitivity)
    * 如果A先于B，B先于C，那么A先于C
* 先行发生保证了代码执行顺序和时间上发生的顺序一致

## Java与线程
### 线程的实现
* 内核线程实现(1:1实现)
  * Kernel-Level Thread, KLT, 直接由操作系统内核支持的线程。
  * 由内核完成线程切换，内核通过操作调度器对线程调度，并负责将线程的任务映射到各个处理器上。
  * 每个内核线程可被视为内核的一个分身，这样操作系统就有能力同时处理多个事情。
  * 程序一般不会直接使用内核线程，而是使用它的高级接口：轻量级进程，也就是线程。
  * 每个轻量级进程都是一个独立的调度单元，即使其中某一个被阻塞了，也不影响整个进程继续工作。
  * 各种线程操作都需要进行系统调用，需要在用户态和内核态之间切换，系统调用代价较高。
  * 消耗内核资源，所以它的数量有限。

* 用户线程实现(1:N实现)
  * 广义上，不是内核线程的线程都可以被认为是用户线程
  * 狭义上的用户线程是完全建立在用户线程的线程库上，系统内核感知不到。
  * 线程操作在用户态中实现，不需要内核的帮助。因此是低消耗和快速的。
  * 劣势是所有的线程操作都要用户自己去处理，实现复杂。

* 混合实现(N:M实现)
  * 将内核线程和用户线程结合起来实现。对两者取优避劣

* Java线程的实现
  * 早期(JDK1.2以前)基于叫作绿色线程的用户线程实现
  * JDK1.3后基于操作系统原生线程来实现，即1:1的线程模型。
  * HotSpot JVM的每个线程都是直接映射到一个操作系统原生线程来实现的，而且中间没有额外的间接结构，JVM自己不会干涉线程调度(可以设置线程优先级)，全权交给OS管理。
  * 其他类型的JVM支持其他线程实现
    * CLDC-HI: 支持1:N实现
    * Solaris: 支持N:M实现

### Java线程调度