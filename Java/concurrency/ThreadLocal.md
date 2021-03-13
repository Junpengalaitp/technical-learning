# 第一章（Java并发编程之美）
## 1.11 ThreadLocal
* 提供线程本地变量，如果你创建了一个ThreadLocal变量，那么访问这个变量的每个线程都会有这个变量的一个本地副本。当多线程操作这个变量时，实际操作的是自己本地内存里的变量，从而避免了线程安全问题
### 1.11.2 ThreadLocal实现原理
* Thread类中有一个threadLocals和一个inheritableThreadLocals, 他们都是ThreadLocalMap类型的变量，初始值都为null， 在第一次调用set或get方法时会创建。
* ThreadLocalMap是一个定制化的HashMap，因为每个线程可以关联多个threadLocal变量。
* ThreadLocal只是一个工具，它通过set方法把变量存放到本地内存，当调用线程调用它的get方法时，再从当前调用线程的threadLocals变量里面将其拿出来使用。
* get和set方法都会调用Thread.currentThread()方法拿到当前线程，然后从当前线程拿到它的ThreadLocalMap进行操作。

### 1.11.4 InheritableThreadLocal
* 使子线程可以获取到父线程的本地变量
* 重写了ThreadLocal的getMap()和createMap方法，使它们指向inheritableThreadLocals
* 在Thread的默认构造器中，会获取到父线程(当前调用线程)，然后检查父线程有没有inheritableThreadLocals存在，如果有，将其复制到子线程(当前构造器线程)的inheritableThreadLocals里

## 11.10 ThreadLocal使用不当可能会导致内存泄漏
* ThreadLocalMap内部是一个Entry数组，继承自WeakReference，其内部的value用来存放ThreadLocal的set方法传递的值，ThreadLocal对象为它的key(弱引用)
* 因为key为弱引用，当它没有被其他地方引用时，会被GC，但value不会（key回收后仍然存在，key变为null）


# ThreadLocalRandom类解析
## 3.1 Random类及其局限性
* 新的随机数生成需要两个步骤
  * 首先根据老的种子来生成新的种子
  * 然后根据新的种子来计算新的随机数
* 需要保证原子性，当多个线程根据同一个老种子来计算新种子时，第一个线程的新种子算出来后，第二个线程要丢弃自己老的种子，而使用第一个线程的新种子来计算自己的新种子，这样才能保证随机性。
* 使用了AtomicLong和CAS，所以会造成大量的线程进行自旋重试，这会降低并发性能。

## 3.2 ThreadLocalRandom
* Random的缺点是多个线程会使用同一个原子性种子变量，从而导致原子变量更新的竞争。
* 如果每个线程都维护一个种子变量，则每个线程生成随机数时都根据自己老的种子计算新的种子，并使用新种子更新老种子，再根据新种子计算随机数，就不会存在竞争问题了。
* 继承自ThreadLocal，类中并没有存放具体的种子，具体的种子存放在具体的调用线程的threadLocalRandomSeed变量里面。
* 当线程调用ThreadLocalRandom.current()方法时，ThreadLocalRandom负责初始化调用线程的threadLocalRandomSeed变量。
