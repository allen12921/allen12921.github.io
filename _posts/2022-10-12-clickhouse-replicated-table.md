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
# 读
  - 对replicated tables读操作不会经过Keeper，因此其性能和读non-replicated table时一致
  - 当读distributed replicated tables时，读的行为还和[max_replica_delay_for_distributed_queries](https://clickhouse.com/docs/en/operations/settings/settings/#settings-max_replica_delay_for_distributed_queries)以及[fallback_to_stale_replicas_for_distributed_queries](https://clickhouse.com/docs/en/operations/settings/settings/#settings-fallback_to_stale_replicas_for_distributed_queries)
# 写
  - 由于INSERT操作会在Keeper中添加数个entries,因此其操作时间会比写non-replicated tables时更长，所以我们需要尽量将多个insert合并在一起进行batch操作，且每秒不要超过1个INSERT操作
  - 默认当INSERT在任意一个replica上执行完成时，就会返回成功，如果此CH突然宕机则可能造成数据丢失，我们可以通过修改[insert_quorum](https://clickhouse.com/docs/en/operations/settings/settings/#settings-insert_quorum)来更改其行为
  - 数据在多个replicas之间的复制是异步的，且多个replics之间无主备之分，INSERT和ALTER在其运行的CH上执行，其产生的数据变化会异步地被复制到其他replicas,执行后台任务的线程数量由[background_schedule_pool_size](https://clickhouse.com/docs/en/operations/settings/settings/#background_schedule_pool_size)决定
  - 每个不大于[max_insert_block_size](https://clickhouse.com/docs/en/operations/settings/settings/#max_insert_block_size)的写入都是具有原子性的
  - INSERT具有简单的去重功能，对于相同data block(相同大小的数据块，以相同的顺序包含相同的行)的连续多次写操作，只会执行一次
# 一个简单的例子
- 在config.xml中添加cluster和zk配置
```xml
<remote_servers>
    <my_cluster>
        <!-- <secret></secret> -->
        <shard>
            <!-- 可选的。写数据时分片权重。 默认: 1. -->
            <weight>1</weight>
            <!-- 可选的。写入分布式表时是否只将数据写入其中一个副本。默认值:false(将数据写入所有副本),设置为ture时，
DT只会写入shard中的单个节点，其它节点依赖*ReplicaMergeTree表内部机制实现复制 -->
            <internal_replication>true</internal_replication>
            <replica>
                <!-- 可选的。负载均衡副本的优先级。默认值:1(值越小优先级越高)。 -->
                <priority>1</priority>
                <host>host-sh01-01</host>
                <port>9000</port>
            </replica>
            <replica>
                <host>host-sh01-02</host>
                <port>9000</port>
            </replica>
        </shard>
    </my_cluster>
</remote_servers>
<zookeeper incl="zookeeper-servers" optional="true" />
    <zookeeper>
      <node>
        <host>zk01</host>
        <port>2181</port>
      </node>
      <node>
        <host>zk02</host>
        <port>2181</port>
      </node>
      <node>
        <host>zk03</host>
        <port>2181</port>
     </node>
</zookeeper>
```
- 利用on cluster在集群的所有机器上创建复制表
  ```sql
  CREATE TABLE IF NOT EXISTS mydb.clients ON CLUSTER my_cluster (
    id              UUID DEFAULT generateUUIDv4(),
    client_id       String,
    created_at      DateTime64(3),
    recorded_at     DateTime DEFAULT now()
    )
    ENGINE = ReplicatedMergeTree(recorded_at)
        ORDER BY (client_id);
  ```



[^1]: Zookeeper或者ClickHouse Keeper 
