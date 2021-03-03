 # ThreadPoolExecutor
 * An ExecutorService that executes each submitted task using one of possibly several pooled threads, normally configured using Executors factory methods.
 * Addresses two problems
   * improve performance on asynchronous tasks
   * bounding and managing resources

## Tuning Guide
* Core and maximum pool sizes
  * core pool size: when a task is submitted by execute() method, if fewer than core pool size threads are running, new thread will be created for running this task, even if other worker thread are idle.
  * max pool size: when a task is submitted by execute() method, if all core pool threads are running, and the queue is full, and max pool size is larger than core pool size, new thread will be created for new task.
* On-demand construction
  * By default, even core threads are initially created and started only when new tasks arrive.
  * Can be overridden dynamically using prestartCoreThread() or prestartAllCoreThreads()
* Creating new threads
  * New threads are created using a ThreadFactory, default to Executors.defaultThreadFactory, which creates threads to all be in the same ThreadGroup and with the same NORM_PRIORITY and non-daemon status.
  * By supplying a different ThreadFactory, you can alter the thread's name, thread group, priority, daemon status
* Keep-alive times
  * excess non-core threads will be terminated if they have been idle for more than this time.
  * core threads can also be terminated if the allowCoreThreadTimeout() returns true.
* Queuing
  * Any BlockingQueue may be uses to transfer and hold submitted tasks.
  * three queuing strategies
    * Direct handoffs: (SynchronousQueue) the queue do not hold tasks, directly hand them to the pool, usually requires unbounded maxPoolSize
    * Unbounded queues: (LinkedBlockingQueue without a predefined capacity).
    * Bounded queues: (ArrayBlockingQueue)
* Rejected tasks
  * when the Executor has been shutdown or it has a bounded queue with maxPoolSize reached.
  * AbortPolicy: throws a runtimeException
  * CallerRunPolicy: thread run itself without the executor
  * DiscardPolicy: drop the new task that cannot be executed
  * DiscardOldestPolicy: remove the oldest(head) task in queue, and try submit new task again.

* Queue maintenance
  * use getQueue() to access work queue, mainly for monitor and debugging purposes, use this for any other reason is strongly discouraged
  * remove() and purge() method: remove cancel tasks (runnable/callable) in the queue

* Finalization
  * A pool that is no longer referenced and no remaining threads will be shutdown automatically.
  * To ensure auto finalization, you must arrange that unused threads eventually die, by setting appropriate keep-alive times(zero core threads or allowCoreThreadTimeOut() returns true)

## Run State
### states
* RUNNING: Accept new tasks and process queued tasks
* SHUTDOWN: Don't accept new tasks, but process queue tasks
* STOP: Don't accept new tasks, don't process queue tasks, and interrupt running tasks
* TIDYING: All tasks have terminated, workerCount is zero, the thread transition to state TIDYING will run the terminated() hook method
* TERMINATED: terminated() has completed
### transitions
* RUNNING -> SHUTDOWN: on invocation of shutdown(), or implicitly in finalize()
* (RUNNING/SHUTDOWN) -> STOP: on invocation of shutdownNow()
* SHUTDOWN -> TIDYING: when both queue and pool are empty
* STOP -> TIDYING: when pool is empty
* TIDYING -> TERMINATED: when the terminated() hook method has completed
  

## Task submission
* ThreadPoolExecutor实现类本身只接受runnable的类型，它可以使用submit(Callable)是其父类AbstractExecutorService实现的, 该方法会将Callable经过newTaskFor()方法转化成一个RunnableFuture类(就是一个接口同时继承了Runnable, Future接口)。再将RunnableFuture作为Runnable提交给ThreadPoolExecutor。
* 因为newTaskFor()的转换，原本的Callable如果继承的其它的接口(比如说Comparable)类型会丢失，所以需要重写newTaskFor()方法以及继承FutureTask来创造一个ComparableFutureTask
* 重写newTaskFor()会返回ComparableFutureTask对象。
* ComparableFutureTask算是装饰者模式，将继承了Comparable的Callable作为变量传入，然后ComparableFutureTask的CompareTo()方法使用这个变量的compareTo()方法比较