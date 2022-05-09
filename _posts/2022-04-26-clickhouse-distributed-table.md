---
title: "ClickHouse之Distributed Table Engine"
categories:
  - Blog
tags:
  - db
  - clickhouse
  - 分布式引擎
---
![ClickHouse](/assets/images/clickhouse-logo.jpeg "ch")
# Clickhouse是什么？
Clickhouse是一个面向列的数据库管理系统(DBMS)，用于在线查询分析处理(OLAP),它支持多种database engine[^4]和table engine，不同的table engine[^5]提供不同的特性，以满足不同的业务场景。

# 分布式表引擎
表引擎众多，可能你不会用到所有的类型，但有一种引擎是迟早会用到的，它就是分布式引擎，因为它是当table中数据达到一定量级时，进行水平扩展的主要方式。
*分布式引擎本身不存储数据*, 但可以在cluster中的多个服务器上进行分布式并发查询。cluster通过配置文件来定义.
分布式表创建语句如下:
```sql
CREATE TABLE [IF NOT EXISTS] [db.]table_name [ON CLUSTER cluster]
(
    name1 [type1] [DEFAULT|MATERIALIZED|ALIAS expr1],
    name2 [type2] [DEFAULT|MATERIALIZED|ALIAS expr2],
    ...
) ENGINE = Distributed(cluster, database, table[, sharding_key[^6][, policy_name]])
[SETTINGS name=value, ...]
```

