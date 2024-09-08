## 面试题

### 数据类型

- string：bitMap
- hash
- set
- sortedSet：geo
- stream

### 缓存穿透，雪崩，击穿的原因和应对策略

1. 缓存穿透

   原因：用户大量访问在缓存和数据库都不存在的key，导致一直向数据库发送请求

   策略：

   - 对不存在的key，在redis中存储空值
   - 增加key的复杂性，防止恶意猜测，在查询前对key的格式做校验，拦截不符合格式的key
   - 布隆过滤器，优点：响应速度快，缺点：不是百分之百准确，实现相对复杂
   - 限制用户的权限，对用户进行限流

2. 缓存雪崩

   原因：大量key同时失效，或者redis直接宕机导致请求直接打到数据库

   策略：

   - 对TTL增加随机值，防止同时过期
   - redis采用集群模式，增加可用性
   - 采用多级缓存
   - 对业务进行降级，限流

3. 缓存击穿

   原因：热点key失效导致大量请求到达数据库，热点key一般是会存在高并发访问，或者重建失效的特点

   策略：

   - 逻辑过期，优点：没有等待时间，缺点：一致性较低，实现困难
   - 互斥锁，优点：一致性高，实现简单，缺点：需要阻塞等待

### Redis 如何保证数据一致性

共有大约三种方式保证数据一致性

1. 数据变动时，主动更新缓存
2. 利用TTL过期机制，过期后再刷新
3. 利用内存淘汰策略

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

#### 缓存更新策略

- 内存淘汰机制：利用redis自带的机制，一致性差，无维护成本
- 超时剔除：给数据设置过期时间，到期自动删除，一致性一般，维护成本低
- 主动更新：数据库写入修改时，手动修改缓存，一致性好，维护成本高

​	低一致性需求的场景，一般使用超时剔除+内存淘汰

​	高一致性需求的场景，一般使用主动更新+超时剔除

#### 缓存主动更新方式

- 手动编码：手动在更新数据库时手动修改缓存
- write through：缓存和数据库整合为一个服务，服务内部处理一致性，调用者只负责调用
- 写回：调用者只操作缓存，由其他线程异步同步数据

#### 手动编码方式更新缓存

1. 删除缓存还是更新缓存

   更新: 每次更新数据库时都更新缓存，如果期间没有读取，会存在过多的无效写操作（不建议使用）

   删除: 更新数据库时删除缓存，查询时再更新缓存 （建议使用）

2. 如何保证操作的原子性(同时成功、同时失败)

   单体系统：缓存和数据库操作在同一个事务内执行

   分布式系统：使用TCC（分布式事务解决方案）

3. 先操作缓存还是数据库

   先操作数据库：发生线程安全问题概率低

   先操作缓存：发生线程安全问题概率高

   ![image-20240813230633627](redis.assets/image-20240813230633627.png)

#### 缓存穿透

客户端请求的数据在缓存和数据库中都不存在，请求会一直提交到数据库

解决方案:

- 缓存空对象：优点：实现简单，缺点：可能存在短期不一致，会产生额外的内存消耗
- 布隆过滤器：查询缓存前先查询布隆过滤器看是否存在，优点：内存占用少，缺点：实现复杂，存在误判的可能
- 增加id复杂度，添加id格式校验
- 增加用户权限管理，用户限流，热点参数限流

 #### 缓存雪崩

大量缓存同时失效，或者redis直接宕机，导致大量请求到达数据库

解决方案：

- TTL添加随机值
- 利用redis集群提高可用性
- 给缓存业务添加降级限流策略
- 给业务添加多级缓存

#### 缓存击穿

热点key过期导致的问题，高并发访问，缓存重建复杂的key失效给数据库造成压力

解决方案:

- 互斥锁，在查询数据库重建缓存的时候设置锁
  - 优点：没有额外内存消耗，保证强一致性，实现简单
  - 缺点：需要等待，性能受影响，可能有死锁风险
- 逻辑过期，数据内部设置逻辑过期时间字段，读取到过期数据后获取锁同时开启新线程来重建缓存
  - 优点：无需等待，性能较好
  - 缺点：不保证一致性，有额外的内存消耗，实现复杂

#### 互斥锁解决缓存击穿

使用string类型的setnx命令实现互斥锁

setnx: 当key不存在时设置值否则失败 

```java
// redis互斥锁简单实现
public boolean lock(String key){
    // 设置互斥锁
    return BooleanUtil.isTrue(
        template.opsForValue().setIfAbsent(key,"1",10,TimeUnit.SECONDS)
    );
}

public void unLock(String key){
    // 释放锁
    template.delete(key);
}
```

查询未命中时，先获取锁，锁成功了再次判断是否命中，若还是未命中则查询数据库更新缓存

若未获取到锁，则循环等待，直到获取到锁或者名字缓存为止

#### 逻辑过期解决缓存击穿

假设是热点数据的场景：提前将数据放入缓存，不考虑数据未命中的情况，只考虑数据逻辑过期

使用线程池实现开启新线程调用

```java
private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);
```

#### 工具类

redis工具类:

```java
@Component
public class RedisUtil {
    @Resource
    private StringRedisTemplate template;

    // 设置json对象
    public void setObject(String key,Object value){
        template.opsForValue().set(key, JSONUtil.toJsonStr(value));
    }

    // 获取json对象
    public <Y> Y getObject(String key,Class<Y> claz){
        String s = template.opsForValue( ).get(key);
        if(StringUtils.isBlank(s)) return null;
        return (Y)JSONUtil.toBean(s,claz);
    }

    public String getRawValue(String key){
        return template.opsForValue().get(key);
    }

    // 添加到排序集合中
    public void addZset(String key, Object value, double vl){
        //
        template.opsForZSet().add(key,JSONUtil.toJsonStr(value),vl);
    }

    // 获取排序集合
    public <T> List<T> getZset(String key, Class<T> claz){
        Long length = template.opsForZSet().zCard(key);
        Set<String> range = template.opsForZSet( ).range(key, 0, length);
        List<T> res = new ArrayList<>(  );
        if(range==null) return res;
        for (String s : range) {
            res.add(JSONUtil.toBean(s,claz));
        }
        return res;
    }

    public void expire(String key,long minutes){
        template.expire(key,minutes, TimeUnit.MINUTES);
    }

    public void expire(String key,long time,TimeUnit unit){
        template.expire(key,time, unit);
    }


    public void remove(String key){
        template.delete(key);
    }

    public boolean lock(String key){
        // 设置互斥锁
        return BooleanUtil.isTrue(
            template.opsForValue().setIfAbsent(key,"1",10,TimeUnit.SECONDS)
        );
    }

    public void unLock(String key){
        // 释放锁
        template.delete(key);
    }
}
```



缓存工具类:

```java
@Component
public class CacheUtils {
    @Resource
    private RedisUtil redisUtil;

    //1.将任意java对象序列化成json并设置ttl
    public void saveWithTTL(String key, Object value, Long time, TimeUnit timeUnit){
        redisUtil.setObject(key,value);
        redisUtil.expire(key,time,timeUnit);
    }
    //2.将任意java对象序列化成json并设置ttl并设置逻辑过期
    public void saveWithLogic(String key, Object value, Long time,TimeUnit unit){
        RedisData redisData = new RedisData( );
        redisData.setData(value);
        redisData.setExpireTime(LocalDateTime.now().plusSeconds(unit.toSeconds(time)));
        redisUtil.setObject(key,redisData);
    }
    //3.根据key返回并反序列化,利用空值解决缓存穿透
    public <T,ID> T getWithThrough(
        String keyPrefix, ID id, Class<T> clazz, Function<ID,T> func,
        Long time,TimeUnit unit
    ){
        String key = keyPrefix+id;
        // 1.查缓存
        T object = redisUtil.getObject(key, clazz);
        // 2.不为空返回
        if(object!=null) return object;
        // 3.为空看是null还是空字符串,是空字符串则返回
        String rawValue = redisUtil.getRawValue(key);
        boolean a = rawValue !=null;
        if(a) return null;
        // 4.是null则更新数据库
        T result = func.apply(id);
        if(result==null){
            this.saveWithTTL(key,"",time,unit);
        }else{
            this.saveWithTTL(key,result,time,unit);
        }
        return result;
    }

    private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);

    //4.根据key返回并反序列化,利用逻辑过期解决缓存击穿

    public <T,ID> T getWithLogicExpire(
            String keyExpire, ID id, Class<T> clazz, Function<ID,T> func,Long time,TimeUnit unit){
        String key = keyExpire+id;
        // 1.查询缓存
        RedisData object = redisUtil.getObject(key, RedisData.class);
        // 2.未命中直接返回null
        if(object==null){
            return null;
        }else{
            // 3.命中后判断逻辑是否过期,逻辑未过期直接返回
            JSONObject data = (JSONObject)object.getData( );
            T shop = JSONUtil.toBean(data, clazz);
            // 4.逻辑过期则获取锁开启新线程同步数据,自己则返回过期数据
            if(object.getExpireTime().isBefore(LocalDateTime.now())){
                String lockKey = RedisConstants.LOCK_SHOP_KEY+id;
                boolean lock = redisUtil.lock(lockKey);
                // 获取锁成功,则开线程
                if(lock){
                    // 二次判断,防止线程安全问题
                    object = redisUtil.getObject(key, RedisData.class);
                    // 真的过期了再更新
                    if(object.getExpireTime().isBefore(LocalDateTime.now())){
                        // 开启一个线程单独处理
                        CACHE_REBUILD_EXECUTOR.submit(()->{
                            try{
                                T obj = func.apply(id);
                                if(obj!=null){
                                    this.saveWithLogic(key,obj,time,unit);
                                }
                            }catch (Exception e){
                                throw new RuntimeException( e );
                            }finally {
                                redisUtil.unLock(lockKey);
                            }
                        });
                    }
                }
            }
            // 过期未过期的,都直接返回
            return shop;
        }
    }
}
```

### Redis秒杀

#### 全局唯一ID

不建议使用数据库自增ID，缺点：

- 规律性过于明显
- 受单表数据量限制

全局唯一ID要符合：

- 唯一性
- 高可用
- 高性能
- 递增性
- 安全性

ID组成(类似雪花算法):

- 符号位：1bit，永远为0
- 时间戳：31bit，秒为单位，可以使用69年
- 序列号：32bit，秒内计数器，每秒产生2^32个不同的ID

