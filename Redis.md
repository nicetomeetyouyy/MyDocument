### 1.Redis概述
   Redis是用C语言开发的一个开源的高性能键值对key-value数据库,支持String: 字符串，Hash: 散列
，List: 列表，Set: 集合，Sorted Set: 有序集合



### 2.String 和一些常用命令
  字符串类型是Redis中最为基础的数据存储类型，存入的数据是二进制安全的（存入和获取数据相同),最大可容纳数据长度是512M


  * set key value:设定key的vuale（key存在会覆盖)
  * get key:获取key的value
  * getset key value: 先获取key的值再设置key的值
  * del key：删除key
  * incr key:将key递增1，如果key不存在，其初始值为0，如果key的值不能转为整型，则执行失败报错.
  * decr key: 将key减1，如果key不存在，其初始值为0，如果key的值不能转为整型，则执行失败报错
  * getrange key start end:返回key字符串中的start到end范围的子字符
  * mget key1 key2 ...:获取多个key的值
  * setex(set expire exist) key num value:设置key和value的存在时间为num秒
  * setnx（set no exist） key value ：当key不存在时，设置key值为valu。
* mset key1 value1 [key2 value2] ...:设置多个key-value

### 3.hash和一些常用命令
  hash可以看成一个<String key,String value>的容器。此类型key-hash可以看成key-Map<String,String>（后一个String可以看成上面的String类型）
  * hset KEY key1 value1 :设置KEY的key1-value1对
  * hmset KEY key1 value1 [key2 value2 key3 value3] ....:设置KEY的多个键值对
  * hget KEY key1 :
获取KEY 的key1的值
  * hmget KEY key1[key2  key3 ]:获取KEY的多个key的值
  * hgetall KEY:获取KEY的所有key-value
  * hdel KEY key1[key2 key3...]:删除KEY的一个或多个key
  * del KEY 删除整个KEY
  * hincrby KEY key1 num:将KEY的key1的value增加num
  * hexits KEY key1:判断KEY的key是否存在
  * hkeys KEY：获取所有的KEY
  * hvals KEY：获取所有的value

### 4.List和一些常用命令
 在Redis中，LIST类型是按照插入顺序排序的链表,可以在left和right添加元素
* lpush/rpush key value1[value2 value3 ...]:往左/往右插入key的value
* lrange key strart end:查看key从左到右start 到end的值从0开始，-1表示尾部的元素，-2 表示倒数第二个，以此类推
* lpop/rpop key：弹出左端/右端第一个元素并get它的值
* llen key：获得key的长度
* lpushx/rpushx key value：往key的左/右插入value
* lset key index value :设置key中index位置的值，0表示最左边，-1最右边
* linsert key before/after value1 balue2:在key中value1前/后插入value2

### 5.ZSET类型和一些常用命令
 类似于Set,不同的是Sorted中的每个成员都分配了一个分数（Score）用于对其中的成员进行排序（升序）。
zset的成员是唯一的,但分数(score)却可以重复
* zadd key score member ：设置， 存在就更新
* zscore key member：查看score值
* zrange key start stop[withscores]：按索引返回key的成员，withscores表示显示score
* zrangebyscore key min max：返回集合中 score 在给定区间的元素
* zrem key member [member...]：移除有序集合中的一个或多个元素，若member不存在则忽略；
* zremrangebyrank min max：删除集合中索引在给定区间的元素
* zremrangebyscore  min max：删除集合中 score 在给定区间的元素

### 6 redis持久化RDB和AOP
Redis持久化分为RDB持久化和AOF持久化：前者将当前数据保存到硬盘，后者则是将每次执行的写命令保存到硬盘（SET）
#### 6.1 RDB持久化
RDB持久化是将当前进程中的数据生成快照保存到硬盘(因此也称作快照持久化)，保存的默认文件dump.rdb；当Redis重新启动时，读取此快照文件恢复数据。
* 自动触发条件
  系统默认配置文件提供了三种触发条件（save m n）：
   * save 900 1
   * save 300 10
   * save 60 10000
  当n个文件在m秒发生改变时保存文件
* 手动触发
   * save:save命令会阻塞Redis服务器进程，直到RDB文件创建完毕为止.在Redis服务器阻塞期间，服务器不能处理任何命令请求
   * basave:创建一个子进程(fork期间阻塞)，由子进程来负责创建RDB文件，父进程(即Redis主进程)则继续处理请求,在自动触发RDB持久化时,用的此方式
   * shutdown:关闭时也会保存到rdb文件
