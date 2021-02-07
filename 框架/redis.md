## 概述

基于键值对的NoSQL数据库，它的值支持多个数据库

将所有的数据都存在内存中，所以它的读写性能十分惊人

同时，Redis还可以将内存中的数据以快照或日志的形式保存在硬盘中，以保证数据的安全性

Redis典型的应用场景包括：缓存、排行版、计数器、社交网络



快照（RDB）：把内存中的数据直接存在硬盘中，优点是内存中数据小，恢复也很快，缺点是存储的比较耗时，而且会产生阻塞，影响其他的业务，适合定时备份

日志（AOF）：每进行一个命令就以日志存下来，实时存，不断追加，比较占磁盘空间，如果恢复的话是所有命令再跑一遍，所以恢复较慢，优点是实时性好



## 常用命令

默认内置16个库，无名字，用索引区分（0-15），默认第0个

redis-cli：连上Redis

select X:选择某个库

flushdb：刷新库里的数据（清空）

```
有两个单词相连，中间用冒号相隔
//String类型的
set test:count 1//添加   redisTemplate.opsForValue().set(redisKey, 1);
get test:count//获取	   redisTemplate.opsForValue().get(redisKey);
incr test:count//增加	   redisTemplate.opsForValue().increment(redisKey)
decr test:count//减少    redisTemplate.opsForValue().decrement(redisKey)

//hash
hset set:user id 1//存     redisTemplate.opsForHash().put(redisKey, "id", 1);
hset set:user username zhangsan	   
				redisTemplate.opsForHash().put(redisKey, "username", "zhangsan");
hget test:user id         redisTemplate.opsForHash().get(redisKey, "id")
hget test:user username    redisTemplate.opsForHash().get(redisKey, "username")

//横向容器
//list,是左进存的，比如依次存1,2,3，那么位置关系是3,2,1
//后出，也就是按存进的顺序出
lpush test:ids 101 102 103//存 redisTemplate.opsForList().leftPush(redisKey, 101);
llen test:ids//长度  redisTemplate.opsForList().size(redisKey)
lindex test:ids 0//某个索引  redisTemplate.opsForList().index(redisKey, 0)
lrange test:ids 0 2//某个范围内 redisTemplate.opsForList().range(redisKey, 0, 2)
rpop test:ids//右侧出一个值  redisTemplate.opsForList().rightPop(redisKey)

//set，实现抽奖，不能重复，无序
sadd test:teachers aaa bbb ccc ddd eee//添加
   redisTemplate.opsForSet().add(redisKey, "刘备", "关羽", "张飞", "赵云", "诸葛亮");
scard test:teachers//数量  redisTemplate.opsForSet().size(redisKey)
spop test:teachers//从集合中随机弹出一个值   redisTemplate.opsForSet().pop(redisKey)
smembers test:teachers//查看所有元素   redisTemplate.opsForSet().members(redisKey)

//zset有序，提供一个按分数集合,默认是由小到大
zadd test:students 10 aaa 20 bbb 30 ccc 40 ddd 50 eee//存
	redisTemplate.opsForZSet().add(redisKey, "唐僧", 80);
	
zcard test:students//数量  redisTemplate.opsForZSet().zCard(redisKey)
	
zscore test:students ccc//查看某个值得分数 	   redisTemplate.opsForZSet().score(redisKey, "八戒")

zrank test:students ccc	//排名 
		redisTemplate.opsForZSet().reverseRank(redisKey, "八戒")
zrange test:students 0 2//范围内
		redisTemplate.opsForZSet().reverseRange(redisKey, 0, 2)

//全局命令
keys *
keys test* 
type test:user
exists key test:user redisTemplate.hasKey("test:user")
del test:user        redisTemplate.delete("test:user");
expire test:students 10//设置key过期时间
					redisTemplate.expire("test:students", 10, TimeUnit.SECONDS);
```



