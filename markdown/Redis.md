# Redis[面试](https://blog.csdn.net/a745233700/category_10727715.html)

## 数据类型

```java
String: 字符串
Hash: 散列
List: 列表
Set: 集合
Sorted Set: 有序集合
sadd  
smembers set01:查看
sismenber key member:判断
scard key:length
srandmenber key number/-number :随机取出
spop key [count]:随机移除
smove set01 set02 key
sinter set01 [set02]:都有
sunion set01 set02 set03
    
list(有序有下标)
lpop/rpop(移除表尾)
lpush/rpush(尾)
lrange
lindex key index:指定列表下标
llen:length
lrem key count value

hash(无序不重复)
hset key  k1 v1 [k2 v2]
hmset stu1002 id 100 name lisi
hget key k1
hmget key k1 k2
hgetall：所有 k和v
hlen
hexists
hvals
hkeys stu001：获取所有key
hvals stu001:获取所以value
hsetnx:k有择失败

zset(有序集合  不能重复)

```

---

## docker-redis


```shell
1. config get requirepass       查看redis密码
2. *docker exec -it  redis /bin/bash*      进入redis容器
3. redis-cli -h host -p port -a password  进入redis
4. docker container rm  删除容器

# 关闭redis服务器
redis-cli -h 127.0.0.1 -p 6379 shutdown
# 杀死redis服务器（比较暴力，谨慎使用）
sudo kill -9 pid 进程号
# 指定配置文件启动redis
sudo redis-server /etc/redis/redis.conf
# 查看redis服务器进程
ps -ef | grep redis
ps aux | grep redis
# 启动redis客户端
redis-cli
redis-cli -h ip地址
redis-cli -p 端口号
```



---

## 1.3springboot集成redis

```java
# 引入maven依赖  
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```



---

## 缓存穿透、缓存击穿、缓存雪崩

