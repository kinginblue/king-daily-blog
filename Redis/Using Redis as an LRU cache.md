# Using Redis as an LRU cache

Redis 作为 LRU 缓存

## Maxmemory配置指令

可通过 redis.cnf 文件配置该指令，也可使用 CONFIG SET 命令运行时设置。

例如配置内存限制为 100mb：

```conf
maxmemery 100m
```

0 代表没有内存限制。对于 64 位系统，这是默认值，对于 32 位系统隐士的内存限制是 3GB。

## 回收策略（Eviction policies）

- noeviction

    return errors when the memory limit was reached and the client is trying to execute commands that could result in more memory to be used (most write commands, but DEL and a few more exceptions).

- allkeys-lru

    evict keys by trying to remove the less recently used (LRU) keys first, in order to make space for the new data added.

- volatile-lru

    evict keys by trying to remove the less recently used (LRU) keys first, but only among keys that have an expire set, in order to make space for the new data added.

- allkeys-random

    evict keys randomly in order to make space for the new data added.

- volatile-random

    evict keys randomly in order to make space for the new data added, but only evict keys with an expire set.

- volatile-ttl

    evict keys with an expire set, and try to evict keys with a shorter time to live (TTL) first, in order to make space for the new data added.

The policies volatile-lru, volatile-random and volatile-ttl behave like noeviction if there are no keys to evict matching the prerequisites.

一般的经验规则:

- 使用allkeys-lru

    当你希望你的请求符合一个幂定律分布，也就是说，你希望部分的子集元素将比其它其它元素被访问的更多。如果你不确定选择什么，这是个很好的选择。

- 使用allkeys-random

    如果你是循环访问，所有的键被连续的扫描，或者你希望请求分布正常（所有元素被访问的概率都差不多）。

- 使用volatile-ttl

    如果你想要通过创建缓存对象时设置TTL值，来决定哪些对象应该被过期。

## 回收处理是如何工作的（How the eviction process works）

- 一个客户端执行新命令，导致更多的数据被添加。
- Redis 检查内存使用情况，如果大于maxmemory的限制，则根据设定好的策略进行回收。
- 一个新的命令被执行，等等。
- 所以我们不断地穿越内存限制的边界，通过不断达到边界然后不断地回收回到边界以下。

如果一个命令的结果导致大量内存被使用（例如很大的集合的交集保存到一个新的键），不用多久内存限制就会被这个内存使用量超越。

## 近似LRU算法（Approximated LRU algorithm）

Redis LRU算法不是一个确切的实现。这意味着Redis并没办法选择最佳候选来进行回收，也就是最久未被访问的键。相反它会尝试运行一个近似LRU的算法，通过对少量keys进行取样，然后回收其中一个最好的key（被访问时间较早的）。

不过从Redis 3.0算法已经改进为回收键的候选池子。这改善了算法的性能，使得更加近似真是的LRU算法的行为。

Redis LRU有个很重要的点，你通过调整每次回收时检查的采样数量，以实现调整算法的精度。这个参数可以通过以下的配置指令调整:

```conf
maxmemory-samples 5
```

Redis为什么不使用真实的LRU实现是因为这需要太多的内存。不过近似的LRU算法对于应用而言应该是等价的。

## 新的 LRU 模型（The new LFU mode）

Redis 4.0 支持了 LFU 回收模型。

配置 LFU 时可使用以下策略：

- volatile-lfu

    Evict using approximated LFU among the keys with an expire set.

- allkeys-lfu

    Evict any key using approximated LFU.

与 LRU 不同的是，LFU 具有一些可调参数。for instance, how fast should a frequent item lower in rank if it gets no longer accessed? It is also possible to tune the Morris counters range in order to better adapt the algorithm to specific use cases.

Redis 4.0 默认配置是：

- Saturate the counter at, around, one million requests.
- Decay the counter every one minute.

源码里面的示例 redis.conf 文件中可以找到这些可调参数：

```conf
lfu-log-factor 10
lfu-decay-time 1
```

The decay time is the obvious one, it is the amount of minutes a counter should be decayed, when sampled and found to be older than that value. A special value of 0 means: always decay the counter every time is scanned, and is rarely useful.

The counter logarithm factor changes how many hits are needed in order to saturate the frequency counter, which is just in the range 0-255. The higher the factor, the more accesses are needed in order to reach the maximum. The lower the factor, the better is the resolution of the counter for low accesses.

## 附录

- 原官方文档：[Using Redis as an LRU cache](https://redis.io/topics/lru-cache)
- 中文（非最新）：[将redis当做使用LRU算法的缓存来使用](http://www.redis.cn/topics/lru-cache.html)