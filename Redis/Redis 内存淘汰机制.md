### Redis 占用内存大小
Redis 基于内存的key-value数据库, 因为系统的内存大小有限制, 所以我们在使用Redis的时候可以配置Redis能使用的最大的内存大小.
#### 1. 通过配置文件配置
```
// 设置Redis最大占用内存大小为100M
maxmemory 100mb
```

#### 2. 通过命令修改
```
// 设置Redis最大占用内存大小为100m
config set maxmemory 100mb
config get maxmemory
```

**注意:** 如果不设置最大内存大小或者设置最大内存大小为0, 在64位操作系统下不限制内存大小, 在32位操作系统下最多使用3GB内存;

### Redis内存淘汰
Redis总有内存使用完毕的情况, 实际上Redis定义了几种策略来处理:
- **noevition:** 对于写请求不再提供服务, 直接返回错误(DEL请求和部分特殊请求除外);
- **allkeys-lr:** 从所有key中使用LRU算法进行淘汰;
- **volatile-ltu:** 从设置了过期时间的key中使用LRU算法进行淘汰;
- **allkeys-random:** 从所有的key中随机淘汰数据;
- **volatile-random:** 从设置了过期时间的key中随机淘汰;
- **volatile-ttl:** 在设置了过期时间的key中, 根据key的过期时间进行淘汰, 越早过期的key越优先被淘汰;
#### 4.0 版本新增淘汰机制LFU(Least Frequently Used)
- **volatile-lfu:** 在设置了过期时间的key中使用LFU算法淘汰key
- **allkeys-lfu:** 在所有的key中使用LFU算法淘汰数据

**注意:** 当使用volatile-lru/volatile-random/volatile-ttl这三种策略时, 如果没有key可以被淘汰, 则和noeviction一样返回错;

### 获取及设置内存淘汰策略

获取当前内存淘汰策略:
```
config get maxmemory-policy
```

通过配置文件设置淘汰策略(修改redis.conf文件):
```
maxmemory-policy allkeys-lru
```

通过命令修改淘汰策略:
```
config set maxmemory-policy allkeys-lru
```

#### LRU算法(Least Recently Used)
即最近最少使用, 是一种缓存置换算法. 在使用内存作为缓存的时候, 缓存的大小一般是固定. 当缓存被占满, 这个时候继续往缓存里面添加数据, 就需要淘汰到一部分老数据, 释放内存空间来存储新的数据. 这个时候就可以使用LRU算法了. 其核心思想是: 如果一个数据在最近一段时间没有被用到, 那么将来被使用的可能性也很小, 所以就可以被淘汰掉;

#### LFU算法(Least Frequently Used)
是Redis4.0里面新加的一种淘汰策略。

**核心思想:**  是根据key的最近被访问的频率进行淘汰，很少被访问的优先被淘汰，被访问的多的则被留下来。

LFU算法能更好的表示一个key被访问的热度。假如你使用的是LRU算法，一个key很久没有被访问到，只刚刚是偶尔被访问了一次，那么它就被认为是热点数据，不会被淘汰，而有些key将来是很有可能被访问到的则被淘汰了。如果使用LFU算法则不会出现这种情况，因为使用一次并不会使一个key成为热点数据。
