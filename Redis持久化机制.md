### Redis持久化机制

#### 1、什么是持久化

`利用永久性存储介质将数据进行保存，在特定的时间将保存的数据进行恢复`

持久化的作用：防止数据的意外丢失，确保数据安全性



持久化的方式：

1. 将**当前数据状态进行保存**，快照形式，存储数据结果，存储格式简单。
2. 将**数据的操作过程进行保存**，日志形式，存储操作过程，存储格式复杂。

![image-20210525160750233](https://gitee.com/Akihij/PicGo/raw/master/img/20210525160757.png)



#### 2、RDB

RDB就是以**快照的形式**对Redis中的数据进行持久化。

RDB的启动方式：

~~~markdown
命令：
	save
作用：手动执行一次保存操作
~~~

其中保存的数据在dump.rdb文件中。



我们可以通过Redis的配置文件去修改dump.rdb文件的信息。

- `dbfilename dump.rdb`

  说明：设置本地数据库文件名，默认是dump.rdb

- `rdbcompression  yes`

  说明：设置存储至本地数据库时是否压缩数据，默认为yes，采用LZE压缩

- `rdbchecksum  yes`

  说明：设置是否进行RDB文件格式校验，该校验过程在写文件和读文件过程均进行



RDB文件如何恢复：

当我们打开Redis服务器之后，Redis中的数据就会恢复到内存中。



save指令出现的问题：

对于单线程来说，当执行save指令时，时间过长，那么会阻塞当前的Redis服务器，直到RDB过程结束。



##### 如何解决阻塞问题：----bgsave

~~~markdown
指令：
	bgsave
作用：手动启动后台保存操作，但不是立即执行，是在后台执行
~~~



bgsave的工作原理：

![image-20210525163405855](https://gitee.com/Akihij/PicGo/raw/master/img/20210525163405.png)



`bgsave命令是针对save阻塞问题做出的优化。Redis内部所有涉及到RDB操作都采用bgsave的方式，save命令可以放弃使用`

~~~markdown
关于bgsave的配置信息：
stop-writes-on-bgsave  yes
说明：后台存储过程中如果出现错误现象，是否停止保存操作
~~~

|      方式      | save | bgsave |
| :------------: | :--: | :----: |
|      读写      | 同步 |  异步  |
| 阻塞客户端指令 |  是  |   否   |
|  额外内存消耗  |  否  |   是   |
|   启动新进程   |  否  |   是   |



##### RDB的自动保存

~~~markdown
配置：在redis配置文件中配置
save second changes
作用：满足限定时间范围内key的变化数量达到指定数量即进行持久化
 - second :监控时间范围
 - changes: 监控key的变化量
 
 save 900 1  # 如果在900s内变化了1个key。则执行save
 save 300 10  # 如果在60s内变化了300个key。则执行save
~~~





##### RDB的优点和缺点

优点

- RDB是一个紧凑压缩的二进制文件，存储效率较高

- RDB内部存储的是redis在某个时间点的数据快照，非常适合用于数据备份，全量复制等场景 

- RDB恢复数据的速度要比AOF快很多

- 应用：服务器中每X小时执行bgsave备份，并将RDB文件拷贝到远程机器中，用于灾难恢复。

缺点

-  RDB方式无论是执行指令还是利用配置，无法做到实时持久化，具有较大的可能性丢失数据

- bgsave指令每次运行要执行fork操作创建子进程，要牺牲掉一些性能

- Redis的众多版本中未进行RDB文件格式的版本统一，有可能出现各版本服务之间数据格式无法兼容现象



#### 3、AOF

##### RDB存储的弊端：

- 当数据量巨大时，效率非常低
- 基于fork创建子进程，内存产生额外消耗
- 宕机带来的数据丢失风险



这些问题AOF都可以解决：

**AOF(append only file)持久化**：以独立日志的方式记录每次写命令，重启时再重新执行AOF文件中命令

达到恢复数据的目的。与RDB相比可以简单描述为改记录数据为记录数据产生的过程

作用：解决了数据持久化的实时性，目前已经是Redis持久化的主流方式



##### AOF写数据的过程

Redis将指令存储在缓存区中，然后将命令同步到aof文件中。

![image-20210525165228432](https://gitee.com/Akihij/PicGo/raw/master/img/20210525165228.png)





##### AOF写数据的三种策略：

- always(每次）

  每次写入操作均同步到AOF文件中，数据零误差，性能较低

- everysec（每秒）

  每秒将缓冲区中的指令同步到AOF文件中，数据准确性较高，性能较高在系统突然宕机的情况下丢失1秒内的数据

- no（系统控制）

  由操作系统控制每次同步到AOF文件的周期，整体过程不可控



AOF方式配置：

~~~markdown
配置：在Redis配置文件中

1. appendonly  yes
是否开启AOF持久化功能。默认不开启

2. appendfsync always|everysec|no
AOF写数据策略

3. appendfilename filename
AOF持久化文件名，默认文件名未appendonly.aof
~~~



##### AOF的重写

如果我们执行以下指令

~~~markdown
set name zs
set name ls
set name ww
~~~

对于这样的操作，我们如何把每一个操作都保存下来，是没有意义的。所以需要AOF重写。

随着命令不断写入AOF，文件会越来越大。

为了解决这个问题，Redis引入了AOF重写机制压缩文件体积。AOF文件重写是将Redis进程内的数据转化为写命令同步到新AOF文件的过程。

简单说就是将对同一个数据的若干个条命令执行结果转化成最终结果数据对应的指令进行记录。



**AOF重写作用**

- 降低磁盘占用量，提高磁盘利用率

- 提高持久化效率，降低持久化写时间，提高IO性能

- 降低数据恢复用时，提高数据恢复效率



**AOF重写规则**

- 进程内已超时的数据不再写入文件

- 忽略无效指令，重写时使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令

  如del key1、 hdel key2、srem key3、set key4 111、set key4 222等 

- 对同一数据的多条写命令合并为一条命令

  如lpush list1 a、lpush list1 b、 lpush list1 c 可以转化为：lpush list1 a b c。

  为防止数据量过大造成客户端缓冲区溢出，对list、set、hash、zset等类型，每条指令最多写入64个元素



AOF重写方式

- 手动重写指令

  `bgrewriteaof` 

- 自动重写配置

  `auto-aof-rewrite-min-size ` size

  `auto-aof-rewrite-percentage` percentage



原理

![image-20210525171339967](https://gitee.com/Akihij/PicGo/raw/master/img/20210525171340.png)



#### 4、RDB和AOF对比

|  持久化方式  |        RDB         |        AOF         |
| :----------: | :----------------: | :----------------: |
| 占用存储空间 | 小（数据级：压缩） | 大（指令级：重写） |
|   存储速度   |         慢         |         块         |
|   恢复速度   |         块         |         慢         |
|  数据安全性  |     会丢失数据     |    依据策略决定    |
|   资源消耗   |     高/重量级      |     低/轻量级      |
|  启动优先级  |         低         |         高         |