下面的例子中我们将会在名为my_cluster的cluster中创建users_all的分布式表,它的数据存储在my_cluster中所有shard上的default.users表中，首先需改所有server上的config.xml，注意其中的internal_replication[^1] 配置:
```xml
<remote_servers>
    <my_cluster>
        <!-- <secret></secret> -->
        <shard>
            <!-- 可选的。写数据时分片权重。 默认: 1. -->
            <weight>1</weight>
            <!-- 可选的。写入分布式表时是否只将数据写入其中一个副本。默认值:false(将数据写入所有副本),设置为ture时，
DT只会写入shard中的单个节点，其它节点依赖*ReplicaMergeTree表内部机制实现复制 -->
            <internal_replication>false</internal_replication>
            <replica>
                <!-- 可选的。负载均衡副本的优先级。默认值:1(值越小优先级越高)。 -->
                <priority>1</priority>
                <host>host-sh01-01</host>
                <port>9000</port>
            </replica>
        </shard>
        <shard>
            <weight>2</weight>
            <internal_replication>false</internal_replication>
            <replica>
                <host>host-sh02-01</host>
                <port>9000</port>
            </replica>
            <replica>
                <host>host-sh02-02</host>
                <secure>1</secure>
                <port>9440</port>
            </replica>
        </shard>
    </my_cluster>
</remote_servers>
```
在cluster中的所有shard上单独创建数据表:
```sql
CREATE TABLE users
(
    user_id UInt64,
    age Int32,
    name String
) ENGINE = MergeTree()
PARTITION BY (user_id)
ORDER BY (age)

```
在应用程序需要直连的CH上创建分布式表,其中`user_id`为我们指定的`sharding_key`(*强烈建议设置sharding_key，这样我们才能通过分布式表进行数据写入*）:
```sql
CREATE TABLE users_all AS users
ENGINE = Distributed(my_cluster, default, users, user_id)
SETTINGS
    fsync_after_insert=0,
    fsync_directories=0;
```
## 读取操作
将查询请求转发到多个远端服务器进行并行查询，然后返回合并后的查询结果。
```sql
SELECT name FROM users_all WHERE user_id in (1,2,3);
```
<div class="mermaid">
graph LR
  client --> dt_table
  dt_table --> shardA
  dt_table --> shardB
  dt_table --> shardN
</div>

## 写入操作
- 直接将请求发送到存储数据的db
```sql
INSERT INTO
  users (user_id, age, name)
VALUES
  (1, 18, 'King');
```
<div class="mermaid">
graph LR
  client --> shardA
  client --> shardB
  client --> shardN  
</div>
  
- 将请求发送到分布式表，再由分布式表所在服务器将请求分配到不同的数据存储服务器,此种模式需要在创建分布式表时包含`sharding_key`参数[^2]
```sql
INSERT INTO
  users_all (user_id, age, name)
VALUES
  (2, 19, 'Queen'),(3, 1, 'Princess');
```
  - CH根据shard选取计算表达式 `sharding_key_value % sum_weight`的值来决定将数据保存到哪个shard,每个shard包含其`[’prev_weight’,’prev_weights + weight’)`范围内的数据，其中`prev_weight`为该分片前面的所有分片的权重和。
<div class="mermaid">
graph LR
  client --> dt_table
  dt_table --> shardA
  dt_table --> shardB
  dt_table --> shardN
</div>

## 分布式表使用技巧
- 查询开启全局GLOBAL IN / GLOBAL JOINs兼容现有SQL并减少出错机率。
  ```sql
   SELECT uniq(user_id) FROM users_all 
   WHERE age = 101 AND user_id GLOBAL IN (SELECT user_id FROM users_all WHERE name like 'allen%')
  ``` 
  首先会在发起查询的机器运行子查询,其结果会被以临时表(_data1)的形式保存在内存中:
  ```sql
  SELECT user_id FROM users_all WHERE name like 'allen%' 
  ```
  然后下面的语句以及临时表都会被发送到cluster中的所有机器执行:
  ```sql
  SELECT uniq(user_id) FROM users_all 
  WHERE age = 101 AND user_id GLOBAL IN _data1
  ```
- 使用分布式DDL(ON CLUSTER条件)进行表管理
  CREATE、DROP、ALTER和RENAME都可以使用ON CLUSTER子句以分布式方式运行在cluster中的所有shard中[^7]
  ```sql
  AlTER TABLE users ON CLUSTER my_cluster ADD COLUMN IF NOT EXISTS gender String AFTER user_id
  ```
- 合理设置sharding_key,减少查询请求，提高查询效率
  查询条件中包含sharding_key，配合设置optimize_skip_unused_shards=1，排除掉不需要的shards
  ```sql
  SELECT age FROM user_all WHERE user_id = '1212322321'
  ```
  利用IN or JOIN在本地表进行查询
  ```sql
   SELECT uniq(name) FROM users_all 
   WHERE age IN (SELECT age FROM users WHERE user_id in (1,2,3))
  ```
- 化整为零，分散压力
  在cluster中所有shard上都创建分布式表，通过LB[^3]将适用的请求按照一定规则转发到shard中
- 保证数据实时性
  默认数据异步写入，会先保存在分布式表本地再发送到远端shard,通过设置insert_distributed_sync=1来保证所有数据在所有shard上保存成功后才返回
  ```sql
  INSERT INTO
  users_all (user_id, age, name)
  VALUES
  (2, 19, 'Queen'),(3, 1, 'Princess') SETTINGS insert_distributed_sync=1;
  ```
-  在大型集群中,对数据按照逻辑进行分层，不同的业务类型的客户端直连shard server，将不同的类型数据直接写入不同的shard
## 待解决的问题
- 写入分布式表时，部分数据写入失败时的处理。

> 参考
> > https://clickhouse.com/docs/en/engines/table-engines/special/distributed

[^1]: internal_replication参数为true时,需要和Replication系列表配合使用，由于分布式表和数据副本并无直接联系，二者可单独使用。且为了方便理解，此刻暂不引入数据副本相关内容。
[^2]: insert_distributed_one_random_shard参数设置为1时，在分布式表定义中不包含sharding_key的情况下，依然可以允许通过分布式表写入数据，此时会随机选择一个shard写入数据。
[^3]: nginx,haproxy,chproxy,cloud elb等
[^4]: 主要使用到的Atomic
[^5]: 最健壮的和广泛使用的是MergeTree（合并树）引擎及该系列（*MergeTree）引擎
[^6]: sharding_key必须是整型类型的字段或者返回整数类型的表达式，因为它会被用于计算数据应该写入哪个shard
[^7]: 需要依赖zookeeper
<script src="{{ "/assets/js/mermaid.min.js" | relative_url }}"></script>