* 配置文件
   * 可以在配置文件中指定rdb文件   dbfilename 文件名.rdb
   * stop-writes-on-bgsave-error：默认yes，保存错误时停止写入
   * rdbcompression：默认yes，压缩文件选项
   * rdbchecksum ：默认yes，用CRC64算法校验文件
   * dir：默认./，指定文件存放目录
   

#### 6.2 AOF持久化
 AOF持久化(Append Only File持久化)，则是将Redis执行的每次写命令记录到单独的日志文件中,保存的默认文件是appendonly.aof，当Redis重启时再次执行AOF文件中的命令来恢复数据。
 * 该模式默认关闭，在配置文件中设为appendonly yes开启
 * 三种模式：
     * no：命令写入aof_buf后调用系统write操作，不对AOF文件做fsync同步；同步由操作系统负责，通常同步周期为30秒
     * everysec：命令写入aof_buf后调用系统write操作，write完成后线程返回；fsync同步文件操作由专门的线程每秒调用一次。everysec是前述两种策略的折中，是性能和数据安全性的平衡，因此是Redis的默认配置，也是我们推荐的配置
     * always:写入立即保存到aof文件中
 * 文件重写
     Redis服务器执行的写命令越来越多，AOF文件也会越来越大；过大的AOF文件不仅会影响服务器的正常运行，也会导致数据恢复需要的时间过长
    * auto-aof-rewrite-percentage 100：自动触发文件重写条件之一
    * auto-aof-rewrite-min-size 64mb：自动触发文件重写条件之一
    * no-appendfsync-on-rewrite no：AOF重写期间是否禁止fsync；如果开启该选项，可以减轻文件重写时CPU和硬盘的负载（尤其是硬盘），但是可能会丢失AOF重写期间的数据；需要在负载和安全性之间进行平衡
 * 可以同时开启RDB和AOF，同时开启启动时选择AOF进行加载
#### 6.3 RDB和AOF总结
    * RDB：RDB文件紧凑，体积小，网络传输快，适合全量复制，是对性能的影响相对较小，但是数据最后一次保存的数据容易丢失
    * AOF：AOF的出现主要为了解决RDB的文件丢失问题，AOF文件出现丢失只是秒级的丢失，缺点是文件大、恢复速度慢、对性能影响较大

### 7.事务
  redis支持简单的事务操作：
 * multi：开启事务
 * discard：取消
 * exec：提交事务<br>
 **redis不支持事务回滚**，如果在一个事务中**的命令出现错误，那么所有的命令都不会执行**，如果在一个事务中出现**运行错误，那么正确的命令会被执行**
 * WATCH：命令可以监控一个或多个键，一旦其中有一个键被修改（或删除），之后的事务就不会执行，监控一直持续到EXEC命令
 * UNWATCH:取消监控<br>

### 8.主从复制
  主节点负责写数据，从节点负责读数据，主节点定期把数据同步到从节点保证数据的一致性
* 作用：读写分离和容灾恢复

#### 8.1 配置
* 配从库不配主库
* 从库配置：slaveof [主库IP] [主库端口]；<br>
**注意：** 每次slave与master断开后，都需要重新连接，除非你配置进redis.conf文件;
* info replication 可以查看redis主从信息
* 当Master挂掉后，Slave可键入命令 slaveof no one使当前redis停止与其他Master redis数据同步，转成Master redis
* 上一个Slave可以是下一个Slave的Master，Slave同样可以接收其他slaves的连接和同步请求

#### 8.2复制原理
* Slave启动成功连接到master后会发送一个sync命令；

* Master接到命令启动后的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，以完成一次完全同步；

* 全量复制：slave服务在受到数据库文件数据后，将其存盘并加载到内存中；

* 增量复制：Master继续将新的所有收集到的修改命令依次传给slave，完成同步；

* 只要是重新连接master，一次完全同步（全量复制）将被自动执行

#### 8.3 哨兵模式
哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例
* 作用：
    * 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器
    * 当哨兵监测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机
* 配置：
   * 在Master对应redis.conf同目录下新建sentinel.conf文件
   * 在sentinel.conf文件写入
   
```
sentinel monitor 被监控数据库名字（自己起名字） 192.168.24.131 6379 1
/*
说明：
macrog-master:监控主数据的名称,自定义即可,可以使用大小写字母和“.-_”符号
192.168.24.131:监控的主数据库的IP
6379:监控的主数据库的端口
1:最低通过票数*/
```

  

  

 