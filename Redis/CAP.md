# CAP原理
* 分布式存储的理论基石。
* Consistency
* Availability
* Partition tolerance
* 当网络分区发生时，一致性和可用性难以两全。
  * 只能保证三个中的两个，CA,CP或AP
* Redis主从数据是异步同步的，所以分布式Redis不满足一致性要求。但它满足可用性
* Redis保证最终一致性，从节点会努力追赶主节点。