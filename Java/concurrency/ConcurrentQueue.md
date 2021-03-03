## 7.1 ConcurrentLinkedQueue
* 使用单向链表的无界队列非阻塞队列
* 使用volatile变量修饰首尾节点，使用CAS操作它们
  * 两个线程可以分别操作头尾节点，性能较高。
  * size()方法需要遍历，O(n) time并且不准确

## BlockingQueue
* 相对于Queue, 增加了2种功能
  * 当队列为空时，进行dequeue相关操作会等待队列有元素进来
  * 当队列满时，进行enqueue相关操作会等待队列可用
* 四种类型的操作
  * Throws exception
    * add(e)
    * remove()
    * element()
  * Special Value
    * offer(e)
    * poll()
    * take()
  * Blocks
    * put(e)
    * take()
  * Times out
    * offer(e, time, unit)
    * poll(time, unit)
* Thread safe, but bulk operations in some implementations is not atomic


## 7.2 ConcurrentBlockingQueue
* 使用单向链表的有界队列阻塞队列
* 使用两个ReenrantLock操作首尾节点
  * 两个线程可以分别操作头尾节点，性能较高
  * size由一个atomicInteger记录, O(1)time, 准确
  
## 7.3 ArrayBlockingQueue
* 使用数组的有界阻塞队列
* 对整个数组加锁，只有一个线程可以增删元素

## 7.4 PriorityBlockingQueue
* 相同优先级不保证FIFO, 因为入队是按序的
* 对整个队列加锁，只有一个线程可以增删元素，size也是准确的

## 7.5 DelayQueue
* 无界阻塞延时队列，只有过期元素才会出队
