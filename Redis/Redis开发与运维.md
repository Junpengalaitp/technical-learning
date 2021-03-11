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


# 第三章 小功能大用处
## 慢查询分析
* Redis执行一条命令的四步骤
  * 发送命令
  * 命令排队
  * 命令执行
  * 返回结果
* 慢查询只统计命令执行的时间

### 3.1.1 慢查询的两个配置参数
* 预设阀值
  * slowlog-log-slower-than
    * 等于0时记录所有，小于0时不记录
* 慢查询记录存放位置
  * slowlog-max-len
    * Redis用了一个列表存放log，用于设置列表最大长度
* 使用config set命令修改
* 使用config rewrite命令将配置持久化到本地配置文件
* 获取慢查询日志
  * slowlog get [n]
* 重置慢查询日志
  * slowlog reset


### 3.1.1 慢查询的最佳实践
* slowlog-max-len建议设的较大，比如1000
* slowlog-log-slower-than建议设置的较小，如1毫秒
* 因为命令执行排队机制，慢查询会导致其它命令级联阻塞，因此当客户端出现请求超时，需要检查该时间点是否有对应的慢查询。
* 由于慢查询日志时一个先进先出的队列，也就是说如果慢查询比较多的话，可能会丢失部分慢查询命令，为了防止这种情况发生，可以定期执行slow get命令将慢查询日志持久化到其它存储中

## Redis Shell
### 3.2.1 redis-cli详解
* -r: repeat, 将命令重复执行指定的次数
* -i: 结合-r使用，指定每次重复的时间间隔
* -x: 从标准输入(stdin)读取数据作为redis-cli的最后一个参数
  * echo "world" | redis-cli -x set hello
* -c: cluster, 连接Redis集群节点时使用，防止moved和ask异常
* -a: auth, 如果配置了密码，使用这个命令就不需要手动输入auth命令
* --scan和--pattern: 扫描指定模式的键，相当于使用scan命令
* --slave: 将当前客户端模拟成当前Redis节点的从节点。
* --rdb: 生成并发送RDB持久化文件
* --pipe: 将命令封装成Redis通信协议定义的数据格式，批量发送给Redis执行
* --bigkeys: 使用scan命令对Redis的键进行采样，从中找到内存占用比较大的键值
* --eval: 用于执行指定的Lua脚本
* --latency
  * --latency: 测试客户端到目标Redis的网络延时
  * --latency-history: 分时段的形式了解延时信息
  * --latency-dist: 使用统计图表形式输出统计信息
* --stat
  * 实时获取Redis的重要统计信息
* --raw和--no-raw
  * --no-raw: 返回原始格式
  * --raw: 返回格式化结果

## 3.3 Pipeline
### 3.3.1 Pipeline概念
* 发送命令+返回结果的时间成为RTT, Round Trip Time
* 非批量操作命令多次执行会消耗大量RTT时间
* Pipeline机制能够将一组redis命令组装，通过一次RTT传输给Redis
### 3.3.3 原生批量命令与Pipeline对比
* 原生命令是原子性的，Pipeline是非原子性的
* 原生命令是一个命令对应多个key, Pipeline支持多个命令
* 原生批量命令是Redis服务端支持实现的，Pipeline需要客户端和服务端共同实现

## 3.4 事务与Lua
### 3.4.1 事务
* 将一组需要一起执行的命令放到multi和exec两个命令之间
* Redis不支持回滚(发生运行时错误需要自己处理)
  
## 3.5 Bitmaps
### 3.5.1 数据结构模型
* Bitmaps本身不是一种数据结构，实际上是字符串，但是可以对其进行位操作

## 3.6 HyperLogLog
* 实际类型为字符串，其实是一种基数算法，可以利用极少的内存空间完成独立总数的统计
* 内存占用非常小，但是存在错误率
* 使用场景
  * 只是为了计算独立总数，不需要获取单条数据
  * 可以容忍一定误差率

## 3.7 发布订阅
### 3.7.1 命令
* 发布消息: publish channel message
* 订阅消息：subscribe channel [channel...]
* 客户端在执行订阅命令后就进入了订阅状态，只能接受subscribe, psubscribe, unsubscribe, punsubscribe命令
* Redis不会对消息进行持久化

## 3.8 GEO
* 支持存储地理位置信息

# 第四章 客户端

# 第五章 持久化
## 5.1 RDB(Redis Database)
* 把当前进程数据生成快照保存到硬盘的过程，可以自动或手动触发
### 5.1.1 触发机制
* 手动触发
  * save命令: 阻塞当前redis服务器，直到RDB过程完成
  * bgsave: redis进程执行fork操作创建子进程，阻塞只发生在fork阶段的很短时间