```java
@Component
public class RedisIDWorker {

    /*
    开始时间戳
     */
    LocalDateTime startTime = LocalDateTime.of(2024,8,20,0,0);
    /*
    序列号占位
     */
    int move = 32;

    @Resource
    StringRedisTemplate stringRedisTemplate;

    public long generate(String keyPrefix){
        // 获取当前时间
        LocalDateTime now = LocalDateTime.now( );
        // 当前时间减去开始时间,得到时间戳
        long time = now.toEpochSecond(ZoneOffset.UTC)-startTime.toEpochSecond(ZoneOffset.UTC);
        // 时间左移32位
        time<<=move;
        // key的一部分
        String format = now.format(DateTimeFormatter.ofPattern(":yyyy:MM:dd"));
        // 拼接key
        String key = "icr:"+keyPrefix+format;
        // 获取自增序列号
        long increment = stringRedisTemplate.opsForValue( ).increment(key);
        // 拼接两部分
        return time | increment;
    }
}
```

#### 设计实现

- 优惠券表：包含优惠券的一些基本信息
- 秒杀券表：包含优惠券的ID，秒杀开始时间，秒杀结束时间，库存量

#### 库存超卖问题

- 悲观锁：认为线程安全问题一定发生，操作数据库时先获取锁，保证所有线程串行执行，java的Synchronize，Lock，数据库互斥锁都属于悲观锁
- 乐观锁：认为线程安全问题不一定发生，因此不加锁，只在更新数据库时判断有没有其他线程对数据做了修改
  - 若没有修改认为安全，自己才更新数据
  - 若修改了说明发生了安全问题，此时可以重试或异常

乐观锁的两种实现方式：

- 版本号法：为数据添加版本号标识，查询时查询版本号，更新前判断版本号是否和自己获取的一致
- CAS法（比较并交换）：业务数据本身带有类似版本号的性质如库存

乐观锁的问题：失败率太高

改进：更新时只判断库存是否大于0即可

#### 一人一单实现

通过加锁实现，synchronize关键字会锁住整个this对象，不建议使用，建议使用用户ID去加锁

在实现扣减库存和增加订单的时候，采用注解事务，但是要想让锁对事务生效，需要获取到spring的代理对象，代码如下

```java
@Override
public Result create(Long id) {
    SeckillVoucher voucher = seckillVoucherService.getById(id);
    if(voucher==null) return Result.fail("优惠券不存在");
    // 1.判断时间是否在开始时间和结束时间中
    if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {
        return Result.fail("秒杀未开始");
    }

    if (voucher.getEndTime().isBefore(LocalDateTime.now())) {
        return Result.fail("秒杀已经结束");
    }
    // 2.判断是否有库存
    if(voucher.getStock()<=0) return Result.fail("库存不足!!");

    Long userId = UserHolder.getUser( ).getId( );
    // 锁住用户ID,先上锁再开启事务
    synchronized (userId.toString().intern()){
        // 获取代理对象
        IVoucherOrderService service =  (IVoucherOrderService) AopContext.currentProxy();
        // 用代理对象去调用,否则事务不生效
        return service.createOrder(id);

    }
}

@Transactional
public Result createOrder(Long id){
    Long userId = UserHolder.getUser( ).getId( );
    // 额外判断,一人一单
    Integer count = this.query( )
        .eq("user_id", userId)
        .eq("voucher_id", id).count( );
    if(count>0) return Result.fail("请勿重复购买!!");
    // 3.扣减库存
    boolean success = seckillVoucherService.update( )
        .setSql("stock = stock-1")
        .eq("voucher_id", id)
        .gt("stock",0)
        // 这里是画蛇添足,其实不需要再次判断
        // .notExists("select 1 from tb_voucher_order where voucher_id='"+id+"' and user_id='"+ userId +"'")
        .update( );
    if(!success) return Result.fail("库存不足");

    // 4.创建订单
    VoucherOrder order = new VoucherOrder( );
    long orderId = idWorker.generate("voucher");
    order.setId(orderId);
    order.setVoucherId(id);
    order.setUserId(userId);
    this.save(order);
    return Result.ok(orderId);
}
```

同时需要引入aop相关依赖和在启动类上使用注解声明暴露代理对象

```java
@EnableAspectJAutoProxy(exposeProxy = true)
```

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
</dependency>
```

#### 集群环境下的线程安全问题

> 首先创建集群环境，在idea启动配置项中使用Ctrl+D可以快速复制启动配置，然后修改复制出来的配置项
>
> 设置jvm参数 -Dserver.port=8082 覆盖端口

> 然后修改ngin配置，实现负载均衡

```conf

worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/json;

    sendfile        on;
    
    keepalive_timeout  65;

    server {
        listen       8080;
        server_name  localhost;
        # 指定前端项目所在的位置
        location / {
            root   html/hmdp;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }


        location /api {  
            default_type  application/json;
            #internal;  
            keepalive_timeout   30s;  
            keepalive_requests  1000;  
            #支持keep-alive  
            proxy_http_version 1.1;  
            rewrite /api(/.*) $1 break;  
            proxy_pass_request_headers on;
            #more_clear_input_headers Accept-Encoding;  
            proxy_next_upstream error timeout;  
            #proxy_pass http://127.0.0.1:8081;
            proxy_pass http://backend;
        }
    }

    upstream backend {
        server 127.0.0.1:8081 max_fails=5 fail_timeout=10s weight=1;
        server 127.0.0.1:8082 max_fails=5 fail_timeout=10s weight=1;
    }
}
```

发现，在集群模式下，JVM的锁无法生效，因为每个JVM使用的是自己的锁，不同JVM的锁不共享

### 分布式锁

> 分布式锁：满足分布式系统或集群模式下，多线程可见并且互斥的锁
>
> 且要做到，高可用，高性能，高安全性

|        | Mysql                     | Redis                       | Zookeeper                        |
| ------ | ------------------------- | --------------------------- | -------------------------------- |
| 互斥   | 利用Mysql本身的互斥锁机制 | 使用setnx指令实现互斥       | 利用节点的唯一性和有序性实现互斥 |
| 高可用 | 好                        | 好                          | 好                               |
| 高性能 | 一般                      | 好                          | 一般                             |
| 安全性 | 断开连接，自动释放锁      | 利用key的过期机制，到期释放 | 临时节点，断开连接自动释放       |

#### 基于redis的分布式锁

实现分布式锁首先需要实现两个方法

- 获取锁: 
  - 二者同时操作保证原子性: set key 值 EX 时间 NX
  - 非阻塞模式：获取成功返回true，失败返回false

- 释放锁: 
  - del key
  - 超时释放

```java
public class SimpleRedisLock implements ILock{
    private final StringRedisTemplate template;
    private final String name;
    private static final String PRE_FIX="lock:";

    public SimpleRedisLock(StringRedisTemplate template, String name) {
        this.template = template;
        this.name = name;
    }

    @Override
    public boolean tryLock(long second) {
        long id = Thread.currentThread( ).getId( );

        Boolean result = template.opsForValue( ).setIfAbsent(PRE_FIX + name, id + "", second, TimeUnit.SECONDS);
        return BooleanUtil.isTrue(result);
    }

    @Override
    public void unLock() {
        template.delete(PRE_FIX+name);
    }
}
```

#### 锁误删除问题1

![image-20240824170049963](redis.assets/image-20240824170049963.png)

线程释放锁时，锁已经自动失效，而导致删除了其他线程创建的锁

解决思路：释放锁之前判断标识是否是自己的

```java
public class SimpleRedisLock implements ILock{
    private final StringRedisTemplate template;
    private final String name;
    private static final String PRE_FIX="lock:";
    private static final String ID_PREFIX = UUID.randomUUID( ).toString(true )+"_";

    public SimpleRedisLock(StringRedisTemplate template, String name) {
        this.template = template;
        this.name = name;
    }

    @Override
    public boolean tryLock(long second) {
        String id = ID_PREFIX+Thread.currentThread( ).getId( );

        Boolean result = template.opsForValue( ).setIfAbsent(PRE_FIX + name, id, second, TimeUnit.SECONDS);
        return BooleanUtil.isTrue(result);
    }

    @Override
    public void unLock() {
        String id = ID_PREFIX+Thread.currentThread( ).getId( );
        String val = template.opsForValue( ).get(PRE_FIX + name);
        if(id.equals(val)){
            template.delete(PRE_FIX+name);
        }
    }
}

```



#### 锁误删除问题2

![image-20240824215541749](redis.assets/image-20240824215541749.png)

在判断完成之后准备删除时，线程阻塞了，之后锁自动超时，其他线程创建了自己的锁导致出现问题

解决思路：保证判断和删除操作的原子性，使用Lua脚本保证操作的原子性 

##### LUA脚本

基础语法

```redis
eval "LUA脚本内容" key参数长度 KEY参数... 其他参数...
取KEY参数时: KEYS[位置]
取其他参数时: ARGV[位置]
例如:
eval "redis.call('set',KEYS[1],ARGV[1])" 1 test2 vvv
```

在java中使用lua脚本

```java
@Resource
StringRedisTemplate template;
 
// 脚本只需要加载一次
private static final DefaultRedisScript<Long> sc;

static {
    sc = new DefaultRedisScript<>(  );
    // 指定脚本的位置(resources目录下)
    sc.setLocation(new ClassPathResource("unlock.lua"));
    // 设置返回值类型
    sc.setResultType(Long.class);
}

@Test
public void luT(){
    System.out.println(sc.getScriptAsString() );
    // 设置键
    ArrayList<String> keys = new ArrayList<>( );
    keys.add("test");
    // 调用获取返回结果
    Object result = template.execute(sc, keys, "2");
    System.out.println(result );
}
```

删除键的脚本内容

```lua
if (ARGV[1] == redis.call('get',KEYS[1])) then
    return redis.call('del',KEYS[1])
end
return 0
```

#### 基于Redis的分布式锁问题

之前实现的锁存在一些问题

1. 不可重入，同一个线程无法同时获取同一个锁
2. 不可重试，获取锁就立即失败，没有重试机制
3. 超时释放，若业务执行时间过长，会导致锁释放，存在安全隐患
4. 主从一致性问题，主写从读，若主宕机，从没有同步到数据，会导致锁失效

#### Redisson

基于redis实现的分布式的工具

提供了分布式对象，分布式集合，分布式锁等功能。

> 1.导入依赖

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.13.6</version>
</dependency>
```

> 2.配置Redisson客户端

```java
@Configuration
public class RedissonConfig {
    @Bean
    public RedissonClient redissonClient(){
        Config config = new Config(  );
        config.useSingleServer()
                .setAddress("redis://ip:6379").setPassword("password");
        return Redisson.create(config);
    }
}
```

> 3.互斥锁使用

