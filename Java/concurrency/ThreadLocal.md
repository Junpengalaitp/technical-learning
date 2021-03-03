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