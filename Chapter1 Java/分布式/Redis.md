# 第2讲：Redis

## 1.缓存

- 穿透查询

  缓存层没有，到存储层查询

- 回种

  存储层存到缓存



## 2.Memcache和Redis区别

---

### **Memcache**

代码层次类似hash

- 支持简单数据类型
- 不支持数据持久化存储
- 不支持主从
- 不支持分片

---

### **redis**

- 数据类型丰富
- 支持数据磁盘持久化存储
- 支持主从
- 支持分片

---

## 3.为什么快

- 完全基于内存
- 数据结构简单，对数据操作简单 key-value
- 采用单线程架构，可以处理高并发
- 使用多路I/O复用模型，非阻塞I/O

> 传统阻塞I/O模型
>
> ![image-20200512203051053](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/image-20200512203051053.png)
>
> 多路I/O复用模型
>
> redis 内部使用文件事件处理器 `file event handler`，这个文件事件处理器是单线程的，所以 redis 才叫做单线程的模型。它采用 IO 多路复用机制同时监听多个 socket，根据 socket 上的事件来选择对应的事件处理器进行处理。
>
> ![Untitled Diagram](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/Untitled%20Diagram.png)
>
> 客户端 socket01 向 redis 的 server socket 请求建立连接，此时 server socket 会产生一个 `AE_READABLE` 事件，IO 多路复用程序监听到 server socket 产生的事件后，将该事件压入队列中。文件事件分派器从队列中获取该事件，交给`连接应答处理器`。连接应答处理器会创建一个能与客户端通信的 socket01，并将该 socket01 的 `AE_READABLE` 事件与命令请求处理器关联。
>
> 假设此时客户端发送了一个 `set key value` 请求，此时 redis 中的 socket01 会产生 `AE_READABLE` 事件，IO 多路复用程序将事件压入队列，此时事件分派器从队列中获取到该事件，由于前面 socket01 的 `AE_READABLE` 事件已经与命令请求处理器关联，因此事件分派器将事件交给命令请求处理器来处理。命令请求处理器读取 socket01 的 `key value` 并在自己内存中完成 `key value` 的设置。操作完成后，它会将 socket01 的 `AE_WRITABLE` 事件与命令回复处理器关联。
>
> 如果此时客户端准备好接收返回结果了，那么 redis 中的 socket01 会产生一个 `AE_WRITABLE` 事件，同样压入队列中，事件分派器找到相关联的命令回复处理器，由命令回复处理器对 socket01 输入本次操作的一个结果，比如 `ok`，之后解除 socket01 的 `AE_WRITABLE` 事件与命令回复处理器的关联。
>
> Redis因地制宜采用各种多路复用函数 epoll/kqueue/evport/select
>
> 优先选择时间复杂度为O(1)的，O(n)的select保底

## 4.数据类型

- String：最基本的数据类型，二进制安全

- Hash：String元素组成的字典，适合用于存储对象

  > 设置hashset
  >
  > hmset lilei name "lilei" age 26
  >
  > 获取
  >
  > hget lilei age  

- List：列表，按照String元素插入顺序排序

  > 添加
  >
  > lpush listname element
  >
  > 查询列表中元素
  >
  > lrange mylist 0 10

- Set：String元素组成的无序集合，通过哈希表实现，不允许重复

  > 添加
  >
  > sadd setname element
  >
  > 查询
  >
  > smember setname  

- Sorted Set：通过分数来为集合中的成员进行从小到大的排序

  > zadd setname score element
  >
  > zrangebyscore setname 0 10

- HyperLog,Geo

## 5.持久化

### **RDB持久化**

Redis可以通过创建快照来获得存储在内存里面的数据在某个时间点上的副本。Redis创建快照之后，可以对快照进行备份，可以将快照复制到其他服务器从而创建具有相同数据的服务器副本（Redis主从结构，主要用来提高Redis性能），还可以将快照留在原地以便重启服务器的时候使用。

快照持久化是Redis默认采用的持久化方式，在redis.conf配置文件中默认有此下配置：

```conf
save 900 1           #在900秒(15分钟)之后，如果至少有1个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

save 300 10          #在300秒(5分钟)之后，如果至少有10个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，Redis就会自动触发BGSAVE命令创建快照。Copy to clipboardErrorCopied
```

SAVE：阻塞Redis服务器进程，知道RDB文件被创建完毕

BGSAVE：Fork出一个子进程创建RDB文件，不阻塞

自动触发的方式

- redis.conf配置里的SAVE m n定时触发(用的是BGSAVE)
- 主从复制时，主节点自动触发
- 执行debug reload
- 执行shutdown且没有开启AOF持久化