```java
@Resource
private RedissonClient redissonClientl;

public void testRedisson() throws InterruptedException {
    RLock lock = redissonClientl.getLock("key");
    // 获取锁,超时时间,过期时间,时间单位
    boolean isLock = lock.tryLock(1,10, TimeUnit.SECONDS);
    if(isLock){
        try {
            System.out.println("执行" );
        }finally {
            lock.unlock();
        }
    }
}
```

#### Redisson可重入锁原理

使用hash结构存储锁，field为线程标识，value为重入次数

获取锁时重入次数+1，重置有效期

删除锁时重入次数-1，次数为0了才删除

获取锁和释放锁都使用lua脚本保证原子性

#### Redisson锁重试原理

获取锁失败后订阅(subscribe)通知

释放锁时发布(publish)通知

#### Redisson看门狗原理

创建锁时先将存有线程id的entry放入一个ConcurrentHashMap中

然后设置定时任务，过了锁过期时间的1/3后，执行续期操作

任务内部递归调用自己

在释放锁时删除entry，取消任务

![image-20240825175621086](redis.assets/image-20240825175621086.png)

#### Redisson解决主从一致性问题

Redisson内部不区分主从，将所有的节点一视同仁，同时存入锁

使用MultiLock实现

### 秒杀优化

首先把业务分成两部分，分别是秒杀资格的判断，和秒杀订单的生成

秒杀资格的判断在redis中进行,耗时短

秒杀订单的生成在数据库中进行，用独立的线程执行

![image-20240826215058551](redis.assets/image-20240826215058551.png)

完整代码:

```java
@Slf4j
@Service
public class VoucherOrderServiceImpl extends ServiceImpl<VoucherOrderMapper, VoucherOrder> implements IVoucherOrderService {

    @Resource
    ISeckillVoucherService seckillVoucherService;

    @Resource
    RedisIDWorker idWorker;

    @Resource
    RedisUtil redisUtil;

    @Resource
    StringRedisTemplate template;

    @Autowired
    RedissonClient redissonClient;

    @Autowired
    RedissonClient redissonClient2;

    // 阻塞队列
    private BlockingQueue<VoucherOrder> orders = new ArrayBlockingQueue<VoucherOrder>(1024*1024);
    // 新建单线程线程池
    private static final ExecutorService SECKILL_ORDER_EXECUTOR = Executors.newSingleThreadExecutor();
    // 异步处理类
    private IVoucherOrderService proxyService;

    private class OrderHandler implements Runnable{
        @Override
        public void run() {
            while (true){
                try {
                    // 取出并创建订单
                    VoucherOrder order = null;
                    order = orders.take( );
                    RLock lock = redissonClient.getLock("order:"+order.getUserId());
                    boolean isLock = lock.tryLock();
                    if(!isLock) {
                        log.error("创建订单失败");
                    }
                    try {
                        // 获取代理对象 用代理对象去调用,否则事务不生效
                        proxyService.createOrderAsync(order);
                    }finally {
                        lock.unlock();
                    }
                } catch (InterruptedException e) {
                    log.error("处理订单异常"+e.getMessage());
                }
            }
        }
    }

    @PostConstruct
    private void init(){
        // 提交任务执行
        SECKILL_ORDER_EXECUTOR.submit(new OrderHandler());
    }



    @Override
    // 创建订单
    public Result create(Long id) throws InterruptedException {
        return createNew(id);
        //return createOld(id);
    }


    // 用新的方法创建
    public Result createNew(Long id) throws InterruptedException {
        SecKillUtils secKillUtils = new SecKillUtils(template);
        if(secKillUtils.canBuy(id,UserHolder.getUser( ).getId( ))){
            // 创建订单对象,
            VoucherOrder voucherOrder = new VoucherOrder( );
            long orderId = idWorker.generate("voucher");
            voucherOrder.setId(orderId);
            voucherOrder.setVoucherId(id);
            voucherOrder.setUserId(UserHolder.getUser( ).getId( ));
            // 存入阻塞队列
            orders.add(voucherOrder);

            // 获取代理对象 用代理对象去调用,否则事务不生效
            proxyService =  (IVoucherOrderService) AopContext.currentProxy();

            return Result.ok(orderId);
        }else{
            return Result.fail("订单创建失败");
        }
    }

    public Result createOld(Long id) throws InterruptedException{
        SeckillVoucher voucher = seckillVoucherService.getById(id);
        if(voucher==null) return Result.fail("优惠券不存在");
        // 1.判断时间是否在开始时间和结束时间中
        if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {
            return Result.fail("秒杀未开始");
        }

        if (voucher.getEndTime().isBefore(LocalDateTime.now())) {
            return Result.fail("秒杀已经结束");
        }
        // 2.判断是否有库存
        if(voucher.getStock()<=0) return Result.fail("库存不足!!");

        Long userId = UserHolder.getUser( ).getId( );
//        // 锁住用户ID,先上锁再开启事务
//        synchronized (userId.toString().intern()){
//            // 获取代理对象
//            IVoucherOrderService service =  (IVoucherOrderService) AopContext.currentProxy();
//            // 用代理对象去调用,否则事务不生效
//            return service.createOrder(id);
//        }

//        ILock lock = new SimpleRedisLock(template,"order:"+userId);
//        boolean isLock = lock.tryLock(1200);
//        if(!isLock) return Result.fail("不允许重复下单,请重试!");
//        try{
//            // 获取代理对象
//            IVoucherOrderService service =  (IVoucherOrderService) AopContext.currentProxy();
//            // 用代理对象去调用,否则事务不生效
//            return service.createOrder(id);
//        }finally {
//            lock.unLock();
//        }

        RLock lock = redissonClient.getLock("order:"+userId);
        boolean isLock = lock.tryLock();
        if(!isLock) return Result.fail("不允许重复下单,请重试!");
        try {
            // 获取代理对象
            IVoucherOrderService service =  (IVoucherOrderService) AopContext.currentProxy();
            // 用代理对象去调用,否则事务不生效
            return service.createOrder(id);
        }finally {
            lock.unlock();
        }
    }

    @Transactional
    public Result createOrder(Long id){
        Long userId = UserHolder.getUser( ).getId( );
        // 额外判断,一人一单
        Integer count = this.query( )
                .eq("user_id", userId)
                .eq("voucher_id", id).count( );
        if(count>0) return Result.fail("请勿重复购买!!");
        // 3.扣减库存
        boolean success = seckillVoucherService.update( )
                .setSql("stock = stock-1")
                .eq("voucher_id", id)
                .gt("stock",0)
                //.notExists("select 1 from tb_voucher_order where voucher_id='"+id+"' and user_id='"+ userId +"'")
                .update( );
        if(!success) return Result.fail("库存不足");

        // 4.创建订单
        VoucherOrder order = new VoucherOrder( );
        long orderId = idWorker.generate("voucher");
        order.setId(orderId);
        order.setVoucherId(id);
        order.setUserId(userId);
        this.save(order);
        return Result.ok(orderId);
    }

    @Transactional
    @Override
    public void createOrderAsync(VoucherOrder order){
        Long userId = order.getUserId();
        // 额外判断,一人一单
        Integer count = this.query( )
                .eq("user_id", userId)
                .eq("voucher_id", order.getVoucherId()).count( );
        if(count>0) log.error("请勿重复购买");
        // 3.扣减库存
        boolean success = seckillVoucherService.update( )
                .setSql("stock = stock-1")
                .eq("voucher_id", order.getVoucherId())
                .gt("stock",0)
                //.notExists("select 1 from tb_voucher_order where voucher_id='"+id+"' and user_id='"+ userId +"'")
                .update( );
        if(!success) log.error("库存不足");

        // 4.创建订单
        this.save(order);
    }
}
```

