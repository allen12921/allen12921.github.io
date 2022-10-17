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
  复制表是ClickHouse实现数据高可用以及提高大规则数据集性能的重要方式，
# 实现
  通过zookeeper保存元数据，记录data parts的变化情况，CH之间直接同步data parts
# 限制
  - 目前只支持MergeTree系列的表（只需在原MergeTree表引擎前加上Replicated关键字，即为对应的复制表引擎）
  - 只支持复制INSERT 和 ALTER语句
  - CREATE, DROP, ATTACH, DETACH and RENAME 都只会在当前执行语句的机器上执行（可以结合ON CLUSTER语句在多台执行）