![redis-BGSAVE](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/redis-BGSAVE.png)

**缺点：**

- 内存数据的全量同步，数据量大会由于I/O而严重影响性能
- 可能会因为Redis挂掉而丢失从当前至最近一次快照间的数据

---

### **AOF持久化**

 AOF( Append-ony-Fie)持久化:保存写状态

- 记录下除了查询以外的所有变更数据库状态的指令
- 以 append的形式追加保存到AOF文件中(增量)

日志重写AOF文件大小不断增大的问题，原理如下

- 调用fork0),创建一个子进程
- 主进程持续将新的变动同时写到内存和原来的AOF里
- 主进程获取子进程重写AOF的完成信号,往新AOF同步增量变动
- 使用新的AOF文件替换掉旧的AOF文件

![redisAOFrewrite](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/redisAOFrewrite.png)

---

### **RDB和AOF的优缺点**

- RDB优点:全量数据快照,文件小,恢复快
- RDB缺点:无法保存最近一次快照之后的数据
- AOF优点:可读性高,适合保存增量数据,数据不易丟失
- AOF缺点:文件体积大,恢复时间长

---

### **RDB-AOF混合持久化**

BGSAVE做镜像全量持久化，AOF做增量持久化

## 6.redis的同步机制

---

### **主从同步原理**

分为全同步和增量同步

- 全同步
  - slave发送sync到master
  - master调用BGSAVE
  - master将保存数据快照期间接收到的写命令缓存起来
  - master完成写，将文件发给slave
  - slave用新的RDB文件替换掉旧的RDB文件
  - master将这期间收到的写操作发到slave
- 增量同步
  - master收到用户的操作指令，判断是否需要传播到slave
  - 将操作记录追加到AOF
  - 将操作传播到其他SLAVE：1、对齐主从库；2、往响应缓存写入指令
  - 将缓存中数据发送给slave

**弊端**

master挂掉slave就无法写入了

---

### **Redis sentinel**

解决主从同步master宕机后的主从切换问题

- 监控：检查主从服务器是否正常运行
- 提醒：通过API向管理员或其他应用程序发送故障通知
- 自动故障迁移：主从切换

---

### **redis集群**

如何从海量数据中快速找到所需？

无中心结构

分片:按照某种规则去划分数据,分散存储在多个节点上，使用goosip协议

常规哈希无法实现节点的动态增减

一致性哈希算法：对2^32去模，将哈希值空间组织成虚拟的圆环

1.将每台节点经过计算放到各自位置上

2.对数据key使用相同的函数Hash计算出哈希值，顺时针寻找最近的节点。

新增或减少节点只对很少一部分数据有效。

**缺点**

可能出现数据倾斜

引入虚拟节点解决数据倾斜的问题



## 7.面试题

---

### **从海量key中查询出某一固定前缀的key**

摸清数据规模，问清楚边界

- keys pattern

  一次性返回所有匹配的key

  键数量过大会使服务卡顿

- scan cursor [MATCH pattern] [count count]

  基于游标的迭代器，需要基于上次游标延续之前的迭代过程

  以0作为游标开始一次新的迭代，直到命令返回游标0完成一次遍历

  不保证每次执行都返回某个给定数量的元素，支持糢糊查询

  不可控返回数量，只能是大概率符合count

  scan 0 match k1* count 10



---

### **如何通过Redis实现分布式锁**

互斥锁、安全性、死锁、容错



SETNX key value 如果key不存在，则创建并赋值 

Expire key second

![image-20200513104653743](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/image-20200513104653743.png)



改进

SET key value [EX seconds] [PX milliseconds] [NX|XX]

> EX 设置过期时间为多少秒
>
> PX 设置过期时间为多少毫秒
>
> NX 键不存在才设置
>
> XX 键存在才设置
>
> 成功返回OK，失败返回nil

---

### **大量key同时过期**

集中过期，由于清除大量的key很耗时，会出现短暂的卡顿现象

在设置key的过期时间时，给每个key加上随机值

---

### **使用redis做异步队列**

使用List作为队列，RPUSH生产消息，LPOP消费消息

- 缺点：没有等待队列里有值就消费
  - 可以在应用层引入sleep机制去调用LPOP重试

BLPOP key [key...] timeout 阻塞直到队列有消息或者超时

- 缺点：只能供一个消费者消费

pub/sub：主题订阅者模式

> 发送者（pub）可以发送消息，订阅者（sub）接收消息
>
> 订阅者可以订阅任意数量的频道
>
> subscribe topic
>
> publish topic xxx
>
> 无状态，无法确保收到

---

### **为什么要使用pipeline**

Pipeline批量执行指令,节省多次IO往返的时间

有顺序的指令建议分批发送