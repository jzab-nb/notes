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
- sunion: 求多个集合的并集

#### SortedSet

>help @sorted_set
>
>可排序的set集合（根据分值排序），类似TreeSet，底层数据结构为跳表加哈希表，可排序，查询快，不可重复，可以用来实现排行榜

- zadd:添加一个或多个元素到sorted set,如果存在则更新score值

- zrem: 删除指定元素

- zscore: 获取指定元素的分数

- zrank: 获取指定元素的排名

- zcard: 获取元素个数

- zcount: 统计分值在区间范围内的元素个数

- zincrby: 按照指定的步长更新某个元素的分值

- zrange: 返回排名在指定范围内的元素

- zrangebyscore: 返回分数在指定范围内的元素

- zinter：求多个集合的交集
- zdiff：求多个集合的差集，在a中不在b中的
- zunion: 求多个集合的并集

涉及排序相关的命令,可以在z后面加rev来进行降序操作

## Redis客户端

### 对比

Jedis: 方法名和redis命令名称相同，学习成本低，但线程不安全

lettuce：基于Nutty，支持响应式和异步编程，支持Redis哨兵、集群、管道

Redisson：基于Redis实现的分布式、可伸缩的Java数据结构集合。包含Map、Queue、Lock、Semaphore、AtomicLong等强大的功能

Spring Data Redis：集成Jedis和lettuce 

### Jedis

引入依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.8.0</version>
</dependency>
```

基本用法

```java
// 建立连接
Jedis jedis = new Jedis("vip.jzab.xyz", 6379);
// 密码认证
jedis.auth("102099");
// 选择数据库
jedis.select(0);
// 命令
Set<String> s = jedis.smembers("lisi");
// 释放资源
jedis.close();
```

 线程池

```java
public class JedisFactory {
    private static final JedisPool jedisPool;

    static{
        JedisPoolConfig config = new JedisPoolConfig();
        // 最大连接
        config.setMaxTotal(8);
        // 最大空闲连接
        config.setMaxIdle(8);
        // 最小空闲连接
        config.setMinIdle(0);
        // 最长等待时间
        config.setMaxWaitMillis(200);
        jedisPool = new JedisPool(config,"vip.jzab.xyz",6379,1000,"102099");
    }

    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}
```

### SpringDataRedis

https://spring.io/projects/spring-data-redis

- 封装了不同的Redis客户端
- 提供统一API来操作
- 支持发布订阅模式
- 支持哨兵和集群
- 支持基于Lettuce的响应式编程
- 支持基于JDK、JSON、字符串、Spring对象的数据序列化和反序列化
- 支持基于Redis的JDKCollection实现

#### 1.引入依赖

```xml
<!--        spring data redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!--        连接池-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

####  2.配置连接

```yaml
spring:
  redis:
    host: vip.jzab.xyz
    port: 6379
    password: 102099
    lettuce: # 默认为lettuce,想使用Jedis需要单独引入
      pool:
        max-active: 8 # 最大连接
        max-idle: 8 # 最大空闲连接
        min-idle: 0 # 最小空闲连接
        max-wait: 100 # 连接等待时间
```

#### 3.注入并使用RedisTemplate

```java
@Resource
RedisTemplate<String,String> redisTemplate;

@Test
void redisTemplateTest(){
    redisTemplate.opsForValue().set("new:key","李四");
    redisTemplate.opsForValue( ).get("new:key");
    // 两种带过期时间的设置方式
    template.opsForValue().set(k,v,Duration.ofMinutes(过期时间-分钟数));
    template.opsForValue().set("k","v",过期时间-分钟数, TimeUnit.MINUTES);
    // 哈希类型的设置方式
    // 哈希类型一次性存入一个map,其中BeanUtil是HUtool包下的
    template.opsForHash().putAll(key,BeanUtil.beanToMap(bean对象));
    template.expire(key,过期时间,TimeUnit.SECONDS);
}
```

#### 序列化

若要使用json序列化，需要先引入依赖

```xml
<!--        Jackson依赖-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

自定义redistemplate的bean，方便使用

```java
@Configuration
public class RedisConfiguration {
    @Bean
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory connectionFactory){
        RedisTemplate<String, Object> template = new RedisTemplate<>( );
        // 设置连接工厂
        template.setConnectionFactory(connectionFactory);
        // UTF_8的序列化器常量
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        // json序列化器
        RedisSerializer<Object> json = RedisSerializer.json( );
        template.setValueSerializer(json);
        template.setHashValueSerializer(json);
        return template;
    }
}

```

存储时会带上类信息

```json
[
    "java.util.ArrayList",
    [
        1,
        "2",
        [
            "[Ljava.lang.String;",
            [
                "3"
            ]
        ]
    ] 
]
```

#### StringRedisTemplate

为了节省内存,key和value都是字符串类型,需要手动处理序列化和反序列化

```java
@Resource
StringRedisTemplate stringRedisTemplate;

// json序列化工具
ObjectMapper mapper = new ObjectMapper( );
// 手动序列化
String s = mapper.writeValueAsString(list);
// 手动反序列化
list = mapper.readValue(r, ArrayList.class);
```

## Redis实战

### Redis实现短信登录

传统的基于session登录的流程

![image-20240708200512675](redis.assets/image-20240708200512675.png)

 **session共享问题**: 多台tomcat无法共享session空间

**基于redis登录的思路**:

1. 使用手机号作为key，存储string类型的验证码
2. 使用随机token作为key，存储用户信息，用户信息的存储可以使用hash结构，也可以使用json格式化的string结构
3. 登录验证时，手动刷新用户key的过期时间

### redis实现缓存

缓存存储临时数据，读写速度快

![image-20240731205146363](redis.assets/image-20240731205146363.png)

作用：

- 价格低后端负载
- 提高读写效率，降低响应时间

成本：

- 数据一致性成本
- 代码维护成本
- 运维成本（避免雪崩，保证高可用）

#### 实现思路

1. 客户端查询数据
2. 缓存命中，直接返回，未命中，查询数据库
3. 数据库命中，刷新到缓存中，数据库未命中，返回404

