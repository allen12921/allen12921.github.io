---
title: "ClickHouse之Replicated Table Engine"
categories:
  - Blog
tags:
  - db
  - clickhouse
  - replicated
---
![ClickHouse](/assets/images/replicated-vs-sharding.png "ch")
# ClickHouse复制表引擎
  如前所述，CH主要依靠分布式表引擎来实现数据库的水平扩展；而对于数据的高可用，则是需要用到复制表引擎，复制表引擎是ClickHouse实现数据高可用以及提高大规模数据集性能的重要方式，
# 实现
  通过Keeper[^1]保存表的元数据，data parts的变化的日志，leader选举情况,data blocks的checksums。CH之间直接同步变化的data parts。
# 限制
  - 是表级别的引擎，且目前只支持MergeTree系列的表（只需在原MergeTree表引擎前加上Replicated关键字，即为对应的复制表引擎）
  - 只支持复制INSERT,ALTER and TRUNCATE 操作
  - CREATE, DROP, ATTACH, DETACH and RENAME 都只会在当前执行语句的机器上执行（可以结合ON CLUSTER语句在多台执行）
# 一个简单的例子
  ```sql
  CREATE TABLE IF NOT EXISTS mydb.clients ON CLUSTER my_cluster
(
    id              UUID DEFAULT generateUUIDv4(),
    client_id       String,
    created_at      DateTime64(3),
    recorded_at     DateTime DEFAULT now()
)
    ENGINE = ReplicatedMergeTree(recorded_at)
        ORDER BY (client_id);
  ```



[^1]: Zookeeper或者ClickHouse Keeper 
