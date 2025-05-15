+++
date = '2025-05-15T14:50:04+08:00'
draft = false
title = 'Redis功能模块与源码对应'
+++

# 02：安装源码并总结redis代码整体结构

按照 Redis 的服务器实例、数据库操作、可靠性和可扩展性保证、辅助功能四个维度，把 Redis 功能源码梳理成了四条代码路径

**服务器实例**

<img src="/wwliu.github.io/images/redis_server.png" />

**数据库数据类型与操作**

<img src="/wwliu.github.io/images/redis_data.png" />
<img src="/wwliu.github.io/images/redis_db.png" />

**高可靠性和高可扩展性**

1. 数据持久化实现

Redis 的数据持久化实现有两种方式：内存快照 RDB 和 AOF 日志，分别实现在了 rdb.h/rdb.c 和 aof.c 中
在使用 RDB 或 AOF 对数据库进行恢复时，RDB 和 AOF 文件可能会因为 Redis 实例所在服务器宕机，而未能完整保存，进而会影响到数据库恢复。因此针对这一问题，Redis 还实现了对这两类文件的检查功能，对应的代码文件分别是 redis-check-rdb.c 和 redis-check-aof.c。

2. 主从复制功能实现

Redis 把主从复制功能实现在了 replication.c 文件中。
另外你还需要知道的是，Redis 的主从集群在进行恢复时，主要是依赖于哨兵机制，而这部分功能则直接实现在了 sentinel.c 文件中。
与 Redis 实现高可靠性保证的功能类似，Redis 高可扩展性保证的功能，是通过 Redis Cluster 来实现的，这部分代码也非常集中，就是在 cluster.h/cluster.c 代码文件中。

**辅助功能**

Redis 还实现了一些用于支持系统运维的辅助功能。比如，为了便于运维人员查看分析不同操作的延迟产生来源，Redis 在 latency.h/latency.c 中实现了操作延迟监控的功能；为了便于运维人员查找运行过慢的操作命令，Redis 在 slowlog.h/slowlog.c 中实现了慢命令的记录功能，等等。

此外，运维人员有时还需要了解 Redis 的性能表现，为了支持这一目标，Redis 实现了对系统进行性能评测的功能，这部分代码在 redis-benchmark.c 中。如果你想要了解如何对 Redis 开展性能测试，这个代码文件也值得一读。

ps: 上文内容都是有极客时间大佬整理，我这都是学习