```java
 // 批量发送命令,节约网络开销.
    @Test
    public void testBoundOperations() {
        String redisKey = "test:count";
        BoundValueOperations operations = redisTemplate.boundValueOps(redisKey);
        operations.increment();
        operations.increment();
        operations.increment();
        operations.increment();
        operations.increment();
        System.out.println(operations.get());
    }

//提交事务的时候会一股脑的发，所以事务内的命令不会立刻执行，所以不要在事务中做查询，一般用编程式事务
// 编程式事务
    @Test
    public void testTransaction() {
        Object result = redisTemplate.execute(new SessionCallback() {
            @Override
            public Object execute(RedisOperations redisOperations) throws DataAccessException {
                String redisKey = "text:tx";

                // 启用事务
                redisOperations.multi();
                redisOperations.opsForSet().add(redisKey, "zhangsan");
                redisOperations.opsForSet().add(redisKey, "lisi");
                redisOperations.opsForSet().add(redisKey, "wangwu");

                System.out.println(redisOperations.opsForSet().members(redisKey));

                // 提交事务
                return redisOperations.exec();
            }
        });
        System.out.println(result);
    }
```





## 基本数据结构

| 数据类型    | 最大存储数据量 |
| ----------- | -------------- |
| key         | 512M           |
| string      | 512M           |
| hash        | 2^32-1         |
| list        | 2^32-1         |
| set         | 2^32-1         |
| sorted set  |                |
| bitmap      | 512M           |
| hyperloglog | 12k            |

HyperLogLog

采用一种基数算法，用于完成独立总数的统计

占据空间小，无论统计多少个数据，只占12K的内存空间

不精确的统计算法，精准误差为0.81%



Bitmap

不是一种独立的数据结构，实际上就是字符串

支持按位存取数据，可以将其看成是byte数组

适合检索大量的连续的数据的布尔值



## 过期策略

​	Redis会把设置了过期时间的key放入一个独立的字典里，在key过期的时候并不会立刻删除它。

​	Redis会通过如下两种策略，来删除过期的key:

- 惰性删除

  客户端访问某个key时，Redis会检查该key是否过期，若过期则删除

- 定期扫描

  Redis默认每秒执行10次过期扫描(配置hz选项)，扫描策略如下：

  1. 从过期字典中随机选择20个key
  2. 删除这20个key中已过期的key
  3. 如果过期的key比例超过25%，则重复步骤1

  

  

## 淘汰策略

当Redis占用内存超过最大限制时，可采用如下策略，让Redis淘汰一些数据，以腾出空间继续提供读写服务：

1. noeviction:对可能导致增大内存的命令返回错误（大多数写命令，DEL除外）
2. volatile-ttl:在设置了过期时间的key中，选择剩余寿命最短的key，将其淘汰
3. volatile-lru:在设置了过期时间的key中，选择最少使用的key，将其淘汰
4. volatile-random:在设置了过期时间的key中，随机选择一些key，将其淘汰
5. allkeys-lru:在所有的key中，选择最小使用的key,将其淘汰
6. allkeys-random:在所有的key中，随机选择一些key，将其淘汰



## LRU算法

维护一个链表，用于顺序存储被访问过的key。在访问数据时，最新访问的key将被移动到表头，即最近访问的key在表头，最少访问的key在表尾。



## 近似LRU算法

​	给每个key维护一个时间戳，淘汰时随机采样5个key，从中淘汰最旧的key，如果还是超出内存限制，则继续随机采样淘汰

​	优点：比LRU算法节约内存，却可以取得非常近似的效果



## 缓存穿透

### 场景

查询根本不存在的数据，使得请求直达存储层，导致其负载过大，导致宕机



### 解决方案

1. 缓存空对象

   存储层未命中后，仍然将空值存入缓存层，再次访问改数据时，缓存层会直接返回空值

2. 布隆过滤器

   将所有存在的key提前存入布隆过滤器，在访问缓存层之前，先通过过滤器拦截，若请求的是不存在的key，则直接访问控制