> 一、缓存穿透
> 描述
>   指访问一个缓存和数据库中都不存在的key，由于这个key在缓存中不存在，则会到数据库中查询，数据库中也不存在该key，无法将数据添加到缓存中，所以每次都会访问数据库导致数据库压力增大。
>
> 解决方法
> 将空key添加到缓存中。
> 使用布隆过滤器过滤空key。
> 一般对于这种访问可能由于遭到攻击引起，可以对请求进行身份鉴权、数据合法行校验等。
> 二、缓存击穿
> 描述
>   指大量请求访问缓存中的一个key时，该key过期了，导致这些请求都去直接访问数据库，短时间大量的请求可能会将数据库击垮。
>
> 解决方法
>
> - 添加互斥锁或分布式锁，让一个线程去访问数据库，将数据添加到缓存中后，其他线程直接从缓存中获取。(通常方案-p44)
>
> ​       添加锁 setnx lock 1 100ttl(防止死锁)
>
> ​       删除锁delete lock
>
> ![image-20230216221945209](http://cdn.pengfanao.top/image-20230216221945209.png)
>
> ![image-20230216222801355](http://cdn.pengfanao.top/image-20230216222801355.png)
>
> - 热点数据key不过期，定时更新缓存，但如果更新出问题会导致缓存中的数据一直为旧数据。(根据业务场景)
>
> - 逻辑过期：子线程重建缓存
>
> ![image-20230216225122765](http://cdn.pengfanao.top/image-20230216225122765.png)
>
> 
>
> 三、缓存雪崩
> 描述
>   指在系统运行过程中，缓存服务宕机或大量的key值同时过期，导致所有请求都直接访问数据库导致数据库压力增大。
>
> 解决方法
> 将key的过期时间打散，避免大量key同时过期。
> 对缓存服务做高可用处理。
> 加互斥锁，同一key值只允许一个线程去访问数据库，其余线程等待写入后直接从缓存中获取。

### 一致性

- redis内存回收

- ttl过期时间

- 主动更新

  - 先删除缓存，在更新数据库
  - 先更新数据库在删除缓存（一般方案，操作内存快，操作数据库慢）

  ![image-20230216203236229](http://cdn.pengfanao.top/image-20230216203236229.png)

### redisssion

- 可重入

![image-20230217213139563](http://cdn.pengfanao.top/image-20230217213139563.png)

![image-20230217213321261](http://cdn.pengfanao.top/image-20230217213321261.png)

![image-20230217213343531](http://cdn.pengfanao.top/image-20230217213343531.png)

```java
//redisson 可重入  可重试锁 

public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
        long time = unit.toMillis(waitTime);
        long current = System.currentTimeMillis();
        long threadId = Thread.currentThread().getId();
        Long ttl = tryAcquire(leaseTime, unit, threadId);
        // lock acquired
    //拿到锁 返回
        if (ttl == null) {
            return true;
        }
         
        time -= System.currentTimeMillis() - current;
        if (time <= 0) {
            acquireFailed(threadId);
            return false;
        }
        
        current = System.currentTimeMillis();
        RFuture<RedissonLockEntry> subscribeFuture = subscribe(threadId);
        if (!await(subscribeFuture, time, TimeUnit.MILLISECONDS)) {
            if (!subscribeFuture.cancel(false)) {
                subscribeFuture.onComplete((res, e) -> {
                    if (e == null) {
                        unsubscribe(subscribeFuture, threadId);
                    }
                });
            }
            acquireFailed(threadId);
            return false;
        }

        try {
            time -= System.currentTimeMillis() - current;
            if (time <= 0) {
                acquireFailed(threadId);
                return false;
            }
        
            while (true) {
                long currentTime = System.currentTimeMillis();
                ttl = tryAcquire(leaseTime, unit, threadId);
                // lock acquired
                if (ttl == null) {
                    return true;
                }

                time -= System.currentTimeMillis() - currentTime;
                if (time <= 0) {
                    acquireFailed(threadId);
                    return false;
                }

                // waiting for message
                currentTime = System.currentTimeMillis();
                if (ttl >= 0 && ttl < time) {
                    getEntry(threadId).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                } else {
                    getEntry(threadId).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
                }

                time -= System.currentTimeMillis() - currentTime;
                if (time <= 0) {
                    acquireFailed(threadId);
                    return false;
                }
            }
        } finally {
            unsubscribe(subscribeFuture, threadId);
        }
//        return get(tryLockAsync(waitTime, leaseTime, unit));
    }
```

### 业务未执行完ttl过期问题

- redisson锁

  > 可重入：redis的hash结构
  >
  > 可重试：waitTime 、leaseTiem（TTl）、锁释放消息订阅
  >
  > 业务超时锁过期问题：判断leasTiem是否等于-1启用watchdog（看门狗）
  >
  > 分布式集群一致性问题：不分主库和从库，multLock多把锁结合成一把锁

### 秒杀业务解耦

![image-20230219132835067](http://cdn.pengfanao.top/image-20230219132835067.png)

![image-20230219133419541](http://cdn.pengfanao.top/image-20230219133419541.png)

**生成订单**

```lua
--  判断是否下单---->
-- 1.参数列表
-- 1.1.优惠券id
local voucherId = ARGV[1]
-- 1.2.用户id
local userId = ARGV[2]
-- 1.3.订单id
local orderId = ARGV[3]

-- 2.数据key
-- 2.1.库存key
local stockKey = 'seckill:stock:' .. voucherId
-- 2.2.订单key
local orderKey = 'seckill:order:' .. voucherId

-- 3.脚本业务
-- 3.1.判断库存是否充足 get stockKey
if(tonumber(redis.call('get', stockKey)) <= 0) then
    -- 3.2.库存不足，返回1
    return 1
end
-- 3.2.判断用户是否下单 SISMEMBER orderKey userId
if(redis.call('sismember', orderKey, userId) == 1) then
    -- 3.3.存在，说明是重复下单，返回2
    return 2
end
-- 3.4.扣库存 incrby stockKey -1
redis.call('incrby', stockKey, -1)
-- 3.5.下单（保存用户）sadd orderKey userId
redis.call('sadd', orderKey, userId)
-- 3.6.发送消息到队列中， XADD stream.orders * k1 v1 k2 v2 ...   下单队列
redis.call('xadd', 'stream.orders', '*', 'userId', userId, 'voucherId', voucherId, 'id', orderId)
return 0
```

**消费订单**

- redis:扣减库存,使用阻塞队列指令LRPUSH、LRPOP

![image-20230219165204989](http://cdn.pengfanao.top/image-20230219165204989.png)

- 订阅型消费

  SUBSCRIBLE、PUBLISH

  ![image-20230219170532961](http://cdn.pengfanao.top/image-20230219170532961.png)

- stream实现消息队列
  - XADD: XADD S! * k1 v1
  - XREDE:XREAD COUNT 0  [block  0] SREAMS S1 0（哪个singal）   $(最新)
  - XLEN
  
  缺点：有消息遗漏，由于是阻塞的消息队列，当拿到最新的消息后，中间有连续几条新校西，只会拿到最后的消息
  
- stream消息消费组

  - XGROUP
