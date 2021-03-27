### 从一个简单键值数据库对比 Redis 有哪些优势变化： 

- `Redis` 主要通过网络框架进行访问，而不再是动态库了，这也使得 `Redis` 可以作为一个基础性的网络服务进行访问，扩大了 `Redis` 的应用范围。
- `Redis` 数据模型中的 `value` 类型很丰富，因此也带来了更多的操作接口，例如面向列表的 `LPUSH/LPOP` ，面向集合的 `SADD/SREM` 等.
- `Redis` 的持久化模块能支持两种方式：`日志（AOF）和快照（RDB）`，这两种持久化方式具有不同的优劣势，影响到 `Redis` 的访问性能和可靠性。
- `SimpleKV` 是个简单的单机键值数据库，但是，`Redis` `支持高可靠集群和高可扩展集群，因此，Redis` 中包含了相应的集群功能支撑模块。

![从SimpleKV到Redis](../../Picture/从simpleKV到Redis.jpeg)