## 缓存击穿

### 场景

一份热点数据，它的访问量非常大，在其缓存失效瞬间，大量请求直达存储层，导致服务崩溃



### 解决方案

1. 加互斥锁

   对数据的访问加互斥锁，当一个线程访问改数据时，其他县城只能等待。这个线程访问过后，缓存中的数据将被重建，届时其他线程就可以直接从缓存取值

2. 永不过期

   不设置过期时间，所以不会出现上诉问题，这是“物理”上的不过期。为每个value设置逻辑过期时间，当发现该值逻辑过期时，使用单独的线程重建缓存



## 缓存雪崩

由于某些原因，缓存层不能提供服务，导致所有的请求直达存储层，造成存储层宕机



## 解决方案

1. 避免同时过期

   设置过期时间时，附加一个随机数，避免大量的key同期过期

2. 构建高可用的Redis缓存

   部署多个Redis实例，个别节点宕机，依然可以保持服务的整体可用

3. 构建多级缓存

   增加本地缓存，在存储层前面多加一级屏障，降低请求直达存储层的几率

4. 启用限流和降级措施

   对存储层增加限流措施，当请求超过措施时，对其提供降级服务



## 分布式锁

### 场景

修改时，经常需要先将数据读取到内存，在内存中修改后再存回去。在分布式应用中，可能多个进程同时执行上述操作，而读取和修改不是原子操作，所以会产生冲突。增加分布式锁，可以解决此类问题



### 基本问题

同步锁：在多个线程都能访问到的地方，做一个标记，标识该数据的访问权限

分布式锁：在多个进程都能访问到的地方，做一个标记，标识改数据的访问权限



### 实现方式

1. 基于数据库实现分布式锁
2. 基于Redis实现分布式锁
3. 基于zookeeper实现分布式锁



### Redis实现分布式锁的原则

1. 安全属性：独享。在任一时刻，只有一个客户端持有锁
2. 活性A：无死锁。即便持有锁的客户端崩溃或者网络被分裂，锁仍然可以被获取
3. 活性B：容错。只要大部分Redis节点都存活，客户端就可以获取和释放锁



## 单Redis实例实现分布式锁

1. 获取锁使用命令

   set 资源名 随机值 nx px 30000

   nx:仅在key不存在时才执行成功（not exist）

   px:设置锁的自动过期时间

2. 通过Lua脚本释放锁

   if redis.call("get",keys[1]==argv[1]) then

   ​		return redis.call("del",keys[1])

   else return 0 end



可以避免删除别的客户端获取成功的锁：

A加锁->A阻塞->因超时释放锁->B加锁->A恢复->释放锁



## 多Redis实例实现分布式锁

Redlock算法，该算法有现成的实现，其Java版本的库为Redisson

1. 获取当前Unix时间，以毫秒为单位。
2. 依次尝试从N个实例，使用相同的key和随机值获取锁，并设置响应超时时间。如果服务器没有在规定时间响应，客户端应该尽快尝试另外一个Redis实例
3. 客户端使用当前时间减去开始获取锁的时间，得到获取锁的使用时间。当且仅当大多数数的Redis节点都取到锁，并且使用的时间小于锁失效时间时，所才算取得成功
4. 如果取到了锁，key的真正有效时间等于有效时间减去获取锁使用的时间
5. 如果获取锁失败，客户端应该在所有的Redis实例上进行解锁







## 在Spring整合Redis

1. 引入依赖

2. 配置Redis:配置数据库参数，编写配置类，构造RedisTemplate

3. 访问Redis

   redisTemplate.opsForValue()

   redisTemplate.opsForHash()

   redisTemplate.opsForList()

   redisTemplate.opsForSet()

   redisTemplate.opsForZSet()



配置类详细写法







# 底层原理之SDS

[底层原理之SDS](https://blog.csdn.net/water12233/article/details/89544385)

