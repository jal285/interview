# redis实现分布式锁

<!--more-->

我们在系统中修改已有数据时，需要先读取，然后进行修改保存，此时很容易遇到并发问题。由于修改和保存不是原子操作，在并发场景下，部分对数据的操作可能会丢失。在单服务器系统我们常用本地锁来避免并发带来的问题，然而，当服务采用集群方式部署时，本地锁无法在多个服务器之间生效，这时候保证数据的一致性就需要分布式锁来实现。

![img](../images/Redis%E5%88%86%E5%B8%83%E9%94%81/redis-lock-01.png)

## 实现



### redisson+Rlock可重入锁





### 使用setNX+Lua脚本

Redis锁主要利用Redis的setnx命令

- 加锁命令：SETNX key value，当键不存在时，对键进行设置操作并返回成功，否则返回失败。KEY 是锁的唯一标识，一般按业务来决定命名。
- 解锁命令：DEL key，通过删除键值对释放锁，以便其他线程可以通过 SETNX 命令来获取锁。
- 锁超时：EXPIRE key timeout, 设置 key 的超时时间，以保证即使锁没有被显式释放，锁也可以在一定时间后自动释放，避免资源被永远锁住。

加锁伪代码

```
if (setnx(key, 1) == 1){
    expire(key, 30)
    try {
        //TODO 业务逻辑
    } finally {
        del(key)
    }
}
```

[后续参考](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/)

setNX 

完整语法：SET key value [EX seconds|PX milliseconds] [NX|XX] [KEEPTTL]

必选参数说明：

- SET：命令
- key：待设置的key
- value：设置的key的value，最好为随机字符串

可选参数说明：

- NX：表示key不存在时才设置，如果存在则返回 null
- XX：表示key存在时才设置，如果不存在则返回NULL
- PX millseconds：设置过期时间，过期时间精确为毫秒
- EX seconds：设置过期时间，过期时间精确为秒

使用setnx的锁实现存在一些问题



### SETNX和EXPIRE 非原子性

SETNX成功,设置的锁超时时间后, 服务器挂掉、重启或网络问题等，导致EXPIR命令没有执行，锁没有设置超时时间变成死锁 

![img](../images/Redis%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81/redis-lock-02.png)

比较多的开源代码解决这个问题，附上一个lua脚本

```
if (redis.call('setnx', KEYS[1], ARGV[1]) < 1)
then return 0;
end;
redis.call('expire', KEYS[1], tonumber(ARGV[2]));
return 1;

// 使用实例
EVAL "if (redis.call('setnx',KEYS[1],ARGV[1]) < 1) then return 0; end; redis.call('expire',KEYS[1],tonumber(ARGV[2])); return 1;" 1 key value 100

```

