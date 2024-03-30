## 面试题

缓存穿透、雪崩、击穿的原因和应对策略
Redis 如何保证数据一致性？
Redis 底层数据结构和 IO 模型
Redis 为什么快？只是因为在内存中操作吗？
Redis 的有序集合为什么用跳表？
Redis 大 Key 的危害以及如何避免和解决
Redis 内存淘汰策略
Redis 持久化机制

## Redis入门

redis是一个key-value结构的NoSql(非关系型数据库)数据库

- k-v型，value支持多种数据类型
- 单线程(6.0引入了多线程的网络请求，但核心命令处理仍是单线程)
- 低延迟、速度快（基于内存，IO多路复用）
- 支持数据持久化
- 支持主从、分片集群
- 支持多语言客户端

### SQL和NoSQl区别

SQL:

- 结构化
- 有关联
- SQL语法
- ACID事务
- 磁盘存储
- 垂直扩展(提升性能大部分时候只能提升本机性能,分库麻烦)

NoSQL:

- 非结构化
- 无关联
- 语法不同
- 不一定保证ACID
- 内存存储
- 水平扩展(多节点,分布式,天然支持分库)

### Redis配置文件

```conf
# 绑定地址 默认127.0.0.1 ::1
bind 0.0.0.0
# 保护模式,禁止远程连接,默认为yes
protected-mode no
# 是否启用后台模式
daemonize yes

# 进程号地址
pidfile /var/run/redis/redis-server.pid
# 日志级别
loglevel notice
# 日志文件地址
logfile /var/log/redis/redis-server.log
# 数据库数量,按数字编号0-15
databases 16

# 持久化文件存储路径
dbfilename dump.rdb
# 工作目录
dir /var/lib/redis
```

### Redis命令行客户端

```shell
redis-cli [options] [command]
```

常用的options有：

-h: 指定主机

-p: 指定端口

-a: 指定密码

command为连接上之后执行的指令

### redis图形化客户端

https://github.com/qishibo/AnotherRedisDesktopManager/

### 值的数据类型

常见的八种:

| 类型      | 含义           | 示例                    |
| --------- | -------------- | ----------------------- |
| String    | 字符串         | "hello world"           |
| Hash      | 哈希表         | {name: "JZAB", age: 21} |
| List      | 列表           | [A -> B -> C -> C]      |
| Set       | 集合(值不重复) | {A,B,C}                 |
| SortedSet | 排序集合       | {A: 1, B: 2, C: 3}      |
| GEO       | 地理坐标       | {A: (120.3, 30.5)}      |
| BitMap    | 二进制数据     | 011011011011011         |
| HyperLog  | 二进制数据     | 011011011011011         |

### key的层级化

在设置key时，允许将key用冒号分层

> xyz:jzab:www
>
> xyz:jzab:vip

### 命令

#### 通用命令

> help @generic

- keys: 查看所有符合模板的key，模板中可以使用*?通配符进行模糊查询，格式：``` keys 模板```
- del: 删除一个或多个指定的key，返回值代表删除成功的数量，格式：```del key0 [key1 key2]```
- exists：查看一个或多个key是否存在，返回值代表存在的个数，格式：```exists key0 [key1 key2]```
- expire：设置一个key的有效期，以秒为单位，格式：```expire key second```
- ttl：查看一个key的有效期，返回-1代表永久，返回-2代笔key不存在，返回其他数字代表key剩余多少秒过期，格式：```ttl key```

#### string类型命令

> help @string
>
> string类型底层都是字节数组，可以存放string，int，float，最大空间不超过512MB

- set：添加或修改键值对
  - ```SET key value [EX seconds|PX milliseconds|KEEPTTL] [NX|XX]```
  - NX为不存在则添加，XX为存在则修改
- get：根据key获取string类型的value
- mset：批量添加修改
- mget：批量获取
- incr：让一个int类型的值自增
- incrby：让一个int类型的值自增指定的步长
- incrbyfloat：让一个float类型的数自增指定的步长
- setnx：如果key不存在则添加，否则无操作
- setex：添加一个string类型的键值对，并指定有效期

#### Hash类型命令

> help @hash
>
> value部分是一个hash表，hash表的键叫做field，值叫做value

- hset：添加或修改hash类型的key的field的值（可批量）
- hget：获取一个hash类型key的field的值
- hmset：同hset
- hmget：批量获取一个hash类型的多个field的值
- hgetall：获取一个hash类型的所有的field和value，奇数行field，偶数行value
- hkeys：获取一个hash类型的field
- hvals：获取一个hash类型的所有value
- hincrby：让一个hash类型某个field的value自增并指定步长
- hsetnx：若某个field不存在则添加，否则不执行

#### List类型

> help @list
>
> 底层类似LinkedList，双向链表，有序，可重复，插入删除快，查询慢

- lpush：左侧插入元素，可以插入多个
- lpop：左侧返回元素，没有返回nil
- rpush：右侧插入元素，可以插入多个
- rpop：右侧返回元素，没有返回nil
- lrange：返回某个下标范围内的元素，（下标从0开始，闭区间）
- blpop：同lpop，没有元素时等待
- brpop：同rpop，没有元素时等待

#### Set类型

> help @set
>
> 类似java中的HashSet，可以看作值为null的hash表，无序、元素不可重复、查找快、支持交集、并集、差集等操作

- sadd: 向set中添加一个或多个元素
- srem：移除set中的指定元素，可以移除多个
- scard：返回set中元素的个数
- sIsMember: 判断一个元素是否在set中，是返回1，不是返回0
- smembers：获取set中所有的元素

多个集合间的操作：

- sinter：求多个集合的交集
- sdiff：求多个集合的差集，在a中不在b中的