依然存在问题: 

1. 数据存放在JVM内存中,有溢出风险
2. 若JVM发生异常,有安全问题

### 消息队列

简单的消息队列包含三种角色

- 生产者：发送消息到消息队列
- 消费者：从消息队列获取消息并进行处理，消费者收到消息后要返回确认信息
- 消息队列：存储和管理消息，也可以称为消息代理

redis实现消息队列的方式：

1. list结构模拟
2. PubSub（发布订阅）：基本的点对点消息模型
3. Stream：比较完善的消息队列模型（5.0推出）

#### List模式

优点：

- 利用redis存储，不受限于JVM内存上限
- 基于Redis持久化机制，保证数据安全性
- 可以满足消息有序

缺点：

- 无法避免消息丢失
- 只支持但消费者

#### PubSub模式

> subscribe 频道1 [频道2...] 订阅一个或多个频道
>
> publish 频道 消息 向一个频道发送消息
>
> psubscribe 通配符1 [通配符2...] 订阅符合通配符的所有频道

通配符:

- \* 任意个字符
- ? 单个字符
- [ae] 字符a或者e

优点:

- 支持多生产者,多消费者

缺点:

- 不支持数据持久化
- 无法避免消息丢失
- 消息堆积有上限，超出时数据丢失

#### Stream类型

redis5.0新引入的**数据类型**，可以实现功能非常完善的消息队列

![image-20240827225639901](redis.assets/image-20240827225639901.png)

接受消息：

> XREAD  COUNT 4 BLOCK 0  STREAMS test $

![image-20240827231141790](redis.assets/image-20240827231141790.png)

阻塞处传0代表永久等待

优点：

- 消息可回溯
- 一个消息可以被多个消费者读取
- 可以阻塞读取

缺点:

- 消息有漏读的风险

**消费者组**：将多个消费者划分到一个组中，监听同一个队列，具备下列特点：

1. 消息分流：组内竞争关系，一条消息只会给组内一个消费者消费
2. 消息标识：维护一个标识，记录最后一个被处理的消息，确保消息被消费
3. 消息确认，消费者获取消息后，消息处于pending状态，并存入pending-list列表，处理完成后需要通过XACK确认消息，标记为已处理，才会从列表中移除

创建消费者组:

> XGROUP CREATE key groupName ID [MKSTREAM]

- key: 队列名称

- groupName: 消费者组名称

- ID: 从哪条消息开始消费，$代表最后一条，0代表第一条

- MKSTREAM: 消息队列不存在时是否创建

删除消费者组

> XGROUP DESTORY key groupName

给指定的消费者组添加消费者

> XGROUP CREATECONSUMER key groupname consumername

删除消费者组中的消费者

> XGROUP DELCONSUMER key groupname consumername

从消费者组读取消息

> XREADGROUP GROUP group consumer [COUNT count] [BLOCK milliseconds] [NOACK] STREAMS key [key ...] ID [ID ...]

- group: 消费者组名称

- consumer: 消费者名称,组内不存在自动创建

- count: 本次查询的最大数量

- BLOCK milliseconds: 最长等待时间

