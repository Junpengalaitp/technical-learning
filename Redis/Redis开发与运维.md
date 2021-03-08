# 第一章 初识Redis
## 1.1 盛赞Redis
* 基于键值对(key-value)的NoSQL数据库
* 值类型
  * string
  * hash
  * list
  * set
  * zset
  * Bitmaps
  * HyperLogLog
  * GEO
* 附加功能
  * 键过期
  * 发布订阅
  * 事务
  * 流水线
  * Lua脚本

## 1.2 Redis特性
* 速度快，官方数据10万/s
* 基于键值对的数据结构服务器
* 丰富的功能
* 简单稳定
  * 源码少
  * 单线程模型
* 客户端语言多
* 持久化
* 主从复制
* 高可用和分布式

## 1.3 Redis使用场景
### 1.3.1 Redis可以做什么
* 缓存
* 排行榜系统
* 计数器应用
* 社交网络
* 消息队列

## 1.4 用好Redis的建议
* 切勿当作黑盒来使用，开发与运维同样重要
* 阅读源码

## 1.5 正确安装并启动Redis

## 1.6 Redis重大版本
* 和Linux版本命名规则一样，偶数为稳定版本，奇数为非稳定版本

# 第二章 API的理解和使用
## 2.1 预备
### 2.1.1 全局命令
* 查看所有键：keys *
* 键总数： dbsize
* 检查键是否存在：exists key
* 删除键：del key [key ...]
* 键过期：expire key seconds
* 键的数据结构类型：type key
### 2.1.2 数据结构和内部编码
* 每种数据结构都有多个自己底层的内部编码实现，Redis会在合适的场景选择合适的内部编码
* 通过object encoding [key]查询
* list
  * 包含了linkedlist, ziplist, quicklist
### 2.1.3 单线程架构
* 单线程模型
  * 命令到达客户端后不会被立刻执行，而是进入命令队列中被逐个执行。
* 为什么单线程还能这么快
  * 纯内存访问
  * 非阻塞I/O，使用了epoll作为I/O多路复用技术的实现，Redis的事件处理模型将epoll中的链接、读写、关闭都转换为事件，不在网络I/O中浪费过多的时间
  * 单线程避免了线程切换开销

## 2.2 字符串
* 键都是string类型，其他几种数据结构都在字符串类型基础上构建。
* 最大512MB
### 2.2.1 命令
* 常用命令
  * 设置值：set key value [ex seconds] [px milliseconds] [nx|xx]
    * ex seconds: 为键设置秒级过期时间
    * px milliseconds: 为键设置毫秒级过期时间
    * nx: 键必须不存在，才可以设置成功，用于添加
    * px: 键必须存在，才可以设置成功，用于更新
    * setex key seconds value
    * setnx key value
  * 获取值
    * get key
  * 批量设置值
    * mset key value [key value ...]
  * 批量获取值
    * mget key [key ...]
  * 计数
    * incr key
### 2.2.2 内部编码
* int: 8个字节的长整型
* embstr: 小于等于39个字节的字符串
* raw: 大于39个字节的字符串

### 2.2.3 典型使用场景
1. 缓存功能
   * 推荐的键名: 业务名:对象名:id:[属性]
2. 计数
3. 共享session
4. 限速

## 2.3 哈希
### 2.3.2 内部编码
* ziplist: 当元素个数小于512个，同时所以值都小于64字节时。更加节省内存
* hashtable: 当哈希类无法满足ziplist条件时
### 2.3.3 使用场景
* 三种方法缓存用户信息
  * 原生字符串: 每个属性一个键
  * 序列化字符串类型: 将用户信息序列化后用一个键保存
  * 哈希类型：每个用户属性用一对field-value

## 2.4 列表
### 2.4.2 内部编码
* ziplist
* linkedlist

### 使用场景
* 消息队列
* 文章列表

## 2.5 集合
### 2.5.2 内部编码
* intset
* hashtable

## 2.6 有序集合
### 2.6.2 内部编码
* ziplist
* skiplist