* 自动触发
  * 使用save相关配置，如“save m n”。表示m秒内数据集存在n次修改时，自动触发bgsave
  * 如果从节点执行全量复制操作，主节点自动执行bgsave生成RDB文件并发送给从节点
  * 执行debug reload命令重新加载Redis时
  * 默认情况下执行shutdown命令时，如果没有开启AOF持久化功能则自动执行bgsave

### 5.1.2 流程说明
* bgsave是主流的触发RDB方式

### 5.1.3 RDB文件的处理
* 保存: RDB文件保存在dir配置指定的目录下，文件名通过dbfilename配置指定。可通过执行config set dir {newDir}和config set dbfilename {newFilename}运行期动态执行，当下次运行时RDB文件会保存到新目录
* 压缩: Redis默认采用LZF算法对生成的RDB文件做压缩处理，压缩后的文件远远小于内存大小，默认开启。可通过参数config set rdbcompression{yes|no}动态修改。
* 校验: 如果Redis加载损坏的RDB文件时拒绝启动，可以用redis-check-dump工具进行检测

### 5.1.4 RDB的优缺点
* 优点
  * RDB是一个紧凑压缩的二进制文件，代表Redis在某个时间点上的数据快照。非常适合于备份，全量复制等场景。比如每6小时执行bgsave备份，并把RDB文件拷贝到远程机器或者文件系统中用于灾备。
  * Redis加载RDB恢复数据远远快于AOF的方式
* 缺点
  * 无法做到实时持久化，属于重量级操作
  * 不向后兼容

## 5.2 AOF(append only file)
### 5.2.1 使用AOF
* 设置配置: append only yes, 默认不开启
* AOF文件名通过appendfilename配置设置，默认文件名是appendonly.aof，保存路径同RDB一致，通过dir配置指定
* AOF工作流程
  * 命令写入(append)
  * 文件同步(sync)
  * 文件重写(rewrite)
  * 重启加载(load)
### 5.2.2 命令写入
* AOF写入的内容直接是文本协议格式
  * 文本协议具有很好的兼容性
  * 开启AOF后，所有写入的命令都包含追加操作，直接采用协议格式，避免了二次处理开销
  * 文本协议具有可读性，方便直接修改和处理
### 5.2.3 文件同步
* 同步策略
  * always: 每次命令写入aof_buf后调用系统fsync操作同步到AOF文件，fsync完成后线程返回。(不建议)
  * everysec: 命令写入aof_buf后调用系统write操作，write完成后线程返回。fsync同步文件操作由专门线程每秒调用一次。(默认配置)
  * no: 不对AOF文件做fsync同步，同步硬盘操作有操作系统负责，通常同步周期最长30秒。(提高了性能，但数据安全性无法保证)

### 5.2.4 重写机制
* 用于压缩AOF文件体积
  * 进程内已经超时的数据不再写入文件
  * 旧的AOF文件含有无效命令
  * 多条写命令可以合并为一个
* 手动触发: 直接调用bgrewriteaof
* 自动触发: 根据auto-aof-rewrite-min-size和auto-aof-rewrite-percentage参数确定自动触发时机
  * auto-aof-rewrite-min-size: 触发重写的最小体积，默认为64MB
  * auto-aof-rewrite-percentage: 代表当前AOF文件空间和上一次重写后AOF文件空间比值
* 重写流程
  * 执行AOF重写请求
    * 如果当前进程正在执行AOF重写，请求不执行
    * 如果正在执行bgsave操作，重写命令延时到bgsave完成后再执行
  * 父进程执行fork创建子进程，开销等同于bgsave过程
  * 子进程根据内存快照，按照命令合并规则写入到新的AOF文件。
  * 新得到AOF文件写入完成后，子进程发送信号给父进程，父进程更新统计信息。

### 5.2.5 重启加载
* AOF持久化开启且存在AOF文件时，优先加载AOF文件
* AOF关闭时加载RDB文件
* 加载AOF/RDB成功后，redis启动成功
* AOF/RDB文件存在错误时，Redis启动失败并打印错误信息

### 5.2.6 文件校验
* 加载损坏的AOF文件时会拒绝启动

## 5.3 问题定位与优化
### 5.3.1 fork操作
* fork创建的子进程不需要拷贝父进程的物理内存空间，但会复制父进程的空间内存页表。
* 优先使用物理机或者高效支持fork操作的虚拟化技术，避免使用Xen
* 控制Redis实例最大可用内存
* 合理配置Linux内存分配策略，避免物理内存不足导致fork失败
* 降低fork操作的频率

### 5.3.2 子进程开销的监控与优化
* CPU
* 内存
* 硬盘

### 5.3.3 AOF追加阻塞