- NOACK: 无需手动确认，收到消息自动确认

- STREAMS key: 指定队列名称

- ID: 获取消息的起始ID

- ">": 从第一个未消费的消息开始

- 其他: 根据ID从pending-list中读取已经消费但未确认的消息，0是从pending-list中的第一个消息开始

确认消息:

> XACK key group ID [ID ...]

- 传入的ID为redis生产的那个ID

查看未确认的消息

> XPENDING key group [[IDLE min-idle-time] start end count [consumer]]

- min-idle-time: 从获取消息后未被确认的时间

- start: 最小ID 传 - 代表从开头开始

- end: 最大ID 传 + 代表从最后结束

- count: 获取的数量

- 例如: XPENDING test g1 IDLE - + 100

#### java实现消费者伪代码

```java
while(true){
    // 从队列中获取消息,阻塞2000ms
    if(获取为空){
        // 没消息了,阻塞等待
        continue;
    }
    try{
        // 处理消息
    }catch(){
        while(true){
            // 从pending-list获取消息
            if(获取为空){
                // 未处理的处理完了,跳出循环
                break;
            }
            try{
                // 处理消息
            }catch(){
                // 再次出现异常,记录日志,继续循环
                continue;
            }
        }
    }
}
```

#### 三种方式对比

|              | List                                   | PubSsub          | Stream                                |
| ------------ | -------------------------------------- | ---------------- | ------------------------------------- |
| 消息持久化   | 支持                                   | 不支持           | 支持                                  |
| 阻塞读取     | 支持                                   | 支持             | 支持                                  |
| 消息堆积处理 | 受内存空间限制，可以用多消费者加快处理 | 受限于消息缓冲区 | 受限于队列长度,可以用消费者组减少堆积 |
| 消息确认机制 | 不支持                                 | 不支持           | 支持                                  |
| 消息回溯     | 不支持                                 | 不支持           | 支持                                  |

#### 代码实现

> 处理队列内容

```java
while(true){
    try {
        // 从队列中读取
        List<MapRecord<String, Object, Object>> result = template.opsForStream( ).read(
            Consumer.from("g1", "c1"),
            StreamReadOptions.empty( )
            .count(1)
            .block(Duration.ofSeconds(2)),
            StreamOffset.create("stream.orders", ReadOffset.lastConsumed( ))
        );
        if(result==null || result.isEmpty()) continue;
        // 读取消息并转换成订单对象
        MapRecord<String, Object, Object> record = result.get(0);
        Map<Object, Object> value = record.getValue( );
        VoucherOrder voucherOrder = BeanUtil.toBean(value, VoucherOrder.class);
        // 订单信息写入数据库
        proxyService.createOrderAsync(voucherOrder);
        template.opsForStream().acknowledge("stream.orders","g1", record.getId());
    }catch (Exception e){
        log.error("创建订单时发生错误"+e.getMessage());
        e.printStackTrace();
        processPendingList();
    }
}
```

> 发生异常时的处理逻辑

```java
while(true){
    try{
        // 取出未确认的
        List<MapRecord<String, Object, Object>> result = template.opsForStream( ).read(
            Consumer.from("g1", "c1"),
            StreamReadOptions.empty( )
            .count(1)
            .block(Duration.ofSeconds(2)),
            StreamOffset.create("stream.orders", ReadOffset.from("0"))
        );
        // 取不到了直接返回
        if(result==null || result.isEmpty()) return;
        // 读取消息并转换成订单对象
        MapRecord<String, Object, Object> record = result.get(0);
        Map<Object, Object> value = record.getValue( );
        VoucherOrder voucherOrder = BeanUtil.toBean(value, VoucherOrder.class);
        // 订单信息写入数据库
        createOrderAsync(voucherOrder);
        // 成功了要确认
        template.opsForStream().acknowledge("stream.orders","g1", record.getId());
    }catch (Exception e){
        log.error("创建订单时发生错误"+e.getMessage());
        // 出异常先休眠
        try {
            Thread.sleep(20);
        } catch (InterruptedException interruptedException) {
            interruptedException.printStackTrace( );
        }
    }
}
```

> 判断秒杀资格和写入队列的脚本

```lua
local USER_ID = ARGV[1];
local VOUCHER_ID = ARGV[2];
local STOCK_KEY = 'seckill:stock:'..VOUCHER_ID;
local BUY_KEY = 'seckill:buy:'..VOUCHER_ID;
local ORDER_ID = ARGV[3];

-- 1.查看库存是否超卖
if (tonumber(redis.call('get',STOCK_KEY))<=0) then
    -- 没有库存返回1
    return 1
end
-- 2.查看是否已经购买
if (tonumber(redis.call('sIsMember',BUY_KEY,USER_ID)) == 1) then
    return 2
end
-- 有资格,扣库存并下单
redis.call('incrby',STOCK_KEY,-1)
redis.call('sadd',BUY_KEY,USER_ID)
-- 向队列中添加消息
redis.call('xadd','stream.orders','*',
    'voucherId',VOUCHER_ID,
    'userId',USER_ID,
    'id',ORDER_ID)
-- 返回0
return 0
```

### 点赞功能

使用Sortedset记录点赞记录，判断是否已经点赞使用score指令

查询排名使用rank指令

>  mysql保证查出的数据按照指定的顺序

```mysql
SELECT 字段
FROM 表
WHERE 排序字段 in (1,2,3)
ORDER BY FIELD(排序字段,1,2,3)
```

