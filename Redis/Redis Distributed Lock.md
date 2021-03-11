* 使用setnx和expire结合的原子命令
  * set lock true ex 5 nx
  * del lock
* 可重入性
  * 对客户端的set方法进行封装，使用线程的ThreadLocal变量存储当前持有锁的计数