# 第七章 Redis的噩梦: 阻塞
* 内在原因
  * 不合理地使用API或数据结构、CPU饱和、持久化阻塞
* 外在原因
  * CPU竞争、内存交换、网络问题等

## 7.1 发现阻塞
* 当Redis阻塞时，线上应用服务器应该最先感知到，这时应用方会收到大量的Redis超时异常
  * 在应用中加入异常统计以及触发报警的时机

## 7.2 内在原因
* 排查
  * API或数据结构使用不合理
  * CPU饱和问题
  * 持久化相关的阻塞

### 7.2.1 API或数据结构使用不合理
* 通常Redis执行命令速度非常快，但也存在例外，如一个包含上万元素的hash结构执行hgetall操作，这条命令的时间复杂度是O(n)
* 使用slowlog get {n} 命令获取最近的慢查询
  * 修改为低算法复杂度的命令，例如hgetall改为hmget等，禁用keys、sort命令
  * 调整大对象: 缩减大对象数据把大对象拆分为多个小对象，防止一次命令操作过多的数据
* 如何发现大对象
  * 命令: redis-cli -h {ip} p {port} --bigkeys

### 7.2.2 CPU饱和
* 使用top命令

### 7.2.3 持久化阻塞
* fork阻塞
* AOF刷盘阻塞
* HugePage写操作阻塞
  * 子进程在执行重写期间利用Linux写时复制技术降低内存开销，因此只有写操作时Redis才复制要修改的内存页

## 7.3 外在原因
### 7.3.1 CPU竞争
* 进程竞争
  * Redis是典型的CPU密集型应用，不适合和其他多核CPU密集型服务部署在一起
* 绑定CPU
  * 部署Redis时为了充分利用多核CPU，通常一台机器部署多个实例。常见的一种优化是把Redis进程绑定到CPU上，用于降低CPU频繁上下文切换的开销。
  * 开启持久化或参与复制的主节点不建议绑定CPU，因为进行重写的子进程会和父进程产生激烈的CPU竞争

### 7.3.2 内存交换(swap)
* 内存交换(swap)
  * 如果操作系统把Redis使用的部分内存换出到硬盘，由于内存与硬盘读写的速度差几个数量级，会导致发生交换后的Redis性能急剧下降。
  * 查询Redis进程号
    * redis-cli -p 6383 info server | grep process_id
  * 根据进程号查询内存交换信息
    * cat /proc/4476/smaps | grep Swap
* 预防内存交换的方法
  * 保证机器充足的可用内存
  * 确保所有Redis实例设置最大可用内存(maxmemory)，防止极端情况下Redis内存不可控的增长
  * 降低系统使用swap优先级

### 7.3.3 网络问题
* 连接拒绝
  * 网络闪断
  * Redis连接拒绝
  * 连接溢出
* 网络延迟
* 网卡软中断

### 网络问题

# 第8章 理解内存
## 8.1 内存消耗
### 8.1.1 内存使用统计
* info memory命令
* 重点关注指标
  * used_memory_rss和used_memory以及它们的比值mem_fragmentation_ratio
  * mem_fragmentation_ratio > 1 时，说明used_memory_rss - used_memory多出的部分内存并没有用于数据存储，而是被内存碎片所消耗，如果两者相差很大，说明碎片率严重
  * mem_fragmentation_ratio <> 1 时，一般出现在操作系统把Redis内存交换(Swap)到硬盘导致，出现这种情况时要格外关注

### 8.1.2 内存消耗划分
* Redis进程内消耗
  * 自身内存 + 对象内存 + 缓冲内存 + 内存碎片
* 对象内存
  * 存储用户所有的数据，内存占用最大的一块
  * 大小为key size + value size，键都是字符串类型，避免使用过长的键
* 缓冲内存
  * 客户端缓冲
    * 所有接入到Redis服务器TCP连接的输入输出缓冲，最大空间为1G，无法控制，如果超过将断开连接
    * 使用参数client-output-buffer-limit控制
      * 普通客户端: 除了复制和订阅的客户端之外的所有连接
      * 从客户端: 
      * 订阅客户端
  * 复制积压缓冲区
  * AOF缓冲区
  
### 8.1.3 子进程内存消耗

## 8.1 内存管理
### 8.2.1 设置内存上限
* 使用maxmemory参数
  * 用于缓存场景，当超出上限时使用LRU等删除策略释放空间
  * 防止所用内存超过物理服务器内存
* 限制的是Redis实际使用的内存，也就是used_memory。由于内存碎片率的存在，实际消耗的内存会更大

### 8.2.3 内存回收策略
* 删除到达过期时间的键对象
* 内存使用达到maxmemory上限时触发内存溢出控制策略