### 好友关注

#### 共同关注

使用set结构存储数据，使用sinter指令求交集，结果就是共同关注

####　关注推送

Feed流常见模式:

- TimeLine: 不做内容筛选，按照时间排序，常用于好友或关注，例如朋友圈
  - 优点: 信息全面,不会缺失,实现简单
  - 缺点:信息噪音多,用户不一定感兴趣,内容获取效率低下

- 智能推荐: 利用算法智能推荐用户感兴趣的信息吸引用户点击
  - 优点: 投喂用户感兴趣的信息，用户粘度高,容易沉迷
  - 缺点: 如果算法不精准,可能起到反作用

推送模式:

- 拉(读扩散): 数据保存在发件人的发件箱中,用户读取时从发件箱读取并排序
- 推(写扩散):数据保存在收件人的收件箱中,并提前排序好,用户读取时直接从收件箱获取
- 推拉结合: 普通人直接采用推模式,大v设置发件箱,对一般粉丝采用拉模式,活跃粉丝采用推模式

![image-20240904232006448](redis.assets/image-20240904232006448.png)

#### 滚动分页

> 记录上次查询的分数，下次根据分数查询而不是角标

![image-20240905204328602](redis.assets/image-20240905204328602.png)

```redis
ZRevRangeByScore key 最大值 最小值 [WITHSCORES] [LIMIT 偏移量 数据个数]
```

最大值处: 第一次取当前时间戳,第二次取上一次查出的最小值

最小值处: 永远是0

偏移量处: 第一次是0 ,后面为上一次查出数据的最小值在集合中的个数

数据个数; 固定每页大小 

### 附近商户

#### GEO数据结构

> GEOADD key 经度 纬度 值 :添加经纬度坐标
>
> GEODIST key 值1 值2: 计算两个点之间的距离
>
> GEOHASH key 值: 将指定点的坐标转换为hash字符串并返回
>
> GEOPOS key 值: 返回指定点的坐标
>
> GEORADIUS: 给出圆心，半径，计算在这个距离内的点并按距离排序 6.2之后舍弃
>
> GEOSEARCH：范围内搜索，可以是圆形或者矩形 6.2新功能
>
> GEOSEARCHSTORE： 搜索并存储到指定的key 6.2新功能

#### 分页实现

```java
// 计算分页的起始记录数和结束记录数
int start=(current-1)*SystemConstants.DEFAULT_PAGE_SIZE;
int end=current*SystemConstants.DEFAULT_PAGE_SIZE;

// 按照geo距离查询分页好的数据
GeoResults<RedisGeoCommands.GeoLocation<String>> radius = template.opsForGeo( ).radius(
    // key
    RedisConstants.SHOP_GEO_KEY + typeId,
    // 定位圆心和距离
    new Circle(new Point(x, y), new Distance(2.6, Metrics.KILOMETERS)),
    // 包含距离,限制返回结果数
    RedisGeoCommands.GeoRadiusCommandArgs.newGeoRadiusArgs().includeDistance().limit(end)
);
if(radius==null) return new Page<>(  );
// 跳过前面的结果,达成分页的效果
List<GeoResult<RedisGeoCommands.GeoLocation<String>>> list = radius.getContent( ).stream( ).skip(start).collect(Collectors.toList( ));
// 记录id和对应的距离
List<String> ids = new ArrayList<>(  );
HashMap<String,Distance> distanceHashMap = new HashMap<>(  );
for (GeoResult<RedisGeoCommands.GeoLocation<String>> geoLocationGeoResult : list) {
    String id = geoLocationGeoResult.getContent( ).getName( );
    ids.add(id);
    distanceHashMap.put(id,geoLocationGeoResult.getDistance());
}
if(ids.isEmpty()) return new Page<>(  );
// 根据id顺序查询商铺
List<Shop> record = this.query( )
    .in("id", ids)
    .last("ORDER BY FIELD(id,"+StrUtil.join(",",ids)+")").list();
// 商铺设置距离
record.forEach(shop->{
    shop.setDistance(distanceHashMap.get(shop.getId().toString()).getValue());
});
Page<Shop> page = new Page<>( );
page.setRecords(record);
return page;
```

### 用户签到

签到数据用数据库行储存过于耗费空间，可以用二进制比特位01来表示是否签到，这种思路就是位图，bitMap

redis用String数据类型实现bitMap

#### bitMap指令

- SETBIT：指定某个二进制位的值

- GETBIT：获取某个二进制位的值

- BITCOUNT：统计值为1的bit位的数量

- BITFIELD：操作（查询，修改，自增）bitMap中bit数据中的指定位置的值

  - ```java
    bitFIELD testBit GET u3 0
    ```

- BITFIELD_RO：获取BitMap中bit数组，只读

- BITOP：将多个BitMap的结果进行位运算

- BITPOS：查找bit数组中指定范围内第一个0或者1出现的位置

### UV统计

概念:

- UV: 全称Unique Vistor，独立访客量，一个用户多次访问一天只记录一次
- PV: 全称PageView，也叫页面访问量，用户每次访问都记录，记录总流量

HyperLogLog：一种概率算法，用于确定非常大的集合的基数，不需要存储所有值

基于string类型实现，单个内存小于16kb，测量结果有小于0.81%的误差。

- PFADD：添加元素
- PFCOUNT：统计结果
- PFMERGE：两个集合合并